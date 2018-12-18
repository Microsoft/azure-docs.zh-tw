---
title: 如何更新現有的 Azure 藍圖指派
description: 了解在 Azure 藍圖中更新現有指派的機制。
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: ecac0fb21a6691874d5e8db49eadd7114d41845f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956195"
---
# <a name="how-to-update-an-existing-blueprint-assignment"></a>如何更新現有的藍圖指派

指派藍圖時，可以更新指派。 更新現有指派的原因有數個，包括：

- 新增或移除[資源鎖定](../concepts/resource-locking.md)
- 變更[動態參數](../concepts/parameters.md#dynamic-parameters)的值
- 將指派升級至藍圖的較新**已發佈**版本

## <a name="updating-assignments"></a>更新指派

1. 在左窗格中按一下 [所有服務]，再搜尋並選取 [原則]，即可在 Azure 入口網站中啟動 Azure 藍圖服務。 請在 [原則] 頁面上，按一下 [藍圖]。

1. 從左邊的頁面選取 [指派藍圖]。

1. 在藍圖清單中，以滑鼠左鍵按一下藍圖指派，然後按一下 [更新指派] 按鈕，或是以滑鼠右鍵按一下藍圖指派，然後選取 [更新指派]。

   ![更新指派](../media/update-existing-assignments/update-assignment.png)

1. [指派藍圖] 頁面隨即會載入，並預先填入原始指派的所有值。 您可以變更**藍圖定義版本**、**鎖定指派**狀態，以及任何存在於藍圖定義的動態參數。 完成變更後，按一下 [指派]。

1. 在更新後的指派詳細資料頁面上，查看新的狀態。 在此範例中，我們對指派新增了**鎖定**。

   ![更新的指派 - 鎖定](../media/update-existing-assignments/updated-assignment.png)

1. 使用下拉式清單探索有關其他**指派作業**的詳細資料。 **受控資源**的資料表會依所選指派作業進行更新。

   ![指派作業](../media/update-existing-assignments/assignment-operations.png)

## <a name="rules-for-updating-assignments"></a>用於更新指派的規則

所更新指派的部署會遵循幾個重要規則。 這些規則會根據所要求的變更以及所要部署或更新的成品資源類型，來決定現有資源會發生什麼事。

- 角色指派
  - 如果角色或角色受託人 (使用者、群組或應用程式) 有所變更，便會建立新的角色指派。 先前已部署的角色指派會留在原處。
- 原則指派
  - 如果原則指派參數有所變更，則現有指派也會更新。
  - 如果原則指派定義有所變更，則會建立新的原則指派。 先前已部署的原則指派會留在原處。
  - 如果從藍圖中移除原則指派成品，則先前部署的原則指派會留在原處。
- Azure 資源管理員範本
  - 範本會透過 Resource Manager 處理為 **PUT**。 因為每個資源類型對此的處理方式不同，請檢閱每個所含資源的文件，以判斷藍圖在執行此動作時會有什麼影響。

## <a name="possible-errors-on-updating-assignments"></a>更新指派時的可能錯誤

在更新指派時，所進行的變更可能會在執行時中斷。 例如，在資源群組部署好之後，變更其位置。 只要是 [Azure Resource Manager](../../../azure-resource-manager/resource-group-overview.md) 所支援的變更均可進行，但會透過 Azure Resource Manager 導致錯誤發生的變更，也會導致指派失敗。

指派的更新次數沒有限制。 因此，如果發生錯誤，不論原因是參數不正確、物件早已存在或 Azure Resource Manager 不允許該變更，請確定錯誤所在，並再次對指派進行更新。

## <a name="next-steps"></a>後續步驟

- 了解[藍圖生命週期](../concepts/lifecycle.md) (英文)
- 了解如何使用[靜態和動態參數](../concepts/parameters.md) (英文)
- 了解如何自訂[藍圖排序順序](../concepts/sequencing-order.md) (英文)
- 了解如何使用[藍圖資源鎖定](../concepts/resource-locking.md) (英文)
- 使用[一般疑難排解](../troubleshoot/general.md)來解決藍圖指派期間發生的問題
