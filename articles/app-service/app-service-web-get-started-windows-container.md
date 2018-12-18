---
title: 在 Azure (預覽) 中執行自訂 Windows 容器 | Microsoft Docs
description: 了解如何將自訂 Windows 容器部署至 Azure App Service。
services: app-service\web
documentationcenter: ''
author: cephalin
manager: jeconnoc
editor: ''
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 09/17/2018
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: e2d058cfe6d6a31f557708277902063e51f54bc5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971360"
---
# <a name="run-a-custom-windows-container-in-azure-preview"></a>在 Azure (預覽) 中執行自訂 Windows 容器

[Azure App Service](app-service-web-overview.md) 會在 Windows 上提供預先定義的應用程式堆疊 (例如 ASP.NET 或 Node.js)，執行於 IIS 上。 預先設定的 Windows 環境會鎖定作業系統的系統管理存取、軟體安裝、對全域組件快取的變更等作業 (請參閱 [Azure App Service 上的作業系統功能](web-sites-available-operating-system-functionality.md))。 如果您的應用程式需要的存取超出預先設定的環境所允許的，您可以改為部署自訂 Windows 容器。 此快速入門說明如何將 Windows 映像中的 ASP.NET 應用程式從 Visual Studio 部署到至 [Docker Hub](https://hub.docker.com/)，並在 Azure App Service 中使用自訂容器執行它。

![](media/app-service-web-get-started-windows-container/app-running-vs.png)

## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

- <a href="https://hub.docker.com/" target="_blank">註冊 Docker Hub 帳戶</a>
- <a href="https://docs.docker.com/docker-for-windows/install/" target="_blank">安裝適用於 Windows 的 Docker</a>。
- <a href="https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10#2-switch-to-windows-containers" target="_blank">切換 Docker 以執行 Windows 容器</a>。
- <a href="https://www.visualstudio.com/downloads/" target="_blank">安裝 Visual Studio 2017</a>，記得包含 **ASP.NET 與網頁程式開發**與 **Azure 開發**工作負載。 若您已安裝 Visual Studio 2017：
    - 按一下 [說明] > [檢查更新] 以安裝最新的 Visual Studio 更新。
    - 按一下 [工具] > [取得工具與功能] 以在 Visual Studio 中新增工作負載。

## <a name="create-an-aspnet-web-app"></a>建立 ASP.NET Web 應用程式

在 Visual Studio 中，選取 [檔案] > [新增] > [專案] 以建立專案。 

在 [新增專案] 對話方塊中，選取 [Visual C#] > [Web] > [ASP.NET Web 應用程式 (.NET Framework)]。

將應用程式命名為 _myFirstAzureWebApp_，然後選取 [確定]。
   
![New Project dialog box](./media/app-service-web-get-started-windows-container/new-project.png)

您可以將任何類型的 ASP.NET web 應用程式部署至 Azure。 在本快速入門中，選取 **MVC** 範本，並確定驗證設定為 [不需要驗證]。

選取 [啟用 Docker Compose 支援]。

選取 [確定]。

![[新增 ASP.NET 專案] 對話方塊](./media/app-service-web-get-started-windows-container/select-mvc-template.png)

如果 _Dockerfile_ 檔案沒有自動開啟，請從 [方案總管] 開啟它。

您需要使用[支援的父映像](#use-a-different-parent-image)。 請使用下列程式碼取代 `FROM` 這一行以變更父映像，然後儲存檔案：

```Dockerfile
FROM microsoft/aspnet:4.7.1
```

從功能表中，選取 [偵錯] > [啟動但不偵錯]，以在本機執行 Web 應用程式。

![在本機執行應用程式](./media/app-service-web-get-started-windows-container/local-web-app.png)

## <a name="publish-to-docker-hub"></a>發佈至 Docker Hub

在 [方案總管] 中，以滑鼠右鍵按一下 **myFirstAzureWebApp** 專案，然後選取 [發佈]。

![從方案總管發佈](./media/app-service-web-get-started-windows-container/solution-explorer-publish.png)

發佈精靈會自動啟動。 選取 [容器登錄] > [Docker Hub] > [發佈]。

![從專案概觀頁面發佈](./media/app-service-web-get-started-windows-container/publish-to-docker.png)

提供 Docker Hub 帳戶認證並按一下 [儲存]。 

等待部署完成。 [發佈] 頁面現在會顯示您稍後將在 App Service 中使用的存放庫名稱。

![從專案概觀頁面發佈](./media/app-service-web-get-started-windows-container/published-docker-repository.png)

複製此存放庫名稱以便稍後使用。

## <a name="sign-in-to-azure"></a>登入 Azure

在 https://portal.azure.com 登入 Azure 入口網站。

## <a name="create-a-windows-container-app"></a>建立 Windows 容器應用程式

1. 選擇 Azure 入口網站左上角的 [建立資源]。

2. 在 Azure Marketplace 資源清單上方的搜尋方塊中，搜尋並選取 [適用於容器的 Web 應用程式]。

3. 提供應用程式名稱 (例如 *win-container-demo*)，接受預設值以建立新的資源群組，然後按一下 [作業系統] 方塊中的 [Windows (預覽)]。

    ![](media/app-service-web-get-started-windows-container/portal-create-page.png)

4. 按一下 [App Service 方案/位置] > [新建]，以建立 App Service 方案。 為新方案指定名稱，並接受預設值，然後按一下 [確定]。

    ![](media/app-service-web-get-started-windows-container/portal-create-plan.png)

5. 按一下 [設定容器]。 在 [映像與選擇性標記] 中，使用在[發佈至 Docker Hub](#publish-to-docker-hub) 中複製的存放庫名稱，然後按一下 [確定]。

    ![](media/app-service-web-get-started-windows-container/portal-configure-container-vs.png)

    如果您在他處有用於 Web 應用程式的自訂映像 (例如，在 [Azure Container Registry](/azure/container-registry/) 中或任何其他私人存放庫中)，您可以在此處加以設定。

6. 按一下 [建立]，並等候 Azure 建立所需的資源。

## <a name="browse-to-the-container-app"></a>瀏覽至容器應用程式

Azure 作業完成時，會顯示通知方塊。

![](media/app-service-web-get-started-windows-container/portal-create-finished.png)

1. 按一下 [前往資源]。

2. 在應用程式頁面中，按一下 [URL] 下方的連結。

新的瀏覽器頁面隨即開啟，並顯示下列頁面：

![](media/app-service-web-get-started-windows-container/app-starting.png)

在幾分鐘後再試一次，直到您看到預設的 ASP.NET 首頁：

![](media/app-service-web-get-started-windows-container/app-running-vs.png)

**恭喜！** 您正在 Azure App Service 中執行您的第一個自訂 Windows 容器。

## <a name="see-container-start-up-logs"></a>檢視容器啟動記錄

Windows 容器載入可能需要一些時間。 若要查看進度，請將 *\<app_name>* 取代為您的應用程式名稱，以瀏覽至下列 URL。
```
https://<app_name>.scm.azurewebsites.net/api/logstream
```

串流處理的記錄會如下所示：

```
2018-07-27T12:03:11  Welcome, you are now connected to log-streaming service.
27/07/2018 12:04:10.978 INFO - Site: win-container-demo - Start container succeeded. Container: facbf6cb214de86e58557a6d073396f640bbe2fdec88f8368695c8d1331fc94b
27/07/2018 12:04:16.767 INFO - Site: win-container-demo - Container start complete
27/07/2018 12:05:05.017 INFO - Site: win-container-demo - Container start complete
27/07/2018 12:05:05.020 INFO - Site: win-container-demo - Container started successfully
```

## <a name="update-locally-and-redeploy"></a>在本機更新和重新部署

從 [方案總管]，開啟 _Views\Home\Index.cshtml_。

尋找頂端附近的 `<div class="jumbotron">` HTML 標籤，並以下列程式碼取代整個元素︰

```HTML
<div class="jumbotron">
    <h1>ASP.NET in Azure!</h1>
    <p class="lead">This is a simple app that we’ve built that demonstrates how to deploy a .NET app to Azure App Service.</p>
</div>
```

若要重新部署至 Azure，請在 [方案總管] 中，以滑鼠右鍵按一下 **myFirstAzureWebApp** 專案，然後選取 [發佈]。

在 [發佈] 頁面上，選取 [發佈] 並等候發佈完成。

若要告知 App Service 從 Docker Hub 提取新映像，請重新啟動應用程式。 返回入口網站的應用程式頁面，按一下 [重新啟動] > [是]。

![在 Azure 中重新啟動 Web 應用程式](./media/app-service-web-get-started-windows-container/portal-restart-app.png)

再次[瀏覽至容器應用程式](#browse-to-the-container-app)。 當您重新整理網頁時，應用程式應該會先還原成 [啟動] 頁面，接著在幾分鐘後再次顯示更新的網頁。

![在 Azure 中更新 Web 應用程式](./media/app-service-web-get-started-windows-container/azure-web-app-updated.png)

## <a name="use-a-different-parent-image"></a>使用不同的父映象

您可以自由使用不同的自訂 Docker 映像來執行應用程式。 不過，您必須為您要的架構選擇正確的[父映像](https://docs.docker.com/develop/develop-images/baseimages/)： 

- 若要部署 .NET Framework 應用程式，請根據 Windows Server Core 2016 [長期維護通道 (LTSC)](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview#long-term-servicing-channel-ltsc) 版本使用正確的父映像。 
- 若要部署 .NET Core 應用程式，請根據 Windows Server Nano 2016 [長期維護通道 (LTSC)](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview#long-term-servicing-channel-ltsc) 版本使用正確的父映像。 

在應用程式啟動期間，下載父映像需要一些時間。 不過，您可以使用下列其中一個已在 Azure App Service 中快取的父映像，以縮短啟動時間：

- [microsoft/iis](https://hub.docker.com/r/microsoft/iis/):windowsservercore-ltsc2016 (最新)
- [microsoft/iis](https://hub.docker.com/r/microsoft/iis/):nanoserver-sac2016
- [microsoft/aspnet](https://hub.docker.com/r/microsoft/aspnet/):4.7.2-windowsservercore-ltsc2016, 4.7.2 (最新)
- [microsoft/dotnet](https://hub.docker.com/r/microsoft/dotnet/):2.1-aspnetcore-runtime
- [microsoft/dotnet](https://hub.docker.com/r/microsoft/dotnet/):2.1-sdk

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [在 Azure 中移轉至 Windows 容器](app-service-web-tutorial-windows-containers-custom-fonts.md)
