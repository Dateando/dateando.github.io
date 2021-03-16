---
layout: post
title: Consultar procesos en SQL Server
---

En esta pildora sobre SQL Server voy a mostrar que vistas podemos consultar sobre los procesos en ejecución. Tenemos las siguientes:

1) master.dbo.sysprocesses (deprecated)
2) master.sys.dm_exec_sessions
3) master.sys.dm_exec_request
4) master.sys.dm_exec_connections

Estas vistas, que no tablas, nos permiten ver los procesos de usuario y de sistema que están en ejecución.
Los procesos de sistema no vienen de ningún sitio mas alla de la maquina donde tenemos la instancia de SQL Server, son generados por el propio motor de SQL Server. Los procesos de sistema son procesos internos que no deberian generar problemas en el rendimiento de la instancia por lo que se suelen descartar a la hora de analizar el entorno.
Los procesos de usuario pueden haber sido lanzados desde la maquina local o desde otras maquinas.
Todo proceso lleva asignado un spid. Antiguamente se consideraba que los procesos que tenian un spid igual o menos a 50 eran los unicos de sistema, pero no es asi, puede haber procesos de sistema con spid superior al 50. 

Vamos a verlas mas detalladamente

**master.dbo.sysprocesses**

Esta vista viene de la version de SQL Server 2000 y está marcada como deprecated. Deprecated en informatica significa que aunque sigue estando disponible, no quieren que te encariñes con la funcionalidad ya que en proximas versions desapareceran.

Aunque esta deprecated, es muy util porque te permite ver mucha información interesante sin necesidad de hacer JOINS con otras.

Columnas interesantes de esta vista:


dbid      Nombre de la base de datos donde se ha conectado el proceso [ DB_NAME(dbid) ]
loginame  Login que ha iniciado el proceso
blocked   spid del proceso que esta bloqueando a este proceso. Muy importante, ya que el numero que aparezca aqui impide avanzar al proceso y es candidato a analizarlo.
status    Estados por los que pasa el proceso. Posibles valores: dormant, running, background, background, rollback, pending, runnable, spinloop, suspended.
hostname  Servidor/maquina desde donde se ha creado la conexión para iniciar el proceso en SQL Server.

Para consultarlo requiere permisos VIEW SERVER STATE
``` T-SQL
GRANT VIEW SERVER STATE TO [loginMio]
```

**master.sys.dm_exec_session**

Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
