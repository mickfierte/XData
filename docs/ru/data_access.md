>Извините. Статья еще находится в стадии разработки, но мы уже работаем над ней...

###Управление доступом к данным
В этой статье рассмотривается как организовать разграничение доступа к данными действиям над ними. В XData есть понятия разграничения доступа к определенным свойствам объектов, к объектам ограниченным определенными условиями, к стандартным (создание, изменение, удаление) и [пользовательским](./tips_and_triks.md#Использование-пользовательской-логики) действиям.

Для управления доступом к данным необходимо реализовать интерфейс [*ISecuritySession*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/373.html) и использовать ссылку на экземпляр его реализации при получении [репозитория](./glossary.md#Репозиторий) с помощью статичного метода [*GetRepository*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/424.html). XData не специфицирует способ авторизации и задания профиля прав пользователя, и ниже например представлена тестовая реализация этого интерфейса:
```csharp
public class MySecuritySession : SecuritySession
    {
        private const string Manager = "Manager";
        private const string Guest = "Guest";
        private const string Chief = "Chief";
        private const string Clerk = "Clerk";

        public static MySecuritySession CreateSecuritySession(string userName, string password)
        {
            if (userName == "Admin" && password == "123") return new MySecuritySession(userName, new string[0], true);
            if (userName == "Manager" && password == "111") return new MySecuritySession(userName, new[] { Manager, Clerk });
            if (userName == "Guest" && password == "222") return new MySecuritySession(userName, new[] { Guest });
            if (userName == "Chief" && password == "333") return new MySecuritySession(userName, new[] { Chief, Manager, Clerk });
            if (userName == "Clerk" && password == "444") return new MySecuritySession(userName, new[] { Clerk });
            RiseUnauthorizedAccessException();
            return null;
        }

        /// <exception cref="XDataSecurityException"></exception>
        [DebuggerStepThrough]
        [DebuggerNonUserCode]
        private static void RiseUnauthorizedAccessException()
        {
            throw new XDataSecurityException("XDataObjectTest system");
        }

        public MySecuritySession(string userName, string[] roles, bool isSupervisor = false) : base(userName, roles, isSupervisor) { }

        protected override void InitializeSession()
        {
            //Hide some columns from Guests
            RegisterRestrictedProperties("", typeof(Invoice), new[] { Guest.SetValue(new[] {
                Property<Invoice>(x => x.DocAmount), 
                Property<Invoice>(x => x.DocLastChange), 
                Property<Invoice>(x => x.Scan), 
                Property<Invoice>(x => x.Source), 
                Property<Invoice>(x => x.CustomerTo), 
                Property<Invoice>(x => x.DeliveryType)
            }) });
            RegisterGrantedActions("", typeof(Invoice), new[] { 
                Chief.SetValue(new [] { Actions.All }), //Grant all actions to Chief
                Manager.SetValue(new [] { Actions.CRUD }), //Grant all CRUD but TestCustomLogic is denied for Manager
                Clerk.SetValue(new [] { Actions.Update }), //Grant update only for Clerk
                Guest.SetValue(new[] { Action(() => Invoice.TestCustomLogic) }) //Grant execute TestCustomLogic only for Guest
            });
            //Hide non ACTIVE invoices from Guests
            RegisterSecurityFilters("", typeof(Invoice), new[] { Guest.SetValue<string, Expression>((Expression<Func<Invoice, bool>>)(x => x.DocStateCode == "ACTIVE")) });
        }
    }
```

>Для облегчения реализации интерфейса [*ISecuritySession*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/4/373.html) предлагается воспользоваться модулем [*XDataObject.Security*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/1/2.html) в котором реализован абстрактный класс [*SecuritySession*](https://htmlpreview.github.io/?https://raw.githubusercontent.com/mickfierte/XData/master/docs/doc/Contents/5/307.html) предоставляющий базовую функциональность пользовательской сессии. В связи с тем, что использование разграничений доступа к данным не является обязательным, данная функциональность вынесена в отдельный модуль.


