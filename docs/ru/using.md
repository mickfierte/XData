### Работа с данными
#### Получение источника данных
Для получения источника данных можно воспользоваться статическим классом XDataManager:
```csharp
var layerId = Guid.NewGuid();
var source = XDataManager.GetRepository<Invoice>(layerId);
```
При этом запрошенный однажды источник данных будет добавлен в кэш для указанного layerId и повторный запрос источника данных в том же уровне не приведет к созданию нового экземпляра источника данных.
#### Доступ к свойствам объекта
К изменяемым свойствам объекта можно обращаться как обычно:
```csharp
var docNumb = XDataManager.GetRepository<Invoice>(layerId).First().DocNumb;
```
Исключение составляют [ссылки (Link)](./mapping.md#Свойства---ссылки-на-другие-объекты) и [большие объекты (Lob и Xml)](./mapping.md#Свойства-типа---бинарный-объект), которые будут детально описаны ниже.
#### Изменение данных
Данные в объектах меняются прямым присваиванием значений редактируемых свойств. Здесь также исключение составляют [ссылки (Link)](./mapping.md#Свойства---ссылки-на-другие-объекты) и [большие объекты (Lob и Xml)](./mapping.md#Свойства-типа---бинарный-объект), которые будут детально описаны ниже. Здесь же приведем пример в котором это наглядно показано:
```csharp
var newInvoice = XDataManager.GetRepository<Invoice>(layer, context: Context, security: security).New();
//поле DocNumb - строка и значение присваивается как обычно
newInvoice.DocNumb = String.Format("СФ-{0}", number.ToString(CultureInfo.InvariantCulture).PadLeft(6, '0'));
//поле DocCatalog - ссылка на объект типа Catalogue
//присвоение с помошью операции += эквивалентно DocCatalog.Source = ...
newInvoice.DocCatalog += XDataManager.GetDictionaryValue<Catalogue>(Owner, x => x.Code == catalogCode, context: Context);
//поле DeliveryType - перечисление и значение присваивается как обычно
newInvoice.DeliveryType = DeliveryTypeEnum.PickUp;
//поле DeliveryType - дата/время и значение присваивается как обычно
newInvoice.DeliveryDate = DateTime.Today.AddDays(1);
//поле Source - Xml и присвоение с помощью операции += эквивалентно
//Source.Document = ...
newInvoice.Source += new XDocument(new XElement("invoice", new XAttribute("number", number), new XAttribute("state", newInvoice.DocStateCode)));
//поле Source - бинарный объект (Lob) и присвоение с помощью 
//операции += эквивалентно Scan.Value = ...
newInvoice.Scan += Encoding.UTF8.GetBytes(newInvoice.Source.Document.ToString());
```
#### Удаление объектов
В XData принято удаление путем пометки объекта на удаление, при этом есть возможность до применения изменений в БД эту отметку снять. Пометка и снятие пометки осуществляется с помощью метода расширения интерфейса *IDataObject* **SetDeleted**
```csharp
invoice.SetDeleted(true); //для пометки на удаление
...
invoice.SetDeleted(false); //для снятия пометки на удаление
```
#### Применение изменений
Применить сделанные изменения в отдельном объекте можно с помощью метода расширения интерфейса *IDataObject* **Submit**
```csharp
newInvoice.Submit();
```
Если требуется применить изменения в нескольких объектах налогичный метод есть в интерфейсе *IRepository<T>* **Submit** отличие только в том, что он принимает в качестве параметра последовательность объектов этого репозитория (*IEnumerable<T>*). В этом случае, если объект не был изменен он будет пропущен при применении изменений в БД.
#### Закрытие слоя
После завершения использования слоя настоятельно рекомендуется его закрыть с помощью команды:
```csharp
XDataManager.Close(layerId);
```
Это позволит освободить память в кэше источников данных.

#### Закрытие сессии работы с данными

Команда *XDataManager* **Close**:
```csharp
XDataManager.Close();
```
позволяет закрыть все открытые слои.
