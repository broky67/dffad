        private ObservableCollection<IedFilterItem> _publisherList;
        private List<IedFilterItem> _selectedPublisherList;
        private ObservableCollection<IedFilterItem> _filteredPublishers;
        private Dictionary<IedFilterItem, List<IedFilterItem>> _publisherDic = new Dictionary<IedFilterItem, List<IedFilterItem>>();

        [MementoIgnore]
        public ObservableCollection<IedFilterItem> PublisherList
        {
            get { return _publisherList; }
            set
            {
                _publisherList = value;
                RaisePropertyChanged(nameof(PublisherList));

                //_eventAggregator.GetEvent<IedFilterItemsChangedEvent>()
                //    .Publish(new IedFilterItemsChangedEventArgs(this, _publisherList, _subscriberList));

                SelectedPublisherList = (!_filteredPublishers.Any())
                    ? _publisherList.ToList()
                    : _filteredPublishers.ToList();
            }
        }

        [Browsable(false)]
        public tSubNetwork Subnetwork
        {
            get { return _subNetwork; }
            protected set { _subNetwork = value; }
        }

        //[MementoIgnore]
        public ObservableCollection<IedFilterItem> FilteredPublishers
        {
            get { return _filteredPublishers; }
            set
            {
                if (_filteredPublishers != value)
                {
                    if (_filteredPublishers != null)
                        _filteredPublishers.CollectionChanged -= OnFilteredPublishersSubcribersChanged;

                    _filteredPublishers = value;

                    if (_filteredPublishers != null)
                        _filteredPublishers.CollectionChanged += OnFilteredPublishersSubcribersChanged;
                }
            }
        }
