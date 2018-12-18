---
title: 在 Azure Stack 中搭配使用 API 版本設定檔與 .NET SDK | Microsoft Docs
description: 了解如何在 Azure Stack 中搭配使用 API 版本設定檔與 .NET。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/25/2018
ms.author: sethm
ms.reviewer: sijuman
ms.openlocfilehash: 35329468ee01d5b70d654c1eb4a908db9d3fcb5d
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47184398"
---
# <a name="use-api-version-profiles-with-net-in-azure-stack"></a>在 Azure Stack 中搭配使用 API 版本設定檔與 .NET

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

適用於 Azure Stack Resource Manager 的 .NET SDK 會提供工具來協助您建置及管理基礎結構。 SDK 中的資源提供者包括計算、網路、儲存體、應用程式服務和 [KeyVault](../../key-vault/key-vault-whatis.md)。 .NET SDK 包含 14 個會合併設定檔資訊的 NuGet 套件，且每次都必須下載到專案解決方案中。 不過，您可以具體下載要用於 2018-03-01-hybrid 或 2017-03-09-profile 的資源提供者，以將應用程式的記憶體最佳化。 每個套件都包含資源提供者、個別的 API 版本和其所屬的 API 設定檔。 .NET SDK 中的 API 設定檔透過協助您在全域 Azure 資源和 Azure Stack 的資源來回切換，提供混合式雲端開發功能。

## <a name="net-and-api-version-profiles"></a>.NET 和 API 版本設定檔

API 設定檔是資源提供者和 API 版本的組合。 您可以使用 API 設定檔，取得資源提供者套件中每個資源類型的最新、最穩定版本。

-   若要利用所有服務的最新版本，請使用套件的**最新**設定檔。 此設定檔是 **Microsoft.Azure.Management** NuGet 套件的一部分。

-   若要使用與 Azure Stack 相容的服務，請使用 **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.ResourceProvider.0.9.0-preview.nupkg** 或 **Microsoft.Azure.Management.Profiles.hybrid\_2017\_03\_09.ResourceProvider.0.9.0-preview.nupkg** 套件。

    -   每個設定檔的每個資源提供者各有兩個套件。

    -   請確保上述 NuGet 套件的 **ResourceProvider** 部分會變更為正確的提供者。

-   若要使用服務的最新 API 版本，請使用特定 NuGet 套件的**最新**設定檔。 例如，如果您想要單獨使用計算服務的**最新 API** 版本，請使用**計算**套件的**最新**設定檔。 **最新**設定檔是 **Microsoft.Azure.Management** NuGet 套件的一部分。

-   若要使用特定資源提供者中某個資源類型的特定 API 版本，請使用套件內所定義的特定 API 版本。

請注意，您可以組合相同應用程式中的所有選項。

## <a name="install-the-azure-net-sdk"></a>安裝 Azure .NET SDK

1.  安裝 GIT。 如需相關指示，請參閱[使用者入門 - 安裝 Git][]。

2.  若要安裝正確的 NuGet 套件，請參閱[尋找和安裝套件][]。

3.  需要安裝的套件取決於您想要使用的設定檔版本。 設定檔版本的套件名稱是：

    1.  **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.ResourceProvider.0.9.0-preview.nupkg**

    2.  **Microsoft.Azure.Management.Profiles.hybrid\_2017\_03\_09.ResourceProvider.0.9.0-preview.nupkg**

4.  若要安裝適用於 Visual Studio Code 的正確 NuGet 套件，請參閱下列連結來下載 [NuGet 套件管理員指示][]。

5.  如果無法使用，請建立訂用帳戶，並儲存訂用帳戶識別碼供後續使用。 如需訂用帳戶的建立指示，請參閱[在 Azure Stack 中建立供應項目的訂用帳戶][]。

6.  建立服務主體，並儲存用戶端識別碼和用戶端密碼。 如需如何建立 Azure Stack 服務主體的指示，請參閱[為 Azure Stack 提供應用程式存取][]。 請注意，在建立服務主體時，用戶端識別碼也稱為應用程式識別碼。

7.  確保服務主體在訂用帳戶中擁有參與者/擁有者角色。 如需如何對服務主體指派角色的指示，請參閱[為 Azure Stack 提供應用程式存取][]。

## <a name="prerequisites"></a>必要條件

若要搭配使用 .NET Azure SDK 與 Azure Stack，您必須提供下列值，然後以環境變數設定值。 若要設定環境變數，請參閱資料表後面適用於作業系統的指示。

| 值                     | 環境變數   | 說明                                                                                                             |
|---------------------------|-------------------------|-------------------------------------------------------------------------------------------------------------------------|
| 租用戶識別碼                 | AZURE_TENANT_ID       | Azure Stack [*租用戶識別碼*][]的值。                                                                          |
| 用戶端識別碼                 | AZURE_CLIENT_ID       | 在本文上一節建立服務主體時所儲存的服務主體應用程式識別碼。 |
| 訂用帳戶識別碼           | AZURE_SUBSCRIPTION_ID | [*訂用帳戶識別碼*][]可供您存取 Azure Stack 中的供應項目。                                                      |
| 用戶端密碼             | AZURE_CLIENT_SECRET   | 建立服務主體時所儲存的服務主體應用程式祕密。                                      |
| Resource Manager 端點 | ARM_ENDPOINT           | 請參閱 [*Azure Stack Resource Manager 端點*][]。                                                                    |

若要尋找 Azure Stack 的租用戶識別碼，請遵循[此處](../azure-stack-csp-ref-operations.md)所提供的指示。 若要設定環境變數，請執行下列作業：

### <a name="microsoft-windows"></a>Microsoft Windows

若要在 Windows 命令提示字元中設定環境變數，請使用下列格式：

```shell
Set Azure_Tenant_ID=Your_Tenant_ID
```

### <a name="macos-linux-and-unix-based-systems"></a>MacOS、Linux 和以 Unix 為基礎的系統

在以 Unix 為基礎的系統中，您可以使用下列命令：

```shell
Export Azure_Tenant_ID=Your_Tenant_ID
```

### <a name="the-azure-stack-resource-manager-endpoint"></a>Azure Stack Resource Manager 端點

Microsoft Azure Resource Manager 是可讓系統管理員進行部署、管理及監視 Azure 資源的管理架構。 Azure Resource Manager 能夠以群組方式處理這些工作，而非個別單獨作業的方式。

您可以從 Resource Manager 端點取得中繼資料資訊。 此端點會傳回 JSON 檔案，該檔案中有執行您程式碼所需的資訊。

請注意下列考量：

- Azure Stack 開發套件 (ASDK) 中的 **ResourceManagerUrl** 為： https://management.local.azurestack.external/

- 整合系統中的 **ResourceManagerUrl** 為：`https://management.<location>.ext-<machine-name>.masd.stbtest.microsoft.com/`。若要擷取所需的中繼資料：`<ResourceManagerUrl>/metadata/endpoints?api-version=1.0`

範例 JSON 檔案：

```json
{ 
   "galleryEndpoint": "https://portal.local.azurestack.external:30015/",
   "graphEndpoint": "https://graph.windows.net/",
   "portal Endpoint": "https://portal.local.azurestack.external/",
   "authentication": 
      {
      "loginEndpoint": "https://login.windows.net/",
      "audiences": ["https://management.yourtenant.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
      }
}
```

## <a name="existing-api-profiles"></a>現有的 API 設定檔

1.  **Microsoft.Azure.Management.Profiles.hybrid\_2018\_03\_01.ResourceProvider.0.9.0-preview.nupkg**：針對 Azure Stack 所建置的最新設定檔。 只要您是在 1808 以上的戳記上，使用此設定檔就能讓服務與 Azure Stack 有最高的相容性。

2.  **Microsoft.Azure.Management.Profiles.hybrid\_2017\_03\_09.ResourceProvider.0.9.0-preview.nupkg**：如果您所在的戳記低於 1808 組建，請使用此設定檔。

3.  **最新**：包含所有服務最新版本的設定檔。 使用所有服務的最新版本。 此設定檔是 **Microsoft.Azure.Management** NuGet 套件的一部分。

如需有關 Azure Stack 和 API 設定檔的詳細資訊，請參閱 [API 設定檔的摘要][]。

## <a name="azure-net-sdk-api-profile-usage"></a>Azure .NET SDK API 設定檔使用方式

請使用下列程式碼將設定檔用戶端具現化。 只有 Azure Stack 或其他私人雲端才需要此參數。 全域 Azure 預設已經有這些設定。

需要有下列程式碼才能在 Azure Stack 上驗證服務主體。 其會依據租用戶識別碼和驗證基底 (專屬於 Azure Stack) 來建立權杖。

```csharp
public class CustomLoginCredentials : ServiceClientCredentials
{
    private string clientId;
    private string clientSecret;
    private string resourceId;
    private string tenantId;

    private const string authenticationBase = "https://login.windows.net/{0}";

    public CustomLoginCredentials(string servicePrincipalId, string servicePrincipalSecret, string azureEnvironmentResourceId, string azureEnvironmentTenandId)
    {
        clientId = servicePrincipalId;
        clientSecret = servicePrincipalSecret;
        resourceId = azureEnvironmentResourceId;
        tenantId = azureEnvironmentTenandId;
    }
```

這可讓您使用 API 設定檔 NuGet 套件，將應用程式成功地部署至 Azure Stack。

## <a name="define-azure-stack-environment-setting-functions"></a>定義 Azure Stack 環境設定功能

若要向 Azure Stack 環境驗證服務主體，請使用下列程式碼：

```csharp
private string AuthenticationToken { get; set; }
public override void InitializeServiceClient<T>(ServiceClient<T> client)
{
    var authenticationContext = new AuthenticationContext(String.Format(authenticationBase, tenantId));
    var credential = new ClientCredential(clientId, clientSecret);
    var result = authenticationContext.AcquireTokenAsync(resource: resourceId,
    clientCredential: credential).Result;
    if (result == null)
    {
        throw new InvalidOperationException("Failed to obtain the JWT token");
    }
    AuthenticationToken = result.AccessToken;
}
```

這會覆寫向 Azure Stack 驗證的初始化服務用戶端。

## <a name="samples-using-api-profiles"></a>使用 API 設定檔的範例

您可以使用 GitHub 存放庫中所提供的下列範例，作為使用 .NET 和 Azure Stack API 設定檔來建立解決方案的參考。

-   [虛擬機器的測試專案、vNet、資源群組和儲存體帳戶][]
-   使用 .NET 管理虛擬機器

### <a name="sample-unit-test-project"></a>單元測試專案範例 

1.  使用下列命令來複製存放庫：

    `git clone <https://github.com/seyadava/azure-sdk-for-net-samples/tree/master/TestProject>`

2.  建立 Azure 服務主體並指派用以存取訂用帳戶的角色。 如需建立服務主體的指示，請參閱[使用 Azure PowerShell 建立具有憑證的服務主體][]。

3.  擷取下列必要值：

    1.  租用戶識別碼
    2.  用戶端識別碼
    3.  用戶端密碼
    4.  訂用帳戶識別碼
    5.  Resource Manager 端點

4.  在使用命令提示字元所建立的服務主體中，以從中擷取的資訊設定下列環境變數：

    1.  export AZURE_TENANT_ID={您的租用戶識別碼}
    2.  export AZURE_CLIENT_ID={您的用戶端識別碼}
    3.  export AZURE_CLIENT_SECRET={您的用戶端密碼}
    4.  export AZURE_SUBSCRIPTION_ID = {您的訂用帳戶識別碼}
    5.  export ARM_ENDPOINT={your Azure Stack Resource manager URL}

   在 Windows 上，請使用 **set** 而非 **export**。

5.  確定位置變數已設定為 AzureStack 的位置。 例如，LOCAL = "local"。

6.  設定可讓您向 Azure Stack 驗證的自訂登入認證。 請注意，這部分的程式碼會包含在此範例的 [授權] 資料夾中。

   ```csharp
   public class CustomLoginCredentials : ServiceClientCredentials
   {
       private string clientId;
       private string clientSecret;
       private string resourceId;
       private string tenantId;
       private const string authenticationBase = "https://login.windows.net/{0}";
       public CustomLoginCredentials(string servicePrincipalId, string servicePrincipalSecret, string azureEnvironmentResourceId, string azureEnvironmentTenandId)
       {
           clientId = servicePrincipalId;
           clientSecret = servicePrincipalSecret;
           resourceId = azureEnvironmentResourceId;
           tenantId = azureEnvironmentTenandId;
       }
   private string AuthenticationToken { get; set; }
   ```

7.  如果您要使用 Azure Stack 覆寫向 Azure Stack 驗證的初始化服務用戶端，請新增下列程式碼。 請注意，部分程式碼已包含在此範例的 [授權] 資料夾中。

   ```csharp
   public override void InitializeServiceClient<T>(ServiceClient<T> client)
   {
      var authenticationContext = new AuthenticationContext(String.Format(authenticationBase, tenantId));
      var credential = new ClientCredential(clientId, clientSecret);
      var result = authenticationContext.AcquireTokenAsync(resource: resourceId,
                clientCredential: credential).Result;
      if (result == null)
      {
          throw new InvalidOperationException("Failed to obtain the JWT token");
      }
      AuthenticationToken = result.AccessToken;
   }
   ```
 
8.  使用 NuGet 套件管理員、搜尋 "2018-03-01-hybrid"，然後安裝與此設定檔相關聯的計算、網路、儲存體、KeyVault 和 App Services 資源提供者適用套件。

2.  在 .cs 檔案的每個工作中，設定要使用 Azure Stack 所需的參數。 工作 `CreateResourceGroupTest` 的範例如下所示：

   ```csharp
   var location = Environment.GetEnvironmentVariable("AZURE_LOCATION");
   var baseUriString = Environment.GetEnvironmentVariable("AZURE_BASE_URL");
   var resourceGroupName = Environment.GetEnvironmentVariable("AZURE_RESOURCEGROUP");
   var servicePrincipalId = Environment.GetEnvironmentVariable("AZURE_CLIENT_ID");
   var servicePrincipalSecret = Environment.GetEnvironmentVariable("AZURE_CLIENT_SECRET");
   var azureResourceId = Environment.GetEnvironmentVariable("AZURE_RESOURCE_ID");
   var tenantId = Environment.GetEnvironmentVariable("AZURE_TENANT_ID");
   var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
   var credentials = new CustomLoginCredentials(servicePrincipalId, servicePrincipalSecret, azureResourceId, tenantId);
   ```

1.  以滑鼠右鍵按一下每個工作，然後選取 [執行測試]。

    1.  側邊窗格視窗上的綠色核取記號會提醒您，系統已成功地根據指定參數建立每個工作。 請檢查 Azure Stack 訂用帳戶，以確定資源已成功建立。

    2.  如需如何執行單元測試的詳細資訊，請參閱[使用測試總管執行單元測試。][]

## <a name="next-steps"></a>後續步驟

如需 API 設定檔的詳細資訊，請參閱：

- [在 Azure Stack 中管理 API 版本設定檔](azure-stack-version-profiles.md)
- [設定檔所支援的資源提供者 API 版本](azure-stack-profiles-azure-resource-manager-versions.md)

  [使用者入門 - 安裝 Git]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
  [尋找和安裝套件]: /nuget/tools/package-manager-ui
  [NuGet 套件管理員指示]: https://marketplace.visualstudio.com/items?itemName=jmrog.vscode-nuget-package-manager
  [在 Azure Stack 中建立供應項目的訂用帳戶]: ../azure-stack-subscribe-plan-provision-vm.md
  [為 Azure Stack 提供應用程式存取]: ../azure-stack-create-service-principals.md
  [租用戶識別碼]: ../azure-stack-identity-overview.md
  [訂用帳戶識別碼]: ../azure-stack-plan-offer-quota-overview.md#subscriptions
  [Azure Stack Resource Manager 端點]: ../user/azure-stack-version-profiles-ruby.md#the-azure-stack-resource-manager-endpoint
  [API 設定檔的摘要]: ../user/azure-stack-version-profiles.md#summary-of-api-profiles
  [虛擬機器的測試專案、vNet、資源群組和儲存體帳戶]: https://github.com/seyadava/azure-sdk-for-net-samples/tree/master/TestProject
  [使用 Azure PowerShell 建立具有憑證的服務主體]: ../azure-stack-create-service-principals.md
  [使用測試總管執行單元測試。]: /visualstudio/test/run-unit-tests-with-test-explorer?view=vs-2017
