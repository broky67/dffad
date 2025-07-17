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
