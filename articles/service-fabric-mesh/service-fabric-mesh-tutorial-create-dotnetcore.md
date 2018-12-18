---
title: 教學課程 - 建立、偵錯、部署及監視 Service Fabric Mesh 的多服務應用程式 | Microsoft Docs
description: 在本教學課程中，您會建立有 ASP.NET Core 網站可與後端 Web 服務進行通訊的多服務 Azure Service Fabric Mesh 應用程式、在本機加以偵錯，然後將其發佈至 Azure。
services: service-fabric-mesh
documentationcenter: .net
author: TylerMSFT
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/18/2018
ms.author: twhitney
ms.custom: mvc, devcenter
ms.openlocfilehash: 09112aafdbabf0cda2b3ae13af73a9223533a6e1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979188"
---
# <a name="tutorial-create-debug-deploy-and-upgrade-a-multi-service-service-fabric-mesh-app"></a>教學課程：建立、偵錯、部署及升級多服務的 Service Fabric Mesh 應用程式

本教學課程是一個系列的第一部分。 您將了解如何使用 Visual Studio 建立具有 ASP.NET Web 前端和 ASP.NET Core Web API 後端服務的 Azure Service Fabric Mesh 應用程式。 接著，您將在本機開發叢集上偵錯應用程式。 您會將應用程式發佈至 Azure，然後變更組態和程式碼並升級應用程式。 最後，您將會清除未使用的 Azure 資源，讓您不需支付未使用的資源。

完成時，您將逐步進行應用程式生命週期管理的大部分階段，並且建置應用程式，以示範 Service Fabric Mesh 應用程式中的服務對服務呼叫。

如果您不想以手動方式建立待辦事項應用程式，您可以針對已完成的應用程式[下載原始程式碼](https://github.com/azure-samples/service-fabric-mesh)，並直接跳到[在本機對應用程式偵錯](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)。

在系列的第一部分中，您將了解如何：

> [!div class="checklist"]
> * 使用 Visual Studio 建立包含 ASP.NET Web 前端的 Service Fabric Mesh 應用程式。
> * 建立用來代表待辦事項項目的模型。
> * 建立後端服務並從中擷取資料。
> * 為後端服務新增控制器和 DataContext，作為該模型檢視控制器模式的一部分。
> * 建立網頁以顯示待辦事項項目。
> * 建立可識別後端服務的環境變數

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * 在 Visual Studio 中建立 Service Fabric Mesh 應用程式
> * [偵錯在本機開發叢集中執行的 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [部署 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * [升級 Service Fabric Mesh 應用程式](service-fabric-mesh-tutorial-upgrade.md)
> * [清除 Service Fabric Mesh 資源](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，您可以在開始前[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* 確定您已[設定開發環境](service-fabric-mesh-howto-setup-developer-environment-sdk.md)，包括安裝 Service Fabric 執行階段、SDK、Docker 和 Visual Studio 2017。

## <a name="create-a-service-fabric-mesh-project-in-visual-studio"></a>在 Visual Studio 中建立 Service Fabric Mesh 專案

執行 Visual Studio，然後選取 [檔案] > [新增] > [專案...]。

在 [新增專案] 對話方塊頂端的 [搜尋] 方塊中，輸入 `mesh`。 選取 **Service Fabric Mesh 應用程式**範本。 如果您未看到此範本，請確定您已安裝 Mesh SDK 和 VS 工具預覽版，如[設定您的開發環境](service-fabric-mesh-howto-setup-developer-environment-sdk.md)中所說明。  

在 [名稱] 方塊中輸入 `todolistapp`，然後在 [位置] 方塊中，設定要用來儲存專案檔案的資料夾路徑。

確定已勾選 [為解決方案建立目錄]，然後按一下 [確定]，以建立 Service Fabric Mesh 專案。

![Visual Studio 的新增 Service Fabric Mesh 專案對話方塊](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-project.png)

接著，您會看到 [新增 Service Fabric 服務] 對話方塊。

### <a name="create-the-web-front-end-service"></a>建立 Web 前端服務

在 [新增 Service Fabric 服務] 對話方塊中，選取 [ASP.NET Core] 專案類型，並確定 [容器 OS] 設定為 [Windows]。

將 [服務名稱] 設定為 **WebFrontEnd**。 按 [確定] 以建立 ASP.NET Core 服務。

![Visual Studio 的新增 Service Fabric Mesh 專案對話方塊](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-service-fabric-service.png)

此時您會看到 [新增 ASP.NET Core Web 應用程式] 對話方塊。 在 [新增 ASP.NET Core Web 應用程式] 對話方塊中，選取 [Web 應用程式]，然後按一下 [確定]。

![Visual Studio 的新增 ASP.NET Core 應用程式](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-aspnetcore-app.png)

現在您已有 Service Fabric Mesh 應用程式。 接下來，請建立待辦事項資訊的模型。

## <a name="create-the-to-do-items-model"></a>建立待辦事項項目模型

為了簡單起見，待辦事項項目會儲存在記憶體中的清單內。 請建立待辦事項項目的類別庫，以及用來存放這些項目的清單。 在目前已載入 **todolistapp** 的 Visual Studio 中，選取 [檔案] > [新增] > [新增專案]。

在 [新增專案] 對話方塊頂端的 [搜尋] 方塊中，輸入 `C# .net core class`。 選取 [類別庫 (.NET Core)] 範本。

在 [名稱] 方塊中，輸入 `Model`。 按一下 [確定] 以建立類別庫。

在 [方案總管] 中的 [模型] 下方，以滑鼠右鍵按一下 **Class1.cs**，並選擇 [重新命名]。 將類別重新命名為 **ToDoItem.cs**。 如果出現提示詢問您是否要重新命名所有參考，請按一下 [是]。

將空白的 `class ToDoItem` 取代為：

```csharp
public class ToDoItem
{
    public string Description { get; set; }
    public int Index { get; set; }
    public bool Completed { get; set; }

    public ToDoItem(string description)
    {
        Description = description;
        Index = 0;
    }

    public static ToDoItem Load(string description, int index, bool completed)
    {
        ToDoItem newItem = new ToDoItem(description)
        {
            Index = index,
            Completed = completed
        };

        return newItem;
    }
}
```

此類別代表個別的待辦事項項目。

在 Visual Studio 中，以滑鼠右鍵按一下 [模型] 類別庫，然後選取 [新增] > [類別...]，以建立用來存放待辦事項項目的清單。 [新增項目] 對話方塊隨即出現。 將 [名稱] 設定為 `ToDoList.cs`，然後按一下 [新增]。

在 **ToDoList.cs** 中，將空的 `class ToDoList` 取代為：

```csharp
public class ToDoList
{
    private List<ToDoItem> _items;

    public string Name { get; set; }
    public IEnumerable<ToDoItem> Items { get => _items; }

    public ToDoList(string name)
    {
        Name = name;
        _items = new List<ToDoItem>();
    }

    public ToDoItem Add(string description)
    {
        var item = new ToDoItem(description);
        _items.Add(item);
        item.Index = _items.IndexOf(item);
        return item;
    }
    public void Add(ToDoItem item)
    {
        _items.Add(item);
        item.Index = _items.Count - 1;
    }

    public ToDoItem RemoveAt(int index)
    {
        if (index >= 0 && index < _items.Count)
        {
            var result = _items[index];
            _items.RemoveAt(index);

            // Reorder items
            for (int i = index; i < _items.Count; i++)
            {
                _items[i].Index = i;
            }

            return result;
        }
        else
        {
            throw new IndexOutOfRangeException();
        }
    }
}
```

接著，建立將會追蹤待辦事項項目的 Service Fabric 服務。

## <a name="create-the-back-end-service"></a>建立後端服務

在 Visual Studio 的 [方案總管] 視窗中，以滑鼠右鍵按一下 **todolistapp**，然後按一下 [新增] > [新增 Service Fabric 服務...]

[新增 Service Fabric 服務] 對話方塊隨即出現。 選取 [ASP.NET Core] 專案類型，並確定 [容器 OS] 設定為 [Windows]。

將 [服務名稱] 設定為 **ToDoService**。 按一下 [確定] 以建立 ASP.NET Core 服務。 此時會出現 [新增 ASP.NET Core Web 應用程式] 對話方塊。 在該對話方塊中選取 [API]，然後選取 [確定]，服務的專案即會新增至解決方案。

![Visual Studio 的新增 ASP.NET Core 應用程式](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-new-webapi.png)

由於後端服務未提供任何 UI，因此請關閉在服務啟動時啟動瀏覽器的功能。 在 [方案總管] 中，以滑鼠右鍵按一下 **ToDoService**，然後選取 [屬性]。 在出現的 [屬性] 視窗中，選取左側的 [偵錯] 索引標籤，然後取消核取 [啟動瀏覽器]。 按 **Ctrl+S** 以儲存變更。

此服務會維護待辦事項資訊，因此請新增模型類別庫的參考。 在 [方案總管] 中，以滑鼠右鍵按一下 **ToDoService**，然後選取 [新增] > [參考...]。[參考管理員] 對話方塊隨即出現。

在 [參考管理員] 中，選取 [模型] 的核取方塊，然後按一下 [確定]。

### <a name="add-a-data-context"></a>新增資料內容

接下來請建立資料內容，用以協調從資料模型提供資料的作業。

若要新增資料內容類別，請在 [方案總管] 中以滑鼠右鍵按一下 **ToDoService**，然後選取 [新增] > [類別]。
在出現的 [新增項目] 對話方塊中，確定已選取 [類別]，並將 [名稱] 設定為 `DataContext`，然後按一下 [新增]。

在 **DataContext.cs** 中，將空白 `class DataContext` 的內容取代為：

```csharp
public static class DataContext
{
    public static Model.ToDoList ToDoList { get; } = new Model.ToDoList("Azure learning List");

    static DataContext()
    {
        // Seed to-do list
        ToDoList.Add(Model.ToDoItem.Load("Learn about microservices", 0, true));
        ToDoList.Add(Model.ToDoItem.Load("Learn about Service Fabric", 1, true));
        ToDoList.Add(Model.ToDoItem.Load("Learn about Service Fabric Mesh", 2, false));
    }
}
```

這個最小的資料內容會填入某些範例待辦事項項目，並提供其存取權。

### <a name="add-a-controller"></a>新增控制器

在建立 **ToDoService** 專案時，範本提供了會處理 HTTP 要求和建立 HTTP 回應的預設控制器。 在 [方案總管] 中的 **ToDoService** 下，開啟 [控制器] 資料夾，以檢視 **ValuesController.cs** 檔案。 

以滑鼠右鍵按一下 **ValuesController.cs**，然後按一下 [重新命名]。 將檔案重新命名為 `ToDoController.cs`。 如果出現將所有參考重新命名的提示，請按一下 [是]。

開啟 **ToDoController.cs** 檔案，並將 `class ToDoController` 的內容取代為：

```csharp
[Route("api/[controller]")]
public class ToDoController : Controller
{
    // GET api/todo
    [HttpGet]
    public IEnumerable<Model.ToDoItem> Get()
    {
        return DataContext.ToDoList.Items;
    }

    // GET api/todo/5
    [HttpGet("{index}")]
    public Model.ToDoItem Get(int index)
    {
        return DataContext.ToDoList.Items.ElementAt(index);
    }

    //// POST api/values
    //[HttpPost]
    //public void Post([FromBody]string value)
    //{
    //}

    //// PUT api/values/5
    //[HttpPut("{id}")]
    //public void Put(int id, [FromBody]string value)
    //{
    //}

    // DELETE api/values/5
    [HttpDelete("{index}")]
    public void Delete(int index)
    {
    }
}
```

本教學課程不會實作新增、刪除等作業，以聚焦在與其他服務的通訊上。

## <a name="create-the-web-page-that-displays-to-do-items"></a>建立會顯示待辦事項項目的網頁

在實作後端服務後，請以程式碼建立網站，以顯示該服務提供的待辦事項項目。 以下是在 **WebFrontEnd** 專案內執行的步驟。

顯示待辦事項項目的網頁必須能夠存取 **ToDoItem** 類別和清單。
在 [方案總管] 中，以滑鼠右鍵按一下 **WebFrontEnd** 並選取 [新增] > [參考...]，以將參考新增至模型專案[參考管理員] 對話方塊隨即出現。

在 [參考管理員] 中，按一下 [模型] 的核取方塊，然後按一下 [確定]。

在 [方案總管] 中，瀏覽至 [WebFrontEnd] > [頁面] > [Index.cshtml] 以開啟 Index.cshtml 頁面。 編輯 **Index.cshtml**。

將整個檔案的內容取代為會定義簡易資料表以顯示待辦事項項目的下列 HTML：

```HTML
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

依序開啟 **Index.cshtml** 和 **Index.cshtml.cs**，以在 [方案總管] 中開啟 [索引] 頁面的程式碼。
在 **Index.cshtml.cs** 頂端，加入 `using System.Net.Http;`

將 `public class IndexModel` 的內容取代為：

```csharp
public class IndexModel : PageModel
{
    public Model.ToDoItem[] Items = new Model.ToDoItem[] { };

    public void OnGet()
    {
        HttpClient client = new HttpClient();

        using (HttpResponseMessage response = client.GetAsync(backendUrl).GetAwaiter().GetResult())
        {
            if (response.StatusCode == System.Net.HttpStatusCode.OK)
            {
                Items = Newtonsoft.Json.JsonConvert.DeserializeObject<Model.ToDoItem[]>(response.Content.ReadAsStringAsync().Result);
            }
        }
    }

    private static string backendDNSName = $"{Environment.GetEnvironmentVariable("ToDoServiceName")}";
    private static Uri backendUrl = new Uri($"http://{backendDNSName}:{Environment.GetEnvironmentVariable("ApiHostPort")}/api/todo");
}
```

### <a name="create-environment-variables"></a>建立環境變數

必須要有後端服務的 URL，才能與該服務進行通訊。 依據本教學課程的目的，下列程式碼摘錄 (在先前定義為 IndexModel 的一部分) 會讀取環境變數以撰寫 URL：

```csharp
private static string backendDNSName = $"{Environment.GetEnvironmentVariable("ToDoServiceName")}";
private static Uri backendUrl = new Uri($"http://{backendDNSName}:{Environment.GetEnvironmentVariable("ApiHostPort")}/api/todo");
```

此 URL 由服務名稱和連接埠所組成。 其完整資訊可在 **ToDoService** 專案內的 service.yaml 檔案中找到。

在 [方案總管] 中瀏覽至 **ToDoService** 專案，然後開啟 [服務資源] > [service.yaml]。

![圖 1 - ToDoService service.yaml 檔案](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-serviceyaml-port.png)

* 服務名稱 `ToDoService` 位於 `services:` 下方的 `name:` 後面，如上圖中的 (1) 所示。
* 連接埠 `20008` 位於 `endpoints:` 下方的 `port:` 後面，如上圖中的 (2) 所示。 您專案的連接埠號碼可能不同。

接著，將會在 WebFrontEnd 專案中定義表示服務名稱和連接埠號碼的環境變數，使該專案能夠呼叫後端服務。

在 [方案總管] 中，瀏覽至 [WebFrontEnd] > [服務資源] > [service.yaml]，以定義指定後端服務位址的變數。

在 service.yaml 檔案中的 `environmentVariables` 下方新增下列變數。 間距有其重要性，因此請對齊您所新增的變數與 `environmentVariables:` 下方的其他變數

> [!IMPORTANT]
> 您必須使用空格 (而非定位字元) 來縮排 service.yaml 檔案中的變數，否則檔案將無法編譯。 Visual Studio 可能會在您建立環境變數時插入定位字元。 請將所有定位字元取代為空格。 雖然您會在**建置**偵錯輸出中看到錯誤，但應用程式仍會啟動。 但在您將定位字元轉換為空格之前，應用程式將無法運作。 若要確保 service.yaml 檔案中沒有定位字元，您可以透過 [編輯]  > [進階]  > [檢視空白字元]，在 Visual Studio 編輯器中顯示空白字元。
> 請注意，service.yaml 檔案會使用英文的地區設定來處理。  比方說，如果您需要使用小數點，便要使用句號，而不是逗號。

**WebFrontEnd** 專案的 **service.yaml** 檔案應會顯示如下內容，但您的 `ApiHostPort` 值可能會不同：

![WebFrontEnd 專案中的 Service.yaml](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-serviceyaml-envvars.png)

現在您已準備好要建置 Service Fabric Mesh 應用程式的映像，並連同後端 Web 服務將其部署至本機叢集。

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解如何：

> [!div class="checklist"]
> * 建立包含 ASP.NET Web 前端的 Service Fabric Mesh 應用程式。
> * 建立用來代表待辦事項項目的模型。
> * 建立後端服務並從中擷取資料。
> * 為後端服務新增控制器和 DataContext，作為該模型檢視控制器模式的一部分。
> * 建立網頁以顯示待辦事項項目。
> * 建立可識別後端服務的環境變數

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [對在本機開發叢集中執行的 Service Fabric Mesh 應用程式進行偵錯](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)