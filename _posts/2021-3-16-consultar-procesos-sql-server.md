---
layout: post
title: SQL Server  Monitorizar los procesos mediante DMV
published-on: 16 March 2021
tag: sqlserver, databases,sessions, process, dmv, dm_exec_requests, dm_exec_connections, dm_exec_sessions
keywords: sqlserver, databases,sessions, process, dmv, dm_exec_requests, dm_exec_connections, dm_exec_sessions 
---

Vamos a ver que vistas podemos utilizar para monitorizar los procesos en ejecución. Tenemos las siguientes:

*  **master.dbo.sysprocesses** *(deprecated)*
*  **sys.dm_exec_sessions**
*  **sys.dm_exec_connection+s**
*  **sys.dm_exec_request**  *(disponble desde SQLServer 2008)*


Estas vistas, que no tablas, nos permiten ver los peticiones de usuario y de sistema que están en ejecución o han sucedido recientemente.
Los procesos de sistema nacen en la misma máquina donde tenemos la instancia de SQL Server, son generados por el propio motor de SQL Server. Los procesos de sistema son procesos internos que no deberian generar problemas en el rendimiento de la instancia por lo que se suelen descartar a la hora de analizar el entorno.
Los procesos de usuario pueden haber sido lanzados desde la maquina local o desde otras maquinas.
Todo proceso lleva asignado un spid. Antiguamente se consideraba que los procesos que tenian un spid igual o menor a 50 eran los únicos de sistema, pero no es asi, puede haber procesos de sistema con spid superior al 50. 

Vamos a ver las DMV mas detalladamente

## master.dbo.sysprocesses 

Esta vista viene de la version de SQL Server 2000 y está marcada como deprecated. Deprecated en informatica significa que aunque sigue estando disponible, no quieren que te encariñes con la funcionalidad ya que en proximas versions desaparecera.

Aunque esta deprecated, es muy util porque te permite ver mucha información interesante sin necesidad de hacer muchas JOINS con otras.

Columnas interesantes de esta vista:

*  *dbid*      Nombre de la base de datos donde se ha conectado el proceso [ DB_NAME(dbid) ]
*  *loginame*  Login que ha iniciado el proceso
*  *blocked*   spid del proceso que esta bloqueando a este proceso. Muy importante, ya que el numero que aparezca aqui impide avanzar al proceso y es candidato a analizarlo.
*  *status*    Estados por los que pasa el proceso. Posibles valores: dormant, running, background, background, rollback, pending, runnable, spinloop, suspended.
*  *hostname*  Servidor/maquina desde donde se ha creado la conexión para iniciar el proceso en SQL Server.


## sys.dm_exec_session

Toda conexión que llega a SQL Server se gestiona como una sesion, tanto procesos de sistema como de usuario. Esta vista nos devolvera todas las sesiones, y cada sesion tiene asociado un spid único.
También ofrece información interesante sobre la conexión que se ha establecido para abrir la nueva sesion y datos acumulados de la sesión. Es importante aclarar que una sesión esta formada por una o mas request/peticiones, y la sesión se mantiene desde que se abre la conexión hasta que se cierra.

Poemos destacar las siguientes columnas:

- *login_name*    Login que ha iniciado la sesión.
- *host_name*     Servidor/maquina desde donde se ha creado la conexión para iniciar el proceso en SQL Server. Si es un proceso interno sera NULL.
- *cpu_time*      Tiempo acumulado en milisegundos que lleva ejecutando la cpu en la sesión.
- *reads*         Número acumulado de paginas leidas en la sesion actual. Una pagina son 8Kbytes.
- *writes*        Número acumulado de paginas escritas en la sesion actual.  
- *program_name*  Aplicación responsable de iniciar la sesión. Si es un proceso interno sera NULL.
- *transaction_isolation_level* Nivel de aislamiento definido para la sesión.

Las columnas que muestran metricas del tipo cpu_time, reads, writes, etc, son valores acumulados de todas las request que se han ejecutado desde dentro de esa sesión.

## sys.dm_exec_connections

Esta vista nos muestra aquellas sesiones que sean de usuario, las que sean de internas de sistema no apareceran. Por tanto, esta vista es normalmente utilizada para ayudar a filtrar en otras vistas que solo se muestren las sesiones y request de usuario mediante un `INNER JOIN` con esta vista por el campo id_session. Veremos que hay consultas que filtran por `id_session > 50` pero la forma mas correcta es mediante el join a esta vista. Tambien es recomendable quitar en el filtro la sesion en la que estamos lanzando nuestra consulta de monitorización de la siguiente forma `id_session <> @@SPID`.

## sys.dm_exec_request
Esta ultima es la mas interesante  porque solo ofrece las request/peticiones que están activas. El ambito de una request son las sesiones, todas las request se ejecutan dentro de una sesion. Las columnas a destacar son:

- *database_id*      Nombre de la base de datos donde se ha conectado el proceso `[ DB_NAME(dbid) ]`
- *blocking_session_id*   spid del proceso que esta bloqueando a este proceso. Muy importante, ya que el numero que aparezca aqui impide avanzar al proceso y es candidato a analizarlo.
- *status*    Estados por los que pasa el proceso. Posibles valores: dormant, running, background, background, rollback, pending, runnable, spinloop, suspended.
- *command*   Tipo de comando que se ejecuta o se ejecuto en la ultima request. Ejemplos: SELECT, INSERT, UPDATE, DELETE, BACKUP LOG, BACKUP DATABASE, DBCC...
- *percent_complete* Muestra el % que lleva alcanzado cuando la request ejecuta alguno de los siguientes comandos:

    - ALTER INDEX REORGANIZE
    - AUTO_SHRINK option with ALTER DATABASE
    - BACKUP DATABASE
    - DBCC CHECKDB
    - DBCC CHECKFILEGROUP 
    - DBCC CHECKTABLE
    - DBCC INDEXDEFRAG
    - DBCC SHRINKDATABASE
    - DBCC SHRINKFILE
    - RECOVERY
    - RESTORE DATABASE
    - ROLLBACK 
    - TDE ENCRYPTION

- *wait_type*  Tipo de espera por la que espera la request/petición en este momento actual. Si vale NULL es que no esta esperando por nada, estará en `status` con valor a `running`. 
- *wait_time*  Tiempo en milisegundos que lleva esperando por los diferentes recursos (cpu, memoria, indices, tablas, etc) 
- *last_wait_type*  Último tipo de espera de la request/petición.
- *wait_resource*  Si wait_type es distinto de NULL y esta esperando por un recurso tipo tabla, índice, etc aparecerá en esta columna.


Para consultar cualquiera de estas vista, se requiere permisos VIEW SERVER STATE
 
~~~ T-SQL:

GRANT VIEW SERVER STATE TO [loginMio] ~~~


La siguiente consulta muestra una foto muy completa de la situación activa en la instancia analizada mediante las DMV vistas:


~~~ SELECT  des.session_id,
        des.status,
        des.login_name,
        des.[HOST_NAME],
        der.blocking_session_id,
        DB_NAME(der.database_id) as database_name,
        der.command,
        des.cpu_time,
        des.reads,
        des.writes,
        dec.last_write,
        des.[program_name],
        emg.requested_memory_kb,
        emg.granted_memory_kb,
        emg.used_memory_kb,
        der.wait_type,
        der.wait_time,
        der.last_wait_type,
        der.wait_resource,
        CASE des.transaction_isolation_level 
                WHEN 0 THEN 'Unspecified' 
                WHEN 1 THEN 'ReadUncommitted' 
                WHEN 2 THEN 'ReadCommitted'
                WHEN 3 THEN 'Repeatable' 
                WHEN 4 THEN 'Serializable' 
                WHEN 5 THEN 'Snapshot' 
        END AS transaction_isolation_level,
        OBJECT_NAME(dest.objectid, 
        der.database_id) as OBJECT_NAME, 
        dest.text as full_query_text,
        SUBSTRING(dest.text, 
        der.statement_start_offset /2,
                (CASE WHEN der.statement_end_offset = -1
                THEN DATALENGTH(dest.text) 
                ELSE der.statement_end_offset 
                END - der.statement_start_offset) /2)
                AS [executing_statement], 
        deqp.query_plan

FROM  sys.dm_exec_sessions des
LEFT JOIN sys.dm_exec_requests der on des.session_id = der.session_id
LEFT JOIN sys.dm_exec_connections dec on des.session_id = dec.session_id
LEFT JOIN sys.dm_exec_query_memory_grants emg  on des.session_id = emg.session_id      
CROSS APPLY sys.dm_exec_sql_text(der.sql_handle) dest
CROSS APPLY sys.dm_exec_query_plan(der.plan_handle) deqp

WHERE des.session_id <> @@SPID

ORDER BY  des.session_id ~~~



