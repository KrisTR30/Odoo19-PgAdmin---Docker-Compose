# Odoo19 con PostgreSQL, PgAdmin y Backup Automático en Docker Compose usando Ubuntu/Ubuntu Sever

Este proyecto despliega un entorno completo de **Odoo19** utilizando **Docker Compose**, junto con **PostgreSQL**, **PgAdmin4** y un **servicio de backup automático** semanal de la base de datos.

## Descripción del proyecto

El objetivo de este entorno es facilitar la instalación y despliegue de **Odoo19** de manera rápida, limpia y reproducible mediante contenedores.  
Incluye:
- Base de datos **PostgreSQL16**
- Panel de administración **PgAdmin4**
- Contenedor de **Odoo19**
- Tarea de **respaldo automático** semanal (`pg_dump`)

## 📂Estructura del proyecto

### Odoo19 <br>
┣ 📜docker-compose.yml <br>
┣ 📂odoo_data/  _Datos persistentes de Odoo_ <br>
┣ 📂postgres_data/  _Datos persistentes de PostgreSQL_ <br>
┣ 📂pgadmin_data/  _Configuración persistente de PgAdmin_ <br>
┣ 📂backups/  _Copias de seguridad automáticas_ <br>

## Crear estructura de carpetas y acceder

- Acceder como root <br>
  ```bash
  sudo su
  ```
  
- Crear estructura de carpetas <br>
   ```bash
   mkdir -p ~/odoo19/{postgres_data,odoo_data,pgadmin_data,backups}
   cd ~/odoo19
   ```
   
- Crear archivo docker-compose.yml <br>
  ```bash
  nano docker-compose.yml
  ```

- Contenido del archivo
  ```bash
  services:
  db:
    image: postgres:16
    container_name: odoo19-db
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo  # Cambiar la contraseña
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
    restart: always

  odoo:
    image: odoo:19
    container_name: odoo19
    depends_on:
      - db
    ports:
      - "8069:8069"
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=odoo  # Debe ser la misma que la del servicio db
    volumes:
      - ./odoo_data:/var/lib/odoo
    restart: always

  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    depends_on:
      - db
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com  # Cambiar el correo
      - PGADMIN_DEFAULT_PASSWORD=admin         # Cambiar la contraseña
    ports:
      - "5050:80"
    volumes:
      - ./pgadmin_data:/var/lib/pgadmin
    restart: always

  backup:
    image: postgres:16
    container_name: odoo19-backup
    depends_on:
      - db
    environment:
      - PGPASSWORD=odoo  # Debe ser la misma que la del servicio db
    volumes:
      - ./backups:/backups
    entrypoint: >
  	  /bin/bash -c "
	    echo '0 3 * * 0 root pg_dump -h db -U odoo -Fc postgres > /backups/odoo_backup_$(date +\%Y-\%m-\%d_\%H-\%M).dump && find /backups -type f -mtime +7 -delete' > /etc/cron.d/pg-backup &&
	    chmod 0644 /etc/cron.d/pg-backup &&
	    crontab /etc/cron.d/pg-backup &&
	    cron -f
	  "
    restart: always
  ```
## ⚙️Configuración de servicios

### 🐘 Base de datos (PostgreSQL)
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
- **Archivo generado:**  `odoo_backup_YYYY-MM-DD_HH-MM.dump`
  
## 🚀 Instrucciones de uso
Dar permiso a las capertas de odoo_data y postgres_data para evitar posibles errores
  ```bash
  sudo chmod -R 777 ~/odoo19/odoo_data
  sudo chmod -R 777 ~/odoo19/postgres_data
  ```

Inicia el contenedor:
   ```bash
   docker-compose up -d
   ```

Esto descargará e iniciará los cuatro servicios.
| Servicio   | Descripción                             | Puerto   |
|------------|-----------------------------------------|----------|
| 🐘 DB      | Base de datos PostgreSQL16              | Interno  |
| 🧩 Odoo    | Servidor Odoo19                         | 8069     |
| 🧠 PgAdmin | Panel de administración PostgreSQL      | 5050     |
| 💾 Backup  | Respaldos automáticos semanales         | Interno  |
   
Accede a las aplicaciones:
   ```bash
   - Odoo: http://<IP_DEL_EQUIPO>:8069
   - PgAdmin: http://<IP_DEL_EQUIPO>:5050
   - Usuario: admin@admin.com # Cambiar usuario
   - Contraseña: admin # Cambiar contraseña

   (Opcional) Verifica los backups automáticos en la carpeta ./backups/
  ```
Backups automáticos
- Carpeta: `~/odoo19/backups/`
- Frecuencia: 1 vez por semana (604800 segundos)
- Formato: `odoo_backup_YYYY-MM-DD_HH-MM.dump`
- Persisten aunque reinicies Docker o el servidor.

Listar los Backups
  ```bash
  ls ~/odoo19/backups  
  ```

Restaurar Backups Manualmente:
  ```bash
  docker exec -i odoo19-db pg_restore -U odoo -d postgres < ~/odoo19/backups/odoo_backup_YYYY-MM-DD_HH-MM.dump  
  ```

## Comando Utiles
  ```bash
  # Detener contenedores
  sudo docker compose down

  # Reiniciar contenedores
  sudo docker compose restart

  # Eliminar todo (contenedores, volúmenes e imágenes)
  sudo docker compose down --rmi all -v
  ```
## Solución a posibles Errores
PgAdmin bloqueado o con errores de inicio de sesión
  ```bash
  cd ~/odoo19 # Ingresamos a la carpeta
  sudo docker compose down # Apagamos el contenedor
  sudo rm -rf ./pgadmin_data # Eliminamos la carpeta de pgadmin_data para reiniciar los valores
  sudo mkdir -p ./pgadmin_data # Creamos nuevamente la carpeta pgadmin_data
  sudo chmod -R 777 ./pgadmin_data # Le damos los permisos respectivos nuevamente
  sudo docker compose up -d  # Levantamos el contenedor
  ```

## Notas importantes
- Asegúrate de que los puertos 8069 y 5050 estén libres antes de iniciar. <br>
- Todos los datos se guardan de forma persistente gracias a los volúmenes de Docker. <br>
- Puedes modificar las contraseñas en el archivo docker-compose.yml si lo deseas. <br>
- El servicio de backup usa el mismo usuario y contraseña de la base de datos principal (odoo19). <br>

# 👤 Autor
Creado con ❤️ por Kris Fabian Tello Rodriguez <br>
📧 krisfab.tello.30@gmail.com <br>
🌐 https://www.linkedin.com/in/kris-tello <br>
