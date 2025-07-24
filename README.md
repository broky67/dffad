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

                    TakeParameterType(platformDescription);

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

        public void TakeParameterType(DeviceDescription platformDescription)
        {
            ParameterType parameterType = null;
            DeviceDescriptionDeviceCollection devices = platformDescription.Device;

            int maxCounter = 20;
            foreach (var device in devices)
            {
                if (device._Name == "[4096] E200-R17xx (1.0.0.0)")
                {
                    object current = device;
                    while (current is _IDeviceDescriptionNode childNode && maxCounter-- > 0)
                    {
                        current = childNode._Children;
                        if (current is ParameterType type)
                        {
                            parameterType = type;
                            parameterType.SetComponent();
                            break;
                        }
                    }
                }
            }
        }


    public interface _IDeviceDescriptionNode
    {
        _IDeviceDescriptionNode _Parent { get; set; }

        string _Name { get; set; }

        IEnumerable<_DeviceDescriptionNode> _Children { get; }

        void ReloadUi();
    }
