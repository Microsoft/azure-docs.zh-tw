---
title: 在 Azure 中的 Service Fabric 上建立容器映像 | Microsoft Docs
description: 在本教學課程中，了解如何建立多容器 Service Fabric 應用程式的容器映像。
services: service-fabric
documentationcenter: ''
author: suhuruli
manager: timlt
editor: suhuruli
tags: servicefabric
keywords: Docker、容器、微服務、Service Fabric、Azure
ms.assetid: ''
ms.service: service-fabric
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/15/2017
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: 0dc92524843f47b1327af5e9d3f237656cad2a37
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46947528"
---
# <a name="tutorial-create-container-images-on-a-linux-service-fabric-cluster"></a>教學課程：在 Linux Service Fabric 叢集上建立容器映像

本教學課程是一個教學課程系列的第一部分，示範如何在 Linux Service Fabric 叢集中使用容器。 在本教學課程中，多容器應用程式已準備好可與 Service Fabric 搭配使用。 在後續的教學課程中，這些映像可用來作為 Service Fabric 應用程式的一部分。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 從 GitHub 複製應用程式來源
> * 從應用程式來源建立容器映像
> * 部署 Azure Container Registry (ACR) 執行個體
> * 標記 ACR 的容器映像
> * 將映像上傳至 ACR

在本教學課程系列中，您將了解如何：

> [!div class="checklist"]
> * 建立 Service Fabric 的容器映像
> * [建置與執行含容器的 Service Fabric 應用程式](service-fabric-tutorial-package-containers.md)
> * [如何在 Service Fabric 中處理容錯移轉和調整](service-fabric-tutorial-containers-failover.md)

## <a name="prerequisites"></a>必要條件

* 已針對 Service Fabric 設定的 Linux 開發環境。 請依照[這裡](service-fabric-get-started-linux.md)的指示來設定 Linux 環境。
* 本教學課程需要您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。
* 此外，它需要您具有可用的 Azure 訂用帳戶。 如需免費試用版的詳細資訊，請移至[這裡](https://azure.microsoft.com/free/)。

## <a name="get-application-code"></a>取得應用程式程式碼

本教學課程中使用的範例應用程式是投票應用程式。 應用程式是由前端 Web 元件和後端 Redis 執行個體所組成。 元件已封裝為容器映像。

使用 git 將應用程式的複本下載至您的開發環境。

```bash
git clone https://github.com/Azure-Samples/service-fabric-containers.git

cd service-fabric-containers/Linux/container-tutorial/
```

方案包含兩個資料夾和 'docker-compose.yml' 檔案。 'azure-vote' 資料夾包含 Python 前端服務，以及用來建置映像的 Dockerfile。 'Voting' 目錄包含部署至叢集的 Service Fabric 應用程式封裝。 這些目錄包含本教學課程所需的資產。

## <a name="create-container-images"></a>建立容器映像

在 **azure-vote** 目錄內，執行下列命令以建置前端 Web 元件的映像。 此命令會使用此目錄中的 Dockerfile 來建置映像。

```bash
docker build -t azure-vote-front .
```
> [!Note]
> 如果您遇到權限遭拒的狀況，請依照[這份](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)文件的說明使用 Docker，而不使用 sudo。

此命令可能需要一些時間，因為必須從 Docker Hub 提取所有必要的相依性。 完成時，使 [docker images](https://docs.docker.com/engine/reference/commandline/images/) 命令來查看所建立的映像。

```bash
docker images
```

請注意，已下載或建立兩個映像。 azure-vote-front 映像包含應用程式。 它衍生自 Docker Hub 的 python 映像。

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
azure-vote-front             latest              052c549a75bf        About a minute ago   708MB
tiangolo/uwsgi-nginx-flask   python3.6           590e17342131        5 days ago           707MB

```

## <a name="deploy-azure-container-registry"></a>部署 Azure Container Registry

先執行 **az login** 命令來登入您的 Azure 帳戶。

```bash
az login
```

接下來，使用 **az account** 命令，選擇您的訂用帳戶來建立 Azure 容器登錄。 您必須輸入您 Azure 訂用帳戶的訂用帳戶識別碼來取代 <subscription_id>。

```bash
az account set --subscription <subscription_id>
```

部署 Azure Container Registry 時，您必須先有資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

使用 **az group create** 命令來建立資源群組。 在此範例中，會在 westus 區域中建立名為 myResourceGroup 的資源群組。

```bash
az group create --name <myResourceGroup> --location westus
```

使用 **az acr create** 命令來建立 Azure Container Registry。 使用您想要在訂用帳戶下建立的容器登錄名稱取代 \<acrName>。 此名稱必須是英數字元，而且是唯一的。

```bash
az acr create --resource-group <myResourceGroup> --name <acrName> --sku Basic --admin-enabled true
```

在本教學課程的其餘部分，我們使用 "acrName" 作為您選擇之容器登錄名稱的預留位置。 請記住這個值。

## <a name="log-in-to-your-container-registry"></a>登入您的容器登錄

請登入您的 ACR 執行個體，再將映像推送到它。 使用 **az acr login** 命令來完成此作業。 在建立容器登錄時，為它提供唯一名稱。

```bash
az acr login --name <acrName>
```

此命令在完成之後會傳回「登入成功」訊息。

## <a name="tag-container-images"></a>標記容器映像

每個容器映像都必須標記登錄的 loginServer 名稱。 將容器映像推送到映像登錄時，此標籤可用於路由傳送。

若要查看目前的映像清單，請使用 [docker images](https://docs.docker.com/engine/reference/commandline/images/) 命令。

```bash
docker images
```

輸出：

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
azure-vote-front             latest              052c549a75bf        About a minute ago   708MB
tiangolo/uwsgi-nginx-flask   python3.6           590e17342131        5 days ago           707MB
```

若要取得 loginServer 名稱，請執行下列命令：

```bash
az acr show --name <acrName> --query loginServer --output table
```

這會輸出如下列結果的表格。 此結果將會用來標記您的 **azure-vote-front** 映像，然後才將映像推送到下一個步驟中的容器登錄。

```bash
Result
------------------
<acrName>.azurecr.io
```

現在，以您容器登錄的 loginServer 標記 *azure-vote-front* 映像。 此外，將 `:v1` 新增至映像名稱的結尾。 此標籤指示映像版本。

```bash
docker tag azure-vote-front <acrName>.azurecr.io/azure-vote-front:v1
```

標記之後，執行 'docker images' 來驗證作業。

輸出：

```bash
REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
azure-vote-front                       latest              052c549a75bf        23 minutes ago      708MB
<acrName>.azurecr.io/azure-vote-front   v1                  052c549a75bf       23 minutes ago      708MB
tiangolo/uwsgi-nginx-flask             python3.6           590e17342131        5 days ago          707MB

```

## <a name="push-images-to-registry"></a>將映像推送到登錄

將 *azure-vote-front* 映像推送到登錄。 

使用下列範例，以您環境中的 loginServer 取代 ACR loginServer 名稱。

```bash
docker push <acrName>.azurecr.io/azure-vote-front:v1
```

Docker push 命令需要數分鐘才能完成。

## <a name="list-images-in-registry"></a>列出登錄中的映像

若要傳回已推送至 Azure Container Registry 的映像清單，請使用 [az acr repository list](/cli/azure/acr/repository#az_acr_repository_list) 命令。 以 ACR 執行個體名稱更新命令。

```bash
az acr repository list --name <acrName> --output table
```

輸出：

```bash
Result
----------------
azure-vote-front
```

當教學課程完成時，私人 Azure Container Registry 執行個體中已儲存容器映像。 在後續的教學課程中，會將此映像從 ACR 部署到 Service Fabric 叢集。

## <a name="next-steps"></a>後續步驟

本教學課程會從 Github 提取應用程式，然後建立容器映像並推送到登錄。 已完成下列步驟：

> [!div class="checklist"]
> * 從 GitHub 複製應用程式來源
> * 從應用程式來源建立容器映像
> * 部署 Azure Container Registry (ACR) 執行個體
> * 標記 ACR 的容器映像
> * 將映像上傳至 ACR

前進到下一個教學課程，以了解如何使用 Yeoman，將容器封裝到 Service Fabric 應用程式。

> [!div class="nextstepaction"]
> [封裝和部署容器作為 Service Fabric 應用程式](service-fabric-tutorial-package-containers.md)