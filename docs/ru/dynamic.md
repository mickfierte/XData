>Извините. Статья еще находится в стадии разработки, но мы уже работаем над ней...

### Динамическое преобразование (dynamic mapping)
>Прежде чем изучать данный раздел убедитесь, что вы ознакомились с [глосcарием](./glossary.md) и [общим описанием объектного преобразования](./mapping.md) которые содержат полезные знания, на которые опирается информация изложенная здесь.

Для быстрого доступа к конкретной части описания воспользуйтесь представленной структурой:
* [Описание параметров объектного отражения](#Описание-параметров-объектного-отражения)
* [Описание преобразования с использованием подзапросов](#Описание-преобразования-с-использованием-подзапросов)
* [Фильтры](#Фильтры)
	* [Группы фильтров](#Группы-фильтров)
	* [Константные фильтры](#Константные-фильтры)
	* [Фильтры на основе SQL выражений](#Фильтры-на-основе-sql-выражений)
	* [Фильтры на основе списка значений](#Фильтры-на-основе-списка-значений)
	* [Фильтры на основе кодов справочников](#Фильтры-на-основе-кодов-справочников)
	* [Связи внутри объекта данных](#Связи-внутри-объекта-данных)
	* [Фильтры по подзапросу](#Фильтры-по-подзапросу)
	* [Связи с подзапросами](#Связи-с-подзапросами)
* [Описание преобразования свойств](#Описание-преобразования-свойств)
	* [Свойства на основании SQL выражений](#Свойства-на-основании-sql-выражений)
	* [Значения по умолчанию для свойств](#Значения-по-умолчанию-для-свойств)
	* [Парамеры группировки свойства](#Парамеры-группировки-свойства)
	* [Скрытые свойства](#Скрытые-свойства)
	* [Свойства внутренних представлений](#Свойства-внутренних-представлений)
	* [Описание ссылок](#Описание-ссылок)
* [Внешние связи](#Внешние-связи)
* [Поддержка таблиц иерархии](#Поддержка-таблиц-иерархии)
* [Описание объектного преобразования хранимых процедур и функций SQL](#Описание-объектного-преобразования-хранимых-процедур-и-функций-SQL)
	* [Описание хранимой процедуры как источника данных](#Описание-хранимой-процедуры-как-источника-данных)
	* [Описание параметра хранимой процедуры](#Описание-параметра-хранимой-процедуры)
	* [Описание возвращаемого набора данных](#Описание-возвращаемого-набора-данных)
	* [Примеры описания объектного преобразования хранимых процедур и функций SQL](#Примеры-описания-объектного-преобразования-хранимых-процедур-и-функций-SQL)

Наряду со статическим описанием объектного преобразования XData поддерживает возможность динамического описания правил преобразования в виде специфичного LINQ выражения. Ниже приведен пример динамического описания преобразования (mapping) объекта данных (того же самого который был описан в разделе ["Статическое преобразование (static mapping)"](./static.md)).
```csharp
    public partial class Invoice: IDataObject
	{
        public static CustomLogic<Invoice> TestCustomLogic;

        private static Expression<CustomMapping<Invoice>> _mapping = (
            () => XDataManager.CustomMapping<Invoice>()
                .DataTable("T_DOC", "D", x => x.DictFilter("T_DOC_TYPE", "doc_type_id", "code", "INVOICE"))
                .DataTable("T_DOC_DOC_STATE", "DS", "D", x => x.Link("D", "doc_id"))
                .DataTable("T_DOC_STATE", "S", x => x.Link("DS", "doc_state_id"))
                .DataTable("T_DOC_CATALOGUE", "DC", "D", x => x.Link("D", "doc_id"), 
                    x => x.SubqueryFilter("catalogue_id", "UT").SetOperation(FilterOperation.In))
                .DataTable("T_CATALOGUE", "U", x => x.Link("DC", "catalogue_id"))
                .DataTable("T_DOC_NUMBER", "N", "D", x => x.Link("D", "doc_id"))
                .DataTable("T_DOC_CUST", "CF", "D", x => x.Link("D", "doc_id"), 
                    x => x.DictFilter("T_DOC_CUST_TYPE", "doc_cust_type_id", "code", "FROM"))
                .DataTable("T_CUSTOMER", "F", x => x.Link("CF", "customer_id"))
                .DataTable("T_DOC_CUST", "CT", "D", x => x.Link("D", "doc_id"), 
                    x => x.DictFilter("T_DOC_CUST_TYPE", "doc_cust_type_id", "code", "TO"))
                .DataTable("T_CUSTOMER", "T", x => x.Link("CT", "customer_id"))
                .DataTable("T_DOC_SCAN", "SC", "D", x => x.Link("D", "doc_id").SetOperation(FilterOperation.OuterJoin))
                .DataTable("T_DOC_SOURCE", "SR", "D", x => x.Link("D", "doc_id").SetOperation(FilterOperation.OuterJoin))
                .DataTable("T_DOC_DELIVERY", "DD", "D", x => x.Link("D", "doc_id").SetOperation(FilterOperation.OuterJoin))
                .Subquery("A", typeof(DocSpecAmounts), "Amount", DataGrouping.Sum, x => x.SubqueryLink("DocId"))
                .Subquery<CatalogueTree>("UT", x => x.CatalogueId, DataGrouping.None)
                .InnerView("H", XDataManager.GetStructure("H", null, DataStructureFlag.Grouping, new Variable[0])
                    .DataTable("T_DOC_HISTORY", "H")
                    .Select(x => new {
                        DocId = x.Field<long?>("H", string.Empty, z => z.Group(DataGrouping.None)),
                        HistoryDate = x.Field<DateTime?>("H", string.Empty, z => z.Group(DataGrouping.Max))
                    }), x => x.SubqueryLink("DocId").SetOperation(FilterOperation.OuterJoin))
                .InnerView<DocBySpecType>("ST", x => x.SubqueryLink("DocId"))
                .Column("DocId", x => x.Field<long?>("D", string.Empty, z => z.Key(), z => z.Default(DefaultType.AutoIncrement)))
                .Column("CatalogueId", x => x.Field<long?>("U", string.Empty))
                .ReadOnlyProperty(x => x.DocStateCode, x => x.Field<string>("S", "code"))
                .ReadOnlyProperty(x => x.Generation, x => x.Field<long>("D", string.Empty, z => z.ConcurrencyToken(), z => z.Default(DefaultType.AutoIncrement)))
                .ReadOnlyProperty(x => x.Changed, x => x.Field<DateTime>("D", string.Empty, z => z.Default(DefaultType.CurrentDateTime, true)))
                .ReadOnlyProperty(x => x.Author, x => x.Field<string>("D", string.Empty, z => z.Default(DefaultType.UserName, true)))
                .ReadOnlyProperty(x => x.DocAmount, x => x.Expr<decimal?>(null, DataExpressionType.SubQuery, "A", DbType.Decimal, z => z.Size(17, 5)))
                .ReadOnlyProperty(x => x.DocLastChange, x => x.Ref<DateTime?>("H", "HistoryDate"))
                .Map(x => new Invoice {
                    DocState = x.Link<string, DocState>("S", "name",
                        z => z.LinkProperty<DocState>(y => y.Name),
                        z => z.LinkProperty<DocState>(y => y.Code, y => y.DocStateCode)),
                    DocCatalog = x.Link<string, Catalogue>("U", "name", z => z.LinkProperty<Catalogue>(y => y.Name)),
                    DocNumb = x.Field<string>("N", "numb"),
                    DocDate = x.Field<DateTime?>("D", string.Empty, z => z.Default(DefaultType.CurrentDate)),
                    Scan = x.Lob("SC", z => z.OuterFlag()),
                    Source = x.Xml("SR", z => z.OuterFlag()),
                    CustomerFrom = x.Link<string, Customer>("F", "name", z => z.LinkProperty<Customer>(y => y.Name)),
                    CustomerTo = x.Link<string, Customer>("T", "name", z => z.LinkProperty<Customer>(y => y.Name)),
                    DeliveryType = x.Field<DeliveryTypeEnum>("DD", string.Empty),
                    DeliveryDate = x.Field<DateTime?>("DD", string.Empty, z => z.Default(DefaultType.CurrentDate))
                }, x => x.ExternalLink<InvoiceSpec>("DocId")).SetBaseTable("D").SetLogicAssembly("XDataObjectTest")
        );
	}
...    
   public partial class Invoice: IDataObject
   {
        public string DocStateCode { get { return this.GetProperty(x => x.DocStateCode); } }
        public Link<string, DocState> DocState { get; set; }
        public Link<string, Catalogue> DocCatalog { get; set; }
        public string DocNumb { get; set; }
        public DateTime? DocDate { get; set; }
        public long Generation { get { return this.GetProperty(x => x.Generation); } }
        public DateTime Changed { get { return this.GetProperty(x => x.Changed); } }
        public string Author { get { return this.GetProperty(x => x.Author); } }
        public decimal? DocAmount { get { return this.GetProperty(x => x.DocAmount); } }
        public DateTime? DocLastChange { get { return this.GetProperty(x => x.DocLastChange); } }
        public Lob Scan { get; set; }
        public Xml Source { get; set; }
        public Link<string, Customer> CustomerFrom { get; set; }
        public Link<string, Customer> CustomerTo { get; set; }
        public DeliveryTypeEnum DeliveryType { get; set; }
        public DateTime? DeliveryDate { get; set; }

	    public IRepository<InvoiceSpec> Spec { get { return this.GetRepository().GetChild<InvoiceSpec>(); } }
   }
```
Следует отметить, что динамическое описание позволяет более широкие возможности для описания правил преобразования в силу того, что оно лишено ограничений использования аттрибутов (Attribute). Имеется возможность использования динамического описания подчиненных запросов и внутренних представлений (inner view) с помощью механизма динамических запросов (см. ["Динамические запросы"](./queries.md)). Имеется возможность описания SQL выражений в стиле LINQ непосредственно по месту описания поля или фильтра на нем основанного, что несомненно повышает наглядность и читаемость преобразования.

Кроме того динамическое описание можно вынести в частичное (partial) описание класса и тем самым разделить описание собственно преобразования и необходимых для него полей объекта (пользовательских действий (или их описателей), фильтров времени исполнения, триггерной логики) от основной логики работы объекта.

Динамическое описание оформляется ввиде приватного статичного поля типа *Expression< CustomMapping < Invoice > >* и его значение инициализируется делегатом который задается вызовом статического метода *XDataManager*.**CustomMapping<Invoice>()** который возвращает ссылку на интерфейс [*IRepositoryStructure<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/20.html) который является основой для описания структуры [репозитория](./glossary.md#Репозиторий). Далее в стиле LINQ выражений выстраивается последовательность вызовов методов аналогичных атрибутам описанным для [статического описания](./static.md). 

####Описание параметров объектного отражения
Для указания параметров динамически отраженных классов служат методы интерфейса [*IRepositoryDescription<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/17.html): 
	- [SetBaseTable](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/167.html) - для указания [базовой таблицы](./glossary.md#Базовая-таблица), (опционально, по умолчанию будет использоваться [виртуальная таблица](./glossary.md#Виртуальная-таблица) с пустым псевдонимом)
	- [SetContext](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/168.html) - [контекст](./glossary.md#Контекст) базы данных (опционально, по умолчанию отображение может быть выполнено для любого контекста)
	- [SetFlags](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/169.html) - [набор флагов источника данных](./glossary.md#Набор-флагов-источника-данных) (опционально, по умолчанию **None**)
	- [SetLogicAssembly](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/170.html) - для указания полного имени сборки в которой реализована логика обработки данных (см. [Использование 3-х уровневой архитектуры](./three_tier.md) и [*IDataLogic<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/55.html))

Для получения ссылки на интерфейс [*IRepositoryDescription<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/17.html) необходимо описать структуру отображения полей [репозитория](./glossary.md#Репозиторий) с помощью метода интерфейса [*IRepositoryStructure<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/20.html) [*Map*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/124.html)
Например:
```csharp
...
	.Map(x => new Invoice {
	    DocState = x.Link<string, DocState>("S", "name",
	        z => z.LinkProperty<DocState>(y => y.Name),
	        z => z.LinkProperty<DocState>(y => y.Code, y => y.DocStateCode)),
	    DocCatalog = x.Link<string, Catalogue>("U", "name", z => z.LinkProperty<Catalogue>(y => y.Name)),
	    DocNumb = x.Field<string>("N", "numb"),
	    DocDate = x.Field<DateTime?>("D", string.Empty, z => z.Default(DefaultType.CurrentDate)),
	    Scan = x.Lob("SC", z => z.OuterFlag()),
	    Source = x.Xml("SR", z => z.OuterFlag()),
	    CustomerFrom = x.Link<string, Customer>("F", "name", z => z.LinkProperty<Customer>(y => y.Name)),
	    CustomerTo = x.Link<string, Customer>("T", "name", z => z.LinkProperty<Customer>(y => y.Name)),
	    DeliveryType = x.Field<DeliveryTypeEnum>("DD", string.Empty),
	    DeliveryDate = x.Field<DateTime?>("DD", string.Empty, z => z.Default(DefaultType.CurrentDate))
	}, x => x.ExternalLink<InvoiceSpec>("DocId")).SetBaseTable("D").SetLogicAssembly("XDataObjectTest")
...	
```
Подробности описания отражения свойств репозитория описаны [ниже](#Описание-преобразования-свойств).

Все таблицы участвующие в запросе [источника данных](./glossary.md#Источник-данных) описываются с помощью вызовов одной из перегрузок метода [*DataTable*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/179.html) интерфейса [*IRepositoryDescription<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/17.html). Те из таблиц, которые участвуют в [иерархии обновляемых таблиц](./glossary.md#Иерархия-обновляемых-таблиц) (кроме базовой таблицы) должны использовать специальную [перегрузку](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/191.html) с помощью которой можно указать псевдоним родительской таблицы.

```csharp
...
	.DataTable("T_DOC", "D", x => x.DictFilter("T_DOC_TYPE", "doc_type_id", "code", "INVOICE"))
        .DataTable("T_DOC_DOC_STATE", "DS", "D", x => x.Link("D", "doc_id"))
...
```
Кроме таблиц подобным образом могут описываться представления (view), однако все используемые поля представления должны быть описаны явно в виде свойств только для чтения (см. [Свойства только для чтения](#Свойства-только-для-чтения)) или скрытых полей (см. [Скрытые поля](#Скрытые-поля)).

Отдельно нужно отметить, что фильтры и связи между таблицами описываются в виде значений параметра filters. Детально описание фильтров при динамическом описании правил отображения объектов рассмотрено [ниже](#Фильтры).

####Описание преобразования с использованием подзапросов
Есть возможность использовать в объектном преобразовании подзапросы. Такая возможность есть как для отражения SQL выражений представляющих собой подзапрос на свойства объектов данных (см. ниже), для описания фильтров использующих подзапросы (см. [Фильтры на основе SQL выражений](#Фильтры-на-основе-SQL-выражений)), так и в качестве описаний внутренних подзапросов (inner view).

Глубина вложенности подзапросов не регламентируется.

В описании объектного преобразования основного класса (класса который использует этот подзапрос) подчитенные запросы указываются с помощью вызовов одной из перегрузок метода [*Subquery* или *InnerView*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/index.html) интерфейса [*IRepositoryDescription<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/17.html) для подзапросов и внутренних представлений (inner view) соответственно.

В случае когда используется параметризованные (generic) варианты методов *Subquery* и *InnerView* с параметром *IQueryDescription<TDobj> sub* имеется возможность использовать нотацию [динамических запросов](./queries.md) для описания структуры подзапросов непосредственно в ходе описания правила отображения внешнего запроса.

```csharp
...
	.Subquery("A", typeof(DocSpecAmounts), "Amount", DataGrouping.Sum, x => x.SubqueryLink("DocId"))
        .Subquery<CatalogueTree>("UT", x => x.CatalogueId, DataGrouping.None)
        .InnerView("H", XDataManager.GetStructure("H", null, DataStructureFlag.Grouping, new Variable[0])
            .DataTable("T_DOC_HISTORY", "H")
            .Select(x => new {
                DocId = x.Field<long?>("H", string.Empty, z => z.Group(DataGrouping.None)),
                HistoryDate = x.Field<DateTime?>("H", string.Empty, z => z.Group(DataGrouping.Max))
            }), x => x.SubqueryLink("DocId").SetOperation(FilterOperation.OuterJoin))
        .InnerView<DocBySpecType>("ST", x => x.SubqueryLink("DocId"))
...        
```
####Фильтры
Фильтры внутри основного запроса объекта данных могут быть описаны как:
* Константные фильтры
* Фильтры на основе SQL выражений
* Фильтры на основе кодов справочников
* Фильтры на основе списка значений
* Фильтры по подзапросу
* Связи внутри объекта данных

Каждый из них описывается соответствующим LINQ выражением (LINQ expression) описывающим параметры описания фильтров в зависимости от контекста их использования и типа самого фильтра (см. ниже). Описания фильтров передаются в качестве значений параметра *filters* методов [*DataTable*, *Subquery*, *InnerView*, *Procedure* интерфейса *IRepositoryDescription<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/177.html) или [*WithRecursive* интерфейса *IQueryWithAdapter<TRoot>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/199.html). При этом каждое значение описывает один фильтр. Тип фильтра определяется вызовом одного из фабричных методов интерфейса одного из нижеперечисленных адаптеров внутри лямбда выражения с входящим параметром типа одного из адаптеров:
	- [*IFilterAdapter*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/87.html) - базовый класс перечисленных ниже адаптеров для описания фильтров,
	- [*IInnerFilterAdapter*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/91.html) - который используется для описания фильтров и связей внутри запроса (*DataTable*, *Procedure*),
	- [*ISubqueryFilterAdapter*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/109.html), [*ISubqueryFilterAdapter<TDObj>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/113.html) и [*ISubqueryFilterAdapter<T,TDObj>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/159.html) - которые применяются в различных перегрузках методов [описания подзапросов](#Описание-преобразования-с-использованием подзапросов) для описания связей с подзапросами.

При этом заполняются обязательные параметры описания фильтров, но есть возможность доопределить описания опциональными параметрами путем продолжения цепочки LINQ вызовов методов расширения:
	- [*SetOperation*]() - для указания [операции фильтра](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/36.html)
	- [*AsPrimary*]() - для указания опционального фильтра
	- [*SetCombination*]()
	- [*SetSourceAlias*]()

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

Для всех из них должны быть определены следующие параметры: *Source* - псевдоним (alias) источника данных к полю которого применяется фильтр, *FieldName* - имя поля, *Operation* - операция фильтра (опционально, по умолчанию *FilterOperation*.**Equal**), *Combination* - имя группы фильтров, к которой относится данный фильтр (опционально, по умолчанию группа фильтров по умолчанию). Кроме этого каждый фильтр имеет свои специфичные параметры описанные в соответствующих разделах ниже.

Список операций представлен перечислением [*FilterOperation*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/36.html):
* **Equal** - равно
* **NotEqual** - не равно
* **LessThan** - меньше чем
* **GreaterThan** - больше чем
* **LessThanOrEqual** - меньше или равно
* **GreaterThanOrEqual** - больше или равно
* **In** - значение входит в список (или подзапрос)
* **NotIn** - значение не входит в список (или подзапрос)
* **Exists** - подзапрос возвращает одну или более строк
* **NotExists** - подзапрос не возвращает ни одной строки
* **Like** - найдено соответствие при поиске по подстроке
* **NotLike** - не найдено соответствий при поиске по подстроке
* **OuterJoin** - необязательная связь между источниками запроса (таблицами и внутренними предствлениями (inner view))
* **Contains** - найдено соответствие при полнотекстовом поиске
* **NotContains** - не найдено соответствие при полнотекстовом поиске

#####Группы фильтров
[Группы фильтров](./glossary.md#Группа-фильтров) могут быть описаны с помощью атрибута [*FilterCombinationAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/227.html) с параметрами: *Name* - уникальное имя группы фильтров, *Combination* - логическая операция объединения фильтров (и подчиненных групп фильтров) внутри описываемой группы фильтров (опционально, по умолчанию *FilterCombination*.**And**), *Parent* - имя родительской группы фильтров (опционально, для групп фильтров подчиненных группе фильтров по умолчанию может быть не указан).

Логическая операция связующая фильтры внутри группы фильтров описывается перечислением [*Combination*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/18.html):
* **AND** - И
* **OR** - ИЛИ
```csharp
[FilterCombination("OrGroup" /* имя группы фильтров */, 
	Combination: FilterCombination.OR /* логическая операция */)]
```

#####Константные фильтры
Фильтры с константным значением позволяют ограничить выборку объекта данных по значению некоторой константы. Использование такого фильтра описывается атрибутом [*ConstantFilterAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/222.html), со специфичными (относительно параметров описанных в пункте [Фильтры](#Фильтры)) параметрами: *Name* - имя фильтра (будет преобразовано в имя параметра запроса без учета префиксов согласно конкретного диалекта SQL), *ConstantType* - тип константы (см. ниже) и *ConstantValue* - значение константы (опционально в зависимости от *ConstantType*).

Список типов константы задается перечислением [*FilterConstantType*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/34.html):
* **Const** - указанное в поле *ConstantValue* значение.
* **Variable** - значение переменной (см. [Переменные](./tips_and_tricks.md/#Переменные)) с именем, указанным в поле *ConstantValue*.
* **CurrentDate** - сегодняшняя дата.
* **CurrentDateTime** - текущие дата и время.
* **CurrentDateTimeUTC** - текущие дата и время (UTC).
```csharp
[ConstantFilter("FilterByZero" /* имя фильтра */, 
	"P" /* псевдоним (alias) таблицы */, 
	"discount" /* имя поля БД */, 
	"0" /* значение константы */, 
	Operation: FilterOperation.Equal /* операция фильтра */, 
	ConstantType: FilterConstantType.Const /* тип константы */)]
```

#####Фильтры на основе SQL выражений
Для описания фильтров на основе SQL выражений используется атрибут [*ExpressionFilterAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/225.html) со специфичными (относительно параметров описанных в пункте [Фильтры](#Фильтры)) параметрами: *ExpressionText* - в зависимости от типа описания SQL выражения может быть: 
- псевдонимом подчиненного запроса,
```csharp
[ExpressionFilter("P" /* псевдоним (alias) таблицы */, 
	"discount" /* имя поля */, 
	"D" /* ExpressionText: псевдоним (alias) подзапроса */, 
	Operation: FilterOperation.In /* операция фильтра */)]
```
- именем статического поля с динамическим описанием SQL выражения,
```csharp
[ExpressionFilter("P" /* псевдоним (alias) таблицы */, 
	"discount" /* имя поля */, 
	"AllowedDiscount" /* ExpressionText: имя статичного поля - описания выражения */, 
	Operation: FilterOperation.NotEqual /* операция фильтра */)]
…
//Статическое поле отражаемого объекта помеченное атрибутом SqlExpression
[SqlExpression]
private static Calculate<int> AllowedDiscount = x => x.Case<Product, int>( 
	z => z.Field<bool>("is_vip"), z => 0, 1.SetValue(10));
```
- текстом SQL выражения.
```csharp
[ExpressionFilter("P" /* псевдоним (alias) таблицы */, 
	"discount" /* имя поля */, 
	"case P.is_vip when 1 then 10 else 0 end" /* ExpressionText: текст SQL выражения */, 
	Operation: FilterOperation.NotEqual /* операция фильтра */)]
```
> При возможности описать SQL выражение в виде динамического описания или
> подзапроса, не рекомендуется использовать непосредственно текст SQL выражения 
> в связи с тем, что оно будет специфично для конкретного диалекта SQL.

#####Фильтры на основе списка значений
Для описания фильтра проверки значения на вхождение в некоторый статичный список используется атрибут [*RangeFilterAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/235.html) со специфичным (относительно параметров описанных в [Фильтры](#Фильтры)) параметром: Range - массив значений списка.
```csharp
[RangeFilter("P" /* псевдоним (alias) таблицы */, 
	"discount" /* имя поля */, 
	new[] {0,5,10} /* список значений */, 
	Operation: FilterOperation.In /* операция фильтра */)]
```

#####Фильтры на основе кодов справочников
Суррогатные ключи справочников могут отличаться в различных инсталляциях, а названия обычно могут быть исправлены пользователями. Однако часто в бизнес логике необходимо оперировать конкретными значениями справочников. По этой причине рекомендуется при проектировании структуры базы данных снабжать справочники специальными уникальными текстовыми полями - мнемокодами.

Подсистема XData поддерживает использование мнемокодов при описании фильтров на основе значений справочников. Для этого используется атрибут [*DictionaryFilterAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/224.html) со специфичными (относительно параметров описанных в пункте [Фильтры](#Фильтры)) параметрами: *DictionaryTable* - имя справочной таблицы, *DictionaryId* - имя поля идентификатора справочной таблицы, *DictionaryCode* - имя поля мнемокода справочной таблицы, *DictionaryValue* - значение мнемокода по которому производится фильтрация, *ValueIsVariable* - признак того, что вместо конкретного значения в параметр *DictionaryValue* будет записано имя переменной (см.  [Переменные](./tips_and_tricks.md/#Переменные)) с нужным значением.

В случае совпадения имени идентификатора справочника и поля, по которому производится фильтрация параметр *FieldName* можно не задавать. Конструкцию есть возможность применять только для справочников с простым суррогатным ключом (из одного поля) и имеющих поле мнемокода.
```csharp
[DictionaryFilter("t_doc_state" /* имя справочной таблицы */, 
	"doc_state_id" /* имя поля идентификатора справочника */, 
	"code" /* имя поля мнемокода */, 
	"CREATED" /* значение мнемокода */, 
	"D" /* псевдоним (alias) таблицы */)]
```
#####Связи внутри объекта данных
Для описания связей внутри запроса объекта данных используется атрибут [*LinkAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/228.html) со специфичными параметрами: *LinkedSourceAlias* - псевдоним (alias) связанной таблицы, *LinkedFieldName* - имя поля в связанной таблице (опционально, может быть не указан при совпадении с *FieldName*). Связи внутри объекта данных могут быть использованы в описании опциональных фильтров (см. [Опциональные фильтры и опциональные подчиненные запросы](./tips_and_tricks.md#Опциональные-фильтры-и-опциональные-подчиненные-запросы)). Для указания того, что связь используется в определении необязательной части запроса может быть указан опциональный флаг первичного фильтра *PrimaryFilter*.
```csharp
[Link("D" /* псевдоним (alias) таблицы */, 
	"doc_id" /* имя поля БД */, 
	"DD" /* псевдоним (alias) связанной таблицы */)]
```

#####Фильтры по подзапросу
Для описания фильтров по подзапросу используется атрибут [*SubqueryFilterAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/237.html) со специфичным (относительно параметров описанных в пункте [Фильтры](#Фильтры)) параметром: *Subquery* - псевдоним (alias) подзапроса (см. "Описание объектного преобразования с использованием подзапросов "). При этом в качестве подзапроса для фильтра не может выступать внутреннее представление (inner view).
```csharp
[SubqueryFilter("D" /* псевдоним (alias) таблицы */, 
	"doc_state_id" /* имя поля */, 
	"A" /* псевдоним (alias) подзапроса */, 
	Operation: FilterOperation.In /* операция фильтра */)]
```
При указании значения флага *PrimaryFilter* логика вычисления границ опциональной части запроса работает следующим образом: если один из связываемых источников данных (таблица или представление (view)) будет "отброшен", то и источник данных связанный с ним будет отброшен. В случае если отбрасывается [базовая таблица](./glossary.md#Базовая-таблица) подзапроса - он будет "отброшен" полностью.

#####Связи с подзапросами
Связи с объектами данных - подзапросами оперируют не полями таблиц, а свойствами связываемых объектов данных, в том числе скрытыми (см. [Скрытые свойства объекта](#Скрытые-свойства-объекта)). Для описания связи с подзапросом или внутренним представлением (inner view) используется атрибут [*SubqueryLinkAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/238.html) с параметрами: *Subquery* - псевдоним (alias) подзапроса, *SubqueryProperty* - свойство в подзапросе по которому производится связывание, *PropertyName* - (опционально) свойство в основном объекте данных по которому производится связывание (если не указано считается, что свзывание производится по одноименному свойству *SubqueryProperty*), кроме того поддерживаются опциональные свойства *Operation*, *Combination* и *PrimaryFilter* описанные выше в пунктах [Фильтры](#Фильтры) и [Связи внутри объекта данных](#Связи-внутри-объекта-данных).
```csharp
[SubqueryLink("H" /* псевдоним (alias) подзапроса */, 
	"DocId" /* имя свойства подзапроса */)]
```
При указании значения флага *PrimaryFilter* логика вычисления границ опциональной части запроса работает следующим образом: если подчиненный запрос будет "отброшен", то и источник данных основного объекта связанный с ним будет отброшен и анализ будет продолжен по связям внутри объекта (см. [Связи внутри объекта данных](#Связи-внутри-объекта-данных)).

####Описание преобразования свойств
Преобразвания (mapping) свойств объекта при статическом описании оформляются с помощью указания атрибутов этого конкретного свойства. Основным атрибутом определяющим параметры преобразования является [*PropertyAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/310.html) с параметрами *Source* - псевдоним (alias) источника данных этого свойства (может быть не указан для [виртуальной таблицы](./glossary.md#Виртуальная-таблица)), *FieldName* - имя поля значение которого будет отражено в значение этого свойства (может не указываться в случае если имя поля и имя свойства соответствуют соглашению о наименовании (например, поле в БД назвается **some_field_name** и оно отражается в свойство объекта **SomeFieldName**)), *Flags* - опционально, битовая маска из [набора флагов свойства](./glossary.md#Набор-флагов-свойства),*NativeSqlType* - опционально, имя SQL типа в базе данных. Параметр *NativeSqlType* указывается в случае конфликта преобразования типа данных используемого по умолчанию.
```csharp
[Property("S" /* псевдоним (alias) таблицы */, 
	"code" /* имя поля */)]
```
#####Свойства на основании SQL выражений
В качестве источника данных для конкретного поля объекта данных, предназначенного только для чтения (см. [Свойства только для чтения](#Свойства-только-для-чтения)). Статическое описание SQL выражений в качестве источников оформляется с помощью атрибута [*PropertyExpressionAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/2/202.html) (Обратите внимание, что он дополняет, но не заменяет атрибут [*PropertyAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/310.html)) с параметрами: *ExprText* - в зависимости от значения *ExprType* (см. далее): псевдоним (alias) подзапроса / SQL выражение / имя свойства описывающего SQL выражение в стиле LINQ, *ExprType* - опционально, [тип описания SQL выражения](./glossary.md#Тип-описания-sql-выражения), по умолчанию *DataExpressionType*.**PlainSql**, *DbType* - тип данных на уровне ADO .Net провайдера (опционально, по умолчанию *DbType*.**String**), *ExprSize* - опционально, длина поля выражения (если применимо), *ExprScale* - опционально, десятичная точность поля выражения (если применимо). Возможно использовать SQL выражения трех видов: 
- подзапросы,
```csharp
[PropertyExpression("A", 
	DataExpressionType.SubQuery, 
	DbType.Decimal, 
	ExprSize = 17, 
	ExprScale = 5)]
```
- LINQ выражения,
```csharp
[PropertyExpression("AllowedDiscount", 
	DataExpressionType.LinqExpression, 
	DbType.Decimal, 
	ExprSize = 17, 
	ExprScale = 5)]
…
//Статическое поле отражаемого объекта помеченное атрибутом SqlExpression
[SqlExpression]
private static Calculate<int> AllowedDiscount = x => x.Case<Product, int>( 
	z => z.Field<bool>("is_vip"), z => 0, 1.SetValue(10));	
```
- текстовые SQL выражения.
```csharp
[PropertyExpression("case P.is_vip when 1 then 10 else 0 end", 
	DataExpressionType.PlainSql, 
	DbType.Decimal, 
	ExprSize = 17, 
	ExprScale = 5)]
```
> При возможности описать SQL выражение в виде динамического описания или
> подзапроса, не рекомендуется использовать непосредственно текст SQL выражения 
> в связи с тем, что оно будет специфично для конкретного диалекта SQL.

Обычно при описании свойств на основании SQL выражений рекомендуется использовать в качестве источника данных [виртуальную таблицу](./glossary.md#Виртуальная-таблица), не указывая псевдоним источника данных в параметре атрибута *PropertyAttribute*.

#####Значения по умолчанию для свойств
Для указания значения свойства по умолчанию при статическом описании используется атрибут [*PropertyDefaultAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/2/204.html) с параметрами: *DefaultSource* - [тип значения по умолчанию](./glossary.md#Тип-значения-свойства-по-умолчанию), *DefaultValue* опционально, в зависимости от значения *DefaultSource*, *AlwaysUseDefault* - опционально, по умолчанию - false, признак того, что значение по умолчанию будет использоваться не только при добавлении новой записи, но и при обновлении.
```csharp
[PropertyDefault(DefaultType.UserName, AlwaysUseDefault = true)]
```

#####Парамеры группировки свойства
Параметры группировки используются в случае описания группированного (group by) объекта для указания использования поля в группировке. Для этого служит атрибут свойства [*PropertyGroupingAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/234.html) с параметрами: *Grouping* - опционально, описывает [тип агрегации](./glossary.md#Тип-агрегации), по умолчанию **None**, *GroupOrder* - опционально, указывает порядок в группировке при условии *Grouping* = *DataGrouping*.**None**.
```csharp
[PropertyGrouping(DataGrouping.Sum)]
```

#####Скрытые свойства
В случае статического описания правил преобразования объекта, для описания [скрытых свойств](./mapping.md#Скрытые-свойства) (среди атрибутов класса) добавляются атрибуты отвечающие за описание такго свойства [*ColumnAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/311.html), [*ColumnExpressionAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/2/203.html) и [*ColumnDefaultAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/2/205.html) по аналогии с подобными атрибутами свойств.

Аттрибут [*ColumnAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/311.html) имеет параметры: *PropertyName* - имя свойства, *PropertyType* - тип свойства, *Source* - псевдоним (alias) испточника данных, *FieldName* - опционально, имя поля в БД (может не указываться в случае если имя поля и имя свойства соответствуют соглашению о наименовании (например, поле в БД назвается **some_field_name** и оно отражается в свойство объекта **SomeFieldName**)), *Flags* - опционально, опционально, битовая маска из [набора флагов свойства](./glossary.md#Набор-флагов-свойства), *Grouping* - опционально, описывает [тип агрегации](./glossary.md#Тип-агрегации), по умолчанию **None**, *GroupOrder* - опционально, указывает порядок в группировке при условии *Grouping* = *DataGrouping*.**None**, *Hidden* - опционально, указывает на отсутствие поля в выражении SELECT результирующего запроса (обычно обусловлено ограничениями накладываемыми на SQL при группировке), по умолчанию false, *NativeSqlType* - опционально, имя SQL типа в базе данных. Параметр *NativeSqlType* указывается в случае конфликта преобразования типа данных используемого по умолчанию.

Аттрибут [*ColumnExpressionAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/2/203.html) с параметрами: *PropertyName* - имя свойства (свойство должно быть описано с помощью атрибута *ColumnAttribute*), *ExprText* - в зависимости от значения *ExprType* (см. далее): псевдоним (alias) подзапроса / SQL выражение / имя свойства описывающего SQL выражение в стиле LINQ, *ExprType* - опционально, [тип описания SQL выражения](./glossary.md#Тип-значения-свойства-по-умолчанию) по умолчанию *DataExpressionType*.**PlainSql**, *DbType* - тип данных на уровне ADO .Net провайдера (опционально, по умолчанию *DbType*.**String**), *ExprSize* - опционально, длина поля выражения (если применимо), *ExprScale* - опционально, десятичная точность поля выражения (если применимо).

Аттрибут [*ColumnDefaultAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/2/205.html) с параметрами: *PropertyName* - имя свойства (свойство должно быть описано с помощью атрибута *ColumnAttribute*), *DefaultSource* - [тип значения по умолчанию](./glossary.md#Тип-значения-свойства-по-умолчанию), *DefaultValue* опционально, в зависимости от значения *DefaultSource*, *AlwaysUseDefault* - опционально, по умолчанию - false, признак того, что значение по умолчанию будет использоваться не только при добавлении новой записи, но и при обновлении.
```csharp
[Column("DocId", typeof(long?), "D", Flags = DataPropertyFlag.Id),
	ColumnDefault("DocId", DefaultType.AutoIncrement)]
```
#####Свойства внутренних представлений
Для ссылки из основоного объекта на свойство внутреннего представления вместо [*PropertyAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/310.html) используется [*ReferenceAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/2/206.html) с параметрами *Source* - псевдоним (alias) внутреннего представления, *PropertyName* - опционально, имя свойства во внутреннем представлении (может не указываться в случае если имя свойтсва во внутреннем представлении и имя свойства в основном объекте совпадают), *Flags* - опционально, битовая маска из [набора флагов свойства](./glossary.md#Набор-флагов-свойства),*NativeSqlType* - опционально, имя SQL типа в базе данных. Параметр *NativeSqlType* указывается в случае конфликта преобразования типа данных используемого по умолчанию.
```csharp
[Reference("H" /* псевдоним (alias) внутреннего представления (inner view) */, 
	"HistoryDate" /* имя свойства во внутреннем представлении */)]
```

#####Описание ссылок
Общее описание ссылок представлено [ранее](./mapping.md#Свойства---ссылки-на-другие-объекты). Здесь же остановимся на особенностях описания ссылок в случае статического описания объектного преобразования. Для определения связей между свойствами текущего объекта и свойствами объекта на который ведет ссылка используется набор атрибутов свойства - ссылки (по одному на каждую пару связанных свойств) [*LinkPropertyAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/229.html) с параметрами *DictSource* - опционально, наименование свойства в ссылочном объекте, *Property* - опционально, наименование свойства в основном объекте. При интерпретации данных работают следующие правила:
* Если не заданы оба свойства - используется имя свойства для которого и определен этот атрибут в основном объекте и считается, что источником данных является одноименное свойство в ссылочном объекте
* Если не задано свойство *DictSource* -  используется свойство *Property* в основном объекте и считается, что источником данных является одноименное *Property* свойство в ссылочном объекте
* Если не задано свойство *Property* -   используется имя свойства для которого и определен этот атрибут в основном объекте и  источником данных принимается свойство *DictSource* в ссылочном объекте
```csharp
[Property("S", "name"),
    LinkProperty("Name") /* поле DocState.Name источник для DocState текущего объекта */,
    LinkProperty("Code", "DocStateCode")  /* поле DocState.Code источник для DocStateCode текущего объекта */]
public Link<string, DocState> DocState { get; set; }
```

####Внешние связи
[Внешние связи](./mapping.md#Внешние-связи) в случае использования статического описания объектного преобразования оформляются с помощью атрибута [*ExternalLinkAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/333.html) с параметрами: *ChildType* - тип зависимого [репозитория](./glossary.md#Репозиторий), *ChildProperty* - имя свойства для связи в зависимом (slave) репозитории, *Property* - опционально, свойство для связи в главном (master) репозитории, если не указано считается равным *ChildProperty*, *FilterName* - опционально, имя фильтра в зависимом репозитории, если не указано принимается равным **"FilterBy"** + *ChildProperty*, *MasterRefresh* - опционально, признак того, что необходимо обновить запись в главном репозитории при изменении данных в зависимом (это используется, например, если объект главного репозитория содержит расчетные данные на основании исходных таблиц зависимого репозитория), по умолчанию **false**, *DirectLink* - опционально, признак "прямой связи" (см. [Особенности работы с дочерними репозториями иерархической сущности](./tips_and_tricks.md#Особенности-работы-с-дочерними-репозториями-иерархической-сущности)), по умолчанию **false** *Operation* - опционально, операция фильтра зависимого объекта (см. [Фильтры](#Фильтры)), по умолчанию *FilterOperation*.**Equal**, *Nullable* - опционально, [метод обработки пустых значений](./glossary.md#Методы-обработки-пустых-значений-фильтров), по умолчанию *FilterNullable*.**Nullable**, *Combination* - опционально, [имя группы фильтров](./glossary.md#Группа-фильтров) зависимого репозитория. Внешние связи могут быть использованы в описании опциональных фильтров (см. [Опциональные фильтры и опциональные подчиненные запросы](./tips_and_tricks.md#Опциональные-фильтры-и-опциональные-подчиненные-запросы)). Для указания того, что связь используется в определении необязательной части запроса может быть указан опциональный флаг первичного фильтра *PrimaryFilter*.
```csharp
[ExternalLink(typeof(InvoiceSpec), "DocId")]
```

####Поддержка псевдонимов источников данных
Описание псевдонимов и способы работы с ними отражены [в общей части описания](./mapping.md#Поддержка-псевдонимов-источников-данных), а здесь рассмотрим оформление статического описания псевдонимов.

Для объявления псевдонима плоского представления, а также базового подзапроса описания иерархического представления в случае статического описания объектного преобразования используется атрибут [*WithAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/239.html) с параметрами: *Alias* - псевдоним представления, *SubqueryType* - тип описывающий плоское представление или корневой подзапрос иерархического представления, *WithType* - опционально, операция объединения между подзапросами иерархического представления, используется только для описания иерархического представления описывается перечислением *WithRecursiveType* со значениями:
* **RecusiveUnion** - по умолчанию, операция объединения **UNION**
* **RecusiveUnionAll** - операция объединения **UNION ALL**
, *Properties* - упорядоченный набор свойств представления.

Для объявления рекурсивного подзапроса описания иерархического представления в случае статического описания объектного преобразования используется атрибут [*WithRecursiveAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/240.html) с параметрами:  *Alias* - псевдоним подзапроса, *SubqueryType* - тип описывающий подзапрос, *InitialAlias* - псевдоним иерархического представления.

Пример описания преобразования с использованием псевдонима плоского представления:
```csharp
/* Описание плоского представления S ссылается на подзапрос Sales */
[With("S", typeof(Sales), "Year", "Region", "Sales"),
	DataTable("S", "P") /* Продажи за прошлый год с псевдонимом P */,
	DataTable("S", "C") /* Продажи текущий год с псевдонимом C */]
```

Пример описания преобразования с использованием псевдонима иерархического представления:
```csharp
[With("CAT", typeof(CatalogueTreeRoot), WithRecursiveType.RecursiveUnion, 
	"CatalogueId", "Name", "Code", "ParentId"),
        WithRecursive("S", typeof(CatalogueTreeFolders), "CAT"),
        SubqueryLink("S", "ParentId", "CatalogueId")]
...
/* Описание корневого подзапроса */
[DataObject("R"),
DataTable("T_CATALOGUE", "R"),
Column("CatalogueId", typeof(long), "R"),
Column("Code", typeof(string), "R"),
Column("Name", typeof(string), "R"),
Column("ParentId", typeof(long?), "R"),
FilterCombination("root", Combination = Combination.Or),
ConstantFilter("FilterByRoot", "R", "catalogue_id", null,  
    Combination = "root", Nullable = FilterNullable.NullsNotAllowed),
ConstantFilter("FilterByRoot", "R", "parent_id", null, 
    Combination = "root", Nullable = FilterNullable.NullsCompared)]
public class CatalogueTreeRoot : ISqlObject {}
/* Описание рекурсивного подзапроса */
[DataObject("S"),
DataTable("T_CATALOGUE", "S"),
Column("CatalogueId", typeof(long), "S"),
Column("Code", typeof(string), "S"),
Column("Name", typeof(string), "S"),
Column("ParentId", typeof(long?), "S")]
public class CatalogueTreeFolders : ISqlObject {}
```

####Поддержка таблиц иерархии
Описание назначения таблиц иерархии отражены [в общей части описания](./mapping.md#Поддержка-таблиц-иерархии), а здесь рассмотрим оформление их статического описания.

Для статического описания таблицы иерархии используется атрибут [*HierarchyAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/2/207.html) с параметрами:
*Source* - псевдоним (alias) таблицы с рекурсивной ссылкой, *TreeTableName* - имя таблицы иерархии, *Parent* - имя поля рекурсивной ссылки в основной таблице, *LinkParent* - имя поля идентификатора корневого узла поддерева, *LinkChild* - имя поля узла поддерева.
```csharp
[Hierarchy("P", "l_rights_policy", "parent_policy_id", "parent_policy", "child_policy")]
```

####Описание объектного преобразования хранимых процедур и функций SQL
Общие сведения о возможностях использования хранимых процедур и функций SQL в XData отражены [в общей части описания](./mapping.md#Использование-хранимых-процедур-и-функций-sql), а здесь рассмотрим оформление их статического описания.

#####Описание хранимой процедуры как источника данных
Для описания хранимой процедуры как источника данных используется атрибут [*ProcedureAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/232.html) с параметрами: *Alias* - псевдоним (alias) источника данных, *Name* - имя процедуры или функции SQL, *ProcedureType* - перечисление *ProcedureType* указывает на то, является ли *Name* хранимой процедурой (*ProcedureType*.**Procedure**) или функцией (*ProcedureType*.**Function**).

#####Описание параметра хранимой процедуры
Для описания параметра хранимой процедуры используется атрибут [*ParameterAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/231.html) с параметрами: *Alias* - псевдоним (alias) источника данных, *Order* - порядковый номер параметра, *Binding* - имя под которым этот параметр будет доступен из программного кода (часто удобно когда имя параметра в коде будет иметь другую нотификацию нежели в БД), *Type* - тип который будет использоваться  для доступа к значению параметра в программном коде, *DbType* - тип данных на уровне ADO .Net провайдера, *Direction* - опционально, направление передачи данных параметра, по умолчанию *ParameterDirection*.**Input**, *Size* - опционально, длина значения параметра (если применимо), *Scale* - опционально, тесятичная точность значения параметра (если применимо), *Name* - опционально, имя параметра, по умолчанию равно *Binding*, *DefaultType* - [тип значения по умолчанию](./glossary.md#Тип-значения-свойства-по-умолчанию), *DefaultValue* опционально, в зависимости от значения *DefaultType*, *NativeSqlType* - опционально, имя SQL типа в базе данных (*NativeSqlType* указывается в случае конфликта преобразования типа данных используемого по умолчанию), *UdtTypeName* - опционально, имя пользовательского типа данных SQL (UDT), *UdtElementTypeName* - опционально, имя пользовательского типа данных SQL (UDT) элемента коллекции *UdtTypeName* (в случае если *UdtTypeName* - коллекция), *IsArray* - опционально, признак того, что параметр является массивом элементов *UdtElementTypeName* (если *UdtElementTypeName* не указан, массивом элементов *DbType*), по умолчанию - **false**.

#####Описание возвращаемого набора данных
Для описания набора данных возвращаемого хранимой процедурой используется атрибут [*ResultSetAttribute*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/3/236.html) с параметрами: *Alias* - псевдоним (alias) источника данных - хранимой процедуры SQL, *Name* - имя возвращаемого набора данных под которым он будет доступен из программного кода, *Order* - опционально, порядковый номер возвращаемого набора данных (для хранимых процедур возвращающих несколько наборов данных указание этого параметра - обязательно), *ResultType* - опционально, тип который будет использован для отражения элементов возвращаемого набора данных, по умолчанию используется тип которому назначен этот атрибут, *IsDefault* - опционально, признак возвращаемого набора данных по умолчанию (используется только для хранимых процедур возвращающих несколько наборов данных), по умолчанию **false**.

#####Примеры описания объектного преобразования хранимых процедур и функций SQL
* Использование в качестве источника данных набора записей возвращаемый хранимой процедурой
```csharp
[DataObject("T"),
	Procedure("T", "dbo.TestProcedure", ProcedureType.Procedure),
	Parameter("T", 1, "param1", typeof(int), DbType.Int32),
	Parameter("T", 2, "param2", typeof(string), DbType.String),
	ResultSet("T", "Result")]
public class TestProcedure : IDataObject
{
	[Property("T", "Id", Flags = DataPropertyFlag.Id)]
	public int Id { get { return this.GetProperty(x => x.Id); } }
	[Property("T", "Name")]
	public string Name { get { return this.GetProperty(x => x.Name); } }
}
```
* Использование хранимых процедур возвращающих несколько наборов данных
```csharp
[DataObject("T"),
	Procedure("T", "dbo.TestProcedure2", ProcedureType.Procedure),
	Parameter("T", 1, "param1", typeof(int), DbType.Int32),
	Parameter("T", 2, "param2", typeof(string), DbType.String),
	Parameter("T", 3, "param3", typeof(int), DbType.Int32),
	Parameter("T", 4, "param4", typeof(string), DbType.String),
	ResultSet("T", "Result", IsDefault = true),
	ResultSet("T", "SecondResult", ResultType = typeof(TestResult), Order = 2)]
public class TestProcedure2 : IDataObject
{
	[Property("T", "Id", Flags = DataPropertyFlag.Id)]
	public int Id { get { return this.GetProperty(x => x.Id); } }
	[Property("T", "Name")]
	public string Name { get { return this.GetProperty(x => x.Name); } }
	public IEnumerable<TestResult> SecondResult 
		{ get { return this.GetResultSet(x => x.SecondResult); } }
}
```
* Использование возвращаемого (out) параметра хранимой процедуры
```csharp
[DataObject("T"),
        Procedure("T", "dbo.TestProcedure3", ProcedureType.Procedure),
        Parameter("T", 1, "param1", typeof(int), DbType.Int32),
        Parameter("T", 2, "param2", typeof(string), DbType.String),
        Parameter("T", 3, "OutParameter", typeof(string), 
        	DbType.String, 
        	Size = 20, 
        	Direction = ParameterDirection.Output, 
        	Name = "param3")]
public class TestProcedure3 : IDataObject
{
	public string OutParameter 
		{ get { return this.GetParameter(x => x.OutParameter); } }
}
```
* Тоже самое совместно с возвращением набора записей
```csharp
[DataObject("T"),
        Procedure("T", "dbo.TestProcedure4", ProcedureType.Procedure),
        Parameter("T", 1, "param1", typeof(int), DbType.Int32),
        Parameter("T", 2, "param2", typeof(string), DbType.String),
        Parameter("T", 3, "OutParameter", typeof(string), 
        	DbType.String, 
        	Size = 20, 
        	Direction = ParameterDirection.Output, 
        	Name = "param3"),
        ResultSet("T", "Result")]
public class TestProcedure4 : IDataObject
{
	[Property("T", "Id", Flags = DataPropertyFlag.Id)]
	public int Id { get { return this.GetProperty(x => x.Id); } }
	[Property("T", "Name")]
	public string Name { get { return this.GetProperty(x => x.Name); } }
	public string OutParameter 
		{ get { return this.GetParameter(x => x.OutParameter); } }
}
```
* Использование возвращаемого значения (return value) хранимой процедуры
```csharp
[DataObject("T"),
        Procedure("T", "dbo.TestProcedure5", ProcedureType.Procedure),
        Parameter("T", 1, "param1", typeof(int), DbType.Int32),
        Parameter("T", 2, "param2", typeof(string), DbType.String),
        Parameter("T", 3, "OutParameter", typeof(string), 
        	DbType.String, 
        	Size = 20, 
        	Direction = ParameterDirection.Output, 
        	Name = "param3")]
public class TestProcedure5 : IDataObject
{
	public string OutParameter 
		{ get { return this.GetParameter(x => x.OutParameter); } }
}
```
* Тоже самое совместно с возвращением набора записей
```csharp
[DataObject("T"),
	Procedure("T", "dbo.TestProcedure6", ProcedureType.Procedure),
	Parameter("T", 1, "param1", typeof(int), DbType.Int32),
	Parameter("T", 2, "param2", typeof(string), DbType.String),
	Parameter("T", 3, "ResultParameter", typeof(int), 
		DbType.Int32, 
		Direction = ParameterDirection.ReturnValue),
	ResultSet("T", "Result")]
public class TestProcedure6 : IDataObject
{
	[Property("T", "Id", Flags = DataPropertyFlag.Id)]
	public int Id { get { return this.GetProperty(x => x.Id); } }
	[Property("T", "Name")]
	public string Name { get { return this.GetProperty(x => x.Name); } }
	public int ResultParameter 
		{ get { return this.GetParameter(x => x.ResultParameter); } }
}
```
* Использование в качестве параметра хранимой процедуры массива простых типов
```csharp
[DataObject("T"),
	Procedure("T", "TestFunction", ProcedureType.Function),
	Parameter("T", 1, "param1", typeof(int[]), DbType.Int32, IsArray = true),
	Parameter("T", 2, "Result", typeof(int), DbType.Int32, 
		Direction = ParameterDirection.ReturnValue)]
public class TestFunction : IDataObject
{
	public int Result { get { return this.GetParameter(x => x.Result); } }
}
```
* Использование в качестве параметра хранимой процедуры набора записей
```csharp
/* тип отражаемый в UDT должен поддерживать 
	Xml сериализацию стандартными методами */
[Serializable]
public class Classifier
{
	[XmlAttribute]
	public int Id { get; set; }
	[XmlAttribute]
	public string Name { get; set; }
}
...
[DataObject("T"),
	Procedure("T", "dbo.TestProcedure7", ProcedureType.Procedure),
	Parameter("T", 1, "param1", 
		typeof(Classifier[]) /* тип параметра должен быть 
			массивом объектов отражаемых на UDT */, 
		DbType.Object /* для использования UDT тип на уровне 
			провайдера должен быть указан равным DbType.Object */, 
		UdtTypeName = "dbo.Classifier" /* рекомендуется 
			использовать полное имя типа */ ),
	ResultSet("T", "Result")]
public class TestProcedure7 : IDataObject
{
	[Property("T", "Id", Flags = DataPropertyFlag.Id)]
	public int Id { get { return this.GetProperty(x => x.Id); } }
	[Property("T", "Name")]
	public string Name { get { return this.GetProperty(x => x.Name); } }
}
```
* Использование в качестве источника данных скалярной функции
```csharp
[DataObject("T"),
	Procedure("T", "dbo.TestFunction2", ProcedureType.Function),
	Parameter("T", 1, "param1", typeof(int), DbType.Int32),
	Parameter("T", 2, "param2", typeof(string), DbType.String),
	Parameter("T", 3, "Result", typeof(string), DbType.String, Direction = ParameterDirection.ReturnValue)]
public class TestFunction2 : IDataObject
{
	public string Result { get { return this.GetParameter(x => x.Result); } }
}
```   
* Использование в качестве источника данных табличной функции
```csharp
[DataObject("T"),
	Procedure("T", "dbo.TestFunction3", ProcedureType.Function),
	Parameter("T", 1, "param1", typeof(int), DbType.Int32),
	Parameter("T", 2, "param2", typeof(string), DbType.String),
	ResultSet("T", "Result")]
public class TestFunction3 : IDataObject
{
	[Property("T", "Id", Flags = DataPropertyFlag.Id)]
	public int Id { get { return this.GetProperty(x => x.Id); } }
	[Property("T", "Name")]
	public string Name { get { return this.GetProperty(x => x.Name); } }
}
```   
* Использование скалярной функции в качестве источника данных свойства
```csharp
[DataObject("T"),
	DataTable("T_DOC_TYPE", "T"),
	Column("DocTypeId", typeof(long), Flags = DataPropertyFlag.Id)]
public class TestFunction4 : IDataObject
{
	[Property("T")]
	public Code { get; set; }
	[Property("T")]
	public Name { get; set; }
	[Property, 
		PropertyExpression("test", DataExpressionType.LinqExpression, ExprSize = 20)]
	public Test { get { return this.GetProperty(x => x.Test); } }
	//Статическое поле отражаемого объекта помеченное атрибутом SqlExpression
	[SqlExpression]
	private static Calculate<string> test z => z.SqlFn<string>("dbo.TestFunction2",
		y => y.Field<long>("T", "doc_type_id"), y => y.Field<string>("T", "name"))
}
```
