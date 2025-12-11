# Задача 1. паттерн IDisposable с финализатором

Реализовать класс DisposableClass, который:
- принимает строку name в конструкторе;
- реализует полный паттерн Dispose с:
  - флагом isDisposed,
  - финализатором ~DisposableClass,
  - методом Dispose(),
  - защищённым методом Dispose(bool fromDisposeMethod);
- при освобождении:
  - если вызван Dispose() вручную (fromDisposeMethod == true) — пишет:
    - Очистка управляемых ресурсов в {name}
    - Очистка неуправляемых ресурсов в {name}
  - если объект собирает GC (финализатор, fromDisposeMethod == false) — пишет:
    - только Очистка неуправляемых ресурсов в {name}

## Шаблон
```
using System;

class DisposableClass : IDisposable
{
    string name;

    public DisposableClass(string name)
    {
        // TODO: сохранить имя
    }

    private bool isDisposed = false;

    ~DisposableClass()
    {
        // TODO: вызвать Dispose(false)
    }

    public void Dispose()
    {
        // TODO: вызвать Dispose(true) и GC.SuppressFinalize(this)
    }

    protected virtual void Dispose(bool fromDisposeMethod)
    {
        // TODO: реализовать логику очистки с проверкой isDisposed
        // if (!isDisposed)
        // {
        //     if (fromDisposeMethod)
        //         Console.WriteLine("Очистка управляемых ресурсов в {0}", name);
        //     Console.WriteLine("Очистка неуправляемых ресурсов в {0}", name);
        //     isDisposed = true;
        // }
    }
}

public class Program
{
    public static void Main()
    {
        var disposable = new DisposableClass("DisposedManually");
        disposable.Dispose();

        using (disposable = new DisposableClass("DisposedWithUsing"))
        {
            Console.WriteLine("Внутри using (без исключения)");
        }

        try
        {
            using (disposable = new DisposableClass("DisposedWithException"))
            {
                Console.WriteLine("Внутри using (с исключением)");
                throw new Exception("Тестовое исключение");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Исключение перехвачено: " + ex.Message);
        }

        Console.WriteLine("Конец Main");
        Console.ReadLine();
    }
}
```

# Задача 2. Файловая сессия

Реализовать класс FileSession, который:
- в конструкторе принимает путь к файлу;
открывает StreamWriter;
- в методе WriteLine(string s) пишет строку в файл;
- в Dispose() корректно закрывает поток.

## Использование
```
using (var session = new FileSession("log.txt"))
{
    session.WriteLine("hello");
}
```
## Шаблон
```
using System;
using System.IO;

class FileSession : IDisposable
{
    private StreamWriter writer;

    public FileSession(string path)
    {
        // TODO: открыть writer
    }

    public void WriteLine(string s)
    {
        // TODO: запись строки
    }

    public void Dispose()
    {
        // TODO: закрыть поток
    }
}

class Program
{
    static void Main()
    {
        using (var s = new FileSession("log.txt"))
        {
            s.WriteLine("test 1");
            s.WriteLine("test 2");
        }

        Console.WriteLine("Готово. Проверьте файл log.txt");
        Console.ReadLine();
    }
}
```

# Задача 3. Вложенные блоки с отступами

Сделать класс IndentedBlock, который:
- при создании печатает { с текущим отступом (4 пробела на уровень);
- при Dispose() печатает } с тем же отступом;
- глобально поддерживается текущий уровень отступа в статическом классе IndentState.
## Пример:
```
using (new IndentedBlock())
{
    using (new IndentedBlock())
    {
    }
    using (new IndentedBlock())
    {
        using (new IndentedBlock())
        {
        }
    }
}
```
## Вывод:
```
{
    {
    }
    {
        {
        }
    }
}
```
## Шаблон
```
using System;

static class IndentState
{
    public static int Level;
}

class IndentedBlock : IDisposable
{
    public IndentedBlock()
    {
        // TODO: вывести "{", увеличить Level
    }

    public void Dispose()
    {
        // TODO: уменьшить Level, вывести "}"
    }
}

class Program
{
    static void Main()
    {
        using (new IndentedBlock())
        {
            using (new IndentedBlock())
            {
            }

            using (new IndentedBlock())
            {
                using (new IndentedBlock())
                {
                }
            }
        }

        Console.ReadLine();
    }
}
```
