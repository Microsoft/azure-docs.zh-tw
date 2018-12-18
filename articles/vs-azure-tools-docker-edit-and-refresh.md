---
title: 偵錯本機 Docker 容器中的應用程式 | Microsoft Docs
description: 了解如何透過「編輯和重新整理」來修改在本機 Docker 容器中執行的應用程式、重新整理容器，以及設定偵錯中斷點
services: azure-container-service
documentationcenter: na
author: ghogen
manager: douge
editor: ''
ms.assetid: 480e3062-aae7-48ef-9701-e4f9ea041382
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 07/22/2016
ms.author: mlearned
ms.openlocfilehash: 01741ba25ac3a6ab187a08636b7e26ee58dbee90
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038065"
---
# <a name="debugging-apps-in-a-local-docker-container"></a>偵錯本機 Docker 容器中的應用程式
## <a name="overview"></a>概觀
Visual Studio 2017 可讓您以一致的方式在 Linux Docker 容器中進行開發，並在本機驗證應用程式。
您不需要每次變更程式碼之後都重新啟動容器。
本文章說明如何使用「編輯及重新整理」功能在本機 Docker 容器中啟動 ASP.NET Core Web 應用程式、進行任何必要的變更，然後重新整理瀏覽器來查看這些變更。
本文章也會示範如何設定中斷點來進行偵錯。

> [!NOTE]
> 未來版本將支援 Windows 容器
>
>

## <a name="prerequisites"></a>必要條件
您的電腦必須安裝下列工具。

* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [Microsoft ASP.NET Core 1.0 SDK](https://go.microsoft.com/fwlink/?LinkID=809122)

若要在本機執行 Docker 容器，您需要本機 Docker 用戶端。
您可以使用需要停用 Hyper-V 的 [Docker 工具箱](https://www.docker.com/products/docker-toolbox)，或是使用利用 Hyper-V 並需要 Windows 10 的 [Docker for Windows](https://www.docker.com/get-docker)。

如果使用 Docker 工具箱，您將需要 [設定 Docker 用戶端](vs-azure-tools-docker-setup.md)

## <a name="1-create-a-web-app"></a>1.建立 Web 應用程式
[!INCLUDE [create-aspnet5-app](../includes/create-aspnet5-app.md)]

## <a name="2-add-docker-support"></a>2.新增 Docker 支援
[!INCLUDE [Add docker support](../includes/vs-azure-tools-docker-add-docker-support.md)]

## <a name="3-edit-your-code-and-refresh"></a>3.編輯您的程式碼並重新整理
若要快速反覆查看變更，您可以啟動容器內的應用程式，並繼續進行變更，而檢視變更的方式與使用 IIS Express 相同。

1. 將 [方案組態] 設定為 `Debug`，然後按 **&lt;CTRL + F5>** 來建置 Docker 映像，然後在本機執行此映像。

    建置好容器映像並在 Docker 容器中執行之後，Visual Studio 就會在您的預設瀏覽器中啟動 Web 應用程式。
    如果您是使用 Microsoft Edge 瀏覽器或者發生錯誤，請參閱 [疑難排解](vs-azure-tools-docker-troubleshooting-docker-errors.md) 一節。
2. 前往「關於」頁面，這是我們要在其中進行變更的頁面。
3. 返回 Visual Studio，然後開啟 `Views\Home\About.cshtml`。
4. 將下列 HTML 內容新增到檔案結尾，然後儲存變更。

    ```
    <h1>Hello from a Docker Container!</h1>
    ```
5. 檢視輸出視窗，當 .NET 組建完成且您看到這些行時，切換回您的瀏覽器，然後重新整理「關於」頁面。

   ```
   Now listening on: http://*:80
   Application started. Press Ctrl+C to shut down
   ```
6. 您的變更已經套用！

## <a name="4-debug-with-breakpoints"></a>4.使用中斷點進行偵錯
通常，需要運用 Visual Studio 的偵錯功能進一步檢查變更。

1. 返回 Visual Studio，然後開啟 `Controllers\HomeController.cs`
2. 將 About() 方法的內容取代為下列內容：

   ```
   string message = "Your application description page from within a Container";
   ViewData["Message"] = message;
   ````
3. 在 `string message`... 行的左側設定中斷點。
4. 按 **&lt;F5>** 開始偵錯。
5. 瀏覽至 [關於] 頁面，來叫用中斷點。
6. 切換至 Visual Studio 來檢視中斷點，然後檢查訊息的值。

   ![][2]

## <a name="summary"></a>總結
透過 Visual Studio 2017 中的 Docker 支援，您將可獲得在本機運作的生產力，以及在 Docker 容器內開發的生產環境真實性。

## <a name="troubleshooting"></a>疑難排解
[疑難排解 Visual Studio Docker 開發](vs-azure-tools-docker-troubleshooting-docker-errors.md)

## <a name="more-about-docker-with-visual-studio-windows-and-azure"></a>進一步了解 Docker 與 Visual Studio、Windows 和 Azure
* [Docker Tools for Visual Studio Team Services](http://aka.ms/dockertoolsforvsts) - 建置和部署 Docker 容器
* [Docker Tools for Visual Studio Code](http://aka.ms/dockertoolsforvscode) - 用於編輯 Docker 檔案的語言服務，將推出更多其他 e2e 案例
* [Windows 容器資訊](http://aka.ms/containers)- Windows Server 和 Nano Server 資訊
* [Azure Container Service](https://azure.microsoft.com/services/container-service/) - [Azure Container Service 內容](http://aka.ms/AzureContainerService)
* 如需更多使用 Docker 的範例，請參閱 [HealthClinic.biz](https://github.com/Microsoft/HealthClinic.biz) 2015 連線[示範](https://blogs.msdn.microsoft.com/visualstudio/2015/12/08/connectdemos-2015-healthclinic-biz/)的[使用 Docker](https://github.com/Microsoft/HealthClinic.biz/wiki/Working-with-Docker)。 如需來自 HealthClinic.biz 示範的更多快速入門，請參閱 [Azure 開發人員工具快速入門](https://github.com/Microsoft/HealthClinic.biz/wiki/Azure-Developer-Tools-Quickstarts)。

## <a name="various-docker-tools"></a>各種 Docker 工具
[一些不錯的 Docker 工具 (Steve Lasker 的部落格)](https://blogs.msdn.microsoft.com/stevelasker/2016/03/25/some-great-docker-tools/)

## <a name="good-articles"></a>不錯的文章
[NGINX 的微服務簡介](https://www.nginx.com/blog/introduction-to-microservices/)

## <a name="presentations"></a>簡報
* [Steve Lasker：VS Live Las Vegas 2016 - Docker e2e](https://github.com/SteveLasker/Presentations/blob/master/VSLive2016/Vegas/)
* [ASP.NET Core @ build 2016 簡介 - 示範](https://channel9.msdn.com/Events/Build/2016/B810)
* [開發容器中的 .NET 應用程式 (Channel 9)](https://blogs.msdn.microsoft.com/stevelasker/2016/02/19/developing-asp-net-apps-in-docker-containers/)

[2]: ./media/vs-azure-tools-docker-edit-and-refresh/breakpoint.png
