---
title: Azure Cosmos DB 的 ASP.NET MVC 教學課程：Web 應用程式開發 | Microsoft Docs
description: 使用 Azure Cosmos DB 建立 MVC Web 應用程式的 ASP.NET MVC 教學課程。 您將在託管於 Azure 網站的待辦事項應用程式儲存 JSON 和存取資料 - ASP NET MVC 教學課程逐步解說。
keywords: asp.net mvc 教學課程, web 應用程式開發, mvc web 應用程式, asp net mvc 教學課程逐步解說
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.workload: azure-vs
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 08/03/2017
ms.author: sngun
ms.custom: devcenter, vs-azure
ms.openlocfilehash: 110f787866296ec08b9937d56461d37574270a43
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46980918"
---
# <a name="_Toc395809351"></a>ASP.NET MVC 教學課程：使用 Azure Cosmos DB 進行 Web 應用程式開發

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](sql-api-python-application.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

為了特別說明您可以如何有效率地利用 Azure Cosmos DB 來儲存和查詢 JSON 文件，本文提供如何使用 Azure Cosmos DB 建置待辦事項應用程式的完整逐步解說。 在 Azure Cosmos DB 中，這些工作將會儲存為 JSON 文件。

![本教學課程所建立的待辦事項清單 Web 應用程式的螢幕擷取畫面 - ASP NET MVC 教學課程逐步解說](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-image01.png)

本逐步解說會說明如何使用 Azure Cosmos DB 服務，來儲存和存取 Azure 上所託管的 ASP.NET MVC Web 應用程式資料。 如果您要尋找僅著重於 Azure Cosmos DB (而不是 ASP.NET MVC 元件) 的教學課程，請參閱 [建置 Azure Cosmos DB C# 主控台應用程式](sql-api-get-started.md)。

> [!TIP]
> 本教學課程假設您先前有過使用 ASP.NET MVC 和 Azure 網站的經驗。 如果您不熟悉 ASP.NET 或[必備工具](#_Toc395637760)，我們建議您從 [GitHub][GitHub] 下載完整的範例專案，並依照此範例的指示進行。 建置完成後，您可以檢閱此文章，以加深對專案內容中程式碼的了解。
> 
> 

## <a name="_Toc395637760"></a>此資料庫教學課程的必要條件
在依照本文中的指示進行之前，您應先確定備妥下列項目：

* 使用中的 Azure 帳戶。  如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [!INCLUDE [cosmos-db-emulator-vs](../../includes/cosmos-db-emulator-vs.md)]  
* Microsoft Azure SDK for .NET for Visual Studio 2017 (可透過 Visual Studio 安裝程式來取得)。

本文中的所有螢幕擷取畫面都是使用 Microsoft Visual Studio Community 2017 所擷取的。 如果您的系統是設定使用不同的版本，則您的畫面和選項可能不會完全相符，但只要您符合上述必要條件，本方案應該還是有效。

## <a name="_Toc395637761"></a>步驟 1：建立 Azure Cosmos DB 資料庫帳戶
我們將從建立 Azure Cosmos DB 帳戶開始著手。 如果您已經擁有 Azure Cosmos DB 的 SQL 帳戶，或如果您正在使用 Azure Cosmos DB 模擬器來進行本教學課程，可以跳到[建立新的 ASP.NET MVC 應用程式](#_Toc395637762)。

[!INCLUDE [create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [keys](../../includes/cosmos-db-keys.md)]

<br/>
我們現在將從頭開始逐步解說如何建立新的 ASP.NET MVC 應用程式。 

## <a name="_Toc395637762"></a>步驟 2：建立新的 ASP.NET MVC 應用程式

1. 在 Visual Studio 的 [檔案] 功能表中，指向 [新增]，然後按一下 [專案]。 [新增專案]  對話方塊隨即出現。

2. 在 [專案類型] 窗格中，依序展開 [範本]、[Visual C#]、[Web]，然後選取 [ASP.NET Web 應用程式]。

      ![[新增專案] 對話方塊的螢幕擷取畫面，內含反白顯示的 ASP.NET Web 應用程式專案類型](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-new-project-dialog.png)

3. 在 [名稱]  方塊中，輸入專案的名稱。 本教學課程使用 "todo" 名稱。 如果您選擇使用其他名稱，則每當本教學課程提及 todo 命名空間時，您必須調整提供的程式碼範例，以便使用您為應用程式所命名的名稱。 
4. 按一下 [瀏覽] 以巡覽至您想要建立專案的資料夾，然後按一下 [確定]。
   
      [新增 ASP.NET Web 應用程式] 對話方塊隨即出現。
   
    ![[新增 ASP.NET Web 應用程式] 對話方塊的螢幕擷取畫面，內含反白顯示的 MVC 應用程式範本](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-MVC.png)
5. 在 [範本] 窗格中，選取 [MVC] 。

6. 按一下 [確定]  ，並讓 Visual Studio 執行有關 Scaffolding 空的 ASP.NET MVC 範本的作業。 

          
7. Visual Studio 建立好未定案 MVC 應用程式之後，您便擁有可以在本機執行的空白 ASP.NET 應用程式。
   
    我們會跳過在本機執行專案，因為我確定我們都已看過 ASP.NET "Hello World" 應用程式。 讓我們直接跳到將 Azure Cosmos DB 新增至此專案，並建置應用程式的步驟。

## <a name="_Toc395637767"></a>步驟 3：將 Azure Cosmos DB 新增至 MVC Web 應用程式專案
既然我們已經完成了此解決方案的大部分 ASP.NET MVC 瑣事，現在可以開始本教學課程的真正目的，也就是將 Azure Cosmos DB 新增至 MVC Web 應用程式。

1. Azure Cosmos DB .NET SDK 會進行封裝，並以 NuGet 封裝形式加以散發。 若要在 Visual Studio 中取得 NuGet 封裝，請使用 Visual Studio 中的 NuGet 封裝管理員，方法是以滑鼠右鍵按一下 [方案總管] 中的專案，然後按一下 [管理 NuGet 封裝]。
   
    ![[方案總管] 中 Web 應用程式專案的滑鼠右鍵選項螢幕擷取畫面，內含反白顯示的 [管理 NuGet 套件]。](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-manage-nuget.png)
   
    [ **管理 NuGet 封裝** ] 對話方塊隨即出現。
2. 在 NuGet [瀏覽] 方塊中，輸入 ***Azure DocumentDB***。 (套件名稱尚未更新為 Azure Cosmos DB)。
   
    從結果中，安裝 [Microsoft.Azure.DocumentDB by Microsoft] 套件。 這會下載和安裝 Azure Cosmos DB 套件，以及所有依存項目 (例如 Newtonsoft.Json)。 按一下 [預覽] 視窗中的 [確定]，以及 [接受授權] 視窗中的 [我接受] 來完成安裝。
   
    ![[管理 NuGet 封裝] 視窗的螢幕擷取畫面，內含反白顯示的 Microsoft Azure Cosmos DB 用戶端程式庫](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-install-nuget.png)
   
      或者，您也可以使用 Package Manager Console 來安裝封裝。 若要這樣做，請在 [工具] 功能表上按一下 [NuGet 封裝管理員]，然後按一下 [Package Manager Console]。 在出現提示時輸入下列內容：
   
        Install-Package Microsoft.Azure.DocumentDB
        
3. 安裝封裝之後，您的 Visual Studio 方案應該類似下列已新增兩個新參考 (Microsoft.Azure.Documents.Client 和 Newtonsoft.Json) 的方案。
   
    ![[方案總管] 中 JSON 資料專案新增兩個參考的螢幕擷取畫面](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-added-references.png)

## <a name="_Toc395637763"></a>步驟 4：設定 ASP.NET MVC 應用程式
現在我們可以開始將模型、檢視和控制站新增至此 MVC 應用程式：

* [新增模型](#_Toc395637764)。
* [新增控制器](#_Toc395637765)。
* [新增檢視](#_Toc395637766)。

### <a name="_Toc395637764"></a>新增 JSON 資料模型
首先，讓我們在 MVC 中建立 **M** (模型)。 

1. 在 [方案總管] 中，以滑鼠右鍵按一下 **Models** 資料夾，再按一下 [新增]，然後按一下 [類別]。
   
      [加入新項目]  對話方塊隨即出現。
2. 將您的新類別命名為 **Item.cs**，然後按一下 [新增]。 
3. 在這個新的 **Item.cs** 檔案中，將下列程式碼加到最後一個 *using 陳述式*後面。
   
        using Newtonsoft.Json;
4. 現在，將此程式碼取代 
   
        public class Item
        {
        }
   
    為下列程式碼。
   
        public class Item
        {
            [JsonProperty(PropertyName = "id")]
            public string Id { get; set; }
   
            [JsonProperty(PropertyName = "name")]
            public string Name { get; set; }
   
            [JsonProperty(PropertyName = "description")]
            public string Description { get; set; }
   
            [JsonProperty(PropertyName = "isComplete")]
            public bool Completed { get; set; }
        }
   
    Azure Cosmos DB 中的所有資料都會透過線路傳遞，並儲存為 JSON。 如需透過 JSON.NET 控管物件序列化/取消序列化，您可以使用在剛才建立的 [項目] 類別中所示範 **JsonProperty** 屬性。 您 **無需** 這樣做，但我想確定所有屬性都會依照 JSON camelCase 的命名慣例命名。 
   
    使用 JSON 時，您不但可以控管屬性名稱格式，也可以跟我命名 **Description** 屬性一樣重新命名您的 .NET 屬性。 

### <a name="_Toc395637765"></a>新增控制器
我們已經建立了 **M**，現在讓我們建立 MVC 中的 **C** (控制器類別)。

1. 在 [方案總管] 中，以滑鼠右鍵按一下 **Controllers** 資料夾，按一下 [新增]，然後按一下 [控制器]。
   
    [ **新增 Scaffold** ] 對話方塊隨即出現。
2. 選取 [Web API 5 控制器 - 空]，然後按一下 [新增]。
   
    ![[新增 Scaffold] 對話方塊的螢幕擷取畫面，內含反白顯示的 [MVC 5 控制器 - 空的] 選項](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-controller-add-scaffold.png)
3. 將新的控制器命名為 **ItemController**
   
    ![[新增控制器] 對話方塊的螢幕擷取畫面](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-add-controller.png)
   
    檔案建立之後，您的 Visual Studio 方案應該類似包含下面 [方案總管] 中新 ItemController.cs 檔案的方案。 稍早建立的新 Item.cs 檔案也會顯示出來。
   
    ![Visual Studio 的螢幕擷取畫面，[方案總管] 內含反白顯示的新 ItemController.cs 檔案和 Item.cs 檔案](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-new-item-solution-explorer.png)
   
    您可以先將 ItemController.cs 關閉，我們稍後會回頭使用此檔案。 

### <a name="_Toc395637766"></a>新增檢視
現在，我們可以開始建立 MVC 中的 **V** (檢視)：

* [新增項目索引檢視](#AddItemIndexView)。
* [新增項目檢視](#AddNewIndexView)。
* [新增編輯項目檢視](#_Toc395888515)。

#### <a name="AddItemIndexView"></a>新增項目索引檢視
1. 在 [方案總管] 中，展開 **Views** 資料夾，以滑鼠右鍵按一下稍早您在建立 **ItemController** 時，Visual Studio 為您建立的空白 **Item** 資料夾，按一下 [新增]，然後按一下 [檢視]。
   
    ![顯示 Visual Studio 方案建立之 Item 資料夾的 [方案總管] 螢幕擷取畫面，內含反白顯示的 [新增檢視] 命令](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-add-view.png)
2. 在 [新增檢視]  對話方塊中，執行下列動作：
   
   * 在 [檢視名稱] 方塊中，輸入***索引***。
   * 在 [範本] 方塊中，選取 [清單]。
   * 在 [模型類別] 方塊中，選取 [項目 (todo.Models)]。
   * 在 [版面配置頁面] 方塊中，輸入 ***~/Views/Shared/_Layout.cshtml***。
     
   ![顯示 [新增檢視] 對話方塊的螢幕擷取畫面](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-add-view-dialog.png)
3. 設定所有這些值之後，按一下 [新增]  ，並讓 Visual Studio 建立新的範本檢視。 完成之後，系統便會開啟剛建立的 cshtml 檔案。 我們稍後會回頭使用此檔案，您可以先在 Visual Studio 中將該檔案關閉。

#### <a name="AddNewIndexView"></a>新增項目檢視
與建立 [項目索引] 檢視的方式類似，現在我們現在可以開始建立新的檢視，以供建立新的**項目**使用。

1. 在 [方案總管] 中，再次以滑鼠右鍵按一下 **Item** 資料夾，按一下 [新增]，然後按一下 [檢視]。
2. 在 [新增檢視]  對話方塊中，執行下列動作：
   
   * 在 [檢視名稱] 方塊中，輸入***建立***。
   * 在 [範本] 方塊中，選取 [建立]。
   * 在 [模型類別] 方塊中，選取 [項目 (todo.Models)]。
   * 在 [版面配置頁面] 方塊中，輸入 ***~/Views/Shared/_Layout.cshtml***。
   * 按一下 [新增] 。
   
#### <a name="_Toc395888515"></a>新增編輯項目檢視
最後，使用與之前相同的方式新增最後一個檢視，以供編輯 **項目** 使用；

1. 在 [方案總管] 中，再次以滑鼠右鍵按一下 **Item** 資料夾，按一下 [新增]，然後按一下 [檢視]。
2. 在 [新增檢視]  對話方塊中，執行下列動作：
   
   * 在 [檢視名稱] 方塊中，輸入***編輯***。
   * 在 [範本] 方塊中，選取 [編輯]。
   * 在 [模型類別] 方塊中，選取 [項目 (todo.Models)]。
   * 在 [版面配置頁面] 方塊中，輸入 ***~/Views/Shared/_Layout.cshtml***。
   * 按一下 [新增] 。

完成這項作業之後，請將 Visual Studio 中的所有 cshtml 文件關閉，我們稍後會回頭使用這些檢視。

## <a name="_Toc395637769"></a>步驟 5︰裝設 Azure Cosmos DB
我們已經建立了標準的 MVC 項目，現在可以開始新增 Azure Cosmos DB 的程式碼。 

在本節中，我們將新增程式碼來處理下列作業：

* [列出未完成項目](#_Toc395637770)。
* [新增項目](#_Toc395637771)。
* [編輯項目](#_Toc395637772)。

### <a name="_Toc395637770"></a>列出 MVC Web 應用程式中的未完成項目
首先要執行的作業是新增類別，其中包含連線至及使用 Azure Cosmos DB 的所有邏輯。 在本教學課程中，我們會將所有邏輯封裝到名為 DocumentDBRepository 的儲存機制類別中。 

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案，按一下 [新增]，然後按一下 [類別]。 將新類別命名為 **DocumentDBRepository**，然後按一下 [新增]。
2. 在剛剛建立的 **DocumentDBRepository** 類別中，在「命名空間」宣告上方新增下列「using 陳述式」
   
        using Microsoft.Azure.Documents; 
        using Microsoft.Azure.Documents.Client; 
        using Microsoft.Azure.Documents.Linq; 
        using System.Configuration;
        using System.Linq.Expressions;
        using System.Threading.Tasks;
        using System.Net;
        
    現在，將此程式碼取代 
   
        public class DocumentDBRepository
        {
        }
   
    為下列程式碼。
   
        public static class DocumentDBRepository<T> where T : class
        {
            private static readonly string DatabaseId = ConfigurationManager.AppSettings["database"];
            private static readonly string CollectionId = ConfigurationManager.AppSettings["collection"];
            private static DocumentClient client;
   
            public static void Initialize()
            {
                client = new DocumentClient(new Uri(ConfigurationManager.AppSettings["endpoint"]), ConfigurationManager.AppSettings["authKey"]);
                CreateDatabaseIfNotExistsAsync().Wait();
                CreateCollectionIfNotExistsAsync().Wait();
            }
   
            private static async Task CreateDatabaseIfNotExistsAsync()
            {
                try
                {
                    await client.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(DatabaseId));
                }
                catch (DocumentClientException e)
                {
                    if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
                    {
                        await client.CreateDatabaseAsync(new Database { Id = DatabaseId });
                    }
                    else
                    {
                        throw;
                    }
                }
            }
   
            private static async Task CreateCollectionIfNotExistsAsync()
            {
                try
                {
                    await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(DatabaseId, CollectionId));
                }
                catch (DocumentClientException e)
                {
                    if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
                    {
                        await client.CreateDocumentCollectionAsync(
                            UriFactory.CreateDatabaseUri(DatabaseId),
                            new DocumentCollection { Id = CollectionId },
                            new RequestOptions { OfferThroughput = 1000 });
                    }
                    else
                    {
                        throw;
                    }
                }
            }
        }
   
    
3. 我們打算從組態中讀取部分值，因此請開啟應用程式的 **Web.config** 檔案，並在 `<AppSettings>` 區段下新增下列幾行。
   
        <add key="endpoint" value="enter the URI from the Keys blade of the Azure Portal"/>
        <add key="authKey" value="enter the PRIMARY KEY, or the SECONDARY KEY, from the Keys blade of the Azure  Portal"/>
        <add key="database" value="ToDoList"/>
        <add key="collection" value="Items"/>
4. 現在，使用 Azure 入口網站的 [金鑰] 刀鋒視窗來更新 [端點] 和 [authKey] 的值。 使用 [金鑰] 刀鋒視窗的 [URI] 做為端點設定的值，並使用 [金鑰] 刀鋒視窗的 [主索引鍵] 或 [次要金鑰] 做為 authKey 設定的值。

    其會負責裝設 Azure Cosmos DB 存放庫，現在讓我們加入我們的應用程式邏輯。

1. 我們想要對 todo 清單應用程式進行的第一件事就是顯示未完成的項目。  在 **DocumentDBRepository** 類別內的任意位置複製並貼上下列程式碼片段。
   
        public static async Task<IEnumerable<T>> GetItemsAsync(Expression<Func<T, bool>> predicate)
        {
            IDocumentQuery<T> query = client.CreateDocumentQuery<T>(
                UriFactory.CreateDocumentCollectionUri(DatabaseId, CollectionId))
                .Where(predicate)
                .AsDocumentQuery();
   
            List<T> results = new List<T>();
            while (query.HasMoreResults)
            {
                results.AddRange(await query.ExecuteNextAsync<T>());
            }
   
            return results;
        }
2. 開啟我們剛剛新增的 **ItemController** ，並在命名空間宣告上方新增下列 *using 陳述式*
   
        using System.Net;
        using System.Threading.Tasks;
        using todo.Models;
   
    如果您的專案名稱不是 "todo"，則您必須使用 "todo.Models" 進行更新，才能反映您的專案名稱。
   
    現在，將此程式碼取代
   
        //GET: Item
        public ActionResult Index()
        {
            return View();
        }
   
    為下列程式碼。
   
        [ActionName("Index")]
        public async Task<ActionResult> IndexAsync()
        {
            var items = await DocumentDBRepository<Item>.GetItemsAsync(d => !d.Completed);
            return View(items);
        }
3. 開啟 **Global.asax.cs** 並將下列一行新增至 **Application_Start** 方法 
   
        DocumentDBRepository<todo.Models.Item>.Initialize();

此時，應該已經可以建置方案，而不會發生任何錯誤。

如果您現在執行應用程式，您可以前往 **HomeController** 及該控制器的 [索引] 檢視。 這是我們在一開始時所選擇的 MVC 範本專案預設行為，但是我們不想要這樣的行為！ 讓我們變更此 MVC 應用程式上的路由以改變此行為。

開啟 ***App\_Start\RouteConfig.cs***，並尋找以 "defaults:" 開頭的程式碼行，然後將它變更為如下所示。

        defaults: new { controller = "Item", action = "Index", id = UrlParameter.Optional }

如果您未在 URL 中指定控制路由行為的值，這會讓 ASP.NET MVC 知道改用 **Item** (**Home**) 作為控制器，並使用使用者**索引**作為檢視。

如果您執行應用程式，它現在會呼叫至您的 **ItemController**，進而呼叫至儲存機制類別，並使用 GetItems 方法將所有未完成的項目傳回 **Views**\\**Item**\\**Index** 檢視。 

如果建置並立即執行此專案，您現在應該會看到如下的內容。    

![本資料庫教學課程所建立的待辦事項清單 Web 應用程式的螢幕擷取畫面](./media/sql-api-dotnet-application/build-and-run-the-project-now.png)

### <a name="_Toc395637771"></a>新增項目
我們可以開始將一些項目放入資料庫中，所以除了空白方格以外，我們還可以看到其他項目。

讓我們將一些程式碼新增至 Azure Cosmos DBRepository 和 ItemController，以在 Azure Cosmos DB 中保留記錄。

1. 將下列方法新增至 **DocumentDBRepository** 類別。
   
       public static async Task<Document> CreateItemAsync(T item)
       {
           return await client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri(DatabaseId, CollectionId), item);
       }
   
   此方法只接受傳遞給它的物件，並將它保留在 Azure Cosmos DB 中。
2. 開啟 ItemController.cs 檔案，並在類別中加入下列程式碼片段。 這是 ASP.NET MVC 得知如何執行 **建立** 動作的方式。 在此情況下，只需轉譯先前建立的關聯 Create.cshtml 檢視。
   
        [ActionName("Create")]
        public async Task<ActionResult> CreateAsync()
        {
            return View();
        }
   
    現在此控制器需要更多程式碼，以接受 [建立]  檢視所提交的資料。
3. 將下一個程式碼區塊新增至 ItemController.cs 類別，以告訴 ASP.NET MVC 如何使用表單 POST 來執行此控制器的作業。
   
        [HttpPost]
        [ActionName("Create")]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> CreateAsync([Bind(Include = "Id,Name,Description,Completed")] Item item)
        {
            if (ModelState.IsValid)
            {
                await DocumentDBRepository<Item>.CreateItemAsync(item);
                return RedirectToAction("Index");
            }
   
            return View(item);
        }
   
    這段程式碼會呼叫 DocumentDBRepository，並且使用 CreateItemAsync 方法將新的待辦事項項目保存到資料庫。 
   
    **安全性注意事項**：此處所使用的 **ValidateAntiForgeryToken** 屬性可協助應用程式防止跨網站偽造要求攻擊。 這不光只是新增此屬性，您的檢視也必須使用這個防偽權杖。 如需此主題的詳細資訊以及如何正確實作此作業的範例，請參閱[防止跨網站偽造要求][Preventing Cross-Site Request Forgery]。 [GitHub][GitHub] 上提供的原始程式碼已有完整實作。
   
    **安全性注意事項**：我們也會在方法參數上使用 **Bind** 屬性，以協助防範 over-posting 攻擊。 如需詳細資訊，請參閱 [ASP.NET MVC 中的基本 CRUD 作業][Basic CRUD Operations in ASP.NET MVC]。

這包括將新項目新增至資料庫所需的程式碼。

### <a name="_Toc395637772"></a>編輯項目
我們最後還要做一件事，那就是新增在資料庫中編輯 **項目** ，以及將它們標示為已完成的功能。 編輯的檢視已經新增至專案，因此，我們只需重新將某些程式碼新增至控制器及 **DocumentDBRepository** 類別即可。

1. 將下列程式碼新增至 **DocumentDBRepository** 類別。
   
        public static async Task<Document> UpdateItemAsync(string id, T item)
        {
            return await client.ReplaceDocumentAsync(UriFactory.CreateDocumentUri(DatabaseId, CollectionId, id), item);
        }
   
        public static async Task<T> GetItemAsync(string id)
        {
            try
            {
                Document document = await client.ReadDocumentAsync(UriFactory.CreateDocumentUri(DatabaseId, CollectionId, id));
                return (T)(dynamic)document;
            }
            catch (DocumentClientException e)
            {
                if (e.StatusCode == HttpStatusCode.NotFound)
                {
                    return null;
                }
                else
                {
                    throw;
                }
            }
        }
   
    這兩個方法中的第一個方法 (GetItem) 會從 Azure Cosmos DB 提取項目，此項目會被傳回 **ItemController**，接著傳至 [編輯] 檢視。
   
    第二個剛剛新增的方法是使用從 **ItemController** 傳入的**文件**版本，來取代 Azure Cosmos DB 中的**文件**。
2. 將下列程式碼新增至 **ItemController** 類別。
   
        [HttpPost]
        [ActionName("Edit")]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> EditAsync([Bind(Include = "Id,Name,Description,Completed")] Item item)
        {
            if (ModelState.IsValid)
            {
                await DocumentDBRepository<Item>.UpdateItemAsync(item.Id, item);
                return RedirectToAction("Index");
            }
   
            return View(item);
        }
   
        [ActionName("Edit")]
        public async Task<ActionResult> EditAsync(string id)
        {
            if (id == null)
            {
                return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
            }
   
            Item item = await DocumentDBRepository<Item>.GetItemAsync(id);
            if (item == null)
            {
                return HttpNotFound();
            }
   
            return View(item);
        }
   
    第一個方法會處理當使用者按一下 [索引] 檢視中的 [編輯] 連結時所發生的 Http GET。 此方法會從 Azure Cosmos DB 中提取[**文件**](http://msdn.microsoft.com/library/azure/microsoft.azure.documents.document.aspx)，並將它傳遞給 [編輯] 檢視。
   
    [編輯] 檢視會接著對 **IndexController** 執行 Http POST。 
   
    所新增的第二個方法會處理此作業，將更新的物件傳遞至 Azure Cosmos DB，並保留在資料庫中。

這樣便大功告成了，這些就是我們必須執行應用程式的所有作業：列出未完成**項目**、新增**項目**，最後是編輯**項目**。

## <a name="_Toc395637773"></a>步驟 6：在本機執行應用程式
若要在本機電腦測試應用程式，請執行下列動作：

1. 在 Visual Studio 中按 F5，即可在偵錯模式下建置應用程式。 這樣應該可以建置應用程式，並啟動含有先前所看過之空白方格頁面的瀏覽器：
   
    ![本資料庫教學課程所建立的待辦事項清單 Web 應用程式的螢幕擷取畫面](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-an-item-a.png)
   
     
2. 按一下 [新建] 連結，並在 [名稱] 和 [描述] 欄位中新增值。 將 [已完成] 核取方塊保持為未選取狀態，否則，**新項目**會以已完成的狀態新增，且不會出現在初始清單中。
   
    ![[建立] 檢視的螢幕擷取畫面](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-new-item.png)
3. 按一下 [建立]，您便會被重新導向回到 [索引] 檢視，且您的**項目**便會出現在清單中。
   
    ![[索引] 檢視的螢幕擷取畫面](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-an-item.png)
   
    依需要新增一些 **項目** 至 [待辦事項] 清單。
    
4. 按一下清單上 [項目] 旁邊的 [編輯]，您便會被帶到 [編輯] 檢視，您可以在此更新物件的任何屬性 (包括 [已完成] 旗標)。 如果您標示 [完成] 旗標，然後按一下 [儲存]，則**項目**會從未完成的工作清單中移除。
   
    ![[索引] 檢視的螢幕擷取畫面，內含勾選的 [已完成] 方塊](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-completed-item.png)
5. 完成測試應用程式後，按 Ctrl + F5 停止偵錯應用程式。 您現在可以開始進行部署。

## <a name="_Toc395637774"></a>步驟 7：將應用程式部署至 Azure App Service 
您已經擁有可在 Azure Cosmos DB 正常運作的完整應用程式，我們現在要將此 Web 應用程式部署至 Azure App Service。  

1. 若要發佈此應用程式，您只需要以滑鼠右鍵按一下 [方案總管] 上的專案，然後按一下 [發佈] 即可。
   
    ![[方案總管] 中 [發佈] 選項的螢幕擷取畫面](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-publish.png)

2. 在 [發佈] 對話方塊中，按一下 [Microsoft Azure App Service]，然後選取 [建立新的] 來建立 App Service 設定檔，或按一下 [選取現有的] 來使用現有設定檔。

    ![Visual Studio 中的 [發佈] 對話方塊](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-publish-to-existing.png)

3. 如果您有現有的 Azure App Service 設定檔，請輸入您的訂用帳戶名稱。 使用 [檢視] 篩選器來依資源群組或資源類型排序，然後選取您的 Azure App Service。 
   
    ![Visual Studio 中的 [App Service] 對話方塊](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-app-service.png)

4. 若要建立新的 Azure App Service 設定檔，請按一下 [發佈] 對話方塊中的 [建立新的]。 在 [建立 App Service] 對話方塊中，輸入您的 Web 應用程式名稱和適當的訂用帳戶、資源群組和 App Service 方案，然後按一下 [建立]。

    ![Visual Studio 中的 [建立 App Service] 對話方塊](./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-app-service.png)

幾秒後，Visual Studio 便會發佈 Web 應用程式並啟動瀏覽器，您可以在瀏覽器中看到您方便好用的應用程式已在 Azure 中執行！



## <a name="_Toc395637775"></a>後續步驟
恭喜！ 您剛剛使用 Azure Cosmos DB 建置您的第一個 ASP.NET MVC Web 應用程式，並將它發佈至 Azure。 您可以從 [GitHub][GitHub]下載或複製完整應用程式 (包括不包含在本教學課程中的詳細資料和刪除功能) 的原始程式碼。 所以如果您想要將程式碼加入您的應用程式，請抓取程式碼，並將它加入這個應用程式。

若要將其他功能新增至您的應用程式，請檢閱 [Azure Cosmos DB .NET 程式庫](/dotnet/api/overview/azure/cosmosdb?view=azure-dotnet)中提供的 API，並歡迎您貢獻到 [GitHub][GitHub] 上的 Azure Cosmos DB .NET 程式庫。 

[\*]: https://microsoft.sharepoint.com/teams/DocDB/Shared%20Documents/Documentation/Docs.LatestVersions/PicExportError
[Visual Studio Express]: http://www.visualstudio.com/products/visual-studio-express-vs.aspx
[Microsoft Web Platform Installer]: http://www.microsoft.com/web/downloads/platform.aspx
[Preventing Cross-Site Request Forgery]: http://go.microsoft.com/fwlink/?LinkID=517254
[Basic CRUD Operations in ASP.NET MVC]: http://go.microsoft.com/fwlink/?LinkId=317598
[GitHub]: https://github.com/Azure-Samples/documentdb-net-todo-app
