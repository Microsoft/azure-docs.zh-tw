---
title: 在 Azure 入口網站中設定和存取 PostgreSQL 的伺服器記錄
description: 本文描述如何從 Azure 入口網站，在適用於 PostgreSQL 的 Azure 資料庫中設定和存取伺服器記錄。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: aa9823c65b342f922ca78a51ecd3055dfac62869
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2018
ms.locfileid: "29692159"
---
# <a name="configure-and-access-server-logs-in-the-azure-portal"></a>在 Azure 入口網站中設定和存取伺服器記錄

您可以從 Azure 入口網站設定、列出及下載[適用於 PostgreSQL 的 Azure 資料庫伺服器記錄](concepts-server-logs.md)。

## <a name="prerequisites"></a>先決條件
若要逐步執行本作法指南，您需要︰
- [適用於 PostgreSQL 的 Azure 資料庫伺服器](quickstart-create-server-database-portal.md)

## <a name="configure-logging"></a>設定記錄
設定查詢記錄和錯誤記錄的存取權。 

1. 登入 [Azure 入口網站](http://portal.azure.com/)。

2. 選取適用於 PostgreSQL 的 Azure 資料庫伺服器。

3. 在提要欄位的 [監視] 區段中，選取 [伺服器記錄]。 

   ![選取 [伺服器記錄]，然後選取 [按一下這裡啟用...]](./media/howto-configure-server-logs-in-portal/1-select-server-logs-configure.png)

4. 選取 [按一下這裡可啟用記錄，並設定記錄參數] 標題，以查看伺服器參數。

5. 變更您需要調整的參數。 您在此工作階段中所做的所有變更都會以紫色顯示。

   變更參數之後，您可以按一下 [儲存]。 或者，也可以 [捨棄] 您的變更。 

   ![較長的含儲存或捨棄變更的參數清單](./media/howto-configure-server-logs-in-portal/3-save-discard.png)

6. 按一下 [伺服器參數] 頁面上的**關閉按鈕** (X 圖示)。

## <a name="view-list-and-download-logs"></a>檢視清單並下載記錄
開始記錄之後，您可以檢視可用記錄的清單，並在 [伺服器記錄] 窗格上下載個別記錄檔。 

1. 開啟 Azure 入口網站。

2. 選取適用於 PostgreSQL 的 Azure 資料庫伺服器。

3. 在提要欄位的 [監視] 區段中，選取 [伺服器記錄]。 這個頁面會顯示記錄檔的清單，如下所示：

   ![伺服器記錄清單](./media/howto-configure-server-logs-in-portal/4-server-logs-list.png)

   > [!TIP]
   > 記錄的命名慣例是 **postgresql-yyyy-mm-dd_hh0000.log**。 檔案名稱中使用的日期和時間是發出記錄的時間。 記錄檔每一小時或每 100 MB 的大小就會輪換 (端視何者先達到)。

4. 如有需要，請使用**搜尋方塊**，根據日期/時間快速縮小至特定記錄。 搜尋是根據記錄的名稱進行。

   ![記錄名稱的範例搜尋](./media/howto-configure-server-logs-in-portal/5-search.png)

5. 使用資料表資料列中每個記錄檔旁邊的**下載**按鈕 (向下箭號圖示) 下載個別記錄檔，如下所示：

   ![按一下下載圖示](./media/howto-configure-server-logs-in-portal/6-download.png)

## <a name="next-steps"></a>後續步驟
- 請參閱[存取 CLI 中的伺服器記錄](howto-configure-server-logs-using-cli.md)，以了解如何透過程式下載記錄。
- 深入了解適用於 PostgreSQL 的 Azure DB 中的[伺服器記錄](concepts-server-logs.md)。 
- 如需參數定義和 PostgreSQL 記錄的相關詳細資訊，請參閱[錯誤報告和記錄](https://www.postgresql.org/docs/current/static/runtime-config-logging.html) \(英文\) 中的 PostgreSQL 文件。

