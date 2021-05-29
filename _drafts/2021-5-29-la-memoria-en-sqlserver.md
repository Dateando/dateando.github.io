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

SQL Server va a ir cogiendo la memoria del sistema segun la va necesitando, pero no la va a liberar aunque no la este utilizando. Pero si el sistema operativo se ve en la necesidad de pedirle a SQL Server que libere memoria porque el la necesita para otros procesos, SQL Server se la ira dando, y cada pagina que libere de su espacio de memoria lo ira llevando al fichero de paginación. Si el parámetro "Lock Pages In Memory" está habilitado, SQL Server no liberará memoria, aunque el sistema operativo se lo esté pidiendo. Por lo general, en los sistemas actuales no es recomendable habilitar "Lock Pages in Memory".

No hay una regla general para configurar el parametros "max server memory", pero una recomendación para un servidor dedicado, es dejar para el sistema operativo 1 GB por cada 4 GB que tengamos de memoria física para servidores con menos de 16 GB,  y 1 GB por cada 8 GB de memoria fisica para sistemas con mas de 16 GB.

Memoria fisica <= 16 GB
1 GB por cada 4 GB de memoria física.
Ejemplo: Servidor de 16 GB: 4 GB para e Sistema Operativo y 12 GB para SQL Server.

Memoria fisica > 16 GB
1 GB por cada 8 GB de memoria física.
Ejemplo: Servidor de 64 GB: 8 GB para e Sistema Operativo y 56 GB para SQL Server.

Ejemplo de como configurar el limite de memoria para SQL Server:
```sql
sp_configure 'max server memory', 4096;
GO
RECONFIGURE;
GO
```

La memoria dedicada a SQL Server se divide en los siguientes componentes:

- Antes de SQL Server 2021: 
  - Buffer Pool (SPA - Single-Page Allocator) [Este espacio es definido mediante los parametros MIN Memory y MAX Memory].
  - MPA (Multi-Page Allocator) + CLR (CLR allocations) + TS (Thread stacks memory) + DWA (Direct allocations)
- Despues de SQL Server 2021: 
  - Buffer Pool (SPA - Single-Page Allocator) + MPA (Multi-Page Allocator) + CLR  (CLR allocations) [Este espacio es definido mediante los parametros MIN Memory y MAX Memory].
  - TS (Thread stacks memory) + DWA (Direct Windows allocations)

"non-Buffer Pool" = CLR + TS + DA + DAW

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

Para saber cuanta memoria tenemos asignada a SQL Server, podemos utilizar la siguiente consulta:

```sql
SELECT c.value, c.value_in_use
FROM sys.configurations c WHERE c.[name] = 'max server memory (MB)'
```

Para conocer el consumo actual de la memoria, podemos utilizar la siguiente consulta:

```sql
SELECT 
  physical_memory_in_use_kb/1024 AS sql_physical_memory_in_use_MB, 
	large_page_allocations_kb/1024 AS sql_large_page_allocations_MB, 
	locked_page_allocations_kb/1024 AS sql_locked_page_allocations_MB,
	virtual_address_space_reserved_kb/1024 AS sql_VAS_reserved_MB, 
	virtual_address_space_committed_kb/1024 AS sql_VAS_committed_MB, 
	virtual_address_space_available_kb/1024 AS sql_VAS_available_MB,
	page_fault_count AS sql_page_fault_count,
	memory_utilization_percentage AS sql_memory_utilization_percentage, 
	process_physical_memory_low AS sql_process_physical_memory_low, 
	process_virtual_memory_low AS sql_process_virtual_memory_low
FROM sys.dm_os_process_memory;  
```
Para saber cuanta memoria del Buffer cache tenemos asignado por cada base de datos, podemos utilizar la siguiente consulta:

```sql
SELECT
  database_id AS DatabaseID,
  DB_NAME(database_id) AS DatabaseName,
  COUNT(file_id) * 8/1024.0 AS BufferSizeInMB
FROM sys.dm_os_buffer_descriptors
GROUP BY DB_NAME(database_id),database_id
ORDER BY BufferSizeInMB DESC
GO 
```

Un muy buen artículo sobre la memoria es el siguiente:
[http://udayarumilli.com/sql-server-memory-usage-sql-server-internals/](http://udayarumilli.com/sql-server-memory-usage-sql-server-internals/)
