Способ 2: Загрузка из JSON/XML (если типы хранятся в конфигах)

Файл DeviceTypes.json:
json

{
  "DeviceTypes": [
    {
      "TypeName": "T_R17xx_Delay",
      "Properties": {
        "ActivationDelay": 100,
        "DeactivationDelay": 50
      }
    }
  ]
}

Код загрузки:
csharp

public class DeviceTypeLoader
{
    public static List<DeviceDescriptionType> LoadFromFile(string path)
    {
        var json = File.ReadAllText(path);
        return JsonConvert.DeserializeObject<DeviceTypesConfig>(json).DeviceTypes;
    }
}

public class DeviceTypesConfig
{
    public List<DeviceDescriptionType> DeviceTypes { get; set; }
}

Использование:
csharp

var types = DeviceTypeLoader.LoadFromFile("DeviceTypes.json");
var delaySettings = types.FirstOrDefault(t => t.TypeName == "T_R17xx_Delay");

2. Связывание свойств через ValueConverter
Пример конвертера для сложных свойств
csharp

[ValueConversion(typeof(Dictionary<string, object>), typeof(string))]
public class DevicePropertiesConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is not Dictionary<string, object> props)
            return string.Empty;

        // Форматируем свойства для отображения (например, в ToolTip)
        return string.Join(", ", props.Select(p => $"{p.Key}: {p.Value}"));
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}

Регистрация в XAML:
xml

<UserControl.Resources>
    <local:DevicePropertiesConverter x:Key="DevicePropsConverter"/>
</UserControl.Resources>

Использование в привязке:
xml

<!-- Отображение сводки свойств -->
<TextBlock 
    Text="{Binding TypeSettings.Properties, Converter={StaticResource DevicePropsConverter}}"
    ToolTip="{Binding TypeSettings.Properties, Converter={StaticResource DevicePropsConverter}}"/>

3. Полная привязка в TreeView
xml

<TreeView ItemsSource="{Binding DeviceDescriptions}">
    <TreeView.ItemTemplate>
        <HierarchicalDataTemplate ItemsSource="{Binding Parameters}">
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="{Binding Name}" FontWeight="Bold"/>
                <TextBlock Text="{Binding TypeSettings.Properties, Converter={StaticResource DevicePropsConverter}}"
                           Margin="10,0" Foreground="Gray"/>
            </StackPanel>

            <HierarchicalDataTemplate.ItemTemplate>
                <DataTemplate>
                    <StackPanel>
                        <!-- Основное значение параметра -->
                        <TextBox Text="{Binding Value}" Margin="5,0"/>

                        <!-- Динамические свойства типа -->
                        <ItemsControl ItemsSource="{Binding TypeSettings.Properties}">
                            <ItemsControl.ItemTemplate>
                                <DataTemplate>
                                    <Grid Margin="10,0">
                                        <Grid.ColumnDefinitions>
                                            <ColumnDefinition Width="150"/>
                                            <ColumnDefinition Width="100"/>
                                        </Grid.ColumnDefinitions>
                                        
                                        <TextBlock Text="{Binding Key}" Grid.Column="0"/>
                                        <TextBox Text="{Binding Value}" Grid.Column="1"/>
                                    </Grid>
                                </DataTemplate>
                            </ItemsControl.ItemTemplate>
                        </ItemsControl>
                    </StackPanel>
                </DataTemplate>
            </HierarchicalDataTemplate.ItemTemplate>
        </HierarchicalDataTemplate>
    </TreeView.ItemTemplate>
</TreeView>

4. Динамическое обновление свойств

Чтобы изменения в Properties сразу применялись:

    Реализуйте INotifyPropertyChanged в DeviceDescriptionType:

csharp

public class DeviceDescriptionType : INotifyPropertyChanged
{
    private Dictionary<string, object> _properties;
    public Dictionary<string, object> Properties
    {
        get => _properties;
        set { _properties = value; OnPropertyChanged(); }
    }

    public event PropertyChangedEventHandler PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string name = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
