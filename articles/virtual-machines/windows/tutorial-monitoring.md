---
title: 教學課程 - 在 Azure 中監視和更新 Windows 虛擬機器 | Microsoft Docs
description: 在本教學課程中，您會了解如何監視 Windows 虛擬機器上的開機診斷和效能計量及管理套件更新
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 05/04/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 0f2388747d60f1911994431d8edbd57e32d5c35a
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37932648"
---
# <a name="tutorial-monitor-and-update-a-windows-virtual-machine-in-azure"></a>教學課程：在 Azure 中監視和更新 Windows 虛擬機器

Azure 監視器使用代理程式從 Azure VM 收集開機和效能資料，將此資料儲存在 Azure 儲存體，並讓資料可透過入口網站、Azure PowerShell 模組和 Azure CLI 存取。 更新管理可讓您管理 Azure Windows VM 的更新和修補程式。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 啟用 VM 上的開機診斷
> * 檢視開機診斷
> * 檢視 VM 主機計量
> * 安裝診斷擴充功能
> * 檢視 VM 計量
> * 建立警示
> * 管理 Windows 更新
> * 監視變更和清查
> * 設定進階監視

本教學課程需要 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

## <a name="create-virtual-machine"></a>建立虛擬機器

若要在本教學課程中設定 Azure 監視和更新管理，您需要在 Azure 中擁有 Windows VM。 首先，使用 [Get-credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 設定 VM 的系統管理員使用者名稱和密碼：

```azurepowershell-interactive
$cred = Get-Credential
```

現在，使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM。 下列範例會在 *EastUS* 位置中建立名為 *myVM* 的 VM。 如果它們尚不存在，建立支援網路資源的資源群組 *myResourceGroupMonitorMonitor*：

```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName "myResourceGroupMonitor" `
    -Name "myVM" `
    -Location "East US" `
    -Credential $cred
```

系統需要花幾分鐘的時間來建立資源和 VM。

## <a name="view-boot-diagnostics"></a>檢視開機診斷

Windows 虛擬機器開機時，開機診斷代理程式會擷取可用於疑難排解的螢幕輸出。 此功能預設為啟用狀態。 擷取的螢幕畫面會儲存在 Azure 儲存體帳戶，這也是預設會建立的帳戶。

您可以使用 [Get-AzureRmVMBootDiagnosticsData](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmbootdiagnosticsdata) 命令取得開機診斷資料。 在下列範例中，開機診斷會下載到 *c:\* 磁碟機的根目錄。

```powershell
Get-AzureRmVMBootDiagnosticsData -ResourceGroupName "myResourceGroupMonitor" -Name "myVM" -Windows -LocalPath "c:\"
```

## <a name="view-host-metrics"></a>檢視主機計量

在 Azure 中有 Windows VM 專用的主機 VM 與它互動。 系統會自動收集主機的計量，而且可以在 Azure 入口網站中檢視這些計量。

1. 從 Azure 入口網站中，按一下 [資源群組]，選取 [myResourceGroupMonitor]，然後選取資源清單中的 [myVM]。
2. 按一下 VM 刀鋒視窗上的 [計量]，然後選取 [可用的計量] 下的任何 [主機] 計量以查看主機 VM 的執行狀況。

    ![檢視主機計量](./media/tutorial-monitoring/tutorial-monitor-host-metrics.png)

## <a name="install-diagnostics-extension"></a>安裝診斷擴充功能

系統提供基本的主機計量，但若要查看更細微或 VM 特定的計量，則需要在 VM 上安裝 Azure 診斷的擴充功能。 Azure 診斷擴充功能可額外提供從 VM 擷取的監視和診斷資料。 您可以檢視這些效能計量，並依據 VM 的執行狀況建立警示。 診斷擴充功能可透過 Azure 入口網站安裝，如下所示︰

1. 從 Azure 入口網站中，按一下 [資源群組]，選取 [myResourceGroupMonitor]，然後選取資源清單中的 [myVM]。
2. 按一下 [診斷設定]。 清單會顯示在上一節已啟用開機診斷。 按一下 [基本計量] 的核取方塊。
3. 按一下 [啟用來賓層級監視] 按鈕。

    ![檢視診斷計量](./media/tutorial-monitoring/enable-diagnostics-extension.png)

## <a name="view-vm-metrics"></a>檢視 VM 計量

您可以檢視 VM 計量，和您檢視主機 VM 計量資訊的方式相同︰

1. 從 Azure 入口網站中，按一下 [資源群組]，選取 [myResourceGroupMonitor]，然後選取資源清單中的 [myVM]。
2. 若要查看 VM 的執行狀況，按一下 VM 刀鋒視窗上的 [計量]，然後選取 [可用的計量] 下的任何診斷計量。

    ![檢視 VM 計量](./media/tutorial-monitoring/monitor-vm-metrics.png)

## <a name="create-alerts"></a>建立警示

您可以根據特定效能計量來建立警示。 舉例來說，可使用警示來通知您平均 CPU 使用量超過特定臨界值、或是可用磁碟空間降到低於某個量。 警示會顯示在 Azure 入口網站，或是可以透過電子郵件傳送。 您也可以觸發 Azure 自動化 Runbook 或 Azure Logic Apps 以回應產生的警示。

以下範例會建立平均 CPU 使用量的警示。

1. 從 Azure 入口網站中，按一下 [資源群組]，選取 [myResourceGroupMonitor]，然後選取資源清單中的 [myVM]。
2. 按一下VM 刀鋒視窗中的 [警示規則]，按一下橫跨在警示刀鋒視窗上方的 [新增計量警示]。
3. 提供警示的 [名稱]，例如 myAlertRule。
4. 若要在 CPU 百分比連續五分鐘超過 1.0 時觸發警示，保留所有其他已選取的預設值。
5. (選擇性) 選取 [電子郵件的擁有者、參與者及讀者] 核取方塊以傳送電子郵件通知。 預設動作是在入口網站中顯示通知。
6. 按一下 [確定] 按鈕。

## <a name="manage-windows-updates"></a>管理 Windows 更新

更新管理可讓您管理 Azure Windows VM 的更新和修補程式。
您可以直接從 VM 快速評估可用更新的狀態、排定何時安裝必要的更新，並檢閱部署結果來確認更新已成功套用至 VM。

如需價格資訊，請參閱[更新管理的自動化價格](https://azure.microsoft.com/pricing/details/automation/)

### <a name="enable-update-management"></a>啟用更新管理

啟用 VM 的更新管理：

1. 在畫面左邊，選取 [虛擬機器]。
2. 從清單中選取 VM。
3. 在 [VM] 畫面的 [作業] 區段中，按一下 [更新管理]。 [啟用更新管理] 畫面隨即開啟。

將會執行驗證來判斷此 VM 是否已啟用更新管理。
驗證包括檢查 Log Analytics 工作區及連結的自動化帳戶，以及解決方法是否在工作區中。

[Log Analytics](../../log-analytics/log-analytics-overview.md) 工作區可用來收集功能和服務 (例如更新管理) 所產生的資料。
工作區提供單一位置來檢閱和分析來自多個來源的資料。
若要在需要更新的 VM 上執行其他動作，Azure 自動化可讓您對 VM 執行 Runbook，例如下載和套用更新。

驗證程序也會檢查 VM 是否以 Microsoft Monitoring Agent (MMA) 和自動化混合式 Runbook 背景工作角色佈建。
此代理程式用來與 VM 通訊，並取得更新狀態的相關資訊。

選擇 Log Analytics 工作區和自動化帳戶，然後按一下 [啟用] 以啟用解決方案。 啟用解決方案最多需要 15 分鐘。

如果在上線期間遺漏下列任何必要條件，就會自動新增：

* [Log Analytics](../../log-analytics/log-analytics-overview.md) 工作區
* [自動化](../../automation/automation-offering-get-started.md)
* VM 上已啟用 [Hybrid Runbook 背景工作角色](../../automation/automation-hybrid-runbook-worker.md)

[更新管理] 畫面隨即開啟。 設定位置、Log Analytics 工作區以及要使用的自動化帳戶，然後按一下 [啟用]。 如果欄位呈現灰色，就表示已啟用 VM 的另一個自動化解決方案，且必須使用相同的工作區和自動化帳戶。

![啟用更新管理解決方案](./media/tutorial-monitoring/manageupdates-update-enable.png)

啟用解決方案可能需要 15 分鐘。 在此期間，請勿關閉瀏覽器視窗。 啟用解決方案之後，有關在 VM 上遺漏更新的相關資訊會流向 Log Analytics。 可能需要 30 分鐘到 6 小時，資料才可供分析。

### <a name="view-update-assessment"></a>檢視更新評量

啟用 [更新管理] 之後，隨即會顯示 [更新管理] 畫面。 完成更新評估之後，您會在 [遺失更新] 索引標籤上看到遺失的更新清單。

 ![檢視更新狀態](./media/tutorial-monitoring/manageupdates-view-status-win.png)

### <a name="schedule-an-update-deployment"></a>排定更新部署

若要安裝更新，請將部署排定在發行排程和服務時段之後。 您可以選擇要在部署中包含的更新類型。 例如，您可以包含重大更新或安全性更新，並排除更新彙總套件。

按一下 [更新管理] 畫面頂端的 [排程更新部署]，以針對 VM 來排程新的更新部署。 在 [新增更新部署] 畫面上，指定下列資訊：

* **名稱** - 提供唯一名稱來識別更新部署。
* **更新分類** - 選取更新部署在部署中包含的軟體類型。 分類類型包括：
  * 重大更新
  * 安全性更新
  * 更新彙總套件
  * Feature Pack
  * Service Pack
  * 定義更新
  * 工具
  * 更新

* **排程設定** - 您可以接受預設的日期和時間 (目前時間之後的 30 分鐘)，或指定不同的時間。
  您也可以指定部署是否為發生一次，或設定週期性排程。 按一下 [週期] 下的 [週期性] 選項，即可設定週期性排程。

  ![更新排程設定畫面](./media/tutorial-monitoring/manageupdates-schedule-win.png)

* **維護時間範圍 (分鐘)** - 指定您要執行更新部署的時段。  這有助於確保在您定義的服務時段內執行變更。

排程設定完成之後，請按一下 [建立] 按鈕，您就會返回狀態儀表板。
請注意，[已排程] 表格會顯示您已建立的部署排程。

> [!WARNING]
> 如果更新需要重新開機，VM 會自動重新啟動。

### <a name="view-results-of-an-update-deployment"></a>檢視更新部署的結果

已排程的部署開始之後，您就可以在 [更新管理] 畫面的 [更新部署] 索引標籤上看到該部署的狀態。
如果該部署目前正在執行，其狀態會顯示為 [進行中]。 當它完成時，如果成功，狀態就會變更為 [成功]。
如果部署中的一或多個更新失敗，則狀態為 [部分失敗]。
按一下已完成的更新部署，以查看該更新部署的儀表板。

![特定部署的更新部署狀態儀表板](./media/tutorial-monitoring/manageupdates-view-results.png)

[更新結果] 磚包含 VM 上的更新總數和部署結果的摘要。
右邊表格是每個更新和安裝結果的詳細解析，可能是下列其中一個值：

* **未嘗試** - 未安裝更新，因為在已定義的維護時段內，沒有足夠的時間可用。
* **成功** - 更新已順利完成
* **失敗** - 更新失敗

按一下 [所有記錄] 以查看部署已建立的所有記錄項目。

按一下 [輸出] 磚，以查看負責在目標 VM 上管理更新部署之 Runbook 的作業串流。

按一下 [錯誤]，即可查看部署所傳回之任何錯誤的詳細資訊。

## <a name="monitor-changes-and-inventory"></a>監視變更和清查

您可以在電腦上收集並檢視軟體、檔案、Linux 精靈、Windows 服務和 Windows 登錄機碼的清查。 追蹤您電腦的設定可協助您找出環境中的操作問題，並進一步了解您電腦的狀態。

### <a name="enable-change-and-inventory-management"></a>啟用變更和清查管理

為您的 VM 啟用「變更」和「清查」管理：

1. 在畫面左邊，選取 [虛擬機器]。
2. 從清單中選取 VM。
3. 在 VM 畫面上，按一下 [作業] 區段中的 [清查] 或 [變更追蹤]。 [啟用變更追蹤與詳細目錄] 畫面隨即開啟。

設定位置、Log Analytics 工作區以及要使用的自動化帳戶，然後按一下 [啟用]。 如果欄位呈現灰色，就表示已啟用 VM 的另一個自動化解決方案，且必須使用相同的工作區和自動化帳戶。 儘管這些解決方案在功能表上是分開的，但它們是相同的解決方案。 啟用其中一個會為您的 VM 同時啟用兩個。

![啟用變更與清查追蹤](./media/tutorial-monitoring/manage-inventory-enable.png)

啟用解決方案之後，可能需要一些時間在 VM 上收集清查，然後才會顯示資料。

### <a name="track-changes"></a>追蹤變更

在您的 VM 上，選取 [作業] 底下的 [變更追蹤]。 按一下 [編輯設定]，就會顯示 [變更追蹤] 頁面。 選取您想要追蹤的設定類型，然後按一下 [+ 新增] 來進行設定。 可用的 Windows 選項包括：

* Windows 登錄
* Windows 檔案

如需有關「變更追蹤」的詳細資訊，請參閱 [針對 VM 上的變更進行疑難排解](../../automation/automation-tutorial-troubleshoot-changes.md)

### <a name="view-inventory"></a>檢視清查

在您的 VM 上，選取 [作業] 底下的 [清查]。 在 [軟體] 索引標籤上，有一份資料表列出已找到的軟體。 每一筆軟體記錄的高階詳細資料都可在資料表中進行檢視。 這些詳細資料包括軟體名稱、版本、發行者、上次重新整理時間。

![檢視清查](./media/tutorial-monitoring/inventory-view-results.png)

### <a name="monitor-activity-logs-and-changes"></a>監視活動記錄和變更

從您 VM 上的 [變更追蹤] 頁面，選取 [管理活動記錄連線]。 此工作會開啟 [Azure 活動記錄] 頁面。 選取 [連線] 可將變更追蹤連線 VM 的 Azure 活動記錄。

此設定啟用時，瀏覽至您 VM 的 [概觀] 頁面，並選取 [停止] 來將您的 VM 停止。 出現提示時，選取 [是] 可停止 VM。 當它解除配置時，選取 [啟動] 可將您的 VM 重新啟動。

將 VM 記錄停止和啟動，可在其活動記錄中記錄事件。 瀏覽回 [變更追蹤] 頁面。 選取頁面底部的 [事件] 索引標籤。 一段時間之後，就會在圖表和資料表中顯示事件。 您可以選取每一個事件來檢視該事件的相關詳細資訊。

![檢視活動記錄中的變更](./media/tutorial-monitoring/manage-activitylog-view-results.png)

圖表會顯示一段時間內已發生的變更。 在您新增活動記錄連線之後，頂端的線條圖表會顯示 Azure 活動記錄事件。 長條圖中的每個資料列都代表不同的可追蹤變更類型。 這些類型為 Linux 精靈、檔案、Windows 登錄機碼、軟體和 Windows 服務。 [變更] 索引標籤會以發生變更時間的遞減順序 (最新的優先)，顯示視覺效果中所顯示變更的詳細資料。

## <a name="advanced-monitoring"></a>進階監視

您可以使用 [Azure 自動化](../../automation/automation-intro.md)所提供的「更新管理」和「變更」與「清查」等解決方案，來進行更進階的 VM 監視。

當您能夠存取 Log Analytics 工作區時，便可以藉由選取 [設定] 底下的 [進階設定]，找出工作區金鑰和工作區識別碼。 請使用 [Set-AzureRmVMExtension](/powershell/module/azurerm.compute/set-azurermvmextension) 命令將 Microsoft Monitoring Agent 延伸模組新增至 VM。 更新以下範例中的變數值，以反映您的 Log Analytics 工作區金鑰和工作區識別碼。

```powershell
$workspaceId = "<Replace with your workspace Id>"
$key = "<Replace with your primary key>"

Set-AzureRmVMExtension -ResourceGroupName "myResourceGroupMonitor" `
  -ExtensionName "Microsoft.EnterpriseCloud.Monitoring" `
  -VMName "myVM" `
  -Publisher "Microsoft.EnterpriseCloud.Monitoring" `
  -ExtensionType "MicrosoftMonitoringAgent" `
  -TypeHandlerVersion 1.0 `
  -Settings @{"workspaceId" = $workspaceId} `
  -ProtectedSettings @{"workspaceKey" = $key} `
  -Location "East US"
```

幾分鐘之後，您應該就會在 Log Analytics 工作區中看到新的 VM。

![OMS 刀鋒視窗](./media/tutorial-monitoring/tutorial-monitor-oms.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您利用 Azure 資訊安全中心設定並檢閱 VM。 您已了解如何︰

> [!div class="checklist"]
> * 建立虛擬網路
> * 建立資源群組和 VM
> * 啟用 VM 上的開機診斷
> * 檢視開機診斷
> * 檢視主機計量
> * 安裝診斷擴充功能
> * 檢視 VM 計量
> * 建立警示
> * 管理 Windows 更新
> * 監視變更和清查
> * 設定進階監視

請前進到下一個教學課程，了解 Azure 資訊安全中心。

> [!div class="nextstepaction"]
> [管理 VM 安全性](./tutorial-azure-security.md)