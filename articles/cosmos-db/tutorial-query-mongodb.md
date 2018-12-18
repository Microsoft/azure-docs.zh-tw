---
title: Azure Cosmos DB：如何使用 MongoDB API 來進行查詢？ | Microsoft Docs
description: 了解如何使用適用於 Azure Cosmos DB 的 MongoDB API 來進行查詢
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: na
ms.topic: tutorial
ms.date: 03/29/2018
ms.author: sngun
ms.custom: mvc
ms.openlocfilehash: e941eb27e9c55f9a4b0e7af8d4cd8efa0220616b
ms.sourcegitcommit: a5eb246d79a462519775a9705ebf562f0444e4ec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39264455"
---
# <a name="tutorial-query-azure-cosmos-db-by-using-the-mongodb-api"></a>教學課程：使用 MongoDB API 查詢 Azure Cosmos DB

Azure Cosmos DB [適用於 MongoDB 的 API](mongodb-introduction.md) 支援 [MongoDB 殼層查詢](https://docs.mongodb.com/manual/tutorial/query-documents/)。 

本文涵蓋下列工作： 

> [!div class="checklist"]
> * 使用 MongoDB 來查詢資料

您可以開始使用此文件中的範例，並觀看[使用 MongoDB 殼層查詢 Azure Cosmos DB](https://azure.microsoft.com/resources/videos/query-azure-cosmos-db-data-by-using-the-mongodb-shell/)的視訊。

## <a name="sample-document"></a>範例文件

本文中的查詢使用下列範例文件。

```json
{
  "id": "WakefieldFamily",
  "parents": [
      { "familyName": "Wakefield", "givenName": "Robin" },
      { "familyName": "Miller", "givenName": "Ben" }
  ],
  "children": [
      {
        "familyName": "Merriam", 
        "givenName": "Jesse", 
        "gender": "female", "grade": 1,
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
        ]
      },
      { 
        "familyName": "Miller", 
         "givenName": "Lisa", 
         "gender": "female", 
         "grade": 8 }
  ],
  "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
  "creationDate": 1431620462,
  "isRegistered": false
}
```
## <a id="examplequery1"></a> 範例查詢 1 

在提供上述範例家族文件的情況下，下列查詢會傳回識別碼欄位符合 `WakefieldFamily` 的文件。

**查詢**
    
    db.families.find({ id: “WakefieldFamily”})

**結果**

    {
    "_id": "ObjectId(\"58f65e1198f3a12c7090e68c\")",
    "id": "WakefieldFamily",
    "parents": [
      {
        "familyName": "Wakefield",
        "givenName": "Robin"
      },
      {
        "familyName": "Miller",
        "givenName": "Ben"
      }
    ],
    "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
        "pets": [
          { "givenName": "Goofy" },
          { "givenName": "Shadow" }
        ]
      },
      {
        "familyName": "Miller",
        "givenName": "Lisa",
        "gender": "female",
        "grade": 8
      }
    ],
    "address": {
      "state": "NY",
      "county": "Manhattan",
      "city": "NY"
    },
    "creationDate": 1431620462,
    "isRegistered": false
    }

## <a id="examplequery2"></a>範例查詢 2 

下一個查詢會傳回家族中的所有小孩。 

**查詢**
    
    db.families.find( { id: “WakefieldFamily” }, { children: true } )

**結果**

    {
    "_id": "ObjectId("58f65e1198f3a12c7090e68c")",
    "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
        "pets": [
          { "givenName": "Goofy" },
          { "givenName": "Shadow" }
        ]
      },
      {
        "familyName": "Miller",
        "givenName": "Lisa",
        "gender": "female",
        "grade": 8
      }
    ]
    }


## <a id="examplequery3"></a>範例查詢 3 

下一個查詢會傳回已註冊的所有家族。 

**查詢**
    
    db.families.find( { "isRegistered" : true })
**結果** 不會傳回任何文件。 

## <a id="examplequery4"></a>範例查詢 4

下一個查詢會傳回未註冊的所有家族。 

**查詢**
    
    db.families.find( { "isRegistered" : false })
**結果**

     {
    "_id": ObjectId("58f65e1198f3a12c7090e68c"),
    "id": "WakefieldFamily",
    "parents": [{
        "familyName": "Wakefield",
        "givenName": "Robin"
    }, {
        "familyName": "Miller",
        "givenName": "Ben"
    }],
    "children": [{
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
        "pets": [{
            "givenName": "Goofy"
        }, {
            "givenName": "Shadow"
        }]
    }, {
        "familyName": "Miller",
        "givenName": "Lisa",
        "gender": "female",
        "grade": 8
    }],
    "address": {
        "state": "NY",
        "county": "Manhattan",
        "city": "NY"
    },
    "creationDate": 1431620462,
    "isRegistered": false
}

## <a id="examplequery5"></a>範例查詢 5

下一個查詢會傳回未註冊且州別為 NY 的所有家族。 

**查詢**
    
     db.families.find( { "isRegistered" : false, "address.state" : "NY" })

**結果**

     {
    "_id": ObjectId("58f65e1198f3a12c7090e68c"),
    "id": "WakefieldFamily",
    "parents": [{
        "familyName": "Wakefield",
        "givenName": "Robin"
    }, {
        "familyName": "Miller",
        "givenName": "Ben"
    }],
    "children": [{
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
        "pets": [{
            "givenName": "Goofy"
        }, {
            "givenName": "Shadow"
        }]
    }, {
        "familyName": "Miller",
        "givenName": "Lisa",
        "gender": "female",
        "grade": 8
    }],
    "address": {
        "state": "NY",
        "county": "Manhattan",
        "city": "NY"
    },
    "creationDate": 1431620462,
    "isRegistered": false
}


## <a id="examplequery6"></a>範例查詢 6

下一個查詢會傳回小孩年級為 8 的所有家族。

**查詢**
  
     db.families.find( { children : { $elemMatch: { grade : 8 }} } )

**結果**

     {
    "_id": ObjectId("58f65e1198f3a12c7090e68c"),
    "id": "WakefieldFamily",
    "parents": [{
        "familyName": "Wakefield",
        "givenName": "Robin"
    }, {
        "familyName": "Miller",
        "givenName": "Ben"
    }],
    "children": [{
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
        "pets": [{
            "givenName": "Goofy"
        }, {
            "givenName": "Shadow"
        }]
    }, {
        "familyName": "Miller",
        "givenName": "Lisa",
        "gender": "female",
        "grade": 8
    }],
    "address": {
        "state": "NY",
        "county": "Manhattan",
        "city": "NY"
    },
    "creationDate": 1431620462,
    "isRegistered": false
}

## <a id="examplequery7"></a>範例查詢 7

下一個查詢會傳回小孩陣列大小為 3 的所有家族。

**查詢**
  
      db.Family.find( {children: { $size:3} } )

**結果**

因為沒有任何家族有兩個以上的小孩，則不會傳回任何結果。 只有當參數為 2 時，此查詢才會成功並傳回整個文件。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列操作：

> [!div class="checklist"]
> * 了解如何使用 MongoDB 來進行查詢 

您現在可以繼續進行到下一個教學課程，以了解如何全域散發您的資料。

> [!div class="nextstepaction"]
> [全域散發您的資料](tutorial-global-distribution-sql-api.md)

