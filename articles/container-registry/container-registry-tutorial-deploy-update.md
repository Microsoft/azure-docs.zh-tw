---
title: Azure Container Registry 教學課程 - 將更新的映像推送至區域部署
description: 將修改的 Docker 映像推送至您進行地理複寫的 Azure Container Registry，然後查看變更自動部署至多個區域中執行的 Web 應用程式。 這是三段式教學課程的第三段。
services: container-registry
author: mmacy
manager: jeconnoc
ms.service: container-registry
ms.topic: tutorial
ms.date: 04/30/2018
ms.author: marsma
ms.custom: mvc
ms.openlocfilehash: 8edb35b91327bde1fa824ec456b8a98962adb7ce
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38634082"
---
# <a name="tutorial-push-an-updated-image-to-regional-deployments"></a>教學課程：將更新的映像推送至區域部署

這是三段式教學課程系列的第三段。 [上一個教學課程](container-registry-tutorial-deploy-app.md)是針對兩個不同區域的 Web 應用程式部署設定地理複寫。 在本教學課程中，您要先修改應用程式，然後建立新的容器映像，再將它推送至您進行地理複寫的登錄中。 最後，您要檢視由 Azure Container Registry Webhook 在兩個 Web 應用程式執行個體中自動部署的變更。

在本教學課程中，系列的最後一段：

> [!div class="checklist"]
> * 修改 Web 應用程式 HTML
> * 建置和標記 Docker 映像
> * 將變更推送至 Azure Container Registry
> * 檢視兩個不同區域中更新後的應用程式

如果您尚未設定兩個*用於容器的 Web 應用程式*區域部署，請先返回系列的上一個教學課程[從 Azure Container Registry 部署 Web 應用程式](container-registry-tutorial-deploy-app.md)。

## <a name="modify-the-web-application"></a>修改 Web 應用程式

在此步驟中，請對 Web 應用程式進行一項在將更新的容器映像推送至 Azure Container Registry 之後很容易發現的變更。

在上一個教學課程中，[從 GitHub 複製的](container-registry-tutorial-prepare-registry.md#get-application-code)應用程式來源中，找出 `AcrHelloworld/Views/Home/Index.cshtml` 檔案，並在慣用的文字編輯器中開啟檔案。 在現有的 `<h1>` 行下方加入下列這行：

```html
<h1>MODIFIED</h1>
```

您修改過的 `Index.cshtml` 應該類似：

```html
@{
    ViewData["Title"] = "Azure Container Registry :: Geo-replication";
}
<style>
    body {
        background-image: url('images/azure-regions.png');
        background-size: cover;
    }
    .footer {
        position: fixed;
        bottom: 0px;
        width: 100%;
    }
</style>

<h1 style="text-align:center;color:blue">Hello World from:  @ViewData["REGION"]</h1>
<h1>MODIFIED</h1>
<div class="footer">
    <ul>
        <li>Registry URL: @ViewData["REGISTRYURL"]</li>
        <li>Registry IP: @ViewData["REGISTRYIP"]</li>
        <li>Registry Region: @ViewData["REGION"]</li>
    </ul>
</div>
```

## <a name="rebuild-the-image"></a>重建映像

因為您更新了 Web 應用程式，所以請重建其容器映像。 像之前一樣，為標記使用完整的映像名稱，包括登入伺服器的完整網域名稱 (FQDN)：

```bash
docker build . -f ./AcrHelloworld/Dockerfile -t <acrName>.azurecr.io/acr-helloworld:v1
```

## <a name="push-image-to-azure-container-registry"></a>將映像推送至 Azure Container Registry

接下來，將更新的 *acr-helloworld* 容器映像推送至您進行異地複寫的登錄。 此時，您要執行一個 `docker push` 命令，將更新的映像同時部署至「美國西部」和「美國東部」兩個區域中的登錄複本。

```bash
docker push <acrName>.azurecr.io/acr-helloworld:v1
```

您的 `docker push`輸出應該類似如下範例：

```console
$ docker push uniqueregistryname.azurecr.io/acr-helloworld:v1
The push refers to a repository [uniqueregistryname.azurecr.io/acr-helloworld]
5b9454e91555: Pushed
d6803756744a: Layer already exists
b7b1f3a15779: Layer already exists
a89567dff12d: Layer already exists
59c7b561ff56: Layer already exists
9a2f9413d9e4: Layer already exists
a75caa09eb1f: Layer already exists
v1: digest: sha256:4c3f2211569346fbe2d1006c18cbea2a4a9dcc1eb3a078608cef70d3a186ec7a size: 1792
```

## <a name="view-the-webhook-logs"></a>檢視 Webhook 記錄

複寫映像時，您會看到觸發的 Azure Container Registry Webhook。

若要查看您在上一個教學課程中將容器部署至*用於容器的 Web 應用程式*時所建立的區域 Webhook，請在 Azure 入口網站中瀏覽至您的容器登錄，然後在 [服務] 下選取 [Webhook]。

![Azure 入口網站中的容器登錄 Webhook][tutorial-portal-01]

選取每個 Webhook 可檢視其呼叫及回應的歷程記錄。 您應該會在兩個 Webhook 記錄中看到**推送**動作的資料列。 此時，位於「美國西部」區域的 Webhook 記錄顯示在上一個步驟中由 `docker push` 觸發的**推送**動作：

![Azure 入口網站中的容器登錄 Webhook 記錄 (美國西部)][tutorial-portal-02]

## <a name="view-the-updated-web-app"></a>檢視更新後的 Web 應用程式

Webhook 會通知 Web 應用程式新的映像已推送至登錄，此作業會自動將更新的容器部署至兩個區域的 Web 應用程式。

在您的網頁瀏覽器中，瀏覽至這兩個區域的 Web 應用程式部署，確認兩個部署中的應用程式都已更新。 請注意，您可以在每個 App Service 概觀 索引標籤右上方找到已部署 Web 應用程式的 URL。

![Azure 入口網站中的 App Service 概觀][tutorial-portal-03]

若要查看更新過的應用程式，請選取 App Service 概觀中的連結。 以下是在「美國西部」中執行之應用程式的範例檢視：

![在美國西部區域中執行之已修改 Web 應用程式的瀏覽器檢視][deployed-app-westus-modified]

在您的瀏覽器中檢視，確認更新的容器映像也已部署至「美國東部」部署。

![在美國東部區域中執行之已修改 Web 應用程式的瀏覽器檢視][deployed-app-eastus-modified]

只要執行一次 `docker push`，您就能自動更新在兩個區域 Web 應用程式部署中執行的 Web 應用程式。 此外，Azure Container Registry 也從最靠近每個部署的存放庫提供容器映像。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您更新了 Web 應用程式容器，並將新版推送到您進行地理複寫的登錄。 Azure Container Registry 中的 Webhook 將該項更新通知了「用於容器的 Web 應用程式」執行個體，因而觸發從最靠近的登錄複本進行本機提取。

### <a name="acr-build-automated-image-build-and-patch"></a>ACR Build：自動映像建置和修補

除了異地複寫之外，ACR Build 是 Azure Container Registry 另一項有助於最佳化容器部署管線的功能。 若要了解 ACR Build 的功能，請從 ACR Build 概觀開始：

[使用 ACR Build 自動進行作業系統和架構修補](container-registry-build-overview.md)

<!-- IMAGES -->
[deployed-app-eastus-modified]: ./media/container-registry-tutorial-deploy-update/deployed-app-eastus-modified.png
[deployed-app-westus-modified]: ./media/container-registry-tutorial-deploy-update/deployed-app-westus-modified.png
[local-container-01]: ./media/container-registry-tutorial-deploy-update/local-container-01.png
[tutorial-portal-01]: ./media/container-registry-tutorial-deploy-update/tutorial-portal-01.png
[tutorial-portal-02]: ./media/container-registry-tutorial-deploy-update/tutorial-portal-02.png
[tutorial-portal-03]: ./media/container-registry-tutorial-deploy-update/tutorial-portal-03.png