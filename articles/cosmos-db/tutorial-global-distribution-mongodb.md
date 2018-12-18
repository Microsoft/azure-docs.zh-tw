---
title: 適用於 MongoDB API 的 Azure Cosmos DB 全域散發教學課程 | Microsoft Docs
description: 了解如何使用 MongoDB API 來設定 Azure Cosmos DB 全域散發。
services: cosmos-db
keywords: 全域散發, MongoDB
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: na
ms.topic: tutorial
ms.date: 05/10/2017
ms.author: sngun
ms.custom: mvc
ms.openlocfilehash: 1885c979fe2532d26b2e7b59111675bebee8ec05
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38668083"
---
# <a name="set-up-azure-cosmos-db-global-distribution-using-the-mongodb-api"></a>使用 MongoDB API 來設定 Azure Cosmos DB 全域散發

在本文中，我們會說明如何使用 Azure 入口網站來設定 Azure Cosmos DB 全域散發，然後使用 MongoDB API 來進行連線。

本文涵蓋下列工作： 

> [!div class="checklist"]
> * 使用 Azure 入口網站來設定全域散發
> * 使用 [MongoDB API](mongodb-introduction.md) 來設定全域散發

[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]

## <a name="verifying-your-regional-setup-using-the-mongodb-api"></a>使用 MongoDB API 來確認您的區域設定
若要在 API for MongoDB 內仔細檢查檢查您的全球組態，最簡單的方法就是從 Mongo 殼層執行 *isMaster()* 命令。

從您的 Mongo 殼層︰

   ```
      db.isMaster()
   ```
   
範例結果︰

   ```JSON
      {
         "_t": "IsMasterResponse",
         "ok": 1,
         "ismaster": true,
         "maxMessageSizeBytes": 4194304,
         "maxWriteBatchSize": 1000,
         "minWireVersion": 0,
         "maxWireVersion": 2,
         "tags": {
            "region": "South India"
         },
         "hosts": [
            "vishi-api-for-mongodb-southcentralus.documents.azure.com:10255",
            "vishi-api-for-mongodb-westeurope.documents.azure.com:10255",
            "vishi-api-for-mongodb-southindia.documents.azure.com:10255"
         ],
         "setName": "globaldb",
         "setVersion": 1,
         "primary": "vishi-api-for-mongodb-southindia.documents.azure.com:10255",
         "me": "vishi-api-for-mongodb-southindia.documents.azure.com:10255"
      }
   ```

## <a name="connecting-to-a-preferred-region-using-the-mongodb-api"></a>使用 MongoDB API 來連線到慣用的區域

MongoDB API 可讓您針對全域分散式資料庫，指定集合的讀取喜好設定。 為了兼顧低延遲讀取和全球高可用性，建議將集合的讀取喜好設定設為 [最接近]。 [最接近] 讀取喜好設定會設定為從最近的區域讀取。

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.Nearest));
```

如果應用程式具有主要讀取/寫入區域和次要地區來因應災害復原 (DR) 情況，我們建議將集合的讀取喜好設定設為 [慣用次要]。 [慣用次要] 讀取喜好設定會設定當主要區域無法使用時，從次要地區讀取。

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.SecondaryPreferred));
```

最後，如果您想要手動指定讀取區域， 您可以在讀取喜好設定內設定區域標記。

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
var tag = new Tag("region", "Southeast Asia");
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.Secondary, new[] { new TagSet(new[] { tag }) }));
```

就這麼簡單，這樣便已完成本教學課程。 您可以透過閱讀 [Azure Cosmos DB 中的一致性層級](consistency-levels.md)，來了解如何管理全域複寫帳戶的一致性。 如需有關 Azure Cosmos DB 中全域資料庫複寫運作方式的詳細資訊，請參閱[使用 Azure Cosmos DB 來全域散發資料](distribute-data-globally.md)。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列操作：

> [!div class="checklist"]
> * 使用 Azure 入口網站來設定全域散發
> * 使用 SQL API 來設定全域散發

您現在可以繼續進行到下一個教學課程，以了解如何使用 Azure Cosmos DB 本機模擬器在本機進行開發。

> [!div class="nextstepaction"]
> [使用模擬器在本機進行開發](local-emulator.md)
