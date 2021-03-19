---
layout: post
title: Introduccion a Azure SQL
published-on: 19 March 2021
keywords: sqlserver, databases, Azure, Azure SQL, Azure SQL Database, Azure SQL Managed Instance, Azure SQL Server, Virtual Machine
---


Azure es la nube o cloud de Microsoft. En Azure tenemos disponible un concepto que se denomina Azure SQL, si en nuestras maquinas tenemos SQL Server, en la nube de Microsoft se le denomina Azure SQL.

Azure SQL se compone basicamente de tres productos:

- Azure SQL *Database* 
- Azure SQL *Managed Instance*
- Azure SQL *Server*

Cada una de estos tres productos ofrece diferentes precios, segun queramos que el producto sea mas o menos completo.

Vamos a ver con un poco mas detalle cada una de ellas.

# Azure SQL Database

Esta podria ser la opción mas barata y donde menos control tenemos sobre el producto y por tanto donde no requiere esfuerzos de administración.
Simplemente esta una base de datos disponible en la nube a la que nos podemos conectar con nuestro Management Studio de siempre (SSMS que es gratuito) o con Azure Portal.
Esta base de datos siempre esta en la ultima versión de SQL Server, es mas, esta en versiones no disponibles comercialmente (Podemos compararlo ejecutando `SELECT @@VERSION`
Incluye una serie de servicicos por defecto:
- Backup (Un Full a la semana, un Diferential cada 12 horas y del Transaction Log cada 5 o 10 minutos según actividad).
- Chequeo de la base de datos.
- Siempre con el ultimo parche disponible.
- Hasta 100 TB de capacidad.
- DR (Disaster Recovery // Recuperación ante Desastres) mediante replicación del almancenamiento o replicación por Availability Groups (AG) segun lo que contratemos.

