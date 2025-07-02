public class HwServerSettings
{
    // Корень приложения (где находится .dll)
    private static readonly string AppRoot = AppContext.BaseDirectory;

    // Относительные пути (папки создадутся рядом с .dll)
    public string XmlFolderPath => Path.Combine(AppRoot, "Xml");
    public string BinFolderPath => Path.Combine(AppRoot, "BinOutput");

    // Синглтон
    private static readonly Lazy<HwServerSettings> _instance = new Lazy<HwServerSettings>(() => new HwServerSettings());
    public static HwServerSettings Instance => _instance.Value;

    private HwServerSettings()
    {
        // Создаем папки при инициализации
        Directory.CreateDirectory(XmlFolderPath);
        Directory.CreateDirectory(BinFolderPath);
    }
}




public class FileLogger : ILogger
{
    private readonly string _filePath;

    public FileLogger(string filePath)
    {
        _filePath = filePath;
    }

    public void Log<TState>(
        LogLevel logLevel,
        EventId eventId,
        TState state,
        Exception exception,
        Func<TState, Exception, string> formatter)
    {
        var message = formatter(state, exception);
        var timestamp = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss.fff");
        var logMessage = $"[{timestamp}] [{logLevel}] {message}{Environment.NewLine}";

        File.AppendAllText(_filePath, logMessage);
    }

    // Остальные методы не меняем
}

[Unit]
Description=Xml to Bin converter service

[Service]
Type=notify

ExecStart=/usr/bin/dotnet /media/sf_UbuntuShare/XmlConverter/Pilot.HwTool.Service

Environment=ASPNETCORE_ENVIRONMENT=Production
[Install]
WantedBy=multi-user.target

'''''
.ConfigureLogging(logging =>
{
    logging.AddConsole(options =>
    {
        options.TimestampFormat = "[yyyy-MM-dd HH:mm:ss] ";
    });
})
