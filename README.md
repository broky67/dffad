public partial class TypedefTypeComponentCollection : _DeviceDescriptionNodeCollection<TypedefTypeComponent>, ICloneable
{
    // ... существующие конструкторы ...

    public object Clone()
    {
        var clone = new TypedefTypeComponentCollection(this.Parent);
        
        // Используем AddRange для эффективного добавления элементов
        clone.AddRange(
            this.Items
                .Where(item => item != null)
                .Select(item => (TypedefTypeComponent)item.Clone())
        );
        
        return clone;
    }
}
`TypeDefComponentCollection` с поддержкой глубокого копирования всех элементов, включая вложенные объекты, вот полное решение:

### 1. Сначала реализуем `ICloneable` для `TypedefTypeComponent`

```csharp
public partial class TypedefTypeComponent : _DeviceDescriptionNode, ICloneable
{
    // ... существующие свойства ...

    public object Clone()
    {
        var clone = new TypedefTypeComponent
        {
            // Копируем все значимые типы и строки
            identifierField = this.identifierField,
            typeField = this.typeField,
            
            // Клонируем сложные объекты
            attributesField = this.attributesField != null 
                ? new ParameterTypeAttributes 
                {
                    // Копируем свойства attributesField
                    // Пример:
                    // SomeProperty = this.attributesField.SomeProperty
                } 
                : null,
                
            defaultField = this.defaultField != null 
                ? new ParameterValueType { Text = (string[])this.defaultField.Text?.Clone() } 
                : null,
                
            visibleNameField = this.visibleNameField != null 
                ? new StringRefType { Value = this.visibleNameField.Value } 
                : null,
                
            unitField = this.unitField != null 
                ? new StringRefType { Value = this.unitField.Value } 
                : null,
                
            descriptionField = this.descriptionField != null 
                ? new StringRefType { Value = this.descriptionField.Value } 
                : null,
                
            anyField = this.anyField != null 
                ? (System.Xml.XmlElement[])this.anyField.Clone() 
                : null
        };

        return clone;
    }
}
```

### 2. Реализуем `ICloneable` для `TypedefTypeComponentCollection`

```csharp
public partial class TypedefTypeComponentCollection : _DeviceDescriptionNodeCollection<TypedefTypeComponent>, ICloneable
{
    // ... существующие конструкторы ...

    public object Clone()
    {
        var clone = new TypedefTypeComponentCollection(this.Parent)
        {
            // Клонируем каждый элемент коллекции
            Items = this.Items.Select(item => item != null 
                ? (TypedefTypeComponent)item.Clone() 
                : null).ToList()
        };

        return clone;
    }
}
```

### 3. Альтернативная реализация (если нужна полная независимость от Parent)

```csharp
public object Clone()
{
    // Создаем коллекцию без родителя (или с новым родителем)
    var clone = new TypedefTypeComponentCollection(null);
    
    // Глубокое копирование элементов
    foreach (var item in this.Items)
    {
        if (item != null)
        {
            var itemClone = (TypedefTypeComponent)item.Clone();
            clone.Add(itemClone);
        }
    }
    
    return clone;
}
```

### 4. Улучшенная версия с проверками

```csharp
public object Clone()
{
    try
    {
        var clone = new TypedefTypeComponentCollection(this.Parent);
        
        foreach (var item in this.Items.Where(i => i != null))
        {
            var clonedItem = item.Clone() as TypedefTypeComponent;
            if (clonedItem != null)
            {
                clone.Add(clonedItem);
            }
        }
        
        return clone;
    }
    catch (Exception ex)
    {
        // Логирование ошибки
        Debug.WriteLine($"Cloning failed: {ex.Message}");
        throw new InvalidOperationException("Failed to clone TypedefTypeComponentCollection", ex);
    }
}
```

### Критические моменты реализации:

1. **Глубина копирования**:
   - Все ссылочные типы должны клонироваться рекурсивно
   - Особое внимание полям `attributesField`, `defaultField` и другим сложным объектам

2. **Производительность**:
   - Для больших коллекций рассмотрите использование `Parallel.ForEach`

3. **Сериализация XmlElement**:
   - Поле `anyField` требует специальной обработки, так как `XmlElement` не поддерживает глубокое копирование по умолчанию

### Пример использования:

```csharp
var originalCollection = new TypedefTypeComponentCollection(parent);
// ... заполнение коллекции ...

var clonedCollection = (TypedefTypeComponentCollection)originalCollection.Clone();

// Проверка, что это разные объекты
Console.WriteLine(ReferenceEquals(originalCollection, clonedCollection)); // False

// Проверка, что элементы тоже клонированы
Console.WriteLine(ReferenceEquals(originalCollection[0], clonedCollection[0])); // False
```

### Дополнительные улучшения:

1. **Реализация `ICloneable<T>`** (рекомендуется):
```csharp
public interface ICloneable<T> : ICloneable
{
    new T Clone();
}

public class TypedefTypeComponentCollection : ICloneable<TypedefTypeComponentCollection>
{
    public TypedefTypeComponentCollection Clone()
    {
        // ... реализация ...
    }
    
    object ICloneable.Clone() => this.Clone();
}
```

2. **Фабричный метод** (альтернатива):
```csharp
public TypedefTypeComponentCollection CreateDeepCopy()
{
    return Clone() as TypedefTypeComponentCollection;
}
```

 
 
 
 public partial class TypedefTypeComponent : _DeviceDescriptionNode
    {

        //TODO
        private ParameterTypeAttributes attributesField;

        private ParameterValueType defaultField;

        private StringRefType visibleNameField;

        private StringRefType unitField;

        private StringRefType descriptionField;

        //private string filterFlagsField;

        //private TypedefTypeComponentStateFlags stateField;

        [System.NonSerialized] private System.Xml.XmlElement[] anyField;

        private string identifierField;

        private string typeField;

        //private TypedefTypeComponentOnlineaccess onlineaccessField;

        //private TypedefTypeComponentOfflineaccess offlineaccessField;
        public TypedefTypeComponent()
        {
            //this.stateField = TypedefTypeComponentStateFlags.none;
            //this.onlineaccessField = TypedefTypeComponentOnlineaccess.readwrite;
            //this.offlineaccessField = TypedefTypeComponentOfflineaccess.readwrite;
            defaultField = new ParameterValueType() { Text = new[] { "0" } };
        }



        /// <remarks/>
        public ParameterTypeAttributes Attributes
        {
            get
            {
                return this.attributesField;
            }
            set
            {
                this.attributesField = value;
                this.RaisePropertyChanged("Attributes");
            }
        }

        /// <remarks/>
        public ParameterValueType Default
        {
            get
            {
                return this.defaultField;
            }
            set
            {
                this.defaultField = value;
                this.RaisePropertyChanged("Default");
            }
        }

        /// <remarks/>
        public StringRefType VisibleName
        {
            get
            {
                return this.visibleNameField;
            }
            set
            {
                this.visibleNameField = value;
                this.RaisePropertyChanged("VisibleName");
            }
        }

        /// <remarks/>
        public StringRefType Unit
        {
            get
            {
                return this.unitField;
            }
            set
            {
                this.unitField = value;
                this.RaisePropertyChanged("Unit");
            }
        }

        /// <remarks/>
        public StringRefType Description
        {
            get
            {
                return this.descriptionField;
            }
            set
            {
                this.descriptionField = value;
                this.RaisePropertyChanged("Description");
            }
        }

        ///// <remarks/>
        //[XmlElement(DataType = "NMTOKENS")]
        //public string FilterFlags
        //{
        //    get
        //    {
        //        return this.filterFlagsField;
        //    }
        //    set
        //    {
        //        this.filterFlagsField = value;
        //        this.RaisePropertyChanged("FilterFlags");
        //    }
        //}

        /// <remarks/>
        [XmlAnyElement]
        public System.Xml.XmlElement[] Any
        {
            get
            {
                return this.anyField;
            }
            set
            {
                this.anyField = value;
                this.RaisePropertyChanged("Any");
            }
        }

        /// <remarks/>
        [XmlAttribute(DataType = "NCName")]
        public string identifier
        {
            get
            {
                return this.identifierField;
            }
            set
            {
                this.identifierField = value;
                this.RaisePropertyChanged("identifier");
            }
        }

        /// <remarks/>
        [XmlAttribute()]
        public string type
        {
            get
            {
                return this.typeField;
            }
            set
            {
                this.typeField = value;
                this.RaisePropertyChanged("type");
            }
        }

/*        public TypedefTypeComponent(ParameterTypeAttributes attributes,
    ParameterValueType defaultField,
    StringRefType visibleName,
    StringRefType unit,
    StringRefType description,
    System.Xml.XmlElement[] any,
    string identifierField,
    string typeField)
        {
            Attributes = attributes;
            Default = defaultField;
            VisibleName = visibleName;
            Unit = unit;
            Description = description;
            Any = any;
            identifier = identifierField;
            type = typeField;
        }
        public object Clone()
        {
            return new TypedefTypeComponent(Attributes, Default, VisibleName, Unit, Description, Any, identifier, type);
        }*/

        ///// <remarks/>
        //[XmlAttribute()]
        //[DefaultValue(TypedefTypeComponentStateFlags.none)]
        //public TypedefTypeComponentStateFlags state
        //{
        //    get
        //    {
        //        return this.stateField;
        //    }
        //    set
        //    {
        //        this.stateField = value;
        //        this.RaisePropertyChanged("state");
        //    }
        //}

        ///// <remarks/>
        //[XmlAttribute()]
        //[DefaultValue(TypedefTypeComponentOnlineaccess.readwrite)]
        //public TypedefTypeComponentOnlineaccess onlineaccess
        //{
        //    get
        //    {
        //        return this.onlineaccessField;
        //    }
        //    set
        //    {
        //        this.onlineaccessField = value;
        //        this.RaisePropertyChanged("onlineaccess");
        //    }
        //}

        ///// <remarks/>
        //[XmlAttribute()]
        //[DefaultValue(TypedefTypeComponentOfflineaccess.readwrite)]
        //public TypedefTypeComponentOfflineaccess offlineaccess
        //{
        //    get
        //    {
        //        return this.offlineaccessField;
        //    }
        //    set
        //    {
        //        this.offlineaccessField = value;
        //        this.RaisePropertyChanged("offlineaccess");
        //    }
        //}
    }

    public partial class TypedefTypeComponentCollection : _DeviceDescriptionNodeCollection<TypedefTypeComponent>
    {
        protected TypedefTypeComponentCollection()
        {
        }

        public TypedefTypeComponentCollection(_IDeviceDescriptionNode parent)
            : base(parent)
        {
        }

        public TypedefTypeComponentCollection(_IDeviceDescriptionNode parent, IEnumerable<TypedefTypeComponent> items)
            : base(parent)
        {
            Items.AddRange(items);
        }

        public object Clone()
        {
            return new TypedefTypeComponentCollection();
        }
    }







            else if (node is ParameterSectionType sectionType)
            {
                var sectionTypeName = (sectionType.Name == null) ? sectionType._Name : sectionType.Name.ToString();
                yield return new EditablePropertyItemModel(sectionType)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    //Name = sectionTypeName,
                    GetName = tag => { return (sectionType.Name == null) ? sectionType._Name : sectionType.Name.ToString(); },
                    SetName = (tag, value) => { sectionType.Name = (StringRefType)value; },
                };

                foreach (var item in sectionType.Items)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
    public class PropertyItemModel : Mvvm.ObservableObject
    {
        private int _indentLevel;
        private bool _isEditableValue;
        private bool _isEditableName;

        public PropertyItemModel(object tag)
        {
            Tag = tag;
        }

        public int IndentLevel
        {
            get { return _indentLevel; }
            set
            {
                if (value != _indentLevel)
                {
                    _indentLevel = value;
                    RaisePropertyChanged();
                }
            }
        }

        public virtual string Name { get; set; }

        public object Tag { get; set; }

        public bool IsEditableValue
        {
            get { return _isEditableValue; }
            set
            {
                _isEditableValue = value;
                RaisePropertyChanged();
            }
        }

        public bool IsEditableName
        {
            get { return _isEditableName; }
            set
            {
                _isEditableName = value;
                RaisePropertyChanged();
            }
        }
    }

    public class EditablePropertyItemModel : PropertyItemModel
    {
        public EditablePropertyItemModel(object tag)
            : base(tag)
        {
        }

        public override string Name
        {
            get { return GetName != null ? GetName(Tag) : base.Name; }
            set
            {
                if (SetName != null)
                    SetName(Tag, value);
                base.Name = value;
            }
        }

        public Func<object, string> GetName;
        public Action<object, string> SetName;
    }



private void InsertDataTreeGridItem(ObservableCollection<DataTreeGridItem> items, int index, object customDataTreeGridItem)
        {
            var dataTreeGridItem = new DataTreeGridItem()
            {
                //Content = GetValue<object>(customDataTreeGridItem, ContentMember, null),
                Indentation = GetValue(customDataTreeGridItem, IndentationMember, 0),
                IsExpanded = GetValue(customDataTreeGridItem, IsExpandedMember, true),
                Tag = customDataTreeGridItem
            };

            ApplyBindings(dataTreeGridItem, customDataTreeGridItem);

            items.Insert(index, dataTreeGridItem);

            if (UpdateSourceOnTargetChanges)
            {
                dataTreeGridItem.PropertyChanged += Item_PropertyChanged;
            }
        }

        private DataTreeGridItem ApplyBindings(DataTreeGridItem dataTreeGridItem, object customDataTreeGridItem)
        {
            if (ContentConverter != null)
            {
                var binding = new Binding(ContentMember) { Source = customDataTreeGridItem, Mode = BindingMode.TwoWay, Converter = ContentConverter };
                dataTreeGridItem.SetBinding(DataTreeGridItem.ContentProperty, binding);
            }
            else
            {
                var binding = new Binding(ContentMember) { Source = customDataTreeGridItem, Mode = BindingMode.TwoWay };
                dataTreeGridItem.SetBinding(DataTreeGridItem.ContentProperty, binding);
            }

            if (IsCheckedMember != null)
            {
                var binding = new Binding(IsCheckedMember) { Source = customDataTreeGridItem, Mode = BindingMode.TwoWay, Converter = IsCheckedConverter };
                dataTreeGridItem.SetBinding(DataTreeGridItem.IsCheckedProperty, binding);
                dataTreeGridItem.CheckBoxVisibility = Visibility.Visible;
            }

            if (IsCheckBoxEnabledMember != null)
            {
                var binding = new Binding(IsCheckBoxEnabledMember) { Source = customDataTreeGridItem };
                dataTreeGridItem.SetBinding(DataTreeGridItem.IsCheckBoxEnabledProperty, binding);
            }

            if (IconSourceMember != null)
            {
                var binding = new Binding(IconSourceMember) { Source = customDataTreeGridItem, Converter = IconSourceConverter };
                dataTreeGridItem.SetBinding(DataTreeGridItem.IconSourceProperty, binding);
                dataTreeGridItem.IconVisibility = Visibility.Visible;
            }

            return dataTreeGridItem;
        }
public class TypeDefComponent
{
    public string Name { get; set; }
    public List<string> Tags { get; set; }

    // Конструктор копирования
    public TypeDefComponent(TypeDefComponent source)
    {
        this.Name = source.Name;
        this.Tags = new List<string>(source.Tags);
    }
}

var original = new TypeDefComponent { ... };
var copy = new TypeDefComponent(original); // Создаем копию





Вот упрощённая реализация без интерфейсов, с полным обходом проекта и сохранением функциональности:

### 1. Упрощённый ViewModel (`FindToolViewModel.cs`)

```csharp
using Catel.MVVM;
using System.Collections.ObjectModel;
using System.Linq;
using System.Threading.Tasks;
using SCLExpress.Core;
using System;
using System.Collections.Generic;

namespace Module.FindTool.Dialogs
{
    [View(typeof(FindToolView))]
    public class FindToolViewModel : ViewModelBase
    {
        private readonly Lazy<ISclObjectManager> _projectManager;
        
        public string SearchText { get; set; }
        public bool CaseSensitive { get; set; }
        public bool IsBusy { get; set; }
        public ObservableCollection<SearchResult> Results { get; private set; }
        public Command SearchCommand { get; private set; }

        [ImportingConstructor]
        public FindToolViewModel(Lazy<ISclObjectManager> projectManager)
        {
            _projectManager = projectManager;
            Results = new ObservableCollection<SearchResult>();
            SearchCommand = new Command(OnSearchExecute, () => !IsBusy && !string.IsNullOrWhiteSpace(SearchText));
        }

        private async void OnSearchExecute()
        {
            IsBusy = true;
            Results.Clear();

            try
            {
                await Task.Run(() =>
                {
                    var project = _projectManager.Value.ActiveProject;
                    if (project == null) return;

                    var comparison = CaseSensitive 
                        ? StringComparison.Ordinal 
                        : StringComparison.OrdinalIgnoreCase;

                    // Поиск в устройствах
                    foreach (var device in project.Devices)
                    {
                        CheckAndAddResult(device, "Device", $"Devices/{device.Name}", comparison);
                        
                        // Поиск в переменных устройства
                        foreach (var variable in device.Variables)
                        {
                            CheckAndAddResult(variable, "Variable", 
                                $"Devices/{device.Name}/Variables/{variable.Name}", comparison);
                        }
                        
                        // Поиск в конфигурациях
                        foreach (var config in device.Configurations)
                        {
                            CheckAndAddResult(config, "Config", 
                                $"Devices/{device.Name}/Configs/{config.Name}", comparison);
                            
                            foreach (var param in config.Parameters)
                            {
                                CheckAndAddResult(param, "Parameter", 
                                    $"Devices/{device.Name}/Configs/{config.Name}/Params/{param.Name}", comparison);
                            }
                        }
                    }
                    
                    // Поиск в соединениях
                    foreach (var connection in project.Connections)
                    {
                        CheckAndAddResult(connection, "Connection", 
                            $"Connections/{connection.SourceDevice}->{connection.TargetDevice}", comparison);
                    }
                });
            }
            finally
            {
                IsBusy = false;
            }
        }

        private void CheckAndAddResult(object obj, string type, string path, StringComparison comparison)
        {
            var name = obj.GetType().GetProperty("Name")?.GetValue(obj) as string;
            var description = obj.GetType().GetProperty("Description")?.GetValue(obj) as string;

            if ((name?.Contains(SearchText, comparison) == true || 
                (description?.Contains(SearchText, comparison) == true))
            {
                DispatcherService.Invoke(() =>
                {
                    Results.Add(new SearchResult
                    {
                        ObjectType = type,
                        Name = name,
                        Path = path,
                        Description = description,
                        OriginalObject = obj
                    });
                });
            }
        }
    }

    public class SearchResult
    {
        public string ObjectType { get; set; }
        public string Name { get; set; }
        public string Path { get; set; }
        public string Description { get; set; }
        public object OriginalObject { get; set; }
    }
}
```

### 2. XAML (`FindToolView.xaml`) - минимальные изменения
```xml
<catel:DataWindow x:Class="Module.FindTool.Dialogs.FindToolView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:catel="http://schemas.catelproject.com"
                  Width="800" Height="600">

    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- Search Panel -->
        <StackPanel Grid.Row="0" Orientation="Horizontal">
            <TextBox Text="{Binding SearchText}" Width="300" Margin="0,0,10,0"/>
            <Button Content="Search" Command="{Binding SearchCommand}" Width="100"/>
            <CheckBox Content="Case Sensitive" IsChecked="{Binding CaseSensitive}" 
                     VerticalAlignment="Center" Margin="10,0,0,0"/>
        </StackPanel>

        <!-- Results -->
        <DataGrid Grid.Row="1" ItemsSource="{Binding Results}" AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Type" Binding="{Binding ObjectType}" Width="100"/>
                <DataGridTextColumn Header="Name" Binding="{Binding Name}" Width="200"/>
                <DataGridTextColumn Header="Path" Binding="{Binding Path}" Width="300"/>
                <DataGridTextColumn Header="Description" Binding="{Binding Description}" Width="*"/>
            </DataGrid.Columns>
        </DataGrid>

        <!-- Status -->
        <TextBlock Grid.Row="2" Text="{Binding Results.Count, StringFormat='Found: {0} items'}"/>
    </Grid>
</catel:DataWindow>
```

### Ключевые особенности:

1. **Без интерфейсов**:
   - Используется рефлексия (`GetProperty`) для доступа к свойствам объектов
   - Прямая работа с типами из `SCLExpress.Core`

2. **Полный обход проекта**:
   - Устройства → Переменные → Конфигурации → Параметры
   - Соединения между устройствами

3. **Простая модель результата**:
   - Класс `SearchResult` содержит все необходимые данные

4. **Сохранённый функционал**:
   - Асинхронный поиск
   - Фильтр по регистру
   - Отображение пути к объекту

5. **Минимальные изменения XAML**:
   - Добавлена только колонка Path
   - Сохранён текущий стиль оформления

Для интеграции с выделением объектов в ProjectManager добавьте:

```csharp
// В MainWindowViewModel или другом месте
private void SelectFoundObject(SearchResult result)
{
    if (result.OriginalObject is Device device)
    {
        _projectManager.SelectedDevice = device;
    }
    else if (result.OriginalObject is DeviceVariable variable)
    {
        _projectManager.SelectedVariable = variable;
    }
    // Аналогично для других типов
}
```








Ошибка возникает из-за того, что свойство `SearchText` в вашей `FindToolViewModel` объявлено только с геттером (get), но в XAML вы пытаетесь сделать двустороннюю привязку (`TwoWay`) через `TextBox`.

### Исправление для ViewModel:

```csharp
public class FindToolViewModel : ViewModelBase
{
    private string _searchText;

    // Исправленное свойство с get и set
    public string SearchText
    {
        get { return _searchText; }
        set 
        {
            if (_searchText != value)
            {
                _searchText = value;
                RaisePropertyChanged(nameof(SearchText));
                
                // Дополнительно: можно добавить логику при изменении текста
                // Например, автоматический поиск
            }
        }
    }

    public FindToolViewModel()
    {
        Title = "Find Tool";
    }
}
```

### Альтернативные решения:

1. **Если нужно сделать свойство read-only** (менее предпочтительно для TextBox):
```csharp
// Только для чтения (не рекомендуется для TextBox)
public string SearchText { get; private set; }

// Тогда в XAML нужно явно указать OneWay binding:
<TextBox Text="{Binding SearchText, Mode=OneWay}" Width="200"/>
```

2. **Если используете Catel MVVM** (лучшее решение):
```csharp
public string SearchText
{
    get { return GetValue<string>(SearchTextProperty); }
    set { SetValue(SearchTextProperty, value); }
}

public static readonly PropertyData SearchTextProperty = 
    RegisterProperty(nameof(SearchText), typeof(string), null);
```

### Почему это важно:
- `TextBox` по умолчанию использует `TwoWay` binding
- При изменении текста в UI он пытается обновить свойство в VM
- Без сеттера это невозможно, отсюда и исключение

### Дополнительные проверки:
1. Убедитесь, что `DataContext` окна установлен на `FindToolViewModel`
2. Проверьте, что пространство имен в XAML правильно указано:
```xml
xmlns:local="clr-namespace:Module.IedExplorer.Dialogs"
```
3. Если используете Catel, убедитесь что атрибут `[View]` указан:
```csharp
[View(typeof(FindToolView))]
public class FindToolViewModel : ViewModelBase
```

После этих изменений привязка должна работать корректно, и окно поиска будет реагировать на ввод текста.

System.InvalidOperationException: 'Привязка типа TwoWay или OneWayToSource не может работать с доступным только для чтения свойством "SearchText" типа "Module.IedExplorer.FindToolViewModel".'


<catel:DataWindow x:Class="Module.IedExplorer.FindToolView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:catel="http://catel.codeplex.com" 
                  xmlns:sys="clr-namespace:System;assembly=mscorlib"
                  xmlns:properties="clr-namespace:Module.IedExplorer.Properties"
                  xmlns:dialogs="clr-namespace:Module.IedExplorer.Dialogs" 
                  xmlns:kivi="http://kivi.dev.ekra.ru"
                  Height="500" Width="700" MinHeight="300" MinWidth="400">
    

    <Grid>
        <Grid Margin="10">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="*"/>
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>

            <StackPanel Grid.Row="0" Orientation="Horizontal">
                <TextBlock Text="Search:" VerticalAlignment="Center" Margin="0,0,5,0"/>
                <TextBox Text="{Binding SearchText}" Width="200"/>
            </StackPanel>

            <Border Grid.Row="1" BorderBrush="LightGray" BorderThickness="1" Margin="0,10">
                <TextBlock Text="Write something" 
                      HorizontalAlignment="Center" VerticalAlignment="Center"
                      Foreground="Gray"/>
            </Border>

            <StackPanel Grid.Row="2" Orientation="Horizontal" HorizontalAlignment="Right">
                <Button Content="Find" Width="80" Margin="0,0,5,0"/>
                <Button Content="Close" Width="80" Command="{Binding CancelCommand}"/>
            </StackPanel>
        </Grid>
    </Grid>
</catel:DataWindow>


using Catel.MVVM;

namespace Module.IedExplorer
{
    [View(typeof(FindToolView))]
    public class FindToolViewModel : ViewModelBase
    {
        private string _searchText;

        public string SearchText
        {
            get
            {
                return _searchText;
            }
            set
            {
                if (_searchText == value) 
                    return;
                _searchText = value;
            }
        }

        public FindToolViewModel()
        {
            Title = "Find Tool";
        }
    }
}
