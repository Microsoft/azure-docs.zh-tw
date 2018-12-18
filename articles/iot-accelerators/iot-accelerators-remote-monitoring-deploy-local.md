---
title: 本機部署遠端監視解決方案 - Azure | Microsoft Docs
description: 此操作指南會示範如何將遠端監視解決方案加速器部署到本機電腦，以進行測試和開發。
author: asdonald
manager: timlt
ms.author: asdonald
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 09/26/2018
ms.topic: conceptual
ms.openlocfilehash: 7007b1406dbcfab3af4700418ac2ce07b9e521c0
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407428"
---
# <a name="deploy-the-remote-monitoring-solution-accelerator-locally"></a>本機部署遠端監視解決方案加速器

此文章會示範如何將遠端監視解決方案加速器部署到本機電腦，以進行測試和開發。 此文章中描述的方法會將微服務部署到本機 Docker 容器，並在雲端中使用 IoT 中樞、Cosmos DB 和 Azure 時間序列深入解析服務。 若要了解如何在您本機電腦上的 IDE 中執行遠端監視解決方案，請參閱 GitHub 上的[在本機環境啟動微服務](https://github.com/Azure/remote-monitoring-services-java/blob/master/docs/LOCAL_DEPLOYMENT.md) \(英文\)。

## <a name="prerequisites"></a>必要條件

若要部署遠端監視解決方案加速器所使用的 Azure 服務，您需要一個有效的 Azure 訂用帳戶。

如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資料，請參閱 [Azure 免費試用](http://azure.microsoft.com/pricing/free-trial/)。

若要完成本機部署，您必須在本機開發機器上安裝下列工具：

* [Git](https://git-scm.com/)
* [Docker](https://www.docker.com)
* [Docker Compose](https://docs.docker.com/compose/install/)
* [Node.js v8](https://nodejs.org/) - 此軟體是指令碼用來建立 Azure 資源之 PCS CLI 的先決條件。 不要使用 Node.js v10。

> [!NOTE]
> 這些工具可在許多平台上取得，包括 Windows、Linux 及 iOS。

### <a name="download-the-source-code"></a>下載原始程式碼

遠端監視原始程式碼 GitHub 存放庫包含您在下載、設定及執行內含微服務的 Docker 映像時所需的 Docker 設定檔。 若要複製及建立本機版本的存放庫，請使用您的命令列環境來瀏覽到您本機電腦上的適當資料夾，然後執行下列其中一個命令：

若要下載最新版本的 Java 微服務實作，請執行：

```cmd/sh
git clone https://github.com/Azure/remote-monitoring-services-java.git
```

若要下載最新版本的 .NET 微服務實作，請執行：

```cmd\sh
git clone https://github.com/Azure/remote-monitoring-services-dotnet.git
```

> [!NOTE]
> 除了您用來在本機執行微服務的指令碼之外，這些命令還會下載所有微服務的原始程式碼。 雖然您在 Docker 中執行微服務時不需要原始程式碼，但如果您稍後打算修改解決方案加速器並在本機測試您的變更，原始程式碼就會很有用。

## <a name="deploy-the-azure-services"></a>部署 Azure 服務

雖然此文章會示範如何在本機執行微服務，但它們會仰賴雲端中執行的 Azure 服務。 您可以[以手動方式透過 Azure 入口網站](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Manual-steps-to-create-azure-resources-for-local-setup) \(英文\) 或使用提供的指令碼部署這些 Azure 服務。 下列指令碼範例假設您在 Windows 電腦上使用 .NET 存放庫。 若您是在另一個環境作業，請適當地調整路徑、檔案副檔名與路徑分隔符號。 使用提供的指令碼來：

### <a name="create-new-azure-resources"></a>建立新的 Azure 資源

若您尚未建立必要的 Azure 資源，請依照下列步驟執行：

1. 在您的命令列環境中，瀏覽到您複製之存放庫複本的 **remote-monitoring-services-dotnet\scripts\local\launch** 資料夾。

2. 執行 **start.cmd** 指令碼並依照提示執行。 指令碼會提示您登入您的 Azure 帳戶並重新啟動指令碼。 指令碼接著會提示您輸入下列資訊：
    * 解決方案名稱。
    * 要使用的 Azure 訂用帳戶。
    * 要使用之 Azure 資料中心的位置。

    指令碼會使用您的解決方案名稱在 Azure 中建立資源群組。 此資源群組包含解決方案加速器使用的 Azure 資源。

3. 當指令碼完成時，它會顯示環境變數清單。 依照命令輸出中的指示將這些變數儲存到 **remote-monitoring-services-dotnet\\scripts\\local\\.env** 檔案。

### <a name="use-existing-azure-resources"></a>使用現有的 Azure 資源

若您已建立必要的 Azure 資源，請將 **remote-monitoring-services-dotnet\\scripts\\local\\.env** 檔案中的環境變數定義編輯為必要的值。 **.env** 檔案包含有關要到哪裡尋找必要值的詳細資訊。

## <a name="run-the-microservices-in-docker"></a>在 Docker 中執行微服務

在本機 Docker 容器中執行的微服務必須能夠存取在 Azure 中執行的服務。 您可以使用下列命令來測試您 Docker 環境的網際網路連線，該命令會啟動一個小型容器並嘗試 Ping 網際網路位址：

```cmd/sh
docker run --rm -ti library/alpine ping google.com
```

若要執行解決方案加速器，請在您的命令列環境中瀏覽到 **remote-monitoring-services-dotnet\\scripts\\local** 資料夾，然後執行下列命令：

```cmd\sh
docker-compose up
```

您第一次執行此命令時，Docker 會下載 Docker 中樞的微服務映像，以在本機建置容器。 在後續執行時，Docker 則會立即執行容器。

您可以使用個別殼層來檢視容器中的記錄。 請先使用 `docker ps -a` 命令來尋找容器識別碼。 然後使用 `docker logs {container-id} --tail 1000` 來檢視指定容器的最後 1000 個記錄項目。

若要存取遠端監視解決方案儀表板，請在瀏覽器中瀏覽至 [http://localhost:8080](http://localhost:8080)。

## <a name="clean-up"></a>清除

為避免不必要的費用，當您完成測試時，請從您的 Azure 訂用帳戶中移除雲端服務。 移除服務的最簡單方式是瀏覽到 [Azure 入口網站](https://ms.portal.azure.com)，然後刪除當您執行 **start.cmd** 指令碼時建立的資源群組。

使用 `docker-compose down --rmi all` 命令來移除 Docker 映像，並釋放您本機電腦上的空間。 您也可以將您從 GitHub 複製原始程式碼時，所建立的遠端監視存放庫本機副本刪除。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定本機開發環境
> * 設定解決方案加速器
> * 部署解決方案加速器
> * 登入解決方案加速器

既然您已部署遠端監視解決方案，下一步便是[探索解決方案儀表板的功能](quickstart-remote-monitoring-deploy.md)。

<!-- Next tutorials in the sequence -->
