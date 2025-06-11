var project = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(fileName);
            if (project != null)
            {
                project.ProjPath = fileName;
                project._Name = Path.GetFileName(fileName);

                UpdateLatest(fileName);

                //UpdateDevices(project);
                var updater = new DeviceRefFinder(project);
                project.Accept(updater);
            }
            return project;
...


public async Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
        {
            try
            {
                
                WaitForFileAvailable(xmlFilePath);

                var xmlContent = await File.ReadAllTextAsync(xmlFilePath);
                var filename = Path.GetFileNameWithoutExtension(xmlFilePath);
                var outputPath = _xmlFilePath + filename + ".bin";


                var xmlSerializer = new XmlSerializer(typeof(DeviceDescription));
                DeviceDescription dataObject;

                using (var reader = new StringReader(xmlContent))           
                {
                    // instead of 
                    // dataObject = (DeviceDescription)xmlSerializer.Deserialize(reader);
                    dataObject = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(filename);
                    dataObject.ProjPath = xmlFilePath;
                    dataObject._Name = Path.GetFileName(xmlFilePath);

                    var updater = new DeviceRefFinder(dataObject);
                    dataObject.Accept(updater);
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
