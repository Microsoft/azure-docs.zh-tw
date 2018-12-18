---
title: 教學課程：在 Power BI 中從 Azure 資料總管將資料視覺化
description: 在本教學課程中，您將了解如何使用 Power BI 來連線至「Azure 資料總管」，然後將資料視覺化。
services: data-explorer
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: tutorial
ms.date: 09/24/2018
ms.openlocfilehash: fc2d96c4dc8184ba26001fd77732ce7c45253d5a
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393707"
---
# <a name="tutorial-visualize-data-from-azure-data-explorer-in-power-bi"></a>教學課程：在 Power BI 中從 Azure 資料總管將資料視覺化

「Azure 資料總管」是一項快速又彈性極佳的資料探索服務，可用於處理記錄和遙測資料。 Power BI 是一個商務分析解決方案，可讓您將資料視覺化並在整個組織共用結果。 在本教學課程中，您會先了解如何在「Azure 資料總管」中轉譯視覺效果。 接著，您會使用 Power BI 來連線至「Azure 資料總管」、根據範例資料來建置報表，然後將報表發佈至 Power BI 服務。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費 Azure 帳戶](https://azure.microsoft.com/free/)。 如果您沒有註冊 Power BI Pro，請先[註冊免費試用](https://app.powerbi.com/signupredirect?pbi_source=web)再開始進行操作。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在 Azure 資料總管中轉譯視覺效果
> * 在 Power BI Desktop 中連線至 Azure 資料總管
> * 在 Power BI Desktop 中處理資料
> * 使用視覺效果來建立報表
> * 發佈和共用報表

## <a name="prerequisites"></a>必要條件

除了 Azure 和 Power BI 訂用帳戶之外，您還需要下列項目，才能完成本教學課程：

* [測試叢集和資料庫](create-cluster-database-portal.md)

* [StormEvents 範例資料](ingest-sample-data.md)

* [Power BI Desktop](https://powerbi.microsoft.com/get-started/) (按一下 [免費下載])

## <a name="render-visuals-in-azure-data-explorer"></a>在 Azure 資料總管中轉譯視覺效果

在進入 Power BI 之前，讓我們先看看如何在「Azure 資料總管」中轉譯視覺效果。 這非常適用於一些快速分析。

1. 登入 [https://dataexplorer.azure.com](https://dataexplorer.azure.com)。

1. 在左窗格中，選取包含 StormEvents 範例資料的測試資料庫。

1. 將下列查詢貼到右側視窗中，然後選取 [執行]。

    ```Kusto
    StormEvents
    | summarize event_count=count() by State
    | where event_count > 1800
    | project State, event_count
    | sort by event_count
    | render columnchart
    ```

    此查詢會計算各州的天氣事件。 接著，它會針對有超過 1800 個天氣事件的所有州別轉譯出一個直條圖。

    ![事件直條圖](media/visualize-power-bi/events-column-chart.png)

1. 將下列查詢貼到右側視窗中，然後選取 [執行]。

    ```Kusto
    StormEvents
    | where State == "WASHINGTON" and StartTime >= datetime(2007-07-01) and StartTime <= datetime(2007-07-31)
    | summarize StormCount = count() by EventType
    | render piechart
    ```

    此查詢會計算華盛頓州 7 月各類型的天氣事件。 接著，它會轉譯出一個顯示每個事件類型之百分比的圓形圖。

    ![事件圓形圖](media/visualize-power-bi/events-pie-chart.png)

現在可以來看看 Power BI，但您還可以在「Azure 資料總管」中對視覺效果進行更多操作。

## <a name="connect-to-azure-data-explorer"></a>連線至 Azure 資料總管

現在您需在 Power BI Desktop 中連線至 Azure 資料總管。

1. 在 Power BI Desktop 中，選取 [取得資料]，然後選取 [更多]。

    ![取得資料](media/visualize-power-bi/get-data-more.png)

1. 搜尋 *Kusto*、選取 [Azure Kusto 搶鮮版 (Beta)]，然後選取 [連接]。

    ![搜尋並取得資料](media/visualize-power-bi/search-get-data.png)

1. 在 [預覽版連接器] 畫面上，選取 [繼續]。

1. 在 [Kusto] 畫面上，輸入測試叢集和資料庫的名稱。 叢集的格式應該是 `https://<ClusterName>.<Region>.kusto.windows.net`。 輸入 *StormEvents* 作為資料表的名稱。 保留所有其他選項的預設值，然後選取 [確定]。

    ![叢集、資料庫、資料表選項](media/visualize-power-bi/cluster-database-table.png)

1. 在資料預覽畫面上，選取 [編輯]。

    資料表會在「Power Query 編輯器」中開啟，您可以先在此編輯器中編輯資料列和資料行，然後再匯入資料。

## <a name="work-with-data-in-power-bi-desktop"></a>在 Power BI Desktop 中處理資料

既然您已連線至「Azure 資料總管」，現在即可在「Power Query 編輯器」中編輯資料。 您需將 [BeginLat] 資料行中具有 Null 值的資料列卸除，並將 [StormSummary] JSON 資料行整個卸除。 這些作業相當簡單，但您也可以在匯入資料時，執行複雜的轉換。

1. 選取 [BeginLat] 資料行的箭號、取消選取 [null] 核取方塊，然後選取 [確定]。

    ![篩選資料行](media/visualize-power-bi/filter-column.png)

1. 以滑鼠右鍵按一下 **StormSummary** 資料行標頭，然後選取 [移除]。

    ![移除資料行](media/visualize-power-bi/remove-column.png)

1. 在 [查詢設定] 窗格中，將名稱從 *Query1* 變更為 *StormEvents*。

    ![變更查詢名稱](media/visualize-power-bi/query-name.png)

1. 在功能區的 [常用] 索引標籤上，選取 [關閉並套用]。

    ![關閉並套用](media/visualize-power-bi/close-apply.png)

    Power Query 會套用您的變更，然後將範例資料匯入到「資料模型」中。 接下來幾個步驟將說明如何充實該資料模型。 同樣地，這只是一個讓您了解可能進行之操作的簡單範例。

1. 在主視窗的左側，選取資料檢視。

    ![資料檢視](media/visualize-power-bi/data-view.png)

1. 在功能區的 [模型] 索引標籤上，選取 [新增資料行]。

    ![新增資料行](media/visualize-power-bi/new-column.png)

1. 在公式列中輸入下列「資料分析運算式」(DAX) 公式，然後按 Enter 鍵。

    ```DAX
    DurationHours = DATEDIFF(StormEvents[StartTime], StormEvents[EndTime], hour)
    ```

    ![公式列](media/visualize-power-bi/formula-bar.png)

    此公式會建立 *DurationHours* 資料行，以計算每個天氣事件會持續多久。 您將在下一節的視覺效果中使用此資料行。

1. 捲動至資料表的右側以查看資料行。

## <a name="create-a-report-with-visuals"></a>使用視覺效果來建立報表

既然您已匯入資料並改善資料模型，現在即可建置含有視覺效果的報表。 您需根據事件持續時間新增一個直條圖，以及一個顯示農損情況的地圖。

1. 在視窗的左側，選取報表檢視。

    ![報表檢視](media/visualize-power-bi/report-view.png)

1. 在 [視覺效果] 窗格中，選取群組直條圖。

    ![新增直條圖](media/visualize-power-bi/add-column-chart.png)

    畫布中會新增一個空白圖表。

    ![空白圖表](media/visualize-power-bi/blank-chart.png)

1. 在 [欄位] 清單中，選取 [DurationHours] 和 [State]。

    ![選取欄位](media/visualize-power-bi/select-fields.png)

    您現在會有一個顯示各州一整年間天氣事件總時數的圖表。

    ![持續時間直條圖](media/visualize-power-bi/duration-column-chart.png)

1. 按一下畫布上直條圖外的任何位置。

1. 在 [視覺效果] 窗格中，選取地圖。

    ![新增地圖](media/visualize-power-bi/add-map.png)

1. 在 [欄位] 清單中，選取 [CropDamage] 和 [State]。 調整地圖大小以便清楚查看美國各州。

    ![農損地圖](media/visualize-power-bi/crop-damage-map.png)

    泡泡的大小代表農損金額值。 將滑鼠游標移至泡泡上方，即可查看詳細資料。

1. 移動及調整視覺效果大小，以便讓報表看起來如下圖。

    ![已完成的報表](media/visualize-power-bi/finished-report.png)

1. 以 *storm-events.pbix* 名稱儲存報表。

## <a name="publish-and-share-the-report"></a>發佈和共用報表

到目前為止，您在 Power BI 中進行的工作都是使用 Power BI Desktop 在本機完成的。 現在您需將報表發佈至 Power BI 服務以便與他人共用。

1. 在 Power BI Desktop 之功能區的 [常用] 索引標籤上，選取 [發佈]。

    ![發佈按鈕](media/visualize-power-bi/publish-button.png)

1. 如果您尚未登入 Power BI，請進行登入程序。

1. 選取 [我的工作區]，然後選取 [選取]。

    ![選取工作區](media/visualize-power-bi/select-workspace.png)

1. 完成發佈時，選取 [在 Power BI 中開啟 storm-events.pbix]。

    ![發佈成功](media/visualize-power-bi/publishing-succeeded.png)

    報表會在服務中開啟，所含的視覺效果和版面配置會與您在 Power BI Desktop 中定義的相同。

1. 在報表的右上角中，選取 [共用]。

    ![[共用] 按鈕](media/visualize-power-bi/share-button.png)

1. 在 [共用報表] 畫面中，新增您組織中的同事、新增附註，然後選取 [共用]。

    ![共用報表](media/visualize-power-bi/share-report.png)

    如果您的同事具備適當的權限，他們便能夠存取您共用的報表。

## <a name="clean-up-resources"></a>清除資源

如果您不想要保留所建立的報表，只要刪除 *storm-events.pbix* 檔案即可。 如果您想要移除所發佈的報表，請依照下列步驟進行操作。

1. 在 [我的工作區] 底下，向下捲動至 [報表] 並找出 [storm-events]。

1. 選取 [storm-events] 旁邊的省略符號 (**. . .**)，然後選取 [移除]。

    ![移除報表](media/visualize-power-bi/remove-report.png)

1. 確認移除。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [撰寫查詢](write-queries.md)
