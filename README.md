# Guía de Instalación Paso a Paso de una Torre AWX

> Esta guía ha sido elaborada por mí y está disponible bajo uso libre.  
>  
> ✨ Siéntete libre de proponer cambios, sugerencias o mejoras para enriquecerla.


## 📋 Tabla de Contenidos

- [Requisitos Previos](#-requisitos-previos)  
- [Preparación del Servidor](#preparacion-del-servidor)  
- [Instalación de AWX](#-instalacion-de-awx)  
- [Configuración Inicial](#-configuracion-inicial)  
- [Acceso a la Interfaz Web](#-acceso-a-la-interfaz-web)
- [Creación de Servicios](#-creacion-de-servicios)
- [Tareas Post-Instalación](#tareas-post-instalacion)  
- [Solución de Problemas](#solución-de-problemas)   

---

## 🔧 Requisitos Previos

1. **Sistema Operativo**: Ubuntu 20.04 LTS (o similar)  
2. **Usuario sudo** con permisos de administrador.  
3. **Conexión a Internet** en la máquina destino.  
4. **Puertos** abiertos:  
   - HTTP: `80`  
   - HTTPS: `443`  
   - PostgreSQL (opcional): `5432`  

---

## Preparacion del Servidor

```bash
# 1. Actualizar paquetes
sudo apt update && sudo apt upgrade -y

# 2. Instalar dependencias
sudo apt install -y git docker.io docker-compose ansible

# 3. Habilitar y arrancar Docker
sudo systemctl enable --now docker

# 4. Agregar tu usuario al grupo 'docker' (cierra sesión y vuelve a entrar para aplicar)
sudo usermod -aG docker $USER
```

---

## 🚀 Instalacion de AWX

```bash
# 1. Clonar el repositorio oficial de AWX
git clone https://github.com/ansible/awx.git
cd awx/installer

# 2. Crear copia de seguridad del inventario por defecto
cp inventory inventory.backup

# 3. Configurar el inventario de Ansible
cat > inventory <<EOF
localhost ansible_connection=local
# Dirección SMTP (opcional)
# smtp_server=mail.midominio.com
# smtp_port=587
# admin_email=admin@midominio.com
# Contraseña del administrador AWX
admin_password=MiPasswordSeguro
# Puerto HTTP de AWX
awx_official=true
# Cambiar si no usas HTTP en el puerto 80
project_data_dir="/var/lib/awx/projects"
EOF

# 4. Ejecutar el playbook de instalación
ansible-playbook -i inventory install.yml
```

---

## 🔑 Configuracion Inicial

1. **Verificar servicios Docker**  
   ```bash
   docker ps
   ```
   Debes ver contenedores como `awx_web`, `awx_postgres`, `awx_redis`, etc.

2. **Configurar credenciales de Ansible**  
   - Usuario: `admin`  
   - Contraseña: la establecida en `inventory` (`MiPasswordSeguro`)

---

## 🌐 Acceso a la Interfaz Web

Abre tu navegador y navega a:

```
http://<IP_O_HOSTNAME>
```

Inicia sesión con las credenciales indicadas arriba.

---

## 📝 Creacion de servicios

Cree unos servicios para cada uno de los docker para que cuando la máquina que ejecuta AWX se encienda se inicien los docker.

Crea estos servicios en local (da igual la carpeta ya que el archivo cuando se ejecuta lo mete en la carpeta necesaria)  para cada uno de los docker cambiando todo lo que contenga cosas de `awx_web` por `awx_postgres` o `awx_redis`

Este es el servicio para el docker `awx_web`:

```
  [Unit]
    Description=AWX Web Docker container
    Requires=docker.service
    After=docker.service

  [Service]
    Restart=always
    ExecStart=/usr/bin/docker start -a awx_web
    ExecStop=/usr/bin/docker stop -t 2 awx_web

  [Install]
    WantedBy=default.target
    sudo cp docker-awx_web.service /etc/systemd/system/
    sudo systemctl enable docker-awx_web.service

 ```

Ejecutalo y cuando se inicien las máquinas se encenderan los dockers de AWX

---

## Tareas Post-Instalacion

- Cambiar la contraseña del usuario `admin` desde la interfaz.  
- Añadir usuarios y equipos según tus necesidades.  
- Configurar tus credenciales y proyectos de Ansible en la sección **_Resources → Credentials_** y **_Projects_**.  
- Crear **_Templates_** de trabajo para ejecutar playbooks.

---

## Solución de Problemas

- **Contenedores no arrancan**  
  ```bash
  docker-compose logs
  ```
  Revisa los logs de los servicios con errores.

- **Error de permisos en `/var/lib/awx`**  
  ```bash
  sudo chown -R 1000:1000 /var/lib/awx
  ```

- **Reinstalar AWX**  
  ```bash
  ansible-playbook -i inventory uninstall.yml
  ansible-playbook -i inventory install.yml
  ```

