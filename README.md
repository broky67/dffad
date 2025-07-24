private ParameterType FindBusAddressParameter(_IDeviceDescriptionNode node)
{
    var queue = new Queue<_IDeviceDescriptionNode>();
    queue.Enqueue(node);

    while (queue.Count > 0)
    {
        var current = queue.Dequeue();
        
        if (current is ParameterType parameterType && 
            parameterType._Name == "Адрес блока на шине")
        {
            return parameterType; // Первое точное совпадение
        }

        foreach (var child in current._Children ?? Enumerable.Empty<_IDeviceDescriptionNode>())
        {
            queue.Enqueue(child); // Обход в ширину
        }
    }
    return null;
}
