---
ms.assetid: ''
title: Azure Key Vault 儲存體帳戶金鑰
description: 儲存體帳戶金鑰提供 Azure Key Vault 與 Azure 儲存體帳戶的金鑰型存取之間的完美整合。
ms.topic: conceptual
services: key-vault
ms.service: key-vault
author: bryanla
ms.author: bryanla
manager: mbaldwin
ms.date: 08/21/2017
ms.openlocfilehash: 7545a035541a4e464a6c82acb9fa9de18cf8e86d
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304317"
---
# <a name="azure-key-vault-storage-account-keys"></a>Azure Key Vault 儲存體帳戶金鑰

在 Azure Key Vault 儲存體帳戶金鑰之前，開發人員必須管理自己的 Azure 儲存體帳戶 (ASA) 金鑰，並以手動方式或透過外部自動化來輪替金鑰。 現在，Key Vault 儲存體帳戶金鑰會實作為 [Key Vault 密碼](https://docs.microsoft.com/rest/api/keyvault/about-keys--secrets-and-certificates#BKMK_WorkingWithSecrets)，以便向 Azure 儲存體帳戶進行驗證。

Azure 儲存體帳戶 (ASA) 金鑰功能可管理密碼輪替， 也提供共用存取簽章 (SAS) 方法，讓您不需要直接接觸 ASA 金鑰。

如需 Azure 儲存體帳戶的更多一般資訊，請參閱[關於 Azure 儲存體帳戶](https://docs.microsoft.com/azure/storage/storage-create-storage-account) 。

## <a name="supporting-interfaces"></a>支援的介面

在[金鑰保存庫開發人員指南](key-vault-developers-guide.md#coding-with-key-vault)中，可以找到我們的程式設計和指令碼介面的完整清單和連結。


## <a name="what-key-vault-manages"></a>Key Vault 的管理內容

當您使用受控儲存體帳戶金鑰時，Key Vault 會代表您執行幾個內部管理功能。

- Azure Key Vault 會管理 Azure 儲存體帳戶 (ASA) 的金鑰。
    - Azure Key Vault 可在內部列出 (同步) Azure 儲存體帳戶金鑰。
    - Azure Key Vault 會定期重新產生 (輪替) 金鑰。
    - 永遠不會傳回金鑰值以回應呼叫者。
    - Azure Key Vault 會管理儲存體帳戶和傳統儲存體帳戶的金鑰。
- Azure Key Vault 可讓身為保存庫/物件擁有者的您建立 SAS (共用存取簽章、帳戶或服務 SAS) 定義。
    - 系統會透過 REST URI 路徑，將使用 SAS 定義建立的 SAS 值傳回作為密碼。 如需詳細資訊，請參閱 [Azure Key Vault REST API 參考](/rest/api/keyvault)中的 SAS 定義作業。

## <a name="naming-guidance"></a>命名指導方針

- 儲存體帳戶名稱必須介於 3 到 24 個字元的長度，而且只能包含數字和小寫字母。
- SAS 定義名稱的長度必須是 1-102 個字元，而且只能包含 0-9、a-z、A-Z。

## <a name="developer-experience"></a>開發人員體驗

### <a name="before-azure-key-vault-storage-keys"></a>Azure Key Vault 儲存體金鑰之前

開發人員之前必須對儲存體帳戶金鑰執行下列實作，才能存取 Azure 儲存體。
1. 將連接字串或 SAS 權杖儲存在 Azure AppService 應用程式設定或另一個儲存體中。
1. 在應用程式啟動時，擷取連接字串或 SAS 權杖。
1. 建立 [CloudStorageAccount](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.cloudstorageaccount)，以便與儲存體互動。

```cs
// The Connection string is being fetched from App Service application settings
var connectionStringOrSasToken = CloudConfigurationManager.GetSetting("StorageConnectionString");
var storageAccount = CloudStorageAccount.Parse(connectionStringOrSasToken);
var blobClient = storageAccount.CreateCloudBlobClient();
 ```

### <a name="after-azure-key-vault-storage-keys"></a>Azure Key Vault 儲存體金鑰之後

開發人員會建立 [KeyVaultClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.keyvault.keyvaultclient)，並利用它來取得他們儲存體的 SAS 權杖。 之後，他們會使用該權仗來建立 [CloudStorageAccount](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.cloudstorageaccount)。

```cs
// Create KeyVaultClient with vault credentials
var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(securityToken));

// Get a SAS token for our storage from Key Vault
var sasToken = await kv.GetSecretAsync("SecretUri");

// Create new storage credentials using the SAS token.
var accountSasCredential = new StorageCredentials(sasToken.Value);

// Use the storage credentials and the Blob storage endpoint to create a new Blob service client.
var accountWithSas = new CloudStorageAccount(accountSasCredential, new Uri ("https://myaccount.blob.core.windows.net/"), null, null, null);

var blobClientWithSas = accountWithSas.CreateCloudBlobClient();

// Use the blobClientWithSas
...

// If your SAS token is about to expire, get the SAS Token again from Key Vault and update it.
sasToken = await kv.GetSecretAsync("SecretUri");
accountSasCredential.UpdateSASToken(sasToken);
```

 ### <a name="developer-guidance"></a>開發人員指引

- 只允許 Key Vault 管理您的 ASA 金鑰。 請勿嘗試自行管理，您會干擾 Key Vault 的處理序。
- 不允許多個 Key Vault 物件管理 ASA 金鑰。
- 如果您需要以手動方式重新產生 ASA 金鑰，建議您透過 Key Vault 重新產生。

## <a name="getting-started"></a>開始使用

### <a name="give-key-vault-access-to-your-storage-account"></a>對儲存體帳戶提供 Key Vault 存取權 

和許多應用程式一樣，Key Vault 會向 Azure AD 註冊以便使用 OAuth 來存取其他服務。 註冊期間會建立[服務主體](/azure/active-directory/develop/app-objects-and-service-principals)物件，以用來在執行階段代表應用程式的身分識別。 服務主體也可用來授權應用程式的身分識別，使其可透過角色型存取控制 (RBAC) 來存取另一個資源。

Azure Key Vault 應用程式身分識別需要「列出」及「重新產生」儲存體帳戶金鑰的權限。 請使用下列步驟來設定這些權限：

```powershell
# Get the resource ID of the Azure Storage Account you want to manage.
# Below, we are fetching a storage account using Azure Resource Manager
$storage = Get-AzureRmStorageAccount -ResourceGroupName "mystorageResourceGroup" -StorageAccountName "mystorage"

# Get Application ID of Azure Key Vault's service principal
$servicePrincipal = Get-AzureRmADServicePrincipal -ServicePrincipalName cfa8b339-82a2-471a-a3c9-0fc0be7a4093

# Assign Storage Key Operator role to Azure Key Vault Identity
New-AzureRmRoleAssignment -ObjectId $servicePrincipal.Id -RoleDefinitionName 'Storage Account Key Operator Service Role' -Scope $storage.Id
```

    >[!NOTE]
    > For a classic account type, set the role parameter to *"Classic Storage Account Key Operator Service Role."*

## <a name="working-example"></a>實用範例

下列範例示範如何建立金鑰保存庫受控 Azure 儲存體帳戶和相關聯的 SAS 定義。

### <a name="prerequisite"></a>必要條件

請確定您已完成[設定角色型存取控制 (RBAC) 權限](#setup-for-role-based-access-control-rbac-permissions)。

### <a name="setup"></a>設定

```powershell
# This is the name of our Key Vault
$keyVaultName = "mykeyVault"

# Fetching all the storage account object, of the ASA we want to manage with KeyVault
$storage = Get-AzureRmStorageAccount -ResourceGroupName "mystorageResourceGroup" -StorageAccountName "mystorage"

# Get ObjectId of Azure KeyVault Identity service principal
$servicePrincipalId = $(Get-AzureRmADServicePrincipal -ServicePrincipalName cfa8b339-82a2-471a-a3c9-0fc0be7a4093).Id
```

接下來，為**您的帳戶**設定權限，以確保您可以管理 Key Vault 中所有儲存體的權限。 在下列範例中，我們的 Azure 帳戶是 _developer@contoso.com_。

```powershell
# Searching our Azure Active Directory for our account's ObjectId
$userPrincipalId = $(Get-AzureRmADUser -SearchString "developer@contoso.com").Id

# We use the ObjectId we found to setting permissions on the vault
Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVaultName -ObjectId $userPrincipalId -PermissionsToStorage all
```

### <a name="create-a-key-vault-managed-storage-account"></a>建立 Key Vault 受控儲存體帳戶

現在，在 Azure Key Vault 中建立受控儲存體帳戶，並使用儲存體帳戶中的存取金鑰建立 SAS 權杖。
- `-ActiveKeyName` 會使用 'key2' 來產生 SAS 權杖。
- `-AccountName` 會用來識別您的受控儲存體帳戶。 為求簡單，下面我們將使用儲存體帳戶名稱，但它可以是任何名稱。
- `-DisableAutoRegenerateKey` 會指定不要重新產生儲存體帳戶金鑰。

```powershell
# Adds your storage account to be managed by Key Vault and will use the access key, key2
Add-AzureKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storage.StorageAccountName -AccountResourceId $storage.Id -ActiveKeyName key2 -DisableAutoRegenerateKey
```

### <a name="key-regeneration"></a>金鑰重新產生

如果您想要讓 Key Vault 定期重新產生儲存體的存取金鑰，您可以設定重新產生期間。 下面我們會設定 3 天的重新產生期間。 3 天之後，Key Vault 會重新產生 'key1'，並將作用中的金鑰從 'key2' 交換為 'key1'。

```powershell
$regenPeriod = [System.Timespan]::FromDays(3)
$accountName = $storage.StorageAccountName

Add-AzureKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $accountName -AccountResourceId $storage.Id -ActiveKeyName key2 -RegenerationPeriod $regenPeriod
```

### <a name="set-sas-definitions"></a>設定 SAS 定義

帳戶 SAS 會以不同的權限提供 Blob 服務的存取權。
在金鑰保存庫中為受控儲存體帳戶設定 SAS 定義。
- `-AccountName` 是 Key Vault 中的受控儲存體帳戶名稱。
- `-Name` 是儲存體中 SAS 權杖的識別碼。
- `-ValidityPeriod` 會設定所產生之 SAS 權杖的到期日。

```powershell
$validityPeriod = [System.Timespan]::FromDays(1)
$readSasName = "readBlobSas"
$writeSasName = "writeBlobSas"

Set-AzureKeyVaultManagedStorageSasDefinition -Service Blob -ResourceType Container,Service -VaultName $keyVaultName -AccountName $accountName -Name $readSasName -Protocol HttpsOnly -ValidityPeriod $validityPeriod -Permission Read,List

Set-AzureKeyVaultManagedStorageSasDefinition -Service Blob -ResourceType Container,Service,Object -VaultName $keyVaultName -AccountName $accountName -Name $writeSasName -Protocol HttpsOnly -ValidityPeriod $validityPeriod -Permission Read,List,Write
```

### <a name="get-sas-tokens"></a>取得 SAS 權杖

取得對應的 SAS 權杖，並對儲存體進行呼叫。 當您執行 [Set-AzureKeyVaultManagedStorageSasDefinition](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Set-AzureKeyVaultManagedStorageSasDefinition) 時，`AccountName` 和 `Name` 參數中的輸入會用來建構 `-SecretName`。

```powershell
$readSasToken = (Get-AzureKeyVaultSecret -VaultName $keyVaultName -SecretName "$accountName-$readSasName").SecretValueText
$writeSasToken = (Get-AzureKeyVaultSecret -VaultName $keyVaultName -SecretName "$accountName-$writeSasName").SecretValueText
```

### <a name="create-storage"></a>建立儲存體

請注意，嘗試使用 $readSasToken 存取會失敗，但是能夠使用 $writeSasToken 進行存取。

```powershell
$context1 = New-AzureStorageContext -SasToken $readSasToken -StorageAccountName $storage.StorageAccountName
$context2 = New-AzureStorageContext -SasToken $writeSasToken -StorageAccountName $storage.StorageAccountName

# Ensure the txt file in command exists in local path mentioned
Set-AzureStorageBlobContent -Container containertest1 -File "./abc.txt" -Context $context1
Set-AzureStorageBlobContent -Container cont1-file "./file.txt" -Context $context2
```

您能夠使用具有寫入權限的 SAS 權杖來存取儲存體 Blob 內容。

### <a name="relevant-powershell-cmdlets"></a>相關的 Powershell Cmdlet

- [Get-AzureKeyVaultManagedStorageAccount](https://docs.microsoft.com/powershell/module/azurerm.keyvault/get-azurekeyvaultmanagedstorageaccount)
- [Add-AzureKeyVaultManagedStorageAccount](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Add-AzureKeyVaultManagedStorageAccount)
- [Get-AzureKeyVaultManagedStorageSasDefinition](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Get-AzureKeyVaultManagedStorageSasDefinition)
- [Update-AzureKeyVaultManagedStorageAccountKey](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Update-AzureKeyVaultManagedStorageAccountKey)
- [Remove-AzureKeyVaultManagedStorageAccount](https://docs.microsoft.com/powershell/module/azurerm.keyvault/remove-azurekeyvaultmanagedstorageaccount)
- [Remove-AzureKeyVaultManagedStorageSasDefinition](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Remove-AzureKeyVaultManagedStorageSasDefinition)
- [Set-AzureKeyVaultManagedStorageSasDefinition](https://docs.microsoft.com/powershell/module/AzureRM.KeyVault/Set-AzureKeyVaultManagedStorageSasDefinition)

## <a name="storage-account-onboarding"></a>儲存體帳戶登入

範例：您身為 Key Vault 物件擁有者，將儲存體帳戶物件新增至 Azure Key Vault，以登入儲存體帳戶。

在登入期間，Key Vault 需要驗證登入帳戶的身分識別具有「列出」及「重新產生」儲存體金鑰的存取權限。 為了確認這些權限，Key Vault 從驗證服務取得 OBO (代表) 權杖，將對象設為 Azure Resource Manager，並且對 Azure 儲存體服務發出「列出」金鑰呼叫。 如果「列出」呼叫失敗，則 Key Vault 物件建立會失敗，並出現 HTTP 狀態碼「禁止」。 以此方式列出的金鑰會透過 Key Vault 實體儲存體來快取。

Key Vault 必須驗證身分識別具有「重新產生」權限，才能取得擁有權來重新產生您的金鑰。 若要驗證透過 OBO 權杖的身分識別及 Key Vault 的第一個憑證者身分識別具有這些權限：

- Key Vault 會列出儲存體帳戶資源的 RBAC 權限。
- Key Vault 會透過比對動作和非動作的規則運算式來驗證回應。

在 [Key Vault - 受控儲存體帳戶金鑰範例](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=)中尋找一些支援的範例。

如果身分識別沒有「重新產生」權限，或如果 Key Vault 的第一個憑證者身分識別沒有「列出」或「重新產生」權限，則登入要求會失敗，並傳回適當的錯誤碼和訊息。

只有在您使用 PowerShell 或 CLI 的第一個憑證者原生用戶端應用程式時，才能使用 OBO 權杖。

## <a name="other-applications"></a>其他應用程式

- 使用 Key Vault 儲存體帳戶金鑰建構的 SAS 權杖，會提供更受控制的 Azure 儲存體帳戶存取權。 如需詳細資訊，請參閱[使用共用存取簽章](https://docs.microsoft.com/azure/storage/storage-dotnet-shared-access-signature-part-1)。

## <a name="see-also"></a>另請參閱

- [關於金鑰、密碼與憑證](https://docs.microsoft.com/rest/api/keyvault/)
- [Key Vault 小組部落格](https://blogs.technet.microsoft.com/kv/)
