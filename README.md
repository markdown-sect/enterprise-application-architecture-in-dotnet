# enterprise-application-architecture-in-dotnet
 
> ### Принципы разработки ПО
## `TDD` (Test-Driven Development)
  - Это процесс, в котором разработчики начинают писать программы с тестов.
  - Разработки начинают с **провального** теста, который описывает ожидаемое поведение.
  - Затем добавляется код, безнес-логика, и расширяются функциональные возможности программы.
### Основные факты, касающиеся подхода `TDD`
- Тесты не являются основной целью подхода `TDD`. Они нацелены на появление идеального проекта в результате непрерывного рефакторинга.
- Конечная цель тестов `TDD` - не широкое покрытие тестов, а проект более высокого качества.
- Непрерывный рефакторинг - это не возможность, а необходимость. Он является наиболее важной частью подхода `TDD`, чем тесты.
 
## `BDD` (Behavior-Driven Design)
  - Цель данного подхода заключается в том, чтобы определить циклические взаимодействия со всеми заказчиками, приводящие к доставке конечного продукта.
  - Одна из основ `BDD` - формулировка чётко определённых выводов в результате каждого взаимодействия.
  - Подход `BDD` предполагает, что мы сначала создали набор операторов, затем эти операторы уточняются разработчиками и формируют конкретную реализацию системы.

### В основе `BDD` лежат следующие аспекты
  - Предварительные условия
  - Действие
  - Вывод
 
Типичный оператор `BDD` выглядит следующим образом:
```
Scenario: User is tracking the score of a water polo match
Given: The user on page /match
And The match is fully configured
And The match is started
When The user clicks on Goal button on the left
Then The system updates the match scoring one goal for Home
```
 
> ### Архитектуры ПО
## `DDD` (Domain-Driven Design)
  - DDD (предметно-ориентированное проектирование) - это подход к проектированию и разработке ПО, сутью которого является _структуризация сложных программных систем_.
  - Данный подход предполагает владение знаниями о конкретной предметной области бизнеса и подразумевает создание модели ПО, которая её точно отражает.
    - Предметная область - это то, как компания ведёт своё собственное дело: её организация, процессы, методы люди и даже язык.
  - Подход DDD становится неудобным и слабым, если у вас недостаточно знаний для того, чтобы превратить его в соответствующую модель предметной области.

Из чего состоит данный подход
  - Аналитическая часть
     - Единый язык - это словарь, которого последовательно придерживаются все участники проекта. Это также _шаблон_, влияющий на выбор имён и структуры классов. Он помогает лучше и быстрее понять требования и упростить связь между сторонами во избежание недопониманий.
     - Ограниченные контексты - фрагмент предметной области, который рассматривается независимо от других, поскольку они используют единый _язык_
     - Создание карты контекстов - создаёт высокоуровневое представление области с точки зрения архитектора ПО, выявляет подобласти и отношения между ними.
  - Стратегическая часть

Данный подход предполагает **_многоуровневую архитектуру_**, разработанную вокруг **_модели предметной области_**
 
## Архитектуры приложений:
   - **Многоуровневая архитектура (Multilayer Architecture)**
     - Разделение приложения на уровень представления данных, бизнес-логики и самих данных.
   - **Многоярусная архитектура (Multitier Architecture)**
     - Архитектура, похожая на многоуровневую за исключением того, что вместо уровней в ней используются ярусы.
   - **Клиент-серверная архитектура (Client/Server Architecture)**
     - Двухуровневая архитектура, состоящая из единственного представления данных и механизма доступа к ним.
   - **Модель предметной области (Domain Model)**
     - Многослойная архитектура, основанная на уровне представления данных, прикладном уровне, уровне предметной области и инфраструктурном уровне, разработанных в соответствии с принципами DDD.
   - **Разделение ответственности команд и запросов (Command-Query Responsibility Segregation - CQRS)**
     - Двухсторонняя многослойная архитектура с параллельными разделами для выполнения команд и запросов. Каждый из разделов может иметь свою предметно-ориентированную или клиент-серверную архитектуру (и другие).
   - **Источник событий (Event Sourcing)**
     - Многослойная архитектура, которая практически всегда основывается на принципе CQRS, а не на данных. События интерпретируются как данные первого класса, а вся запрашиваемая информация выводится из хранимых событий.
   - **Монолитная архитектура (Monolithic Architecture)**
     - Контекст представляет собой отдельное приложение или службу, предоставляющую интерфейс прикладного программирования внешнему миру.
   - **Микрослужбы (Micros-Services)**

Различие между уровнями (layers) и ярусами (tier):
  - _Уровень_ - это логический контейнер для различных частей кода
  - _Ярус_ - это физический контейнер для кода, и относится к своему собственному пространству процесса или машины.
  - Все уровни фактически разворачиваются на физическом уровне, но разные уровни могут состоять из разных ярусов.
 
### Многоуровневая и многоярусная архитектура
Многоуровневая и Многоярусная архитектуры отличаются друг от друга тем, что в первой блоки отделяются друг от друга логически, а во второй - физически.

Плюсы и минусы разделения на ярусы (физического разделения)
  - При таком подходе мы теряем общую производительность системы засчёт задержки между ярусами.
  - Обслуживание многоярусной системы более сложное и дорогостоящее.
  - Засчёт ярусов можно легко масштабировать приложение.

Зачастую при написании многоуровневого приложения используется 2 яруса: Представление данных && Бизнес логика и отдельно Данные.

### Улучшенная (современная) многоуровневая архитектура
После появления Модели Предметной области и DDD, классическая многоуровневая архитектура претерпела некоторые изменения:
  - Уровень представления
  - Прикладной уровень
  - Смысловой уровень
    - Модели
    - Службы
  - Инфраструктурный уровень
    - IoC (Inversion of Control)
    - Кеш
    - Хранилища

Инфраструктурный уровень стал включать в себя уровень данных, а также ORM, IoC (inversion of control), механизмы обеспечения безопасности, кеширования и другие.
Уровень бизнес-логики разделился на "прикладной уровень" и "уровень предметной области".
Также бизнес-логика вышла за пределы своего слоя и стала реализовываться как на уровне представления, так и на уровне данных.
Вышеупомянутые изменения говорят о том, что для классической многоуровневой архитектуры был применён принцип разделения функций (SoC - Separation of Concerns).

Немного о самих уровнях и об их обязанностях:
  - Уровень представления
    - Отвечает за обеспечение пользовательского интерфейса (UI), необходимого для решения любых задач.
    - Уровень представления апеллирует моделями представления, и зачастую эти модели совпадают с моделями входных данных, с которыми уже работает Прикладной уровень.
  - Прикладной уровень 
    - Дополнительный слой, который передаёт информацию представлению и синхронизирует выполнение всех других бизнес-операций.
    - Это место, где организовывается реализация сценариев использования.
    - Он является "точкой входа в прикладную часть системы" и контакта между представлением и прикладной частью.
    - Этот уровень состоит из методов, почти однозначно связанных со сценариями использования на уровне представления.
    - Этот уровень отвечает за реализацию сценариев использования приложений, организовывает выполнение задач и делегирует работу другим нижележащим уровням.
    - _Судя по всему, это тот уровень, где расположены контроллеры нашего API_
    - Этот уровень содержит ссылки на уровень предметной области и инфраструктурный уровень. Кроме того, этот уровень ничего не знает о бизнес-правилах и не хранит никакой бизнес-информации.
  - Уровень предметной области
    - Содержит всё бизнес-логику, которая связана с одним или несколькими сценариями использования.
    - Состоит из модели (модели предметной области) и, возможно, семейства служб.
    - Сущности, хранящиеся на этом уровне, должны отражать данные и поведение (Rich Domain Model). Однако может использоваться противоположный подход, называемый Anemic Domain Model.
    - Содержит службы предметной области - части логики предметной области, которые по некоторым причинам не совпадают ни с одной из сущностей. Служба предметной области - это класс, группирующий логически связанные функции, которые обычно воздействуют на сущности предметной области.
    - Службы предметной области требуют доступа к инфраструктурному уровню для выполнения операций чтения-записи.
  - Инфраструктурный уровень
    - Связан с использованием конкретных технологий хранения данных (ORM), механизмов обеспечения безопасности, регистрации, трассировки, кеширования, контейнеров IoC и т.д.
    - Самый важный компонент этого уровня - уровень постоянного хранения данных (уровень доступа к данным), который может быть расширен для того, чтобы охватить несколько источников данных.
