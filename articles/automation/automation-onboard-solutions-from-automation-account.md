---
title: 了解如何讓 Azure 自動化中的更新管理、變更追蹤和清查解決方案上線
description: 了解如何在 Azure 虛擬機器上讓屬於 Azure 自動化一部分的更新管理、變更追蹤和清查解決方案上線
services: automation
ms.service: automation
author: georgewallace
ms.author: gwallace
ms.date: 06/06/2018
ms.topic: conceptual
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: 5b906b4a90dbceb62c6f2381d0ffa8bc1bee7ef1
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033816"
---
# <a name="onboard-update-management-change-tracking-and-inventory-solutions"></a>讓更新管理、變更追蹤和清查解決方案上線

Azure 自動化提供的解決方案可管理作業系統安全性更新、追蹤變更，以及清查您的電腦上安裝的項目。 讓機器上線的方式有很多種，您可以[從虛擬機器](automation-onboard-solutions-from-vm.md)、[瀏覽多部機器時](automation-onboard-solutions-from-browse.md)、從您的自動化帳戶，或透過 [Runbook](automation-onboard-solutions.md) 來讓解決方案上線。 此文章說明如何從您的自動化帳戶讓這些解決方案上線。

## <a name="log-in-to-azure"></a>登入 Azure

在 https://portal.azure.com 登入 Azure

## <a name="enable-solutions"></a>啟用解決方案

瀏覽至您的自動化帳戶，然後選取 [組態管理] 下的 [清查] 或 [變更追蹤]。

選擇 Log Analytics 工作區與自動化帳戶，然後按一下 [啟用] 來啟用解決方案。 啟用解決方案最多需要 15 分鐘。

![讓清查解決方案上線](media/automation-onboard-solutions-from-automation-account/onboardsolutions.png)

變更追蹤和清查解決方案可讓您在虛擬機器上[追蹤變更](automation-vm-change-tracking.md)和[進行清查](automation-vm-inventory.md)。 在此步驟中，您會在虛擬機器上啟用解決方案。

當變更追蹤與清查解決方案上線通知完成時，按一下 [組態管理] 下的 [更新管理]。

更新管理解決方案可讓您管理 Azure Windows 虛擬機器的更新和修補程式。 您可以評估可用更新的狀態、排定何時安裝必要的更新，並檢閱部署結果來確認更新已成功套用至虛擬機器。 此動作已針對您的 VM 啟用解決方案。

選取 [更新管理] 下的 [更新管理]。 選取的記錄分析工作區與上一個步驟中使用的工作區相同。 按一下 [啟用]，讓更新管理解決方案上線。 啟用解決方案最多需要 15 分鐘。

![讓更新解決方案上線](media/automation-onboard-solutions-from-automation-account/onboardsolutions2.png)

## <a name="scope-configuration"></a>範圍設定

每個解決方案都會使用工作區中的範圍設定，來設定取得解決方案的電腦。 範圍設定是一或多個已儲存搜尋的群組，用以將解決方案的範圍限定於特定電腦。 若要存取自動化帳戶中的範圍設定，請在 [相關資源] 之下，選取 [工作區]。 接著，在工作區的 [工作區資料來源] 底下，選取 [範圍設定]。

如果選取的工作區還沒有 [更新管理] 或 [變更追蹤] 解決方案，則系統會建立以下範圍設定：

* **MicrosoftDefaultScopeConfig-ChangeTracking**

* **MicrosoftDefaultScopeConfig-Updates**

如果選取的工作區已經有該解決方案。 系統不會重新部署解決方案，範圍設定也不會新增至解決方案。

## <a name="saved-searches"></a>已儲存的搜尋

當電腦新增至「更新管理」或「變更追蹤」和「清查」解決方案時，它們會被新增至工作區中兩個已儲存搜尋的其中一個。 這些已儲存的搜尋都是查詢，其中包含這些解決方案的目標電腦。

瀏覽至您的自動化帳戶，然後選取 [一般] 下的 [已儲存搜尋]。 下表顯示這些解決方案所使用的兩個已儲存搜尋：

|名稱     |類別  |Alias  |
|---------|---------|---------|
|MicrosoftDefaultComputerGroup     |  ChangeTracking       | ChangeTracking__MicrosoftDefaultComputerGroup        |
|MicrosoftDefaultComputerGroup     | 更新        | Updates__MicrosoftDefaultComputerGroup         |

選取任一個已儲存搜尋，以檢視用來填入群組的查詢。 下圖顯示查詢與其結果：

![已儲存的搜尋](media/automation-onboard-solutions-from-automation-account/savedsearch.png)

## <a name="onboard-azure-vms"></a>使 Azure VM 上線

從您的自動化帳戶中選取 [組態管理] 下的 [清查] 或 [變更追蹤]，或選取 [更新管理] 下的 [更新管理]。

按一下 [+ 加入 Azure VM]，從清單中選取一或多個 VM。 無法啟用的虛擬機器會呈現灰色且無法選取。 在 [啟用更新管理] 頁面上，按一下 [啟用]。 這會為解決方案將選取的 VM 加入至電腦群組儲存的搜尋。

![啟用 Azure VM](media/automation-onboard-solutions-from-automation-account/enable-azure-vms.png)

## <a name="onboard-a-non-azure-machine"></a>讓非 Azure 機器上線

您必須手動加入不在 Azure 中的機器。 從您的自動化帳戶中選取 [組態管理] 下的 [清查] 或 [變更追蹤]，或選取 [更新管理] 下的 [更新管理]。

按一下 [加入非 Azure 電腦]。 這會開啟新的瀏覽器視窗，[指示如何在電腦上安裝和設定 Microsoft Monitoring Agent](../log-analytics/log-analytics-concept-hybrid.md)，讓電腦可以開始向解決方案回報。 如果您讓目前受 System Center Operations Manager 管理的機器上線，不需要新的代理程式，就會將工作區資訊輸入現有的代理程式。

## <a name="onboard-machines-in-the-workspace"></a>讓工作區中的機器上線

您必須將手動安裝的機器或已向您工作區回報的機器加入至 Azure 自動化，才能啟用解決方案。 從您的自動化帳戶中選取 [組態管理] 下的 [清查] 或 [變更追蹤]，或選取 [更新管理] 下的 [更新管理]。

選取 [管理機器]。 這會開啟 [管理機器] 頁面。 此頁面可讓您啟用一組特定機器、所有可用機器上的解決方案，或啟用所有目前機器的解決方案，以及啟用未來所有機器的解決方案。

![已儲存的搜尋](media/automation-onboard-solutions-from-automation-account/managemachines.png)

### <a name="all-available-machines"></a>所有可用的機器

若要為所有可用的機器啟用解決方案，請選取 [在所有可用的機器上啟用]。 這會停用個別新增機器的控制項。 此工作會將向工作區回報之所有機器的名稱新增至電腦群組儲存的搜尋查詢。

### <a name="all-available-and-future-machines"></a>所有可用與未來的機器

若要為所有可用的機器與所有未來的機器啟用解決方案，請選取 [在所有可用及未來的機器上啟用]。 此選項會從工作區中刪除已儲存搜尋和範圍設定。 這會開啟向工作區回報之所有 Azure 與非 Azure 機器的解決方案。

### <a name="selected-machines"></a>選取的機器

若要啟用一或多部機器的解決方案，請選取 [在選取的機器上啟用]，然後按一下想要新增至解決方案的每部機器旁邊的 [新增]。 此工作會為解決方案將所選取機器的名稱加入至電腦群組儲存的搜尋查詢。

## <a name="unlink-workspace"></a>取消連結工作區

下列解決方案相依於 Log Analytics 工作區：

* [更新管理](automation-update-management.md)
* [變更追蹤](automation-change-tracking.md)
* [於下班時間啟動/停止 VM](automation-solution-vm-management.md)

若決定不想再讓自動化帳戶與 Log Analytics 整合，您可以直接從 Azure 入口網站將您的帳戶取消連結。  繼續之前，您必須先移除稍早所述的解決方案，否則無法進行此程序。 檢閱已匯入特定解決方案的相關文章，以了解移除解決方案所需的步驟。

移除這些解決方案之後，您可以執行下列步驟以將您的自動化帳戶取消連結。

> [!NOTE]
> 某些包含舊版 Azure SQL 監視解決方案的解決方案可能已建立自動化資產，在取消連結工作區之前，可能也需要先加以移除。

1. 從 Azure 入口網站開啟您的自動化帳戶，然後在 [自動化帳戶] 頁面上，在左側的 [相關資源] 區段下選取 [已取消連結的工作區]。

1. 在 [取消連結工作區] 頁面上，按一下 [取消連結工作區]。

   ![取消連結工作區頁面](media/automation-onboard-solutions-from-automation-account/automation-unlink-workspace-blade.png).

   您會收到提示，確認您想要繼續。

1. 當 Azure 自動化嘗試將您的帳戶從 Log Analytics 工作區取消連結時，您可以從功能表在 [通知] 下追蹤進度。

若使用「更新管理」解決方案，您可以在移除解決方案之後選擇移除已不再需要的下列項目。

* 更新排程 - 每個都會有符合您建立的更新部署名稱

* 針對解決方案建立的混合式背景工作角色群組 - 每個都會具有如下名稱：machine1.contoso.com_9ceb8108-26c9-4051-b6b3-227600d715c8。

若使用「於下班時間啟動/停止 VM」解決方案，您可以在移除解決方案之後選擇移除已不再需要的下列項目。

* 啟動及停止 VM Runbook 排程
* 啟動及停止 VM Runbook
* 變數

## <a name="next-steps"></a>後續步驟

繼續進行解決方案的教學課程以了解如何使用。

* [教學課程 - 管理 VM 的更新](automation-tutorial-update-management.md)

* [教學課程 - 識別 VM 上的軟體](automation-tutorial-installed-software.md)

* [教學課程 - 對 VM 的變更進行疑難排解](automation-tutorial-troubleshoot-changes.md)