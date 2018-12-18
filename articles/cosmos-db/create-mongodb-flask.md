---
title: Azure Cosmos DB︰使用 Python 和 Azure Cosmos DB MongoDB API 建置 Flask Web 應用程式 | Microsoft Docs
description: 提供 Python Flask 程式碼範例，可讓您用來連線及查詢 Azure Cosmos DB MongoDB API
services: cosmos-db
author: slyons
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.custom: quick start connect, mvc
ms.devlang: python
ms.topic: quickstart
ms.date: 10/2/2017
ms.author: sclyon
ms.openlocfilehash: faba9b2999445152f700ddfa1a7e56983cbb03f4
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43700664"
---
# <a name="azure-cosmos-db-build-a-flask-app-with-the-mongodb-api"></a>Azure Cosmos DB：使用 MongoDB API 建置 Flask 應用程式

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-golang.md)
>  

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。

此快速入門指南會使用下列 [Flask 範例](https://github.com/Azure-Samples/CosmosDB-Flask-Mongo-Sample)，並示範如何使用 [Azure Cosmos DB 模擬器](local-emulator.md)和 Azure Cosmos DB [MongoDB API](mongodb-introduction.md) (而不是 MongoDB) 來建置簡單的待辦事項 Flask 應用程式。

## <a name="prerequisites"></a>必要條件

- 下載 [Azure Cosmos DB 模擬器](local-emulator.md)。 目前只有 Windows 支援此模擬器。 此範例示範如何搭配來自 Azure 的生產金鑰使用範例，這可以在任何平台上進行。

- 如果您還沒有安裝 Visual Studio Code，可以快速安裝適用於您平台 (Windows、Mac、Linux) 的 [VS Code](https://code.visualstudio.com/Download) \(英文\)。

- 請務必安裝其中一個常用的 Python 擴充功能，以新增 Python 語言支援。
    1. 選取擴充功能。
    2. 在命令選擇區 `Ctrl+Shift+P` 中輸入 `ext install` 以安裝擴充功能。

    本文件中的範例使用由 Don Jayamanne 所撰寫之熱門且全功能的 [Python 擴充功能](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python) \(英文\)。

## <a name="clone-the-sample-application"></a>複製範例應用程式

現在讓我們從 GitHub 複製 Flask-MongoDB API 應用程式，設定連接字串，然後執行它。 您會看到，以程式設計方式來處理資料有多麼的容易。

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
    git clone https://github.com/Azure-Samples/CosmosDB-Flask-Mongo-Sample.git
    ```
3. 執行下列命令來安裝 Python 模組。

    ```bash 
    pip install -r .\requirements.txt
    ```
4. 在 Visual Studio Code 中開啟資料夾。

## <a name="review-the-code"></a>檢閱程式碼

此為選用步驟。 若您想要瞭解如何在程式碼中建立資料庫資源，則可檢閱下列程式碼片段。 或是，您可以直接跳到[執行 Web 應用程式](#run-the-web-app)。 

下列程式碼片段皆取自 app.py 檔案，並使用本機 Azure Cosmos DB 模擬器的連接字串。 密碼必須分割 (如下所示)，以容納無法剖析的正斜線。

* 初始化 MongoDB 用戶端，擷取資料庫，然後驗證。

    ```python
    client = MongoClient("mongodb://127.0.0.1:10250/?ssl=true") #host uri
    db = client.test    #Select the database
    db.authenticate(name="localhost",password='C2y6yDjf5' + r'/R' + '+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw' + r'/Jw==')
    ```

* 擷取集合，或是在集合尚未存在的情況下建立它。

    ```python
    todos = db.todo #Select the collection
    ```

* 建立應用程式

    ```Python
    app = Flask(__name__)
    title = "TODO with Flask"
    heading = "ToDo Reminder"
    ```
    
## <a name="run-the-web-app"></a>執行 Web 應用程式

1. 確認 Azure Cosmos DB 模擬器正在執行。

2. 開啟終端機視窗，並 `cd` 至儲存應用程式的目錄。

3. 接著，使用 `set FLASK_APP=app.py` 來設定 Flask 應用程式的環境變數；如果您使用的是 Mac，請改為使用 `export FLASK_APP=app.py`。

4. 使用 `flask run` 執行應用程式，並瀏覽至 [http://127.0.0.1:5000/](http://127.0.0.1:5000/)。

5. 新增及移除工作，您將能在集合中看到那些工作的新增和變更。

## <a name="create-a-database-account"></a>建立資料庫帳戶

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

## <a name="update-your-connection-string"></a>更新您的連接字串

如果您想要針對即時 Azure Cosmos DB 帳戶測試程式碼，請移至 Azure 入口網站以建立帳戶，並取得您的連接字串資訊。 接著將它複製到應用程式。

1. 在 [Azure 入口網站](http://portal.azure.com/)中，於您 Azure Cosmos DB 帳戶的左側瀏覽區中，按一下 [連接字串]，然後按一下 [讀寫金鑰]。 在下一個步驟中，您將使用畫面右側的複製按鈕，將使用者名稱、密碼和主機複製到 Dal.cs 檔案中。

2. 開啟根目錄中的 **app.py** 檔案。

3. 從入口網站複製您的 [使用者名稱] 值 (使用 [複製] 按鈕)，並使它成為 **app.py** 檔案中的 **name** 值。

4. 接著，從入口網站複製您的 [連接字串] 值，並使它成為 **app.py** 檔案中的 MongoClient 值。

5. 最後，從入口網站複製您的 [密碼] 值，並使它成為 **app.py** 檔案中的 **password** 值。

您現已更新應用程式，使其具有與 Azure Cosmos DB 通訊所需的所有資訊。 您可以使用與之前相同的方式執行它。

## <a name="deploy-to-azure"></a>部署至 Azure

若要部署此應用程式，您可以在 Azure 中建立新的 Web 應用程式，並搭配此 GitHub 存放庫的分支啟用持續部署。 請遵循此[教學課程](https://docs.microsoft.com/azure/app-service-web/app-service-continuous-deployment)，以在 Azure 中搭配 GitHub 設定持續部署。

部署至 Azure 時，您應該移除應用程式金鑰，並確定以下區段未註解化：

```python
    client = MongoClient(os.getenv("MONGOURL"))
    db = client.test    #Select the database
    db.authenticate(name=os.getenv("MONGO_USERNAME"),password=os.getenv("MONGO_PASSWORD"))
```

接著，您需要將 MONGOURL、MONGO_PASSWORD 和 MONGO_USERNAME 新增到應用程式設定。 您可以遵循此[教學課程](https://docs.microsoft.com/azure/app-service-web/web-sites-configure#application-settings)，以深入了解 Azure Web Apps 中的應用程式設定。

如果您不想要建立此存放庫的分支，也可以按一下下方的 [部署至 Azure] 按鈕。 接著，您應該移至 Azure 並使用 Cosmos DB 帳戶資訊設定應用程式設定。

<a href="https://deploy.azure.com/?repository=https://github.com/heatherbshapiro/To-Do-List---Flask-MongoDB-Example" target="_blank">
<img src="http://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> 如果您打算將程式碼儲存在 GitHub 或其他原始檔控制選項之中，請務必從程式碼中移除您的連接字串。 您可以改成搭配 Web 應用程式的應用程式來設定它們。

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 入口網站中檢閱 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何建立 Azure Cosmos DB 帳戶，以及使用適用於 MongoDB 的 API 來執行 Flask 應用程式。您現已可以將其他資料匯入 Cosmos DB 帳戶。

> [!div class="nextstepaction"]
> [將資料匯入 MongoDB API 的 Azure Cosmos DB](mongodb-migrate.md)
