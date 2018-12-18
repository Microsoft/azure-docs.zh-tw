---
title: 以範例作為起始
description: 本文將介紹開始使用 Power BI 工作區集合範例。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: d8a9ef78-ad4e-4bc7-9711-89172dc5c548
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/25/2017
ms.author: maghan
ms.openlocfilehash: cd7da8d2bbd684df29432bb924bdae3cd9ac7e64
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43044432"
---
# <a name="get-started-with-power-bi-workspace-collections-sample"></a>開始使用 Power BI 工作區集合範例

運用 **Microsoft Power BI 工作區集合**，您可以將 Power BI 報表整合至您的 Web 應用程式或行動應用程式。 本文將介紹開始使用 **Power BI 工作區集合**範例。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

您會想要在我們繼續討論之前先儲存下列資源：它們也可協助您將 Power BI 報表整合到範例應用程式和您自己的應用程式。

* [範例工作區 Web 應用程式](http://go.microsoft.com/fwlink/?LinkId=761493)
* [Power BI 工作區集合 API 參考](https://msdn.microsoft.com/library/azure/mt711507.aspx)
* [Power BI .NET SDK ](http://go.microsoft.com/fwlink/?LinkId=746472) \(英文\) (透過 NuGet 提供)
* [JavaScript 報告內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo)

> [!NOTE]
> 在您可以設定及執行開始使用 Power BI 工作區集合範例之前，需要在您的 Azure 訂用帳戶中至少建立一個**工作區集合**。 若要了解如何在 Azure 入口網站中建立**工作區集合**，請參閱[開始使用 Power BI 工作區集合](get-started.md)。

## <a name="configure-the-sample-app"></a>設定範例應用程式

讓我們逐步引導您設定 Visual Studio 開發環境，以存取執行範例應用程式時所需的元件。

1. 下載 GitHub 上的 [Power BI 工作區集合 - 將報表整合到 Web 應用程式](http://go.microsoft.com/fwlink/?LinkId=761493) \(英文\) 範例，然後解壓縮。
2. 在 Visual Studio 中開啟 **PowerBI-embedded.sln** 。 您可能需要在 NuGet 封裝管理員主控台中執行 **Update-Package** 命令，以更新此方案中使用的封裝。
3. 建置方案。
4. 執行 **ProvisionSample** 主控台應用程式。 您可以在範例主控台應用程式中，佈建工作區並匯入 PBIX 檔案。
5. 若要佈建新的**工作區**，請選取選項 1 [集合管理]，然後選取選項 6 [佈建新的工作區]。
6. 若要匯入新的**報告**，請選取選項 2 [報告管理]，然後選取選項 3 [將 PBIX 桌面檔案匯入工作區]。

7. 輸入您的 [工作區集合] 名稱，以及 [存取金鑰]。 您可以在 **Azure 入口網站**中取得這些項目。 若要深入了解如何取得存取金鑰 ，請參閱＜開始使用 Microsoft Power BI Embedded＞中的 [檢視 Power BI API 存取金鑰](get-started.md#view-power-bi-api-access-keys) 。

    ![存取 Azure 入口網站內的金鑰](media/get-started-sample/azure-portal.png)
8. 複製並儲存新建立的「工作區識別碼」  ，以供稍後在此文章中使用。 建立**工作區識別碼**之後，您就可以在 **Azure 入口網站**中找到它。

    ![Azure 入口網站內的工作區識別碼](media/get-started-sample/workspace-id.png)
9. 若要將 PBIX 檔案匯入到您的**工作區**，請選取選項 **6。** \[將 PBIX Desktop 檔案匯入現有的工作區\]。 如果您現在沒有 PBIX 檔案，可以下載[零售分析範例 PBIX](http://go.microsoft.com/fwlink/?LinkID=780547)。
10. 如果出現提示，請輸入易記名稱做為您「資料集」 的名稱。

您應該會看到像這樣的回應：

```
Checking import state... Publishing
Checking import state... Succeeded
```

> [!NOTE]
> 如果您的 PBIX 檔案包含任何直接查詢連接，請執行選項 7 來更新連接字串。

此時，您已經將 Power BI PBIX 報表匯入到您的**工作區**。 現在讓我們看一下如何執行開始使用 **Power BI 工作區集合**範例 Web 應用程式。

## <a name="run-the-sample-web-app"></a>執行範例 Web 應用程式

Web 應用程式範例是一個範例應用程式，會轉譯匯入到您**工作區**的報表。 以下說明如何設定 Web 應用程式範例。

1. 在 **PowerBI-embedded** Visual Studio 方案中，用滑鼠右鍵按一下 [EmbedSample] Web 應用程式，然後選擇 [設定為啟始專案]。
2. 在 **web.config** 中，於 **EmbedSample** Web 應用程式中編輯 **appSettings**：**AccessKey**、**WorkspaceCollection** 名稱，及 **WorkspaceId**。

    ```
    <appSettings>
        <add key="powerbi:AccessKey" value="" />
        <add key="powerbi:ApiUrl" value="https://api.powerbi.com" />
        <add key="powerbi:WorkspaceCollection" value="" />
        <add key="powerbi:WorkspaceId" value="" />
    </appSettings>
    ```
3. 執行 **EmbedSample** Web 應用程式。

在您執行 **EmbedSample** Web 應用程式之後，左邊瀏覽窗格應該就會包含一個 [多個報表] 功能表。 若要檢視您匯入的報表，請展開 [多個報表]，然後按一下報表。 如果您匯入了[零售分析範例 PBIX](http://go.microsoft.com/fwlink/?LinkID=780547)，範例 Web 應用程式看起來就會像這樣：

![範例應用程式中的範例左瀏覽](media/get-started-sample/sample-left-nav.png)

在您按一下報表之後，**EmbedSample** Web 應用程式應該看起來像這樣：

![應用程式中顯示的範例報表](media/get-started-sample/sample-web-app.png)

## <a name="explore-the-sample-code"></a>探討範例程式碼

**Microsoft Power BI 工作區集合**範例是一個範例 Web 應用程式，會示範如何將 **Power BI** 報表整合到您的應用程式。 它會使用「模型-檢視-控制器」(MVC) 設計樣式來示範最佳作法。 本節重點在於 **PowerBI-embedded** Web 應用程式方案中您可以探討的部分範例程式碼。 「模型-檢視-控制器」(MVC) 樣式會依據使用者在三種個別類型中的輸入來分隔網域、簡報及動作的模型製作：模型、檢視及控制器。 若要詳細了解 MVC，請參閱[了解 ASP.NET](http://www.asp.net/mvc)。

**Microsoft Power BI 工作區集合**範例程式碼的各部分如下。 每個區段都包含 PowerBI-embedded.sln 解決方案中的檔案名稱，因此您可以很容易地在範例中找到程式碼。

> [!NOTE]
> 本節是示範程式碼撰寫方式之範例程式碼的摘要。 若要檢視完整範例，請在 Visual Studio 中載入 PowerBI-embedded.sln 解決方案。

### <a name="model"></a>模型

範例有 **ReportsViewModel** 和 **ReportViewModel**。

**ReportsViewModel.cs**：代表 Power BI Reports。

    public class ReportsViewModel
    {
        public List<Report> Reports { get; set; }
    }

**ReportViewModel.cs**：代表 Power BI Report。

    public classReportViewModel
    {
        public IReport Report { get; set; }

        public string AccessToken { get; set; }
    }

### <a name="connection-string"></a>連接字串

連接字串必須為下列格式：

```
Data Source=tcp:MyServer.database.windows.net,1433;Initial Catalog=MyDatabase
```

使用通用的伺服器和資料庫屬性會失敗。 例如：Server=tcp:MyServer.database.windows.net,1433;Database=MyDatabase,

### <a name="view"></a>檢視

**檢視**可管理 Power BI **Reports** 和 **Power BI Report** 的顯示。

**Reports.cshtml**：反覆執行 **Model.Reports** 來建立 **ActionLink**。 **ActionLink** 是由以下項目組成：

| 部分 | 說明 |
| --- | --- |
| 標題 |報表名稱。 |
| QueryString |報表識別碼的連結。 |

    <div id="reports-nav" class="panel-collapse collapse">
        <div class="panel-body">
            <ul class="nav navbar-nav">
                @foreach (var report in Model.Reports)
                {
                    var reportClass = Request.QueryString["reportId"] == report.Id ? "active" : "";
                    <li class="@reportClass">
                        @Html.ActionLink(report.Name, "Report", new { reportId = report.Id })
                    </li>
                }
            </ul>
        </div>
    </div>

Report.cshtml：設定 **Model.AccessToken**，以及 **PowerBIReportFor** 的 Lambda 運算式。

    @model ReportViewModel

    ...

    <div class="side-body padding-top">
        @Html.PowerBIAccessToken(Model.AccessToken)
        @Html.PowerBIReportFor(m => m.Report, new { style = "height:85vh" })
    </div>

### <a name="controller"></a>Controller

**DashboardController.cs**：建立會傳遞**應用程式權杖**的 PowerBIClient。 JSON Web 權杖 (JWT) 是從**簽署金鑰**產生，可用於取得**認證**。 **Credentials** 是用來建立 **PowerBIClient** 的執行個體。 在您擁有 **PowerBIClient** 的執行個體之後，您就可以呼叫 GetReports() 與 GetReportsAsync()。

CreatePowerBIClient()

    private IPowerBIClient CreatePowerBIClient()
    {
        var credentials = new TokenCredentials(accessKey, "AppKey");
        var client = new PowerBIClient(credentials)
        {
            BaseUri = new Uri(apiUrl)
        };

        return client;
    }

ActionResult Reports()

    public ActionResult Reports()
    {
        using (var client = this.CreatePowerBIClient())
        {
            var reportsResponse = client.Reports.GetReports(this.workspaceCollection, this.workspaceId);

            var viewModel = new ReportsViewModel
            {
                Reports = reportsResponse.Value.ToList()
            };

            return PartialView(viewModel);
        }
    }


Task<ActionResult> Report(string reportId)

    public async Task<ActionResult> Report(string reportId)
    {
        using (var client = this.CreatePowerBIClient())
        {
            var reportsResponse = await client.Reports.GetReportsAsync(this.workspaceCollection, this.workspaceId);
            var report = reportsResponse.Value.FirstOrDefault(r => r.Id == reportId);
            var embedToken = PowerBIToken.CreateReportEmbedToken(this.workspaceCollection, this.workspaceId, report.Id);

            var viewModel = new ReportViewModel
            {
                Report = report,
                AccessToken = embedToken.Generate(this.accessKey)
            };

            return View(viewModel);
        }
    }

### <a name="integrate-a-report-into-your-app"></a>將報表整合到您的應用程式中

在您擁有 **Report** 之後，您就可以使用 **IFrame** 來內嵌 Power BI **Report**。 以下是來自 **Microsoft Power BI 工作區集合**範例中 powerbi.js 的程式碼片段。

```
init: function() {
    var embedUrl = this.getEmbedUrl();
    var iframeHtml = '<igrame style="width:100%;height:100%;" src="' + embedUrl + 
        '" scrolling="no" allowfullscreen="true"></iframe>';
    this.element.innerHTML = iframeHtml;
    this.iframe = this.element.childNodes[0];
    this.iframe.addEventListener('load', this.load.bind(this), false);
}
```

## <a name="filter-reports-embedded-in-your-application"></a>篩選內嵌在應用程式中的報表

您可以使用 URL 語法，篩選內嵌的報表。 若要這樣做，請將含有 **eq** 運算子的 **$filter** 查詢字串參數新增到含指定篩選的 iFrame src URL。 以下是篩選的查詢語法︰

```
https://app.powerbi.com/reportEmbed
?reportId=d2a0ea38-...-9673-ee9655d54a4a&
$filter={tableName/fieldName}%20eq%20'{fieldValue}'
```

> [!NOTE]
> {表格名稱/欄位名稱} 不能包含空格或特殊字元。 {欄位值} 接受單一類別目錄值。  

## <a name="see-also"></a>另請參閱

[常見的 Microsoft Power BI 工作區集合案例](scenarios.md)  
[在 Power BI 工作區集合中驗證和授權](app-token-flow.md)  
[內嵌報告](embed-report.md)  
[從資料集建立新的報告](create-report-from-dataset.md)  
[Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/)  
[JavaScript 內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)
