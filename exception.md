# Задача 1

Напишите консольный калькулятор без использования локальных try/catch нет.
Все ошибки обрабатываются централизованно через
AppDomain.CurrentDomain.UnhandledException.

```
using System;

class Program
{
    static int Main()
    {
        // TODO: подключить глобальный обработчик необработанных исключений

        Console.WriteLine("Enter: <a> <op> <b>");
        var input = Console.ReadLine();

        var (a, op, b) = Parse(input);
        var result = Calculate(a, op, b);

        Console.WriteLine(result);

        return 0;
    }

    // TODO
    static ... Parse(string? input)
    {
       return;
    }

    // TODO
    static double Calculate(...) 
    {
        return 0.0;
    };
}
```

# Задача 2

Дан массив строк. Нужно просуммировать только корректные целые числа. Ошибки обрабатываются минимальным try/catch.

```
using System;

class Program
{
    static void Main()
    {
        string[] data = { "10", "7", "abc", "42" };

        // TODO: пройти по массиву, преобразовать строки в int и сложить корректные значения
    }
}
```

# Задача 3
Нужно обработать команду вида ACTION:VALUE. Все ошибки должны быть обёрнуты в исключение CommandProcessingException.

Виды команд:
- INC - увеличить число на 1
- DEC - уменьшить число на 1
```
using System;

class Program
{
    static void Main()
    {
        try
        {
            
            Console.WriteLine("Enter command:");
            var input = Console.ReadLine();
            Console.WriteLine(ProcessCommand(input!));
        }
        catch (CommandProcessingException ex)
        {
            Console.Error.WriteLine(...);
        }
    }

    static int ProcessCommand(string command)
    {
        // TODO: разобрать команду, выполнить действие и обернуть любые ошибки
        return 0;
    }
}

class CommandProcessingException : Exception
{
    // TODO: Реализовать конструктор
}
```
