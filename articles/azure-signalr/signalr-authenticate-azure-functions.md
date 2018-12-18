---
title: 教學課程：使用 Azure Functions 進行 Azure SignalR 服務驗證 | Microsoft Docs
description: 在本教學課程中，您將會了解如何驗證 Azure SignalR 服務用戶端
services: signalr
documentationcenter: ''
author: sffamily
manager: cfowler
editor: ''
ms.assetid: ''
ms.service: signalr
ms.workload: tbd
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 09/18/2018
ms.author: zhshang
ms.openlocfilehash: 8af657c39217f3edcadef6ec0981a31ec7e89aa6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978411"
---
# <a name="tutorial-azure-signalr-service-authentication-with-azure-functions"></a>教學課程：使用 Azure Functions 進行 Azure SignalR 服務驗證

此教學課程將逐步解說如何使用 Azure Functions、App Service 驗證和 SignalR 服務建置驗證和私人傳訊的聊天室。

## <a name="introduction"></a>簡介

### <a name="technologies-used"></a>使用的技術

* [Azure Functions](https://azure.microsoft.com/services/functions/?WT.mc_id=serverlesschatlab-tutorial-antchu) - 用來驗證使用者及傳送聊天訊息的後端 API
* [Azure SignalR 服務](https://azure.microsoft.com/services/signalr-service/?WT.mc_id=serverlesschatlab-tutorial-antchu) - 將新訊息廣播至已連線的聊天用戶端
* [Azure 儲存體](https://azure.microsoft.com/services/storage/?WT.mc_id=serverlesschatlab-tutorial-antchu) - 裝載聊天用戶端 UI 的靜態網站

### <a name="prerequisites"></a>必要條件

以下是建置本教學課程所需的軟體。

* [Git](https://git-scm.com/downloads)
* [Node.js](https://nodejs.org/en/download/) (10.x 版)
* [.NET SDK](https://www.microsoft.com/net/download) (2.x 版，此為 Functions 擴充功能的必要項目)
* [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools) (第 2 版)
* 具有下列擴充功能的 [Visual Studio Code](https://code.visualstudio.com/) (VS Code)
    * [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) - 在 VS Code 中使用 Azure Functions
    * [即時伺服器](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) - 在本機提供網頁以供測試之用


## <a name="sign-into-the-azure-portal"></a>登入 Azure 入口網站

移至 [Azure 入口網站](https://portal.azure.com/)，並以您的認證登入。


## <a name="create-an-azure-signalr-service-instance"></a>建立 Azure SignalR 服務執行個體

您將在本機建置並測試 Azure Functions 應用程式。 應用程式會存取 Azure 中必須事先建立的 SignalR 服務執行個體。

1. 按一下 [建立資源] (**+**) 按鈕以建立新的 Azure 資源。

1. 搜尋 [SignalR 服務] 並加以選取。 按一下頁面底部的 [新增] 。

    ![新增 SignalR 服務](media/signalr-authenticate-azure-functions/signalr-quickstart-new.png)

1. 輸入以下資訊：

    | 名稱 | 值 |
    |---|---|
    | 資源名稱 | SignalR 服務執行個體的唯一名稱 |
    | 資源群組 | 建立新的資源群組 |
    | 位置 | 選取您附近的位置 |
    | 定價層 | 免費 |
    
1. 按一下頁面底部的 [新增] 。


## <a name="initialize-the-function-app"></a>初始化函式應用程式

### <a name="create-a-new-azure-functions-project"></a>建立新的 Azure Functions 專案

1. 在新的 VS Code 視窗中，使用功能表中的 `File > Open Folder`，在適當位置建立並開啟一個空的資料夾。 您建置的應用程式將以此作為主要專案資料夾。

1. 在 VS Code 中使用 Azure Functions 擴充功能，初始化主要專案資料夾中的函式應用程式。
    1. 從功能表中選取 [檢視] > [命令選擇區] (快速鍵 `Ctrl-Shift-P`，macOS：`Cmd-Shift-P`)，以在 VS Code 中開啟命令選擇區。
    1. 搜尋 [Azure Functions：建立新專案] 命令，並加以選取。
    1. 此時應會出現主要專案資料夾。 請加以選取 (或使用 [瀏覽] 找出此資料夾)。
    1. 若要在提示中選擇語言，請選取 [JavaScript]。

    ![建立函數應用程式](media/signalr-authenticate-azure-functions/signalr-create-vscode-app.png)


### <a name="install-function-app-extensions"></a>安裝函式應用程式擴充功能

本教學課程將使用 Azure Functions 與 Azure SignalR 服務互動。 如同其他大部分的繫結，SignalR 服務繫結也是以擴充功能的形式提供，且必須使用 Azure Functions Core Tools CLI 進行安裝，才能夠使用。

1. 從功能表中選取 [檢視] > [整合式終端機] (Ctrl-`)，以在 VS Code 中開啟終端機。

1. 確定主要專案資料夾是目前的目錄。

1. 安裝 SignalR 服務函式應用程式擴充功能。
    ```
    func extensions install -p Microsoft.Azure.WebJobs.Extensions.SignalRService -v 1.0.0-preview1-10002
    ```

### <a name="configure-application-settings"></a>設定應用程式設定

在本機執行和偵錯 Azure Functions 執行階段時，會從 **local.settings.json** 讀取應用程式設定。 請使用您先前建立的 SignalR 服務執行個體所具備的連接字串，來更新此檔案。

1. 在 VS Code 的 [總管] 窗格中選取 **local.settings.json**，並加以開啟。

1. 將檔案的內容取代為下列內容。
    ```json
    {
        "IsEncrypted": false,
        "Values": {
            "AzureSignalRConnectionString": "<signalr-connection-string>",
            "WEBSITE_NODE_DEFAULT_VERSION": "10.0.0",
            "FUNCTIONS_WORKER_RUNTIME": "node"
        },
        "Host": {
            "LocalHttpPort": 7071,
            "CORS": "*"
        }
    }
    ```

    * 將 Azure SignalR 服務連接字串輸入名為 `AzureSignalRConnectionString` 的設定中。 請在 Azure 入口網站中，從 Azure SignalR 服務資源的 [金鑰] 頁面上取得其值；您可以使用主要或次要連接字串。
    * `WEBSITE_NODE_DEFAULT_VERSION` 設定不會在本機使用，但在部署至 Azure 時即需使用。
    * `Host` 區段會設定本機 Functions 主機的連接埠和 CORS 設定 (此設定在 Azure 中執行時沒有效用)。

    ![取得 SignalR 服務金鑰](media/signalr-authenticate-azure-functions/signalr-get-key.png)

1. 儲存檔案。

    ![更新本機設定](media/signalr-authenticate-azure-functions/signalr-update-local-settings.png)


## <a name="create-a-function-to-authenticate-users-to-signalr-service"></a>建立向 SignalR 服務驗證使用者的函式

當聊天應用程式首先在瀏覽器中開啟時，它需要有效的連線認證以連線至 Azure SignalR 服務。 您將在您的函式應用程式中建立名為 *SignalRInfo* 的 HTTP 觸發函式，以傳回此連線資訊。

1. 開啟 VS Code 命令選擇區 (`Ctrl-Shift-P`，macOS：`Cmd-Shift-P`)。

1. 搜尋並選取 [Azure Functions：建立函式] 命令。

1. 出現提示時，請提供下列資訊。

    | 名稱 | 值 |
    |---|---|
    | 函式應用程式資料夾 | 選取主要專案資料夾 |
    | 範本 | HTTP 觸發程序 |
    | 名稱 | SignalRInfo |
    | 授權層級 | 匿名 |
    
    此時會建立名為 **SignalRInfo** 的資料夾，其中包含新的函式。

1. 開啟 **SignalRInfo/function.json** 以設定函式的繫結。 將檔案的內容修改為下列內容。 這會新增輸入繫結，為用戶端產生用來連線至 Azure SignalR 服務中樞 `chat` 的有效認證。

    ```json
    {
        "disabled": false,
        "bindings": [
            {
                "authLevel": "anonymous",
                "type": "httpTrigger",
                "direction": "in",
                "name": "req"
            },
            {
                "type": "http",
                "direction": "out",
                "name": "res"
            },
            {
                "type": "signalRConnectionInfo",
                "name": "connectionInfo",
                "userId": "",
                "hubName": "chat",
                "direction": "in"
            }
        ]
    }
    ```

    `signalRConnectionInfo` 繫結中的 `userId` 屬性可用來建立已驗證的 SignalR 服務連線。 在本機開發中，請將該屬性保留為空白。 當函式應用程式部署至 Azure 時，將會使用該屬性。

1. 開啟 **SignalRInfo/index.js** 以檢視函式的主體。 將檔案的內容修改為下列內容。

    ```javascript
    module.exports = function (context, req, connectionInfo) {
        context.res = { body: connectionInfo };
        context.done();
    };
    ```

    此函式會取得輸入繫結中的 SignalR 連線資訊，並在 HTTP 回應本文中將其傳回至用戶端。


## <a name="create-a-function-to-send-chat-messages"></a>建立傳送聊天訊息的函式

Web 應用程式也需要 HTTP API 以傳送聊天訊息。 您將建立名為 *SendMessage* 的 HTTP 觸發函式，以使用 SignalR 服務將訊息傳送至所有已連線的用戶端。

1. 開啟 VS Code 命令選擇區 (`Ctrl-Shift-P`，macOS：`Cmd-Shift-P`)。

1. 搜尋並選取 [Azure Functions：建立函式] 命令。

1. 出現提示時，請提供下列資訊。

    | 名稱 | 值 |
    |---|---|
    | 函式應用程式資料夾 | 選取主要專案資料夾 |
    | 範本 | HTTP 觸發程序 |
    | 名稱 | SendMessage |
    | 授權層級 | 匿名 |
    
    此時會建立名為 **SendMessage** 的資料夾，其中包含新的函式。

1. 開啟 **SendMessage/function.json** 以設定函式的繫結。 將檔案的內容修改為下列內容。
    ```json
    {
        "disabled": false,
        "bindings": [
            {
                "authLevel": "anonymous",
                "type": "httpTrigger",
                "direction": "in",
                "name": "req",
                "route": "messages",
                "methods": [
                    "post"
                ]
            },
            {
                "type": "http",
                "direction": "out",
                "name": "res"
            },
            {
                "type": "signalR",
                "name": "signalRMessages",
                "hubName": "chat",
                "direction": "out"
            }
        ]
    }
    ```
    這會對原始函式進行兩項變更：
    * 變更 `messages` 的路由，並將 HTTP 觸發程序限定於 **POST** HTTP 方法。
    * 新增 SignalR 服務輸出繫結，以將訊息傳送至所有已連線至 SignalR 服務中樞 `chat` 的用戶端。

1. 儲存檔案。

1. 開啟 **SendMessage/index.js** 以檢視函式的主體。 將檔案的內容修改為下列內容。
    ```javascript
    module.exports = function (context, req) {
        const message = req.body;
        message.sender = req.headers && req.headers['x-ms-client-principal-name'] || '';
            
        let recipientUserId = '';
        if (message.recipient) {
            recipientUserId = message.recipient;
            message.isPrivate = true;
        }
    
        context.bindings.signalRMessages = [{
            'userId': recipientUserId,
            'target': 'newMessage',
            'arguments': [ message ]
        }];
        context.done();
    };
    ```
    此函式會取得 HTTP 要求中的本文，並將其傳送至已連線至 SignalR 服務的用戶端，在每個用戶端上叫用名為 `newMessage` 的函式。

    此函式可讀取寄件者的身分識別，並可接受訊息本文中的*收件者*值，讓訊息能夠私下傳送給單一使用者。 稍後在教學課程中將會用到這些功能。

1. 儲存檔案。


## <a name="create-and-run-the-chat-client-web-user-interface"></a>建立並執行聊天用戶端 Web 使用者介面

聊天應用程式的 UI 是以 Vue JavaScript 架構建立的簡易單頁應用程式 (SPA)。 它會與函式應用程式分開來裝載。 在本機上，您將使用即時伺服器 VS Code 擴充功能來執行 Web 介面。

1. 在 VS Code 中，請在主要專案資料夾的根目錄建立名為**內容**的新資料夾。

1. 在**內容**資料夾中，建立名為 **index.html** 的新檔案。

1. 複製並貼上 **[index.html](https://raw.githubusercontent.com/Azure-Samples/signalr-service-quickstart-serverless-chat/master/docs/demo/chat-with-auth/index.html)** 的內容。

1. 儲存檔案。

1. 按 **F5** 在本機執行函式應用程式，並連結偵錯工具。

1. 在 **index.html** 開啟後，開啟 VS Code 命令選擇區 (`Ctrl-Shift-P`，macOS：`Cmd-Shift-P`)，然後選取 [即時伺服器：開啟即時伺服器]，以啟動即時伺服器。 即時伺服器將會在瀏覽器中開啟應用程式。

1. 應用程式隨即開啟。 在聊天方塊中輸入訊息，然後按 Enter 鍵。 重新整理應用程式，以查看新的訊息。 由於未設定任何驗證，因此所有訊息都會以「匿名」的形式傳送。


## <a name="deploy-to-azure-and-enable-authentication"></a>部署至 Azure 並啟用驗證

您已在本機執行函式應用程式和聊天應用程式。 現在，您要將其部署至 Azure，並在應用程式中啟用驗證和私人傳訊。


### <a name="log-into-azure-with-vs-code"></a>使用 VS Code 登入 Azure

1. 開啟 VS Code 命令選擇區 (`Ctrl-Shift-P`，macOS：`Cmd-Shift-P`)。

1. 搜尋並選取 [Azure：登入] 命令。

1. 在您的瀏覽器中依照指示完成登入程序。


### <a name="configure-function-app-for-authentication"></a>進行函式應用程式的驗證設定

到目前為止，聊天應用程式皆以匿名方式運作。 在 Azure 中，您將使用 [App Service 驗證](https://docs.microsoft.com/azure/app-service/app-service-authentication-overview)來驗證使用者。 已驗證的使用者所使用的使用者識別碼或使用者名稱可傳至 *SignalRConnectionInfo* 繫結，以產生驗證為使用者的連線資訊。

在傳送訊息時，應用程式可以決定是要將其傳送給所有已連線的用戶端，還是只傳送給已對指定的使用者驗證的用戶端。

1. 在 VS Code 中，開啟 **SendMessage/function.json**。

1. 將[繫結運算式](https://docs.microsoft.com/azure/azure-functions/functions-triggers-bindings#binding-expressions-and-patterns)插入 *SignalRConnectionInfo* 繫結的 *userId*屬性中：`{headers.x-ms-client-principal-name}`。 這會將值設為已驗證使用者的使用者名稱。 其屬性此時應該會顯示如下。

    ```json
    {
        "type": "signalRConnectionInfo",
        "name": "connectionInfo",
        "userId": "{headers.x-ms-client-principal-name}",
        "hubName": "chat",
        "direction": "in"
    }
    ```

1. 儲存檔案。


### <a name="deploy-function-app"></a>部署函式應用程式

1. 開啟 VS Code 命令選擇區 (`Ctrl-Shift-P`，macOS：`Cmd-Shift-P`)，然後選取 [Azure Functions：部署至函式應用程式]。 

1. 出現提示時，請提供下列資訊。

    | 名稱 | 值 |
    |---|---|
    | 要部署的資料夾 | 選取主要專案資料夾 |
    | 訂用帳戶 | 選取您的訂用帳戶 |
    | 函式應用程式 | 選取 [建立新的函式應用程式] |
    | 函式應用程式名稱 | 輸入唯一名稱 |
    | 資源群組 | 選取與 SignalR 服務執行個體相同的資源群組 |
    | 儲存體帳戶 | 選取 [建立新的儲存體帳戶] |
    | 儲存體帳戶名稱 | 輸入唯一的名稱 (3 到 24 個字元，僅限英數字元) |
    | 位置 | 選取您附近的位置 |
    
    Azure 中會建立新的函式應用程式，並開始進行部署。 等待部署完成。


### <a name="upload-function-app-local-settings"></a>上傳函式應用程式本機設定

1. 開啟 VS Code 命令選擇區 (`Ctrl-Shift-P`，macOS：`Cmd-Shift-P`)。

1. 搜尋並選取 [Azure Functions：上傳本機設定] 命令。

1. 出現提示時，請提供下列資訊。

    | 名稱 | 值 |
    |---|---|
    | 本機設定檔 | local.settings.json |
    | 訂用帳戶 | 選取您的訂用帳戶 |
    | 函式應用程式 | 選取先前部署的函式應用程式 |
    | 函式應用程式名稱 | 輸入唯一名稱 |

本機設定會上傳至 Azure 中的函式應用程式。 如果系統提示您覆寫現有設定，請選取 [全部皆是]。


### <a name="enable-function-app-cross-origin-resource-sharing-cors"></a>啟用跨原始來源資源共用 (CORS) 的函式應用程式

雖然 **local.settings.json** 中有 CORS 設定，但此設定並未傳播至 Azure 中的函式應用程式。 您必須個別加以設定。

1. 開啟 VS Code 命令選擇區 (`Ctrl-Shift-P`，macOS：`Cmd-Shift-P`)。

1. 搜尋並選取 [Azure Functions：在入口網站中開啟] 命令。

1. 選取訂用帳戶和函式應用程式名稱，以在 Azure 入口網站中開啟函式應用程式。

1. 在開啟的入口網站中，選取 [平台功能] 索引標籤下方的 [CORS]。

    ![尋找 CORS](media/signalr-authenticate-azure-functions/signalr-find-cors.png)

1. 新增具有值 `*` 的項目。

1. 移除所有其他現有的項目。

1. 按一下 [儲存] 以保存 CORS 設定。

    ![設定 CORS](media/signalr-authenticate-azure-functions/signalr-set-cors.png)

> [!NOTE]
> 在實際的應用程式中，您不應允許所有原始來源的 CORS (`*`)，更安全的方法是針對每個有需要的網域輸入特定的 CORS 項目。


### <a name="update-the-web-app"></a>更新 Web 應用程式

1. 在 Azure 入口網站中，瀏覽至函式應用程式的 [概觀] 頁面。

1. 複製函式應用程式的 URL。

    ![取得 URL](media/signalr-authenticate-azure-functions/signalr-get-url.png)


1. 在 VS Code 中開啟 **index.html**，並將 `apiBaseUrl` 的值取代為函式應用程式的 URL。

1. 應用程式可以使用 Azure Active Directory、Facebook、Twitter、Microsoft 帳戶或 Google 設定驗證。 請設定 `authProvider` 的值以選取您所將使用的驗證提供者。

1. 儲存檔案。


### <a name="deploy-the-web-application-to-blob-storage"></a>將 Web 應用程式部署至 Blob 儲存體

Web 應用程式會使用 Azure Blob 儲存體的靜態網站功能進行裝載。

1. 按一下 [新增] (**+**) 按鈕以建立新的 Azure 資源。

1. 在 [儲存體] 下方，選取 [儲存體帳戶]。

1. 輸入以下資訊：

    | 名稱 | 值 |
    |---|---|
    | 名稱 | Blob 儲存體帳戶的唯一名稱 |
    | 帳戶類型 | StorageV2 (一般用途 V2) |
    | 位置 | 選取與您的其他資源相同的區域 |
    | 複寫 | 本地備援儲存體 (LRS) |
    | 效能 | 標準 |
    | 存取層 | 經常性存取 |
    | 資源群組 | 在本教學課程中，請選取與其他資源相同的資源群組 |

1. 按一下頁面底部的 [新增] 。

    ![建立儲存體](media/signalr-authenticate-azure-functions/signalr-create-storage.png)

1. 當儲存體帳戶建立後，在 Azure 入口網站中加以開啟。

1. 在左側導覽中選取 [靜態網站 (預覽)]。

1. 選取 [啟用]。

1. 輸入 `index.html` 作為**索引文件名稱**。

1. 按一下 [檔案] 。

    ![設定靜態網站](media/signalr-authenticate-azure-functions/signalr-static-websites.png)

1. 按一下頁面上的 **$web** 連結，以開啟 Blob 容器。

1. 按一下 [上傳]，並上傳**內容**資料夾中的 **index.html**。

1. 回到 [靜態網站] 頁面。 請記下**主要端點**。 這是 Web 應用程式的 URL。


### <a name="enable-app-service-authentication"></a>啟用 App Service 驗證

App Service 驗證支援以 Azure Active Directory、Facebook、Twitter、Microsoft 帳戶和 Google 進行驗證。

1. 在入口網站中的函式應用程式仍開啟的情況下，找出 [平台功能] 索引標籤，選取 [驗證/授權]。

1. **開啟** App Service 驗證。

1. 在 [當要求未經驗證時所要採取的動作] 中，選取 [使用 {您先前選取的驗證提供者} 登入]。

1. 在 [允許的外部重新導向 URL] 中，輸入您先前記下的儲存體帳戶主要 Web 端點的 URL。

1. 根據您選擇的登入提供者，依照其隨附文件中的指示完成設定。

    - [Azure Active Directory](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-active-directory-authentication)
    - [Facebook](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-facebook-authentication)
    - [Twitter](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-twitter-authentication)
    - [Microsoft 帳戶](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-microsoft-authentication)
    - [Google](https://docs.microsoft.com/azure/app-service/app-service-mobile-how-to-configure-google-authentication)


### <a name="try-the-application"></a>嘗試使用應用程式

1. 在瀏覽器中，瀏覽至儲存體帳戶的主要 Web 端點。

1. 選取 [登入]，使用您選擇的驗證提供者進行驗證。

1. 將公用訊息輸入主要聊天方塊中，以傳送該訊息。

1. 按一下聊天記錄中的使用者名稱，以傳送私人訊息。 只有選取的收件者會收到這些訊息。

恭喜！ 您已部署即時、無伺服器的聊天應用程式！

![示範](media/signalr-authenticate-azure-functions/signalr-serverless-chat.gif)


## <a name="clean-up-resources"></a>清除資源

若要清除在本教學課程中建立的資源，請使用 Azure 入口網站刪除資源群組。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何搭配使用 Azure Functions 與 Azure SignalR 服務。 請深入了解如何使用適用於 Azure Functions 的 SignalR Service 繫結建置即時無伺服器應用程式。

> [!div class="nextstepaction"]
> [使用 Azure Functions 建置即時應用程式](signalr-overview-azure-functions.md)
