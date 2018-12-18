---
title: 在 Azure App Service 中建置 .NET Core 和 SQL Database Web 應用程式 | Microsoft Docs
description: 了解如何讓 .NET Core 應用程式在 Azure App Service 中運作，並連線至 SQL Database。
services: app-service\web
documentationcenter: dotnet
author: cephalin
manager: syntaxc4
editor: ''
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/11/2018
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: f870902e5bd5ef92d12d1e5e846696c4b26362a3
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39425097"
---
# <a name="tutorial-build-a-net-core-and-sql-database-web-app-in-azure-app-service"></a>教學課程：在 Azure App Service 中建置 .NET Core 和 SQL Database Web 應用程式

> [!NOTE]
> 本文會將應用程式部署至 Windows 上的 App Service。 若要在 _Linux_ 上部署至 App Service，請參閱[在 Linux 上的 Azure App Service 中建置 .NET Core 和 SQL Database Web 應用程式](./containers/tutorial-dotnetcore-sqldb-app.md)。
>

[App Service](app-service-web-overview.md) 可在 Azure 中提供可高度擴充、自我修復的 Web 主控服務。 本教學課程示範如何建立 .NET Core Web 應用程式，並將它連線到 SQL Database。 當您完成時，您就會有在 App Service 中執行的 .NET Core MVC 應用程式。

![在 App Service 中執行的應用程式](./media/app-service-web-tutorial-dotnetcore-sqldb/azure-app-in-browser.png)

您會了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立 SQL Database
> * 將 .NET Core 應用程式連線到 SQL Database
> * 將應用程式部署至 Azure
> * 將資料模型更新並將應用程式重新部署
> * 來自 Azure 的串流診斷記錄
> * 在 Azure 入口網站中管理應用程式

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

* [安裝 Git](https://git-scm.com/)
* [安裝 .NET Core](https://www.microsoft.com/net/core/)

## <a name="create-local-net-core-app"></a>建立本機 .NET Core 應用程式

在此步驟中，您要設定本機的 .NET Core 專案。

### <a name="clone-the-sample-application"></a>複製範例應用程式

在終端機視窗中，使用 `cd` 移至工作目錄。

執行下列命令來複製範例存放庫，然後變更至其根目錄。

```bash
git clone https://github.com/azure-samples/dotnetcore-sqldb-tutorial
cd dotnetcore-sqldb-tutorial
```

範例專案包含一個基本 CRUD (建立-讀取-更新-刪除) 應用程式，使用 [Entity Framework Core](https://docs.microsoft.com/ef/core/)。

### <a name="run-the-application"></a>執行應用程式

執行下列命令以安裝必要的套件、執行資料庫移轉，然後啟動應用程式。

```bash
dotnet restore
dotnet ef database update
dotnet run
```

在瀏覽器中，瀏覽至 `http://localhost:5000` 。 選取 [新建] 連結，並且建立幾個 [待辦事項] 項目。

![成功連線至 SQL Database](./media/app-service-web-tutorial-dotnetcore-sqldb/local-app-in-browser.png)

如需隨時停止 .NET Core，請在終端機上按下 `Ctrl+C`。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-production-sql-database"></a>建立生產環境 SQL Database

在此步驟中，您要在 Azure 中建立 SQL Database。 當您的應用程式部署至 Azure 時，它會使用此雲端資料庫。

對於 SQL Database，本教學課程使用 [Azure SQL Database](/azure/sql-database/)。

### <a name="create-a-resource-group"></a>建立資源群組

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)]

### <a name="create-a-sql-database-logical-server"></a>連線至 SQL Database 邏輯伺服器

在 Cloud Shell 中，使用 [`az sql server create`](/cli/azure/sql/server?view=azure-cli-latest#az-sql-server-create) 命令建立 SQL Database 邏輯伺服器。

將 \<server_name> 預留位置取代為唯一的 SQL Database 名稱。 這個名稱會用來作為 SQL Database 端點 `<server_name>.database.windows.net` 的一部分，因此，這個名稱在 Azure 中的所有邏輯伺服器必須是唯一的。 名稱只能包含小寫字母、數字及連字號 (-) 字元，且長度必須為 3 到 50 個字元。 此外，將 \<db_username> 和 \<db_password> 取代為您選擇的使用者名稱和密碼。 


```azurecli-interactive
az sql server create --name <server_name> --resource-group myResourceGroup --location "West Europe" --admin-user <db_username> --admin-password <db_password>
```

建立 SQL Database 邏輯伺服器後，Azure CLI 會顯示類似下列範例的資訊：

```json
{
  "administratorLogin": "sqladmin",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "<server_name>.database.windows.net",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/<server_name>",
  "identity": null,
  "kind": "v12.0",
  "location": "westeurope",
  "name": "<server_name>",
  "resourceGroup": "myResourceGroup",
  "state": "Ready",
  "tags": null,
  "type": "Microsoft.Sql/servers",
  "version": "12.0"
}
```

### <a name="configure-a-server-firewall-rule"></a>設定伺服器防火牆規則

使用 [`az sql server firewall create`](/cli/azure/sql/server/firewall-rule?view=azure-cli-latest#az-sql-server-firewall-rule-create) 命令建立 [Azure SQL Database 伺服器層級防火牆規則](../sql-database/sql-database-firewall-configure.md)。 當起始 IP 和結束 IP 都設為 0.0.0.0 時，防火牆只會為其他 Azure 資源開啟。 

```azurecli-interactive
az sql server firewall-rule create --resource-group myResourceGroup --server <server_name> --name AllowYourIp --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP] 
> [僅使用您的應用程式所用的輸出 IP 位址](app-service-ip-addresses.md#find-outbound-ips)，讓您的防火牆規則更具限制性。
>

### <a name="create-a-database"></a>建立資料庫

使用 [`az sql db create`](/cli/azure/sql/db?view=azure-cli-latest#az-sql-db-create) 命令在伺服器中建立具有 [S0 效能等級](../sql-database/sql-database-service-tiers-dtu.md)的資料庫。

```azurecli-interactive
az sql db create --resource-group myResourceGroup --server <server_name> --name coreDB --service-objective S0
```

### <a name="create-connection-string"></a>建立連接字串

將下列字串取代為您稍早使用的 \<server_name>、\<db_username> 和 \<db_password>。

```
Server=tcp:<server_name>.database.windows.net,1433;Database=coreDB;User ID=<db_username>;Password=<db_password>;Encrypt=true;Connection Timeout=30;
```

此為您 .NET Core 應用程式的連接字串。 複製它以供稍後使用。

## <a name="deploy-app-to-azure"></a>將應用程式部署到 Azure

在此步驟中，您會將與 SQL Database 連線的 .NET Core 應用程式部署至 App Service。

### <a name="configure-local-git-deployment"></a>設定本機 git 部署

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>建立應用程式服務方案

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-no-h.md)]

### <a name="create-a-web-app"></a>建立 Web 應用程式

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-dotnetcore-win-no-h.md)] 

### <a name="configure-an-environment-variable"></a>設定環境變數

若要設定 Azure 應用程式的連接字串，請在 Cloud Shell 中使用 [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 在下列命令中，將 \<app name> 以及 \<connection_string> 參數取代為您稍早建立的連接字串。

```azurecli-interactive
az webapp config connection-string set --resource-group myResourceGroup --name <app name> --settings MyDbConnection='<connection_string>' --connection-string-type SQLServer
```

接下來，將 `ASPNETCORE_ENVIRONMENT` 應用程式設定設為_生產_。 此設定可讓您知道您是否正在 Azure 中執行，因為您針對本機開發環境使用 SQLite，針對 Azure 環境使用 SQL Database。

下列範例會在 Azure Web 應用程式中設定 `ASPNETCORE_ENVIRONMENT` 應用程式設定。 取代 \<app_name> 預留位置。

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group myResourceGroup --settings ASPNETCORE_ENVIRONMENT="Production"
```

### <a name="connect-to-sql-database-in-production"></a>在生產環境中連線到 SQL Database

在您的本機存放庫中，開啟 Startup.cs 並且尋找下列程式碼：

```csharp
services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
```

將它取代為下列程式碼，該程式碼使用您先前設定的環境變數。

```csharp
// Use SQL Database if in Azure, otherwise, use SQLite
if(Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") == "Production")
    services.AddDbContext<MyDatabaseContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
else
    services.AddDbContext<MyDatabaseContext>(options =>
            options.UseSqlite("Data Source=localdatabase.db"));

// Automatically perform database migration
services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
```

如果此程式碼偵測到正在生產環境 (表示 Azure 環境) 中執行，則它會使用您設定的連接字串，連線到 SQL Database。

`Database.Migrate()` 呼叫可在於 Azure 中執行時協助您，因為它會根據其移轉設定，自動建立您的 .NET Core 應用程式需要的資料庫。 

儲存變更，然後將變更認可至 Git 存放庫中。 

```bash
git add .
git commit -m "connect to SQLDB in Azure"
```

### <a name="push-to-azure-from-git"></a>從 Git 推送至 Azure

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

```bash
Counting objects: 98, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (92/92), done.
Writing objects: 100% (98/98), 524.98 KiB | 5.58 MiB/s, done.
Total 98 (delta 8), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: .
remote: Updating submodules.
remote: Preparing deployment for commit id '0c497633b8'.
remote: Generating deployment script.
remote: Project file path: ./DotNetCoreSqlDb.csproj
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling ASP.NET Core Web Application deployment.
remote: .
remote: .
remote: .
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Deployment successful.
remote: App container will begin restart within 10 seconds.
To https://<app_name>.scm.azurewebsites.net/<app_name>.git
 * [new branch]      master -> master
```

### <a name="browse-to-the-azure-web-app"></a>瀏覽至 Azure Web 應用程式

使用 Web 瀏覽器，瀏覽至已部署的 Web 應用程式。

```bash
http://<app_name>.azurewebsites.net
```

新增幾個待辦事項項目。

![在 App Service 中執行的應用程式](./media/app-service-web-tutorial-dotnetcore-sqldb/azure-app-in-browser.png)

**恭喜！** 您正在 App Service 中執行資料驅動的 .NET Core 應用程式。

## <a name="update-locally-and-redeploy"></a>在本機更新和重新部署

在此步驟中，您會對資料庫結構描述進行變更，並且將它發佈至 Azure。

### <a name="update-your-data-model"></a>更新資料模型

在程式碼編輯器中開啟 _Models\Todo.cs_。 將下列屬性加入至 `ToDo` 類別：

```csharp
public bool Done { get; set; }
```

### <a name="run-code-first-migrations-locally"></a>在本機執行 Code First 移轉

執行數個命令以進行本機資料庫的更新。

```bash
dotnet ef migrations add AddProperty
```

更新本機資料庫：

```bash
dotnet ef database update
```

### <a name="use-the-new-property"></a>使用新屬性

在您的程式碼中進行一些變更以使用 `Done` 屬性。 為了簡單起見，在本教學課程中，您僅需變更 `Index` 和 `Create` 檢視，以查看作用中的屬性。

開啟 _Controllers\TodosController.cs_。

尋找 `Create([Bind("ID,Description,CreatedDate")] Todo todo)` 方法，並將 `Done` 加入至 `Bind` 屬性 (Attribute) 中的屬性 (Property) 清單。 完成時，您的 `Create()` 方法簽章應該如以下程式碼所示：

```csharp
public async Task<IActionResult> Create([Bind("ID,Description,CreatedDate,Done")] Todo todo)
```

開啟 _Views\Todos\Create.cshtml_。

在 Razor 程式碼中，您應該會看到 `Description` 的 `<div class="form-group">` 元素，然後是另一個 `CreatedDate` 的 `<div class="form-group">` 元素。 在這兩個元素的正後方，新增另一個 `Done` 的 `<div class="form-group">` 元素：

```csharp
<div class="form-group">
    <label asp-for="Done" class="col-md-2 control-label"></label>
    <div class="col-md-10">
        <input asp-for="Done" class="form-control" />
        <span asp-validation-for="Done" class="text-danger"></span>
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

尋找包含 `asp-action` 標記協助程式的 `<td>` 元素。 在此元素的正上方，新增下列 Razor 程式碼：

```csharp
<td>
    @Html.DisplayFor(modelItem => item.Done)
</td>
```

您只需查看 `Index` 和 `Create` 檢視中的變更。

### <a name="test-your-changes-locally"></a>本機測試您的變更

在本機執行應用程式。

```bash
dotnet run
```

在瀏覽器中，瀏覽至 `http://localhost:5000/`。 您現在可以新增待辦事項項目，並且勾選 [完成]。 然後，它應該會在您的首頁中顯示為已完成的項目。 請記住，[`Edit`] 檢視不會顯示 [`Done`] 欄位，因為您沒有變更 [`Edit`] 檢視。

### <a name="publish-changes-to-azure"></a>將變更發佈至 Azure

```bash
git add .
git commit -m "added done field"
git push azure master
```

完成 `git push` 之後，瀏覽至 Azure Web 應用程式，然後嘗試執行新功能。

![Code First 移轉之後的 Azure Web 應用程式](./media/app-service-web-tutorial-dotnetcore-sqldb/this-one-is-done.png)

仍會顯示您現有的所有待辦事項項目。 當您重新發佈 .NET Core 應用程式時，您 SQL Database 中現有的資料不會遺失。 此外，Entity Framework Code 移轉只會變更資料結構描述，並讓現有的資料保持不變。

## <a name="manage-your-azure-web-app"></a>管理您的 Azure Web 應用程式

請移至 [Azure 入口網站](https://portal.azure.com)，以查看您所建立的 Web 應用程式。

按一下左側功能表中的 [應用程式服務]，然後按一下 Azure Web 應用程式的名稱。

![入口網站瀏覽至 Azure Web 應用程式](./media/app-service-web-tutorial-dotnetcore-sqldb/access-portal.png)

根據預設，入口網站會顯示 Web 應用程式的 [概觀] 分頁。 此頁面可讓您檢視應用程式的執行方式。 您也可以在這裡執行基本管理工作，像是瀏覽、停止、啟動、重新啟動及刪除。 分頁左側的索引標籤會顯示您可開啟的各種設定分頁。

![Azure 入口網站中的 App Service 頁面](./media/app-service-web-tutorial-dotnetcore-sqldb/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>後續步驟

您已了解如何︰

> [!div class="checklist"]
> * 在 Azure 中建立 SQL Database
> * 將 .NET Core 應用程式連線到 SQL Database
> * 將應用程式部署至 Azure
> * 將資料模型更新並將應用程式重新部署
> * 將記錄從 Azure 串流到終端機
> * 在 Azure 入口網站中管理應用程式

前往下一個教學課程，了解如何將自訂的 DNS 名稱對應至 Web 應用程式。

> [!div class="nextstepaction"]
> [將現有的自訂 DNS 名稱對應至 Azure Web Apps](app-service-web-tutorial-custom-domain.md)
