public static class SetComponentService
    {
        public static void TakeParameterType(DeviceDescription platformDescription)
        {
            if (platformDescription?.Device == null) return;

            foreach (var device in platformDescription.Device)
            {
                FindParameterComponents(device);
            }
        }
        public static void FindParameterComponents(_IDeviceDescriptionNode node)
        {
            if (node == null)
                return;
            try
            {
                var children = node._Children;
                if (children != null)
                {
                    foreach (var child in children)
                    {
                        if (child is _DeviceDescriptionFolderNode<DeviceConnector> connectorList)
                        {
                            foreach (var item in connectorList._Children)
                            {
                                FindParameterComponents(item);
                            }
                        }

                        if (child is ParameterSet paramList)
                        {
                            foreach (var item in paramList._Children)
                            {
                                FindParameterComponents(item);
                            }
                        }

                        else
                        {
                            FindParameterComponents(child);
                        }
                    }
                }
                if (node is DeviceConnector connector)
                {
                    foreach (var item in connector.HostParameterSet)
                    {
                        FindParameterComponents(item);
                    }
                }

                if (node is ParameterSet sectionSet)
                {
                    foreach (var item in sectionSet.Items)
                    {
                        FindParameterComponents(item);
                    }
                }

                if (node is DeviceDescriptionParameterSet deviceSet)
                {
                    foreach (var item in deviceSet.Items)
                    {
                        FindParameterComponents(item);
                    }
                }

                if (node is ParameterSectionType sectionType)
                {
                    foreach (var item in sectionType.Items)
                    {
                        FindParameterComponents(item);
                    }
                }
                if (node is ParameterType parameterType)
                {
                    parameterType.SetComponent();
                    return;
                }
            }
            catch
            {
            }
        }
    }


public class Property2ViewModel : Mvvm.ViewModelBase
    {
        private PropertyItemModel _selectedItem;
        private PropertyItemCollection _items;

        [ImportingConstructor]
        public Property2ViewModel()
        {

        }

        public PropertyItemModel SelectedItem
        {
            get => _selectedItem;
            set
            {
                _selectedItem = value;
                RaisePropertyChanged();
            }
        }

        public PropertyItemCollection Items
        {
            get { return _items; }
            private set
            {
                _items = value;
                foreach (var item in _items)
                {
                    if (item.Tag.GetType().Name == "DeviceDescriptionDevice")
                    { 
                        var devDescDevice = item.Tag;

                        if (item.Tag is ParameterType parameterType)
                        {
                            SetComponentService.FindParameterComponents((DeviceDescriptionDevice)devDescDevice);
                        }
                    }

                }
                RaisePropertyChanged();
            }
        }
        public void UpdateItems(_DeviceDescriptionNode node)
        {
            if (Items != null)
            {
                Items.Clear();
                Items = null;
            }

            if (node != null)
            {
                var items = GenerateItems(node, 0);
                Items = new PropertyItemCollection(items);
                if (Items.Count <= 1)
                    Items.Add(new PropertyItemModel(null)); // add a empty row last for selection purposes
            }
        }

        private IEnumerable<PropertyItemModel> GenerateItems(_DeviceDescriptionNode node, int indentLevel)
        {
            if (node is DeviceDescriptionDevice device)
            {
                var editorName = (device.DeviceInfo == null) ? device._Name : device.DeviceInfo.ToString();

                yield return new EditablePropertyItemModel(device)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = editorName,
                    SetName = (tag, value) => { device.DeviceInfo.Name = (StringRefType)value; },
                };

                foreach (var item in device.Connector)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }

                foreach (var item in device.ConnectorGroup)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }

                foreach (var pm in GenerateItems(device.DeviceParameterSet, indentLevel + 1))
                {
                    yield return pm;
                }
            }
            if (node is _DeviceDescriptionFolderNode<DeviceConnector> connectorList)
            {
                var editorName = connectorList._Name;

                yield return new PropertyItemModel(connectorList)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = editorName,
                };

                foreach (var item in connectorList._Children)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
            if (node is DeviceConnectorGroup connectorGroup)
            {
                var editorName = (connectorGroup.Name == null) ? connectorGroup._Name : connectorGroup.Name.ToString();

                yield return new EditablePropertyItemModel(connectorGroup)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = editorName,
                    SetName = (tag, value) => { connectorGroup.Name = (StringRefType)value; },
                };

                foreach (var item in connectorGroup.Items)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
            else if (node is DeviceConnector connector)
            {
                var editorName = (connector.InterfaceName == null) ? connector._Name : connector.InterfaceName.ToString();

                yield return new EditablePropertyItemModel(connector)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = editorName,
                    SetName = (tag, value) => { connector.InterfaceName = (StringRefType)value; },
                };

                foreach (var item in connector.HostParameterSet)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
            else if (node is ParameterSet sectionSet)
            {
                var editorName = (sectionSet.EditorName == null) ? sectionSet._Name : sectionSet.EditorName.ToString();

                yield return new EditablePropertyItemModel(sectionSet)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = editorName,
                    SetName = (tag, value) => { sectionSet.EditorName = (StringRefType)value; },
                };

                foreach (var item in sectionSet.Items)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
            else if (node is _DeviceDescriptionFolderNode<_DeviceDescriptionNode> paramSet)
            {
                var editorName = paramSet._Name;
                yield return new PropertyItemModel(paramSet)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = editorName,
                };

                foreach (var item in paramSet._Children)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
            else if (node is DeviceDescriptionParameterSet deviceSet)
            {
                var editorName = (deviceSet.EditorName == null) ? deviceSet._Name : deviceSet.EditorName.ToString();
                yield return new EditablePropertyItemModel(deviceSet)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = editorName,
                    SetName = (tag, value) => { deviceSet.EditorName = (StringRefType)value; },
                };

                foreach (var item in deviceSet.Items)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
            else if (node is ParameterSectionType sectionType)
            {
                var sectionTypeName = (sectionType.Name == null) ? sectionType._Name : sectionType.Name.ToString();
                yield return new EditablePropertyItemModel(sectionType)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    //Name = sectionTypeName,
                    GetName = tag => { return (sectionType.Name == null) ? sectionType._Name : sectionType.Name.ToString(); },
                    SetName = (tag, value) => { sectionType.Name = (StringRefType)value; },
                };

                foreach (var item in sectionType.Items)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
            else if (node is ParameterType parameterType)
            {
                //var pm = new ParameterTypePropertyItemModel(parameterType)
               /* parameterType.SetComponent();*/
                var pm = new EditablePropertyItemModel(parameterType)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = parameterType.Name.ToString(),
                    SetName = (tag, value) => { parameterType.Name = (StringRefType)value; },
                };
                yield return pm;
            }
            //yield return Enumerable.Empty<TreeGridItemViewModel>();
        }
    }
