---
title: 教學課程 - 使用 Azure Container Registry 工作在基底映像更新時自動執行容器映像建置
description: 在本教學課程中，您將了解如何設定在基底映像更新時可在雲端中自動觸發容器映像建置的工作。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 54e8892787fa2b7b093609ee5d09f3a87e103411
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48856576"
---
# <a name="tutorial-automate-image-builds-on-base-image-update-with-azure-container-registry-tasks"></a>教學課程：使用 Azure Container Registry 工作在基底映像更新時自動執行映像建置

ACR 工作支援在容器的基底映像更新時自動執行建置，例如，當您在其中一個基底映像中修補作業系統或應用程式架構時。 在本教學課程中，您將了解如何在 ACR 工作中建立工作，以在容器的基底映像已推送至登錄時於雲端中觸發建置。

在本教學課程中 (系列的最後一個)：

> [!div class="checklist"]
> * 建置基底映像
> * 建立應用程式映像建置工作
> * 更新基底映像以觸發應用程式映像工作
> * 顯示已觸發的工作
> * 確認更新的應用程式映像

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您想要在本機使用 Azure CLI，您必須安裝 Azure CLI **2.0.46** 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級 CLI，請參閱[安裝 Azure CLI][azure-cli]。

## <a name="prerequisites"></a>必要條件

### <a name="complete-the-previous-tutorials"></a>完成先前的教學課程

本教學課程假設您已完成本系列前兩個教學課程中的步驟，其間您完成了下列作業：

* 建立 Azure Container Registry
* 派生範例存放庫
* 複製範例存放庫
* 建立 GitHub 個人存取權杖

如果您尚未完成前兩個教學課程，請先加以完成，再繼續操作：

[使用 Azure Container Registry 工作在雲端中建置容器映像](container-registry-tutorial-quick-task.md)

[使用 Azure Container Registry 工作自動執行容器映像建置](container-registry-tutorial-build-task.md)

### <a name="configure-the-environment"></a>建立環境

請在這些殼層環境變數中填入您的環境適用的值。 此步驟並不是必要動作，但可簡化在本教學課程中執行多行 Azure CLI 命令的作業。 若未填入這些環境變數，則必須在命令範例中的每個對應之處手動取代各個變數。

```azurecli-interactive
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the second tutorial
```

## <a name="base-images"></a>基礎映像

定義大多數容器映像的 Dockerfile，會指定供其作為基礎的父映像，通常稱為其*基底映像*。 基底映像通常包含會套用容器其餘各層的作業系統，例如 [Alpine Linux][base-alpine] 或 [Windows Nano Server][base-windows]。 此外也可能包含應用程式架構，例如 [Node.js][base-node] 或 [.NET Core][base-dotnet]。

### <a name="base-image-updates"></a>基底映像更新

基底映像常會由映像維護程式進行更新，以在映像中納入作業系統或架構的新功能或增強功能。 安全性修補程式是基底映像進行更新的另一個常見原因。

基底映像進行更新時，您將需要據以在登錄中重建任何容器映像，以納入新功能和修正程式。 ACR 工作具有在容器的基底映像更新時自動為您建置映像的能力。

### <a name="base-image-update-scenario"></a>基底映像更新案例

本教學課程將逐步說明基底映像更新案例。 [程式碼範例][code-sample]包含兩個 Dockerfile：一個應用程式映像，及其指定作為基底的映像。 在以下幾節中，您會建立一項 ACR 工作，以在基底映像的新版本推送至容器登錄時自動觸發應用程式映像的建置。

[Dockerfile-app][dockerfile-app]：一個小型 Node.js Web 應用程式，會呈現一個靜態網頁，顯示它所依據的 Node.js 版本。 系統會模擬版本字串：它會顯示基底映像中定義的環境變數 `NODE_VERSION` 的內容。

[Dockerfile-base][dockerfile-base]：`Dockerfile-app` 指定作為其基底的映像。 它本身會以[節點][base-node]映像為基礎，且包含 `NODE_VERSION` 環境變數。

在以下幾節中，您會建立工作、更新基底映像 Dockerfile 中的 `NODE_VERSION` 值，然後使用 ACR 工作來建置基底映像。 當 ACR 工作將新的基底映像推送至您的登錄時，它會自動觸發應用程式映像的建置。 您可以選擇性地在本機執行應用程式容器映像，以查看已建置的映像中不同的版本字串。

## <a name="build-the-base-image"></a>建置基底映像

首先請使用 ACR 工作*快速工作*來建置基底映像。 如本系列的[第一個教學課程](container-registry-tutorial-quick-task.md)所討論的，此程序不僅會建置映像，也會在建置成功時將映像推送至您的容器登錄。

```azurecli-interactive
az acr build --registry $ACR_NAME --image baseimages/node:9-alpine --file Dockerfile-base .
```

## <a name="create-a-task"></a>建立工作

接著，請使用 [az acr task create][az-acr-task-create] 建立工作：

```azurecli-interactive
az acr task create \
    --registry $ACR_NAME \
    --name taskhelloworld \
    --image helloworld:{{.Run.ID}} \
    --arg REGISTRY_NAME=$ACR_NAME.azurecr.io \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git \
    --file Dockerfile-app \
    --branch master \
    --git-access-token $GIT_PAT
```

> [!IMPORTANT]
> 如果您先前已在預覽期間使用 `az acr build-task` 命令建立工作，則必須使用 [az acr task][az-acr-task] 命令重新建立這些工作。

此工作類似於[上一個教學課程](container-registry-tutorial-build-task.md)中建立的快速工作。 它會指示 ACR 工作在認可推送至 `--context` 所指定的存放庫時觸發映像建置。

其不同之處在於其行為，亦即它也會在*基底映像*更新時觸發映像的建置。 `--file` 引數所指定的 Dockerfile [Dockerfile-app][dockerfile-app] 支援從與基底相同的登錄中指定映像：

```Dockerfile
FROM ${REGISTRY_NAME}/baseimages/node:9-alpine
```

當您執行工作時，ACR 工作會偵測映像的相依性。 如果 `FROM` 陳述式所指定的基底映像位於相同的登錄或公用 Docker Hub 存放庫中，則會新增一個 Hook，以確保此映像會在其基底有所更新時進行重建。

## <a name="build-the-application-container"></a>建置應用程式容器

若要讓 ACR 工作能夠判斷和追蹤容器映像的相依性 (包括其基底映像)，您**必須**先觸發其工作**至少一次**。

請使用 [az acr task run][az-acr-task-run] 手動觸發工作，並建置應用程式映像：

```azurecli-interactive
az acr task run --registry $ACR_NAME --name taskhelloworld
```

工作完成後，如果您想要完成下列選擇性步驟，請記下**回合識別碼** (例如 "da6")。

### <a name="optional-run-application-container-locally"></a>選擇性：在本機執行應用程式容器

如果您在本機工作 (而不是在 Cloud Shell 中)，且您已安裝 Docker，請先執行容器以檢視在網頁瀏覽器中呈現的應用程式，再重建其基底映像。 如果您使用 Cloud Shell，請略過本節 (Cloud Shell 不支援 `az acr login` 或 `docker run`)。

首先，使用 [az acr login][az-acr-login] 登入您的容器登錄：

```azurecli
az acr login --name $ACR_NAME
```

現在，請使用 `docker run` 在本機執行容器。 請將 **\<run-id\>** 取代為在上一個步驟的輸出中找到的回合識別碼 (例如 "da6")。

```azurecli
docker run -d -p 8080:80 $ACR_NAME.azurecr.io/helloworld:<run-id>
```

在瀏覽器中瀏覽至 http://localhost:8080，您應該會看到 Node.js 版本號碼呈現於網頁中，如下所示。 在後續步驟中，您可以在版本字串中加上 "a"，以變更版本。

![呈現在瀏覽器中的範例應用程式的螢幕擷取畫面][base-update-01]

## <a name="list-the-builds"></a>列出組建

接著，使用 [az acr task list-runs][az-acr-task-list-runs] 命令，列出 ACR 工作已為登錄完成的工作回合：

```azurecli-interactive
az acr task list-runs --registry $ACR_NAME --output table
```

如果您已完成上一個教學課程 (且未刪除登錄)，您應該會看到如下的輸出。 請記下工作回合數目和最新的回合識別碼，以便在下一節更新基底映像後比較輸出。

```console
$ az acr task list-runs --registry $ACR_NAME --output table

RUN ID    TASK            PLATFORM    STATUS     TRIGGER     STARTED               DURATION
--------  --------------  ----------  ---------  ----------  --------------------  ----------
da6       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T23:07:22Z  00:00:38
da5                       Linux       Succeeded  Manual      2018-09-17T23:06:33Z  00:00:31
da4       taskhelloworld  Linux       Succeeded  Git Commit  2018-09-17T23:03:45Z  00:00:44
da3       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T22:55:35Z  00:00:35
da2       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T22:50:59Z  00:00:32
da1                       Linux       Succeeded  Manual      2018-09-17T22:29:59Z  00:00:57
```

## <a name="update-the-base-image"></a>更新基底映像

在此您將模擬基底映像中的架構修補程式。 請編輯 **Dockerfile-base**，並在 `NODE_VERSION` 中定義的版本號碼後面加上 "a"：

```Dockerfile
ENV NODE_VERSION 9.11.2a
```

執行快速工作，以建置經過修改的基底映像。 請記下輸出中的**回合識別碼**。

```azurecli-interactive
az acr build --registry $ACR_NAME --image baseimages/node:9-alpine --file Dockerfile-base .
```

當建置完成，且 ACR 工作已將新的基底映像推送至您的登錄後，它會觸發應用程式映像的建置。 您先前建立的工作可能需要一些時間才能觸發應用程式映像建置，因為它必須偵測最新建置並推送的基底映像。

## <a name="list-updated-build"></a>列出更新的組建

現在您已更新基底映像，接下來可以再次列出您的工作回合，將其與先前的清單比較。 若起初輸出並無差異，請定期執行命令，以查看清單中出現的新工作回合。

```azurecli-interactive
az acr task list-runs --registry $ACR_NAME --output table
```

輸出大致如下。 最後執行之建置的 TRIGGER 應為 "Image Update"，表示工作是由基底映像的快速工作所起始。

```console
$ az acr task list-builds --registry $ACR_NAME --output table

Run ID    TASK            PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
da8       taskhelloworld  Linux       Succeeded  Image Update  2018-09-17T23:11:50Z  00:00:33
da7                       Linux       Succeeded  Manual        2018-09-17T23:11:27Z  00:00:35
da6       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T23:07:22Z  00:00:38
da5                       Linux       Succeeded  Manual        2018-09-17T23:06:33Z  00:00:31
da4       taskhelloworld  Linux       Succeeded  Git Commit    2018-09-17T23:03:45Z  00:00:44
da3       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T22:55:35Z  00:00:35
da2       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T22:50:59Z  00:00:32
da1                       Linux       Succeeded  Manual        2018-09-17T22:29:59Z  00:00:57
```

如果您想要執行下列選擇性步驟以執行新建置的容器，並查看更新的版本號碼，請記下映像更新觸發之組建的 **RUN ID** 值 (在上述輸出中，其值為 "da8")。

### <a name="optional-run-newly-built-image"></a>選擇性：執行新建置的映像

如果您在本機工作 (而不是在 Cloud Shell 中)，且您已安裝 Docker，請在新的應用程式映像建置完成後執行該映像。 請將 `<run-id>` 取代為您在上一個步驟中取得的 RUN ID。 如果您使用 Cloud Shell，請略過本節 (Cloud Shell 不支援 `docker run`)。

```bash
docker run -d -p 8081:80 $ACR_NAME.azurecr.io/helloworld:<run-id>
```

在瀏覽器中瀏覽至 http://localhost:8081，您應該會在網頁中看到更新的 Node.js 版本號碼 (附有 "a")：

![呈現在瀏覽器中的範例應用程式的螢幕擷取畫面][base-update-02]

務必留意的是，您是使用新的版本號碼更新**基底**映像，但最後建置的**應用程式**映像會顯示新版本。 ACR 工作會取用您對基底映像的變更，並自動重建您的應用程式映像。

## <a name="clean-up-resources"></a>清除資源

若要移除您在本教學課程系列中建立的所有資源 (包括容器登錄、容器執行個體、金鑰保存庫和服務主體)，請發出下列命令：

```azurecli-interactive
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用工作，在映像的基底映像有所更新時自動觸發容器映像建置。 接下來請了解關於容器登錄驗證的資訊。

> [!div class="nextstepaction"]
> [Azure Container Registry 中的驗證](container-registry-authentication.md)

<!-- LINKS - External -->
[base-alpine]: https://hub.docker.com/_/alpine/
[base-dotnet]: https://hub.docker.com/r/microsoft/dotnet/
[base-node]: https://hub.docker.com/_/node/
[base-windows]: https://hub.docker.com/r/microsoft/nanoserver/
[code-sample]: https://github.com/Azure-Samples/acr-build-helloworld-node
[dockerfile-app]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-app
[dockerfile-base]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-base

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build-run
[az-acr-task-create]: /cli/azure/acr#az-acr-task-create
[az-acr-task-run]: /cli/azure/acr#az-acr-task-run
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-task-list-runs]: /cli/azure/acr#az-acr-task-list-runs
[az-acr-task]: /cli/azure/acr#az-acr-task

<!-- IMAGES -->
[base-update-01]: ./media/container-registry-tutorial-base-image-update/base-update-01.png
[base-update-02]: ./media/container-registry-tutorial-base-image-update/base-update-02.png