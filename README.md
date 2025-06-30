Понял! Если вам нужно **писать логи в файл** в .NET Worker Service, но без использования `Serilog` или `Microsoft.Extensions.Logging.File` (который не входит в стандартную библиотеку), можно реализовать простой **кастомный провайдер логгирования**. Вот как это сделать:

---

## **1. Создаем кастомный `FileLoggerProvider`**
Этот класс будет записывать логи в файл, используя стандартный `ILogger`.

### **FileLoggerProvider.cs**
```csharp
using Microsoft.Extensions.Logging;
using System;
using System.IO;

public class FileLoggerProvider : ILoggerProvider
{
    private readonly string _filePath;

    public FileLoggerProvider(string filePath)
    {
        _filePath = filePath;
        Directory.CreateDirectory(Path.GetDirectoryName(_filePath)); // Создаем папку, если её нет
    }

    public ILogger CreateLogger(string categoryName)
    {
        return new FileLogger(_filePath);
    }

    public void Dispose()
    {
    }
}

public class FileLogger : ILogger
{
    private readonly string _filePath;

    public FileLogger(string filePath)
    {
        _filePath = filePath;
    }

    public IDisposable BeginScope<TState>(TState state) => null;

    public bool IsEnabled(LogLevel logLevel) => true; // Логируем всё

    public void Log<TState>(
        LogLevel logLevel,
        EventId eventId,
        TState state,
        Exception exception,
        Func<TState, Exception, string> formatter)
    {
        if (!IsEnabled(logLevel))
            return;

        var message = formatter(state, exception);
        var logMessage = $"{DateTime.Now:yyyy-MM-dd HH:mm:ss} [{logLevel}] {message}{Environment.NewLine}";

        File.AppendAllText(_filePath, logMessage); // Записываем в файл
    }
}
```

---

## **2. Регистрируем провайдер в `Program.cs`**
Теперь добавляем наш `FileLoggerProvider` в конфигурацию логгирования.

### **Program.cs**
```csharp
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;

var builder = Host.CreateDefaultBuilder(args)
    .ConfigureLogging((hostContext, logging) =>
    {
        logging.ClearProviders(); // Очищаем стандартные провайдеры (опционально)
        
        // Добавляем консольное логгирование (для отладки)
        logging.AddConsole();
        
        // Добавляем файловое логгирование
        logging.AddProvider(new FileLoggerProvider("Logs/service.log"));
    })
    .ConfigureServices(services =>
    {
        services.AddHostedService<Worker>();
        services.AddSingleton<HwServerSettings>(HwServerSettings.Instance);
        services.AddSingleton<Service>();
    });

await builder.Build().RunAsync();
```

---

## **3. Использование в сервисах**
Теперь `ILogger` автоматически пишет логи в файл `Logs/service.log`.

### **Пример в классе `Service`**
```csharp
public class Service
{
    private readonly ILogger<Service> _logger;

    public Service(ILogger<Service> logger)
    {
        _logger = logger;
    }

    public void DoWork()
    {
        _logger.LogInformation("Сервис запущен"); // Запишется в файл и консоль
        try
        {
            // Логируем ошибки
            _logger.LogError("Пример ошибки");
        }
        catch (Exception ex)
        {
            _logger.LogCritical(ex, "Критическая ошибка!");
        }
    }
}
```

--
