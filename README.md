    public class ConverterService
    {
        private readonly string _xmlFilePath = @"C:\Users\ba-romanov\xmltest\";
        private ProjectViewModel _projectViewModel;
        public ConverterService(ProjectViewModel projectViewModel)
        {
            _projectViewModel = projectViewModel;
        }
        public async Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
        {
            try
            {
                
                WaitForFileAvailable(xmlFilePath);

                var xmlContent = await File.ReadAllTextAsync(xmlFilePath);
                var outputPath = _xmlFilePath + Path.GetFileNameWithoutExtension(xmlFilePath) + ".bin";


                DeviceDescription selectedProject = _projectViewModel.GetSelectedProject();

                var xmlSerializer = new XmlSerializer(selectedProject.GetType());
                DeviceDescription dataObject;
                using (var reader = new StringReader(xmlContent))
                {
                    dataObject = (DeviceDescription)xmlSerializer.Deserialize(reader);
                }

                await using (var stream = File.Create(outputPath))
                {
                    new BinaryFormatter().Serialize(stream, dataObject);
                }
            }
            catch (Exception ex)
            {
                throw;
            }
        }
        private void WaitForFileAvailable(string filePath, int timeoutMs = 5000)
        {
            var sw = Stopwatch.StartNew();
            while (sw.ElapsedMilliseconds < timeoutMs)
            {
                try
                {
                    using (var fs = File.Open(filePath, FileMode.Open, FileAccess.Read, FileShare.None))
                    {
                        return;
                    }
                }
                catch (Exception ex)
                {
                }
            }
            throw new Exception("time exception");
        }
    }

<DeviceDescription xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.ekra.ru/schemas/PlatformDescription-1.0.xsd">
<Types/>
<Resources/>
<ParameterSet/>
</DeviceDescription>
