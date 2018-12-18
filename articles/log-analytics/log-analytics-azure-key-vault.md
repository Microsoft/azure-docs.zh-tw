---
title: Log Analytics 中的 Azure 金鑰保存庫解決方案 | Microsoft Docs
description: 您可以使用 Log Analytics 中的 Azure 金鑰保存庫解決方案來檢閱 Azure 金鑰保存庫記錄檔。
services: log-analytics
documentationcenter: ''
author: richrundmsft
manager: jochan
editor: ''
ms.assetid: 5e25e6d6-dd20-4528-9820-6e2958a40dae
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/09/2017
ms.author: richrund
ms.component: na
ms.openlocfilehash: 47158d0c2b5a80ceae25b275836d45b63db50ff4
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38309334"
---
# <a name="azure-key-vault-analytics-solution-in-log-analytics"></a>Log Analytics 中的 Azure Key Vault 分析解決方案

![Key Vault 符號](./media/log-analytics-azure-keyvault/key-vault-analytics-symbol.png)

您可以使用 Log Analytics 中的 Azure 金鑰保存庫解決方案來檢閱 Azure 金鑰保存庫 AuditEvent 記錄檔。

若要使用此解決方案，您需要啟用 Azure Key Vault 診斷的記錄，並將診斷導向至 Log Analytics 工作區。 不需要將記錄寫入 Azure Blob 儲存體。

> [!NOTE]
> 從 2017 年 1 月開始，從 Key Vault 傳送記錄到 Log Analytics 的支援方式已變更。 如果您使用的 Key Vault 解決方案標題中顯示 *(已過時)*，請參閱[從舊的 Key Vault 解決方案進行移轉](#migrating-from-the-old-key-vault-solution)，以取得您必須遵循的步驟。
>
>

## <a name="install-and-configure-the-solution"></a>安裝和設定解決方案
使用下列指示來安裝和設定 Azure 金鑰保存庫解決方案︰

1. 從 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.KeyVaultAnalyticsOMS?tab=Overview) 或使用[從方案庫新增 Log Analytics 方案](log-analytics-add-solutions.md)中所述的程序，啟用 Azure Key Vault 解決方案。
2. 使用[入口網站](#enable-key-vault-diagnostics-in-the-portal)或 [PowerShell](#enable-key-vault-diagnostics-using-powershell)，針對要監視的 Key Vault 資源啟用診斷記錄

### <a name="enable-key-vault-diagnostics-in-the-portal"></a>在入口網站中啟用 Key Vault 診斷

1. 在 Azure 入口網站中，瀏覽至要監視的 Key Vault 資源
2. 選取 [診斷記錄] 以開啟下列頁面

   ![Azure 金鑰保存庫圖格的影像](./media/log-analytics-azure-keyvault/log-analytics-keyvault-enable-diagnostics01.png)
3. 按一下 [開啟診斷] 以開啟下列頁面

   ![Azure 金鑰保存庫圖格的影像](./media/log-analytics-azure-keyvault/log-analytics-keyvault-enable-diagnostics02.png)
4. 若要開啟診斷，請按一下 [狀態] 下的 [開啟]
5. 按一下 [傳送到 Log Analytics] 核取方塊
6. 選取現有的 Log Analytics 工作區，或建立工作區
7. 若要啟用 *AuditEvent* 記錄，請按一下 [記錄] 下的核取方塊
8. 按一下 [儲存] 以啟用 Log Analytics 的診斷記錄

### <a name="enable-key-vault-diagnostics-using-powershell"></a>使用 PowerShell 啟用 Key Vault 診斷
下列 PowerShell 指令碼示範如何使用 `Set-AzureRmDiagnosticSetting` 啟用 Key Vault 的診斷記錄︰
```
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$kv = Get-AzureRmKeyVault -VaultName 'ContosoKeyVault'

Set-AzureRmDiagnosticSetting -ResourceId $kv.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```



## <a name="review-azure-key-vault-data-collection-details"></a>檢閱 Azure 金鑰保存庫資料收集的詳細資料
Azure Key Vault 解決方案會直接從 Key Vault 收集診斷記錄。
不需要將記錄寫入 Azure Blob 儲存體，也不需要代理程式來收集資料。

下表顯示 Azure 金鑰保存庫的資料收集方法及如何收集資料的其他詳細資料。

| 平台 | 直接代理程式 | Systems Center Operations Manager 代理程式 | Azure | 是否需要 Operations Manager？ | 透過管理群組傳送的 Operations Manager 代理程式資料 | 收集頻率 |
| --- | --- | --- | --- | --- | --- | --- |
| Azure |  |  |&#8226; |  |  | 與抵達同時 |

## <a name="use-azure-key-vault"></a>使用 Azure 金鑰保存庫
在您[安裝解決方案](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.KeyVaultAnalyticsOMS?tab=Overview)之後，按一下 Log Analytics [概觀] 頁面的 [Azure Key Vault] 圖格來檢視金鑰保存庫資料。

![Azure 金鑰保存庫圖格的影像](./media/log-analytics-azure-keyvault/log-analytics-keyvault-tile.png)

按一下 [概觀] 圖格之後，您可以檢視記錄檔的摘要，然後深入下列類別的詳細資訊︰

* 經過一段時間的所有金鑰保存庫作業的數量
* 經過一段時間的失敗作業數量
* 依作業的平均作業延遲
* 作業的服務品質，顯示花費超過 1000 毫秒的作業數目及花費超過 1000 毫秒的作業清單

![Azure 金鑰保存庫儀表板的影像](./media/log-analytics-azure-keyvault/log-analytics-keyvault01.png)

![Azure 金鑰保存庫儀表板的影像](./media/log-analytics-azure-keyvault/log-analytics-keyvault02.png)

### <a name="to-view-details-for-any-operation"></a>檢視任何作業的詳細資料
1. 在 [概觀] 頁面上，按一下 [Azure 金鑰保存庫] 圖格。
2. 在 [Azure 金鑰保存庫] 儀表板上，檢閱其中一個刀鋒視窗中的摘要資訊，然後按一下其中一個，在記錄搜尋頁面中檢視詳細資訊。

    您可以在任何 [記錄搜尋] 頁面上，按時間、詳細結果和您的記錄搜尋記錄來檢視結果。 您也可以按 Facet 篩選以縮減結果。

## <a name="log-analytics-records"></a>Log Analytics 記錄
Azure 金鑰保存庫解決方案會分析從 Azure 診斷的 [AuditEvent 記錄檔](../key-vault/key-vault-logging.md)收集的 **KeyVaults** 類型記錄。  下表是這些記錄的屬性：  

| 屬性 | 說明 |
|:--- |:--- |
| 類型 |*AzureDiagnostics* |
| SourceSystem |*Azure* |
| CallerIpAddress |提出要求之用戶端的 IP 位址 |
| 類別 | *AuditEvent* |
| CorrelationId |選擇性的 GUID，用戶端可傳遞此 GUID 來讓用戶端記錄與服務端 (金鑰保存庫) 記錄相互關聯。 |
| DurationMs |服務 REST API 要求時所花費的時間，以毫秒為單位。 這個時間不包括網路延遲，因此在用戶端所測量到的時間可能不符合此時間。 |
| httpStatusCode_d |由要求傳回的 HTTP 狀態碼 (例如 *200*) |
| id_s |要求的唯一識別碼 |
| identity_claim_appid_g | 應用程式識別碼的 GUID |
| OperationName |[Azure 金鑰保存庫記錄](../key-vault/key-vault-logging.md)中所記載的作業名稱 |
| OperationVersion |用戶端所要求的 REST API 版本 (例如 *2015-06-01*) |
| requestUri_s |要求的 Uri |
| 資源 |金鑰保存庫的名稱 |
| ResourceGroup |金鑰保存庫的資源群組 |
| ResourceId |Azure 資源管理員資源識別碼。 對於 Key Vault 記錄來說，這是 Key Vault 的資源識別碼。 |
| ResourceProvider |*MICROSOFT.KEYVAULT* |
| ResourceType | *VAULTS* |
| ResultSignature |HTTP 狀態 (例如 *OK*) |
| ResultType |REST API 要求的結果 (例如 *Success*) |
| SubscriptionId |包含金鑰保存庫的訂用帳戶的 Azure 訂用帳戶識別碼 |

## <a name="migrating-from-the-old-key-vault-solution"></a>從舊的 Key Vault 解決方案進行移轉
從 2017 年 1 月開始，從 Key Vault 傳送記錄到 Log Analytics 的支援方式已變更。 這些變更可提供下列優點︰
+ 記錄會直接寫入 Log Analytics，而不需要使用儲存體帳戶
+ 當 Log Analytics 中具有產生的記錄時，延遲會變得較低
+ 較少的組態步驟
+ 所有 Azure 診斷類型的通用格式

若要使用更新的解決方案︰

1. [將診斷設定為直接從 Key Vault 傳送到 Log Analytics](#enable-key-vault-diagnostics-in-the-portal)  
2. 使用[從方案庫新增 Log Analytics 解決方案](log-analytics-add-solutions.md)中所述的程序，啟用 Azure Key Vault 解決方案
3. 更新任何已儲存的查詢、儀表板或警示，以使用新的資料類型
  + 類型從 KeyVaults 變更為 AzureDiagnostics。 您可以使用 ResourceType 篩選 Key Vault 記錄。
  - 與其使用 `KeyVaults`，請改用 `AzureDiagnostics | where ResourceType'=="VAULTS"`
  + 欄位：(欄位名稱區分大小寫)
  - 針對任何名稱尾碼有 \_s、\_d 或 \_g 的欄位，請將第一個字元變更為小寫
  - 針對任何名稱尾碼有 \_o 的欄位，資料會根據巢狀欄位名稱分割為個別欄位。 例如，呼叫端的 UPN 會儲存在欄位 `identity_claim_http_schemas_xmlsoap_org_ws_2005_05_identity_claims_upn_s`
   - 欄位 CallerIpAddress 變更為 CallerIPAddress
   - 欄位 RemoteIPCountry 已不存在
4. 移除 *Key Vault 分析 (已過時)* 解決方案。 如果您是使用 PowerShell，請使用 `Set-AzureOperationalInsightsIntelligencePack -ResourceGroupName <resource group that the workspace is in> -WorkspaceName <name of the log analytics workspace> -IntelligencePackName "KeyVault" -Enabled $false`

在變更之前所收集的資料不會顯示在新的解決方案中。 您可以繼續使用舊的類型和欄位名稱查詢此資料。

## <a name="troubleshooting"></a>疑難排解
[!INCLUDE [log-analytics-troubleshoot-azure-diagnostics](../../includes/log-analytics-troubleshoot-azure-diagnostics.md)]

## <a name="next-steps"></a>後續步驟
* 使用 [Log Analytics 中的記錄搜尋](log-analytics-log-searches.md)來檢視詳細的 Azure 金鑰保存庫資料。
