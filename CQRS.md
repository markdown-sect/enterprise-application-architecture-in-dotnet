# Архитектура CQRS (Command Query Responsibility Segregation)

## Основная идея

Основной идеей данной арзитектуры является **разделение команд от запросов**. Запросы и команды _(или операции чтения и записи)_ не имеют ничего общего и должны выполняться независимо друг от друга.

**Запрос** - работа, котоаря никогда не изменяет состояние системы и просто возвращает данные.

**Команда** - работа, которая изменяет состояние системы и обычно не возвращает данные, кроме кода статуса.

В сложных приложениях единственная модель, используемая _запросами_ и _командами_ быстро начинает усложняться и разростаться - именно поэтому было принято решение о разделении этих операций.

- Архитектура CQRS использует два разных уровня предметной области
- Мы группируем все операции запросов на одном уровне, а все операции команд - на другом
- У каждого уровня есть своя собственная архитектура и свой собственный набор служб.

### Плюсы CQRS

1. Упрощение проекта
2. Потенциально более высокая масштабируемость
3. Сложность _(условная)_ CQRS равна N + N при N * N сложности обычной системы без разделения запросов и команд.

## Команды и события

Команды и собятия являются производным от **сообщения**.

**Сообщение** - это объект передачи данных (DTO), содержащий простые данные, предназначенные для дальнейшей обработки _(контейнер данных)_.

```csharp
public class Message
{
    // несколько необязательных свойств
    // класс может быть простым индикатором без свойств
}
```

### Команда

**Команда** - императивное сообщение, которое похоже на явный запрос, направленный в систему для выполнения определённых задач.

- Команда направляется одному обработчику
- Команда может быть отклонена системой
- Выполнение команды может завершиться сбоем
- Результат выполнения команды может варьироваться в зависимости от текущего состояния системы
- Команда обычно не пересекает границы конкретного ограниченного контекста
- Соглашение об именовании команд должно отражать их императивный характер и указывать их предназначение.

```csharp
public class CheckoutCommand : Message
{
    public string CartId { get; private set; }
    public string CustomerId { get; private set; }

    public CheckoutCommand(string cartId, string customerId)
    {
        CartId = cartId;
        CustomerId = customerId;
    }
}
```

### Событие

**Событие** - это сообщение, которое служит уведомлением о чём-то, что уже произошло. Класс события является неизменяемым.

- Событие не может быть отклонено или аннулировано системой
- Событие может иметь сколько угодно обработчиков
- Обработка события может генерировать другие события, напрвляемые другим обработчиком
- Событие может иметь подписчиков, находящихся за пределами ограниченного контекста, в котором оно появилось.

```csharp
public class DomainEvent : Message
{
    // имя или идентификатор пользователя
    // номер версии события

    /* другие необходимые данные */

    public DateTime TimeStamp { get; private set; }

    public DomainEvent()
    {
        TimeStamp = DateTime.Now;
    }
}

public class OrderCreatedEvent : DomainEvent
{
    public string OrderId { get; private set; }
    public string TrackingId { get; private set; }
    public string TransactionId { get; private set; }

    public OrderCreatedEvent(string orderId, string trackingId, string transactionId)
    {
        OrderId = orderId;
        TrackingId = trackingId;
        TransactionId = transactionId;
    }
}
```

## Обработка комманд и событий

Командами управляет процессор, который обычно называется _**шиной команд (command bus)**_.

Событиями управляет _**шина событий (event bus)**_.

### Шина команд

- Шина команд содержит список известных бизнесс-процессов, которые могут быть инициированы командами
- Обработка команды иногда может генерировать событие в предметной области
- Процессы, которые обрабатывают команды и связанные с ними события, называют _сагами (sagas)_.

**Шина команд** - это отдельный класс, который получает сообщения _(запросы на выполнение команд и уведомления о событиях)_ и ищет способ их обработать.

Фактически, шина не выполняет всё работу, а только **выбирает зарегистрированный обработчик**, который может обработать команду или событие.

У шины есть два внутренних словаря - один для отображения стартовых сообщений и типов саги, другой - для отслеживания экземпляров выполняемых саг.

> Обычно шина команд отделяет прикладной уровень от служб предметной области.

```csharp
public interface IHandles
{
    void Handle(T message);
}

public class Bus
{
    private static readonly Dictionary<Type, Type> SagaStarters = new ();
    private static readonly Dictionary<string, object> SagaInstances = new ();

    public static void RegisterSaga<TStartMessage, TSaga>()
    {
        SagaStarters.Add(typeof(TStartMessage), typeof(TSaga));
    }

    public static void Send<T>(T message) 
        where T : Message
    {
        // публикуем событие
        if (message is IDomainEvent)
        {
            // вызываем все зарегистрированные саги
            // и даём каждой обработать событие
            foreach(var saga in SagaInstances)
            {
                var handler = (IHandles<T>)saga;

                if (handler is not null)
                    handler.Handle(message);
            }
        }

        // проверяем, может ли событие запустить зарегистрированную сагу
        if (SagaStarters.ContainsKey(typeof(T)))
        {
            // запускаем сагу, создавая новый экземпляр указанного типа
            var typeOfSaga = SagaStarters(typeof(T));
            var instance = (IHandles<T>)Activator.CreateInstance(typeOfSaga);

            instance.Handle(message);

            // в этой точке сага получает идентификатор
            // сохраняем экземпляр в словаре (расположенном в памяти)
            // для дальнейшего использования
            var saga = (SagaBase)instance;
            SagaInstances.Add(saga.Data.Id, instance);
        }

        // событие не запускает ни одну сагу
        // проверить, можно ли доставить это событие существующей саге
        if (SagaInstances.ContainsKey(message.Id))
        {
            var saga = (IHandles<T>)SagaInstances[message.Id];
            saga.Handle(message);

            // сохраняем сагу в словаре или удаляем после завершения
            if (saga.IsComplete())
                SagaInstances.Remove(message.Id);
            else 
                SagaInstances[message.Id] = saga;
        }
    }
}
```

### Сага

Сага похожа на коллекцию логически связанных методов и обработчиков событий.

Каждая сага - это компонент, который объявляет следующую информацию:

- Команда или событие, запускающее процесс, связанный с сагой
- Команды, которые может обработать сага, и события, которые нужны для неё.

> #### Грубо говоря, сага - это поток операций с начальными и конечными точками или процесс, обрабатывающий команды в контексте потока операций

> #### По сути, сага - это развитие концепции прикладной службы _(сервисов)_

События, сгенерированные сагами, проходят через шину, и шина передаёт их как сообщения всем, кто подписался на это событие.

#### Пример использования шины в контроллере

```csharp
public ActionResult CompleteProcessOrder(string transactionId)
{
    // определяем карточку пользователя по стостоянию сеанса
    var cart = RetrieveCurrentShoppingCart();

    // подготавливаем и ставим в очередь команду ProcessOrder
    var command = new ProcessOrderCommand(transactionId, cart.CartId);
    Bus.Send(command);

    // перенаправляем запрос к HttpGet методу для обновления представления
    return RedirectToAction(nameof(Done));
}
```

### Использование саги

> Сага - это логический компонент, в котором разработчики уточняют части бизне-логики

Сага, совместно с шиной, организует все задачи, которые должны быть выполнены для реализации сценария использования.

### Пример саги

```csharp
public class SagaBase<T>
{
    public string Id { get; init; }
    public T Data { get; set; }
}

public interface ICanHandleMessage<T>
    where T : Message
{
    void Handle(T message);
}

public interface IStartWithMessage<T>
    where T : Message
{
    void Handle(T message);
}

// пример саги
public class CheckoutSaga : SagaBase<CheckoutSagaData>, IStartWithMessage<CheckoutCommand>,
                            ICanHandleMessage<CheckoutCommand>, ICanHandleMessage<CheckGoodsInStockCommand>,
                            ICanHandleMessage<GoodsInStockEvent>, ICanHandleMessage<GoodsNotInStockEvent>,
                            ICanHandleMessage<CheckCustomerPaymentHistoryCommand>, ICanHandleMessage<PaymentHistoryPositiveEvent>, ICanHandleMessage<PaymentHistoryNegativeEvent>
{
    // ...
}

// пример команды для обработки сагой через шину 
public class CheckoutCommand : Message
{
    public string CartId { get; private set; }
    public string CustomerId { get; private set; }

    public CheckoutCommand(string cartId, string ciustomerId)
    {
        CartId = cartId;
        CustomerId = customerId;
    }
}

// использование в контроллере
public ActionResult Checkout(string customerId)
{
    var command = new CheckoutCommand(Session.Session.Id, customerId);
    Bus.Send(command);

    return RedirectToAction("done");
}
```

Различие между `ICanHandleMessage<T>` и `IStartWithMessage<T>` лишь в названиях, по факту - это просто маркеры, с говорящими именами.

#### Пример метода обработки команды

```csharp
public void Handle(CheckoutCommand message)
{
    // устанавливаем идентификатор саги
    Data.Id = message.CartId;
    Data.Cart = message.Cart;
    Data.CustomerId = message.CustomerId;

    // начинаем сагу
    var command = new CheckGoodsInStockCommand(message.Cart);
    command.SagaId = message.Cart.Id;
    Bus.Send(command);
}
```

