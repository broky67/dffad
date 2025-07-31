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

                var node = _tlviVm.Tag as ISCLE_Node;
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
   
                return true;
            }

            return HasDropAction(_tlviVm, e, out IDropAction action);
        }
