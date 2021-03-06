---
title: 檢視和篩選 Azure 資源資訊
description: 篩選資訊並使用不同的檢視以進一步了解您的 Azure 資源。
ms.topic: how-to
ms.date: 09/11/2020
ms.openlocfilehash: d1bd00a9e7f8c9c18484378f7c21d3bacdac2d3f
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745888"
---
# <a name="view-and-filter-azure-resource-information"></a>檢視和篩選 Azure 資源資訊

Azure 入口網站可讓您瀏覽所有 Azure 訂用帳戶中關於資源的詳細資訊。 本文說明如何篩選資訊並使用不同的檢視，以進一步了解您的資源。

本文著重於下列螢幕擷取畫面所示的 [所有資源] 畫面。 個別資源類型 (例如虛擬機器) 的畫面會有不同的選項，例如啟動和停止 VM。

:::image type="content" source="media/manage-filter-resource-views/all-resources.png" alt-text="所有資源的 Azure 入口網站檢視":::

## <a name="filter-resources"></a>篩選資源

藉由使用篩選將焦點放在資源子集，以開始探索 [所有資源]。 下列螢幕擷取畫面顯示正在對資源群組進行篩選，並在訂用帳戶的六個資源群組中選取了其中兩個。

:::image type="content" source="media/manage-filter-resource-views/filter-resource-group.png" alt-text="以資源群組為基礎的篩選檢視":::

如下列螢幕擷取畫面所示，您可以合併各種篩選，包括以文字搜尋為基礎的篩選。 在本案例中，結果的範圍會侷限在已選取的兩個資源群組其中之一內包含 "SimpleWinVM" 的資源。

:::image type="content" source="media/manage-filter-resource-views/filter-simplewinvm.png" alt-text="以文字輸入為基礎的篩選檢視":::

若要變更要包含在檢視中的資料行，請依序選取 [管理檢視] 和 [編輯資料行]。

:::image type="content" source="media/manage-filter-resource-views/edit-columns.png" alt-text="編輯檢視中顯示的資料行":::

## <a name="save-use-and-delete-views"></a>儲存、使用和刪除檢視

您可以儲存包含所選篩選和資料行的檢視。 若要儲存和使用檢視：

1. 依序選取 [管理檢視] 和 [儲存檢視]。

1. 輸入檢視的名稱，然後選取 [確定]。 所儲存的檢視便會立即出現在 [管理檢視] 功能表中。

    :::image type="content" source="media/manage-filter-resource-views/simple-view.png" alt-text="所儲存的檢視":::

1. 若要使用檢視，請在 [預設] 和您自己的其中一個檢視之間切換，以查看這會如何影響所顯示的資源清單。

若要刪除檢視：

1. 依序選取 [管理檢視] 和 [瀏覽所有檢視]。

1. 在 ["所有資源" 的已儲存檢視] 窗格中選取檢視，然後選取 [刪除] 圖示 ![刪除檢視圖示](media/manage-filter-resource-views/icon-delete.png)。

## <a name="summarize-resources-with-visuals"></a>使用視覺效果來摘要列出資源

我們到目前為止所探討的檢視都是 _清單檢視_，但另有包含視覺效果的 _摘要檢視_。 您可以儲存和使用這些檢視，方法就和列出檢視時一樣。 篩選會在這兩種類型的檢視之間保存下來。 有標準檢視，像是如下所示的 [位置] 檢視，以及與特定服務相關的檢視，例如 Azure 儲存體的 [狀態] 檢視。

:::image type="content" source="media/manage-filter-resource-views/summary-map.png" alt-text="地圖檢視中的資源摘要":::

若要儲存和使用摘要檢視：

1. 從 [檢視] 功能表中選取 [摘要檢視]。

    :::image type="content" source="media/manage-filter-resource-views/menu-summary-view.png" alt-text="[摘要檢視] 功能表":::

1. [摘要檢視] 可讓您依不同屬性來摘要列出，包括 **位置** 和 **類型**。 選取 [摘要方式] 選項和適當的視覺效果。 下列螢幕擷取畫面顯示具有 [長條圖] 視覺效果的 [類型摘要]。

    :::image type="content" source="media/manage-filter-resource-views/type-summary-bar-chart.png" alt-text="顯示長條圖的類型摘要":::

1. 依序選取 [管理檢視] 和 [儲存] 來儲存此檢視，方法就和清單檢視一樣。

1. 在 [摘要檢視] 的 [類型摘要] 底下，選取圖表中的長條。 選取長條會提供向下篩選至一種資源類型的清單。

    :::image type="content" source="media/manage-filter-resource-views/all-resources-filtered-type.png" alt-text="依類型篩選的所有資源":::

## <a name="run-queries-in-azure-resource-graph"></a>在 Azure Resource Graph 中執行查詢

Azure Resource Graph 提供有效率且高效能的資源瀏覽功能，讓您能夠跨一組訂用帳戶進行大規模查詢。 Azure 入口網站中的 [所有資源] 畫面包含一個連結，可供開啟以目前篩選的檢視為範圍的 Resource Graph 查詢。

若要執行 Resource Graph 查詢：

1. 選取 [開啟查詢]。

    :::image type="content" source="media/manage-filter-resource-views/open-query.png" alt-text="開啟 Azure Resource Graph 查詢":::

1. 在 [Azure Resource Graph Explorer] 中選取 [執行查詢] 以查看結果。

    :::image type="content" source="media/manage-filter-resource-views/run-query.png" alt-text="執行 Azure Resource Graph 查詢":::

    如需詳細資訊，請參閱[使用 Azure Resource Graph Explorer 執行您的第一個 Resource Graph 查詢](../governance/resource-graph/first-query-portal.md)。

## <a name="next-steps"></a>後續步驟

[Azure 入口網站概觀](azure-portal-overview.md)

[在 Azure 入口網站中建立和共用儀表板](azure-portal-dashboards.md)
