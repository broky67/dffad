Если нужно избежать `dynamic`, можно явно определить интерфейс или базовый класс для узлов, у которых есть `_Parent`, и использовать приведение типов.  

### **1. Определим интерфейс для узлов с родителем**  
Допустим, у всех узлов есть свойство `_Parent` типа `object` (или более конкретного базового типа). Создадим интерфейс:  

```csharp
public interface IParentNode
{
    object _Parent { get; }  // или более конкретный тип, если известен
}
```

### **2. Используем явное приведение типов**  
Теперь код можно переписать без `dynamic`, полагаясь на интерфейс:  

```csharp
else if (node is ParameterType parameterType)
{
    var paramTypeField = parameterType.type;
    var paramTypeFieldWithoutLocal = paramTypeField.Substring(paramTypeField.IndexOf(':') + 1);

    TypedefTypeComponentCollection components = null;

    // Ищем DeviceDescription, поднимаясь по родителям
    DeviceDescription devDescNode = null;
    object current = node;

    while (current is IParentNode parentNode)
    {
        current = parentNode._Parent;
        if (current is DeviceDescription desc)
        {
            devDescNode = desc;
            break;
        }
    }

    if (devDescNode != null)
    {
        var constStringName = devDescNode.Types.Items
            .FirstOrDefault(x => x._Name?.Contains(paramTypeFieldWithoutLocal) == true)?._Name;

        if (constStringName != null)
        {
            var matchingItem = devDescNode.Types.Items
                .FirstOrDefault(x => x._Name == constStringName);

            components = matchingItem switch
            {
                StructdefType structDef => structDef.Component,
                BitfielddefType bitDef => bitDef.Component,
                _ => null
            };
        }
    }
}
```

### **Ключевые изменения:**
1. **Убран `dynamic`** → используется интерфейс `IParentNode` и явное приведение типов.  
2. **Проверка `_Name?.Contains()`** → безопасная обработка `null`.  
3. **Использован `switch`** для выбора типа компонента (альтернатива `if-else`).  

### **Если нельзя изменить исходные классы (нет доступа к `IParentNode`)**
Можно использовать приведение к `object` и проверку на `null`:  

```csharp
DeviceDescription devDescNode = null;
object current = node;

while (current != null)
{
    if (current is DeviceDescription desc)
    {
        devDescNode = desc;
        break;
    }

    // Получаем _Parent через рефлексию (если нет интерфейса)
    var parentProperty = current.GetType().GetProperty("_Parent");
    current = parentProperty?.GetValue(current);
}
```

### **Итог:**
- **Лучший вариант** → добавить `IParentNode` и работать без `dynamic`.  
- **Если нельзя менять код** → использовать рефлексию (медленнее, но гибко).  
- **Самая простая альтернатива** → явный цикл с приведением к `object` (как в первом примере).  

Код стал типобезопасным и не требует `dynamic` 🚀.
