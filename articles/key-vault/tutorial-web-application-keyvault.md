---
title: 設定 Azure Web 應用程式以從 Key Vault 讀取密碼的教學課程 | Microsoft Docs
description: 教學課程：設定 ASP.Net Core 應用程式，以從 Key Vault 讀取祕密
services: key-vault
documentationcenter: ''
author: barclayn
manager: mbaldwin
ms.assetid: 0e57f5c7-6f5a-46b7-a18a-043da8ca0d83
ms.service: key-vault
ms.workload: identity
ms.topic: tutorial
ms.date: 09/05/2018
ms.author: barclayn
ms.custom: mvc
ms.openlocfilehash: 521b6423550bf3e2d0bc90212b7e3fe0cbeddfc4
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49167064"
---
# <a name="tutorial-configure-an-azure-web-application-to-read-a-secret-from-key-vault"></a>教學課程：設定 Azure 網路應用程式，以從 Key Vault 讀取祕密

在本教學課程中，我們將引導您完成讓 Azure Web 應用程式使用 Azure 資源受控識別從 Key Vault 讀取資訊所需的步驟。 您會了解如何：

> [!div class="checklist"]
> * 建立 Key Vault。
> * 將密碼儲存在 Key Vault 中。
> * 建立 Azure Web 應用程式。
> * 啟用 Web 應用程式的受控識別。
> * 授與應用程式從 Key Vault 讀取資料所需的權限。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

若要使用 CLI 登入 Azure，您可以輸入：

```azurecli
az login
```

## <a name="create-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

下列範例會在 eastus 位置建立名為 ContosoResourceGroup 的資源群組。

```azurecli
# To list locations: az account list-locations --output table
az group create --name "ContosoResourceGroup" --location "East US"
```

本教學課程中會使用您該建立的資源群組。

## <a name="create-an-azure-key-vault"></a>建立 Azure Key Vault

接下來，您會在上一個步驟中建立的資源群組中建立 Key Vault。 雖然 “ContosoKeyVault” 作為本教學課程中的 Key Vault 名稱，但您必須使用唯一的名稱。 請提供下列資訊：

* 保存庫名稱 **ContosoKeyVault**。
* 資源群組名稱 **ContosoResourceGroup**。
* 位置為 [美國東部]。

```azurecli
az keyvault create --name "ContosoKeyVault" --resource-group "ContosoResourceGroup" --location "East US"
```

此命令的輸出會顯示新建立 Key Vault 的屬性。 請記下下列兩個屬性：

* **保存庫名稱**：在此範例中是 **ContosoKeyVault**。 您會將您的 Key Vault 名稱使用於所有 Key Vault 命令。
* **保存庫 URI**：在此範例中是 https://<YourKeyVaultName>.vault.azure.net/。 透過其 REST API 使用保存庫的應用程式必須使用此 URI。

>[!IMPORTANT]
> 如果您收到以下錯誤：'vault_name' 參數必須符合下列樣式：'^[a-zA-Z0-9-]{3,24}$'。-name 參數值不是唯一的，或者並未符合由英數字元 (長度 3 到 24 個字元) 組成的字串。

此時，您的 Azure 帳戶是唯一獲得授權在此新保存庫上執行任何作業的帳戶。

## <a name="add-a-secret-to-key-vault"></a>將密碼新增至 Key Vault

我們將新增密碼，以協助說明其運作方式。 您可能正在儲存 SQL 連接字串，或任何您必須安全保存但可供您的應用程式使用的其他資訊。 在本教學課程中，密碼會稱為 **AppSecret**，並將在其中儲存 **MySecret** 的值。

輸入下列命令，以在 Key Vault 中建立稱為 **AppSecret** 並將儲存 **MySecret** 值的祕密：

```azurecli
az keyvault secret set --vault-name "ContosoKeyVault" --name "AppSecret" --value "MySecret"
```

若要以純文字檢視包含在祕密中的值：

```azurecli
az keyvault secret show --name "AppSecret" --vault-name "ContosoKeyVault"
```

此命令會顯示祕密資訊，包括 URI。 完成這些步驟之後，您的 Azure Key Vault 中應該會有密碼的 URI。 請記下此資訊。 您在稍後的步驟中需要此資訊。

## <a name="create-a-web-app"></a>建立 Web 應用程式

在這一節中，您會在 Azure 中建立 ASP.NET MVC 應用程式並將它部署為 Web 應用程式。 如需有關 Azure Web Apps 的詳細資訊，請參閱 [Web Apps 概觀](../app-service/app-service-web-overview.md)。

1. 在 Visual Studio 中，選取 [檔案] > [新增] > [專案] 以建立專案。 

2. 在 [新增專案] 對話方塊中，選取 [Visual C#] > [Web] > [ASP.NET Core Web 應用程式]。

3. 將應用程式命名為 **WebKeyVault**，然後選取 [確定]。
   >[!IMPORTANT]
   > 您必須將應用程式命名為 WebKeyVault，如此您複製並貼上的程式碼才會符合命名空間。 如果您將網站命名為任何您所需的名稱，以修改程式碼來符合網站名稱。

    ![[新增 ASP.NET 專案] 對話方塊](media/tutorial-web-application-keyvault/aspnet-dialog.png)

4. 您可以將任何類型的 ASP.NET Core Web 應用程式部署至 Azure。 在本教學課程中，選取 [Web 應用程式] 範本，並確定驗證設定為 [不需要驗證]。

    ![ASPNET 無驗證對話方塊](media/tutorial-web-application-keyvault/aspnet-noauth.png)

5. 選取 [確定] 。

6. ASP.NET Core 專案建立之後，ASP.NET Core 的歡迎頁面會隨即顯示，並提供許多可協助您開始使用的資源連結。

7. 從功能表中，選取 [偵錯] > [啟動但不偵錯]，以在本機執行 Web 應用程式。

## <a name="modify-the-web-app"></a>修改 Web 應用程式

您的 Web 應用程式有兩個必須安裝的 NuGet 套件。 若要加以安裝，請依照下列步驟操作：

1. 在 [方案總管] 中，於您的網站名稱上按一下滑鼠右鍵。
2. 選取 [管理解決方案的 NuGet 套件...]
3. 選取搜尋方塊旁的核取方塊。 **包含搶鮮版**
4. 搜尋下列三個 NuGet 套件，並接受將它們新增至您的解決方案：

    * [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) - 讓您輕鬆擷取 Service-to-Azure-Service 驗證案例的存取權杖。 
    * [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault) - 包含與 Key Vault 互動的方法。
    * [Microsoft.Extensions.Configuration.AzureKeyVault](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureKeyVault/) - 包含 Azure Key Vault 的 `IConfiguration` 擴充功能

5. 使用 [方案總管] 來開啟 `Program.cs`，並以下列程式碼取代 Program.cs 檔案的內容。 將 ```<YourKeyVaultName>``` 替換成您的金鑰保存庫名稱：

    ```csharp
    
    using Microsoft.AspNetCore;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Azure.KeyVault;
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureKeyVault;
    
    namespace WebKeyVault
    {
       public class Program
       {
           public static void Main(string[] args)
           {
               BuildWebHost(args).Run();
           }

           public static IWebHost BuildWebHost(string[] args) =>
           WebHost.CreateDefaultBuilder(args)
               .ConfigureAppConfiguration((ctx, builder) =>
               {
                   var keyVaultEndpoint = GetKeyVaultEndpoint();
                   if (!string.IsNullOrEmpty(keyVaultEndpoint))
                   {
                       var azureServiceTokenProvider = new AzureServiceTokenProvider();
                       var keyVaultClient = new KeyVaultClient(
                           new KeyVaultClient.AuthenticationCallback(
                               azureServiceTokenProvider.KeyVaultTokenCallback));
                       builder.AddAzureKeyVault(
                           keyVaultEndpoint, keyVaultClient, new DefaultKeyVaultSecretManager());
                   }
               }
            ).UseStartup<Startup>()
             .Build();

           private static string GetKeyVaultEndpoint() => "https://<YourKeyVaultName>.vault.azure.net";
         }
    }
    ```

6. 使用 [方案總管] 巡覽至 **Pages** 區段並開啟 `About.cshtml`。 以下列程式碼取代 **About.cshtml.cs** 的內容：

    ```csharp
    
    using Microsoft.AspNetCore.Mvc.RazorPages;
    using Microsoft.Extensions.Configuration;
    
    namespace WebKeyVault.Pages
    {
        public class AboutModel : PageModel
        {
            public AboutModel(IConfiguration configuration)
            {
                _configuration = configuration;
            }
    
            private readonly IConfiguration _configuration = null;
            public string Message { get; set; }
    
            public void OnGet()
            {
                Message = "My key val = " +  _configuration["AppSecret"];
            }
        }
    }
    
    ```

7. 從主功能表中選擇 [偵錯] > [啟動但不偵錯]。 當瀏覽器出現時，巡覽至 [關於] 頁面。 AppSecret 的值隨即顯示。

>[!IMPORTANT]
> 如果您收到 HTTP 錯誤「502.5 - 處理失敗」訊息
> > 請確認在 `Program.cs` 中指定的 Key Vault 名稱

## <a name="publish-the-web-application-to-azure"></a>將 Web 應用程式發佈至 Azure

1. 在編輯器上方，選取 **WebKeyVault**。
2. 選取 [發佈]，然後選取 [開始]。
3. 建立新的 **App Service**，然後選取 [發佈]。
4. 選取 [建立] 。

>[!IMPORTANT]
> 瀏覽器視窗隨即開啟，您將會看見「502.5 - 處理失敗」訊息。 這是預期行為。 您必須授與應用程式識別從 Key Vault 讀取祕密的權限。

## <a name="enable-a-managed-identity-for-the-web-app"></a>啟用 Web 應用程式的受控識別

Azure Key Vault 可安全地儲存認證和其他金鑰及密碼，但是您的程式碼必須向 Key Vault 進行驗證，才可取得這些項目。 [Azure 資源受控識別概觀](../active-directory/managed-identities-azure-resources/overview.md)可在 Azure Active Directory (Azure AD) 中將受控識別自動提供給 Azure 服務，而降低解決此問題的難度。 您可以使用此身分識別來完成任何支援 Azure AD 驗證的服務驗證 (包括 Key Vault)，不需要任何您程式碼中的認證。

1. 返回 Azure CLI
2. 執行 assign-identity 命令來建立此應用程式的識別：

```azurecli
az webapp identity assign --name "WebKeyVault" --resource-group "ContosoResourcegroup"
```

>[!NOTE]
>此命令等同於前往入口網站，並在 Web 應用程式屬性中將 [身分識別/系統指派] 設定切換為 [開啟]。

## <a name="grant-rights-to-the-application-identity"></a>將權限授與應用程式識別

使用 Azure 入口網站，移至 Key Vault 的存取原則，並授與您本身對 Key Vault 的祕密管理存取權。 這可讓您在本機開發電腦上執行此應用程式。

1. 在 Azure 入口網站的 [搜尋資源] 對話方塊中搜尋您的 Key Vault。
2. 選取 [存取原則]。
3. 選取 [新增]，在 [祕密權限] 區段中選取 [取得] 和 [列出]。
4. 選取 [選取主體]，然後新增應用程式識別。 它將會與應用程式同名。
5. 選擇 [確定]。

您在 Azure 中帳戶和應用程式識別現在具有從 Key Vault 讀取資訊的權限。 在重新整理頁面時，您應該會看到網站的登陸頁面。 如果您選取 [關於]，則會看到儲存在 Key Vault 中的值。

## <a name="clean-up-resources"></a>清除資源

若要刪除資源群組和其所有資源，請使用 **az group delete** 命令。

  ```azurecli
  az group delete -n "ContosoResourceGroup"
  ```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Azure Key Vault 開發人員指南](key-vault-developers-guide.md)
