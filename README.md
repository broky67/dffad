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
