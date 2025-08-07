using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Diagnostics;
using System.Linq;
using Catel.MVVM;

namespace Module.IedExplorer.Dialogs
{
    using RM = Properties.Resources;
    using SM = Properties.Settings;

    /// <summary>
    /// The DiscoverIedDialogViewModel view model.
    /// </summary>
    [View(typeof(DiscoverIedDialogView))]
    public partial class DiscoverIedDialogViewModel : ViewModelBase
    {
        private string DefaultTitle = RM.DiscoverIedDialog_Title;
        private IedConnectionParam _selectedParam;
        private IedConnectionParam _selectedParamCopy;
        private bool _isOffline;
        private bool _isComParamCheck;
        private bool _isAdvParamCheck;
        private readonly DiscoverIedDialogProperties _discoverIedDialogProperties;

        /// <summary>
        /// Gets the title of the view model.
        /// </summary>
        /// <value>The title.</value>
        public override string Title
        {
            get { return DefaultTitle; }
        }

        #region ctor
        /// <summary>
        /// Initializes a new instance of the <see cref="DiscoverIedDialogViewModel" /> class.
        /// </summary>
        //public DiscoverIedDialogViewModel(IedConnectionParam connParam)
        public DiscoverIedDialogViewModel()
        {
            #region Load dialog settings
            if (SM.Default.DiscoverIedDialogProperties == null)
            {
                SM.Default.DiscoverIedDialogProperties = new DiscoverIedDialogProperties();
            }
            _discoverIedDialogProperties = SM.Default.DiscoverIedDialogProperties;
            #endregion

            //LastParams.Add(connParam);
            //LastParams.Add(new IedConnectionParam("1.1.1.1"));
            //LastParams.Add(new IedConnectionParam("2.2.2.2"));
            //LastParams.Add(new IedConnectionParam("3.3.3.3"));
            //LastParams.Add(new IedConnectionParam("4.4.4.4"));
            //LastParams.Add(new IedConnectionParam("rtyhtju"));

            LastParams = new ObservableCollection<IedConnectionParam>();
            SelectedParamCopy = new IedConnectionParam();

            InitializeUiCommands();
        }
        #endregion

        [Model]
        public DiscoverIedDialogProperties Model
        {
            get { return _discoverIedDialogProperties; }
        }

        public ObservableCollection<IedConnectionParam> LastParams { get; set; }

        public IedConnectionParam SelectedParam
        {
            get { return _selectedParam; }
            set
            {
                if (Equals(value, _selectedParam)) return;
                _selectedParam = value;
                RaisePropertyChanged(nameof(SelectedParam));

                if (_selectedParam != null)
                {
                    // Set copy of current param
                    SelectedParamCopy = (IedConnectionParam)((ICloneable)_selectedParam).Clone();
                }
            }
        }

        public IedConnectionParam SelectedParamCopy
        {
            get { return _selectedParamCopy; }
            set
            {
                if (Equals(value, _selectedParamCopy)) return;
                _selectedParamCopy = value;
                RaisePropertyChanged(nameof(SelectedParamCopy));
            }
        }

        public bool IsOffline
        {
            get { return _isOffline; }
            set
            {
                if (value == _isOffline) return;
                _isOffline = value;
                RaisePropertyChanged(nameof(IsOffline));
            }
        }

        public bool IsComParamCheck
        {
            get { return _isComParamCheck; }
            set
            {
                if (value == _isComParamCheck) return;
                _isComParamCheck = value;
                RaisePropertyChanged(nameof(IsComParamCheck));
            }
        }

        public bool IsAdvParamCheck
        {
            get { return _isAdvParamCheck; }
            set
            {
                if (value == _isAdvParamCheck) return;
                _isAdvParamCheck = value;
                RaisePropertyChanged(nameof(IsAdvParamCheck));
            }
        }

        #region Init/Save/Cancel/Close

        protected override void Initialize()
        {
            base.Initialize();

            IsOffline = _discoverIedDialogProperties.IsOffline;
            IsComParamCheck = _discoverIedDialogProperties.IsComParamCheck;
            IsAdvParamCheck = _discoverIedDialogProperties.IsAdvParamCheck;

            foreach (var connectionParam in _discoverIedDialogProperties.IedConnectionParams)
            {
                LastParams.Add(connectionParam);
            }

            //TODO: Set the copy
            //SelectedParamCopy = _discoverIedDialogProperties.LastIedConnectionParam ?? new IedConnectionParam();
        }

        protected override void Close(bool? result)
        {
            //Save dialog settings if dialog was saved
            if (result == true)
            {
                Properties.Settings.Default.Save();
            }

            base.Close(result);
        }

        protected override bool Cancel()
        {
            return base.Cancel();
        }

        protected override bool Save(bool force)
        {
            bool result = base.Save(force);

            _discoverIedDialogProperties.IsComParamCheck = IsComParamCheck;
            _discoverIedDialogProperties.IsAdvParamCheck = IsAdvParamCheck;
            _discoverIedDialogProperties.IsOffline = IsOffline;

            var item = LastParams.FirstOrDefault(x => x.IP_Address == SelectedParamCopy.IP_Address);
            if (item != null)
            {
                _discoverIedDialogProperties.IedConnectionParams.Remove(item);
            }

            //_discoverIedDialogProperties.LastIedConnectionParam = SelectedParamCopy;

            _discoverIedDialogProperties.IedConnectionParams.Insert(0, SelectedParamCopy);

            return result;
        }

        #endregion

        protected override bool Validate(bool force = false, bool notifyChangedPropertiesOnly = true)
        {
            //TODO: Validate SelectedParamCopy
            return SelectedParamCopy != null && !string.IsNullOrEmpty(SelectedParamCopy.IP_Address) && SelectedParamCopy.IP_Address != "0.0.0.0";
            return base.Validate(force, notifyChangedPropertiesOnly);
        }
    }
}



<catel:DataWindow x:Class="Module.IedExplorer.Dialogs.DiscoverIedDialogView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:catel="http://catel.codeplex.com" 
                  xmlns:sys="clr-namespace:System;assembly=mscorlib"
                  xmlns:properties="clr-namespace:Module.IedExplorer.Properties"
                  xmlns:dialogs="clr-namespace:Module.IedExplorer.Dialogs" 
                  xmlns:kivi="http://kivi.dev.ekra.ru"
                  Height="500" Width="700" MinHeight="300" MinWidth="400">

    <catel:DataWindow.Resources>
        <Style x:Key="ExpandCollapseArrowStyle" TargetType="{x:Type ToggleButton}">
            <Setter Property="HorizontalAlignment" Value="Left"/>
            <Setter Property="Focusable" Value="False"/>
            <Setter Property="Width" Value="20"/>
            <Setter Property="Height" Value="16"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="{x:Type ToggleButton}">
                        <ControlTemplate.Resources>
                            <PathGeometry x:Key="TreeArrowPath">
                                <PathGeometry.Figures>
                                    <PathFigureCollection>
                                        <PathFigure IsFilled="True" StartPoint="0 0" IsClosed="True">
                                            <PathFigure.Segments>
                                                <PathSegmentCollection>
                                                    <LineSegment Point="0 6"/>
                                                    <LineSegment Point="6 0"/>
                                                </PathSegmentCollection>
                                            </PathFigure.Segments>
                                        </PathFigure>
                                    </PathFigureCollection>
                                </PathGeometry.Figures>
                            </PathGeometry>
                        </ControlTemplate.Resources>
                        <Border Background="Transparent" Padding="5,5,5,5">
                            <Path x:Name="ExpandPath" Fill="Transparent" Stroke="#FF989898" Data="{StaticResource TreeArrowPath}">
                                <Path.RenderTransform>
                                    <RotateTransform Angle="135" CenterX="3" CenterY="3"/>
                                </Path.RenderTransform>
                            </Path>
                        </Border>
                        <ControlTemplate.Triggers>
                            <Trigger Property="IsChecked" Value="True">
                                <Setter TargetName="ExpandPath" Property="RenderTransform">
                                    <Setter.Value>
                                        <RotateTransform Angle="180" CenterX="3" CenterY="3"/>
                                    </Setter.Value>
                                </Setter>
                                <Setter TargetName="ExpandPath" Property="Fill" Value="#FF595959"/>
                                <Setter TargetName="ExpandPath" Property="Stroke" Value="#FF262626"/>
                            </Trigger>
                            <Trigger Property="IsMouseOver" Value="True">
                                <Setter TargetName="ExpandPath" Property="Stroke" Value="#FF1BBBFA"/>
                                <Setter TargetName="ExpandPath" Property="Fill" Value="Transparent"/>
                            </Trigger>
                        </ControlTemplate.Triggers>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
        <Style x:Key="BorderStyle" TargetType="{x:Type Border}">
            <Setter Property="Margin" Value="0,10,0,4"/>
            <Setter Property="BorderBrush" Value="Black"/>
            <Setter Property="BorderThickness" Value="0,0,0,1"/>
            <Setter Property="Padding" Value="0,0,0,3"/>
            <!--<Setter Property="Height" Value="23"/>-->
        </Style>
        <dialogs:IpAddress2StringConverter x:Key="IpAddress2StringConverter"/>
        <dialogs:AdvParamVisibilityConverter x:Key="AdvParamVisibilityConverter"/>
    </catel:DataWindow.Resources>
    
    <Grid Margin="10">
        <Grid.Resources>
            <Style TargetType="{x:Type TextBox}">
                <Setter Property="Margin" Value="0,0,0,4"/>
                <!--<Setter Property="Height" Value="23"/>-->
            </Style>
        </Grid.Resources>
        <DockPanel LastChildFill="True">
            <Grid DockPanel.Dock="Left" Width="215">
                <Grid.Resources>
                    <Style TargetType="TextBox">
                        <Setter Property="Height" Value="23"/>
                        <Setter Property="VerticalContentAlignment" Value="Center"/>
                        <Setter Property="Margin" Value="0,0,0,2"/>
                    </Style>
                </Grid.Resources>
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition/>
                    <RowDefinition Height="Auto"/>
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="2*"/>
                    <ColumnDefinition Width="3*"/>
                </Grid.ColumnDefinitions>

                <Border Grid.Column="0" Grid.Row="0" Grid.ColumnSpan="2" Style="{StaticResource BorderStyle}">
                    <TextBlock Text="{x:Static properties:Resources.ConnParams}" VerticalAlignment="Center" FontWeight="Bold"/>
                </Border>

                <TextBlock Text="{x:Static properties:Resources.IpAddress_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="1"/>
                <kivi:IPAddressBox SelectedAddress="{Binding SelectedParamCopy.IP_Address, Mode=TwoWay, Converter={StaticResource IpAddress2StringConverter}}" Grid.Column="1" Grid.Row="1"/>
                <!--<TextBox Text="{Binding SelectedItem.IP_Address, Mode=TwoWay, ElementName=ListView}" Grid.Column="1" Grid.Row="1"/>-->

                <Border Grid.Column="0" Grid.Row="2" Grid.ColumnSpan="2" Style="{StaticResource BorderStyle}">
                    <StackPanel Orientation="Horizontal">
                        <ToggleButton x:Name="ComParamSwitch" Style="{StaticResource ExpandCollapseArrowStyle}" IsChecked="{Binding IsComParamCheck}"/>
                        <TextBlock Text="{x:Static properties:Resources.CommonParams}" VerticalAlignment="Center" FontWeight="Bold"/>
                    </StackPanel>
                </Border>

                <Grid Grid.Row="3" Grid.ColumnSpan="2" Visibility="{Binding Path=IsChecked, ElementName=ComParamSwitch, Converter={StaticResource AdvParamVisibilityConverter}}">
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition/>
                    </Grid.RowDefinitions>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="2*"/>
                        <ColumnDefinition Width="3*"/>
                    </Grid.ColumnDefinitions>

                    <TextBlock Text="{x:Static properties:Resources.Port_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="3"/>
                    <TextBox Text="{Binding SelectedParamCopy.Port}" Grid.Column="1" Grid.Row="3"/>

                    <TextBlock Text="{x:Static properties:Resources.Timeout_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="4"/>
                    <TextBox Text="{Binding SelectedParamCopy.Timeout}" Grid.Column="1" Grid.Row="4"/>
                </Grid>

                <Border Grid.Column="0" Grid.Row="4" Grid.ColumnSpan="2" Style="{StaticResource BorderStyle}">
                    <StackPanel Orientation="Horizontal">
                        <ToggleButton x:Name="AdvParamSwitch" Style="{StaticResource ExpandCollapseArrowStyle}" IsChecked="{Binding IsAdvParamCheck}"/>
                        <TextBlock Text="{x:Static properties:Resources.AdvancedParams}" VerticalAlignment="Center" FontWeight="Bold"/>
                    </StackPanel>
                </Border>

                <Grid Grid.Row="5" Grid.ColumnSpan="2" Visibility="{Binding Path=IsChecked, ElementName=AdvParamSwitch, Converter={StaticResource AdvParamVisibilityConverter}}">
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition/>
                    </Grid.RowDefinitions>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="2*"/>
                        <ColumnDefinition Width="3*"/>
                    </Grid.ColumnDefinitions>

                    <TextBlock Text="{x:Static properties:Resources.ApTitle_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="4"/>
                    <TextBox Text="{Binding SelectedParamCopy.AP_Title}" Grid.Column="1" Grid.Row="4"/>

                    <TextBlock Text="{x:Static properties:Resources.AeQualifier_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="5"/>
                    <TextBox Text="{Binding SelectedParamCopy.AE_Qualifier}" Grid.Column="1" Grid.Row="5"/>

                    <TextBlock Text="{x:Static properties:Resources.PSelector_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="6"/>
                    <TextBox Text="{Binding SelectedParamCopy.P_Sel}" Grid.Column="1" Grid.Row="6"/>

                    <TextBlock Text="{x:Static properties:Resources.SSelector_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="7"/>
                    <TextBox Text="{Binding SelectedParamCopy.S_Sel}" Grid.Column="1" Grid.Row="7"/>

                    <TextBlock Text="{x:Static properties:Resources.TSelector_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="8"/>
                    <TextBox Text="{Binding SelectedParamCopy.T_Sel}" Grid.Column="1" Grid.Row="8"/>

                    <TextBlock Text="{x:Static properties:Resources.AeInvokeId_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="9"/>
                    <TextBox Text="{Binding SelectedParamCopy.AE_ID}" Grid.Column="1" Grid.Row="9"/>

                    <TextBlock Text="{x:Static properties:Resources.ApInvokeId_}" VerticalAlignment="Center" Grid.Column="0" Grid.Row="10"/>
                    <TextBox Text="{Binding SelectedParamCopy.AP_ID}" Grid.Column="1" Grid.Row="10"/>
                </Grid>

                <!--<CheckBox IsChecked="{Binding IsOffline}" Content="{x:Static properties:Resources.Offline}" Grid.Column="0" Grid.Row="12"/>-->
            </Grid>
            <Grid Margin="10,0,0,0">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition/>
                </Grid.RowDefinitions>

                <Border Grid.Column="0" Grid.Row="0" Style="{StaticResource BorderStyle}">
                    <TextBlock Text="{x:Static properties:Resources.PrevConn}" VerticalAlignment="Center" FontWeight="Bold"/>
                </Border>
                <ListView x:Name="ListView" Grid.Column="0" Grid.Row="1" ItemsSource="{Binding LastParams}" SelectedIndex="0"
                          SelectedItem="{Binding SelectedParam}" 
                          SelectionMode="Single"
                          VirtualizingStackPanel.ScrollUnit="Pixel"
                          kivi:CommandBindings.CommandBindings="{Binding CommandBindings}">
                    <ListView.ContextMenu>
                        <ContextMenu>
                            <MenuItem Header="{x:Static properties:Resources.ClearRecent}" Command="{Binding ClearRecentCommand}"
                                CommandTarget="{Binding Path=PlacementTarget, RelativeSource={RelativeSource AncestorType=ContextMenu}}" >
                            </MenuItem>
                        </ContextMenu>
                    </ListView.ContextMenu>
                    <ListView.ItemContainerStyle>
                        <Style TargetType="{x:Type ListViewItem}">
                            <Style.Resources>
                                <SolidColorBrush x:Key="{x:Static SystemColors.HighlightBrushKey}" Color="Transparent"/>
                            </Style.Resources>
                            <Setter Property="ContextMenu">
                                <Setter.Value>
                                    <ContextMenu>
                                        <MenuItem Command="{x:Static ApplicationCommands.Delete}">
                                            <!--CommandTarget="{Binding Path=PlacementTarget, RelativeSource={RelativeSource AncestorType=ContextMenu}}"
                                            CommandParameter="{Binding Path=PlacementTarget.SelectedItem, RelativeSource={RelativeSource AncestorType=ContextMenu}}">-->
                                        </MenuItem>
                                    </ContextMenu>
                                </Setter.Value>
                            </Setter>
                            <Style.Triggers>
                                <EventTrigger RoutedEvent="MouseEnter">
                                    <EventTrigger.Actions>
                                        <BeginStoryboard>
                                            <Storyboard>
                                                <DoubleAnimation Duration="0:0:0.300" Storyboard.TargetProperty="FontSize" To="16"/>
                                            </Storyboard>
                                        </BeginStoryboard>
                                    </EventTrigger.Actions>
                                </EventTrigger>
                                <EventTrigger RoutedEvent="MouseLeave">
                                    <EventTrigger.Actions>
                                        <BeginStoryboard>
                                            <Storyboard>
                                                <DoubleAnimation Duration="0:0:0.200" Storyboard.TargetProperty="FontSize" To="12"/>
                                            </Storyboard>
                                        </BeginStoryboard>
                                    </EventTrigger.Actions>
                                </EventTrigger>
                                <EventTrigger RoutedEvent="MouseDoubleClick">
                                    <EventTrigger.Actions>
                                        <dialogs:MyTriggerAction/>
                                    </EventTrigger.Actions>
                                </EventTrigger>
                            </Style.Triggers>
                        </Style>
                    </ListView.ItemContainerStyle>
                    <ListView.ItemTemplate>
                        <DataTemplate>
                            <Grid Margin="10">
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition/>
                                    <ColumnDefinition/>
                                </Grid.ColumnDefinitions>
                                <Grid.RowDefinitions>
                                    <RowDefinition/>
                                    <RowDefinition/>
                                </Grid.RowDefinitions>
                                <Image Grid.RowSpan="2" Source="/Module.ProjectManager;component/Resources/Images/PLC/device.png"/>
                                <TextBlock Text="{Binding IP_Address}" Grid.Column="1" Margin="15,0" FontWeight="SemiBold"/>
                                <TextBlock Text="{Binding IedName}" Grid.Column="1" Grid.Row="1" Margin="15,0"/>
                                <!--<GridViewColumn Width="160" Header="{x:Static properties:Resources.IpAddress}" DisplayMemberBinding="{Binding IP_Address}" />
                                <GridViewColumn Width="220" Header="{x:Static properties:Resources.Description}" DisplayMemberBinding="{Binding IedName}"/>-->
                            </Grid>
                        </DataTemplate>
                    </ListView.ItemTemplate>
                </ListView>
            </Grid>
        </DockPanel>
    </Grid>
</catel:DataWindow>

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Text;
using System.Windows;
using Catel.IO;
using IEC61850.Client;
using Pilot.Shell.Infrastructure.ComponentModel;
using Pilot.Shell.Infrastructure.Logging;
using Pilot.Shell.Infrastructure.Services;
using SCLExpress.Core;
using SCLExpress.DataModel;
using SCLExpress.Project;

namespace Module.IedExplorer
{
    using RM = Properties.Resources;

    public class IedExplorerAsync : PilotService, IDisposable
    {
        private static readonly ILog Log = LogManager.GetCurrentClassLogger();
        private Ookii.Dialogs.Wpf.ProgressDialog _progressDialog;// = new Ookii.Dialogs.Wpf.ProgressDialog();
        private SCLE_Project _project;

        public IedExplorerAsync(SCLE_Project project)
        {
            if (project == null)
                throw new ArgumentNullException("project");

            _project = project;

            _progressDialog = new Ookii.Dialogs.Wpf.ProgressDialog()
            {
                WindowTitle = Application.Current.MainWindow.Title,
                Text = "Discover IED",
                Description = "Processing...",
                ShowTimeRemaining = true,
            };
            _progressDialog.DoWork += ProgressDialogOnDoWork;
            _progressDialog.RunWorkerCompleted += ProgressDialogOnRunWorkerCompleted;
            _progressDialog.ShowCancelButton = true;
            //_progressDialog.Animation = new AnimationResource(@"C:\Imgs\animaLOL.gif", 19411945);
        }

        public event EventHandler<ProgressChangedEventArgs> ProgressChanged;

        public event EventHandler<DiscoverIED_WorkResult> Completed;

        public bool IsBusy
        {
            get { return _progressDialog.IsBusy; }
        }

        #region Discover IED Async

        public bool Start(DiscoverIED_WorkArgs args)
        {
            if (!_progressDialog.IsBusy)
            {
                //TODO:
                _progressDialog.Text = "Discover IED " + args.ConParam.IP_Address;
                _progressDialog.Show(args);

                return true;
            }

            return false;
        }

        private void LongTaskProgressCallback(object sender, ProgressChangedEventArgs e)
        {
            var progressDialog = (Ookii.Dialogs.Wpf.ProgressDialog)sender;

            // cancellation support
            var cancelable = e as CancelableProgressChangedEventArgs;
            if (cancelable != null)
            {
                cancelable.Cancel = progressDialog.CancellationPending;
            }

            int perc = e.ProgressPercentage;
            object state = e.UserState;
            string text = (state == null) ? "Processing..." : state.ToString();
            string desc = string.Format("Progress {0}%", perc);
            //string desc = string.Format(RM.ProcessingProgress_Format, perc);

            progressDialog.ReportProgress(perc, text, desc, state);

            OnProgressChanged(e);
        }

        private void ProgressDialogOnDoWork(object sender, DoWorkEventArgs e)
        {
            var progressDialog = (Ookii.Dialogs.Wpf.ProgressDialog)sender;

            var wa = (DiscoverIED_WorkArgs)e.Argument;

            bool result = false;

            SCLExpress.DataModel.SCL sclInst = null;

            IClient61850 client = null;

            EventHandler<ProgressChangedEventArgs> clientProgressChanged =
                (o, args) => LongTaskProgressCallback(progressDialog, args);

            try
            {
                IClient61850Manager mngr = GetService<IClient61850Manager>();

                //TODO: The Long-Long Task
                client = (IClient61850) mngr.GetClient(wa.ConParam);
                if (client != null)
                {
                    client.ProgressChanged += clientProgressChanged;

#if DEBUG
                    client.IsOffline = wa.IsOffline;
#endif

                    if (client.Connect())
                    {
                        if (client.TryGetScl(out sclInst))
                        {
                            result = true;
                        }
                    }
                    else
                    {
                        Log.Warning("Can't establish connection with {0}.", client.ToString());
                        //TODO: Say that can't connect
                        //Thread.Sleep(500);
                    }
                }
            }
            catch (FileNotFoundException fnfException)
            {
                //TODO?
                IMessageService myMessageService = GetService<IMessageService>();
                myMessageService.ShowError("Program " + Process.GetCurrentProcess().ProcessName + " can't find file: \"iec61850.dll\".");
                Log.Error(fnfException);
            }
            catch (IedConnectionException iedException)
            {
                //TODO?
                IMessageService myMessageService = GetService<IMessageService>();
                myMessageService.ShowError("Error connecting to " + wa.ConParam.IP_Address);
                Log.Error(iedException);
            }
            catch (Exception ex)
            {
                Log.Error("Can't proccess device {0}:{1}", wa.ConParam.IP_Address, wa.ConParam.Port);
                Log.Error(ex.Message);
            }
            finally
            {
                if (client != null)
                {
                    client.ProgressChanged -= clientProgressChanged;
                }

                e.Cancel = progressDialog.CancellationPending;

                // sets result
                e.Result = new DiscoverIED_WorkResult(wa, sclInst, result);
            }
        }

        private void ProgressDialogOnRunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
        {
            var progressDialog = (Ookii.Dialogs.Wpf.ProgressDialog)sender;

            if (e.Cancelled)
            {
                OnCompleted(DiscoverIED_WorkResult.Empty);
                return;
            }

            var wr = (DiscoverIED_WorkResult)e.Result;
            OnCompleted(wr);

            if (wr.Result == false)
            {
                //Log.Warning(RM.Check_LN_Failed, ":(:(:(:(:(:(");
                return;
            }
            // TODO: SHOW DIALOG THERE.

            var ldOldList = new List<string>(new IedLdNamesEnumerable(_project.GetIeds()));

            var ldNewList = new List<string>(new IedLdNamesEnumerable(wr.Conf));

            var ldCommonsList = new List<string>(ldOldList.Intersect(ldNewList));

            // If there were found same LDs.
            if (ldCommonsList.Any())
            {
                var msgService = GetService<IMessageService>();

                var sbWarning = new StringBuilder();

                //Project already exists Logical Device(-s) "LDNames". To add them into project u have to delete them first.
                sbWarning.Append(RM.Project);
                sbWarning.Append(' ');
                if (ldCommonsList.Count == 1)
                {
                    sbWarning.Append(RM.Already_exists_Logical_Device);

                    sbWarning.Append(' ');

                    sbWarning.Append(ldCommonsList.Last());
                }
                else
                {
                    sbWarning.Append(RM.Already_exists_Logical_Devices);

                    sbWarning.Append(' ');

                    // Маааам, я филолололог. Разделяются ", "
                    foreach (var ldName in ldCommonsList)
                    {
                        sbWarning.Append(ldName);

                        if (ldName != ldCommonsList.Last())
                        {
                            sbWarning.Append(", ");
                        }
                    }
                }


                // а последний '.'
                sbWarning.Append(". ");

                sbWarning.Append(RM.To_add_them_into_project_u_have_to_delete_them_first);

                msgService.ShowWarning(sbWarning.ToString());

            }

            using (var importer = new SclImporter_2(_project))
            {
                var from = wr.Args.ConParam.ToString();
                importer.ImportConfiguration(wr.Conf, from);
            }

            //Log.Info(RM.Check_LN_Success, ":):):):):):)");

            //_messageService.ShowInformation(RM.Import_Complete);
        }

        [Serializable]
        public class DiscoverIED_WorkArgs
        {
            public DiscoverIED_WorkArgs(/*SCLExpress.DataModel.SCL conf,*/ IedConnectionParam conParam)
            {
                if (conParam == null)
                    throw new ArgumentNullException("conParam");

                //Conf = conf;
                ConParam = conParam;
            }

            //public SCLExpress.DataModel.SCL Conf { get; private set; }
            public IedConnectionParam ConParam { get; private set; }
            public bool IsOffline { get; set; }
        }

        [Serializable]
        public class DiscoverIED_WorkResult : EventArgs
        {
            public new static readonly DiscoverIED_WorkResult Empty
                = new DiscoverIED_WorkResult() { IsCancelled = true };

            private DiscoverIED_WorkResult()
            {
            }

            public DiscoverIED_WorkResult(DiscoverIED_WorkArgs args, SCLExpress.DataModel.SCL conf, bool? result)
            {
                Args = args;
                Result = result;
                Conf = conf;
            }

            public DiscoverIED_WorkArgs Args { get; private set; }
            public bool IsCancelled { get; private set; }
            public bool? Result { get; private set; }
            public SCLExpress.DataModel.SCL Conf { get; private set; }
        } 

        #endregion

        private void OnProgressChanged(ProgressChangedEventArgs e)
        {
            var handler = ProgressChanged;
            if (handler != null)
                handler(this, e);
        }

        private void OnCompleted(DiscoverIED_WorkResult e)
        {
            var handler = Completed;
            if (handler != null)
                handler(this, e);
        }

        #region IDisposable overrides

        private bool _disposed;

        protected override void Dispose(bool disposing)
        {
            // Check to see if Dispose has already been called.
            if (!this._disposed)
            {
                // If disposing equals true, dispose all managed
                // and unmanaged resources.
                if (disposing)
                {
                    // Dispose managed resources.
                    _progressDialog.DoWork -= ProgressDialogOnDoWork;
                    _progressDialog.RunWorkerCompleted -= ProgressDialogOnRunWorkerCompleted;
                    _progressDialog.Dispose();
                    _progressDialog = null;
                }

                // Clean up unmanaged resources here.
                //CloseHandle(handle);
                //handle = IntPtr.Zero;

                // Note disposing has been done.
                _disposed = true;
            }
        } 

        #endregion
    }
}
