# XData
XData is .Net domain object oriented data access layer component. It is not "yet another ORM", but а comlete feature rich data access layer for Your solutions. Basic idea to create this component was a suppling high quality tool to professional developers, having deep competences in data storage modeling and access.

version 1.1.0
## NuGet packages

[XData Data Access Layer package](https://github.com/mickfierte/XData/packages/95851/)

[XData UnitOfWork package](https://github.com/mickfierte/XData.WorkSet/packages/95894/) - required to use [XData UnitOfWork](http://mickfierte.github.io/XData/tutorial/unit_of_work.html) implementation

[XData Security package](https://github.com/mickfierte/XData.Security/packages/95893/) - [_ISecuritySession_](http://mickfierte.github.io/XData/api/XData.Interfaces.ISecuritySession.html) interface implementation helper required to use [XData security](http://mickfierte.github.io/XData/tutorial/security.html) functionality

[XData Three-tier client proxy package](https://github.com/mickfierte/XData.Remote/packages/95892/) - client proxy to use in three-tier envirounment (Net 4.0 only, required full Net 4.0 Framework version)

[XData Cache package](https://github.com/mickfierte/XData.Cache/packages/95880/) - time limited object caching (Net 4.0 only, required full Net 4.0 Framework version)

[XData MsSqlSever adapter package](https://github.com/mickfierte/XData.MsSqlServer/packages/95896/) - Ms SQL Server database dialect and adapter

[XData Oracle adapter package](https://github.com/mickfierte/XData.Oracle.Odp/packages/95899/) - Oracle database dialect and adapter based on Oracle ODP provider

[XData PostgreSql adapter package](https://github.com/mickfierte/XData.PostgreSql.NpgSql/packages/95902/) - Postgre SQL database dialect and adapter based on NpgSql provider

[XData SQLite adapter package](https://github.com/mickfierte/XData.SQLite/packages/95901/) - SQLite database dialect and adapter (Net 4.0 version has used System.Data.SQLite provider, Net Standard 2.0 version has used Microsoft.Data.Sqlite provider)

[XData MySql adapter package](https://github.com/mickfierte/XData.MySql/packages/95898/) - MySQL database dialect and adapter based on MySql.Data provider

[XData net 4.0 System.Diagnistics.Trace log writer package](https://github.com/mickfierte/XData.Logging.Trace/packages/95890/) - (Net 4.0 only) Log writer over System.Diagnistics.Trace

[XData net 4.0 log4net log writer package](https://github.com/mickfierte/XData.Logging.Log4Net/packages/95889/) - (Net 4.0 only) Log writer over System.Diagnistics.Trace

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

### NuGet configuration

To get XData NuGet packages You will need moify nuget.config as described below and place it in project root folder near Your solution file.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <clear />
        <add key="xdata" value="https://nuget.pkg.github.com/mickfierte/index.json" />
		<add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    </packageSources>
	<packageSourceCredentials>
        <xdata>
            <add key="Username" value="API_KEY" />
            <add key="ClearTextPassword" value="API_KEY" />
        </xdata>
    </packageSourceCredentials>
</configuration>
```
* replace API_KEY with API key provided by author

>[!TIP]
>Some "temporary implementation features" of the NuGet server inside GitHub packages require you to enable the "Include prerelease" switch and not pay attention to the mismatch of the package description fields. Hope it gets better soon.

### Net 4.0

>1) Install Nuget package of XData [engine package](https://github.com/mickfierte/XData/packages/95851/).
>
>2) Install one of logger Nuget packages for [System.Diagnistics.Trace](https://github.com/mickfierte/XData.Logging.Trace/packages/95890/) or [log4net](https://github.com/mickfierte/XData.Logging.Log4Net/packages/95889/)
>
>3) Install XData database adapter Nuget package (see [above](http://mickfierte.github.io/XData/#nuget-packages)) for Your database (or multiple databases).
>
>4) Configuration is described [here](http://mickfierte.github.io/XData/tutorial/config.html)

### (Optional) Net 4.0 three-tier architecture client libraries

>1) Additionally required to install XData [three tier client package](https://github.com/mickfierte/XData.Remote/packages/95892/)
>
>2) Configuration is described [here](http://mickfierte.github.io/XData/tutorial/three_tier.html)

### (Optional) Net 4.0 three-tier architecture server

>1) Download and deploy [XData three-tier service](http://mickfierte.github.io/XData/index.html#Net-4.0) on any You like WCF host.
>
>2) Copy data mapping and data logic assemblies to service catalog.
>
>3) Service configuration is described [here](http://mickfierte.github.io/XData/tutorial/config.html)

### NetStandard 2.0

>1) Install Nuget package of XData [engine package](https://github.com/mickfierte/XData/packages/95851/).
>
>2) Install XData database adapter Nuget package (see [above](http://mickfierte.github.io/XData/#nuget-packages)) for Your database (or multiple databases).
>
>3) Configuration is described [here](http://mickfierte.github.io/XData/tutorial/config.html)


## Usage example
Net 4.0:
```csharp
using (var dataEngine = XDataManager.InitXData(x => 
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

* 1.2.0
	* SQL Blocks is now supported
	* Bugs are fixed, algorithms are improved
* 1.0.0
    * Initially published (deprecated now)
* 1.1.0
    * .Net Standard 2.0 version released
	* MySql is now supported
	* XML data sources is now supported in dynamic queries
	* Local temporary tables is now supported in dynamic queries
	* Lot of bugs are fixed
	* Documetation is available now

## Contacts

Denis Dawydenko – d.dawydenko@gmail.com

Distributed under the MIT license. See ``LICENSE`` for more information.