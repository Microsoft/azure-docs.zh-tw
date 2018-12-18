---
title: 使用 Docker API 管理 Azure Swarm 叢集
description: 在 Azure Container Service 中將容器部署至 Docker Swarm 叢集
services: container-service
author: rgardler
manager: madhana
ms.service: container-service
ms.topic: article
ms.date: 09/13/2016
ms.author: rogardle
ms.custom: mvc
ms.openlocfilehash: 3f8d18bc053bc303ab124ba38c8621d4ee2e8cb8
ms.sourcegitcommit: 5d3e99478a5f26e92d1e7f3cec6b0ff5fbd7cedf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/06/2017
ms.locfileid: "26331540"
---
# <a name="container-management-with-docker-swarm"></a>使用 Docker Swarm 管理容器

Docker Swarm 提供跨一組匯集的 Docker 主機來部署容器化工作負載的環境。 Docker Swarm 使用原生 Docker API。 用來管理 Docker Swarm 上容器的工作流程與在單一容器主機時的工作流程幾乎相同。 本文件提供在 Docker Swarm 的 Azure 容器服務執行個體中部署容器化工作負載的簡單範例。 如需有關 Docker Swarm 的更深入文件，請參閱 [Docker.com 上的 Docker Swarm](https://docs.docker.com/swarm/)。

[!INCLUDE [container-service-swarm-mode-note](../../../includes/container-service-swarm-mode-note.md)]

本文件中之練習的先決條件︰

[在 Azure 容器服務中建立 Swarm 叢集](container-service-deployment.md)

[連接到 Azure 容器服務中的 Swarm 叢集](../container-service-connect.md)

## <a name="deploy-a-new-container"></a>部署新容器
若要在 Docker Swarm 中建立新容器，請使用 `docker run` 命令 (確定您已根據上述必要條件開啟主機的 SSH 通道)。 此範例會從 `yeasy/simple-web` 映像建立容器：

```bash
user@ubuntu:~$ docker run -d -p 80:80 yeasy/simple-web

4298d397b9ab6f37e2d1978ef3c8c1537c938e98a8bf096ff00def2eab04bf72
```

建立容器之後，使用 `docker ps` 傳回容器的相關資訊。 請注意，這裡會列出裝載容器的 Swarm 代理程式：

```bash
user@ubuntu:~$ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
4298d397b9ab        yeasy/simple-web    "/bin/sh -c 'python i"   31 seconds ago      Up 9 seconds        10.0.0.5:80->80/tcp   swarm-agent-34A73819-1/happy_allen
```  

您現在可以透過 Swarm 代理程式負載平衡器的公用 DNS 名稱來存取此容器中執行的應用程式。 您可在 Azure 入口網站中找到此資訊：  

![實際瀏覽結果](./media/container-service-docker-swarm/real-visit.jpg)  

根據預設，Load Balancer 開啟連接埠 80、8080 和 443。 如果您想要在另一個連接埠上連接，您必須在 Azure Load Balancer 上針對代理程式集區開啟該連接埠。

## <a name="deploy-multiple-containers"></a>部署多個容器
藉由多次執行 'docker run' 來啟動多個容器時，您可以使用 `docker ps` 命令來查看容器執行所在的主機。 在以下範例中，三個容器平均分散在三個 Swarm 代理程式中：  

```bash
user@ubuntu:~$ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
11be062ff602        yeasy/simple-web    "/bin/sh -c 'python i"   11 seconds ago      Up 10 seconds       10.0.0.6:83->80/tcp   swarm-agent-34A73819-2/clever_banach
1ff421554c50        yeasy/simple-web    "/bin/sh -c 'python i"   49 seconds ago      Up 48 seconds       10.0.0.4:82->80/tcp   swarm-agent-34A73819-0/stupefied_ride
4298d397b9ab        yeasy/simple-web    "/bin/sh -c 'python i"   2 minutes ago       Up 2 minutes        10.0.0.5:80->80/tcp   swarm-agent-34A73819-1/happy_allen
```  

## <a name="deploy-containers-by-using-docker-compose"></a>使用 Docker Compose 來部署容器
您可以使用 Docker Compose 來自動部署和設定多個容器。 若要這麼做，請確定已建立安全殼層 (SSH) 通道，並已設定 DOCKER_HOST 變數 (請參閱上述的必要元件)。

在您的本機系統上建立 docker-compose.yml 檔案。 若要這樣做，請使用此 [範例](https://raw.githubusercontent.com/rgardler/AzureDevTestDeploy/master/docker-compose.yml)。

```bash
web:
  image: adtd/web:0.1
  ports:
    - "80:80"
  links:
    - rest:rest-demo-azure.marathon.mesos
rest:
  image: adtd/rest:0.1
  ports:
    - "8080:8080"

```

執行 `docker-compose up -d` 以啟動容器部署：

```bash
user@ubuntu:~/compose$ docker-compose up -d
Pulling rest (adtd/rest:0.1)...
swarm-agent-3B7093B8-0: Pulling adtd/rest:0.1... : downloaded
swarm-agent-3B7093B8-2: Pulling adtd/rest:0.1... : downloaded
swarm-agent-3B7093B8-3: Pulling adtd/rest:0.1... : downloaded
Creating compose_rest_1
Pulling web (adtd/web:0.1)...
swarm-agent-3B7093B8-3: Pulling adtd/web:0.1... : downloaded
swarm-agent-3B7093B8-0: Pulling adtd/web:0.1... : downloaded
swarm-agent-3B7093B8-2: Pulling adtd/web:0.1... : downloaded
Creating compose_web_1
```

最後，將會傳回執行中容器的清單。 這份清單會反映使用 Docker Compose 所部署的容器︰

```bash
user@ubuntu:~/compose$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                     NAMES
caf185d221b7        adtd/web:0.1        "apache2-foreground"   2 minutes ago       Up About a minute   10.0.0.4:80->80/tcp       swarm-agent-3B7093B8-0/compose_web_1
040efc0ea937        adtd/rest:0.1       "catalina.sh run"      3 minutes ago       Up 2 minutes        10.0.0.4:8080->8080/tcp   swarm-agent-3B7093B8-0/compose_rest_1
```

當然，您可以使用 `docker-compose ps`，只檢查 `compose.yml` 檔案中定義的容器。

## <a name="next-steps"></a>後續步驟
[深入了解 Docker Swarm](https://docs.docker.com/swarm/)

