---
layout: post
title: SQL Server CPU, Cores, Schedulers, Threads, Tasks
published-on: 25 March 2021
keywords: sqlserver, databases,sessions, process, dmv, dm_exec_requests, dm_exec_connections, dm_exec_sessions, CPU, Cores, Schedulers, Threads, Tasks 
---

Voy a intentar explicar como SQL Server gestiona los procesos que procesa. Primero veamos las fases que se producen:

1 - Se crea una conexión en SQL Server al recibir una petición de un cliente externo. En procesos internos, esta fase no se produce.
2 - Se crea una session (session_id) al realizarse la petición con exito.
3 - Se genera una request/solicitud que hay que procesar.
4 - Se genera una task/tarea para procesar la request/solicitud y se solicitan una serie de recursos para poder trabajar (cpu, memoria, tablas, indices, etc).
5 - Se enlaza la tarea con un worker.
6 - La tarea se ejecuta en un scheduler.

Veamos conceptos:

Request: es la representación lógica de una query o un proceso batch. Tambien abarca operaciones como un checkpoint o un una escritua en el log de transacciones.
Las request pasan por varias fases y sufren diferentes esperas por los recursos que necesita. Las requests se dividen en minitareas llamadas task/tarea. Cada tarea debe ser procesada por un worker y los workers son gestionados por schedulers.

Para que una request se pueda procesar por la cpu, tiene que tener previamente todos los recursos que necesita. Cuando lo tiene todo, espera en la cola para que se le asigne un "trozo de tiempo" de cpu, porque las cpu procesan las peticiones por trozos de tiempo, no se asignan a una solicitud hasta que termina, va dedicando "trozos de tiempo" a las request que estan en la cola.


Worker: Es la representación lógica de un thread dentro del SQLOS. Son los responsables de ejecutar una tarea en un scheduler.

Scheduler (SOS Scheduler) – Hay un scheduler por cada core logico que se asigna a la instancia (por defecto tiene asignados todos los cores). El scheduler gestiona el acceso de los workers a su correspondiente core. Asigna "trozos de tiempos de core" en modo de hilos/threads a los workers, siendo ese core exclusivo para el worker en esa trozo de tiempo. (consultar sys.dm_os_schedulers). Es como si fuese una persona que abre una puerta para estar en la habitacion del core por un tiempo determinado y se encarga de sacarlo una vez finalizado el tiempo disponible.

Task – Una tarea representa el trabajo que hay que procesar (consultar la DMV sys.dm_os_tasks ). Una tarea puede ser alguno de los siguietes eventos: query (RPC event o Language event),  prelogin (prelogin event),   login (connect event),  logout  (disconnect event), query cancellation (Attention event), bulk load (bulk load event), distributed transaction (transaction manager event). 

Worker: Es la representación lógica de un thread dentro del SQLOS. Son los responsables de ejecutar una tarea en un scheduler.
