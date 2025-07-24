private void FindParameterComponents(_IDeviceDescriptionNode node)
        {
            if (node == null)
                return;

            if (node is ParameterType parameterType)
            {
                parameterType.SetComponent();
                return;
            }

            try
            {
                var children = node._Children ?? Enumerable.Empty<_IDeviceDescriptionNode>();
                foreach (var child in children)
                {
                    if (child is _DeviceDescriptionFolderNode<DeviceConnector> connectorList)
                    {
                        foreach (var item in connectorList._Children)
                        {
                            FindParameterComponents(item);
                        }
                    }
                    else
                        FindParameterComponents(child);
                }

                
            }
            catch
            {
            }
        }
