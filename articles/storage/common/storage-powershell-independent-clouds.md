---
title: 使用 Azure PowerShell 在 Azure 獨立雲端中管理儲存體 | Microsoft Docs
description: 使用 Azure PowerShell 在中國雲端、政府雲端和德國雲端中管理儲存體
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 10/24/2017
ms.author: rogarana
ms.component: common
ms.openlocfilehash: 75a3dcb5aeb3e30da570eb57d0d1495710624e54
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42144019"
---
# <a name="managing-storage-in-the-azure-independent-clouds-using-powershell"></a>使用 PowerShell 在 Azure 獨立雲端中管理儲存體

大多數人會針對其全域 Azure 部署使用 Azure 公用雲端。 也有因為主權等等原因的一些獨立部署 Microsoft Azure。 這些獨立部署稱為「環境」。 下列清單詳細說明目前可用的獨立雲端。

* [Azure Government 雲端](https://azure.microsoft.com/features/gov/)
* [21Vianet 在中國所操作的 Azure 中國雲端](http://www.windowsazure.cn/)
* [Azure 德國雲端](../../germany/germany-welcome.md)

## <a name="using-an-independent-cloud"></a>使用獨立雲端 

若要在其中一個獨立雲端中使用 Azure 儲存體，您要連線到該雲端，而非 Azure 公用。 若要使用其中一個獨立雲端，而非 Azure 公用：

* 指定要連線的「環境」。
* 決定並且使用可用的區域。
* 使用正確的端點尾碼，它與 Azure 公用的尾碼不同。

這些範例需要 Azure PowerShell 模組 4.4.0 版或更新版本。 在 PowerShell 視窗中，執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果未列出任何項目，或者您必須升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 

## <a name="log-in-to-azure"></a>登入 Azure

執行 [Get-AzureRmEnvironment](/powershell/module/servicemanagement/azurerm.profile/get-azurermenvironment) Cmdlet，查看可用的 Azure 環境：
   
```powershell
Get-AzureRmEnvironment
```

登入您的帳戶，該帳戶可存取您想要連線並且設定環境的雲端。 這個範例示範如何登入使用 Azure Government 雲端的帳戶。   

```powershell
Connect-AzureRmAccount –Environment AzureUSGovernment
```

若要存取中國雲端，請使用環境 **AzureChinaCloud**。 若要存取德國雲端，請使用 **AzureGermanCloud**。

此時，如果您需要用來建立儲存體帳戶或其他資源的位置清單，您可以使用 [Get-AzureRmLocation](/powershell/module/azurerm.resources/get-azurermlocation) 查詢適用於選取之雲端的位置。

```powershell
Get-AzureRmLocation | select Location, DisplayName
```

下表顯示針對德國雲端傳回的位置。

|位置 | DisplayName |
|----|----|
| germanycentral | 德國中部|
| germanynortheast | 德國東北部 | 


## <a name="endpoint-suffix"></a>端點尾碼

端點尾碼，這些環境的每個尾碼都與 Azure 公用端點不同。 例如，Azure 公用的 blob 端點尾碼是 **blob.core.windows.net**。 針對政府雲端，blob 端點尾碼是 **blob.core.usgovcloudapi.net**。 

### <a name="get-endpoint-using-get-azurermenvironment"></a>使用 Get-AzureRMEnvironment 取得端點 

使用 [Get-AzureRMEnvironment](/powershell/module/azurerm.profile/get-azurermenvironment) 擷取端點尾碼。 端點是環境的「StorageEndpointSuffix」屬性。 下列程式碼片段示範如何執行這項作業。 這些命令都會傳回類似 "core.cloudapp.net" 或 "core.cloudapi.de" 等等的資訊。將這個項目附加至儲存體服務，以存取該服務。 例如，"queue.core.cloudapi.de" 會在德國雲端中存取佇列服務。

此程式碼片段會擷取所有環境及其端點尾碼。

```powershell
Get-AzureRmEnvironment | select Name, StorageEndpointSuffix 
```

此命令會傳回下列結果。

| 名稱| StorageEndpointSuffix|
|----|----|
| AzureChinaCloud | core.chinacloudapi.cn|
| AzureCloud | core.windows.net |
| AzureGermanCloud | core.cloudapi.de|
| AzureUSGovernment | core.usgovcloudapi.net |

若要擷取指定環境的所有屬性，請呼叫 **Get-AzureRmEnvironment** 並且指定雲端名稱。 此程式碼片段會傳回屬性清單。在清單中尋找 **StorageEndpointSuffix**。 下列範例適用於德國雲端。

```powershell
Get-AzureRmEnvironment -Name AzureGermanCloud 
```

結果如下所示：

|屬性名稱|值|
|----|----|
| 名稱 | AzureGermanCloud |
| EnableAdfsAuthentication | False |
| ActiveDirectoryServiceEndpointResourceI | http://management.core.cloudapi.de/ |
| GalleryURL | https://gallery.cloudapi.de/ |
| ManagementPortalUrl | https://portal.microsoftazure.de/ | 
| ServiceManagementUrl | https://manage.core.cloudapi.de/ |
| PublishSettingsFileUrl| https://manage.microsoftazure.de/publishsettings/index |
| ResourceManagerUrl | http://management.microsoftazure.de/ |
| SqlDatabaseDnsSuffix | .database.cloudapi.de |
| **StorageEndpointSuffix** | core.cloudapi.de |
| ... | ... | 

若只要擷取儲存體端點尾碼屬性，請擷取特定雲端，並且只要求該單一屬性。

```powershell
$environment = Get-AzureRmEnvironment -Name AzureGermanCloud
Write-Host "Storage EndPoint Suffix = " $environment.StorageEndpointSuffix 
```

這樣會傳回下列資訊。

```
Storage Endpoint Suffix = core.cloudapi.de
```

### <a name="get-endpoint-from-a-storage-account"></a>從儲存體帳戶取得端點

您也可以檢查儲存體帳戶的屬性以擷取端點。 如果您已經在 PowerShell 指令碼中使用儲存體帳戶，這樣做很有幫助，您可以只擷取您需要的端點。 

```powershell
# Get a reference to the storage account.
$resourceGroup = "myexistingresourcegroup"
$storageAccountName = "myexistingstorageaccount"
$storageAccount = Get-AzureRmStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageAccountName 
  # Output the endpoints.
Write-Host "blob endpoint = " $storageAccount.PrimaryEndPoints.Blob 
Write-Host "file endpoint = " $storageAccount.PrimaryEndPoints.File
Write-Host "queue endpoint = " $storageAccount.PrimaryEndPoints.Queue
Write-Host "table endpoint = " $storageAccount.PrimaryEndPoints.Table
```

針對政府雲端中的儲存體帳戶，這會傳回下列項目： 

```
blob endpoint = http://myexistingstorageaccount.blob.core.usgovcloudapi.net/
file endpoint = http://myexistingstorageaccount.file.core.usgovcloudapi.net/
queue endpoint = http://myexistingstorageaccount.queue.core.usgovcloudapi.net/
table endpoint = http://myexistingstorageaccount.table.core.usgovcloudapi.net/
```

## <a name="after-setting-the-environment"></a>設定環境之後

從這裡開始，您可以使用與用來管理儲存體帳戶及存取資料層的相同 PowerShell，如同[使用 Azure PowerShell 與 Azure 儲存體](storage-powershell-guide-full.md)一文所述。

## <a name="clean-up-resources"></a>清除資源

如果您針對這個練習建立新的資源群組和儲存體帳戶，您可以藉由移除資源群組來移除所有資產。 這會同時刪除群組內含的所有資源。 在本例中，它會移除建立的儲存體帳戶和資源群組本身。

```powershell
Remove-AzureRmResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>後續步驟

* [在 PowerShell 工作階段之間保存使用者登入](/powershell/azure/context-persistence)
* [Azure Government 儲存體](../../azure-government/documentation-government-services-storage.md)
* [Microsoft Azure Government 開發人員指南](../../azure-government/documentation-government-developer-guide.md)
* [Azure 中國應用程式的開發人員注意事項](https://msdn.microsoft.com/library/azure/dn578439.aspx)
* [Azure 德國文件](../../germany/germany-welcome.md)
