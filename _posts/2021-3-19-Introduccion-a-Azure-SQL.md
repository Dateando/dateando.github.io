---
layout: post
title: Introducción a Azure SQL
published-on: 19 Marzo 2021
keywords: sqlserver, databases, Azure, Azure SQL, Azure SQL Database, Azure SQL Managed Instance, Azure SQL Server, Virtual Machine, PaaS, IaaS
---


Azure es la nube o cloud de Microsoft. En Azure tenemos disponible una solución que es **Azure SQL**. Si en nuestras maquinas instalamos SQL Server, en la nube de Microsoft a las instalaciones de SQL Server se las denomina Azure SQL.



La solución Azure SQL se compone basicamente de tres productos:

- **Azure SQL *Database* (SQL DB)**. Tiene 3 arquitecturas o niveles/tiers posibles:
    - General Purpose/Standard
    - Hyperscale
    - Business Critical/Premium
    
- **Azure SQL *Managed Instance* (SQL MI)**
- **Azure SQL *Server* on Azure Virtual Machine**

Estos tres productos se pueden clasificar en dos tipologias:
 
- **PaaS**: Platform as a Service. Esta tipología engloba a Azure SQL Database y Azure SQL Managed Instance, porque solo accedemos al servicio del motor de base de datos.
- **IaaS**: Infraestructure as a Service. Esta tipología se corresponde con Azure SQL Server on Virtual Machine, ya que tenemos acceso al sistema operativo y a instalar lo que queramos en la maquina virtual.

Azure SQL se puede dimensionar y especialmente facturar mediante dos modelos:

- DTUs (Database Transaction Unit): Es una medida de rendimiento que abarca los cores, memoria y operaciones de I/O.
- vCore (Virtual Cores): Se selecciona el numero de cores que queremos para nuestra plataforma en la nube, y según el numero de cores, tendremos una cantidad de almacenamiento y memoria. Este modelo no se puede en Azure SQL Server on Azure Virtual Machine.

<img src="/imaged/azure.jpg" alt="Azure New Logo 2021" style="width:200px;display: block;margin-left: auto;margin-right: auto;width: 50%;">

Vamos a ver cada uno de los tres tipos básicos de productos de Azure SQL:

## Azure SQL Database *(PaaS y DaaS*

Esta podria ser la opción mas barata y donde menos control tenemos sobre el SQL Server y por tanto donde no requiere esfuerzos de administración. Esta orientado en tener una base de datos siempre disponible, actualizada y facil de modificar sus recursos de memoria, cpu y almacenamiento.
Simplemente es una base de datos disponible en la nube a la que nos podemos conectar con nuestro Management Studio de siempre (SSMS que es gratuito) o con Azure Portal o con nuestra aplicación.
Esta base de datos siempre está en la áltima versión de SQL Server, es mas, está en versiones no disponibles comercialmente (podemos compararlo ejecutando `SELECT @@VERSION`). Tiene disponibles todas las funcionalidades que se le puede pedir a una base de datos.

Incluye una serie de servicicos por defecto:
- Capacidad de almancenamiento hasta 4 TB.
- Backup (Un Full a la semana, un Differential cada 12 horas y del Transaction Log cada 5 o 10 minutos según actividad). La retención mas básoca son 7 días.
- Chequeo de la base de datos.
- Siempre con el ultimo parche disponible.
- Hasta 100 TB de capacidad.
- Alta Disponibilidad (HA) por defecto mediante replicación del almancenamiento o replicación por Availability Groups (AG) segun lo que contratemos.

Excetpo Hyperscale, los otros dos nivels/tiers o arquitecturas de SQL DB se pueden agrupar en lo que se llama **Elastic Pool** en los que configuramos una cantidad de memoria, cores y almacenamiento y lo distribuimos entre las bases de datos del Elastic Pool. 

## Azure SQL Managed Instance *(PaaS)*

En este caso estamos pagando por una instancia (siempre en la ultima versión disponible, no podemos modificarlo) que es auto administrada. Podemos meter todas las bases de datos que queramos dentro del almacenamiento que tengamos disonible en el contrato. No es exactamente como administrar una instancia en nuestras instalaciones, ya que hay algunas cosas a las que no tenemos acceso. No tenemos acceso al sistema operativo, pero es la mejor opción si queremos migrar nuestra instancia al cloud de Azure.


## Azure SQL Server on Azure Virtual Machine *(IaaS)*

Es producto nos ofrece una maquina virtual en la que podemos poner la versión que queramos de SQL Server, tenemos acceso al sistema operativo e incluso podemos instalar otras aplicaciones necesarias.


