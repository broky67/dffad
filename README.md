Вы абсолютно правы! Отдельный SearchCategory класс избыточен для этой задачи. Вот упрощенная реализация без него:

Упрощенная ViewModel

```csharp
[View(typeof(FindToolView))]
public class FindToolViewModel : ViewModelBase
{
    private readonly SCLE_Project _project;
    private string _searchText;
    private bool _isSearching;
    private bool _matchCase;

    // Простые коллекции строк вместо сложных объектов
    public ObservableCollection<string> AvailableCategories { get; } = new ObservableCollection<string>();
    public ObservableCollection<string> SelectedCategories { get; } = new ObservableCollection<string>();

    public ObservableCollection<SearchResult> Results { get; } = new ObservableCollection<SearchResult>();
    public ICommand SearchCommand { get; }

    public string SearchText
    {
        get => _searchText;
        set { _searchText = value; RaisePropertyChanged(nameof(SearchText)); }
    }

    public bool MatchCase
    {
        get => _matchCase;
        set { _matchCase = value; RaisePropertyChanged(nameof(MatchCase)); }
    }

    public FindToolViewModel(SCLE_Project project)
    {
        _project = project ?? throw new ArgumentNullException(nameof(project));
        
        InitializeCategories();
        SearchCommand = new RelayCommand(ExecuteSearch, CanExecuteSearch);
    }

    private void InitializeCategories()
    {
        // Просто добавляем строки с названиями категорий
        AvailableCategories.Add("Библиотеки");
        AvailableCategories.Add("Типы данных");
        AvailableCategories.Add("POU");
        AvailableCategories.Add("F-узлы");
        AvailableCategories.Add("Подсети");
        AvailableCategories.Add("Подстанции");
        AvailableCategories.Add("IED устройства");

        // По умолчанию выбираем все категории
        foreach (var category in AvailableCategories)
        {
            SelectedCategories.Add(category);
        }
    }

    private bool CanExecuteSearch()
    {
        return !_isSearching && 
               !string.IsNullOrWhiteSpace(SearchText) &&
               SelectedCategories.Count > 0;
    }

    private async void ExecuteSearch()
    {
        _isSearching = true;
        Results.Clear();

        try
        {
            await Task.Run(PerformSearch);
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Ошибка при выполнении поиска: {ex.Message}", 
                          "Ошибка", 
                          MessageBoxButton.OK, 
                          MessageBoxImage.Error);
        }
        finally
        {
            _isSearching = false;
        }
    }

    private void PerformSearch()
    {
        var comparison = MatchCase 
            ? StringComparison.Ordinal 
            : StringComparison.OrdinalIgnoreCase;

        var searchText = SearchText;

        // Простая проверка по строковым названиям категорий
        foreach (var category in SelectedCategories)
        {
            switch (category)
            {
                case "Библиотеки" when _project.LibraryItems != null:
                    SearchInCollection(_project.LibraryItems, "Библиотеки", searchText, comparison);
                    break;
                case "Типы данных" when _project.DataTypeItems != null:
                    SearchInCollection(_project.DataTypeItems, "Типы данных", searchText, comparison);
                    break;
                case "POU" when _project.PouItems != null:
                    SearchInCollection(_project.PouItems, "POU", searchText, comparison);
                    break;
                case "F-узлы" when _project.FNodeTypes != null:
                    SearchInCollection(_project.FNodeTypes, "F-узлы", searchText, comparison);
                    break;
                case "Подсети" when _project.SubnetworkItems != null:
                    SearchInCollection(_project.SubnetworkItems, "Подсети", searchText, comparison);
                    break;
                case "Подстанции" when _project.SubstationItems != null:
                    SearchInCollection(_project.SubstationItems, "Подстанции", searchText, comparison);
                    break;
                case "IED устройства" when _project.IedItems != null:
                    SearchInCollection(_project.IedItems, "IED устройства", searchText, comparison);
                    break;
            }
        }
    }

    private void SearchInCollection(IEnumerable collection, string category, string searchText, StringComparison comparison)
    {
        foreach (var item in collection)
        {
            if (item == null) continue;

            var nameProperty = item.GetType().GetProperty("Name");
            if (nameProperty == null) continue;

            var name = nameProperty.GetValue(item) as string;
            if (name == null) continue;

            if (name.Contains(searchText, comparison))
            {
                Application.Current.Dispatcher.Invoke(() => 
                    Results.Add(new SearchResult
                    {
                        Name = name,
                        Category = category,
                        SourceObject = item
                    }));
            }
        }
    }
}

// Максимально простая модель результата
public class SearchResult
{
    public string Name { get; set; }
    public string Category { get; set; }
    public object SourceObject { get; set; }
}
```

Упрощенный XAML

```xml
<catel:DataWindow x:Class="Module.IedExplorer.Dialogs.FindToolView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:catel="http://catel.codeplex.com"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:xctk="http://schemas.xceed.com/wpf/xaml/toolkit"
                  Height="500" Width="800">

    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Панель поиска -->
        <StackPanel Grid.Row="0" Orientation="Horizontal">
            <TextBox Text="{Binding SearchText}" Width="250" Margin="0,0,10,0"/>
            <Button Content="Поиск" Command="{Binding SearchCommand}" Width="80"/>
            <CheckBox Content="Учитывать регистр" IsChecked="{Binding MatchCase}" Margin="10,0,0,0"/>
        </StackPanel>

        <!-- CheckComboBox с простыми строками -->
        <StackPanel Grid.Row="1" Margin="0,10">
            <TextBlock Text="Категории для поиска:" Margin="0,0,0,5"/>
            <xctk:CheckComboBox 
                ItemsSource="{Binding AvailableCategories}"
                SelectedItemsOverride="{Binding SelectedCategories}"
                Width="300"/>
        </StackPanel>

        <!-- Статус -->
        <TextBlock Grid.Row="2" 
                   Text="{Binding Results.Count, StringFormat='Найдено: {0}'}"
                   Margin="0,5" Foreground="Gray"/>

        <!-- Результаты -->
        <DataGrid Grid.Row="3" ItemsSource="{Binding Results}" AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Название" Binding="{Binding Name}" Width="*"/>
                <DataGridTextColumn Header="Категория" Binding="{Binding Category}" Width="150"/>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</catel:DataWindow>
```

Преимущества упрощенного подхода:

1. Меньше кода - нет лишних классов
2. Проще отладка - только строки вместо сложных объектов
3. Лучшая производительность - меньше накладных расходов
4. Легче поддерживать - понятная структура
5. Меньше зависимостей - не нужны интерфейсы INotifyPropertyChanged

Если нужна обратная связь при изменении выбора:

```csharp
// Добавить в ViewModel
private void OnSelectedCategoriesChanged(object sender, NotifyCollectionChangedEventArgs e)
{
    if (CanExecuteSearch())
    {
        ExecuteSearch();
    }
}

// В конструкторе после инициализации:
SelectedCategories.CollectionChanged += OnSelectedCategoriesChanged;
```

Такой подход сохраняет всю функциональность, но делает код значительно проще и понятнее.


Вот как разбить результаты на две колонки с отображением категории:

1. Обновляем ViewModel - возвращаем SearchResult

```csharp
public class SearchResult
{
    public string Name { get; set; }
    public string Category { get; set; }
    public object SourceObject { get; set; }
}

// В методе SearchInCollection заменяем:
Application.Current.Dispatcher.Invoke(() => 
    Results.Add(new SearchResult
    {
        Name = name,
        Category = category.DisplayName,
        SourceObject = item
    }));
```

2. XAML с DataGrid (рекомендуется)

```xml
<DataGrid Grid.Row="3" 
          ItemsSource="{Binding Results}"
          AutoGenerateColumns="False"
          IsReadOnly="True"
          HeadersVisibility="Column"
          SelectionMode="Single">
    
    <DataGrid.Columns>
        <DataGridTextColumn Header="Название" 
                           Binding="{Binding Name}" 
                           Width="*"
                           ElementStyle="{StaticResource WrapTextStyle}"/>
        
        <DataGridTextColumn Header="Категория" 
                           Binding="{Binding Category}" 
                           Width="120"
                           CanUserSort="True"/>
    </DataGrid.Columns>
    
    <DataGrid.Style>
        <Style TargetType="{x:Type DataGrid}">
            <Setter Property="AlternationCount" Value="2"/>
            <Setter Property="AlternatingRowBackground" Value="#F5F5F5"/>
        </Style>
    </DataGrid.Style>
</DataGrid>
```

3. Альтернатива с ListView

```xml
<ListView Grid.Row="3" ItemsSource="{Binding Results}">
    <ListView.View>
        <GridView>
            <GridViewColumn Header="Название" DisplayMemberBinding="{Binding Name}" Width="300"/>
            <GridViewColumn Header="Категория" DisplayMemberBinding="{Binding Category}" Width="120"/>
        </GridView>
    </ListView.View>
    
    <ListView.ItemContainerStyle>
        <Style TargetType="{x:Type ListViewItem}">
            <Setter Property="HorizontalContentAlignment" Value="Stretch"/>
        </Style>
    </ListView.ItemContainerStyle>
</ListView>
```

4. Простой вариант с Grid

```xml
<ItemsControl Grid.Row="3" ItemsSource="{Binding Results}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Grid Margin="0,2">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="120"/>
                </Grid.ColumnDefinitions>
                
                <TextBlock Grid.Column="0" Text="{Binding Name}" TextWrapping="Wrap"/>
                <TextBlock Grid.Column="1" Text="{Binding Category}" 
                          Margin="10,0,0,0" Foreground="Gray"/>
            </Grid>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

5. Полный XAML с DataGrid (лучший вариант)

```xml
<catel:DataWindow x:Class="Module.IedExplorer.Dialogs.FindToolView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:catel="http://catel.codeplex.com"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:xctk="http://schemas.xceed.com/wpf/xaml/toolkit"
                  Height="500" Width="800" MinHeight="300" MinWidth="500">

    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Панель поиска -->
        <StackPanel Grid.Row="0" Orientation="Horizontal">
            <TextBox Text="{Binding SearchText}" 
                     Width="250"
                     Margin="0,0,10,0"/>
            <Button Content="Поиск" 
                    Command="{Binding SearchCommand}"
                    Width="80"/>
            <CheckBox Content="Учитывать регистр" 
                      IsChecked="{Binding MatchCase}"
                      Margin="10,0,0,0"
                      VerticalAlignment="Center"/>
        </StackPanel>

        <!-- CheckComboBox для выбора категорий -->
        <StackPanel Grid.Row="1" Margin="0,10">
            <TextBlock Text="Категории для поиска:" Margin="0,0,0,5"/>
            <xctk:CheckComboBox 
                ItemsSource="{Binding AvailableCategories}"
                DisplayMemberPath="DisplayName"
                SelectedItemsOverride="{Binding FilteredCategories}"
                Width="300"
                HorizontalAlignment="Left"/>
        </StackPanel>

        <!-- Статус -->
        <TextBlock Grid.Row="2" 
                   Text="{Binding Results.Count, StringFormat='Найдено элементов: {0}'}"
                   Margin="0,5" 
                   Foreground="Gray"
                   FontStyle="Italic"/>

        <!-- Результаты в две колонки -->
        <DataGrid Grid.Row="3" 
                  ItemsSource="{Binding Results}"
                  AutoGenerateColumns="False"
                  IsReadOnly="True"
                  HeadersVisibility="Column"
                  SelectionMode="Single"
                  VerticalScrollBarVisibility="Auto"
                  HorizontalScrollBarVisibility="Disabled">
            
            <DataGrid.Columns>
                <DataGridTextColumn Header="Название объекта" 
                                   Binding="{Binding Name}" 
                                   Width="*"
                                   MinWidth="200"/>
                
                <DataGridTextColumn Header="Категория" 
                                   Binding="{Binding Category}" 
                                   Width="150"
                                   CanUserSort="True"/>
            </DataGrid.Columns>
            
            <!-- Стиль для чередования строк -->
            <DataGrid.RowStyle>
                <Style TargetType="{x:Type DataGridRow}">
                    <Style.Triggers>
                        <Trigger Property="ItemsControl.AlternationIndex" Value="0">
                            <Setter Property="Background" Value="White"/>
                        </Trigger>
                        <Trigger Property="ItemsControl.AlternationIndex" Value="1">
                            <Setter Property="Background" Value="#FAFAFA"/>
                        </Trigger>
                    </Style.Triggers>
                </Style>
            </DataGrid.RowStyle>
        </DataGrid>
    </Grid>
</catel:DataWindow>
```

6. Дополнительные улучшения (опционально)

Добавьте в ресурсы для переноса текста:

```xml
<Window.Resources>
    <Style x:Key="WrapTextStyle" TargetType="{x:Type TextBlock}">
        <Setter Property="TextWrapping" Value="Wrap"/>
        <Setter Property="VerticalAlignment" Value="Center"/>
    </Style>
</Window.Resources>
```

И используйте в колонке:

```xml
<DataGridTextColumn Header="Название" Binding="{Binding Name}" Width="*">
    <DataGridTextColumn.ElementStyle>
        <Style TargetType="{x:Type TextBlock}">
            <Setter Property="TextWrapping" Value="Wrap"/>
        </Style>
    </DataGridTextColumn.ElementStyle>
</DataGridTextColumn>
```

Преимущества DataGrid:

· Сортировка по клику на заголовок
· Автоматическое выравнивание
· Чередование строк для лучшей читаемости
· Гибкая настройка колонок
· Встроенная прокрутка

Теперь результаты будут отображаться в двух колонках: название объекта и его категория.


Отлично! Сделаю похожим образом с использованием CheckComboBox и паттерна из вашего другого проекта. Вот полная реализация:

ViewModel

```csharp
[View(typeof(FindToolView))]
public class FindToolViewModel : ViewModelBase
{
    private readonly SCLE_Project _project;
    private string _searchText;
    private bool _isSearching;
    private bool _matchCase;

    private ObservableCollection<SearchCategory> _availableCategories;
    private ObservableCollection<SearchCategory> _filteredCategories;
    private Dictionary<SearchCategory, List<object>> _categoryItemsDict = new Dictionary<SearchCategory, List<object>>();

    public ObservableCollection<string> Results { get; } = new ObservableCollection<string>();
    public ICommand SearchCommand { get; }

    [MementoIgnore]
    public ObservableCollection<SearchCategory> AvailableCategories
    {
        get { return _availableCategories; }
        set
        {
            _availableCategories = value;
            RaisePropertyChanged(nameof(AvailableCategories));
            
            // Автоматически выбираем все категории при инициализации
            FilteredCategories = new ObservableCollection<SearchCategory>(_availableCategories);
        }
    }

    public ObservableCollection<SearchCategory> FilteredCategories
    {
        get { return _filteredCategories; }
        set
        {
            if (_filteredCategories != value)
            {
                if (_filteredCategories != null)
                    _filteredCategories.CollectionChanged -= OnFilteredCategoriesChanged;

                _filteredCategories = value;

                if (_filteredCategories != null)
                    _filteredCategories.CollectionChanged += OnFilteredCategoriesChanged;

                RaisePropertyChanged(nameof(FilteredCategories));
            }
        }
    }

    public string SearchText
    {
        get => _searchText;
        set { _searchText = value; RaisePropertyChanged(nameof(SearchText)); }
    }

    public bool MatchCase
    {
        get => _matchCase;
        set { _matchCase = value; RaisePropertyChanged(nameof(MatchCase)); }
    }

    public FindToolViewModel(SCLE_Project project)
    {
        _project = project ?? throw new ArgumentNullException(nameof(project));
        
        InitializeCategories();
        SearchCommand = new RelayCommand(ExecuteSearch, CanExecuteSearch);
    }

    private void InitializeCategories()
    {
        AvailableCategories = new ObservableCollection<SearchCategory>
        {
            new SearchCategory { Key = "Libraries", DisplayName = "Библиотеки", SourceCollection = _project.LibraryItems },
            new SearchCategory { Key = "DataTypes", DisplayName = "Типы данных", SourceCollection = _project.DataTypeItems },
            new SearchCategory { Key = "Pous", DisplayName = "POU", SourceCollection = _project.PouItems },
            new SearchCategory { Key = "Fnodes", DisplayName = "F-узлы", SourceCollection = _project.FNodeTypes },
            new SearchCategory { Key = "Subnetworks", DisplayName = "Подсети", SourceCollection = _project.SubnetworkItems },
            new SearchCategory { Key = "Substations", DisplayName = "Подстанции", SourceCollection = _project.SubstationItems },
            new SearchCategory { Key = "Ieds", DisplayName = "IED устройства", SourceCollection = _project.IedItems }
        };

        // Инициализируем словарь для быстрого доступа
        foreach (var category in AvailableCategories)
        {
            _categoryItemsDict[category] = category.SourceCollection?.Cast<object>().ToList() ?? new List<object>();
        }
    }

    private void OnFilteredCategoriesChanged(object sender, NotifyCollectionChangedEventArgs e)
    {
        // Автоматически запускаем поиск при изменении выбранных категорий
        if (CanExecuteSearch())
        {
            ExecuteSearch();
        }
    }

    private bool CanExecuteSearch()
    {
        return !_isSearching && 
               !string.IsNullOrWhiteSpace(SearchText) &&
               FilteredCategories?.Count > 0;
    }

    private async void ExecuteSearch()
    {
        _isSearching = true;
        Results.Clear();

        try
        {
            await Task.Run(PerformSearch);
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Ошибка при выполнении поиска: {ex.Message}", 
                          "Ошибка", 
                          MessageBoxButton.OK, 
                          MessageBoxImage.Error);
        }
        finally
        {
            _isSearching = false;
        }
    }

    private void PerformSearch()
    {
        var comparison = MatchCase 
            ? StringComparison.Ordinal 
            : StringComparison.OrdinalIgnoreCase;

        var searchText = SearchText;

        foreach (var category in FilteredCategories)
        {
            if (_categoryItemsDict.TryGetValue(category, out var items))
            {
                SearchInCollection(items, searchText, comparison);
            }
        }
    }

    private void SearchInCollection(List<object> collection, string searchText, StringComparison comparison)
    {
        if (collection == null) return;

        foreach (var item in collection)
        {
            if (item == null) continue;

            var nameProperty = item.GetType().GetProperty("Name");
            if (nameProperty == null) continue;

            var name = nameProperty.GetValue(item) as string;
            if (name == null) continue;

            if (name.Contains(searchText, comparison))
            {
                Application.Current.Dispatcher.Invoke(() => Results.Add(name));
            }
        }
    }
}

// Модель категории поиска
public class SearchCategory : INotifyPropertyChanged
{
    public string Key { get; set; }
    public string DisplayName { get; set; }
    public IEnumerable SourceCollection { get; set; }

    public event PropertyChangedEventHandler PropertyChanged;
    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

XAML с CheckComboBox

```xml
<catel:DataWindow x:Class="Module.IedExplorer.Dialogs.FindToolView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:catel="http://catel.codeplex.com"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:xctk="http://schemas.xceed.com/wpf/xaml/toolkit"
                  Height="500" Width="700" MinHeight="300" MinWidth="400">

    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Панель поиска -->
        <StackPanel Grid.Row="0" Orientation="Horizontal">
            <TextBox Text="{Binding SearchText}" 
                     Width="200"
                     Margin="0,0,10,0"/>
            <Button Content="Поиск" 
                    Command="{Binding SearchCommand}"
                    Width="80"/>
            <CheckBox Content="Учитывать регистр" 
                      IsChecked="{Binding MatchCase}"
                      Margin="10,0,0,0"
                      VerticalAlignment="Center"/>
        </StackPanel>

        <!-- CheckComboBox для выбора категорий -->
        <StackPanel Grid.Row="1" Margin="0,10">
            <TextBlock Text="Категории для поиска:" Margin="0,0,0,5"/>
            <xctk:CheckComboBox 
                ItemsSource="{Binding AvailableCategories}"
                DisplayMemberPath="DisplayName"
                SelectedItemsOverride="{Binding FilteredCategories}"
                Width="300"
                HorizontalAlignment="Left"/>
        </StackPanel>

        <!-- Статус -->
        <TextBlock Grid.Row="2" Text="{Binding FilteredCategories.Count, StringFormat='Выбрано категорий: {0}'}"
                   Margin="0,5" Foreground="Gray"/>

        <!-- Результаты -->
        <ListBox Grid.Row="3" 
                 ItemsSource="{Binding Results}" 
                 HorizontalContentAlignment="Stretch"/>
    </Grid>
</catel:DataWindow>
```

Ключевые особенности реализации:

1. Паттерн из вашего проекта:
   · AvailableCategories - все доступные категории
   · FilteredCategories - выбранные категории
   · CollectionChanged событие для автоматического обновления
2. Словарь для быстрого доступа:
   · _categoryItemsDict хранит связь категория → коллекция элементов
   · Избегаем повторного приведения типов
3. Автоматический поиск:
   · При изменении выбранных категорий поиск запускается автоматически
   · Не нужно нажимать кнопку "Поиск" после изменения фильтров
4. Совместимость с вашим кодом:
   · Использует те же паттерны (MementoIgnore, CollectionChanged)
   · Сохраняет архитектурный стиль
5. Производительность:
   · Предварительное кэширование коллекций
   · Минимальное использование reflection во время поиска

Эта реализация максимально близка к паттерну из вашего другого проекта и обеспечивает удобный интерфейс для фильтрации категорий поиска.






private ObservableCollection<IedFilterItem> _publisherList;
        private List<IedFilterItem> _selectedPublisherList;
        private ObservableCollection<IedFilterItem> _filteredPublishers;
        private Dictionary<IedFilterItem, List<IedFilterItem>> _publisherDic = new Dictionary<IedFilterItem, List<IedFilterItem>>();

        [MementoIgnore]
        public ObservableCollection<IedFilterItem> PublisherList
        {
            get { return _publisherList; }
            set
            {
                _publisherList = value;
                RaisePropertyChanged(nameof(PublisherList));

                //_eventAggregator.GetEvent<IedFilterItemsChangedEvent>()
                //    .Publish(new IedFilterItemsChangedEventArgs(this, _publisherList, _subscriberList));

                SelectedPublisherList = (!_filteredPublishers.Any())
                    ? _publisherList.ToList()
                    : _filteredPublishers.ToList();
            }
        }

        [Browsable(false)]
        public tSubNetwork Subnetwork
        {
            get { return _subNetwork; }
            protected set { _subNetwork = value; }
        }

        //[MementoIgnore]
        public ObservableCollection<IedFilterItem> FilteredPublishers
        {
            get { return _filteredPublishers; }
            set
            {
                if (_filteredPublishers != value)
                {
                    if (_filteredPublishers != null)
                        _filteredPublishers.CollectionChanged -= OnFilteredPublishersSubcribersChanged;

                    _filteredPublishers = value;

                    if (_filteredPublishers != null)
                        _filteredPublishers.CollectionChanged += OnFilteredPublishersSubcribersChanged;
                }
            }
        }
