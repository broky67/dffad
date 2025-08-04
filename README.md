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
                if (compCollection is ComponentValueType)
                {
                    Console.WriteLine();
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
/*
            if (compValue is ComponentValueType cvt)
            {
                if (cvt.Text != null && cvt.Text.Length > 0)
                {
                    return string.Join(";", cvt.Text);
                }
                return string.Empty;
            }*/

            return string.Empty;
        }

        public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
        {
            var str = value as string;
            if (!string.IsNullOrEmpty(str))
            {
                return new object[]
                {
                    new ComponentValueType() { Text = str.Split(';') },
                    new ParameterValueType() { Text = str.Split(';') }
                };
            }
            return null;
        }
    }


                <TextBox.Text>
                    <MultiBinding Converter="{StaticResource parameterValueConverter}"
                                  Mode="OneWay">
                        <Binding Path="Tag.Tag.Component"/>
                        <Binding Path="Tag.Tag.Default"/>
                    </MultiBinding>
                </TextBox.Text>





        public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
        {
            var str = value as string;
            if (!string.IsNullOrEmpty(str))
            {
                return new object[]
                {     
                    new ParameterValueType() { Text = str.Split(';') }      
                };
            }
            return null;
        }


Проблема возникает из-за конфликта типов между `FnAlgorithm` и `ISCLE_Node`. Вот исправленная версия метода `CanDrop` с правильной проверкой типов:

```csharp
protected override bool CanDrop(object dragData, DragEventArgs e)
{
    // Получаем элемент дерева под курсором
    _tlvi = UIHelper.GetItemContainerFromPointInTreeControl<TreeListViewItem>(
        AssociatedObject, e.GetPosition(AssociatedObject));
    if (_tlvi == null)
        return false;

    _tlviVm = _tlvi.DataContext as SCLE_NodeViewModel;
    if (_tlviVm == null)
        return false;

    var list = dragData as IList;

    // Специальная обработка для Algorithm
    if (_tlviVm is SCLE_FnAlgorithmViewModel algorithmVm)
    {
        var pou = (list != null && list.Count == 1) 
            ? list[0] as SCLE_ProjectPou 
            : dragData as SCLE_ProjectPou;

        if (pou != null)
        {
            // Проверяем что POU еще не добавлен в Algorithm
            return !algorithmVm.PouLinks.Any(x => x.Name == pou.Name);
        }
        return false;
    }

    // Оригинальная логика для ISCLE_Node
    if (list != null && list.Count > 0 && list[0] is ISCLE_Node)
    {
        if (!e.AllowedEffects.HasFlag(DragDropEffects.Move))
            return false;

        if (!(_tlviVm.Tag is ISCLE_Node node))
            return false;

        // Проверяем что перетаскиваемые элементы не содержатся в целевом узле
        if (node.GetNodes(false).Intersect(list.Cast<ISCLE_Node>()).Any())
            return false;

        // Проверяем что целевой узел может принять все перетаскиваемые элементы
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
```

Ключевые изменения:

1. Разделил проверки для `SCLE_FnAlgorithmViewModel` и `ISCLE_Node` на два отдельных блока
2. Проверяю тип ViewModel перед проверкой `ISCLE_Node`:
   ```csharp
   if (_tlviVm is SCLE_FnAlgorithmViewModel algorithmVm)
   {
       // Обработка для Algorithm
   }
   ```
3. Убрал преобразование `Tag` к `ISCLE_Node` для случая с Algorithm

4. Сохранил всю оригинальную логику для `ISCLE_Node`

Дополнительно нужно убедиться, что:

1. `SCLE_FnAlgorithmViewModel` действительно представляет `FnAlgorithm`
2. `SCLE_ProjectPou` соответствует вашей модели Program (POU)
3. `PouLinks` в `SCLE_FnAlgorithmViewModel` содержит корректные данные

Если `FnAlgorithm` должен реализовывать `ISCLE_Node`, нужно:
1. Добавить интерфейс к классу:
   ```csharp
   public class FnAlgorithm : ISCLE_Node
   {
       // Реализация методов интерфейса
   }
   ```
2. Или создать адаптер, который будет реализовывать `ISCLE_Node` для `FnAlgorithm`

Такой подход обеспечит:
- Отсутствие ошибок приведения типов
- Четкое разделение логики для разных типов узлов
- Сохранение всей существующей функциональности
-
-
-
-
-
-
-
-
-
-   protected override bool CanDrop(object dragData, DragEventArgs e)
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

                if (_tlviVm.Tag is ISCLE_Node node)
                {
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
                }

                if (_tlviVm is SCLE_FnAlgorithmViewModel algorithm)
                {
                    var pou = (list != null && list.Count == 1) ? list[0] as SCLE_ProjectPou : dragData as SCLE_ProjectPou;

                    if (pou != null)
                    {
                        var res = false;
                        /*var res = algorithm.PouLinks.Any(x => x.Name == pou.Name);*/
                        return !res;
                    }
                    return false;
                }
                return true;
            }

            return HasDropAction(_tlviVm, e, out IDropAction action);
        }



Вот модифицированная версия метода `CanDrop` с добавлением обработки для добавления Program (Pou) в Algorithm:

```csharp
protected override bool CanDrop(object dragData, DragEventArgs e)
{
    // Получаем элемент дерева под курсором
    _tlvi = UIHelper.GetItemContainerFromPointInTreeControl<TreeListViewItem>(
        AssociatedObject, e.GetPosition(AssociatedObject));
    if (_tlvi == null)
        return false;

    _tlviVm = _tlvi.DataContext as SCLE_NodeViewModel;
    if (_tlviVm == null)
        return false;

    var list = dragData as IList;

    // Обработка перетаскивания Program в Algorithm
    if (_tlviVm.Tag is FnAlgorithm algorithm)
    {
        // Проверяем что перетаскиваем Program или список Programs
        var program = (list != null && list.Count == 1) ? list[0] as ProgramInstance : dragData as ProgramInstance;
        
        if (program != null)
        {
            // Проверяем что Program еще не добавлен в Algorithm
            return !algorithm.PouLinks.Any(l => l.Name == program.Name);
        }
        return false;
    }

    // Оригинальная логика для других случаев
    if (list != null && list.Count > 0 && list[0] is ISCLE_Node)
    {
        if (!e.AllowedEffects.HasFlag(DragDropEffects.Move))
            return false;

        var node = _tlviVm.Tag as ISCLE_Node;
        if (node == null)
            return false;

        // Проверяем что перетаскиваемые элементы не содержатся в целевом узле
        if (node.GetNodes(false).Intersect(list.Cast<ISCLE_Node>()).Any())
            return false;

        // Проверяем что целевой узел может принять все перетаскиваемые элементы
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
```

Ключевые изменения:

1. Добавлена специальная обработка для случая, когда цель - `FnAlgorithm`:
   ```csharp
   if (_tlviVm.Tag is FnAlgorithm algorithm)
   {
       var program = (list != null && list.Count == 1) ? list[0] as ProgramInstance : dragData as ProgramInstance;
       if (program != null)
       {
           return !algorithm.PouLinks.Any(l => l.Name == program.Name);
       }
       return false;
   }
   ```

2. Особенности реализации:
   - Поддерживает как перетаскивание одиночного `ProgramInstance`, так и списка программ
   - Проверяет, что программа еще не добавлена в алгоритм
   - Сохраняет всю оригинальную логику для других случаев

3. Совместимость:
   - Работает с существующей системой перетаскивания
   - Не нарушает текущие проверки для других типов узлов
   - Сохраняет все оригинальные условия проверки

Для полной реализации вам также нужно обновить метод `DoDropOperation`:

```csharp
protected override bool DoDropOperation(object dragData, DragEventArgs e)
{
    if (_tlviVm?.Tag is FnAlgorithm algorithm)
    {
        var program = (dragData is IList list && list.Count == 1) ? 
            list[0] as ProgramInstance : 
            dragData as ProgramInstance;

        if (program != null)
        {
            // Создаем и добавляем ссылку на программу
            var pouLink = new FnPouLink(program.Name)
            {
                Type = program.Type,
                FNode = program.FNode
            };
            algorithm.PouLinks.Add(pouLink);
            return true;
        }
    }

    // Оригинальная логика для других случаев
    // ...
}
```


-
-
-
-
-
-
-                                 protected override bool CanDrop(object dragData, DragEventArgs e)
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
