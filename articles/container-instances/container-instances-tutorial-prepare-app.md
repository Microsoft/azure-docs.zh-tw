---
title: Azure Container Instances 教學課程 - 準備您的應用程式
description: Azure 容器執行個體教學課程第 1 部分 (共 3 部分) - 準備應用程式以部署至 Azure 容器執行個體
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: tutorial
ms.date: 03/21/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: dfe494eba64f546616e4e6cc9898f9d63d81eaa2
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854774"
---
# <a name="tutorial-create-container-for-deployment-to-azure-container-instances"></a>教學課程：建立容器以部署至 Azure Container Instances

Azure Container Instances 能夠將 Docker 容器部署至 Azure 基礎結構，而不需要佈建任何虛擬機器，或採用較高層級的服務。 在本教學課程中，您會將小型 Node.js Web 應用程式封裝在可使用 Azure Container Instances 執行的容器中。

在本文 (本系列的第 1 部分) 中，您將：

> [!div class="checklist"]
> * 從 GitHub 複製應用程式原始程式碼
> * 從應用程式來源建立容器映像
> * 在本機 Docker 環境中測試映像

在教學課程的第 2 和第 3 部分中，您會將映像上傳至 Azure Container Registry，然後部署至 Azure Container Instances。

## <a name="before-you-begin"></a>開始之前

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="get-application-code"></a>取得應用程式程式碼

本教學課程中的範例應用程式是一個以 [Node.js][nodejs] 建置的簡單 Web 應用程式。 應用程式會提供靜態 HTML 頁面，類似以下螢幕擷取畫面︰

![在瀏覽器中顯示的教學課程應用程式][aci-tutorial-app]

使用 Git 複製範例應用程式的存放庫：

```bash
git clone https://github.com/Azure-Samples/aci-helloworld.git
```

您也可以直接從 GitHub [下載 ZIP 封存檔][aci-helloworld-zip]。

## <a name="build-the-container-image"></a>建置容器映像

範例應用程式中的 Dockerfile 會示範如何建置容器。 首先會使用以 [Alpine Linux][alpine-linux] 為基礎的[官方 Node.js 映像][docker-hub-nodeimage]，這是一個很適合用於容器的小型散發。 接著會將應用程式檔案複製到容器、使用節點套件管理員來安裝相依性，最後就啟動應用程式。

```Dockerfile
FROM node:8.9.3-alpine
RUN mkdir -p /usr/src/app
COPY ./app/ /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
CMD node /usr/src/app/index.js
```

使用 [docker build][docker-build] 命令來建立容器映像，並標記成 aci-tutorial-app：

```bash
docker build ./aci-helloworld -t aci-tutorial-app
```

[docker build][docker-build] 命令的輸出與以下類似 (為便於閱讀已將輸出內容截斷)：

```console
$ docker build ./aci-helloworld -t aci-tutorial-app
Sending build context to Docker daemon  119.3kB
Step 1/6 : FROM node:8.9.3-alpine
8.9.3-alpine: Pulling from library/node
88286f41530e: Pull complete
84f3a4bf8410: Pull complete
d0d9b2214720: Pull complete
Digest: sha256:c73277ccc763752b42bb2400d1aaecb4e3d32e3a9dbedd0e49885c71bea07354
Status: Downloaded newer image for node:8.9.3-alpine
 ---> 90f5ee24bee2
...
Step 6/6 : CMD node /usr/src/app/index.js
 ---> Running in f4a1ea099eec
 ---> 6edad76d09e9
Removing intermediate container f4a1ea099eec
Successfully built 6edad76d09e9
Successfully tagged aci-tutorial-app:latest
```

使用 [docker images][docker-images] 命令查看已建置的映像：

```bash
docker images
```

您新建置的映像應該會出現在清單中：

```console
$ docker images
REPOSITORY          TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app    latest    5c745774dfa9    39 seconds ago    68.1 MB
```

## <a name="run-the-container-locally"></a>在本機執行容器

將容器部署至 Azure Container Instances 之前，請先使用 [docker run][docker-run] 在本機執行它並確認可以運作。 `-d` 參數可讓容器在背景中執行，而 `-p` 可讓您將電腦上的任意連接埠對應至容器中的連接埠 80。

```bash
docker run -d -p 8080:80 aci-tutorial-app
```

如果 `docker run` 命令執行成功，此命令的輸出會顯示執行中容器的 ID：

```console
$ docker run -d -p 8080:80 aci-tutorial-app
a2e3e4435db58ab0c664ce521854c2e1a1bda88c9cf2fcff46aedf48df86cccf
```

現在，在瀏覽器中瀏覽至 http://localhost:8080 ，以確認容器正在執行。 您應該會看到如下所示的網面：

![在本機瀏覽器中執行應用程式][aci-tutorial-app-local]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立可部署在 Azure Container Instances 中的容器映像，並確認它可在本機執行。 到目前為止，您已完成下列作業：

> [!div class="checklist"]
> * 從 GitHub 複製應用程式來源
> * 從應用程式來源建立容器映像
> * 在本機測試容器

前往本系列中的下一個教學課程，了解如何在 Azure Container Registry 中儲存容器映像：

> [!div class="nextstepaction"]
> [將映像推送至 Azure Container Registry](container-instances-tutorial-prepare-acr.md)

<!--- IMAGES --->
[aci-tutorial-app]:./media/container-instances-quickstart/aci-app-browser.png
[aci-tutorial-app-local]: ./media/container-instances-tutorial-prepare-app/aci-app-browser-local.png

<!-- LINKS - External -->
[aci-helloworld-zip]: https://github.com/Azure-Samples/aci-helloworld/archive/master.zip
[alpine-linux]: https://alpinelinux.org/
[docker-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-hub-nodeimage]: https://store.docker.com/images/node
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[nodejs]: http://nodejs.org

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
