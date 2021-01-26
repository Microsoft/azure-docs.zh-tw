---
title: 使用 Azure Sentinel watchlists
description: 本文說明如何使用 Azure Sentinel watchlists 調查威脅、匯入商務資料、建立允許清單和擴充事件資料。
services: sentinel
author: yelevin
ms.author: yelevin
ms.assetid: 1721d0da-c91e-4c96-82de-5c7458df566b
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: conceptual
ms.custom: mvc
ms.date: 09/06/2020
ms.openlocfilehash: e31128687cfcc1f4e32879328ad3227182efb9ce
ms.sourcegitcommit: 95c2cbdd2582fa81d0bfe55edd32778ed31e0fe8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98797366"
---
# <a name="use-azure-sentinel-watchlists"></a>使用 Azure Sentinel watchlists

> [!IMPORTANT]
> Watchlists 功能目前為 **預覽** 狀態。 請參閱 [Microsoft Azure 預覽的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) ，以取得適用于 Azure 功能（Beta、預覽或尚未發行正式運作）的其他法律條款。

Azure Sentinel watchlists 可讓您從外部資料源收集資料，以便與您 Azure Sentinel 環境中的事件相互關聯。 一旦建立之後，您就可以在搜尋、偵測規則、威脅搜尋和回應手冊中使用 watchlists。 Watchlists 會以名稱/值組的形式儲存在 Azure Sentinel 工作區中，並快取以獲得最佳查詢效能和低延遲。

使用 watchlists 的常見案例包括：

- 透過快速匯入 IP 位址、檔案雜湊和 CSV 檔案中的其他資料，快速 **調查威脅** 並回應事件。 匯入之後，您就可以在警示規則、威脅搜尋、活頁簿、筆記本和一般查詢中使用關注清單的名稱/值組來進行聯結和篩選。

- 將 **商務資料匯入** 為 watchlists。 例如，匯入具有特殊許可權的系統存取權的使用者清單或終止的員工，然後使用關注清單來建立允許和拒絕清單，用來偵測或防止這些使用者登入網路。

- **減少警示疲勞**。 建立允許清單來隱藏使用者群組中的警示，例如來自已授權 IP 位址的使用者，這些使用者會執行通常會觸發警示的工作，以及防止良性事件變成警示。

- 擴充 **事件資料**。 使用 watchlists，透過衍生自外部資料源的名稱/值組合來擴充您的事件資料。

## <a name="create-a-new-watchlist"></a>建立新的關注清單

1. 在 Azure 入口網站中，流覽至 [ **Azure Sentinel** 設定關注清單]，  >    >  然後選取 [**加入新** 的]。

    > [!div class="mx-imgBorder"]
    > ![新關注清單](./media/watchlists/sentinel-watchlist-new.png)

1. 在 [ **一般** ] 頁面上，提供關注清單的 [名稱]、[描述] 和 [別名]，然後選取 **[下一步]**。

    > [!div class="mx-imgBorder"]
    > ![關注清單一般頁面](./media/watchlists/sentinel-watchlist-general.png)

1. 在 [ **來源** ] 頁面上，選取資料集類型、上傳檔案，然後選取 **[下一步]**。

    :::image type="content" source="./media/watchlists/sentinel-watchlist-source.png" alt-text="關注清單來源頁面" lightbox="./media/watchlists/sentinel-watchlist-source.png":::

    > [!NOTE]
    >
    > 檔案上傳目前僅限於大小上限為 3.8 MB 的檔案。

1. 檢查資訊、確認資訊正確無誤，然後選取 [ **建立**]。

    > [!div class="mx-imgBorder"]
    > ![關注清單審核頁面](./media/watchlists/sentinel-watchlist-review.png)

    建立關注清單之後，就會出現通知。

    :::image type="content" source="./media/watchlists/sentinel-watchlist-complete.png" alt-text="關注清單成功的建立通知" lightbox="./media/watchlists/sentinel-watchlist-complete.png":::

## <a name="use-watchlists-in-queries"></a>在查詢中使用 watchlists

1. 在 Azure 入口網站中，流覽至 [ **Azure Sentinel** 設定  >    >  **關注清單**]，選取您要使用的關注清單，然後選取 [**在 Log Analytics 中查看**]。

    :::image type="content" source="./media/watchlists/sentinel-watchlist-queries-list.png" alt-text="在查詢中使用 watchlists" lightbox="./media/watchlists/sentinel-watchlist-queries-list.png":::

1. 系統會自動為您的查詢解壓縮關注清單中的專案，而且會出現在 [ **結果** ] 索引標籤上。下列範例顯示 [ **ServerName** ] 和 [ **IpAddress** ] 欄位的解壓縮結果。

    > [!NOTE]
    > 查詢 UI 和排程警示中的時間戳記將會忽略。

    :::image type="content" source="./media/watchlists/sentinel-watchlist-queries-fields.png" alt-text="具有關注清單欄位的查詢" lightbox="./media/watchlists/sentinel-watchlist-queries-fields.png":::
    
1. 您可以藉由將關注清單視為聯結和查閱的資料表，針對關注清單中的資料查詢任何資料表中的資料。

    ```kusto
    Heartbeat
    | lookup kind=leftouter _GetWatchlist('IPlist') 
     on $left.ComputerIP == $right.IPAddress
    ```
    :::image type="content" source="./media/watchlists/sentinel-watchlist-queries-join.png" alt-text="針對關注清單進行查詢作為查閱":::

## <a name="use-watchlists-in-analytics-rules"></a>在分析規則中使用 watchlists

若要在分析規則中使用 watchlists，請在 Azure 入口網站中，流覽至 **Azure Sentinel** 設定  >    >  **分析**，並 `_GetWatchlist('<watchlist>')` 在查詢中使用函數建立規則。

1. 在此範例中，請使用下列值來建立名為 "ipwatchlist" 的關注清單：

    :::image type="content" source="./media/watchlists/create-watchlist.png" alt-text="關注清單的四個專案清單":::

    :::image type="content" source="./media/watchlists/sentinel-watchlist-new-2.png" alt-text="建立具有四個專案的關注清單":::

1. 接下來，建立分析規則。  在此範例中，我們只會在關注清單中包含來自 IP 位址的事件：

    ```kusto
    //Watchlist as a variable
    let watchlist = (_GetWatchlist('ipwatchlist') | project IPAddress);
    Heartbeat
    | where ComputerIP in (watchlist)
    ```
    ```kusto
    //Watchlist inline with the query
    Heartbeat
    | where ComputerIP in ( 
        (_GetWatchlist('ipwatchlist')
        | project IPAddress)
    )
    ```

:::image type="content" source="./media/watchlists/sentinel-watchlist-analytics-rule-2.png" alt-text="在分析規則中使用 watchlists":::

## <a name="view-list-of-watchlists-aliases"></a>查看 watchlists 別名的清單

若要取得關注清單別名的清單，請從 Azure 入口網站中，流覽至 **Azure Sentinel**  >  **一般**  >  **記錄** 檔，然後執行下列查詢： `_GetWatchlistAlias` 。 您可以在 [ **結果** ] 索引標籤中看到別名清單。

> [!div class="mx-imgBorder"]
> ![列出 watchlists](./media/watchlists/sentinel-watchlist-alias.png)

## <a name="next-steps"></a>後續步驟
在本檔中，您已瞭解如何在 Azure Sentinel 中使用 watchlists 來擴充資料及改進調查。 若要深入了解 Azure Sentinel，請參閱下列文章：
- 深入了解如何[取得資料的可見度以及潛在威脅](quickstart-get-visibility.md)。
- 開始[使用 Azure Sentinel 偵測威脅](./tutorial-detect-threats-built-in.md)。
- [使用活頁簿](tutorial-monitor-your-data.md)監視資料。