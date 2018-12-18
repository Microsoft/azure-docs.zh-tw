---
title: 針對 Azure 虛擬機器上的變更進行疑難排解 | Microsoft Docs
description: 使用變更追蹤來針對 Azure 虛擬機器上的變更進行疑難排解。
services: automation
ms.service: automation
ms.component: change-inventory-management
keywords: 變更, 追蹤, 自動化
author: jennyhunter-msft
ms.author: jehunte
ms.date: 09/12/2018
ms.topic: tutorial
ms.custom: mvc
manager: carmonm
ms.openlocfilehash: 16d5a025f0c0ff571298e0f528fb9119e37950f3
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995244"
---
# <a name="troubleshoot-changes-in-your-environment"></a>針對您環境中的變更進行疑難排解

在本教學課程中，您會了解如何針對 Azure 虛擬機器上的變更進行疑難排解。 您可以啟用變更追蹤，來追蹤軟體、檔案、Linux 精靈、Windows 服務和 Windows 登錄機碼的變更。
識別這些組態變更可協助您找出環境中的操作問題。

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 將 VM 上架以進行變更追蹤和清查
> * 搜尋已停止服務的變更記錄
> * 設定變更追蹤
> * 啟用活動記錄連線
> * 觸發事件
> * 檢視變更
> * 設定警示

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要：

* Azure 訂用帳戶。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [自動化帳戶](automation-offering-get-started.md)可保存監看員和動作 Runbook，以及監看員工作。
* 要上架的[虛擬機器](../virtual-machines/windows/quick-create-portal.md)。

## <a name="sign-in-to-azure"></a>登入 Azure

在 http://portal.azure.com 登入 Azure 入口網站。

## <a name="enable-change-tracking-and-inventory"></a>啟用變更追蹤和清查

您必須先在本教學課程中啟用 VM 的變更追蹤和清查。 如果您先前已啟用 VM 的另一個自動化解決方案，就不需要此步驟。

1. 在左側功能表上，選取 [虛擬機器]，然後從清單中選取 VM
1. 在左側功能表的 [作業] 區段下，按一下 [清查]。 [變更追蹤] 頁面隨即開啟。

![啟用變更](./media/automation-tutorial-troubleshoot-changes/enableinventory.png) [變更追蹤] 畫面隨即開啟。 設定位置、記錄分析工作區以及要使用的自動化帳戶，然後按一下 [啟用]。 如果欄位呈現灰色，就表示已啟用 VM 的另一個自動化解決方案，且必須使用相同的工作區和自動化帳戶。

[Log Analytics](../log-analytics/log-analytics-overview.md?toc=%2fazure%2fautomation%2ftoc.json) 工作區用來收集功能和服務 (例如清查) 所產生的資料。
工作區提供單一位置來檢閱和分析來自多個來源的資料。

在上線期間，會使用 Microsoft Monitoring Agent (MMA) 和混合式背景工作佈建 VM。
此代理程式用來與 VM 通訊，並取得已安裝軟體的相關資訊。

啟用解決方案可能需要 15 分鐘。 在此期間，請勿關閉瀏覽器視窗。
啟用解決方案之後，有關在 VM 上已安裝軟體和變更的相關資訊會流向 Log Analytics。
可能需要 30 分鐘到 6 小時，資料才可供分析。

## <a name="using-change-tracking-in-log-analytics"></a>使用 Log Analytics 中的變更追蹤

變更追蹤所產生的記錄資料會傳送到 Log Analytics。
若要透過執行查詢來搜尋記錄，請選取 [變更追蹤] 視窗頂端的 [Log Analytics]。
變更追蹤資料會儲存在 **ConfigurationChange** 類型之下。
下列範例 Log Analytics 查詢會傳回所有已停止的 Windows 服務。

```
ConfigurationChange
| where ConfigChangeType == "WindowsServices" and SvcState == "Stopped"
```

若要深入了解在 Log Analytics 中執行和搜尋記錄檔，請參閱 [Azure Log Analytics](../log-analytics/log-analytics-queries.md)。

## <a name="configure-change-tracking"></a>設定變更追蹤

變更追蹤可讓您追蹤您 VM 上的設定變更。 下列步驟說明如何設定登錄機碼和檔案的追蹤。

如需選擇要收集及追蹤的檔案和登錄機碼，請選取 [變更追蹤] 頁面頂端的 [編輯設定]。

> [!NOTE]
> 清查和變更追蹤會使用相同的集合設定，且設定會在工作區層級進行設定。

在 [工作區設定] 視窗中，新增要追蹤的 Windows 登錄機碼、Windows 檔案或 Linux 檔案，如接下來三節所述。

### <a name="add-a-windows-registry-key"></a>新增 Windows 登錄機碼

1. 在 [Windows 登錄] 索引標籤上，選取 [新增]。
    [為變更追蹤新增 Windows 登錄] 視窗隨即開啟。

1. 在 [為變更追蹤新增 Windows 登錄] 上，輸入要追蹤的機碼資訊，然後按一下 [儲存]

|屬性  |說明  |
|---------|---------|
|已啟用     | 判斷是否已套用設定        |
|項目名稱     | 所要追蹤檔案的易記名稱        |
|群組     | 用於將檔案以邏輯方式分組的群組名稱        |
|Windows 登錄機碼   | 要檢查檔案的路徑。例如："HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Common Startup"      |

### <a name="add-a-windows-file"></a>新增 Windows 檔案

1. 在 [Windows 檔案] 索引標籤上，選取 [新增]。 [為變更追蹤新增 Windows 檔案] 視窗隨即開啟。

1. 在 [為變更追蹤新增 Windows 檔案] 上，輸入要追蹤的檔案或目錄資訊，然後按一下 [儲存]

|屬性  |說明  |
|---------|---------|
|已啟用     | 判斷是否已套用設定        |
|項目名稱     | 所要追蹤檔案的易記名稱        |
|群組     | 用於將檔案以邏輯方式分組的群組名稱        |
|輸入路徑     | 要檢查檔案的路徑，例如："c:\temp\\\*.txt"<br>您也可以使用環境變數，例如 "%winDir%\System32\\\*.*"         |
|遞迴     | 決定在尋找所要追蹤的項目時是否使用遞迴。        |
|上傳所有的檔案內容設定| 開啟或關閉追蹤變更上的檔案內容上傳。 可用的選項：**True** 或 **False**。|

### <a name="add-a-linux-file"></a>新增 Linux 檔案

1. 在 [Linux 檔案] 索引標籤上，選取 [新增]。 [為變更追蹤新增 Linux 檔案] 視窗隨即開啟。

1. 在 [為變更追蹤新增 Linux 檔案] 上，輸入要追蹤的檔案或目錄資訊，然後按一下 [儲存]

|屬性  |說明  |
|---------|---------|
|已啟用     | 判斷是否已套用設定        |
|項目名稱     | 所要追蹤檔案的易記名稱        |
|群組     | 用於將檔案以邏輯方式分組的群組名稱        |
|輸入路徑     | 要檢查檔案的路徑。例如："/etc/*.conf"       |
|路徑類型     | 要追蹤的項目類型，可能值為 [檔案] 和 [目錄]        |
|遞迴     | 決定在尋找所要追蹤的項目時是否使用遞迴。        |
|使用 Sudo     | 此設定會決定在檢查項目時是否使用 sudo。         |
|連結     | 此設定會決定在周遊目錄時處理符號連結的方式。<br> **忽略** - 忽略符號連結，而不包含參考的檔案/目錄<br>**遵循** - 遵循遞迴期間的符號連結，而且包含參考的檔案/目錄<br>**管理** - 遵循符號連結並允許變更所傳回內容的處理方式      |
|上傳所有的檔案內容設定| 開啟或關閉追蹤變更上的檔案內容上傳。 可用的選項：**True** 或 **False**。|

   > [!NOTE]
   > 不建議選擇「管理」連結選項。 不支援檔案內容擷取。

## <a name="enable-activity-log-connection"></a>啟用活動記錄連線

從您 VM 上的 [變更追蹤] 頁面，選取 [管理活動記錄連線]。 此工作會開啟 [Azure 活動記錄] 頁面。 選取 [連線] 可將變更追蹤連線 VM 的 Azure 活動記錄。

此設定啟用時，瀏覽至您 VM 的 [概觀] 頁面，並選取 [停止] 來將您的 VM 停止。 出現提示時，選取 [是] 可停止 VM。 當它解除配置時，選取 [啟動] 可將您的 VM 重新啟動。

將 VM 記錄停止和啟動，可在其活動記錄中記錄事件。 瀏覽回 [變更追蹤] 頁面。 選取頁面底部的 [事件] 索引標籤。 一段時間之後，就會在圖表和資料表中顯示事件。 如同在先前步驟中，可以選取每一個事件來檢視關於事件的詳細資訊。

![在入口網站中檢視變更詳細資料](./media/automation-tutorial-troubleshoot-changes/viewevents.png)

## <a name="view-changes"></a>檢視變更

一旦啟用變更追蹤和清查解決方案後，您就可以檢視 [變更追蹤] 頁面上的結果。

從您的 VM 內，選取 [作業] 下的 [變更追蹤]。

![顯示 VM 變更清單的螢幕擷取畫面](./media/automation-tutorial-troubleshoot-changes/change-tracking-list.png)

圖表會顯示一段時間內已發生的變更。
在您新增活動記錄連線之後，頂端的線條圖表會顯示 Azure 活動記錄事件。
長條圖中的每個資料列都代表不同的可追蹤變更類型。
這些類型為 Linux 精靈、檔案、Windows 登錄機碼、軟體和 Windows 服務。
[變更] 索引標籤會以發生變更時間的遞減順序 (最新的優先)，顯示視覺效果中所顯示變更的詳細資料。
[事件] 索引標籤，資料表會顯示已連線的活動記錄事件，及其對應的詳細資料 (最新的優先)。

您可以在結果中看到系統已進行多次變更，包括服務和軟體的變更。 您可以使用頁面頂端的篩選條件，依 [變更類型] 或依時間範圍來篩選結果。

選取 **WindowsServices** 變更，隨即開啟 [變更詳細資料] 視窗。 變更詳細資料視窗會顯示關於變更的詳細資料，以及變更之前和之後的值。 在本範例中，軟體保護服務已停止。

![在入口網站中檢視變更詳細資料](./media/automation-tutorial-troubleshoot-changes/change-details.png)

## <a name="configure-alerts"></a>設定警示

在 Azure 入口網站中檢視變更會有幫助，但能夠在發生變更 (例如服務已停止) 時收到警示會更有用。

若要在 Azure 入口網站中針對停止的服務新增警示，請移至 [監視器]。 然後在 [共用服務] 下選取 [警示]，並按一下 [+ 新增警示規則]

在 [1. 定義警示條件] 下，按一下 [+ 選取目標]。 在 [依資源類型篩選] 下，選取 [Log Analytics]。 選取您的 Log Analytics 工作區，然後選取 [完成]。

![選取資源](./media/automation-tutorial-troubleshoot-changes/select-a-resource.png)

選取 [+ 新增準則]。
在 [設定訊號邏輯] 之下的資料表中，選取 [自訂記錄檔搜尋]。 在 [搜尋查詢] 文字方塊中，輸入下列查詢：

```loganalytics
ConfigurationChange | where ConfigChangeType == "WindowsServices" and SvcName == "W3SVC" and SvcState == "Stopped" | summarize by Computer
```

此查詢會傳回在指定時間範圍內有 W3SVC 服務停止的電腦。

在 [警示邏輯] 之下，針對 [閾值]，輸入 **0**。 完成之後，選取 [完成]。

![設定訊號邏輯](./media/automation-tutorial-troubleshoot-changes/configure-signal-logic.png)

在 **2.** 定義警示詳細資料 之下，輸入警示的名稱和描述。 將 [嚴重性] 設定為 [資訊 (Sev 2)]、[警告 (Sev 1)] 或 [重大 (Sev 0)]。

![定義警示詳細資料](./media/automation-tutorial-troubleshoot-changes/define-alert-details.png)

在 [3. 定義動作群組] 之下，選取 [新增動作群組]。 動作群組是一組可讓您跨多個警示使用的動作。 這些動作可包括 (但不限於) 電子郵件通知、Runbook、Webhook 和等多種項目。 若要深入了解動作群組，請參閱[建立及管理動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)。

在 [動作群組名稱] 方塊中，輸入警示名稱和簡短名稱。 使用這個群組傳送通知時，會使用簡短名稱來取代完整的動作群組名稱。

在 [動作] 底下，輸入動作的名稱，像是 [傳送電子郵件給系統管理員]。 在 [動作類型] 之下，選取 [電子郵件/簡訊/推播/語音]。 在 [詳細資料] 下，選取 [編輯詳細資料]。

![新增動作群組](./media/automation-tutorial-troubleshoot-changes/add-action-group.png)

在 [電子郵件/簡訊/推播/語音] 窗格中，輸入名稱。 選取 [電子郵件] 核取方塊，然後輸入有效的電子郵件地址。 在 [電子郵件/簡訊/推播/語音] 頁面上按一下 [確定]，然後按一下 [新增動作群組] 頁面上的 [確定]。

若要自訂警示電子郵件的主旨，請在 [建立規則] 的 [自訂動作] 之下，選取 [電子郵件主旨]。 當您完成時，選取 [建立警示規則]。 警示會在更新部署成功時，告知您有哪些機器包含在該更新部署執行中。

下圖是 W3SVC 服務停止時所收到的電子郵件範例。

![電子郵件](./media/automation-tutorial-troubleshoot-changes/email.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 將 VM 上架以進行變更追蹤和清查
> * 搜尋已停止服務的變更記錄
> * 設定變更追蹤
> * 啟用活動記錄連線
> * 觸發事件
> * 檢視變更
> * 設定警示

如需深入了解，請繼續閱讀變更追蹤和清查解決方案的概觀。

> [!div class="nextstepaction"]
> [變更管理及清查解決方案](automation-change-tracking.md)
