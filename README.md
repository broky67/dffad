        public async Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
        {
            try
            {
                var filename = Path.GetFileNameWithoutExtension(xmlFilePath);
                var outputPath = _xmlFilePath + filename + ".bin";
                WaitForFileAvailable(xmlFilePath);

                DeviceDescription dataObject = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(xmlFilePath);
                if (dataObject == null)
                {
                    throw new InvalidOperationException("Failed to deserialize XML file");
                }

                dataObject.ProjPath = xmlFilePath;
                dataObject._Name = Path.GetFileName(xmlFilePath);
                var updater = new DeviceRefFinder(dataObject);
                dataObject.Accept(updater);


  /*              await using (var stream = File.Create(outputPath))
                {*/
                await Pilot.XmlHelper.SerializeAsync(filename, dataObject);

               /*     dataObject.ProjPath = filename;
                    dataObject._Name = Path.GetFileName(filename);*/
                /*}*/
            }
            catch (Exception ex)
            {
                throw new InvalidOperationException($"Exception {xmlFilePath}", ex);
            }
        }
