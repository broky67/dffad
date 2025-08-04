    class ParameterValueConverter : IMultiValueConverter
    {
        public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
        {
            var compCollection= values[0];
            var paramValue = values[1];

            if (paramValue is ParameterValueType[] pvts && pvts.Length > 0)
            {
                if (compCollection is TypedefTypeComponentCollection components)
                {
                    if (components.Count > 0)
                    {   
                        return string.Empty;
                    }
                }

                var firstPvt = pvts[0];
                if (firstPvt.Text != null && firstPvt.Text.Length > 0)
                {
                    return string.Join(";", firstPvt.Text);
                }
                return string.Empty;
            }

            if (paramValue is ParameterValueType pvt)
            {
                if (pvt.Text != null && pvt.Text.Length > 0)
                {
                    return string.Join(";", pvt.Text);
                }
                return string.Empty;
            }

            if (paramValue is ComponentValueType cvt)
            {
                if (cvt.Text != null && cvt.Text.Length > 0)
                {
                    return string.Join(";", cvt.Text);
                }
                return string.Empty;
            }
            return string.Empty;
        }

        public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
        {
            string[] values = null;
            var str = value as string;
            if (!string.IsNullOrEmpty(str))
                return values = str.Split(';');
            return values;
        }
    }

        <TextBlock>
                <TextBlock.Text>
                    <MultiBinding Converter="{StaticResource parameterValueConverter}"  Mode="TwoWay">
                        <Binding Path="Tag.Tag.Component"  Mode="TwoWay"/>
                        <Binding Path="Tag.Tag.Default"  Mode="TwoWay"/>
                    </MultiBinding>
                </TextBlock.Text>
            </TextBlock>
        </DataTemplate>
        <DataTemplate x:Key="DefaultValueEditTemplate">
            <DataTemplate.Resources>
                <local:ParameterValueConverter x:Key="parameterValueConverter"/>
            </DataTemplate.Resources>
            <TextBox>
                <TextBox.Text>
                    <MultiBinding Converter="{StaticResource parameterValueConverter}" Mode="TwoWay">
                        <Binding Path="Tag.Tag.Default"  Mode="TwoWay"/>
                        <Binding Path="Tag.Tag.Default"  Mode="TwoWay"/>
                    </MultiBinding>
                </TextBox.Text>
            </TextBox>




----


protected override bool CanDrop(object dragData, DragEventArgs e)
        {
            // gets the tree node where mouse
            _tlvi = UIHelper.GetItemContainerFromPointInTreeControl<TreeListViewItem>(AssociatedObject, e.GetPosition(AssociatedObject));
            if (_tlvi == null)
                return false;

            _tlviVm = _tlvi.DataContext as SCLE_NodeViewModel;
            if (_tlviVm == null)
                return false;

            var list = dragData as IList;

            if (list != null && list.Count > 0 && list[0] is ISCLE_Node)
            {
                if (!e.AllowedEffects.HasFlag(DragDropEffects.Move))
                    return false;

                if (_tlviVm.Tag is ISCLE_Node node)
                {
                    if (node == null)
                        return false;

                    // check dragged items is not contains in target node
                    if (node.GetNodes(false).Intersect(list.Cast<ISCLE_Node>()).Any())
                        return false;

                    // check all dragged items can be accepted by target node
                    if (!list.Cast<ISCLE_Node>().All(x => node.CanAddNode(x)))
                        return false;

                    if (!(_tlviVm.CanUserAddNode && _tlviVm.CanUserRemoveNode))
                        return false;

                    if (list.Contains(node))
                        return false;
                }

                if (_tlviVm is SCLE_FnAlgorithmViewModel algorithm)
                {
                    if (list[0] is SCLE_ProjectPou pou)
                    {
                        System.Console.WriteLine();
                    }

                    if (!(_tlviVm.CanUserAddNode && _tlviVm.CanUserRemoveNode))
                        return false;

                    /*if (pou != null)
                    {
                        var res = true;
                        *//*var res = algorithm.PouLinks.Any(x => x.Name == pou.Name);*//*
                        return !res;
                    }
                    return false;*/
                }
                return true;
            }

            return HasDropAction(_tlviVm, e, out IDropAction action);
        }


        protected override bool DoDropOperation(object dragData, DragEventArgs e)
        {
            int index = -1;

            // gets index of this tree item
            var parent = VisualHelper.GetVisualParent<ItemsControl>(_tlvi);
            if (parent != null)
                index = parent.ItemContainerGenerator.IndexFromContainer(_tlvi);

            if (index < 0)
                return false;

            // expand folder for load its items
            if (!_tlviVm.IsExpanded)
                _tlviVm.IsExpanded = true;

            var list = dragData as IList;
            if (list != null && list.Count > 0 && list[0] is ISCLE_Node)
            {
                e.Effects = DragDropEffects.Move;

                foreach (var node in list.OfType<ISCLE_Node>())
                {
                    // remove parent (or remove from source collection)
                    node.Parent = null;

                    // add node to project tree
                    ((ISCLE_Node)_tlviVm.Tag).Add(node);
                }
            }

            if (list != null && list.Count > 0 && list[0] is FnAlgorithm)
            {
                e.Effects = DragDropEffects.Move;

                foreach (var node in list.OfType<FnAlgorithm>())
                {
                    // remove parent (or remove from source collection)
                    node.Parent = null;

                    // add node to project tree
                    ((ISCLE_Node)_tlviVm.Tag).Add(node);
                }
            }

            if (HasDropAction(_tlviVm, e, out IDropAction action))
            {
                action.Drop(_tlviVm, e);
            }

            return true;
        }
