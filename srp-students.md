# Задача 1. Отчёт по заказу

## Условие

Есть класс OrderReporter, который:
- сам хранит список заказов,
- сам ищет заказ по orderId,
- сам формирует текст отчёта,
- сам «отправляет» отчёт по email (через Console.WriteLine),
- сам пишет лог (тоже через Console.WriteLine).

## Нужно:
- Найти нарушения SRP (слишком много причин для изменения).
- Разбить код на несколько классов/интерфейсов:
  - хранилище заказов (поиск по Id),
  - форматирование отчёта,
  - отправка отчёта пользователю,
  - логирование.
- Оставить OrderReporter только «координатором», который связывает эти компоненты.

## Шаблон
```
using System;
using System.Collections.Generic;

public class Order
{
    public int Id { get; set; }
    public decimal Total { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class OrderReporter
{
    private readonly List<Order> _orders = new()
    {
        new Order { Id = 1, Total = 100m, CreatedAt = new DateTime(2024, 01, 10, 10, 30, 0) },
        new Order { Id = 2, Total = 250.5m, CreatedAt = new DateTime(2024, 02, 05, 14, 00, 0) },
    };

    public void SendReport(int orderId, string email)
    {
        // 1. Ищем заказ
        Order? order = null;
        foreach (var o in _orders)
        {
            if (o.Id == orderId)
            {
                order = o;
                break;
            }
        }

        if (order == null)
        {
            Console.WriteLine("Заказ не найден");
            return;
        }

        // 2. Формируем текст отчёта
        var body =
            $"Заказ #{order.Id}\n" +
            $"Сумма: {order.Total}\n" +
            $"Создан: {order.CreatedAt:dd.MM.yyyy HH:mm}";

        // 3. "Отправляем" email
        Console.WriteLine($"Отправка письма на {email}");
        Console.WriteLine("Тема: Отчёт по заказу");
        Console.WriteLine("Текст:");
        Console.WriteLine(body);

        // 4. Пишем лог
        Console.WriteLine($"[LOG] {DateTime.Now}: report for {order.Id} sent to {email}");
    }
}
```

# Задача 2. Консольный конвертер валют
## Условие
Есть консольное приложение, которое:
- спрашивает сумму в евро,
- само решает, какой курс использовать,
- само считает перевод в рубли,
- само выводит всё в консоль.

Всё это в одном классе → нарушение SRP.

## Нужно:
- Выделить:
  - ввод/вывод (UI),
  - источник курса,
  - логику конвертации.
- Сделать так, чтобы конвертацию можно было протестировать без консоли.

## Шаблон
```
using System;

public class CurrencyConverterApp
{
    public void Run()
    {
        Console.Write("Введите сумму в евро: ");
        var input = Console.ReadLine();

        if (!decimal.TryParse(input, out var eur))
        {
            Console.WriteLine("Некорректное число");
            return;
        }

        // "Получаем" курс (здесь просто фиксированный)
        var rate = GetRate();
        var result = eur * rate;

        Console.WriteLine($"Текущий курс: {rate}");
        Console.WriteLine($"В рублях: {result}");
        Console.WriteLine("Нажмите любую клавишу...");
        Console.ReadKey();
    }

    private decimal GetRate()
    {
        // Представим, что это внешнее API / БД и т.п.
        return 100.50m;
    }
}
```

# Задача 3. TODO-список и сохранение
## Условие
Класс TodoManager:
- валидирует текст задачи,
- хранит список задач,
- пишет сообщения в консоль,
- форматирует текст для файла,
- сохраняет файл.

Нарушение SRP: слишком много причин изменения.

## Нужно:
- Разделить:
  - управление задачами,
  - форматирование списка,
  - сохранение в файл,
  - уведомления пользователя.
- Оставить маленькие, чёткие классы.

## Шаблон
```
using System;
using System.Collections.Generic;
using System.IO;
using System.Text;

public class TodoManager
{
    private readonly List<string> _items = new();

    public void Add(string text)
    {
        if (string.IsNullOrWhiteSpace(text))
        {
            Console.WriteLine("Пустая задача");
            return;
        }

        _items.Add(text.Trim());
        Console.WriteLine("Задача добавлена");
    }

    public void SaveToFile()
    {
        var sb = new StringBuilder();
        sb.AppendLine("TODO list:");
        for (int i = 0; i < _items.Count; i++)
        {
            sb.AppendLine($"{i + 1}. {_items[i]}");
        }

        File.WriteAllText("todo.txt", sb.ToString());
        Console.WriteLine("Сохранено в todo.txt");
    }
}
```
