---
title: 適用於 PostgreSQL 的 Azure 資料庫中的伺服器記錄
description: 本文說明適用於 PostgreSQL 的 Azure 資料庫產生查詢和錯誤記錄的方式，以及設定記錄保留的方式。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: bcca8ce8d11482dd8517992297b7e8a5b94ac8b1
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37435485"
---
# <a name="server-logs-in-azure-database-for-postgresql"></a>適用於 PostgreSQL 的 Azure 資料庫中的伺服器記錄 
適用於 PostgreSQL 的 Azure 資料庫會產生查詢和錯誤記錄。 不過，不支援存取交易記錄。 查詢和錯誤記錄可用來針對組態錯誤及未達最佳效能的情況，進行識別、疑難排解及修復。 如需詳細資訊，請參閱[錯誤報告和記錄 (英文)](https://www.postgresql.org/docs/9.6/static/runtime-config-logging.html)。

## <a name="access-server-logs"></a>存取伺服器記錄
您可以使用 Azure 入口網站、[Azure CLI](howto-configure-server-logs-using-cli.md) 和 Azure REST API，來列出和下載 Azure PostgreSQL 伺服器錯誤記錄。

## <a name="log-retention"></a>記錄保留
您可以使用與您伺服器相關聯的 **log\_retention\_period** 參數，來設定系統記錄的保留期。 此參數的單位為天。 預設值為 3 天。 最大值為 7 天。 您的伺服器必須配置足夠的儲存體，才能包含保留的記錄檔。
記錄檔每一小時或每 100 MB 的大小就會輪換 (端視何者先達到)。

## <a name="configure-logging-for-azure-postgresql-server"></a>設定 Azure PostgreSQL 伺服器的記錄
您可以針對伺服器啟用查詢記錄和錯誤記錄。 錯誤記錄可包含自動清空、連線和檢查點資訊。

您可以藉由設定下列兩個伺服器參數，為 PostgreSQL DB 執行個體啟用查詢記錄功能︰`log_statement` 和 `log_min_duration_statement`。

**log\_statement** 參數控制要記錄哪些 SQL 陳述式。 我們建議將此參數設為 ***all*** 以記錄所有陳述式；預設值為 none。

**log\_min\_duration\_statement** 參數會以毫秒為單位，設定要記錄之陳述式的限制。 系統會記錄所有執行時間比此參數設定還長的 SQL 陳述式。 預設會停用此參數並設為負 1 (-1)。 啟用此參數，有助於追蹤應用程式中未最佳化的查詢。

**log\_min\_messages** 讓您能夠控制要將哪些訊息層級寫入伺服器記錄。 預設值為 WARNING。 

如需有關這些設定的詳細資訊，請參閱[錯誤報告和記錄 (英文)](https://www.postgresql.org/docs/9.6/static/runtime-config-logging.html) 文件。 若要特別設定「適用於 PostgreSQL 的 Azure 資料庫」伺服器參數，請參閱[使用 Azure CLI 自訂伺服器設定參數](howto-configure-server-parameters-using-cli.md)。

## <a name="next-steps"></a>後續步驟
- 若要使用 Azure CLI 命令列介面來存取記錄，請參閱[使用 Azure CLI 設定和存取伺服器記錄](howto-configure-server-logs-using-cli.md)。
- 如需伺服器參數的詳細資訊，請參閱[使用 Azure CLI 自訂伺服器設定參數](howto-configure-server-parameters-using-cli.md)。
