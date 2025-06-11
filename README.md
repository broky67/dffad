public async Task ConvertXmlToBinAsync(string xmlFilePath, string outputDirectory)
{
    try
    {
        var filename = Path.GetFileNameWithoutExtension(xmlFilePath);
        var outputPath = Path.Combine(outputDirectory, filename + ".bin");
        
        // Ожидание доступности файла
        await WaitForFileAvailable(xmlFilePath);

        // Десериализация XML
        DeviceDescription dataObject = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(xmlFilePath);
        if (dataObject == null)
        {
            throw new InvalidOperationException("Failed to deserialize XML file");
        }

        // Обновление свойств объекта
        dataObject.ProjPath = xmlFilePath;
        dataObject._Name = Path.GetFileName(xmlFilePath);
        var updater = new DeviceRefFinder(dataObject);
        dataObject.Accept(updater);

        // Бинарная сериализация через существующий метод Serialize()
        byte[] binaryData = dataObject.Serialize(); // Вызываем ваш метод Serialize()
        
        // Сохранение бинарных данных
        await File.WriteAllBytesAsync(outputPath, binaryData);
    }
    catch (Exception ex)
    {
        throw new InvalidOperationException($"Error processing {xmlFilePath}", ex);
    }
}


