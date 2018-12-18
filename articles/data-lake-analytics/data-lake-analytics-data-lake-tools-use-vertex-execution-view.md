---
title: 在適用於 Visual Studio 的 Data Lake 工具中使用 Vertex 執行檢視
description: 本文描述如何使用頂點執行檢視來測驗 Data Lake Analytics 作業。
services: data-lake-analytics
ms.service: data-lake-analytics
author: mumian
ms.author: jgao
ms.reviewer: jasonwhowell
ms.assetid: 5366d852-e7d6-44cf-a88c-e9f52f15f7df
ms.topic: conceptual
ms.date: 10/13/2016
ms.openlocfilehash: 9f834d697c0d3fe537bbdb190b0ba0d0f294ac87
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041008"
---
# <a name="use-the-vertex-execution-view-in-data-lake-tools-for-visual-studio"></a>在適用於 Visual Studio 的 Data Lake 工具中使用 Vertex 執行檢視
了解如何使用頂點執行檢視測驗 Data Lake Analytics 作業。


## <a name="open-the-vertex-execution-view"></a>開啟頂點執行檢視
在 Data Lake Tools for Visual Studio 中開啟 U-SQL 作業。 按一下左下角的 [頂點執行檢視]。 系統可能會提示您先載入設定檔，這可能需要一些時間，視您的網路連線而定。

![Data Lake Analytics 工具頂點執行檢視](./media/data-lake-analytics-data-lake-tools-use-vertex-execution-view/data-lake-tools-open-vertex-execution-view.png)

## <a name="understand-vertex-execution-view"></a>了解頂點執行檢視
[頂點執行檢視] 有三個部分：

![Data Lake Analytics 工具頂點執行檢視](./media/data-lake-analytics-data-lake-tools-use-vertex-execution-view/data-lake-tools-vertex-execution-view.png)

左邊的**頂點選取器**可讓您根據功能選取頂點 (例如前 10 個讀取的資料，或根據階段選擇)。 其中一個最常使用的篩選條件是查看**關鍵路徑的頂點**。 **關鍵路徑** 是 U-SQL 作業頂點的最長鏈結。 了解關鍵路徑有助於最佳化您的作業，方法是檢查哪一個頂點耗用最長時間。
  
![Data Lake Analytics 工具頂點執行檢視](./media/data-lake-analytics-data-lake-tools-use-vertex-execution-view/data-lake-tools-vertex-execution-view-pane2.png)

正上方的窗格顯示**所有頂點的執行狀態**。
  
![Data Lake Analytics 工具頂點執行檢視](./media/data-lake-analytics-data-lake-tools-use-vertex-execution-view/data-lake-tools-vertex-execution-view-pane3.png)

正下方的窗格顯示每個頂點的相關資訊：
* 處理序名稱︰頂點執行個體的名稱。 由 StageName|VertexName|VertexRunInstance 中的不同部分組成。 例如，SV7_Split[62].v1 頂點表示階段 SV7_Split 中第 62 號頂點的第二個執行中執行個體 (.v1，從 0 開始編制索引)。
* 總資料讀取/寫入︰此頂點讀取/寫入的資料。
* 狀態/結束狀態︰頂點結束時的最終狀態。
* 結束代碼/失敗類型︰頂點失敗時的錯誤。
* 建立原因︰為什麼建立頂點。
* 資源延遲/處理序延遲/PN 佇列延遲︰頂點等待資源、處理資料以及保持在佇列中所花費的時間。
* 處理序/建立者 GUID：目前執行的頂點或其建立者的 GUID。
* 版本︰執行中頂點的第 N 個執行個體 (系統可能會針對許多原因排程頂點的新執行個體，例如容錯移轉、計算備援等等。)
* 版本建立時間。
* 處理序建立開始時間/處理序佇列時間/處理序開始時間/處理序完成時間：頂點處理序開始建立時；頂點處理序開始佇列時；特定頂點處理序開始時；特定頂點完成時。

## <a name="next-steps"></a>後續步驟
* 若要記錄診斷資訊，請參閱 [為 Azure Data Lake Analytics 存取診斷記錄檔](data-lake-analytics-diagnostic-logs.md)
* 若要了解更複雜的查詢，請參閱 [使用 Azure 資料湖分析來分析網站記錄檔](data-lake-analytics-analyze-weblogs.md)。
* 若要檢視作業詳細資料，請參閱[針對 Azure Data Lake Analytics 作業使用作業瀏覽器和作業檢視](data-lake-analytics-data-lake-tools-view-jobs.md)
