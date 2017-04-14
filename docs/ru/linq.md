## Поддержка LINQ
[Репозитории](./glossary.md#Репозиторий) и [динамические запросы](./queries.md) XData реализуют интерфейс [*IOrderedQueryable<T>*](https://msdn.microsoft.com/ru-ru/library/bb340178(v=vs.110).aspx) и поддерживают полный набор функций IQueriable.

>**ВАЖНОЕ ЗАМЕЧАНИЕ:** Методы *SkipWhile*, *TakeWhile*, *Join*, *Select* и *SelectMany* поддерживаются, но приводят к запросу исходных данных и выполняются уже на уровне Linq2Object. Это ограничение связано с невозможностью преобразования кода делегатов - обработчиков для первых двух в SQL и изменения результирующего типа для остальных. Для обеспечения эффективного доступа к данным на пересечении нескольких объектов предметной области рекомендуется использовать отдельное описание объектного преобразования, либо механизм [динамических запросов](./queries.md)

Кроме поддержки стандартных методов LINQ XData предоставляет расширения для удобной работы с внешними объединениями [*LeftOuterJoin*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/332.html), [*RightOuterJoin*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/334.html), [*FullOuterJoin*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/329.html), и маркер - методы расширения [*Inner*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/330.html) и [*Outer*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/333.html):
```csharp
Console.WriteLine("*** Inner join");
var innerJoined = XDataManager.GetRepository<Invoice>(Owner, context: Context)
    .Join(XDataManager.GetRepository<DocState>(Owner, context: Context),
        x => x.DocStateCode, y => y.Code, (x, y) => new { x.DocNumb, y.Code });
foreach (var z in innerJoined) Console.WriteLine("{0}:{1}", z.DocNumb, z.Code);
Console.WriteLine("*** LINQ-style outer join");
var invoice = XDataManager.GetRepository<Invoice>(Owner, context: Context);
var xx1 = XDataManager.GetRepository<DocState>(Owner, context: Context)
                .SelectMany(x => invoice.Where(y => x.Code == y.DocStateCode).DefaultIfEmpty(),
                    (x, y) => new { x.Code, x.Name, Another = y == null ? null : y.DocStateCode });
foreach (var z in xx1) Console.WriteLine("{0}:{1}:{2}", z.Code, z.Name, z.Another);
Console.WriteLine("*** Left outer join");
var leftOuterJoined = XDataManager.GetRepository<DocState>(Owner, 
  context: Context).LeftOuterJoin(XDataManager.GetRepository<Invoice>(Owner, context: Context),
    x => x.Code, x => x.DocStateCode, (x, y) => new { x.Code, x.Name, Another = y == null ? null : y.DocStateCode });
foreach (var z in leftOuterJoined) Console.WriteLine("{0}:{1}:{2}", z.Code, z.Name, z.Another);
Console.WriteLine("*** Left outer join using .Outer()");
var anotherLeftOuterJoined = XDataManager.GetRepository<DocState>(Owner, 
  context: Context).Outer().Join(XDataManager.GetRepository<Invoice>(Owner, context: Context),
    x => x.Code, x => x.DocStateCode, (x, y) => new { x.Code, x.Name, Another = y == null ? null : y.DocStateCode });
foreach (var z in anotherLeftOuterJoined) Console.WriteLine("{0}:{1}:{2}", z.Code, z.Name, z.Another);
Console.WriteLine("*** Full outer join");
var fullOuterJoined = XDataManager.GetRepository<DocState>(Owner, 
  context: Context).FullOuterJoin(XDataManager.GetRepository<Invoice>(Owner, context: Context),
    x => x.Code, x => x.DocStateCode, (x, y) => new { x.Code, x.Name, Another = y == null ? null : y.DocStateCode });
foreach (var z in fullOuterJoined) Console.WriteLine("{0}:{1}:{2}", z.Code, z.Name, z.Another);
Console.WriteLine("*** Cross join");
var other = XDataManager.GetRepository<DocType>(Owner, context: Context);
var crossJoined = XDataManager.GetRepository<DocSpecType>(Owner, context: Context)
  .SelectMany(x => other, (x, y) => new { x.Code, x.Name, Another = y.Code });
foreach (var z in crossJoined) Console.WriteLine("{0}:{1}:{2}", z.Code, z.Name, z.Another);
```
