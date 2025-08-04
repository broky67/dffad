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
