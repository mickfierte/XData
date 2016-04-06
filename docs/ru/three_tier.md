##Использование 3-х уровневой архитектуры
Одной из отличительных свойств XData является возможность работы систем с ее использованием как в 2-х уровневой архитектуре (клиент - сервер базы данных), так и в 3-х уровневой архитектуре (клиент - сервер приложений - сервер базы данных). При этом существует уникальная возможность **переключения** системы реализованной в парадигме **3-х уровневой архитектуры в 2-х уровневую** на уровне **файлов конфигурации**!!! Эта особенность позволяет производить отладку сложных многоуровневых приложений с полным стеком вызовов и прочим комфортом... Кто производил отладку кода на сервере приложений, тот понимает о чем речь...

>**Возьмите на заметку**: В 3-звенном режиме работы наличие установленного провайдера БД на клиентском компютере не требуется!

Для выделения части логики системы с использованием XData на уровень сервера приложений необходимо: 
* выделить отраженные классы используемые клиентской стороной в отдельную сборку (сборку модели) или несколько сборок если это оправдано логикой приложения
* выделить серверную логику в отдельную сборку (серверную сборку)или в несколько сборок если это оправдано логикой приложения
* на уровне серверной сборки по мере необходимоти создать классы реализующие интерфейс - маркер (пустой интерфейс) [*IDataLogic<T>*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/54.html), где T - отраженнный класс который использует серверную логику (в том числе [триггерную логику](./tips_and_tricks.md#Использование-триггерной-логики)) и оформить вызовы серверной логики из обработчиков пользовательской логики
```csharp
public abstract class InvoiceLogic : IDataLogic<Invoice>
{
    [Action(DataActionType.AfterInsert), Action(DataActionType.AfterUpdate)]
    public readonly static Trigger<Invoice> UpdateHistory = 
    ((ref Invoice invoice, ref DataTriggerFlag flag) =>
    {
        if (!invoice.CheckState(DataObjectState.New) && !invoice.IsChanged(x => x.DocState)) return true;
        var rep = invoice.GetRepository();
        var hist = XDataManager.GetRepository<DocHistory>(rep.Layer, context: rep.Context)
            .Reset()
            .SetFilterValue(DocHistory.FilterByDocId, invoice.GetProperty<long>("DocId"))
            .SetFilterValue(DocHistory.FilterByDocStateId, invoice.DocState.Key);
        var newHist = hist.New();
        newHist.HistoryDate = DateTime.UtcNow;
        return hist.Submit(ref newHist);
    });

    [Action(DataActionType.BeforeDelete)]
    public readonly static Trigger<Invoice> ClearHistory = 
    ((ref Invoice invoice, ref DataTriggerFlag flag) => 
    {
        var i = invoice;
        return XDataManager.GetRepository<DocHistory>(i.GetLayer(), context: i.GetContext()).Reset()
            .Clear(x => x.GetProperty<long>("DocId") == i.GetProperty<long>("DocId"));
    });

    [Action(DataActionType.BeforeClear)]
    public readonly static RepositoryTrigger<Invoice> ClearHistoryBatch = 
    ((IRepository<Invoice> invoiceRepository, ref DataTriggerFlag flag) =>
        XDataManager.GetRepository<DocHistory>(invoiceRepository.Layer, context: invoiceRepository.Context).Reset()
            .Clear(x => invoiceRepository.Any(z => x.GetProperty<long>("DocId") == z.GetProperty<long>("DocId"))));

    public static readonly CustomLogic<Invoice> TestCustomLogic = (objects => 
    {
        Log.Write(MessageType.Information, 
          () => String.Format("TestCustomLogic called with {0} objects", objects.Length)); 
        foreach (var invoice in objects)
        {
            var i = invoice;
            invoice.PostData("testPost", () => Encoding.UTF8.GetBytes(i.DocNumb));
            var p = Encoding.UTF8.GetBytes(i.DocNumb);
            var r = i.Callback("testCall", ref p);
            Log.Write(MessageType.Information, 
              () => String.Format("Call for \"{0}\" returned \"{1}\" with data \"{2}\"", 
                i.DocNumb, r, p == null ? null : Encoding.UTF8.GetString(p)));
        }
        return true;
    });
}
```

* указать имя соответсвующей серверной сборки в описании объектного преобразования в сборке модели 
```csharp
[DataObject("D", LogicAssemblyName = "InvoiceServerLogic")] //для статического описания
//или
...
.SetLogicAssembly("InvoiceServerLogic")  //для динамического описания
```

* зарегистрировать обработчики пользовательской логики в соответсвуюших классах сборки модели
```csharp
public static CustomLogic<Invoice> TestCustomLogic;
```

* организовать вызовы [пользовательской логики](./tips_and_tricks.md#Использование-пользовательской-логики) из клиентского кода
```csharp
var random = new Random();
return XDataManager.GetRepository<Invoice>(layer, context: Context, security: security).ToArray().Execute(
() => Invoice.TestCustomLogic, 
  "testPost".SetValue((Action<byte[]>)(data => 
    Console.WriteLine("Post message received (data=\"{0}\")", data == null 
      ? null 
      : Encoding.UTF8.GetString(data)))).AsEnum().ToDictionary(),
  "testCall".SetValue((Func<byte[], byte[]>)(data =>
    {
        Console.WriteLine("Call received \"{0}\")", data == null ? null : Encoding.UTF8.GetString(data));
        return random.NextDouble() >= 0.5 
          ? null 
          : Encoding.UTF8.GetBytes(string.Format("reply for \"{0}\"", data == null 
            ? null 
            : Encoding.UTF8.GetString(data)));
    })).AsEnum().ToDictionary());
```

>Обратите внимание, что обработчики пользовательской логики позволяют организовать как синхронные так и асинхронные [обратные вызовы](./tips_and_tricks.md#Обратный-вызов-из-пользовательской-логики) и анализ результатов синхронных обратных вызовов. Такая функциональность позволяет реализовать с помощью обработчиков пользовательской логики и триггерной логики очень сложные (и в том числе интерактивные) сценарии реализации бизнес процессов. При этом важно отметить, что серверная логика становится прозрачной за счет цельного описания бизнес процесса в едином обработчике пользовательской логики.
