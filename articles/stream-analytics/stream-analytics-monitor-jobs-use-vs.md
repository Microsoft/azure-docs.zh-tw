---
title: 使用 Visual Studio 監視和管理 Azure 串流分析工作
description: 本文說明如何使用 Visual Studio 監視和管理 Azure 串流分析工作。
services: stream-analytics
author: su-jie
ms.author: sujie
manager: kfile
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 07/10/2018
ms.openlocfilehash: 9cc65fc5119bedc816f825b7aae6120f4bca3b12
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "40246586"
---
# <a name="monitor-and-manage-stream-analytics-jobs-with-visual-studio"></a>使用 Visual Studio 監視和管理串流分析工作

本文示範如何在 Visual Studio 中監視您的串流分析工作。 適用於 Visual Studio 的 Azure 串流分析工具提供類似 Azure 入口網站的監視體驗，而不需要離開 IDE。 當您從您的 **Script.asaql** **提交至 Azure** 時，即可開始監視工作，或者您可以監視現有的工作，無論它們的建立方式為何。 

## <a name="job-summary"></a>工作摘要

[工作摘要] 和 [工作計量] 提供工作的概略資訊。 您可以輕鬆判斷工作的狀態和事件資訊。

<img src="./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-job-summary-metrics.png" alt="Stream Analytics job summary and job metrics" width="300px"/> 


## <a name="job-metrics"></a>作業計量

您可以將 [工作摘要] 摺疊，然後按一下 [工作計量] 索引標籤來檢視包含重要計量的圖表。 選取和取消選取計量類型以將它們新增到圖表或從圖表移除。

![串流分析計量](./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-vs-metrics.png)


## <a name="error-monitoring"></a>錯誤監視

您也可以在 [錯誤] 索引標籤上按一下來監視錯誤。

![串流分析錯誤](./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-vs-errors.png)


## <a name="get-support"></a>取得支援
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。 

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [使用 Visual Studio 建立 Azure 串流分析工作](stream-analytics-quick-create-vs.md)
* [安裝適用於 Visual Studio 的 Azure 串流分析工具](stream-analytics-tools-for-visual-studio-install.md) 


