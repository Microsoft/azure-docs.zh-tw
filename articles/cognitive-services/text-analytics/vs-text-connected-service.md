---
title: 教學課程：搭配 C# 使用文字分析
titleSuffix: Azure Cognitive Services
description: 從 ASP.NET Core Web 應用程式連線至文字分析。
services: cognitive-services
author: ghogen
manager: cgronlun
ms.service: cognitive-services
ms.component: text-analytics
ms.topic: tutorial
ms.date: 09/12/2018
ms.author: ghogen
ms.openlocfilehash: 95de174087fb2a73ad6c75a2e96caf872c351854
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45605227"
---
# <a name="tutorial-connect-to-the-text-analytics-cognitive-service-by-using-connected-services-in-visual-studio"></a>教學課程：使用 Visual Studio 中已連線的服務連線到文字分析認知服務

您可以使用文字分析服務擷取豐富的資訊，進而分類及處理視覺化資料，並對影像進行機器輔助仲裁，以協助籌備您的服務。

此文章及其附屬文章提供使用適用於文字分析服務之 Visual Studio 已連線的服務功能的詳細資料。 此功能適用於已安裝認知服務延伸模組的 Visual Studio 2017 15.7 或更新版本。

## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶。 如果您沒有訂用帳戶，您可以註冊 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
- Visual Studio 2017 15.7 版，並且已安裝網頁程式開發工作負載。 [立即下載](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs)。

[!INCLUDE [vs-install-cognitive-services-vsix](../../../includes/vs-install-cognitive-services-vsix.md)]

## <a name="add-support-to-your-project-for-the-text-analytics-service"></a>將文字分析服務的支援加入至您的專案中

1. 建立一個稱為 TextAnalyticsDemo 的新 ASP.NET Core Web 專案。 使用具有所有預設設定的 [Web 應用程式 (模型-檢視-控制器)] 專案範本。 請務必將專案命名為 MyWebApplication，這樣當您將程式碼複製到專案時，命名空間才會相符。  此文章中的範例使用 MVC，但您可以使用文字分析連線服務搭配任何 ASP.NET 專案類型。

1. 在 [方案總管] 中，按兩下 [連線服務] 項目。
   [連線服務] 頁面隨即出現，並顯示您可新增至專案的服務。

   ![[方案總管] 中的連線服務螢幕擷取畫面](../media/vs-common/Connected-Services-Solution-Explorer.PNG)

1. 在可用服務的功能表中，選擇 [使用文字分析評估情感]。

   ![[已連的服務] 畫面的螢幕擷取畫面](./media/vs-text-connected-service/Cog-Text-Connected-Service-0.PNG)

   如果您已登入 Visual Studio，並具有與您帳戶相關聯的 Azure 訂用帳戶，則會出現一個頁面，其中顯示含有您訂用帳戶的下拉式清單。

   ![[文字分析連線服務] 畫面的螢幕擷取畫面](media/vs-text-connected-service/Cog-Text-Connected-Service-1.PNG)

1. 選取您想要使用的訂用帳戶，然後選擇文字分析服務的名稱，或選擇 [編輯] 連結來修改自動產生的名稱，選擇資源群組及定價層。

   ![[資源群組] 與 [定價層] 欄位的螢幕擷取畫面](media/vs-text-connected-service/Cog-Text-Connected-Service-2.PNG)

   瀏覽此連結，以取得定價層的詳細資料。

1. 選擇 [新增] 以新增連線服務的支援。
   Visual Studio 會修改專案，以新增 NuGet 套件、設定檔項目與其他變更，以支援文字分析服務的連線。 [輸出視窗] 會顯示您的專案發生什麼情形的記錄。 您應該會看到如下的內容：

   ```output
    [6/1/2018 3:04:02.347 PM] Adding Text Analytics to the project.
    [6/1/2018 3:04:02.906 PM] Creating new Text Analytics...
    [6/1/2018 3:04:06.314 PM] Installing NuGet package 'Microsoft.Azure.CognitiveServices.Language' version 1.0.0-preview...
    [6/1/2018 3:04:56.759 PM] Retrieving keys...
    [6/1/2018 3:04:57.822 PM] Updating appsettings.json setting: 'ServiceKey' = '<service key>'
    [6/1/2018 3:04:57.827 PM] Updating appsettings.json setting: 'ServiceEndPoint' = 'https://westus.api.cognitive.microsoft.com/text/analytics/v2.0'
    [6/1/2018 3:04:57.832 PM] Updating appsettings.json setting: 'Name' = 'TextAnalyticsDemo'
    [6/1/2018 3:05:01.840 PM] Successfully added Text Analytics to the project.
    ```
 
## <a name="use-the-text-analytics-service-to-detect-the-language-for-a-text-sample"></a>使用文字分析服務偵測文字範例的語言。

1. 在 Startup.cs 中，新增下列 using 陳述式。
 
   ```csharp
   using System.IO;
   using System.Net.Http;
   using System.Net.Http.Headers;
   using System.Text;
   using Microsoft.Extensions.Configuration;
   ```
 
1. 新增 [設定] 欄位，然後在 Startup 類別中，新增初始化 [設定] 欄位的建構函式，以啟用程式中的設定。

   ```csharp
      private IConfiguration configuration;

      public Startup(IConfiguration configuration)
      {
          this.configuration = configuration;
      }
   ```

1. 在 Controllers 資料夾中，新增一個稱為 DemoTextAnalyzeController 的類別檔案，並將其內容取代為下列程式碼：

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Net.Http;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
    using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
    using Microsoft.Extensions.Configuration;
    using TextAnalyticsDemo.Models;
    
    namespace TextAnalyticsDemo.Controllers
    {
        public class DemoTextAnalyzeController : Controller
        {
            private IConfiguration configuration;
    
            public DemoTextAnalyzeController(IConfiguration configuration)
            {
                this.configuration = configuration;
            }
    
            public IActionResult TextAnalyzeResult(TextAnalyzeModel model)
            {
    
                if (!string.IsNullOrWhiteSpace(model.TextStr))
                {
                    ITextAnalyticsAPI client = this.GetTextAnalyzeClient(new MyHandler());
                    model.AnalyzeResult = client.DetectLanguage(
                        new BatchInput(
                            new List<Input>()
                            {
                                new Input("id",model.TextStr)
                            }));
                }
                return View(model);
            }
    
            [HttpPost("Analyze")]
            public IActionResult Analyze(TextAnalyzeModel model)
            {
                return RedirectToAction("TextAnalyzeResult", model);
            }
    
            // Using the ServiceKey from the configuration file,
            // get an instance of the Text Analytics client.
            private ITextAnalyticsAPI GetTextAnalyzeClient(DelegatingHandler handler)
            {
                string key = configuration.GetSection("CognitiveServices")["TextAnalytics:ServiceKey"];
    
                ITextAnalyticsAPI client = new TextAnalyticsAPI( handlers: handler);
                client.SubscriptionKey = key;
                client.AzureRegion = AzureRegions.Westus;
    
                return client;
            }
        }
    }
    ```
    
    此程式碼包含 GetTextAnalyzeClient，可取得您能夠用來呼叫文字分析 API 的用戶端物件，並要求能夠針對指定的文字，呼叫 DetectLanguage 的處理常式。

1. 新增上述程式碼使用的 MyHandler 協助程式類別。

    ```csharp
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

1. 在 [模型] 資料夾中，新增模型的類別。

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading.Tasks;
    using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
    
    namespace Demo.Models
    {
        public class TextAnalyzeModel
        {
            public string TextStr { get; set; }
    
            public LanguageBatchResult AnalyzeResult { get; set; }
    
            public SentimentBatchResult AnalyzeResult2 { get; set; }
        }
    }
    ```

1. 新增檢視以顯示分析的文字、判定的語言，以及在分析中表示信賴等級的分數。 若要這樣做，請以滑鼠右鍵按一下 [檢視] 資料夾，選擇 [新增]，然後選擇 [檢視]。 在出現的對話方塊中，提供名稱 _TextAnalyzeResult_ 並接受預設值，以便在 [檢視] 資料夾中新增一個稱為 _TextAnalyzeResult.cshtml_ 的新檔案，然後將下列內容複製到其中：
    
    ```cshtml
    @using System
    @model Demo.Models.TextAnalyzeModel
    
    @{
        ViewData["Title"] = "TextAnalyzeResult";
    }
    
    <h2>Text Language</h2>
    
    <div class="row">
        <section>
            <form asp-controller="DemoTextAnalyze" asp-action="Analyze" method="POST"
                  class="form-horizontal" enctype="multipart/form-data">
                <table width="90%">
                    <tr>
                        <td>
                            <input type="text" name="TextStr" class="form-control" />
                        </td>
                        <td>
                            <button type="submit" class="btn btn-default">Analyze</button>
                        </td>
                    </tr>
                </table>
            </form>
        </section>
    </div>
    
    <h2>Result</h2>
    <div>
        <dl class="dl-horizontal">
            <dt>
                Text :
            </dt>
            <dd>
                @Html.DisplayFor(model => model.TextStr)
            </dd>
            <dt>
                Language Name :
            </dt>
            <dd>
                @Html.DisplayFor(model => model.AnalyzeResult.Documents[0].DetectedLanguages[0].Name)
            </dd>
            <dt>
                Score :
            </dt>
            <dd>
                @Html.DisplayFor(model => model.AnalyzeResult.Documents[0].DetectedLanguages[0].Score)
            </dd>
        </dl>
    </div>
    <div>
        <hr />
        <p>
            <a asp-controller="Home" asp-action="Index">Return to Index</a>
        </p>
    </div>
    
    ```
 
1. 在本機建置並執行範例。 輸入一些文字，然後查看文字分析偵測到何種語言。
   
## <a name="clean-up-resources"></a>清除資源

不再需要資源群組時，請將它刪除。 這會刪除認知服務與相關資源。 若要透過入口網站刪除資源群組：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入資源群組的名稱。 當您在搜尋結果中看到此教學課程中使用的資源群組時，請加以選取。
2. 選取 [刪除資源群組]。
3. 在 [輸入資源群組名稱:] 方塊中輸入資源群組的名稱，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

若要深入了解文字分析服務，請閱讀[文字分析服務文件](index.yml)。
