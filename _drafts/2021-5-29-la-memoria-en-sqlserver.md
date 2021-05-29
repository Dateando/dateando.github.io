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

Hay dos tipos de memoria en nuestros sistemas Windows, la **memoria física** y la **memoria virtual**. La memoria fisica son los GBs que tenemos en los chips de las placas de memoria de nuestro máquina y la virtual es la física mas el fichero de paginación que suele estar en nuestra unidad C: (C:\pagefile.sys). Esta memoria virtual es conocida como Virtual Address Space (VAS).

SQL Server va a ir cogiendo la memoria del sistema segun la va necesitando, pero no la va a liberar aunque no la este utilizando. Pero si el sistema operativo se ve en la necesidad de pedirle a SQL Server que libere memoria porque el la necesita para otros procesos, SQL Server se la ira dando, y cada pagina que libere de su espacio de memoria lo ira llevando al fichero de paginación. Si el parámetro "Lock Pages In Memory" está habilitado, SQL Server no liberará memoria, aunque el sistema operativo se lo esté pidiendo.

La memoria dedicada a SQL Server se divide en los siguientes componentes:

- Antes de SQL Server 2021: 
  - Buffel Pool (SPA) (Este espacio es definido mediante los parametros MIN Memory y MAX Memory).
  - MPA + CLR + TS + DA 
- Despues de SQL Server 2021: 
  - Buffel Pool (SPA) + MPA + CLR (Este espacio es definido mediante los parametros MIN Memory y MAX Memory).
  - TS + DA 

El Buffer Pool (SPA) se compone de los siguientes elementos:

- Plan Cache
- Log Cache
- Buffer Cache
- System Data Structures
- User Connections

El Virtual Address Space (VAS) se compone de los siguientes elementos:

- Libraries DLL's
- SQL Server code
- Extended Procs
- COM Objects
- Open Data Services
- Linked Servers
- Distributed QUeries
- Multipage Allocation


```sql
USE master
GO
CREATE MASTER KEY ENCRYPTION BY PASSWORD='miContraseniaSecret4'
GO
```





Un muy buen artículo sobre la memoria es el siguiente:
[http://udayarumilli.com/sql-server-memory-usage-sql-server-internals/](http://udayarumilli.com/sql-server-memory-usage-sql-server-internals/)
