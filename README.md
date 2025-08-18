using Catel.MVVM;
using kivi.Commands;
using SCLExpress.Project;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Input;

namespace Module.IedExplorer
{
    [View(typeof(FindToolView))]
    public class FindToolViewModel : ViewModelBase
    {
        private readonly SCLE_Project _project;
        private string _searchText;
        public ObservableCollection<SearchResult> Results { get; set; } = new ObservableCollection<SearchResult>();
        public ICommand SearchCommand { get; }

        public string SearchText
        {
            get { return _searchText; }
            set 
            { 
                _searchText = value;
                RaisePropertyChanged(nameof(SearchText));
            }
        }

        public FindToolViewModel(SCLE_Project project)
        {
            _project = project;
            SearchCommand = new RelayCommand(ExecuteSearch);
        } 
        private async void ExecuteSearch()
        {
            Results.Clear();

            await Task.Run(() =>
            {
                SearchInCollection(_project.LibraryItems, "LibraryItems");
                SearchInCollection(_project.DataTypeItems, "DataTypeItems");
                SearchInCollection(_project.PouItems, "PouItems");
                SearchInCollection(_project.FNodeTypes, "FNodeTypes");
                SearchInCollection(_project.SubnetworkItems, "SubnetworkItems");
                SearchInCollection(_project.SubstationItems, "SubstationItems");
                SearchInCollection(_project.IedItems, "IEDItems");

                
            });
        }

        private void SearchInCollection(IEnumerable<object> items, string category)
        {
            if (items == null)
                return;

            foreach (var item in items)
            {
                if (item == null)
                    continue;

                var name = item.GetType().GetProperty("Name").GetValue(item).ToString();
                if (name.Contains(SearchText))
                {
                    Application.Current.Dispatcher.Invoke(() =>
                    {
                        Results.Add(new SearchResult
                        {
                            Name = name,
                            Category = category
                        });
                    });
                }
            }
        }
    }

    public class SearchResult
    {
        public string Name { get; set; }
        public string Category { get; set; }
    }
}
