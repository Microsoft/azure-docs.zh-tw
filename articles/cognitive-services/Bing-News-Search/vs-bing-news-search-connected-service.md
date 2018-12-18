---
title: 教學課程：Bing 新聞搜尋，C#
titleSuffix: Azure Cognitive Services
description: 從 ASP.NET Core Web 應用程式連線至 Bing 新聞搜尋。
services: cognitive-services
author: ghogen
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-news-search
ms.topic: tutorial
ms.date: 03/01/2018
ms.author: ghogen
ms.openlocfilehash: f1f5c590216975ce6b0813da6d9d98279d591454
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48804339"
---
# <a name="tutorial-connect-to-bing-news-search-api-by-using-connected-services-in-visual-studio"></a>教學課程：使用 Visual Studio 中的連線服務連線到 Bing 新聞搜尋 API

使用 Bing 新聞搜尋，您可以讓應用程式與服務利用範圍限定在 Web 的無廣告搜尋引擎。 Bing 新聞搜尋是認知服務可供使用的搜尋服務之一。

本文提供使用 Visual Studio 連線服務功能進行 Bing 新聞搜尋的詳細資料。 此功能適用於已安裝的認知服務擴充功能的 Visual Studio 2017 15.7 或更新版本。

## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶。 如果您沒有訂用帳戶，您可以註冊 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
- Visual Studio 2017 15.7 版，並且已安裝網頁程式開發工作負載。 [立即下載](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs)。

[!INCLUDE [vs-install-cognitive-services-vsix](../../../includes/vs-install-cognitive-services-vsix.md)]

## <a name="add-support-to-your-project-for-bing-news-search-api"></a>對您的專案新增 Bing 新聞搜尋 API 的支援

1. 建立名為 MyWebApplication 的新 ASP.NET Core Web 專案。 使用 [Web 應用程式 (模型-檢視-控制器)] 專案範本，使用所有預設設定。 請務必將專案命名為 MyWebApplication，這樣當您將程式碼複製到專案時，命名空間才會相符。 

1. 在 [方案總管] 中，選擇 [新增] > [連線服務]。
   [連線服務] 頁面隨即出現，並顯示您可新增至專案的服務。

   ![[新增連線服務] 功能表項目的螢幕擷取畫面](../media/vs-common/Connected-Service-Menu.PNG)

1. 在可用服務的功能表中，選擇 [為應用程式帶入智慧搜尋]。

   ![連線服務清單的螢幕擷取畫面](./media/vs-bing-news-search-connected-service/Cog-Search-Connected-Service-0.PNG)

   如果您已登入 Visual Studio，並具有與您帳戶相關聯的 Azure 訂用帳戶，則會出現一個頁面，其中顯示含有您訂用帳戶的下拉式清單。 選取您想要使用的訂用帳戶，然後選擇 Bing 新聞搜尋 API 的名稱。 您也可以選擇 [編輯]，以修改自動產生的名稱。

   ![訂用帳戶和名稱欄位的螢幕擷取畫面](media/vs-bing-news-search-connected-service/Cog-Search-Connected-Service-1.PNG)

1. 選擇資源群組與定價層。

   ![資源群組與定價層欄位的螢幕擷取畫面](media/vs-bing-news-search-connected-service/Cog-Search-Connected-Service-2.PNG)

   如果您想要更詳細的定價層資料，請選取 [檢閱定價]。

1. 選擇 [新增] 以新增連線服務的支援。
   Visual Studio 會修改專案，以新增 NuGet 套件、設定檔項目及其他變更，以支援 Bing 新聞搜尋 API 連線。 輸出會顯示您的專案發生什麼情形的記錄。 您應該會看到如下的內容：

   ```output
   [5/4/2018 12:41:21.084 PM] Adding Intelligent Search to the project.
   [5/4/2018 12:41:21.271 PM] Creating new Intelligent Search...
   [5/4/2018 12:41:24.128 PM] Installing NuGet package 'Microsoft.Azure.CognitiveServices.Search.ImageSearch' version 1.2.0...
   [5/4/2018 12:41:24.135 PM] Installing NuGet package 'Microsoft.Azure.CognitiveServices.Search.NewsSearch' version 1.2.0...
   [5/4/2018 12:41:24.154 PM] Installing NuGet package 'Microsoft.Azure.CognitiveServices.Search.WebSearch' version 1.2.0...
   [5/4/2018 12:41:24.168 PM] Installing NuGet package 'Microsoft.Azure.CognitiveServices.Search.CustomSearch' version 1.2.0...
   [5/4/2018 12:41:24.187 PM] Installing NuGet package 'Microsoft.Azure.CognitiveServices.Search.VideoSearch' version 1.2.0...
   [5/4/2018 12:42:07.287 PM] Retrieving keys...
   [5/4/2018 12:42:07.741 PM] Updating appsettings.json setting: 'ServiceKey' = 'c271412f3e4c4e1dacc7c4145fa0572a'
   [5/4/2018 12:42:07.745 PM] Updating appsettings.json setting: 'ServiceEndPoint' = 'https://api.cognitive.microsoft.com/bing/v7.0'
   [5/4/2018 12:42:07.749 PM] Updating appsettings.json setting: 'Name' = 'WebApplicationCore-Search_IntelligentSearch'
   [5/4/2018 12:42:10.217 PM] Successfully added Intelligent Search to the project.
   ```

   appsettings.json 檔案現在包含下列新設定：

   ```json
   "CognitiveServices": {
     "IntelligentSearch": {
       "ServiceKey": "<your service key>",
       "ServiceEndPoint": "https://api.cognitive.microsoft.com/bing/v7.0",
       "Name": "WebApplicationCore-Search_IntelligentSearch"
     }
   }
   ```
 
## <a name="use-the-bing-news-search-api-to-add-search-functionality-to-a-web-page"></a>使用 Bing 新聞搜尋 API 將搜尋功能新增至網頁

現在，您已將 Bing 新聞搜尋 API 的支援新增到專案，以下是如何使用 API 來為網頁新增智慧型搜尋功能。

1.  在 *Startup.cs* 的 `ConfigureServices` 方法中，加入對 `IServiceCollection.AddSingleton` 的呼叫。 這可讓包含金鑰設定的設定物件，可供專案中的程式碼使用。
 
   ```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
            services.AddSingleton<IConfiguration>(Configuration);
        }
   ```


1. 在 **Models** 資料夾下，新增名為 *BingNewsModel.cs* 的新類別檔案。 如果您以不同的名稱命名專案，請使用您自己的專案命名空間，而不是 MyWebApplication。 以下列程式碼取代內容：
 
    ```csharp
    using Microsoft.Azure.CognitiveServices.Search.NewsSearch.Models;
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading.Tasks;
    
    namespace MyWebApplication.Models
    {
        public class BingNewsModel
        {
            public News SearchResult { get; set; } 
            public string SearchText { get; set; }
        }
    }
    ```

   此模型是用來儲存對 Bing 新聞搜尋服務呼叫的結果。
 
1. 在 **Controllers** 資料夾中，新增名為 *IntelligentSearchController.cs* 的新類別檔案。 以下列程式碼取代內容：

   ```csharp
    using System.Net.Http;
    using System.Threading.Tasks;
    using MyWebApplication.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Azure.CognitiveServices.Search.NewsSearch;
    using Microsoft.Extensions.Configuration;
    
    namespace MyWebApplication.Controllers
    {
        // A controller to handle News Search requests
        public class IntelligentSearchController : Controller
        {
            private IConfiguration configuration;
  
            // Set up the configuration that contains the keys
            // (from the appsettings.json file)
            // that you will use to access the service  
            public IntelligentSearchController(IConfiguration configuration)
            {
                this.configuration = configuration;
            }

            // Call the Bing News Search API and put the result in the model object.    
            public async Task<IActionResult> BingSearchResult(BingNewsModel model)
            {
                if (!string.IsNullOrWhiteSpace(model.SearchText))
                { 
                    INewsSearchAPI client = this.GetNewsSearchClient(new MyHandler());
                    model.SearchResult = await client.News.SearchAsync(model.SearchText);
                }
                return View(model);
            }
    
            // Forward requests to the Search endpoint to the BingSearchResult method
            [HttpPost("Search")]
            public IActionResult Search(BingNewsModel model)
            {
                return RedirectToAction("BingSearchResult", model);
            }

            // Get the search client object
            private INewsSearchAPI GetNewsSearchClient(DelegatingHandler handler)
            {
                string key =
                   configuration.GetSection("CognitiveServices")["IntelligentSearch:ServiceKey"];
    
                INewsSearchAPI client = new NewsSearchAPI(
                   new ApiKeyServiceClientCredentials(key), handlers: handler);
    
                return client;
            }
        }
    }
   ```

   在此程式碼中，建構函式會設定包含金鑰的設定物件。 `Search` 路由的方法只會重新導向至 `BingSearchResult` 函式。 這會呼叫 `GetNewsSearchClient` 方法來取得 `NewsSearchAPI` 用戶端物件。  `NewsSearchAPI` 用戶端物件包含 `SearchAsync` 方法，此方法會實際呼叫服務，並以您剛才建立的 `SearchResult` 模型傳回結果。 

1. 新增上述程式碼中參考的類別 `MyHandler`。 這會將搜尋服務的非同步呼叫委派給其基底類別 `DelegatingHandler`。

   ```csharp
    using System.Net.Http;
    using System.Threading.Tasks;
    using System.Threading;

    class MyHandler : DelegatingHandler
    {
        protected async override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
        {
            // Call the inner handler.
            var response = await base.SendAsync(request, cancellationToken);
            
            return response;
        }
    }
   ```

1. 若要新增提交搜尋和檢視結果的支援，請在 **Views** 資料夾中，建立名為 **IntelligentSearch** 的新資料夾。 在這個新資料夾中，新增檢視 *BingSearchResult.cshtml*。 複製下列程式碼：

    ```cshtml
    @using System
    @model MyWebApplication.Models.BingNewsModel
    
    @{
        ViewData["Title"] = "BingSearchResult";
    }
    
    <h2>Search News</h2>
    
    <div class="row">
        <section>
            <form asp-controller="IntelligentSearch" asp-action="Search" method="POST"
                  class="form-horizontal" enctype="multipart/form-data">
                <table width ="90%">
                    <tr>
                        <td>
                            <input type="text" name="SearchText" class="form-control" />
                        </td>
                        <td>
                            <button type="submit" class="btn btn-default">Search</button>
                        </td>
                    </tr>
                </table>
            </form>
        </section>
    </div>
    <h2>Search Result</h2=
    <table>
    @if (!string.IsNullOrEmpty(Model.SearchText)) {
        foreach (var item in Model.SearchResult.Value) {
        <tr>
            <td rowspan="2" width="90">
               <img src=@item?.Image?.Thumbnail?.ContentUrl width="80" height="80" />
            </td>
            <td><a href=@item.Url>@item.Name</a></td>
        </tr>   
        <tr>
            <td>@item.Description</td>
        </tr>
        <tr height="10">
            <td/><td/>
         </tr>
        } }
     </table>
    <div>
        <hr />
        <p>
            <a asp-controller="Home" asp-action="Index">Return to Index</a>
        </p>
    </div>
    ```

1. 在本機啟動 Web 應用程式，輸入剛剛所建立之頁面的 URL (/IntelligentSearch/BingSearchResult)，然後使用 [搜尋] 按鈕，張貼搜尋要求。

   ![Bing 新聞搜尋結果的螢幕擷取畫面](media/vs-bing-news-search-connected-service/Cog-News-Search-Results.PNG)
           
## <a name="clean-up-resources"></a>清除資源

當不再需要資源群組時，您可以將它刪除。 這會刪除認知服務與相關資源。 若要透過入口網站刪除資源群組：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入資源群組的名稱。 選取您想要刪除的資源群組。
2. 選取 [刪除資源群組]。
3. 在 [輸入資源群組名稱] 方塊中，輸入資源群組的名稱，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

若要深入了解 Bing 新聞搜尋 API，請參閱[什麼是 Bing 新聞搜尋？](index.yml)。
