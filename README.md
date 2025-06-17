        <Connector connectorId="2" explicit="false" hostpath="1" interface="Ekra.P18xx.Delay" moduleType="11504" role="parent">
      <InterfaceName name="local:Delay">Параметры выходов с настраиваемой выдержкой</InterfaceName>
      <Slot allowEmpty="true" count="1" />

      <ParameterSet>
        <ParameterSection>
          <Name name="local:Delay">Параметры выходов с настраиваемой выдержкой</Name>
          <Parameter ParameterId="4005" type="local:T_R17xx_Delay">
            <Default>0</Default>
            <Name name="local:Id4005">Параметры выходов с настраиваемой выдержкой</Name>
          </Parameter>
        </ParameterSection>
      </ParameterSet>
    </Connector>

<StructType name="T_R17xx_Delay">
      <Component identifier="Срабатывание" type="std:INT">
        <Default>15</Default>
        <VisibleName name="local:Item0">выдержка на срабатывание, в миллисекундах</VisibleName>
      </Component>
      <Component identifier="Возврат" type="std:INT">
        <Default>6</Default>
        <VisibleName name="local:Item1">выдержка на возврат, в миллисекундах</VisibleName>
      </Component>
    </StructType>





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
