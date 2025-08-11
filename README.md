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
