
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

        public ParameterValueType[] DefaultMapping
        {
            get
            {
                return this.defaultMappingField;
            }
            set
            {
                this.defaultMappingField = value;
                this.RaisePropertyChanged("DefaultMapping");
            }
        }


    public partial class ParameterValueType : _DeviceDescriptionNode
    {

        private ValueTypeElement[] elementField;

        private string[] textField;

        /// <remarks/>
        [XmlElement("Element")]
        public ValueTypeElement[] Element
        {
            get
            {
                return this.elementField;
            }
            set
            {
                this.elementField = value;
                this.RaisePropertyChanged("Element");
            }
        }

        /// <remarks/>
        [XmlText]
        public string[] Text
        {
            get
            {
                return this.textField;
            }
            set
            {
                this.textField = value;
                this.RaisePropertyChanged("Text");
            }
        }
    }
    public partial class ParameterValueType
    {
        public override void Accept(PlatformVisitor visitor)
        {
            if (visitor == null)
                throw new ArgumentNullException("visitor");

            visitor.PreviewVisit(this);

            if (elementField != null)
                foreach (var item in elementField)
                    item.Accept(visitor);

            visitor.Visit(this);
        }
    }




public partial class ParameterType : _DeviceDescriptionNode
    {

        private ParameterTypeAttributes attributesField;

        private ParameterValueType[] defaultField;

        private StringRefType nameField;

        private StringRefType unitField;

        private StringRefType descriptionField;

        private string filterFlagsField;

        private ParameterValueType[] defaultMappingField;

        [NonSerialized] private XmlElement[] anyField;

        private uint parameterIdField;

        private string typeField;



