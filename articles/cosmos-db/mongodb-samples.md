---
title: 使用 MongoDB API 建置 Azure Cosmos DB 應用程式 | Microsoft Docs
description: 使用適用於 MongoDB 的 Azure Cosmos DB API 建立線上資料庫的教學課程。
keywords: mongodb 範例
services: cosmos-db
author: SnehaGunda
manager: kfile
editor: ''
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: sample
ms.date: 03/23/2018
ms.author: sngun
ms.openlocfilehash: 188b192cf9b86a2d28a578bbcec0d6b19a8cc5d0
ms.sourcegitcommit: d1eefa436e434a541e02d938d9cb9fcef4e62604
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/28/2018
ms.locfileid: "37084897"
---
# <a name="build-an-azure-cosmos-db-api-for-mongodb-app-using-nodejs"></a>使用 Node.js 建置 Azure Cosmos DB：適用於 MongoDB 的 API 應用程式
> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [.NET Core](sql-api-dotnetcore-get-started.md)
> * [Java](sql-api-java-get-started.md)
> * [Node.js for MongoDB](mongodb-samples.md)
> * [Node.js](sql-api-nodejs-get-started.md)
>

此範例將示範如何使用 Node.js 建置 Azure Cosmos DB：適用於 MongoDB 的 API 主控台應用程式。

若要使用此範例，您必須︰

* [建立](create-mongodb-dotnet.md#create-account) Azure Cosmos DB：適用於 MongoDB 的 API 帳戶。
* 擷取 MongoDB [連接字串](connect-mongodb-account.md)資訊。

## <a name="create-the-app"></a>建立應用程式

1. 建立 *app.js* 檔案，複製並貼上下列程式碼。

    ```nodejs
    var MongoClient = require('mongodb').MongoClient;
    var assert = require('assert');
    var ObjectId = require('mongodb').ObjectID;
    var url = 'mongodb://<username>:<password>@<endpoint>.documents.azure.com:10255/?ssl=true';

    var insertDocument = function(db, callback) {
    db.collection('families').insertOne( {
            "id": "AndersenFamily",
            "lastName": "Andersen",
            "parents": [
                { "firstName": "Thomas" },
                { "firstName": "Mary Kay" }
            ],
            "children": [
                { "firstName": "John", "gender": "male", "grade": 7 }
            ],
            "pets": [
                { "givenName": "Fluffy" }
            ],
            "address": { "country": "USA", "state": "WA", "city": "Seattle" }
        }, function(err, result) {
        assert.equal(err, null);
        console.log("Inserted a document into the families collection.");
        callback();
    });
    };
    
    var findFamilies = function(db, callback) {
    var cursor =db.collection('families').find( );
    cursor.each(function(err, doc) {
        assert.equal(err, null);
        if (doc != null) {
            console.dir(doc);
        } else {
            callback();
        }
    });
    };
    
    var updateFamilies = function(db, callback) {
    db.collection('families').updateOne(
        { "lastName" : "Andersen" },
        {
            $set: { "pets": [
                { "givenName": "Fluffy" },
                { "givenName": "Rocky"}
            ] },
            $currentDate: { "lastModified": true }
        }, function(err, results) {
        console.log(results);
        callback();
    });
    };
    
    var removeFamilies = function(db, callback) {
    db.collection('families').deleteMany(
        { "lastName": "Andersen" },
        function(err, results) {
            console.log(results);
            callback();
        }
    );
    };
    
    MongoClient.connect(url, function(err, client) {
    assert.equal(null, err);
    var db = client.db('familiesdb');
    insertDocument(db, function() {
        findFamilies(db, function() {
        updateFamilies(db, function() {
            removeFamilies(db, function() {
                client.close();
            });
        });
        });
    });
    });
    ```
    
    **選擇性**：如果您使用 **MongoDB Node.js 2.2 驅動程式**，請取代以下程式碼片段：

    原始：

    ```nodejs
    MongoClient.connect(url, function(err, client) {
    assert.equal(null, err);
    var db = client.db('familiesdb');
    insertDocument(db, function() {
        findFamilies(db, function() {
        updateFamilies(db, function() {
            removeFamilies(db, function() {
                client.close();
            });
        });
        });
    });
    });
    ```
    
    應取代為：

    ```nodejs
    MongoClient.connect(url, function(err, db) {
    assert.equal(null, err);
    insertDocument(db, function() {
        findFamilies(db, function() {
        updateFamilies(db, function() {
            removeFamilies(db, function() {
                db.close();
            });
        });
        });
    });
    });
    ```
    
2. 在 *app.js* 中根據每個帳戶設定 (了解如何尋找您的[連接字串](connect-mongodb-account.md)) 修改下列變數：

    > [!IMPORTANT]
    > **MongoDB Node.js 3.0 驅動程式**要求您為 Cosmos DB 密碼中的特殊字元編碼。 請務必將 '=' 字元編碼為 %3D
    >
    > 範例：密碼 *jm1HbNdLg5zxEuyD86ajvINRFrFCUX0bIWP15ATK3BvSv==* 應編碼為 *jm1HbNdLg5zxEuyD86ajvINRFrFCUX0bIWP15ATK3BvSv%3D%3D*
    >
    > **MongoDB Node.js 2.2 驅動程式**不要求您為 Cosmos DB 密碼中的特殊字元編碼。
    >
    >
   
    ```nodejs
    var url = 'mongodb://<endpoint>:<password>@<endpoint>.documents.azure.com:10255/?ssl=true';
    ```
     
3. 開啟您最愛的終端機，執行 **npm install mongodb --save**，然後使用 **node app.js** 執行您的應用程式

## <a name="next-steps"></a>後續步驟
* 了解如何[使用 MongoChef](mongodb-mongochef.md) 搭配您的 Azure Cosmos DB：適用於 MongoDB 的 API 帳戶。
