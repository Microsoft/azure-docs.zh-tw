---
title: 使用 Node.js 將 MongoDB 應用程式連線至 Azure Cosmos DB | Microsoft Docs
description: 了解如何將現有的 Node.js MongoDB 應用程式連線至 Azure Cosmos DB
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.custom: quick start connect, mvc, devcenter
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 06/19/2017
ms.author: sngun
ms.openlocfilehash: 00824dc7a4fa7589fd01568b82351a68e1d44faa
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46983560"
---
# <a name="azure-cosmos-db-migrate-an-existing-nodejs-mongodb-web-app"></a>Azure Cosmos DB︰移轉現有的 Node.js MongoDB Web 應用程式 

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-golang.md)
>  

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。 

此快速入門示範如何使用以 Node.js 撰寫的現有 MongoDB 應用程式，並將其連線至可使用 [MongoDB API](mongodb-introduction.md) 來支援 MongoDB 用戶端連線的 Azure Cosmos DB 資料庫。 換句話說，您的 Node.js 應用程式只知道它使用 MongoDB API 連線到資料庫。 對於資料儲存在 Azure Cosmos DB 中的應用程式而言是透明的。

完成之後，MEAN 應用程式 (MongoDB、Express、Angular 及 Node.js) 便會在 [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 上執行。 

![在 Azure App Service 中執行的 MEAN.js 應用程式](./media/create-mongodb-nodejs/meanjs-in-azure.png)


[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本主題會要求您執行 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="prerequisites"></a>必要條件 
如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 
[!INCLUDE [cosmos-db-emulator-mongodb](../../includes/cosmos-db-emulator-mongodb.md)]

除了 Azure CLI，您還需要在本機安裝 [Node.js](https://nodejs.org/) 和 [Git](http://www.git-scm.com/downloads)，才能執行 `npm` 和 `git` 命令。

您應具備 Node.js 的使用知識。 本快速入門通常不是為了協助您開發 Node.js 應用程式。

## <a name="clone-the-sample-application"></a>複製範例應用程式

執行下列命令來複製範例存放庫。 此範例存放庫包含預設 [MEAN.js](http://meanjs.org/) 應用程式。

1. 開啟命令提示字元，建立名為 git-samples 的新資料夾，然後關閉命令提示字元。

    ```bash
    md "C:\git-samples"
    ```

2. 開啟 git 終端機視窗 (例如 git bash)，並使用 `cd` 命令變更至要安裝範例應用程式的新資料夾。

    ```bash
    cd "C:\git-samples"
    ```

3. 執行下列命令來複製範例存放庫。 此命令會在您的電腦上建立範例應用程式副本。 

    ```bash
    git clone https://github.com/prashanthmadi/mean
    ```

## <a name="run-the-application"></a>執行應用程式

安裝必要的封裝，然後啟動應用程式。

```bash
cd mean
npm install
npm start
```
應用程式會嘗試連線到 MongoDB 來源且失敗，當輸出傳回 "[MongoError: connect ECONNREFUSED 127.0.0.1:27017]" 時，請繼續結束應用程式。

## <a name="log-in-to-azure"></a>登入 Azure

如果您要使用已安裝的 Azure CLI，請使用 [az login](/cli/azure/reference-index#az-login) 命令來登入 Azure 訂用帳戶，並遵循畫面上的指示進行。 如果您是使用 Azure Cloud Shell，可以跳過此步驟。

```azurecli
az login 
``` 
   
## <a name="add-the-azure-cosmos-db-module"></a>新增 Azure Cosmos DB 模組

如果您是使用已安裝的 Azure CLI，請檢查是否已安裝 `cosmosdb` 元件，方法是執行 `az` 命令。 如果 `cosmosdb` 位於基底命令清單中，請繼續進行下一個命令。 如果您是使用 Azure Cloud Shell，可以跳過此步驟。

如果 `cosmosdb` 不在基底命令的清單中，請重新安裝 [Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](../azure-resource-manager/resource-group-overview.md) 來建立[資源群組](/cli/azure/group#az-group-create)。 Azure 資源群組是在其中部署與管理 Azure 資源 (如 Web 應用程式、資料庫和儲存體帳戶) 的邏輯容器。 

下列範例會在西歐區域中建立一個資源群組。 選擇資源群組的唯一名稱。

如果您是使用 Azure Cloud Shell，請按一下 [試用]、遵循畫面上的提示登入，然後將命令複製到命令提示字元。

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

## <a name="create-an-azure-cosmos-db-account"></a>建立 Azure Cosmos DB 帳戶

使用 [az cosmosdb create](/cli/azure/cosmosdb#az-cosmosdb-create) 命令來建立 Azure Cosmos DB 帳戶。

在下列命令中，請在您看見 `<cosmosdb-name>` 預留位置的地方，替代成您自己的唯一 Azure Cosmos DB 帳戶名稱。 這個唯一名稱會用來作為 Azure Cosmos DB 端點 (`https://<cosmosdb-name>.documents.azure.com/`) 的一部分，所以這個名稱在 Azure 中的所有 Azure Cosmos DB 帳戶上必須是唯一的。 

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

`--kind MongoDB` 參數會啟用 MongoDB 用戶端連接。

建立 Azure Cosmos DB 帳戶之後，Azure CLI 會顯示類似下列範例的資訊。 

> [!NOTE]
> 此範例會使用 JSON 作為 Azure CLI 輸出格式，這是預設值。 若要使用另一種輸出格式，請參閱 [Azure CLI 命令的輸出格式](https://docs.microsoft.com/cli/azure/format-output-azure-cli)。

```json
{
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb-name>.documents.azure.com:443/",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Document
DB/databaseAccounts/<cosmosdb-name>",
  "kind": "MongoDB",
  "location": "West Europe",
  "name": "<cosmosdb-name>",
  "readLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ],
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "writeLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ]
} 
```

## <a name="connect-your-nodejs-application-to-the-database"></a>將 Node.js 應用程式連線到資料庫

在此步驟中，您要使用 MongoDB 連接字串，將 MEAN.js 範例應用程式連線至您剛才建立的 Azure Cosmos DB 資料庫。 

<a name="devconfig"></a>
## <a name="configure-the-connection-string-in-your-nodejs-application"></a>在 Node.js 應用程式中設定連接字串

在您的 MEAN.js 存放庫中，開啟 `config/env/local-development.js`。

以下列程式碼取代此檔案的內容。 務必也要使用您的 Azure Cosmos DB 帳戶名稱來取代這兩個 `<cosmosdb-name>` 預留位置。

```javascript
'use strict';

module.exports = {
  db: {
    uri: 'mongodb://<cosmosdb-name>:<primary_master_key>@<cosmosdb-name>.documents.azure.com:10255/mean-dev?ssl=true&sslverifycertificate=false'
  }
};
```

## <a name="retrieve-the-key"></a>擷取金鑰

若要連線至 Azure Cosmos DB 資料庫，您需要資料庫金鑰。 使用 [az cosmosdb list-keys](/cli/azure/cosmosdb#list-keys) 命令來擷取主要金鑰。

```azurecli-interactive
az cosmosdb list-keys --name <cosmosdb-name> --resource-group myResourceGroup --query "primaryMasterKey"
```

Azure CLI 會輸出類似下列範例的資訊。 

```json
"RUayjYjixJDWG5xTqIiXjC..."
```

複製 `primaryMasterKey` 的值。 將此值貼到 `local-development.js` 中的 `<primary_master_key>` 上。

儲存您的變更。

### <a name="run-the-application-again"></a>再次執行應用程式。

再次執行 `npm start`。 

```bash
npm start
```

主控台訊息現在應告訴您開發環境已啟動並在執行中。 

在瀏覽器中，瀏覽至 `http://localhost:3000` 。 按一下上層功能表中的 [註冊]，然後嘗試建立兩位虛擬使用者。 

MEAN.js 範例應用程式會將使用者資料儲存於資料庫中。 如果您成功且 MEAN.js 自動登入至所建立的使用者，則您的 Cosmos DB 連線正在運作中。 

![MEAN.js 成功連線至 MongoDB](./media/create-mongodb-nodejs/mongodb-connect-success.png)

## <a name="view-data-in-data-explorer"></a>在資料總管中檢視資料

可以在 Azure 入口網站中檢視、查詢 Azure Cosmos DB 所儲存的資料，以及在其上執行商務邏輯。

若要檢視、查詢及處理在前一個步驟中建立的使用者資料，請在 Web 瀏覽器中登入 [Azure 入口網站](https://portal.azure.com)。

在頂端的 [搜尋] 方塊中，輸入 Azure Cosmos DB。 當您的 Cosmos DB 帳戶刀鋒視窗開啟時，選取您的 Cosmos DB 帳戶。 在左側導覽中，按一下 [資料總管]。 在 [集合] 窗格中展開您的集合，然後您可以檢視集合中的文件、查詢資料，甚至是建立及執行預存程序、觸發程序和 UDF。 

![Azure 入口網站中的資料總管](./media/create-mongodb-nodejs/cosmosdb-connect-mongodb-data-explorer.png)


## <a name="deploy-the-nodejs-application-to-azure"></a>將 Node.js 應用程式部署至 Azure

在此步驟中，您要將已與 MongoDB 連線接的 Node.js 應用程式部署至 Azure Cosmos DB。

您可能已注意到，您稍早變更的組態檔適用於開發環境 (`/config/env/local-development.js`)。 當您將應用程式部署至 App Service 時，它會依照預設在生產環境中執行。 所以現在，您需要對個別的組態檔進行相同的變更。

在您的 MEAN.js 存放庫中，開啟 `config/env/production.js`。

在 `db` 物件中，取代 `uri` 的值，如下列範例中所示。 請務必取代預留位置，如同之前一樣。

```javascript
'mongodb://<cosmosdb-name>:<primary_master_key>@<cosmosdb-name>.documents.azure.com:10255/mean?ssl=true&sslverifycertificate=false',
```

> [!NOTE] 
> `ssl=true` 選項很重要，因為 [Azure Cosmos DB 需要 SSL](connect-mongodb-account.md#connection-string-requirements)。 
>
>

在終端機中，將所有的變更認可至 Git。 您可以複製這兩個命令，以便同時執行它們。

```bash
git add .
git commit -m "configured MongoDB connection string"
```
## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何建立 Azure Cosmos DB 帳戶，以及如何使用 [資料總管] 建立 MongoDB 集合。 您現在可以將 MongoDB 資料移轉到 Azure Cosmos DB。  

> [!div class="nextstepaction"]
> [將 MongoDB 資料匯入到 Azure Cosmos DB](mongodb-migrate.md)
