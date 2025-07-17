Чтобы вывести компоненты (`Component`) из `Tag` в `DataTreeGrid` в WPF, вам нужно правильно настроить привязки и шаблоны. Вот как это можно сделать:

### 1. **Добавьте конвертер для извлечения компонентов из `Tag`**
Если `Tag` содержит объект типа `Component`, создайте конвертер, который будет извлекать нужные данные:

```csharp
public class ComponentConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is Component component)
        {
            // Возвращаем коллекцию компонентов или конкретное свойство
            return component.Items; // Например, если Component содержит Items
        }
        return null;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}
```

### 2. **Добавьте конвертер в ресурсы XAML**
```xml
<UserControl.Resources>
    <local:ComponentConverter x:Key="ComponentConverter"/>
</UserControl.Resources>
```

### 3. **Настройте `DataTreeGrid` для отображения компонентов**
Измените привязку `ItemsSource` в `DataTreeGrid`, чтобы она использовала `Tag` и конвертер:

```xml
<kivi:DataTreeGrid ItemsSource="{Binding Tag, Converter={StaticResource ComponentConverter}}">
    <!-- Колонки -->
    <kivi:DataTreeGrid.Columns>
        <DataGridTextColumn Header="Name" Binding="{Binding Name}"/>
        <DataGridTextColumn Header="Type" Binding="{Binding Type}"/>
        <!-- Другие колонки -->
    </kivi:DataTreeGrid.Columns>
</kivi:DataTreeGrid>
```

### 4. **Если `Component` содержит вложенные элементы**
Если `Component` имеет иерархическую структуру (например, родительские и дочерние элементы), используйте `HierarchicalDataTemplate`:

```xml
<TreeView ItemsSource="{Binding Tag, Converter={StaticResource ComponentConverter}}">
    <TreeView.ItemTemplate>
        <HierarchicalDataTemplate ItemsSource="{Binding Children}"> <!-- Если есть вложенность -->
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="{Binding Name}"/>
                <TextBlock Text="{Binding Type}" Margin="10,0,0,0"/>
            </StackPanel>
        </HierarchicalDataTemplate>
    </TreeView.ItemTemplate>
</TreeView>
```

### 5. **Пример для `DataTreeGrid` с вложенностью**
Если используется `DataTreeGrid` (аналог `TreeView`), настройте его так:

```xml
<kivi:DataTreeGrid ItemsSource="{Binding Tag, Converter={StaticResource ComponentConverter}}">
    <kivi:DataTreeGrid.Columns>
        <kivi:DataTreeGridColumn Header="Name" Binding="{Binding Name}"/>
        <kivi:DataTreeGridColumn Header="Type" Binding="{Binding Type}"/>
    </kivi:DataTreeGrid.Columns>
    <kivi:DataTreeGrid.ChildrenBinding>
        <Binding Path="Children"/> <!-- Если есть вложенность -->
    </kivi:DataTreeGrid.ChildrenBinding>
</kivi:DataTreeGrid>
```

### 6. **Если `Tag` — это `ParameterType`**
Если `Tag` — это `ParameterType`, и нужно отобразить его свойства:

```xml
<kivi:DataTreeGrid ItemsSource="{Binding Tag}">
    <kivi:DataTreeGrid.Columns>
        <DataGridTextColumn Header="Name" Binding="{Binding Name}"/>
        <DataGridTextColumn Header="Type" Binding="{Binding Type}"/>
        <DataGridTextColumn Header="Value" Binding="{Binding Default}"/>
    </kivi:DataTreeGrid.Columns>
</kivi:DataTreeGrid>
```

### Итог
- Используйте **конвертеры** для извлечения данных из `Tag`.
- Настройте **привязки** в `DataTreeGrid` или `TreeView`.
- Для иерархических данных используйте `HierarchicalDataTemplate` или `ChildrenBinding`.

Если нужно больше деталей, уточните структуру `Component` и какие именно данные нужно отображать.






using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Text;
using System.Threading.Tasks;

namespace Pilot.HwTool.Mvvm
{
    [Serializable]
    public class ObservableObject : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        protected void RaisePropertyChanged([CallerMemberName] string propertyName = "")
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }

    public class ViewModelBase : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        protected void RaisePropertyChanged([CallerMemberName] string propertyName = "")
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        public virtual void Initialize()
        {

        }

        public virtual void Save()
        {

        }

        public virtual void Cancel()
        {

        }

        public virtual void Close()
        {

        }
    }
}
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Pilot.HwTool.Mvvm
{
    public class PropertyItemCollection : ObservableCollection<PropertyItemModel>
    {
        public PropertyItemCollection()
        {
        }

        public PropertyItemCollection(IEnumerable<PropertyItemModel> items)
            : base(items)
        {
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

    public class ParameterTypePropertyItemModel : PropertyItemModel
    {
        private Pilot.TargetPlatform.ParameterType _tag;

        public ParameterTypePropertyItemModel(Pilot.TargetPlatform.ParameterType tag)
            : base(tag)
        {
            _tag = tag;
        }

        public new Pilot.TargetPlatform.StringRefType Name { get => _tag.Name; set => _tag.Name = value; }
    }
}
using Pilot.HwTool.Core;
using Pilot.HwTool.Mvvm;
using Pilot.TargetPlatform;
using System;
using System.Collections.ObjectModel;
using System.ComponentModel.Composition;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Input;

namespace Pilot.HwTool
{
    [Export]
    [PartCreationPolicy(CreationPolicy.NonShared)]
    public class LibraryViewModel : Mvvm.ViewModelBase
    {
        private _DeviceDescriptionNode selectedNode;

        [ImportingConstructor]
        public LibraryViewModel()
        {
            HwRoot = new ObservableCollection<_DeviceDescriptionNode>();

            Import_StateBitsCommand = new RelayCommand(Import_StateBitsExecute);
            Import_LibSensorOutCommand = new RelayCommand(Import_LibSensorOutExecute);
            Module_AnalogInputCommand = new RelayCommand(Module_AnalogInputExecute);
            Export_CCodeCommand = new RelayCommand(Export_CCodeExecute, Export_CCodeCanExecute);
        }

        public ObservableCollection<_DeviceDescriptionNode> HwRoot { get; set; }

        public _DeviceDescriptionNode SelectedNode
        {
            get => selectedNode;
            set
            {
                selectedNode = value;
                RaisePropertyChanged();
            }
        }

        #region Import/Export

        public ICommand Import_StateBitsCommand { get; set; }
        public ICommand Import_LibSensorOutCommand { get; set; }
        public ICommand Module_AnalogInputCommand { get; set; }
        public ICommand Export_CCodeCommand { get; set; }

        private void Import_StateBitsExecute()
        {
            var text = Clipboard.GetText();
            if (!string.IsNullOrEmpty(text))
            {
                try
                {
                    var types = LzgConv.Import_xxxStateBits(text);
                    DeviceDescription dd = new DeviceDescription()
                    {
                        Types = new DeviceDescriptionTypes()
                        {
                            Items = new DeviceDescriptionNodeCollection(null, types),
                            //ItemsElementName = new ItemsChoiceType[] { },
                            //ItemsElementName = new[] { ItemsChoiceType.ArrayType, ItemsChoiceType.BitfieldType, ItemsChoiceType.EnumType, ItemsChoiceType.RangeType, ItemsChoiceType.StructType, ItemsChoiceType.UnionType },
                        }
                    };
                    var xmlstr = XmlHelper.Serialize2Str(dd);
                    Clipboard.SetText(xmlstr);
                    System.Media.SystemSounds.Asterisk.Play();
                }
                catch (Exception ex)
                {
                    Clipboard.Clear();
                    System.Media.SystemSounds.Exclamation.Play();
                }
            }
        }

        private void Import_LibSensorOutExecute()
        {
            var text = Clipboard.GetText();
            if (!string.IsNullOrEmpty(text))
            {
                try
                {
                    var typeList = LzgConv.Import_LibSensorOut_as_Range(text);
                    var paramSectList = LzgConv.Import_LibSensorOut_as_Param(text);
                    DeviceDescription dd = new DeviceDescription()
                    {
                        Types = new DeviceDescriptionTypes()
                        {
                            Items = new DeviceDescriptionNodeCollection(null, typeList),
                        },
                        ParameterSet = new DeviceDescriptionParameterSet
                        {
                            Items = new DeviceDescriptionNodeCollection(null, paramSectList),
                        }
                    };
                    var xmlstr = XmlHelper.Serialize2Str(dd);
                    Clipboard.SetText(xmlstr);
                    System.Media.SystemSounds.Asterisk.Play();
                }
                catch (Exception ex)
                {
                    Clipboard.Clear();
                    System.Media.SystemSounds.Exclamation.Play();
                }
            }
        }

        private void Module_AnalogInputExecute()
        {
            var text = Clipboard.GetText();
            if (!string.IsNullOrEmpty(text))
            {
                try
                {
                    var ddd = LzgConv.Import_Module_AnalogInput(text);
                    DeviceDescription dd = new DeviceDescription()
                    {
                        Device = new DeviceDescriptionDeviceCollection(null, ddd),
                    };
                    var xmlstr = XmlHelper.Serialize2Str(dd);
                    Clipboard.SetText(xmlstr);
                    System.Media.SystemSounds.Asterisk.Play();
                }
                catch (Exception ex)
                {
                    Clipboard.Clear();
                    System.Media.SystemSounds.Exclamation.Play();
                }
            }
        }

        private bool Export_CCodeCanExecute()
        {
            return SelectedNode is DeviceDescription;// || SelectedNode is DeviceDescriptionDevice;
        }

        private void Export_CCodeExecute()
        {
            if (SelectedNode is DeviceDescription devdesc)
            {
                try
                {
                    var cstr = new CTarget(devdesc).Export_CCode();
                    Clipboard.SetText(cstr);
                    System.Media.SystemSounds.Asterisk.Play();
                }
                catch (Exception ex)
                {
                    Clipboard.Clear();
                    System.Media.SystemSounds.Exclamation.Play();
                }
            }
        }

        #endregion

        public override async void Initialize()
        {
            await LoadLibraries();
        }

        private async Task LoadLibraries()
        {
            DeviceDescription platformDescription = null;
            try
            {
                var files = Directory.GetFiles("libraries", "*.xml");
                foreach (var item in files)
                {
                    platformDescription = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(item);
                    //platformDescription.ProjPath = Path.GetFullPath(item);
                    platformDescription._Name = Path.GetFileName(item);

                    Thread.Sleep(1);

                    Application.Current.Dispatcher.Invoke(() =>
                    {
                        HwRoot.Add(platformDescription);
                    });
                }
            }
            catch (Exception e)
            {

            }
        }
    }
}
<Window x:Class="Pilot.HwTool.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Pilot.HwTool"
        xmlns:mvvm="clr-namespace:Pilot.HwTool.Mvvm"
        mc:Ignorable="d"
        Title="Platform Configurator" WindowStartupLocation="CenterScreen">
    <Window.InputBindings>
        <KeyBinding Command="{Binding ProjectTab.UpdateCommand}" Key="F5"/>
        <KeyBinding Command="{Binding ProjectTab.NewCommand}" Key="N" Modifiers="Ctrl"/>
        <KeyBinding Command="{Binding ProjectTab.OpenCommand}" Key="O" Modifiers="Ctrl"/>
        <KeyBinding Command="{Binding ProjectTab.SaveCommand}" Key="S" Modifiers="Ctrl"/>
        <KeyBinding x:Name="cmd1" Command="{Binding LibraryTab.Import_StateBitsCommand}" Key="F10"/>
        <KeyBinding x:Name="cmd2" Command="{Binding LibraryTab.Import_LibSensorOutCommand}" Key="F11"/>
        <KeyBinding x:Name="cmd3" Command="{Binding LibraryTab.Module_AnalogInputCommand}" Key="F12"/>
    </Window.InputBindings>
    <Window.Resources>
        <Style x:Key="StretchableTreeViewItemTemplate" TargetType="TreeViewItem" BasedOn="{StaticResource {x:Type TreeViewItem}}">
            <!--<Setter Property="HorizontalContentAlignment" Value="Center" />-->
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="TreeViewItem">
                        <StackPanel>
                            <Grid>
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="Auto" MinWidth="19" />
                                    <ColumnDefinition Width="*" />
                                </Grid.ColumnDefinitions>
                                <Grid.RowDefinitions>
                                    <RowDefinition Height="Auto" />
                                    <RowDefinition />
                                </Grid.RowDefinitions>
                                <!--
                                     Note that the following do not work, but I believe the top 2 should?!
                                     <ToggleButton IsChecked="{TemplateBinding IsExpanded}" ClickMode="Press" Name="Expander">
                                     <ToggleButton IsChecked="{TemplateBinding Property=IsExpanded}" ClickMode="Press" Name="Expander">
                                     <ToggleButton IsChecked="{TemplateBinding Path=IsExpanded}" ClickMode="Press" Name="Expander">
                                -->
                                <ToggleButton IsChecked="{Binding Path=IsExpanded, RelativeSource={RelativeSource TemplatedParent}}" ClickMode="Press" Name="Expander">
                                    <ToggleButton.Style>
                                        <Style TargetType="ToggleButton">
                                            <Setter Property="UIElement.Focusable" Value="false" />
                                            <Setter Property="FrameworkElement.Width" Value="16" />
                                            <Setter Property="FrameworkElement.Height" Value="16" />
                                            <Setter Property="Control.Template">
                                                <Setter.Value>
                                                    <ControlTemplate TargetType="ToggleButton">
                                                        <Border Padding="5,5,5,5" Background="#00FFFFFF" Width="16" Height="16">
                                                            <Path Fill="#00FFFFFF" Stroke="#FF989898" Name="ExpandPath">
                                                                <Path.Data>
                                                                    <PathGeometry Figures="M0,0L0,6L6,0z" />
                                                                </Path.Data>
                                                                <Path.RenderTransform>
                                                                    <RotateTransform Angle="135" CenterX="3" CenterY="3" />
                                                                </Path.RenderTransform>
                                                            </Path>
                                                        </Border>
                                                        <ControlTemplate.Triggers>
                                                            <Trigger Property="UIElement.IsMouseOver" Value="True">
                                                                <Setter TargetName="ExpandPath" Property="Shape.Stroke" Value="#FF1BBBFA" />
                                                                <Setter TargetName="ExpandPath" Property="Shape.Fill" Value="#00FFFFFF" />
                                                            </Trigger>
                                                            <Trigger Property="ToggleButton.IsChecked" Value="True">
                                                                <Setter TargetName="ExpandPath" Property="UIElement.RenderTransform">
                                                                    <Setter.Value>
                                                                        <RotateTransform Angle="180" CenterX="3" CenterY="3" />
                                                                    </Setter.Value>
                                                                </Setter>
                                                                <Setter TargetName="ExpandPath" Property="Shape.Fill" Value="#FF595959" />
                                                                <Setter TargetName="ExpandPath" Property="Shape.Stroke" Value="#FF262626" />
                                                            </Trigger>
                                                        </ControlTemplate.Triggers>
                                                    </ControlTemplate>
                                                </Setter.Value>
                                            </Setter>
                                        </Style>
                                    </ToggleButton.Style>
                                </ToggleButton>
                                <Border x:Name="Bd"
                                    HorizontalAlignment="Stretch"
                                    BorderThickness="{TemplateBinding Border.BorderThickness}"
                                    BorderBrush="{TemplateBinding Border.BorderBrush}"
                                    Padding="{TemplateBinding Control.Padding}"
                                    Background="{TemplateBinding Panel.Background}"
                                    SnapsToDevicePixels="True"
                                    Grid.Column="1">
                                    <ContentPresenter x:Name="PART_Header"
                                        Content="{TemplateBinding HeaderedContentControl.Header}"
                                        ContentTemplate="{TemplateBinding HeaderedContentControl.HeaderTemplate}"
                                        ContentStringFormat="{TemplateBinding HeaderedItemsControl.HeaderStringFormat}"
                                        ContentTemplateSelector="{TemplateBinding HeaderedItemsControl.HeaderTemplateSelector}"
                                        ContentSource="Header"
                                        HorizontalAlignment="{TemplateBinding Control.HorizontalContentAlignment}"
                                        SnapsToDevicePixels="{TemplateBinding UIElement.SnapsToDevicePixels}" />
                                </Border>
                                <ItemsPresenter x:Name="ItemsHost"
                                    Grid.Column="1"
                                    Grid.Row="1" />
                            </Grid>
                        </StackPanel>
                        <ControlTemplate.Triggers>
                            <Trigger Property="TreeViewItem.IsExpanded" Value="False">
                                <Setter TargetName="ItemsHost" Property="UIElement.Visibility" Value="Collapsed" />
                            </Trigger>
                            <Trigger Property="ItemsControl.HasItems" Value="False">
                                <Setter TargetName="Expander" Property="UIElement.Visibility" Value="Hidden" />
                            </Trigger>
                            <Trigger Property="TreeViewItem.IsSelected" Value="True">
                                <Setter TargetName="Bd" Property="Panel.Background" Value="{DynamicResource {x:Static SystemColors.HighlightBrushKey}}" />
                                <Setter Property="TextElement.Foreground" Value="{DynamicResource {x:Static SystemColors.HighlightTextBrushKey}}" />
                            </Trigger>
                            <MultiTrigger>
                                <MultiTrigger.Conditions>
                                    <Condition Property="TreeViewItem.IsSelected" Value="True" />
                                    <Condition Property="Selector.IsSelectionActive" Value="False" />
                                </MultiTrigger.Conditions>
                                <Setter TargetName="Bd" Property="Panel.Background" Value="{DynamicResource {x:Static SystemColors.ControlBrushKey}}" />
                                <Setter Property="TextElement.Foreground" Value="{DynamicResource {x:Static SystemColors.ControlTextBrushKey}}" />
                            </MultiTrigger>
                            <Trigger Property="UIElement.IsEnabled" Value="False">
                                <Setter Property="TextElement.Foreground" Value="{DynamicResource {x:Static SystemColors.GrayTextBrushKey}}" />
                            </Trigger>
                        </ControlTemplate.Triggers>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </Window.Resources>
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="2*"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="3*"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="2*"/>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="4*"/>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="2*"/>
        </Grid.ColumnDefinitions>
        <Grid Grid.ColumnSpan="5">
            <Menu>
                <MenuItem Header="File">
                    <MenuItem Header="New" Command="{Binding DataContext.NewCommand, ElementName=ProjectTab}" InputGestureText="Ctrl+N"/>
                    <MenuItem Header="Open" Command="{Binding ProjectTab.OpenCommand}" InputGestureText="Ctrl+O"/>
                    <MenuItem Header="Save" Command="{Binding ProjectTab.SaveCommand}" InputGestureText="Ctrl+S"/>
                    <MenuItem Header="Save As..." Command="{Binding ProjectTab.SaveAsCommand}"/>
                    <Separator/>
                    <MenuItem Header="Close" Command="{Binding ProjectTab.CloseCommand}"/>
                </MenuItem>
                <MenuItem Header="Import">
                    <MenuItem Header="StateBits" Command="{Binding LibraryTab.Import_StateBitsCommand}" InputGestureText="{Binding Key, ElementName=cmd1}"/>
                    <MenuItem Header="LibSensorOut" Command="{Binding LibraryTab.Import_LibSensorOutCommand}"/>
                    <MenuItem Header="Module_AnalogInput" Command="{Binding LibraryTab.Module_AnalogInputCommand}"/>
                </MenuItem>
                <MenuItem Header="Export">
                    <MenuItem Header="Export_CSources" Command="{Binding ProjectTab.Export_CSourcesCommand}"/>
                    <MenuItem Header="Export_Binary" Command="{Binding ProjectTab.Export_BinaryCommand}"/>
                    <Separator/>
                    <MenuItem Header="Clipboard_CCode" Command="{Binding ProjectTab.Export_CCodeCommand}"/>
                    <MenuItem Header="Clipboard_CHeader" Command="{Binding ProjectTab.Export_CHeaderCommand}"/>
                </MenuItem>
            </Menu>
        </Grid>
        <!--<Grid Grid.Column="0">
            <TreeView ItemsSource="{Binding HwRoot}" ItemContainerStyle="{StaticResource StretchableTreeViewItemTemplate}" ItemTemplate="{StaticResource NodeTemplate}">
            </TreeView>
        </Grid>-->
        <Grid Grid.Column="0" Grid.Row="1" Grid.RowSpan="3" DataContext="{Binding ProjectTab}">
            <local:ProjectView x:Name="ProjectTab"/>
        </Grid>
        <GridSplitter Grid.Column="1" Grid.Row="1" Grid.RowSpan="3" Width="5" HorizontalAlignment="Center"/>
        <GridSplitter Grid.Column="3" Grid.Row="1" Grid.RowSpan="3" Width="5" HorizontalAlignment="Center"/>
        <Grid Grid.Column="4" Grid.Row="1" Grid.RowSpan="3" DataContext="{Binding LibraryTab}">
            <local:LibraryView/>
        </Grid>
        <Grid Grid.Column="2" Grid.Row="1" DataContext="{Binding PropertyTab}">
            <local:PropertyView/>
        </Grid>
        <GridSplitter Grid.Column="2" Grid.Row="2" Height="5" HorizontalAlignment="Stretch" VerticalAlignment="Center"/>
        <Grid Grid.Column="2" Grid.Row="3" DataContext="{Binding PropertyTab2}">
            <local:Property2View/>
        </Grid>
        <GridSplitter Grid.Column="0" Grid.Row="4" Grid.ColumnSpan="5" Height="5" HorizontalAlignment="Stretch" VerticalAlignment="Center"/>
        <Grid Grid.Column="0" Grid.Row="5" Grid.ColumnSpan="5" DataContext="{Binding LoggerTab}">
            <local:LoggerView/>
        </Grid>
    </Grid>
</Window>
<mvvm:UserControl x:Class="Pilot.HwTool.Property2View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:sys="clr-namespace:System;assembly=mscorlib"
             xmlns:local="clr-namespace:Pilot.HwTool"
             xmlns:mvvm="clr-namespace:Pilot.HwTool.Mvvm"
             xmlns:kivi="http://kivi.dev.ekra.ru"
             xmlns:pilot="http://pilot.ekra.ru"
             mc:Ignorable="d" 
             d:DesignHeight="450" d:DesignWidth="800">
    <UserControl.Resources>
        <BooleanToVisibilityConverter x:Key="bool2vis"/>
        <kivi:VisibilityToBooleanConverter x:Key="vis2bool"/>
        <local:DataTreeGridAncestorConverter x:Key="dtgac"/>
        <local:DataTreeGridColumnConverter x:Key="dtgcc"/>
        <local:AttributeValueConverter x:Key="attributeValueConverter"/>
        <local:StringRefTypeConverter x:Key="strRefConv"/>
        <local:DebugConverter x:Key="DebugConverter"/>
        <DataTemplate x:Key="DefaultValueTemplate">
            <DataTemplate.Resources>
                <local:ParameterValueConverter x:Key="parameterValueConverter"/>
            </DataTemplate.Resources>
            <TextBlock Text="{Binding Tag.Tag.Default, Converter={StaticResource parameterValueConverter}, UpdateSourceTrigger=PropertyChanged}"/>
        </DataTemplate>
        <DataTemplate x:Key="DefaultValueEditTemplate">
            <DataTemplate.Resources>
                <local:ParameterValueConverter x:Key="parameterValueConverter"/>
            </DataTemplate.Resources>
            <TextBox Text="{Binding Tag.Tag.Default, Converter={StaticResource parameterValueConverter}, UpdateSourceTrigger=PropertyChanged}"/>
        </DataTemplate>
        <DataTemplate x:Key="DataTypeTemplate">
            <DataTemplate.Resources>
                <local:ParameterValueConverter x:Key="parameterValueConverter"/>
            </DataTemplate.Resources>
            <TextBlock Text="{Binding Tag.Tag.type, UpdateSourceTrigger=PropertyChanged}"/>
        </DataTemplate>
        <DataTemplate x:Key="DataTypeEditTemplate">
            <DataTemplate.Resources>
                <local:ParameterValueConverter x:Key="parameterValueConverter"/>
                <x:Array x:Key="DataTypeSource" Type="{x:Type sys:String}">
                    <sys:String>std:BIT</sys:String>
                    <sys:String>std:BOOL</sys:String>
                    <sys:String>std:BYTE</sys:String>
                    <sys:String>std:SINT</sys:String>
                    <sys:String>std:USINT</sys:String>
                    <sys:String>std:CHAR</sys:String>
                    <sys:String>std:INT</sys:String>
                    <sys:String>std:UINT</sys:String>
                    <sys:String>std:WORD</sys:String>
                    <sys:String>std:DINT</sys:String>
                    <sys:String>std:UDINT</sys:String>
                    <sys:String>std:DWORD</sys:String>
                    <sys:String>std:REAL</sys:String>
                    <sys:String>std:DATE</sys:String>
                    <sys:String>std:TIME</sys:String>
                    <sys:String>std:LINT</sys:String>
                    <sys:String>std:ULINT</sys:String>
                    <sys:String>std:LWORD</sys:String>
                    <sys:String>std:LREAL</sys:String>
                    <sys:String>std:LDATE</sys:String>
                    <sys:String>std:LTIME</sys:String>
                    <sys:String>std:STRING</sys:String>
                </x:Array>
            </DataTemplate.Resources>
            <ComboBox Text="{Binding Tag.Tag.type, UpdateSourceTrigger=PropertyChanged}" ItemsSource="{Binding Source={StaticResource DataTypeSource}}" IsEditable="True">
            </ComboBox>
        </DataTemplate>
        <ObjectDataProvider x:Key="ParameterTypeAttributesChannelSource" MethodName="GetValues"
                            ObjectType="{x:Type sys:Enum}">
            <ObjectDataProvider.MethodParameters>
                <x:Type TypeName="pilot:ParameterTypeAttributesChannel"/>
            </ObjectDataProvider.MethodParameters>
        </ObjectDataProvider>
        <ObjectDataProvider x:Key="TypedefTypeComponentStateFlagsSource" MethodName="GetValues"
                            ObjectType="{x:Type sys:Enum}">
            <ObjectDataProvider.MethodParameters>
                <x:Type TypeName="pilot:TypedefTypeComponentStateFlags"/>
            </ObjectDataProvider.MethodParameters>
        </ObjectDataProvider>
        <local:NameStringRefConverter x:Key="NameStringRefConverter" />
        <DataTemplate x:Key="AttributesChannelTemplate">
            <TextBlock Text="{Binding Tag.Tag.Attributes.channel}"/>
        </DataTemplate>
        <DataTemplate x:Key="AttributesChannelEditTemplate">
            <ComboBox Text="{Binding Tag.Tag.Attributes.channel}" ItemsSource="{Binding Source={StaticResource ParameterTypeAttributesChannelSource}}">
                <ComboBox.Style>
                    <Style TargetType="ComboBox">
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding Tag.Tag.Attributes}" Value="{x:Null}">
                                <Setter Property="IsEnabled" Value="False"/>
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>
                </ComboBox.Style>
            </ComboBox>
        </DataTemplate>
        <DataTemplate x:Key="AttributesStateTemplate">
            <TextBlock Text="{Binding Tag.Tag.Attributes.state}"/>
        </DataTemplate>
        <DataTemplate x:Key="AttributesStateEditTemplate">
            <ComboBox SelectedItem="{Binding Path=Tag.Tag.Attributes.state}" ItemsSource="{Binding Source={StaticResource TypedefTypeComponentStateFlagsSource}}">
                <ComboBox.Style>
                    <Style TargetType="ComboBox">
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding Tag.Tag.Attributes}" Value="{x:Null}">
                                <Setter Property="IsEnabled" Value="False"/>
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>
                </ComboBox.Style>
            </ComboBox>
        </DataTemplate>
    </UserControl.Resources>
    <Grid>
        <kivi:DataTreeGrid SelectionUnit="FullRow" CanUserAddRows="False" RowHeight="23">
            <kivi:DataTreeGrid.Items>
                <Binding Path="Items">
                    <Binding.Converter>
                        <kivi:NodeItemsConverter ContentMember="Name" IndentationMember="IndentLevel" UpdateTargetOnSourceChanges="True"
                                                 ContentConverter="{StaticResource NameStringRefConverter}"/>
                    </Binding.Converter>
                </Binding>
            </kivi:DataTreeGrid.Items>
            <kivi:DataTreeGrid.Columns>
                <kivi:DataTreeGridColumn mvvm:DTGAP.Key="Name" Header="Name" Width="350"/>
                <DataGridTextColumn mvvm:DTGAP.Key="Id" Header="Id" Binding="{Binding Tag.Tag.DisplayId, UpdateSourceTrigger=LostFocus}" Width="Auto" />
                <!--<DataGridTextColumn mvvm:DTGAP.Key="Name" Header="Name" Binding="{Binding Tag.Tag.Name}" Width="150"/>-->
                <DataGridTemplateColumn mvvm:DTGAP.Key="Type" Header="Type" Width="100"
                    CellTemplate="{StaticResource DataTypeTemplate}" CellEditingTemplate="{StaticResource DataTypeEditTemplate}"/>
                <DataGridTemplateColumn mvvm:DTGAP.Key="Value" Header="Value" Width="150"
                    CellTemplate="{StaticResource DefaultValueTemplate}" CellEditingTemplate="{StaticResource DefaultValueEditTemplate}"/>
                <DataGridTextColumn mvvm:DTGAP.Key="Unit" Header="Unit" Binding="{Binding Tag.Tag.Unit, Converter={StaticResource strRefConv}}" Width="100"/>
                <!--<DataGridTextColumn mvvm:DTGAP.Key="Attributes" Header="Attributes" Width="150"
                    Binding="{Binding Tag.Tag.Attributes, Converter={StaticResource attributeValueConverter}}"/>-->
                <DataGridTemplateColumn mvvm:DTGAP.Key="Channel" Header="Channel" Width="70"
                    CellTemplate="{StaticResource AttributesChannelTemplate}" CellEditingTemplate="{StaticResource AttributesChannelEditTemplate}"/>
                <DataGridTemplateColumn mvvm:DTGAP.Key="State" Header="State" Width="70"
                    CellTemplate="{StaticResource AttributesStateTemplate}" CellEditingTemplate="{StaticResource AttributesStateEditTemplate}"/>
                <DataGridTextColumn mvvm:DTGAP.Key="Description" Header="Description" Binding="{Binding Tag.Tag.Description, Converter={StaticResource strRefConv}}" Width="150"/>
                <!-- stub column for fill empty space at right -->
                <!--<DataGridTextColumn Width="*"/>-->
            </kivi:DataTreeGrid.Columns>
            <kivi:DataTreeGrid.ColumnHeaderStyle>
                <Style TargetType="{x:Type DataGridColumnHeader}">
                    <Setter Property="HorizontalContentAlignment" Value="Center"/>
                    <Setter Property="ToolTip" Value="{Binding}"/>
                    <Setter Property="ContextMenu">
                        <Setter.Value>
                            <!--TODO: apply styles for MenuItem-->
                            <!--<ContextMenu ItemsSource="{Binding PlacementTarget, RelativeSource={RelativeSource Self}, Converter={StaticResource dtgcc}}">
                                <ContextMenu.ItemContainerStyle>
                                    <Style TargetType="MenuItem">
                                        <Setter Property="IsCheckable" Value="True"/>
                                        <Setter Property="Header" Value="{Binding Header}"/>
                                        <Setter Property="IsChecked" Value="{Binding Visibility, Converter={StaticResource vis2bool}}"/>
                                    </Style>
                                </ContextMenu.ItemContainerStyle>
                            </ContextMenu>-->
                            <ContextMenu Tag="{Binding PlacementTarget, RelativeSource={RelativeSource Self}, Converter={StaticResource dtgac}}">
                                <MenuItem IsCheckable="True" Header="{Binding Tag.Columns[0].Header, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                    IsChecked="{Binding Tag.Columns[0].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
                                <MenuItem IsCheckable="True" Header="{Binding Tag.Columns[1].Header, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                    IsChecked="{Binding Tag.Columns[1].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
                                <MenuItem IsCheckable="True" Header="{Binding Tag.Columns[2].Header, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                    IsChecked="{Binding Tag.Columns[2].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
                                <MenuItem IsCheckable="True" Header="{Binding Tag.Columns[3].Header, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                    IsChecked="{Binding Tag.Columns[3].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
                                <MenuItem IsCheckable="True" Header="{Binding Tag.Columns[4].Header, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                    IsChecked="{Binding Tag.Columns[4].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
                                <MenuItem IsCheckable="True" Header="{Binding Tag.Columns[5].Header, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                    IsChecked="{Binding Tag.Columns[5].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
                                <MenuItem IsCheckable="True" Header="{Binding Tag.Columns[6].Header, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                    IsChecked="{Binding Tag.Columns[6].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
                                <MenuItem IsCheckable="True" Header="{Binding Tag.Columns[7].Header, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                    IsChecked="{Binding Tag.Columns[7].Visibility, Converter={StaticResource vis2bool}, RelativeSource={RelativeSource AncestorType=ContextMenu}}"/>
                            </ContextMenu>
                        </Setter.Value>
                    </Setter>
                </Style>
            </kivi:DataTreeGrid.ColumnHeaderStyle>
        </kivi:DataTreeGrid>
    </Grid>
</mvvm:UserControl>
