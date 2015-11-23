XData
======
version 1.0
## Description
.Net domain object oriented data access layer component.
### Basic features
* Single domain object can be mapped on multiple tables (or views, or procedures) in DB. Every DB object can be mapped multiple times in one (or more mappings).
* _IQueryable_ interface is fully supported.
* Surrogate primary and foreign keys can be used in atomatical or manual mode.
* Lazy large object (LOB, XML) binding.
* Mapping on UDT (user data types) SQL procedure parameters (limited by choosen ADO .Net provider's capabilities).
* Enumeration types mapping support (including bitmask).
* Master-slave data mapped entities realtion is supported.
* Hierarchy organized data mapping.
* Optional mapping SQL parts.
* Dynamic (linq-style, fluent) or static (based on attributes) mapping can be used.
* Using statically mapped subqueries in dinamic mapping expression.
* Using mapping variables can change query structure dinamically.
* Programmer can use (or not use) dictionary data cache using its own decision.
* Using SQL procedures and functions as base of mapping.
* Same programming modules can be used in two-tier mode (e.g. for debuging) and three-tier mode (in production) based on configuration file settings.
* Declarative Unit of Work realization.
* Might and flaxible data access control rules can be used.

SQL dialets are suported:
* Microsoft SQL Server
* ORACLE
* PostgreSql
* SQLite
* MySQL

ADO .Net providers are supported:
* Microsoft SQL Server (Microsoft SQL Client)
* ORACLE (Microsoft Oracle Client and Oracle Data Provider)
* PostgreSql (NpqSql Provider and Devart DotConnect PostgreSQL adapter)
* SQLite (SQLite Data Provider)
* MySQL (MySQL Data Provider)

> Microsoft Oracle Client is supported but not recomended cause it has deprecated state and very low speed.

New SQL dialect support module and/or provider adapter module will be added as needed. Each such unit is usually not more than 100 lines of code and is not difficult to implement (1 or 2 days).

Detailed description available on [english :uk: :us:](./docs/en/common.md) and [russian :ru:](./docs/ru/common.md) languages
