private DateTime _lastProcessedTime = DateTime.MinValue;
private string _lastProcessedFile = null;

private void OnFileChanged(object sender, FileSystemEventArgs e)
{
    // Игнорируем событие, если тот же файл изменялся менее 1 секунды назад
    if (e.Name == _lastProcessedFile && (DateTime.Now - _lastProcessedTime).TotalSeconds < 1)
        return;

    _lastProcessedTime = DateTime.Now;
    _lastProcessedFile = e.Name;
    FileChanged?.Invoke(this, e);
}


public FileWatcherService(ILogger<FileWatcherService> logger)
        {
            _logger = logger;
            _path = hwServerSettings.XmlFolderPath;

            _watcher = new FileSystemWatcher(_path, "*.xml")
            {
                NotifyFilter = NotifyFilters.LastWrite,
                EnableRaisingEvents = true
            };

            _watcher.Changed += new FileSystemEventHandler(OnFileChanged);
        }

        private void OnFileChanged(object sender, FileSystemEventArgs e)
        {
            FileChanged?.Invoke(this, e);
           /* _logger.LogInformation($"added: {e.Name}");*/
        }


        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            _logger.LogInformation(RM.Worker_running_at + $" {DateTimeOffset.Now}");
            while (!stoppingToken.IsCancellationRequested)
            {
                _fileWatcher.FileChanged += (sender, e) =>
                {
                    try
                    {
                        _converterService.ConvertXmlToBin(e.FullPath, _outputPath);
                        _logger.LogInformation("added ----");
                    }
                    catch (Exception ex)
                    {
                        _logger.LogError(ex, RM.Error_processing_file + $" {e.Name}");
                    }
                };
                await Task.Delay(5000);
            }
        }


public class Service
    {
        private readonly HwServerSettings hwServerSettings = HwServerSettings.Instance;
        private readonly ILogger<Service> _logger;
        public Service(ILogger<Service> logger)
        {
            _logger = logger;
        }
        public void ConvertXmlToBin(string xmlFilePath, string outputDirectory)
        {
            try
            {
                var isAvailable = IsFileReady(xmlFilePath);
                if (isAvailable)
                {
                    var filename = Path.GetFileNameWithoutExtension(xmlFilePath);
                    var outputPath = hwServerSettings.BinFolderPath + filename + ".bin";

                    // Taken from OpenFile HwTool
                    DeviceDescription dataObject = XmlHelper.Deserialize<DeviceDescription>(xmlFilePath); 
                    if (dataObject == null)
                    {
                        _logger.LogError(RM.Error_deserialize);
                        throw new InvalidOperationException(RM.Error_deserialize);
                    }

                    dataObject.ProjPath = xmlFilePath;
                    dataObject._Name = Path.GetFileName(xmlFilePath);
                    var updater = new DeviceRefFinder(dataObject);
                    dataObject.Accept(updater);

                    // Save Binary
                    var bytesArr = new BinaryTarget(dataObject).Export_BinaryTable();
                    if (bytesArr == null)
                    {
                        _logger.LogError(RM.Error_binary_is_empty);
                        throw new NullReferenceException(RM.Error_binary_is_empty);
                    }
                    File.WriteAllBytes(outputPath, bytesArr);
               /* _logger.LogInformation($"added---");*/
                }
            }
            catch (Exception ex)
            {
                _logger.LogError(RM.Error_couldnt_convert, ex);
                throw new InvalidOperationException(RM.Error_couldnt_convert, ex);
            }       
        }
        public bool IsFileReady(string filePath)
        {
            try
            {
                using (var stream = File.Open(filePath, FileMode.Open, FileAccess.Read, FileShare.None))
                {
                    if (stream.Length > 0)
                    {
                        return true;
                    }
                    else
                    {
                        return false;
                    }
                }
            }
            catch (Exception)
            {
                return false;
            }
        }

        /*public bool WaitForFileAvailable(string filePath)
        {
            while (true)
            {
                if (IsFileReady(filePath))
                {
                    return true;
                }
                Thread.Sleep(hwServerSettings.FileCheckInterval);
            }
        }*/
    }
