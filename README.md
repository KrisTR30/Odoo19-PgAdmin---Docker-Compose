# Odoo 19 con PostgreSQL, PgAdmin y Backup Automático en Docker Compose usando Ubuntu/Ubuntu Sever

Este proyecto despliega un entorno completo de **Odoo 19** utilizando **Docker Compose**, junto con **PostgreSQL**, **PgAdmin4** y un **servicio de backup automático** semanal de la base de datos.

## Descripción del proyecto

El objetivo de este entorno es facilitar la instalación y despliegue de **Odoo 19** de manera rápida, limpia y reproducible mediante contenedores.  
Incluye:
- Base de datos **PostgreSQL 16**
- Panel de administración **PgAdmin4**
- Contenedor de **Odoo 19**
- Tarea de **respaldo automático** semanal (`pg_dump`)

## Estructura del proyecto

odoo-docker <br>
┣ docker-compose.yml <br>
┣ odoo_data/  Datos persistentes de Odoo <br>
┣ postgres_data/  Datos persistentes de PostgreSQL <br>
┣ pgadmin_data/ Configuración persistente de PgAdmin <br>
┣ backups/ Copias de seguridad automáticas <br>
┗ README.md

## Configuración de servicios
🐘 Base de datos (PostgreSQL)
- **Imagen:** `postgres:16`
- **Usuario:** `odoo`
- **Contraseña:** `odoo`
- **Base de datos:** `postgres`
- **Volumen:** `./postgres_data:/var/lib/postgresql/data`
### 🧱 Odoo
- **Imagen:** `odoo:19`
- **Puerto:** `8069`
- **Variables de entorno:**
  - `HOST=db`
  - `USER=odoo`
  - `PASSWORD=odoo`
- **Volumen:** `./odoo_data:/var/lib/odoo`
### 🧰 PgAdmin4
- **Imagen:** `dpage/pgadmin4`
- **Puerto:** `5050`
- **Usuario:** `admin@admin.com`
- **Contraseña:** `admin`
- **Volumen:** `./pgadmin_data:/var/lib/pgadmin`
### 💾 Backup automático
- **Imagen:** `postgres:16`
- **Frecuencia:** Cada 7 días (604800 segundos)
- **Destino:** `./backups`
- **Archivo generado:**  
  `odoo_backup_YYYY-MM-DD_HH-MM.dump`
## 🚀 Instrucciones de uso

1. Clona este repositorio:
   ```bash
   git clone https://github.com/tuusuario/odoo-docker.git
   cd odoo-docker
2. Inicia los contenedores:
   ```bash
   docker-compose up -d
3. Accede a las aplicaciones:
   ```bash
   Odoo: http://<IP_DEL_EQUIPO>:8069
   PgAdmin: http://<IP_DEL_EQUIPO>:5050
   Usuario: admin@admin.com # Cambiar usuario
   Contraseña: admin # Cambiar contraseña

   (Opcional) Verifica los backups automáticos en la carpeta ./backups/

## Notas importantes
Asegúrate de que los puertos 8069 y 5050 estén libres antes de iniciar.
Todos los datos se guardan de forma persistente gracias a los volúmenes de Docker.
Puedes modificar las contraseñas en el archivo docker-compose.yml si lo deseas.
El servicio de backup usa el mismo usuario y contraseña de la base de datos principal (odoo).

# 👤 Autor
Creado con ❤️ por Kris Tello
📧 krisfab.tello.30@gmail.com
🌐 linkedin.com/in/kris-tello
