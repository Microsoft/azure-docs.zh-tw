---
title: 快速入門 - 適用於 Linux 的 Azure Docker CE 叢集
description: 快速了解如何在 Azure Container Service 中使用 Azure CLI 建立適用於 Linux 容器的 Docker CE 叢集。
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 07/16/2018
ms.author: iainfou
ms.custom: ''
ms.openlocfilehash: c8f9db2674976d2c1efa7686b8b224b48a19a534
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46964440"
---
# <a name="deploy-docker-ce-cluster"></a>部署 Docker CE 叢集

本快速入門中會使用 Azure CLI 來部署 Docker CE 叢集。 接著，在叢集上部署和執行多容器應用程式，其中包含 Web 前端和 Redis 執行個體。 完成後，即可透過網際網路來存取應用程式。

Azure Container Service 上的 Docker CE 處於預覽狀態，**不得用於生產工作負載**。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯群組。

下列範例會在 *westus2* 位置建立名為 *myResourceGroup* 的資源群組。

```azurecli-interactive
az group create --name myResourceGroup --location westus2
```

輸出：

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup",
  "location": "westus2",
  "managedBy": null,
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-docker-swarm-cluster"></a>建立 Docker Swarm 叢集

使用 [az acs create](/cli/azure/acs#az-acs-create) 命令，在 Azure Container Service 中建立 Docker CE 叢集。 如需可取得 Docker CE 之區域的相關資訊，請參閱 [Docker CE 的 ACS 區域](https://github.com/Azure/ACS/blob/master/announcements/2017-08-04_additional_regions.md) \(英文\)

下列範例會建立一個名為 mySwarmCluster 的叢集，其中包含一個 Linux 主要節點和三個 Linux 代理程式節點。

```azurecli-interactive
az acs create --name mySwarmCluster --orchestrator-type dockerce --resource-group myResourceGroup --generate-ssh-keys
```

在有限試用之類的某些情況下，Azure 訂用帳戶只擁有 Azure 資源的有限存取權。 如果部署因可用核心受限而失敗，請將 `--agent-count 1` 加入 [az acs create](/cli/azure/acs#az-acs-create) 命令來減少預設代理程式的數量。 

幾分鐘之後，此命令就會完成，並以 JSON 格式傳回叢集的相關資訊。

## <a name="connect-to-the-cluster"></a>連接到叢集

在本快速入門中，您需要 Docker Swarm 主機和 Docker 代理程式集區的 FQDN。 執行下列命令以傳回主機和代理程式 FQDN。


```bash
az acs list --resource-group myResourceGroup --query '[*].{Master:masterProfile.fqdn,Agent:agentPoolProfiles[0].fqdn}' -o table
```

輸出：

```bash
Master                                                               Agent
-------------------------------------------------------------------  --------------------------------------------------------------------
myswarmcluster-myresourcegroup-d5b9d4mgmt.ukwest.cloudapp.azure.com  myswarmcluster-myresourcegroup-d5b9d4agent.ukwest.cloudapp.azure.com
```

建立 Swarm 主機的 SSH 通道。 以 Swarm 主機的 FQDN 位址取代 `MasterFQDN`。

```bash
ssh -p 2200 -fNL localhost:2374:/var/run/docker.sock azureuser@MasterFQDN
```

設定 `DOCKER_HOST` 環境變數。 這可讓您對 Docker Swarm 執行 Docker 命令，而不需指定主機名稱。

```bash
export DOCKER_HOST=localhost:2374
```

您現在已經準備好在 Docker Swarm 上執行 Docker 服務。


## <a name="run-the-application"></a>執行應用程式

建立名為 `azure-vote.yaml` 的檔案，並將下列內容複製到其中。


```yaml
version: '3'
services:
  azure-vote-back:
    image: redis
    ports:
        - "6379:6379"

  azure-vote-front:
    image: microsoft/azure-vote-front:v1
    environment:
      REDIS: azure-vote-back
    ports:
        - "80:80"
```

執行 [docker stack deploy](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 命令來建立 Azure 投票服務。

```bash
docker stack deploy azure-vote --compose-file azure-vote.yaml
```

輸出：

```bash
Creating network azure-vote_default
Creating service azure-vote_azure-vote-back
Creating service azure-vote_azure-vote-front
```

使用 [docker stack ps](https://docs.docker.com/engine/reference/commandline/stack_ps/) 命令來傳回應用程式的部署狀態。

```bash
docker stack ps azure-vote
```

一旦每項服務的 `CURRENT STATE` 成為 `Running`，應用程式便已準備就緒。

```bash
ID                  NAME                            IMAGE                                 NODE                               DESIRED STATE       CURRENT STATE                ERROR               PORTS
tnklkv3ogu3i        azure-vote_azure-vote-front.1   microsoft/azure-vote-front:v1   swarmm-agentpool0-66066781000004   Running             Running 5 seconds ago                            
lg99i4hy68r9        azure-vote_azure-vote-back.1    redis:latest                          swarmm-agentpool0-66066781000002   Running             Running about a minute ago
```

## <a name="test-the-application"></a>測試應用程式

瀏覽至 Swarm 代理程式集區的 FQDN，以測試 Azure 投票應用程式。

![瀏覽至 Azure 投票的影像](media/container-service-docker-swarm-mode-walkthrough/azure-vote.png)

## <a name="delete-cluster"></a>刪除叢集
若不再需要叢集，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令來移除資源群組、容器服務和所有相關資源。

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="get-the-code"></a>取得程式碼

在本快速入門中，預先建立的容器映像已用來建立 Docker 服務。 在 GitHub 上可取得相關的應用程式程式碼、Dockerfile 和 Compose 檔案。

[https://github.com/Azure-Samples/azure-voting-app-redis](https://github.com/Azure-Samples/azure-voting-app-redis.git)

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署 Docker Swarm 叢集，並將多容器應用程式部署到此叢集。

若要了解如何整合 Docker Swarm 與 Azure DevOps，請繼續進行搭配 Docker Swarm 和 Azure DevOps 的 CI/CD。

> [!div class="nextstepaction"]
> [搭配 Docker Swarm 和 Azure DevOps 的 CI/CD](./container-service-docker-swarm-setup-ci-cd.md)
