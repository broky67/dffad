        public TypedefTypeComponentCollection Component
        {
            get
            {
                return this.componentField;
            }
            set
            {
                this.componentField = SetComponent();
                if (componentField != null)
                    componentField._Parent = this;
                this.RaisePropertyChanged("Component");
            }
        }


public void SetComponent(_DeviceDescriptionNode node)
        {
            if (node is ParameterType parameterType)
            {
                var nodeType = node.GetType();
                var nodeTypeName = nodeType.Name;

                var paramTypeField = parameterType.type;

                string prefix = "local";
                bool res = paramTypeField.StartsWith(prefix);
                while (res)
                {
                    var paramTypeFieldWithoutLocal = paramTypeField.Substring(paramTypeField.IndexOf(':') + 1);

                    /*TypedefTypeComponentCollection components;*/

                    if (nodeTypeName is "ParameterType")
                    {
                        DeviceDescription devDescNode = null;
                        object current = node;
                        int maxCounter = 10;
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
                            constStringName = items.FirstOrDefault(x => x._Name.Contains(paramTypeFieldWithoutLocal))._Name;  //"T_R17xx_Delay (StructType)"

                        }
                        catch (Exception ex)
                        {
                            throw new NullReferenceException("Такого имени нет в DescriptionTypes", ex);
                        }


                        var matchingItem = items.FirstOrDefault(x => x._Name == constStringName);

                        if (matchingItem is StructdefType structDef)
                        {
                            parameterType.Component = structDef.Component;
                            /* аналогично для BitfielddefType*/
                            break;
                        }
                        else if (matchingItem is BitfielddefType bitDef)
                        {
                            parameterType.Component = bitDef.Component;
                            /* использовать как новое свойство parametertype как локальную переменную*/
                            /*components = bitDef.Component;*/ /*TypedefTypeComponentCollection components;*/
                            break;
                        }
                    }
                }
            }
        }
