---
title: 適用於 MariaDB 的 Azure 資料庫中的伺服器記錄
description: 描述適用於 MariaDB 的 Azure 資料庫中的可用記錄，以及啟用不同記錄層級的可用參數。
author: rachel-msft
ms.author: raagyema
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: d982af64517ed452d907f62a39e975d472951392
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992978"
---
# <a name="server-logs-in-azure-database-for-mariadb"></a>適用於 MariaDB 的 Azure 資料庫中的伺服器記錄
在適用於 MariaDB 的 Azure 資料庫中，使用者可以使用慢速查詢記錄檔。 不支援存取交易記錄。 慢速查詢記錄檔可以用來找出效能瓶頸，以進行疑難排解。

如需慢速查詢記錄檔的詳細資訊，請參閱 MariaDB 文件的[慢速查詢記錄檔](https://mariadb.com/kb/en/library/slow-query-log-overview/) (英文)。

## <a name="access-server-logs"></a>存取伺服器記錄
您可以使用 Azure 入口網站和 Azure CLI，來列出和下載適用於 MariaDB 的 Azure 資料庫伺服器記錄。

在 Azure 入口網站中，選取適用於 MariaDB 的 Azure 資料庫伺服器。 在 [監視] 標題下方，選取 [伺服器記錄檔] 頁面。

<!-- For more information on Azure CLI, see [Configure and access server logs using Azure CLI](howto-configure-server-logs-in-cli.md).-->

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

如需慢速查詢記錄檔參數的完整描述，請參閱 MariaDB [慢速查詢記錄檔文件](https://mariadb.com/kb/en/library/slow-query-log-overview/)。

## <a name="next-steps"></a>後續步驟
- [如何從 Azure 入口網站設定和存取伺服器記錄](howto-configure-server-logs-portal.md)。
