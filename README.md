 var constStringName = "T_R17xx_Delay (StructType)";
                var testn = (DeviceDescription)device._Parent._Parent; //e200_new.xml,   gettype - {Name = "DeviceDescription" FullName = "Pilot.TargetPlatform.DeviceDescription"}
                var descriptonTypes = testn.Types;
                var items = descriptonTypes.Items;
                StructdefType result = null;
                foreach (var item in items.Where(x => x._Name == constStringName))
                    result = (StructdefType)item;


                var components = result.Component;
