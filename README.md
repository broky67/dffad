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


