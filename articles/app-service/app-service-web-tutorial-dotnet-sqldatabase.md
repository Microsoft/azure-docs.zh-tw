---
title: 在 Azure 中搭配 SQL Database 來建置 ASP.NET 應用程式 | Microsoft Docs
description: 了解如何將採用 SQL Server 資料庫的 C# ASP.NET 應用程式部署到 Azure。
services: app-service\web
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: 03c584f1-a93c-4e3d-ac1b-c82b50c75d3e
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: csharp
ms.topic: tutorial
ms.date: 06/25/2018
ms.author: cephalin
ms.custom: mvc, devcenter, vs-azure
ms.openlocfilehash: 783bf93c8507e76717a4293b2b29a9c11e9a1eed
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353811"
---
# <a name="tutorial-build-an-aspnet-app-in-azure-with-sql-database"></a>教學課程：在 Azure 中搭配 SQL Database 來建置 ASP.NET 應用程式

[Azure Web Apps](app-service-web-overview.md) 提供可高度擴充、自我修復的 Web 主機服務。 本教學課程示範如何在 Azure 中開發資料導向的 ASP.NET Web 應用程式，並且將它連線到 [Azure SQL Database](../sql-database/sql-database-technical-overview.md)。 完成時，您的 ASP.NET 應用程式將會在 Azure 中執行，並已連線到 SQL Database。

![已在 Azure Web 應用程式中發佈的 ASP.NET 應用程式](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-app-in-browser.png)

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立 SQL Database
> * 將 ASP.NET 應用程式連線到 SQL Database
> * 將應用程式部署至 Azure
> * 將資料模型更新並將應用程式重新部署
> * 將記錄從 Azure 串流到終端機
> * 在 Azure 入口網站中管理應用程式

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

安裝包含 **ASP.NET 和 Web 開發**工作負載的 <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a>。

如果您已安裝 Visual Studio，請按一下 [工具] > [取得工具和功能] 在 Visual Studio 中新增工作負載。

## <a name="download-the-sample"></a>下載範例

- [下載範例專案](https://github.com/Azure-Samples/dotnet-sqldb-tutorial/archive/master.zip)。
- 擷取 (解壓縮) dotnet-sqldb-tutorial-master.zip 檔案。

範例專案包含一個使用 [Entity Framework Code First](/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application) 的基本 [ASP.NET MVC](https://www.asp.net/mvc) 建立-讀取-更新-刪除 (CRUD) 應用程式。

### <a name="run-the-app"></a>執行應用程式

在 Visual Studio 中開啟 dotnet-sqldb-tutorial-master/DotNetAppSqlDb.sln 檔案。 

輸入 `Ctrl+F5` 以執行應用程式而不偵錯。 應用程式會在預設瀏覽器中顯示。 選取 [新建] 連結，並且建立幾個 [待辦事項] 項目。 

![[新增 ASP.NET 專案] 對話方塊](media/app-service-web-tutorial-dotnet-sqldatabase/local-app-in-browser.png)

測試 [編輯]、[詳細資料] 和 [刪除] 連結。

應用程式會使用資料庫內容，與資料庫連線。 在這個範例中，資料庫內容會使用名為 `MyDbConnection` 的連接字串。 連接字串是在 Web.config 檔案中設定，並在 Models/MyDatabaseContext.cs 檔案中進行參考。 稍後會在教學課程中使用連接字串名稱，將 Azure Web 應用程式連線到 Azure SQL Database。 

## <a name="publish-to-azure-with-sql-database"></a>發佈至含有 SQL Database 的 Azure

在 [方案總管] 中，以滑鼠右鍵按一下 [DotNetAppSqlDb] 專案，然後選取 [發佈]。

![從方案總管發佈](./media/app-service-web-tutorial-dotnet-sqldatabase/solution-explorer-publish.png)

確定已選取 [Microsoft Azure App Service]，然後按一下 [發佈]。

![從專案概觀頁面發佈](./media/app-service-web-tutorial-dotnet-sqldatabase/publish-to-app-service.png)

發佈會開啟 [建立 App Service] 對話方塊，協助您建立在 Azure 中執行 ASP.NET Web 應用程式所需的所有 Azure 資源。

### <a name="sign-in-to-azure"></a>登入 Azure

在 [建立 App Service] 對話方塊中，按一下 [新增帳戶]，然後登入您的 Azure 訂用帳戶。 如果您已登入 Microsoft 帳戶，請確定該帳戶保留您的 Azure 訂用帳戶。 如果登入的 Microsoft 帳戶沒有 Azure 訂用帳戶，請按一下它來新增正確的帳戶。 

> [!NOTE]
> 如果您已經登入，請勿選取 [建立]。
>
>
   
![登入 Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/sign-in-azure.png)

### <a name="configure-the-web-app-name"></a>設定 Web 應用程式名稱

您可以保留產生的 Web 應用程式名稱，或將它變更為另一個唯一的名稱 (有效的字元是 `a-z`、`0-9` 和 `-`)。 Web 應用程式名稱是作為應用程式預設 URL 的一部分 (`<app_name>.azurewebsites.net`，其中 `<app_name>` 是您的 Web 應用程式名稱)。 Web 應用程式名稱在 Azure 中的所有應用程式之間必須是唯一的。 

![建立 App Service 對話方塊](media/app-service-web-tutorial-dotnet-sqldatabase/wan.png)

### <a name="create-a-resource-group"></a>建立資源群組

[!INCLUDE [resource-group](../../includes/resource-group.md)]

按一下 [資源群組] 旁邊的 [新增]。

![按一下 [資源群組] 旁邊的 [新增]。](media/app-service-web-tutorial-dotnet-sqldatabase/new_rg2.png)

將資源群組命名為 **myResourceGroup**。

### <a name="create-an-app-service-plan"></a>建立應用程式服務方案

[!INCLUDE [app-service-plan](../../includes/app-service-plan.md)]

按一下 [App Service 方案] 旁邊的 [新增]。 

在 [設定 App Service 方案] 對話方塊中，使用下列設定來設定新的 App Service 方案︰

![建立 App Service 方案](./media/app-service-web-tutorial-dotnet-sqldatabase/configure-app-service-plan.png)

| 設定  | 建議的值 | 取得詳細資訊 |
| ----------------- | ------------ | ----|
|**App Service 方案**| myAppServicePlan | [App Service 方案](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md) |
|**位置**| 西歐 | [Azure 區域](https://azure.microsoft.com/regions/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) |
|**大小**| 免費 | [定價層](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)|

### <a name="create-a-sql-server-instance"></a>建立 SQL Server 執行個體

建立資料庫之前，您需要 [Azure SQL Database 邏輯伺服器](../sql-database/sql-database-features.md)。 邏輯伺服器包含一組當作群組管理的資料庫。

按一下 [建立 SQL Database]。

![建立 SQL Database](media/app-service-web-tutorial-dotnet-sqldatabase/web-app-name.png)

在 [設定 SQL Database] 對話方塊中，按一下 [SQL Server] 旁的 [新增]。 

唯一的伺服器名稱隨即產生。 這個名稱是作為邏輯伺服器預設 URL 的一部分，`<server_name>.database.windows.net`。 它在 Azure 中的所有邏輯伺服器執行個體之間必須是唯一的。 您可以變更伺服器名稱，但是在本教學課程中，請保留產生的值。

新增系統管理員使用者名稱和密碼。 如需密碼複雜性需求，請參閱[密碼原則](/sql/relational-databases/security/password-policy)。

請記住這個使用者名稱和密碼。 您稍後需要它們以便管理邏輯伺服器執行個體。

> [!IMPORTANT]
> 即使您在連接字串中的密碼已經遮罩處理 (在 Visual Studio 以及 App Service 中)，但是它於某處受到維護的事實增加了您應用程式受到攻擊的可能性。 App Service 可以使用[受控服務識別](app-service-managed-service-identity.md)來降低此風險，方法是完全免除在您的程式碼或應用程式組態中維護祕密的需求。 如需詳細資訊，請參閱[後續步驟](#next-steps)。

![建立 SQL Server 執行個體](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database-server.png)

按一下 [確定]。 尚不要關閉 [設定 SQL Database] 對話方塊。

### <a name="create-a-sql-database"></a>建立 SQL Database

在 [設定 SQL Database] 對話方塊中︰ 

* 保留預設產生的**資料庫名稱**。
* 在 [連接字串名稱] 中，輸入 *MyDbConnection*。 此名稱必須符合 Models/MyDatabaseContext.cs 中所參考的連接字串。
* 選取 [確定] 。

![設定 SQL Database](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database.png)

[建立 App Service] 對話方塊會顯示您已設定的資源。 按一下頁面底部的 [新增] 。 

![您已建立的資源](media/app-service-web-tutorial-dotnet-sqldatabase/app_svc_plan_done.png)

一旦精靈完成建立 Azure 資源，它會將 ASP.NET 應用程式發佈至 Azure。 預設瀏覽器隨即啟動，瀏覽至部署應用程式的 URL。 

新增幾個待辦事項項目。

![已在 Azure Web 應用程式中發佈的 ASP.NET 應用程式](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-app-in-browser.png)

恭喜！ 您的資料導向 ASP.NET 應用程式正在 Azure App Service 中執行。

## <a name="access-the-sql-database-locally"></a>在本機存取 SQL Database

Visual Studio 可讓您在 [SQL Server 物件總管] 中，輕鬆地探索和管理新的 SQL Database。

### <a name="create-a-database-connection"></a>建立資料庫連接

從 [檢視] 功能表選取 [SQL Server 物件總管]。

在 [SQL Server 物件總管] 上方，按一下 [加入 SQL Server] 按鈕。

### <a name="configure-the-database-connection"></a>設定資料庫連接

在 [連接] 對話方塊中，展開 [Azure] 節點。 此處會列出 Azure 中您所有的 SQL Database 執行個體。

選取您稍早建立的 SQL Database。 系統會自動在下方填入您稍早建立的連線。

輸入您稍早建立的資料庫管理員密碼，然後按一下 [連線]。

![從 Visual Studio 設定資料庫連接](./media/app-service-web-tutorial-dotnet-sqldatabase/connect-to-sql-database.png)

### <a name="allow-client-connection-from-your-computer"></a>允許來自您電腦的用戶端連接

[建立新的防火牆規則] 對話方塊即會開啟。 根據預設，您的 SQL Database 執行個體僅允許來自 Azure 服務 (例如 Azure Web 應用程式) 的連線。 若要連線到您的資料庫，請在 SQL Database 執行個體中建立防火牆規則。 防火牆規則允許本機電腦的公用 IP 位址。

對話方塊中已經填入您的電腦公用 IP 位址。

確定已選取 [加入我的用戶端 IP]，然後按一下 [確定]。 

![設定 SQL Database 執行個體的防火牆](./media/app-service-web-tutorial-dotnet-sqldatabase/sql-set-firewall.png)

當 Visual Studio 完成建立 SQL Database 執行個體的防火牆設定時，您的連線就會出現在 [SQL Server 物件總管] 中。

您可以在此處執行最常見的資料庫作業，例如執行查詢、建立檢視表和預存程序，以及其他更多作業。 

展開您的連線 > [資料庫]  >  **&lt;your database>** >  [資料表]。 以滑鼠右鍵按一下 `Todoes` 資料表並選取 [檢視資料]。 

![探索 SQL Database 物件](./media/app-service-web-tutorial-dotnet-sqldatabase/explore-sql-database.png)

## <a name="update-app-with-code-first-migrations"></a>使用 Code First 移轉更新應用程式

您可以使用 Visual Studio 中熟悉的工具來更新您在 Azure 中的資料庫與 Web 應用程式。 在此步驟中，您會使用 Entity Framework 中的 Code First 移轉來變更資料庫結構描述，然後將它發佈至 Azure。

如需有關使用 Entity Framework Code First 移轉的詳細資訊，請參閱[使用 MVC 5 開始使用 Entity Framework 6 Code First](https://docs.microsoft.com/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application)。

### <a name="update-your-data-model"></a>更新資料模型

在程式碼編輯器中開啟 _Models\Todo.cs_。 將下列屬性加入至 `ToDo` 類別：

```csharp
public bool Done { get; set; }
```

### <a name="run-code-first-migrations-locally"></a>在本機執行 Code First 移轉

執行數個命令以進行本機資料庫的更新。 

從 [工具] 功能表中，按一下 [NuGet 封裝管理員] > [封裝管理員主控台]。

在 [套件管理員主控台] 視窗中，啟用 Code First 移轉：

```PowerShell
Enable-Migrations
```

新增移轉：

```PowerShell
Add-Migration AddProperty
```

更新本機資料庫：

```PowerShell
Update-Database
```

輸入 `Ctrl+F5` 以執行應用程式。 測試編輯、詳細資料和建立連結。

如果應用程式載入而不會出現錯誤，則表示 Code First 移轉已成功。 不過，您的頁面仍然看起來一樣，因為您的應用程式邏輯尚未使用這個新屬性。 

### <a name="use-the-new-property"></a>使用新屬性

在您的程式碼中進行一些變更以使用 `Done` 屬性。 為了簡單起見，在本教學課程中，您僅需變更 `Index` 和 `Create` 檢視，以查看作用中的屬性。

開啟 _Controllers\TodosController.cs_。

尋找位於行 52 的 `Create()` 方法，並將 `Done` 新增至 `Bind` 屬性 (Attribute) 中的屬性 (Property) 清單。 完成時，您的 `Create()` 方法簽章應該如以下程式碼所示：

```csharp
public ActionResult Create([Bind(Include = "Description,CreatedDate,Done")] Todo todo)
```

開啟 _Views\Todos\Create.cshtml_。

在 Razor 程式碼中，您應該會看到使用 `model.Description` 的 `<div class="form-group">` 元素，然後是另一個使用 `model.CreatedDate` 的 `<div class="form-group">` 元素。 在這兩個元素的正後方，新增另一個使用 `model.Done` 的 `<div class="form-group">` 元素：

```csharp
<div class="form-group">
    @Html.LabelFor(model => model.Done, htmlAttributes: new { @class = "control-label col-md-2" })
    <div class="col-md-10">
        <div class="checkbox">
            @Html.EditorFor(model => model.Done)
            @Html.ValidationMessageFor(model => model.Done, "", new { @class = "text-danger" })
        </div>
    </div>
</div>
```

開啟 _Views\Todos\Index.cshtml_。

搜尋空白的 `<th></th>` 元素。 在此元素的正上方，新增下列 Razor 程式碼：

```csharp
<th>
    @Html.DisplayNameFor(model => model.Done)
</th>
```

尋找包含 `Html.ActionLink()` Helper 方法的 `<td>` 元素。 在這個 `<td>` _上方_，使用下列 Razor 程式碼新增另一個 `<td>` 元素：

```csharp
<td>
    @Html.DisplayFor(modelItem => item.Done)
</td>
```

您只需查看 `Index` 和 `Create` 檢視中的變更。 

輸入 `Ctrl+F5` 以執行應用程式。

您現在可以新增待辦事項項目，並且勾選 [完成]。 然後，它應該會在您的首頁中顯示為已完成的項目。 請記住，[`Edit`] 檢視不會顯示 [`Done`] 欄位，因為您沒有變更 [`Edit`] 檢視。

### <a name="enable-code-first-migrations-in-azure"></a>啟用 Azure 中的 Code First 移轉

您的程式碼變更會立即運作 (包括資料庫移轉)，您也可以將它發佈至 Azure Web 應用程式，然後使用 Code First 移轉來更新您的 SQL Database。

就像之前一樣，以滑鼠右鍵按一下專案，然後選取 [發佈]。

按一下 [設定] 來開啟發佈設定。

![開啟發佈設定](./media/app-service-web-tutorial-dotnet-sqldatabase/publish-settings.png)

在精靈中，按 [下一步]。

確定 [MyDatabaseContext (MyDbConnection)] 中已填入您的 SQL Database 連接字串。 您可能需要從下拉式清單中選取 [myToDoAppDb] 資料庫。 

選取 [執行 Code First 移轉 (在應用程式啟動時執行)] ，然後按一下 [儲存]。

![在 Azure Web 應用程式中啟用 Code First 移轉](./media/app-service-web-tutorial-dotnet-sqldatabase/enable-migrations.png)

### <a name="publish-your-changes"></a>發佈您的變更

現在，您已在 Azure Web 應用程式中啟用 Code First 移轉，只需發佈您的程式碼變更即可。

在發佈頁面中，按一下 [發佈]。

嘗試再次新增待辦事項，然後選取 [完成]，而它們應該會在您的首頁中顯示為已完成的項目。

![Code First 移轉之後的 Azure Web 應用程式](./media/app-service-web-tutorial-dotnet-sqldatabase/this-one-is-done.png)

仍會顯示您現有的所有待辦事項項目。 當您重新發佈 ASP.NET 應用程式時，您 SQL Database 中現有的資料不會遺失。 此外，Code First 移轉只會變更資料結構描述，並讓現有的資料保持不變。


## <a name="stream-application-logs"></a>資料流應用程式記錄

您可以將追蹤訊息從 Azure Web 應用程式直接資料流處理到 Visual Studio。

開啟 _Controllers\TodosController.cs_。

每個動作都是以 `Trace.WriteLine()` 方法開始。 新增此程式碼的用意是，示範如何將追蹤訊息新增至 Azure Web 應用程式。

### <a name="open-server-explorer"></a>開啟伺服器總管

從 [檢視] 功能表選取 [伺服器總管]。 您可以在 [伺服器總管] 中設定 Azure Web 應用程式的記錄。 

### <a name="enable-log-streaming"></a>啟用記錄資料流

在 [伺服器總管] 中，展開 [Azure] > [App Service]。

展開 [myResourceGroup] 資源群組，這是您在第一次建立 Azure Web 應用程式時所建立的資源群組。

以滑鼠右鍵按一下您的 Azure Web 應用程式，然後選取 [檢視資料流記錄]。

![啟用記錄資料流](./media/app-service-web-tutorial-dotnet-sqldatabase/stream-logs.png)

現在已將記錄資料流處理至 [輸出] 視窗。 

![[輸出] 視窗中的記錄資料流](./media/app-service-web-tutorial-dotnet-sqldatabase/log-streaming-pane.png)

不過，您還不會看到任何追蹤訊息。 這是因為當您第一次選取 [檢視資料流記錄] 時，您的 Azure Web 應用程式會將追蹤層級設定為 `Error`，這只會記錄錯誤事件 (使用 `Trace.TraceError()` 方法)。

### <a name="change-trace-levels"></a>變更追蹤層級

若要變更追蹤層級以輸出其他追蹤訊息，請返回 [伺服器總管]。

再次以滑鼠右鍵按一下您的 Azure Web 應用程式，然後選取 [檢視設定]。

在 [應用程式記錄 (檔案系統)] 下拉式清單中，選取 [詳細資訊]。 按一下 [檔案] 。

![將追蹤層級變更為詳細資訊](./media/app-service-web-tutorial-dotnet-sqldatabase/trace-level-verbose.png)

> [!TIP]
> 您可以試驗不同的追蹤層級，以查看針對每個層級顯示的訊息類型。 例如，[資訊] 層級包含 `Trace.TraceInformation()`、`Trace.TraceWarning()` 和`Trace.TraceError()` 所建立的所有記錄，但不包含 `Trace.WriteLine()` 所建立的記錄。
>
>

在您的瀏覽器中，再次巡覽至位於 *http://&lt;your app name>.azurewebsites.net* 的您的 Web 應用程式，然後在 Azure 中嘗試按一按待辦事項清單應用程式。 追蹤訊息現在會資料流處理到 Visual Studio 的 [輸出] 視窗中。

```console
Application: 2017-04-06T23:30:41  PID[8132] Verbose     GET /Todos/Index
Application: 2017-04-06T23:30:43  PID[8132] Verbose     GET /Todos/Create
Application: 2017-04-06T23:30:53  PID[8132] Verbose     POST /Todos/Create
Application: 2017-04-06T23:30:54  PID[8132] Verbose     GET /Todos/Index
```



### <a name="stop-log-streaming"></a>停止記錄資料流

若要停止記錄資料流服務，按一下 [輸出] 視窗中的 [停止監視] 按鈕。

![停止記錄資料流](./media/app-service-web-tutorial-dotnet-sqldatabase/stop-streaming.png)

## <a name="manage-your-azure-web-app"></a>管理您的 Azure Web 應用程式

請移至 [Azure 入口網站](https://portal.azure.com)，以查看您所建立的 Web 應用程式。 



按一下左側功能表中的 [App Service]，然後按一下 Azure Web 應用程式的名稱。

![入口網站瀏覽至 Azure Web 應用程式](./media/app-service-web-tutorial-dotnet-sqldatabase/access-portal.png)

您已來到 Web 應用程式的分頁。 

根據預設，入口網站會顯示 [概觀] 分頁。 此頁面可讓您檢視應用程式的執行方式。 您也可以在這裡執行基本管理工作，像是瀏覽、停止、啟動、重新啟動及刪除。 分頁左側的索引標籤會顯示您可開啟的各種設定分頁。 

![Azure 入口網站中的 App Service 頁面](./media/app-service-web-tutorial-dotnet-sqldatabase/web-app-blade.png)

[!INCLUDE [Clean up section](../../includes/clean-up-section-portal-web-app.md)]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立 SQL Database
> * 將 ASP.NET 應用程式連線到 SQL Database
> * 將應用程式部署至 Azure
> * 將資料模型更新並將應用程式重新部署
> * 將記錄從 Azure 串流到終端機
> * 在 Azure 入口網站中管理應用程式

前進到下一個教學課程，以了解如何輕鬆改善 Azure SQL Database 連線的安全性。

> [!div class="nextstepaction"]
> [使用 Azure 資源的受控識別安全地存取 SQL Database](app-service-web-tutorial-connect-msi.md)
