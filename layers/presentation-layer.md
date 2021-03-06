# Уровень презентации

## `MVC` (Model-View-Controller)

- Основная цель данного шаблона состоит в том, чтобы разделить приложение на 3 логических состовляющих: модель, представление и контроллер.
- `Модель` относится к состоянию приложения, она охватывает функции приложения и уведомляет представление об изменении его состояния.
- `Представление` относится к порождению любых графических элементов, выведенных на экран для пользователя. Оно получает и обрабатывает любые пользовательские запросы/действия.
- `Контроллер` отображает пользовательские запросы/действия в действия над моделью и выбирает следующее представление.

## `MVP` (Model-View-Presenter)

- Это шаблон, появившийся в ходе эволюции `MVC`. Тут элемент _представления_ заменил _контроллер_ и взял на себя больше обязанностей: от подведения итогов диспетчеризации задач до прорисовки представления.
  - В этом шаблоне _представление_ и _модель_ отделены друг от друга, а _предъявитель (presenter)_ выстраивает взаимодействие между ними.

## `MVVC` (Model-View-ViewModel)

- В этом шаблоне есть класс, который включает в себя и командную модель, и модель представления для пользовательского интерфейса.
  - Отдельный класс - объект модели представления - связывает свойства с визуальными компонентами пользовательского интерфейса, а методы - с событиями пользовательского интерфейса.