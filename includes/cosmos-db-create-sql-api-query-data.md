---
title: 包含檔案
description: 包含檔案
services: cosmos-db
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 04/13/2018
ms.author: sngun
ms.custom: include file
ms.openlocfilehash: e6e70da1f939547cdb589ae7bcb6ed1f6148f22e
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38733758"
---
您現可在 [資料總管] 中，使用查詢來擷取和篩選您的資料。

1. 依預設，此查詢會設為 `SELECT * FROM c`。 此預設查詢會擷取並顯示集合中的所有文件。 

    ![[資料總管] 中的預設查詢為 `SELECT * FROM c`](./media/cosmos-db-create-sql-api-query-data/azure-cosmosdb-data-explorer-query.png)

2. 停留在 [文件] 索引標籤，藉由按一下 [編輯篩選條件] 按鈕、將 `ORDER BY c._ts DESC` 新增至查詢述詞方塊，然後再按一下 [套用篩選條件] 即可變更查詢。

    ![新增 ORDER BY c._ts DESC 並按一下 [套用篩選]，以變更預設查詢](./media/cosmos-db-create-sql-api-query-data/azure-cosmosdb-data-explorer-edit-query.png)

此經過修改的查詢清單會根據文件的時間戳記，依遞減順序列出文件，因此現會最先列出您的第二份文件。 若您熟悉 SQL 語法，則可在此方塊中輸入任一受支援的 [SQL 查詢](../articles/cosmos-db/sql-api-sql-query.md)。 

其會在 [資料總管] 中完成工作。 繼續使用程式碼之前，請注意您也可以使用 [資料總管] 來建立預存程序、UDF 和觸發程序，以執行伺服器端商務邏輯及調整輸送量。 [資料總管] 會公開 API 中所有可用的內建程式設計資料存取，但可讓您在 Azure 入口網站中輕鬆存取您的資料。