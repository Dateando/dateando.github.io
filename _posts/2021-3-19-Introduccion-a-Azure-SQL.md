---
layout: post
title: Introducción a Azure SQL
published-on: 19 Marzo 2021
keywords: sqlserver, databases, Azure, Azure SQL, Azure SQL Database, Azure SQL Managed Instance, Azure SQL Server, Virtual Machine, PaaS, IaaS
---


Azure es la nube o cloud de Microsoft. En Azure tenemos disponible un concepto que se denomina Azure SQL, si en nuestras maquinas tenemos SQL Server, en la nube de Microsoft se le denomina Azure SQL.

Azure SQL se compone basicamente de tres productos:

- Azure SQL *Database* 
- Azure SQL *Managed Instance*
- Azure SQL *Server*

Estos tres prductos se pueden clasificar en dos tipologias:
 
- PaaS: Platform as a Service. Esta tipología engloba a Azure SQL Database y Azure SQL Managed Instance, porque solo accedemos al servicio del motor de base de datos.
- IaaS: Infraestructure as a Service. Esta tipología se corresponde con Azure SQL Server in Virtual Machine, ya que tenemos acceso al sistema operativo y a instalar lo que queramos en la maquina virtual.

Cada una de estos tres productos ofrece diferentes precios, segun queramos que el producto sea mas o menos completo.

Vamos a ver con un poco mas detalle cada una de ellas.

## Azure SQL Database *(PaaS)*

Esta podria ser la opción mas barata y donde menos control tenemos sobre el producto y por tanto donde no requiere esfuerzos de administración.
Simplemente es una base de datos disponible en la nube a la que nos podemos conectar con nuestro Management Studio de siempre (SSMS que es gratuito) o con Azure Portal o con nuestra aplicación.
Esta base de datos siempre esta en la ultima versión de SQL Server, es mas, está en versiones no disponibles comercialmente (podemos compararlo ejecutando `SELECT @@VERSION`). Tiene disponibles todas las funcionalidades que se le puede pedir a una base de datos.

Incluye una serie de servicicos por defecto:
- Capacidad de almancenamiento hasta 4 TB.
- Backup (Un Full a la semana, un Differential cada 12 horas y del Transaction Log cada 5 o 10 minutos según actividad). La retención mas básoca son 7 días.
- Chequeo de la base de datos.
- Siempre con el ultimo parche disponible.
- Hasta 100 TB de capacidad.
- Alta Disponibilidad (HA) por defecto mediante replicación del almancenamiento o replicación por Availability Groups (AG) segun lo que contratemos.


## Azure SQL Managed Instance *(PaaS)*

En este caso estamos pagando por una instancia (siempre en la ultima versión disponible, no podemos modificarlo) que es auto administrada. Podemos meter todas las bases de datos que queramos dentro del almacenamiento que tengamos disonible en el contrato. No es exactamente como administrar una instancia en nuestras instalaciones, ya que hay algunas cosas a las que no tenemos acceso. No tenemos acceso al sistema operativo, pero es la mejor opción si queremos migrar nuestra instancia al cloud de Azure.


## Azure SQL Server *(IaaS)*

Es producto nos ofrece una maquina virtual en la que podemos poner la versión que queramos de SQL Server, tenemos acceso al sistema operativo e incluso podemos instalar otras aplicaciones necesarias.


