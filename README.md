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



Puntero en MySQL:

```mysql
DELIMITER $$

USE `retornados`$$

DROP PROCEDURE IF EXISTS `valesyrecibos`$$

CREATE DEFINER=`migracion`@`%` PROCEDURE `valesyrecibos`( fechainicio DATE, fechafin DATE)
BEGIN
	DECLARE cidusuario INT(11);
	DECLARE idusuario1 INT(11);
	DECLARE idusuario2 INT(11);
	DECLARE idusuario3 INT(11);
	DECLARE idusuario4 INT(11);
	DECLARE idusuario5 INT(11);
	DECLARE idusuario6 INT(11);
	
	DECLARE nombre1 VARCHAR(200);
	DECLARE nombre2 VARCHAR(200);
	DECLARE nombre3 VARCHAR(200);
	DECLARE nombre4 VARCHAR(200);
	DECLARE nombre5 VARCHAR(200);
	DECLARE nombre6 VARCHAR(200);
	
	DECLARE finished INTEGER DEFAULT 0;
	DECLARE sqlstring VARCHAR(4000);
	
	
	-- CREAR CONSULTA DEL PUNTERO
	DECLARE puntero 
		CURSOR FOR 
			SELECT DISTINCT u.idUsuario
			FROM retornados.recibo_dinero e
			INNER JOIN integrado.usuario u ON e.entregadoPor=u.idUsuario
			INNER JOIN retornados.fondo_retornados f ON f.IdFondo=e.idFondo
			WHERE e.fecha >=fechainicio AND e.fecha < DATE_ADD(fechafin, INTERVAL 1 DAY) AND e.entregado=1
			GROUP BY e.fecha, u.idUsuario
			ORDER BY e.fecha, u.idUsuario ASC;

	-- DECLARA FIN Y CONTINUACION
	DECLARE CONTINUE HANDLER 
        FOR NOT FOUND SET finished = 1;

	OPEN puntero;
	getIds: LOOP
		FETCH puntero INTO cidusuario;
		-- select cidusuario;
		IF finished = 1 THEN 
			LEAVE getIds;
		END IF;
		IF idusuario1 IS NULL THEN
			SELECT cidusuario INTO idusuario1;
			SELECT CONCAT(IFNULL(u.primerNombre,''),' ',IFNULL(u.segundoNombre,''),' ',IFNULL(u.primerApellido,''),' ',IFNULL(u.segundoApellido,'')) 
			INTO nombre1
			FROM integrado.usuario u WHERE u.idUsuario=idusuario1;
		ELSEIF idusuario2 IS NULL THEN
			SELECT cidusuario INTO idusuario2;
			SELECT CONCAT(IFNULL(u.primerNombre,''),' ',IFNULL(u.segundoNombre,''),' ',IFNULL(u.primerApellido,''),' ',IFNULL(u.segundoApellido,'')) 
			INTO nombre2
			FROM integrado.usuario u WHERE u.idUsuario=idusuario2;
		ELSEIF idusuario3 IS NULL THEN
			SELECT cidusuario INTO idusuario3;
			SELECT CONCAT(IFNULL(u.primerNombre,''),' ',IFNULL(u.segundoNombre,''),' ',IFNULL(u.primerApellido,''),' ',IFNULL(u.segundoApellido,'')) 
			INTO nombre3
			FROM integrado.usuario u WHERE u.idUsuario=idusuario3;
		ELSEIF idusuario4 IS NULL THEN
			SELECT cidusuario INTO idusuario4;
			SELECT CONCAT(IFNULL(u.primerNombre,''),' ',IFNULL(u.segundoNombre,''),' ',IFNULL(u.primerApellido,''),' ',IFNULL(u.segundoApellido,'')) 
			INTO nombre4
			FROM integrado.usuario u WHERE u.idUsuario=idusuario4;
		ELSEIF idusuario5 IS NULL THEN
			SELECT cidusuario INTO idusuario5;
			SELECT CONCAT(IFNULL(u.primerNombre,''),' ',IFNULL(u.segundoNombre,''),' ',IFNULL(u.primerApellido,''),' ',IFNULL(u.segundoApellido,'')) 
			INTO nombre5
			FROM integrado.usuario u WHERE u.idUsuario=idusuario5;
		ELSE
			SELECT cidusuario INTO idusuario6;
			SELECT CONCAT(IFNULL(u.primerNombre,''),' ',IFNULL(u.segundoNombre,''),' ',IFNULL(u.primerApellido,''),' ',IFNULL(u.segundoApellido,'')) 
			INTO nombre6
			FROM integrado.usuario u WHERE u.idUsuario=idusuario6;
		END IF;
	END LOOP getIds;
	CLOSE puntero;
	
	-- SELECT idusuario1,idusuario2,idusuario3,idusuario4,idusuario5;
	
	-- set idusuario5 = null;
	
	IF idusuario1 IS NULL THEN
		
		SELECT CONCAT("SELECT now();") 
		INTO sqlstring;
		
	ELSEIF idusuario2 IS NULL THEN
	
		SELECT CONCAT("SELECT  
			f.NFondo as 'N° Vales', 
			f.MontoAsignado 'Monto del vale', 
			'$5.00' as 'Valor del recibo', 
			ROUND(f.MontoAsignado/5, 0) as 'Recibos disponibles', 
			DATE_FORMAT(e.fecha,'%d/%m/%Y') as 'Fecha', 
			
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario1," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre1,"',
			
			ROUND((f.MontoAsignado-MontoExistencia)/5,0) as 'Total recibos entregados', 
						
			ROUND(f.MontoExistencia/5, 0) as 'Recibos sobrantes', 
			ROUND(f.MontoExistencia, 0) as 'Saldo restante'
		FROM retornados.recibo_dinero e INNER JOIN integrado.usuario u ON e.entregadoPor=u.idUsuario INNER JOIN retornados.fondo_retornados f ON f.IdFondo=e.idFondo 
		WHERE e.fecha >='",fechainicio,"' AND e.fecha < DATE_ADD('",fechafin,"', INTERVAL 1 DAY) AND e.entregado=1 
		GROUP BY f.IdFondo, e.fecha 
		ORDER BY e.fecha ASC") 
		INTO sqlstring;
		
		PREPARE stmt FROM sqlstring;
		EXECUTE stmt;
		DEALLOCATE PREPARE stmt;
	
	ELSEIF idusuario3 IS NULL THEN
		
		SELECT CONCAT("SELECT  
			f.NFondo as 'N° Vales', 
			f.MontoAsignado 'Monto del vale', 
			'$5.00' as 'Valor del recibo', 
			ROUND(f.MontoAsignado/5, 0) as 'Recibos disponibles', 
			DATE_FORMAT(e.fecha,'%d/%m/%Y') as 'Fecha', 
			
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario1," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre1,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario2," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre2,"',
			
			ROUND((f.MontoAsignado-MontoExistencia)/5,0) as 'Total recibos entregados', 
			
			ROUND(f.MontoExistencia/5, 0) as 'Recibos sobrantes', 
			ROUND(f.MontoExistencia, 0) as 'Saldo restante'
		FROM retornados.recibo_dinero e INNER JOIN integrado.usuario u ON e.entregadoPor=u.idUsuario INNER JOIN retornados.fondo_retornados f ON f.IdFondo=e.idFondo 
		WHERE e.fecha >='",fechainicio,"' AND e.fecha < DATE_ADD('",fechafin,"', INTERVAL 1 DAY) AND e.entregado=1 
		GROUP BY f.IdFondo, e.fecha 
		ORDER BY e.fecha ASC") 
		INTO sqlstring;
		
		PREPARE stmt FROM sqlstring;
		EXECUTE stmt;
		DEALLOCATE PREPARE stmt;
	
	ELSEIF idusuario4 IS NULL THEN
	
		SELECT CONCAT("SELECT  
			f.NFondo as 'N° Vales', 
			f.MontoAsignado 'Monto del vale', 
			'$5.00' as 'Valor del recibo', 
			ROUND(f.MontoAsignado/5, 0) as 'Recibos disponibles', 
			DATE_FORMAT(e.fecha,'%d/%m/%Y') as 'Fecha', 
			
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario1," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre1,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario2," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre2,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario3," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre3,"',
			

				
			ROUND((f.MontoAsignado-MontoExistencia)/5,0) as 'Total recibos entregados', 
						
			ROUND(f.MontoExistencia/5, 0) as 'Recibos sobrantes', 
			ROUND(f.MontoExistencia, 0) as 'Saldo restante'
		FROM retornados.recibo_dinero e INNER JOIN integrado.usuario u ON e.entregadoPor=u.idUsuario INNER JOIN retornados.fondo_retornados f ON f.IdFondo=e.idFondo 
		WHERE e.fecha >='",fechainicio,"' AND e.fecha < DATE_ADD('",fechafin,"', INTERVAL 1 DAY) AND e.entregado=1 
		GROUP BY f.IdFondo, e.fecha 
		ORDER BY e.fecha ASC") 
		INTO sqlstring;
		
		PREPARE stmt FROM sqlstring;
		EXECUTE stmt;
		DEALLOCATE PREPARE stmt;
	
	ELSEIF idusuario5 IS NULL THEN
	
		SELECT CONCAT("SELECT  
			f.NFondo as 'N° Vales', 
			f.MontoAsignado 'Monto del vale', 
			'$5.00' as 'Valor del recibo', 
			ROUND(f.MontoAsignado/5, 0) as 'Recibos disponibles', 
			DATE_FORMAT(e.fecha,'%d/%m/%Y') as 'Fecha', 
			
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario1," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre1,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario2," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre2,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario3," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre3,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario4," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre4,"',
			
			ROUND((f.MontoAsignado-MontoExistencia)/5,0) as 'Total recibos entregados', 
			
			ROUND(f.MontoExistencia/5, 0) as 'Recibos sobrantes', 
			ROUND(f.MontoExistencia, 0) as 'Saldo restante'
		FROM retornados.recibo_dinero e INNER JOIN integrado.usuario u ON e.entregadoPor=u.idUsuario INNER JOIN retornados.fondo_retornados f ON f.IdFondo=e.idFondo 
		WHERE e.fecha >='",fechainicio,"' AND e.fecha < DATE_ADD('",fechafin,"', INTERVAL 1 DAY) AND e.entregado=1 
		GROUP BY f.IdFondo, e.fecha 
		ORDER BY e.fecha ASC") 
		INTO sqlstring;
		
		PREPARE stmt FROM sqlstring;
		EXECUTE stmt;
		DEALLOCATE PREPARE stmt;

	ELSEIF idusuario6 IS NULL THEN

		SELECT CONCAT("SELECT  
			f.NFondo as 'N° Vales', 
			f.MontoAsignado 'Monto del vale', 
			'$5.00' as 'Valor del recibo', 
			ROUND(f.MontoAsignado/5, 0) as 'Recibos disponibles', 
			DATE_FORMAT(e.fecha,'%d/%m/%Y') as 'Fecha', 
			
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario1," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre1,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario2," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre2,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario3," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre3,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario4," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre4,"',
			(SELECT COUNT(ee.idRecibo) FROM retornados.recibo_dinero ee WHERE ee.entregado=1 AND ee.entregadoPor=",idusuario5," AND ee.fecha=e.fecha AND ee.idFondo=e.idFondo) AS '",nombre5,"',
			
			ROUND((f.MontoAsignado-MontoExistencia)/5,0) as 'Total recibos entregados', 
						
			ROUND(f.MontoExistencia/5, 0) as 'Recibos sobrantes', 
			ROUND(f.MontoExistencia, 0) as 'Saldo restante'
		FROM retornados.recibo_dinero e INNER JOIN integrado.usuario u ON e.entregadoPor=u.idUsuario INNER JOIN retornados.fondo_retornados f ON f.IdFondo=e.idFondo 
		WHERE e.fecha >='",fechainicio,"' AND e.fecha < DATE_ADD('",fechafin,"', INTERVAL 1 DAY) AND e.entregado=1 
		GROUP BY f.IdFondo, e.fecha 
		ORDER BY e.fecha ASC") 
		INTO sqlstring;
		
		PREPARE stmt FROM sqlstring;
		EXECUTE stmt;
		DEALLOCATE PREPARE stmt;
		
	ELSE
	
		SELECT CONCAT("SELECT now();") 
		INTO sqlstring;
		
	END IF;
	
END$$
DELIMITER ;
```
