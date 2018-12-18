---
title: 管理多部 Azure 虛擬機器的更新
description: 本文說明如何管理 Azure 虛擬機器的更新。
services: automation
ms.service: automation
ms.component: update-management
author: georgewallace
ms.author: gwallace
ms.date: 08/29/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 231a9876c7a84953a7d9a88b761a1da9475d1f48
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43248136"
---
# <a name="manage-updates-for-multiple-machines"></a>管理多部機器的更新

您可以使用「更新管理」解決方案來管理 Windows 和 Linux 虛擬機器的更新和修補程式。 您可以從您的 [Azure 自動化](automation-offering-get-started.md)帳戶：

- 讓虛擬機器上線
- 評估可用更新的狀態
- 排定必要更新的安裝
- 檢閱部署結果，以確認更新已成功套用至已啟用「更新管理」的所有虛擬機器。

## <a name="prerequisites"></a>必要條件

若要使用「更新管理」，您需要：

- Azure 自動化執行身分帳戶。 若要了解帳戶的建立方式，請參閱[開始使用 Azure 自動化](automation-offering-get-started.md)。
- 已安裝其中一個支援作業系統的虛擬機器或電腦。

## <a name="supported-operating-systems"></a>受支援的作業系統

下列作業系統支援「更新管理」：

|作業系統  |注意  |
|---------|---------|
|Windows Server 2008、Windows Server 2008 R2 RTM    | 僅支援更新評估。         |
|Windows Server 2008 R2 SP1 和更新版本     |必須要有 Windows PowerShell 4.0 或更新的版本。 ([下載 WMF 4.0](https://www.microsoft.com/download/details.aspx?id=40855))</br> 建議使用 Windows PowerShell 5.1 以增加可靠性。 ([下載 WMF 5.1](https://www.microsoft.com/download/details.aspx?id=54616))         |
|CentOS 6 (x86/x64) 和 7 (x64)      | Linux 代理程式必須能夠存取更新存放庫。        |
|Red Hat Enterprise 6 (x86/x64) 和 7 (x64)     | Linux 代理程式必須能夠存取更新存放庫。        |
|SUSE Linux Enterprise Server 11 (x86/x64) 和 12 (x64)     | Linux 代理程式必須能夠存取更新存放庫。        |
|Ubuntu 12.04 LTS、14.04 LTS 和 16.04 LTS (x86/x64)      |Linux 代理程式必須能夠存取更新存放庫。         |

> [!NOTE]
> 若要避免在 Ubuntu 維護期間以外套用更新，請將自動安裝升級套件重新設定為停用自動更新。 如需詳細資訊，請參閱 [Ubuntu Server 指南中的自動更新主題](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)。

Linux 代理程式必須能夠存取更新存放庫。

這個解決方案不支援適用於 Linux 且設定為向多個 Azure Log Analytics 工作區回報的 Operations Management Suite (OMS) 代理程式。

## <a name="enable-update-management-for-azure-virtual-machines"></a>為 Azure 虛擬機器啟用「更新管理」

在 Azure 入口網站中，開啟您的自動化帳戶，然後選取 [更新管理]。

選取 [加入 Azure VM]。

![[加入 Azure VM] 索引標籤](./media/manage-update-multi/update-onboard-vm.png)

選取要上線的虛擬機器。 

在 [啟用更新管理] 下方，選取 [啟用] 以讓虛擬機器上線。

![啟用 [更新管理] 對話方塊](./media/manage-update-multi/update-enable.png)

完成上線之後，隨即會為您的虛擬機器啟用「更新管理」。

## <a name="enable-update-management-for-non-azure-virtual-machines-and-computers"></a>為非 Azure 虛擬機器和電腦啟用「更新管理」

若要了解如何為非 Azure Windows 虛擬機器和電腦啟用「更新管理」，請參閱[將 Windows 電腦連線到 Azure 中的 Log Analytics 服務](../log-analytics/log-analytics-windows-agent.md)。

若要了解如何為非 Azure Linux 虛擬機器和電腦啟用「更新管理」，請參閱[將您的 Linux 電腦連線到 Log Analytics](../log-analytics/log-analytics-agent-linux.md)。

## <a name="view-computers-attached-to-your-automation-account"></a>檢視連結到自動化帳戶的電腦

為您的電腦啟用「更新管理」之後，就可以選取 [電腦] 以檢視機器資訊。 您可以看到電腦的機器名稱、合規性狀態、環境、OS 類型、已安裝的重大和安全性更新、其他已安裝的更新和*更新代理程式整備程度*等相關資訊。

  ![檢視電腦索引標籤](./media/manage-update-multi/update-computers-tab.png)

最近啟用「更新管理」的電腦可能尚未經過評估。 那些電腦的合規性狀態是 [未評估]。 以下是可能的合規性狀態值清單：

- **符合規範**：未錯過重大或安全性更新的電腦。

- **不符合規範**：至少錯過一次重大或安全性更新的電腦。

- **未評估**：在預期的時間範圍內，未收到來自電腦的更新評估資料。 針對 Linux 電腦，預期時間範圍是過去 3 小時以內。 針對 Windows 電腦，預期時間範圍是過去 12 小時以內。

若要檢視代理程式的狀態，請選取 [更新代理程式整備程度] 欄中的連結。 選取這個選項會開啟 [混合式背景工作角色] 窗格，並顯示混合式背景工作角色的狀態。 下圖顯示長期未連線至「更新管理」的代理程式範例：

![檢視電腦索引標籤](./media/manage-update-multi/update-agent-broken.png)

## <a name="view-an-update-assessment"></a>檢視更新評估

啟用 [更新管理] 之後，[更新管理] 窗格隨即開啟。 您可以在 [遺失更新] 索引標籤上看到遺失的更新清單。

## <a name="collect-data"></a>收集資料

虛擬機器和電腦上安裝的代理程式會收集關於更新的資料。 代理程式會將資料傳送至 Azure 更新管理。

### <a name="supported-agents"></a>支援的代理程式

下表描述本解決方案支援的連線來源：

| 連線的來源 | 支援 | 說明 |
| --- | --- | --- |
| Windows 代理程式 |是 |「更新管理」會從 Windows 代理程式收集系統更新的相關資訊，然後起始必要更新的安裝。 |
| Linux 代理程式 |是 |「更新管理」會從 Linux 代理程式收集系統更新的相關資訊，然後在支援的發行版本上起始必要更新的安裝。 |
| Operations Manager 管理群組 |是 |「更新管理」會從所連線之管理群組中的代理程式收集系統更新的相關資訊。 |
| Azure 儲存體帳戶 |否 |Azure 儲存體不包含系統更新的相關資訊。 |

### <a name="collection-frequency"></a>收集頻率

針對每部受控 Windows 電腦，每天會掃描兩次。 系統會每隔 15 分鐘呼叫一次 Windows API 來查詢上次更新時間，以判斷狀態是否已變更。 如果狀態已變更，則會開始合規性掃描。 針對每部受控 Linux 電腦，每 3 小時會掃描一次。

儀表板可能需要 30 分鐘到 6 小時，才能顯示來自受控電腦的已更新資料。

## <a name="schedule-an-update-deployment"></a>排定更新部署

若要安裝更新，請將部署排定在發行排程和服務時段之後。 您可以選擇要在部署中包含的更新類型。 例如，您可以包含重大更新或安全性更新，並排除更新彙總套件。

若要為一或多部虛擬機器排定新的更新部署，請在 [更新管理] 下，選取 [排程更新部署]。

在 [新增更新部署] 窗格中，指定下列資訊：

- **名稱**：輸入唯一名稱來識別更新部署。
- **作業系統**：選取 [Windows] 或 [Linux]。
- **要更新的機器**：選取已儲存的搜尋或已匯入的群組，或選取機器，以選擇您想要更新的機器。 如果您選擇 [機器]，機器的整備程度會顯示於 [更新代理程式整備程度] 欄中。 您可以在排程更新部署之前，先查看機器的健康情況。 若要深入了解在 Log Analytics 中建立電腦群組的不同方法，請參閱 [Log Analytics 中的電腦群組](../log-analytics/log-analytics-computer-groups.md)

  ![[新增更新部署] 窗格](./media/manage-update-multi/update-select-computers.png)

- **更新分類**：選取要包括在更新部署中的軟體類型。 如需分類類型的說明，請參閱[更新分類](automation-update-management.md#update-classifications)。 分類類型包括：
  - 重大更新
  - 安全性更新
  - 更新彙總套件
  - Feature Pack
  - Service Pack
  - 定義更新
  - 工具
  - 更新

- **要排除的更新**；選取這個選項會開啟 [排除] 頁面。 輸入要排除的 KB 文章或套件名稱。

- **排程設定**：您可以接受預設的日期和時間 (目前時間之後的 30 分鐘)。 您也可以指定不同的時間。

   您也可以指定部署是否為發生一次，或為週期性排程。 若要設定週期性排程，請選取 [週期] 下的 [定期]。

   ![排程 [設定] 對話方塊](./media/manage-update-multi/update-set-schedule.png)
- **維護時間範圍 (分鐘)**：指定您要執行更新部署的時段。 此設定有助於確保在您定義的服務時段內執行變更。

- **重新啟動控制** - 此設定可決定如何處理更新部署的重新開機。

   |選項|說明|
   |---|---|
   |在必要時重新開機| **(預設值)** 如有需要，會在維護時段允許的情況下啟動重新開機。|
   |一律重新開機|不論是否有必要，皆會啟動重新開機。 |
   |永不重新開機|不論是否需要重新開機，皆不啟動重新開機。|
   |僅重新開機 - 將不會安裝更新|此選項會略過安裝更新，而且只會啟動重新開機。|

完成排程設定之後，請選取 [建立] 按鈕以返回狀態儀表板。 [已排定] 表格會顯示您建立的部署排程。

## <a name="view-results-of-an-update-deployment"></a>檢視更新部署的結果

已排程的部署開始之後，您就可以在 [更新管理] 之下的 [更新部署] 索引標籤上看到該部署的狀態。

如果目前正在執行部署，其狀態會是 [進行中]。 部署順利完成後，狀態會變更為 [已成功]。

如果部署中的一或多個更新失敗，則狀態會是 [部分失敗]。

![更新部署的狀態](./media/manage-update-multi/update-view-results.png)

若要查看更新部署的儀表板，請選取完成的部署。

[更新結果] 窗格會顯示虛擬機器的更新總數和部署結果。 右邊的表格會提供每個更新的明細與安裝結果。 安裝結果可為下列其中一個值：

- **未嘗試**：未安裝更新，因為以所定義的維護時間範圍為基礎，可用的時間不足。
- **成功**：更新成功。
- **失敗**：更新失敗。

若要查看部署已建立的所有記錄項目，請選取 [所有記錄]。

若要查看管理目標虛擬機器上更新部署之 Runbook 的作業串流，請選取 [輸出] 圖格。

若要查看部署所傳回之任何錯誤的詳細資訊，請選取 [錯誤]。

## <a name="next-steps"></a>後續步驟

- 若要深入了解「更新管理」(包括記錄、輸出和錯誤)，請參閱 [Azure 中的更新管理解決方案](../operations-management-suite/oms-solution-update-management.md)。
