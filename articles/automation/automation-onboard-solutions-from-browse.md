---
title: 了解如何在 Azure 自動化中讓多個 VM 的更新管理、變更追蹤和清查解決方案上架
description: 了解如何在 Azure 虛擬機器上讓屬於 Azure 自動化一部分的更新管理、變更追蹤和清查解決方案上線
services: automation
ms.service: automation
author: georgewallace
ms.author: gwallace
ms.date: 06/06/2018
ms.topic: article
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: 5bb36c693db5b2d7d46b772fd8b92bcda3667dc7
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47039423"
---
# <a name="enable-update-management-change-tracking-and-inventory-solutions-on-multiple-vms"></a>在多個 VM 上啟用更新管理、變更追蹤和清查解決方案

Azure 自動化提供的解決方案可管理作業系統安全性更新、追蹤變更，以及清查您的電腦上安裝的項目。 讓機器上架的方式有很多種，您可以[從虛擬機器](automation-onboard-solutions-from-vm.md)、從[自動化帳戶](automation-onboard-solutions-from-automation-account.md)、透過瀏覽虛擬機器，或透過 [Runbook](automation-onboard-solutions.md) 來讓解決方案上架。 本文會說明如何在瀏覽 Azure 中的虛擬機器時，讓這些解決方案上架。

## <a name="log-in-to-azure"></a>登入 Azure

在 https://portal.azure.com 登入 Azure

## <a name="enable-solutions"></a>啟用解決方案

在 Azure 入口網站中，瀏覽至 [虛擬機器]。

使用核取方塊來選取虛擬機器，也就是您想要讓其與「變更追蹤」和「清查」或「更新管理」一起上架的虛擬機器。 上架一次可包含三個不同的資源群組。

![VM 清單](media/automation-onboard-solutions-from-browse/vmlist.png)
> [!TIP]
> 使用篩選控制項修改虛擬機器清單，然後按一下上方的核取方塊，選取清單中的所有虛擬機器。

從命令列按一下 [服務]，然後選取 [變更追蹤]、[清查] 或 [更新管理]。

> [!NOTE]
> [變更追蹤] 和 [清查] 使用相同的解決方案，因此啟用其中之一時，另一個也會啟用。

下圖顯示的是更新管理。 變更追蹤和清查有相同的配置和行為。

虛擬機器清單會加以篩選，只顯示位在相同訂用帳戶及位置中的虛擬機器。 如果您的虛擬機器位於三個以上的資源群組中，則會選取前三個資源群組。

### <a name="resource-group-limit"></a> 上線限制

您可用於上線的資源群組數目受限於 [Resource Manager 部署限制](../azure-resource-manager/resource-manager-cross-resource-group-deployment.md)。 Resource Manager 部署 (不要與更新部署混淆) 受限於每個部署 5 個資源群組。 為了確保上線的完整性，系統會保留其中 2 個資源群組來設定 Log Analytics 工作區、自動化帳戶和相關資源。 這讓您有 3 個資源群組可選取以供部署。

使用篩選控制項從不同訂用帳戶、位置和資源群組中選取虛擬機器。

![讓更新管理解決方案上架](media/automation-onboard-solutions-from-browse/onboardsolutions.png)

檢閱 Log Analytics 工作區與自動化帳戶的選項。 預設會選取現有的工作區及自動化帳戶。 如果您想使用不同的 Log Analytics 工作區和自動化帳戶，請按一下 [自訂]，從 [自訂組態] 頁面中選取。 當您選擇 Log Analytics 工作區時，系統會進行檢查，以判斷它是否與自動化帳戶連結。 如果找到連結的自動化帳戶，您會看到下列畫面。 完成時，按一下 [確定]。

![選取工作區和帳戶](media/automation-onboard-solutions-from-browse/selectworkspaceandaccount.png)

如果所選的工作區並未連結到自動化帳戶，您會看到下列畫面。 選取自動化帳戶，然後在完成時按一下 [確定]。

![沒有工作區](media/automation-onboard-solutions-from-browse/no-workspace.png)

針對您不想啟用的虛擬機器，取消選取它旁邊的核取方塊。 無法啟用的虛擬機器已取消選取。

按一下 [啟用] 以啟用解決方案。 啟用解決方案最多需要 15 分鐘。

## <a name="unlink-workspace"></a>取消連結工作區

下列解決方案相依於 Log Analytics 工作區：

* [更新管理](automation-update-management.md)
* [變更追蹤](automation-change-tracking.md)
* [於下班時間啟動/停止 VM](automation-solution-vm-management.md)

若決定不想再讓自動化帳戶與 Log Analytics 整合，您可以直接從 Azure 入口網站將您的帳戶取消連結。 繼續之前，您必須先移除稍早所述的解決方案，否則無法進行此程序。 檢閱已匯入特定解決方案的相關文章，以了解移除解決方案所需的步驟。

移除這些解決方案之後，您可以執行下列步驟以將您的自動化帳戶取消連結。

> [!NOTE]
> 某些包含舊版 Azure SQL 監視解決方案的解決方案可能已建立自動化資產，在取消連結工作區之前，可能也需要先加以移除。

1. 從 Azure 入口網站開啟您的自動化帳戶，然後在 [自動化帳戶] 頁面上，在左側的 [相關資源] 區段下選取 [已取消連結的工作區]。

1. 在 [取消連結工作區] 頁面上，按一下 [取消連結工作區]。

   ![取消連結工作區頁面](media/automation-onboard-solutions-from-browse/automation-unlink-workspace-blade.png).

   您會收到提示，確認您想要繼續。

1. 當 Azure 自動化嘗試將您的帳戶從 Log Analytics 工作區取消連結時，您可以從功能表在 [通知] 下追蹤進度。

若使用「更新管理」解決方案，您可以在移除解決方案之後選擇移除已不再需要的下列項目。

* 更新排程 - 每個都會有符合您建立的更新部署名稱

* 針對解決方案建立的混合式背景工作角色群組 - 每個都會具有如下名稱：machine1.contoso.com_9ceb8108-26c9-4051-b6b3-227600d715c8。

若使用「於下班時間啟動/停止 VM」解決方案，您可以在移除解決方案之後選擇移除已不再需要的下列項目。

* 啟動及停止 VM Runbook 排程
* 啟動及停止 VM Runbook
* 變數

## <a name="troubleshooting"></a>疑難排解

上架多部機器時，可能會有機器顯示為 [無法啟用]。 有許多不同原因會造成部分機器無法啟用。 下列章節將說明嘗試上架 VM 時，出現 [無法啟用] 狀態的可能原因。

### <a name="vm-reports-to-a-different-workspace-workspacename--change-configuration-to-use-it-for-enabling"></a>VM 向不同的工作區報告：'\<workspaceName\>'。  將組態變更為啟用所需的組態

**原因**：此錯誤表示您嘗試上架的 VM 是向另一個工作區報告。

**解決方案**：按一下 [作為組態使用]，變更目標自動化帳戶和 Log Analytics 工作區。

### <a name="vm-reports-to-a-workspace-that-is-not-available-in-this-subscription"></a>VM 向此訂用帳戶中無法使用的工作區報告

**原因**：虛擬機器報告的目標工作區：

* 位於不同訂用帳戶，或
* 已不存在，或
* 位於您無權存取的資源群組中

**解決方案**：找出哪個自動化帳戶與 VM 報告的目標工作區相關聯，然後藉由變更範圍組態來將該虛擬機器上架。

### <a name="vm-operating-system-version-or-distribution-is-not-supported"></a>不支援 VM 作業系統版本或發行版本

**原因：** 所有 Linux 發行版本或所有 Windows 版本皆不支援解決方案。

**解決方案：** 請參閱[支援的用戶端清單](automation-update-management.md#clients)以解決此問題。

### <a name="classic-vms-cannot-be-enabled"></a>無法啟用傳統 VM

**原因**：不支援使用傳統部署模型的 VM。

**解決方案**：將虛擬機器遷移至資源管理員部署模型。 若要了解如何執行這項操作，請參閱[遷移傳統部署模型資源](../virtual-machines/windows/migration-classic-resource-manager-overview.md)。

### <a name="vm-is-stopped-deallocated"></a>VM 已停止。 (已解除配置)

**原因**：虛擬機器的狀態不是 [執行中]。

**解決方案**：VM 必須是執行中的狀態，才能將該 VM 上架到解決方案。 按一下 [啟動 VM] 的內嵌連結來啟動 VM (不用離開此頁面)。

## <a name="next-steps"></a>後續步驟

現在，適用您虛擬機器的解決方案已啟用，請瀏覽更新管理的概觀文章，了解如何檢視機器的更新評估。

> [!div class="nextstepaction"]
> [更新管理 - 檢視更新評估](./automation-update-management.md#viewing-update-assessments)

有關解決方案和其使用方式的教學課程：

* [教學課程 - 管理 VM 的更新](automation-tutorial-update-management.md)

* [教學課程 - 識別 VM 上的軟體](automation-tutorial-installed-software.md)

* [教學課程 - 對 VM 的變更進行疑難排解](automation-tutorial-troubleshoot-changes.md)