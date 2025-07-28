        public void SetComponent()
        {
            if (this is ParameterType parameterType)
            {
                var paramTypeField = parameterType.type;

                const string prefix = "local:";
                var paramTypeFieldWithoutLocal = paramTypeField.Substring(prefix.Length);
                bool isPrefix = paramTypeField.StartsWith(prefix);
				// to think about complex types that don't exist in DescriptionTypes but exist in DeviceDescription and have local in their name
                while (isPrefix && (paramTypeFieldWithoutLocal == "T_E295x_Threshold" || paramTypeFieldWithoutLocal == "T_R17xx_Delay"))
                {
                    DeviceDescription devDescNode = null;
                    object current = this;

                    int maxCounter = 20;
                    while (current is _IDeviceDescriptionNode parentNode && maxCounter-- > 0)
                    {
                        current = parentNode._Parent;
                        if (current is DeviceDescription desc)
                        {
                            devDescNode = desc;
                            break;
                        }
                    }

                    var descriptonTypes = devDescNode.Types;
                    var items = descriptonTypes.Items;
                    var constStringName = string.Empty;
                    try
                    {
                        constStringName = items.FirstOrDefault(x => x._Name.Contains(paramTypeFieldWithoutLocal))._Name;

                    }
                    catch (Exception ex)
                    {
                        throw new NullReferenceException("Такого имени нет в DescriptionTypes", ex);
                    }

                    var matchingItem = items.FirstOrDefault(x => x._Name == constStringName);
                    var typ = matchingItem.GetType().Name;

                    if (matchingItem is StructdefType structDef)
                    {
                        parameterType.Component = structDef.Component;
                        break;
                    }
                    else if (matchingItem is BitfielddefType bitDef)
                    {
                        parameterType.Component = bitDef.Component;
                        break;
                    }
                }
            }
        }
