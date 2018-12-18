---
title: 針對 Web App for Containers 使用自訂 Docker 映像 - Azure | Microsoft Docs
description: 如何針對用於容器的 Web 應用程式使用自訂 Docker 映像。
keywords: azure app service, web 應用程式, linux, docker, 容器
services: app-service
documentationcenter: ''
author: SyntaxC4
manager: SyntaxC4
editor: ''
ms.assetid: b97bd4e6-dff0-4976-ac20-d5c109a559a8
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/24/2017
ms.author: cfowler
ms.custom: mvc
ms.openlocfilehash: 517926a3c78e9c95a9ecef9ae058c33e47566adb
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432602"
---
# <a name="use-a-custom-docker-image-for-web-app-for-containers"></a>針對用於容器的 Web 應用程式使用自訂 Docker 映像

[Web App for Containers](app-service-linux-intro.md) 在 Linux 上提供內建的 Docker 映像，且支援特定的版本，例如 PHP 7.0 和 Node.js 4.5。 適用於容器的 Web 應用程式會使用 Docker 容器技術，來裝載內建映像和自訂映像作為平台即服務。 在本教學課程中，您可以了解如何建置自訂的 Docker 映像，並將它部署至適用於容器的 Web 應用程式。 當內建的映像不包含您所選擇的語言，或當您應用程式所需的特定組態未在內建的映像中提供時，此模式相當有用。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 將自訂 Docker 映像部署至 Azure
> * 設定環境變數來執行容器
> * 更新 Docker 映像並加以重新部署
> * 使用 SSH 連線到容器
> * 將私人 Docker 映像部署至 Azure

[!INCLUDE [Free trial note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要：

* [Git](https://git-scm.com/downloads)
* [Docker](https://docs.docker.com/get-started/#setup)
* [Docker Hub 帳戶](https://docs.docker.com/docker-id/)

## <a name="download-the-sample"></a>下載範例

在終端機視窗中執行下列命令，將範例應用程式存放庫複製到本機電腦，然後變更為包含範例程式碼的目錄。

```bash
git clone https://github.com/Azure-Samples/docker-django-webapp-linux.git --config core.autocrlf=input
cd docker-django-webapp-linux
```

## <a name="build-the-image-from-the-docker-file"></a>從 Docker 檔案建立映像

在 Git 存放庫中，看看 _Dockerfile_。 此檔案會描述執行您應用程式所需的 Python 環境。 此外，映像會設定 [SSH](https://www.ssh.com/ssh/protocol/) 伺服器，以在容器與主機之間進行安全通訊。

```docker
FROM python:3.4

RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/

# ssh
ENV SSH_PASSWD "root:Docker!"
RUN apt-get update \
        && apt-get install -y --no-install-recommends dialog \
        && apt-get update \
    && apt-get install -y --no-install-recommends openssh-server \
    && echo "$SSH_PASSWD" | chpasswd 

COPY sshd_config /etc/ssh/
COPY init.sh /usr/local/bin/
    
RUN chmod u+x /usr/local/bin/init.sh
EXPOSE 8000 2222
#CMD ["python", "/code/manage.py", "runserver", "0.0.0.0:8000"]
ENTRYPOINT ["init.sh"]
```

若要建置 Docker 映像，請執行 `docker build` 命令，然後提供名稱 (mydockerimage) 和標記 (v1.0.0)。 將 _\<docker-id>_ 替換為 Docker Hub 帳戶識別碼。

```bash
docker build --tag <docker-id>/mydockerimage:v1.0.0 .
```

命令會產生類似下列的輸出：

```
# The output from the commands in this article has been shortened for brevity.

Sending build context to Docker daemon  5.558MB
Step 1/13 : FROM python:3.4
 ---> 9ff45ddb54e9
Step 2/13 : RUN mkdir /code
 ---> Using cache
 ---> f3f3ac01db0a
Step 3/13 : WORKDIR /code
 ---> Using cache
 ---> 38b32f15b442
.
.
.
Step 13/13 : ENTRYPOINT init.sh
 ---> Running in 5904e4c70043
 ---> e7cf08275692
Removing intermediate container 5904e4c70043
Successfully built e7cf08275692
Successfully tagged cephalin/mydockerimage:v1.0.0
```

測試組建運作的方式是執行 Docker 容器。 發出 [`docker run`](https://docs.docker.com/engine/reference/commandline/run/) 命令，並將映像的名稱和標記傳遞給它。 請務必使用 `-p` 引數來指定連接埠。

```bash
docker run -p 2222:8000 <docker-ID>/mydockerimage:v1.0.0
```

驗證 web 應用程式和容器是否會正常運作，方法是瀏覽至 `http://localhost:2222`。

![在本機測試 Web 應用程式](./media/app-service-linux-using-custom-docker-image/app-service-linux-browse-local.png)

> [!NOTE] 
> 您也可以使用 SSH、SFTP 或 Visual Studio Code (適用於 Node.js 應用程式即時偵錯)，直接從本機開發電腦連線到應用程式容器。 如需詳細資訊，請參閱 [Linux 上 App Service 中的遠端偵錯和 SSH](https://aka.ms/linux-debug)。
>

## <a name="push-the-docker-image-to-docker-hub"></a>將 Docker 映像推送至 Docker Hub

登錄是裝載映像並提供服務映像和容器服務的應用程式。 您必須將映像推送到登錄，才能共用您的映像。 

<!-- Depending on your requirements, you may have your docker images in a Public Docker Registry, such as Docker Hub, or a Private Docker Registry such as Azure Container Registry. Select the appropriate tab for your scenario below (your selection will switch multiple tabs on this page). -->

> [!NOTE]
> 推送到私人 Docker 登錄嗎？ 請參閱選擇性指示以[使用任何私人登錄的 Docker 映像](#use-a-docker-image-from-any-private-registry-optional)。

<!--## [Docker Hub](#tab/docker-hub)-->

Docker Hub 是 Docker 映像的登錄，可讓您裝載自己的公用或私人存放庫。 若要將自訂 Docker 映像推送到公用 Docker Hub，請使用 [`docker push`](https://docs.docker.com/engine/reference/commandline/push/) 命令，並提供完整的映像名稱和標記。 完整的映像名稱和標記看起來像下列範例：

```
<docker-id>/image-name:tag
```

您必須先使用 [`docker login`](https://docs.docker.com/engine/reference/commandline/login/) 命令登入 Docker Hub，才能推送映像。 以您的帳戶名稱取代 _\<docker-id>_，並在出現提示時，於主控台輸入您的密碼。

```bash
docker login --username <docker-id>
```

「登入成功」訊息會確認您已登入。 一旦登入之後，您可以使用 [`docker push`](https://docs.docker.com/engine/reference/commandline/push/) 命令將映像推送到 Docker Hub。

```bash
docker push <docker-id>/mydockerimage:v1.0.0
```

檢查命令的輸出來確認已成功推送。

```
The push refers to a repository [docker.io/<docker-id>/mydockerimage:v1.0.0]
c33197c3f6d4: Pushed
ccd2c850ee43: Pushed
02dff2853466: Pushed
6ce78153632a: Pushed
efef3f03cc58: Pushed
3439624d77fb: Pushed
3a07adfb35c5: Pushed
2fcec228e1b7: Mounted from library/python
97d2d3bae505: Mounted from library/python
95aadeabf504: Mounted from library/python
b456afdc9996: Mounted from library/python
d752a0310ee4: Mounted from library/python
db64edce4b5b: Mounted from library/python
d5d60fc34309: Mounted from library/python
c01c63c6823d: Mounted from library/python
v1.0.0: digest: sha256:21f2798b20555f4143f2ca0591a43b4f6c8138406041f2d32ec908974feced66 size: 3676
```

<!--
# [Private Registry](#tab/private-registry)

// Move Private Registry instructions here when Tabbed Conceptual bug is fixed

---
-->

[!INCLUDE [Try Cloud Shell](../../../includes/cloud-shell-try-it.md)]

## <a name="deploy-app-to-azure"></a>將應用程式部署到 Azure

您可以使用 Azure Web 應用程式在雲端中裝載原生 Linux 應用程式。 若要建立 Web App for Containers，您必須執行 Azure CLI 命令，可建立群組、然後是服務方案，以及最後是 web 應用程式本身。 

### <a name="create-a-resource-group"></a>建立資源群組

[!INCLUDE [Create resource group](../../../includes/app-service-web-create-resource-group-linux-no-h.md)] 

### <a name="create-a-linux-app-service-plan"></a>建立 Linux App Service 方案

[!INCLUDE [Create app service plan](../../../includes/app-service-web-create-app-service-plan-linux-no-h.md)] 

### <a name="create-a-web-app"></a>建立 Web 應用程式

在 Cloud Shell 中，使用 [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) 命令，在 `myAppServicePlan` App Service 方案中建立 [Web 應用程式](app-service-linux-intro.md)。 別忘了將 _<appname>_ 取代成唯一的應用程式名稱，並將 _\<docker-ID>_ 取代成您的 Docker 識別碼。

```azurecli-interactive
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --deployment-container-image-name <docker-ID>/mydockerimage:v1.0.0
```

建立 Web 應用程式後，Azure CLI 會顯示類似下列範例的輸出：

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "deploymentLocalGitUrl": "https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

### <a name="configure-environment-variables"></a>設定環境變數

大部分的 Docker 映像都具有必須設定的環境變數。 如果您使用的是其他人所建置的現有 Docker 映像，映像可能會使用 80 以外的連接埠。 可告訴 Azure 您映像所使用的連接埠，方法是使用 `WEBSITES_PORT` 應用程式設定。 [本教學課程中的 Python 範例](https://github.com/Azure-Samples/docker-django-webapp-linux) GitHub 頁面說明您必須將 `WEBSITES_PORT` 設定為 _8000_。

若要設定應用程式的設定，請在 Cloud Shell 中使用 [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 應用程式設定為區分大小寫和空格分隔。

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app_name> --settings WEBSITES_PORT=8000
```

<!-- Depending on your requirements, you may have your docker images in a Public Docker Registry, such as Docker Hub, or a Private Docker Registry, such as Azure Container Registry. Select the appropriate tab for your scenario below: -->

> [!NOTE]
> 從私人 Docker 登錄部署？ 請參閱選擇性指示以[使用任何私人登錄的 Docker 映像](#use-a-docker-image-from-any-private-registry-optional)。

<!-- # [Docker Hub](#tab/docker-hub)-->

<!-- # [Private Registry](#tab/private-registry)

// Place Private Registry text back here once Tabbed Conceptual bug is fixed

---
-->

### <a name="test-the-web-app"></a>測試 Web 應用程式

瀏覽至 Web 應用程式以確認它可運作 (`http://<app_name>azurewebsites.net`)。 

![測試 web 應用程式連接埠設定](./media/app-service-linux-using-custom-docker-image/app-service-linux-browse-azure.png)

## <a name="change-web-app-and-redeploy"></a>變更 Web 應用程式並重新部署

在您的本機 Git 存放庫中，開啟 app/templates/app/index.html。 找出第一個 HTML 元素，並將它加以變更。

```python
<nav class="navbar navbar-inverse navbar-fixed-top">
    <div class="container">
      <div class="navbar-header">         
        <a class="navbar-brand" href="#">Azure App Service - Updated Here!</a>       
      </div>            
    </div>
  </nav> 
```

一旦您修改了 Python 檔案並加以儲存後，必須重建並推送新的 Docker 映像。 接著，重新啟動 Web 應用程式，變更才會生效。 使用您先前已在本教學課程中使用的相同命令。 您可以參考[從 Docker 檔案建立映像](#build-the-image-from-the-docker-file)和[將 Docker 映像推送至 Docker Hub](#push-the-docker-image-to-docker-hub)。 遵循[測試 Web 應用程式](#test-the-web-app)中的指示來測試 web 應用程式。

## <a name="connect-to-web-app-for-containers-using-ssh"></a>使用 SSH 連線到 Web App for Containers

SSH 可讓容器和用戶端之間進行安全通訊。 您必須將自訂的 Docker 映像建置到 Dockerfile，它才能支援 SSH。 您要在 Docker 檔案本身啟用 SSH。 已將 SSH 指示新增至範例 Dockerfile 中，因此您可以使用自己的自訂映像來遵循這些指示：

* [執行](https://docs.docker.com/engine/reference/builder/#run)指示會呼叫 `apt-get`，然後將根帳戶的密碼設為 `"Docker!"`。

    ```docker
    ENV SSH_PASSWD "root:Docker!"
    RUN apt-get update \
            && apt-get install -y --no-install-recommends dialog \
            && apt-get update \
      && apt-get install -y --no-install-recommends openssh-server \
      && echo "$SSH_PASSWD" | chpasswd 
    ```

    > [!NOTE]
    > 此設定不允許容器的外部連線。 SSH 只能透過 Kudu/SCM 站台提供。 Kudu/SCM 站台會向發佈認證進行驗證。

* [複製](https://docs.docker.com/engine/reference/builder/#copy)指示會指示 Docker 引擎將 [sshd_config](http://man.openbsd.org/sshd_config) 檔案複製到 /etc/ssh/ 目錄。 您的組態檔應該以 [這個 sshd_config 檔案](https://github.com/Azure-App-Service/node/blob/master/6.11.1/sshd_config)作為基礎。

    ```docker
    COPY sshd_config /etc/ssh/
    ```

    > [!NOTE]
    > sshd_config 檔案必須包含下列項目︰ 
    > * `Ciphers` 必須在此清單中包含至少一個項目：`aes128-cbc,3des-cbc,aes256-cbc`。
    > * `MACs` 必須在此清單中包含至少一個項目：`hmac-sha1,hmac-sha1-96`。

* [公開](https://docs.docker.com/engine/reference/builder/#expose)指示會在容器中公開連接埠 2222。 雖然已知根密碼，但無法從網際網路存取連接埠 2222。 它是供內部使用的連接埠，只有私人虛擬網路之橋接網路內的容器可以存取。 在這之後，命令會複製 SSH 設定詳細資料並啟動 `ssh` 服務。

    ```docker
    EXPOSE 8000 2222
    ```

* 請確定使用 /bin 目錄中的殼層指令碼，來[啟動 ssh 服務](https://github.com/Azure-App-Service/node/blob/master/6.9.3/startup/init_container.sh)。
 
    ```bash
    #!/bin/bash
    service ssh start
    ```
     
### <a name="open-ssh-connection-to-container"></a>開啟對容器的 SSH 連線

適用於容器的 Web 應用程式不允許容器的外部連線。 SSH 只能透過 Kudu 站台提供，可在 `https://<app_name>.scm.azurewebsites.net` 存取。

若要連線，請瀏覽至 `https://<app_name>.scm.azurewebsites.net/webssh/host` 並以您的 Azure 帳戶登入。

然後，系統會將您重新導向至一個顯示互動式主控台的頁面。 

您需要確認特定應用程式正在容器中執行。 若要檢查容器並確認執行的流程，在提示字元提出 `top` 命令。

```bash
top
```

`top` 命令會公開容器中所有執行中的程序。

```
PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 1 root      20   0  945616  35372  15348 S  0.0  2.1   0:04.63 node
20 root      20   0   55180   2776   2516 S  0.0  0.2   0:00.00 sshd
42 root      20   0  944596  33340  15352 S  0.0  1.9   0:05.80 node /opt/s+
56 root      20   0   59812   5244   4512 S  0.0  0.3   0:00.93 sshd
58 root      20   0   20228   3128   2664 S  0.0  0.2   0:00.00 bash
62 root      20   0   21916   2272   1944 S  0.0  0.1   0:03.15 top
63 root      20   0   59812   5344   4612 S  0.0  0.3   0:00.03 sshd
65 root      20   0   20228   3140   2672 S  0.0  0.2   0:00.00 bash
71 root      20   0   59812   5380   4648 S  0.0  0.3   0:00.02 sshd
73 root      20   0   20228   3160   2696 S  0.0  0.2   0:00.00 bash
77 root      20   0   21920   2304   1972 R  0.0  0.1   0:00.00 top
```

恭喜！ 您已針對「用於容器的 Web App」設定自訂 Docker 映像。

## <a name="use-a-private-image-from-docker-hub-optional"></a>從 Docker Hub 使用私人映像(選擇性)

在[建立 Web 應用程式](#create-a-web-app)中，您可以在 `az webapp create` 命令中指定 Docker Hub 上的映像。 這很適用於公用映像。 若要使用私人映像，您必須在 Azure Web 應用程式中設定您的 Docker 帳戶識別碼和密碼。

在 Cloud Shell 中，`az webapp create` 命令後面緊接著 [`az webapp config container set`](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set)。 將 \<app_name>，以及 _\<docker-id>_ 和 _\<password>_ 替換為您的 Docker 識別碼與密碼。

```azurecli-interactive
az webapp config container set --name <app_name> --resource-group myResourceGroup --docker-registry-server-user <docker-id> --docker-registry-server-password <password>
```

此命令顯示的輸出類似下列的 JSON 字串，顯示設定變更成功：

```json
[
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "false"
  },
  {
    "name": "DOCKER_REGISTRY_SERVER_USERNAME",
    "slotSetting": false,
    "value": "<docker-id>"
  },
  {
    "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
    "slotSetting": false,
    "value": null
  },
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "DOCKER|<image-name-and-tag>"
  }
]
```

## <a name="use-a-docker-image-from-any-private-registry-optional"></a>從任何私人登錄使用 Docker 映像 (選擇性)

在本節中，您會了解如何在適用於容器的 Web 應用程式中從私人登錄使用 Docker 映像，並會使用 Azure Container Registry 作為範例。 使用其他私人登錄的步驟很類似。 

Azure Container Registry 是 Azure 中用來裝載私人映像的受控 Docker 服務。 部署可能是任何類型，包括 [Docker Swarm](https://docs.docker.com/engine/swarm/)、[Kubernetes](https://kubernetes.io/) 和適用於容器的 Web 應用程式。 

### <a name="create-an-azure-container-registry"></a>建立 Azure Container Registry

在 Cloud Shell 中，使用 [`az acr create`](/cli/azure/acr?view=azure-cli-latest#az-acr-create) 命令來建立 Azure Container Registry。 將 SKU 的名稱、資源群組和 `Basic` 傳入。 可用的 SKU 為 `Classic`、`Basic`、`Standard` 和 `Premium`。

```azurecli-interactive
az acr create --name <azure-container-registry-name> --resource-group myResourceGroup --sku Basic --admin-enabled true
```

建立容器時，會產生下列輸出：

```
 - Finished ..
Create a new service principal and assign access:
  az ad sp create-for-rbac --scopes /subscriptions/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/<azure-container-registry-name> --role Owner --password <password>

Use an existing service principal and assign access:
  az role assignment create --scope /subscriptions/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/<azure-container-registry-name> --role Owner --assignee <app-id>
{
  "adminUserEnabled": false,
  "creationDate": "2017-08-09T04:21:09.654153+00:00",
  "id": "/subscriptions/<subscriptionId>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/<azure-container-registry-name>",
  "location": "westeurope",
  "loginServer": "<azure-container-registry-name>.azurecr.io",
  "name": "<azure-container-registry-name>",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "storageAccount": {
    "name": "myazurecontainerre042025"
  },
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

### <a name="log-in-to-azure-container-registry"></a>登入 Azure Container Registry

您必須提供認證讓登錄接受推送，才能將映像推送至登錄。 您可以在 Cloud Shell 中使用 [`az acr show`](/cli/azure/acr?view=azure-cli-latest#az-acr-show) 命令來擷取這些認證。 

```azurecli-interactive
az acr credential show --name <azure-container-registry-name>
```

此命令所顯示的兩個密碼可與使用者名稱搭配使用。

```json
<
  "passwords": [
    {
      "name": "password",
      "value": "{password}"
    },
    {
      "name": "password2",
      "value": "{password}"
    }
  ],
  "username": "<registry-username>"
}
```

使用 `docker login` 命令從本機終端機視窗登入 Azure Container Registry。 需要伺服器名稱才能登入。 請使用此格式：`{azure-container-registry-name>.azurecr.io`。 出現提示時，在主控台中輸入您的密碼。

```bash
docker login <azure-container-registry-name>.azurecr.io --username <registry-username>
```

確認登入成功。 

### <a name="push-an-image-to-azure-container-registry"></a>將映像推送至 Azure Container Registry

> [!NOTE]
> 如果您是使用自己的映像，請標記映像，如下所示：
> ```bash
> docker tag <azure-container-registry-name>.azurecr.io/mydockerimage
> ```

使用 `docker push` 命令來推送映像。 以登錄名稱再接著映像名稱和標籤來標記映像。

```bash
docker push <azure-container-registry-name>.azurecr.io/mydockerimage:v1.0.0
```

列出 ACR 存放庫以確認推送已成功將容器新增至登錄。 

```azurecli-interactive
az acr repository list -n <azure-container-registry-name>
```

在登錄中列出映像會確認 `mydockerimage` 位於登錄中。

```json
[
  "mydockerimage"
]
```

### <a name="configure-web-app-to-use-the-image-from-azure-container-registry-or-any-private-registry"></a>將 Web 應用程式設定成使用 Azure Container Registry 中的映像 (或任何私人登錄)

您可以設定適用於容器的 Web 應用程式，讓它們執行儲存在 Azure Container Registry 中的容器。 使用 Azure Container Registry 就如同使用任何私人登錄，因此如果您需要使用自己的私人登錄，完成這項工作的步驟就會很類似。

在 Cloud Shell 中，執行 [`az acr credential show`](/cli/azure/acr/credential?view=azure-cli-latest#az-acr-credential-show) 可顯示 Azure Container Registry 的使用者名稱和密碼。 複製使用者名稱和其中一個密碼，因此您可以使用它來設定下一個步驟中的 web 應用程式。

```bash
az acr credential show --name <azure-container-registry-name>
```

```json
{
  "passwords": [
    {
      "name": "password",
      "value": "password"
    },
    {
      "name": "password2",
      "value": "password2"
    }
  ],
  "username": "<registry-username>"
}
```

在 Cloud Shell 中，執行 [`az webapp config container set`](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set) 命令，將自訂 Docker 映像指派給 Web 應用程式。 取代 *\<app_name>*、*\<docker-registry-server-url>*、_\<registry-username>_ 及 _\<password>_。 針對 Azure Container Registry，\<docker-registry-server-url> 的格式為 `https://<azure-container-registry-name>.azurecr.io`。 如果您使用任何 Docker Hub 以外的登錄，映像名稱必須以登錄的完整網域名稱 (FQDN) 開頭。 若為 Azure Container Registry，這會看起來像 `<azure-container-registry>.azurecr.io/mydockerimage`。 

```azurecli-interactive
az webapp config container set --name <app_name> --resource-group myResourceGroup --docker-custom-image-name <azure-container-registry-name>.azurecr.io/mydockerimage --docker-registry-server-url https://<azure-container-registry-name>.azurecr.io --docker-registry-server-user <registry-username> --docker-registry-server-password <password>
```

> [!NOTE]
> \<docker-registry-server-url>中需要 `https://`。
>

此命令顯示的輸出類似下列的 JSON 字串，顯示設定變更成功：

```json
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "slotSetting": false,
    "value": "mydockerimage"
  },
  {
    "name": "DOCKER_REGISTRY_SERVER_URL",
    "slotSetting": false,
    "value": "<azure-container-registry-name>.azurecr.io"
  },
  {
    "name": "DOCKER_REGISTRY_SERVER_USERNAME",
    "slotSetting": false,
    "value": "<registry-username>"
  },
  {
    "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
    "slotSetting": false,
    "value": null
  }
]
```

[!INCLUDE [Clean-up section](../../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [在 Azure 中建置 Docker Python 和 PostgreSQL Web 應用程式](tutorial-python-postgresql-app.md)
