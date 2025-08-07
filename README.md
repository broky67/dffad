Проблема в том, что вы создаете ViewModel, но не показываете диалоговое окно. Давайте исправим ваш код:

## 1. Исправленный FindToolModule.cs

```csharp
using Catel.MVVM;
using Microsoft.Practices.Prism.Events;
using Module.FindTool.Dialogs;
using Pilot.Shell.Infrastructure.Modules;
using Pilot.Shell.Infrastructure.Services;
using SCLExpress.Core;
using System;
using System.Collections;
using System.ComponentModel.Composition;
using System.Windows;
using System.Windows.Input;

namespace Module.FindTool
{
    public class FindToolModule : ModuleBase
    {
        private readonly IEventAggregator _eventAggregator;
        private readonly IUIVisualizerService _uiVisualizerService;
        
        [Import]
        private IWorkspaceManager _workspaceManager;
        
        [Import]
        private Lazy<ISclObjectManager> _projectManager;
        
        [Import]
        private IRibbonService _ribbonService;

        public const string Name = "Find Tool";

        [ImportingConstructor]
        public FindToolModule(IEventAggregator eventAggregator, IUIVisualizerService uiVisualizerService)
            : base(Name)
        {
            _eventAggregator = eventAggregator;
            _uiVisualizerService = uiVisualizerService;
        }

        public override ILicenseInfo GetLicense()
        {
            return new LicenseInfo() { Url = "www.ekra.ru" };
        }

        protected override void OnInitialized()
        {
            base.OnInitialized();
            InitializeRibbon(_ribbonService);
        }

        protected void InitializeRibbon(IRibbonService ribbonService)
        {
            var assembly = this.GetType().Assembly;
            try
            {
                var resPath = $"/{assembly.GetName().Name};component/FindToolRibbon.xaml";
                var resUri = new Uri(resPath, UriKind.RelativeOrAbsolute);
                var resource = new ResourceDictionary() { Source = resUri };
                ribbonService.AddRibbonItem((FrameworkElement)resource["Tools"], this);
            }
            catch (Exception e)
            {
                // Логирование ошибки
                Console.WriteLine(e);
            }
        }

        private ICommand _showFindToolCommand;

        public ICommand ShowFindToolDialogCommand
        {
            get { return _showFindToolCommand ?? (_showFindToolCommand = new Command(ShowFindToolDialog, ShowFindToolDialogCanExecute)); }
        }

        private bool ShowFindToolDialogCanExecute()
        {
            return _projectManager?.Value?.ActiveProject != null;
        }

        private async void ShowFindToolDialog()
        {
            var active = _projectManager.Value.ActiveProject;
            if (active == null) return;

            var viewModel = new FindToolViewModel();
            await _uiVisualizerService.ShowDialogAsync(viewModel);
        }
    }
}
```

## 2. Исправленный FindToolViewModel.cs

```csharp
using Catel.MVVM;

namespace Module.FindTool.Dialogs
{
    [View(typeof(FindToolView))]
    public class FindToolViewModel : ViewModelBase
    {
        private string _searchText;

        public string SearchText
        {
            get => _searchText;
            set => SetProperty(ref _searchText, value);
        }

        public FindToolViewModel()
        {
            Title = "Find Tool";
        }
    }
}
```

## 3. FindToolRibbon.xaml (новый файл)

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:fluent="urn:fluent-ribbon"
                    xmlns:ribbon="clr-namespace:Pilot.Shell.Infrastructure.Ribbon;assembly=Pilot.Shell.Infrastructure"
                    xmlns:pilot="http://pilot.dev.ekra.ru">

    <fluent:RibbonTabItem Header="{x:Static ribbon:RibbonTabs.Tools}" x:Key="Tools">
        <fluent:RibbonGroupBox Header="Search">
            <fluent:Button Header="Find Tool"
                           Command="{Binding ShowFindToolDialogCommand}"
                           Icon="/Module.FindTool;component/Resources/find_icon.png"
                           LargeIcon="/Module.FindTool;component/Resources/32/find_icon.png"
                           ToolTip="Search in project"/>
        </fluent:RibbonGroupBox>
    </fluent:RibbonTabItem>

</ResourceDictionary>
```

## 4. FindToolView.xaml

```xml
<catel:DataWindow x:Class="Module.FindTool.Dialogs.FindToolView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:catel="http://schemas.catelproject.com"
                  Width="400" Height="300"
                  SizeToContent="Manual"
                  ResizeMode="CanResize"
                  WindowStartupLocation="CenterOwner">
    
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        
        <!-- Search Box -->
        <StackPanel Grid.Row="0" Orientation="Horizontal">
            <TextBlock Text="Search:" VerticalAlignment="Center" Margin="0,0,5,0"/>
            <TextBox Text="{Binding SearchText}" Width="200"/>
        </StackPanel>
        
        <!-- Results Area -->
        <Border Grid.Row="1" BorderBrush="LightGray" BorderThickness="1" Margin="0,10">
            <TextBlock Text="Enter search term and press Find" 
                      HorizontalAlignment="Center" VerticalAlignment="Center"
                      Foreground="Gray"/>
        </Border>
        
        <!-- Buttons -->
        <StackPanel Grid.Row="2" Orientation="Horizontal" HorizontalAlignment="Right">
            <Button Content="Find" Width="80" Margin="0,0,5,0"/>
            <Button Content="Close" Width="80" Command="{Binding CancelCommand}"/>
        </StackPanel>
    </Grid>
</catel:DataWindow>
```

## 5. FindToolView.xaml.cs

```csharp
using Catel.Windows;

namespace Module.FindTool.Dialogs
{
    public partial class FindToolView : DataWindow
    {
        public FindToolView()
            : base(DataWindowMode.OkCancel)
        {
            InitializeComponent();
        }
    }
}
```

## Ключевые исправления:

1. **Показ диалогового окна**: В методе `ShowFindToolDialog` добавлен вызов `_uiVisualizerService.ShowDialogAsync(viewModel)`

2. **Свойство SearchText**: Добавлен сеттер с вызовом `SetProperty` для правильной работы привязки

3. **Отдельный ResourceDictionary**: Создан отдельный файл `FindToolRibbon.xaml` вместо использования ресурсов IedExplorer

4. **Исправлена привязка команды**: В Ribbon кнопка теперь правильно привязана к `ShowFindToolDialogCommand`

5. **Добавлен View**: Созданы файлы View для отображения диалога

## Что нужно проверить:

1. Убедитесь, что файл `FindToolRibbon.xaml` добавлен в проект с правильными свойствами сборки (Build Action = Page)

2. Проверьте, что в проекте есть папка Resources с иконками (или укажите правильные пути)

3. Убедитесь, что модуль зарегистрирован в основном приложении

Если после этих изменений кнопка все равно не появляется, проверьте:
- Логи инициализации модуля
- Наличие исключений при загрузке ResourceDictionary
- Корректность привязки команд в Ribbon
-
-
-
-
-
-
- using Catel.MVVM;
using Microsoft.Practices.Prism.Events;
using Module.FindTool.Dialogs;
using Pilot.Shell.Infrastructure.Modules;
using Pilot.Shell.Infrastructure.Services;
using SCLExpress.Core;
using System;
using System.Collections;
using System.ComponentModel.Composition;
using System.Windows;
using System.Windows.Input;

namespace Module.FindTool
{
    public class FindToolModule : ModuleBase
    {

        private readonly IEventAggregator _eventAggregator;
        private readonly IUIVisualizerService _uiVisualizerService;
#pragma warning disable 0649
        [Import]
        private IWorkspaceManager _workspaceManager;
        [Import]
        private Lazy<ISclObjectManager> _projectManager;
        [Import]
        private IRibbonService _ribbonService;

        #region Constants

        /// <summary>
        /// The module name.
        /// </summary>
        public const string Name = "Find Tool";

        #endregion

        #region Constructors

        /// <summary>
        /// Initializes a new instance of the <see cref="FindToolModule" /> class.
        /// </summary>
        [ImportingConstructor]
        public FindToolModule(IEventAggregator eventAggregator, IUIVisualizerService uiVisualizerService)
            : base(Name)
        {
            _eventAggregator = eventAggregator;
            _uiVisualizerService = uiVisualizerService;
        }

        #endregion

        #region Methods

        /// <summary>
        /// Gets the license info.
        /// <para />
        /// If method returns null, it is assumed that module has no license.
        /// </summary>
        /// <returns>The license info.</returns>
        public override ILicenseInfo GetLicense()
        {
            return
                new LicenseInfo() { Url = "www.ekra.ru" };
        }

        /// <summary>
        /// The on initialized.
        /// </summary>
        protected override void OnInitialized()
        {
            base.OnInitialized();

            InitializeRibbon(_ribbonService);
        }

        /// <summary>
        /// Initializes the ribbon.
        /// <para />
        /// Use this method to hook up views to ribbon items.
        /// </summary>
        /// <param name="ribbonService">The ribbon service.</param>
        protected void InitializeRibbon(IRibbonService ribbonService)
        {
            var assembly = this.GetType().Assembly;
            try
            {
                var resPath = string.Format("/{0};component/IedExplorerRibbon.xaml", assembly.GetName().Name);
                var resUri = new Uri(resPath, UriKind.RelativeOrAbsolute);
                var resource = new ResourceDictionary() { Source = resUri };
                foreach (DictionaryEntry entry in resource)
                {
                    ribbonService.AddRibbonItem((FrameworkElement)entry.Value, this);
                }
            }
            catch (Exception e)
            {
            }
        }

        private ICommand _showFindToolCommand;

        public ICommand ShowFindToolDialogCommand
        {
            get { return _showFindToolCommand ?? (_showFindToolCommand = new Command(ShowFindToolDialog, ShowFindToolDialogCanExecute)); }
        }

        private bool ShowFindToolDialogCanExecute()
        {
            if (_projectManager == null)
                return false;

            return _projectManager.Value.ActiveProject != null;
        }

        private void ShowFindToolDialog()
        {
            var active = _projectManager.Value.ActiveProject;

            if (active == null)
                return;

            //Create dialog
            new FindToolViewModel();
            }
        }

        #endregion
}

using Catel.MVVM;

namespace Module.FindTool.Dialogs
{
    [View(typeof(FindToolView))]
    public class FindToolViewModel : ViewModelBase
    {
        private string _searchText;

        public string SearchText
        {
            get => _searchText;
        }

        public FindToolViewModel()
        {
            Title = "Find Tool";
        }
    }
}



<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:fluent="urn:fluent-ribbon"
                    xmlns:ribbon="clr-namespace:Pilot.Shell.Infrastructure.Ribbon;assembly=Pilot.Shell.Infrastructure"
                    xmlns:properties="clr-namespace:Module.IedExplorer.Properties"
                    xmlns:pilot="http://pilot.dev.ekra.ru"
                    xmlns:local="clr-namespace:Module.IedExplorer">

    <fluent:RibbonTabItem Header="{x:Static ribbon:RibbonTabs.Tools}" x:Key="Tools">
        <fluent:RibbonGroupBox Header="{x:Static ribbon:RibbonTabGroups.Online}">
            <fluent:Button Header="{x:Static properties:Resources.Scan}"
                           Command="{Binding ShowDiscoverIedDialogCommand}"
                           Icon="/Module.IedExplorer;component/Resources/Images/ied_scan.png"
                           LargeIcon="/Module.IedExplorer;component/Resources/Images/32/ied_scan.png">
            </fluent:Button>
            <fluent:Button Header="FindTool"
                           Command="{Binding ShowFindToolDialogCommand}"
                           Icon="/Module.IedExplorer;component/Resources/Images/ied_scan.png"
                           LargeIcon="/Module.IedExplorer;component/Resources/Images/32/ied_scan.png">
            </fluent:Button>
        </fluent:RibbonGroupBox>
    </fluent:RibbonTabItem>

</ResourceDictionary>


