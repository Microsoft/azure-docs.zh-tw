---
title: 使用 Mongoose 架構搭配 Azure Cosmos DB | Microsoft Docs
description: 了解如何將 Node.js Mongoose 應用程式連線至 Azure Cosmos DB
services: cosmos-db
author: slyons
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 01/08/2018
ms.author: sclyon
ms.openlocfilehash: aa178a24f0c36a1c5fb56b342141b066c150c7c3
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037974"
---
# <a name="azure-cosmos-db-using-the-mongoose-framework-with-azure-cosmos-db"></a>Azure Cosmos DB：使用 Mongoose 架構搭配 Azure Cosmos DB

本教學課程示範在 Azure Cosmos DB 中儲存資料時如何使用 [Mongoose 架構](http://mongoosejs.com/) \(英文\)。 此逐步解說針對 Azure Cosmos DB 使用 MongoDB API。 對於不熟悉的你們而言，Mongoose 是 Node.js 中適用於 MongoDB 的物件模型建構架構，提供簡單、以結構描述為基礎的解決方案來為您的應用程式資料建立模型。

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

[Node.js](https://nodejs.org/) v0.10.29 版或更高版本。

## <a name="create-an-azure-cosmos-db-account"></a>建立 Azure Cosmos DB 帳戶

讓我們來建立 Azure Cosmos DB 帳戶。 如果您已經擁有想要使用的帳戶，就可以直接跳到[設定您的 Node.js 應用程式](#SetupNode)。 如果您是使用「Azure Cosmos DB 模擬器」，請依照 [Azure Cosmos DB 模擬器](local-emulator.md)的步驟來設定模擬器，然後直接跳到[設定您的 Node.js 應用程式](#SetupNode)。

[!INCLUDE [cosmos-db-create-dbaccount-mongodb](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

## <a name="set-up-your-nodejs-application"></a>設定您的 Node.js 應用程式

>[!Note]
> 如果您只想逐步操作範例程式碼，而不是設定應用程式本身，請複製本教學課程使用的[範例](https://github.com/Azure-Samples/Mongoose_CosmosDB)，並在 Azure Cosmos DB 上建置您的 Node.js Mongoose 應用程式。

1. 如果要在您選擇的資料夾中建立 Node.js 應用程式，請在節點命令提示字元中執行下列命令。

    ```npm init```

    回答下列問題，您的專案將會準備就緒。

1. 將新檔案加入到資料夾，並將它命名為 ```index.js```。
1. 使用其中一種 ```npm install``` 選項安裝必要的套件：
    * Mongoose：```npm install mongoose --save```
    * Dotenv (如果您想要從 .env 檔案載入您的密碼)：```npm install dotenv --save```

    >[!Note]
    > ```--save``` 旗標會將相依性加入到 package.json 檔案。

1. 匯入您 index.js 檔案中的相依性。
    ```JavaScript
    var mongoose = require('mongoose');
    var env = require('dotenv').load();    //Use the .env file to load the variables
    ```

1. 將您的 Cosmos DB 連接字串和 Cosmos DB 名稱加入到 ```.env``` 檔案。

    ```JavaScript
    COSMOSDB_CONNSTR={Your MongoDB Connection String Here}
    COSMOSDB_DBNAME={Your DB Name Here}
    ```

1. 將下列程式碼加入到 index.js 結尾，以使用 Mongoose 架構連線到 Azure Cosmos DB。
    ```JavaScript
    mongoose.connect(process.env.COSMOSDB_CONNSTR+process.env.COSMOSDB_DBNAME+"?ssl=true&replicaSet=globaldb"); //Creates a new DB, if it doesn't already exist

    var db = mongoose.connection;
    db.on('error', console.error.bind(console, 'connection error:'));
    db.once('open', function() {
    console.log("Connected to DB");
    });
    ```
    >[!Note]
    > 此處的環境變數是使用 'dotenv' npm 套件以 process.env.{variableName} 載入。

    一旦您連線到 Azure Cosmos DB，就可以在 Mongoose 中立即開始設定物件模型。

## <a name="caveats-to-using-mongoose-with-azure-cosmos-db"></a>使用 Mongoose 搭配 Azure Cosmos DB 的注意事項

對於您建立的每個模型，基本上 Mongoose 會建立新的 MongoDB 集合。 不過，考慮到 Azure Cosmos DB 的按集合計費模型，如果您有多個以稀疏方式填入的物件模型，它可能不是最具成本效益的方式。

本逐步解說涵蓋兩種模型。 首先我們會逐步解說每個集合儲存一種類型的資料。 這是 Mongoose 的實際行為。

Mongoose 也有稱為[鑑別子](http://mongoosejs.com/docs/discriminators.html) \(英文\) 的概念。 鑑別子是結構描述繼承機制。 它們可讓您在相同的底層 MongoDB 集合上建立多個有重疊之結構描述的模型。

您可以將各種資料模型儲存在同一集合中，然後在查詢時使用篩選子句，只提取所需的資料。

### <a name="one-collection-per-object-model"></a>每個物件模型一個集合

預設的 Mongoose 行為是在每次您建立物件模型時建立一個 MongoDB 集合。 本節探討如何使用適用於 Azure Cosmos DB 的 MongoDB 達成這個目的。 當您的物件模型含有大量資料時，這是建議使用於 Azure Cosmos DB 的方法。 這是 Mongoose 的預設操作模型，因此，如果您精通 Mongoose，您可能已很熟悉。

1. 再次開啟您的 ```index.js```。

1. 針對 'Family' 建立結構描述定義。

    ```JavaScript
    const Family = mongoose.model('Family', new mongoose.Schema({
        lastName: String,
        parents: [{
            familyName: String,
            firstName: String,
            gender: String
        }],
        children: [{
            familyName: String,
            firstName: String,
            gender: String,
            grade: Number
        }],
        pets:[{
            givenName: String
        }],
        address: {
            country: String,
            state: String,
            city: String
        }
    }));
    ```

1. 針對 'Family' 建立物件。

    ```JavaScript
    const family = new Family({
        lastName: "Volum",
        parents: [
            { firstName: "Thomas" },
            { firstName: "Mary Kay" }
        ],
        children: [
            { firstName: "Ryan", gender: "male", grade: 8 },
            { firstName: "Patrick", gender: "male", grade: 7 }
        ],
        pets: [
            { givenName: "Blackie" }
        ],
        address: { country: "USA", state: "WA", city: "Seattle" }
    });
    ```

1. 最後，我們將物件儲存到 Azure Cosmos DB。 基本上，這會建立集合。

    ```JavaScript
    family.save((err, saveFamily) => {
        console.log(JSON.stringify(saveFamily));
    });
    ```

1. 現在，讓我們來建立另一個結構描述和物件。 這次，我們針對家庭成員可能感興趣的 'Vacation Destinations' 建立結構描述和物件。
    1. 和上次一樣，我們來建立配置
    ```JavaScript
    const VacationDestinations = mongoose.model('VacationDestinations', new mongoose.Schema({
        name: String,
        country: String
    }));
    ```

    1. 建立範例物件 (您可以將多個物件加入到此結構描述) 並將它儲存。
    ```JavaScript
    const vacaySpot = new VacationDestinations({
        name: "Honolulu",
        country: "USA"
    });

    vacaySpot.save((err, saveVacay) => {
        console.log(JSON.stringify(saveVacay));
    });
    ```

1. 現在，移至 Azure 入口網站，您會發現 Azure Cosmos DB 中建立了兩個集合。

    ![Node.js 教學課程 - Azure 入口網站的螢幕擷取畫面，其中顯示 Azure Cosmos DB 帳戶，並反白顯示多個集合名稱 - 節點資料庫][mutiple-coll]

1. 最後，我們要從 Azure Cosmos DB 讀取資料。 由於我們使用的是預設的 Mongoose 作業模型，讀取作業會和 Mongoose 的其他讀取作業相同。

    ```JavaScript
    Family.find({ 'children.gender' : "male"}, function(err, foundFamily){
        foundFamily.forEach(fam => console.log("Found Family: " + JSON.stringify(fam)));
    });
    ```

### <a name="using-mongoose-discriminators-to-store-data-in-a-single-collection"></a>使用 Mongoose 鑑別子在單一集合中儲存資料

在這種方法中，我們使用 [Mongoose 鑑別子](http://mongoosejs.com/docs/discriminators.html) \(英文\) 以協助最佳化每個 Azure Cosmos DB 集合的成本。 鑑別子可讓您定義區分用的「索引鍵」，它可讓您儲存、區別和篩選不同的物件模型。

此處，我們建立一個基底物件模型、定義區分用的索引鍵，並將 'Family' 和 'VacationDestinations' 做為擴充加入到基底模型。

1. 讓我們設定基底設定，並定義鑑別子索引鍵。

    ```JavaScript
    const baseConfig = {
        discriminatorKey: "_type", //If you've got a lot of different data types, you could also consider setting up a secondary index here.
        collection: "alldata"   //Name of the Common Collection
    };
    ```

1. 接下來，我們定義通用物件模型

    ```JavaScript
    const commonModel = mongoose.model('Common', new mongoose.Schema({}, baseConfig));
    ```

1. 現在，我們定義 'Family' 模型。 請注意，我們使用的是 ```commonModel.discriminator``` 而不是 ```mongoose.model```。 此外，我們也將基底設定加入至 mongoose 結構描述。 因此，此處的 discriminatorKey 是 ```FamilyType```。

    ```JavaScript
    const Family_common = commonModel.discriminator('FamilyType', new     mongoose.Schema({
        lastName: String,
        parents: [{
            familyName: String,
            firstName: String,
            gender: String
        }],
        children: [{
            familyName: String,
            firstName: String,
           gender: String,
            grade: Number
        }],
        pets:[{
            givenName: String
        }],
        address: {
            country: String,
            state: String,
            city: String
        }
    }, baseConfig));
    ```

1. 同樣地，讓我們加入另一個結構描述，這次是針對 'VacationDestinations'。 此處的 DiscriminatorKey 是 ```VacationDestinationsType```。

    ```JavaScript
    const Vacation_common = commonModel.discriminator('VacationDestinationsType', new mongoose.Schema({
        name: String,
        country: String
    }, baseConfig));
    ```

1. 最後，讓我們來建立模型物件，並將它儲存。
    1. 讓我們將物件加入到 'Family' 模型。
    ```JavaScript
    const family_common = new Family_common({
        lastName: "Volum",
        parents: [
            { firstName: "Thomas" },
            { firstName: "Mary Kay" }
        ],
        children: [
            { firstName: "Ryan", gender: "male", grade: 8 },
            { firstName: "Patrick", gender: "male", grade: 7 }
        ],
        pets: [
            { givenName: "Blackie" }
        ],
        address: { country: "USA", state: "WA", city: "Seattle" }
    });

    family_common.save((err, saveFamily) => {
        console.log("Saved: " + JSON.stringify(saveFamily));
    });
    ```

    1. 接下來，我們將物件加入到 'VacationDestinations' 模型，並將它儲存。
    ```JavaScript
    const vacay_common = new Vacation_common({
        name: "Honolulu",
        country: "USA"
    });

    vacay_common.save((err, saveVacay) => {
        console.log("Saved: " + JSON.stringify(saveVacay));
    });
    ```

1. 現在，如果您返回 Azure 入口網站，您會發現您只有一個稱為 ```alldata``` 的集合，其中含有 'Family' 和 'VacationDestinations' 資料。

    ![Node.js 教學課程 - Azure 入口網站的螢幕擷取畫面，其中顯示 Azure Cosmos DB 帳戶，並反白顯示集合名稱 - 節點資料庫][alldata]

1. 此外，請注意，每個物件有另一個稱為 ```__type``` 的屬性，可協助您區分兩個不同的物件模型。

1. 最後，我們要讀取儲存在 Azure Cosmos DB 中的資料。 Mongoose 負責依據模型篩選資料。 因此，讀取資料時，您不需執行任何不同動作。 只需指定您的模型 (在此案例中是 ```Family_common```)，Mongoose 就會針對 'DiscriminatorKey' 處理篩選。

    ```JavaScript
    Family_common.find({ 'children.gender' : "male"}, function(err, foundFamily){
        foundFamily.forEach(fam => console.log("Found Family (using discriminator): " + JSON.stringify(fam)));
    });
    ```

如您所見，使用 Mongoose 鑑別子很容易。 因此，如果您有使用 Mongoose 架構的應用程式，本教學課程是讓您的應用程式在 Azure Cosmos DB 的 MongoDB API 上開始執行的方法，而且不需要太多變更。

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在 [MongoDB 功能和語法的 MongoDB API 支援](mongodb-feature-support.md)中深入了解 Azure Cosmos DB MongoDB API 支援的 MongoDB 作業、運算子、階段、命令和選項。

[alldata]: ./media/mongodb-mongoose/mongo-collections-alldata.png
[mutiple-coll]: ./media/mongodb-mongoose/mongo-mutliple-collections.png
