---
title: Log Analytics 中的 Azure 網路分析解決方案 | Microsoft Docs
description: 您可以使用 Log Analytics 中的 Azure 網路分析解決方案，以檢閱 Azure 網路安全性群組記錄檔和 Azure 應用程式閘道記錄檔。
services: log-analytics
documentationcenter: ''
author: richrundmsft
manager: ewinner
editor: ''
ms.assetid: 66a3b8a1-6c55-4533-9538-cad60c18f28b
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/21/2018
ms.author: richrund
ms.component: na
ms.openlocfilehash: 8a92bf7b031899ee75fbf2bb2fdfd7dced3bc1ad
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38481985"
---
# <a name="azure-networking-monitoring-solutions-in-log-analytics"></a>Log Analytics 中的 Azure 網路監視解決方案

Log Analytics 提供下列解決方案來監視您的網路︰
* 網路效能監視器 (NPM) 以
 * 監視網路的健康狀態
* 要檢閱的 Azure 應用程式閘道分析
 * Azure 應用程式閘道記錄檔
 * Azure 應用程式閘道計量
* 監視和稽核雲端網路上網路活動的解決方案
* [流量分析](https://docs.microsoft.com/azure/networking/network-monitoring-overview#traffic-analytics) 
* Azure 網路安全性群組分析

## <a name="network-performance-monitor-npm"></a>網路效能監視器 (NPM)

[網路效能監視器](https://docs.microsoft.com/azure/networking/network-monitoring-overview)管理解決方案是網路監視解決方案，可監視網路的健康狀態、可用性和連線能力。  可用來監視下列項目之間的連線能力︰

* 公用雲端與內部部署環境
* 資料中心與使用者地點 (分公司)
* 裝載多層式應用程式各層的子網路。

如需詳細資訊，請參閱[網路效能監視器](https://docs.microsoft.com/azure/networking/network-monitoring-overview)。

## <a name="azure-application-gateway-and-network-security-group-analytics"></a>Azure 應用程式閘道和網路安全性群組分析
若要使用解決方案：
1. 將管理解決方案新增至 Log Analytics，以及
2. 啟用診斷功能以將診斷導向 Log Analytics 工作區。 不需要將記錄寫入 Azure Blob 儲存體。

您可以針對應用程式閘道和網路安全性群組其中之一 (或兩者) 啟用診斷與對應的解決方案。

如果您沒有針對特定資源類型啟用診斷記錄，但是安裝了解決方案，該資源的儀表板刀鋒視窗會是空白，並顯示一則錯誤訊息。

> [!NOTE]
> 從 2017 年 1 月開始，從應用程式閘道和網路安全性群組傳送記錄到 Log Analytics 的支援方式已變更。 如果您看到 **Azure 網路分析 (已過時)** 解決方案，請參閱[從舊的網路分析解決方案進行移轉](#migrating-from-the-old-networking-analytics-solution)，以取得您必須遵循的步驟。
>
>

## <a name="review-azure-networking-data-collection-details"></a>檢閱 Azure 網路資料集合詳細資料
Azure 應用程式閘道分析和網路安全性群組分析管理解決方案，會直接從 Azure 應用程式閘道和網路安全性群組收集診斷記錄。 不需要將記錄寫入 Azure Blob 儲存體，也不需要代理程式來收集資料。

下表顯示資料收集方法，與其他有關如何針對 Azure 應用程式閘道分析和網路安全性群組分析收集資料的詳細資料。

| 平台 | 直接代理程式 | Systems Center Operations Manager 代理程式 | Azure | 是否需要 Operations Manager？ | 透過管理群組傳送的 Operations Manager 代理程式資料 | 收集頻率 |
| --- | --- | --- | --- | --- | --- | --- |
| Azure |  |  |&#8226; |  |  |登入時 |


## <a name="azure-application-gateway-analytics-solution-in-log-analytics"></a>Log Analytics 中的 Azure 應用程式閘道分析解決方案

![Azure 應用程式閘道分析符號](./media/log-analytics-azure-networking/azure-analytics-symbol.png)

應用程式閘道支援下列記錄檔︰

* ApplicationGatewayAccessLog
* ApplicationGatewayPerformanceLog
* ApplicationGatewayFirewallLog

應用程式閘道支援下列計量︰again


* 5 分鐘輸送量

### <a name="install-and-configure-the-solution"></a>安裝和設定解決方案
使用下列指示來安裝和設定 Azure 應用程式閘道分析解決方案：

1. 從 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureAppGatewayAnalyticsOMS?tab=Overview) 或使用[從方案庫新增 Log Analytics 方案](log-analytics-add-solutions.md)中所述的程序，啟用 Azure 應用程式閘道分析解決方案。
2. 針對您想要監視的[應用程式閘道](../application-gateway/application-gateway-diagnostics.md)啟用診斷記錄。

#### <a name="enable-azure-application-gateway-diagnostics-in-the-portal"></a>在入口網站中啟用 Azure 應用程式閘道診斷

1. 在 Azure 入口網站中，瀏覽至要監視的應用程式閘道資源
2. 選取 [診斷記錄] 以開啟下列頁面

   ![Azure 應用程式閘道資源的影像](./media/log-analytics-azure-networking/log-analytics-appgateway-enable-diagnostics01.png)
3. 按一下 [開啟診斷] 以開啟下列頁面

   ![Azure 應用程式閘道資源的影像](./media/log-analytics-azure-networking/log-analytics-appgateway-enable-diagnostics02.png)
4. 若要開啟診斷，請按一下 [狀態] 下的 [開啟]
5. 按一下 [傳送到 Log Analytics] 核取方塊
6. 選取現有的 Log Analytics 工作區，或建立工作區
7. 針對每一個要收集的記錄類型，按一下 [記錄] 下的核取方塊
8. 按一下 [儲存] 以啟用 Log Analytics 的診斷記錄

#### <a name="enable-azure-network-diagnostics-using-powershell"></a>使用 PowerShell 啟用 Azure 網路診斷

下列 PowerShell 指令碼示範如何啟用應用程式閘道診斷記錄的範例。

```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$gateway = Get-AzureRmApplicationGateway -Name 'ContosoGateway'

Set-AzureRmDiagnosticSetting -ResourceId $gateway.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

### <a name="use-azure-application-gateway-analytics"></a>使用 Azure 應用程式閘道分析
![Azure 應用程式閘道分析圖格的影像](./media/log-analytics-azure-networking/log-analytics-appgateway-tile.png)

在您按一下 [概觀] 上的 [Azure 應用程式閘道分析] 圖格之後，您可以檢視記錄摘要，然後深入探索下列類別的詳細資訊：

* 應用程式閘道存取記錄檔
  * 應用程式閘道存取記錄檔的用戶端和伺服器錯誤
  * 每個應用程式閘道的每小時要求數
  * 每個應用程式閘道的每小時失敗要求數
  * 應用程式閘道依使用者代理程式分類的錯誤
* 應用程式閘道效能
  * 應用程式閘道的主機健康狀態
  * 應用程式閘道失敗要求的最大和第 95 個百分位數

![Azure 應用程式閘道分析儀表板的影像](./media/log-analytics-azure-networking/log-analytics-appgateway01.png)

![Azure 應用程式閘道分析儀表板的影像](./media/log-analytics-azure-networking/log-analytics-appgateway02.png)

在 [Azure 應用程式閘道分析] 儀表板上，檢閱其中一個刀鋒視窗中的摘要資訊，然後按一下其中一個以在記錄搜尋頁面中檢視詳細資訊。

您可以在任何 [記錄搜尋] 頁面上，按時間、詳細結果和您的記錄搜尋記錄來檢視結果。 您也可以按 Facet 篩選以縮減結果。


## <a name="azure-network-security-group-analytics-solution-in-log-analytics"></a>Log Analytics 中的 Azure 網路安全性群組分析解決方案

![Azure 網路安全性群組分析符號](./media/log-analytics-azure-networking/azure-analytics-symbol.png)

> [!NOTE]
> 網路安全性群組分析解決方案將會移至社群支援，因為其功能已由[流量分析](../network-watcher/traffic-analytics.md)取代。
> - 解決方案現已在 [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/oms-azurensg-solution/)中提供，而且將很快不再於 Azure Marketplace 中提供。
> - 對於已將此解決方案新增至其工作區的現有客戶，它會繼續運作而不需要變更。
> - Microsoft 將繼續支援使用診斷設定，將 NSG 診斷記錄傳送至您的工作區。

網路安全性群組支援下列記錄檔︰

* NetworkSecurityGroupEvent
* NetworkSecurityGroupRuleCounter

### <a name="install-and-configure-the-solution"></a>安裝和設定解決方案
使用下列指示來安裝和設定 Azure 網路分析解決方案︰

1. 從 [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.AzureNSGAnalyticsOMS?tab=Overview) 或使用[從方案庫新增 Log Analytics 方案](log-analytics-add-solutions.md)中所述的程序，啟用 Azure 網路安全性群組分析解決方案。
2. 針對您想要監視的[網路安全性群組](../virtual-network/virtual-network-nsg-manage-log.md)資源啟用診斷記錄。

### <a name="enable-azure-network-security-group-diagnostics-in-the-portal"></a>在入口網站中啟用 Azure 網路安全性群組診斷

1. 在 Azure 入口網站中，瀏覽至要監視的網路安全性群組資源
2. 選取 [診斷記錄] 以開啟下列頁面

   ![Azure 網路安全性群組資源的影像](./media/log-analytics-azure-networking/log-analytics-nsg-enable-diagnostics01.png)
3. 按一下 [開啟診斷] 以開啟下列頁面

   ![Azure 網路安全性群組資源的影像](./media/log-analytics-azure-networking/log-analytics-nsg-enable-diagnostics02.png)
4. 若要開啟診斷，請按一下 [狀態] 下的 [開啟]
5. 按一下 [傳送到 Log Analytics] 核取方塊
6. 選取現有的 Log Analytics 工作區，或建立工作區
7. 針對每一個要收集的記錄類型，按一下 [記錄] 下的核取方塊
8. 按一下 [儲存] 以啟用 Log Analytics 的診斷記錄

### <a name="enable-azure-network-diagnostics-using-powershell"></a>使用 PowerShell 啟用 Azure 網路診斷

下列 PowerShell 指令碼提供如何啟用網路安全性群組診斷記錄的範例
```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$nsg = Get-AzureRmNetworkSecurityGroup -Name 'ContosoNSG'

Set-AzureRmDiagnosticSetting -ResourceId $nsg.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

### <a name="use-azure-network-security-group-analytics"></a>使用 Azure 網路安全性群組分析
在您按一下 [概觀] 上的 [Azure 網路安全性群組分析] 圖格之後，您可以檢視記錄摘要，然後深入探索下列類別的詳細資訊：

* 網路安全性群組封鎖流量
  * 網路安全性群組規則與封鎖流量
  * MAC 位址與封鎖流量
* 網路安全性群組允許流量
  * 網路安全性群組規則與允許流量
  * MAC 位址與允許流量

![Azure 網路安全性群組分析儀表板的影像](./media/log-analytics-azure-networking/log-analytics-nsg01.png)

![Azure 網路安全性群組分析儀表板的影像](./media/log-analytics-azure-networking/log-analytics-nsg02.png)

在 [Azure 網路安全性群組分析] 儀表板上，檢閱其中一個刀鋒視窗中的摘要資訊，然後按一下其中一個以在記錄搜尋頁面中檢視詳細資訊。

您可以在任何 [記錄搜尋] 頁面上，按時間、詳細結果和您的記錄搜尋記錄來檢視結果。 您也可以按 Facet 篩選以縮減結果。

## <a name="migrating-from-the-old-networking-analytics-solution"></a>從舊的網路分析解決方案進行移轉
從 2017 年 1 月開始，從 Azure 應用程式閘道和 Azure 網路安全性群組傳送記錄到 Log Analytics 的支援方式已變更。 這些變更可提供下列優點︰
+ 記錄會直接寫入 Log Analytics，而不需要使用儲存體帳戶
+ 當 Log Analytics 中具有產生的記錄時，延遲會變得較低
+ 較少的組態步驟
+ 所有 Azure 診斷類型的通用格式

若要使用更新的解決方案：

1. [將診斷設定為直接從 Azure 應用程式閘道傳送到 Log Analytics](#enable-azure-application-gateway-diagnostics-in-the-portal)
2. [將診斷設定為直接從 Azure 網路安全性群組傳送到 Log Analytics](#enable-azure-network-security-group-diagnostics-in-the-portal)
2. 使用[從方案庫新增 Log Analytics 解決方案](log-analytics-add-solutions.md)中所述的程序，啟用「Azure 應用程式閘道分析」和「Azure 網路安全性群組分析」解決方案
3. 更新任何已儲存的查詢、儀表板或警示，以使用新的資料類型
  + 類型是 AzureDiagnostics。 您可以使用 ResourceType 來篩選 Azure 網路記錄。

    | 不要使用： | 使用︰ |
    | --- | --- |
    | NetworkApplicationgateways &#124; 其中 OperationName=="ApplicationGatewayAccess" | AzureDiagnostics &#124; 其中 ResourceType="APPLICATIONGATEWAYS" 且 OperationName=="ApplicationGatewayAccess" |
    | NetworkApplicationgateways &#124; 其中 OperationName=="ApplicationGatewayPerformance" | AzureDiagnostics &#124; 其中 ResourceType=="APPLICATIONGATEWAYS" 且 OperationName=ApplicationGatewayPerformance |
    | NetworkSecuritygroups | AzureDiagnostics &#124; 其中 ResourceType=="NETWORKSECURITYGROUPS" |

   + 針對任何名稱尾碼有 \_s、\_d 或 \_g 的欄位，請將第一個字元變更為小寫
   + 針對任何名稱尾碼有 \_o 的欄位，資料會根據巢狀欄位名稱分割為個別欄位。
4. 移除 *Azure 網路分析 (已過時)* 解決方案。
  + 如果您是使用 PowerShell，請使用 `Set-AzureOperationalInsightsIntelligencePack -ResourceGroupName <resource group that the workspace is in> -WorkspaceName <name of the log analytics workspace> -IntelligencePackName "AzureNetwork" -Enabled $false`

在變更之前所收集的資料不會顯示在新的解決方案中。 您可以繼續使用舊的類型和欄位名稱查詢此資料。

## <a name="troubleshooting"></a>疑難排解
[!INCLUDE [log-analytics-troubleshoot-azure-diagnostics](../../includes/log-analytics-troubleshoot-azure-diagnostics.md)]

## <a name="next-steps"></a>後續步驟
* 使用 [Log Analytics 中的記錄搜尋](log-analytics-log-searches.md)檢視詳細的 Azure 診斷資料。
