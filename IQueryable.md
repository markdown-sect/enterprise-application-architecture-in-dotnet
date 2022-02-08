# Немного об вездесущем IQueryable

`IQueryable` - это базовый интерфейс LINQ для анализа запросов к источникам данных. С помощью данного итерфейса требуемый запрос проходит через ряд фильтров и фактическая проекция данных определяется в последний момент _(например, на прикладном уровне)_.

## Примеры

1. Не заканчивается вызовом `.ToList()`, `.First()` и т.п.

```csharp
var queryProducts = (from p in CatalogServices.GetProductAsAvailableForSale())
                        orderby p.UnitsInStock descending
                        select new ProductDescriptor
                        {
                            Id = p.Id,
                            Name = p.Name,
                            UnitPrice = p.UnitPrice,
                            UnitsInStock = p.UnitsInStock,
                        }).Take(3);
```

2. Заканчивается вызовом `.SingleOrDefaultAsync()`

```csharp
var userName = _securityService.GetUserName();
var currentEmployee = await _database
            .Employees
            .AsNoTracking()
            .WhereEmployeeIsCUrrentUser(userName)
            .Select(employee =>
                new CurrentEmployeeDTO
                {
                    EmployeeId = employee.PersonalInformation.Id,
                    FirstName = employee.PersonalInformation.FirstName,
                    LastName = employee.PersonalInformation.LastName,
                    Email = employee.PersonalInformation.Email,
                    Identifier = employee.PersonalInformation.Identifier,
                    JobTitle = employee.JobTitle,
                    IsManager = employee.IsTeamManager,
                    TeamId = employee.TeamId,
                }).SingleOrDefaultAsync();

currentEmployee.PictureUrl = Url.Link("EmployeePicture", new { employeeId = currentEmployee.EmployeeId });
```

Интерфейс IQueryable позволяет определять запрос к провайдеру LINQ, такому как безе данных.

Однако выполнение запроса откладывается, поэтому его можно создавать в несколько этапов.

Доступ к базе данных не выполняется, пока вы не вызовете метод выполнения _(например, `.ToList()`)_.

Например, когда вы запрашиваете все продукты, которые находятся в продаже, вы не получаете все 200000 записей, которые соответствуют данным критериям. Добавляя команду `.Take(3)`, вы просто уточняете запрос. Сам запрос выполняется лишь тогда, когда вызывается следующий код:

```csharp
var featuredProducts = query.Products.ToList();
```

Код SQL тогда имеет следующий вид:

```sql
SELECT TOP 3 ... WHERE ...
```

В итоге, вы передаёте интерфейс IQueryalbe через уровни, и каждый уровень по сути может добавлять фильтры, уточняя запрос. В итоге вы получаете просто подмножество данных, в которых нуждаетесь в определённом сценарии использования.
