else if (node is ParameterType parameterType)
{
    var paramTypeField = parameterType.type;
    var paramTypeFieldWithoutLocal = paramTypeField.Substring(paramTypeField.IndexOf(':') + 1);

    TypedefTypeComponentCollection components = null;

    // Получаем DeviceDescription через вспомогательный метод
    var devDescNode = FindAncestorOfType<DeviceDescription>(node);
    if (devDescNode != null)
    {
        var descriptonTypes = devDescNode.Types;
        var items = descriptonTypes.Items;

        var constStringName = items.FirstOrDefault(x => x._Name.Contains(paramTypeFieldWithoutLocal))?._Name;
        if (constStringName != null)
        {
            var matchingItem = items.FirstOrDefault(x => x._Name == constStringName);

            if (matchingItem is StructdefType structDef)
            {
                components = structDef.Component;
            }
            else if (matchingItem is BitfielddefType bitDef)
            {
                components = bitDef.Component;
            }
        }
    }
}

// Вспомогательный метод для поиска родителя определенного типа
private static T FindAncestorOfType<T>(object node) where T : class
{
    dynamic current = node;
    while (current != null)
    {
        if (current is T result)
            return result;
        
        current = current._Parent; // предполагается, что у всех узлов есть _Parent
    }
    return null;
}
