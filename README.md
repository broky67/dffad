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

    public class PropertyItemModel : Mvvm.ObservableObject
    {
        private int _indentLevel;
        private bool _isEditableValue;
        private bool _isEditableName;

        public PropertyItemModel(object tag)
        {
            Tag = tag;
        }

        public int IndentLevel
        {
            get { return _indentLevel; }
            set
            {
                if (value != _indentLevel)
                {
                    _indentLevel = value;
                    RaisePropertyChanged();
                }
            }
        }

        public virtual string Name { get; set; }

        public object Tag { get; set; }

        public bool IsEditableValue
        {
            get { return _isEditableValue; }
            set
            {
                _isEditableValue = value;
                RaisePropertyChanged();
            }
        }

        public bool IsEditableName
        {
            get { return _isEditableName; }
            set
            {
                _isEditableName = value;
                RaisePropertyChanged();
            }
        }
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


private void UpdateItemsSource()
        {
            if (!_isTemplateApplied)
                return;
            if (_items != null)
            {
                _items.ItemPropertyChanged -= Items_ItemPropertyChanged;
                _items.CollectionChanged -= Items_CollectionChanged;
                _items.Dispose();
            }
            if (_originalItems != null)
            {
                _originalItems.CollectionChanged -= OriginalItems_CollectionChanged;
            }
            if (_asyncTimer != null)
            {
                _asyncTimer.Stop();
                _asyncTimer.Tick -= AsyncTimer_Tick;
                _asyncTimer = null;
            }
            _originalItems = Items;
            ItemsSource = null;
            if (_originalItems == null)
            {
                return;
            }
            foreach (DataTreeGridItem current in _originalItems)
            {
                AttachItem(current);
            }
            _items = new DataTreeGridItemCollection();
            if (!IsAsyncPresentationEnabled)
            {
                _asyncItemCount = 0;
                foreach (DataTreeGridItem current2 in _originalItems)
                {
                    _items.Add(current2);
                }
                _asyncItemCount = -1;
            }
            else
            {
                _asyncItemCount = 0;
                int num = Math.Min(_originalItems.Count, IsAsyncPresentationEnabledMinCount);
                for (int i = 0; i < num; i++)
                {
                    _items.Add(_originalItems[i]);
                }
                _asyncItemCount = num;
                if (_asyncTimer == null)
                {
                    _asyncTimer = new DispatcherTimer();
                    _asyncTimer.Tick += AsyncTimer_Tick;
                }
                _asyncTimer.Start();
            }
            _originalItems.CollectionChanged += OriginalItems_CollectionChanged;
            _internalUpdateItemsSourceCount++;
            Dispatcher.BeginInvoke(new ThreadStart(delegate
            {
                if (--this._internalUpdateItemsSourceCount > 0)
                {
                    return;
                }
                this._items.CollectionChanged += this.Items_CollectionChanged;
                this._items.ItemPropertyChanged += this.Items_ItemPropertyChanged;
            }), new object[0]);
            ItemsSource = _items;
            UpdateActualNodeCollapsibility();
        }


    public class ObservableObject : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        protected void RaisePropertyChanged([CallerMemberName] string propertyName = "")
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }

    public class ViewModelBase : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        protected void RaisePropertyChanged([CallerMemberName] string propertyName = "")
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        public virtual void Initialize()
        {

        }

        public virtual void Save()
        {

        }

        public virtual void Cancel()
        {

        }

        public virtual void Close()
        {

        }
    }
