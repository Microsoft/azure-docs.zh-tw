---
title: 從 Azure 資料總管刪除資料
description: 此文章說明 Azure 資料總管中的大量刪除案例，包括清除和以保留期為基礎的刪除。
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
services: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 571a005dd3f50690f291a7ffa3c1174ea15cb0ed
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045850"
---
# <a name="delete-data-from-azure-data-explorer"></a>從 Azure 資料總管刪除資料

在此文章中，我們將討論 Azure 資料總管支援的數種大量刪除方法。 它不支援即時刪除每筆記錄，因為它針對快速讀取存取進行了最佳化。

* 如果不再需要資料庫，請使用 drop database 命令來刪除它。

    ```Kusto
    .drop database <DatabaseName>
    ```

* 如果不再需要一或多個資料表，則可使用 drop table 或 drop tables 命令來刪除它們。

    ```Kusto
    .drop table <TableName>

    .drop tables (<TableName1>, <TableName2>,...)
    ```

* 如果不再需要舊資料，可透過變更資料庫或資料表層級的保留期限來刪除它。

    假設有一個保留期設為 90 天的資料庫或資料表。 由於商務需求發生變更，現在只需要 60 天的資料。 在此情況下，以下列其中一種方式刪除較舊的資料。

    ```Kusto
    .alter-merge database <DatabaseName> policy retention softdelete = 60d

    .alter-merge table <TableName> policy retention softdelete = 60d
    ```

    如需詳細資訊，請參閱[保留原則](https://docs.microsoft.com/azure/kusto/concepts/retentionpolicy) \(英文\)。

* 您可以根據像是 `where CustomerName == 'contoso'` 的述詞，使用 *purge* 作業來刪除個別記錄。 也就是說，清除是一種並非設計來進行即時刪除的大量刪除。 下列範例示範清除。

    ```Kusto
    .purge table Customer records
    | where CustomerName =='contoso'
    ```

若您需要協助來解決資料刪除問題，請在 [Azure 入口網站](https://portal.azure.com)中開立支援要求。