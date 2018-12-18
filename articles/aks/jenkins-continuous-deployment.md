---
title: 在 Azure Kubernetes Service 中搭配 Kubernetes 進行 Jenkins 持續部署
description: 如何使用 Jenkins 將持續部署自動化，在 Azure Kubernetes Service 中部署和升級 Kubernetes 上的容器化應用程式
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 03/26/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: a1a6799bc049fea829f8e32d12705e26e3a41dc0
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39425753"
---
# <a name="continuous-deployment-with-jenkins-and-azure-kubernetes-service"></a>透過 Jenkins 與 Azure Kubernetes Service 持續部署

本文示範如何設定 Jenkins 與 Azure Kubernetes Service (AKS) 叢集之間的基本持續部署工作流程。

範例工作流程包含下列步驟：

> [!div class="checklist"]
> * 將 Azure 投票應用程式部署至 Kubernetes 叢集。
> * 更新 Azure 投票應用程式程式碼並推送至 GitHub 存放庫，以啟動持續部署程序。
> * Jenkins 會複製存放庫，然後使用更新過的程式碼來組建新的容器映像。
> * 此映像會推送至 Azure Container Registry (ACR)。
> * 在 AKS 叢集中執行的應用程式會更新成新的容器映像。

## <a name="prerequisites"></a>必要條件

您需要下列項目，才能完成本文中的步驟。

- 對 Kubernetes、Git、CI/CD 及 Azure Container Registry (ACR) 的基本了解。
- [Azure Kubernetes Service (AKS) 叢集][aks-quickstart]，並在您的開發系統上設定好 [AKS 認證][aks-credentials]。
- [Azure Container Registry (ACR) 登錄][acr-quickstart]、ACR 登入伺服器名稱，以及具備推送和提取存取權的 [ACR 認證][acr-authentication]。
- 在您的開發系統上安裝 Azure CLI。
- 在您的開發系統上安裝 Docker。
- GitHub 帳號、[GitHub 個人存取權杖][git-access-token]，以及在您的開發系統上安裝 Git 用戶端。

## <a name="prepare-application"></a>準備應用程式

本文件中全程使用的 Azure 投票應用程式包含一個裝載在一或多個 Pod 中的 Web 介面，以及裝載 Redis 來作為暫時資料儲存體的第二個 Pod。

在建置 Jenkins / AKS 整合之前，請先備妥 Azure 投票應用程式並將其部署至 AKS 叢集。 請將此視為應用程式的第一版。

對以下 GitHub 存放庫進行派生。

```
https://github.com/Azure-Samples/azure-voting-app-redis
```

建立派生之後，將其複製到您的開發系統。 複製此存放庫時，請確定您使用的是派生的 URL。

```bash
git clone https://github.com/<your-github-account>/azure-voting-app-redis.git
```

變更目錄以從複製的目錄操作。

```bash
cd azure-voting-app-redis
```

執行 `docker-compose.yaml` 檔案以建立 `azure-vote-front` 容器映像，並啟動應用程式。

```bash
docker-compose up -d
```

完成時，請使用 [docker images][docker-images] 命令來查看所建立的映像。

請注意，已下載或建立三個映像。 `azure-vote-front` 映像含有應用程式，並使用 `nginx-flask` 映像做為基礎映像。 `redis` 映像可用來啟動 Redis 執行個體。

```console
$ docker images

REPOSITORY                   TAG        IMAGE ID            CREATED             SIZE
azure-vote-front             latest     9cc914e25834        40 seconds ago      694MB
redis                        latest     a1b99da73d05        7 days ago          106MB
tiangolo/uwsgi-nginx-flask   flask      788ca94b2313        9 months ago        694MB
```

使用 [az acr list][az-acr-list] 命令來取得 ACR 登入伺服器。 請務必將資源群組名稱更新成裝載您 ACR 登錄的資源群組。

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

使用 [docker tag][docker-tag] 命令來為映像加上登錄伺服器名稱和版本號碼 `v1`的標籤。

```bash
docker tag azure-vote-front <acrLoginServer>/azure-vote-front:v1
```

將 ACR 登入伺服器值更新成您的 ACR 登入伺服器名稱，然後將 `azure-vote-front` 映像推送至登錄。

```bash
docker push <acrLoginServer>/azure-vote-front:v1
```

## <a name="deploy-application-to-kubernetes"></a>將應用程式部署至 Kubernetes

您可以在 Azure 投票存放庫的根目錄找到 Kubernetes 資訊清單檔，並可使用它將應用程式部署至 Kubernetes 叢集。

首先，將 **azure-vote-all-in-one-redis.yaml** 資訊清單檔更新成您 ACR 登錄的位置。 使用任何文字編輯器來開啟檔案，並將 `microsoft` 值取代成 ACR 登入伺服器名稱。 您可以在資訊清單檔的第 **47** 行找到這個值。

```yaml
containers:
- name: azure-vote-front
  image: microsoft/azure-vote-front:v1
```

接下來使用 [kubectl apply][kubectl-apply] 命令來執行應用程式。 此命令會剖析資訊清單檔，並建立已定義的 Kubernetes 物件。

```bash
kubectl apply -f azure-vote-all-in-one-redis.yaml
```

系統會建立一個 [Kubernetes 服務][kubernetes-service]，以將應用程式公開至網際網路。 此程序需要數分鐘的時間。

若要監視進度，請使用 [kubectl get service][kubectl-get] 命令搭配 `--watch` 引數。

```bash
kubectl get service azure-vote-front --watch
```

一開始，*azure-vote-front* 服務的 *EXTERNAL-IP* 會顯示為 *pending*。

```
azure-vote-front   10.0.34.242   <pending>     80:30676/TCP   7s
```

當 *EXTERNAL-IP* 位址從 *pending* 變成一個 *IP 位址*之後，請使用 `control+c` 來停止 kubectl 監看式流程。

```
azure-vote-front   10.0.34.242   13.90.150.118   80:30676/TCP   2m
```

若要查看應用程式，請瀏覽至外部 IP 位址。

![Azure 上 Kubernetes 叢集的影像](media/aks-jenkins/azure-vote-safari.png)

## <a name="deploy-jenkins-to-vm"></a>將 Jenkins 部署至 VM

已預先建立一個指令碼來部署虛擬機器、設定網路存取，以及完成 Jenkins 的基本安裝。 此外，此指令碼也會將 Kubernetes 設定檔從開發系統複製到 Jenkins 系統。 此檔案會用於 Jenkins 與 AKS 叢集之間的驗證。

請執行下列命令來下載及執行指令碼。 以下 URL 也可用來檢閱指令碼的內容。

> [!WARNING]
> 此範例指令碼僅用於示範，說明如何快速佈建可在 Azure VM 上執行的 Jenkins 環境。 它會使用 Azure 自訂指令碼擴充功能來設定 VM，然後顯示所需的認證。 您的 ~/.kube/config 會複製到 Jenkins VM。

```console
curl https://raw.githubusercontent.com/Azure-Samples/azure-voting-app-redis/master/jenkins-tutorial/deploy-jenkins-vm.sh > azure-jenkins.sh
sh azure-jenkins.sh
```

當指令碼完成時，它會輸出 Jenkins 伺服器的位址，以及可將 Jenkins 解除鎖定的金鑰。 瀏覽至 URL、輸入金鑰，然後依照畫面上的提示來完成 Jenkins 設定。

```console
Open a browser to http://52.166.118.64:8080
Enter the following to Unlock Jenkins:
667e24bba78f4de6b51d330ad89ec6c6
```

## <a name="jenkins-environment-variables"></a>Jenkins 環境變數

Jenkins 環境變數可用來存放 Azure Container Registry (ACR) 登入伺服器名稱。 進行 Jenkins 持續部署作業期間會參考此變數。

在 Jenkins 管理入口網站上時，按一下 [Manage Jenkins] \(管理 Jenkins\) > [Configure System] \(設定系統\)。

在 [Global Properties] \(全域屬性\) 底下，選取 [Environment variables] \(環境變數\)，然後新增名稱為 `ACR_LOGINSERVER` 且值為您 ACR 登入伺服器的變數。

![Jenkins 環境變數](media/aks-jenkins/env-variables.png)

完成時，在 Jenkins 設定頁面上按一下 [Save] \(儲存\)。

## <a name="jenkins-credentials"></a>Jenkins 認證

現在，請將 ACR 認證儲存在 Jenkins 認證物件中。 進行 Jenkins 組建作業時，會參考這些認證。

回到 Jenkins 管理入口網站，按一下 [Credentials] \(認證\) > [Jenkins] > [Global credentials (unrestricted)] \(全域認證 (無限制)\) > [Add Credentials] \(新增認證\)。

確定認證種類是 [Username with password] \(使用者名稱搭配密碼\)，然後輸入下列項目：

- **Username** \(使用者名稱\) - 用來向 ACR 登錄進行驗證之服務主體的識別碼。
- **Password** \(密碼\) - 用來向 ACR 登錄進行驗證之服務主體的用戶端密碼。
- **ID** \(識別碼\) - 認證識別碼，例如 `acr-credentials`。

完成時，認證形式應該看起來會類似下圖：

![ACR 認證](media/aks-jenkins/acr-credentials.png)

按一下 [OK] \(確定\)，然後返回 Jenkins 管理入口網站。

## <a name="create-jenkins-project"></a>建立 Jenkins 專案

從 Jenkins 管理入口網站中，按一下 [New Item] \(新增項目\)。

輸入專案名稱 (例如 `azure-vote`)，選取 [Freestyle Project] \(Freestyle 專案\)，然後按一下 [OK] \(確定\)。

![Jenkins 專案](media/aks-jenkins/jenkins-project.png)

在 [General] \(一般\) 底下，選取 [GitHub project] \(GitHub 專案\)，然後輸入您 Azure 投票 GitHub 專案派生的 URL。

![GitHub 專案](media/aks-jenkins/github-project.png)

在 [Source Code Management] \(原始程式碼管理\) 底下，選取 [Git]，然後輸入 Azure 投票 GitHub 存放庫派生的 URL。

針對認證，按一下 [Add] \(新增\) > [Jenkins]。 在 [Kind] \(種類\) 底下，選取 [Secret text] \(祕密文字\)，然後輸入您的 [GitHub 個人存取權杖][git-access-token]作為祕密。

完成時，選取 [Add] \(新增\)。

![GitHub 認證](media/aks-jenkins/github-creds.png)

在 [Build Triggers] \(建置觸發程序\) 底下，選取 [GitHub hook trigger for GITScm polling] \(GITScm 輪詢的 GitHub 連結觸發程序\)。

![Jenkins 建置觸發程序](media/aks-jenkins/build-triggers.png)

在 [Build Environment] \(建置環境\)底下，選取 [Use secret texts or files] \(使用祕密文字或檔案\)。

![Jenkins 建置環境](media/aks-jenkins/build-environment.png)

在 [Bindings] \(繫結\) 底下，選取 [Add] \(新增\) > [Username and password (separated)] \(使用者名稱和密碼 (分隔)\)。

針對 [Username Variable] \(使用者名稱變數\) 輸入 `ACR_ID`，並針對 [Password Variable] \(密碼變數\)輸入 `ACR_PASSWORD`。

![Jenkins 繫結](media/aks-jenkins/bindings.png)

新增一個 [Execute shell] \(執行殼層\) 類型的 [Build Step] \(建置步驟\)，然後使用下列文字。 此指令碼會建置新的容器映像，然後將其推送至 ACR 登錄。

```bash
# Build new image and push to ACR.
WEB_IMAGE_NAME="${ACR_LOGINSERVER}/azure-vote-front:kube${BUILD_NUMBER}"
docker build -t $WEB_IMAGE_NAME ./azure-vote
docker login ${ACR_LOGINSERVER} -u ${ACR_ID} -p ${ACR_PASSWORD}
docker push $WEB_IMAGE_NAME
```

新增另一個 [Execute shell] \(執行殼層\) 類型的 [Build Step] \(建置步驟\)，然後使用下列文字。 此指令碼會更新 Kubernetes 部署。

```bash
# Update kubernetes deployment with new image.
WEB_IMAGE_NAME="${ACR_LOGINSERVER}/azure-vote-front:kube${BUILD_NUMBER}"
kubectl set image deployment/azure-vote-front azure-vote-front=$WEB_IMAGE_NAME --kubeconfig /var/lib/jenkins/config
```

完成後，按一下 [Save] \(儲存\)。

## <a name="test-the-jenkins-build"></a>測試 Jenkins 組建

繼續進行之前，請先測試 Jenkins 組建。 這會驗證是否已正確設定組建作業、已備妥適當的 Kubernetes 驗證檔案，以及已提供適當的 ACR 認證。

按一下專案左側功能表中的 [Build Now] \(立即組建\)。

![Jenkins 測試組建](media/aks-jenkins/test-build.png)

在進行此程序的期間，會將 GitHub 存放庫複製到 Jenkins 組建伺服器。 系統會建立一個新的容器映像並推送至 ACR 登錄。 最後，在 AKS 叢集上執行的 Azure 投票應用程式會更新成使用新的映像。 由於並沒有對應用程式程式碼進行任何變更，因此並未變更應用程式。

在此程序完成後，按一下組建記錄底下的「組建 #1」，然後選取 [主控台輸出] 以查看組建程序的所有輸出。 最後一行應該會指出組建成功。

## <a name="create-github-webhook"></a>建立 GitHub webhook

接下來，請將應用程式存放庫連結至 Jenkins 組建伺服器，以便在進行任何認可時，觸發新的組建。

1. 瀏覽至派生的 GitHub 存放庫。
2. 選取 [設定]，然後選取左側的 [Webhook]。
3. 選擇 [新增 Webhook]。 針對「承載 URL」，輸入 `http://<publicIp:8080>/github-webhook/`，其中 `publicIp` 是 Jenkins 伺服器的 IP 位址。 請務必包含尾端的斜線 (/)。 保留內容類型的其他預設值，並針對「推送」事件觸發。
4. 選取 [新增 Webhook]。

    ![GitHub webhook](media/aks-jenkins/webhook.png)

## <a name="test-cicd-process-end-to-end"></a>進行 CI/CD 程序端對端測試

在您的開發電腦上，使用程式碼編輯器來開啟所複製的應用程式。

在 **/azure-vote/azure-vote** 目錄底下，尋找名為 **config_file.cfg** 的檔案。 請將此檔案中的 vote 值更新成 cats 和 dogs 以外的值。

以下範例顯示一個已更新的 **config_file.cfg** 檔案。

```bash
# UI Configurations
TITLE = 'Azure Voting App'
VOTE1VALUE = 'Blue'
VOTE2VALUE = 'Purple'
SHOWHOST = 'false'
```

完成時，請儲存檔案、認可變更，然後將這些變更推送至您的 GitHub 存放庫派生。 認可完成後，GitHub Webhook 會觸發一個新的 Jenkins 組建，這會更新容器映像和 AKS 部署。 請在 Jenkins 管理主控台上監視組建程序。

組建完成時，請再次瀏覽至應用程式端點來觀察變更。

![更新後的 Azure 投票](media/aks-jenkins/azure-vote-updated-safari.png)

此時，已完成一個簡單的持續部署程序。 本範例中顯示的步驟和設定，可用來組建出一個更健全且符合生產需求的持續組建自動化作業。

<!-- LINKS - external -->
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[git-access-token]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-service]: https://kubernetes.io/docs/concepts/services-networking/service/

<!-- LINKS - internal -->
[az-acr-list]: /cli/azure/acr#az-acr-list
[acr-authentication]: ../container-registry/container-registry-auth-aks.md
[acr-quickstart]: ../container-registry/container-registry-get-started-azure-cli.md
[aks-credentials]: /cli/azure/aks#az-aks-get-credentials
[aks-quickstart]: kubernetes-walkthrough.md
[azure-cli-install]: /cli/azure/install-azure-cli