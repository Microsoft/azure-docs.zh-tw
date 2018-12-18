---
title: Azure Container Service 快速入門 - 部署 DC/OS 叢集
description: Azure Container Service 快速入門 - 部署 DC/OS 叢集
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: quickstart
ms.date: 02/26/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 4cd7f2284f3449fe8abdf1a6f8e85b9b0b712d7a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998554"
---
# <a name="deploy-a-dcos-cluster"></a>部署 DC/OS 叢集

DC/OS 所提供的分散式平台可執行現代及容器化的應用程式。 透過 Azure Container Service 可簡單又快速地佈建生產環境就緒 DC/OS 叢集。 本快速入門將詳細說明部署 DC/OS 叢集和執行基本工作負載所需的基本步驟。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

本教學課程需要 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="log-in-to-azure"></a>登入 Azure 

使用 [az login](/cli/azure/reference-index#az-login) 命令登入 Azure 訂用帳戶並遵循畫面上的指示。

```azurecli
az login
```

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli
az group create --name myResourceGroup --location eastus
```

## <a name="create-dcos-cluster"></a>建立 DC/OS 叢集

使用 [az acs create](/cli/azure/acs#az-acs-create) 命令來建立 DC/OS 叢集。

下列範例會建立名為 myDCOSCluster 的 DC/OS 叢集，並建立 SSH 金鑰 (如果它們尚未存在)。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。  

```azurecli
az acs create --orchestrator-type dcos --resource-group myResourceGroup --name myDCOSCluster --generate-ssh-keys
```

在有限試用之類的某些情況下，Azure 訂用帳戶只擁有 Azure 資源的有限存取權。 如果部署因可用核心受限而失敗，請將 `--agent-count 1` 加入 [az acs create](/cli/azure/acs#az-acs-create) 命令來減少預設代理程式的數量。 

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

您可瀏覽至 `http://localhost` 來測試 SSH 通道。 如果已使用 80 以外的連接埠，可調整位置來進行比對。 

如果已成功建立 SSH 通道，就會傳回 DC/OS 入口網站。

![DCOS UI](./media/container-service-dcos-quickstart/dcos-ui.png)

## <a name="install-dcos-cli"></a>安裝 DC/OS CLI

DC/OS 命令列介面可用來從命令列管理 DC/OS 叢集。 使用 [az acs dcos install-cli](/cli/azure/acs/dcos#az-acs-dcos-install-cli) 命令來安裝 DC/OS CLI。 如果您是使用 Azure CloudShell，就已安裝 DC/OS CLI。 

如果您是在 Mac OS 或 Lunix 上執行 Azure CLI，可能需要搭配 sudo 來執行命令。

```azurecli
az acs dcos install-cli
```

在 CLI 可與叢集搭配使用之前，它必須先設定為使用 SSH 通道。 若要這樣做，請執行下列命令，並視需要調整連接埠。

```azurecli
dcos config set core.dcos_url http://localhost
```

## <a name="run-an-application"></a>執行應用程式

ACS DC/OS 叢集的預設排程機制為 Marathon。 Marathon 可用來啟動應用程式及管理 DC/OS 叢集上的應用程式狀態。 若要透過 Marathon 排程應用程式，請建立名為 *marathon-app.json* 的檔案，並將下列內容複製到其中。 

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
dcos marathon app add marathon-app.json
```

若要查看應用程式的部署狀態，請執行下列命令。

```azurecli
dcos marathon app list
```

當 **WAITING** 資料行值從 True 切換為 False 時，應用程式部署就已完成。

```azurecli
ID     MEM  CPUS  TASKS  HEALTH  DEPLOYMENT  WAITING  CONTAINER  CMD   
/test   32   1     1/1    ---       ---      False      DOCKER   None
```

取得 DC/OS 叢集代理程式的公用 IP 位址。

```azurecli
az network public-ip list --resource-group myResourceGroup --query "[?contains(name,'dcos-agent')].[ipAddress]" -o tsv
```

瀏覽到這個地址會傳回預設的 NGINX 站台。

![NGINX](./media/container-service-dcos-quickstart/nginx.png)

## <a name="delete-dcos-cluster"></a>刪除 DC/OS 叢集

若不再需要，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令將資源群組、DC/OS 叢集和所有相關資源移除。

```azurecli
az group delete --name myResourceGroup --no-wait
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署 DC/OS 叢集，並已在叢集上執行簡單的 Docker 容器。 若要深入了解 Azure Container Service，請繼續進行 ACS 教學課程。

> [!div class="nextstepaction"]
> [管理 ACS DC/OS 叢集](container-service-dcos-manage-tutorial.md)