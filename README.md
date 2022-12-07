Conexion remota a MariaDB:

```mysql
mysql --host=192.168.x.y --user=root --password=mios --port=3306
```

Crear Backup SQL:

mysqldump --host=192.168.x.y --user=root --password=mios --port=3306 base_de_datos > backup.sql

Restaurar Backup SQL:

mysql --host=192.168.x.y --user=root --password=mios --port=3306 base_de_datos < backup.sql

MOVER TABLAS PARA PSEUDORENOMBRAR BASES:

for table in `mysql --user=root --password=mios -s -N -e "use OLDDB;show tables from OLDDB;"`; do mysql --user=root --password=mios -s -N -e "use OLDDB;rename table OLDDB.$table to NEWDB.$table;"; done;
