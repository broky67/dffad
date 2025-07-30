public class ParameterValueConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        // Ваша текущая реализация Convert
        // ...
    }

    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        if (value == null)
            return new object[] { null };

        string stringValue = value.ToString();

        // Обработка пустой строки (когда было "0")
        if (string.IsNullOrEmpty(stringValue))
        {
            return new object[] 
            { 
                new ParameterValueType() 
                { 
                    Text = new[] { "0" } // Или null, если нужно сохранять пустоту
                }
            };
        }

        // Обработка обычных значений
        return new object[]
        {
            new ParameterValueType()
            {
                Text = stringValue.Split(new[] { ';' }, StringSplitOptions.RemoveEmptyEntries)
            }
        };
    }
}
