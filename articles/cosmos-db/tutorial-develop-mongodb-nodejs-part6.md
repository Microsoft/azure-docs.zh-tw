---
title: 適用於 Azure 的 MongoDB、Angular 及 Node 教學課程 - 第 6 部分 | Microsoft Docs
description: 本教學課程系列的第 6 部分，有關使用您用於 MongoDB 的完全相同 API，以 Azure Cosmos DB 上的 Angular 和 Node 建立 MongoDB 應用程式
services: cosmos-db
author: johnpapa
manager: kfile
editor: ''
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 06/17/2018
ms.author: jopapa
ms.custom: mvc
ms.openlocfilehash: a1705913e1656901d0a87a3cebb2eb69a6c7ad63
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43698582"
---
# <a name="create-a-mongodb-app-with-angular-and-azure-cosmos-db---part-6-add-post-put-and-delete-functions-to-the-app"></a>使用 Angular 和 Azure Cosmos DB 建立 MongoDB 應用程式 - 第 6 部分：將 Post、Put 和 Delete 函式新增至應用程式

此多部分的教學課程示範如何使用 Express 和 Angular，建立以 Node.js 撰寫的新 [MongoDB API](mongodb-introduction.md) 應用程式，然後將它連線至您的 Azure Cosmos DB 資料庫。

本教學課程的第 6 部分是以[第 5 部分](tutorial-develop-mongodb-nodejs-part5.md)為基礎並涵蓋下列工作：

> [!div class="checklist"]
> * 針對 Hero 服務新增 Post、Put 和 Delete 函式
> * 執行應用程式

> [!VIDEO https://www.youtube.com/embed/Y5mdAlFGZjc]

## <a name="prerequisites"></a>必要條件

開始本教學課程的這個部分之前，請確定您已完成本教學課程[第 5 部分](tutorial-develop-mongodb-nodejs-part5.md)中的步驟。

> [!TIP]
> 本教學課程會為您解說逐步建置應用程式的步驟。 如果您需要下載已完成的專案，您可以從 GitHub 上的 [angular-cosmosdb 存放庫](https://github.com/Azure-Samples/angular-cosmosdb)取得已完成的應用程式。

## <a name="add-a-post-function-to-the-hero-service"></a>將 Post 函式新增至 Hero 服務

1. 在 Visual Studio Code 中，按 [分割編輯器] 按鈕![Visual Studio 中的分割編輯器按鈕](./media/tutorial-develop-mongodb-nodejs-part6/split-editor-button.png)，並排開啟 **routes.js** 和 **hero.service.js**。

    您會看到 routes.js 第 7 行正在呼叫 **hero.service.js** 中第 5 行上的 `getHeroes` 函式。  我們需要針對 post、put 和 delete 函式建立此相同配對。 

    ![Visual Studio Code 中的 routes.js 和 hero.service.js](./media/tutorial-develop-mongodb-nodejs-part6/routes-heroservicejs.png)
    
    讓我們開始撰寫 Hero 服務的程式碼。 

2. 將下列程式碼複製到 **hero.service.js** 中的 `getHeroes` 函式之後、`module.exports` 之前。 此程式碼：  
   * 使用 Hero 模型來張貼新 hero。
   * 檢查回應，查看是否有錯誤並傳回狀態值為 500。

   ```javascript
   function postHero(req, res) {
     const originalHero = { uid: req.body.uid, name: req.body.name, saying: req.body.saying };
     const hero = new Hero(originalHero);
     hero.save(error => {
       if (checkServerError(res, error)) return;
       res.status(201).json(hero);
       console.log('Hero created successfully!');
     });
   }

   function checkServerError(res, error) {
     if (error) {
       res.status(500).send(error);
       return error;
     }
   }
   ```

3. 在 **hero.service.js** 中，更新 `module.exports` 以包含新的 `postHero` 函式。 

    ```javascript
    module.exports = {
      getHeroes,
      postHero
    };
    ```

4. 在 **routes.js** 中，在 `get` 路由器之後為 `post` 函式新增路由器。 此路由器會一次張貼一個 Hero。 以這種方式建構路由器檔案，可清楚顯示所有可用的 API 端點並離開的實際工作**hero.service.js**檔案。

    ```javascript
    router.post('/hero', (req, res) => {
      heroService.postHero(req, res);
    });
    ```

5. 執行應用程式，檢查一切是否運作正常。 在 Visual Studio Code 中，儲存所有的變更，按一下左側的 [偵錯] 按鈕![Visual Studio Code 中的偵錯圖示](./media/tutorial-develop-mongodb-nodejs-part6/debug-button.png)，然後按一下 [開始偵錯] 按鈕![Visual Studio Code 中的開始偵錯圖示](./media/tutorial-develop-mongodb-nodejs-part6/start-debugging-button.png)。

6. 現在返回您的網際網路瀏覽器，並且按 F12 (在大部分的機器上) 開啟開發人員工具 [網路] 索引標籤。 瀏覽至 [http://localhost:3000](http://localhost:3000) 以監看透過網路撥打的電話。

    ![Chrome 中的 [網路] 索引標籤會顯示網路活動](./media/tutorial-develop-mongodb-nodejs-part6/add-new-hero.png)

7. 按一下 [新增 Hero] 按鈕來新增 Hero。 輸入識別碼為 "999"、名稱為 "Fred"，且招呼語為 "Hello"，然後按一下 [儲存]。 您應會在 [網路] 索引標籤中看到您已針對新 Hero 傳送 POST 要求。 

    ![Chrome 中的 [網路] 索引標籤會顯示 Get 和 Post 函式的網路活動](./media/tutorial-develop-mongodb-nodejs-part6/post-new-hero.png)

    現在返回，並將 Post 和 Delete 函式新增至應用程式。

## <a name="add-the-put-and-delete-functions"></a>新增 Put 和 Delete 函式

1. 在 **routes.js** 中，在 post 路由器之後新增 `put` 和 `delete` 路由器。

    ```javascript
    router.put('/hero/:uid', (req, res) => {
      heroService.putHero(req, res);
    });

    router.delete('/hero/:uid', (req, res) => {
      heroService.deleteHero(req, res);
    });
    ```

2. 將下列程式碼複製到 **hero.service.js** 中的 `checkServerError` 函式之後。 此程式碼：
   * 建立 `put` 和 `delete` 函式
   * 執行是否找到 Hero 的檢查
   * 執行錯誤處理 

   ```javascript
   function putHero(req, res) {
     const originalHero = {
       uid: parseInt(req.params.uid, 10),
       name: req.body.name,
       saying: req.body.saying
     };
     Hero.findOne({ uid: originalHero.uid }, (error, hero) => {
       if (checkServerError(res, error)) return;
       if (!checkFound(res, hero)) return;

      hero.name = originalHero.name;
       hero.saying = originalHero.saying;
       hero.save(error => {
         if (checkServerError(res, error)) return;
         res.status(200).json(hero);
         console.log('Hero updated successfully!');
       });
     });
   }

   function deleteHero(req, res) {
     const uid = parseInt(req.params.uid, 10);
     Hero.findOneAndRemove({ uid: uid })
       .then(hero => {
         if (!checkFound(res, hero)) return;
         res.status(200).json(hero);
         console.log('Hero deleted successfully!');
       })
       .catch(error => {
         if (checkServerError(res, error)) return;
       });
   }

   function checkFound(res, hero) {
     if (!hero) {
       res.status(404).send('Hero not found.');
       return;
     }
     return hero;
   }
   ```

3. 在 **hero.service.js** 中，匯出新模組：

   ```javascript
    module.exports = {
      getHeroes,
      postHero,
      putHero,
      deleteHero
    };
    ```

4. 我們現已更新程式碼，請按一下 Visual Studio Code 中的 [重新啟動] 按鈕![Visual Studio Code 中的重新啟動按鈕](./media/tutorial-develop-mongodb-nodejs-part6/restart-debugger-button.png)。

5. 重新整理網際網路瀏覽器中的頁面，然後按一下 [新增 Hero] 按鈕。 新增識別碼為 "9"、名稱為 "Starlord" 的 Hero，且招呼語為 "Hi"。 按一下 [儲存] 按鈕以儲存新 Hero。

6. 現在選取 **Starlord** Hero，並將招呼語從 "Hi" 變更為 "Bye"，然後按一下 [儲存] 按鈕。 

    您現在可以在 [網路] 索引標籤中選取識別碼來顯示承載。 您可以在承載中看到招呼語現在已設定為 "Bye"。

    ![顯示承載的 Hero 應用程式和網路索引標籤](./media/tutorial-develop-mongodb-nodejs-part6/put-hero-function.png) 

    您也可以在 UI 中刪除其中一個 Hero，並查看完成刪除作業所需的時間。 試著針對名為 "Fred" 的 Hero 按一下 [刪除] 按鈕。

    ![顯示完成函式所需時間的 Hero 應用程式和網路索引標籤](./media/tutorial-develop-mongodb-nodejs-part6/times.png) 

    如果您重新整理頁面，[網路] 索引標籤會顯示取得 Hero 所需的時間。 雖然時間迅速，但大部分取決於您的資料所在的地理位置，以及在接近使用者的區域中異動複寫資料的能力。 您可以在接下來 (即將發行) 的教學課程中，進一步了解異地複寫。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列作業：

> [!div class="checklist"]
> * 已將 Post、Put 和 Delete 函式新增至應用程式 

稍後返回查看本教學課程系列中的其他影片。

