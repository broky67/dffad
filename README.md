public class Service
    {
        private readonly HwServerSettings hwServerSettings = HwServerSettings.Instance;
        public void ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
        {

            try
            {
                var isAvailable = WaitForFileAvailable(xmlFilePath);
                if (isAvailable)
                {
                    var filename = Path.GetFileNameWithoutExtension(xmlFilePath);
                    var outputPath = hwServerSettings.BinFolderPath + filename + ".bin";

                    // Taken from OpenFile HwTool
                    DeviceDescription dataObject = XmlHelper.Deserialize<DeviceDescription>(xmlFilePath);  //check what to use async or sync
                    if (dataObject == null)
                    {
                        throw new InvalidOperationException(RM.Error_deserialize);
                        /*FileLogger.*/
                    }

                    dataObject.ProjPath = xmlFilePath;
                    dataObject._Name = Path.GetFileName(xmlFilePath);
                    var updater = new DeviceRefFinder(dataObject);
                    dataObject.Accept(updater);

                    // Save Binary
                    var bytesArr = new BinaryTarget(dataObject).Export_BinaryTable();
                    if (bytesArr == null)
                    {
                        throw new NullReferenceException(RM.Error_binary_is_empty);
                    }
                    File.WriteAllBytes(outputPath, bytesArr);
                }
            }
            catch (Exception ex)

            {
                throw new InvalidOperationException(RM.Error_couldnt_convert, ex);
            }       
        }
        public bool WaitForFileAvailable(string filePath)
        {
            while (true)
            {
                if (File.Exists(filePath))
                {
                    return true;
                }
                Thread.Sleep(hwServerSettings.FileCheckInterval);
            }
        }
    }
