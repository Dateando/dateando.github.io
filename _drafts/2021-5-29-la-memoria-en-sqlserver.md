---
layout: post
title: "SQL SERVER  La memoria"
permalink: /SQLSERVER/memoria/
published-on: 1 June 2021
date: 2021-06-01
tag: sqlserver, databases,sessions, memoria, buffer cache, buffer pool
keywords: sqlserver, databases,sessions, memoria, buffer cache, buffer pool
categories: SQLSERVER MEMORY
---

Hay varias formas de encriptar los datos que almacena SQL Server, y una de las mas conocidas y menos intrusivas en las aplicaciones es TDE o Transparent Data Encryption. Como su nombre indica, la encriptación de los datos es transparente, pero transparente para la aplicación porque ella ni se entera. 

**PRIMER PASO:**

Creamos en la base de datos de sistema *master* una CLAVE MAESTRA (MASTER KEY ENCRYPTION) que se genera con una contraseña que debemos definir y guardar:
```sql
USE master
GO
CREATE MASTER KEY ENCRYPTION BY PASSWORD='miContraseniaSecret4'
GO
```

- El proceso de encriptacion sobre una base de datos se realiza en segundo plano (background), ejecutandose con baja prioridad sin sobrecargar el sistema.
- Si TDE lo implementamos sobre un ALWAYS ON, debemos ejecutar los dos primeros pasos en todas las replicas, ya que ALWAYS ON no replica las bases de datos de sistema.
- En el momento en que se encripta la primera base de datos de usuario, automenticamente se encripta la base de datos de sistema `TEMPDB`.
- Los ficheros de `FILESTREAM` no se pueden ecriptar con TDE.
- Deshabilitar TDE sobre una base de datos encriptada, es tan sencillo como habilitarla:
```sql
ALTER DATABASE MiBaseDeDatos SET ENCRYPTION OFF
GO
```



Podemos consultar y profundizar sobre TDE Transparent Data Encryption, podemos acudir a la documentación oficial:
[https://docs.microsoft.com/es-es/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-ver15](https://docs.microsoft.com/es-es/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-ver15)
