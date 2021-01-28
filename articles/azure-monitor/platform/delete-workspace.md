---
title: 刪除和復原 Azure Log Analytics 工作區 | Microsoft Docs
description: 了解如何刪除您的 Log Analytics 工作區 (如果您已在個人訂用帳戶中建立工作區) 或重組您的工作區模型。
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 12/20/2020
ms.openlocfilehash: 5bb072d0ea710e35b4f741836aed143e962a326e
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98927693"
---
# <a name="delete-and-recover-azure-log-analytics-workspace"></a>刪除和復原 Azure Log Analytics 工作區

本文會說明 Azure Log Analytics 工作區虛刪除的概念，以及如何復原已刪除的工作區。

## <a name="considerations-when-deleting-a-workspace"></a>刪除工作區時的考量

當您刪除 Log Analytics 工作區時，會執行虛刪除作業，以允許在 14 天內復原工作區，包括其資料和連線的代理程式 (無論刪除是意外還是刻意進行的)。 在虛刪除期間之後，工作區資源和其資料將無法復原，而且會排入佇列以在30天內完全清除。 該工作區名稱是「已發行」，您可以將其用來建立新的工作區。

> [!NOTE]
> 如果您想要覆寫虛刪除行為，並永久刪除您的工作區，請依照 [永久工作區刪除](#permanent-workspace-delete)中的步驟執行。

刪除工作區時，請小心謹慎，因為可能有會對服務作業造成負面影響的重要資料和設定。 檢查哪些代理程式、解決方案和其他 Azure 服務會將其資料儲存在 Log Analytics 中，例如：

* 管理解決方案
* Azure 自動化
* 在 Windows 與 Linux 虛擬機器上執行的代理程式
* 在您環境中的 Windows 與 Linux 電腦上執行的代理程式
* System Center Operations Manager

虛刪除作業會刪除工作區資源，且任何相關聯的使用者權限都會遭到破壞。 如果這些使用者與其他工作區相關聯，則可以繼續搭配其他工作區使用 Log Analytics。

## <a name="soft-delete-behavior"></a>虛刪除行為

工作區刪除作業會移除工作區的 Resource Manager 資源，但其設定和資料會保留 14 天，同時提供工作區已遭到刪除的樣子。 設定為向工作區報告的任何代理程式和 System Center Operations Manager 管理群組，在虛刪除期間都會維持在孤立狀態。 此服務會進一步提供一種機制讓您復原已刪除的工作區，包括其資料和連結的資源，基本上來說，就是將刪除作業復原。

> [!NOTE] 
> 在刪除時，已安裝的解決方案和連結的服務 (例如 Azure 自動化帳戶) 會從工作區中永久移除，且無法復原。 在進行復原作業後應重新設定這些項目，才能讓工作區進入其先前設定的狀態。

您可以使用 [PowerShell](/powershell/module/azurerm.operationalinsights/remove-azurermoperationalinsightsworkspace)、[REST API](/rest/api/loganalytics/workspaces/delete) 或 [Azure 入口網站](https://portal.azure.com)來刪除工作區。

### <a name="azure-portal"></a>Azure 入口網站

1. 登入 [Azure 入口網站](https://portal.azure.com)。 
2. 在 Azure 入口網站中，選取 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics 工作區]。
3. 在 Log Analytics 工作區的清單中選取某個工作區，然後從中間窗格的頂端按一下 [刪除]。
4. 此時會出現確認頁面，並顯示過去一週擷取到工作區的資料。 
5. 如果您想要永久刪除工作區，移除稍後要復原的選項，請選取 [ **永久刪除工作區** ] 核取方塊。
6. 輸入工作區的名稱來進行確認，然後按一下 [刪除]。

   ![確認刪除工作區](media/delete-workspace/workspace-delete.png)

### <a name="powershell"></a>PowerShell
```PowerShell
PS C:\>Remove-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name" -Name "workspace-name"
```

## <a name="permanent-workspace-delete"></a>永久刪除工作區
虛刪除方法可能無法適合某些情況，例如在開發和測試時，您需要重複地使用相同設定和工作區名稱來進行部署。 在這類情況下，您可以永久刪除工作區，並「覆寫」虛刪除期間。 永久刪除工作區的作業會釋出工作區名稱，然後您就可以使用相同名稱來建立新的工作區。

> [!IMPORTANT]
> 請小心使用永久刪除工作區作業，此作業無法復原，而且您將無法復原工作區和其資料。

若要使用 Azure 入口網站永久刪除工作區，請選取 [ **永久刪除工作區** ] 核取方塊，然後按一下 [ **刪除** ] 按鈕。

若要使用 PowerShell 永久刪除工作區，請新增 '-ForceDelete ' 標記以永久刪除您的工作區。 '-ForceDelete ' 選項目前可搭配 Az. OperationalInsights 2.3.0 或更高版本使用。 

```powershell
PS C:\>Remove-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name" -Name "workspace-name" -ForceDelete
```

## <a name="recover-workspace"></a>復原工作區
當您意外或刻意刪除 Log Analytics 工作區時，服務會將工作區置於虛刪除狀態，使其無法供任何作業存取。 在虛刪除期間，已刪除工作區的名稱會保留起來，而且無法用於建立新的工作區。 在虛刪除期間過後，工作區就無法復原，其會排程進行永久刪除，其名稱會釋放出來而可用於建立新的工作區。

您可以在虛刪除期間復原工作區，包括其資料、設定和已連線的代理程式。 在虛刪除作業之前，您需要有工作區所在訂用帳戶和資源群組的參與者權限。 工作區復原的執行方式是重新建立 Log Analytics 工作區，並提供已刪除工作區的詳細資料，包括：

- 訂用帳戶識別碼
- 資源群組名稱
- 工作區名稱
- 區域

> [!IMPORTANT]
> 如果您的工作區已在資源群組刪除操作中刪除，您必須先重新建立資源群組。

### <a name="azure-portal"></a>Azure 入口網站

1. 登入 [Azure 入口網站](https://portal.azure.com)。 
2. 在 Azure 入口網站中，選取 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics 工作區]。 您會在選取的範圍中看到您所擁有的工作區清單。
3. 按一下左上方功能表上的 [復原] 以開啟某個頁面，內有可復原的虛刪除狀態工作區。

   ![[Log Analytics 工作區] Azure 入口網站畫面的螢幕擷取畫面，其中已在功能表列上反白顯示 [復原]。](media/delete-workspace/recover-menu.png)

4. 選取工作區，然後按一下 [復原] 來復原該工作區。

   ![[復原已刪除的 Log Analytics 工作區] Azure 入口網站對話方塊的螢幕擷取畫面，其中已反白顯示工作區並選取 [復原] 按鈕。](media/delete-workspace/recover-workspace.png)


### <a name="powershell"></a>PowerShell
```PowerShell
PS C:\>Select-AzSubscription "subscription-name-the-workspace-was-in"
PS C:\>New-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name-the-workspace-was-in" -Name "deleted-workspace-name" -Location "region-name-the-workspace-was-in"
```

工作區及其所有資料都會在復原作業之後恢復過來。 解決方案和已連結的服務在遭到刪除時就已從工作區中永久移除，請重新設定這些項目，才能讓工作區進入其先前設定的狀態。 在工作區復原後，某些資料可能無法用於查詢，直到您重新安裝相關聯的解決方案，並將其結構描述新增至工作區後才可進行。

## <a name="troubleshooting"></a>疑難排解

您至少必須擁有「Log Analytics 參與者」權限才能刪除工作區。

* 如果您不確定已刪除的工作區處於虛刪除狀態且可加以復原，請按一下 [在 *Log Analytics 工作區* 中 [開啟回收站](#recover-workspace)] 頁面，以查看每個訂用帳戶的虛刪除工作區清單。 清單中不包含永久刪除的工作區。
* 如果您在建立工作區時收到錯誤訊息「此工作區名稱已在使用中」或「衝突」，可能是因為：
  * 工作區名稱無法使用，組織中的某人或其他客戶正在使用中。
  * 工作區已在最近 14 天內刪除，而其名稱已針對虛刪除期間保留起來。 若要覆寫虛刪除並永久刪除您的工作區，以建立具有相同名稱的新工作區，請遵循下列步驟來先復原工作區，然後執行永久刪除：<br>
    1. [復原](#recover-workspace)您的工作區。
    2. [永久刪除](#permanent-workspace-delete)您的工作區。
    3. 使用相同工作區名稱建立新的工作區。
 
      在後端成功完成刪除呼叫之後，您可以還原工作區，並在先前建議的其中一個方法中完成永久刪除作業。

* 如果您在刪除工作區時取得204回應碼，但 *卻找不到資源* ，則可能會發生連續的重試作業。 204是空回應，這通常表示資源不存在，所以刪除完成，而不執行任何動作。
* 如果您刪除資源群組和工作區，您可以在 [ [開啟回收站](#recover-workspace) ] 頁面中看到已刪除的工作區，但由於資源群組不存在，復原作業會失敗並顯示錯誤碼 404--請重新建立您的資源群組，然後再次嘗試復原。
