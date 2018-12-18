---
title: 教學課程 - 使用 Azure Container Registry 工作自動執行容器映像建置
description: 在本教學課程中，您將了解如何設定將原始程式碼認可至 Git 存放庫時可在雲端中自動觸發容器映像建置的工作。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 27dbee3b292a9139ce53ef7b09a4cceba56082e4
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857222"
---
# <a name="tutorial-automate-container-image-builds-with-azure-container-registry-tasks"></a>教學課程：使用 Azure Container Registry 工作自動執行容器映像建置

除了[快速工作](container-registry-tutorial-quick-task.md)以外，ACR 工作也支援透過「建置工作」的自動化 Docker 容器映像建置。 在本教學課程中，您會使用 Azure CLI 建立工作，以在將原始程式碼認可至 Git 存放庫時在雲端中自動觸發映像建置。

在本教學課程中，系列的第二段：

> [!div class="checklist"]
> * 建立工作
> * 測試工作
> * 檢視工作狀態
> * 透過程式碼認可觸發工作

本教學課程假設您已完成[上一個教學課程](container-registry-tutorial-quick-task.md)中的步驟。 如果您尚未完成上一個教學課程的[必要條件](container-registry-tutorial-quick-task.md#prerequisites)一節中的步驟，請先加以完成，再繼續操作。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您想要在本機使用 Azure CLI，必須安裝 Azure CLI **2.0.46** 版或更新版本，並使用 [az login][az-login] 登入。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級 CLI，請參閱[安裝 Azure CLI][azure-cli]。

## <a name="prerequisites"></a>必要條件

### <a name="get-sample-code"></a>取得範例程式碼

本教學課程假設您已完成[上一個教學課程](container-registry-tutorial-quick-task.md)中的步驟，並已派生和複製範例存放庫。 如果您尚未完成上一個教學課程的[必要條件](container-registry-tutorial-quick-task.md#prerequisites)一節中的步驟，請先加以完成，再繼續操作。

### <a name="container-registry"></a>容器登錄

您的 Azure 訂用帳戶中必須要有 Azure 容器登錄，才能完成本教學課程。 如果您需要登錄，請參閱[上一個教學課程](container-registry-tutorial-quick-task.md)或[快速入門：使用 Azure CLI 建立容器登錄](container-registry-get-started-azure-cli.md)。

## <a name="overview-of-acr-tasks"></a>ACR 工作概觀

工作會定義自動化建置的屬性，包括容器映像原始程式碼的位置和觸發建置的事件。 工作中定義的事件發生時 (例如認可至 Git 存放庫)，ACR 工作即會在雲端中起始容器映像建置。 根據預設，它接著會將已成功建置的映像推送至工作中指定的 Azure 容器登錄。

ACR 工作目前支援下列觸發程序：

* 認可至 Git 存放庫
* 基底映像更新

## <a name="create-a-build-task"></a>建立建置工作

在本節中，您會先建立用於 ACR 工作的 GitHub 個人存取權杖 (PAT)。 接著，您會建立工作，以在程式碼認可至存放庫的分支時觸發建置。

### <a name="create-a-github-personal-access-token"></a>建立 GitHub 個人存取權杖

為了在認可至 Git 存放庫時觸發建置，ACR 工作需要以個人存取權杖 (PAT) 存取存放庫。 請依照下列步驟在 GitHub 中產生 PAT：

1. 瀏覽至 GitHub 上的 PAT 建立頁面 (https://github.com/settings/tokens/new)
1. 輸入權杖的簡短**說明**，例如「ACR 工作示範」
1. 在 **repo** 下，啟用 **repo:status** 和 **public_repo**

   ![GitHub 中的個人存取權杖產生頁面的螢幕擷取畫面][build-task-01-new-token]

1. 選取 [產生權杖] 按鈕 (系統可能會要求您確認密碼)
1. 在**安全的位置**複製並儲存產生的權杖 (當您在下一節定義工作時，將會使用此權杖)

   ![GitHub 中已產生的個人存取權杖的螢幕擷取畫面][build-task-02-generated-token]

### <a name="create-the-build-task"></a>建立建置工作

現在，您已完成啟用 ACR 工作以讀取認可狀態以及在存放庫中建立 Webhook 所需的步驟，接下來可以建立工作，以在認可至存放庫時觸發容器映像建置。

首先，請在這些殼層環境變數中填入您的環境適用的值。 此步驟並不是必要動作，但可簡化在本教學課程中執行多行 Azure CLI 命令的作業。 若未填入這些環境變數，則必須在命令範例中的每個對應之處手動取代各個變數。

```azurecli-interactive
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the previous section
```

現在，請執行下列 [az acr task create][az-acr-task-create] 命令以建立工作：

```azurecli-interactive
az acr task create \
    --registry $ACR_NAME \
    --name taskhelloworld \
    --image helloworld:{{.Run.ID}} \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git \
    --branch master \
    --file Dockerfile \
    --git-access-token $GIT_PAT
```

> [!IMPORTANT]
> 如果您先前已在預覽期間使用 `az acr build-task` 命令建立工作，則必須使用 [az acr task][az-acr-task] 命令重新建立這些工作。

此工作會指定只要有程式碼認可至 `--context` 所指定之存放庫中的「主要」分支，ACR 工作即會從該分支中的程式碼建置容器映像。 系統會使用 `--file` 所指定、位於存放庫根目錄中的 Dockerfile。 `--image` 引數會針對映像標記的版本部分指定 `{{.Run.ID}}` 的參數化值，以確保建置的映像會與特定的組建相互關聯，並加上唯一標記。

成功執行的 [az acr task create][az-acr-task-create] 命令會產生如下的輸出：

```console
$ az acr task create \
>     --registry $ACR_NAME \
>     --name taskhelloworld \
>     --image helloworld:{{.Run.ID}} \
>     --context https://github.com/$GIT_USER/acr-build-helloworld-node.git \
>     --branch master \
>     --file Dockerfile \
>     --git-access-token $GIT_PAT
{
  "agentConfiguration": {
    "cpu": 2
  },
  "creationDate": "2018-09-14T22:42:32.972298+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myregistry/providers/Microsoft.ContainerRegistry/registries/myregistry/tasks/taskhelloworld",
  "location": "westcentralus",
  "name": "taskhelloworld",
  "platform": {
    "architecture": "amd64",
    "os": "Linux",
    "variant": null
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "myregistry",
  "status": "Enabled",
  "step": {
    "arguments": [],
    "baseImageDependencies": null,
    "contextPath": "https://github.com/gituser/acr-build-helloworld-node",
    "dockerFilePath": "Dockerfile",
    "imageNames": [
      "helloworld:{{.Run.ID}}"
    ],
    "isPushEnabled": true,
    "noCache": false,
    "type": "Docker"
  },
  "tags": null,
  "timeout": 3600,
  "trigger": {
    "baseImageTrigger": {
      "baseImageTriggerType": "Runtime",
      "name": "defaultBaseimageTriggerName",
      "status": "Enabled"
    },
    "sourceTriggers": [
      {
        "name": "defaultSourceTriggerName",
        "sourceRepository": {
          "branch": "master",
          "repositoryUrl": "https://github.com/gituser/acr-build-helloworld-node",
          "sourceControlAuthProperties": null,
          "sourceControlType": "Github"
        },
        "sourceTriggerEvents": [
          "commit"
        ],
        "status": "Enabled"
      }
    ]
  },
  "type": "Microsoft.ContainerRegistry/registries/tasks"
}
```

## <a name="test-the-build-task"></a>測試建置工作

現在，您已有定義組建的工作。 若要測試組建管線，請執行 [az acr task run][az-acr-task-run] 命令以手動觸發建置：

```azurecli-interactive
az acr task run --registry $ACR_NAME --name taskhelloworld
```

根據預設，`az acr task run` 命令會在您執行命令時將記錄輸出串流處理至主控台。

```console
$ az acr task run --registry $ACR_NAME --name taskhelloworld

2018/09/17 22:51:00 Using acb_vol_9ee1f28c-4fd4-43c8-a651-f0ed027bbf0e as the home volume
2018/09/17 22:51:00 Setting up Docker configuration...
2018/09/17 22:51:02 Successfully set up Docker configuration
2018/09/17 22:51:02 Logging in to registry: myregistry.azurecr.io
2018/09/17 22:51:03 Successfully logged in
2018/09/17 22:51:03 Executing step: build
2018/09/17 22:51:03 Obtaining source code and scanning for dependencies...
2018/09/17 22:51:05 Successfully obtained source code and scanned for dependencies
Sending build context to Docker daemon  23.04kB
Step 1/5 : FROM node:9-alpine
9-alpine: Pulling from library/node
Digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
Status: Image is up to date for node:9-alpine
 ---> a56170f59699
Step 2/5 : COPY . /src
 ---> 5f574fcf5816
Step 3/5 : RUN cd /src && npm install
 ---> Running in b1bca3b5f4fc
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN helloworld@1.0.0 No repository field.

up to date in 0.078s
Removing intermediate container b1bca3b5f4fc
 ---> 44457db20dac
Step 4/5 : EXPOSE 80
 ---> Running in 9e6f63ec612f
Removing intermediate container 9e6f63ec612f
 ---> 74c3e8ea0d98
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in 7382eea2a56a
Removing intermediate container 7382eea2a56a
 ---> e33cd684027b
Successfully built e33cd684027b
Successfully tagged myregistry.azurecr.io/helloworld:da2
2018/09/17 22:51:11 Executing step: push
2018/09/17 22:51:11 Pushing image: myregistry.azurecr.io/helloworld:da2, attempt 1
The push refers to repository [myregistry.azurecr.io/helloworld]
4a853682c993: Preparing
[...]
4a853682c993: Pushed
[...]
da2: digest: sha256:c24e62fd848544a5a87f06ea60109dbef9624d03b1124bfe03e1d2c11fd62419 size: 1366
2018/09/17 22:51:21 Successfully pushed image: myregistry.azurecr.io/helloworld:da2
2018/09/17 22:51:21 Step id: build marked as successful (elapsed time in seconds: 7.198937)
2018/09/17 22:51:21 Populating digests for step id: build...
2018/09/17 22:51:22 Successfully populated digests for step id: build
2018/09/17 22:51:22 Step id: push marked as successful (elapsed time in seconds: 10.180456)
The following dependencies were found:
- image:
    registry: myregistry.azurecr.io
    repository: helloworld
    tag: da2
    digest: sha256:c24e62fd848544a5a87f06ea60109dbef9624d03b1124bfe03e1d2c11fd62419
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 9-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git:
    git-head-revision: 68cdf2a37cdae0873b8e2f1c4d80ca60541029bf


Run ID: da2 was successful after 27s
```

## <a name="trigger-a-build-with-a-commit"></a>透過認可觸發建置

現在您已藉由手動執行測試了工作，接下來請透過原始程式碼的變更自動加以觸發。

首先，請確定您位於[存放庫][sample-repo]的本機複本所在的目錄中：

```azurecli-interactive
cd acr-build-helloworld-node
```

接著，執行下列命令以建立新檔案，然後將其認可並推送至 GitHub 上的存放庫分支：

```azurecli-interactive
echo "Hello World!" > hello.txt
git add hello.txt
git commit -m "Testing ACR Tasks"
git push origin master
```

在您執行 `git push` 命令時，系統可能會要求您提供 GitHub 認證。 請提供您的 GitHub 使用者名稱，並輸入您先前為密碼建立的個人存取權杖 (PAT)。

```console
$ git push origin master
Username for 'https://github.com': <github-username>
Password for 'https://githubuser@github.com': <personal-access-token>
```

在您將認可推送至存放庫後，ACR 工作所建立的 Webhook 即會在 Azure Container Registry 中引發並啟動建置。 請顯示目前所執行工作的記錄，以確認並監視建置進度：

```azurecli-interactive
az acr task logs --registry $ACR_NAME
```

輸出會類似於下列內容，顯示目前正在執行 (或最後執行) 的工作：

```console
$ az acr task logs --registry $ACR_NAME
Showing logs of the last created run.
Run ID: da4

[...]

Run ID: da4 was successful after 38s
```

## <a name="list-builds"></a>列出建置

若要查看 ACR 工作已為登錄完成的工作流程執行，請執行 [az acr task list-runs][az-acr-task-list-runs] 命令：

```azurecli-interactive
az acr task list-runs --registry $ACR_NAME --output table
```

此命令的輸出應類似於下列內容。 ACR 工作已執行的流程執行會顯示出來，且最新工作的 [觸發程序] 資料行中會出現「Git 認可」：

```console
$ az acr task list-runs --registry $ACR_NAME --output table

RUN ID    TASK             PLATFORM    STATUS     TRIGGER     STARTED               DURATION
--------  --------------  ----------  ---------  ----------  --------------------  ----------
da4       taskhelloworld  Linux       Succeeded  Git Commit  2018-09-17T23:03:45Z  00:00:44
da3       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T22:55:35Z  00:00:35
da2       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T22:50:59Z  00:00:32
da1                       Linux       Succeeded  Manual      2018-09-17T22:29:59Z  00:00:57
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用工作，在將原始程式碼認可至 Git 存放庫時於 Azure 中自動觸發容器映像建置。 請進入下一個教學課程，以了解如何建立會在容器映像的基底映像更新時觸發建置的工作。

> [!div class="nextstepaction"]
> [在基底映像更新時自動執行建置](container-registry-tutorial-base-image-update.md)

<!-- LINKS - External -->
[sample-repo]: https://github.com/Azure-Samples/acr-build-helloworld-node

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task]: /cli/azure/acr#az-acr-task
[az-acr-task-create]: /cli/azure/acr#az-acr-task-create
[az-acr-task-run]: /cli/azure/acr#az-acr-task-run
[az-acr-task-list-runs]: /cli/azure/acr#az-acr-task-list-runs
[az-login]: /cli/azure/reference-index#az-login

<!-- IMAGES -->
[build-task-01-new-token]: ./media/container-registry-tutorial-build-tasks/build-task-01-new-token.png
[build-task-02-generated-token]: ./media/container-registry-tutorial-build-tasks/build-task-02-generated-token.png
