# Задача 1 — Builder

## Постановка
Реализовать класс UserBuilder, который позволяет настраивать объект User в стиле Fluent API:
```
SetName(string)
SetEmail(string)
Build()
```

Ограничение: имя не длиннее 50 символов.

## Заготовка:

```
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
}

public class UserBuilder
{
    // TODO: приватные поля
    // TODO: методы SetName, SetEmail, Build
}
```
# Задача 2 — Конфигуратор объекта

## Постановка
Создать конфигуратор CarConfigurator с Fluent-методами:
```
SetModel(string)
SetYear(int)
AddOption(string) (можно вызывать несколько раз)
Build() возвращает готовый объект Car.
```
## Заготовка
```
public class Car
{
    public string Model { get; set; }
    public int Year { get; set; }
    public List<string> Options { get; set; }
}

public class CarConfigurator
{
    // TODO: реализовать fluent API
}
```
# Задача 3
## Постановка
Нужно реализовать простой механизм валидации с fluent-цепочкой на основе функций.

```
public class Validator<T>
{
    public Validator<T> AddRule(
        Func<T, bool> predicate,
        string errorMessage
    )
    {
        // TODO
    }

    public bool Validate(T value, Action<string> onError)
    {
        // TODO
    }
}

public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
}

public class Program
{
    public static void Main()
    {
        var stringValidator = new Validator<string>()
            .AddRule(
                s => !string.IsNullOrEmpty(s),
                "Строка не должна быть пустой."
            )
            .AddRule(
                s => s.Length >= 5,
                "Длина строки должна быть не меньше 5 символов."
            );

        var value = "abc";

        bool ok = stringValidator.Validate(value, error => { Console.WriteLine("Ошибка (string): " + error); });

        Console.WriteLine("Результат string-валидации: " + (ok ? "OK" : "FAIL"));
    }
}
```
### Требуется:
Внутри Validator<T> хранить набор правил, где каждое правило описывается:
- функцией Func<T, bool> predicate,
- строкой errorMessage.

Метод AddRule должен:
- добавлять новое правило во внутреннюю коллекцию,
- возвращать this, чтобы можно было выстраивать fluent-цепочку.

Метод Validate должен:
- последовательно применять все правила к значению value,
для каждого правила, которое вернуло false, вызывать onError(errorMessage),
- вернуть true, если все правила прошли, и false, если было хотя бы одно нарушение.
### Дополнительно:
дописать в Main пример валидации объекта User с помощью Validator<User>,

правила для User:
- Name не должен быть пустым,
- Email должен содержать символ '@'.
