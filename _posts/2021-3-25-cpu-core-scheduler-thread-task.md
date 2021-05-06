---
layout: post
title: SQL Server CPU, Cores, Schedulers, Threads, Tasks
published-on: 25 March 2021
keywords: sqlserver, databases,sessions, process, dmv, dm_exec_requests, dm_exec_connections, dm_exec_sessions, CPU, Cores, Schedulers, Threads, Tasks 
---

Voy a intentar explicar como SQL Server gestiona los procesos que procesa. Primero veamos las fases que se producen:

- 1 - Cuando una aplicación (SSMS, sqlcmd, .net, phyton,...) intenta conectarse a SQL Serer, este le autentica y abre una conexión y crea una sesion para esa conexión. En procesos internos, esta fase no se produce, directamente se crea una sesion para el proceso interno.

- 2 - Cuando se crea una sesion (session) al establecerse la conexión on exito, se le asocia una session_id.

- 3 - La aplicación manda una request/solicitud que puede ser por ejemplo una SELECT y que hay que procesar.

- 4 - Para esta request/solicitud, se genera una o mas tasks/tareas para procesarla y se solicitan una serie de recursos para poder trabajar (cpu, memoria, tablas, indices, etc).

- 5 - Se enlaza cada tarea con un worker.

- 6 - Cada tarea se ejecuta en un scheduler.

Veamos conceptos:

**Request**: es la representación lógica de una query o un proceso batch. Tambien abarca operaciones como un checkpoint o un una escritua en el log de transacciones.
Las request pasan por varias fases y sufren diferentes esperas por los recursos que necesita. Las requests se dividen en minitareas llamadas task/tarea. Cada tarea debe ser procesada por un worker y los workers son gestionados por schedulers.

Para que una request se pueda procesar por la cpu, tiene que tener previamente todos los recursos que necesita. Cuando lo tiene todo, espera en la cola para que se le asigne un "trozo de tiempo" de cpu, porque las cpu procesan las peticiones por trozos de tiempo, no se asignan a una solicitud hasta que termina, va dedicando "trozos de tiempo" a las request que estan en la cola. Si una request recibe un "trozo de tiempo" pero le falta otro recursos, cede su "trozo de tiempo" para que sea utilizado por otro request (procesos colaborarivos).


**Worker**: Es la representación lógica de un thread dentro del SQLOS. Son los responsables de ejecutar una tarea en un scheduler.

**Scheduler** (SOS Scheduler) – Hay un scheduler por cada core logico que se asigna a la instancia (por defecto tiene asignados todos los cores). El scheduler gestiona el acceso de los workers a su correspondiente core. Asigna "trozos de tiempos de core" en modo de hilos/threads a los workers, siendo ese core exclusivo para el worker en esa trozo de tiempo. (consultar sys.dm_os_schedulers). Es como si fuese una persona que abre una puerta para estar en la habitacion del core por un tiempo determinado y se encarga de sacarlo una vez finalizado el tiempo disponible.

**Task** – Una tarea representa el trabajo que hay que procesar (consultar la DMV sys.dm_os_tasks ). Una tarea puede ser alguno de los siguietes eventos: query (RPC event o Language event),  prelogin (prelogin event),   login (connect event),  logout  (disconnect event), query cancellation (Attention event), bulk load (bulk load event), distributed transaction (transaction manager event). 


