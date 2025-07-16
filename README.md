            else if (node is ParameterType parameterType)
            {
                var nodeType = node.GetType();
                var nodeTypeName = nodeType.Name;

                var paramTypeField = parameterType.type;
                var paramTypeFieldWithoutLocal = paramTypeField.Substring(paramTypeField.IndexOf(':') + 1);

                TypedefTypeComponentCollection components;

                if (nodeTypeName is "ParameterType")
                {
                    var devDescNode = (DeviceDescription)node._Parent._Parent._Parent._Parent._Parent._Parent._Parent._Parent; //2 for e200rx
                    var descriptonTypes = devDescNode.Types;
                    var items = descriptonTypes.Items;

                    var constStringName = items.FirstOrDefault(x => x._Name.Contains(paramTypeFieldWithoutLocal))._Name;  //"T_R17xx_Delay (StructType)"

                    var matchingItem = items.FirstOrDefault(x => x._Name == constStringName);

                    if (matchingItem is StructdefType structDef)
                    {
                        components = structDef.Component;
                    }
                    else if (matchingItem is BitfielddefType bitDef)
                    {
                        components = bitDef.Component;
                    }
                }
