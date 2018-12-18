---
title: 使用 .NET 進行 Azure Key Vault 的服務對服務驗證
description: 使用 Microsoft.Azure.Services.AppAuthentication 程式庫以利用 .NET 向 Azure Key Vault 進行驗證。
keywords: Azure Key Vault 驗證的本機認證
author: bryanla
manager: mbaldwin
services: key-vault
ms.author: bryanla
ms.date: 09/05/2018
ms.topic: conceptual
ms.prod: ''
ms.service: key-vault
ms.technology: ''
ms.assetid: 4be434c4-0c99-4800-b775-c9713c973ee9
ms.openlocfilehash: d9fc845316d6e785d8215ac738b893ebc080d911
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44300965"
---
# <a name="service-to-service-authentication-to-azure-key-vault-using-net"></a>使用 .NET 進行 Azure Key Vault 的服務對服務驗證

若要向 Azure Key Vault 進行驗證，您需要 Azure Active Directory (AD) 認證 (共用密碼或憑證)。 管理這類認證並不簡單，而且我們很容易會將認證包含在來源或設定檔中，藉此與應用程式結合在一起。

.NET 程式庫的 `Microsoft.Azure.Services.AppAuthentication` 可簡化此問題。 它會使用開發人員的認證，在本機開發期間進行驗證。 當解決方案在稍後部署至 Azure 時，程式庫會自動切換至應用程式認證。  

在本機開發期間使用開發人員認證會比較安全，因為您不需要建立 Azure AD 認證，或在開發人員之間共用認證。

`Microsoft.Azure.Services.AppAuthentication` 程式庫會自動管理驗證，這可讓您將焦點放在您的方案，而不是認證上。

`Microsoft.Azure.Services.AppAuthentication` 程式庫支援使用 Microsoft Visual Studio、Azure CLI 或 Azure AD 整合式驗證的本機開發。 部署到 Azure App Services 或 Azure 虛擬機器 (VM) 時，程式庫會自動使用 [Azure 服務的受控識別](/azure/active-directory/msi-overview)。 不需要任何程式碼或設定變更。 當受控識別無法使用，或在本機開發期間無法判斷開發人員的安全性內容時，程式庫也支援直接使用 Azure AD [用戶端認證](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authenticate-service-principal)。

<a name="asal"></a>
## <a name="using-the-library"></a>使用程式庫

對於 .NET 應用程式，使用受控識別最簡單的方式是透過 `Microsoft.Azure.Services.AppAuthentication` 套件。 如何開始使用：

1. 將 [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) NuGet 套件的參考新增至應用程式。

2. 新增下列程式碼：

    ``` csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;

    // ...

    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(
    azureServiceTokenProvider.KeyVaultTokenCallback));

    // or

    var azureServiceTokenProvider = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync(
       "https://management.azure.com/").ConfigureAwait(false);
    ```

`AzureServiceTokenProvider` 類別會快取記憶體中的權杖，並在到期之前從 Azure AD 擷取權杖。 因此，您不再需要先檢查到期日，再呼叫 `GetAccessTokenAsync` 方法。 當您想要使用權杖時，可直接呼叫方法。 

`GetAccessTokenAsync` 方法需要資源識別碼。 若要深入了解，請參閱[哪些 Azure 服務支援 Azure 資源的受控識別](https://docs.microsoft.com/azure/active-directory/msi-overview#which-azure-services-support-managed-service-identity)。


<a name="samples"></a>
## <a name="samples"></a>範例

下列範例會顯示作用中的 `Microsoft.Azure.Services.AppAuthentication` 程式庫：

1. [在執行階段使用受控識別從 Azure Key Vault 擷取秘密](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet)

2. [使用受控識別以程式設計方式從 Azure VM 部署 Azure Resource Manager 範本](https://github.com/Azure-Samples/windowsvm-msi-arm-dotnet)。

3. [使用 .NET Core 範例和受控識別從 Azure Linux VM 呼叫 Azure 服務](https://github.com/Azure-Samples/linuxvm-msi-keyvault-arm-dotnet/)。


<a name="local"></a>
## <a name="local-development-authentication"></a>本機開發驗證

針對本機開發，以下是兩個主要驗證案例：

- [向 Azure 服務驗證](#authenticating-to-azure-services)
- [向自訂服務驗證](#authenticating-to-custom-services)

你可以從這裡了解每個案例和支援工具的需求。


### <a name="authenticating-to-azure-services"></a>向 Azure 服務驗證

本機電腦不支援 Azure 資源的受控識別。  因此，`Microsoft.Azure.Services.AppAuthentication` 程式庫會使用您的開發人員認證，以在本機開發環境中執行。 當解決方案部署至 Azure 時，程式庫會使用受控識別切換至 OAuth 2.0 用戶端的認證授與流程。  這表示您可以放心地在本機或遠端測試相同程式碼。

針對本機開發，`AzureServiceTokenProvider` 會使用 **Visual Studio**、**Azure 命令列介面** (CLI)，或 **Azure AD 整合式驗證**來擷取權杖。 會依序嘗試每個選項，而程式庫會使用第一個成功的選項。 如果沒有選項可用，`AzureServiceTokenProviderException` 會擲回包含詳細資訊的例外狀況。

### <a name="authenticating-with-visual-studio"></a>使用 Visual Studio 進行驗證

若要使用 Visual Studio，請確認下列事項：

1. 您已安裝 [Visual Studio 2017 v15.5](https://blogs.msdn.microsoft.com/visualstudio/2017/10/11/visual-studio-2017-version-15-5-preview/) 或更新版本。

2. 已安裝 [Visual Studio 的應用程式驗證擴充功能](https://go.microsoft.com/fwlink/?linkid=862354)。
 
3. 您已登入 Visual Studio，並已選取要用於本機開發的帳戶。 使用 [工具]&nbsp;>&nbsp;[選項]&nbsp;>&nbsp;[Azure 服務驗證] 來選擇本機開發帳戶。 

如果您使用 Visual Studio 時遇到問題，例如發生有關權杖提供者檔案的錯誤，請仔細檢閱這些步驟。 

此時也可能需要重新驗證您的開發人員權杖。  若要這樣做，請移至 [工具]&nbsp;>&nbsp;[選項]>[Azure&nbsp;服務驗證]，然後在選取的帳戶下尋找 [重新驗證] 連結。&nbsp;  選取該項目並進行驗證。 

### <a name="authenticating-with-azure-cli"></a>使用 Azure CLI 進行驗證

若要將 Azure CLI 用於本機開發：

1. 安裝 [Azure CLI v2.0.12](/cli/azure/install-azure-cli) 或更新版本。 升級較早的版本。 

2. 使用 **az login** 登入 Azure。

使用 `az account get-access-token` 驗證存取權。  如果您收到錯誤訊息，請確認您已成功完成步驟 1。 

如果 Azure CLI 未安裝至預設目錄中，您可能會收到一則錯誤訊息，指出 `AzureServiceTokenProvider` 找不到 Azure CLI 的路徑。  使用 **AzureCLIPath** 環境變數，可定義 Azure CLI 的安裝資料夾。 必要時，`AzureServiceTokenProvider` 會將 **AzureCLIPath** 環境變數中指定的目錄新增至**路徑**環境變數。

如果您使用多個帳戶登入 Azure CLI，或您的帳戶具有多個訂用帳戶的存取權，則需要指定要使用的特定訂用帳戶。  若要這樣做，請使用：

```
az account set --subscription <subscription-id>
```

此命令只會在失敗時產生輸出。  若要確認目前的帳戶設定，請使用：

```
az account list
```

### <a name="authenticating-with-azure-ad-integrate-authentication"></a>使用 Azure AD 整合驗證進行驗證

若要使用 Azure AD 驗證，請確認下列事項：

- 您的內部部署 Active Directory 會[同步至 Azure AD](/azure/active-directory/connect/active-directory-aadconnect)。

- 您的程式碼正在已加入網域的電腦上執行。


### <a name="authenticating-to-custom-services"></a>向自訂服務驗證

當服務呼叫 Azure 服務時，上述步驟即會開始作用，因為 Azure 服務允許存取使用者和應用程式。  

建立會呼叫自訂服務的服務時，請在本機開發驗證上使用 Azure AD 用戶端認證。  有兩個選項： 

1.  使用服務主體登入 Azure：

    1.  [建立服務主體](/cli/azure/create-an-azure-service-principal-azure-cli)。

    2.  使用 Azure CLI 登入：

        ```
        az login --service-principal -u <principal-id> --password <password>
           --tenant <tenant-id> --allow-no-subscriptions
        ```

        因為服務主體可能無法存取訂用帳戶，所以使用 `--allow-no-subscriptions` 引數。

2.  使用環境變數來指定服務主體詳細資料。  如需詳細資訊，請參閱[使用服務主體執行應用程式](#running-the-application-using-a-service-principal)。

當您登入 Azure 之後，`AzureServiceTokenProvider` 會使用服務主體來擷取用於本機開發的權杖。

這只適用於本機開發。 當您將解決方案部署至 Azure 時，程式庫會切換為以受控識別進行驗證。

<a name="msi"></a>
## <a name="running-the-application-using-managed-identity"></a>使用受控識別執行應用程式 

當您在 Azure App Service 或啟用受控識別的 Azure VM 上執行程式碼時，程式庫會自動使用受控識別。 不需要變更程式碼。 


<a name="sp"></a>
## <a name="running-the-application-using-a-service-principal"></a>使用服務主體執行應用程式 

這可能需要建立 Azure AD 用戶端認證，才可進行驗證。 常見範例包括：

1. 您的程式碼在本機開發環境中執行，但不是在開發人員的身分識別下執行。  例如，Service Fabric 在本機開發上使用 [NetworkService 帳戶](/azure/service-fabric/service-fabric-application-secret-management)。
 
2. 您的程式碼在本機開發環境中執行，而且向自訂服務進行驗證，因此您無法使用您的開發人員身分識別。 
 
3. 您的程式碼會在對 Azure 資源的受控識別尚不支援的 Azure 計算資源上執行，例如 Azure Batch。

若要使用憑證登入 Azure AD:

1. 建立[服務主體憑證](/azure/azure-resource-manager/resource-group-authenticate-service-principal)。 

2. 將憑證部署至 _LocalMachine_ 或 _CurrentUser_ 存放區。 

3. 將名為 **AzureServicesAuthConnectionString** 的環境變數設定為：

    ```
    RunAs=App;AppId={AppId};TenantId={TenantId};CertificateThumbprint={Thumbprint};
          CertificateStoreLocation={LocalMachine or CurrentUser}
    ```
 
    使用步驟 1 產生的值取代 _{AppId}_、_{TenantId}_ 和 _{Thumbprint}_。

    根據您的部署計劃，**CertificateStoreLocation**必須是 _CurrentUser_ 或 _LocalMachine_。

4. 執行應用程式。 

若要使用共用密碼認證登入 Azure AD：

1. 建立[使用密碼的服務主體](/azure/azure-resource-manager/resource-group-authenticate-service-principal)，並授與 Key Vault 的存取權。 

2. 將名為 **AzureServicesAuthConnectionString** 的環境變數設定為：

    ```
    RunAs=App;AppId={AppId};TenantId={TenantId};AppKey={ClientSecret} 
    ```

    使用步驟 1 產生的值取代 _{AppId}_、_{TenantId}_ 和 _{ClientSecret}_。

3. 執行應用程式。 

當所有項目都正確地設定之後，便不必再變更任何程式碼。  `AzureServiceTokenProvider` 會使用環境變數和憑證來向 Azure AD 進行驗證。 

<a name="connectionstrings"></a>
## <a name="connection-string-support"></a>連接字串支援

依預設，`AzureServiceTokenProvider` 會使用多個方法來擷取權杖。 

若要控制處理程序，請使用傳遞至 `AzureServiceTokenProvider` 建構函式或 *AzureServicesAuthConnectionString* 環境變數中指定的連接字串。 

支援下列選項：

| 連接&nbsp;字串&nbsp;選項 | 案例 | 註解|
|:--------------------------------|:------------------------|:----------------------------|
| `RunAs=Developer; DeveloperTool=AzureCli` | 本機開發 | AzureServiceTokenProvider 會使用 AzureCli 來取得權杖。 |
| `RunAs=Developer; DeveloperTool=VisualStudio` | 本機開發 | AzureServiceTokenProvider 會使用 Visual Studio 來取得權杖。 |
| `RunAs=CurrentUser;` | 本機開發 | AzureServiceTokenProvider 會使用 Azure AD 整合式驗證來取得權杖。 |
| `RunAs=App;` | Azure 資源的受控識別 | AzureServiceTokenProvider 會使用受控識別取得權杖。 |
| `RunAs=App;AppId={AppId};TenantId={TenantId};CertificateThumbprint`<br>`   ={Thumbprint};CertificateStoreLocation={LocalMachine or CurrentUser}`  | 服務主體 | `AzureServiceTokenProvider` 會使用憑證從 Azure AD 取得權杖。 |
| `RunAs=App;AppId={AppId};TenantId={TenantId};`<br>`   CertificateSubjectName={Subject};CertificateStoreLocation=`<br>`   {LocalMachine or CurrentUser}` | 服務主體 | `AzureServiceTokenProvider` 會使用憑證從 Azure AD 取得權杖|
| `RunAs=App;AppId={AppId};TenantId={TenantId};AppKey={ClientSecret}` | 服務主體 |`AzureServiceTokenProvider` 會使用密碼從 Azure AD 取得權杖。 |


## <a name="next-steps"></a>後續步驟

- 深入了解 [Azure 資源的受控識別](/azure/app-service/app-service-managed-service-identity)。

- 了解[驗證和授權應用程式](/azure/app-service/app-service-authentication-overview)的不同方式。

- 深入了解 Azure AD [驗證案例](/azure/active-directory/develop/active-directory-authentication-scenarios#web-browser-to-web-application)。
