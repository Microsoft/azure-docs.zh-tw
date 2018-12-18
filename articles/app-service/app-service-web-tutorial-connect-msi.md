---
title: 使用受控識別保護來自 App Service 的 Azure SQL Database 連線 | Microsoft Docs
description: 了解如何使用受控識別，讓資料庫連線更加安全，以及如何將此受控服務識別套用到其他 Azure 服務。
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
ms.date: 10/24/2018
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: 6a3bb5511828d9f8ea7168ffa4748b141484299f
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376425"
---
# <a name="tutorial-secure-azure-sql-database-connection-from-app-service-using-a-managed-identity"></a>教學課程：使用受控識別保護來自 App Service 的 Azure SQL Database 連線

[App Service](app-service-web-overview.md) 可在 Azure 中提供可高度擴充、自我修復的 Web 主控服務。 它也為您的應用程式提供[受控識別](app-service-managed-service-identity.md)，這是用於保護 [Azure SQL Database](/azure/sql-database/) 和其他 Azure 服務存取權的周全解決方案。 App Service 中的受控識別可藉由從應用程式刪除祕密 (例如連接字串中的認證)，讓應用程式更加安全。 在本教學課程中，您會將受控識別新增至您在[教學課程：在 Azure 中搭配 SQL Database 來建置 ASP.NET 應用程式](app-service-web-tutorial-dotnet-sqldatabase.md)中建立的範例 ASP.NET Web 應用程式。 當您完成時，範例應用程式不需要使用者名稱和密碼，即可安全地連線到 SQL Database。

> [!NOTE]
> 目前只有 .NET Framework 4.6 和以上版本支援此案例，但 [.NET Core 2.1](https://www.microsoft.com/net/learn/get-started/windows) 不提供支援。 [.NET Core 2.2](https://www.microsoft.com/net/download/dotnet-core/2.2) 支援此案例，但是尚未包含在 App Service 的預設映像中。 
>

您會了解如何：

> [!div class="checklist"]
> * 啟用受控身分識別
> * 將 SQL Database 存取權授與受控識別
> * 設定應用程式程式碼，以使用 Azure Active Directory 驗證向 SQL Database 進行驗證
> * 將 SQL Database 的最低權限授與受控識別

> [!NOTE]
> Azure Active Directory 驗證與內部部署 Active Directory (AD DS) 中的[整合式 Windows 驗證](/previous-versions/windows/it-pro/windows-server-2003/cc758557(v=ws.10))_不同_。 AD DS 和 Azure Active Directory 使用完全不同的驗證通訊協定。 如需詳細資訊，請參閱 [Windows Server AD DS 與 Azure AD 之間的差異](../active-directory/fundamentals/understand-azure-identity-solutions.md#the-difference-between-windows-server-ad-ds-and-azure-ad)。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

本文繼續進行您未完成的[教學課程：在 Azure 中搭配 SQL Database 來建置 ASP.NET 應用程式](app-service-web-tutorial-dotnet-sqldatabase.md)。 如果您還沒這麼做，請先依照該教學課程進行。 或者，您可以使用 SQL Database 針對自己的 ASP.NET 應用程式調整步驟。

<!-- ![app running in App Service](./media/app-service-web-tutorial-dotnetcore-sqldb/azure-app-in-browser.png) -->

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="enable-managed-identities"></a>啟用受控身分識別

若要啟用 Azure 應用程式的受控識別，請在 Cloud Shell 中使用 [az webapp identity assign](/cli/azure/webapp/identity?view=azure-cli-latest#az-webapp-identity-assign) 命令。 在下列命令中，取代 *\<app name>*。

```azurecli-interactive
az webapp identity assign --resource-group myResourceGroup --name <app name>
```

在 Azure Active Directory 中建立身分識別之後，輸出範例如下：

```json
{
  "additionalProperties": {},
  "principalId": "21dfa71c-9e6f-4d17-9e90-1d28801c9735",
  "tenantId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
  "type": "SystemAssigned"
}
```

您將在下一個步驟中使用 `principalId` 值。 如果您想要查看 Azure Active Directory 中新身分識別的詳細資料，請搭配 `principalId` 值執行下列選擇性命令：

```azurecli-interactive
az ad sp show --id <principalid>
```

## <a name="grant-database-access-to-identity"></a>將資料庫存取權授與身分識別

接下來，您將在 Cloud Shell 中使用 [`az sql server ad-admin create`](/cli/azure/sql/server/ad-admin?view=azure-cli-latest#az-sql-server-ad-admin_create) 命令，將資料庫存取權授與您應用程式的受控識別。 在下列命令中，取代 \<server_name> 和 <principalid_from_last_step>。 針對 \<admin_user> 輸入管理員名稱。

```azurecli-interactive
az sql server ad-admin create --resource-group myResourceGroup --server-name <server_name> --display-name <admin_user> --object-id <principalid_from_last_step>
```

受控識別現在可以存取 Azure SQL Database 伺服器。

## <a name="modify-connection-string"></a>修改連接字串

若要修改您先前為應用程式設定的連線，請在 Cloud Shell 中使用 [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 在下列命令中，以您的應用程式名稱取代 \<app name>，並以您的 SQL Database 資料取代 \<server_name> 和 \<db_name>。

```azurecli-interactive
az webapp config connection-string set --resource-group myResourceGroup --name <app name> --settings MyDbConnection='Server=tcp:<server_name>.database.windows.net,1433;Database=<db_name>;' --connection-string-type SQLAzure
```

## <a name="modify-aspnet-code"></a>修改 ASP.NET 程式碼

在 Visual Studio 的 **DotNetAppSqlDb** 專案中，開啟 packages.config 並在套件清單中新增下列一行。

```xml
<package id="Microsoft.Azure.Services.AppAuthentication" version="1.1.0-preview" targetFramework="net461" />
<package id="Microsoft.IdentityModel.Clients.ActiveDirectory" version="3.14.2" targetFramework="net461" />
```

開啟 Models\MyDatabaseContext.cs，然後將下列 `using` 陳述式新增至檔案頂端：

```csharp
using System.Data.SqlClient;
using Microsoft.Azure.Services.AppAuthentication;
using System.Web.Configuration;
```

在 `MyDatabaseContext` 類別中，新增下列建構函式：

```csharp
public MyDatabaseContext(SqlConnection conn) : base(conn, true)
{
    conn.ConnectionString = WebConfigurationManager.ConnectionStrings["MyDbConnection"].ConnectionString;
    // DataSource != LocalDB means app is running in Azure with the SQLDB connection string you configured
    if(conn.DataSource != "(localdb)\\MSSQLLocalDB")
        conn.AccessToken = (new AzureServiceTokenProvider()).GetAccessTokenAsync("https://database.windows.net/").Result;

    Database.SetInitializer<MyDatabaseContext>(null);
}
```

這個建構函式會設定自訂 SqlConnection 物件，以便從 App Service 使用 Azure SQL Database 的存取權杖。 有了存取權杖，您的 App Service 應用程式可利用其受控識別向 Azure SQL Database 進行驗證。 如需詳細資訊，請參閱[取得 zure 資源的權杖](app-service-managed-service-identity.md#obtaining-tokens-for-azure-resources)。 `if` 陳述式可讓您繼續使用 LocalDB 在本機測試應用程式。

> [!NOTE]
> 目前只有.NET Framework 4.6 和以上版本，以及 [.NET Core 2.2](https://www.microsoft.com/net/download/dotnet-core/2.2) 支援 `SqlConnection.AccessToken` ，而 [.NET Core 2.1](https://www.microsoft.com/net/learn/get-started/windows) 不提供支援。
>

若要使用這個新的建構函式，請開啟 `Controllers\TodosController.cs` 並尋找 `private MyDatabaseContext db = new MyDatabaseContext();` 一行。 現有程式碼會使用預設 `MyDatabaseContext` 控制器來建立使用標準連接字串的資料庫，該連接字串中包含[您進行變更](#modify-connection-string)前的使用者名稱和密碼 (純文字格式)。

將這一整行取代為下列程式碼：

```csharp
private MyDatabaseContext db = new MyDatabaseContext(new System.Data.SqlClient.SqlConnection());
```

### <a name="publish-your-changes"></a>發佈您的變更

現在只剩下將您的變更發佈至 Azure。

在 [方案總管] 中，以滑鼠右鍵按一下 [DotNetAppSqlDb] 專案，然後選取 [發佈]。

![從方案總管發佈](./media/app-service-web-tutorial-dotnet-sqldatabase/solution-explorer-publish.png)

在發佈頁面中，按一下 [發佈]。 當新的網頁顯示待辦事項清單時，表示應用程式正使用受控識別連線到資料庫。

![Code First 移轉之後的 Azure Web 應用程式](./media/app-service-web-tutorial-dotnet-sqldatabase/this-one-is-done.png)

您現在應該能夠如以往一樣編輯待辦事項清單。

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="grant-minimal-privileges-to-identity"></a>將最小權限授與身分識別

在先前步驟中，您可能會注意到受控識別會以 Azure AD 管理員身分連線到 SQL Server。 若要將最小權限授與受控識別，您需要以 Azure AD 管理員身分登入 Azure SQL Database 伺服器，然後新增包含受控識別的 Azure Active Directory 群組。 

### <a name="add-managed-identity-to-an-azure-active-directory-group"></a>將受控識別新增至 Azure Active Directory 群組

在 Cloud Shell 中，將您應用程式的受控識別新增到名為 myAzureSQLDBAccessGroup 的新 Azure Active Directory 群組，如下列指令碼所示：

```azurecli-interactive
groupid=$(az ad group create --display-name myAzureSQLDBAccessGroup --mail-nickname myAzureSQLDBAccessGroup --query objectId --output tsv)
msiobjectid=$(az webapp identity show --resource-group <group_name> --name <app_name> --query principalId --output tsv)
az ad group member add --group $groupid --member-id $msiobjectid
az ad group member list -g $groupid
```

如果您想要查看每個命令的完整 JSON 輸出，請拿掉 `--query objectId --output tsv` 參數。

### <a name="reconfigure-azure-ad-administrator"></a>重新設定 Azure AD 管理員

先前，您將受控識別指派為 SQL Database 的 Azure AD 管理員。 您無法使用這個身分識別進行互動式登入 (以新增資料庫使用者)，所以您必須使用真實的 Azure AD 使用者。 若要新增 Azure AD 使用者，請遵循[為 Azure SQL Database 伺服器佈建 Azure Active Directory 管理員](../sql-database/sql-database-aad-authentication-configure.md#provision-an-azure-active-directory-administrator-for-your-azure-sql-database-server)的步驟。 

### <a name="grant-permissions-to-azure-active-directory-group"></a>將權限授與 Azure Active Directory 群組

在 Cloud Shell 中，使用 SQLCMD 命令登入 SQL Database。 以您的 SQL Database 伺服器名稱取代 _\<server\_name>_，以您的應用程式使用的資料庫名稱取代 _\<db\_name>_，並以 Azure AD 使用者的認證取代 _\<AADuser\_name>_ 和 _\<AADpassword>_。

```azurecli-interactive
sqlcmd -S <server_name>.database.windows.net -d <db_name> -U <AADuser_name> -P "<AADpassword>" -G -l 30
```

在您所需資料庫的 SQL 提示字元中，執行下列命令來新增您稍早建立的 Azure Active Directory 群組，並授與您的應用程式所需的權限。 例如， 

```sql
CREATE USER [myAzureSQLDBAccessGroup] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [myAzureSQLDBAccessGroup];
ALTER ROLE db_datawriter ADD MEMBER [myAzureSQLDBAccessGroup];
ALTER ROLE db_ddladmin ADD MEMBER [myAzureSQLDBAccessGroup];
GO
```

輸入 `EXIT` 以返回 Cloud Shell 提示字元。 

## <a name="next-steps"></a>後續步驟

您已了解如何︰

> [!div class="checklist"]
> * 啟用受控身分識別
> * 將 SQL Database 存取權授與受控識別
> * 設定應用程式程式碼，以使用 Azure Active Directory 驗證向 SQL Database 進行驗證
> * 將 SQL Database 的最低權限授與受控識別

前往下一個教學課程，了解如何將自訂的 DNS 名稱對應至 Web 應用程式。

> [!div class="nextstepaction"]
> [將現有的自訂 DNS 名稱對應至 Azure Web Apps](app-service-web-tutorial-custom-domain.md)
