---
title: 教學課程 - 使用 Azure Container Registry 工作在雲端中建置容器映像
description: 在本教學課程中，您將了解如何使用 Azure Container Registry 工作 (ACR 工作) 在 Azure 中建置 Docker 容器映像，然後將其部署至 Azure 容器執行個體。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 7ac4fb62c6832920634de96c3e5befb15ce3b3a4
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854723"
---
# <a name="tutorial-build-container-images-in-the-cloud-with-azure-container-registry-tasks"></a>教學課程：使用 Azure Container Registry 工作在雲端中建置容器映像

**ACR 工作**是 Azure Container Registry 內的一組功能，可在 Azure 中提供精簡而有效率的 Docker 容器映像組建。 在本文中，您將了解如何使用 ACR 工作的*快速工作*功能。

「內部迴圈」開發週期是在認可至原始檔控制之前撰寫程式碼、建置和測試應用程式的反覆程序。 快速工作可將您的內部迴圈延伸至雲端，讓您能夠進行建置成功驗證，並將成功建置的映像自動推送至容器登錄。 您的映像依原生狀態會建置在雲端中接近您的登錄之處，因此有助於快速部署。

您的所有 Dockerfile 專業知識都可直接轉移至 ACR 工作。 您不需要變更 Dockerfile，而是需要變更您所執行的命令，即可使用 ACR 工作在雲端中進行建置。

本教學課程是一個系列的第一部分：

> [!div class="checklist"]
> * 取得範例應用程式的原始程式碼
> * 在 Azure 中建置容器映像
> * 將容器部署至 Azure 容器執行個體

在後續的教學課程中，您將了解如何將 ACR 工作工作用於程式碼認可和基礎映像更新的自動化容器映像建置。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您想要在本機使用 Azure CLI，您必須安裝 Azure CLI **2.0.46** 版或更新版本，並使用 [az login][az-login] 登入。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級 CLI，請參閱[安裝 Azure CLI][azure-cli]。

## <a name="prerequisites"></a>必要條件

### <a name="github-account"></a>GitHub 帳戶

在 https://github.com 上建立帳戶 (如果您還沒有帳戶)。 本教學課程系列會使用 GitHub 存放庫來示範使用 ACR 工作的自動化映像建置。

### <a name="fork-sample-repository"></a>派生範例存放庫

接著，請使用 GitHub UI 將範例存放庫派生到您的 GitHub 帳戶中。 在本教學課程中，您會從存放庫中的來源建置容器映像，而在下一個教學課程中，您會將認可推送至該存放庫的分支，以開始進行自動化工作。

派生此存放庫： https://github.com/Azure-Samples/acr-build-helloworld-node

![GitHub 中的 [分支] 按鈕 (醒目提示) 的螢幕擷取畫面][quick-build-01-fork]

### <a name="clone-your-fork"></a>複製您的分支

在您派生存放庫後，請複製您的分支，並進入包含本機複本的目錄。

使用 `git` 複製存放庫，並將 **\<your-github-username\>** 取代為您的 GitHub 使用者名稱：

```azurecli-interactive
git clone https://github.com/<your-github-username>/acr-build-helloworld-node
```

進入包含原始程式碼的目錄：

```azurecli-interactive
cd acr-build-helloworld-node
```

### <a name="bash-shell"></a>Bash 殼層

本教學課程系列中的命令採用 Bash 殼層適用的格式。 如果您想要使用 PowerShell、命令提示字元或其他殼層，您可能需要據以調整行接續符號和環境變數格式。

## <a name="build-in-azure-with-acr-tasks"></a>使用 ACR 工作在 Azure 中進行建置

現在，您已將原始程式碼提取至電腦，接著請依照下列步驟建立容器登錄，並使用 ACR 工作來建置容器映像。

為了方便執行範例命令，在這一系列的教學課程中，將會使用殼層環境變數。 請執行下列命令以設定 `ACR_NAME` 變數。 請將 **\<registry-name\>** 取代為新容器登錄的唯一名稱。 登錄名稱在 Azure 內必須是唯一的，且包含 5-50 個英數字元。 您在本教學課程中建立的其他資源將以此名稱為基礎，因此，您只需要修改第一個變數，即此變數。

```azurecli-interactive
ACR_NAME=<registry-name>
```

在填入容器登錄環境變數後，您現在應該可以複製並貼上教學課程中的其餘命令，而不需編輯的任何值。 請執行下列命令，以建立資源群組和容器登錄：

```azurecli-interactive
RES_GROUP=$ACR_NAME # Resource Group name

az group create --resource-group $RES_GROUP --location eastus
az acr create --resource-group $RES_GROUP --name $ACR_NAME --sku Standard --location eastus
```

現在您已具有登錄，接著請使用 ACR 工作，從範例程式碼建置容器映像。 請執行 [az acr build][az-acr-build] 命令以執行*快速工作*：

```azurecli-interactive
az acr build --registry $ACR_NAME --image helloacrtasks:v1 .
```

[az acr build][az-acr-build] 命令的輸出如下。 您可以檢視上傳至 Azure 的原始程式碼 (「內容」)，以及 ACR 工作在雲端中執行之 `docker build` 作業的詳細資料。 由於 ACR 工作會使用 `docker build` 建置您的映像，因此您無須對 Dockerfile 進行任何變更，即可立即開始使用 ACR 工作。

```console
$ az acr build --registry $ACR_NAME --image helloacrtasks:v1 .
Packing source code into tar file to upload...
Sending build context (4.813 KiB) to ACR...
Queued a build with build ID: da1
Waiting for build agent...
2018/08/22 18:31:42 Using acb_vol_01185991-be5f-42f0-9403-a36bb997ff35 as the home volume
2018/08/22 18:31:42 Setting up Docker configuration...
2018/08/22 18:31:43 Successfully set up Docker configuration
2018/08/22 18:31:43 Logging in to registry: myregistry.azurecr.io
2018/08/22 18:31:55 Successfully logged in
Sending build context to Docker daemon   21.5kB
Step 1/5 : FROM node:9-alpine
9-alpine: Pulling from library/node
Digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
Status: Image is up to date for node:9-alpine
 ---> a56170f59699
Step 2/5 : COPY . /src
 ---> 88087d7e709a
Step 3/5 : RUN cd /src && npm install
 ---> Running in e80e1263ce9a
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN helloworld@1.0.0 No repository field.

up to date in 0.1s
Removing intermediate container e80e1263ce9a
 ---> 26aac291c02e
Step 4/5 : EXPOSE 80
 ---> Running in 318fb4c124ac
Removing intermediate container 318fb4c124ac
 ---> 113e157d0d5a
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in fe7027a11787
Removing intermediate container fe7027a11787
 ---> 20a27b90eb29
Successfully built 20a27b90eb29
Successfully tagged myregistry.azurecr.io/helloacrtasks:v1
2018/08/22 18:32:11 Pushing image: myregistry.azurecr.io/helloacrtasks:v1, attempt 1
The push refers to repository [myregistry.azurecr.io/helloacrtasks]
6428a18b7034: Preparing
c44b9827df52: Preparing
172ed8ca5e43: Preparing
8c9992f4e5dd: Preparing
8dfad2055603: Preparing
c44b9827df52: Pushed
172ed8ca5e43: Pushed
8dfad2055603: Pushed
6428a18b7034: Pushed
8c9992f4e5dd: Pushed
v1: digest: sha256:b038dcaa72b2889f56deaff7fa675f58c7c666041584f706c783a3958c4ac8d1 size: 1366
2018/08/22 18:32:43 Successfully pushed image: myregistry.azurecr.io/helloacrtasks:v1
2018/08/22 18:32:43 Step ID acb_step_0 marked as successful (elapsed time in seconds: 15.648945)
The following dependencies were found:
- image:
    registry: myregistry.azurecr.io
    repository: helloacrtasks
    tag: v1
    digest: sha256:b038dcaa72b2889f56deaff7fa675f58c7c666041584f706c783a3958c4ac8d1
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 9-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git: {}

Run ID: da1 was successful after 1m9.970148252s
```

在輸出結尾附近，ACR 工作會顯示它為您的映像探索到的相依性。 這可讓 ACR 工作在基底映像更新時自動執行映像建置，例如，在使用作業系統或架構修補程式更新基底映像時。 您將在本教學課程系列後續的內容中了解 ACR 工作對基底映像更新的支援。

## <a name="deploy-to-azure-container-instances"></a>部署至 Azure 容器執行個體

ACR 工作依預設會自動將已建置的映像順利推送至登錄，讓您能夠立即從登錄加以部署。

在本節中，您會建立 Azure Key Vault 和服務主體，然後使用服務主體的認證將容器部署至 Azure 容器執行個體 (ACI)。

### <a name="configure-registry-authentication"></a>設定登錄驗證

所有生產案例均應使用[服務主體][service-principal-auth]來存取 Azure 容器登錄。 服務主體可讓您針對容器映像提供角色型存取控制。 例如，您可以設定服務主體具有僅限提取登錄的存取權。

#### <a name="create-a-key-vault"></a>建立金鑰保存庫

如果您在 [Azure Key Vault](/azure/key-vault/) 中還沒有保存庫，使用 Azure CLI 以下列命令建立一個。

```azurecli-interactive
AKV_NAME=$ACR_NAME-vault

az keyvault create --resource-group $RES_GROUP --name $AKV_NAME
```

#### <a name="create-a-service-principal-and-store-credentials"></a>建立服務主體並儲存認證

您現在需要建立服務主體，並將它的認證儲存在金鑰保存庫中。

使用 [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] 命令建立服務主體，並使用 [az keyvault secret set][az-keyvault-secret-set] 將服務主體的**密碼**儲存在保存庫中：

```azurecli-interactive
# Create service principal, store its password in AKV (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name $ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role reader \
                --query password \
                --output tsv)
```

在前面的命令中，`--role` 引數設定服務主體具有 reader 角色，授與主體僅限提取登錄的存取權。 若要同時授與發送和提取存取權，請將 `--role` 引數變更為 contributor。

接下來，在保存庫中儲存服務主體的 appId，也就是您傳遞給 Azure Container Registry 進行驗證的**使用者名稱**：

```azurecli-interactive
# Store service principal ID in AKV (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

您已建立 Azure Key Vault，並在其中儲存兩個祕密：

* `$ACR_NAME-pull-usr`：服務主體識別碼，作為容器登錄的**使用者名稱**。
* `$ACR_NAME-pull-pwd`：服務主體密碼，作為容器登錄的**密碼**。

現在，當您或應用程式和服務從登錄提取映像時，您可以依名稱參考這些祕密。

### <a name="deploy-a-container-with-azure-cli"></a>使用 Azure CLI 部署容器

現在，服務主體認證已儲存 Azure Key Vault 密碼，應用程式和服務可以使用它們來存取您的私人登錄。

執行下列 [az container create][az-container-create] 命令來部署容器執行個體。 此命令會使用儲存在 Azure Key Vault 中的服務主體認證向您的容器登錄進行驗證。

```azurecli-interactive
az container create \
    --resource-group $RES_GROUP \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label acr-tasks-$ACR_NAME \
    --query "{FQDN:ipAddress.fqdn}" \
    --output table
```

`--dns-name-label` 值在 Azure 內必須是唯一的，因此，上述命令會將您容器登錄的名稱附加至容器的 DNS 名稱標籤。 命令的輸出會顯示容器的完整網域名稱 (FQDN)，例如：

```console
$ az container create \
>     --resource-group $RES_GROUP \
>     --name acr-tasks \
>     --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
>     --registry-login-server $ACR_NAME.azurecr.io \
>     --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
>     --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
>     --dns-name-label acr-tasks-$ACR_NAME \
>     --query "{FQDN:ipAddress.fqdn}" \
>     --output table
FQDN
----------------------------------------------
acr-tasks-myregistry.eastus.azurecontainer.io
```

請記下容器的 FQDN，您將下一節用到。

### <a name="verify-the-deployment"></a>驗證部署

若要查看容器的啟動程序，請使用 [az container attach][az-container-attach] 命令：

```azurecli-interactive
az container attach --resource-group $RES_GROUP --name acr-tasks
```

`az container attach` 輸出會先顯示容器在提取映像和啟動時的狀態，然後將本機主控台的 STDOUT 和 STDERR 繫結至容器的 STDOUT 和 STDERR。

```console
$ az container attach --resource-group $RES_GROUP --name acr-tasks
Container 'acr-tasks' is in state 'Running'...
(count: 1) (last timestamp: 2018-08-22 18:39:10+00:00) pulling image "myregistry.azurecr.io/helloacrtasks:v1"
(count: 1) (last timestamp: 2018-08-22 18:39:15+00:00) Successfully pulled image "myregistry.azurecr.io/helloacrtasks:v1"
(count: 1) (last timestamp: 2018-08-22 18:39:17+00:00) Created container
(count: 1) (last timestamp: 2018-08-22 18:39:17+00:00) Started container

Start streaming logs:
Server running at http://localhost:80
```

當 `Server running at http://localhost:80` 出現時，請瀏覽至容器在您瀏覽器中的 FQDN，以查看執行中的應用程式。 FQDN 應已顯示在您在上一節中執行的 `az container create` 命令所產生的輸出中。

![呈現在瀏覽器中的範例應用程式的螢幕擷取畫面][quick-build-02-browser]

若要從容器將您的主控台中斷連結，請點按 `Control+C`。

## <a name="clean-up-resources"></a>清除資源

使用 [az container delete][az-container-delete] 命令停止容器執行個體：

```azurecli-interactive
az container delete --resource-group $RES_GROUP --name acr-tasks
```

若要移除您在本教學課程中建立的*所有*資源 (包括容器登錄、金鑰保存庫和服務主體)，請發出下列命令。 不過，本系列的[下一個教學課程](container-registry-tutorial-build-task.md)將會使用這些資源，因此如果您要直接移至下一個教學課程，可以選擇加以保留。

```azurecli-interactive
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## <a name="next-steps"></a>後續步驟

現在，您已使用快速工作測試您的內部迴圈，接下來請設定您將原始程式碼認可至 Git 存放庫時用來觸發容器映像建置的**建置工作**：

> [!div class="nextstepaction"]
> [使用工作觸發自動建置](container-registry-tutorial-build-task.md)

<!-- LINKS - External -->
[sample-archive]: https://github.com/Azure-Samples/acr-build-helloworld-node/archive/master.zip

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-create]: /cli/azure/container#az-container-create
[az-container-delete]: /cli/azure/container#az-container-delete
[az-keyvault-create]: /cli/azure/keyvault/secret#az-keyvault-create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[az-login]: /cli/azure/reference-index#az-login
[service-principal-auth]: container-registry-auth-service-principal.md

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
