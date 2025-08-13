private void FilterParameterSections(DeviceDescription description)
        {
            foreach (var device in description.Device)
            {
                foreach (var con in device.Connector)
                {
                    if (con.HostParameterSet != null)
                    {
                        var filteredItems = new List<_DeviceDescriptionNode>();

                        foreach (var node in con.HostParameterSet)
                        {
                            if (node is ParameterSectionType section)
                            {
                                if (section.Items != null)
                                {
                                    filteredItems.AddRange(section.Items);
                                }
                            }
                            else
                            {
                                filteredItems.Add(node);
                            }
                        }
                        con.HostParameterSet = new DeviceDescriptionNodeCollection(null, filteredItems);
                    }
                }
            }
        }

public static class ParentHelper
    {
        /// <summary>
        /// Gets the parent of the specified type.
        /// </summary>
        /// <typeparam name="TParent">The type of the parent.</typeparam>
        /// <param name="child">The child model.</param>
        /// <param name="maxLevels">The maximum levels to search. If <c>-1</c>, the number is unlimited.</param>
        /// <returns>The parent or <c>null</c> if the parent is not found.</returns>
        /// <exception cref="ArgumentNullException">The <paramref name="child" /> is <c>null</c>.</exception>
        public static TParent GetParent<TParent>(this _IDeviceDescriptionNode child, int maxLevels = -1)
            where TParent : class
        {
            Argument.IsNotNull("child", child);

            var parent = child._Parent;
            while (parent != null && maxLevels-- != 0)
            {
                if (parent is TParent)
                {
                    //Parent of type TParent is found
                    return (TParent)parent;
                }

                parent = parent._Parent;
            }
            return null;
        }
    }

Чтобы исключить отображение `ParameterSectionType` в `LibraryViewModel`, сохранив при этом его в модели данных, внесем следующие изменения:

### 1. Модифицируем метод `LoadLibraries` для фильтрации
```csharp
private async Task LoadLibraries()
{
    DeviceDescription platformDescription = null;
    try
    {
        var files = Directory.GetFiles("libraries", "*.xml");
        foreach (var item in files)
        {
            platformDescription = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(item);
            platformDescription._Name = Path.GetFileName(item);
            
            // Применяем фильтрацию перед добавлением
            FilterParameterSections(platformDescription);
            
            ComponentService.ApplyComponents(platformDescription);
            Thread.Sleep(1);

            Application.Current.Dispatcher.Invoke(() =>
            {
                HwRoot.Add(platformDescription);
            });
        }
    }
    catch (Exception e)
    {
        // Обработка ошибок
    }
}
```

### 2. Добавляем метод фильтрации
```csharp
private void FilterParameterSections(DeviceDescription description)
{
    // Обрабатываем ParameterSet, если он есть
    if (description.ParameterSet?.Items != null)
    {
        var filteredItems = new List<_DeviceDescriptionNode>();
        
        foreach (var node in description.ParameterSet.Items)
        {
            if (node is ParameterSectionType section)
            {
                // Добавляем только дочерние элементы секции
                if (section.Items != null)
                {
                    filteredItems.AddRange(section.Items);
                }
            }
            else
            {
                filteredItems.Add(node);
            }
        }
        
        description.ParameterSet.Items = new DeviceDescriptionNodeCollection(null, filteredItems);
    }
}
```

### 3. Альтернативный вариант (если нужно сохранить оригинальные данные)
Создадим отдельную коллекцию для отображения:
```csharp
public ObservableCollection<_DeviceDescriptionNode> FilteredHwRoot { get; } = new();

private void RefreshFilteredView()
{
    FilteredHwRoot.Clear();
    foreach (var rootNode in HwRoot)
    {
        if (rootNode is DeviceDescription desc)
        {
            var filtered = new DeviceDescription
            {
                _Name = desc._Name,
                Types = desc.Types,
                ParameterSet = desc.ParameterSet != null 
                    ? new DeviceDescriptionParameterSet 
                    { 
                        Items = FilterNodes(desc.ParameterSet.Items) 
                    } 
                    : null
            };
            FilteredHwRoot.Add(filtered);
        }
        else
        {
            FilteredHwRoot.Add(rootNode);
        }
    }
}

private DeviceDescriptionNodeCollection FilterNodes(DeviceDescriptionNodeCollection source)
{
    var filtered = new List<_DeviceDescriptionNode>();
    foreach (var node in source)
    {
        if (node is ParameterSectionType section)
        {
            filtered.AddRange(section.Items ?? Enumerable.Empty<_DeviceDescriptionNode>());
        }
        else
        {
            filtered.Add(node);
        }
    }
    return new DeviceDescriptionNodeCollection(null, filtered);
}
```

### 4. Обновляем биндинги в XAML
Замените привязку с `HwRoot` на `FilteredHwRoot`:
```xml
<TreeView ItemsSource="{Binding FilteredHwRoot}"/>
```

### 5. Важные нюансы:
1. **Сохранение оригинальных данных**:
   - Оригинальная коллекция `HwRoot` остается неизменной
   - Фильтрация применяется только для отображения

2. **Производительность**:
   - Для больших библиотек используйте `VirtualizingStackPanel`:
     ```xml
     <TreeView.ItemsPanel>
         <ItemsPanelTemplate>
             <VirtualizingStackPanel/>
         </ItemsPanelTemplate>
     </TreeView.ItemsPanel>
     ```

3. **Обновление при изменениях**:
   - Если данные динамически меняются, вызывайте `RefreshFilteredView()`

4. **Совместимость с экспортом**:
   - Метод `Export_CCodeExecute` будет работать корректно, так как использует `SelectedNode` из оригинальной модели

### Полный модифицированный класс:
```csharp
[Export]
[PartCreationPolicy(CreationPolicy.NonShared)]
public class LibraryViewModel : ViewModelBase
{
    // Оригинальная коллекция (полные данные)
    public ObservableCollection<_DeviceDescriptionNode> HwRoot { get; } = new();

    // Отфильтрованная коллекция (без ParameterSectionType)
    public ObservableCollection<_DeviceDescriptionNode> FilteredHwRoot { get; } = new();

    [ImportingConstructor]
    public LibraryViewModel()
    {
        // ... инициализация команд ...
    }

    private async Task LoadLibraries()
    {
        // ... загрузка данных в HwRoot ...
        RefreshFilteredView(); // Обновляем отфильтрованное представление
    }

    private void RefreshFilteredView()
    {
        FilteredHwRoot.Clear();
        foreach (var rootNode in HwRoot)
        {
            if (rootNode is DeviceDescription desc)
            {
                var filtered = new DeviceDescription
                {
                    _Name = desc._Name,
                    Types = desc.Types,
                    ParameterSet = desc.ParameterSet != null 
                        ? new DeviceDescriptionParameterSet 
                        { 
                            Items = FilterNodes(desc.ParameterSet.Items) 
                        } 
                        : null
                };
                FilteredHwRoot.Add(filtered);
            }
            else
            {
                FilteredHwRoot.Add(rootNode);
            }
        }
    }

    // ... остальные методы без изменений ...
}
```

Эти изменения:
1. Сохранят `ParameterSectionType` в исходных данных
2. Исключат его из отображения
3. Не повлияют на функциональность экспорта/импорта
4. Поддержат правильную вложенность элементов
5.
6. using Pilot.HwTool.Core;
using Pilot.HwTool.Mvvm;
using Pilot.HwTool.Mvvm.Helpers;
using Pilot.TargetPlatform;
using System;
using System.Collections.ObjectModel;
using System.ComponentModel.Composition;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Input;

namespace Pilot.HwTool
{
    [Export]
    [PartCreationPolicy(CreationPolicy.NonShared)]
    public class LibraryViewModel : Mvvm.ViewModelBase
    {
        private _DeviceDescriptionNode selectedNode;

        [ImportingConstructor]
        public LibraryViewModel()
        {
            HwRoot = new ObservableCollection<_DeviceDescriptionNode>();

            Import_StateBitsCommand = new RelayCommand(Import_StateBitsExecute);
            Import_LibSensorOutCommand = new RelayCommand(Import_LibSensorOutExecute);
            Module_AnalogInputCommand = new RelayCommand(Module_AnalogInputExecute);
            Export_CCodeCommand = new RelayCommand(Export_CCodeExecute, Export_CCodeCanExecute);
        }

        public ObservableCollection<_DeviceDescriptionNode> HwRoot { get; set; }

        public _DeviceDescriptionNode SelectedNode
        {
            get => selectedNode;
            set
            {
                selectedNode = value;
                RaisePropertyChanged();
            }
        }

        #region Import/Export

        public ICommand Import_StateBitsCommand { get; set; }
        public ICommand Import_LibSensorOutCommand { get; set; }
        public ICommand Module_AnalogInputCommand { get; set; }
        public ICommand Export_CCodeCommand { get; set; }

        private void Import_StateBitsExecute()
        {
            var text = Clipboard.GetText();
            if (!string.IsNullOrEmpty(text))
            {
                try
                {
                    var types = LzgConv.Import_xxxStateBits(text);
                    DeviceDescription dd = new DeviceDescription()
                    {
                        Types = new DeviceDescriptionTypes()
                        {
                            Items = new DeviceDescriptionNodeCollection(null, types),
                            //ItemsElementName = new ItemsChoiceType[] { },
                            //ItemsElementName = new[] { ItemsChoiceType.ArrayType, ItemsChoiceType.BitfieldType, ItemsChoiceType.EnumType, ItemsChoiceType.RangeType, ItemsChoiceType.StructType, ItemsChoiceType.UnionType },
                        }
                    };
                    var xmlstr = XmlHelper.Serialize2Str(dd);
                    Clipboard.SetText(xmlstr);
                    System.Media.SystemSounds.Asterisk.Play();
                }
                catch (Exception ex)
                {
                    Clipboard.Clear();
                    System.Media.SystemSounds.Exclamation.Play();
                }
            }
        }

        private void Import_LibSensorOutExecute()
        {
            var text = Clipboard.GetText();
            if (!string.IsNullOrEmpty(text))
            {
                try
                {
                    var typeList = LzgConv.Import_LibSensorOut_as_Range(text);
                    /*var paramSectList = LzgConv.Import_LibSensorOut_as_Param(text);*/
                    DeviceDescription dd = new DeviceDescription()
                    {
                        Types = new DeviceDescriptionTypes()
                        {
                            Items = new DeviceDescriptionNodeCollection(null, typeList),
                        },
                   /*     ParameterSet = new DeviceDescriptionParameterSet
                        {
                            Items = new DeviceDescriptionNodeCollection(null, paramSectList),
                        }*/
                    };
                    var xmlstr = XmlHelper.Serialize2Str(dd);
                    Clipboard.SetText(xmlstr);
                    System.Media.SystemSounds.Asterisk.Play();
                }
                catch (Exception ex)
                {
                    Clipboard.Clear();
                    System.Media.SystemSounds.Exclamation.Play();
                }
            }
        }

        private void Module_AnalogInputExecute()
        {
            var text = Clipboard.GetText();
            if (!string.IsNullOrEmpty(text))
            {
                try
                {
                    var ddd = LzgConv.Import_Module_AnalogInput(text);
                    DeviceDescription dd = new DeviceDescription()
                    {
                        Device = new DeviceDescriptionDeviceCollection(null, ddd),
                    };
                    var xmlstr = XmlHelper.Serialize2Str(dd);
                    Clipboard.SetText(xmlstr);
                    System.Media.SystemSounds.Asterisk.Play();
                }
                catch (Exception ex)
                {
                    Clipboard.Clear();
                    System.Media.SystemSounds.Exclamation.Play();
                }
            }
        }

        private bool Export_CCodeCanExecute()
        {
            return SelectedNode is DeviceDescription;// || SelectedNode is DeviceDescriptionDevice;
        }

        private void Export_CCodeExecute()
        {
            if (SelectedNode is DeviceDescription devdesc)
            {
                try
                {
                    var cstr = new CTarget(devdesc).Export_CCode();
                    Clipboard.SetText(cstr);
                    System.Media.SystemSounds.Asterisk.Play();
                }
                catch (Exception ex)
                {
                    Clipboard.Clear();
                    System.Media.SystemSounds.Exclamation.Play();
                }
            }
        }

        #endregion

        public override async void Initialize()
        {
            await LoadLibraries();
        }

        private async Task LoadLibraries()
        {
            DeviceDescription platformDescription = null;
            try
            {
                var files = Directory.GetFiles("libraries", "*.xml");
                foreach (var item in files)
                {
                    platformDescription = await Pilot.XmlHelper.DeserializeAsync<DeviceDescription>(item);
                    //platformDescription.ProjPath = Path.GetFullPath(item);
                    platformDescription._Name = Path.GetFileName(item);

                    
                    ComponentService.ApplyComponents(platformDescription);

                    Thread.Sleep(1);

                    Application.Current.Dispatcher.Invoke(() =>
                    {
                        HwRoot.Add(platformDescription);
                    });
                }
            }
            catch (Exception e)
            {

            }
        }
    }
}
