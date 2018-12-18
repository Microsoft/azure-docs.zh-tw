---
title: 以視覺化方式監視 Azure 資料處理站 | Microsoft Docs
description: 了解如何以視覺化方式監視 Azure 資料處理站
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/12/2018
ms.author: shlo
ms.openlocfilehash: 4b3828e1857d17a128de346449d5cf2041709e50
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39041070"
---
# <a name="visually-monitor-azure-data-factories"></a>以視覺化方式監視 Azure 資料處理站
Azure Data Factory 是雲端式資料整合服務，可讓您在雲端建立資料驅動工作流程，以便協調及自動進行資料移動和資料轉換。 使用 Azure Data Factory，您可以建立和排程資料驅動工作流程 (稱為管線)，這類工作流程可以從不同資料存放區內嵌資料，使用計算服務 (例如 Azure HDInsight Hadoop、Spark、Azure Data Lake Analytics 和 Azure Machine Learning) 來處理/轉換資料，以及將輸出資料發佈至資料存放區 (例如 Azure SQL 資料倉儲)，以供商業智慧 (BI) 應用程式使用。
在此快速入門中，您將了解如何在不用撰寫任何程式碼的情況下，以視覺方式監視資料處理站 v2 管線。
如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="monitor-data-factory-v2-pipelines"></a>監視資料處理站 v2 管線

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
2. 登入 [Azure 入口網站](https://portal.azure.com/)。
3. 瀏覽至 Azure 入口網站中的 [建立資料處理站] 刀鋒視窗，然後按一下 [監視及管理] 磚。 這將會啟動 ADF v2 視覺監視體驗。

## <a name="list-view-monitoring"></a>清單檢視監視

以簡單的清單檢視介面監視管線和活動回合。 所有回合都會以本機瀏覽器的時區顯示。 您可以變更時區，這會將所有的日期時間欄位調整至選取的時區。  

#### <a name="monitoring-pipeline-runs"></a>監視管線回合
顯示資料處理站 v2 管線之每個管線回合的清單檢視。 包含下列資料行：

| **資料行名稱** | **說明** |
| --- | --- |
| 管線名稱 | 管線的名稱。 |
| 動作 | 可供檢視活動回合的單一動作。 |
| 回合開始 | 管線回合開始日期時間 (YYYY/MM/DD，上午/下午 HH:MM:SS) |
| Duration | 回合持續時間 (HH:MM:SS) |
| 觸發方式 | [手動觸發]、[排程觸發] |
| 狀態 | [失敗]、[成功]、[進行中] |
| 參數 | 管線回合參數 (名稱、值組) |
| Error | 管線回合錯誤 (若有) |
| 回合識別碼 | 管線回合的識別碼 |

![監視管線回合](media/monitor-visually/pipeline-runs.png)

#### <a name="monitoring-activity-runs"></a>監視活動回合
顯示對應至每個管線回合之活動回合的清單檢視。 按一下位於 [活動] 資料行底下的 [活動回合] 圖示，以檢視每個管線回合的活動回合。 包含下列資料行：

| **資料行名稱** | **說明** |
| --- | --- |
| 活動名稱 | 管線內的活動名稱。 |
| 活動類型 | 活動的類型 (也就是 [複製]、[HDInsightSpark]、[HDInsightHive] 等) |
| 回合開始 | 活動回合開始日期時間 (YYYY/MM/DD，上午/下午 HH:MM:SS) |
| Duration | 回合持續時間 (HH:MM:SS) |
| 狀態 | [失敗]、[成功]、[進行中] |
| 輸入 | 描述活動輸入的 JSON 陣列 |
| 輸出 | 描述活動輸出的 JSON 陣列 |
| Error | 活動回合錯誤 (若有) |

![監視活動回合](media/monitor-visually/activity-runs.png)

> [!IMPORTANT]
> 您必須按一下位於上方的 [重新整理] 圖示，以重新整理管線和活動回合的清單。 目前不支援自動重新整理。
>

![重新整理](media/monitor-visually/refresh.png)

## <a name="features"></a>特性

#### <a name="select-a-data-factory-to-monitor"></a>選取要監視的資料處理站
將滑鼠游標暫留於左上方的 **Data Factory** 圖示上。 按一下「箭號」圖示以查看可監視的 Azure 訂用帳戶及資料處理站清單。

![選取 Data Factory](media/monitor-visually/select-datafactory.png)

#### <a name="rich-ordering-and-filtering"></a>豐富的排序和篩選功能

依 [回合開始] 以遞減/遞增的順序對管線回合進行排序，然後依下列資料行對管線回合進行篩選：

| **資料行名稱** | **說明** |
| --- | --- |
| 管線名稱 | 管線的名稱。 選項包括針對 [過去 24 小時]、[上週]、[過去 30 天] 的快速篩選，或是選取自訂日期時間。 |
| 回合開始 | 管線回合開始日期時間 |
| 回合狀態 | 依狀態 (也就是 [成功]、[失敗]、[進行中]) 篩選回合 |

![Filter](media/monitor-visually/filter.png)

#### <a name="addremove-columns-in-list-view"></a>在清單檢視中新增/移除資料行
以滑鼠右鍵按一下清單檢視標頭，然後選擇要顯示於清單檢視中的資料行

![資料行](media/monitor-visually/columns.png)

#### <a name="reorder-column-widths-in-list-view"></a>重新排列清單檢視中的資料行寬度
若要增加或減少清單檢視中的資料行寬度，請直接將滑鼠游標暫留於資料行標頭上方

#### <a name="user-properties"></a>使用者屬性

您可以將任何管線活動屬性升階為使用者屬性，讓它變成您可以監視的實體。 例如，您可以將管線中「複製」活動的 **Source** 和 **Destination** 屬性升階為使用者屬性。 您也可以選取 [自動產生]，為「複製」活動產生 **Source** 和 **Destination** 使用者屬性。

![建立使用者屬性](media/monitor-visually/monitor-user-properties-image1.png)

> [!NOTE]
> 您最多只能將 5 個管線活動屬性升階為使用者屬性。

建立使用者屬性之後，您就可以在監視清單檢視中監視它們。 如果「複製」活動的來源是資料表名稱，您可以活動執行清單檢視中資料行的形式，監視來源資料表名稱。

![活動執行清單沒有使用者屬性](media/monitor-visually/monitor-user-properties-image2.png)

![將使用者屬性的資料行新增到活動執行清單](media/monitor-visually/monitor-user-properties-image3.png)

![活動執行清單具有使用者屬性的資料行](media/monitor-visually/monitor-user-properties-image4.png)

#### <a name="guided-tours"></a>導覽
按一下左下方的 [資訊] 圖示，然後按一下 [導覽] 以取得監視管線及活動回合的逐步指示。

![導覽](media/monitor-visually/guided-tours.png)

#### <a name="feedback"></a>意見反應
按一下 [意見反應] 圖示以針對各種功能或您遇到的任何問題向我們提供意見反應。

![意見反應](media/monitor-visually/feedback.png)

## <a name="next-steps"></a>後續步驟

請參閱[以程式設計方式監視和管理管線](https://docs.microsoft.com/azure/data-factory/monitor-programmatically)文章，以了解如何監視和管理管線
