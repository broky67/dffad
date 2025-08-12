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
