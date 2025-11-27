# Задача 1 (OCP). Форматирование отчётов

## Условие

Есть класс Report, описывающий отчёт, и класс ReportPrinter, который печатает отчёт в одном формате (plain text).

Нужно сделать так, чтобы система поддерживала разные форматы вывода (Text, Html, Json и т.п.), при этом не приходилось менять код ReportPrinter при добавлении нового формата.

Подробнее:
- Ввести интерфейс форматтера отчёта, который умеет:
  - принимать Report,
  - возвращать строку,
  - сообщать своё имя формата ("text", "html" и т.п.).
- Реализовать минимум два форматтера, например:
  - TextReportFormatter
  - HtmlReportFormatter
- Переделать ReportPrinter, чтобы:
  - он принимал коллекцию форматтеров в конструкторе,
  -  по строке format выбирал нужный форматтер,
  - не требовалось менять код ReportPrinter при добавлении нового форматтера.


Шаблон:
```
using System;

namespace OcpPractice
{
    public class Report
    {
        public string Title { get; }
        public string Body { get; }

        public Report(string title, string body)
        {
            Title = title;
            Body = body;
        }
    }

    // Сейчас поддерживается только текстовый вывод
    public class ReportPrinter
    {
        public void PrintAsText(Report report)
        {
            Console.WriteLine("=== " + report.Title + " ===");
            Console.WriteLine(report.Body);
        }

        // TODO:
        // Заказчик попросил HTML и JSON-форматы.
        // НЕЛЬЗЯ каждый раз дописывать сюда новые методы/if-ы.
    }

    class Program
    {
        static void Main()
        {
            var report = new Report("Отчёт за день", "Продажи выросли на 15%.");

            var printer = new ReportPrinter();

            // Сейчас поддерживается только текст
            printer.PrintAsText(report);

            // TODO (после рефакторинга):
            // Хотим иметь что-то вроде:
            // printer.Print(report, "text");
            // printer.Print(report, "html");
            // printer.Print(report, "json");
        }
    }
}
```


# Задача 2 (DIP).  Платёжный сервис

Цель: Потренировать принцип Dependency Inversion — высокоуровневый код зависит от абстракций, а не от конкретных классов.

## Условие
Есть платёжный сервис PaymentService, который сам создаёт StripeClient и вызывает его метод Charge.

Такой код тяжело тестировать и трудно менять платёжного провайдера (например, на PayPal).

Нужно:
- ввести интерфейс платёжного провайдера,
- сделать так, чтобы PaymentService зависел от этого интерфейса,
показать, как подменить реализацию (Stripe / Mock).

Подробнее: 
- Создать интерфейс, например IPaymentProvider с методом void Pay(decimal amount).
- Создать адаптер StripePaymentProvider, который:
  - внутри использует StripeClient,
  - реализует IPaymentProvider.
- Переделать PaymentService, чтобы:
  - получать IPaymentProvider через конструктор (DI),
  - вызывать _provider.Pay(amount) вместо прямого StripeClient.
- (Опционально) Добавить MockPaymentProvider и в Main показать пример вызова через мок.


## Шаблон
```
using System;

namespace DipPractice
{
    // Представим, что это SDK Stripe
    public class StripeClient
    {
        private readonly string _apiKey;

        public StripeClient(string apiKey)
        {
            _apiKey = apiKey;
        }

        public void Charge(decimal amount)
        {
            Console.WriteLine($"[Stripe] Charged {amount}.");
        }
    }

    // ПЛОХО: жёстко зависит от StripeClient
    public class PaymentService
    {
        public void Pay(decimal amount)
        {
            var client = new StripeClient("SECRET_KEY"); // жёсткая привязка
            client.Charge(amount);
        }
    }

    class Program
    {
        static void Main()
        {
            var service = new PaymentService();
            service.Pay(100m);

            // TODO:
            // - Ввести интерфейс платёжного провайдера
            // - Заставить PaymentService использовать этот интерфейс
            // - Показать, как можно подставить другую реализацию в Main
        }
    }
}
```

# Задача 3. DI-контейнеры: Microsoft DI и Ninject

Условие

Есть простое консольное приложение для отправки уведомлений пользователю.

Сейчас зависимости создаются вручную через new:
- INotificationSender — интерфейс отправки уведомлений;
- EmailNotificationSender — отправка на e-mail;
- NotificationService — высокоуровневый сервис, который использует INotificationSender.

## Нужно:
- Оставить/дописать интерфейсы и реализации.
- Убрать ручное создание зависимостей из Main.
- Сделать два варианта точки входа:
  - MainWithMicrosoftDi() — использование ServiceCollection и BuildServiceProvider();
  - MainWithNinject() — использование StandardKernel и kernel.Get<NotificationService>().

Для компиляции:
- Вариант a): добавить пакет Microsoft.Extensions.DependencyInjection
- Вариант b): добавить пакет Ninject

## Шаблон:
```
using System;
// TODO: для варианта a) понадобится using Microsoft.Extensions.DependencyInjection;
// TODO: для варианта b) понадобится using Ninject;

namespace DiPractice
{
    public interface INotificationSender
    {
        void Send(string to, string message);
    }

    // Реализация через Email
    public class EmailNotificationSender : INotificationSender
    {
        public void Send(string to, string message)
        {
            Console.WriteLine($"[Email] To: {to}. Message: {message}");
        }
    }

    // Высокоуровневый сервис
    public class NotificationService
    {
        private readonly INotificationSender _sender;

        public NotificationService(INotificationSender sender)
        {
            _sender = sender;
        }

        public void NotifyUser(string userEmail, string text)
        {
            _sender.Send(userEmail, text);
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // ПЛОХО: ручная "сборка графа объектов"
            var sender = new EmailNotificationSender();
            var service = new NotificationService(sender);

            service.NotifyUser("user@example.com", "Welcome!");

            // TODO:
            // 1) Удалить ручное new из Main
            // 2) Реализовать MainWithMicrosoftDi()
            // 3) Реализовать MainWithNinject()
        }

        // TODO: Вариант a) - точка входа с Microsoft.Extensions.DependencyInjection
        static void MainWithMicrosoftDi()
        {
            // здесь должна быть регистрация зависимостей и получение NotificationService
        }

        // TODO: Вариант b) - точка входа с Ninject
        static void MainWithNinject()
        {
            // здесь должна быть регистрация зависимостей и получение NotificationService
        }
    }
}
```
