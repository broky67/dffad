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

_______
1. Бинарная сериализация (для простых объектов)
csharp

private byte[] ConvertXmlToBinary(string xml)
{
    // 1. Десериализация XML в объект
    var serializer = new XmlSerializer(typeof(DataModel));
    using var xmlReader = new StringReader(xml);
    var data = (DataModel)serializer.Deserialize(xmlReader)!;

    // 2. Бинарная сериализация
    using var memoryStream = new MemoryStream();
    var binaryFormatter = new BinaryFormatter();
    
    binaryFormatter.Serialize(memoryStream, data);
    return memoryStream.ToArray();
}

Где DataModel - ваш класс для хранения данных:
csharp

[Serializable]
[XmlRoot("Data")]
public class DataModel
{
    [XmlElement("Id")]
    public int Id { get; set; }
    
    [XmlElement("Name")]
    public string Name { get; set; }
    
    [XmlArray("Items")]
    [XmlArrayItem("Item")]
    public List<string> Items { get; set; }
}

2. Ручная упаковка (для максимальной производительности)
csharp

private byte[] ConvertXmlToBinary(string xml)
{
    using var xmlDoc = XDocument.Parse(xml);
    using var memoryStream = new MemoryStream();
    using var writer = new BinaryWriter(memoryStream);

    // Сериализация основных полей
    var root = xmlDoc.Root!;
    writer.Write(root.Element("Id")?.Value ?? "0");
    writer.Write(root.Element("Name")?.Value ?? "");
    
    // Сериализация коллекции
    var items = root.Element("Items")?.Elements("Item");
    writer.Write(items?.Count() ?? 0);
    foreach (var item in items ?? Enumerable.Empty<XElement>())
    {
        writer.Write(item.Value);
    }

    return memoryStream.ToArray();
}

3. Альтернатива с System.Text.Json (для .NET Core+)
csharp

private byte[] ConvertXmlToBinary(string xml)
{
    // XML -> Объект
    using var xmlDoc = XDocument.Parse(xml);
    var data = new DataModel
    {
        Id = int.Parse(xmlDoc.Root?.Element("Id")?.Value ?? "0"),
        Name = xmlDoc.Root?.Element("Name")?.Value ?? ""
    };

    // Объект -> Binary JSON
    return JsonSerializer.SerializeToUtf8Bytes(data);
}

Полный пример с обработкой ошибок
csharp

private byte[] ConvertXmlToBinary(string xml)
{
    try
    {
        // Валидация XML
        if (string.IsNullOrWhiteSpace(xml))
            throw new ArgumentException("Empty XML content");

        using var xmlDoc = XDocument.Parse(xml);
        
        // Проверка обязательных полей
        if (xmlDoc.Root?.Element("Id") == null)
            throw new InvalidOperationException("Missing required Id field");

        // Основная логика сериализации
        using var memoryStream = new MemoryStream();
        using (var writer = new BinaryWriter(memoryStream))
        {
            writer.Write(xmlDoc.Root.Element("Id")!.Value);
            writer.Write(xmlDoc.Root.Element("Name")?.Value ?? "");
            
            // Пример обработки коллекции
            var items = xmlDoc.Root.Element("Items")?.Elements() 
                ?? Enumerable.Empty<XElement>();
            
            writer.Write(items.Count());
            foreach (var item in items)
            {
                writer.Write(item.Value);
            }
        }

        return memoryStream.ToArray();
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "XML to binary conversion failed");
        throw; // или возврат пустого массива
    }
}
