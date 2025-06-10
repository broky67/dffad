            try
            {
                
                WaitForFileAvailable(xmlFilePath);

                var xmlContent = await File.ReadAllTextAsync(xmlFilePath);
                var outputPath = _xmlFilePath + Path.GetFileNameWithoutExtension(xmlFilePath) + ".bin";

                var xmlSerializer = new XmlSerializer(typeof(DeviceDescription));

                ProjectViewModel project;
                var temp = project.GetSelectedProject();

                using (var reader = new StringReader(xmlContent))
                {
                    dataObject = (DeviceDescription)xmlSerializer.Deserialize(reader);
                }

                await using (var stream = File.Create(outputPath))
                {
                    new BinaryFormatter().Serialize(stream, dataObject);
                }
            }

            Severity	Code	Description	Project	File	Line	Suppression State
Error	CS0165	Use of unassigned local variable 'project'	XmlToBinaryConverterService	C:\Users\ba-romanov\projects\Pilot\XmlToBinaryConverterService\ConverterService.cs	29	Active
