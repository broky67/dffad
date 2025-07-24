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

                    ParameterType parameterType = null;
                    object current = platformDescription.Device;

                    int maxCounter = 20;
                    while (current is _IDeviceDescriptionNode childNode && maxCounter-- > 0)
                    {
                        current = childNode._Children;
                        var t = current.GetType().Name;
                        if (current is ParameterType type)
                        {
                            parameterType = type;
                            parameterType.SetComponent();
                            break;
                        }
                    }


                    Application.Current.Dispatcher.Invoke(() =>
                    {
                        HwRoot.Add(platformDescription);
                    });

 
                }
            }
            catch (Exception ex)
            {
            }
        }



    public interface _IDeviceDescriptionNode
    {
        _IDeviceDescriptionNode _Parent { get; set; }

        string _Name { get; set; }

        IEnumerable<_DeviceDescriptionNode> _Children { get; }

        void ReloadUi();
    }
