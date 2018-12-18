---
title: 快速入門：資料表 API 與 Python - Azure Cosmos DB | Microsoft Docs
description: 本快速入門示範如何使用 Azure Cosmos DB 資料表 API，以使用 Azure 入口網站與 Python 建立應用程式
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: python
ms.topic: quickstart
ms.date: 04/10/2018
ms.author: sngun
ms.custom: mvc
ms.openlocfilehash: 614e22cbbb6a94d9b148b29474a5fc82c3f0fef4
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696780"
---
# <a name="quickstart-build-a-table-api-app-with-python-and-azure-cosmos-db"></a>快速入門：使用 Python 與 Azure Cosmos DB 建置資料表 API 應用程式

> [!div class="op_single_selector"]
> * [.NET](create-table-dotnet.md)
> * [Java](create-table-java.md)
> * [Node.js](create-table-nodejs.md)
> * [Python](create-table-python.md)
> 

本快速入門示範如何使用 Python 與 Azure Cosmos DB [資料表 API](table-introduction.md)，以藉由從 GitHub 複製範例來建置應用程式。 本快速入門也會示範如何建立 Azure Cosmos DB 帳戶，以及如何使用資料總管在以 Web 為基礎的 Azure 入口網站中建立資料表和實體。

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值、整個資料行及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。 

## <a name="prerequisites"></a>必要條件

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

此外：

* 如果尚未安裝 Visual Studio 2017，您可以下載並使用**免費的** [Visual Studio 2017 Community 版本](https://www.visualstudio.com/downloads/)。 請務必在 Visual Studio 安裝期間選取 [Azure 開發] 和 [Python 開發] 工作負載。
* 此外也請選取 [Python 開發] 工作負載中的 [Python 2] 選項，或從 [python.org](https://www.python.org/downloads/release/python-2712/) 下載 Python 2.7。

## <a name="create-a-database-account"></a>建立資料庫帳戶

> [!IMPORTANT] 
> 您需要建立新的資料表 API 帳戶，以與正式推出的資料表 API SDK 搭配使用。 正式推出的 SDK 不支援在預覽期間建立的資料表 API 帳戶。
>

[!INCLUDE [cosmos-db-create-dbaccount-table](../../includes/cosmos-db-create-dbaccount-table.md)]

## <a name="add-a-table"></a>新增資料表

[!INCLUDE [cosmos-db-create-table](../../includes/cosmos-db-create-table.md)]

## <a name="add-sample-data"></a>新增範例資料

[!INCLUDE [cosmos-db-create-table-add-sample-data](../../includes/cosmos-db-create-table-add-sample-data.md)]

## <a name="clone-the-sample-application"></a>複製範例應用程式

現在，我們將從 Github 複製「資料表」應用程式、設定連接字串，然後執行它。 您會看到，以程式設計方式來處理資料有多麼的容易。 

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
    git clone https://github.com/Azure-Samples/storage-python-getting-started.git
    ```

3. 然後在 Visual Studio 中開啟方案檔案。 

## <a name="update-your-connection-string"></a>更新您的連接字串

現在，返回 Azure 入口網站以取得連接字串資訊，並將它複製到應用程式中。 這可讓您的應用程式與託管資料庫進行通訊。 

1. 在 [Azure 入口網站](http://portal.azure.com/)中，按一下 [連接字串]。 

    ![在 [連接字串] 窗格中檢視及複製「連接字串」。](./media/create-table-python/connection-string.png)

2. 使用右側的按鈕複製 ACCOUNT NAME。

3. 開啟 config.py 檔案，然後將 ACCOUNT NAME 從入口網站貼到第 19 行上的 STORAGE_ACCOUNT_NAME 值中。

4. 返回入口網站，並複製 PRIMARY KEY。

5. 將 PRIMARY KEY 從入口網站貼到第 20 行上的 STORAGE_ACCOUNT_KEY 值中。

3. 儲存 config.py 檔案。

## <a name="run-the-app"></a>執行應用程式

1. 在 Visual Studio 中，於 [方案總管] 中的專案上按一下滑鼠右鍵，選取目前的 Python 環境，然後按一下滑鼠右鍵。

2. 選取 [安裝 Python 套件]，然後輸入 **azure-storage-table**

3. 按 F5 鍵執行應用程式。 您的應用程式會顯示在瀏覽器中。 

您現在可以返回 [資料總管]，以查看、查詢、修改及使用這項新資料。 

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 入口網站中檢閱 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何建立 Azure Cosmos DB 帳戶、如何使用資料總管來建立資料表，以及如何執行應用程式。  現在，您可以使用資料表 API 來查詢您的資料。  

> [!div class="nextstepaction"]
> [將資料表資料匯入至資料表 API](table-import.md)
