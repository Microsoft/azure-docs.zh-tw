---
title: 透過 Azure 入口網站設定適用於 PostgreSQL 之 Azure 資料庫的伺服器參數
description: 本文說明如何使用 Azure 入口網站，在適用於 PostgreSQL 的 Azure 資料庫中設定伺服器參數。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 6d43cac79c19e117385549b1678a464dc5731bd7
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2018
ms.locfileid: "29687861"
---
# <a name="configure-server-parameters-in-azure-portal"></a>在 Azure 入口網站中設定伺服器參數
您可以透過 Azure 入口網站列出、顯示、更新適用於 PostgreSQL 的 Azure 資料庫伺服器的設定參數。

## <a name="prerequisites"></a>先決條件
為了逐步執行本作法指南，您需要︰
- [適用於 PostgreSQL 的 Azure 資料庫伺服器](quickstart-create-server-database-portal.md)

## <a name="viewing-and-editing-parameters"></a>檢視和編輯參數
1. 開啟 [Azure 入口網站](https://portal.azure.com)。

2. 選取適用於 PostgreSQL 的 Azure 資料庫伺服器。

3. 在 [設定] 區段下，選取 [伺服器參數]。 頁面會顯示一份參數、值和說明的清單。
![參數的概觀頁面](./media/howto-configure-server-parameters-in-portal/3-overview-of-parameters.png)

4. 選取**下拉**按鈕可查看列舉型別參數 (如 client_min_messages) 的可能值。
![列舉下拉按鈕](./media/howto-configure-server-parameters-in-portal/4-enum-drop-down.png)

5. 選取或將滑鼠停留在 [i] (資訊) 按鈕上可查看數值參數 (如 cpu_index_tuple_cost) 的可能值範圍。
![資訊按鈕](./media/howto-configure-server-parameters-in-portal/4-information-button.png)

6. 如有需要，使用 [搜尋方塊] 縮小至特定參數。 搜尋作業是依據參數的名稱和說明。
![搜尋結果](./media/howto-configure-server-parameters-in-portal/5-search.png)

7. 變更您想要調整的參數值。 您在工作階段中所做的所有變更都會以紫色顯示。 變更值之後，可以選取 [儲存]。 或者，也可以 [捨棄] 您的變更。
![儲存或捨棄變更](./media/howto-configure-server-parameters-in-portal/6-save-and-discard-buttons.png)

8. 儲存新的參數值後，隨時可以選取 [全部重設為預設值] 回復為所有參數的預設值。
![全部重設為預設值](./media/howto-configure-server-parameters-in-portal/7-reset-to-default-button.png)

## <a name="next-steps"></a>後續步驟
了解：
- [適用於 PostgreSQL 的 Azure 資料庫中的伺服器參數概觀](concepts-servers.md)
- [使用 Azure CLI 設定參數](howto-configure-server-parameters-using-cli.md)
