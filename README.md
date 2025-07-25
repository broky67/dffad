Проблема возникает из-за того, что `item.Tag` является `object` и напрямую не приводится к `_IDeviceDescriptionNode`. Вот исправленная версия с правильной проверкой типов:

```csharp
public PropertyItemCollection Items
{
    get => _items;
    private set
    {
        _items = value;
        
        // Инициализация компонентов
        if (_items != null)
        {
            foreach (var item in _items)
            {
                // Безопасная проверка типа через as
                var node = item.Tag as _IDeviceDescriptionNode;
                if (node != null)
                {
                    SetComponentService.FindParameterComponents(node);
                }
            }
        }
        
        RaisePropertyChanged();
    }
}
```

Или альтернативный вариант с явной проверкой:

```csharp
public PropertyItemCollection Items
{
    get => _items;
    private set
    {
        _items = value;
        
        if (_items != null)
        {
            foreach (var item in _items)
            {
                // Проверка через is с приведением
                if (item.Tag is _IDeviceDescriptionNode node)
                {
                    SetComponentService.FindParameterComponents(node);
                }
            }
        }
        
        RaisePropertyChanged();
    }
}
```

### Полное решение для безопасной работы с Tag:

1. Добавьте метод-помощник в `Property2ViewModel`:

```csharp
private bool TryGetDeviceDescriptionNode(object tag, out _IDeviceDescriptionNode node)
{
    node = tag as _IDeviceDescriptionNode;
    return node != null;
}
```

2. Используйте его в Items:

```csharp
public PropertyItemCollection Items
{
    get => _items;
    private set
    {
        _items = value;
        
        if (_items != null)
        {
            foreach (var item in _items)
            {
                if (TryGetDeviceDescriptionNode(item.Tag, out var node))
                {
                    SetComponentService.FindParameterComponents(node);
                }
            }
        }
        
        RaisePropertyChanged();
    }
}
```

### Дополнительная проверка для DeviceDescriptionDevice:

Если вам нужно особое поведение для DeviceDescriptionDevice:

```csharp
foreach (var item in _items)
{
    if (item.Tag is DeviceDescriptionDevice device)
    {
        // Специальная обработка для DeviceDescriptionDevice
        SetComponentService.FindParameterComponents(device);
    }
    else if (TryGetDeviceDescriptionNode(item.Tag, out var node))
    {
        // Общая обработка для других узлов
        SetComponentService.FindParameterComponents(node);
    }
}
```

Это решение:
1. Корректно работает с Tag как object
2. Избегает исключений приведения типов
3. Сохраняет всю бизнес-логику
4. Позволяет добавлять специальные обработчики для конкретных типов
