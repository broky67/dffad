Если вы убрали `credentials` из параметров метода в интерфейсе, передавать их нужно через **HTTP-заголовки**. Вот как это правильно реализовать:

---

### 1. Модифицируем интерфейс (убираем credentials из параметров)
```csharp
[ServiceContract]
public interface ICheckForUpdateApi
{
    [OperationContract]
    [WebGet(UriTemplate = "check/{uid}?ver={ver}&lang={lang}", 
            ResponseFormat = WebMessageFormat.Json)]
    VersionDetails GetVersion(string uid, string ver, string lang = "ru");
}
```

---

### 2. Реализация клиента с передачей credentials в заголовках
```csharp
public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
{
    private readonly NetworkCredential _credentials;

    public CheckForUpdateClient(string address, NetworkCredential credentials)
        : base(new WebHttpBinding() { 
            MaxReceivedMessageSize = int.MaxValue,
            Security = new WebHttpSecurity {
                Mode = WebHttpSecurityMode.TransportCredentialOnly,
                Transport = new HttpTransportSecurity {
                    ClientCredentialType = HttpClientCredentialType.Basic
                }
            }
        }, new EndpointAddress(address))
    {
        _credentials = credentials;
        this.Endpoint.Behaviors.Add(new WebHttpBehavior());
    }

    public VersionDetails GetVersion(string uid, string ver, string lang = "ru")
    {
        using (new OperationContextScope(this.InnerChannel))
        {
            // Добавляем Basic Auth в заголовки
            var authHeader = Convert.ToBase64String(
                Encoding.UTF8.GetBytes($"{_credentials.UserName}:{_credentials.Password}"));

            WebOperationContext.Current.OutgoingRequest.Headers.Add(
                HttpRequestHeader.Authorization, 
                $"Basic {authHeader}");

            return base.Channel.GetVersion(uid, ver, lang);
        }
    }
}
```

---

### 3. Использование в коде проверки обновлений
```csharp
private static void BwOnDoWork(object sender, DoWorkEventArgs e)
{
    // ... предыдущий код
    
    try
    {
        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower();
        var credentials = new NetworkCredential("ekra", "test"); // или получайте из конфига
        var webClient = new CheckForUpdateClient(baseUri.ToString(), credentials);
        
        resp = webClient.GetVersion(appKey, appVer.ToString(), "ru");
    }
    catch (Exception ex)
    {
        Log.Error("Ошибка авторизации", ex);
    }
    
    // ... обработка результата
}
```

---

### Ключевые моменты:

1. **Где хранятся credentials**:
   - Теперь они передаются в конструктор клиента и хранятся в поле `_credentials`

2. **Как передаются**:
   - Через стандартный HTTP-заголовок `Authorization: Basic <base64>`

3. **Безопасность**:
   ```csharp
   // Всегда используйте HTTPS для Basic Auth!
   Mode = WebHttpSecurityMode.Transport // вместо TransportCredentialOnly
   ```

4. **Альтернатива для сложных случаев**:
   Если сервер требует особый формат аутентификации, можно добавить кастомный заголовок:
   ```csharp
   WebOperationContext.Current.OutgoingRequest.Headers.Add(
       "X-Auth-Token", 
       "your-special-token");
   ```

---

### Если сервер требует POST с телом:

1. Измените интерфейс:
```csharp
[WebInvoke(Method = "POST", UriTemplate = "check", BodyStyle = WebMessageBodyStyle.WrappedRequest)]
VersionDetails GetVersion(CheckRequest request);
```

2. Создайте DTO-класс:
```csharp
[DataContract]
public class CheckRequest
{
    [DataMember] public string Uid { get; set; }
    [DataMember] public string Ver { get; set; }
    [DataMember] public string Lang { get; set; }
    [DataMember] public NetworkCredential Credentials { get; set; }
}
```

Выбор способа зависит от требований серверного API.







Ошибка возникает из-за несоответствия между объявлением контракта WCF и его реализацией. Давайте исправим это:

### Проблема:
Вы используете `WebGet` (для HTTP GET), но пытаетесь передать credentials как параметр тела сообщения, что невозможно для GET-запросов.

### Решение 1: Передача credentials через заголовки (рекомендуется)

1. Измените интерфейс `ICheckForUpdateApi`:

```csharp
[ServiceContract]
public interface ICheckForUpdateApi
{
    [OperationContract]
    [WebGet(UriTemplate = "check/{uid}?ver={ver}&lang={lang}", ResponseFormat = WebMessageFormat.Json)]
    VersionDetails GetVersion(string uid, string ver, string lang = "ru");
}
```

2. Модифицируйте клиент для передачи credentials в заголовках:

```csharp
public VersionDetails GetVersion(string uid, string ver, NetworkCredential credentials, string lang = "ru")
{
    using (new OperationContextScope(this.InnerChannel))
    {
        // Добавляем Basic Auth в заголовки
        var authHeader = Convert.ToBase64String(
            Encoding.ASCII.GetBytes($"{credentials.UserName}:{credentials.Password}"));
        
        var request = WebOperationContext.Current.OutgoingRequest;
        request.Headers.Add(HttpRequestHeader.Authorization, $"Basic {authHeader}");
        
        return base.Channel.GetVersion(uid, ver, lang);
    }
}
```

### Решение 2: Использование WebInvoke (POST)

Если сервер требует передачи credentials в теле:

1. Измените интерфейс:

```csharp
[ServiceContract]
public interface ICheckForUpdateApi
{
    [OperationContract]
    [WebInvoke(Method = "POST", 
               UriTemplate = "check/{uid}?ver={ver}&lang={lang}", 
               BodyStyle = WebMessageBodyStyle.WrappedRequest,
               ResponseFormat = WebMessageFormat.Json)]
    VersionDetails GetVersion(string uid, string ver, string lang, NetworkCredential credentials);
}
```

2. Обновите клиент:

```csharp
public VersionDetails GetVersion(string uid, string ver, NetworkCredential credentials, string lang = "ru")
{
    return base.Channel.GetVersion(uid, ver, lang, credentials);
}
```

### Доработка BwOnDoWork:

```csharp
try
{
    var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower();
    var credentials = new NetworkCredential("ekra", "test");
    ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
    
    // Для решения 1:
    resp = webClient.GetVersion(appKey, appVer.ToString(), credentials, "ru");
    
    // Или для решения 2:
    resp = webClient.GetVersion(appKey, appVer.ToString(), "ru", credentials);
}
```

### Дополнительные рекомендации:

1. **Проверка серверной части**:
   - Убедитесь, что сервер ожидает аутентификацию именно в таком формате
   - Проверьте документацию API нового сайта

2. **Безопасность**:
   - Рассмотрите использование HTTPS
   - Не храните пароли в коде - используйте защищённое хранилище

3. **Логирование**:
   - Добавьте логирование запросов/ответов для отладки

Выберите решение, которое соответствует требованиям вашего сервера. Решение 1 (с заголовками) более стандартно для REST API.












if (!bool.TryParse(Properties.Settings.Default.IgnoreRevision, out _ignoreRevision))
            {
                _ignoreRevision = false;
            }
        }

        private void SaveSettings()
        {
            Properties.Settings.Default.LastCheck = DateTime.Now.ToString(CultureInfo.InvariantCulture);
            Properties.Settings.Default.Interval = _intervalKind.ToString();
            Properties.Settings.Default.IgnoreRevision = _ignoreRevision.ToString();
            Properties.Settings.Default.Save();
        }

        public bool CheckForUpdates(bool silent)
        {
            if (_dialog == null || _dialog.IsClosed)
            {
                // create a new instance of dialog VM every time
                _dialog = _updateDialog.CreateExport().Value;
            }

            //if (silent || _intervalKind != UpdateIntervalKind.Never)
            {
                if (_dialog.State == UpdateState.Busy)
                    return false;

                _dialog.Url = DefaultUrl;
                _dialog.State = UpdateState.Busy;

                BackgroundWorker bw = new BackgroundWorker();
                bw.DoWork += BwOnDoWork;
                bw.RunWorkerCompleted += BwOnRunWorkerCompleted;
                bw.RunWorkerAsync();
            }

            if (!silent)
            {
                ShowDialog();
            }

            return true;
        }

        private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                        },
                    };

                    VersionDetails resp = null;

                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }

        private void BwOnRunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.RunWorkerCompleted -= BwOnRunWorkerCompleted;

            if (e.Cancelled)
                return;

            var res = e.Result as CheckForUpdatesResult;
            if (res == null || !res.Result)
            {
                _dialog.State = UpdateState.ConnectionError;
                return;
            }

            var remoteVer = res.RemoteVer;
            var appInfo = GetAppFileInfo();
            var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);

            var remote = (_dialog.IgnoreRevision)
                ? new Version(remoteVer.Major, remoteVer.Minor, remoteVer.Build)
                : remoteVer;

            var local = (_dialog.IgnoreRevision)
                ? new Version(appVer.Major, appVer.Minor, appVer.Build)
                : appVer;

#if DEBUG
            // uncomment the following line to display the remote information anyway
            //local = new Version("0.0.0.0");
#endif

            if (remote > local)
            {
                _dialog.Local = appVer.ToString();
                _dialog.Remote = res.RemoteVer.ToString();
                _dialog.Url = res.RemoteUrl;
                _dialog.Notes = res.RemoteNotes;
                _dialog.State = UpdateState.Available;

                if (_intervalKind != UpdateIntervalKind.Never)
                {
                    // do not show dialog when we send anonymous update request
                    ShowDialog();
                }
            }
            else
            {
                _dialog.State = UpdateState.NotAvailable;
            }
        }

        /// <summary>
        /// Gets the application file info.
        /// </summary>
        /// <returns></returns>
        private static FileVersionInfo GetAppFileInfo()
        {
            var assembly = System.Reflection.Assembly.GetEntryAssembly();
            //var name = assembly.GetName();
            var appInfo = FileVersionInfo.GetVersionInfo(assembly.Location);
            //var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
            return appInfo;
        }

        private void ShowDialog()
        {
            if (_dialog.IsInitialized)
                return;

            // create dialog instance
            if (_uiVisualizerService.ShowDialog(_dialog) == true)
            {
                return;
            }
        }
    }
}


Понял, игнорируем предыдущие обсуждения про `ParameterType` и `ParameterSectionType`. Давайте полностью пересоберем `FindToolViewModel` для нового задания - поиска по активному проекту с отображением результатов в диалоговом окне.

### Полная реализация FindToolViewModel

```csharp
[Export]
[PartCreationPolicy(CreationPolicy.NonShared)]
public class FindToolViewModel : ViewModelBase
{
    private readonly Lazy<IProjectManager> _projectManager;
    private string _searchText;
    private bool _isBusy;
    private bool _caseSensitive;
    private bool _searchInNames = true;
    private bool _searchInValues;

    public ObservableCollection<SearchResult> Results { get; } = new();
    public ICommand SearchCommand { get; }
    public ICommand OpenResultCommand { get; }

    [ImportingConstructor]
    public FindToolViewModel(Lazy<IProjectManager> projectManager)
    {
        _projectManager = projectManager;
        
        SearchCommand = new RelayCommand(ExecuteSearch, CanExecuteSearch);
        OpenResultCommand = new RelayCommand<SearchResult>(OpenSelectedResult);
    }

    public string SearchText
    {
        get => _searchText;
        set => SetProperty(ref _searchText, value);
    }

    public bool IsBusy
    {
        get => _isBusy;
        set => SetProperty(ref _isBusy, value);
    }

    public bool CaseSensitive
    {
        get => _caseSensitive;
        set => SetProperty(ref _caseSensitive, value);
    }

    public bool SearchInNames
    {
        get => _searchInNames;
        set => SetProperty(ref _searchInNames, value);
    }

    public bool SearchInValues
    {
        get => _searchInValues;
        set => SetProperty(ref _searchInValues, value);
    }

    private bool CanExecuteSearch()
    {
        return !IsBusy && 
               !string.IsNullOrWhiteSpace(SearchText) && 
               _projectManager.Value.ActiveProject != null;
    }

    private async void ExecuteSearch()
    {
        IsBusy = true;
        Results.Clear();

        try
        {
            await Task.Run(() =>
            {
                var project = _projectManager.Value.ActiveProject;
                if (project == null) return;

                var comparison = CaseSensitive 
                    ? StringComparison.Ordinal 
                    : StringComparison.OrdinalIgnoreCase;

                SearchInProject(project, comparison);
            });
        }
        finally
        {
            IsBusy = false;
        }
    }

    private void SearchInProject(Project project, StringComparison comparison)
    {
        // Поиск по узлам проекта
        foreach (var node in project.Nodes)
        {
            if (node is IProjectItem item)
            {
                CheckAndAddResult(item, comparison);
            }

            if (node is IProjectContainer container)
            {
                SearchInContainer(container, comparison);
            }
        }
    }

    private void SearchInContainer(IProjectContainer container, StringComparison comparison)
    {
        foreach (var child in container.Children)
        {
            if (child is IProjectItem item)
            {
                CheckAndAddResult(item, comparison);
            }

            if (child is IProjectContainer childContainer)
            {
                SearchInContainer(childContainer, comparison);
            }
        }
    }

    private void CheckAndAddResult(IProjectItem item, StringComparison comparison)
    {
        var matches = new List<string>();

        if (SearchInNames && Matches(item.Name, comparison))
        {
            matches.Add("Name");
        }

        if (SearchInValues && item is IValueItem valueItem && 
            Matches(valueItem.Value?.ToString(), comparison))
        {
            matches.Add("Value");
        }

        if (matches.Count > 0)
        {
            var result = new SearchResult
            {
                Item = item,
                Name = item.Name,
                Path = GetItemPath(item),
                MatchFields = string.Join(", ", matches),
                MatchScore = CalculateMatchScore(item, matches.Contains("Name"), matches.Contains("Value"))
            };

            Application.Current.Dispatcher.Invoke(() => Results.Add(result));
        }
    }

    private bool Matches(string text, StringComparison comparison)
    {
        return !string.IsNullOrEmpty(text) && 
               text.Contains(SearchText, comparison);
    }

    private string GetItemPath(IProjectItem item)
    {
        var path = new List<string>();
        var current = item.Parent;

        while (current != null)
        {
            path.Insert(0, current.Name);
            current = current.Parent;
        }

        return string.Join("/", path);
    }

    private double CalculateMatchScore(IProjectItem item, bool nameMatch, bool valueMatch)
    {
        // Простая логика расчета релевантности
        double score = 0;
        if (nameMatch) score += 0.7;
        if (valueMatch) score += 0.3;
        return score;
    }

    private void OpenSelectedResult(SearchResult result)
    {
        if (result?.Item != null)
        {
            _projectManager.Value.SelectItem(result.Item);
        }
    }
}

public class SearchResult
{
    public IProjectItem Item { get; set; }
    public string Name { get; set; }
    public string Path { get; set; }
    public string MatchFields { get; set; }
    public double MatchScore { get; set; }
}
```

### Соответствующий XAML (FindToolView.xaml)

```xml
<controls:MetroWindow x:Class="FindToolView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:controls="http://metro.mahapps.com/winfx/xaml/controls"
        Title="Project Search" 
        Height="600" Width="800"
        GlowBrush="{DynamicResource AccentColorBrush}">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Search Panel -->
        <StackPanel Grid.Row="0" Orientation="Horizontal">
            <TextBox controls:TextBoxHelper.Watermark="Search text..."
                     Text="{Binding SearchText, UpdateSourceTrigger=PropertyChanged}"
                     Width="300" 
                     Margin="0,0,10,0"/>
            <Button Content="Search" 
                    Command="{Binding SearchCommand}" 
                    Width="80"/>
            <CheckBox Content="Case sensitive" 
                      IsChecked="{Binding CaseSensitive}" 
                      Margin="10,0,0,0"
                      VerticalAlignment="Center"/>
        </StackPanel>

        <!-- Filters -->
        <StackPanel Grid.Row="1" Orientation="Horizontal" Margin="0,10">
            <CheckBox Content="Search in names" 
                      IsChecked="{Binding SearchInNames}" 
                      Margin="0,0,10,0"/>
            <CheckBox Content="Search in values" 
                      IsChecked="{Binding SearchInValues}"/>
        </StackPanel>

        <!-- Results -->
        <DataGrid Grid.Row="2" 
                 ItemsSource="{Binding Results}" 
                 AutoGenerateColumns="False"
                 SelectionMode="Single"
                 IsReadOnly="True">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Name" 
                                   Binding="{Binding Name}" 
                                   Width="200"/>
                <DataGridTextColumn Header="Path" 
                                   Binding="{Binding Path}" 
                                   Width="300"/>
                <DataGridTextColumn Header="Found in" 
                                   Binding="{Binding MatchFields}" 
                                   Width="150"/>
                <DataGridTextColumn Header="Relevance" 
                                   Binding="{Binding MatchScore, StringFormat={}{0:P0}}" 
                                   Width="80"/>
                <DataGridTemplateColumn Width="80">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="Open" 
                                    Command="{Binding Path=DataContext.OpenResultCommand, 
                                              RelativeSource={RelativeSource AncestorType=DataGrid}}"
                                    CommandParameter="{Binding}"
                                    Style="{StaticResource AccentedButtonStyle}"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</controls:MetroWindow>
```

### Ключевые особенности:

1. **Гибкий поиск**:
   - Поддержка поиска по именам и значениям
   - Учет регистра
   - Рекурсивный обход всей структуры проекта

2. **Информативные результаты**:
   - Отображение пути к объекту
   - Показатель релевантности
   - Указание, где найдено совпадение

3. **Интеграция**:
   - Навигация к найденным элементам через ProjectManager
   - Поддержка MVVM через инъекцию зависимостей

4. **Производительность**:
   - Асинхронное выполнение поиска
   - Постепенное добавление результатов

5. **UI-фичи**:
   - Водяной знак в поле поиска
   - Стилизованные кнопки
   - Сортировка по колонкам

Для использования просто вызовите окно из главного ViewModel:

```csharp
public ICommand ShowFindToolCommand => new RelayCommand(() =>
{
    var findTool = ServiceLocator.GetInstance<FindToolViewModel>();
    var window = new FindToolView { DataContext = findTool };
    window.ShowDialog();
});
```
аьаьаьаьпьатаьаь
3. **Использование клиента с аутентификацией**:
   В методе `BwOnDoWork`:

```csharp
try
{
    var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower();
    var credentials = new NetworkCredential("ekra", "test");
    ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString(), credentials);
    resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credentials, lang: "ru");
}
```

### Дополнительные проверки:

1. **Проверьте тип аутентификации**:
   - Basic
   - NTLM
   - Digest
   - Negotiate

   Возможно, нужно изменить `HttpClientCredentialType`:

```csharp
ClientCredentialType = HttpClientCredentialType.Ntlm // или другой тип
```

2. **Логирование трафика** (для отладки):
   Добавьте в конфигурацию:

```xml
<system.diagnostics>
    <sources>
        <source name="System.ServiceModel" switchValue="Information, ActivityTracing">
            <listeners>
                <add name="traceListener" 
                     type="System.Diagnostics.XmlWriterTraceListener" 
                     initializeData="WcfTrace.svclog"/>
            </listeners>
        </source>
    </sources>
</system.diagnostics>
```

3. **Проверка URL и прокси**:
   Убедитесь, что:
   - URL нового сайта корректный
   - Прокси-сервер правильно настроен
   - Учетные данные верные

### Если проблема сохраняется:

1. **Обход прокси** (если возможно):
```csharp
webClient.Proxy = GlobalProxySelection.GetEmptyWebProxy();
```

2. **Использование HttpClient** (альтернативный вариант):
```csharp
using (var handler = new HttpClientHandler {
    Credentials = new NetworkCredential("ekra", "test"),
    Proxy = WebRequest.DefaultWebProxy
})
using (var client = new HttpClient(handler))
{
    // Ваш запрос
}
```

Попробуйте эти варианты и посмотрите, какой сработает в вашем случае. Если проблема останется, нужны более детальные данные о структуре запросов/ответов.


public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
    {
        public CheckForUpdateClient(string address)
            : base(new WebHttpBinding() { MaxReceivedMessageSize = int.MaxValue }, new EndpointAddress(address))
        {
            if (this.Endpoint.ListenUri.Scheme == "https")
            {
                // configure SSL
                var binding = (WebHttpBinding)this.Endpoint.Binding;
                binding.Security.Mode = WebHttpSecurityMode.Transport;
            }

            this.Endpoint.Behaviors.Add(new WebHttpBehavior());
        }

        public VersionDetails GetVersion(string uid, string ver, NetworkCredential credentials, string lang = "ru")
        {
            using (new OperationContextScope(this.InnerChannel))
            {
                var test = base.Channel.GetVersion(uid, ver, credentials, lang);
                return test;
            }
        }

        private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                            Credential = new NetworkCredential(
                                userName: "ekra",
                                password: "test")
                        },
                    };

                    VersionDetails resp = null;

                    var credential = new NetworkCredential(
                                userName: "",
                                password: "");
                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credential, lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }

Вот модифицированный код на русском для обработки JSON и отображения информации о новой версии:

private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                            Credential = new NetworkCredential(
                                userName: "ekra",
                                password: "test")
                        },
                    };

                    VersionDetails resp = null;

                    var credential = new NetworkCredential(
                                userName: "",
                                password: "");
                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credential, lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }


    public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
    {
        public CheckForUpdateClient(string address)
            : base(new WebHttpBinding() { MaxReceivedMessageSize = int.MaxValue }, new EndpointAddress(address))
        {
            if (this.Endpoint.ListenUri.Scheme == "https")
            {
                // configure SSL
                var binding = (WebHttpBinding)this.Endpoint.Binding;
                binding.Security.Mode = WebHttpSecurityMode.Transport;
            }

            this.Endpoint.Behaviors.Add(new WebHttpBehavior());
        }

        public VersionDetails GetVersion(string uid, string ver, NetworkCredential credentials, string lang = "ru")
        {
            using (new OperationContextScope(this.InnerChannel))
            {
                var test = base.Channel.GetVersion(uid, ver, credentials, lang);
                return test;
            }
        }
### Доработки в ViewModel:

Добавьте в ваш `UpdateDialogViewModel` следующие свойства:

```csharp
public Version Remote { get; private set; }
public DateTime Released { get; private set; }
public string Notes { get; private set; }
public Uri DownloadUri { get; private set; }

// Метод для обработки результата проверки
private void ProcessUpdateResult(CheckForUpdatesResult result)
{
    State = result.State;
    
    if (result.State == UpdateState.Available)
    {
        Remote = result.RemoteVersion;
        Released = result.ReleaseDate;
        Notes = result.Notes;
        DownloadUri = result.DownloadUrl;
        
        // Можно добавить логирование
        Logger.Info($"Доступна новая версия: {Remote} от {Released:d}");
    }
}
```

Теперь ваше приложение будет:
1. Получать данные о версиях в JSON-формате
2. Корректно парсить и сравнивать версии
3. Отображать всю информацию о новой версии в диалоговом окне
4. Показывать список изменений в удобном формате

Для работы кода нужно добавить NuGet пакет Newtonsoft.Json и настроить обработку ошибок подключения.
<catel:DataWindow x:Class="Module.Updater.Dialogs.UpdateDialogView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:catel="http://catel.codeplex.com"
                  xmlns:dialogs="clr-namespace:Module.Updater.Dialogs"
                  xmlns:properties="clr-namespace:Module.Updater.Properties"
                  xmlns:kivi="http://kivi.dev.ekra.ru"
                  xmlns:pilot="http://pilot.dev.ekra.ru"
                  Icon="/Module.Updater;component/Resources/Images/application_update.png" 
                  Height="400" Width="600"
                  kivi:CommandBindings.CommandBindings="{Binding CommandBindings}">

    <catel:DataWindow.Resources>
        <ResourceDictionary>
            <!--<ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="/Catel.MVVM;component/themes/generic.xaml" />
            </ResourceDictionary.MergedDictionaries>-->

            <BooleanToVisibilityConverter x:Key="bool2vis"/>
            <kivi:NegatedBooleanToVisibilityConverter x:Key="notbool2vis"/>
            <dialogs:NegateBooleanConverter x:Key="negatebool"/>
            <dialogs:UpdateStateConverter x:Key="state2vis"/>
            <dialogs:UpdateState2EnableConverter x:Key="state2ena"/>
            <pilot:LocalizableStringConverter x:Key="LocalizableStringConverter" ResourceManager="{x:Static properties:Resources.ResourceManager}"/>

        </ResourceDictionary>
    </catel:DataWindow.Resources>

    <Grid Margin="6">
        <DockPanel LastChildFill="True">
            <Grid DockPanel.Dock="Top">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="100"/>
                    <ColumnDefinition/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <TextBlock Text="{x:Static properties:Resources.Check_for_update_interval_}" VerticalAlignment="Center"/>
                <ComboBox Grid.Column="1" SelectedItem="{Binding Interval}" ItemsSource="{Binding Intervals}">
                    <ComboBox.ItemTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding Path=., Mode=OneWay, Converter={StaticResource LocalizableStringConverter}}"/>
                        </DataTemplate>
                    </ComboBox.ItemTemplate>
                </ComboBox>
                <!--<CheckBox Grid.Column="2" IsChecked="{Binding HasUpdate}"/>-->
                <Button Grid.Column="3" Command="{Binding CheckForUpdatesCommand}" Content="{x:Static properties:Resources.Check_For_Updates}"
                        IsEnabled="{Binding State, Converter={StaticResource state2ena}}">
                </Button>
            </Grid>
            <Grid DockPanel.Dock="Bottom">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <CheckBox Grid.Column="0" IsChecked="{Binding IgnoreRevision}" Margin="10,0" VerticalAlignment="Center"
                          Content="{x:Static properties:Resources.IgnoreRevNum}">
                    <CheckBox.ToolTip>
                        <TextBlock Text="{x:Static properties:Resources.IgnoreRevNumTip}" MaxWidth="300" TextWrapping="Wrap"/>
                    </CheckBox.ToolTip>
                </CheckBox>
            </Grid>
          <Grid Margin="0,10">
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.Busy}}">
                <TextBlock Text="{x:Static properties:Resources.Wait_}"/>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.NotAvailable}}">
                <TextBlock Text="{x:Static properties:Resources.No_updates_available}"/>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.ConnectionError}}">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition/>
                </Grid.RowDefinitions>

                <Grid Grid.Row="0">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="Auto"/>
                        <ColumnDefinition/>
                    </Grid.ColumnDefinitions>
                    <Image Source="/Module.Updater;component/Resources/Images/error.png" Width="16" Margin="0,0,3,0"/>
                    <TextBlock Text="{x:Static properties:Resources.Connection_error_}" Grid.Column="1"/>
                </Grid>
                <Label Grid.Row="1">
                    <Hyperlink Command="{x:Static NavigationCommands.GoToPage}">
                        <Run Text="{x:Static properties:Resources.Click_to_open_web_site}" ToolTip="{x:Static properties:Resources.Click_to_open_web_site}"/>
                    </Hyperlink>
                    <Label.ContextMenu>
                        <ContextMenu>
                            <MenuItem Command="{x:Static ApplicationCommands.Copy}" Icon="/Module.Updater;component/Resources/Images/page_copy.png"/>
                        </ContextMenu>
                    </Label.ContextMenu>
                </Label>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.Available}}">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition/>
                </Grid.RowDefinitions>

                <TextBlock Grid.Row="0" Text="{x:Static properties:Resources.New_version_available}" FontWeight="Bold"/>
                <!--<TextBlock Grid.Row="1" Text="{Binding Local, StringFormat={x:Static properties:Resources.LocalVerFormat}}"/>-->
                <TextBlock Grid.Row="2" Text="{Binding Remote, StringFormat={x:Static properties:Resources.RemoteVerFormat}}"/>
                <!--<TextBlock Grid.Row="3" Text="Update?"/>-->
                <Grid Grid.Row="3" Margin="0,10">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="Auto"/>
                        <ColumnDefinition/>
                        <!--<ColumnDefinition Width="Auto"/>-->
                    </Grid.ColumnDefinitions>
                    <Image Source="/Module.Updater;component/Resources/Images/error.png" Width="16" Margin="0,0,3,0"/>
                    <TextBlock Grid.Column="1">
                        <Run Text="{x:Static properties:Resources.Manual_update_offer}"></Run>
                        <Hyperlink Command="{x:Static NavigationCommands.GoToPage}">
                            <Hyperlink.ContextMenu>
                                <ContextMenu>
                                    <MenuItem Command="{x:Static ApplicationCommands.Copy}" Icon="/Module.Updater;component/Resources/Images/page_copy.png"/>
                                </ContextMenu>
                            </Hyperlink.ContextMenu>
                            <Run Text="{x:Static properties:Resources.Visit_us}" ToolTip="{x:Static properties:Resources.Click_to_open_folder}"/>
                        </Hyperlink>
                        <Run>.</Run>
                    </TextBlock>
                    <!--<Button Grid.Column="2" Command="{Binding DownloadUpdatesCommand}">
                        <StackPanel Orientation="Horizontal">
                            <Image Source="/Module.Updater;component/Resources/Images/download.png" Width="16" Margin="0,0,3,0"/>
                            <TextBlock Text="{x:Static properties:Resources.Download}"/>
                        </StackPanel>
                    </Button>-->
                </Grid>
                <TextBlock Grid.Row="4" Text="{x:Static properties:Resources.Changelog_}" FontWeight="Bold"/>
                <kivi:MarkdownBox Grid.Row="5" Source="{Binding Notes}" IsReadOnly="True" BorderThickness="0" ZoomBarVisibility="Collapsed"/>
            </Grid>
          </Grid>
        </DockPanel>
    </Grid>
</catel:DataWindow>


{
  "product": "scle",
  "version": "1.2.0.7719",
  "released": "2024-04-09 13:16:00",
  "description": "Улучшения:\n- Добавлена возможность лицензирования с USB ключом без кода оборудования;\n- Доработана визуальная часть меню Файл\nИсправления:\n- Исправлен баг загрузки стартовой страницы;",
  "url": "/soft/scle/downloads/software/scl_express/scle_setup.exe",
  "Theme": "Основное программное обеспечение",
  "Section": "SCL Express",
  "error": "OK"
}


using System;
using System.IO;
using System.Reflection;
using System.Runtime.Serialization;
using System.ServiceModel;
using System.ServiceModel.Web;

namespace Module.Updater.Rest
{
    [ServiceContract]
    public interface ICheckForUpdateApi
    {
        [OperationContract]
        [WebGet(UriTemplate = "check/{uid}?ver={ver}&lang={lang}", ResponseFormat = WebMessageFormat.Json)]
        VersionDetails GetVersion(string uid, string ver, string lang = "ru");

        [OperationContract]
        [WebGet(UriTemplate = "download/{uid}?ver={ver}")]
        Stream Download(string uid, string ver);

        [OperationContract]
        [WebInvoke(Method = "POST", UriTemplate = "upload/{uid}?filename={filename}")]
        void Upload(string uid, string filename, Stream stream);

        [OperationContract]
        [WebGet]
        string Echo(string input);
    }

    [Obfuscation(Exclude = true)]
    [DataContract(Name = "details", Namespace = "http://schemas.ekra.ru/2019/11/PilotApp")]
    public class VersionDetails
    {
        [DataMember(Name = "product")]
        public string Name;

        [DataMember(Name = "version")]
        public string Version;

        [DataMember(Name = "released")]
        public string Released { get; set; }

        [DataMember(Name = "description")]
        public string Description { get; set; }

        [DataMember(Name = "error")]
        public string ErrorMsg { get; set; }

        [DataMember(Name = "url")]
        public string Url;

        public override string ToString()
        {
            return string.Format("name = '{0}'; version = {1}; url = {2}", Name, Version, Url);
        }
    }

    [DataContract]
    public class RemoteFileInfo : IDisposable
    {
        [DataMember(Name = "name")]
        public string FileName;

        [DataMember(Name = "length")]
        public long FileLength;

        [DataMember(Name = "data", Order = 1)]
        public Stream FileStream;

        public void Dispose()
        {
            if (FileStream != null)
            {
                FileStream.Close();
                FileStream = null;
            }
        }
    }
}

using System;
using System.ComponentModel;
using System.ComponentModel.Composition;
using System.Configuration;
using System.Diagnostics;
using System.Globalization;
using System.IO;
using System.Text;
using Module.Updater.Dialogs;
using Module.Updater.Rest;
using Pilot.Licensing;
using Pilot.Shell.Infrastructure.Logging;
using Pilot.Shell.Infrastructure.Services;

namespace Module.Updater
{
    [Export(typeof(IUpdateController))]
    [PartCreationPolicy(CreationPolicy.Shared)]
    public class UpdateControllerImpl : IUpdateController
    {
        private static readonly ILog Log = LogManager.GetCurrentClassLogger();
        private readonly IUIVisualizerService _uiVisualizerService;
        private readonly IWorkspaceManager _workspaceManager;
        private static System.Windows.Threading.DispatcherTimer timer = new System.Windows.Threading.DispatcherTimer();
        private UpdateDialogViewModel _dialog;
        private DateTime _lastCheck;
        private static UpdateIntervalKind _intervalKind;
        private static bool _ignoreRevision;
        private int _tickCount;
        private const string DefaultUrl = /*"https://soft.ekra.ru/scle/software/"*/ "https://soft-ekra.tw1.ru/soft/scle/updates/check?uid=%22amdjsp-l6bc6j-t6c14j-hzy4v9%22";

#pragma warning disable 0649
        [Import]
        private ExportFactory<UpdateDialogViewModel> _updateDialog;
#pragma warning restore 0649

        [ImportingConstructor]
        public UpdateControllerImpl(IWorkspaceManager workspaceManager, IUIVisualizerService uiVisualizerService)
        {
            _workspaceManager = workspaceManager;
            _uiVisualizerService = uiVisualizerService;

            _workspaceManager.ShellClosing += WorkspaceManagerOnShellClosing;

            LoadSettings();

            // keep the current value
            var intervalKind = GetInterval();

            if (intervalKind != UpdateIntervalKind.OnStartup)
                SetInterval(UpdateIntervalKind.Never); // send an anonymous update request even when 'Never' selected
            else
                SetInterval(UpdateIntervalKind.OnStartup);

            timer.Tick += (sender, args) =>
            {
                _tickCount++;

                #region MyRegion
                if (_tickCount == 1)
                {
                    // restore the interval value at first tick
                    SetInterval(intervalKind);

                    var current = timer.Interval;
                    var elapsed = DateTime.Now - _lastCheck;
                    Log.Info("Last check for updates time '{0}'.", _lastCheck);
                    if (elapsed < current)
                    {
                        timer.Interval = current - elapsed;
                        Log.Debug("Adjust update timer interval with '{0}'.", timer.Interval);
                    }
                }
                else
                {
                    // re-apply the interval value at every ticks
                    SetInterval(_intervalKind);
                }
                #endregion

                if (_intervalKind != UpdateIntervalKind.Never)
                    _lastCheck = DateTime.Now;

                Log.Info("Begin check for updates...");

                CheckForUpdates(true);
            };

#if !DEBUG
            // start the update timer for Never or OnStartup interval selected
            timer.Start();
#endif
        }

        private void WorkspaceManagerOnShellClosing(object sender, CancelEventArgs e)
        {
            SaveSettings();
        }

        public UpdateIntervalKind GetInterval()
        {
            return _intervalKind;
        }

        public void SetInterval(UpdateIntervalKind kind)
        {
            _intervalKind = kind;

            Log.Debug("Setup check for updates interval '{0}'", kind);

            timer.Stop();

            switch (kind)
            {
                case UpdateIntervalKind.Never:
                    timer.Interval = TimeSpan.FromSeconds(3);
                    break;
                case UpdateIntervalKind.OnStartup:
                    timer.Interval = TimeSpan.FromSeconds(10);
                    break;
                case UpdateIntervalKind.Hour:
                    timer.Interval = TimeSpan.FromHours(1);
                    timer.Start();
                    break;
                case UpdateIntervalKind.Day:
                    timer.Interval = TimeSpan.FromDays(1);
                    timer.Start();
                    break;
                case UpdateIntervalKind.Week:
                    timer.Interval = TimeSpan.FromDays(7);
                    timer.Start();
                    break;
            }

            Log.Debug("Setup update timer interval with '{0}'. Update timer is {1}.", timer.Interval, timer.IsEnabled? "started" : "stopped");
        }

        public object GetProperty(string propertyName)
        {
            switch (propertyName)
            {
                case "Interval":
                    return _intervalKind;
                case "IgnoreRevision":
                    return _ignoreRevision;
            }
            return null;
        }

        public void SetProperty(string propertyName, object propertValue)
        {
            switch (propertyName)
            {
                case "Interval":
                    _intervalKind = (UpdateIntervalKind)propertValue; return;
                case "IgnoreRevision":
                    _ignoreRevision = (bool)propertValue; return;
            }
        }

        private void LoadSettings()
        {
            if (!DateTime.TryParse(Properties.Settings.Default.LastCheck, CultureInfo.InvariantCulture, DateTimeStyles.None, out _lastCheck))
            {
                _lastCheck = DateTime.Now;
            }

            if (!Enum.TryParse(Properties.Settings.Default.Interval, out _intervalKind))
            {
                _intervalKind = UpdateIntervalKind.Hour;
            }

            if (!bool.TryParse(Properties.Settings.Default.IgnoreRevision, out _ignoreRevision))
            {
                _ignoreRevision = false;
            }
        }

        private void SaveSettings()
        {
            Properties.Settings.Default.LastCheck = DateTime.Now.ToString(CultureInfo.InvariantCulture);
            Properties.Settings.Default.Interval = _intervalKind.ToString();
            Properties.Settings.Default.IgnoreRevision = _ignoreRevision.ToString();
            Properties.Settings.Default.Save();
        }

        public bool CheckForUpdates(bool silent)
        {
            if (_dialog == null || _dialog.IsClosed)
            {
                // create a new instance of dialog VM every time
                _dialog = _updateDialog.CreateExport().Value;
            }

            //if (silent || _intervalKind != UpdateIntervalKind.Never)
            {
                if (_dialog.State == UpdateState.Busy)
                    return false;

                _dialog.Url = DefaultUrl;
                _dialog.State = UpdateState.Busy;

                BackgroundWorker bw = new BackgroundWorker();
                bw.DoWork += BwOnDoWork;
                bw.RunWorkerCompleted += BwOnRunWorkerCompleted;
                bw.RunWorkerAsync();
            }

            if (!silent)
            {
                ShowDialog();
            }

            return true;
        }

        private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                        },
                    };

                    VersionDetails resp = null;

                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }

        private void BwOnRunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.RunWorkerCompleted -= BwOnRunWorkerCompleted;

            if (e.Cancelled)
                return;

            var res = e.Result as CheckForUpdatesResult;
            if (res == null || !res.Result)
            {
                _dialog.State = UpdateState.ConnectionError;
                return;
            }

            var remoteVer = res.RemoteVer;
            var appInfo = GetAppFileInfo();
            var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);

            var remote = (_dialog.IgnoreRevision)
                ? new Version(remoteVer.Major, remoteVer.Minor, remoteVer.Build)
                : remoteVer;

            var local = (_dialog.IgnoreRevision)
                ? new Version(appVer.Major, appVer.Minor, appVer.Build)
                : appVer;

#if DEBUG
            // uncomment the following line to display the remote information anyway
            //local = new Version("0.0.0.0");
#endif

            if (remote > local)
            {
                _dialog.Local = appVer.ToString();
                _dialog.Remote = res.RemoteVer.ToString();
                _dialog.Url = res.RemoteUrl;
                _dialog.Notes = res.RemoteNotes;
                _dialog.State = UpdateState.Available;

                if (_intervalKind != UpdateIntervalKind.Never)
                {
                    // do not show dialog when we send anonymous update request
                    ShowDialog();
                }
            }
            else
            {
                _dialog.State = UpdateState.NotAvailable;
            }
        }

        /// <summary>
        /// Gets the application file info.
        /// </summary>
        /// <returns></returns>
        private static FileVersionInfo GetAppFileInfo()
        {
            var assembly = System.Reflection.Assembly.GetEntryAssembly();
            //var name = assembly.GetName();
            var appInfo = FileVersionInfo.GetVersionInfo(assembly.Location);
            //var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
            return appInfo;
        }

        private void ShowDialog()
        {
            if (_dialog.IsInitialized)
                return;

            // create dialog instance
            if (_uiVisualizerService.ShowDialog(_dialog) == true)
            {
                return;
            }
        }
    }
}


Понял, игнорируем предыдущие обсуждения про `ParameterType` и `ParameterSectionType`. Давайте полностью пересоберем `FindToolViewModel` для нового задания - поиска по активному проекту с отображением результатов в диалоговом окне.

### Полная реализация FindToolViewModel

```csharp
[Export]
[PartCreationPolicy(CreationPolicy.NonShared)]
public class FindToolViewModel : ViewModelBase
{
    private readonly Lazy<IProjectManager> _projectManager;
    private string _searchText;
    private bool _isBusy;
    private bool _caseSensitive;
    private bool _searchInNames = true;
    private bool _searchInValues;

    public ObservableCollection<SearchResult> Results { get; } = new();
    public ICommand SearchCommand { get; }
    public ICommand OpenResultCommand { get; }

    [ImportingConstructor]
    public FindToolViewModel(Lazy<IProjectManager> projectManager)
    {
        _projectManager = projectManager;
        
        SearchCommand = new RelayCommand(ExecuteSearch, CanExecuteSearch);
        OpenResultCommand = new RelayCommand<SearchResult>(OpenSelectedResult);
    }

    public string SearchText
    {
        get => _searchText;
        set => SetProperty(ref _searchText, value);
    }

    public bool IsBusy
    {
        get => _isBusy;
        set => SetProperty(ref _isBusy, value);
    }

    public bool CaseSensitive
    {
        get => _caseSensitive;
        set => SetProperty(ref _caseSensitive, value);
    }

    public bool SearchInNames
    {
        get => _searchInNames;
        set => SetProperty(ref _searchInNames, value);
    }

    public bool SearchInValues
    {
        get => _searchInValues;
        set => SetProperty(ref _searchInValues, value);
    }

    private bool CanExecuteSearch()
    {
        return !IsBusy && 
               !string.IsNullOrWhiteSpace(SearchText) && 
               _projectManager.Value.ActiveProject != null;
    }

    private async void ExecuteSearch()
    {
        IsBusy = true;
        Results.Clear();

        try
        {
            await Task.Run(() =>
            {
                var project = _projectManager.Value.ActiveProject;
                if (project == null) return;

                var comparison = CaseSensitive 
                    ? StringComparison.Ordinal 
                    : StringComparison.OrdinalIgnoreCase;

                SearchInProject(project, comparison);
            });
        }
        finally
        {
            IsBusy = false;
        }
    }

    private void SearchInProject(Project project, StringComparison comparison)
    {
        // Поиск по узлам проекта
        foreach (var node in project.Nodes)
        {
            if (node is IProjectItem item)
            {
                CheckAndAddResult(item, comparison);
            }

            if (node is IProjectContainer container)
            {
                SearchInContainer(container, comparison);
            }
        }
    }

    private void SearchInContainer(IProjectContainer container, StringComparison comparison)
    {
        foreach (var child in container.Children)
        {
            if (child is IProjectItem item)
            {
                CheckAndAddResult(item, comparison);
            }

            if (child is IProjectContainer childContainer)
            {
                SearchInContainer(childContainer, comparison);
            }
        }
    }

    private void CheckAndAddResult(IProjectItem item, StringComparison comparison)
    {
        var matches = new List<string>();

        if (SearchInNames && Matches(item.Name, comparison))
        {
            matches.Add("Name");
        }

        if (SearchInValues && item is IValueItem valueItem && 
            Matches(valueItem.Value?.ToString(), comparison))
        {
            matches.Add("Value");
        }

        if (matches.Count > 0)
        {
            var result = new SearchResult
            {
                Item = item,
                Name = item.Name,
                Path = GetItemPath(item),
                MatchFields = string.Join(", ", matches),
                MatchScore = CalculateMatchScore(item, matches.Contains("Name"), matches.Contains("Value"))
            };

            Application.Current.Dispatcher.Invoke(() => Results.Add(result));
        }
    }

    private bool Matches(string text, StringComparison comparison)
    {
        return !string.IsNullOrEmpty(text) && 
               text.Contains(SearchText, comparison);
    }

    private string GetItemPath(IProjectItem item)
    {
        var path = new List<string>();
        var current = item.Parent;

        while (current != null)
        {
            path.Insert(0, current.Name);
            current = current.Parent;
        }

        return string.Join("/", path);
    }

    private double CalculateMatchScore(IProjectItem item, bool nameMatch, bool valueMatch)
    {
        // Простая логика расчета релевантности
        double score = 0;
        if (nameMatch) score += 0.7;
        if (valueMatch) score += 0.3;
        return score;
    }

    private void OpenSelectedResult(SearchResult result)
    {
        if (result?.Item != null)
        {
            _projectManager.Value.SelectItem(result.Item);
        }
    }
}

public class SearchResult
{
    public IProjectItem Item { get; set; }
    public string Name { get; set; }
    public string Path { get; set; }
    public string MatchFields { get; set; }
    public double MatchScore { get; set; }
}
```

### Соответствующий XAML (FindToolView.xaml)

```xml
<controls:MetroWindow x:Class="FindToolView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:controls="http://metro.mahapps.com/winfx/xaml/controls"
        Title="Project Search" 
        Height="600" Width="800"
        GlowBrush="{DynamicResource AccentColorBrush}">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Search Panel -->
        <StackPanel Grid.Row="0" Orientation="Horizontal">
            <TextBox controls:TextBoxHelper.Watermark="Search text..."
                     Text="{Binding SearchText, UpdateSourceTrigger=PropertyChanged}"
                     Width="300" 
                     Margin="0,0,10,0"/>
            <Button Content="Search" 
                    Command="{Binding SearchCommand}" 
                    Width="80"/>
            <CheckBox Content="Case sensitive" 
                      IsChecked="{Binding CaseSensitive}" 
                      Margin="10,0,0,0"
                      VerticalAlignment="Center"/>
        </StackPanel>

        <!-- Filters -->
        <StackPanel Grid.Row="1" Orientation="Horizontal" Margin="0,10">
            <CheckBox Content="Search in names" 
                      IsChecked="{Binding SearchInNames}" 
                      Margin="0,0,10,0"/>
            <CheckBox Content="Search in values" 
                      IsChecked="{Binding SearchInValues}"/>
        </StackPanel>

        <!-- Results -->
        <DataGrid Grid.Row="2" 
                 ItemsSource="{Binding Results}" 
                 AutoGenerateColumns="False"
                 SelectionMode="Single"
                 IsReadOnly="True">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Name" 
                                   Binding="{Binding Name}" 
                                   Width="200"/>
                <DataGridTextColumn Header="Path" 
                                   Binding="{Binding Path}" 
                                   Width="300"/>
                <DataGridTextColumn Header="Found in" 
                                   Binding="{Binding MatchFields}" 
                                   Width="150"/>
                <DataGridTextColumn Header="Relevance" 
                                   Binding="{Binding MatchScore, StringFormat={}{0:P0}}" 
                                   Width="80"/>
                <DataGridTemplateColumn Width="80">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="Open" 
                                    Command="{Binding Path=DataContext.OpenResultCommand, 
                                              RelativeSource={RelativeSource AncestorType=DataGrid}}"
                                    CommandParameter="{Binding}"
                                    Style="{StaticResource AccentedButtonStyle}"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</controls:MetroWindow>
```

### Ключевые особенности:

1. **Гибкий поиск**:
   - Поддержка поиска по именам и значениям
   - Учет регистра
   - Рекурсивный обход всей структуры проекта

2. **Информативные результаты**:
   - Отображение пути к объекту
   - Показатель релевантности
   - Указание, где найдено совпадение

3. **Интеграция**:
   - Навигация к найденным элементам через ProjectManager
   - Поддержка MVVM через инъекцию зависимостей

4. **Производительность**:
   - Асинхронное выполнение поиска
   - Постепенное добавление результатов

5. **UI-фичи**:
   - Водяной знак в поле поиска
   - Стилизованные кнопки
   - Сортировка по колонкам

Для использования просто вызовите окно из главного ViewModel:

```csharp
public ICommand ShowFindToolCommand => new RelayCommand(() =>
{
    var findTool = ServiceLocator.GetInstance<FindToolViewModel>();
    var window = new FindToolView { DataContext = findTool };
    window.ShowDialog();
});
```


Операция \"GetVersion\" в контракте \"ICheckForUpdateApi\" использует GET, но также содержит параметр текста сообщения \"credentials\". Операции GET не могут содержать текст сообщения. Либо измените параметр \"credentials\" на параметр UriTemplate, либо перейдите с WebGetAttribute на WebInvokeAttribute."}

Понял ситуацию. У вас есть 407 ошибка (требуется аутентификация прокси), и вы пытаетесь добавить NetworkCredentials. Давайте разберёмся, как правильно настроить аутентификацию для вашего `CheckForUpdateClient`.

### Решение для 407 ошибки (Proxy Authentication Required)

1. **Проверка текущего кода**:
   Вы правильно добавляете NetworkCredentials, но есть несколько моментов, которые нужно проверить:

```csharp
var credential = new NetworkCredential(
    userName: "ekra", 
    password: "test");
```

2. **Модификация клиента**:
   Нужно модифицировать `CheckForUpdateClient` для правильной обработки аутентификации:

```csharp
public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
{
    public CheckForUpdateClient(string address, NetworkCredential credentials)
        : base(new WebHttpBinding() { 
            MaxReceivedMessageSize = int.MaxValue,
            Security = new WebHttpSecurity {
                Mode = WebHttpSecurityMode.TransportCredentialOnly,
                Transport = new HttpTransportSecurity {
                    ClientCredentialType = HttpClientCredentialType.Basic,
                    ProxyCredentialType = HttpProxyCredentialType.Basic
                }
            }
        }, new EndpointAddress(address))
    {
        this.Endpoint.Behaviors.Add(new WebHttpBehavior());
        
        // Устанавливаем credentials
        this.ClientCredentials.UserName.UserName = credentials.UserName;
        this.ClientCredentials.UserName.Password = credentials.Password;
        
        // Для прокси-аутентификации
        if (WebRequest.DefaultWebProxy != null)
        {
            WebRequest.DefaultWebProxy.Credentials = credentials;
        }
    }

    // ... остальные методы
}
```

3. **Использование клиента с аутентификацией**:
   В методе `BwOnDoWork`:

```csharp
try
{
    var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower();
    var credentials = new NetworkCredential("ekra", "test");
    ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString(), credentials);
    resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credentials, lang: "ru");
}
```

### Дополнительные проверки:

1. **Проверьте тип аутентификации**:
   - Basic
   - NTLM
   - Digest
   - Negotiate

   Возможно, нужно изменить `HttpClientCredentialType`:

```csharp
ClientCredentialType = HttpClientCredentialType.Ntlm // или другой тип
```

2. **Логирование трафика** (для отладки):
   Добавьте в конфигурацию:

```xml
<system.diagnostics>
    <sources>
        <source name="System.ServiceModel" switchValue="Information, ActivityTracing">
            <listeners>
                <add name="traceListener" 
                     type="System.Diagnostics.XmlWriterTraceListener" 
                     initializeData="WcfTrace.svclog"/>
            </listeners>
        </source>
    </sources>
</system.diagnostics>
```

3. **Проверка URL и прокси**:
   Убедитесь, что:
   - URL нового сайта корректный
   - Прокси-сервер правильно настроен
   - Учетные данные верные

### Если проблема сохраняется:

1. **Обход прокси** (если возможно):
```csharp
webClient.Proxy = GlobalProxySelection.GetEmptyWebProxy();
```

2. **Использование HttpClient** (альтернативный вариант):
```csharp
using (var handler = new HttpClientHandler {
    Credentials = new NetworkCredential("ekra", "test"),
    Proxy = WebRequest.DefaultWebProxy
})
using (var client = new HttpClient(handler))
{
    // Ваш запрос
}
```

Попробуйте эти варианты и посмотрите, какой сработает в вашем случае. Если проблема останется, нужны более детальные данные о структуре запросов/ответов.


public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
    {
        public CheckForUpdateClient(string address)
            : base(new WebHttpBinding() { MaxReceivedMessageSize = int.MaxValue }, new EndpointAddress(address))
        {
            if (this.Endpoint.ListenUri.Scheme == "https")
            {
                // configure SSL
                var binding = (WebHttpBinding)this.Endpoint.Binding;
                binding.Security.Mode = WebHttpSecurityMode.Transport;
            }

            this.Endpoint.Behaviors.Add(new WebHttpBehavior());
        }

        public VersionDetails GetVersion(string uid, string ver, NetworkCredential credentials, string lang = "ru")
        {
            using (new OperationContextScope(this.InnerChannel))
            {
                var test = base.Channel.GetVersion(uid, ver, credentials, lang);
                return test;
            }
        }

        private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                            Credential = new NetworkCredential(
                                userName: "ekra",
                                password: "test")
                        },
                    };

                    VersionDetails resp = null;

                    var credential = new NetworkCredential(
                                userName: "",
                                password: "");
                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credential, lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }

Вот модифицированный код на русском для обработки JSON и отображения информации о новой версии:

private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                            Credential = new NetworkCredential(
                                userName: "ekra",
                                password: "test")
                        },
                    };

                    VersionDetails resp = null;

                    var credential = new NetworkCredential(
                                userName: "",
                                password: "");
                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credential, lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }


    public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
    {
        public CheckForUpdateClient(string address)
            : base(new WebHttpBinding() { MaxReceivedMessageSize = int.MaxValue }, new EndpointAddress(address))
        {
            if (this.Endpoint.ListenUri.Scheme == "https")
            {
                // configure SSL
                var binding = (WebHttpBinding)this.Endpoint.Binding;
                binding.Security.Mode = WebHttpSecurityMode.Transport;
            }

            this.Endpoint.Behaviors.Add(new WebHttpBehavior());
        }

        public VersionDetails GetVersion(string uid, string ver, NetworkCredential credentials, string lang = "ru")
        {
            using (new OperationContextScope(this.InnerChannel))
            {
                var test = base.Channel.GetVersion(uid, ver, credentials, lang);
                return test;
            }
        }
### Доработки в ViewModel:

Добавьте в ваш `UpdateDialogViewModel` следующие свойства:

```csharp
public Version Remote { get; private set; }
public DateTime Released { get; private set; }
public string Notes { get; private set; }
public Uri DownloadUri { get; private set; }

// Метод для обработки результата проверки
private void ProcessUpdateResult(CheckForUpdatesResult result)
{
    State = result.State;
    
    if (result.State == UpdateState.Available)
    {
        Remote = result.RemoteVersion;
        Released = result.ReleaseDate;
        Notes = result.Notes;
        DownloadUri = result.DownloadUrl;
        
        // Можно добавить логирование
        Logger.Info($"Доступна новая версия: {Remote} от {Released:d}");
    }
}
```

Теперь ваше приложение будет:
1. Получать данные о версиях в JSON-формате
2. Корректно парсить и сравнивать версии
3. Отображать всю информацию о новой версии в диалоговом окне
4. Показывать список изменений в удобном формате

Для работы кода нужно добавить NuGet пакет Newtonsoft.Json и настроить обработку ошибок подключения.
<catel:DataWindow x:Class="Module.Updater.Dialogs.UpdateDialogView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:catel="http://catel.codeplex.com"
                  xmlns:dialogs="clr-namespace:Module.Updater.Dialogs"
                  xmlns:properties="clr-namespace:Module.Updater.Properties"
                  xmlns:kivi="http://kivi.dev.ekra.ru"
                  xmlns:pilot="http://pilot.dev.ekra.ru"
                  Icon="/Module.Updater;component/Resources/Images/application_update.png" 
                  Height="400" Width="600"
                  kivi:CommandBindings.CommandBindings="{Binding CommandBindings}">

    <catel:DataWindow.Resources>
        <ResourceDictionary>
            <!--<ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="/Catel.MVVM;component/themes/generic.xaml" />
            </ResourceDictionary.MergedDictionaries>-->

            <BooleanToVisibilityConverter x:Key="bool2vis"/>
            <kivi:NegatedBooleanToVisibilityConverter x:Key="notbool2vis"/>
            <dialogs:NegateBooleanConverter x:Key="negatebool"/>
            <dialogs:UpdateStateConverter x:Key="state2vis"/>
            <dialogs:UpdateState2EnableConverter x:Key="state2ena"/>
            <pilot:LocalizableStringConverter x:Key="LocalizableStringConverter" ResourceManager="{x:Static properties:Resources.ResourceManager}"/>

        </ResourceDictionary>
    </catel:DataWindow.Resources>

    <Grid Margin="6">
        <DockPanel LastChildFill="True">
            <Grid DockPanel.Dock="Top">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="100"/>
                    <ColumnDefinition/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <TextBlock Text="{x:Static properties:Resources.Check_for_update_interval_}" VerticalAlignment="Center"/>
                <ComboBox Grid.Column="1" SelectedItem="{Binding Interval}" ItemsSource="{Binding Intervals}">
                    <ComboBox.ItemTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding Path=., Mode=OneWay, Converter={StaticResource LocalizableStringConverter}}"/>
                        </DataTemplate>
                    </ComboBox.ItemTemplate>
                </ComboBox>
                <!--<CheckBox Grid.Column="2" IsChecked="{Binding HasUpdate}"/>-->
                <Button Grid.Column="3" Command="{Binding CheckForUpdatesCommand}" Content="{x:Static properties:Resources.Check_For_Updates}"
                        IsEnabled="{Binding State, Converter={StaticResource state2ena}}">
                </Button>
            </Grid>
            <Grid DockPanel.Dock="Bottom">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <CheckBox Grid.Column="0" IsChecked="{Binding IgnoreRevision}" Margin="10,0" VerticalAlignment="Center"
                          Content="{x:Static properties:Resources.IgnoreRevNum}">
                    <CheckBox.ToolTip>
                        <TextBlock Text="{x:Static properties:Resources.IgnoreRevNumTip}" MaxWidth="300" TextWrapping="Wrap"/>
                    </CheckBox.ToolTip>
                </CheckBox>
            </Grid>
          <Grid Margin="0,10">
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.Busy}}">
                <TextBlock Text="{x:Static properties:Resources.Wait_}"/>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.NotAvailable}}">
                <TextBlock Text="{x:Static properties:Resources.No_updates_available}"/>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.ConnectionError}}">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition/>
                </Grid.RowDefinitions>

                <Grid Grid.Row="0">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="Auto"/>
                        <ColumnDefinition/>
                    </Grid.ColumnDefinitions>
                    <Image Source="/Module.Updater;component/Resources/Images/error.png" Width="16" Margin="0,0,3,0"/>
                    <TextBlock Text="{x:Static properties:Resources.Connection_error_}" Grid.Column="1"/>
                </Grid>
                <Label Grid.Row="1">
                    <Hyperlink Command="{x:Static NavigationCommands.GoToPage}">
                        <Run Text="{x:Static properties:Resources.Click_to_open_web_site}" ToolTip="{x:Static properties:Resources.Click_to_open_web_site}"/>
                    </Hyperlink>
                    <Label.ContextMenu>
                        <ContextMenu>
                            <MenuItem Command="{x:Static ApplicationCommands.Copy}" Icon="/Module.Updater;component/Resources/Images/page_copy.png"/>
                        </ContextMenu>
                    </Label.ContextMenu>
                </Label>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.Available}}">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Hei
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    {"Операция \"GetVersion\" в контракте \"ICheckForUpdateApi\" использует GET, но также содержит параметр текста сообщения \"credentials\". Операции GET не могут содержать текст сообщения. Либо измените параметр \"credentials\" на параметр UriTemplate, либо перейдите с WebGetAttribute на WebInvokeAttribute."}

Понял ситуацию. У вас есть 407 ошибка (требуется аутентификация прокси), и вы пытаетесь добавить NetworkCredentials. Давайте разберёмся, как правильно настроить аутентификацию для вашего `CheckForUpdateClient`.

### Решение для 407 ошибки (Proxy Authentication Required)

1. **Проверка текущего кода**:
   Вы правильно добавляете NetworkCredentials, но есть несколько моментов, которые нужно проверить:

```csharp
var credential = new NetworkCredential(
    userName: "ekra", 
    password: "test");
```

2. **Модификация клиента**:
   Нужно модифицировать `CheckForUpdateClient` для правильной обработки аутентификации:

```csharp
public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
{
    public CheckForUpdateClient(string address, NetworkCredential credentials)
        : base(new WebHttpBinding() { 
            MaxReceivedMessageSize = int.MaxValue,
            Security = new WebHttpSecurity {
                Mode = WebHttpSecurityMode.TransportCredentialOnly,
                Transport = new HttpTransportSecurity {
                    ClientCredentialType = HttpClientCredentialType.Basic,
                    ProxyCredentialType = HttpProxyCredentialType.Basic
                }
            }
        }, new EndpointAddress(address))
    {
        this.Endpoint.Behaviors.Add(new WebHttpBehavior());
        
        // Устанавливаем credentials
        this.ClientCredentials.UserName.UserName = credentials.UserName;
        this.ClientCredentials.UserName.Password = credentials.Password;
        
        // Для прокси-аутентификации
        if (WebRequest.DefaultWebProxy != null)
        {
            WebRequest.DefaultWebProxy.Credentials = credentials;
        }
    }

    // ... остальные методы
}
```

3. **Использование клиента с аутентификацией**:
   В методе `BwOnDoWork`:

```csharp
try
{
    var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower();
    var credentials = new NetworkCredential("ekra", "test");
    ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString(), credentials);
    resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credentials, lang: "ru");
}
```

### Дополнительные проверки:

1. **Проверьте тип аутентификации**:
   - Basic
   - NTLM
   - Digest
   - Negotiate

   Возможно, нужно изменить `HttpClientCredentialType`:

```csharp
ClientCredentialType = HttpClientCredentialType.Ntlm // или другой тип
```

2. **Логирование трафика** (для отладки):
   Добавьте в конфигурацию:

```xml
<system.diagnostics>
    <sources>
        <source name="System.ServiceModel" switchValue="Information, ActivityTracing">
            <listeners>
                <add name="traceListener" 
                     type="System.Diagnostics.XmlWriterTraceListener" 
                     initializeData="WcfTrace.svclog"/>
            </listeners>
        </source>
    </sources>
</system.diagnostics>
```

3. **Проверка URL и прокси**:
   Убедитесь, что:
   - URL нового сайта корректный
   - Прокси-сервер правильно настроен
   - Учетные данные верные

### Если проблема сохраняется:

1. **Обход прокси** (если возможно):
```csharp
webClient.Proxy = GlobalProxySelection.GetEmptyWebProxy();
```

2. **Использование HttpClient** (альтернативный вариант):
```csharp
using (var handler = new HttpClientHandler {
    Credentials = new NetworkCredential("ekra", "test"),
    Proxy = WebRequest.DefaultWebProxy
})
using (var client = new HttpClient(handler))
{
    // Ваш запрос
}
```

Попробуйте эти варианты и посмотрите, какой сработает в вашем случае. Если проблема останется, нужны более детальные данные о структуре запросов/ответов.


public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
    {
        public CheckForUpdateClient(string address)
            : base(new WebHttpBinding() { MaxReceivedMessageSize = int.MaxValue }, new EndpointAddress(address))
        {
            if (this.Endpoint.ListenUri.Scheme == "https")
            {
                // configure SSL
                var binding = (WebHttpBinding)this.Endpoint.Binding;
                binding.Security.Mode = WebHttpSecurityMode.Transport;
            }

            this.Endpoint.Behaviors.Add(new WebHttpBehavior());
        }

        public VersionDetails GetVersion(string uid, string ver, NetworkCredential credentials, string lang = "ru")
        {
            using (new OperationContextScope(this.InnerChannel))
            {
                var test = base.Channel.GetVersion(uid, ver, credentials, lang);
                return test;
            }
        }

        private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                            Credential = new NetworkCredential(
                                userName: "ekra",
                                password: "test")
                        },
                    };

                    VersionDetails resp = null;

                    var credential = new NetworkCredential(
                                userName: "",
                                password: "");
                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credential, lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }

Вот модифицированный код на русском для обработки JSON и отображения информации о новой версии:

private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                            Credential = new NetworkCredential(
                                userName: "ekra",
                                password: "test")
                        },
                    };

                    VersionDetails resp = null;

                    var credential = new NetworkCredential(
                                userName: "",
                                password: "");
                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), credential, lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }


    public class CheckForUpdateClient : ClientBase<ICheckForUpdateApi>, ICheckForUpdateApi
    {
        public CheckForUpdateClient(string address)
            : base(new WebHttpBinding() { MaxReceivedMessageSize = int.MaxValue }, new EndpointAddress(address))
        {
            if (this.Endpoint.ListenUri.Scheme == "https")
            {
                // configure SSL
                var binding = (WebHttpBinding)this.Endpoint.Binding;
                binding.Security.Mode = WebHttpSecurityMode.Transport;
            }

            this.Endpoint.Behaviors.Add(new WebHttpBehavior());
        }

        public VersionDetails GetVersion(string uid, string ver, NetworkCredential credentials, string lang = "ru")
        {
            using (new OperationContextScope(this.InnerChannel))
            {
                var test = base.Channel.GetVersion(uid, ver, credentials, lang);
                return test;
            }
        }
### Доработки в ViewModel:

Добавьте в ваш `UpdateDialogViewModel` следующие свойства:

```csharp
public Version Remote { get; private set; }
public DateTime Released { get; private set; }
public string Notes { get; private set; }
public Uri DownloadUri { get; private set; }

// Метод для обработки результата проверки
private void ProcessUpdateResult(CheckForUpdatesResult result)
{
    State = result.State;
    
    if (result.State == UpdateState.Available)
    {
        Remote = result.RemoteVersion;
        Released = result.ReleaseDate;
        Notes = result.Notes;
        DownloadUri = result.DownloadUrl;
        
        // Можно добавить логирование
        Logger.Info($"Доступна новая версия: {Remote} от {Released:d}");
    }
}
```

Теперь ваше приложение будет:
1. Получать данные о версиях в JSON-формате
2. Корректно парсить и сравнивать версии
3. Отображать всю информацию о новой версии в диалоговом окне
4. Показывать список изменений в удобном формате

Для работы кода нужно добавить NuGet пакет Newtonsoft.Json и настроить обработку ошибок подключения.
<catel:DataWindow x:Class="Module.Updater.Dialogs.UpdateDialogView"
                  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                  xmlns:catel="http://catel.codeplex.com"
                  xmlns:dialogs="clr-namespace:Module.Updater.Dialogs"
                  xmlns:properties="clr-namespace:Module.Updater.Properties"
                  xmlns:kivi="http://kivi.dev.ekra.ru"
                  xmlns:pilot="http://pilot.dev.ekra.ru"
                  Icon="/Module.Updater;component/Resources/Images/application_update.png" 
                  Height="400" Width="600"
                  kivi:CommandBindings.CommandBindings="{Binding CommandBindings}">

    <catel:DataWindow.Resources>
        <ResourceDictionary>
            <!--<ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="/Catel.MVVM;component/themes/generic.xaml" />
            </ResourceDictionary.MergedDictionaries>-->

            <BooleanToVisibilityConverter x:Key="bool2vis"/>
            <kivi:NegatedBooleanToVisibilityConverter x:Key="notbool2vis"/>
            <dialogs:NegateBooleanConverter x:Key="negatebool"/>
            <dialogs:UpdateStateConverter x:Key="state2vis"/>
            <dialogs:UpdateState2EnableConverter x:Key="state2ena"/>
            <pilot:LocalizableStringConverter x:Key="LocalizableStringConverter" ResourceManager="{x:Static properties:Resources.ResourceManager}"/>

        </ResourceDictionary>
    </catel:DataWindow.Resources>

    <Grid Margin="6">
        <DockPanel LastChildFill="True">
            <Grid DockPanel.Dock="Top">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="100"/>
                    <ColumnDefinition/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <TextBlock Text="{x:Static properties:Resources.Check_for_update_interval_}" VerticalAlignment="Center"/>
                <ComboBox Grid.Column="1" SelectedItem="{Binding Interval}" ItemsSource="{Binding Intervals}">
                    <ComboBox.ItemTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding Path=., Mode=OneWay, Converter={StaticResource LocalizableStringConverter}}"/>
                        </DataTemplate>
                    </ComboBox.ItemTemplate>
                </ComboBox>
                <!--<CheckBox Grid.Column="2" IsChecked="{Binding HasUpdate}"/>-->
                <Button Grid.Column="3" Command="{Binding CheckForUpdatesCommand}" Content="{x:Static properties:Resources.Check_For_Updates}"
                        IsEnabled="{Binding State, Converter={StaticResource state2ena}}">
                </Button>
            </Grid>
            <Grid DockPanel.Dock="Bottom">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <CheckBox Grid.Column="0" IsChecked="{Binding IgnoreRevision}" Margin="10,0" VerticalAlignment="Center"
                          Content="{x:Static properties:Resources.IgnoreRevNum}">
                    <CheckBox.ToolTip>
                        <TextBlock Text="{x:Static properties:Resources.IgnoreRevNumTip}" MaxWidth="300" TextWrapping="Wrap"/>
                    </CheckBox.ToolTip>
                </CheckBox>
            </Grid>
          <Grid Margin="0,10">
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.Busy}}">
                <TextBlock Text="{x:Static properties:Resources.Wait_}"/>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.NotAvailable}}">
                <TextBlock Text="{x:Static properties:Resources.No_updates_available}"/>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.ConnectionError}}">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition/>
                </Grid.RowDefinitions>

                <Grid Grid.Row="0">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="Auto"/>
                        <ColumnDefinition/>
                    </Grid.ColumnDefinitions>
                    <Image Source="/Module.Updater;component/Resources/Images/error.png" Width="16" Margin="0,0,3,0"/>
                    <TextBlock Text="{x:Static properties:Resources.Connection_error_}" Grid.Column="1"/>
                </Grid>
                <Label Grid.Row="1">
                    <Hyperlink Command="{x:Static NavigationCommands.GoToPage}">
                        <Run Text="{x:Static properties:Resources.Click_to_open_web_site}" ToolTip="{x:Static properties:Resources.Click_to_open_web_site}"/>
                    </Hyperlink>
                    <Label.ContextMenu>
                        <ContextMenu>
                            <MenuItem Command="{x:Static ApplicationCommands.Copy}" Icon="/Module.Updater;component/Resources/Images/page_copy.png"/>
                        </ContextMenu>
                    </Label.ContextMenu>
                </Label>
            </Grid>
            <Grid Visibility="{Binding State, Converter={StaticResource state2vis}, ConverterParameter={x:Static dialogs:UpdateState.Available}}">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition/>
                </Grid.RowDefinitions>

                <TextBlock Grid.Row="0" Text="{x:Static properties:Resources.New_version_available}" FontWeight="Bold"/>
                <!--<TextBlock Grid.Row="1" Text="{Binding Local, StringFormat={x:Static properties:Resources.LocalVerFormat}}"/>-->
                <TextBlock Grid.Row="2" Text="{Binding Remote, StringFormat={x:Static properties:Resources.RemoteVerFormat}}"/>
                <!--<TextBlock Grid.Row="3" Text="Update?"/>-->
                <Grid Grid.Row="3" Margin="0,10">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="Auto"/>
                        <ColumnDefinition/>
                        <!--<ColumnDefinition Width="Auto"/>-->
                    </Grid.ColumnDefinitions>
                    <Image Source="/Module.Updater;component/Resources/Images/error.png" Width="16" Margin="0,0,3,0"/>
                    <TextBlock Grid.Column="1">
                        <Run Text="{x:Static properties:Resources.Manual_update_offer}"></Run>
                        <Hyperlink Command="{x:Static NavigationCommands.GoToPage}">
                            <Hyperlink.ContextMenu>
                                <ContextMenu>
                                    <MenuItem Command="{x:Static ApplicationCommands.Copy}" Icon="/Module.Updater;component/Resources/Images/page_copy.png"/>
                                </ContextMenu>
                            </Hyperlink.ContextMenu>
                            <Run Text="{x:Static properties:Resources.Visit_us}" ToolTip="{x:Static properties:Resources.Click_to_open_folder}"/>
                        </Hyperlink>
                        <Run>.</Run>
                    </TextBlock>
                    <!--<Button Grid.Column="2" Command="{Binding DownloadUpdatesCommand}">
                        <StackPanel Orientation="Horizontal">
                            <Image Source="/Module.Updater;component/Resources/Images/download.png" Width="16" Margin="0,0,3,0"/>
                            <TextBlock Text="{x:Static properties:Resources.Download}"/>
                        </StackPanel>
                    </Button>-->
                </Grid>
                <TextBlock Grid.Row="4" Text="{x:Static properties:Resources.Changelog_}" FontWeight="Bold"/>
                <kivi:MarkdownBox Grid.Row="5" Source="{Binding Notes}" IsReadOnly="True" BorderThickness="0" ZoomBarVisibility="Collapsed"/>
            </Grid>
          </Grid>
        </DockPanel>
    </Grid>
</catel:DataWindow>


{
  "product": "scle",
  "version": "1.2.0.7719",
  "released": "2024-04-09 13:16:00",
  "description": "Улучшения:\n- Добавлена возможность лицензирования с USB ключом без кода оборудования;\n- Доработана визуальная часть меню Файл\nИсправления:\n- Исправлен баг загрузки стартовой страницы;",
  "url": "/soft/scle/downloads/software/scl_express/scle_setup.exe",
  "Theme": "Основное программное обеспечение",
  "Section": "SCL Express",
  "error": "OK"
}


using System;
using System.IO;
using System.Reflection;
using System.Runtime.Serialization;
using System.ServiceModel;
using System.ServiceModel.Web;

namespace Module.Updater.Rest
{
    [ServiceContract]
    public interface ICheckForUpdateApi
    {
        [OperationContract]
        [WebGet(UriTemplate = "check/{uid}?ver={ver}&lang={lang}", ResponseFormat = WebMessageFormat.Json)]
        VersionDetails GetVersion(string uid, string ver, string lang = "ru");

        [OperationContract]
        [WebGet(UriTemplate = "download/{uid}?ver={ver}")]
        Stream Download(string uid, string ver);

        [OperationContract]
        [WebInvoke(Method = "POST", UriTemplate = "upload/{uid}?filename={filename}")]
        void Upload(string uid, string filename, Stream stream);

        [OperationContract]
        [WebGet]
        string Echo(string input);
    }

    [Obfuscation(Exclude = true)]
    [DataContract(Name = "details", Namespace = "http://schemas.ekra.ru/2019/11/PilotApp")]
    public class VersionDetails
    {
        [DataMember(Name = "product")]
        public string Name;

        [DataMember(Name = "version")]
        public string Version;

        [DataMember(Name = "released")]
        public string Released { get; set; }

        [DataMember(Name = "description")]
        public string Description { get; set; }

        [DataMember(Name = "error")]
        public string ErrorMsg { get; set; }

        [DataMember(Name = "url")]
        public string Url;

        public override string ToString()
        {
            return string.Format("name = '{0}'; version = {1}; url = {2}", Name, Version, Url);
        }
    }

    [DataContract]
    public class RemoteFileInfo : IDisposable
    {
        [DataMember(Name = "name")]
        public string FileName;

        [DataMember(Name = "length")]
        public long FileLength;

        [DataMember(Name = "data", Order = 1)]
        public Stream FileStream;

        public void Dispose()
        {
            if (FileStream != null)
            {
                FileStream.Close();
                FileStream = null;
            }
        }
    }
}

using System;
using System.ComponentModel;
using System.ComponentModel.Composition;
using System.Configuration;
using System.Diagnostics;
using System.Globalization;
using System.IO;
using System.Text;
using Module.Updater.Dialogs;
using Module.Updater.Rest;
using Pilot.Licensing;
using Pilot.Shell.Infrastructure.Logging;
using Pilot.Shell.Infrastructure.Services;

namespace Module.Updater
{
    [Export(typeof(IUpdateController))]
    [PartCreationPolicy(CreationPolicy.Shared)]
    public class UpdateControllerImpl : IUpdateController
    {
        private static readonly ILog Log = LogManager.GetCurrentClassLogger();
        private readonly IUIVisualizerService _uiVisualizerService;
        private readonly IWorkspaceManager _workspaceManager;
        private static System.Windows.Threading.DispatcherTimer timer = new System.Windows.Threading.DispatcherTimer();
        private UpdateDialogViewModel _dialog;
        private DateTime _lastCheck;
        private static UpdateIntervalKind _intervalKind;
        private static bool _ignoreRevision;
        private int _tickCount;
        private const string DefaultUrl = /*"https://soft.ekra.ru/scle/software/"*/ "https://soft-ekra.tw1.ru/soft/scle/updates/check?uid=%22amdjsp-l6bc6j-t6c14j-hzy4v9%22";

#pragma warning disable 0649
        [Import]
        private ExportFactory<UpdateDialogViewModel> _updateDialog;
#pragma warning restore 0649

        [ImportingConstructor]
        public UpdateControllerImpl(IWorkspaceManager workspaceManager, IUIVisualizerService uiVisualizerService)
        {
            _workspaceManager = workspaceManager;
            _uiVisualizerService = uiVisualizerService;

            _workspaceManager.ShellClosing += WorkspaceManagerOnShellClosing;

            LoadSettings();

            // keep the current value
            var intervalKind = GetInterval();

            if (intervalKind != UpdateIntervalKind.OnStartup)
                SetInterval(UpdateIntervalKind.Never); // send an anonymous update request even when 'Never' selected
            else
                SetInterval(UpdateIntervalKind.OnStartup);

            timer.Tick += (sender, args) =>
            {
                _tickCount++;

                #region MyRegion
                if (_tickCount == 1)
                {
                    // restore the interval value at first tick
                    SetInterval(intervalKind);

                    var current = timer.Interval;
                    var elapsed = DateTime.Now - _lastCheck;
                    Log.Info("Last check for updates time '{0}'.", _lastCheck);
                    if (elapsed < current)
                    {
                        timer.Interval = current - elapsed;
                        Log.Debug("Adjust update timer interval with '{0}'.", timer.Interval);
                    }
                }
                else
                {
                    // re-apply the interval value at every ticks
                    SetInterval(_intervalKind);
                }
                #endregion

                if (_intervalKind != UpdateIntervalKind.Never)
                    _lastCheck = DateTime.Now;

                Log.Info("Begin check for updates...");

                CheckForUpdates(true);
            };

#if !DEBUG
            // start the update timer for Never or OnStartup interval selected
            timer.Start();
#endif
        }

        private void WorkspaceManagerOnShellClosing(object sender, CancelEventArgs e)
        {
            SaveSettings();
        }

        public UpdateIntervalKind GetInterval()
        {
            return _intervalKind;
        }

        public void SetInterval(UpdateIntervalKind kind)
        {
            _intervalKind = kind;

            Log.Debug("Setup check for updates interval '{0}'", kind);

            timer.Stop();

            switch (kind)
            {
                case UpdateIntervalKind.Never:
                    timer.Interval = TimeSpan.FromSeconds(3);
                    break;
                case UpdateIntervalKind.OnStartup:
                    timer.Interval = TimeSpan.FromSeconds(10);
                    break;
                case UpdateIntervalKind.Hour:
                    timer.Interval = TimeSpan.FromHours(1);
                    timer.Start();
                    break;
                case UpdateIntervalKind.Day:
                    timer.Interval = TimeSpan.FromDays(1);
                    timer.Start();
                    break;
                case UpdateIntervalKind.Week:
                    timer.Interval = TimeSpan.FromDays(7);
                    timer.Start();
                    break;
            }

            Log.Debug("Setup update timer interval with '{0}'. Update timer is {1}.", timer.Interval, timer.IsEnabled? "started" : "stopped");
        }

        public object GetProperty(string propertyName)
        {
            switch (propertyName)
            {
                case "Interval":
                    return _intervalKind;
                case "IgnoreRevision":
                    return _ignoreRevision;
            }
            return null;
        }

        public void SetProperty(string propertyName, object propertValue)
        {
            switch (propertyName)
            {
                case "Interval":
                    _intervalKind = (UpdateIntervalKind)propertValue; return;
                case "IgnoreRevision":
                    _ignoreRevision = (bool)propertValue; return;
            }
        }

        private void LoadSettings()
        {
            if (!DateTime.TryParse(Properties.Settings.Default.LastCheck, CultureInfo.InvariantCulture, DateTimeStyles.None, out _lastCheck))
            {
                _lastCheck = DateTime.Now;
            }

            if (!Enum.TryParse(Properties.Settings.Default.Interval, out _intervalKind))
            {
                _intervalKind = UpdateIntervalKind.Hour;
            }

            if (!bool.TryParse(Properties.Settings.Default.IgnoreRevision, out _ignoreRevision))
            {
                _ignoreRevision = false;
            }
        }

        private void SaveSettings()
        {
            Properties.Settings.Default.LastCheck = DateTime.Now.ToString(CultureInfo.InvariantCulture);
            Properties.Settings.Default.Interval = _intervalKind.ToString();
            Properties.Settings.Default.IgnoreRevision = _ignoreRevision.ToString();
            Properties.Settings.Default.Save();
        }

        public bool CheckForUpdates(bool silent)
        {
            if (_dialog == null || _dialog.IsClosed)
            {
                // create a new instance of dialog VM every time
                _dialog = _updateDialog.CreateExport().Value;
            }

            //if (silent || _intervalKind != UpdateIntervalKind.Never)
            {
                if (_dialog.State == UpdateState.Busy)
                    return false;

                _dialog.Url = DefaultUrl;
                _dialog.State = UpdateState.Busy;

                BackgroundWorker bw = new BackgroundWorker();
                bw.DoWork += BwOnDoWork;
                bw.RunWorkerCompleted += BwOnRunWorkerCompleted;
                bw.RunWorkerAsync();
            }

            if (!silent)
            {
                ShowDialog();
            }

            return true;
        }

        private static void BwOnDoWork(object sender, DoWorkEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.DoWork -= BwOnDoWork;

            CheckForUpdatesResult res = null;
            var config = ConfigurationManager.GetSection("updater") as UpdaterConfigSection;
            if (config != null)
            {
                var appInfo = GetAppFileInfo();
                var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
                foreach (UpdaterElement element in config.Elements)
                {
                    var baseUri = new Uri(element.Url);

                    Log.Debug("Send request to '{0}'", baseUri);

                    RequestMessage req = new RequestMessage
                    {
                        Product = appInfo.ProductName,
                        User = System.Environment.UserName,
                        Data = new RequestMessageData
                        {
                            Name = System.Environment.MachineName,
                            User = System.Environment.UserName,
                            OS = System.Environment.OSVersion.ToString(),
                            Is64Bit = System.Environment.Is64BitOperatingSystem,
                            UpdateInterval = _intervalKind.ToString(),
                        },
                    };

                    VersionDetails resp = null;

                    try
                    {
                        var appKey = LicenseHandler.GenerateUID("UnifiedPilotApp").ToLower(); //TODO: remove hard reference
                        ICheckForUpdateApi webClient = new CheckForUpdateClient(baseUri.ToString());
                        resp = webClient.GetVersion(appKey, ver: appVer.ToString(), lang: "ru");
                    }
                    catch (Exception ex)
                    {
                        Log.Info(ex.Message);
                    }

                    if (resp != null && resp.ErrorMsg == "OK")
                    {
                        Log.Debug("Received valid response. Latest version: {0}", resp.Version);

                        Version newest;
                        if (!Version.TryParse(resp.Version, out newest))
                            newest = new Version();

                        res = new CheckForUpdatesResult(true)
                        {
                            RemoteVer = newest,
                            RemoteMajor = newest.Major,
                            RemoteMinor = newest.Minor,
                            RemoteBuild = newest.Build,
                            RemoteRevision = newest.Revision,
                            RemoteDate = resp.Released,
                            RemoteNotes = resp.Description,
                            RemoteUrl = resp.Url,
                        };
                        break;
                    }
                    else
                    {
                        Log.Debug("Received invalid response!");
                    }
                }
            }
            e.Result = res;
        }

        private void BwOnRunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
        {
            var bw = (BackgroundWorker)sender;
            bw.RunWorkerCompleted -= BwOnRunWorkerCompleted;

            if (e.Cancelled)
                return;

            var res = e.Result as CheckForUpdatesResult;
            if (res == null || !res.Result)
            {
                _dialog.State = UpdateState.ConnectionError;
                return;
            }

            var remoteVer = res.RemoteVer;
            var appInfo = GetAppFileInfo();
            var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);

            var remote = (_dialog.IgnoreRevision)
                ? new Version(remoteVer.Major, remoteVer.Minor, remoteVer.Build)
                : remoteVer;

            var local = (_dialog.IgnoreRevision)
                ? new Version(appVer.Major, appVer.Minor, appVer.Build)
                : appVer;

#if DEBUG
            // uncomment the following line to display the remote information anyway
            //local = new Version("0.0.0.0");
#endif

            if (remote > local)
            {
                _dialog.Local = appVer.ToString();
                _dialog.Remote = res.RemoteVer.ToString();
                _dialog.Url = res.RemoteUrl;
                _dialog.Notes = res.RemoteNotes;
                _dialog.State = UpdateState.Available;

                if (_intervalKind != UpdateIntervalKind.Never)
                {
                    // do not show dialog when we send anonymous update request
                    ShowDialog();
                }
            }
            else
            {
                _dialog.State = UpdateState.NotAvailable;
            }
        }

        /// <summary>
        /// Gets the application file info.
        /// </summary>
        /// <returns></returns>
        private static FileVersionInfo GetAppFileInfo()
        {
            var assembly = System.Reflection.Assembly.GetEntryAssembly();
            //var name = assembly.GetName();
            var appInfo = FileVersionInfo.GetVersionInfo(assembly.Location);
            //var appVer = new Version(appInfo.FileMajorPart, appInfo.FileMinorPart, appInfo.FileBuildPart, appInfo.ProductPrivatePart);
            return appInfo;
        }

        private void ShowDialog()
        {
            if (_dialog.IsInitialized)
                return;

            // create dialog instance
            if (_uiVisualizerService.ShowDialog(_dialog) == true)
            {
                return;
            }
        }
    }
}


Понял, игнорируем предыдущие обсуждения про `ParameterType` и `ParameterSectionType`. Давайте полностью пересоберем `FindToolViewModel` для нового задания - поиска по активному проекту с отображением результатов в диалоговом окне.

### Полная реализация FindToolViewModel

```csharp
[Export]
[PartCreationPolicy(CreationPolicy.NonShared)]
public class FindToolViewModel : ViewModelBase
{
    private readonly Lazy<IProjectManager> _projectManager;
    private string _searchText;
    private bool _isBusy;
    private bool _caseSensitive;
    private bool _searchInNames = true;
    private bool _searchInValues;

    public ObservableCollection<SearchResult> Results { get; } = new();
    public ICommand SearchCommand { get; }
    public ICommand OpenResultCommand { get; }

    [ImportingConstructor]
    public FindToolViewModel(Lazy<IProjectManager> projectManager)
    {
        _projectManager = projectManager;
        
        SearchCommand = new RelayCommand(ExecuteSearch, CanExecuteSearch);
        OpenResultCommand = new RelayCommand<SearchResult>(OpenSelectedResult);
    }

    public string SearchText
    {
        get => _searchText;
        set => SetProperty(ref _searchText, value);
    }

    public bool IsBusy
    {
        get => _isBusy;
        set => SetProperty(ref _isBusy, value);
    }

    public bool CaseSensitive
    {
        get => _caseSensitive;
        set => SetProperty(ref _caseSensitive, value);
    }

    public bool SearchInNames
    {
        get => _searchInNames;
        set => SetProperty(ref _searchInNames, value);
    }

    public bool SearchInValues
    {
        get => _searchInValues;
        set => SetProperty(ref _searchInValues, value);
    }

    private bool CanExecuteSearch()
    {
        return !IsBusy && 
               !string.IsNullOrWhiteSpace(SearchText) && 
               _projectManager.Value.ActiveProject != null;
    }

    private async void ExecuteSearch()
    {
        IsBusy = true;
        Results.Clear();

        try
        {
            await Task.Run(() =>
            {
                var project = _projectManager.Value.ActiveProject;
                if (project == null) return;

                var comparison = CaseSensitive 
                    ? StringComparison.Ordinal 
                    : StringComparison.OrdinalIgnoreCase;

                SearchInProject(project, comparison);
            });
        }
        finally
        {
            IsBusy = false;
        }
    }

    private void SearchInProject(Project project, StringComparison comparison)
    {
        // Поиск по узлам проекта
        foreach (var node in project.Nodes)
        {
            if (node is IProjectItem item)
            {
                CheckAndAddResult(item, comparison);
            }

            if (node is IProjectContainer container)
            {
                SearchInContainer(container, comparison);
            }
        }
    }

    private void SearchInContainer(IProjectContainer container, StringComparison comparison)
    {
        foreach (var child in container.Children)
        {
            if (child is IProjectItem item)
            {
                CheckAndAddResult(item, comparison);
            }

            if (child is IProjectContainer childContainer)
            {
                SearchInContainer(childContainer, comparison);
            }
        }
    }

    private void CheckAndAddResult(IProjectItem item, StringComparison comparison)
    {
        var matches = new List<string>();

        if (SearchInNames && Matches(item.Name, comparison))
        {
            matches.Add("Name");
        }

        if (SearchInValues && item is IValueItem valueItem && 
            Matches(valueItem.Value?.ToString(), comparison))
        {
            matches.Add("Value");
        }

        if (matches.Count > 0)
        {
            var result = new SearchResult
            {
                Item = item,
                Name = item.Name,
                Path = GetItemPath(item),
                MatchFields = string.Join(", ", matches),
                MatchScore = CalculateMatchScore(item, matches.Contains("Name"), matches.Contains("Value"))
            };

            Application.Current.Dispatcher.Invoke(() => Results.Add(result));
        }
    }

    private bool Matches(string text, StringComparison comparison)
    {
        return !string.IsNullOrEmpty(text) && 
               text.Contains(SearchText, comparison);
    }

    private string GetItemPath(IProjectItem item)
    {
        var path = new List<string>();
        var current = item.Parent;

        while (current != null)
        {
            path.Insert(0, current.Name);
            current = current.Parent;
        }

        return string.Join("/", path);
    }

    private double CalculateMatchScore(IProjectItem item, bool nameMatch, bool valueMatch)
    {
        // Простая логика расчета релевантности
        double score = 0;
        if (nameMatch) score += 0.7;
        if (valueMatch) score += 0.3;
        return score;
    }

    private void OpenSelectedResult(SearchResult result)
    {
        if (result?.Item != null)
        {
            _projectManager.Value.SelectItem(result.Item);
        }
    }
}

public class SearchResult
{
    public IProjectItem Item { get; set; }
    public string Name { get; set; }
    public string Path { get; set; }
    public string MatchFields { get; set; }
    public double MatchScore { get; set; }
}
```

### Соответствующий XAML (FindToolView.xaml)

```xml
<controls:MetroWindow x:Class="FindToolView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:controls="http://metro.mahapps.com/winfx/xaml/controls"
        Title="Project Search" 
        Height="600" Width="800"
        GlowBrush="{DynamicResource AccentColorBrush}">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Search Panel -->
        <StackPanel Grid.Row="0" Orientation="Horizontal">
            <TextBox controls:TextBoxHelper.Watermark="Search text..."
                     Text="{Binding SearchText, UpdateSourceTrigger=PropertyChanged}"
                     Width="300" 
                     Margin="0,0,10,0"/>
            <Button Content="Search" 
                    Command="{Binding SearchCommand}" 
                    Width="80"/>
            <CheckBox Content="Case sensitive" 
                      IsChecked="{Binding CaseSensitive}" 
                      Margin="10,0,0,0"
                      VerticalAlignment="Center"/>
        </StackPanel>

        <!-- Filters -->
        <StackPanel Grid.Row="1" Orientation="Horizontal" Margin="0,10">
            <CheckBox Content="Search in names" 
                      IsChecked="{Binding SearchInNames}" 
                      Margin="0,0,10,0"/>
            <CheckBox Content="Search in values" 
                      IsChecked="{Binding SearchInValues}"/>
        </StackPanel>

        <!-- Results -->
        <DataGrid Grid.Row="2" 
                 ItemsSource="{Binding Results}" 
                 AutoGenerateColumns="False"
                 SelectionMode="Single"
                 IsReadOnly="True">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Name" 
                                   Binding="{Binding Name}" 
                                   Width="200"/>
                <DataGridTextColumn Header="Path" 
                                   Binding="{Binding Path}" 
                                   Width="300"/>
                <DataGridTextColumn Header="Found in" 
                                   Binding="{Binding MatchFields}" 
                                   Width="150"/>
                <DataGridTextColumn Header="Relevance" 
                                   Binding="{Binding MatchScore, StringFormat={}{0:P0}}" 
                                   Width="80"/>
                <DataGridTemplateColumn Width="80">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="Open" 
                                    Command="{Binding Path=DataContext.OpenResultCommand, 
                                              RelativeSource={RelativeSource AncestorType=DataGrid}}"
                                    CommandParameter="{Binding}"
                                    Style="{StaticResource AccentedButtonStyle}"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</controls:MetroWindow>
```

### Ключевые особенности:

1. **Гибкий поиск**:
   - Поддержка поиска по именам и значениям
   - Учет регистра
   - Рекурсивный обход всей структуры проекта

2. **Информативные результаты**:
   - Отображение пути к объекту
   - Показатель релевантности
   - Указание, где найдено совпадение

3. **Интеграция**:
   - Навигация к найденным элементам через ProjectManager
   - Поддержка MVVM через инъекцию зависимостей

4. **Производительность**:
   - Асинхронное выполнение поиска
   - Постепенное добавление результатов

5. **UI-фичи**:
   - Водяной знак в поле поиска
   - Стилизованные кнопки
   - Сортировка по колонкам

Для использования просто вызовите окно из главного ViewModel:

```csharp
public ICommand ShowFindToolCommand => new RelayCommand(() =>
{
    var findTool = ServiceLocator.GetInstance<FindToolViewModel>();
    var window = new FindToolView { DataContext = findTool };
    window.ShowDialog();
});
```
