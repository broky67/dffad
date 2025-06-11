        private async Task<DeviceDescription> OpenProject(string fileName)
        {
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
        }

        private async Task SaveProject(string fileName, DeviceDescription project)
        {
            if (await Pilot.XmlHelper.SerializeAsync(fileName, project))
            {
                project.ProjPath = fileName;
                project._Name = Path.GetFileName(fileName);

                UpdateLatest(fileName);
            }
        }

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
                    dataObject = (DeviceDescription)xmlSerializer.Deserialize(reader);
                    // dataObject = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(filename);
                    dataObject.ProjPath = xmlFilePath;
                    dataObject._Name = Path.GetFileName(xmlFilePath);

                    var updater = new DeviceRefFinder(dataObject);
                    dataObject.Accept(updater);
                }



                /*await using (var stream = File.Create(outputPath))
                {
                    new BinaryFormatter().Serialize(stream, dataObject);
                }*/

                await using (var stream = File.Create(outputPath))
                {
                    Pilot.XmlHelper.SerializeAsync(filename, dataObject);

                    dataObject.ProjPath = filename;
                    dataObject._Name = Path.GetFileName(filename);
                }
            }

            catch (Exception ex)
            {
                throw;
            }
        }

public static bool Serialize(string fileName, object obj, XmlSerializerNamespaces xmlns = null)
        {
            // Failed by default
            bool succeeded = false;

            try
            {
                if (obj != null)
                {
                    string parentDir = Directory.GetParent(fileName).FullName;
                    if (!Directory.Exists(parentDir))
                        Directory.CreateDirectory(parentDir);

                    using (Stream stream = File.Create(fileName))
                    {
                        var xs = new XmlSerializer(obj.GetType());
                        xs.Serialize(stream, obj, xmlns);

                        succeeded = true;
                    }
                }
            }
            catch (Exception ex)
            {
                // silent ignore
                PrintInnerEx(ex);
            }

            return succeeded;
        }

        /// <summary>
        /// Serializes the XML asynchronously.
        /// </summary>
        /// <param name="fileName">Name of the file.</param>
        /// <param name="obj">The object.</param>
        /// <param name="xmlns">Extra XML namespaces.</param>
        /// <returns><c>true</c> if the object is serialized to xml successfully; otherwise <c>false</c>.</returns>
        public async static Task<bool> SerializeAsync(string fileName, object obj, XmlSerializerNamespaces xmlns = null)
        {
            bool succeeded = await new TaskFactory().StartNew(() =>
                XmlHelper.Serialize(fileName, obj, xmlns));
            return succeeded;
        }

        /// <summary>
        /// Deserializes the specified file name to an object.
        /// </summary>
        /// <typeparam name="T">Type of the object that is contained in the file.</typeparam>
        /// <param name="fileName">Name of the file.</param>
        /// <returns>Deserialized type or <c>null</c> if not successful.</returns>
        public static T Deserialize<T>(string fileName)
            where T : class
        {
            T result = default(T);

            try
            {
                using (Stream stream = File.Open(fileName, FileMode.Open, FileAccess.Read))
                {
                    var xs = new XmlSerializer(typeof(T));
                    result = (T)xs.Deserialize(stream);
                }
            }
            catch (Exception ex)
            {
                // silent ignore
                PrintInnerEx(ex);
            }

            return result;
        }

        /// <summary>
        /// Deserializes the specified file name to an object asynchronously.
        /// </summary>
        /// <typeparam name="T">Type of the object that is contained in the file.</typeparam>
        /// <param name="fileName">Name of the file.</param>
        /// <returns>Deserialized type or <c>null</c> if not successful.</returns>
        public async static Task<T> DeserializeAsync<T>(string fileName)
            where T : class
        {
            T result = await new TaskFactory<T>().StartNew(() =>
                XmlHelper.Deserialize<T>(fileName));
            return result;
        }
