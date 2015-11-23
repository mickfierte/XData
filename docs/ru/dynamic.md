>Извините. Статья еще находится в стадии разработки, но мы уже работаем над ней...

### Динамическое преобразование (dynamic mapping)
Наряду со статическим описанием объектного преобразования XData поддерживает возможность динамического описания правил преобразования в виде специфичного LINQ выражения. Ниже приведен пример динамического описания преобразования (mapping) объекта данных (того же самого который был описан в разделе "Статическое преобразование (static mapping)").
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
Следует отметить, что динамическое описание позволяет более широкие возможности для описания правил преобразования в силу того, что оно лишено ограничений использования аттрибутов (Attribute). Имеется возможность использования динамического описания подчиненных запросов и внутренних представлений (inner view) с помощью механизма динамических запросов (см. "Динамические запросы"). Имеется возможность описания SQL выражений в стиле LINQ непосредственно по месту описания поля или фильтра на нем основанного, что несомненно повышает наглядность и читаемость преобразования.

Кроме того динамическое описание можно вынести в частичное (partial) описание класса и тем самым разделить описание собственно преобразования и необходимых для него полей объекта (пользовательских действий (или их описателей), фильтров времени исполнения, триггерной логики) от основной логики работы объекта.

Динамическое описание оформляется ввиде приватного статичного поля типа *Expression<CustomMapping<Invoice>>* и его значение инициализируется делегатом который задается вызовом статического метода *XDataManager*.**CustomMapping<Invoice>()**. Далее в стиле LINQ выражений выстраивается последовательность вызовов методов аналогичных атрибутам описанным выше для статического описания. Параметры преобразования 
