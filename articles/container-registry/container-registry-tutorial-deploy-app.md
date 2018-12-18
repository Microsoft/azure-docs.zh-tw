---
title: Azure Container Registry 教學課程 - 從 Azure Container Registry 部署 Web 應用程式
description: 使用異地複寫的 Azure Container Registry 中的容器映像，部署以 Linux 為基礎的 Web 應用程式。 這是三段式教學課程的第二段。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: tutorial
ms.date: 08/20/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: d1924d730f2c2a64063496d4be759194cef99175
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48853856"
---
# <a name="tutorial-deploy-web-app-from-azure-container-registry"></a>教學課程：從 Azure Container Registry 部署 Web 應用程式

這是三段式教學課程系列的第二段。 在[第一段](container-registry-tutorial-prepare-registry.md)中，已建立異地複寫的容器登錄，也已從來源建立容器映像並推送至登錄中。 在本文中，您要將容器部署至兩個不同 Azure 區域中的 Web 應用程式執行個體，以便獲得異地複寫的登錄因網路位置接近的好處。 接著，每個執行個體將從最接近的登錄提取容器映像。

在本教學課程中，系列的第二段：

> [!div class="checklist"]
> * 將容器映像部署到兩個*用於容器的 Web 應用程式*執行個體
> * 確認部署的應用程式

如果您尚未建立異地複寫的登錄，並將容器化的應用程式範例映像推送至登錄中，請返回系列的上一個教學課程，[準備異地複寫的 Azure Container Registry](container-registry-tutorial-prepare-registry.md)。

在系列的下一篇文章中，您要更新應用程式，然後將更新的容器映像推送至登錄中。 最後，您要瀏覽至每個執行的 Web 應用程式執行個體，查看兩者當中皆已自動反映變更，顯示出 Azure Container Registry 異地複寫和 Webhook 起作用。

## <a name="automatic-deployment-to-web-apps-for-containers"></a>自動部署至用於容器的 Web 應用程式

Azure Container Registry 支援將容器化的應用程式直接部署至[用於容器的 Web 應用程式](../app-service/containers/index.yml)。 在本教學課程中，您要使用 Azure 入口網站，將上一個教學課程中建立的容器映像，部署至位於不同 Azure 區域的兩個 Web 應用程式方案。

當您從登錄中的容器映像部署 Web 應用程式，而且您在相同的區域中有異地複寫的登錄時，Azure Container Registry 會為您建立映像部署 [Webhook](container-registry-webhook.md)。 當您將新的映像推送至您的容器存放庫時，Webhook 會挑出變更，並自動將新的容器映像部署到您的 Web 應用程式。

## <a name="deploy-a-web-app-for-containers-instance"></a>部署用於容器的 Web 應用程式執行個體

在此步驟中，您要在「美國西部」區域中建立用於容器的 Web 應用程式執行個體。

登入 [Azure 入口網站](https://portal.azure.com)並瀏覽至您在上一個教學課程中建立的登錄。

選取 [存放庫] > [acr-helloworld]，然後在 [標記] 下的 [v1] 標記上按一下滑鼠右鍵，接著選取 [部署至 Web 應用程式]：

![部署至 Azure 入口網站中的 App Service][deploy-app-portal-01]

如果 [部署至 Web 應用程式] 停用，表示您可能未依照第一個教學課程的[建立容器登錄](container-registry-tutorial-prepare-registry.md#create-a-container-registry)中的指示來啟用登錄管理員使用者。 您可以在 Azure 入口網站的 [設定] > [存取金鑰] 中啟用管理員使用者。

在您選取 [部署至 Web 應用程式] 之後所顯示的 [適用於容器的 Web 應用程式] 下方，為每個設定指定下列值：

| 設定 | 值 |
|---|---|
| **網站名稱** | Web 應用程式的全域唯一名稱。 在此範例中，我們使用格式 `<acrName>-westus` 以便更容易找出登錄，以及部署 Web 應用程式的區域。 |
| **資源群組** | **使用現有的** > `myResourceGroup` |
| **App Service 方案/位置** | 在**美國西部**區域中，建立名為 `plan-westus` 的新方案。 |
| **映像** | `acr-helloworld:v1`

選取 [建立] 以將 Web 應用程式佈建至「美國西部」區域。

![Azure 入口網站中 Linux 上的 Web 應用程式設定][deploy-app-portal-02]

## <a name="view-the-deployed-web-app"></a>檢視部署的 Web 應用程式

部署完成時，您可以在瀏覽器中瀏覽至其 URL 檢視執行的應用程式。

在入口網站中，選取 [應用程式服務]，然後選取您在上一個步驟中佈建的 Web 應用程式。 在此範例中，Web 應用程式名為 *uniqueregistryname-westus*。

選取 **App Service** 概觀右上方 Web 應用程式的超連結 URL，以在您的瀏覽器中檢視執行中的應用程式。

![Azure 入口網站中 Linux 上的 Web 應用程式設定][deploy-app-portal-04]

從異地複寫的容器登錄部署 Docker 映像之後，網站顯示的映像就代表裝載容器登錄的 Azure 區域。

![在瀏覽器中檢視部署的 Web 應用程式][deployed-app-westus]

## <a name="deploy-second-web-app-for-containers-instance"></a>部署第二個用於容器的 Web 應用程式執行個體

使用上一節所述的程序，將第二個 Web 應用程式部署到「美國東部」區域。 在 [用於容器的 Web App] 下，指定下列值：

| 設定 | 值 |
|---|---|
| **網站名稱** | Web 應用程式的全域唯一名稱。 在此範例中，我們使用格式 `<acrName>-eastus` 以便更容易找出登錄，以及部署 Web 應用程式的區域。 |
| **資源群組** | **使用現有的** > `myResourceGroup` |
| **App Service 方案/位置** | 在**美國東部**區域中，建立名為 `plan-eastus` 的新方案。 |
| **映像** | `acr-helloworld:v1`

選取 [建立] 以將 Web 應用程式佈建至「美國東部」區域。

![Azure 入口網站中 Linux 上的 Web 應用程式設定][deploy-app-portal-06]

## <a name="view-the-deployed-web-app"></a>檢視部署的 Web 應用程式

像之前一樣，您可以在瀏覽器中瀏覽至其 URL 檢視執行的應用程式。

在入口網站中，選取 [應用程式服務]，然後選取您在上一個步驟中佈建的 Web 應用程式。 在此範例中，Web 應用程式名為 *uniqueregistryname-eastus*。

選取 [App Service 概觀] 右上方 Web 應用程式的超連結 URL，在您的瀏覽器中檢視執行的應用程式。

![Azure 入口網站中 Linux 上的 Web 應用程式設定][deploy-app-portal-07]

從異地複寫的容器登錄部署 Docker 映像之後，網站顯示的映像就代表裝載容器登錄的 Azure 區域。

![在瀏覽器中檢視部署的 Web 應用程式][deployed-app-eastus]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您從異地複寫的 Azure Container Registry 部署了兩個用於容器的 Web 應用程式執行個體。

請前進到下一個教學課程更新容器映像，並將新的容器映像部署到容器登錄中，然後確認這兩個區域中執行的 Web 應用程式都已自動更新。

> [!div class="nextstepaction"]
> [對異地複寫的容器映像部署更新](./container-registry-tutorial-deploy-update.md)

<!-- IMAGES -->
[deploy-app-portal-01]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-01.png
[deploy-app-portal-02]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-02.png
[deploy-app-portal-03]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-03.png
[deploy-app-portal-04]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-04.png
[deploy-app-portal-05]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-05.png
[deploy-app-portal-06]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-06.png
[deploy-app-portal-07]: ./media/container-registry-tutorial-deploy-app/deploy-app-portal-07.png
[deployed-app-westus]: ./media/container-registry-tutorial-deploy-app/deployed-app-westus.png
[deployed-app-eastus]: ./media/container-registry-tutorial-deploy-app/deployed-app-eastus.png