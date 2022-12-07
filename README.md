Conexion remota a MariaDB:

```sh
mysql --host=192.168.x.y --user=root --password=mios --port=3306
```

Crear Backup SQL:

```sh
mysqldump --host=192.168.x.y --user=root --password=mios --port=3306 base_de_datos > backup.sql
```

Restaurar Backup SQL:

```sh
mysql --host=192.168.x.y --user=root --password=mios --port=3306 base_de_datos < backup.sql
```

MOVER TABLAS PARA PSEUDORENOMBRAR BASES:

```sh
for table in `mysql --user=root --password=mios -s -N -e "use OLDDB;show tables from OLDDB;"`; do mysql --user=root --password=mios -s -N -e "use OLDDB;rename table OLDDB.$table to NEWDB.$table;"; done;
```

Unir registros en Tabla de consulta:

```mysql
SELECT uniones.fila1, uniones.fila2 FROM
((SELECT 'ALUMNAS' AS fila1, '60' AS fila2)
UNION ALL
(SELECT 'ALUMNOS' AS fila1, '53' AS fila2)) AS uniones
```

Condicion en consulta:

```mysql
SELECT
CASE
WHEN (edad) BETWEEN 0 AND 15 THEN 'Niñ@'
WHEN (edad) BETWEEN 16 AND 18 THEN 'Adolecente'
WHEN (edad) BETWEEN 19 AND 70 THEN 'Adult@'
WHEN (edad) BETWEEN 71 AND 200 THEN 'Ancian@'
ELSE 'Indeterminado'
END AS clasificaicon;
```

Triger en MySQL:

```mysql
CREATE DATABASE demodetriger;
USE demodetriger;
CREATE TABLE IF NOT EXISTS principal(id int(11) NOT NULL AUTO_INCREMENT, nombres varchar(200) NOT NULL, PRIMARY KEY (id));
CREATE TABLE IF NOT EXISTS papelera(id int(11) NULL, nombres varchar(200) NULL);
delimiter |
CREATE TRIGGER borrador BEFORE delete ON principal
FOR EACH ROW
BEGIN
INSERT INTO papelera SET id = OLD.id, nombres = OLD.nombres;
END;
|
delimiter ;
```

TryCatch en MariaDB:

```mariadb
DELIMITER $$
CREATE PROCEDURE simulatrycatch( consulta TEXT )
BEGIN
-- Obtenemos la Excepcion en caso ocurra
DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
BEGIN
GET DIAGNOSTICS CONDITION 1 @estado = RETURNED_SQLSTATE, @mensaje = MESSAGE_TEXT;
SELECT @estado, @mensaje;
END;
-- query a ejecutar
PREPARE myquery FROM consulta;
EXECUTE myquery;
END;
$$
DELIMITER ;
CALL simulatrycatch('select now()');
CALL simulatrycatch('insert abc(d,b) values(2,3)');
```
