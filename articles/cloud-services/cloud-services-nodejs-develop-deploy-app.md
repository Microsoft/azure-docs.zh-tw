---
title: Node.js 入門指南
description: 了解如何建立簡單的 Node.js Web 應用程式，並將它部署至 Azure 雲端服務。
services: cloud-services
documentationcenter: nodejs
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 50951a87-fed4-48e0-bcfa-453b9e50452e
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: hero-article
ms.date: 08/17/2017
ms.author: jeconnoc
ms.openlocfilehash: 7af18272c335c835a2aa406fc635f3e7afda2a3a
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39003447"
---
# <a name="build-and-deploy-a-nodejs-application-to-an-azure-cloud-service"></a>建立 Node.js 應用程式並部署到 Azure 雲端服務

本教學課程示範如何建立在 Azure 雲端服務中執行的簡單 Node.js 應用程式。 雲端服務是 Azure 中可擴充雲端應用程式的建置組塊。 可以個別獨立管理和向外延展應用程式的前端和後端元件。  雲端服務提供穩定代管各個角色的強固專用虛擬機器。

如需雲端服務及其相較於 Azure 網站和虛擬機器的詳細資訊，請參閱 [Azure 網站、雲端服務與虛擬機器的比較]。

> [!TIP]
> 尋求建置簡單的網站？ 如果您只需要簡單的網站前端，請考慮[使用輕量型 Web 應用程式]。 隨著 Web 應用程式擴大以及需求改變，您可以輕易地升級到雲端服務。

按照本教學課程進行，您將建立在 Web 角色內代管的簡單 Web 應用程式。 您將使用計算模擬器在本機測試您的應用程式，然後使用 PowerShell 命令列工具部署應用程式。

應用程式是簡單的 "hello world" 應用程式：

![顯示 Hello World 網頁的網頁瀏覽器][A web browser displaying the Hello World web page]

## <a name="prerequisites"></a>必要條件
> [!NOTE]
> 本教學課程使用 Azure PowerShell (需要 Windows)。

* 安裝並設定 [Azure PowerShell]。
* 下載並安裝 [Azure SDK for .NET 2.7]。 在安裝過程中，選取：
  * MicrosoftAzureAuthoringTools
  * MicrosoftAzureComputeEmulator

## <a name="create-an-azure-cloud-service-project"></a>建立 Azure 雲端服務專案
執行下列工作，建立新的 Azure 雲端服務專案以及基本的 Node.js 樣板：

1. 以系統管理員身分執行 **Windows PowerShell**；從 [開始功能表] 或 [開始畫面] 中，搜尋 [Windows PowerShell]。
2. [連線 PowerShell] 至您的訂用帳戶。
3. 輸入下列 PowerShell Cmdlet 來建立專案：

        New-AzureServiceProject helloworld

    ![New-AzureService helloworld 命令的結果][The result of the New-AzureService helloworld command]

    **New-AzureServiceProject** Cmdlet 會產生可將 Node.js 應用程式發佈至雲端服務的基本結構。 其中包含發佈到 Azure 所需的設定檔。 該 Cmdlet 也會將您的工作目錄變更為服務的目錄。

    Cmdlet 會建立下列檔案：

   * **ServiceConfiguration.Cloud.cscfg**、**ServiceConfiguration.Local.cscfg** 和 **ServiceDefinition.csdef**：是發佈應用程式時需使用的 Azure 特定檔案。 如需詳細資訊，請參閱 [雲端服務]。
   * **deploymentSettings.json**：儲存 Azure PowerShell 部署 Cmdlet 使用的本機設定。
4. 輸入下列命令以新增 Web 角色：

       Add-AzureNodeWebRole

   ![Add-AzureNodeWebRole 命令的輸出][The output of the Add-AzureNodeWebRole command]

   **Add-AzureNodeWebRole** Cmdlet 可建立基本的 Node.js 應用程式。 也可修改 **.csfg** 和 **.csdef** 檔案，以新增新角色的組態項目。

   > [!NOTE]
   > 如果您未指定角色名稱，系統會使用預設名稱。 您可以提供一個名稱做為第一個 Cmdlet 參數： `Add-AzureNodeWebRole MyRole`

Node.js app 是在 **server.js** 檔案中定義，該檔案位於 Web 角色 (預設為 **WebRole1**) 的目錄中。 程式碼如下：

    var http = require('http');
    var port = process.env.port || 1337;
    http.createServer(function (req, res) {
        res.writeHead(200, { 'Content-Type': 'text/plain' });
        res.end('Hello World\n');
    }).listen(port);

此程式碼基本上和 [nodejs.org] 網站上的「Hello World」範例一樣，但它使用雲端環境指派的連接埠號碼。

## <a name="deploy-the-application-to-azure"></a>將應用程式部署至 Azure

> [!NOTE]
> 若要完成此教學課程，您需要 Azure 帳戶。 您可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF)或是[註冊免費帳戶](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A85619ABF)。

### <a name="download-the-azure-publishing-settings"></a>下載 Azure 發佈設定
若要將應用程式部署到 Azure，您必須先下載您 Azure 訂閱的發佈設定。

1. 執行下列 Azure PowerShell Cmdlet：

       Get-AzurePublishSettingsFile

   這將使用瀏覽器瀏覽到發佈設定下載頁面。 可能提示您使用 Microsoft 帳戶登入。 若是如此，請使用與您的 Azure 訂閱相關聯的帳戶。

   將下載的設定檔儲存到您可以輕鬆存取的檔案位置。
2. 執行下列 Cmdlet 來匯入您所下載的發佈設定檔：

       Import-AzurePublishSettingsFile [path to file]

    > [!NOTE]
    > 在匯入發佈設定之後，請考慮刪除下載的 .publishSettings 檔案，因為它包含了可能讓他人存取您帳戶的資訊。

### <a name="publish-the-application"></a>發佈應用程式
若要發佈，請執行下列命令：

      $ServiceName = "NodeHelloWorld" + $(Get-Date -Format ('ddhhmm'))
    Publish-AzureServiceProject -ServiceName $ServiceName  -Location "East US" -Launch

* **-ServiceName** 指定部署的名稱。 這必須是唯一名稱，否則發佈程序將失敗。 **Get-Date** 命令會添加到應該讓名稱唯一的日期/時間字串。
* **-Location** 指定託管應用程式的資料中心。 若要查看可用資料中心的清單，請使用 **Get-AzureLocation** Cmdlet。
* **-Launch** 可在部署完成後，開啟瀏覽器視窗並瀏覽至託管服務。

發佈成功後，您將看見如下的回應：

![Publish-AzureService 命令的輸出][The output of the Publish-AzureService command]

> [!NOTE]
> 第一次發佈應用程式時，從部署到可以使用需要數分鐘的時間。

部署完成後，瀏覽器視窗將開啟，並瀏覽到雲端服務。

![顯示 hello world 頁面的瀏覽器視窗；URL 會指出此頁面裝載在 Azure 上。][A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]

您的應用程式現在成功在 Azure 上執行了！

**Publish-AzureServiceProject** Cmdlet 將執行下列步驟：

1. 建立部署的封裝。 該封裝包含 node.js 應用程式資料夾中所有的檔案。
2. 建立新的 **儲存體帳戶** (如果不存在)。 Azure 儲存體帳戶將用來在部署期間儲存應用程式封裝。 部署完成後，即可安全刪除儲存體帳戶。
3. 建立新的 **雲端服務** (如果不存在)。 **雲端服務** 是應用程式部署到 Azure 時代管應用程式的容器。 如需詳細資訊，請參閱 [雲端服務]。
4. 將部署封裝發佈到 Azure。

## <a name="stopping-and-deleting-your-application"></a>停止並刪除您的應用程式
部署您的應用程式後，您可能會想要將它停用，以便避免額外的成本。 Azure 會對於 Web 角色執行個體的伺服器使用時間時數計費。 部署應用程式之後，即使執行個體未執行而在停止狀態中，也會使用伺服器時間。

1. 在 Windows PowerShell 視窗中，使用下列 Cmdlet 停止上一個小節中建立的服務部署：

       Stop-AzureService

   停止服務可能需要幾分鐘的時間。 服務停止時，您將收到表示停止的訊息。

   ![Stop-AzureService 命令的狀態][The status of the Stop-AzureService command]
2. 若要刪除服務，請呼叫下列 Cmdlet：

       Remove-AzureService

   出現提示時，輸入 **Y** 以刪除服務。

   刪除服務可能需要幾分鐘的時間。 刪除服務後，您將收到表示已刪除服務的訊息。

   ![Remove-AzureService 命令的狀態][The status of the Remove-AzureService command]

   > [!NOTE]
   > 刪除服務不會刪除初次發佈服務時建立的儲存體帳戶，而且將持續對使用的儲存體計費。 如果沒有其他項目正在使用儲存體，您可以將它刪除。

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱 [Node.js 開發人員中心]。

<!-- URL List -->

[Azure 網站、雲端服務與虛擬機器的比較]: ../app-service/choose-web-site-cloud-service-vm.md
[使用輕量型 Web 應用程式]: ../app-service/app-service-web-get-started-nodejs.md
[Azure PowerShell]: /powershell/azureps-cmdlets-docs
[Azure SDK for .NET 2.7]: http://www.microsoft.com/en-us/download/details.aspx?id=48178
[連線 PowerShell]: /powershell/azureps-cmdlets-docs#step-3-connect
[nodejs.org]: http://nodejs.org/
[雲端服務]: https://azure.microsoft.com/documentation/services/cloud-services/
[Node.js 開發人員中心]: https://azure.microsoft.com/develop/nodejs/

<!-- IMG List -->

[The result of the New-AzureService helloworld command]: ./media/cloud-services-nodejs-develop-deploy-app/node9.png
[The output of the Add-AzureNodeWebRole command]: ./media/cloud-services-nodejs-develop-deploy-app/node11.png
[A web browser displaying the Hello World web page]: ./media/cloud-services-nodejs-develop-deploy-app/node14.png
[The output of the Publish-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node19.png
[A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]: ./media/cloud-services-nodejs-develop-deploy-app/node21.png
[The status of the Stop-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node48.png
[The status of the Remove-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node49.png
