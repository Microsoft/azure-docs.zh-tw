---
title: 使用 Log Analytics 收集 Azure PaaS 資源計量 | Microsoft Docs
description: 了解如何使用 PowerShell 來啟用 Azure PaaS 資源計量收集，以便在 Log Analytics 中保留和分析。
services: log-analytics
documentationcenter: log-analytics
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 11/13/2017
ms.author: magoedte
ms.component: na
ms.openlocfilehash: b44a6627ab12c8a4ad21e7beded7c5fd2c2e1d39
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37128457"
---
# <a name="configure-collection-of-azure-paas-resource-metrics-with-log-analytics"></a>使用 Log Analytics 設定 Azure PaaS 資源計量的收集

Azure 平台即服務 (PaaS) 資源 (例如 Azure SQL 和網站 (Web 應用程式)) 原本就會將效能計量資料發出至 Log Analytics。 此指令碼可讓使用者對已部署於特定資源群組或整個訂用帳戶的 PaaS 資源，啟用計量記錄功能。 

目前，沒有任何方法可透過 Azure 入口網站來對 PaaS 資源啟用計量記錄功能。 因此，您需要使用 PowerShell 指令碼。 這項原生計量記錄功能以及 Log Analytics 監視功能，可讓您大規模監視 Azure 資源。 

## <a name="prerequisites"></a>先決條件
先確認您已在電腦上安裝下列 Azure Resource Manager 模組，再繼續執行：

- AzureRM.Insights
- AzureRM.OperationalInsights
- AzureRM.Resources
- AzureRM.profile

>[!NOTE]
>我們建議所有的 Azure Resource Manager 模組都是相同的版本，以確保您從 PowerShell 執行 Azure Resource Manager 命令時的相容性。
>
若要在電腦上安裝最新版的 Azure Resource Manager 模組，請參閱[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-4.4.1#update-azps)。  

## <a name="enable-azure-diagnostics"></a>啟用 Azure 診斷  
執行指令碼 **Enable-AzureRMDiagnostics.ps1** (可從 [PowerShell 資源庫](https://www.powershellgallery.com/packages/Enable-AzureRMDiagnostics/2.52/DisplayScript)取得)，即可完成 PaaS 資源的 Azure 診斷設定。  此指令碼支援下列案例：
  
* 指定訂用帳戶中一或多個資源群組的相關資源  
* 指定訂用帳戶中特定資源群組的相關資源  
* 重新設定資源以轉送到不同的工作區

只有支援使用 Azure 診斷來收集計量並直接傳送到 Log Analytics 的資源會受到支援。  如需詳細的清單，請檢閱[收集 Azure 服務的記錄和計量以便使用於 Log Analytics](log-analytics-azure-storage.md) 

執行下列步驟，下載並執行指令碼。

1.  在 Windows [開始] 畫面中，輸入 **PowerShell** 並以滑鼠右鍵按一下搜尋結果中的 PowerShell。  從功能表中選取 [以系統管理員身分執行]。   
2. 執行下列命令並提供可儲存指令碼的路徑，以在本機儲存 **Enable-AzureRMDiagnostics.ps1** 指令碼檔案。    

    ```
    PS C:\> save-script -Name Enable-AzureRMDiagnostics -Path "C:\users\<username>\desktop\temp"
    ```

3. 執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。   
4. 執行下列指令碼 `.\Enable-AzureRmDiagnostics.ps1` (不使用任何參數) 來啟用從您訂用帳戶中的特定資源收集資料，，或使用參數 `-ResourceGroup <myResourceGroup>` 來指定特定資源群組中的資源。   
5. 如果您有多個訂用帳戶，請輸入正確的值，以便從清單中選取適當的訂用帳戶。<br><br> ![選取指令碼所傳回的訂用帳戶](./media/log-analytics-collect-azurepass-posh/script-select-subscription.png)<br> 否則，它會自動選取可用的單一訂用帳戶。
6. 接下來，指令碼會傳回訂用帳戶中已註冊的 Log Analytics 工作區清單。  從清單中選取適當的工作區。<br><br> ![選取指令碼所傳回的工作區](./media/log-analytics-collect-azurepass-posh/script-select-workspace.png)<br> 
7. 選取您想要啟用收集的 Azure 資源。 例如，如果您輸入 5，您會啟用 SQL Azure Database 的資料收集。<br><br> ![選取指令碼所傳回的資源類型](./media/log-analytics-collect-azurepass-posh/script-select-resource.png)<br>
   您只能選取支援使用 Azure 診斷來收集計量並直接傳送到 Log Analytics 的資源。  此指令碼會針對它在您的訂用帳戶或指定的資源群組中找到的資源清單，在 [計量] 資料行之下顯示 **True** 值。    
8. 系統會提示您確認您的選取項目。  輸入 **Y** 以針對已定義範圍的所有已選資源 (在我們的範例中是訂用帳戶中的所有 SQL 資料庫) 啟用計量記錄功能。  

此指令碼會針對符合所選準則的每個資源執行，並為其啟用計量收集。 完成之後，您會看到一則訊息，指出已完成設定。  

完成之後，您很快就會開始在您的 Log Analytics 存放庫中看到來自 Azure PaaS 資源的資料。  系統會建立 `AzureMetrics` 類型的記錄，而分析這些記錄則是由 [Azure SQL 分析](log-analytics-azure-sql.md)和[Azure Web Apps 分析](log-analytics-azure-web-apps-analytics.md)管理解決方案支援。   

## <a name="update-a-resource-to-send-data-to-another-workspace"></a>更新資源以將資料傳送到另一個工作區
如果您擁有已將資料傳送到 Log Analytics 工作區的資源，而且稍後決定將它重新設定為參考另一個工作區，您可以使用 `-Update` 參數來執行指令碼。  

**範例：** 
`PS C:\users\<username>\Desktop\temp> .\Enable-AzureRMDiagnostics.ps1 -Update`

系統會提示您回答在您執行指令碼來執行初始設定時的相同資訊。  

## <a name="next-steps"></a>後續步驟

* 了解 [記錄搜尋](log-analytics-log-searches.md) ，其可分析從資料來源和方案所收集的資料。 

* 使用[自訂欄位](log-analytics-custom-fields.md) ，將事件記錄剖析至個別欄位。

* 檢閱[建立自訂儀表板以便在 Log Analytics 中使用](log-analytics-dashboards.md)，了解如何以對組織有意義的方式來呈現您的記錄搜尋。
