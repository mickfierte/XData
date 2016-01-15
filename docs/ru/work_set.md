###Обработка взаимосвязанных объектов (Unit of Work)
Несмотря, на тот факт, что XData поддерживает работу со сложными бизнес объектами хранящимися в сложно связанном конгломерате таблиц, возникают ситуации, когда необходмо обеспечить транзакционную поддержку изменений на уровне уже связанного конгломерата бизнес объектов. 

Например у нас есть один бизнес объект верхнего уровня к которому привязаны одна или несколько коллекций подчиненных объектов (к которым в общем случае могут быть в свою очередь привязаны уже свои зависимые объекты). И существует бизнес логика по которой мы работаем с описанной структурой как с единым целым и должны сохранять в БД их изменения все вместе.

Возможно обрамить вызов этой логики в [транзакцию](./tips_and_triks.md#Управление-транзакциями), но при этом нам придется писать много не самого тривиального кода для поддержания коллекций объектов которые возможно еще не сохранены в БД.

Элегантным решением для этого, предлагаемым так же и в среди ORM средств, явлется реализация шаблона Unit of Work. Но в XData во-первых это решение применяется действительно на том уровне где его использование логично - для работы с конгломератом связанных экземпляров бизнес объектов, а во-вторых использование этого шалобна в XData декларативно (не требует от программиста бизнес логики реализации конкретного класса для использования этого шаблона).

Покажем это на конкретном примере:

Инициализация нового UoW конейнера для нового корневого объекта (insert) производится с помощью статичного метода [*Add*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/335.html) класса [*Work*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/323.html):
```csharp
var rep = XDataManager.GetRepository<Model.Patient>(Instance);
var instance = rep.New();
/* 
  С помошью статичного класса Work мы инициализируем UoW конейнер новым корневым объектом 
  и указываем какие коллекции его подчиненных объектов будут входить в UoW конейнер
  При этом так как объект у нас пустой коллекции мы не читаем (используем Work.Empty)
*/
using (var work = Work.Add(instance, 
    Work.Empty<Model.Patient, Model.PatientDiagnosis>(x => x.PatientDiagnosisList),
    Work.Empty<Model.Patient, Model.PatientOper>(x => x.PatientOperList)))
{
    if (new PatientWindow { DataContext = new Patient(work), Owner = window }.ShowDialog() == true)
        work.Submit(); // применяются измения к UoW конейнеру
}
```

Инициализация нового UoW конейнера для существующего корневого объекта (update):
```csharp
/* 
  С помошью статичного класса Work мы инициализируем UoW конейнер существующим корневым 
  объектом и указываем какие коллекции его подчиненных объектов будут входить в UoW конейнер
  При этом так как объект у нас уже существует коллекции мы читаем из БД (используем Work.Fill)
*/
using (var work = Work.Add(patient, 
    Work.Fill<Model.Patient, Model.PatientDiagnosis>(x => x.PatientDiagnosisList),
    Work.Fill<Model.Patient, Model.PatientOper>(x => x.PatientOperList)))
{
    if (new PatientWindow { DataContext = new Patient(work), Owner = window }.ShowDialog() != true) return;
    work.Submit(); // применяются измения к UoW конейнеру
}
```

Инициализация единичного подчиненного объекта в UoW конейнере (в том случае когда свойство в основном объекте не коллекция, а единичный экземляр) производится с помощью статичного метода [*Get*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/338.html) класса [*Work*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/323.html). Его использование аналогично методам [*Empty*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/336.html) и [*Fill*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/337.html).

UoW конейнер можно также инциализировать коллекцией корневых объектов. Для этого используется соответствующая перегрузка метода [*Add*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/340.html) класса [*Work*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/323.html) которая принимает в качестве первого параметра последовательность (*IEnumerable*) корневых объектов.

Получение единичного объекта из UoW конейнера производится с помощью метода [*Get*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/354.html) в котором можно указать лямбда предиткат для выборки из коллекции корневых объектов, после получения объекта его можно будет редактировать (или удалять как в примере ниже):
```csharp
_patient = work.Get();
_patient.SetDeleted(True);
```

Получение набора объектов из UoW конейнера производится с помощью метода [*Select*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/358.html):
```csharp
var newOperations = work.Find(x => x.PatientOperList).Select(x => x.Date > startDate);
```

Получение единичного подчиненного UoW конейнера производится с помощью метода [*Find*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/352.html) в котором можно указать лямбда предиткат для выборки из коллекции корневых объектов, а Добавление нового объекта в UoW конейнер производится с помощью метода [*Add*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/348.html):
```csharp
work.Find(x => x.PatientOperList).Add(() => n.Data);
```

Замену объекта в UoW конейнере производится с помощью метода [*Assign*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/350.html) (при этом все все подчиненные UoW контейнеры будут сброшены и все изменения в них будут потеряны):
```csharp
work.Assign(x => true, repository.New());
```

>Не используйте метод Assign для редактирования свойств объекта! Редактирование объекта в UoW контейнере производится путем получения экземпляра объекта с помощью метода *Get* и редактирования его свойств. Метод [*Assign*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/350.html) служит именно для замены объекта целиком вместе с подчиненными ему UoW контейнерами!

<!-- -->
>**ВАЖНОЕ ЗАМЕЧАНИЕ:** Понимая, что несмотря на несомненную полезность данного функционала, он не является постоянно необходимым программисту, возможность использования совместной обработки взаимосвязанных объектов вынесена в отдельный модуль [*XDataObject.WorkSet*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/3.html) и при необходимости может быть подключена программистом отдельно.
