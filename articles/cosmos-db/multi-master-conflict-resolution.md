---
title: Azure Cosmos DB 中的多重主機衝突解決方法
description: 本文說明 Azure Cosmos DB 多重主機中的衝突類別和衝突解決模式，例如「最後寫入為準 (LWW)」、「自訂 – 使用者定義程序」、「自訂 – 非同步」。
services: cosmos-db
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: mjbrown
ms.reviewer: sngun
ms.openlocfilehash: 5feefdb8fe6204bc8ef42a5e65bf1e30354e0cf9
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47393922"
---
# <a name="multi-master-conflict-resolution-in-azure-cosmos-db"></a>Azure Cosmos DB 中的多重主機衝突解決方法 

Azure Cosmos DB 多重主機提供全域衝突解決模式，以確保資料在其複寫所在的所有區域中保有一致性。

## <a name="conflict-categories"></a>衝突類別

在使用 Azure Cosmos DB 資料時可能發生三種衝突：

* **插入衝突：** 應用程式同時從兩個或更多區域插入具有相同唯一索引 (例如識別碼) 的兩個或更多文件時，即會發生此類型的衝突。 在此情況下，所有的寫入最初可能都會成功，但根據您所選擇的衝突解決原則，將只會認可一份具有原始識別碼的文件。

* **取代衝突：** 應用程式同時從兩個或更多區域更新單一文件時，即會發生此類型的衝突。

* **刪除衝突：** 應用程式從單一區域刪除某文件後，從兩個或更多區域加以更新時，即會發生此類型的衝突。 

## <a name="conflict-resolution-modes"></a>衝突解決模式

Azure Cosmos DB 提供了三種衝突解決模式。 系統會為每個集合定義衝突解決模式。

> [!NOTE]
> SQL API 使用者可從三種不同的衝突解決模式中擇一使用。 對於所有其他的 API 模型 (MongoDB、Cassandra、圖形和資料表)，都會使用「最後寫入為準」原則來解決衝突。

### <a name="last-writer-wins-lww"></a>最後寫入為準 (LWW)

「最後寫入為準」是預設的衝突解決模式。 在此模式中，會根據在文件的屬性中傳入的數值來解決衝突。

下列程式碼片段是在使用 .Net SDK 時如何設定「最後寫入為準」衝突解決原則的範例。 "ConflictResolutionPath" 會定義用來解決衝突的屬性路徑。 在此範例中，`/userDefinedId` 是衝突解決路徑，而具有最大 `userDefinedId` 值的文件將一律在衝突中勝出。 若要註冊「最後寫入為準」解決模式，請使用 ConflictResolutionPolicy 佈建集合，如下所示。

```csharp
DocumentCollection lwwCollection = await myClient.CreateDocumentCollectionIfNotExistsAsync(UriFactory.CreateDatabaseUri("myDatabase"), new DocumentCollection
{
   Id = "myCollection", 
   ConflictResolutionPolicy = new ConflictResolutionPolicy
   {
      Mode = ConflictResolutionMode.LastWriterWins,
      ConflictResolutionPath = "/userDefinedId"
   }
});
```

 「最後寫入為準」解決模式會套用至不同的資料衝突類別，如下所示：

* **插入和更新衝突：** 如果有兩個或更多文件的插入或取代作業發生衝突，則具有最大的衝突解決途徑值的文件將會勝出 (也就是 userDefinedId)。 如果有多份文件的衝突解決途徑具有相同數值，則選取的優勝者將不確定。 不過，所有區域將會匯整出單一優勝者。

* **刪除衝突：** 如果涉及刪除衝突，則刪除一律會優先於其他取代衝突，不論衝突解決途徑的值為何。

### <a name="custom--user-defined-procedure"></a>自訂 – 使用者定義程序

在此模式中，使用者可藉由對集合註冊使用者定義程序 (UDP) 來控制衝突解決方式。 此 UDP 具有特定簽章。 如果您選取此選項，但未能註冊 UDP，或如果 UDP 在執行階段擲回例外狀況，則衝突會寫入至衝突摘要，在此處可以個別解決衝突。

若要註冊「自訂 – 使用者定義程序」解決模式，請使用 ConflictResolutionPolicy 佈建集合，如下所示。

```csharp
DocumentCollection udpCollection = await myClient.CreateDocumentCollectionIfNotExistsAsync(UriFactory.CreateDatabaseUri("myDatabase"), new DocumentCollection
{
  Id = "myCollection",
  ConflictResolutionPolicy = new ConflictResolutionPolicy
  {
     Mode = ConflictResolutionMode.Custom,
     ConflictResolutionProcedure = UriFactory.CreateStoredProcedureUri("myDatabase","myCollection","myUdpStoredProcedure").ToString()
  }
});
```

接著，請將使用者定義的程序新增至集合，如下所示。

```csharp
StoredProcedure myUdpStoredProc = new StoredProcedure
{
   Id = "myUdpStoredProcedure",
   Body = myUdpStoredProcText
};
await myClient.UpsertStoredProcedureAsync(UriFactory.CreateDocumentCollectionUri("myDatabase", "myCollection"), myUdpStoredProc);
```

對集合註冊的預存程序會具有特殊簽章。 在下列範例中，UDP 會使用屬性 `UserDefinedId` 來解決衝突。 具有最大值的文件會在衝突中勝出。

```javascript
function myUdpStoredProcedure(incomingDocument, existingDocument, isDeleteConflict, conflictingDocuments){
    var collection = getContext().getCollection();

    if (!incomingDocument) {
        if (existingDocument) {

            collection.deleteDocument(existingDocument._self, {}, function(err, responseOptions) {
                if (err) throw err;
            });
        }
    } else if (isDeleteConflict) {
        // delete always wins.
    } else {
        var documentToUse = incomingDocument;

        if (existingDocument) {
            if (documentToUse.userDefinedId < existingDocument.userDefinedId) {
                documentToUse = existingDocument;
            }
        }

        var i;
        for (i = 0; i < conflictingDocuments.length; i++) {
            if (documentToUse.userDefinedId < conflictingDocuments[i].userDefinedId) {
                documentToUse = conflictingDocuments[i];
            }
        }

        tryDelete(conflictingDocuments, incomingDocument, existingDocument, documentToUse);
    }

    function tryDelete(documents, incoming, existing, documentToInsert) {
        if (documents.length > 0) {
            collection.deleteDocument(documents[0]._self, {}, function(err, responseOptions) {
                if (err) throw err;

                documents.shift();
                tryDelete(documents, incoming, existing, documentToInsert);
            });
        } else if (existing) {
            collection.replaceDocument(existing._self, documentToInsert,
                function(err, documentCreated) {
                    if (err) throw err;
                });
        } else {
            collection.createDocument(collection.getSelfLink(), documentToInsert,
                function(err, documentCreated) {
                    if (err) throw err;
                });
        }
    }
}

```

此程序有四個參數：

* **incomingDocument：** 指定文件的版本衝突。 衝突可以是插入、取代或刪除衝突。 就刪除衝突而言，這將是沒有內容的刪除影像 (標記)。

* **existingDocument：** 指定認可版本的文件，該文件具有與 incomingDocument 相同的 "rid" 值。 針對插入與刪除衝突，此參數會設定為 Null。

* **isDeleteConflict：** 一個布林旗標，指出內送文件是否與先前刪除的文件衝突。 當此參數設定為 true 時，existingDocument 也會是 Null。

* **conflictingDocuments：** 指定資料庫中所有文件的認可版本集合，且這些文件與識別碼資料行上的 incomingDocument 或任何其他唯一索引欄位相衝突。 與 incomingDocument 比較時，這些文件會有不同的 "rid" 值。

使用者定義的程序具有 Cosmos DB 分割區索引鍵的完整存取權，並且可執行任何儲存作業以解決衝突。 如果使用者定義的程序未認可衝突版本，系統將會捨棄該衝突，而仍認可 existingDocument。 如果使用者定義的程序失敗或不存在，則 Azure Cosmos DB 會將衝突新增至唯讀衝突摘要中，讓衝突可進行非同步處理，如[非同步衝突解決模式]()所說明。 

### <a name="custom--asynchronous"></a>自訂 – 非同步  

在此模式中，Azure Cosmos DB 會將所有衝突 (插入、取代和刪除) 排除於認可作業外，並將其註冊到唯讀衝突摘要中，延後給使用者的應用程式解決。 應用程式可用非同步方式執行衝突解決，並使用任何邏輯或參考任何外部來源、應用程式或服務來解決衝突。

若要註冊「自訂 – 非同步」解決模式，請使用 ConflictResolutionPolicy 佈建集合，如下所示。

```csharp
DocumentCollection manualCollection = await myClient.CreateDocumentCollectionIfNotExistsAsync(UriFactory.CreateDatabaseUri("myDatabase"), new DocumentCollection
{
    Id = "myCollection",
    ConflictResolutionPolicy = new ConflictResolutionPolicy
    {
        Mode = ConflictResolutionMode.Custom
    }
});
```

若要讀取及處理衝突摘要中的任何衝突，請依照下列方式實作程式碼。 儲存在衝突摘要中的資料會增加些許儲存成本。 因此，建議您在處理完儲存於衝突摘要中的資料之後加以刪除。

```csharp
FeedResponse<Conflict> response = await myClient.ReadConflictFeedAsync(myCollectionUri);
  foreach (Conflict conflict in response)
  {
      switch(conflict.OperationKind)
      {
         case OperationKind.Create:
         //Process Insert Conflicts.
         …
         case OperationKind.Replace:
         //Process Replace Conflicts
         …
         case OperationKind.Delete:
         //Process Delete Conflicts
         …
      }
  //Optionally delete the conflict after processed
  await myClient.DeleteConflictAsync(conflict.SelfLink);
  }
```

> [!NOTE]
> 衝突摘要不包含可傳送通知以進行下游處理 (例如 Cosmos DB 中的變更摘要) 的接聽程式。 您必須實作邏輯以輪詢衝突摘要，並判斷是否有衝突存在。

## <a name="code-samples"></a>程式碼範例

下列範例應用程式將示範如何解決所列 API 的衝突。 每個範例都會在容器內產生衝突，然後示範在每個支援的衝突解決模式中如何解決衝突。

|API 模型  | SDK |範例 |
|---------|---------|---------|
|SQL API    | .NET    |[azure-cosmos-db-sql-dotnet-multi-master](https://github.com/Azure-Samples/azure-cosmos-db-sql-dotnet-multi-master)  |
|SQL API    | 節點    |[azure-cosmos-js/samples/MultiRegionWrite/](https://github.com/Azure/azure-cosmos-js/tree/master/samples/MultiRegionWrite)  |
|SQL API    | Java    |[azure-cosmos-db-sql-java-multi-master](https://github.com/Azure-Samples/azure-cosmos-db-sql-java-multi-master)  |
|MongoDB  | .NET    |[azure-cosmos-db-mongodb-dotnet-multi-master](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-multi-master)   |
|資料表 API  | .NET    |[azure-cosmos-db-table-dotnet-multi-master](https://github.com/Azure-Samples/azure-cosmos-db-table-dotnet-multi-master)       |
|Gremlin API | .NET | [azure-cosmos-db-gremlin-dontnet-multi-master](https://github.com/Azure-Samples/azure-cosmos-db-gremlin-dontnet-multi-master)|

## <a name="next-steps"></a>後續步驟

在本文中，您已了解 Azure Cosmos DB 的衝突類別和衝突解決模式。 接著，請參閱下列資源：

* [搭配多重主機使用 MongoDB 用戶端](multi-master-oss-nosql.md)
