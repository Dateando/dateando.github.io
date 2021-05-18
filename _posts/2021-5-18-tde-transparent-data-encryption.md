---
layout: post
title: SQL Server ::  TDE - transparent data encryption
published-on: 18 May 2021
tag: sqlserver, databases,sessions, tde, transparent data encryption
keywords: sqlserver, databases,sessions, tde, transparent data encryption 
---

Hay varias formas de encriptar los datos que almacena SQL Server, y una de las mas conocidas y menos intrusivas en las aplicaciones es TDE o Transparent Data Encryption. Como su nombre indica, la encriptación de los datos es transparente, transparente para la aplicación porque ella ni se entera. 

Esta encriptacion de datos es a nivel de base de datos, y sobre datos frios, es decir, aquellos datos que estan en disco. En el momento que un usuario o aplicación hace una consulta sobre unos datos y estos pasan del disco a memoria y luego viajan por la red hasta el proceso solicitante, los datos dejan de estar encriptados.
Pero nos estamos asegurando que si alguien se hace con los ficheros de datos y log de transacciones de la base de datos encriptada, no podran ser utilizados en otra instancia. Y esto mismo ocurre con los backups realizados sobre una base de datos encriptada con TDE, no es posible restaurarlos en otra instancia.

El proceso para habilitar TDE - Transparent Data Encription en una instancia y luego en cada base de datos que se desee es muy sencillo, y se hace en tres simples pasos, que requieren hacerlo por T-SQL, ya que SQL Server Manageent Studio no permite hacerlo mediante un wizard.

