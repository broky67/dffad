<catel:DataWindow x:Class="Module.IedExplorer.Dialogs.FindToolView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:catel="http://catel.codeplex.com"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
                  xmlns:xctk="http://schemas.xceed.com/wpf/xaml/toolkit"
                  Height="500" Width="700" MinHeight="300" MinWidth="400">

<UserControl x:Class="Module.NetworkDesigner.FlowView.SubNetworkFlowView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:views="clr-namespace:Module.NetworkDesigner.FlowView"
             xmlns:kivi="http://kivi.dev.ekra.ru"
             xmlns:scm="clr-namespace:System.ComponentModel;assembly=WindowsBase"
             xmlns:local="clr-namespace:Module.NetworkDesigner"
             xmlns:filter="http://datagridfilterlibrary.dev.ekra.ru"
             xmlns:xctk="http://schemas.xceed.com/wpf/xaml/toolkit"
             xmlns:behavior="clr-namespace:Module.NetworkDesigner.Behavior"
             xmlns:sys="clr-namespace:System;assembly=mscorlib"
             xmlns:properties="clr-namespace:Module.NetworkDesigner.Properties"
             mc:Ignorable="d" d:DesignHeight="300" d:DesignWidth="650"
             d:DataContext="{d:DesignInstance views:DesignTimeSubNetworkFlowViewModel, IsDesignTimeCreatable=True}"
             kivi:CommandBindings.CommandBindings="{Binding CommandBindings}">


            <xctk:CheckComboBox x:Name="PublishersComboBox" Style="{StaticResource CheckComboBoxStyle}"
                          ItemsSource="{Binding PublisherList}" DisplayMemberPath="IedName">
                <i:Interaction.Behaviors>
                    <behavior:XMultiSelectBehavior SelectedItems="{Binding FilteredPublishers}"/>
                </i:Interaction.Behaviors>
            </xctk:CheckComboBox>
