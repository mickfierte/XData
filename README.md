# XData
XData is .Net domain object oriented data access layer component. It is not "yet another ORM", but а comlete feature rich data access layer for Your solutions. Basic idea to create this component was a suppling high quality tool to professional developers, having deep competences in data storage modeling and access.

version 1.1.0
## NuGet packages

[XData Data Access Layer package](https://packages.nuget.org/packages/XData.DataAccessLayer_BrokenLink/)
[![NuGet](https://img.shields.io/nuget/v/XData.DataAccessLayer_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.DataAccessLayer_BrokenLink.svg?style=plastic)]()

[XData UnitOfWork package](https://packages.nuget.org/packages/XData.WorkSet_BrokenLink/) - required to use [XData UnitOfWork](http://mickfierte.github.io/XData/tutorial/unit_of_work.html) implementation
[![NuGet](https://img.shields.io/nuget/v/XData.WorkSet_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.WorkSet_BrokenLink.svg?style=plastic)]()

[XData Security package](https://packages.nuget.org/packages/XData.Security.Abstractions_BrokenLink/) - [_ISecuritySession_](http://mickfierte.github.io/XData/api/XData.Interfaces.ISecuritySession.html) interface implementation helper required to use [XData security](http://mickfierte.github.io/XData/tutorial/security.html) functionality
[![NuGet](https://img.shields.io/nuget/v/XData.WorkSet_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.WorkSet_BrokenLink.svg?style=plastic)]()

[XData Three-tier client proxy package](https://packages.nuget.org/packages/XData.Remote_BrokenLink/) - client proxy to use in three-tier envirounment (Net 4.0 only, required full Net 4.0 Framework version)
[![NuGet](https://img.shields.io/nuget/v/XData.Remote_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.Remote_BrokenLink.svg?style=plastic)]()

[XData Cache package](https://packages.nuget.org/packages/XData.Cache_BrokenLink/) - time limited object caching (Net 4.0 only, required full Net 4.0 Framework version)
[![NuGet](https://img.shields.io/nuget/v/XData.Cache_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.Cache_BrokenLink.svg?style=plastic)]()

[XData MsSqlSever adapter package](https://packages.nuget.org/packages/XData.MsSqlServer_BrokenLink/) - Ms SQL Server database dialect and adapter
[![NuGet](https://img.shields.io/nuget/v/XData.MsSqlServer_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.MsSqlServer_BrokenLink.svg?style=plastic)]()

[XData Oracle adapter package](https://packages.nuget.org/packages/XData.Oracle.Odp_BrokenLink/) - Oracle database dialect and adapter based on Oracle ODP provider
[![NuGet](https://img.shields.io/nuget/v/XData.Oracle.Odp_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.Oracle.Odp_BrokenLink.svg?style=plastic)]()

[XData PostgreSql adapter package](https://packages.nuget.org/packages/XData.PostgreSql.NpgSql_BrokenLink/) - Postgre SQL database dialect and adapter based on NpgSql provider
[![NuGet](https://img.shields.io/nuget/v/XData.PostgreSql.NpgSql_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.PostgreSql.NpgSql_BrokenLink.svg?style=plastic)]()

[XData SQLite adapter package](https://packages.nuget.org/packages/XData.SQLite_BrokenLink/) - SQLite database dialect and adapter (Net 4.0 version use System.Data.SQLite provider, Net Standard 2.0 version use Microsoft.Data.Sqlite provider)
[![NuGet](https://img.shields.io/nuget/v/XData.SQLite_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.SQLite_BrokenLink.svg?style=plastic)]()

[XData MySql adapter package](https://packages.nuget.org/packages/XData.MySql_BrokenLink/) - MySQL database dialect and adapter based on MySql.Data provider
[![NuGet](https://img.shields.io/nuget/v/XData.MySql_BrokenLink.svg?style=plastic)]()
[![NuGet](https://img.shields.io/nuget/dt/XData.MySql_BrokenLink.svg?style=plastic)]()

## Basic features
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

## Installation

### Net 4.0

>1) Install Nuget package of XData [engine package](https://packages.nuget.org/packages/XData.DataAccessLayer_BrokenLink/).
>
>2) (Optional) Download one of [logger binaries](http://mickfierte.github.io/XData/index.html#Net-4.0)
>
>3) Install XData database adapter Nuget package (see [above](http://mickfierte.github.io/XData/#nuget-packages)) for Your database (or multiple databases).
>
>4) Configuration is described [here](http://mickfierte.github.io/XData/tutorial/config.html)

### (Optional) Net 4.0 three-tier architecture client libraries

>1) Additionally required to install XData [three tier client package](https://packages.nuget.org/packages/XData.Remote_BrokenLink/)
>
>2) Configuration is described [here](http://mickfierte.github.io/XData/tutorial/three_tier.html)

### (Optional) Net 4.0 three-tier architecture server

>1) Download and deploy [XData three-tier service](http://mickfierte.github.io/XData/index.html#Net-4.0) on any You like WCF host.
>
>2) Copy data mapping and data logic assemblies to service catalog.
>
>3) Service configuration is described [here](http://mickfierte.github.io/XData/tutorial/config.html)

### NetStandard 2.0

>1) Install Nuget package of XData [engine package](https://packages.nuget.org/packages/XData.DataAccessLayer_BrokenLink/).
>
>2) Install XData database adapter Nuget package (see [above](http://mickfierte.github.io/XData/#nuget-packages)) for Your database (or multiple databases).
>
>3) Configuration is described [here](http://mickfierte.github.io/XData/tutorial/config.html)


## Usage example
Net 4.0:
```csharp
using (var dataEngine = XDataManager.AddXData(x => 
	x.UseConfiguration(ConfigurationManager
		.OpenExeConfiguration(ConfigurationUserLevel.None))))
{
    using (var dataScope = dataEngine.NewDataScope())
    {
        foreach(var data in dataScope.GetRepository<SomeObject>())
            Console.WriteLine(string.Format("{0}, {1}", data.SomeId, data.Name));
    }                     
}
```
NetStandard 2.0:
```csharp
//Set configuration file
var builder = new ConfigurationBuilder();
builder.AddXmlFile("MyConsoleApp.config");
var configFile = builder.Build();

//Set services
using(var serviceProvider = new ServiceCollection()
    .AddLogging()
    // Localization only for ASP.NET Core
    .AddSingleton(typeof(IStringLocalizerFactory), x => null)
    .AddSingleton(typeof(IConfigurationRoot), x => configFile)
    .AddXData()
    .BuildServiceProvider())
{
	//Configure logging
	serviceProvider.GetRequiredService<ILoggerFactory>()
		.AddConsole(LogLevel.Warning).AddDebug();

	var dataEngine = serviceProvider.GetRequiredService<IDataEngine>();
	
	using (var dataScope = dataEngine.NewDataScope())
	{
		foreach(var data in dataScope.GetRepository<SomeObject>())
			Console.WriteLine($"{data.SomeId}, {data.Name}");
	}
}
```
static mapping:
```csharp
[DataObject("T"),
DataTable("T_SOME", "T"),
Column("SomeId", typeof(long?), "T", Flags = DataPropertyFlag.Id),
ColumnDefault(string.Empty, DefaultType.AutoIncrement)]
public class SomeObject : IDataObject
{
    [Property("T")]
    public string Name { get; set; }
}
```
dynamic mapping:
```csharp
public partial class SomeObject
{
    public string Name { get; set; }
}
```
```csharp
public partial class SomeObject : IDataObject
{
    private static Expression<CustomMapping<SomeObject>> _mapping = () =>
        XDataMapping.GetStructure<SomeObject>("T")
            .DataTable("T_SOME", "T")
            .Column("SomeId", x => x.Field<int?>("T", string.Empty, z => z.Key()))
            .Map(x => Name = x.Field<string>("T", string.Empty))
            .SetBaseTable("T");
}
```

_For more examples and usage, please refer to the [documentation](http://mickfierte.github.io/XData/)._

## Release History

* 1.0.0
    * Initially published
* 1.1.0
    * .Net Standard 2.0 version released
	* MySql is now supported
	* Lot of bugs are fixed
	* Documetation is realized now

## Contacts

Denis Dawydenko AKA Mick Fierte – d.dawydenko@gmail.com

Distributed under the MIT license. See ``LICENSE`` for more information.