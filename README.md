public async Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
{
    try
    {
        await WaitForFileAvailable(xmlFilePath);

        // 1. Десериализация XML
        DeviceDescription dataObject = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(xmlFilePath);
        if (dataObject == null)
        {
            throw new InvalidOperationException("Failed to deserialize XML file");
        }

        // 2. Обновление свойств (как в OpenProject)
        dataObject.ProjPath = xmlFilePath;
        dataObject._Name = Path.GetFileName(xmlFilePath);
        var updater = new DeviceRefFinder(dataObject);
        dataObject.Accept(updater);

        // 3. Создание бинарного файла
        var outputPath = Path.Combine(outputDirectory, 
            Path.GetFileNameWithoutExtension(xmlFilePath) + ".bin");

        // 4. Настоящая бинарная сериализация
        await using (var stream = File.Create(outputPath))
        {
            var formatter = new BinaryFormatter();
            formatter.Serialize(stream, dataObject);
        }
    }
    catch (Exception ex)
    {
        throw new InvalidOperationException($"Failed to convert {xmlFilePath}", ex);
    }
}
