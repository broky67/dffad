        internal void Export_BinaryExecute()
        {
            DeviceDescription devdesc = GetSelectedProject();
            if (devdesc != null)
            {
                _saveFile.FileName = Path.GetFileNameWithoutExtension(devdesc.ProjPath);
                _saveFile.Filter = "Platform description tables (*.bin)|*.bin|All files (*.*)|*.*";
                if (_saveFile.DetermineFile())
                {
                    try
                    {
                        var barr = new BinaryTarget(devdesc).Export_BinaryTable();
                        File.WriteAllBytes(_saveFile.FileName, barr);
                        System.Media.SystemSounds.Asterisk.Play();
                    }
                    catch (Exception ex)
                    {
                        Clipboard.Clear();
                        System.Media.SystemSounds.Exclamation.Play();
                    }
                }
            }
        }

        public DeviceDescription GetSelectedProject()
        {
            _IDeviceDescriptionNode node = SelectedNode;
            while (node != null)
            {
                if (node is DeviceDescription proj)
                    return proj;
                else if (node != null)
                    node = node._Parent;
            }
            return ProjRoot.FirstOrDefault();
        }


public class ConverterService
    {
        private readonly string _xmlFilePath = @"C:\Users\ba-romanov\xmltest\";
        private DeviceDescription dataObject;
        private Pilot.HwTool.ProjectViewModel _projectView;
        public ConverterService(Pilot.HwTool.ProjectViewModel projectView)
        {
            _projectView = projectView;
        }

        public async Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
        {
            try
            {
                WaitForFileAvailable(xmlFilePath);

                var xmlContent = await File.ReadAllTextAsync(xmlFilePath);
                var outputPath = _xmlFilePath + Path.GetFileNameWithoutExtension(xmlFilePath) + ".bin";


                DeviceDescription devdesc = _projectView.GetSelectedProject();

                var xmlSerializer = new XmlSerializer(devdesc.GetType());
               
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


var xmlSerializer = new XmlSerializer(devdesc.GetType());
System.NullReferenceException: 'Object reference not set to an instance of an object.'
