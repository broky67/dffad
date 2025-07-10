public partial class ParameterType : _DeviceDescriptionNode
{
    // Существующие поля
    private TypedefTypeComponentCollection _linkedComponents;
    
    public ParameterType()
    {
        _linkedComponents = new TypedefTypeComponentCollection(this);
        typeField = "std:INT";
        defaultField = new[] { new ParameterValueType() { Text = new[] { "0" } } };
    }

    [XmlIgnore]
    public TypedefTypeComponentCollection LinkedComponents
    {
        get => _linkedComponents;
        set
        {
            _linkedComponents = value;
            RaisePropertyChanged("LinkedComponents");
        }
    }

    // Остальные существующие свойства...
}

2. Добавляем связывающий метод
csharp

public class ComponentLinker
{
    public void LinkComponents(DeviceDescription device)
    {
        // Находим все StructdefType в устройстве
        var structDefs = device.Items.OfType<StructdefType>().ToList();
        
        // Находим все ParameterType
        var parameters = device.Items.OfType<ParameterType>().ToList();

        foreach (var param in parameters)
        {
            // Ищем подходящие компоненты по соглашению об именах
            foreach (var structDef in structDefs)
            {
                if (structDef.Component == null) continue;
                
                foreach (var component in structDef.Component)
                {
                    // Пример: если имя параметра содержится в имени компонента
                    if (component.Name.ToString().Contains(param.Name.ToString()))
                    {
                        param.LinkedComponents.Add(component);
                    }
                }
            }
        }
    }
}

3. Пример использования
csharp

// После загрузки XML
var device = LoadDeviceDescription();

// Связываем компоненты
new ComponentLinker().LinkComponents(device);

// Теперь в каждом ParameterType.LinkedComponents будут связанные компоненты

4. Модификация для отображения
csharp

private IEnumerable<PropertyItemModel> GenerateItems(_DeviceDescriptionNode node, int indentLevel)
{
    if (node is ParameterType parameter)
    {
        yield return new EditablePropertyItemModel(parameter)
        {
            IndentLevel = indentLevel,
            Name = parameter.Name.ToString()
        };

        // Добавляем связанные компоненты
        foreach (var component in parameter.LinkedComponents)
        {
            yield return new EditablePropertyItemModel(component)
            {
                IndentLevel = indentLevel + 1,
                Name = component.Name.ToString()
            };
        }
    }
    // ... остальная логика ...
}
