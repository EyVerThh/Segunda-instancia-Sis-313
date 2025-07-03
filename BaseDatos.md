
# Configuración de base de datos- MariaDB (192.168.105.13)
## Objetivo del Servidor Maestro

Configurar y gestionar el servidor principal de base de datos `MariaDB` que contendrá la base de datos original (`biblioteca`) y generará los logs binarios que serán replicados hacia el servidor esclavo. Este servidor será responsable de recibir las operaciones de escritura (INSERT, UPDATE, DELETE) y asegurar la disponibilidad de los datos para replicación.

El maestro funcionará como la **fuente única de verdad**, enviando automáticamente todos los cambios registrados a su base de datos al servidor esclavo configurado.

## Paso 1: Configurar el archivo del servidor
Editar el archivo:
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Agregar al final:
```ini
[mysqld]
server-id=1
log-bin=mysql-bin
binlog-do-db=biblioteca
```

## Paso 2: Reiniciar el servicio MariaDB
```bash
sudo systemctl restart mariadb
```

## Paso 3: Crear usuario de replicación
```sql
CREATE USER 'eyverth'@'%' IDENTIFIED BY '1234';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';
FLUSH PRIVILEGES;
```

## Paso 4: Crear base de datos y tabla de ejemplo
```sql
CREATE DATABASE crud_db;
USE crud_db;

CREATE TABLE items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(255) NOT NULL
);
```

```sql
INSERT INTO items (nombre) VALUES ('Primer Dato');
```
```sql
SELECT * FROM items;
```

## Paso 5: Crear un volcado (`.sql`) con información de replicación
```bash
mysqldump -u root --databases crud_db --master-data --single-transaction > crud_db.sql
```

Buscar en el archivo el bloque:
```sql
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=328;
```




