---
title: 在 Visual Studio 中檢視 Azure 串流分析工作
description: 本文說明如何在 Visual Studio 中檢視串流分析工作。
services: stream-analytics
author: su-jie
ms.author: sujie
manager: kfile
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 07/10/2018
ms.openlocfilehash: f9bc9bf540f7f44e07bc4bb0ae8ef6910ae78476
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222381"
---
# <a name="use-visual-studio-to-view-azure-stream-analytics-jobs"></a>使用 Visual Studio 檢視 Azure 串流分析工作

適用於 Visual Studio 的 Azure 串流分析工具讓開發人員能直接從 IDE 輕鬆地管理其串流分析工作。 使用 Azure 串流分析工具，您可以：
- [建立新的工作](stream-analytics-quick-create-vs.md)
- 啟動、停止和[監視工作](stream-analytics-monitor-jobs-use-vs.md)
- 檢查工作結果
- 將現有工作匯出到專案
- 測試輸入和輸出連線
- [在本機執行查詢](stream-analytics-vs-tools-local-run.md)

了解如何[安裝適用於 Visual Studio 的 Azure 串流分析工具](stream-analytics-tools-for-visual-studio-install.md)。

## <a name="explore-the-job-view"></a>瀏覽作業檢視

您可以使用作業檢視，從 Visual Studio 與 Azure 串流分析工作互動。

### <a name="open-the-job-view"></a>開啟作業檢視

1. 在 [伺服器總管] 中，選取 [串流分析工作]，然後選取 [重新整理]。 您的工作應該會顯示在 [串流分析工作] 下。

    ![串流分析伺服器總管清單](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-list-jobs-01.png)



2. 展開作業節點並按兩下 [作業檢視] 節點來開啟作業檢視。
    
   ![展開的作業節點](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-job-view-01.png)

### <a name="start-and-stop-jobs"></a>開始和停止工作

從 Visual Studio 的作業檢視中可完全管理 Azure 串流分析工作。 使用控制項來啟動、停止或刪除工作。
    
   ![串流分析工作控制項](./media/stream-analytics-vs-tools/azure-stream-analytics-job-view-controls.png)


## <a name="check-job-results"></a>檢查工作結果

適用於 Visual Studio 的串流分析工具目前支援 Azure Data Lake Storage 和 Blob 儲存體的輸出預覽。 若要檢視結果，只要按兩下 [作業檢視] 中工作圖表的輸出節點，然後輸入適當的認證。

   ![串流分析工作輸出](./media/stream-analytics-vs-tools/stream-analytics-blob-preview.png)


## <a name="export-jobs-to-a-project"></a>將工作匯出到專案

有兩種方式可以將現有的作業匯出到專案。

1. 在 [伺服器總管] 的 [資料流分析工作] 節點下，以滑鼠右鍵按一下作業節點。 選取 [匯出至新的串流分析專案]。
    
   ![將工作匯出到專案](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-export-job-01.png)
    
    產生的專案會在 [方案總管] 中顯示。
    
   ![方案總管](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-export-job-02.png)

2. 在作業檢視中，選取 [產生專案]。
    
   ![從作業檢視產生專案](./media/stream-analytics-vs-tools/stream-analytics-tools-for-vs-export-job-03.png)

## <a name="test-connections"></a>測試連接

在 [作業檢視] 中，從 [測試連接] 下拉式清單選取選項即可測試輸入和輸出連線。

   ![[測試連接] 下拉式清單](./media/stream-analytics-vs-tools/stream-analytics-test-connection-dropdown.png)

[測試連接] 結果會顯示在 [輸出] 視窗。

   ![測試連接結果](./media/stream-analytics-vs-tools/stream-analytics-test-connection-results.png)

## <a name="next-steps"></a>後續步驟

* [使用 Visual Studio 監視和管理 Azure 串流分析工作](stream-analytics-monitor-jobs-use-vs.md)
* [快速入門：使用 Visual Studio 建立串流分析工作](stream-analytics-quick-create-vs.md)
* [教學課程：使用 Azure Pipelines 部署具有 CI/CD 的 Azure 串流分析作業](stream-analytics-tools-visual-studio-cicd-vsts.md)
* [使用串流分析工具持續進行整合及開發](stream-analytics-tools-for-visual-studio-cicd.md)
