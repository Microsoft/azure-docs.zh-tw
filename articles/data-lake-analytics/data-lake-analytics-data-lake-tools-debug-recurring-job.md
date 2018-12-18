---
title: 在 Azure Data Lake Analytics 中針對週期性作業進行偵錯
description: 了解如何使用 Azure Data Lake Tools for Visual Studio 針對異常的週期性作業進行偵錯。
services: data-lake-analytics
author: yanancai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.assetid: dc9b21d8-c5f4-4f77-bcbc-eff458f48de2
ms.service: data-lake-analytics
ms.topic: conceptual
ms.date: 05/20/2018
ms.openlocfilehash: 33c3b91e7bf9fa64e3ba3f98a9396045753d0c2a
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045689"
---
# <a name="troubleshoot-an-abnormal-recurring-job"></a>針對異常的週期性作業進行疑難排解

本文顯示如何使用 [Azure Data Lake Tools for Visual Studio](http://aka.ms/adltoolsvs)，針對週期性作業進行疑難排解。 從 [Azure Data Lake and Azure HDInsight 部落格](https://blogs.msdn.microsoft.com/azuredatalake/2017/09/19/managing-pipeline-recurring-jobs-in-azure-data-lake-analytics-made-easy/) 中，深入了解管線和週期性工作。

週期性作業通常會共用相同的查詢邏輯和類似的輸入資料。 例如，假設您在每個星期一早上 8 點有一個週期性作業， 計算過去一週的每週作用中使用者數目。 這些作業的指令碼會共用一個包含查詢邏輯的指令碼範本。 這些作業的輸入是過去一週的使用量資料。 共用相同的查詢邏輯和類似的輸入時，通常表示這些作業的效能類似且穩定。 如果其中一個週期性作業突然執行異常、失敗或變得十分緩慢，您可能想要：

- 查看週期性作業前幾次執行的統計資料報表，了解發生了什麼事。
- 比較異常作業與正常作業，找出變更之處。

Azure Data Lake Tools for Visual Studio 中的**相關作業檢視**可協助您加快兩種案例的疑難排解進度。

## <a name="step-1-find-recurring-jobs-and-open-related-job-view"></a>步驟 1：找出週期性作業，並開啟 [相關作業檢視]

若要使用 [相關作業檢視] 針對週期性作業問題進行疑難排解，您必須先在 Visual Studio 中找出週期性作業，然後開啟 [相關作業檢視]。

### <a name="case-1-you-have-the-url-for-the-recurring-job"></a>案例 1：您有週期性作業的 URL

透過 [工具] > [Data Lake] > [作業檢視]，您可以貼上作業 URL，以在 Visual Studio 中開啟 [作業檢視]。 選取 [檢視相關作業]，開啟 [相關作業檢視]。

![Data Lake Analytics 工具中的 [檢視相關作業] 連結](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/view-related-job.png)
 
### <a name="case-2-you-have-the-pipeline-for-the-recurring-job-but-not-the-url"></a>案例 2：您有週期性作業的管線，而不是 URL

在 Visual Studio 中，您可以透過 [伺服器總管] > 您的 Azure Data Lake Analytics 帳戶 > [管線] 來開啟管線瀏覽器  (如果您在伺服器總管中找不到此節點，請[下載最新外掛程式。](http://aka.ms/adltoolsvs))。 

![選取 [管線] 節點](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/pipeline-browser.png)

在管線瀏覽器中，Data Lake Analytics 帳戶的所有管線都會列在左側。 您可以展開管線來尋找所有週期性作業，然後選取發生問題的週期性作業。 即會在右側開啟 [相關作業檢視]。

![選取管線並開啟 [相關作業檢視]](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-view.png)

## <a name="step-2-analyze-a-statistics-report"></a>步驟 2：分析統計資料報表

摘要和統計資料報表會顯示在 [相關作業檢視] 頂端。 您可以在這裡找到問題的潛在根本原因。 

1.  在報表中，X 軸會顯示作業提交時間。 使用它，可以找出異常作業。
2.  使用下圖中的程序來檢查統計資料，並深入了解問題和可能的解決方案。

![檢查統計資料的程序流程圖](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-metrics-debugging-flow.png)

## <a name="step-3-compare-the-abnormal-job-to-a-normal-job"></a>步驟 3：比較異常作業與正常作業

您可以透過 [相關作業檢視] 底部的作業清單，找出所有已提交的週期性作業。 若要尋找更多深入資訊和可能的解決方案，請以滑鼠右鍵按一下異常作業。 使用 [作業差異] 檢視，以比較異常作業與先前的正常作業。

![比較作業的捷徑功能表](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/compare-job.png)

請注意這兩個作業的大型差異。 這些差異可能會造成效能問題。 若要進一步檢查，請使用下圖中的步驟：

![檢查作業之間差異的流程圖](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-diff-debugging-flow.png)

## <a name="next-steps"></a>後續步驟

* [解決資料扭曲問題](data-lake-analytics-data-lake-tools-data-skew-solutions.md)
* [對 U-SQL 失敗作業的使用者定義 C# 程式碼進行偵錯](data-lake-analytics-debug-u-sql-jobs.md)
