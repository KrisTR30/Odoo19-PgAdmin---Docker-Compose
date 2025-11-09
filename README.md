# Odoo 19 con PostgreSQL, PgAdmin y Backup Automático en Docker

Este proyecto despliega un entorno completo de **Odoo 19** utilizando **Docker Compose**, junto con **PostgreSQL**, **PgAdmin4** y un **servicio de backup automático** semanal de la base de datos.

# Descripción del proyecto

El objetivo de este entorno es facilitar la instalación y despliegue de **Odoo 19** de manera rápida, limpia y reproducible mediante contenedores.  
Incluye:
- Base de datos **PostgreSQL 16**
- Panel de administración **PgAdmin4**
- Contenedor de **Odoo 19**
- Tarea de **respaldo automático** semanal (`pg_dump`)
