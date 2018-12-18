---
title: Azure Cosmos DB 的伺服器端 JavaScript 程式設計 | Microsoft Docs
description: 了解如何使用 Azure Cosmos DB 來撰寫 JavaScript 預存程序、資料庫觸發程序和使用者定義函數 (UDF)。 取得資料庫程式設計秘訣等等。
keywords: 資料庫觸發程序, 預存程序, 預存程序, 資料庫程式, sproc, azure, Microsoft azure
services: cosmos-db
author: aliuy
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: andrl
ms.openlocfilehash: 8377b13014e2f97518bbc779ee809aaa10d6eb45
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287439"
---
# <a name="azure-cosmos-db-server-side-programming-stored-procedures-database-triggers-and-udfs"></a>Azure Cosmos DB 伺服器端程式設計：預存程序、資料庫觸發程序和 UDF

了解 Azure Cosmos DB 的語言整合式、交易式 JavaScript 執行如何讓開發人員使用 [ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/) JavaScript 以原生方式撰寫**預存程序**、**觸發程序**及**使用者定義函式 (UDF)**。 Javascript 整合可讓您撰寫能夠直接在資料庫儲存體分割區內傳送和執行的程式邏輯。 

在這篇文章中，您可找到下列問題的答案：  

* 如何使用 JavaScript 來撰寫預存程序、觸發程序或 UDF？
* Cosmos DB 如何提供 ACID 保證？
* 如何在 Cosmos DB 中執行交易工作？
* 什麼是預先觸發程序和後續觸發程序？其撰寫方法為何？
* 如何註冊及使用 HTTP 以 RESTful 方式執行預存程序、觸發程序或 UDF？
* 哪些 Cosmos DB SDK 可用來建立及執行預存程序、觸發程序和 UDF？

## <a name="introduction-to-stored-procedure-and-udf-programming"></a>預存程序和 UDF 程式設計簡介
這種「 *以 JavaScript 做為新式 T-SQL* 」的方式，可讓應用程式開發人員不必傷腦筋處理複雜的類型系統不符問題和物件關聯式對應技術。 此外，它本身還有一些可加以利用以便建置豐富應用程式的優勢：  

* **程序邏輯** ：JavaScript 是高階程式設計語言，可提供豐富且熟悉的介面來表達商務邏輯。 您可以用更接近資料的方式執行一連串的複雜作業。
* **不可部分完成的交易**：Cosmos DB 可確保在單一預存程序或觸發程序內執行的資料庫作業成為不可部分完成的作業。 這個不可部分完成的功能可讓應用程式將相關作業合併在單一批次中，讓所有作業不是一起成功就是一起失敗。 
* **效能**：由於 JSON 本身就對應至 Javascript 語言類型系統並同時是 Cosmos DB 中儲存體基本單位，因此可提供許多最佳化功能，例如在緩衝集區中對 JSON 文件執行滯後具體化，並讓執行中程式碼可以依需求使用這些文件。 除此之外，還有其它與傳送商務邏輯至資料庫相關聯的效能優點：
  
  * 批次處理 - 開發人員可以群組多個作業 (例如插入)，大量進行提交。 因此，網路流量延遲成本以及建立個別交易的額外儲存負荷得以大幅降低。 
  * 預先編譯 - Cosmos DB 會預先編譯預存程序、觸發程序和使用者定義函式 (UDF)，以避免在每次叫用時產生 JavaScript 編譯成本。 因此，建置程序邏輯位元組程式碼的額外負荷已降到最低。
  * 排序 - 許多作業都需要副作用 (「觸發程序」)，其可能涉及執行一項或多項次要儲存作業。 除了不可部分完成的作業之外，這在移至伺服器時會有更好的效能。 
* **封裝** ：預存程序可以用來將商務邏輯群組在一個位置，這有兩個優點：
  * 它會在未經處理的資料上方新增抽象層，讓資料架構設計人員發展其應用程式，而不會動到資料。 當資料無結構描述時，這個抽象層相當有利，因為如果他們必須直接處理資料，就可能需要將這些短暫的假設納入應用程式考量。  
  * 這個抽象層讓企業得以透過指令碼簡化存取來確保資料安全。  

許多平台 (包括 .NET、Node.js 和 JavaScript) 都透過 [Azure 入口網站](https://portal.azure.com)、[REST API](/rest/api/cosmos-db/)、[Azure DocumentDB Studio](https://github.com/mingaliu/DocumentDBStudio/releases) 和[用戶端 SDK](sql-api-sdk-dotnet.md)，以支援資料庫觸發程序、預存程序及自訂查詢運算子的建立和執行。

本教學課程使用 [Node.js SDK 搭配 Q Promises](http://azure.github.io/azure-documentdb-node-q/) 來說明預存程序、觸發程序及 UDF 的語法和用法。   

## <a name="stored-procedures"></a>預存程序
### <a name="example-write-a-stored-procedure"></a>範例：撰寫預存程序
我們先來看看簡單的預存程序，此程序會傳回 "Hello World" 回應。

```javascript
var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
}
```

預存程序是按照集合來註冊，而且可以作用於該集合中現有的文件和附件。 下列程式碼片段說明如何向集合註冊 helloWorld 預存程序。 


```javascript
// register the stored procedure
var createdStoredProcedure;
client.createStoredProcedureAsync('dbs/testdb/colls/testColl', helloWorldStoredProc)
    .then(function (response) {
        createdStoredProcedure = response.resource;
        console.log("Successfully created stored procedure");
    }, function (error) {
        console.log("Error", error);
    });
```


註冊好預存程序之後，您便可以針對集合執行預存程序，並將結果讀取回用戶端。 

```javascript
// execute the stored procedure
client.executeStoredProcedureAsync('dbs/testdb/colls/testColl/sprocs/helloWorld')
    .then(function (response) {
        console.log(response.result); // "Hello, World"
    }, function (err) {
        console.log("Error", error);
    });
```

內容物件提供可對 Cosmos DB 儲存體執行之所有作業的存取權，以及要求和回應物件的存取權。 在此案例中，您會使用回應物件來設定傳回給用戶端的回應本文。 如需詳細資料，請參閱 [Azure Cosmos DB JavaScript 伺服器 SDK 文件](https://azure.github.io/azure-cosmosdb-js-server/) \(英文\)。  

我們將以這個範例為基礎進行擴充，將更多資料庫相關功能新增到預存程序中。 預存程序可以建立、更新、讀取、查詢及刪除集合內的文件和附件。    

### <a name="example-write-a-stored-procedure-to-create-a-document"></a>範例：撰寫建立文件的預存程序
下一個程式碼片段說明如何使用內容物件來與 Cosmos DB 資源互動。

```javascript
var createDocumentStoredProc = {
    id: "createMyDocument",
    serverScript: function createMyDocument(documentToCreate) {
        var context = getContext();
        var collection = context.getCollection();

        var accepted = collection.createDocument(collection.getSelfLink(),
              documentToCreate,
              function (err, documentCreated) {
                  if (err) throw new Error('Error' + err.message);
                  context.getResponse().setBody(documentCreated.id)
              });
        if (!accepted) return;
    }
}
```


此預存程序將 documentToCreate (要在目前集合中建立的文件本文) 做為輸入。 所有這類作業都是非同步的，而且需仰賴 JavaScript 函數回呼。 回呼函數有兩個參數，一個用於作業失敗時的錯誤物件，一個用於已建立的物件。 在回呼內，使用者可以處理例外狀況或擲回錯誤。 如果未提供回呼，而且發生錯誤，則 Azure Cosmos DB 執行階段會擲回錯誤。   

在上面的範例中，回呼會在作業失敗時擲回錯誤。 否則，會將已建立之文件的識別碼設定為用戶端回應的本文。 此預存程序與輸入參數搭配執行的方式如下。

```javascript
// register the stored procedure
client.createStoredProcedureAsync('dbs/testdb/colls/testColl', createDocumentStoredProc)
    .then(function (response) {
        var createdStoredProcedure = response.resource;

        // run stored procedure to create a document
        var docToCreate = {
            id: "DocFromSproc",
            book: "The Hitchhiker’s Guide to the Galaxy",
            author: "Douglas Adams"
        };

        return client.executeStoredProcedureAsync('dbs/testdb/colls/testColl/sprocs/createMyDocument',
              docToCreate);
    }, function (error) {
        console.log("Error", error);
    })
.then(function (response) {
    console.log(response); // "DocFromSproc"
}, function (error) {
    console.log("Error", error);
});
```

您可以修改此預存程序，將一批文件本文作為輸入，並將這些本文全都建立在相同的預存程序執行內，而不是用多個要求個別建立這些本文。 您可以使用此預存程序來實作有效率的 Cosmos DB 大量匯入工具 (本教學課程稍後將會討論)。   

上面描述的範例已示範如何使用預存程序。 接著，您將在本教學課程後續的內容中了解觸發程序和使用者定義函式 (UDF)。

### <a name="known-issues"></a>已知問題

使用 Azure 入口網站定義預存程序時，輸入參數一律會以字串形式傳送到預存程序。 即使您以輸入形式傳遞字串陣列，陣列會轉換成字串並傳送至預存程序。 若要解決此問題，您可以在預存程序內定義一個函式，將字串剖析為陣列。 下列程式碼是將字串剖析為陣列的範例： 

```javascript
function sample(arr) {
    if (typeof arr === "string") arr = JSON.parse(arr);
    
    arr.forEach(function(a) {
        // do something here
        console.log(a);
    });
}
```

## <a name="database-program-transactions"></a>資料庫程式交易
一般資料庫中的交易可以定義為以單一工作邏輯單位執行的一連串作業。 每個交易都提供「ACID 保證」 。 ACID 是一個已知的縮寫，代表四個屬性：不可部分完成性 (Atomicity)、一致性 (Consistency)、隔離性 (Isolation) 及耐用性 (Durability)。  

簡言之，不可部分完成的作業保證會將交易內完成的所有工作視為單一單位，所有工作不是全部認可就是一個都不認可。 「一致性」確保資料在交易中一律處於良好內部狀態。 「隔離」保證兩個交易不會彼此干擾；一般而言，大部分的商業系統都會提供多個可以根據應用程式的需要來使用的隔離等級。 「持久性」確保資料庫中所認可的任何變更一律會存在。   

在 Cosmos DB 中，會在與資料庫相同的記憶體空間中裝載 JavaScript。 因此，在預存程序和觸發程序內提出的要求會在資料庫工作階段的相同範圍中執行。 此功能讓 Cosmos DB 能夠為屬於單一預存程序/觸發程序的所有作業提供 ACID 保證。 我們看一下下列預存程序定義：

```javascript
// JavaScript source code
var exchangeItemsSproc = {
    id: "exchangeItems",
    serverScript: function (playerId1, playerId2) {
        var context = getContext();
        var collection = context.getCollection();
        var response = context.getResponse();

        var player1Document, player2Document;

        // query for players
        var filterQuery = 'SELECT * FROM Players p where p.id  = "' + playerId1 + '"';
        var accept = collection.queryDocuments(collection.getSelfLink(), filterQuery, {},
            function (err, documents, responseOptions) {
                if (err) throw new Error("Error" + err.message);

                if (documents.length != 1) throw "Unable to find both names";
                player1Document = documents[0];

                var filterQuery2 = 'SELECT * FROM Players p where p.id = "' + playerId2 + '"';
                var accept2 = collection.queryDocuments(collection.getSelfLink(), filterQuery2, {},
                    function (err2, documents2, responseOptions2) {
                        if (err2) throw new Error("Error" + err2.message);
                        if (documents2.length != 1) throw "Unable to find both names";
                        player2Document = documents2[0];
                        swapItems(player1Document, player2Document);
                        return;
                    });
                if (!accept2) throw "Unable to read player details, abort ";
            });

        if (!accept) throw "Unable to read player details, abort ";

        // swap the two players’ items
        function swapItems(player1, player2) {
            var player1ItemSave = player1.item;
            player1.item = player2.item;
            player2.item = player1ItemSave;

            var accept = collection.replaceDocument(player1._self, player1,
                function (err, docReplaced) {
                    if (err) throw "Unable to update player 1, abort ";

                    var accept2 = collection.replaceDocument(player2._self, player2,
                        function (err2, docReplaced2) {
                            if (err) throw "Unable to update player 2, abort"
                        });

                    if (!accept2) throw "Unable to update player 2, abort";
                });

            if (!accept) throw "Unable to update player 1, abort";
        }
    }
}

// register the stored procedure in Node.js client
client.createStoredProcedureAsync(collection._self, exchangeItemsSproc)
    .then(function (response) {
        var createdStoredProcedure = response.resource;
    }
);
```

此預存程序使用遊戲應用程式內的交易，透過單一作業讓兩位玩家交易項目。 預存程序嘗試讀取兩份文件，這兩份文件各自對應到以引數形式傳入的玩家 ID。 如果有找到這兩份玩家文件，則預存程序會透過交換他們的項目來更新文件。 如果過程中發生任何錯誤，則會擲回以隱含方式中止交易的 JavaScript 例外狀況。

如果預存程序註冊的集合是單一分割集合，則交易的範圍為集合中的所有文件。 如果集合已分割，則預存程序會在單一分割索引鍵的交易範圍內執行。 然後每個預存程序執行必須包含分割索引鍵值，該值對應至必須在其下執行交易的範圍。 如需詳細資訊，請參閱 [Azure Cosmos DB 資料分割](partition-data.md)。

### <a name="commit-and-rollback"></a>認可和回復
交易原本就深入整合至 Cosmos DB 的 JavaScript 程式設計模型。 在 JavaScript 函數內，會將所有作業自動包裝在單一交易內。 如果 JavaScript 完成，而且沒有任何例外狀況，就會認可資料庫作業。 在 Cosmos DB 中，關聯式資料庫中的 "BEGIN TRANSACTION" 和 "COMMIT TRANSACTION" 陳述式實際上是隱含的。  

如果有任何透過指令碼傳播的例外狀況，則 Cosmos DB 的 JavaScript 執行階段將會復原整個交易。 如稍早的範例所示，擲回例外狀況的作用等同於 Cosmos DB 中的 "ROLLBACK TRANSACTION"。

### <a name="data-consistency"></a>資料一致性
預存程序和觸發程序一律會在 Azure Cosmos DB 容器的主要複本上執行。 這確保從預存程序讀取的資料有強式一致性。 使用「使用者定義函式」的查詢可以在主要或任何次要複本上執行，但是您會透過選擇適當的複本，確保符合所要求的一致性層級。

## <a name="bounded-execution"></a>界限執行
所有 Cosmos DB 作業都必須在伺服器指定的要求逾時期間內完成。 此條件約束也適用於 JavaScript 函式 (預存程序、觸發程序和使用者定義函式)。 如果作業未在該時限內完成，則會回復交易。 JavaScript 函式必須在時限內完成，或實作接續式模型來批次處理/繼續執行。  

若要簡化預存程序和觸發程序的開發流程以因應此時限，集合物件下的所有函數 (用於建立、讀取、取代與刪除文件和附件) 都會傳回布林值，以指出該作業是否會完成。 如果此值是 false，則表示即將到達時限，該程序必須包裝執行作業。  如果預存程序及時完成，而且佇列中已無其他要求，則在第一個不被接受之儲存作業之前排入佇列的作業保證會完成。  

JavaScript 函數能使用的資源也受到限制。 Cosmos DB 會保留每一集合或一組容器的輸送量。 輸送量是以 CPU、記憶體和 IO 使用量的標準單位 (稱為要求單位或 RU) 來表示。 JavaScript 函數有可能會在短時間內使用大量 RU，如果達到集合限制，速率便會受到限制。 系統也可能隔離使用大量資源的預存程序，以確保基本資料庫作業的可用性。  

### <a name="example-bulk-importing-data-into-a-database-program"></a>範例：將大量資料匯入資料庫程式
以下的預存程序範例，其撰寫目的是要將文件大量匯入集合之中。 請注意預存程序如何透過檢查 createDocument 所傳回的布林值處理界限執行，然後使用每次叫用預存程序時所插入的文件計數來追蹤和繼續各批次的進度。

```javascript
function bulkImport(docs) {
    var collection = getContext().getCollection();
    var collectionLink = collection.getSelfLink();

    // The count of imported docs, also used as current doc index.
    var count = 0;

    // Validate input.
    if (!docs) throw new Error("The array is undefined or null.");

    var docsLength = docs.length;
    if (docsLength == 0) {
        getContext().getResponse().setBody(0);
    }

    // Call the create API to create a document.
    tryCreate(docs[count], callback);

    // Note that there are 2 exit conditions:
    // 1) The createDocument request was not accepted. 
    //    In this case the callback will not be called, we just call setBody and we are done.
    // 2) The callback was called docs.length times.
    //    In this case all documents were created and we don’t need to call tryCreate anymore. Just call setBody and we are done.
    function tryCreate(doc, callback) {
        var isAccepted = collection.createDocument(collectionLink, doc, callback);

        // If the request was accepted, callback will be called.
        // Otherwise report current count back to the client, 
        // which will call the script again with remaining set of docs.
        if (!isAccepted) getContext().getResponse().setBody(count);
    }

    // This is called when collection.createDocument is done in order to process the result.
    function callback(err, doc, options) {
        if (err) throw err;

        // One more document has been inserted, increment the count.
        count++;

        if (count >= docsLength) {
            // If we created all documents, we are done. Just set the response.
            getContext().getResponse().setBody(count);
        } else {
            // Create next document.
            tryCreate(docs[count], callback);
        }
    }
}
```

## <a id="trigger"></a> 資料庫觸發程序
### <a name="database-pre-triggers"></a>資料庫預先觸發程序
Cosmos DB 提供作業在文件上執行或觸發的觸發程序。 例如，您可以在建立文件時指定預先觸發程序；此預先觸發程序會在建立文件之前執行。 下列範例說明如何使用預先觸發程序來驗證所要建立之文件的屬性：

```javascript
var validateDocumentContentsTrigger = {
    id: "validateDocumentContents",
    serverScript: function validate() {
        var context = getContext();
        var request = context.getRequest();

        // document to be created in the current operation
        var documentToCreate = request.getBody();

        // validate properties
        if (!("timestamp" in documentToCreate)) {
            var ts = new Date();
            documentToCreate["my timestamp"] = ts.getTime();
        }

        // update the document that will be created
        request.setBody(documentToCreate);
    },
    triggerType: TriggerType.Pre,
    triggerOperation: TriggerOperation.Create
}
```

以及觸發程序的對應 Node.js 用戶端註冊程式碼：

```javascript
// register pre-trigger
client.createTriggerAsync(collection.self, validateDocumentContentsTrigger)
    .then(function (response) {
        console.log("Created", response.resource);
        var docToCreate = {
            id: "DocWithTrigger",
            event: "Error",
            source: "Network outage"
        };

        // run trigger while creating above document 
        var options = { preTriggerInclude: "validateDocumentContents" };

        return client.createDocumentAsync(collection.self,
              docToCreate, options);
    }, function (error) {
        console.log("Error", error);
    })
.then(function (response) {
    console.log(response.resource); // document with timestamp property added
}, function (error) {
    console.log("Error", error);
});
```

預先觸發程序不能有任何輸入參數。 要求物件可以用來操作與作業相關聯的要求訊息。 在這裡，預先觸發程序會在建立文件時執行，而且要求訊息本文包含要以 JSON 格式建立的文件。   

註冊觸發程序時，使用者可以指定與之搭配執行的作業。 此觸發程序是使用 TriggerOperation.Create 所建立，這表示不允許在下列程式碼中將觸發程序用於取代作業中。

```javascript
var options = { preTriggerInclude: "validateDocumentContents" };

client.replaceDocumentAsync(docToReplace.self,
              newDocBody, options)
.then(function (response) {
    console.log(response.resource);
}, function (error) {
    console.log("Error", error);
});

// Fails, can’t use a create trigger in a replace operation
```
### <a name="database-post-triggers"></a>資料庫後續觸發程序
後續觸發程序與預先觸發程序類以，都與文件上的作業相關聯，且未採用任何輸入參數。 它們是在作業完成「之後」  執行，而且可以存取傳送給用戶端的回應訊息。   

下列範例說明起作用的後續觸發程序：
```javascript
var updateMetadataTrigger = {
    id: "updateMetadata",
    serverScript: function updateMetadata() {
        var context = getContext();
        var collection = context.getCollection();
        var response = context.getResponse();

        // document that was created
        var createdDocument = response.getBody();

        // query for metadata document
        var filterQuery = 'SELECT * FROM root r WHERE r.id = "_metadata"';
        var accept = collection.queryDocuments(collection.getSelfLink(), filterQuery,
            updateMetadataCallback);
        if(!accept) throw "Unable to update metadata, abort";

        function updateMetadataCallback(err, documents, responseOptions) {
            if(err) throw new Error("Error" + err.message);
                     if(documents.length != 1) throw 'Unable to find metadata document';

                     var metadataDocument = documents[0];

                     // update metadata
                     metadataDocument.createdDocuments += 1;
                     metadataDocument.createdNames += " " + createdDocument.id;
                     var accept = collection.replaceDocument(metadataDocument._self,
                           metadataDocument, function(err, docReplaced) {
                                  if(err) throw "Unable to update metadata, abort";
                           });
                     if(!accept) throw "Unable to update metadata, abort";
                     return;                    
        }                                                                                            
    },
    triggerType: TriggerType.Post,
    triggerOperation: TriggerOperation.All
}

```
觸發程序可以如下列範例所示進行註冊。
```javascript
// register post-trigger
client.createTriggerAsync('dbs/testdb/colls/testColl', updateMetadataTrigger)
    .then(function(createdTrigger) { 
        var docToCreate = { 
            name: "artist_profile_1023",
            artist: "The Band",
            albums: ["Hellujah", "Rotators", "Spinning Top"]
        };

        // run trigger while creating above document 
        var options = { postTriggerInclude: "updateMetadata" };

        return client.createDocumentAsync(collection.self,
              docToCreate, options);
    }, function(error) {
        console.log("Error" , error);
    })
.then(function(response) {
    console.log(response.resource); 
}, function(error) {
    console.log("Error" , error);
});
```

此觸發程序會查詢中繼資料文件，並使用新建立之文件的詳細資料加以更新。  

有一點務必要注意，那就是在 Cosmos DB 中觸發程序的「交易式」執行。 此後續觸發程序會在與建立原始文件時的相同交易過程中執行。 因此，如果從後續觸發程序擲出例外狀況 (例如，如果您無法更新中繼資料文件的話)，則整個交易會失敗並予以回復。 此時不會建立任何文件，並且會傳回例外狀況。  

## <a id="udf"></a>使用者定義函數
使用者定義的函式 (UDF) 可用來擴充 Azure Cosmos DB SQL 查詢語言文法及實作自訂商務邏輯。 UDF 只能從內部查詢進行呼叫。 它們無法存取內容物件，只能做為計算用途的 JavaScript。 因此，UDF 可以在 Cosmos DB 服務的次要複本上執行。  

下列範例會建立 UDF，根據各種收入級距的稅率計算所得稅，然後在查詢內使用它來尋找繳稅超過 $20,000 的所有人員。

```javascript
var taxUdf = {
    id: "tax",
    serverScript: function tax(income) {

        if(income == undefined) 
            throw 'no input';

        if (income < 1000) 
            return income * 0.1;
        else if (income < 10000) 
            return income * 0.2;
        else
            return income * 0.4;
    }
}
```

此 UDF 之後可以用於如下列範例所示的查詢中：

```javascript
// register UDF
client.createUserDefinedFunctionAsync('dbs/testdb/colls/testColl', taxUdf)
    .then(function(response) { 
        console.log("Created", response.resource);

        var query = 'SELECT * FROM TaxPayers t WHERE udf.tax(t.income) > 20000'; 
        return client.queryDocuments('dbs/testdb/colls/testColl',
               query).toArrayAsync();
    }, function(error) {
        console.log("Error" , error);
    })
.then(function(response) {
    var documents = response.feed;
    console.log(response.resource); 
}, function(error) {
    console.log("Error" , error);
});
```

## <a name="javascript-language-integrated-query-api"></a>JavaScript Language Integrated Query API
除了使用 Azure Cosmos DB 的 SQL 文法發出查詢，伺服器端 SDK 可讓您使用流暢的 JavaScript 介面執行最佳化查詢，不需具備任何 SQL 的知識。 JavaScript 的查詢 API 使用 ECMAScript5 陣列內建和受歡迎的 JavaScript 程式庫如 lodash 所熟悉的語法，將述詞函式傳遞至可鏈結式函式呼叫，藉此以程式設計方式建立查詢。 JavaScript 執行階段會剖析查詢，以使用 Azure Cosmos DB 的索引有效地執行該查詢。

> [!NOTE]
> `__` (雙底線) 是 `getContext().getCollection()` 的別名。
> <br/>
> 換句話說，您可以使用 `__` 或 `getContext().getCollection()` 來存取 JavaScript 查詢 API。
> 
> 

支援的函式包括︰

<ul>
<li>
<b>chain() ... .value([callback] [, options])</b>
<ul>
<li>
啟動鏈結的呼叫，此呼叫必須以 value() 終止。
</li>
</ul>
</li>
<li>
<b>filter(predicateFunction [, options] [, callback])</b>
<ul>
<li>
使用述詞函式來篩選輸入，此函式會傳回 true/false 以在結果集內篩出/篩除輸入文件。 此函式的行為類似於 SQL 中的 WHERE 子句。
</li>
</ul>
</li>
<li>
<b>map(transformationFunction [, options] [, callback])</b>
<ul>
<li>
套用投射，其中提供了一個會將每個輸入項目對應至 JavaScript 物件或值的轉換函式。 此函式的行為類似於 SQL 中的 SELECT 子句。
</li>
</ul>
</li>
<li>
<b>pluck([propertyName] [, options] [, callback])</b>
<ul>
<li>
此函式是一個對應的捷徑，此對應會從每個輸入項目擷取單一屬性值。
</li>
</ul>
</li>
<li>
<b>flatten([isShallow] [, options] [, callback])</b>
<ul>
<li>
合併並且壓平每個輸入項目的陣列成為單一陣列。 此函式的行為類似於 LINQ 中的 SelectMany。
</li>
</ul>
</li>
<li>
<b>sortBy([predicate] [, options] [, callback])</b>
<ul>
<li>
使用指定述詞以遞增順序排序輸入文件串流中的文件，產生一組新的文件。 此函式的行為類似於 SQL 中的 ORDER BY 子句。
</li>
</ul>
</li>
<li>
<b>sortByDescending([predicate] [, options] [, callback])</b>
<ul>
<li>
使用指定述詞以遞減順序排序輸入文件串流中的文件，產生一組新的文件。 此函式的行為類似於 SQL 中的 ORDER BY x DESC 子句。
</li>
</ul>
</li>
</ul>


當裡面包含述詞和/或選取器函式時，下列 JavaScript 建構會自動取得最佳化，以便直接在 Azure Cosmos DB 索引上執行：

* 簡單運算子：= + - * / % | ^ &amp; == != === !=== &lt; &gt; &lt;= &gt;= || &amp;&amp; &lt;&lt; &gt;&gt; &gt;&gt;&gt;! ~
* 常值，包含物件常值：{}
* var、return

下列 JavaScript 建構不會取得 Azure Cosmos DB 索引的最佳化：

* 控制流程 (例如 if、for、while)
* 函式呼叫

如需詳細資訊，請參閱 [伺服器端 JSDocs](https://azure.github.io/azure-cosmosdb-js-server/) \(英文\)。

### <a name="example-write-a-stored-procedure-using-the-javascript-query-api"></a>範例：使用 JavaScript 查詢 API 撰寫預存程序
下列程式碼範例示範 JavaScript 查詢 API 如何運用於預存程序的背景下。 預存程序會依照指定的輸入參數插入文件，然後使用 `__.filter()` 方法根據輸入文件的大小屬性，以 minSize、maxSize 和 totalSize 來更新中繼資料文件。

```javascript
/**
 * Insert actual doc and update metadata doc: minSize, maxSize, totalSize based on doc.size.
 */
function insertDocumentAndUpdateMetadata(doc) {
  // HTTP error codes sent to our callback funciton by DocDB server.
  var ErrorCode = {
    RETRY_WITH: 449,
  }

  var isAccepted = __.createDocument(__.getSelfLink(), doc, {}, function(err, doc, options) {
    if (err) throw err;

    // Check the doc (ignore docs with invalid/zero size and metaDoc itself) and call updateMetadata.
    if (!doc.isMetadata && doc.size > 0) {
      // Get the meta document. We keep it in the same collection. it's the only doc that has .isMetadata = true.
      var result = __.filter(function(x) {
        return x.isMetadata === true
      }, function(err, feed, options) {
        if (err) throw err;

        // We assume that metadata doc was pre-created and must exist when this script is called.
        if (!feed || !feed.length) throw new Error("Failed to find the metadata document.");

        // The metadata document.
        var metaDoc = feed[0];

        // Update metaDoc.minSize:
        // for 1st document use doc.Size, for all the rest see if it's less than last min.
        if (metaDoc.minSize == 0) metaDoc.minSize = doc.size;
        else metaDoc.minSize = Math.min(metaDoc.minSize, doc.size);

        // Update metaDoc.maxSize.
        metaDoc.maxSize = Math.max(metaDoc.maxSize, doc.size);

        // Update metaDoc.totalSize.
        metaDoc.totalSize += doc.size;

        // Update/replace the metadata document in the store.
        var isAccepted = __.replaceDocument(metaDoc._self, metaDoc, function(err) {
          if (err) throw err;
          // Note: in case concurrent updates causes conflict with ErrorCode.RETRY_WITH, we can't read the meta again 
          //       and update again because due to Snapshot isolation we will read same exact version (we are in same transaction).
          //       We have to take care of that on the client side.
        });
        if (!isAccepted) throw new Error("replaceDocument(metaDoc) returned false.");
      });
      if (!result.isAccepted) throw new Error("filter for metaDoc returned false.");
    }
  });
  if (!isAccepted) throw new Error("createDocument(actual doc) returned false.");
}
```

## <a name="sql-to-javascript-cheat-sheet"></a>SQL 到 Javascript 的速查表
下列表格包含各種不同的 SQL 查詢和相對應的 JavaScript 查詢。

與 SQL 查詢相同，文件屬性索引鍵 (例如 `doc.id`) 會區分大小寫。

|SQL| JavaScript 查詢 API|下列說明|
|---|---|---|
|SELECT *<br>FROM docs| __.map(function(doc) { <br>&nbsp;&nbsp;&nbsp;&nbsp;return doc;<br>});|1|
|SELECT docs.id, docs.message AS msg, docs.actions <br>FROM docs|__.map(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;id: doc.id,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;msg: doc.message,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;actions:doc.actions<br>&nbsp;&nbsp;&nbsp;&nbsp;};<br>});|2|
|SELECT *<br>FROM docs<br>WHERE docs.id="X998_Y998"|__.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return doc.id ==="X998_Y998";<br>});|3|
|SELECT *<br>FROM docs<br>WHERE ARRAY_CONTAINS(docs.Tags, 123)|__.filter(function(x) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return x.Tags && x.Tags.indexOf(123) > -1;<br>});|4|
|SELECT docs.id, docs.message AS msg<br>FROM docs<br>WHERE docs.id="X998_Y998"|__.chain()<br>&nbsp;&nbsp;&nbsp;&nbsp;.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return doc.id ==="X998_Y998";<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.map(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;id: doc.id,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;msg: doc.message<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>.value();|5|
|SELECT VALUE tag<br>FROM docs<br>JOIN tag IN docs.Tags<br>ORDER BY docs._ts|__.chain()<br>&nbsp;&nbsp;&nbsp;&nbsp;.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return doc.Tags && Array.isArray(doc.Tags);<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.sortBy(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return doc._ts;<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.pluck("Tags")<br>&nbsp;&nbsp;&nbsp;&nbsp;.flatten()<br>&nbsp;&nbsp;&nbsp;&nbsp;.value()|6|

以下說明解說上表中的每個查詢。
1. 所有文件中的結果 (使用連續權杖分頁)。
2. 投射識別碼、訊息 (別名為 msg)，和所有文件中的動作。
3. 使用 predicate: id = "X998_Y998" 查詢文件。
4. 查詢具有 Tags 屬性的文件，且 Tags 是包含值 123 的陣列。
5. 使用 predicate, id = "X998_Y998" 查詢文件，然後投射識別碼和訊息 (別名為 msg)。
6. 篩選具有陣列屬性 Tags 的文件，並且依據 _ts 時間戳記系統屬性來排序結果文件，然後投射及壓平合併 Tags 陣列。


## <a name="runtime-support"></a>執行階段支援
Azure Cosmos DB [JavaScript 伺服器端 API](https://azure.github.io/azure-cosmosdb-js-server/) 支援以 [ECMA-262](http://www.ecma-international.org/publications/standards/Ecma-262.htm) 作為標準的大部分主流 JavaScript 語言功能。

### <a name="security"></a>安全性
JavaScript 預存程序和觸發程序是在沙箱中執行，除非通過資料庫層級的快照交易隔離機制，否則某個指令碼的效果不會傳遞到另一個指令碼。 每次執行之後，都會對執行階段環境進行集區化處理，但會清除內容。 因此，環境與環境彼此之間絕對不會有任何未預期的副作用。

### <a name="pre-compilation"></a>預先編譯
預存程序、觸發程序和 UDF 會隱含地預先編譯成位元組程式碼格式，以避免在每次叫用指令碼時產生編譯成本。 預先編譯可確保能夠快速叫用預存程序，且只需耗費少量資源。

## <a name="client-sdk-support"></a>用戶端 SDK 支援
除了 Azure Cosmos DB [Node.js](sql-api-sdk-node.md) API 之外，Azure Cosmos DB 也具有適用於 SQL API 的 [.NET](sql-api-sdk-dotnet.md)、[.NET Core](sql-api-sdk-dotnet-core.md)、[Java](sql-api-sdk-java.md)、[JavaScript](sql-api-sdk-node.md) 和 [Python SDK](sql-api-sdk-python.md)。 您也可以使用上述任何 SDK 來建立和執行預存程序、觸發程序和 UDF。 下列範例說明如何使用 .NET 用戶端建立和執行預存程序。 請注意 .NET 類型是如何在預存程序中以 JSON 形式傳入及讀回。

```javascript
var markAntiquesSproc = new StoredProcedure
{
    Id = "ValidateDocumentAge",
    Body = @"
            function(docToCreate, antiqueYear) {
                var collection = getContext().getCollection();    
                var response = getContext().getResponse();    

                if(docToCreate.Year != undefined && docToCreate.Year < antiqueYear){
                    docToCreate.antique = true;
                }

                collection.createDocument(collection.getSelfLink(), docToCreate, {}, 
                    function(err, docCreated, options) { 
                        if(err) throw new Error('Error while creating document: ' + err.message);                              
                        if(options.maxCollectionSizeInMb == 0) throw 'max collection size not found'; 
                        response.setBody(docCreated);
                });
         }"
};

// register stored procedure
StoredProcedure createdStoredProcedure = await client.CreateStoredProcedureAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"), markAntiquesSproc);
dynamic document = new Document() { Id = "Borges_112" };
document.Title = "Aleph";
document.Year = 1949;

// execute stored procedure
Document createdDocument = await client.ExecuteStoredProcedureAsync<Document>(UriFactory.CreateStoredProcedureUri("db", "coll", "ValidateDocumentAge"), document, 1920);
```

此範例示範如何使用 [SQL .NET API](/dotnet/api/overview/azure/cosmosdb?view=azure-dotnet) 來建立預先觸發程序，以及建立已啟用觸發程序的文件。 

```javascript
Trigger preTrigger = new Trigger()
{
    Id = "CapitalizeName",
    Body = @"function() {
        var item = getContext().getRequest().getBody();
        item.id = item.id.toUpperCase();
        getContext().getRequest().setBody(item);
    }",
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Pre
};

Document createdItem = await client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"), new Document { Id = "documentdb" },
    new RequestOptions
    {
        PreTriggerInclude = new List<string> { "CapitalizeName" },
    });
```

下列範例則說明如何建立使用者定義函式 (UDF) 並將它用於 [SQL 查詢](sql-api-sql-query.md)中。

```javascript
UserDefinedFunction function = new UserDefinedFunction()
{
    Id = "LOWER",
    Body = @"function(input) 
    {
        return input.toLowerCase();
    }"
};

foreach (Book book in client.CreateDocumentQuery(UriFactory.CreateDocumentCollectionUri("db", "coll"),
    "SELECT * FROM Books b WHERE udf.LOWER(b.Title) = 'war and peace'"))
{
    Console.WriteLine("Read {0} from query", book);
}
```

## <a name="rest-api"></a>REST API
所有 Azure Cosmos DB 作業都可以用 RESTful 方式來執行。 您可以使用 HTTP POST，將預存程序、觸發程序及使用者定義函式註冊在某個集合下。 下列範例說明如何註冊預存程序：

    POST https://<url>/sprocs/ HTTP/1.1
    authorization: <<auth>>
    x-ms-date: Thu, 07 Aug 2014 03:43:10 GMT


    var x = {
      "name": "createAndAddProperty",
      "body": function (docToCreate, addedPropertyName, addedPropertyValue) {
                var collectionManager = getContext().getCollection();
                collectionManager.createDocument(
                    collectionManager.getSelfLink(),
                    docToCreate,
                    function(err, docCreated) {
                      if(err) throw new Error('Error:  ' + err.message);
                      docCreated[addedPropertyName] = addedPropertyValue;
                      getContext().getResponse().setBody(docCreated);
                    });
            }
    }


預存程序的註冊方式是使用包含要建立的預存程序的本文，對 URI dbs/testdb/colls/testColl/sprocs 執行 POST 要求。 觸發程序和 UDF可以透過分別發出 /triggers 和 /udfs 的 POST 要求，以類似方式進行註冊。
您可以接著對其資源連結發出 POST 要求來執行這個預存程序：

    POST https://<url>/sprocs/<sproc> HTTP/1.1
    authorization: <<auth>>
    x-ms-date: Thu, 07 Aug 2014 03:43:20 GMT


    [ { "name": "TestDocument", "book": "Autumn of the Patriarch"}, "Price", 200 ]


在這裡，預存程序的輸入會傳入要求本文中。 輸入會以 JSON 輸入參數陣列的形式傳遞。 預存程序會將第一個輸入做為文件 (即回應本文)。 您收到的回應如下：

    HTTP/1.1 200 OK

    { 
      name: 'TestDocument',
      book: 'Autumn of the Patriarch',
      id: 'V7tQANV3rAkDAAAAAAAAAA==',
      ts: 1407830727,
      self: 'dbs/V7tQAA==/colls/V7tQANV3rAk=/docs/V7tQANV3rAkDAAAAAAAAAA==/',
      etag: '6c006596-0000-0000-0000-53e9cac70000',
      attachments: 'attachments/',
      Price: 200
    }


觸發程序與預存程序不同，無法直接執行， 而是在對文件執行作業時執行。 您可以使用 HTTP 標頭指定要與要求搭配執行的觸發程序。 下列程式碼示範建立文件的要求。

    POST https://<url>/docs/ HTTP/1.1
    authorization: <<auth>>
    x-ms-date: Thu, 07 Aug 2014 03:43:10 GMT
    x-ms-documentdb-pre-trigger-include: validateDocumentContents 
    x-ms-documentdb-post-trigger-include: bookCreationPostTrigger

    {
       "name": "newDocument",
       "title": "The Wizard of Oz",
       "author": "Frank Baum",
       "pages": 92
    }


在這裏，要與要求搭配執行的預先觸發程序指定於 x-ms-documentdb-pre-trigger-include 標頭中。 相對應地，後續觸發程序則是指定於 x-ms-documentdb-post-trigger-include 標頭中。 您可以為指定的要求指定預先和後續觸發程序。

## <a name="sample-code"></a>範例程式碼
您可以在 [GitHub 存放庫](https://github.com/Azure/azure-cosmosdb-js-server/tree/master/samples)中找到更多的伺服器端程式碼範例 (包括 [bulk-delete](https://github.com/Azure/azure-cosmosdb-js-server/blob/master/samples/stored-procedures/bulkDelete.js) 和 [update](https://github.com/Azure/azure-cosmosdb-js-server/blob/master/samples/stored-procedures/update.js))。

要共用您絕佳的預存程序嗎？請參與存放庫的製作，並建立提取要求！ 

## <a name="next-steps"></a>後續步驟
一旦您建立一或多個預存程序、觸發程序和使用者定義函數，您可以使用資料總管在 Azure 入口網站中載入並檢視這些項目。

您還可以在您的路徑中找到下列有用的參考和資源，以深入了解 Azure Cosmos DB 伺服器端程式設計：

* [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md)
* [DocumentDB Studio](https://github.com/mingaliu/DocumentDBStudio/releases)
* [JSON](http://www.json.org/) 
* [JavaScript ECMA-262](http://www.ecma-international.org/publications/standards/Ecma-262.htm)
* [安全和可攜式資料庫擴充性](http://dl.acm.org/citation.cfm?id=276339) 
* [服務導向資料庫架構](http://dl.acm.org/citation.cfm?id=1066267&coll=Portal&dl=GUIDE) 
* [在 Microsoft SQL Server 中託管 .NET 執行階段](http://dl.acm.org/citation.cfm?id=1007669)
