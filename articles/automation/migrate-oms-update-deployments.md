---
title: 將您的 OMS 更新部署遷移至 Azure
description: 本文說明如何將您現有的 OMS 更新部署遷移至 Azure
services: automation
ms.service: automation
ms.component: update-management
author: georgewallace
ms.author: gwallace
ms.date: 07/16/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: d0b380aa6046daa235098516a8c93d3ba72533a6
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42141072"
---
# <a name="migrate-your-oms-update-deployments-to-azure"></a>將您的 OMS 更新部署遷移至 Azure

Operations Management Suite (OMS) 入口網站已被[取代](../log-analytics/log-analytics-oms-portal-transition.md)。 OMS 入口網站中可供更新管理使用的所有功能都可在 Azure 入口網站中使用。 本文提供您遷移至 Azure 入口網站所需的資訊。

## <a name="key-information"></a>重要資訊

* 現有的部署將會繼續運作。 一旦您在 Azure 中重新建立部署之後，就可從 OMS 刪除舊的部署。
* 您在 OMS 中擁有的所有現有功能都可在 Azure 中使用，若要深入了解更新管理，請參閱[更新管理概觀](automation-update-management.md)。

## <a name="access-the-azure-portal"></a>存取 Azure 入口網站

從您的 OMS 工作區，按一下 [在 Azure 中開啟]。 這會瀏覽至 OMS 所使用的 Log Analytics 工作區。

![在 Azure 中開啟 - OMS 入口網站](media/migrate-oms-update-deployments/link-to-azure-portal.png)

在 Azure 入口網站中，按一下 [自動化帳戶]。

![Log Analytics](media/migrate-oms-update-deployments/log-analytics.png)

在您的自動化帳戶中，選取 [更新管理] 以開啟更新管理。

![更新管理](media/migrate-oms-update-deployments/azure-automation.png)

未來您可直接前往 Azure 入口網站，在 [所有服務] 下方，選取 [管理工具] 下方的 [自動化帳戶]、選取適當的自動化帳戶，然後按一下 [更新管理]。

## <a name="recreate-existing-deployments"></a>重新建立現有的部署

在 OMS 入口網站中建立的所有更新部署都有一個[已儲存的搜尋](../log-analytics/log-analytics-computer-groups.md) (也稱為電腦群組)，且名稱與現有的更新部署相同。 已儲存的搜尋包含已在更新部署中排程的電腦清單。

![更新管理](media/migrate-oms-update-deployments/oms-deployment.png)

若要使用這個現有的已儲存搜尋，請遵循下列步驟：

若要建立新的更新部署，移至 Azure 入口網站、選取要使用的自動化帳戶，然後按一下 [更新管理]。 按一下 [排程更新部署]。

![排程更新部署](media/migrate-oms-update-deployments/schedule-update-deployment.png)

[新增更新部署] 窗格隨即開啟。 為下表描述的屬性輸入相關的值，然後按一下 [建立]：

針對要更新的電腦，選取現有 OMS 部署所使用的已儲存搜尋。

| 屬性 | 說明 |
| --- | --- |
|名稱 |用以識別更新部署的唯一名稱。 |
|作業系統| 選取 [Linux] 或 [Windows]。|
|要更新的機器 |選取已儲存的搜尋、已匯入的群組，或從下拉式清單中選擇 [機器]，然後選取個別的機器。 如果您選擇 [機器]，機器的整備程度會顯示於 [更新代理程式整備程度] 欄中。</br> 若要深入了解在 Log Analytics 中建立電腦群組的不同方法，請參閱 [Log Analytics 中的電腦群組](../log-analytics/log-analytics-computer-groups.md) |
|更新分類|選取您需要的所有更新分類。 CentOS 預設不支援這種更新分類。|
|要排除的更新|輸入要排除的更新。 針對 Windows，輸入不含 **KB** 前置詞的 KB 文章。 針對 Linux，輸入套件名稱或使用萬用字元。  |
|排程設定|選取開始時間，然後選取 [一次] 或 [週期性] 以定期執行。|| 維護時間範圍 |為更新設定的分鐘數。 此值不可小於 30 分鐘，且不可超過 6 小時。 |
| 維護時間範圍 |為更新設定的分鐘數。 此值不可小於 30 分鐘，且不可超過 6 小時 |
| 重新開機控制| 決定應該如何處理重新開機。</br>可用選項包括：</br>在必要時重新開機 (預設值)</br>一律重新開機</br>永不重新開機</br>僅重新開機 - 將不會安裝更新|

按一下 [排程的更新部署]，以檢視新建立的更新部署狀態。

![新增更新部署](media/migrate-oms-update-deployments/new-update-deployment.png)

如前所述，當您透過 Azure 入口網站設定新的部署之後，就可以從 OMS 入口網站移除現有的部署。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 中的更新管理，請參閱[更新管理](automation-update-management.md)。