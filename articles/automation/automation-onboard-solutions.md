---
title: 將更新與變更追蹤解決方案上線至 Azure 自動化
description: 了解如何將更新與變更追蹤解決方案上線至 Azure 自動化。
services: automation
ms.service: automation
author: eamonoreilly
ms.author: eamono
manager: carmonm
ms.topic: tutorial
ms.date: 05/10/2018
ms.custom: mvc
ms.openlocfilehash: 5d03ec7c5e0aaa68c1db34fedbd428f264e49e43
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34830515"
---
# <a name="onboard-update-and-change-tracking-solutions-to-azure-automation"></a>將更新與變更追蹤解決方案上線至 Azure 自動化

在本教學課程中，您會了解如何將適用於 VM 的更新、變更追蹤與清查解決方案上線至 Azure 自動化：

> [!div class="checklist"]
> * 使 Azure 虛擬機器上線
> * 啟用解決方案
> * 安裝和更新模組
> * 匯入上線 Runbook
> * 啟動 Runbook

## <a name="prerequisites"></a>先決條件

若要完成此教學課程，需要有下列項目：

* Azure 訂用帳戶。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [自動化帳戶](automation-offering-get-started.md)，以管理電腦。
* 要上架的[虛擬機器](../virtual-machines/windows/quick-create-portal.md)。

## <a name="onboard-an-azure-vm"></a>使 Azure 虛擬機器上線

讓機器上線的方式有很多種，您可以[從虛擬機器](automation-onboard-solutions-from-vm.md)、[從瀏覽多部電腦](automation-onboard-solutions-from-browse.md)、[從您的自動化帳戶](automation-onboard-solutions-from-automation-account.md)，或透過 Runbook 來讓解決方案上線。 本教學課程會引導您透過 Runbook 啟用「更新管理」。 若要使 Azure 虛擬機器大規模上線，現有的虛擬機器必須使用變更追蹤或更新管理解決方案來讓現有虛擬機器上線。 在此步驟中，您會使用更新管理和變更追蹤來讓虛擬機器上線。

### <a name="enable-change-tracking-and-inventory"></a>啟用變更追蹤和清查

變更追蹤和清查解決方案可讓您在虛擬機器上[追蹤變更](automation-vm-change-tracking.md)和[進行清查](automation-vm-inventory.md)。 在此步驟中，您會在虛擬機器上啟用解決方案。

1. 從左側功能表中，選取 [自動化帳戶]，然後在清單中選取您的自動化帳戶。
1. 在 [組態管理] 下選取 [清查]。
1. 選取現有的 Log Analytics 工作區，或建立新的工作區。 按一下 [啟用] 按鈕。

![使更新解決方案上線](media/automation-onboard-solutions/inventory-onboard.png)

當變更追蹤與清查解決方案上線通知完成時，按一下 [組態管理] 下的 [更新管理]。

### <a name="enable-update-management"></a>啟用更新管理

更新管理解決方案可讓您管理 Azure Windows 虛擬機器的更新和修補程式。 您可以評估可用更新的狀態、排定何時安裝必要的更新，並檢閱部署結果來確認更新已成功套用至虛擬機器。 在此步驟中，您會為您的虛擬機器啟用解決方案。

1. 從您的自動化帳戶中，選取 [更新管理] 下的 [更新管理]。
1. 選取的記錄分析工作區與上一個步驟中使用的工作區相同。 按一下 [啟用]，讓更新管理解決方案上線。

![使更新解決方案上線](media/automation-onboard-solutions/update-onboard.png)

更新管理解決方案正在安裝時，您會看到藍色橫幅。 解決方案啟用後，請選取 [組態管理] 下的 [變更追蹤] 前往下一個步驟。

### <a name="select-azure-vm-to-be-managed"></a>選取要管理的 Azure 虛擬機器

現在解決方案已啟用，您可以新增要上線至這些解決方案的 Azure 虛擬機器。

1. 從您的自動化帳戶中，在 [變更追蹤] 頁面上選取 [+ 新增 Azure 虛擬機器] 來新增您的虛擬機器。

1. 從清單中選取您的虛擬機器，並選取 [啟用]。 此動作會啟用虛擬機器的變更追蹤和清查解決方案。

   ![啟用虛擬機器的更新解決方案](media/automation-onboard-solutions/enable-change-tracking.png)

1. 完成虛擬機器上線通知後，從您的自動化帳戶中選取 [更新管理] 下的 [更新管理]。

1. 選取 [+ 新增 Azure 虛擬機器] 來新增您的虛擬機器。

1. 從清單中選取您的虛擬機器，並選取 [啟用]。 此動作會啟用虛擬機器的更新管理解決方案。

   ![啟用虛擬機器的更新解決方案](media/automation-onboard-solutions/enable-update.png)

> [!NOTE]
> 如果您沒有等待其他解決方案完成，在啟用下一個解決方案時，您會收到內容如下的訊息：此虛擬機器或不同虛擬機器上正在安裝另一個解決方案。安裝完成時，即可使用 [啟用] 按鈕，然後您就可以要求在這部虛擬機器上安裝解決方案。

## <a name="install-and-update-modules"></a>安裝和更新模組

需要更新為最新的 Azure 模組並匯入 `AzureRM.OperationalInsights`，才能成功將解決方案上線自動化。

## <a name="update-azure-modules"></a>更新 Azure 模組

從您的自動化帳戶中，選取 [共用資源] 下的 [模組]。 選取 [更新 Azure 模組]，可將 Azure 模組更新為最新版本。 在提示中選取 [是]，可將所有現有的 Azure 模組更新為最新版本。

![更新模組](media/automation-onboard-solutions/update-modules.png)

### <a name="install-azurermoperationalinsights-module"></a>安裝 AzureRM.OperationalInsights 模組

從 [模組] 頁面上，選取 [瀏覽資源庫] 以開啟模組資源庫。 搜尋 AzureRM.OperationalInsights，並將此模組匯入至自動化帳戶。

![匯入 OperationalInsights 模組](media/automation-onboard-solutions/import-operational-insights-module.png)

## <a name="import-the-onboarding-runbook"></a>匯入上線 Runbook

1. 從您的自動化帳戶中，選取 [程序自動化] 下的 [Runbook]。
1. 選取 [瀏覽資源庫]。
1. 搜尋 [更新與變更追蹤]，按一下 [Runbook] 並在 [檢視來源] 頁面上選取 [匯入]。 選取 [確定]，然後將 Runbook 匯入至自動化帳戶。

  ![匯入上線 Runbook](media/automation-onboard-solutions/import-from-gallery.png)

1. 在 [Runbook] 頁面上，選取 [編輯]，然後選取 [發佈]。 在 [發佈 Runbook] 對話方塊中，選取 [是] 以發佈 Runbook。

## <a name="start-the-runbook"></a>啟動 Runbook

您必須已將變更追蹤或更新解決方案上線至 Azure VM，才能啟動這個 Runbook。 它需要目前具有已針對參數上線之解決方案的虛擬機器和資源群組。

1. 開啟 Enable-MultipleSolution Runbook。

   ![多個解決方案 Runbook](media/automation-onboard-solutions/runbook-overview.png)

1. 按一下 [開始] 按鈕，並針對參數輸入下列值。

   * **VMNAME** - 保留為空白。 要上線至更新或變更追蹤解決方案的現有 VM 名稱。 藉由將此保留空白，系統會將資源群組中的所有虛擬機器上線。
   * **VMRESOURCEGROUP** - 要上線的虛擬機器資源群組名稱。
   * **SUBSCRIPTIONID** - 保留為空白。 要上線之新虛擬機器的訂用帳戶識別碼。 若保留空白，系統會使用工作區的訂用帳戶。 指定不同的訂用帳戶識別碼時，也應該將此自動化帳戶的 RunAs 帳戶新增為此訂用帳戶的參與者。
   * **ALREADYONBOARDEDVM** - 已手動上線至更新或變更追蹤解決方案的虛擬機器名稱。
   * **ALREADYONBOARDEDVMRESOURCEGROUP** - 虛擬機器所屬的資源群組名稱。
   * **SOLUTIONTYPE** - 輸入 **Updates** 或 **ChangeTracking**

   ![Enable-MultipleSolution Runbook 參數。](media/automation-onboard-solutions/runbook-parameters.png)

1. 選取 [確定] 以啟動 Runbook 作業。
1. 在 Runbook 作業頁面上監視進度和任何錯誤。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 手動將 Azure 虛擬機器上線。
> * 安裝及更新所需的 Azure 模組。
> * 匯入能將 Azure VM 上線的 Runbook。
> * 啟動會自動將 Azure VM 上線的 Runbook。

遵循此連結，即可深入了解 Runbook 排程。

> [!div class="nextstepaction"]
> [為 RunBook 安排排程](automation-schedules.md)。
