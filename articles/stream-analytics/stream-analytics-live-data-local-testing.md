---
title: 使用適用於 Visual Studio 的 Azure 串流分析工具 (預覽) 在本機測試即時資料
description: 了解如何使用即時串流資料，在本機測試您的 Azure 串流分析作業。
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: f0a8978a9c2e0538a2e7bc4eab202604913e700b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984155"
---
# <a name="test-live-data-locally-using-azure-stream-analytics-tools-for-visual-studio-preview"></a>使用適用於 Visual Studio 的 Azure 串流分析工具 (預覽) 在本機測試即時資料

適用於 Visual Studio 的 Azure 串流分析工具可讓您從 Azure 事件中樞、IoT 中樞和 Blob 儲存體中使用即時事件串流，在本機測試 IDE 的作業。 即時資料本機測試不能取代您可在雲端執行的[效能和延展性測試](stream-analytics-streaming-unit-consumption.md)，但您可以在功能測試期間因為不需每次想要測試串流分析作業時都要提交到雲端而節省時間。 此功能目前處於預覽狀態，不應用於生產環境工作負載。

## <a name="testing-options"></a>測試選項

支援下列本機測試選項：

|**輸入**  |**輸出**  |**作業類型**  |
|---------|---------|---------|
|本機靜態資料   |  本機靜態資料   |   雲端/Edge |
|即時輸入資料   |  本機靜態資料   |   雲端 |
|即時輸入資料   |  即時輸出資料   |   雲端 |

## <a name="local-testing-with-live-data"></a>使用即時資料進行本機測試

1. 當您建立 [Visual Studio 中的 Azure 串流分析雲端專案](stream-analytics-quick-create-vs.md)之後，請開啟 **script.asaql**。 本機測試預設會使用本機輸入和本機輸出。

   ![Azure 串流分析 Visual Studio 使用本機輸入和本機輸出進行本機測試](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-local-input-output.png)

2. 若要測試即時資料，從下拉式清單方塊中選擇 [使用雲端輸入]。

   ![Azure 串流分析 Visual Studio 使用即時雲端輸入進行本機測試](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-input.png)


3. 設定**開始時間**，以定義作業將開始處理輸入資料的時間。 作業可能需要提前讀取輸入資料，以確保會有精確的結果。 預設的時間會設定為目前時間的前 30 分鐘。

   ![Azure 串流分析 Visual Studio 使用即時資料進行本機測試的開始時間](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-input-start-time.png)

4. 按一下 [在本機執行]。 主控台視窗隨即出現，其中具有執行中的進度和作業計量。 如果您想要停止此程序，可以手動進行。 

   ![Azure 串流分析 Visual Studio 使用即時資料進行本機測試的處理視窗](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-input-process-window.png)

   在本機執行結果視窗中，每隔三秒就會使用前 500 列輸出資料列重新整理一次輸出結果，並將輸出檔案放置於您的專案路徑 **ASALocalRun** 資料夾中。 您也可以在本機執行結果視窗中按一下 [開啟結果資料夾] 按鈕，來開啟輸出檔案。

   ![Azure 串流分析 Visual Studio 使用即時資料進行本機測試的開啟結果資料夾](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-input-open-results-folder.png)

5. 如果您想要將結果輸出至雲端輸出來源，從第二個下拉式清單方塊中選擇 [輸出至雲端]。 Power BI 和 Azure Data Lake Storage 均不支援輸出來源。

   ![Azure 串流分析 Visual Studio 使用即時資料進行本機測試的輸出至雲端](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-output.png)
 
## <a name="limitations"></a>限制

* Power BI 和 Azure Data Lake Storage 由於驗證模型的限制而不支援輸出來源。

* 只有雲端輸入選項具備[時間原則](stream-analytics-out-of-order-and-late-events.md)支援，而本機輸入選項則沒有。

## <a name="next-steps"></a>後續步驟

* [使用適用於 Visual Studio 的 Azure 串流分析工具建立串流分析作業](stream-analytics-quick-create-vs.md)
* [安裝適用於 Visual Studio 的 Azure 串流分析工具](stream-analytics-tools-for-visual-studio-install.md)
* [使用 Visual Studio 在本機測試串流分析查詢](stream-analytics-vs-tools-local-run.md)
* [使用 Visual Studio 檢視 Azure 串流分析工作](stream-analytics-vs-tools.md)