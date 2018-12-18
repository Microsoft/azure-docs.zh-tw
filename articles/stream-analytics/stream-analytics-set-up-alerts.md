---
title: 設定 Azure 串流分析作業的監視警示
description: 本文說明如何使用 Azure 入口網站設定 Azure 串流分析作業的監視和警示。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/26/2017
ms.openlocfilehash: 4c676ab3039a02a4fda27ab00312133e5de8077a
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47090961"
---
# <a name="set-up-alerts-for-azure-stream-analytics-jobs"></a>設定 Azure 串流分析工作的警示
您可以設定會在計量達到您指定的條件時觸發的警示。 例如，您可能如下所示設定條件的警示：

`If there are zero input events in the last 5 minutes, send email notification to sa-admin@example.com`

您可以透過入口網站來針對計量設定規則，或是透過作業記錄檔資料以 [程式設計方式](https://code.msdn.microsoft.com/windowsazure/Receive-Email-Notifications-199e2c9a) 來設定規則。

## <a name="set-up-alerts-in-the-azure-portal"></a>在 Azure 入口網站中設定警示
1. 在 Azure 入口網站中，開啟您想要建立警示的串流分析作業。 

2. 在 [作業] 刀鋒視窗中，按一下 [監視] 區段。  

3. 在 [計量] 刀鋒視窗中，按一下 [新增警示] 命令。

      ![Azure 入口網站設定](./media/stream-analytics-set-up-alerts/06-stream-analytics-set-up-alerts.png)  

4. 輸入名稱和描述。

5. 使用選取器來定義據以傳送警示的條件。

6. 提供警示目的地的相關資訊。

      ![設定 Azure 串流分析作業的警示](./media/stream-analytics-set-up-alerts/stream-analytics-add-alert.png)  

如需有關在 Azure 入口網站中設定警示的更多詳細資料，請參閱[接收警示通知](../monitoring-and-diagnostics/insights-receive-alert-notifications.md)。  


## <a name="get-help"></a>取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

