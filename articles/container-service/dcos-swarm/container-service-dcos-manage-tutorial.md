---
title: Azure Container Service 教學課程 - 管理 DC/OS
description: Azure Container Service 教學課程 - 管理 DC/OS
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 02/26/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 8f4b1c1c66243c06d46ef3fddec8ea495163fa07
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46999608"
---
# <a name="azure-container-service-tutorial---manage-dcos"></a>Azure Container Service 教學課程 - 管理 DC/OS

DC/OS 所提供的分散式平台可執行現代及容器化的應用程式。 透過 Azure Container Service 可簡單又快速地佈建生產環境就緒 DC/OS 叢集。 本快速入門將詳細說明部署 DC/OS 叢集和執行基本工作負載所需的基本步驟。

> [!div class="checklist"]
> * 建立 ACS DC/OS 叢集
> * 連接到叢集
> * 安裝 DC/OS CLI
> * 將應用程式部署到叢集
> * 調整叢集上的應用程式
> * 調整 DC/OS 叢集節點
> * DC/OS 的基本管理
> * 刪除 DC/OS 叢集

本教學課程需要 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="create-dcos-cluster"></a>建立 DC/OS 叢集

首先，使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 westeurope 位置建立名為 myResourceGroup 的資源群組。

```azurecli
az group create --name myResourceGroup --location westeurope
```

接下來，使用 [az acs create](/cli/azure/acs#az-acs-create) 命令來建立 DC/OS 叢集。

下列範例會建立名為 myDCOSCluster 的 DC/OS 叢集，並建立 SSH 金鑰 (如果它們尚未存在)。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。  

```azurecli
az acs create \
  --orchestrator-type dcos \
  --resource-group myResourceGroup \
  --name myDCOSCluster \
  --generate-ssh-keys
```

幾分鐘之後，此命令就會完成，且會傳回部署的相關資訊。

## <a name="connect-to-dcos-cluster"></a>連線到 DC/OS 叢集

一旦建立 DC/OS 叢集之後，可透過 SSH 通道加以存取。 執行下列命令，可傳回 DC/OS 主機的公用 IP 位址。 此 IP 位址儲存在變數中，且用於下一個步驟。

```azurecli
ip=$(az network public-ip list --resource-group myResourceGroup --query "[?contains(name,'dcos-master')].[ipAddress]" -o tsv)
```

若要建立 SSH 通道，請執行下列命令，並遵循螢幕上的指示。 如果連接埠 80 已在使用中，命令就會失敗。 將通道連接埠更新為非使用中的連接埠，例如 `85:localhost:80`。 

```azurecli
sudo ssh -i ~/.ssh/id_rsa -fNL 80:localhost:80 -p 2200 azureuser@$ip
```

## <a name="install-dcos-cli"></a>安裝 DC/OS CLI

使用 [az acs dcos install-cli](/cli/azure/acs/dcos#az-acs-dcos-install-cli) 命令來安裝 DC/OS CLI。 如果您是使用 Azure CloudShell，就已安裝 DC/OS CLI。 如果您是在 Mac OS 或 Lunix 上執行 Azure CLI，可能需要搭配 sudo 來執行命令。

```azurecli
az acs dcos install-cli
```

在 CLI 可與叢集搭配使用之前，它必須先設定為使用 SSH 通道。 若要這樣做，請執行下列命令，並視需要調整連接埠。

```azurecli
dcos config set core.dcos_url http://localhost
```

## <a name="run-an-application"></a>執行應用程式

ACS DC/OS 叢集的預設排程機制為 Marathon。 Marathon 可用來啟動應用程式及管理 DC/OS 叢集上的應用程式狀態。 若要透過 Marathon 排程應用程式，請建立名為 **marathon-app.json** 的檔案，並將下列內容複製到其中。 

```json
{
  "id": "demo-app-private",
  "cmd": null,
  "cpus": 1,
  "mem": 32,
  "disk": 0,
  "instances": 1,
  "container": {
    "docker": {
      "image": "nginx",
      "network": "BRIDGE",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp",
          "name": "80",
          "labels": null
        }
      ]
    },
    "type": "DOCKER"
  }
}
```

執行下列命令可排程要在 DC/OS 叢集上執行的應用程式。

```azurecli
dcos marathon app add marathon-app.json
```

若要查看應用程式的部署狀態，請執行下列命令。

```azurecli
dcos marathon app list
```

當 **TASKS** 資料行值從 0/1 切換為 1/1 時，應用程式部署就已完成。

```azurecli
ID     MEM  CPUS  TASKS  HEALTH  DEPLOYMENT  WAITING  CONTAINER  CMD   
/test   32   1     0/1    ---       ---      False      DOCKER   None
```

## <a name="scale-marathon-application"></a>調整 Marathon 應用程式

在上述範例中，已建立單一執行個體應用程式。 若要更新此部署，以便讓應用程式的三個執行個體可供使用，請開啟 **marathon-app.json** 檔案，並將執行個體屬性更新為 3。

```json
{
  "id": "demo-app-private",
  "cmd": null,
  "cpus": 1,
  "mem": 32,
  "disk": 0,
  "instances": 3,
  "container": {
    "docker": {
      "image": "nginx",
      "network": "BRIDGE",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp",
          "name": "80",
          "labels": null
        }
      ]
    },
    "type": "DOCKER"
  }
}
```

使用 `dcos marathon app update` 命令更新應用程式。

```azurecli
dcos marathon app update demo-app-private < marathon-app.json
```

若要查看應用程式的部署狀態，請執行下列命令。

```azurecli
dcos marathon app list
```

當 **TASKS** 資料行值從 1/3 切換為 3/1 時，應用程式部署就已完成。

```azurecli
ID     MEM  CPUS  TASKS  HEALTH  DEPLOYMENT  WAITING  CONTAINER  CMD   
/test   32   1     1/3    ---       ---      False      DOCKER   None
```

## <a name="run-internet-accessible-app"></a>執行網際網路可存取應用程式

ACS DC/OS 叢集包含兩個節點集合，一個是可在網際網路上存取的公用節點集合，以及一個不可在網際網路上存取的私人節點集合。 預設設定是私人節點，會在最後一個範例中使用。

若要讓應用程式可在網際網路上存取，請將其部署至公用節點集合。 若要這樣做，請為 `acceptedResourceRoles` 物件提供 `slave_public` 的值。

建立名為 **nginx-public.json** 的檔案，並將下列內容複製到其中。

```json
{
  "id": "demo-app",
  "cmd": null,
  "cpus": 1,
  "mem": 32,
  "disk": 0,
  "instances": 1,
  "container": {
    "docker": {
      "image": "nginx",
      "network": "BRIDGE",
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp",
          "name": "80",
          "labels": null
        }
      ]
    },
    "type": "DOCKER"
  },
  "acceptedResourceRoles": [
    "slave_public"
  ]
}
```

執行下列命令可排程要在 DC/OS 叢集上執行的應用程式。

```azurecli 
dcos marathon app add nginx-public.json
```

取得 DC/OS 公用叢集代理程式的公用 IP 位址。

```azurecli 
az network public-ip list --resource-group myResourceGroup --query "[?contains(name,'dcos-agent')].[ipAddress]" -o tsv
```

瀏覽到這個地址會傳回預設的 NGINX 站台。

![NGINX](./media/container-service-dcos-manage-tutorial/nginx.png)

## <a name="scale-dcos-cluster"></a>調整 DC/OS 叢集

在上一個範例中，應用程式會調整為多個執行個體。 也可以調整 DC/OS 基礎結構以提供較多或較少的計算容量。 方法是使用 [az acs scale](/cli/azure/acs#az-acs-scale) 命令。 

若要查看目前 DC/OS 代理程式的計數，請使用 [az acs show](/cli/azure/acs#az-acs-show) 命令。

```azurecli
az acs show --resource-group myResourceGroup --name myDCOSCluster --query "agentPoolProfiles[0].count"
```

若要將計數增加至 5，請使用 [az acs scale](/cli/azure/acs#az-acs-scale) 命令。 

```azurecli
az acs scale --resource-group myResourceGroup --name myDCOSCluster --new-agent-count 5
```

## <a name="delete-dcos-cluster"></a>刪除 DC/OS 叢集

若不再需要，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令將資源群組、DC/OS 叢集和所有相關資源移除。

```azurecli 
az group delete --name myResourceGroup --no-wait
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解有關基本的 DC/OS 管理工作，如下所示。 

> [!div class="checklist"]
> * 建立 ACS DC/OS 叢集
> * 連接到叢集
> * 安裝 DC/OS CLI
> * 將應用程式部署到叢集
> * 調整叢集上的應用程式
> * 調整 DC/OS 叢集節點
> * 刪除 DC/OS 叢集

前往下一個教學課程，可了解在 Azure 上 DC/OS 中的負載平衡應用程式。 

> [!div class="nextstepaction"]
> [負載平衡應用程式](container-service-load-balancing.md)
