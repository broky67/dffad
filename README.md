public partial class ParameterSectionType : _DeviceDescriptionNode
    {

        private StringRefType nameField;

        private StringRefType descriptionField;

        [NonSerialized] private XmlElement[] anyField;

        private DeviceDescriptionNodeCollection itemsField;

        public ParameterSectionType()
        {
            itemsField = new DeviceDescriptionNodeCollection(this);
        }

        /// <remarks/>
        public StringRefType Name
        {
            get
            {
                return this.nameField;
            }
            set
            {
                this.nameField = value;
                this.RaisePropertyChanged("Name");
            }
        }

        //component
        private TypedefTypeComponentCollection componentField;

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
        public StringRefType Description
        {
            get
            {
                return this.descriptionField;
            }
            set
            {
                this.descriptionField = value;
                this.RaisePropertyChanged("Description");
            }
        }

        /// <remarks/>
        [XmlElement("Parameter", typeof(ParameterType))]
        [XmlElement("ParameterSection", typeof(ParameterSectionType))]
        public DeviceDescriptionNodeCollection Items
        {
            get
            {
                return this.itemsField;
            }
            set
            {
                this.itemsField = value;
                if (this.itemsField != null)
                    this.itemsField._Parent = this;
                this.RaisePropertyChanged("Items");
            }
        }

        /// <remarks/>
        [XmlAnyElement]
        public XmlElement[] Any
        {
            get
            {
                return this.anyField;
            }
            set
            {
                this.anyField = value;
                this.RaisePropertyChanged("Any");
            }
        }
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
