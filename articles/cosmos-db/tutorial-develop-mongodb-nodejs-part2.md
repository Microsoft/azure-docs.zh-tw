---
title: 適用於 Azure 的 MongoDB、Angular 及 Node 教學課程 - 第 2 部分 | Microsoft Docs
description: 本教學課程系列的第 2 部分，有關使用您用於 MongoDB 的完全相同 API，以 Azure Cosmos DB 上的 Angular 和 Node 建立 MongoDB 應用程式。
services: cosmos-db
author: johnpapa
manager: kfile
editor: ''
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 09/05/2017
ms.author: jopapa
ms.custom: mvc
ms.openlocfilehash: 99fc6c7dc43f66d92b17f25ad0fba52ce8f34413
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "41918065"
---
# <a name="create-a-mongodb-app-with-angular-and-azure-cosmos-db---part-2-create-a-nodejs-express-app-with-the-angular-cli"></a>使用 Angular 和 Azure Cosmos DB 建立 MongoDB 應用程式 - 第 2 部分：使用 Angular CLI 建立 Node.js Expres 應用程式 

此多部分的教學課程示範如何使用 Express、Angular 和您的 Azure Cosmos DB 資料庫，建立以 Node.js 撰寫的新 [MongoDB API](mongodb-introduction.md) 應用程式。

本教學課程的第 2 部分是以[簡介](tutorial-develop-mongodb-nodejs.md)為基礎並涵蓋下列工作：

> [!div class="checklist"]
> * 安裝 Angular CLI 和 TypeScript
> * 使用 Angular 建立新專案
> * 使用 Express 架構建置應用程式
> * 在 Postman 中測試應用程式

## <a name="video-walkthrough"></a>影片逐步解說

> [!VIDEO https://www.youtube.com/embed/lIwJIYcGSUg]

## <a name="prerequisites"></a>必要條件

開始本教學課程的這個部分之前，請確定您已觀看[簡介影片](tutorial-develop-mongodb-nodejs.md)。

本教學課程還需要： 
* [Node.js](https://nodejs.org/) 8.4.0 版或更新版本。
* [Postman](https://www.getpostman.com/)
* [Visual Studio Code](https://code.visualstudio.com/) 或您最愛的程式碼編輯器。

> [!TIP]
> 本教學課程會為您解說逐步建置應用程式的步驟。 如果您需要下載已完成的專案，您可以從 GitHub 上的 [angular-cosmosdb 存放庫](https://github.com/Azure-Samples/angular-cosmosdb)取得已完成的應用程式。

## <a name="install-the-angular-cli-and-typescript"></a>安裝 Angular CLI 和 TypeScript

1. 開啟 Windows 命令提示字元或 Mac 終端機視窗，並安裝 Angular CLI。

    ```bash
    npm install -g @angular/cli
    ```

2. 在提示字元中輸入下列命令以安裝 TypeScript。 

    ```bash
    npm install -g typescript
    ```

## <a name="use-the-angular-cli-to-create-a-new-project"></a>使用 Angular CLI 建立新的專案

1. 在命令提示字元，變更為您要在其中建立新專案的資料夾，然後執行下列命令。 此命令會建立新的資料夾和名為 angular-cosmosdb 的專案，並安裝新應用程式所需的 Angular 元件。 它會使用最小安裝 (--minimal)，並指定專案使用 Sass (具有旗標 --style scss 的類似 CSS 語法)。

    ```bash
    ng new angular-cosmosdb --minimal --style scss
    ```

2. 一旦命令完成，將目錄變更為 src/client 資料夾。

    ```bash
    cd angular-cosmosdb
    ```

3. 然後在 Visual Studio Code 中開啟此資料夾。

    ```bash
    code .
    ```

## <a name="build-the-app-using-the-express-framework"></a>使用 Express 架構建置應用程式

1. 在 Visual Studio Code 的 [總管] 窗格中，以滑鼠右鍵按一下 **src** 資料夾，按一下 [新增資料夾]，並將新資料夾命名為 server。

2. 在 [總管] 窗格中，以滑鼠右鍵按一下 **server** 資料夾中，按一下 [新增檔案]，並將新檔案命名為 index.js。

3. 回到命令提示字元，使用下列命令來安裝本文剖析器。 這可協助我們的應用程式剖析透過 API 傳入的 JSON 資料。

    ```bash
    npm i express body-parser --save
    ```

4. 在 Visual Studio Code 中，將下列程式碼複製到 index.js 檔案中。 此程式碼：
    * 參考 Express
    * 提取本文剖析器以供讀取要求本文中的 JSON 資料
    * 使用稱為 path 的內建功能
    * 設定根變數，讓您更輕鬆地找到程式碼所在的位置
    * 設定連接埠
    * 啟動 Express
    * 告知應用程式如何使用將用來為伺服器提供服務的中介軟體
    * 為 dist 資料夾 (將是靜態內容) 中的所有項目提供服務
    * 為應用程式提供服務，並針對未在伺服器上找到的任何 GET 要求提供 index.html (適用於深層連結)
    * 透過 app.listen 啟動伺服器
    * 使用 arow 函式記錄連接埠保持運作中
    
   ```node
   const express = require('express');
   const bodyParser = require('body-parser');
   const path = require('path');
   const routes = require('./routes');

   const root = './';
   const port = process.env.PORT || '3000';
   const app = express();

   app.use(bodyParser.json());
   app.use(bodyParser.urlencoded({ extended: false }));
   app.use(express.static(path.join(root, 'dist/angular-cosmosdb')));
   app.use('/api', routes);
   app.get('*', (req, res) => {
     res.sendFile('dist/angular-cosmosdb/index.html', {root});
   });

   app.listen(port, () => console.log(`API running on localhost:${port}`));
   ```

5. 在 Visual Studio Code 的 [總管] 窗格中，以滑鼠右鍵按一下 **server** 資料夾，然後按一下 [新增檔案]。 將新檔案命名為 *routes.js*。 

6. 將以下程式碼複製到 **routes.js** 中。 此程式碼：
   * 參考 Express 路由器
   * 取得 Hero
   * 針對已定義的 Hero 送回 JSON

   ```node
   const express = require('express');
   const router = express.Router();

   router.get('/heroes', (req, res) => {
    res.send(200, [
       {"id": 10, "name": "Starlord", "saying": "oh yeah"}
    ])
   });

   module.exports=router;
   ```

7. 儲存所有已修改的檔案。 

8. 在 Visual Studio Code 中，按一下 [偵錯] 按鈕![Visual Studio Code 中的偵錯圖示](./media/tutorial-develop-mongodb-nodejs-part2/debug-button.png)，按一下齒輪按鈕![Visual Studio Code 中的齒輪按鈕](./media/tutorial-develop-mongodb-nodejs-part2/gear-button.png)。 新的 launch.json 檔案隨即在 Visual Studio Code 中開啟。

8. 在 launch.json 檔案的第 11 行，將 `"${workspaceFolder}\\server"` 變更為 `"program": "${workspaceRoot}/src/server/index.js"` 並儲存檔案。

9. 按一下 [開始偵錯] 按鈕![Visual Studio 程式碼中的偵錯圖示](./media/tutorial-develop-mongodb-nodejs-part2/start-debugging-button.png)以執行應用程式。

    應用程式應會執行且沒有任何錯誤。

## <a name="use-postman-to-test-the-app"></a>使用 Postman 來測試應用程式

1. 現在開啟 Postman 並在 GET 方塊中放入 `http://localhost:3000/api/heroes`。 

2. 按一下 [傳送] 按鈕，並從應用程式取得 json 回應。 

    此回應會顯示應用程式已啟動並在本機執行中。 

    ![顯示要求和回應的 Postman](./media/tutorial-develop-mongodb-nodejs-part2/azure-cosmos-db-postman.png)


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列作業：

> [!div class="checklist"]
> * 已使用 Angular CLI 建立 Node.js 專案
> * 已使用 Postman 來測試應用程式

您可以繼續進行本教學課程的下一個部分，以建置 UI。

> [!div class="nextstepaction"]
> [使用 Angular 建置 UI](tutorial-develop-mongodb-nodejs-part3.md)
