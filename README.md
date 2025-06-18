Для добавления дополнительного раскрывающегося списка с параметрами (например, для `T_R17xx_Delay` - время срабатывания и возврата) в ваш `DataTreeGrid`, предлагаю следующее решение:

### 1. Добавление нового DataTemplate для параметров задержки
В ресурсы UserControl добавьте новые шаблоны:

```xml
<UserControl.Resources>
    <!-- ... существующие ресурсы ... -->

    <!-- Шаблон для отображения параметров задержки -->
    <DataTemplate x:Key="DelayParametersTemplate">
        <StackPanel Orientation="Horizontal">
            <TextBlock Text="Активация:" Margin="5,0"/>
            <TextBlock Text="{Binding Tag.Tag.ActivationDelay}" Width="50"/>
            <TextBlock Text="мс" Margin="5,0"/>
            <TextBlock Text="Возврат:" Margin="5,0"/>
            <TextBlock Text="{Binding Tag.Tag.DeactivationDelay}" Width="50"/>
            <TextBlock Text="мс"/>
        </StackPanel>
    </DataTemplate>

    <!-- Шаблон для редактирования параметров задержки -->
    <DataTemplate x:Key="DelayParametersEditTemplate">
        <StackPanel Orientation="Horizontal">
            <TextBlock Text="Активация:" Margin="5,0"/>
            <TextBox Text="{Binding Tag.Tag.ActivationDelay}" Width="50"/>
            <TextBlock Text="мс" Margin="5,0"/>
            <TextBlock Text="Возврат:" Margin="5,0"/>
            <TextBox Text="{Binding Tag.Tag.DeactivationDelay}" Width="50"/>
            <TextBlock Text="мс"/>
        </StackPanel>
    </DataTemplate>
</UserControl.Resources>
```

### 2. Добавление новой колонки в DataTreeGrid
Вставьте новую колонку в список колонок:

```xml
<kivi:DataTreeGrid.Columns>
    <!-- ... существующие колонки ... -->
    
    <DataGridTemplateColumn mvvm:DTGAP.Key="DelayParams" Header="Параметры задержки" Width="250"
        CellTemplate="{StaticResource DelayParametersTemplate}" 
        CellEditingTemplate="{StaticResource DelayParametersEditTemplate}"/>
</kivi:DataTreeGrid.Columns>
```

### 3. Обновление контекстного меню
Добавьте пункт для управления видимостью новой колонки:

```xml
<ContextMenu Tag="{Binding PlacementTarget, RelativeSource={RelativeSource Self}, Converter={StaticResource dtgac}}">
    <!-- ... существующие пункты меню ... -->
    <MenuItem IsCheckable="True" Header="Параметры задержки"
        IsChecked="{Binding Tag.Columns[8].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
</ContextMenu>
```

### 4. Модификация модели данных
Добавьте соответствующие свойства в класс параметра:

```csharp
public class Parameter
{
    // ... существующие свойства ...
    
    public int ActivationDelay { get; set; }  // Выдержка на срабатывание (мс)
    public int DeactivationDelay { get; set; } // Выдержка на возврат (мс)
}
```

### 5. Условное отображение (опционально)
Если нужно показывать эту колонку только для определенных типов, можно использовать конвертер:

```xml
<DataGridTemplateColumn.Visibility>
    <Binding Path="Tag.Tag.type" Converter="{StaticResource DelayParamsVisibilityConverter}"/>
</DataGridTemplateColumn.Visibility>
```

С самим конвертером:

```csharp
public class DelayParamsVisibilityConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return value?.ToString() == "T_R17xx_Delay" ? Visibility.Visible : Visibility.Collapsed;
    }
    
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}
```

### Альтернативный подход: Использование DataTemplateSelector
Если параметры разных типов сильно отличаются, можно создать селектор шаблонов:

```csharp
public class ParameterTemplateSelector : DataTemplateSelector
{
    public DataTemplate DelayTemplate { get; set; }
    public DataTemplate DefaultTemplate { get; set; }

    public override DataTemplate SelectTemplate(object item, DependencyObject container)
    {
        var param = (item as YourItemType)?.Tag?.Tag;
        if (param?.type == "T_R17xx_Delay")
            return DelayTemplate;
        
        return DefaultTemplate;
    }
}
```

И использовать его в XAML:

```xml
<DataGridTemplateColumn 
    CellTemplateSelector="{StaticResource ParameterTemplateSelector}"
    CellEditingTemplateSelector="{StaticResource ParameterTemplateSelector}"/>
```

Это решение позволит:
1. Добавить отдельную колонку для параметров задержки
2. Редактировать значения прямо в таблице
3. Гибко управлять видимостью колонки
4. Легко расширять для других типов параметров









      <ParameterSet>
        <ParameterSection>
          <Name name="local:SpiParameters">Параметры связи по SPI</Name>
          <Parameter ParameterId="0" type="std:INT">
            <Default>0</Default>
            <Name name="local:Id0">Адрес блока на шине</Name>
          </Parameter>
          <Parameter ParameterId="5" type="std:INT">
            <Default>0</Default>
            <Name name="local:Id5">версия протокола связи по spi ( 0 или 1 )</Name>
          </Parameter>
          <Parameter ParameterId="9" type="std:INT">
            <Default>0</Default>
            <Name name="local:Id9">Тип протокола связи по spi ( 8bit или 16bit )</Name>
          </Parameter>
        </ParameterSection>
      </ParameterSet>
    </Connector>
