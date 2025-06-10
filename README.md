public async Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
{
    await WaitForFileAvailable(xmlFilePath);
    
    // 1. Чтение XML
    var xmlContent = await File.ReadAllTextAsync(xmlFilePath);
    
    // 2. Десериализация XML в объект (если нужно)
    var xmlSerializer = new XmlSerializer(typeof(YourDataClass));
    YourDataClass dataObject;
    using (var reader = new StringReader(xmlContent))
    {
        dataObject = (YourDataClass)xmlSerializer.Deserialize(reader);
    }
    
    // 3. Сериализация в бинарный формат через ваш статический метод
    var outputPath = Path.Combine(outputDirectory, Path.GetFileNameWithoutExtension(xmlFilePath) + ".bin");
    ExternalSerializer.Serialize(outputPath, dataObject); // Ваш метод
    
    _logger.LogInformation($"Converted {xmlFilePath} → {outputPath}");
}
вовововов
вовоовв
Services/IFileWatcherService.cs
csharp

public interface IFileWatcherService
{
    event EventHandler<FileSystemEventArgs> FileCreated;
    void StartWatching(string path, string filter);
    void StopWatching();
}

Services/FileWatcherService.cs
csharp

public class FileWatcherService : IFileWatcherService, IDisposable
{
    private readonly ILogger<FileWatcherService> _logger;
    private FileSystemWatcher _watcher;

    public event EventHandler<FileSystemEventArgs> FileCreated;

    public FileWatcherService(ILogger<FileWatcherService> logger)
    {
        _logger = logger;
    }

    public void StartWatching(string path, string filter)
    {
        _watcher = new FileSystemWatcher
        {
            Path = path,
            Filter = filter,
            NotifyFilter = NotifyFilters.FileName | NotifyFilters.LastWrite
        };

        _watcher.Created += OnFileCreated;
        _watcher.EnableRaisingEvents = true;
        
        _logger.LogInformation($"Started watching folder: {path} for {filter} files");
    }

    private void OnFileCreated(object sender, FileSystemEventArgs e)
    {
        _logger.LogInformation($"New file detected: {e.Name}");
        FileCreated?.Invoke(this, e);
    }

    public void StopWatching()
    {
        if (_watcher != null)
        {
            _watcher.EnableRaisingEvents = false;
            _watcher.Dispose();
        }
    }

    public void Dispose() => StopWatching();
}

Services/IXmlConverterService.cs
csharp

public interface IXmlConverterService
{
    Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory);
}

Services/XmlConverterService.cs
csharp

public class XmlConverterService : IXmlConverterService
{
    private readonly ILogger<XmlConverterService> _logger;

    public XmlConverterService(ILogger<XmlConverterService> logger)
    {
        _logger = logger;
    }

    public async Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
    {
        try
        {
            // 1. Проверяем существование файла
            await WaitForFileAvailable(xmlFilePath);
            
            // 2. Читаем XML
            var xmlContent = await File.ReadAllTextAsync(xmlFilePath);
            
            // 3. Конвертируем в бинарный формат (ваша логика)
            byte[] binaryData = Encoding.UTF8.GetBytes(xmlContent);
            
            // 4. Сохраняем результат
            var outputPath = Path.Combine(outputDirectory, 
                Path.GetFileNameWithoutExtension(xmlFilePath) + ".bin");
            
            await File.WriteAllBytesAsync(outputPath, binaryData);
            
            _logger.LogInformation($"Successfully converted {xmlFilePath} to {outputPath}");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, $"Error converting file {xmlFilePath}");
            throw;
        }
    }

    private async Task WaitForFileAvailable(string filePath, int timeoutMs = 5000)
    {
        var sw = Stopwatch.StartNew();
        while (sw.ElapsedMilliseconds < timeoutMs)
        {
            try
            {
                using (var fs = File.Open(filePath, FileMode.Open, FileAccess.Read, FileShare.None))
                {
                    return; // Файл доступен
                }
            }
            catch (IOException)
            {
                await Task.Delay(100);
            }
        }
        throw new TimeoutException($"File {filePath} is locked after {timeoutMs}ms");
    }
}

3. Реализация Worker
Workers/ConversionWorker.cs
csharp

public class ConversionWorker : BackgroundService
{
    private readonly IFileWatcherService _fileWatcher;
    private readonly IXmlConverterService _converter;
    private readonly IConfiguration _config;
    private readonly ILogger<ConversionWorker> _logger;

    public ConversionWorker(
        IFileWatcherService fileWatcher,
        IXmlConverterService converter,
        IConfiguration config,
        ILogger<ConversionWorker> logger)
    {
        _fileWatcher = fileWatcher;
        _converter = converter;
        _config = config;
        _logger = logger;
    }

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var inputPath = _config["Folders:Input"] ?? "C:/XmlInput";
        var outputPath = _config["Folders:Output"] ?? "C:/BinOutput";
        
        // Создаем выходную директорию, если ее нет
        Directory.CreateDirectory(outputPath);

        _fileWatcher.FileCreated += async (sender, e) => 
        {
            try
            {
                await _converter.ConvertXmlToBinAsync(e.FullPath, outputPath);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, $"Error processing file {e.Name}");
            }
        };

        _fileWatcher.StartWatching(inputPath, "*.xml");
        
        _logger.LogInformation($"Worker started. Watching: {inputPath}");

        return Task.CompletedTask;
    }
}

4. Настройка в Program.cs
csharp

var builder = Host.CreateDefaultBuilder(args);

builder.ConfigureServices((context, services) =>
{
    // Конфигурация
    services.Configure<FolderSettings>(context.Configuration.GetSection("Folders"));
    
    // Сервисы
    services.AddSingleton<IFileWatcherService, FileWatcherService>();
    services.AddSingleton<IXmlConverterService, XmlConverterService>();
    
    // Worker
    services.AddHostedService<ConversionWorker>();
    
    // Логирование
    services.AddLogging(configure => configure.AddConsole());
});

var host = builder.Build();
await host.RunAsync();

5. Конфигурация (appsettings.json)
json

{
  "Folders": {
    "Input": "C:/XmlInput",
    "Output": "C:/BinOutput"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  }
}



ррррр

using System.Xml.Serialization;
using ExternalProject; // Пространство имён, где находится ExternalSerializer

public class XmlConverterService
{
    public void ConvertXmlFileToBinary(string xmlFilePath, string outputBinaryPath)
    {
        // 1. Читаем XML-файл и десериализуем его в объект
        var xmlSerializer = new XmlSerializer(typeof(YourDataClass)); // Замените YourDataClass на ваш класс
        YourDataClass dataObject;

        using (var reader = new StreamReader(xmlFilePath))
        {
            dataObject = (YourDataClass)xmlSerializer.Deserialize(reader);
        }

        // 2. Сериализуем объект в бинарный формат через статический метод
        ExternalSerializer.Serialize(outputBinaryPath, dataObject);
    }
}
