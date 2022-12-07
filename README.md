# QUERYS

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

```mysql
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

Funciones en MySQL:

```mysql
CREATE DATABASE DEMO;
USE DEMO;
CREATE TABLE EJERCICIOS(N1 DECIMAL(5,2) NOT NULL, N2 DECIMAL(5,2) NOT NULL, SIGNO CHAR(1) NOT NULL);
INSERT EJERCICIOS(N1,N2,SIGNO)
VALUES(2.00,3.00,'+'),(2.00,3.00,'-'),(2.00,3.00,'*'),(2.00,3.00,'/');
DELIMITER $$
DROP FUNCTION IF EXISTS CALCULADORA$$
CREATE FUNCTION CALCULADORA(INPUT_N1 DECIMAL(5,2), INPUT_N2 DECIMAL(5,2), INPUT_SIGNO CHAR(1))
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
DECLARE TOTAL DECIMAL(5,2);
IF(INPUT_SIGNO = '+') THEN
SET TOTAL = INPUT_N1 + INPUT_N2;
ELSEIF(INPUT_SIGNO = '-') THEN
SET TOTAL = INPUT_N1 - INPUT_N2;
ELSEIF(INPUT_SIGNO = '*') THEN
SET TOTAL = INPUT_N1 * INPUT_N2;
ELSEIF(INPUT_SIGNO = '/') THEN
SET TOTAL = INPUT_N1 / INPUT_N2;
ELSE
SET TOTAL = NULL;
END IF;
RETURN (TOTAL);
END$$
DELIMITER ;
SELECT N1,SIGNO,N2, CALCULADORA(N1,N2,SIGNO) FROM EJERCICIOS;
```

Conteo de registros en MySQL:

```mysql
CREATE DATABASE migracion;
USE migracion;
CREATE TABLE repatriados(id_repatriados INT,nombres VARCHAR(100));
INSERT INTO repatriados VALUES(1,'RICARDO VALLDARES'),(2,'VERONICA ARIAS'),(3,'JOSE LOPEZ');
CREATE TABLE entrevistas(id_entrevistas INT,id_repatriados INT);
INSERT INTO entrevistas VALUES(1,1),(2,2),(3,2),(4,3),(5,3);
SELECT (SELECT COUNT(*) FROM entrevistas WHERE id_repatriados = r.id_repatriados) AS reincidencias, r.nombres
FROM repatriados r;
```

