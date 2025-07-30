namespace Pilot.LanguageModel
{
    public enum PouLinkKind
    {
        Algorithm,
        Settings,
        Measurements,
        IOSignals,
    }

    [Serializable]
    public class FnPouLinkCollection : ProjectNodeCollection<FnPouLink>
    {
        private readonly object _lock = new object();

        protected FnPouLinkCollection()
        {
            //For XML serializer only
        }

        public FnPouLinkCollection(ProjectNode parent)
            : base(parent)
        {
        }

        /// <summary>
        /// Gets the PouLink by name.
        /// </summary>
        /// <param name="name"></param>
        /// <returns>Variable with specified name, or null if not exist.</returns>
        public FnPouLink this[string name]
        {
            get
            {
                lock (_lock)
                {
                    var item = Items.FirstOrDefault(x => x.Name == name);
                    return item;
                }
            }
        }
    }

    [Serializable]
    public abstract class FnBaseContainer : ProjectNode
    {
        protected FnPouLinkCollection _pouLinks;

        [XmlElement("PouLink")]
        public FnPouLinkCollection PouLinks
        {
            get
            {
                if (_pouLinks == null)
                {
                    _pouLinks = new FnPouLinkCollection(this);
                }
                return _pouLinks;
            }
        }

        protected VariableExWorksheetCollection _worksheets;

        [XmlElement("Worksheet")]
        public VariableExWorksheetCollection Worksheets
        {
            get 
            { 
                if (_worksheets == null)
                {
                    _worksheets = new VariableExWorksheetCollection(this);
                }
                return _worksheets; 
            }
        }

        public abstract PouLinkKind Kind { get; }
    }

    [Serializable]
    public class FnAlgorithm : FnBaseContainer
    {
        public FnAlgorithm() 
            : this(string.Empty, string.Empty) { }

        public FnAlgorithm(string name)
            : this(name, string.Empty) { }

        public FnAlgorithm(string name, string desc)
        {
            base.Name = name;
            base.Description = desc;
        }

        public override PouLinkKind Kind { get { return PouLinkKind.Algorithm; } }

        public override void Accept(IProjectNodeVisitor visitor)
        {
            if (visitor == null)
            {
                throw new ArgumentNullException("visitor");
            }
            visitor.PreviewVisitFnAlgorithm(this);
            if (_pouLinks != null)
            {
                foreach (FnPouLink pouLink in _pouLinks)
                {
                    pouLink.Accept(visitor);
                }
            }
            visitor.VisitFnAlgorithm(this);
        }
    }

    [Serializable]
    public class FnSettings : FnBaseContainer
    {
        public FnSettings()
            : this(string.Empty, string.Empty) { }

        public FnSettings(string name)
            : this(name, string.Empty) { }

        public FnSettings(string name, string desc)
        {
            base.Name = name;
            base.Description = desc;
        }

        public override PouLinkKind Kind { get { return PouLinkKind.Settings; } }

        public override void Accept(IProjectNodeVisitor visitor)
        {
            if (visitor == null)
            {
                throw new ArgumentNullException("visitor");
            }
            visitor.PreviewVisitFnSettings(this);
            if (_pouLinks != null)
            {
                foreach (FnPouLink pouLink in _pouLinks)
                {
                    pouLink.Accept(visitor);
                }
            }
            if (_worksheets != null)
            {
                foreach (VariableExWorksheet worksheet in _worksheets)
                {
                    worksheet.Accept(visitor);
                }
            }
            visitor.VisitFnSettings(this);
        }
    }

    [Serializable]
    public class FnMeasurements : FnBaseContainer
    {
        public FnMeasurements()
            : this(string.Empty, string.Empty) { }

        public FnMeasurements(string name)
            : this(name, string.Empty) { }

        public FnMeasurements(string name, string desc)
        {
            base.Name = name;
            base.Description = desc;
        }

        public override PouLinkKind Kind { get { return PouLinkKind.Measurements; } }

        public override void Accept(IProjectNodeVisitor visitor)
        {
            if (visitor == null)
            {
                throw new ArgumentNullException("visitor");
            }
            visitor.PreviewVisitFnMeasurements(this);
            if (_pouLinks != null)
            {
                foreach (FnPouLink pouLink in _pouLinks)
                {
                    pouLink.Accept(visitor);
                }
            }
            if (_worksheets != null)
            {
                foreach (VariableExWorksheet worksheet in _worksheets)
                {
                    worksheet.Accept(visitor);
                }
            }
            visitor.VisitFnMeasurements(this);
        }
    }

    [Serializable]
    public class FnIOSignals : FnBaseContainer
    {
        public FnIOSignals()
            : this(string.Empty, string.Empty) { }

        public FnIOSignals(string name)
            : this(name, string.Empty) { }

        public FnIOSignals(string name, string desc)
        {
            base.Name = name;
            base.Description = desc;
        }

        public override PouLinkKind Kind { get { return PouLinkKind.IOSignals; } }

        public override void Accept(IProjectNodeVisitor visitor)
        {
            if (visitor == null)
            {
                throw new ArgumentNullException("visitor");
            }
            visitor.PreviewVisitFnIOSignals(this);
            if (_worksheets != null)
            {
                foreach (VariableExWorksheet worksheet in _worksheets)
                {
                    worksheet.Accept(visitor);
                }
            }
            visitor.VisitFnIOSignals(this);
        }
    }
}


using System;

namespace Pilot.LanguageModel
{
    [Serializable]
    public sealed class ProgramInstance : ProjectNode
    {
        private string _type;
        private string _fNode;

        public ProgramInstance()
            : this(string.Empty, string.Empty)
        {
        }

        public ProgramInstance(string name, string type)
        {
            base.Name = name;
            _type = type;
        }

        public override void Accept(IProjectNodeVisitor visitor)
        {
            if (visitor == null)
            {
                throw new ArgumentNullException("visitor");
            }
            visitor.PreviewVisitProgramInstance(this);
            // TODO: visit child here
            visitor.VisitProgramInstance(this);
        }

        public string Type
        {
            get { return _type; }
            set { _type = value; RaisePropertyChanged(nameof(Type)); }
        }

        public string FNode
        {
            get { return _fNode; }
            set { _fNode = value; RaisePropertyChanged(nameof(FNode)); }
        }
    }
}

namespace Pilot.LanguageModel
{

    [Serializable]
    public class FNodeBase : ProjectNode
    {
        private DateTime _timeStamp;

        private FnAlgorithm _algorithm;
        private FnSettings _settings;
        private FnMeasurements _measurements;
        private FnIOSignals _ioSingals;
        private FnConnections _connections;

        public FNodeBase()
            : this(string.Empty) { }

        public FNodeBase(string name)
        {
            base.Name = name;
            _timeStamp = DateTime.Now;

            Algorithm = new FnAlgorithm("Algorithm");
            Settings = new FnSettings("Settings");
            Measurements = new FnMeasurements("Measurements");
            IOSignals = new FnIOSignals("IOSignals");
            Connections = new FnConnections("Connections");
        }

        public FnAlgorithm Algorithm
        {
            get { return _algorithm; }
            set
            {
                if (value != _algorithm)
                {
                    if (_algorithm != null)
                        _algorithm.Parent = null;

                    _algorithm = value;

                    if (_algorithm != null)
                        _algorithm.Parent = this;

                    RaisePropertyChanged("Algorithm");
                }
            }
        }

        public FnSettings Settings
        {
            get { return _settings; }
            set
            {
                if (value != _settings)
                {
                    if (_settings != null)
                        _settings.Parent = null;

                    _settings = value;

                    if (_settings != null)
                        _settings.Parent = this;

                    RaisePropertyChanged("Settings");
                }
            }
        }

        public FnMeasurements Measurements
        {
            get { return _measurements; }
            set
            {
                if (value != _measurements)
                {
                    if (_measurements != null)
                        _measurements.Parent = null;

                    _measurements = value;

                    if (_measurements != null)
                        _measurements.Parent = this;

                    RaisePropertyChanged("Measurements");
                }
            }
        }

        public FnIOSignals IOSignals
        {
            get { return _ioSingals; }
            set
            {
                if (value != _ioSingals)
                {
                    if (_ioSingals != null)
                        _ioSingals.Parent = null;

                    _ioSingals = value;

                    if (_ioSingals != null)
                        _ioSingals.Parent = this;

                    RaisePropertyChanged("IOSignals");
                }
            }
        }

        public FnConnections Connections
        {
            get { return _connections; }
            set
            {
                if (value != _connections)
                {
                    if (_connections != null)
                        _connections.Parent = null;

                    _connections = value;

                    if (_connections != null)
                        _connections.Parent = this;

                    RaisePropertyChanged("Connections");
                }
            }
        }
        public override void Accept(IProjectNodeVisitor visitor)
        {
            if (_algorithm != null)
                _algorithm.Accept(visitor);

            if (_settings != null)
                _settings.Accept(visitor);

            if (_measurements != null)
                _measurements.Accept(visitor);

            if (_ioSingals != null)
                _ioSingals.Accept(visitor);

            if (_connections != null)
                _connections.Accept(visitor);
        }
    }
}

 class Drag : DragBehaviorBase<TreeListView, LibraryViewModel>
    {
        protected override bool CanDrag(object dragData)
        {
            if (dragData is DeviceDescriptionDevice)
                return true;

            //return false;
            return base.CanDrag(dragData);
        }

        protected override IDataObject CombineDragData(object dragData)
        {
            return base.CombineDragData(dragData);
        }

        protected override void OnDragStarted(object dragData)
        {
            base.OnDragStarted(dragData);
        }
    }

namespace Module.ProjectManager.DragDrop
{
    class Drop : DropBehaviorBase<TreeListView, ProjectViewModel>
    {
        private TreeListViewItem _tlvi;
        private DropController _dropctrl;

        public Drop()
        {
            _dropctrl = new DropController(null);
        }

        public string Message
        {
            get { return (string)GetValue(MessageProperty); }
            set { SetValue(MessageProperty, value); }
        }

        // Using a DependencyProperty as the backing store for Message.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty MessageProperty =
            DependencyProperty.Register("Message", typeof(string), typeof(Drop), new PropertyMetadata(default(string)));

        protected override bool CanDrop(object dragData, DragEventArgs e)
        {
            // gets the tree node where mouse
            _tlvi = UIHelper.GetItemContainerFromPointInTreeControl<TreeListViewItem>(AssociatedObject, e.GetPosition(AssociatedObject));
            if (_tlvi == null)
                return false;

            var node = _tlvi.DataContext as _DeviceDescriptionNode;
            var childDev = dragData as DeviceDescriptionDevice;
            if (node != null && childDev != null)
            {
                bool res = _dropctrl.CanDrop(node, childDev);

                Message = _dropctrl.Message;

                return res;
            }

            return base.CanDrop(dragData, e);
        }

        protected override DropAdorner OnInitializeDropAdorner(DragEventArgs e)
        {
            var adornerLayer = AdornerLayer.GetAdornerLayer(AssociatedObject);
            if (adornerLayer != null && _tlvi != null)
            {
                var height = _tlvi.IsExpanded ? CalculateHeight(_tlvi) : _tlvi.ActualHeight;
                var size = new Size(_tlvi.ActualWidth, height);

                var dropAdorner = new MyDropAdorner(_tlvi, adornerLayer, size);
                return dropAdorner;
            }

            // some errors occured
            return null;
        }

        private double CalculateHeight(TreeListViewItem tlvi)
        {
            double totalH = tlvi.ActualHeight;
            foreach (var item in tlvi.Items)
            {
                var container = tlvi.ItemContainerGenerator.ContainerFromItem(item) as TreeListViewItem;
                if (container != null)
                    totalH -= container.ActualHeight;
            }
            Debug.Assert(totalH > 0);
            return totalH;
        }

        class MyDropAdorner : DropAdorner
        {
            private Size _size;

            #region ctor

            /// <summary>
            /// Initializes a new instance of the <see cref="MyDropAdorner"/> class.
            /// </summary>
            /// <param name="adornedElement"></param>
            /// <param name="adornerLayer"></param>
            /// <param name="size"></param>
            public MyDropAdorner(UIElement adornedElement, AdornerLayer adornerLayer, Size size)
                : base(adornedElement, adornerLayer)
            {
                _size = size;
            }

            #endregion

            protected override Point[] DetermineRectPoints(double width, double height, double padding = 0)
            {
                return base.DetermineRectPoints(_size.Width, _size.Height);
            }
        }

        protected override bool DoDropOperation(object dragData, DragEventArgs e)
        {
            if (_tlvi == null)
                return false;

            var node = _tlvi.DataContext as _DeviceDescriptionNode;
            var childDev = dragData as DeviceDescriptionDevice;
            if (node != null && childDev != null)
            {
                if (!ObjectCloner.TryClone(childDev, out DeviceDescriptionDevice childDevCopy))
                {
                    return false;
                }

                childDevCopy.ModuleId = Guid.NewGuid().ToString();

                bool needAddToDevList = _dropctrl.DoDropOperation(node, childDevCopy);
                if (needAddToDevList)
                {
                    // add device to the deviceList;
                    var deviceList = ((DeviceDescription)AssociatedViewModel.ProjRoot[0]).Device;
                    if (!deviceList.Contains(childDevCopy))
                        deviceList.Add(childDevCopy);

                    node.ReloadUi();
                }

                return needAddToDevList;
            }

            return base.DoDropOperation(dragData, e);
        }
    }

    static class ObjectCloner
    {
        public static bool TryClone<TObj>(TObj obj, out TObj clone)
        {
            try
            {
                var stream = new System.IO.MemoryStream();
                var serializer = new System.Runtime.Serialization.Formatters.Binary.BinaryFormatter();
                serializer.Serialize(stream, obj);
                stream.Position = 0L;
                clone = (TObj)serializer.Deserialize(stream);
                stream.Close();
                return true;
            }
            catch (Exception ex)
            {
                clone = default(TObj);
                return false;
            }
        }
    }
}

namespace Module.ProjectManager.DragDrop
{
    public class DropController
    {
        private DeviceDescription _project;

        public DropController(DeviceDescription project)
        {
            _project = project;
        }

        public string Message { get; set; }

        public bool CanDrop(_DeviceDescriptionNode node, DeviceDescriptionDevice childDev)
        {
            if (node == null || childDev == null)
                return false;

            var childCon = childDev.Connectors.Where(x => x.role == ConnectorTypeRole.child).ToList();

            if (node is _DeviceDescriptionFolderNode<DeviceDescriptionDevice> list)
            {
                foreach (var ccon in childCon)
                {
                    foreach (var adapter in ccon.Adapter)
                    {
                        if (adapter is ConnectorTypeFixed adapterFixed)
                        {
                            return false;
                        }
                        else if (adapter is ConnectorTypeSlot adapterSlot)
                        {
                            if (!adapterSlot.allowEmpty)
                                return false;
                        }
                        else if (adapter is ConnectorTypeVar adapterVar)
                        {
                            return false;
                        }
                        break; // only 1 adapter supported;
                    }
                }
                return true;
            }
            else if (node is ConnectorTypeFixed connectorTypeFixed ||
                node is ConnectorTypeSlot connectorTypeSlot ||
                node is ConnectorTypeVar connectorTypeVar)
            {
                var con = (DeviceConnector)node._Parent._Parent;
                return IsCompatible(con, childDev, out DeviceConnector chCon);
            }
            else if (node is DeviceConnector connector)
            {
                var con = connector;
                return IsCompatible(con, childDev, out DeviceConnector chCon);
            }
            else if (node is DeviceDescriptionDevice device)
            {
                bool isAllCompatible = childCon.Any();
                var parentCon = device.Connectors.Where(x => x.role == ConnectorTypeRole.parent).ToList();

                foreach (var ccon in childCon)
                {
                    bool isCompatible = false;
                    DeviceRefType[] allowed;
                    bool allowEmpty = false;
                    int max = 0;

                    foreach (var adapter in ccon.Adapter)
                    {
                        if (adapter is ConnectorTypeFixed adapterFixed)
                        {
                            allowed = adapterFixed.Module;
                        }
                        else if (adapter is ConnectorTypeSlot adapterSlot)
                        {
                            allowEmpty = adapterSlot.allowEmpty;
                            max = adapterSlot.count;
                        }
                        else if (adapter is ConnectorTypeVar adapterVar)
                        {
                            max = adapterVar.max;
                        }
                        break; // only 1 adapter supported;
                    }

                    foreach (var pcon in parentCon)
                    {
                        if (pcon.@interface == ccon.@interface &&
                            pcon.role == ConnectorTypeRole.parent &&
                            ccon.role == ConnectorTypeRole.child)
                        {
                            isCompatible = true; // ok

                            /*foreach (var adapter in pcon.Adapter)
                            {
                                if (adapter is ConnectorTypeFixed adapterFixed)
                                {
                                    allowed = adapterFixed.Module;
                                }
                                else if (adapter is ConnectorTypeSlot adapterSlot)
                                {
                                    allowEmpty = adapterSlot.allowEmpty;
                                    max = adapterSlot.count;
                                }
                                else if (adapter is ConnectorTypeVar adapterVar)
                                {
                                    max = adapterVar.max;
                                }
                                break; // only 1 adapter supported;
                            }*/

                            break;
                        }
                        else
                        {
                            //isCompatible = false;
                        }
                    }

                    isAllCompatible &= (isCompatible || allowEmpty);
                }
                return isAllCompatible;
            }

            return false;
        }

        private bool IsAnyCompatible(DeviceConnector parentCon, IEnumerable<DeviceConnector> childCon)
        {
            bool res = false;

            foreach (var child in childCon)
            {
                if (parentCon.@interface == child.@interface &&
                    parentCon.role == ConnectorTypeRole.parent &&
                    child.role == ConnectorTypeRole.child)
                {
                    res = true; // ok
                    break;
                }
            }

            return res;
        }

        private bool IsCompatible(DeviceConnector parentCon, DeviceDescriptionDevice childDev, out DeviceConnector connector)
        {
            bool res = false;
            connector = null;

            foreach (var item in childDev.Connector)
            {
                if (parentCon.@interface == item.@interface &&
                    parentCon.role == ConnectorTypeRole.parent &&
                    item.role == ConnectorTypeRole.child)
                {
                    connector = item;
                    res = true; // ok
                    break;
                }
            }

            foreach (var g in childDev.ConnectorGroup)
            {
                foreach (var item in g.Items)
                {
                    if (parentCon.@interface == item.@interface &&
                        parentCon.role == ConnectorTypeRole.parent &&
                        item.role == ConnectorTypeRole.child)
                    {
                        connector = item;
                        res = true; // ok
                        break;
                    }
                }
            }

            if (res)
                Message = $"";
            else
                Message = $"{childDev._Name} is not compatible with interface {parentCon.@interface}";

            return res;
        }

        public bool DoDropOperation(_DeviceDescriptionNode node, DeviceDescriptionDevice childDev)
        {
            if (node == null || childDev == null)
                return false;

            bool needAddToDevList = false;
            var childCon = childDev.Connectors.Where(x => x.role == ConnectorTypeRole.child).ToList();

            //TODO DeviceDescriptionDeviceCollection is not _DeviceDescriptionNode :(
            //if (node is DeviceDescriptionDeviceCollection list)
            if (node is _DeviceDescriptionFolderNode<DeviceDescriptionDevice> list)
            {
                // add device to the deviceList
                list.Add(childDev);
            }
            else if (node is ConnectorTypeFixed connectorTypeFixed ||
                node is ConnectorTypeSlot connectorTypeSlot ||
                node is ConnectorTypeVar connectorTypeVar)
            {
                var parentCon = (DeviceConnector)node._Parent._Parent;
                if (IsCompatible(parentCon, childDev, out DeviceConnector chCon))
                {
                    needAddToDevList = AddDeviceToConnector(parentCon, childDev);

                    // add cross-reference to parent device too
                    var parentDev = (DeviceDescriptionDevice)parentCon._Parent._Parent;
                    AddDeviceToConnector(chCon, parentDev);
                }
            }
            else if (node is DeviceConnector connector)
            {
                if (IsCompatible(connector, childDev, out DeviceConnector chCon))
                {
                    needAddToDevList = AddDeviceToConnector(connector, childDev);

                    // add cross-reference to parent device too
                    var parentDev = (DeviceDescriptionDevice)connector._Parent._Parent;
                    AddDeviceToConnector(chCon, parentDev);
                }
            }
            else if (node is DeviceDescriptionDevice device)
            {
                needAddToDevList = false;
                foreach (var parentCon in device.Connectors)
                {
                    if (IsCompatible(parentCon, childDev, out DeviceConnector chCon))
                    {
                        needAddToDevList |= AddDeviceToConnector(parentCon, childDev);

                        // add cross-reference to parent device too
                        var parentDev = device;
                        AddDeviceToConnector(chCon, parentDev);
                    }
                }
            }

            /*if (needAddToDevList)
            {
                // add device to the deviceList;
                var deviceList = _project.Device;
                if (!deviceList.Contains(childDev))
                    deviceList.Add(childDev);

                node.ReloadUi();
            }*/

            return needAddToDevList;
        }

        private bool AddDeviceToConnector(DeviceConnector parentCon, DeviceDescriptionDevice childDev)
        {
            bool needAddToDevList = false;

            foreach (var adapter in parentCon.Adapter)
            {
                if (adapter is ConnectorTypeFixed adapterFixed)
                {
                    var l = (adapterFixed.Module ?? new DeviceRefType[] { }).ToList();
                    l.Add(new DeviceRefType() { Item = childDev.ModuleId, RefDevice = childDev });
                    adapterFixed.Module = l.ToArray();
                    needAddToDevList = true;
                }
                else if (adapter is ConnectorTypeSlot adapterSlot)
                {
                    var l = (adapterSlot.DefaultModule ?? new DeviceRefType[] { }).ToList();
                    l.Add(new DeviceRefType() { Item = childDev.ModuleId, RefDevice = childDev });
                    adapterSlot.DefaultModule = l.ToArray();
                    needAddToDevList = true;
                }
                else if (adapter is ConnectorTypeVar adapterVar)
                {
                    var l = (adapterVar.VarModule ?? new DeviceRefType[] { }).ToList();
                    l.Add(new DeviceRefType() { Item = childDev.ModuleId, RefDevice = childDev });
                    adapterVar.VarModule = l.ToArray();
                    needAddToDevList = true;
                }
                break; // only 1 adapter supported;
            }

            return needAddToDevList;
        }
    }
}

class DragDropTemplateSelector : DataTemplateSelector
    {
        public DataTemplate Default { get; set; }
        public DataTemplate DeviceDescriptionDevice { get; set; }

        public override DataTemplate SelectTemplate(object item, DependencyObject container)
        {
            if (item is DeviceDescriptionDevice)
                return DeviceDescriptionDevice;

            return Default ?? base.SelectTemplate(item, container);
        }
    }
