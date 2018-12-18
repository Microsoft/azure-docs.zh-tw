---
title: 適用於 Azure 的 MongoDB、Angular 及 Node 教學課程 - 第 5 部分 | Microsoft Docs
description: 本教學課程系列的第 5 部分，有關使用您用於 MongoDB 的完全相同 API，以 Azure Cosmos DB 上的 Angular 和 Node 建立 MongoDB 應用程式。
services: cosmos-db
author: johnpapa
manager: kfile
editor: ''
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 09/05/2017
ms.author: john.papa
ms.custom: mvc
ms.openlocfilehash: 8674e5daee003ea0688ea8702362165ebb8ccfcf
ms.sourcegitcommit: 156364c3363f651509a17d1d61cf8480aaf72d1a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39249201"
---
# <a name="create-a-mongodb-app-with-angular-and-azure-cosmos-db---part-5-use-mongoose-to-connect-to-azure-cosmos-db"></a>使用 Angular 和 Azure Cosmos DB 建立 MongoDB 應用程式 - 第 5 部分：使用 Mongoose 來連線至 Azure Cosmos DB

此多部分的教學課程示範如何使用 Express、Angular 和您的 Azure Cosmos DB 資料庫，建立以 Node.js 撰寫的新 [MongoDB API](mongodb-introduction.md) 應用程式。

本教學課程的第 5 部分是以[第 4 部分](tutorial-develop-mongodb-nodejs-part4.md)為基礎並涵蓋下列工作：

> [!div class="checklist"]
> * 使用 Mongoose 連線至 Azure Cosmos DB
> * 從 Azure Cosmos DB 取得連接字串資訊
> * 建立 Hero 模型
> * 建立 Hero 服務以取得 Hero 資料
> * 在本機執行應用程式

## <a name="video-walkthrough"></a>影片逐步解說

> [!VIDEO https://www.youtube.com/embed/sI5hw6KPPXI]


## <a name="prerequisites"></a>必要條件

開始本教學課程的這個部分之前，請確定您已完成本教學課程[第 4 部分](tutorial-develop-mongodb-nodejs-part4.md)中的步驟。

> [!TIP]
> 本教學課程會為您解說逐步建置應用程式的步驟。 如果您需要下載已完成的專案，您可以從 GitHub 上的 [angular-cosmosdb 存放庫](https://github.com/Azure-Samples/angular-cosmosdb)取得已完成的應用程式。

## <a name="use-mongoose-to-connect-to-azure-cosmos-db"></a>使用 Mongoose 連線至 Azure Cosmos DB

1. 安裝 mongoose npm 模組，這是通常用來 MongoDB 的 API。

    ```bash
    npm i mongoose --save
    ```

2. 立即在 **server** 資料夾中建立新檔案 (稱為**mongo.js**)。 在此檔案中，新增 Azure Cosmos DB 資料庫的所有連線資訊。

3. 將以下程式碼複製到 **mongo.js** 中。 此程式碼：
    * 需要 Mongoose。
    * 覆寫 Mongo 承諾，以使用 ES6/ES2015 內建和以上的基本承諾。
    * 呼叫 env 檔案，該檔案可讓您根據您是在預備、生產或開發環境來設定某些項目。 我們很快就會建立該檔案。
    * 包含 MongoDB 連接字串，該字串將會設定於 env 檔案中。
    * 建立可呼叫 Mongoose 的連線函式。

    ```javascript
    const mongoose = require('mongoose');
    /**
     * Set to Node.js native promises
     * Per http://mongoosejs.com/docs/promises.html
     */
    mongoose.Promise = global.Promise;

    const env = require('./env/environment');

    // eslint-disable-next-line max-len
    const mongoUri = `mongodb://${env.accountName}:${env.key}@${env.accountName}.documents.azure.com:${env.port}/${env.databaseName}?ssl=true`;

    function connect() {
     mongoose.set('debug', true);
     return mongoose.connect(mongoUri, { useMongoClient: true });
    }

    module.exports = {
      connect,
      mongoose
    };
    ```
    
4. 在 [總管] 窗格中，與 **server** 之下建立稱為 **environment** 的資料夾，以及在 **environment** 資料夾中建立稱為 **environment.js** 的新檔案。

5. 從 mongo.js 檔案，我們知道我們必須包含`dbName`、`key` 和 `cosmosPort`，所以將下列程式碼複製到 **environment.js**。

    ```javascript
    // TODO: replace if yours are different
    module.exports = {
      accountName: 'your-cosmosdb-account-name-goes-here',
      databaseName: 'admin', 
      key: 'your-key-goes-here',
      port: 10255
    };
    ```

## <a name="get-the-connection-string-information"></a>取得連接字串資訊

1. 在 **environment.js** 中，將 `port` 的值變更為 10255。 (您可以在 Azure 入口網站中找到您的 Cosmos DB 連接埠)

    ```javascript
    const port = 10255;
    ```

2. 在 **environment.js** 中，將 `accountName` 的值變更為您在[步驟 4](tutorial-develop-mongodb-nodejs-part4.md) 中建立的 Azure Cosmos DB 帳戶名稱。 

3. 在終端機視窗中使用下列的 CLI 命令，擷取 Azure Cosmos DB 帳戶的主索引鍵： 

    ```azure-cli-interactive
    az cosmosdb list-keys --name <cosmosdb-name> -g myResourceGroup
    ```    
    
    * `<cosmosdb-name>` 是您在[步驟 4](tutorial-develop-mongodb-nodejs-part4.md) 中建立的 Azure Cosmos DB 名稱帳戶。

4. 將主索引鍵複製到 environment.js 檔案中作為`key` 值。

    您的應用程式現在具有連線到 Azure Cosmos DB 所需的所有資訊。 在入口網站中也可以擷取這項資訊。 如需詳細資訊，請參閱[取得要自訂的 MongoDB 連接字串](connect-mongodb-account.md#GetCustomConnection)。 入口網站中的「使用者名稱」等同於 environments.js 中的 dbName。 

## <a name="create-a-hero-model"></a>建立 Hero 模型

1.  在 [總管] 窗格中，於 **server** 資料夾之下建立 **hero.model.js** 檔案。

2. 將以下程式碼複製到 **hero.model.js** 中。 此程式碼：
   * 需要 Mongoose。
   * 建立具有識別碼、名稱和招呼語的新結構描述。
   * 使用結構描述建立模型。
   * 匯出磁碟。 
   * 將集合命名為 Heroes (而不是 Heros，這會成為以 Mongoose 複數命名規則為基礎的集合預設名稱)。

   ```javascript
   const mongoose = require('mongoose');

   const Schema = mongoose.Schema;

   const heroSchema = new Schema(
     {
       id: { type: Number, required: true, unique: true },
       name: String,
       saying: String
     },
     {
       collection: 'Heroes'
     }
   );

   const Hero = mongoose.model('Hero', heroSchema);

   module.exports = Hero;
   ```

## <a name="create-a-hero-service"></a>建立 Hero 服務

1.  在 [總管] 窗格中，於 **server** 資料夾之下建立 **hero.service.js** 檔案。

2. 將以下程式碼複製到 **hero.service.js** 中。 此程式碼：
   * 取得您剛建立的模型
   * 連線到資料庫
   * 建立 docquery 變數，其使用 hero.find 方法來定義可傳回所有 Hero 的查詢。
   * 使用具有承諾的 docquery.exec 執行查詢，以取得所有 Hero 的清單，其中的回應狀態為 200。 
   * 如果狀態為 500，則會送回錯誤訊息
   * 因為我們正在使用模組，所以它會取得 hero。 

   ```javascript
   const Hero = require('./hero.model');

   require('./mongo').connect();

   function getHeroes() {
     const docquery = Hero.find({});
     docquery
       .exec()
       .then(heroes => {
         res.status(200).json(heroes);
       })
       .catch(error => {
         res.status(500).send(error);
         return;
       });
   }

   module.exports = {
     getHeroes
   };
   ```

## <a name="add-the-hero-service-to-routesjs"></a>將 Hero 服務新增至 routes.js

1. 在 Visual Studio Code 中，在 **routes.js** 中將可傳送範例 Hero 資料的 `res.send` 函式註解化，並新增一行來呼叫 `heroService.getHeroes` 函式。

    ```javascript
    router.get('/heroes', (req, res) => {
      heroService.getHeroes(req, res);
    //  res.send(200, [
    //      {"id": 10, "name": "Starlord", "saying": "oh yeah"}
    //  ])
    });
    ```

2. **routes.js** 需要 Hero 服務：

    ```javascript
    const heroService = require('./hero.service'); 
    ```

3. 在 **hero.service.js** 中，更新 getHeroes 函式以接受 `req` 和 `res` 參數，如下所示：

    ```javascript
    function getHeroes(req, res) {
    ```

    讓我們花點時間檢閱並在此逐步解說呼叫鏈結。 我們會先進入 `index.js` (其可設定節點伺服器)，並注意這在設定和定義我們的路由。 routes.js 檔案會接著與 Hero 服務交談，並告訴它去取得我們的函式 (例如 getHeroes) 並傳遞要求和回應。 這裡的 hero.service.js 即將抓取模型並連線到 Mongo，然後將會在我們呼叫 getHeroes 時執行它，並傳回回應 200。 然後它會透過鏈結顯現出來。 

## <a name="run-the-app"></a>執行應用程式

1. 現在我們再次執行應用程式。 在 Visual Studio Code 中，儲存所有的變更，按一下左側的 [偵錯] 按鈕![Visual Studio Code 中的偵錯圖示](./media/tutorial-develop-mongodb-nodejs-part5/debug-button.png)，然後按一下 [開始偵錯] 按鈕![Visual Studio Code 中的偵錯圖示](./media/tutorial-develop-mongodb-nodejs-part5/start-debugging-button.png)。

3. 現在翻轉至瀏覽器，開啟開發人員工具和 [網路] 索引標籤，然後瀏覽至 http://localhost:3000 可找到我們的應用程式。

    ![Azure 入口網站中的新 Azure Cosmos DB 帳戶](./media/tutorial-develop-mongodb-nodejs-part5/azure-cosmos-db-heroes-app.png)

   應用程式中尚未儲存 Hero，但在本教學課程的下一個步驟中，我們將會新增 put、push 和 delete 功能，以便從 UI 使用對 Azure Cosmos DB 資料庫的 Mongoose 連線來新增、更新和刪除 Hero。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列作業：

> [!div class="checklist"]
> * 已使用 Mongoose API 將 Hero 應用程式連線到 Azure Cosmos DB 
> * 已將取得 Hero 功能新增至應用程式

您可以繼續進行本教學課程的下一個部分，將 Post、Put 和 Delete 函式新增至應用程式。

> [!div class="nextstepaction"]
> [將 Post、Put 和 Delete 函式新增至應用程式](tutorial-develop-mongodb-nodejs-part6.md)
