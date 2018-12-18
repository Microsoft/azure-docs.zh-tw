---
title: 適用於 Azure 的 MongoDB、Angular 及 Node 教學課程 - 第 4 部分 | Microsoft Docs
description: 本教學課程系列的第 4 部分，有關使用您用於 MongoDB 的完全相同 API，以 Azure Cosmos DB 上的 Angular 和 Node 建立 MongoDB 應用程式
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
ms.openlocfilehash: b6654afa27255b0ebd0cc80b94212f44bbf16f34
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960062"
---
# <a name="create-a-mongodb-app-with-angular-and-azure-cosmos-db---part-4-create-an-azure-cosmos-db-account-using-the-azure-cli"></a>使用 Angular 和 Azure Cosmos DB 建立 MongoDB 應用程式 - 第 4 部分：使用 Azure CLI 建立 Azure Cosmos DB 帳戶

此多部分的教學課程示範如何使用 Express、Angular 和您的 Azure Cosmos DB 資料庫，建立以 Node.js 撰寫的新 [MongoDB API](mongodb-introduction.md) 應用程式。

本教學課程的第 4 部分是以[第 3 部分](tutorial-develop-mongodb-nodejs-part3.md)為基礎並涵蓋下列工作：

> [!div class="checklist"]
> * 使用 Azure CLI 建立 Azure 資源群組
> * 使用 Azure CLI 建立 Azure Cosmos DB 帳戶

## <a name="video-walkthrough"></a>影片逐步解說

> [!VIDEO https://www.youtube.com/embed/hfUM-AbOh94]

## <a name="prerequisites"></a>必要條件

開始本教學課程的這個部分之前，請確定您已完成本教學課程[第 3 部分](tutorial-develop-mongodb-nodejs-part3.md)中的步驟。 

在本教學課程區段中，您可以使用 Azure Cloud Shell (在您的網際網路瀏覽器中) 或本機上安裝的 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [Log in to Azure](../../includes/login-to-azure.md)]

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group.md)]

> [!TIP]
> 本教學課程會為您解說逐步建置應用程式的步驟。 如果您需要下載已完成的專案，您可以從 GitHub 上的 [angular-cosmosdb 存放庫](https://github.com/Azure-Samples/angular-cosmosdb)取得已完成的應用程式。

## <a name="create-an-azure-cosmos-db-account"></a>建立 Azure Cosmos DB 帳戶

使用 [`az cosmosdb create`](/cli/azure/cosmosdb#az-cosmosdb-create) 命令建立 Azure Cosmos DB 帳戶。

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

* 針對 `<cosmosdb-name>`，使用您自己唯一的 Azure Cosmos DB 帳戶名稱，此名稱在 Azure 的所有 Azure Cosmos DB 帳戶名稱中必須是唯一的。
* `--kind MongoDB` 設定可讓 Azure Cosmos DB 有 MongoDB 用戶端連線。

此命令可能需要一或兩分鐘的時間來完成。 完成時，終端機視窗會顯示新資料庫的相關資訊。 

一旦建立 Azure Cosmos DB 帳戶：
1. 開啟新的瀏覽器視窗並移至 [https://portal.azure.com](https://portal.azure.com)
1. 按一下左列上的 Azure Cosmos DB 標誌， ![Azure 入口網站中的 Azure Cosmos DB 圖示](./media/tutorial-develop-mongodb-nodejs-part4/azure-cosmos-db-icon.png) 即可顯示您擁有的所有 Azure Cosmos DB。
1. 按一下您剛建立的 Azure Cosmos DB 帳戶，請選取 [概觀] 索引標籤並向下捲動以檢視資料庫所在的地圖。 

    ![Azure 入口網站中的新 Azure Cosmos DB 帳戶](./media/tutorial-develop-mongodb-nodejs-part4/azure-cosmos-db-angular-portal.png)

4. 向下捲動左側導覽列並按一下 [全域複寫資料] 索引標籤，這會顯示您可在其中查看可複寫至之不同區域的地圖。 例如，您可以按一下澳洲東南部或澳洲東部，並將您的資料複寫至澳洲。 您可以在[如何使用 Azure Cosmos DB 全域散發資料](distribute-data-globally.md)中深入了解全域複寫。 現在，我們只要保留一個執行個體，而當我們想要複寫時，我們就會知道怎麼做。

    ![Azure 入口網站中的新 Azure Cosmos DB 帳戶](./media/tutorial-develop-mongodb-nodejs-part4/azure-cosmos-db-replicate-portal.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列作業：

> [!div class="checklist"]
> * 已使用 Azure CLI 建立 Azure 資源群組
> * 已使用 Azure CLI 建立 Azure Cosmos DB 帳戶

您可以繼續本教學課程的下一個部分，使用 Mongoose 將 Azure Cosmos DB 連線至您的應用程式。

> [!div class="nextstepaction"]
> [使用 Mongoose 連線至 Azure Cosmos DB](tutorial-develop-mongodb-nodejs-part5.md)
