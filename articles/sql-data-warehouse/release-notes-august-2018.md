---
title: 2018 年 8 月 Azure SQL 資料倉儲版本資訊 | Microsoft Docs
description: Azure SQL 資料倉儲的版本資訊。
services: sql-data-warehouse
author: twounder
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 08/13/2018
ms.author: twounder
ms.reviewer: twounder
ms.openlocfilehash: c52fcd544e22c274d0e9053da93d2b9dee0df695
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286627"
---
# <a name="whats-new-in-azure-sql-data-warehouse-august-2018"></a>Azure SQL 資料倉儲有哪些最新功能？ 2018 年 8 月
Azure SQL 資料倉儲會持續改進。 本文說明 2018 年 8 月導入的新功能和變更。

## <a name="automatic-intelligent-insights"></a>Automatic Intelligent Insights
Microsoft 推出了 [Automatic Intelligent Insights](https://azure.microsoft.com/blog/automatic-intelligent-insights-to-optimize-performance-with-sql-data-warehouse/)，以履行資料倉儲的自動化雲端承諾。 您不必再監視資料倉儲中的資料扭曲和次佳資料表統計資料。 SQL 資料倉儲會呈現所有 Gen2 執行個體的智慧見解，不需額外收費。 與 [Azure Advisor](https://docs.microsoft.com/azure/advisor/advisor-performance-recommendations) 整合後，您即可自動接收最佳做法建議，以改善作用中工作負載的效能。 SQL 資料倉儲會分析工作負載，並根據您的使用量呈現建議。 這項分析會每日進行，以便您監視使用量報告和建議，進而改善您的工作負載。

您可以在 Azure Advisor 入口網站中檢視建議：![適用於 Azure SQL 資料倉儲的 Azure Advisor 入口網站建議](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/4e205b6d-df04-48db-8eec-d591f2592cf4.png)

您可以深入探詢每個類別，以查看特定警示的建議：![適用於 Azure SQL 資料倉儲的 Azure Advisor 入口網站建議詳細資料](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/3c42426e-6969-46e3-9025-c34c0755a302.png)


## <a name="bug-fixes"></a>錯誤修正

| 標題 | 說明 |
|:---|:---|
| **分割計數超過最大限制時的潛在查詢錯誤** |超過 1 百萬個檔案分割上限時，未處理的例外狀況會造成 SQL 引擎發生傾印，而且所有查詢都會失敗。 此修正程式會透過正確地處理例外狀況來解決問題，並在不造成查詢失敗的情況下傳回錯誤。 |
| **增加 ExternalMoveReadersPerNode 預設值來改善載入效能** |造成此問題的原因是 ExternalMoveReadersPerNode 屬性設定與 Service Fabric 設定不同步。 此迴歸造成 Gen2 負載效能降低。 修正程式會在經過最佳化設計的參數內恢復 Gen2 載入效能。|


## <a name="next-steps"></a>後續步驟
現在您已稍微了解 SQL 資料倉儲，請了解如何快速[建立 SQL 資料倉儲][create a SQL Data Warehouse]。 如果您不熟悉 Azure，您可能會發現 [Azure 詞彙][Azure glossary]在您遇到新術語時很有幫助。 或者，也可以看一下其中一些其他 SQL 資料倉儲資源。  

* [客戶成功案例]
* [部落格]
* [功能要求]
* [影片]
* [客戶諮詢小組部落格]
* [Stack Overflow 論壇]
* [Twitter]


[部落格]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[客戶諮詢小組部落格]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[客戶成功案例]: https://azure.microsoft.com/case-studies/?service=sql-data-warehouse
[功能要求]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[Stack Overflow 論壇]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[影片]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse
[create a SQL Data Warehouse]: ./create-data-warehouse-portal.md
[Azure glossary]: ../azure-glossary-cloud-terminology.md