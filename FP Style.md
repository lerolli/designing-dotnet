FP Style
---

1. Value-объекты (предпочтительно immutable).
2. Чистые функции, принимающие и возвращающие Value-объекты.
3. Dependency Injection via ФВП (вручную).
3. Побочные эффекты примешиваются к чистым ФВП.

*ФВП - функции высшего порядка.

# Задание 1

Написать чистую функцию обработки списка целых чисел:
- принимает IReadOnlyList<int> numbers
- удаляет все отрицательные числа
- оставшиеся умножает на 2
- сортирует по убыванию
- возвращает новый список, исходный не изменяется

**Использовать только LINQ (без for / foreach).**

```
using System;
using System.Collections.Generic;
using System.Linq;

namespace FPPractice
{
    public static class Task1
    {
        /// <summary>
        /// Принимает список чисел, удаляет отрицательные, умножает оставшиеся на 2
        /// и сортирует по убыванию. Исходный список не изменяется.
        /// </summary>
        public static IReadOnlyList<int> ProcessNumbers(IReadOnlyList<int> numbers)
        {
            // TODO: реализовать с помощью LINQ
            throw new NotImplementedException();
        }
    }
}
```

# Задание 2

Реализовать функции высшего порядка Map, Filter, Reduce как методы-расширения для IEnumerable<T> и продемонстрировать их использование:

Реализовать:
```
IEnumerable<TResult> Map<TSource, TResult>(
    this IEnumerable<TSource> source,
    Func<TSource, TResult> selector);

IEnumerable<TSource> Filter<TSource>(
    this IEnumerable<TSource> source,
    Func<TSource, bool> predicate);

TResult Reduce<TSource, TResult>(
    this IEnumerable<TSource> source,
    TResult seed,
    Func<TResult, TSource, TResult> aggregator);
```
Используя эти методы, для списка
`var items = new List<string> { "1", "2", "3", "4", "5" };`
получить сумму квадратов чётных чисел.

```
using System;
using System.Collections.Generic;

namespace FPPractice
{
    public static class FunctionalExtensions
    {
        // Map: проекция элементов
        public static IEnumerable<TResult> Map<TSource, TResult>(
            this IEnumerable<TSource> source,
            Func<TSource, TResult> selector)
        {
            // TODO: реализовать
            throw new NotImplementedException();
        }

        // Filter: фильтрация по предикату
        public static IEnumerable<TSource> Filter<TSource>(
            this IEnumerable<TSource> source,
            Func<TSource, bool> predicate)
        {
            // TODO: реализовать
            throw new NotImplementedException();
        }

        // Reduce: аккумуляция к одному значению
        public static TResult Reduce<TSource, TResult>(
            this IEnumerable<TSource> source,
            TResult seed,
            Func<TResult, TSource, TResult> aggregator)
        {
            // TODO: реализовать
            throw new NotImplementedException();
        }
    }

    public static class Task2
    {
        public static int SumOfEvenSquares(IEnumerable<string> items)
        {
            // TODO: использовать Map / Filter / Reduce
            // Преобразовать строки в числа, оставить чётные,
            // возвести в квадрат, просуммировать.
            throw new NotImplementedException();
        }
    }
}
```

# Задание 3

Есть класс:
- читает строки заказов из источника IDataSource
- парсит строку в Order через IOrderParser
- пропускает заказы со статусом "Cancelled"
- форматирует заказ в строку через IOrderFormatter
- пишет результат в файл
- логирует прогресс через ILogger

Сейчас все зависимости передаются через конструктор, и в методе много побочных эффектов.

**Требуется:**

- Переписать класс так, чтобы конструктор OrderReportGenerator не принимал никаких сервисов.
- Все зависимости должны передаваться в метод Generate.

Отделить максимум логики от побочных эффектов:
- чтение строк из IDataSource
- преобразование строк → Order
- фильтрация по статусу
- форматирование
- запись в файл
- логирование

Ввести чистые методы, которые:
принимают коллекции / данные и возвращают результат
не обращаются к File, Stream, Console, IDataSource и т.п.
Оставить в методе Generate минимальный слой кода с побочными эффектами (открытие ресурсов, запись в файл, логирование).

```
using System;
using System.Collections.Generic;
using System.IO;

namespace FPPractice
{
    public record Order(int Id, decimal Amount, string Status);

    public interface IDataSource : IDisposable
    {
        string? NextLine();
    }

    public interface IOrderParser
    {
        Order Parse(string line);
    }

    public interface IOrderFormatter
    {
        string Format(Order order);
    }

    public interface ILogger
    {
        void Info(string message);
    }

    public class OrderReportGenerator
    {
        /*
            Отрефакторите код.

            1. Уберите зависимости из конструктора.
               Конструктор должен быть без параметров.

            2. Вынесите доменную логику (парсинг, фильтрация, форматирование)
               в чистые методы.

            3. Оставьте в методе Generate только "тонкий" слой побочных эффектов
               (открытие источника, запись файла, логирование).
        */

        // Конструктор без зависимостей
        public OrderReportGenerator()
        {
        }

        // Старая реализация для ориентира (можно удалить после рефакторинга)
        public void GenerateOld(
            Func<IDataSource> openDataSource,
            IOrderParser parser,
            IOrderFormatter formatter,
            ILogger logger,
            string outputFilename)
        {
            using (var data = openDataSource())
            using (var writer = new StreamWriter(outputFilename))
            {
                var processed = 0;

                while (true)
                {
                    var line = data.NextLine();
                    if (line == null) break;

                    processed++;

                    var order = parser.Parse(line);

                    if (order.Status == "Cancelled")
                        continue;

                    var text = formatter.Format(order);
                    writer.WriteLine(text);

                    if (processed % 100 == 0)
                        logger.Info($"processed {processed} orders");
                }
            }
        }

        // TODO: реализовать новую версию Generate в функциональном стиле.
        public void Generate(
            Func<IDataSource> openDataSource,
            IOrderParser parser,
            IOrderFormatter formatter,
            ILogger logger,
            string outputFilename)
        {
            throw new NotImplementedException();
        }
    }

    // (по желанию) можно использовать вспомогательные методы / классы:
    public static class Enumeration
    {
        public static IEnumerable<T> RepeatUntilNull<T>(Func<T?> get)
            where T : class
        {
            // TODO: реализовать при необходимости
            throw new NotImplementedException();
        }

        public static IEnumerable<T> AfterEvery<T>(
            this IEnumerable<T> items,
            int period,
            Action<int> afterNth)
        {
            // TODO: реализовать при необходимости
            throw new NotImplementedException();
        }
    }
}
```
