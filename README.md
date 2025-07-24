{ Pilot.TargetPlatform.ParameterSectionType.< get__Children > d__20} - обрабатывает правильно
             {Pilot.TargetPlatform._DeviceDescriptionFolderNode<Pilot.TargetPlatform.DeviceConnector>} - children count = 2, но при этом отрабатывает этот участок кода else if (node._Children == null)
            {
                Enumerable.Empty<_IDeviceDescriptionNode>();
            }

private void FindAndSetParameterComponents(_IDeviceDescriptionNode node)
        {
            if (node == null)
                return;

            // Проверяем текущий узел
            if (node is ParameterType parameterType)
            {
                parameterType.SetComponent();
                return;
            }
            /*{ Pilot.TargetPlatform.ParameterSectionType.< get__Children > d__20} - обрабатывает правильно
             {Pilot.TargetPlatform._DeviceDescriptionFolderNode<Pilot.TargetPlatform.DeviceConnector>} - children count = 2, но при этом отрабатывает этот участок кода else if (node._Children == null)
            {
                Enumerable.Empty<_IDeviceDescriptionNode>();
            }
             */
            // Рекурсивно проверяем детей
            if (node._Children != null)
            { 
                foreach (var child in node._Children)
                {
                    FindAndSetParameterComponents(child);
                }
            }
            else if (node._Children == null)
            {
                Enumerable.Empty<_IDeviceDescriptionNode>();
            }
        }
