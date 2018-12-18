---
title: 適用於資料表 API 的 Azure Cosmos DB 全域散發教學課程 | Microsoft Docs
description: 了解如何使用資料表 API 來設定 Azure Cosmos DB 全域散發。
services: cosmos-db
keywords: 全域散發, 資料表
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: na
ms.topic: tutorial
ms.date: 12/13/2017
ms.author: sngun
ms.custom: mvc
ms.openlocfilehash: 7baa1e5b4b50d1f420b514854fa4d4c9ef786c99
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38582130"
---
# <a name="set-up-azure-cosmos-db-global-distribution-using-the-table-api"></a>使用資料表 API 來設定 Azure Cosmos DB 全域散發

本文涵蓋下列工作： 

> [!div class="checklist"]
> * 使用 Azure 入口網站來設定全域散發
> * 使用[資料表 API](table-introduction.md) 來設定全域散發

[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]


## <a name="connecting-to-a-preferred-region-using-the-table-api"></a>使用資料表 API 來連線到慣用的區域

為了充分運用 [全球發佈](distribute-data-globally.md)，用戶端應用程式可以指定已排序的區域喜好設定清單，以用來執行文件作業。 這可藉由設定 [TableConnectionPolicy.PreferredLocations](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.table.tableconnectionpolicy.preferredlocations?view=azure-dotnet#Microsoft_Azure_CosmosDB_Table_TableConnectionPolicy_PreferredLocations) 屬性來完成。 Azure Cosmos DB 資料表 API SDK 會根據帳戶組態、目前的區域可用性及所指定的喜好設定清單，挑選最適合通訊的端點。

PreferredLocations 應該包含一個供讀取使用的慣用 (多路連接) 位置逗號分隔清單。 每個用戶端執行個體可以依慣用順序指定這些區域的子集，供低延遲讀取使用。 指定這些區域時，必須使用其[顯示名稱](https://msdn.microsoft.com/library/azure/gg441293.aspx)，例如 `West US`。

所有讀取都會傳送至 PreferredLocations 清單中的第一個可用區域。 如果要求失敗，用戶端將無法往下到清單中的下一個區域，依此類推。

SDK 會嘗試從 PreferredLocations 中指定的區域讀取。 因此，比方說，如果資料庫帳戶可供三個區域使用，但用戶端只針對 PreferredLocations 指定其中兩個非寫入區域，則將不會在該寫入區域以外的地方提供讀取服務，即使發生容錯移轉也一樣。

SDK 會自動將所有寫入傳送至目前的寫入區域。

如果未設定 PreferredLocations 屬性，將會從目前的寫入區域為所有要求提供服務。

就這麼簡單，這樣便已完成本教學課程。 您可以透過閱讀 [Azure Cosmos DB 中的一致性層級](consistency-levels.md)，來了解如何管理全域複寫帳戶的一致性。 如需有關 Azure Cosmos DB 中全域資料庫複寫運作方式的詳細資訊，請參閱[使用 Azure Cosmos DB 來全域散發資料](distribute-data-globally.md)。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列操作：

> [!div class="checklist"]
> * 使用 Azure 入口網站來設定全域散發
> * 使用 Azure Cosmos DB 資料表 API 來設定全域散發

