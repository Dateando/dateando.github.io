---
layout: post
title: Base de datos TEMPDB en SQL Server
published-on: 20 Abril 2021
keywords: sqlserver, sql server, tempdb, databases, Azure, Azure SQL
---


Tempdb es una base de datos de sistema de SQL Server. Solo existe una por instancia/instalación y esta utilizada por todas las bases de datos de usuaario. por ello es importante configurarla de forma correcta. TEMPDB es una base de datos que se resetea o inicializa cada vez que se reiniciar la instancia, por tanto los datos que tiene son temporales. 
A la hora de ajustar la configuracion de la base de datos TEMPDB debemos fijarnos principalmente en tres factores:

** Número de ficheros (data files) **
Cada fichero de datos (data file) de una base de datos tiene una pagina de metadatos que controla las paginas de datos que estan libres u ocupadas en el fichero, por eso cada vez que un proceso/request solicita espacio libre, tiene que preguntar a esta pagina de metadatos donde hay espacio para su proceso. Para realizar esta pregunta, existe un *queu latch*, esta cola va gestionando las peticiones de una en una, por tanto si aumentamos el número de ficheros de datos, tendremos una pagina de metadatos y un latch queu por cada fichero y por tanto los procesos podran solicitar espacio de forma paraleta a cada latch, evitando la saturación de una única cola. Cada solicitud será gestionada por un scheduler que normalmente existe uno por cada core asignado a la instancia. A la hora decidir el número de ficheros a crear, lo recomendao para empezar es crear un 25% o 50% del número de cores que tengamos, no superando la cantidad de 8 ficheros de datos. El log de transacciones  es suficiente con que sea uno.

** Tamaño de los ficheros **
El tamaño de los ficheros de datos debe ser el mismo en todos. SQL Server utiliza un algoritmo para asignar espacio en cada fichero, que está optimizado para que asigne el espacio libre del fichero de datos que mas espacio libre tenga. Si tenemos unos ficheros de datos mucho mayores que otros, SQL Server va a tender a asignar espacio en esos ficheros ya que logicamente deberían tener mas espacio libre, saturando sus colas de petición de espacio libre y las operaciones de lectura/escritura. Si todos los ficheros tienen el mismo tamaño, el uso y saturación será homogeneo entre todos los ficheros. 
El crecimento de los ficheros de datos en general no debe ser definido en %, sino que debemos fijar el tamaño fijo en que tienen que crecer si es necesario. Lo optimo es asignar el tamaño necesario a la hora de configurar la base de datos, debido a que si dejamos que la base de datos vaya haciendo crecer sus ficheros a demanda de las necesidades de los procesos, generaremos fragmentacion externas en los discos y por tanto menor rendimiento en las operaciones de lectura/escritura (I/O).
No es recomendable reducir (shrink) la base de datos TEMPDB, debido a que si en algún momento a llegar a necesitar ese tamaño, es muy probable que en el futuro lo vuelva a necesitar, y de esta forma evitamos que SQL Server tenga que buscar espacio libre disponible en los discos para asignarselos. 
Una importante recomendación a la hora de asignar nuevo espacio a un fichero de datos es tener configurado *Enable Instant File Initialization*, lo que permitira crear o añadir espacio libre a los ficheros de datos de una forma mucho mas rápida.

** Localización de los ficheros **

La base de datos TEMPDB debe ser ubicada en un almacenamiento dedicado. No es necesario que este almacenamiento sea replicado ya que es temporal, pero es muy importante que sea de acceso muy rapido, por lo que se recomienda que se utilicen discos SSD (Discos de estado sólido).

Las bases de datos TEMPDB se utilizan generalmente en las siguientes situaciones:

- Uso de Tablas temporales.
- Triggers.
- Table variables.
- Prpcedimientos Alamacenados temporales.
- Procesos de mantenimiento de índices en TEMPDB (SORT_IN_TEMPDB)
- Ordenaciones pesadas.
- Join pesados.
- Nivel de aislamiento READ COMMITTED SNAPSHOT ISOLATION (RCSI) .
