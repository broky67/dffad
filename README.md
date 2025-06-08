1. Архитектура решения
text

WorkerService/
├── Services/
│   ├── FileWatcherService.cs    // Мониторит папку
│   ├── XmlToBinaryConverter.cs  // Логика конвертации
└── Worker.cs                   // Основной сервис

2. Ключевые компоненты

FileWatcherService.cs - отслеживает новые файлы:
csharp

public class FileWatcherService : IDisposable
{
    private readonly FileSystemWatcher _watcher;
    private readonly string _inputPath;

    public event Action<string>? NewXmlFileDetected;

    public FileWatcherService(string path)
    {
        _inputPath = path;
        Directory.CreateDirectory(path); // Создаём папку если её нет
        
        _watcher = new FileSystemWatcher(path, "*.xml")
        {
            NotifyFilter = NotifyFilters.FileName | NotifyFilters.LastWrite,
            EnableRaisingEvents = true
        };

        _watcher.Created += OnFileCreated;
    }

    private void OnFileCreated(object sender, FileSystemEventArgs e)
    {
        NewXmlFileDetected?.Invoke(e.FullPath);
    }

    public void Dispose() => _watcher.Dispose();
}

XmlToBinaryConverter.cs - конвертирует XML:
csharp

public class XmlToBinaryConverter
{
    private readonly string _outputPath;

    public XmlToBinaryConverter(string outputPath)
    {
        _outputPath = outputPath;
        Directory.CreateDirectory(outputPath);
    }

    public async Task ConvertAsync(string xmlFilePath)
    {
        try
        {
            var xmlData = await File.ReadAllTextAsync(xmlFilePath);
            var binaryData = ConvertXmlToBinary(xmlData);
            
            string outputFile = Path.Combine(
                _outputPath, 
                $"{Path.GetFileNameWithoutExtension(xmlFilePath)}.bin");
            
            await File.WriteAllBytesAsync(outputFile, binaryData);
        }
        catch (Exception ex)
        {
            // Логирование ошибок
        }
    }

    private byte[] ConvertXmlToBinary(string xml)
    {
        // Реальная логика конвертации
        using var stream = new MemoryStream();
        using var writer = new BinaryWriter(stream);
        // ... ваша логика сериализации ...
        return stream.ToArray();
    }
}

Worker.cs - связующий компонент:
csharp

public class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;
    private readonly FileWatcherService _fileWatcher;
    private readonly XmlToBinaryConverter _converter;

    public Worker(
        ILogger<Worker> logger,
        FileWatcherService fileWatcher,
        XmlToBinaryConverter converter)
    {
        _logger = logger;
        _fileWatcher = fileWatcher;
        _converter = converter;
    }

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _fileWatcher.NewXmlFileDetected += async file => 
        {
            _logger.LogInformation($"Processing new file: {file}");
            await _converter.ConvertAsync(file);
        };
        
        return Task.CompletedTask;
    }

    public override void Dispose()
    {
        _fileWatcher.Dispose();
        base.Dispose();
    }
}

3. Настройка в Program.cs
csharp

var builder = Host.CreateApplicationBuilder(args);

// Конфигурация путей
string inputPath = Path.Combine(AppContext.BaseDirectory, "XmlInput");
string outputPath = Path.Combine(AppContext.BaseDirectory, "BinaryOutput");

// Регистрация сервисов
builder.Services.AddSingleton(new FileWatcherService(inputPath));
builder.Services.AddSingleton(new XmlToBinaryConverter(outputPath));
builder.Services.AddHostedService<Worker>();

var host = builder.Build();
host.Run();

4. Дополнительные улучшения

Обработка существующих файлов при старте:
csharp

public class FileWatcherService : IDisposable
{
    // ... остальной код ...
    
    public void ProcessExistingFiles()
    {
        foreach (var file in Directory.GetFiles(_inputPath, "*.xml"))
        {
            NewXmlFileDetected?.Invoke(file);
        }
    }
}

В Worker.cs:
csharp

protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    _fileWatcher.ProcessExistingFiles(); // Обработка файлов при старте
    
    _fileWatcher.NewXmlFileDetected += async file => 
    {
        await _converter.ConvertAsync(file);
    };
    
    while (!stoppingToken.IsCancellationRequested)
    {
        await Task.Delay(1000, stoppingToken);
    }
}
