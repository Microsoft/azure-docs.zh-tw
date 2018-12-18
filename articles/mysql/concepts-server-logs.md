---
title: 適用於 MySQL 的 Azure 資料庫中的伺服器記錄
description: 描述適用於 MySQL 的 Azure 資料庫中的可用記錄，以及啟用不同記錄層級的可用參數。
services: mysql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 09/17/2018
ms.openlocfilehash: ac5be20815b552c08e5cd1054bf24d7a10b56498
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46124264"
---
# <a name="server-logs-in-azure-database-for-mysql"></a>適用於 MySQL 的 Azure 資料庫中的伺服器記錄
在適用於 MySQL 的 Azure 資料庫中，使用者可以使用慢速查詢記錄。 不支援存取交易記錄。 慢速查詢記錄檔可以用來找出效能瓶頸，以進行疑難排解。 

如需 MySQL 慢速查詢記錄的詳細資訊，請參閱 MySQL 參考手冊的[慢速查詢記錄章節](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)。

## <a name="access-server-logs"></a>存取伺服器記錄
您可以使用 Azure 入口網站和 Azure CLI，來列出和下載適用於 MySQL 的 Azure 資料庫伺服器記錄。

在 Azure 入口網站中，選取適用於 MySQL 的 Azure 資料庫伺服器。 在 [監視] 標題下方，選取 [伺服器記錄檔] 頁面。

如需 Azure CLI 的詳細資訊，請參閱[使用 Azure CLI 設定和存取伺服器記錄](howto-configure-server-logs-in-cli.md)。

## <a name="log-retention"></a>記錄保留
記錄最多可以從建立開始算起保留七天。 如果可用記錄的大小總計超過 7 GB，則除非有空間可用，否則會刪除最舊檔案。 

記錄會每隔 24 小時或 7 GB 旋轉一次，先到者先用。


## <a name="configure-logging"></a>設定記錄 
預設會停用慢速查詢記錄。 若要啟用它，請將 slow_query_log 設為 ON。

您可以調整的其他參數包含：

- **long_query_time**：如果查詢所需的時間比記錄該查詢的 long_query_time (秒) 還要久。 預設值為 10 秒。
- **log_slow_admin_statements**：如果 ON 在寫入至 slow_query_log 的陳述式中包含 ALTER_TABLE 和 ANALYZE_TABLE 這類管理陳述式。
- **log_queries_not_using_indexes**：決定是否將未使用索引的查詢記錄至 slow_query_log
- **log_throttle_queries_not_using_indexes**：這個參數會限制可寫入至慢速查詢記錄的非索引查詢次數。 log_queries_not_using_indexes 設為 ON 時，這個參數會生效。

如需慢速查詢記錄參數的完整描述，請參閱 MySQL [慢速查詢記錄文件](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)。

## <a name="next-steps"></a>後續步驟
- [如何從 Azure CLI 設定和存取伺服器記錄](howto-configure-server-logs-in-cli.md)。
