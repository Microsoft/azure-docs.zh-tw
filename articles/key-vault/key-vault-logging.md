---
title: Azure 金鑰保存庫記錄 | Microsoft Docs
description: 使用本教學課程來協助您開始使用 Azure 金鑰保存庫記錄。
services: key-vault
documentationcenter: ''
author: barclayn
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 43f96a2b-3af8-4adc-9344-bc6041fface8
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 10/16/2017
ms.author: barclayn
ms.openlocfilehash: e7dcb3778de31258f4aa3c946ffa214d87cb858a
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32178818"
---
# <a name="azure-key-vault-logging"></a>Azure 金鑰保存庫記錄
大部分地區均提供 Azure 金鑰保存庫。 如需詳細資訊，請參閱 [金鑰保存庫價格頁面](https://azure.microsoft.com/pricing/details/key-vault/)。

## <a name="introduction"></a>簡介
在建立一或多個金鑰保存庫之後，您可能會想要監視金鑰保存庫的存取方式、時間和存取者。 想要做到這點，您可以啟用金鑰保存庫記錄，以在您提供的 Azure 儲存體帳戶中儲存這方面的資訊。 系統會自動為您指定的儲存體帳戶建立新的容器，其名稱為 **insights-logs-auditevent** ，而您也可以使用同一個儲存體帳戶來收集多個金鑰保存庫的記錄。

您最多在執行過金鑰保存庫作業 10 分鐘後，就能存取其記錄資訊。 但大多不用這麼久。  儲存體帳戶中的記錄由您全權管理：

* 請使用標準的 Azure 存取控制方法限制可存取記錄的人員，藉此來保護記錄。
* 刪除不想繼續保留在儲存體帳戶中的記錄。

請使用本教學課程來協助您開始使用 Azure 金鑰保存庫記錄、建立儲存體帳戶、啟用記錄，以及解譯所收集到的記錄資訊。  

> [!NOTE]
> 本教學課程不會指示如何建立金鑰保存庫、金鑰或密碼。 如需這方面的資訊，請參閱 [開始使用 Azure 金鑰保存庫](key-vault-get-started.md)。 或者，如需跨平台命令列介面的指示，請參閱 [這個對等的教學課程](key-vault-manage-with-cli2.md)。
>
> 目前，您無法在 Azure 入口網站中設定 Azure 金鑰保存庫。 請改用這些 Azure PowerShell 指示。
>
>

如需 Azure 金鑰保存庫的概觀資訊，請參閱 [什麼是 Azure 金鑰保存庫？](key-vault-whatis.md)

## <a name="prerequisites"></a>先決條件
若要完成本教學課程，您必須具備下列項目：

* 所使用的現有金鑰保存庫。  
* Azure PowerShell ( **至少必須是 1.0.1 版**)。 若要安裝 Azure PowerShell，並將它與 Azure 訂用帳戶建立關聯，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 如果您已安裝 Azure PowerShell 但不知道版本，請在 Azure PowerShell 主控台中輸入 `(Get-Module azure -ListAvailable).Version`。  
* 足夠的 Azure 儲存體以儲存金鑰保存庫記錄。

## <a id="connect"></a>連線到您的訂用帳戶
開始 Azure PowerShell 工作階段，並使用下列命令登入您的 Azure 帳戶：  

    Connect-AzureRmAccount

在快顯瀏覽器視窗中，輸入您的 Azure 帳戶使用者名稱與密碼。 Azure PowerShell 會取得與此帳戶相關聯的所有訂用帳戶，並依預設使用第一個訂用帳戶。

如果您有多個訂用帳戶，您可能必須指定用來建立 Azure 金鑰保存庫的那一個特定訂用帳戶。 輸入下列命令以查看您帳戶的訂用帳戶：

    Get-AzureRmSubscription

然後，輸入下列命令以指定與所要記錄之金鑰保存庫相關聯的訂用帳戶：

    Set-AzureRmContext -SubscriptionId <subscription ID>

> [!NOTE]
> 這個步驟很重要，如果您有多個與帳戶相關聯的訂用帳戶會特別實用。 若您略過此步驟，則可能會收到註冊 Microsoft.Insights 的錯誤。
>   
>

如需設定 Azure PowerShell 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

## <a id="storage"></a>建立新的儲存體帳戶來儲存記錄
雖然您可以使用現有儲存體帳戶來儲存記錄，但我們將建立新的儲存體帳戶來專用儲存金鑰保存庫記錄。 為了方便起見，在稍後遇到必須指定此帳戶時，我們會將詳細資料儲存到名為 **sa**的變數中。

為了進一步簡化管理，我們也會使用包含我們的金鑰保存庫的同一個資源群組。 在 [開始使用教學課程](key-vault-get-started.md)中，此資源群組的名稱是 **ContosoResourceGroup** ，並且我們會繼續使用「東亞」位置。 請視情況將這些值替換成您自己的值：

    $sa = New-AzureRmStorageAccount -ResourceGroupName ContosoResourceGroup -Name contosokeyvaultlogs -Type Standard_LRS -Location 'East Asia'


> [!NOTE]
> 如果您決定使用現有儲存體帳戶，則必須使用和金鑰保存庫相同的訂用帳戶，此外也必須使用 Resource Manager 部署模型，而非傳統部署模型。
>
>

## <a id="identify"></a>識別用來儲存記錄的金鑰保存庫
在開始使用教學課程中，金鑰保存庫名稱是 **ContosoKeyVault**，因此我們會繼續使用該名稱，並將詳細資料儲存到名為 **kv** 的變數中：

    $kv = Get-AzureRmKeyVault -VaultName 'ContosoKeyVault'


## <a id="enable"></a>啟用記錄
為了啟用金鑰保存庫記錄，我們將使用 Set-AzureRmDiagnosticSetting Cmdlet，並搭配針對新儲存體帳戶和金鑰保存庫所建立的變數。 我們也會將 **-Enabled** 旗標設定為 **$true**，並將類別設定為 AuditEvent (金鑰保存庫記錄適用的唯一類別)：

    Set-AzureRmDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Categories AuditEvent

此命令的輸出包含：

    StorageAccountId   : /subscriptions/<subscription-GUID>/resourceGroups/ContosoResourceGroup/providers/Microsoft.Storage/storageAccounts/ContosoKeyVaultLogs
    ServiceBusRuleId   :
    StorageAccountName :
        Logs
        Enabled           : True
        Category          : AuditEvent
        RetentionPolicy
        Enabled : False
        Days    : 0


這個結果可確認金鑰保存庫記錄現已啟用，系統會將資訊儲存到儲存體帳戶中。

您也可以選擇性地設定記錄檔的保留原則，以便自動刪除較舊的記錄檔。 例如，使用 **-RetentionEnabled** 旗標將保留原則設為 **$true** 並將 **-RetentionInDays** 參數設為 **90**，以便自動刪除超過 90 天的舊記錄檔。

    Set-AzureRmDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Categories AuditEvent -RetentionEnabled $true -RetentionInDays 90

所記錄的內容：

* 會記錄所有已驗證的 REST API 要求，包括因為存取權限、系統錯誤或要求錯誤而發生的失敗要求。
* 對金鑰保存庫本身所執行的作業，包括建立、刪除、設定金鑰保存庫存取原則，以及更新金鑰保存庫屬性 (例如標記)。
* 對金鑰保存庫中的金鑰和密碼所執行的作業，包括建立、修改或刪除這些金鑰或密碼，以及簽署、驗證、加密、解密、包裝和解除包裝金鑰、取得密碼、列出金鑰和密碼及其版本等等的作業。
* 產生 401 回應的未經驗證要求。 例如，沒有持有人權杖的要求，或格式不正確或已過期的要求，或具有無效權杖的要求。  

## <a id="access"></a>存取記錄
金鑰保存庫記錄儲存在所提供儲存體帳戶的 **insights-logs-auditevent** 容器中。 若要列出此容器中的所有 blob，請輸入：

首先，建立容器名稱的變數。 此變數會用於逐步解說剩餘的各個部分。

    $container = 'insights-logs-auditevent'

若要列出此容器中的所有 blob，請輸入：

    Get-AzureStorageBlob -Container $container -Context $sa.Context
其輸出類似如下範例：

**容器 Uri：https://contosokeyvaultlogs.blob.core.windows.net/insights-logs-auditevent**

**名稱**

- - -
**resourceId=/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSORESOURCEGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT/y=2016/m=01/d=05/h=01/m=00/PT1H.json**

**resourceId=/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSORESOURCEGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT/y=2016/m=01/d=04/h=02/m=00/PT1H.json**

**resourceId=/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSORESOURCEGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT/y=2016/m=01/d=04/h=18/m=00/PT1H.json**\*\*

在此輸出中我們可以看到，blob 遵循以下命名慣例：**resourceId=<ARM resource ID>/y=<year>/m=<month>/d=<day of month>/h=<hour>/m=<minute>/filename.json**

日期和時間值使用 UTC。

因為可以使用相同的儲存體帳戶來收集多個資源的記錄，blob 名稱中的完整資源識別碼很適合用來只存取或下載所需 blob。 但在這麼做之前，我們要先討論如何下載所有 blob。

首先，建立資料夾來下載 blob。 例如︰

    New-Item -Path 'C:\Users\username\ContosoKeyVaultLogs' -ItemType Directory -Force

然後取得所有 blob 的清單：  

    $blobs = Get-AzureStorageBlob -Container $container -Context $sa.Context

透過 'Get-AzureStorageBlobContent' 以管道傳送這份清單，將 blob 下載到目的地資料夾：

    $blobs | Get-AzureStorageBlobContent -Destination 'C:\Users\username\ContosoKeyVaultLogs'

在執行第二個命令時，blob 名稱中的 **/** 分隔符號會在目的地資料夾下建立完整資料夾結構，而此結構將會用來下載 blob 並儲存為檔案。

若要有所選擇地下載 blob，請使用萬用字元。 例如︰

* 如果您有多個金鑰保存庫，並且只想下載其中的 CONTOSOKEYVAULT3 金鑰保存庫的記錄：

        Get-AzureStorageBlob -Container $container -Context $sa.Context -Blob '*/VAULTS/CONTOSOKEYVAULT3
* 如果您有多個資源群組，並且只想下載其中某個資源群組的記錄，請使用 `-Blob '*/RESOURCEGROUPS/<resource group name>/*'`：

        Get-AzureStorageBlob -Container $container -Context $sa.Context -Blob '*/RESOURCEGROUPS/CONTOSORESOURCEGROUP3/*'
* 如果您想下載 2016 年 1 月份當月的所有記錄，請使用 `-Blob '*/year=2016/m=01/*'`：

        Get-AzureStorageBlob -Container $container -Context $sa.Context -Blob '*/year=2016/m=01/*'

您現在已做好準備，可以開始查看記錄中有何內容。 但在開始之前，您可能還需要了解 Get-AzureRmDiagnosticSetting 的兩個參數：

* 若要查詢金鑰保存庫資源的診斷設定狀態：`Get-AzureRmDiagnosticSetting -ResourceId $kv.ResourceId`
* 若要停用金鑰保存庫資源記錄： `Set-AzureRmDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $false -Categories AuditEvent`

## <a id="interpret"></a>解譯金鑰保存庫記錄
各個 blob 皆會儲存為文字，並格式化為 JSON blob。 以下是執行 `Get-AzureRmKeyVault -VaultName 'contosokeyvault'`後所得到的範例記錄項目：

    {
        "records":
        [
            {
                "time": "2016-01-05T01:32:01.2691226Z",
                "resourceId": "/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSOGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT",
                "operationName": "VaultGet",
                "operationVersion": "2015-06-01",
                "category": "AuditEvent",
                "resultType": "Success",
                "resultSignature": "OK",
                "resultDescription": "",
                "durationMs": "78",
                "callerIpAddress": "104.40.82.76",
                "correlationId": "",
                "identity": {"claim":{"http://schemas.microsoft.com/identity/claims/objectidentifier":"d9da5048-2737-4770-bd64-XXXXXXXXXXXX","http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn":"live.com#username@outlook.com","appid":"1950a258-227b-4e31-a9cf-XXXXXXXXXXXX"}},
                "properties": {"clientInfo":"azure-resource-manager/2.0","requestUri":"https://control-prod-wus.vaultcore.azure.net/subscriptions/361da5d4-a47a-4c79-afdd-XXXXXXXXXXXX/resourcegroups/contosoresourcegroup/providers/Microsoft.KeyVault/vaults/contosokeyvault?api-version=2015-06-01","id":"https://contosokeyvault.vault.azure.net/","httpStatusCode":200}
            }
        ]
    }


下表列出各個欄位的名稱和其描述。

| 欄位名稱 | 說明 |
| --- | --- |
| 分析 |日期和時間 (UTC)。 |
| ResourceId |Azure 資源管理員資源識別碼。 對於金鑰保存庫記錄來說，這一律是金鑰保存庫的資源識別碼。 |
| operationName |下一份表格所述作業的名稱。 |
| operationVersion |這是用戶端所要求的 REST API 版本。 |
| category |對於金鑰保存庫記錄來說，AuditEvent 是其唯一可用的值。 |
| resultType |REST API 要求的結果。 |
| resultSignature |HTTP 狀態。 |
| resultDescription |結果的其他描述 (若有提供)。 |
| durationMs |服務 REST API 要求時所花費的時間，以毫秒為單位。 此時間不包括網路延遲，因此在用戶端所測量到的時間可能不符合此時間。 |
| callerIpAddress |提出要求之用戶端的 IP 位址。 |
| correlationId |選擇性的 GUID，用戶端可傳遞此 GUID 來讓用戶端記錄與服務端 (金鑰保存庫) 記錄相互關聯。 |
| 身分識別 |在提出 REST API 要求時所提供之權杖中的身分識別。 和 Azure PowerShell Cmdlet 所產生之要求的案例一樣，這通常是「使用者」、「服務主體」或「使用者+appId」的組合。 |
| properties |在不同作業 (operationName) 中，此欄位會包含不同的資訊。 在大部分情況下，會包含用戶端資訊 (用戶端所傳遞的使用者代理程式字串)、確切的 REST API 要求 URI 和 HTTP 狀態碼。 此外，在因為提出要求 (例如，KeyCreate 或 VaultGet) 而傳回物件時，此欄位還將包含金鑰 URI (形式為 "id")、保存庫 URI 或密碼 URI。 |

**operationName** 欄位值的格式為 ObjectVerb。 例如︰

* 所有金鑰保存庫作業都有 'Vault`<action>`' 格式，例如 `VaultGet` 和 `VaultCreate`。
* 所有金鑰作業都有 'Key`<action>`' 格式，例如 `KeySign` 和 `KeyList`。
* 所有密碼作業都有 'Secret`<action>`' 格式，例如 `SecretGet` 和 `SecretListVersions`。

下表列出 operationName 和相對應的 REST API 命令。

| operationName | REST API 命令 |
| --- | --- |
| 驗證 |透過 Azure Active Directory 端點 |
| VaultGet |[取得金鑰保存庫的相關資訊](https://msdn.microsoft.com/library/azure/mt620026.aspx) |
| VaultPut |[建立或更新金鑰保存庫](https://msdn.microsoft.com/library/azure/mt620025.aspx) |
| VaultDelete |[刪除金鑰保存庫](https://msdn.microsoft.com/library/azure/mt620022.aspx) |
| VaultPatch |[更新金鑰保存庫](https://msdn.microsoft.com/library/azure/mt620025.aspx) |
| VaultList |[列出資源群組中的所有金鑰保存庫](https://msdn.microsoft.com/library/azure/mt620027.aspx) |
| KeyCreate |[建立金鑰](https://msdn.microsoft.com/library/azure/dn903634.aspx) |
| KeyGet |[取得金鑰的相關資訊](https://msdn.microsoft.com/library/azure/dn878080.aspx) |
| KeyImport |[將金鑰匯入保存庫](https://msdn.microsoft.com/library/azure/dn903626.aspx) |
| KeyBackup |[備份金鑰](https://msdn.microsoft.com/library/azure/dn878058.aspx)。 |
| KeyDelete |[刪除金鑰](https://msdn.microsoft.com/library/azure/dn903611.aspx) |
| KeyRestore |[還原金鑰](https://msdn.microsoft.com/library/azure/dn878106.aspx) |
| KeySign |[使用金鑰簽署](https://msdn.microsoft.com/library/azure/dn878096.aspx) |
| KeyVerify |[使用金鑰驗證](https://msdn.microsoft.com/library/azure/dn878082.aspx) |
| KeyWrap |[包裝金鑰](https://msdn.microsoft.com/library/azure/dn878066.aspx) |
| KeyUnwrap |[解除包裝金鑰](https://msdn.microsoft.com/library/azure/dn878079.aspx) |
| KeyEncrypt |[使用金鑰加密](https://msdn.microsoft.com/library/azure/dn878060.aspx) |
| KeyDecrypt |[使用金鑰解密](https://msdn.microsoft.com/library/azure/dn878097.aspx) |
| KeyUpdate |[更新金鑰](https://msdn.microsoft.com/library/azure/dn903616.aspx) |
| KeyList |[列出保存庫中的金鑰](https://msdn.microsoft.com/library/azure/dn903629.aspx) |
| KeyListVersions |[列出金鑰的版本](https://msdn.microsoft.com/library/azure/dn986822.aspx) |
| SecretSet |[建立密碼](https://msdn.microsoft.com/library/azure/dn903618.aspx) |
| SecretGet |[取得密碼](https://msdn.microsoft.com/library/azure/dn903633.aspx) |
| SecretUpdate |[更新密碼](https://msdn.microsoft.com/library/azure/dn986818.aspx) |
| SecretDelete |[刪除秘密](https://msdn.microsoft.com/library/azure/dn903613.aspx) |
| SecretList |[列出保存庫中的密碼](https://msdn.microsoft.com/library/azure/dn903614.aspx) |
| SecretListVersions |[列出密碼的版本](https://msdn.microsoft.com/library/azure/dn986824.aspx) |

## <a id="loganalytics"></a>使用 Log Analytics

您可以使用 Log Analytics 中的 Azure 金鑰保存庫解決方案來檢閱 Azure 金鑰保存庫 AuditEvent 記錄檔。 如需詳細資訊 (包括如何進行此設定)，請參閱 [Log Analytics 中的 Azure Key Vault 解決方案](../log-analytics/log-analytics-azure-key-vault.md)。 如果您需要從 Log Analytics 預覽版時期所提供的舊 Key Vault 解決方案 (其中您是先將記錄檔路由遞送至「Azure 儲存體」帳戶，然後將 Log Analytics 設定成從該處讀取) 移轉，本文也包含相關指示。

## <a id="next"></a>後續步驟
如需在 Web 應用程式中使用 Azure 金鑰保存庫的教學課程，請參閱 [從 Web 應用程式使用 Azure 金鑰保存庫](key-vault-use-from-web-application.md)。

如需程式設計參考，請參閱 [Azure 金鑰保存庫開發人員指南](key-vault-developers-guide.md)。

如需 Azure 金鑰保存庫的 Azure PowerShell 1.0 Cmdlet 清單，請參閱 [Azure 金鑰保存庫 Cmdlet](/powershell/module/azurerm.keyvault/#key_vault)。

如需有關 Azure 金鑰保存庫的金鑰輪替和記錄檔稽核的教學課程，請參閱 [如何使用端對端金鑰輪替和稽核設定金鑰保存庫](key-vault-key-rotation-log-monitoring.md)。
