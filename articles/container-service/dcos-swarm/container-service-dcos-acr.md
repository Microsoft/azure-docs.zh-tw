---
title: 搭配 Azure DC/OS 叢集使用 ACR
description: 在 Azure Container Service 中搭配使用 Azure Container Registry 與 DC/OS 叢集
services: container-service
author: julienstroheker
manager: dcaro
ms.service: container-service
ms.topic: tutorial
ms.date: 03/23/2017
ms.author: juliens
ms.custom: mvc
ms.openlocfilehash: 81b5bd18ea367b68adfc41ee97768b3b28967a42
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986622"
---
# <a name="use-acr-with-a-dcos-cluster-to-deploy-your-application"></a>搭配使用 ACR 與 Azure DC/OS 叢集以部署應用程式

本文探討如何搭配使用 Azure Container Registry 與 DC/OS 叢集。 使用 ACR 可讓您私下儲存和管理容器映像。 本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 部署 Azure Container Registry (如有需要)
> * 在 DC/OS 叢集上設定 ACR 驗證
> * 將映像上傳至 Azure Container Registry
> * 從 Azure Container Registry 執行容器映像

您需要有 ACS DC/OS 叢集，才能完成本教學課程中的步驟。 如有需要，[此指令碼範例](./../kubernetes/scripts/container-service-cli-deploy-dcos.md)可為您建立一個叢集。

本教學課程需要 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="deploy-azure-container-registry"></a>部署 Azure Container Registry

如有需要，使用 [az acr create](/cli/azure/acr#az-acr-create) 命令來建立 Azure Container Registry。 

下列範例會以隨機產生的名稱建立登錄。 使用 `--admin-enabled` 引數也可以用管理帳戶設定登錄。

```azurecli-interactive
az acr create --resource-group myResourceGroup --name myContainerRegistry$RANDOM --sku Basic
```

一旦建立登錄，Azure CLI 就會輸出類似下列的資料。 請記下 `name` 和 `loginServer`，在稍後的步驟中會用到這些資料。

```azurecli
{
  "adminUserEnabled": false,
  "creationDate": "2017-06-06T03:40:56.511597+00:00",
  "id": "/subscriptions/f2799821-a08a-434e-9128-454ec4348b10/resourcegroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry23489",
  "location": "eastus",
  "loginServer": "mycontainerregistry23489.azurecr.io",
  "name": "myContainerRegistry23489",
  "provisioningState": "Succeeded",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "storageAccount": {
    "name": "mycontainerregistr034017"
  },
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

使用 [az acr credential show](/cli/azure/acr/credential) 命令取得容器登錄認證。 以最後一個步驟中記下的名稱替代 `--name`。 記下一組密碼，稍後的步驟中會用到此密碼。

```azurecli-interactive
az acr credential show --name myContainerRegistry23489
```

如需 Azure Container Registry 的詳細資訊，請參閱[私人 Docker 容器登錄簡介](../../container-registry/container-registry-intro.md)。 

## <a name="manage-acr-authentication"></a>管理 ACR 驗證

從私人登錄推送和提取映像的傳統方式是先向登錄進行驗證。 若要這樣做，您可以在任何需要存取私人登錄的用戶端上使用 `docker login` 命令。 因為 DC/OS 叢集可以包含許多節點，而所有節點都必須向 ACR 驗證，所以有助於在每個節點上自動執行此程序。 

### <a name="create-shared-storage"></a>建立共用儲存體

此程序會使用已掛接在叢集中每個節點上的 Azure 檔案共用。 如果尚未設定共用儲存體，請參閱[在 DC/OS 叢集內設定檔案共用](container-service-dcos-fileshare.md)。

### <a name="configure-acr-authentication"></a>設定 ACR 驗證

首先，取得 DC/OS 主機的 FQDN，並將它儲存在變數中。

```azurecli-interactive
FQDN=$(az acs list --resource-group myResourceGroup --query "[0].masterProfile.fqdn" --output tsv)
```

建立 SSH 與以 DC/OS 為基礎的叢集主機 (或主要主機) 的連線。 如果建立叢集時已使用非預設值，請更新使用者名稱。

```azurecli-interactive
ssh azureuser@$FQDN
```

執行下列命令來登入 Azure Container Registry。 以容器登錄的名稱取代 `--username`，並以其中一個提供的密碼取代 `--password`。 以容器登錄的 loginServer 名稱取代範例中的最後一個引數 *mycontainerregistry.azurecr.io*。 

此命令會將驗證值儲存在本機的 `~/.docker` 路徑下。

```azurecli-interactive
docker -H tcp://localhost:2375 login --username=myContainerRegistry23489 --password=//=ls++q/m+w+pQDb/xCi0OhD=2c/hST mycontainerregistry.azurecr.io
```

建立壓縮檔案，其中包含容器登錄驗證值。

```azurecli-interactive
tar czf docker.tar.gz .docker
```

將此檔案複製到叢集共用儲存體。 這個步驟讓此檔案可用於 DC/OS 叢集的所有節點上。

```azurecli-interactive
cp docker.tar.gz /mnt/share/dcosshare
```

## <a name="upload-image-to-acr"></a>將映像上傳至 ACR

現在從開發電腦或任何已安裝 Docker 的其他系統，建立映像並將它上傳至 Azure Container Registry。

從 Ubuntu 映像建立容器。

```azurecli-interactive
docker run ubuntu --name base-image
```

現在將容器擷取至新映像。 映像名稱必須包含容器登錄的 `loginServer` 名稱，其格式為 `loginServer/imageName`。

```azurecli-interactive
docker -H tcp://localhost:2375 commit base-image mycontainerregistry30678.azurecr.io/dcos-demo
````

登入 Azure Container Registry。 以 loginServer 名稱取代名稱，以容器登錄的名稱取代 --username，並以其中一個提供的密碼取代 --password。

```azurecli-interactive
docker login --username=myContainerRegistry23489 --password=//=ls++q/m+w+pQDb/xCi0OhD=2c/hST mycontainerregistry2675.azurecr.io
```

最後，將映像上傳至 ACR 登錄。 這個範例會上傳名為 dcos-demo 的映像。

```azurecli-interactive
docker push mycontainerregistry30678.azurecr.io/dcos-demo
```

## <a name="run-an-image-from-acr"></a>從 ACR 執行映像

若要使用 ACR 登錄中的映像，請建立名為 *acrDemo.json* 的檔案並將下列文字複製到其中。 以容器登錄 loginServer 名稱和映像名稱 (例如 `loginServer/imageName`) 取代映像名稱。 請記下 `uris` 屬性。 這個屬性會保留容器登錄驗證檔案的位置，在此例中是掛接在 DC/OS 叢集中每個節點上的 Azure 檔案共用。

```json
{
  "id": "myapp",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "mycontainerregistry30678.azurecr.io/dcos-demo",
      "network": "BRIDGE",
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp",
          "name": "80",
          "labels": null
        }
      ],
      "forcePullImage":true
    }
  },
  "instances": 3,
  "cpus": 0.1,
  "mem": 65,
  "healthChecks": [{
      "protocol": "HTTP",
      "path": "/",
      "portIndex": 0,
      "timeoutSeconds": 10,
      "gracePeriodSeconds": 10,
      "intervalSeconds": 2,
      "maxConsecutiveFailures": 10
  }],
  "uris":  [
       "file:///mnt/share/dcosshare/docker.tar.gz"
   ]
}
```

使用 DC/OC CLI 部署應用程式。

```azurecli-interactive
dcos marathon app add acrDemo.json
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已將 DC/OS 設定為使用 Azure Container Registry，包括下列工作：

> [!div class="checklist"]
> * 部署 Azure Container Registry (如有需要)
> * 在 DC/OS 叢集上設定 ACR 驗證
> * 將映像上傳至 Azure Container Registry
> * 從 Azure Container Registry 執行容器映像
