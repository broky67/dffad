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
