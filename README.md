
### **1. Расширенный `GeneralSettings` с логированием**  
```csharp
using System;
using System.IO;

public static class GeneralSettings
{
    // --- Настройки файлов ---
    public static string XmlFilePath { get; set; } = @"C:\Data\input.xml";
    public static string BinFilePath { get; set; } = @"C:\Data\output.bin";

    // --- Настройки таймеров ---
    public static TimeSpan FileCheckInterval { get; set; } = TimeSpan.FromSeconds(1);
    public static TimeSpan MaxWaitTime { get; set; } = TimeSpan.FromSeconds(30);

    // --- Настройки логирования ---
    public static string LogFilePath { get; set; } = @"C:\Logs\service.log";
    public static LogLevel LogLevel { get; set; } = LogLevel.Info; // Уровень логирования
    public static int MaxLogFileSizeMB { get; set; } = 10; // Макс. размер лог-файла в МБ

    // Проверка и создание директории для логов
    static GeneralSettings()
    {
        EnsureDirectoryExists(LogFilePath);
    }

    private static void EnsureDirectoryExists(string filePath)
    {
        var dir = Path.GetDirectoryName(filePath);
        if (!Directory.Exists(dir))
        {
            Directory.CreateDirectory(dir);
        }
    }
}

// Уровни логирования
public enum LogLevel
{
    Debug,
    Info,
    Warning,
    Error
}
```

---

### **2. Класс для логирования (`FileLogger`)**  
Реализуем простой и эффективный логгер с ротацией файлов.  

```csharp
using System;
using System.IO;

public static class FileLogger
{
    public static void Log(string message, LogLevel level = LogLevel.Info)
    {
        // Проверяем уровень логирования
        if (level < GeneralSettings.LogLevel)
            return;

        // Форматируем сообщение
        var logMessage = $"{DateTime.Now:yyyy-MM-dd HH:mm:ss} [{level}] {message}";

        // Ротация логов (если файл слишком большой)
        RotateLogFileIfNeeded();

        // Запись в файл
        try
        {
            File.AppendAllText(GeneralSettings.LogFilePath, logMessage + Environment.NewLine);
        }
        catch (Exception ex)
        {
            // Фолбэк на консоль, если не удалось записать в файл
            Console.WriteLine($"Ошибка записи в лог: {ex.Message}");
        }
    }

    private static void RotateLogFileIfNeeded()
    {
        try
        {
            if (!File.Exists(GeneralSettings.LogFilePath))
                return;

            var fileInfo = new FileInfo(GeneralSettings.LogFilePath);
            if (fileInfo.Length > GeneralSettings.MaxLogFileSizeMB * 1024 * 1024)
            {
                var backupPath = Path.Combine(
                    Path.GetDirectoryName(GeneralSettings.LogFilePath),
                    $"{Path.GetFileNameWithoutExtension(GeneralSettings.LogFilePath)}_{DateTime.Now:yyyyMMddHHmmss}.log"
                );
                File.Move(GeneralSettings.LogFilePath, backupPath);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Ошибка ротации логов: {ex.Message}");
        }
    }
}
```

---

### **3. Пример использования**  
Добавляем логирование в ключевых местах, например в `WaitForAvailable`:  

```csharp
public bool WaitForAvailable(string filePath)
{
    FileLogger.Log($"Начало ожидания файла: {filePath}", LogLevel.Debug);
    var timeout = DateTime.Now + GeneralSettings.MaxWaitTime;

    try
    {
        while (DateTime.Now < timeout)
        {
            if (File.Exists(filePath))
            {
                FileLogger.Log($"Файл {filePath} найден!", LogLevel.Info);
                return true;
            }
            Thread.Sleep(GeneralSettings.FileCheckInterval);
        }

        FileLogger.Log($"Таймаут ожидания файла {filePath}", LogLevel.Warning);
        return false;
    }
    catch (Exception ex)
    {
        FileLogger.Log($"Ошибка при ожидании файла: {ex.Message}", LogLevel.Error);
        throw;
    }
}
```

---

### **4. Дополнительные улучшения**  
#### **Форматирование логов**  
Можно добавить шаблоны для логов (например, JSON):  
```csharp
public static string LogFormat { get; set; } = "{time} [{level}] {message}";
```

#### **Асинхронная запись**  
Чтобы не блокировать основной поток:  
```csharp
public static async Task LogAsync(string message, LogLevel level)
{
    await File.AppendAllTextAsync(GeneralSettings.LogFilePath, $"{DateTime.Now} [{level}] {message}\n");
}
```

#### **Фильтрация по уровню**  
Уже встроено в метод `Log`:  
```csharp
if (level < GeneralSettings.LogLevel) return; // Пропускаем сообщения ниже заданного уровня
```

---

### **Итоговый пример**  
```csharp
// Настройка путей и логирования
GeneralSettings.XmlFilePath = @"D:\Data\input.xml";
GeneralSettings.LogFilePath = @"D:\Logs\service.log";
GeneralSettings.LogLevel = LogLevel.Debug;

// Запуск сервиса с логированием
FileLogger.Log("Сервис запущен", LogLevel.Info);
if (WaitForAvailable(GeneralSettings.XmlFilePath))
{
    FileLogger.Log("Обработка файла...", LogLevel.Info);
}
```

---
