---
title: 整合 Azure 自動化與事件格線 | Microsoft Docs
description: 了解如何在新的 VM 建立時自動加入標記，並傳送通知給 Microsoft Teams。
keywords: 自動化, Runbook, 小組, 事件格線, 虛擬機器, VM
services: automation
documentationcenter: ''
author: eamonoreilly
manager: ''
editor: ''
ms.service: automation
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/06/2017
ms.author: eamono
ms.openlocfilehash: 6270f8bad893798f46d8db91e7b1140b6a125350
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39049859"
---
# <a name="integrate-azure-automation-with-event-grid-and-microsoft-teams"></a>將 Azure 自動化與事件格線和 Microsoft Teams 整合

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 匯入事件格線範例 Runbook。
> * 建立選用的 Microsoft Teams Webhook。
> * 建立 Runbook 的 Webhook。
> * 建立事件格線訂用帳戶。
> * 建立觸發 Runbook 的 VM。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，需要有 [Azure 自動化帳戶](../automation/automation-offering-get-started.md)，才能保留從 Azure 事件格線訂用帳戶觸發的 Runbook。

* 需將 `AzureRM.Tags` 模組載入自動化帳戶中，請參閱[如何將模組匯入 Azure 自動化](../automation/automation-update-azure-modules.md)，了解如何將模組匯入 Azure 自動化。

## <a name="import-an-event-grid-sample-runbook"></a>匯入事件格線範例 Runbook

1. 選取您的自動化帳戶，然後選取 [Runbook] 頁面。

   ![選取 Runbook](./media/ensure-tags-exists-on-new-virtual-machines/select-runbooks.png)

2. 選取 [瀏覽資源庫] 按鈕。

3. 搜尋**事件格線**，然後選取 [整合 Azure 自動化與事件格線]。

    ![匯入資源庫 Runbook](media/ensure-tags-exists-on-new-virtual-machines/gallery-event-grid.png)

4. 選取 [匯入]，並將其命名為 **Watch-VMWrite**。

5. 匯入之後，選取 [編輯] 以檢視 Runbook 來源。 選取 [發佈] 按鈕。

> [!NOTE]
> 指令碼中第 74 行需有變更為 `Update-AzureRmVM -ResourceGroupName $VMResourceGroup -VM $VM -Tag $Tag | Write-Verbose` 的這一行。 `-Tags` 參數現在是 `-Tag`。

## <a name="create-an-optional-microsoft-teams-webhook"></a>建立選用的 Microsoft Teams Webhook

1. 在 Microsoft Teams 中，選取通道名稱旁的 [更多選項]，然後選取 [連接器]。

    ![Microsoft Teams 連線](media/ensure-tags-exists-on-new-virtual-machines/teams-webhook.png)

2. 將連接器清單捲動至 [傳入的 Webhook]，並選取 [新增]。

3. 輸入 **AzureAutomationIntegration** 作為名稱，然後選取 [建立]。

4. 將 Webhook 複製到剪貼簿並加以儲存。 Webhook URL 是用來將資訊傳送給 Microsoft Teams。

5. 選取 [完成] 以儲存此 Webhook。

## <a name="create-a-webhook-for-the-runbook"></a>建立 Runbook 的 Webhook

1. 開啟 Watch-VMWrite Runbook。

2. 選取 [Webhook]，然後選取 [新增 Webhook] 按鈕。

3. 輸入 **WatchVMEventGrid** 作為名稱。 將 URL 複製到剪貼簿並加以儲存。

    ![設定 Webhook 名稱](media/ensure-tags-exists-on-new-virtual-machines/copy-url.png)

4. 選取 [設定參數並執行設定]，並針對 **CHANNELURL** 輸入 Microsoft Teams Webhook URL。 保留 **WEBHOOKDATA** 空白。

    ![設定 Webhook 參數](media/ensure-tags-exists-on-new-virtual-machines/configure-webhook-parameters.png)

5. 選取 [確定] 以建立自動化 Runbook Webhook。


## <a name="create-an-event-grid-subscription"></a>建立事件格線訂用帳戶
1. 在 [自動化帳戶] 概觀頁面上，選取 [事件格線]。

    ![選取 Event Grid](media/ensure-tags-exists-on-new-virtual-machines/select-event-grid.png)

2. 選取 [+ 事件訂閱] 按鈕。

3. 使用下列資訊來設定訂用帳戶：

    *   輸入 **AzureAutomation** 作為名稱。
    *   在 [主題類型] 中，選取 [Azure 訂用帳戶]。
    *   清除 [訂閱所有事件類型] 核取方塊。
    *   在 [事件類型] 中，選取 [資源寫入成功]。
    *   在 [訂閱者端點] 中，輸入 Watch-VMWrite Runbook 的 Webhook URL。
    *   在 [前置詞篩選] 中，輸入您要在其中建立新 VM 的訂用帳戶和資源群組。 它看起來應該像：`/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines`

4. 選取 [建立] 以儲存事件格線訂用帳戶。

## <a name="create-a-vm-that-triggers-the-runbook"></a>建立觸發 Runbook 的 VM
1. 在事件格線訂用帳戶 [前置詞篩選] 中您所指定的資源群組內，建立新的 VM。

2. 這應該會呼叫 Watch-VMWrite Runbook，並將新的標記新增至 VM。

    ![VM 標記](media/ensure-tags-exists-on-new-virtual-machines/vm-tag.png)

3. 新的訊息會傳送至 Microsoft Teams 通道。

    ![Microsoft Teams 通知](media/ensure-tags-exists-on-new-virtual-machines/teams-vm-message.png)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您將設定事件格線和自動化之間的整合。 您已了解如何︰

> [!div class="checklist"]
> * 匯入事件格線範例 Runbook。
> * 建立選用的 Microsoft Teams Webhook。
> * 建立 Runbook 的 Webhook。
> * 建立事件格線訂用帳戶。
> * 建立觸發 Runbook 的 VM。

> [!div class="nextstepaction"]
> [使用 Event Grid 建立和路由傳送自訂事件](../event-grid/custom-event-quickstart.md)
