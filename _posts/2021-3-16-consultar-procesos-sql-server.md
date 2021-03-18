---
layout: post
title: Consultar procesos en SQL Server mediante DMV
tags: sqlserver, databases,sessions, process, dmv,dm_exec_requests,dm_exec_connections,dm_exec_sessions 


---
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-GEF11HDH3Q"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-GEF11HDH3Q');
</script>

H1 Analizar procesos de SQL Server mediante DMV *

En esta pildora sobre SQL Server voy a mostrar que vistas podemos consultar sobre los procesos en ejecución. Tenemos las siguientes:

*  **master.dbo.sysprocesses** *(deprecated)*
*  **sys.dm_exec_sessions**
*  **sys.dm_exec_connections**
*  **sys.dm_exec_request**


Estas vistas, que no tablas, nos permiten ver los procesos de usuario y de sistema que están en ejecución.
Los procesos de sistema nacen en la misma máquina donde tenemos la instancia de SQL Server, son generados por el propio motor de SQL Server. Los procesos de sistema son procesos internos que no deberian generar problemas en el rendimiento de la instancia por lo que se suelen descartar a la hora de analizar el entorno.
Los procesos de usuario pueden haber sido lanzados desde la maquina local o desde otras maquinas.
Todo proceso lleva asignado un spid. Antiguamente se consideraba que los procesos que tenian un spid igual o menos a 50 eran los unicos de sistema, pero no es asi, puede haber procesos de sistema con spid superior al 50. 

Vamos a verlas mas detalladamente

## master.dbo.sysprocesses 

Esta vista viene de la version de SQL Server 2000 y está marcada como deprecated. Deprecated en informatica significa que aunque sigue estando disponible, no quieren que te encariñes con la funcionalidad ya que en proximas versions desapareceran.

Aunque esta deprecated, es muy util porque te permite ver mucha información interesante sin necesidad de hacer JOINS con otras.

Columnas interesantes de esta vista:

*  *dbid*      Nombre de la base de datos donde se ha conectado el proceso [ DB_NAME(dbid) ]
*  *loginame*  Login que ha iniciado el proceso
*  *blocked*   spid del proceso que esta bloqueando a este proceso. Muy importante, ya que el numero que aparezca aqui impide avanzar al proceso y es candidato a analizarlo.
*  *status*    Estados por los que pasa el proceso. Posibles valores: dormant, running, background, background, rollback, pending, runnable, spinloop, suspended.
*  *hostname*  Servidor/maquina desde donde se ha creado la conexión para iniciar el proceso en SQL Server.

Para consultarlo requiere permisos VIEW SERVER STATE
``` T-SQL
GRANT VIEW SERVER STATE TO [loginMio]
```

## sys.dm_exec_session

Todo proceso que llega a SQL Server se gestiona como una sesion, tanto procesos de sistema como de usuario. Esta vista nos devolvera todas los spid.
También ofrece información interesante sobre la conexión que se ha establecido para abrir la nueva sesion y datos acumulados de la sesión. Es importante aclarar que una sesión esta formada por una o mas request, y la sesión se mantiene desde que se abre la conexión hasta que se cierra.

De aqui nos interesan principalmente las siguientes columnas:

- *login_name*    Login que ha iniciado el proceso
- *host_name*     Servidor/maquina desde donde se ha creado la conexión para iniciar el proceso en SQL Server. Si es un proceso interno sera NULL.
- *cpu_time*      Tiempo en milisegundos que lleva ejecutando la cpu en la sesion actual.
- *reads*         Numero de paginas leidas en la sesion actual. Una pagina son 8Kbytes.
- *writes*        Número de paginas escritas en la sesion actual. 
- *last_write*    
- *program_name*  aplicación que lanzar el proceso. Si es un proceso interno sera NULL.

## sys.dm_exec_connections

Esta vista nos filtra aquellas sesiones que sean de usuario, las que sean de internas de sistema no apareceran. Por tanto, la mejor forma de filtrar en el resto de vistas para que se muestre solo las lineas correspondientes a procesos de usuario es hacer un `INNER JOIN` con esta vista por el campo id_session. Veremos que hay consultas que filtran por `id_session > 50` pero la forma mas correcta es mediante el join a esta vista. Tambien es recomendable quitar en el filtro la sesion en la que estamos lanzando nuestra consulta de monitorización de la siguiente forma `id_session <> @@SPID`.

## sys.dm_exec_request
Esta ultima es la mas interesante y de ella sacararemos la mayor parte de la información que nos interesa. El ambito de una request son las sesiones, todas las request se ejecutan dentro de una sesion. Esta vista nos muestra los procesos que estan en ejecución. Las columnas a destacar son:

- *database_id*      Nombre de la base de datos donde se ha conectado el proceso `[ DB_NAME(dbid) ]`
- *blocking_session_id*   spid del proceso que esta bloqueando a este proceso. Muy importante, ya que el numero que aparezca aqui impide avanzar al proceso y es candidato a analizarlo.
- *status*    Estados por los que pasa el proceso. Posibles valores: dormant, running, background, background, rollback, pending, runnable, spinloop, suspended.
- *hostname*  Servidor/maquina desde donde se ha creado la conexión para iniciar el proceso en SQL Server.
- *percent_complete* Muestra el % que lleva alcanzado cuando la request ejecuta alguno de los siguientes comandos:
                > **ALTER INDEX REORGANIZE
                > **AUTO_SHRINK option with ALTER DATABASE
                > **BACKUP DATABASE
                > **DBCC CHECKDB
                > **DBCC CHECKFILEGROUP
                > **DBCC CHECKTABLE
                > **DBCC INDEXDEFRAG
                > **DBCC SHRINKDATABASE
                > **DBCC SHRINKFILE
                > **RECOVERY
                > **RESTORE DATABASE
                > **ROLLBACK 
                > **TDE ENCRYPTION
- *command*   Tipo de comando que se ejecuta o se ejecuto en la ultima request. Ejemplos: SELECT, INSERT, UPDATE, DELETE, BACKUP LOG, BACKUP DATABASE, DBCC...
- *wait_type*  Tipo de espera por la que espera la request en este momento actual, en caso de estar espeando por algo. Si vale NULL es que no esta esperand por nada, estará en `status` con valor a `running`. 
- *wait_time*  Tiempo en milisegundos que lleva esperando por los diferentes recursos (cpu, memoria, indices, tablas, etc) 
- *last_wait_type*  Último tipo de espera de la request.
- *wait_resource*  Si wait_type es distinto de NULL y esta esperando por un recurso tipo tabla, índice, etc aparecerá en esta columna.




