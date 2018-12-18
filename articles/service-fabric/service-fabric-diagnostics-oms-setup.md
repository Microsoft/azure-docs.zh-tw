---
title: Azure Service Fabric - 使用 Log Analytics 設定監視 | Microsoft Docs
description: 了解如何設定 Log Analytics 對事件進行視覺化及分析，以供監視 Azure Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: srrengar
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 9/11/2018
ms.author: srrengar
ms.openlocfilehash: a73a288852eea713623b65324853761e10fad282
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220460"
---
# <a name="set-up-log-analytics-for-a-cluster"></a>為叢集設定 Log Analytics

建議您使用 Log Analytics 來監視叢集層級事件。 您可以透過 Azure Resource Manager、PowerShell 或 Azure Marketplace 來設定 Log Analytics 工作區。 如果要維護部署的 Resource Manager 更新範本以供未來使用，請使用相同的範本來設定您的 Log Analytics 環境。 如果已經在啟用診斷的情況下部署叢集，則透過 Marketplace 部署會更輕鬆。 如果您要用來部署的帳戶中沒有訂用帳戶層級的存取權，請使用 PowerShell 或 Resource Manager 範本進行部署。

> [!NOTE]
> 若要設定 Log Analytics 來監視您的叢集，您需要啟用診斷，才能檢視叢集層級或平台層級事件。 請參閱[如何在 Windows 叢集中設定診斷功能](service-fabric-diagnostics-event-aggregation-wad.md)和[如何在 Linux 叢集中設定診斷功能](service-fabric-diagnostics-event-aggregation-lad.md)，以取得詳細資訊

## <a name="deploy-a-log-analytics-workspace-by-using-azure-marketplace"></a>使用 Azure Marketplace 部署 Log Analytics 工作區

如果您想要在部署叢集之後新增 Log Analytics 工作區，請在入口網站中前往 Azure Marketplace，並尋找 **Service Fabric 分析**。 這是具有 Service Fabric 特有資料且適用於 Service Fabric 部署的自訂解決方案。 在此程序中，您將建立解決方案 (用來檢視深入解析的儀表板) 和工作區 (基礎叢集資料的彙總)。

1. 在左側導覽功能表上選取 [新增]。 

2. 搜尋 **Service Fabric 分析**。 請選取顯示的資源。

3. 選取 [建立] 。

    ![Marketplace 中的 Service Fabric 分析](media/service-fabric-diagnostics-event-analysis-oms/service-fabric-analytics.png)

4. 在 Service Fabric 分析建立視窗中，針對 [OMS 工作區] 欄位選取 [選取工作區]，然後按一下 [建立新的工作區]。 填寫必要的項目。 這裡唯一的需求是 Service Fabric 叢集與工作區的訂用帳戶必須相同。 所有項目都經過驗證後，您的工作區就會開始部署。 部署只需要幾分鐘的時間。

5. 完成時，再次選取 Service Fabric 分析建立視窗底部的 [建立]。 請確認新的工作區顯示在 [OMS 工作區] 底下。 此動作會將解決方案新增至您建立的工作區。

如果使用的是 Windows，請繼續進行下列步驟，以將 Log Analytics 連線到儲存叢集事件所在的儲存體帳戶。 

>[!NOTE]
>目前尚未對 Linux 叢集實施此體驗。 

### <a name="connect-the-log-analytics-workspace-to-your-cluster"></a>將 Log Analytics 工作區連線至叢集 

1. 工作區需要連線到來自叢集的診斷資料。 移至您建立 Service Fabric 分析解決方案所在的資源群組。 選取 **ServiceFabric\<nameOfWorkspace\>**，並移至其概觀頁面。 您可以從那裡變更解決方案設定、工作區設定，並存取 Log Analytics 工作區。

2. 在左側導覽功能表，選取 [工作區資料來源] 底下的 [儲存體帳戶記錄]。

3. 在 [儲存體帳戶記錄] 分頁上，選取頂端的 [新增] 以將您的叢集記錄新增至工作區。

4. 選取 [儲存體帳戶] 以新增叢集中建立的適當帳戶。 如果您使用了預設名稱，則儲存體帳戶為 **sfdg\<resourceGroupName\>**。 藉由檢查用於 **applicationDiagnosticsStorageAccountName** 的值，也可以透過用來部署叢集的 Azure Resource Manager 範本來確認這點。 如果未顯示此名稱，請向下捲動並選取 [載入更多]。 選取儲存體帳戶名稱。

5. 指定 [資料類型]。 將它設定為 [Service Fabric 事件]。

6. 確保來源自動設定為 **WADServiceFabric\*EventTable**。

7. 選取 [確定] 以將您的工作區連線至您的叢集記錄。

    ![將儲存體帳戶記錄新增至 Log Analytics](media/service-fabric-diagnostics-event-analysis-oms/add-storage-account.png)

帳戶現在會在工作區資料來源中顯示成為儲存體帳戶記錄的一部分。

您已在 Log Analytics 工作區中新增 Service Fabric 分析解決方案，該工作區現在已正確連線至叢集平台和應用程式記錄資料表。 您可以相同方式將其他來源新增至工作區。


## <a name="deploy-log-analytics-by-using-a-resource-manager-template"></a>使用 Resource Manager 範本部署 Log Analytics

使用 Resource Manager 範本部署叢集時，該範本會建立新的 Log Analytics 工作區、將 Service Fabric 解決方案新增至此工作區，並將它設定為讀取適當儲存體資料表中的資料。

您可以使用和修改[這個範例範本](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-OMS-UnSecure)以符合您的需求。 此範本將會執行下列動作：

* 建立 5 個節點的 Service Fabric 叢集
* 建立 Log Analytics 工作區與 Service Fabric 解決方案
* 設定 OMS 代理程式以收集並傳送 2 個範例效能計數器到工作區
* 設定 WAD 以收集 Service Fabric 並將其傳送到 Azure 儲存體資料表 (WADServiceFabric*EventTable)
* 設定 Log Analytics 工作區以從這些資料表讀取事件


您可以在 AzureRM PowerShell 模組中使用 `New-AzureRmResourceGroupDeployment` API，以 Resource Manager 升級的方式將範本部署至您的叢集。 範例命令可能像這樣：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName "<resourceGroupName>" -TemplateFile "<templatefile>.json" 
``` 

Azure Resource Manager 會偵測到此命令是現有資源的更新。 它只會處理驅動現有部署之範本和提供之新範本之間的變更。

## <a name="deploy-log-analytics-by-using-azure-powershell"></a>使用 Azure PowerShell 部署 Log Analytics

您也可以使用 `New-AzureRmOperationalInsightsWorkspace` 命令，透過 PowerShell 部署 Log Analytics 資源。 若要使用此方法，請確定您已安裝 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-5.1.1)。 請使用此指令碼建立新的 Log Analytics 工作區，並將 Service Fabric 解決方案新增至該工作區： 

```PowerShell

$SubscriptionName = "<Name of your subscription>"
$ResourceGroup = "<Resource group name>"
$Location = "<Resource group location>"
$WorkspaceName = "<Log Analytics workspace name>"
$solution = "ServiceFabric"

# Log in to Azure and access the correct subscription
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionId $SubID 

# Create the resource group if needed
try {
    Get-AzureRmResourceGroup -Name $ResourceGroup -ErrorAction Stop
} catch {
    New-AzureRmResourceGroup -Name $ResourceGroup -Location $Location
}

New-AzureRmOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -Sku Standard -ResourceGroupName $ResourceGroup
Set-AzureRmOperationalInsightsIntelligencePack -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -IntelligencePackName $solution -Enabled $true

```

當您完成時，請按照前一節的步驟將 Log Analytics 連線到適當的儲存體帳戶。

您也可以新增其他解決方案，或使用 PowerShell 對 Log Analytics 工作區進行其他修改。 若要深入了解，請參閱[使用 PowerShell 管理 Log Analytics](../log-analytics/log-analytics-powershell-workspace-configuration.md)。

## <a name="next-steps"></a>後續步驟
* [部署 Log Analytics 代理程式](service-fabric-diagnostics-oms-agent.md)至您的節點，以收集效能計數器，並收集您容器的 Docker 統計資料和記錄
* 熟悉 Log Analytics 的[記錄搜尋和查詢](../log-analytics/log-analytics-log-searches.md)功能
* [在 Log Analytics 中使用檢視設計工具來建立自訂檢視](../log-analytics/log-analytics-view-designer.md)
