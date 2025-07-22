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

                    if (platformDescription._Name == "e200_new.xml")
                    {
                        foreach (var device in platformDescription.Device)
                        {
                            if (device._Name == "[4096] E200-R17xx (1.0.0.0)")
                            { 
                                foreach (var it in device._Children)
                                {
                                    while (it is _IDeviceDescriptionNode parentNode)
                                    {
                                        
                                    }
                                }
                            }
                        }
                    }
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
