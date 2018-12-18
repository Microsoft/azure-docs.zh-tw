---
title: 在使用自訂映像 (預覽版) 的 Linux 上建立函式 | Microsoft Docs
description: 了解如何建立在自訂 Linux 映像上執行的 Azure Functions。
services: functions
keywords: ''
author: ggailey777
ms.author: glenga
ms.date: 11/15/2017
ms.topic: tutorial
ms.service: azure-functions
ms.custom: mvc
ms.devlang: azure-cli
manager: jeconnoc
ms.openlocfilehash: a77018d5ee1738f24518742c2386e6e261a7c6a8
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48901443"
---
# <a name="create-a-function-on-linux-using-a-custom-image-preview"></a>在使用自訂映像 (預覽版) 的 Linux 上建立函式

Azure Functions 可讓您在 Linux 的自訂容器中裝載函式。 您也可以[在預設的 Azure App Service 容器上裝載](functions-create-first-azure-function-azure-cli-linux.md)。 這項功能目前為預覽狀態並且需要 [Functions 2.0 執行階段](functions-versions.md)。

在本教學課程中，您將了解如何部署函式應用程式作為自訂 Docker 映像。 當您需要自訂內建的 App Service 容器映像時，此模式相當有用。 當您的函式需要特定的語言版本，或需要內建映像未提供的特定相依性或設定時，您可能會想使用自訂映像。

本教學課程會引導您如何使用 Azure Functions 來建立和推送自訂映像至 Docker Hub。 然後使用此映像作為 Linux 上執行之函式應用程式的部署來源。 使用 Docker 建置和推送映像。 使用 Azure CLI 來建立函式應用程式，並從 Docker Hub 部署映像。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 使用 Docker 建置自訂映像。
> * 將自訂映像發佈到容器登錄中。 
> * 建立 Azure 儲存體帳戶。 
> * 建立 Linux App Service 方案。 
> * 從 Docker Hub 部署函式應用程式。
> * 將應用程式設定加入函式應用程式。 

下列步驟適用於 Mac、Windows 或 Linux 電腦。  

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要：

* [Git](https://git-scm.com/downloads)
* 有效的 [Azure 訂用帳戶](https://azure.microsoft.com/pricing/free-trial/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
* [Docker](https://docs.docker.com/install/)
* [Docker Hub 帳戶](https://docs.docker.com/docker-id/)

[!INCLUDE [Free trial note](../../includes/quickstarts-free-trial-note.md)]

## <a name="download-the-sample"></a>下載範例

在終端機視窗中執行下列命令，將範例應用程式存放庫複製到本機電腦，然後變更為包含範例程式碼的目錄。

```bash
git clone https://github.com/Azure-Samples/functions-linux-custom-image.git --config core.autocrlf=input
cd functions-linux-custom-image
```

## <a name="build-the-image-from-the-docker-file"></a>從 Docker 檔案建立映像

在此 Git 存放庫中，看看 _Dockerfile_。 此檔案描述在 Linux 上執行函式應用程式所需的環境。 

```docker
# Base the image on the built-in Azure Functions Linux image.
FROM microsoft/azure-functions-runtime:2.0.0-jessie
ENV AzureWebJobsScriptRoot=/home/site/wwwroot

# Add files from this repo to the root site folder.
COPY . /home/site/wwwroot 
```
>[!NOTE]
> 在私人容器登錄中裝載映像時，您應該使用 Dockerfile 中的 **ENV** 變數將連線設定加入函式應用程式。 本教學課程無法保證您使用私人登錄，因此[在部署後使用 Azure CLI 加入](#configure-the-function-app)連線設定是最佳安全性做法。   

### <a name="run-the-build-command"></a>執行建置命令
若要建置 Docker 映像，請執行 `docker build` 命令，然後提供名稱、`mydockerimage` 和標記、`v1.0.0`。 將 `<docker-id>` 取代為 Docker Hub 帳戶識別碼。

```bash
docker build --tag <docker-id>/mydockerimage:v1.0.0 .
```

命令會產生類似下列的輸出：

```bash
Sending build context to Docker daemon  169.5kB
Step 1/3 : FROM microsoft/azure-functions-runtime:v2.0.0-jessie
v2.0.0-jessie: Pulling from microsoft/azure-functions-runtime
b178b12f7913: Pull complete
2d9ce077a781: Pull complete
4775d4ba55c8: Pull complete
Digest: sha256:073f45fc167b3b5c6642ef4b3c99064430d6b17507095...
Status: Downloaded newer image for microsoft/azure-functions-runtime:v2.0.0-jessie
 ---> 217799efa500
Step 2/3 : ENV AzureWebJobsScriptRoot /home/site/wwwroot
 ---> Running in 528fa2077d17
 ---> 7cc6323b8ae0
Removing intermediate container 528fa2077d17
Step 3/3 : COPY . /home/site/wwwroot
 ---> 5bdac9878423
Successfully built 5bdac9878423
Successfully tagged ggailey777/mydockerimage:v1.0.0
```

### <a name="test-the-image-locally"></a>在本機測試映像
要確認建立的映像是否運作的方式，是在本機容器中執行 Docker 映像。 發出 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令，並將映像的名稱和標記傳遞給它。 請務必使用 `-p` 引數來指定連接埠。

```bash
docker run -p 8080:80 -it <docker-ID>/mydockerimage:v1.0.0
```

在本機 Docker 容器中執行自訂映像，瀏覽至 <http://localhost:8080> 以確認函式應用程式和容器是否正常運作。

![在本機測試函式應用程式。](./media/functions-create-function-linux-custom-image/run-image-local-success.png)

在容器中驗證過函式應用程式容器之後，停止執行。 現在，您可以將自訂映像推送到 Docker Hub 帳戶。

## <a name="push-the-custom-image-to-docker-hub"></a>將自訂映像推送至 Docker Hub

登錄是裝載映像並提供服務映像和容器服務的應用程式。 您必須將映像推送到登錄，才能共用您的映像。 Docker Hub 是 Docker 映像的登錄，可讓您裝載自己的公用或私人存放庫。 

推送映像之前，必須先使用 [docker login](https://docs.docker.com/engine/reference/commandline/login/) 命令登入 Docker Hub。 在命令提示字元中，以您的帳戶名稱取代 `<docker-id>`，並於主控台輸入您的密碼。 如需其他 Docker Hub 密碼選項，請參閱 [docker login 命令文件](https://docs.docker.com/engine/reference/commandline/login/) \(英文\)。

```bash
docker login --username <docker-id> 
```

「登入成功」訊息會確認您已登入。 登入之後，使用 [docker push](https://docs.docker.com/engine/reference/commandline/push/) 命令將映像推送到 Docker Hub。

```bash
docker push <docker-id>/mydockerimage:v1.0.0
```

檢查命令的輸出來確認已成功推送。

```bash
The push refers to a repository [docker.io/<docker-id>/mydockerimage:v1.0.0]
24d81eb139bf: Pushed
fd9e998161c9: Mounted from microsoft/azure-functions-runtime
e7796c35add2: Mounted from microsoft/azure-functions-runtime
ae9a05b85848: Mounted from microsoft/azure-functions-runtime
45c86e20670d: Mounted from microsoft/azure-functions-runtime
v1.0.0: digest: sha256:be080d80770df71234eb893fbe4d... size: 2422
```
現在，您可以使用此映像作為 Azure 中的新函式應用程式的部署來源。 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本主題需要 Azure CLI 2.0.21 版或更新版本。 執行 `az --version` 以尋找您擁有的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

[!INCLUDE [functions-create-resource-group](../../includes/functions-create-resource-group.md)]

[!INCLUDE [functions-create-storage-account](../../includes/functions-create-storage-account.md)]

## <a name="create-a-linux-app-service-plan"></a>建立 Linux App Service 方案

使用方案目前不支援以 Linux 裝載 Functions。 您必須在 Linux App Service 方案上執行。 若要深入了解裝載，請參閱 [Azure Functions 裝載方案比較](functions-scale.md)。 

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]


## <a name="create-and-deploy-the-custom-image"></a>建立和部署自訂映像

函式應用程式可裝載函式的執行。 使用 [az functionapp create](/cli/azure/functionapp#az-functionapp-create) 命令從 Docker Hub 映像建立函式應用程式。 

在下列命令中，使用唯一函式應用程式名稱來替代您看見 `<app_name>` 預留位置的地方，並使用儲存體帳戶名稱來替代 `<storage_name>`。 `<app_name>` 會作為函式應用程式的預設 DNS 網域，所以此名稱在 Azure 的所有應用程式中都必須是唯一的名稱。 如先前所述，`<docker-id>` 是您的 Docker 帳戶名稱。

```azurecli-interactive
az functionapp create --name <app_name> --storage-account  <storage_name>  --resource-group myResourceGroup \
--plan myAppServicePlan --deployment-container-image-name <docker-id>/mydockerimage:v1.0.0 
```
建立函式應用程式後，Azure CLI 會顯示類似下列範例的資訊：

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "containerSize": 1536,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "quickstart.azurewebsites.net",
  "enabled": true,
  "enabledHostNames": [
    "quickstart.azurewebsites.net",
    "quickstart.scm.azurewebsites.net"
  ],
   ....
    // Remaining output has been truncated for readability.
}
```

_deployment-container-image-name_ 參數表示裝載於 Docker Hub 上用於建立函式應用程式的映像。 


## <a name="configure-the-function-app"></a>設定函式應用程式

此函式需要連接字串以連接到預設儲存體帳戶。 當您將自訂映像發佈至私人容器帳戶時，應使用 [ENV 指令](https://docs.docker.com/engine/reference/builder/#env) \(英文\) 或對等指令將 Dockerfile 中的這些應用程式設定改設為環境變數。 

在本例中，`<storage_account>` 是您建立的儲存體帳戶名稱。 使用 [az storage account show-connection-string](/cli/azure/storage/account#show-connection-string) 命令取得連接字串。 使用 [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings#az-functionapp-config-appsettings-set) 命令，在函式應用程式中新增這些應用程式設定。

```azurecli-interactive
storageConnectionString=$(az storage account show-connection-string \
--resource-group myResourceGroup --name <storage_account> \
--query connectionString --output tsv)

az functionapp config appsettings set --name <function_app> \
--resource-group myResourceGroup \
--settings AzureWebJobsDashboard=$storageConnectionString \
AzureWebJobsStorage=$storageConnectionString
```

您現在可以在 Azure 中測試在 Linux 上執行的函式。

[!INCLUDE [functions-test-function-code](../../includes/functions-test-function-code.md)]

[!INCLUDE [functions-cleanup-resources](../../includes/functions-cleanup-resources.md)]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用 Docker 建置自訂映像。
> * 將自訂映像發佈到容器登錄中。 
> * 建立 Azure 儲存體帳戶。 
> * 建立 Linux App Service 方案。 
> * 從 Docker Hub 部署函式應用程式。
> * 將應用程式設定加入函式應用程式。

了解如何啟用核心 App Service 平台內建的持續整合功能。 您可以設定函式應用程式，以便當您在 Docker Hub 中更新您的映像時重新部署容器。

> [!div class="nextstepaction"] 
> [使用用於容器的 Web 應用程式進行持續部署](../app-service/containers/app-service-linux-ci-cd.md)
