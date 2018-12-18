---
title: 使用 Socket.io 的 Node.js 應用程式 - Azure
description: 學習如何在裝載於 Azure 的 node.js 應用程式中使用 socket.io。
services: cloud-services
documentationcenter: nodejs
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 7f9435e0-7732-4aa1-a4df-ea0e894b847f
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 08/17/2017
ms.author: jeconnoc
ms.openlocfilehash: 0fae47f248d5662b69a0d1a12c82b7ded33badd6
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001978"
---
# <a name="build-a-nodejs-chat-application-with-socketio-on-an-azure-cloud-service"></a>在 Azure 雲端服務上使用 Socket.IO 建立 Node.js 交談應用程式

Socket.IO 提供 node.js 伺服器和用戶端之間的即時通訊。 本教學課程引導您將 socket.IO 型交談應用程式裝載於 Azure 上。 如需 Socket.IO 的詳細資訊，請參閱 [socket.io](http://socket.io)。

完成之應用程式的螢幕擷取畫面如下：

![A browser window displaying the service hosted on Azure][completed-app]  

## <a name="prerequisites"></a>必要條件
請確定已安裝下列產品及版本，以順利完成本文中的範例：

* 安裝 [Visual Studio](https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx)
* 安裝 [Node.js](https://nodejs.org/download/)
* 安裝 [Python 版本 2.7.10](https://www.python.org/)

## <a name="create-a-cloud-service-project"></a>建立雲端服務專案
下列步驟會建立雲端服務專案來裝載 Socket.IO 應用程式。

1. 從 [開始功能表] 或 [開始畫面] 搜尋 **Windows PowerShell**。 最後，用滑鼠右鍵按一下 [Windows PowerShell]，然後選取 [以系統管理員身分執行]。
   
    ![Azure PowerShell icon][powershell-menu]
2. 建立名為 **c:\\node** 的目錄。 
   
        PS C:\> md node
3. 切換至 **c:\\node** 目錄，
   
        PS C:\> cd node
4. 然後輸入下列命令來建立名為 **chatapp** 的新方案和名為 **WorkerRole1** 的背景工作角色：
   
        PS C:\node> New-AzureServiceProject chatapp
        PS C:\Node> Add-AzureNodeWorkerRole
   
    您將看見下列回應：
   
    ![The output of the new-azureservice and add-azurenodeworkerrolecmdlets](./media/cloud-services-nodejs-chat-app-socketio/socketio-1.png)

## <a name="download-the-chat-example"></a>下載交談範例
在此專案中，我們使用 [Socket.IO GitHub 儲存機制]中的交談範例。 請執行下列步驟來下載範例，並將它加入至您先前建立的專案。

1. 使用 [Clone]  按鈕來建立儲存機制的本機複本。 您也可以使用 [ZIP]  按鈕來下載專案。
   
   ![檢視 https://github.com/LearnBoost/socket.io/tree/master/examples/chat 的瀏覽器視窗，且 ZIP 下載圖示反白顯示][chat-example-view]
2. 瀏覽本機儲存機制的目錄結構，直到找到 **examples\\chat** 目錄為止。 將此目錄的內容複製到稍早建立的 **C:\\node\\chatapp\\WorkerRole1** 目錄。
   
   ![總管，會顯示擷取自封存的 examples\\chat 目錄內容][chat-contents]
   
   上方螢幕擷取畫面中反白顯示的項目是從 **examples\\chat** 目錄複製的檔案
3. 在 **C:\\node\\chatapp\\WorkerRole1** 目錄中，刪除 **server.js** 檔案，然後將 **app.js** 檔案重新命名為 **server.js**。 這樣會移除先前由 **Add-AzureNodeWorkerRole** Cmdlet 建立的預設 **server.js** 檔案，並取代為交談範例中的應用程式檔案。

### <a name="modify-serverjs-and-install-modules"></a>修改 Server.js 和安裝模組
在 Azure 模擬器中測試應用程式之前，我們必須稍做一些修改。 請對 server.js 檔案執行下列步驟：

1. 在 Visual Studio 或其他文字編輯器中開啟 **server.js** 檔案。
2. 在 server.js 開頭找出 **Module dependencies** 區段，將含有 **sio = require('..//..//lib//socket.io')** 的那一行變更為 **sio = require('socket.io')**，如下所示：
   
       var express = require('express')
         , stylus = require('stylus')
         , nib = require('nib')
       //, sio = require('..//..//lib//socket.io'); //Original
         , sio = require('socket.io');                //Updated
         var port = process.env.PORT || 3000;         //Updated
3. 為了確保應用程式在正確的連接埠上接聽，請在 [記事本] 或您喜歡的編輯器中開啟 server.js，然後變更下列這一行，將 **3000** 改為 **process.env.port**，如下所示：
   
       //app.listen(3000, function () {            //Original
       app.listen(process.env.port, function () {  //Updated
         var addr = app.address();
         console.log('   app listening on http://' + addr.address + ':' + addr.port);
       });

儲存 **server.js**的變更之後，請使用下列步驟來安裝必要的模組，然後在 Azure 模擬器中測試應用程式：

1. 使用 **Azure PowerShell**，切換至 **C:\\node\\chatapp\\WorkerRole1** 目錄，再使用下列命令來安裝此應用程式所需的模組：
   
       PS C:\node\chatapp\WorkerRole1> npm install
   
   這將會安裝 package.json 檔案中列出的模組。 命令完成之後，您應該會看到類似這樣的輸出：
   
   ![The output of the npm install command][The-output-of-the-npm-install-command]
2. 由於此範例原本為 Socket.IO GitHub 儲存機制的一部分，依相對路徑來直接參考 Socket.IO 程式庫，且 package.json 檔案中並沒有參考 Socket.IO，所以我們必須使用下列指令來安裝它：
   
       PS C:\node\chatapp\WorkerRole1> npm install socket.io --save

### <a name="test-and-deploy"></a>測試和部署
1. 發出下列命令來啟動模擬器：
   
       PS C:\node\chatapp\WorkerRole1> Start-AzureEmulator -Launch
   
   > [!NOTE]
   > 如果您在啟動模擬器時遇到問題，例如：Start-AzureEmulator：發生未預期的失敗。  詳細資料：發生未預期的錯誤。通訊物件 System.ServiceModel.Channels.ServiceChannel 無法用於通訊，因為它處於錯誤狀態。
   
      重新安裝 AzureAuthoringTools v 2.7.1 和 AzureComputeEmulator v 2.7 - 請確定版本相符。
   >
   >


2. 開啟瀏覽器並瀏覽至 **http://127.0.0.1**。
3. 當瀏覽器視窗開啟時，請輸入暱稱，然後按 Enter 鍵。
   這可讓您以特定的暱稱來張貼訊息。 若要測試多使用者功能，請使用相同 URL 開啟其他瀏覽器視窗，並輸入不同的暱稱。
   
   ![兩個瀏覽器視窗顯示 User1 和 User2 的交談訊息](./media/cloud-services-nodejs-chat-app-socketio/socketio-8.png)
4. 測試應用程式之後，發出下列命令來停止模擬器：
   
       PS C:\node\chatapp\WorkerRole1> Stop-AzureEmulator
5. 若要將應用程式部署至 Azure，請使用 **Publish-AzureServiceProject** Cmdlet。 例如︰
   
       PS C:\node\chatapp\WorkerRole1> Publish-AzureServiceProject -ServiceName mychatapp -Location "East US" -Launch
   
   > [!IMPORTANT]
   > 務必使用唯一的名稱，否則發行程序會失敗。 部署完成之後，瀏覽器會開啟並瀏覽至已部署的服務。
   > 
   > 如果出現錯誤指出匯入的發行設定檔中沒有您所提供的訂用帳戶名稱，則在部署至 Azure 之前，您必須下載並匯入訂用帳戶的發行設定檔。 請參閱＜ **建立 Node.js 應用程式並部署至 Azure 雲端服務** ＞的＜ [將應用程式部署至 Azure](https://azure.microsoft.com/develop/nodejs/tutorials/getting-started/)
   > 
   > 
   
   ![A browser window displaying the service hosted on Azure][completed-app]
   
   > [!NOTE]
   > 如果出現錯誤指出匯入的發行設定檔中沒有您所提供的訂用帳戶名稱，則在部署至 Azure 之前，您必須下載並匯入訂用帳戶的發行設定檔。 請參閱＜ **建立 Node.js 應用程式並部署至 Azure 雲端服務** ＞的＜ [將應用程式部署至 Azure](https://azure.microsoft.com/develop/nodejs/tutorials/getting-started/)
   > 
   > 

應用程式現在已在 Azure 上執行，且可以使用 Socket.IO 在不同用戶端之間轉送聊天訊息。

> [!NOTE]
> 為了簡化起見，此範例只能讓連線至相同執行個體的使用者聊天。 這表示如果雲端服務建立兩個背景工作角色執行個體，則使用者只能夠與連線至相同背景工作角色執行個體的其他使用者交談。 若要擴充應用程式來處理多個角色執行個體，您可以使用服務匯流排之類的技術，以便跨執行個體來共用 Socket.IO 儲存狀態。 如需相關範例，請參閱＜ [Azure SDK for Node.js GitHub 儲存機制](https://github.com/WindowsAzure/azure-sdk-for-node)＞(英文) 中的服務匯流排佇列和主題使用範例。
> 
> 

## <a name="next-steps"></a>後續步驟
在本教學課程中，您學到如何建立裝載於 Azure 雲端服務的基本交談應用程式。 若要了解如何在 Azure 網站中裝載此應用程式，請參閱[在 Azure 網站上使用 Socket.IO 建立 Node.js 交談應用程式][chatwebsite]。

如需詳細資訊，也請參閱 [Node.js 開發人員中心](https://docs.microsoft.com/javascript/azure/?view=azure-node-latest)。

[chatwebsite]: https://docs.microsoft.com/azure/cloud-services/cloud-services-nodejs-develop-deploy-app

[Azure SLA]: http://www.windowsazure.com/support/sla/
[Azure SDK for Node.js GitHub repository]: https://github.com/WindowsAzure/azure-sdk-for-node
[completed-app]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-10.png
[Azure SDK for Node.js]: https://www.windowsazure.com/develop/nodejs/
[Node.js Web Application]: https://www.windowsazure.com/develop/nodejs/tutorials/getting-started/
[Socket.IO GitHub 儲存機制]: https://github.com/LearnBoost/socket.io/tree/0.9.14
[Azure Considerations]: #windowsazureconsiderations
[Hosting the Chat Example in a Worker Role]: #hostingthechatexampleinawebrole
[Summary and Next Steps]: #summary
[powershell-menu]: ./media/cloud-services-nodejs-chat-app-socketio/azure-powershell-start.png

[chat example]: https://github.com/LearnBoost/socket.io/tree/master/examples/chat
[chat-example-view]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-22.png


[chat-contents]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-5.png
[The-output-of-the-npm-install-command]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-7.png
[The output of the Publish-AzureService command]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-9.png


