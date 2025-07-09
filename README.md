    public partial class StructdefType
    {
        public override IEnumerable<_DeviceDescriptionNode> _Children
        {
            get
            {
                if (componentField != null)
                    foreach (_DeviceDescriptionNode item in componentField)
                        yield return item;
            }
        }

        public override void Accept(PlatformVisitor visitor)
        {
            if (visitor == null)
                throw new ArgumentNullException("visitor");

            visitor.PreviewVisit(this);

            if (componentField != null)
                foreach (var item in componentField)
                    item.Accept(visitor);

            visitor.Visit(this);
        }
    }

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


public partial class ParameterType
    {
        public ParameterType()
        {
            typeField = "std:INT";
            defaultField = new[] { new ParameterValueType() { Text = new[] { "0" } } };
        }

        [Bindable(true)]
        [XmlIgnore]
        public string DisplayId
        {
            get { return parameterIdField.ToString(); }
            set
            {
                if (uint.TryParse(value, out parameterIdField))
                    RaisePropertyChanged();
            }
        }

        public override string _Name
        {
            get
            {
                return ToString();
            }
        }

        public override string ToString()
        {
            string str;
            if (nameField == null)
                str = $"[{parameterIdField}] {base._Name}";
            else
                str = $"[{parameterIdField}] {nameField.ToString()}";
            if (typeField != null)
                str += $" : {typeField}";
            if (unitField != null)
                str += $" ({unitField})";
            return str;
        }

        public override void Accept(PlatformVisitor visitor)
        {
            if (visitor == null)
                throw new ArgumentNullException("visitor");

            visitor.PreviewVisit(this);

            if (attributesField != null)
                attributesField.Accept(visitor);

            if (defaultField != null)
                foreach (var item in defaultField)
                    item.Accept(visitor);

            if (nameField != null)
                nameField.Accept(visitor);

            if (unitField != null)
                unitField.Accept(visitor);

            if (descriptionField != null)
                descriptionField.Accept(visitor);

            if (defaultMappingField != null)
                foreach (var item in defaultMappingField)
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

        /// <remarks/>
        public ParameterTypeAttributes Attributes
        {
            get
            {
                return this.attributesField;
            }
            set
            {
                this.attributesField = value;
                this.RaisePropertyChanged("Attributes");
            }
        }

        /// <remarks/>
        [XmlElement("Default")]
        public ParameterValueType[] Default
        {
            get
            {
                return this.defaultField;
            }
            set
            {
                this.defaultField = value;
                this.RaisePropertyChanged("Default");
            }
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

        /// <remarks/>
        public StringRefType Unit
        {
            get
            {
                return this.unitField;
            }
            set
            {
                this.unitField = value;
                this.RaisePropertyChanged("Unit");
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
        [XmlElement(DataType = "NMTOKENS")]
        public string FilterFlags
        {
            get
            {
                return this.filterFlagsField;
            }
            set
            {
                this.filterFlagsField = value;
                this.RaisePropertyChanged("FilterFlags");
            }
        }

        /// <remarks/>
        [XmlElement("DefaultMapping")]
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

        /// <remarks/>
        [XmlAttribute()]
        public uint ParameterId
        {
            get
            {
                return this.parameterIdField;
            }
            set
            {
                this.parameterIdField = value;
                this.RaisePropertyChanged("ParameterId");
            }
        }

        /// <remarks/>
        [XmlAttribute()]
        public string type
        {
            get
            {
                return this.typeField;
            }
            set
            {
                this.typeField = value;
                this.RaisePropertyChanged("type");
            }
        }
    }
