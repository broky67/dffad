private static void SetComponents(DeviceDescription device, string typeName, ParameterType parameterType)
        {
            var matchingItem = device.Types.Items.FirstOrDefault(item => item._Name.Contains(typeName));

            if (matchingItem is StructdefType structDef)
            {
                parameterType.Component = structDef.Component;
            }
            else if (matchingItem is BitfielddefType bitDef)
            {
                parameterType.Component = bitDef.Component;
            }
        }

 public static class CollectionExtension
    {
        //public static void AddRange(this IList collection, IEnumerable items)
        //{
        //    foreach (var item in items)
        //    {
        //        collection.Add(item);
        //    }
        //}

        public static void AddRange<TItem>(this IList<TItem> collection, IEnumerable<TItem> items)
        {
            foreach (var item in items)
            {
                collection.Add(item);
            }
        }

        public static void MergeRange<TItem>(this IList<TItem> collection, IEnumerable<TItem> items, bool skipDuplicates)
        {
            foreach (var item in items)
            {
                if (!skipDuplicates || !collection.Contains(item))
                    collection.Add(item);
            }
        }
    }


Хотел сделать так parameterType.Component.AddRange(structDef.Component);, но будет ошибка collection = null
