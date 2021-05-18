---
layout: post
title: SQL SERVER  TDE - transparent data encryption
published-on: 18 May 2021
tag: sqlserver, databases,sessions, tde, transparent data encryption
keywords: sqlserver, databases,sessions, tde, transparent data encryption 
---

Hay varias formas de encriptar los datos que almacena SQL Server, y una de las mas conocidas y menos intrusivas en las aplicaciones es TDE o Transparent Data Encryption. Como su nombre indica, la encriptación de los datos es transparente, transparente para la aplicación porque ella ni se entera. 

Esta encriptacion de datos es a nivel de base de datos, y sobre datos frios, es decir, aquellos datos que estan en disco. En el momento que un usuario o aplicación hace una consulta sobre unos datos y estos pasan del disco a memoria y luego viajan por la red hasta el proceso solicitante, los datos dejan de estar encriptados.
Pero nos estamos asegurando que si alguien se hace con los ficheros de datos y log de transacciones de la base de datos encriptada, no podran ser utilizados en otra instancia. Y esto mismo ocurre con los backups realizados sobre una base de datos encriptada con TDE, no es posible restaurarlos en otra instancia.

El proceso para habilitar TDE - Transparent Data Encription en una instancia y luego en cada base de datos que se desee es muy sencillo, y se hace en cuatro pasos, que requieren hacerlo por T-SQL, ya que SQL Server Manageent Studio no ofrece un wizard para poder realizarlo da una manera aun mas sencilla.

**PRIMER PASO:**

Creamos en la base de datos de sistema *master* una CLAVE MAESTRA (MASTER KEY ENCRYPTION) que se genera con una contraseña que debemos definir y guardar:
~~~
USE master
GO
CREATE MASTER KEY ENCRYPTION BY PASSWORD='miContraseniaSecret4'
GO
~~~

**SEGUNDO PASO:**

Creamos un certificado, que será encriptado mediante la CLAVE MAESTRA (MASTER KEY) creada en el paso uno:
~~~
USE master
GO
CREATE CERTIFICATE cerInstanciaDateando01
WITH SUBJECT='CERTIFICADO DATEANDO 01'
GO
~~~

**TERCER PASO:**

Creamos un CLAVE DE CIFRADO DE BASE DE DATOS (DATABASE ENCRYPTION KEY) sobre la base de datos que queremos encriptar, con el algoritmo que elijamos, y utilizando el certificado creado anteriormente para protegerla:

~~~
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256 ENCRYPTION BY SERVER CERTIFICATE cerInstanciaDateando01
GO
~~~

**CUARTO PASO:**

Habilitamos la encriptación en la base de datos de usuario donde hemos creado el CLAVE DE CIFRADO DE BASE DE DATOS (DATABASE ENCRYPTION KEY):

~~~
ALTER DATABASE MiBaseDeDatos SET ENCRYPTION ON
GO
~~~

Ahora los ficheros de nuestra base de datos están protegidos, y los siguientes backups que se se realicen sobre ellos.

Puntos a tener en cuenta:

- Es muy importante hacer un backup del certificado que hemos creado en la instancia y guardarlo en un lugar seguro. Lo podemos hacer de la siguiente forma:
~~~
BACKUP CERTIFICATE cerInstanciaDateando01
TO FILE='C:\MIPATH\dd_mm_aaaa-cerInstanciaDateando01_backup'
WITH PRIVATE KEY
(
  FILE='cerInstanciaDateando01PRIVADO'
  ENCRYPTION BY PASSWORD='miContraseniaSecret4'
)
GO
~~~
- El proceso de encriptacion sobre una base de datos se realiza en segundo plano (background), ejecutandose con baja prioridad sin sobrecargar el sistema.
- Si TDE lo implementamos sobre cluster de ALWAYS ON, debemos ejecutar los dos primeros pasos en todos las replicas, ya que ALWAYS ON no replica las bases de datos de sistema.
- En el momento en que se encripta la primera base de datos de usuario, automenticamente se encripta la base de datos de sistema `TEMPDB`.
- Los ficheros de `FILESTREAM` no se pueden ecriptar con TDE.
- Dehabilitar TDE sobre una base de datos encriptada, es tan sencillo como habilitarla:
 ~~~
ALTER DATABASE MiBaseDeDatos SET ENCRYPTION OFF
GO
~~~


Podemos consultar y profundizar sobre TDE Transparent Data Encryption, podemos acudir a la documentación oficial:
https://docs.microsoft.com/es-es/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-ver15

