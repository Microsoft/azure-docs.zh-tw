---
title: 建置適用於 Azure Cosmos DB 的 Node.js Web 應用程式 | Microsoft Docs
description: 這個 Node.js 教學課程會探索如何使用 Microsoft Azure Cosmos DB，從 Azure 網站上裝載的 Node.js Express Web 應用程式來儲存和存取資料。
services: cosmos-db
author: SnehaGunda
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: sngun
ms.openlocfilehash: 82711ea96f6b3f8544a411ed1b6636c8473ed7e9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957341"
---
# <a name="_Toc395783175"></a>使用 JavaScript SDK 建置 Node.js Web 應用程式來管理 Azure Cosmos DB SQL API 資料

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](sql-api-python-application.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

本 Node.js 教學課程說明如何使用 Azure Cosmos DB SQL API 帳戶，來儲存和存取 Azure 網站上所託管的 Node.js 快速應用程式資料。 在本教學課程中，您會建置一個簡單的網頁型應用程式 (待辦事項清單應用程式)，以便能夠建立、擷取和完成工作。 在 Azure Cosmos DB 中，這些工作會儲存為 JSON 文件。 下圖顯示待辦事項應用程式的螢幕擷取畫面：

![本 Node.js 教學課程所建立的「我的待辦事項清單」應用程式螢幕擷取畫面](./media/sql-api-nodejs-application/cosmos-db-node-js-mytodo.png)

本教學課程示範如何使用 Azure 入口網站來建立 Azure Cosmos DB SQL API 帳戶。 然後，您會建置和執行以 Node.js SDK 為基礎的 Web 應用程式，以建立資料庫、容器，並在容器中新增項目。 本教學課程使用 JavaScript SDK 2.0 版。

您也可以從 [GitHub][GitHub] 取得完整範例。 如需有關如何執行應用程式的指示，只需閱讀 [Readme](https://github.com/Azure-Samples/documentdb-node-todo-app/blob/master/README.md) 檔案即可。

## <a name="_Toc395783176"></a>必要條件

在依照本文中的指示進行之前，您應先確定備妥下列項目：

* 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js][Node.js] 6.10 版或更高版本。
* [Express 產生器](http://www.expressjs.com/starter/generator.html) (您可以透過 `npm install express-generator -g` 進行安裝)
* [Git][Git]。

## <a name="_Toc395637761"></a>步驟 1：建立 Azure Cosmos DB 資料庫帳戶
我們將從建立 Azure Cosmos DB 帳戶開始著手。 如果您已經擁有帳戶，或如果您正在使用 Azure Cosmos DB 模擬器來進行本教學課程，可以跳到[步驟 2：建立新的 Node.js 應用程式](#_Toc395783178)。

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [cosmos-db-keys](../../includes/cosmos-db-keys.md)]

## <a name="_Toc395783178"></a>步驟 2 - 建立新的 Node.js 應用程式
現在，我們來了解如何使用 [Express](http://expressjs.com/) 架構來建立基本的 Hello World Node.js 專案。

1. 開啟您喜好的終端機，例如 Node.js 命令提示字元。
2. 瀏覽至您想要在其中儲存新應用程式的目錄。
3. 使用 Express 產生器來產生名為 **todo**的新應用程式。

   ```bash
   express todo
   ```
4. 開啟您的新 **todo** 目錄並安裝相依性。

   ```bash
   cd todo
   npm install
   ```
5. 執行新的應用程式。

   ```bash
   npm start
   ```

6. 您可以檢視新的應用程式，請導覽瀏覽器至 [http://localhost:3000](http://localhost:3000)。
   
    ![了解 Node.js - Hello World 應用程式在瀏覽器視窗中的螢幕擷取畫面](./media/sql-api-nodejs-application/cosmos-db-node-js-express.png)

 在終端機視窗中使用 CTRL+C 停止應用程式，然後按一下 **y** 以終止批次作業。

## <a name="_Toc395783179"></a>步驟 3：安裝必要的模組

**package.json** 檔案是建立在專案根目錄中的其中一個檔案。 這個檔案包含 Node.js 應用程式所需的其他模組清單。 之後，當您將此應用程式部署至 Azure 網站時，此檔案可用來決定 Azure 上需要安裝哪些模組才能支援您的應用程式。 您需要再安裝兩個本教學課程所需的套件。

1. 開啟終端機，透過 npm 安裝 **async** 模組。

   ```bash
   npm install async --save
   ```

2. 透過 npm 安裝 **@azure/cosmos** 模組。 

   ```bash
   npm install @azure/cosmos
   ```

## <a name="_Toc395783180"></a>步驟 4：在節點應用程式中使用 Azure Cosmos DB 服務
您已完成初始安裝和設定，接下來您會撰寫所需的程式碼，以供待辦事項應用程式與 Azure Cosmos DB 通訊。

### <a name="create-the-model"></a>建立模型
1. 在專案目錄的根目錄中，建立一個名為 **models** 的新目錄  

2. 在 **models** 目錄中，建立名為 **taskDao.js** 的新檔案。 此檔案包含必要程式碼，可供建立資料庫、容器，並定義方法以在 Azure Cosmos DB 中讀取、更新、建立和尋找工作。 

3. 將下列程式碼複製到 **taskDao.js** 檔案

   ```nodejs
   // @ts-check
   const CosmosClient = require("@azure/cosmos").CosmosClient;
   const debug = require("debug")("todo:taskDao");
   class TaskDao {
     /**
      * Manages reading, adding, and updating Tasks in Cosmos DB
      * @param {CosmosClient} cosmosClient
      * @param {string} databaseId
      * @param {string} containerId
      */
     constructor(cosmosClient, databaseId, containerId) {
       this.client = cosmosClient;
       this.databaseId = databaseId;
       this.collectionId = containerId;

       this.database = null;
       this.container = null;
     }

     async init() {
       debug("Setting up the database...");
       const dbResponse = await this.client.databases.createIfNotExists({
         id: this.databaseId
       });
       this.database = dbResponse.database;
       debug("Setting up the database...done!");
       debug("Setting up the container...");
       const coResponse = await this.database.containers.createIfNotExists({
         id: this.collectionId
       });
       this.container = coResponse.container;
       debug("Setting up the container...done!");
     }

     async find(querySpec) {
       debug("Querying for items from the database");
       if (!this.container) {
         throw new Error("Collection is not initialized.");
       }
       const { result: results } = await this.container.items
        .query(querySpec)
        .toArray();
      return results;
    }

    async addItem(item) {
      debug("Adding an item to the database");
      item.date = Date.now();
      item.completed = false;
      const { body: doc } = await this.container.items.create(item);
      return doc;
    }

    async updateItem(itemId) {
      debug("Update an item in the database");
      const doc = await this.getItem(itemId);
      doc.completed = true;

      const { body: replaced } = await this.container.item(itemId).replace(doc);
      return replaced;
    }

    async getItem(itemId) {
      debug("Getting an item from the database");
      const { body } = await this.container.item(itemId).read();
      return body;
    }
  }

   module.exports = TaskDao;
   ```
4. 儲存並關閉 **taskDao.js** 檔案。  

### <a name="create-the-controller"></a>建立控制器

1. 在專案的 **routes** 目錄中，建立名為 **tasklist.js** 的新檔案。  

2. 在 **tasklist.js**中加入以下程式碼。 這會載入供 **tasklist.js** 使用的 CosmosClient 和 async 模組。 這也會定義 **TaskList** 類別，此類別會傳遞作為我們稍早定義的 **TaskDao** 物件執行個體：
   
   ```nodejs
   const TaskDao = require("../models/TaskDao");

   class TaskList {
     /**
      * Handles the various APIs for displaying and managing tasks
      * @param {TaskDao} taskDao
     */
    constructor(taskDao) {
    this.taskDao = taskDao;
    }
    async showTasks(req, res) {
      const querySpec = {
        query: "SELECT * FROM root r WHERE r.completed=@completed",
        parameters: [
          {
            name: "@completed",
            value: false
          }
        ]
      };

      const items = await this.taskDao.find(querySpec);
      res.render("index", {
        title: "My ToDo List ",
        tasks: items
      });
    }

    async addTask(req, res) {
      const item = req.body;

      await this.taskDao.addItem(item);
      res.redirect("/");
    }

    async completeTask(req, res) {
      const completedTasks = Object.keys(req.body);
      const tasks = [];

      completedTasks.forEach(task => {
        tasks.push(this.taskDao.updateItem(task));
      });

      await Promise.all(tasks);

      res.redirect("/");
    }
  }

  module.exports = TaskList;
   ```

3. 儲存並關閉 **tasklist.js** 檔案。

### <a name="add-configjs"></a>新增 config.js

1. 在專案目錄的根目錄中，建立名為 **config.js** 的新檔案。 

2. 將下列程式碼新增至 **config.js** 檔案。 此程式碼會定義組態設定和應用程式所需的值。
   
   ```nodejs
   const config = {};

   config.host = process.env.HOST || "[the endpoint URI of your Azure Cosmos DB account]";
   config.authKey =
     process.env.AUTH_KEY || "[the PRIMARY KEY value of your Azure Cosmos DB account";
   config.databaseId = "ToDoList";
   config.containerId = "Items";

   if (config.host.includes("https://localhost:")) {
     console.log("Local environment detected");
     console.log("WARNING: Disabled checking of self-signed certs. Do not have this code in production.");
     process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
     console.log(`Go to http://localhost:${process.env.PORT || '3000'} to try the sample.`);
   }

   module.exports = config;
   ```

3. 在 **config.js** 檔案中，使用 [Microsoft Azure 入口網站](https://portal.azure.com)上 Azure Cosmos DB 帳戶的 [金鑰] 頁面中找到的值，更新 HOST 和 AUTH_KEY 的值。 

4. 儲存並關閉 **config.js** 檔案。

### <a name="modify-appjs"></a>修改 app.js
1. 在專案目錄中，開啟 **app.js** 檔案。 這是稍早建立 Express Web 應用程式時所建立的檔案。  

2. 將下列程式碼新增至 **app.js** 檔案。 此程式碼會定義要使用的組態檔，並繼續讀出此檔案中的值到我們即將使用的變數。 
   
   ```nodejs
   const CosmosClient = require("@azure/cosmos").CosmosClient;
   const config = require("./config");
   const TaskList = require("./routes/tasklist");
   const TaskDao = require("./models/taskDao");

   const express = require("express");
   const path = require("path");
   const logger = require("morgan");
   const cookieParser = require("cookie-parser");
   const bodyParser = require("body-parser");

   const app = express();

   // view engine setup
   app.set("views", path.join(__dirname, "views"));
   app.set("view engine", "jade");

   // uncomment after placing your favicon in /public
   //app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
   app.use(logger("dev"));
   app.use(bodyParser.json());
   app.use(bodyParser.urlencoded({ extended: false }));
   app.use(cookieParser());
   app.use(express.static(path.join(__dirname, "public")));

   //Todo App:
   const cosmosClient = new CosmosClient({
     endpoint: config.host,
     auth: {
       masterKey: config.authKey
     }
   });
   const taskDao = new TaskDao(cosmosClient, config.databaseId, config.containerId);
   const taskList = new TaskList(taskDao);
   taskDao
     .init(err => {
       console.error(err);
     })
     .catch(err => {
       console.error(err);
       console.error("Shutting down because there was an error settinig up the database.");
       process.exit(1);
     });

   app.get("/", (req, res, next) => taskList.showTasks(req, res).catch(next));
   app.post("/addtask", (req, res, next) => taskList.addTask(req, res).catch(next));
   app.post("/completetask", (req, res, next) => taskList.completeTask(req, res).catch(next));
   app.set("view engine", "jade");

   // catch 404 and forward to error handler
   app.use(function(req, res, next) {
     const err = new Error("Not Found");
     err.status = 404;
     next(err);
   });

   // error handler
   app.use(function(err, req, res, next) {
     // set locals, only providing error in development
     res.locals.message = err.message;
     res.locals.error = req.app.get("env") === "development" ? err : {};

     // render the error page
     res.status(err.status || 500);
     res.render("error");
   });

   module.exports = app;
   ```

3. 最後，儲存並關閉 **app.js** 檔案，我們就差不多快完成了。

## <a name="_Toc395783181"></a>步驟 5：建置使用者介面
現在，讓我們將注意力轉到建置使用者介面，以便使用者可以實際與我們的應用程式互動。 我們建立的 Express 應用程式使用 **Jade** 做為檢視引擎。 如需 Jade 的詳細資訊，請參閱 [http://jade-lang.com/](http://jade-lang.com/)。

1. **views** 目錄中的 **layout.jade** 檔是用來作為其他 **.jade** 檔案的全域範本。 在此步驟中，您將修改它以使用 [Twitter Bootstrap](https://github.com/twbs/bootstrap)，這個工具組能夠方便設計美觀的網站。  

2. 開啟在 **views** 資料夾中找到的 **layout.jade** 檔案，並將其中的內容取代為下列內容：

   ```html
   doctype html
   html
     head
       title= title
       link(rel='stylesheet', href='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/css/bootstrap.min.css')
       link(rel='stylesheet', href='/stylesheets/style.css')
     body
       nav.navbar.navbar-inverse.navbar-fixed-top
         div.navbar-header
           a.navbar-brand(href='#') My Tasks
       block content
       script(src='//ajax.aspnetcdn.com/ajax/jQuery/jquery-1.11.2.min.js')
       script(src='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/bootstrap.min.js')
   ```

    這段程式碼實際上會指示 **Jade** 引擎轉譯出我們應用程式的部分 HTML，並建立稱為 **content** 的**區塊**，讓我們能在其中提供內容頁面的配置。

    儲存並關閉此 **layout.jade** 檔案。

3. 現在，開啟 **index.jade** 檔案 (應用程式即將使用的檢視)，並將檔案中的內容取代為下列內容；

   ```html
   extends layout
   block content
        h1 #{title}
        br
        
        form(action="/completetask", method="post")
         table.table.table-striped.table-bordered
            tr
              td Name
              td Category
              td Date
              td Complete
            if (typeof tasks === "undefined")
              tr
                td
            else
              each task in tasks
                tr
                  td #{task.name}
                  td #{task.category}
                  - var date  = new Date(task.date);
                  - var day   = date.getDate();
                  - var month = date.getMonth() + 1;
                  - var year  = date.getFullYear();
                  td #{month + "/" + day + "/" + year}
                  td
                   if(task.completed) 
                    input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
                   else
                    input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
          button.btn.btn-primary(type="submit") Update tasks
        hr
        form.well(action="/addtask", method="post")
          label Item Name:
          input(name="name", type="textbox")
          label Item Category:
          input(name="category", type="textbox")
          br
          button.btn(type="submit") Add item
   ```

這個程式碼會擴充配置，並為我們在前面的 **layout.jade** 檔案中看到的 **content** 預留位置提供內容。
   
在此配置中，我們建立了兩個 HTML 表單。

第一個表單包含資料的表格，以及可讓我們透過張貼到控制器的 **/completeTask** 方法來更新項目的按鈕。
    
第二個表單包含兩個輸入欄位，以及可讓我們透過張貼到控制器的 **/addtask** 方法來建立項目的按鈕。

這應該就是要讓應用程式開始運作所需的所有程式碼。

## <a name="_Toc395783181"></a>步驟 6：在本機執行您的應用程式
1. 若要在本機電腦上測試應用程式，請在終端機中執行 `npm start` 以啟動應用程式，然後重新整理 [http://localhost:3000](http://localhost:3000) 瀏覽器頁面。 此頁面現在看起來應該類似以下影像：
   
    ![[我的待辦事項清單] 應用程式在瀏覽器視窗中的螢幕擷取畫面](./media/sql-api-nodejs-application/cosmos-db-node-js-localhost.png)

    > [!TIP]
    > 如果您收到有關 layout.jade 檔案或 index.jade 檔案縮排的錯誤，請確定這兩個檔案的前兩行靠左對齊 (沒有空格)。 如果前兩行之前有空格，請將空格移除，儲存這兩個檔案，然後重新整理瀏覽器視窗。 

2. 使用 [項目]、[項目名稱] 和 [類別] 欄位來輸入新工作，然後按一下 [新增項目]。 便會使用這些屬性在 Azure Cosmos DB 中建立文件。 
3. 系統應該會更新此頁面，以在 [待辦事項] 清單中顯示新建立的項目。
   
    ![[待辦事項] 清單中包含一個新項目的應用程式螢幕擷取畫面](./media/sql-api-nodejs-application/cosmos-db-node-js-added-task.png)
4. 若要完成工作，您只需勾選 [已完成] 資料行中的核取方塊，然後按一下 [更新工作] 。 這會更新您已建立的文件，並從檢視中加以移除。

5. 若要停止應用程式，請在終端機視窗中按 CTRL+C，然後按一下 **Y** 以終止批次作業。

## <a name="_Toc395783182"></a>步驟 7：將應用程式開發專案部署至 Azure 網站
1. 如果您還沒有這麼做，請為您的 Azure 網站提供一個 Git 儲存機制。 您可以在 [Azure App Service 的本機 Git 部署](../app-service/app-service-deploy-local-git.md) 主題中找到有關如何執行這項操作的指示。
2. 新增您的 Azure 網站做為 Git 遠端。
   
        git remote add azure https://username@your-azure-website.scm.azurewebsites.net:443/your-azure-website.git
3. 透過推送到遠端進行部署。
   
        git push azure master
4. 幾秒後，Git 便會發佈 Web 應用程式並啟動瀏覽器，您可以在瀏覽器中看到您方便好用的應用程式已在 Azure 中執行！

    恭喜！ 您剛剛已使用 Azure Cosmos DB 建置您的第一個 Node.js Express Web 應用程式，並將它發佈至 Azure 網站。

    如果您想要下載或參考本教學課程的完整參考應用程式，可以從 [GitHub][GitHub] 進行下載。

## <a name="_Toc395637775"></a>後續步驟

* 需要使用 Azure Cosmos DB 來執行規模和效能測試嗎？ 請參閱 [Azure Cosmos DB 的效能和規模測試](performance-testing.md)
* 了解如何[監視 Azure Cosmos DB 帳戶](monitor-accounts.md)。
* 在 [Query Playground](https://www.documentdb.com/sql/demo)中，針對範例資料集執行查詢。
* 探索 [Azure Cosmos DB 文件](https://docs.microsoft.com/azure/cosmos-db/)。

[Node.js]: http://nodejs.org/
[Git]: http://git-scm.com/
[GitHub]: https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-todo-app

