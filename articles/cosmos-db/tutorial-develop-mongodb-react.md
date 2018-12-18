---
title: 適用於 Azure 的 MongoDB、React 及 Node.js 教學課程 | Microsoft Docs
description: 了解如何透過此影片式教學課程系列，使用您用於 MongoDB 的完全相同 API，以 Azure Cosmos DB 上的 React 和 Node.js 建立 MongoDB 應用程式。
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
ms.openlocfilehash: 28651f0b9a2c775292b5c9406f676b45fc4e5d14
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43701777"
---
# <a name="create-a-mongodb-app-with-react-and-azure-cosmos-db"></a>使用 React 和 Azure Cosmos DB 建立 MongoDB 應用程式  

此多部分的影片教學課程示範如何建立具有 前端 React 的 Hero 追蹤應用程式。 將 Node 和 Express 用於伺服器的應用程式，會使用 [MongoDB API](mongodb-introduction.md) 連線至 Azure Cosmos DB，然後將 React 前端連線到應用程式的伺服器部分。 本教學課程也會示範如何在 Azure 入口網站中進行 Azure Cosmos DB 的點按式調整，以及如何將應用程式部署到網際網路，讓每個人都可以追蹤他們喜愛的 Hero。 

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 支援 MongoDB 用戶端連線，因此您可以使用 Azure Cosmos DB 取代 MongoDB，但使用您用於 MongoDB 應用程式的相同程式碼會有附加好處，例如簡單雲端部署、調整，以及超快速的讀取和寫入。  

此多部分教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 簡介
> * 安裝專案
> * 使用 React 建置 UI
> * 使用 Azure 入口網站建立 Azure Cosmos DB 帳戶
> * 使用 Mongoose 連線至 Azure Cosmos DB
> * 將 React、Create、Update 和 Delete 作業新增至應用程式

您想要以 Angular 建置此範例應用程式嗎？ 請參閱 [Angular 教學課程影片系列](tutorial-develop-mongodb-nodejs.md)。

## <a name="prerequisites"></a>必要條件
* [Node.js](https://www.nodejs.org)

### <a name="finished-project"></a>已完成的專案
[從 GitHub](https://github.com/Azure-Samples/react-cosmosdb) 取得已完成的應用程式。

## <a name="introduction"></a>簡介 

在這部影片中，Burke Holland 提供 Azure Cosmos DB 的簡介，並逐步解說您在這系列影片中建立的應用程式。 

> [!VIDEO https://www.youtube.com/embed/58IflnJbYJc]

## <a name="project-setup"></a>專案設定

這部影片示範如何在相同專案中設定 Express 和 React。 Burke 會接著提供專案中程式碼的逐步解說。

> [!VIDEO https://www.youtube.com/embed/ytFUPStJJds]

## <a name="build-the-ui"></a>建置 UI

這段影片示範如何使用 React 建立應用程式的使用者介面 (UI)。 

> [!NOTE]
> 這段影片中參考的 CSS 位於 [react-cosmosdb GitHub 存放庫](https://github.com/Azure-Samples/react-cosmosdb/blob/master/src/index.css)中。

> [!VIDEO https://www.youtube.com/embed/SzHzX0fTUUQ]

## <a name="connect-to-azure-cosmos-db"></a>連線至 Azure Cosmos DB

這段影片示範如何在 Azure 入口網站中建立 Azure Cosmos DB 帳戶，安裝 MongoDB 和 Mongoose 套件，然後使用 Azure Cosmos DB 連接字串將應用程式連線到新建立的帳戶。 

> [!VIDEO https://www.youtube.com/embed/0U2jV1thfvs]

## <a name="read-and-create-heroes-in-the-app"></a>在應用程式中讀取和建立 Hero

這部影片示範如何讀取 Hero 和在 Cosmos DB 資料庫中建立 Hero，以及如何使用 Postman 和 React UI 測試這些方法。 

> [!VIDEO https://www.youtube.com/embed/AQK9n_8fsQI] 

## <a name="delete-and-update-heroes-in-the-app"></a>在應用程式中刪除和更新 Hero

這部影片示範如何從應用程式刪除和更新 Hero，以及在 UI 中顯示更新。 

> [!VIDEO https://www.youtube.com/embed/YmaGT7ztTQM] 

## <a name="complete-the-app"></a>完成應用程式

這部影片示範如何完成應用程式，並完成將 UI 連接到後端 API。 

> [!VIDEO https://www.youtube.com/embed/TcSm2ISfTu8]

## <a name="clean-up-resources"></a>清除資源

如果您將不繼續使用此應用程式，請使用下列步驟，在 Azure 入口網站中刪除本教學課程所建立的所有資源。 

1. 從 Azure 入口網站的左側功能表中，按一下 [資源群組]，然後按一下您所建立資源的名稱。 
2. 在資源群組頁面上，按一下 [刪除]，在文字方塊中輸入要刪除之資源的名稱，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用 React、Node、Express 和 Azure Cosmos DB 建立應用程式 
> * 建立 Azure Cosmos DB 帳戶
> * 將應用程式連線至 Azure Cosmos DB 帳戶
> * 使用 Postman 來測試應用程式
> * 執行此應用程式並將 Hero 新增至資料庫

請返回查看本教學課程系列中的其他影片，該影片涵蓋部署應用程式及全域複寫您的資料。

您可以繼續進行下一個教學課程，了解如何將 MongoDB 資料匯入 Azure Cosmos DB中。  

> [!div class="nextstepaction"]
> [將 MongoDB 資料匯入到 Azure Cosmos DB](mongodb-migrate.md)
 
