Чтобы исправить отображение значений в `Property2View` (заменить "ParameterValueType" на числовое значение `0` и убрать дублирование значений), нужно внести следующие изменения:

---

### 1. **Обновить конвертер значений** (самое важное)
Добавьте/измените `ParameterValueConverter`:

```csharp
public class ParameterValueConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        // Если значение - ParameterValueType, преобразуем в int
        if (value is ParameterValueType pvt)
        {
            return pvt.ToInt32(); // Или pvt.Value, если есть свойство
        }
        
        // Если значение - Component, берем его числовое представление
        if (value is TypedefTypeComponent component)
        {
            return component.Value is ParameterValueType compPvt 
                ? compPvt.ToInt32() 
                : component.Value;
        }
        
        return value ?? 0; // Значение по умолчанию
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (int.TryParse(value?.ToString(), out int intValue))
        {
            return new ParameterValueType(intValue);
        }
        return new ParameterValueType(0);
    }
}
```

---

### 2. **Обновить XAML-привязки**
В `Property2View.xaml`:

```xml
<DataGridTemplateColumn Header="Value">
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <!-- Привязка к Tag.Default с конвертером -->
            <TextBlock Text="{Binding Tag.Default, Converter={StaticResource ParameterValueConverter}}"/>
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>
    <DataGridTemplateColumn.CellEditingTemplate>
        <DataTemplate>
            <TextBox Text="{Binding Tag.Default, Converter={StaticResource ParameterValueConverter}, UpdateSourceTrigger=PropertyChanged}"/>
        </DataTemplate>
    </DataGridTemplateColumn.CellEditingTemplate>
</DataGridTemplateColumn>
```

Убедитесь, что конвертер объявлен в ресурсах:
```xml
<UserControl.Resources>
    <local:ParameterValueConverter x:Key="ParameterValueConverter"/>
</UserControl.Resources>
```

---

### 3. **Изменить генерацию `PropertyItemModel`**
В `Property2ViewModel.GenerateItems()` для `ParameterType`:

```csharp
else if (node is ParameterType parameterType)
{
    yield return new EditablePropertyItemModel(parameterType)
    {
        IndentLevel = indentLevel,
        Name = parameterType.Name.ToString(),
        // Убираем дублирование - используем только Default
        GetValue = tag => ((ParameterType)tag).Default,
        SetValue = (tag, value) => 
        {
            if (value is ParameterValueType pvt)
            {
                ((ParameterType)tag).Default = pvt;
            }
        }
    };
}
```

---

### 4. **Обновить `TypedefTypeComponent` (если нужно)**
Если компонент должен возвращать `int` вместо `ParameterValueType`:

```csharp
public class TypedefTypeComponent
{
    private object _value;
    
    public object Value
    {
        get => _value is ParameterValueType pvt ? pvt.ToInt32() : _value;
        set => _value = value;
    }
}
```

---

### 5. **Убрать дублирование в данных**
Проверьте метод `SetComponent` в `ParameterType`, чтобы он не дублировал значения:

```csharp
public void SetComponent()
{
    if (this.Component != null) 
    {
        // Если компонент уже установлен, синхронизируем значения
        this.Default = this.Component.Value is ParameterValueType pvt 
            ? pvt 
            : new ParameterValueType(0);
    }
    else
    {
        // Логика инициализации компонента
    }
}
```

---

### Итоговые изменения:
1. **Конвертер** преобразует `ParameterValueType` → `int` для отображения.
2. **XAML** отображает только `Tag.Default` через конвертер.
3. **Генерация элементов** больше не создает дубликатов.
4. **Компонент** возвращает числовое значение вместо типа.

После этих изменений:
- В интерфейсе будет отображаться `0` вместо "ParameterValueType".
- Не будет дублирования значений между `ParameterType` и его `Component`.
- Редактирование значений останется рабочим.




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
