---
title: 搭配 Azure Cosmos DB MongoDB API 使用 MongoDB 讀取喜好設定 | Microsoft 文件
description: 了解如何搭配 Azure Cosmos DB MongoDB API 使用 MongoDB 讀取喜好設定
services: cosmos-db
author: vidhoonv
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.custom: ''
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 02/26/2018
ms.author: sclyon
ms.openlocfilehash: 90c8d73e32f4c99c6871ce9cdb7839cd1d380b9b
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140892"
---
# <a name="how-to-globally-distribute-reads-using-read-preference-with-the-azure-cosmos-db-mongodb-api"></a>如何搭配 Azure Cosmos DB MongoDB API 使用讀取喜好設定全域散發讀取 

本文說明如何搭配 Azure Cosmos DB 的 MongoDB API 使用 [MongoDB 讀取喜好設定](https://docs.mongodb.com/manual/core/read-preference/) \(英文\) 全域散發讀取作業。 

## <a name="prerequisites"></a>必要條件 
如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 
[!INCLUDE [cosmos-db-emulator-mongodb](../../includes/cosmos-db-emulator-mongodb.md)]

如需使用 Azure 入口網站透過全域散發來設定 Azure Cosmos DB 帳戶，然後使用 MongoDB API 來連線的指示，請參閱此[快速入門](tutorial-global-distribution-mongodb.md)文章。

## <a name="clone-the-sample-application"></a>複製範例應用程式

開啟 Git 終端機視窗 (例如 Git Bash)，然後使用 `cd` 來切換到工作目錄。  

執行下列命令來複製範例存放庫。 根據您感興趣的平台，使用下列其中一個範例存放庫：

1. [.NET 範例應用程式](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference)
2. [NodeJS 範例應用程式]( https://github.com/Azure-Samples/azure-cosmos-db-mongodb-node-geo-readpreference)
3. [Mongoose 範例應用程式](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-mongoose-geo-readpreference)
4. [Java 範例應用程式](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference)
5. [SpringBoot 範例應用程式](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-spring)


```bash
git clone <sample repo url>
```

## <a name="run-the-application"></a>執行應用程式

根據使用的平台，安裝必要的套件，然後啟動應用程式。 若要安裝相依性，請依照範例應用程式存放庫中包含的讀我檔案進行。 例如，在 NodeJS 範例應用程式中，使用下列命令安裝必要套件，然後啟動應用程式。

```bash
cd mean
npm install
node index.js
```
應用程式會嘗試連線到 MongoDB 來源，並因為連接字串無效而失敗。 請依照讀我檔案中的步驟執行以更新連接字串 `url`。 此外，請將 `readFromRegion` 更新為 Azure Cosmos DB 帳戶中的讀取區域。 下列指示出自 NodeJS 範例：

```
* Next, substitute the `url`, `readFromRegion` in App.Config with your Cosmos DB account's values. 
```

執行這些步驟之後，範例應用程式會執行並會產生下列輸出：

```
connected!
readDefaultfunc query completed!
readFromNearestfunc query completed!
readFromRegionfunc query completed!
readDefaultfunc query completed!
readFromNearestfunc query completed!
readFromRegionfunc query completed!
readDefaultfunc query completed!
readFromSecondaryfunc query completed!
```

## <a name="read-using-read-preference-mode"></a>使用讀取喜好設定模式來讀取

MongoDB 會提供下列讀取喜好模式供用戶端使用：

1. PRIMARY
2. PRIMARY_PREFERRED
3. SECONDARY
4. SECONDARY_PREFERRED
5. NEAREST

如需這些讀取喜好設定模式各項行為的詳細資訊，請參閱詳盡的 [MongoDB 讀取喜好設定行為](https://docs.mongodb.com/manual/core/read-preference-mechanics/#replica-set-read-preference-behavior) \(英文\) 文件。 在 Azure Cosmos DB 中，主要區域會對應至寫入區域，而次要區域會對應至讀取區域。

根據常見案例，建議您使用下列設定：

1. 如果需要**低延遲讀取區域**，請使用 **NEAREST** 讀取喜好設定模式。 此設定會將讀取作業導向至最接近的可用區域。 請注意，如果最接近的區域是寫入區域，則會將這些作業導向至該區域。
2. 如果需要**高可用性與異地複寫的讀取** (延遲不是條件約束)，請使用 **SECONDARY_PREFERRED** 讀取喜好設定模式。 此設定會將讀取作業導向至可用的讀取區域。 如果沒有任何可用的讀取區域，則會將要求導向至寫入區域。

下列來自範例應用程式的程式碼片段示範如何在 NodeJS 中設定 NEAREST 讀取喜好設定：

```javascript
  var query = {};
  var readcoll = client.db('regionDB').collection('regionTest', {readPreference: ReadPreference.NEAREST});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromNearestfunc query completed!");
  });
```

同樣地，下面的程式碼片段會示範如何在 NodeJS 中設定 SECONDARY_PREFERRED 讀取喜好設定：

```javascript
  var query = {};
  var readcoll = client.db('regionDB').collection('regionTest', {readPreference: ReadPreference.SECONDARY_PREFERRED});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromSecondaryPreferredfunc query completed!");
  });
```

請參閱其他平台對應的範例應用程式存放庫，例如 [.NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference) 和 [Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference)。

## <a name="read-using-tags"></a>使用標記讀取

除了讀取喜好設定模式之外，MongoDB 還允許使用標記來導向讀取作業。 在適用於 MongoDB API 的 Azure Cosmos DB 中，預設會包含 `region` 標記當做 `isMaster` 回應的一部分：

```json
"tags": {
         "region": "West US"
      }
```

因此，MongoClient 可以將 `region` 標記和區域名稱一起使用，將讀取作業導向至特定區域。 針對 Azure Cosmos DB 帳戶，您可以在 Azure 入口網站左邊的 [設定] -> [全域複寫資料] 下找到區域名稱。 此設定對於達成**讀取隔離**很有幫助，但只限用戶端應用程式想要將讀取作業導向至特定區域的情況。 此設定適用於在背景中執行且不是實際執行的重要服務之類的非生產/分析類型案例。

下列來自範例應用程式的程式碼片段示範如何在 NodeJS 中搭配標記來設定讀取喜好設定：

```javascript
 var query = {};
  var readcoll = client.db('regionDB').collection('regionTest',{readPreference: new ReadPreference(ReadPreference.SECONDARY_PREFERRED, {"region": "West US"})});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromRegionfunc query completed!");
  });
```

請參閱其他平台對應的範例應用程式存放庫，例如 [.NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference) 和 [Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference)。

在本文中，您已了解如何搭配 Azure Cosmos DB 的 MongoDB API 使用讀取喜好設定來全域散發讀取作業。

## <a name="clean-up-resources"></a>清除資源

如果您將不繼續使用此應用程式，請使用下列步驟，在 Azure 入口網站中刪除此文章所建立的所有資源：

1. 從 Azure 入口網站的左側功能表中，按一下 [資源群組]，然後按一下您所建立資源的名稱。 
2. 在資源群組頁面上，按一下 [刪除]，在文字方塊中輸入要刪除之資源的名稱，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

* [將 MongoDB 資料匯入到 Azure Cosmos DB](mongodb-migrate.md)
* [設定全域複寫的 Azure Cosmos DB 帳戶並搭配 MongoDB API 使用](tutorial-global-distribution-mongodb.md)
* [使用模擬器在本機進行開發](local-emulator.md)
