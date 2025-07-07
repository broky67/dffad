/*  DeviceDescriptionTypes: 
                componentField = StructedType
        public partial class StructdefType : TypedefType
    {

        private TypedefTypeComponentCollection componentField;

        private string iecTypeField;

        //private string iecTypeLibField;

        public StructdefType()
        {
            componentField = new TypedefTypeComponentCollection(this);
        }

        /// <remarks/>
        [XmlElement("Component")]
        public TypedefTypeComponentCollection Component
        {
            get
            {
                return this.componentField;
            }
            set
            {
                this.componentField = value;
                if (componentField != null)
                    componentField._Parent = this;
                this.RaisePropertyChanged("Component");
            }
        }

        /// <remarks/>
        [XmlAttribute()]
        public string iecType
        {
            get
            {
                return this.iecTypeField;
            }
            set
            {
                this.iecTypeField = value;
                this.RaisePropertyChanged("iecType");
            }
        }

        ///// <remarks/>
        //[XmlAttribute()]
        //public string iecTypeLib
        //{
        //    get
        //    {
        //        return this.iecTypeLibField;
        //    }
        //    set
        //    {
        //        this.iecTypeLibField = value;
        //        this.RaisePropertyChanged("iecTypeLib");
        //    }
        //}

        public override ItemsChoiceType Kind => ItemsChoiceType.StructType;
    }




                
            DeviceDescriprion:
                parameterSectionType
                    else if (node is ParameterSectionType sectionType)
            {
                var sectionTypeName = (sectionType.Name == null) ? sectionType._Name : sectionType.Name.ToString();
                yield return new EditablePropertyItemModel(sectionType)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    //Name = sectionTypeName,
                    GetName = tag => { return (sectionType.Name == null) ? sectionType._Name : sectionType.Name.ToString(); },
                    SetName = (tag, value) => { sectionType.Name = (StringRefType)value; },
                };

                foreach (var item in sectionType.Items)
                {
                    foreach (var pm in GenerateItems(item, indentLevel + 1))
                        yield return pm;
                }
            }
            else if (node is ParameterType parameterType)
            {
                //var pm = new ParameterTypePropertyItemModel(parameterType)
                var pm = new EditablePropertyItemModel(parameterType)
                {
                    IndentLevel = indentLevel,
                    //IsEditableValue = isEditableValue,
                    //IsEditableName = isEditableName,
                    Name = parameterType.Name.ToString(),
                    SetName = (tag, value) => { parameterType.Name = (StringRefType)value; },
                    
                };
                yield return pm;
            }



        public class EditablePropertyItemModel : PropertyItemModel
    {
        public EditablePropertyItemModel(object tag)
            : base(tag)
        {
        }

        public override string Name
        {
            get { return GetName != null ? GetName(Tag) : base.Name; }
            set
            {
                if (SetName != null)
                    SetName(Tag, value);
                base.Name = value;
            }
        }

        public Func<object, string> GetName;
        public Action<object, string> SetName;
    }


                public void UpdateItems(_DeviceDescriptionNode node)
        {
            if (Items != null)
            {
                Items.Clear();
                Items = null;
            }

            if (node != null)
            {
                var items = GenerateItems(node, 0);
                Items = new PropertyItemCollection(items);
                if (Items.Count <= 1)
                    Items.Add(new PropertyItemModel(null)); // add a empty row last for selection purposes
            }
        }




        DeviceDescriptionTypes:
        protected void RaisePropertyChanged([CallerMemberName] string propertyName = "")
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }


                private void MakeSingleSelection(ItemInfo info)
        {
            var data = info.Item;
            var item = info.Container;

            #region Manage SelectedItems
            
            foreach (var info2 in _selectedInfos.ToList())
            {
                if (info2 != info)
                {
                    info2.Container.IsSelected = false;

                    // sync remove
                    _selectedInfos.Remove(info2);
                    SelectedItems.Remove(info2.Item);
                }
            }

            if (!_selectedInfos.Contains(info))
            {
                info.Container.IsSelected = true;

                // sync add
                _selectedInfos.Add(info);
                SelectedItems.Add(info.Item);
            } 

            #endregion

            #region Manage SelectedItem

            var oldValue = SelectedItem;
            var newValue = info.Item;

            if (oldValue != newValue)
            {
                SetSelectedItem(newValue);
                //_selectedContainer = info.Container;
                //SelectedItem = newValue;
                OnSelectedItemChanged(oldValue, newValue);
            } 

            #endregion
        }
         */
