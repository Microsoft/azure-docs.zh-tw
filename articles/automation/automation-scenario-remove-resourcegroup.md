---
title: 使用 Azure 自動化自動移除資源群組
description: Azure 自動化案例的 PowerShell 工作流程版本，包括可移除訂用帳戶中所有資源群組的 Runbook。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/19/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: f0ec42a1e62c4aa35bb6bdffce0acf00a971cb7d
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34194514"
---
# <a name="azure-automation-scenario---automate-removal-of-resource-groups"></a>Azure 自動化案例 - 自動移除資源群組
許多客戶會建立一個以上的資源群組。 有些可能會用來管理生產應用程式，其他的可能會用來作為開發、測試和預備環境。 自動部署這些資源是一回事，但按一下按鈕就能夠將資源群組解除委任則是另一回事。 您可以使用 Azure 自動化來簡化這個一般管理工作。 如果您使用的 Azure 訂用帳戶因為成員優惠 (像是 MSDN 或 Microsoft Partner Network Cloud Essentials 方案) 而有消費限制，則這會很有幫助。

此案例是以 PowerShell Runbook 為基礎，其設計目的是要移除一個或多個您在訂用帳戶中指定的資源群組。 Runbook 的預設設定是先測試後再繼續。 這可確保您在準備好完成此程序之前，不會不小心刪除資源群組。   

## <a name="getting-the-scenario"></a>取得案例
此案例包含您可以從 [PowerShell 資源庫](https://www.powershellgallery.com/packages/Remove-ResourceGroup/1.0/DisplayScript)下載的 PowerShell Runbook。 您也可以直接從 Azure 入口網站中的 [Runbook 資源庫](automation-runbook-gallery.md) 將它匯入。<br><br>

| Runbook | 說明 |
| --- | --- |
| Remove-ResourceGroup |從訂用帳戶移除一個或多個 Azure 資源群組和相關聯的資源。 |

<br>
此 Runbook 定義了下列輸入參數︰

| 參數 | 說明 |
| --- | --- |
| NameFilter (必要) |指定名稱篩選器以限制您要刪除的資源群組。 您可以使用逗號分隔清單傳遞多個值。<br>此篩選器不區分大小寫，而且會比對任何包含字串的資源群組。 |
| PreviewMode (選擇性) |執行 Runbook 以查看會刪除哪些資源群組，但不採取任何動作。<br>預設值是 **true**，可協助避免意外刪除一個或多個傳遞至 Runbook 的資源群組。 |

## <a name="install-and-configure-this-scenario"></a>安裝和設定此案例
### <a name="prerequisites"></a>先決條件
此 Runbook 需使用 [Azure 執行身分帳戶](automation-sec-configure-azure-runas-account.md)進行驗證。    

### <a name="install-and-publish-the-runbooks"></a>安裝並發佈 Runbook
下載 Runbook 之後，您可以使用[匯入 Runbook 程序](automation-creating-importing-runbook.md#importing-a-runbook-from-a-file-into-azure-automation)中的程序加以匯入。 在 Runbook 成功匯入到您的自動化帳戶之後予以發佈。

## <a name="using-the-runbook"></a>使用 Runbook
下列步驟會引導您執行此 Runbook，並協助您熟悉其運作方式。 在此範例中，您只會測試 Runbook，而不會實際刪除資源群組。  

1. 從 Azure 入口網站中，開啟您的自動化帳戶，然後按一下 [Runbook]。
2. 選取 **Remove-ResourceGroup** Runbook，然後按一下 [啟動]。
3. 當您啟動 Runbook 時，[啟動 Runbook] 頁面會開啟，可供您設定參數。 輸入資源群組的名稱，這些是您訂用帳戶中可用來測試且如果意外刪除也不會造成損害的資源群組。

   > [!NOTE]
   > 請確定 [預覽模式] 選項已設定為 [true]，以免刪除選取的資源群組。 此 Runbook 不會移除執行此 Runbook 之「自動化」帳戶所在的資源群組。  
   >
   >
1. 設定完所有參數值之後，按一下 [確定]，然後 Runbook 就會排入佇列等候執行。  

若要在 Azure 入口網站中檢視 **Remove-ResourceGroup** Runbook 作業的詳細資料，請在 [資源] 底下選取 Runbook 中的 [作業]。 選取您想要檢視的作業。 除了作業和任何發生的例外狀況一般資訊以外，作業摘要也會顯示輸入參數和輸出串流。<br> ![Remove-ResourceGroup Runbook 作業狀態](media/automation-scenario-remove-resourcegroup/remove-resourcegroup-runbook-job-status.png)。

[作業摘要]  包括來自輸出、警告和錯誤串流的訊息。 選取 [輸出] 以檢視 Runbook 執行的詳細結果。<br> ![Remove-ResourceGroup Runbook 輸出結果](media/automation-scenario-remove-resourcegroup/remove-resourcegroup-runbook-job-output.png)

## <a name="next-steps"></a>後續步驟
* 若要開始建立您自己的 Runbook，請參閱 [在 Azure 自動化中建立或匯入 Runbook](automation-creating-importing-runbook.md)。
* 若要開始使用 PowerShell 工作流程 Runbook，請參閱[我的第一個 PowerShell 工作流程 Runbook](automation-first-runbook-textual.md)。
