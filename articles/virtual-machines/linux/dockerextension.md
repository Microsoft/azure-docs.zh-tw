---
title: 使用 Azure Docker VM 延伸模組 | Microsoft Docs
description: 了解如何使用 Resource Manager 範本和 Azure CLI，在 Azure 中使用 Docker VM 延伸模組快速且安全地部署 Docker 環境
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: 936d67d7-6921-4275-bf11-1e0115e66b7f
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/18/2017
ms.author: cynthn
ms.openlocfilehash: 59dbbb8374455088d759a5e837b8d3bc22145d3e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981292"
---
# <a name="create-a-docker-environment-in-azure-using-the-docker-vm-extension"></a>使用 Docker VM 延伸模組在 Azure 中建立 Docker 環境

Docker 是常用的容器管理和映像處理平台，它能讓您在 Linux 上快速地操作容器。 在 Azure 中，您可以根據您的需求使用幾個方式來部署 Docker。 此文章著重於搭配 Azure CLI 使用 Docker VM 延伸模組與 Azure Resource Manager 範本。 

> [!WARNING]
> 適用於 Linux 的 Azure Docker VM 延伸模組已被取代，且即將在 2018 年 11 月淘汰。
> 該延伸模組只會安裝 Docker，因此像是 cloud-init 或自訂指令碼延伸模組等替代方案，都是安裝選定 Docker 版本的更佳方法。 如需 cloud-init 使用方法的詳細資訊，請參閱[使用 cloud-init 來自訂 Linux VM](tutorial-automate-vm-deployment.md)。

## <a name="azure-docker-vm-extension-overview"></a>Azure Docker VM 延伸模組概觀
Azure Docker VM 延伸模組會在您的 Linux 虛擬機器 (VM) 中安裝並設定 Docker 精靈、Docker 用戶端和 Docker Compose。 相較於只使用 Docker 電腦或自行建立 Docker 主機，您使用 Azure Docker VM 延伸模組會有更多控制權和功能。 這些額外功能，例如 [Docker Compose](https://docs.docker.com/compose/overview/)，讓 Azure Docker VM 延伸模組適用於更穩固的開發人員或生產環境。

如需不同部署方法的詳細資訊，包括使用 Docker 電腦和 Azure Container Service，請參閱下列文章︰

* 如需快速應用程式原型，您可以使用 [Docker 電腦](docker-machine.md)建立單一 Docker 主機。
* 如需建置提供其他排程和管理工具的生產就緒、可調整環境，您可以在 Azure Container Service 上部署 [Kubernetes](../../container-service/kubernetes/index.yml) 或 [Docker Swarm](../../container-service/dcos-swarm/index.yml) 叢集。


## <a name="deploy-a-template-with-the-azure-docker-vm-extension"></a>使用 Azure Docker VM 延伸模組部署範本
使用現有的快速入門範本建立使用 Azure Docker VM 延伸模組的 Ubuntu VM，以安裝及設定 Docker 主機。 您可以在這裡檢視範本︰ [使用 Docker 簡易部署 Ubuntu VM](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-simple-on-ubuntu)。 您需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。

首先，使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli
az group create --name myResourceGroup --location eastus
```

接下來，使用 [az group deployment create](/cli/azure/group/deployment#az_group_deployment_create) 來部署 VM，其中包含來自 [GitHub 上此 Azure Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-simple-on-ubuntu)的 Azure Docker VM 延伸模組。 出現提示時，針對 newStorageAccountName、adminUsername、adminPassword 和 dnsNameForPublicIP 提供您自己唯一的值：

```azurecli
az group deployment create --resource-group myResourceGroup \
    --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/docker-simple-on-ubuntu/azuredeploy.json
```

需要幾分鐘的時間才能完成部署。


## <a name="deploy-your-first-nginx-container"></a>部署您的第一個 NGINX 容器
若要檢視 VM 的詳細資料，包括 DNS 名稱，請使用 [az vm show](/cli/azure/vm#az_vm_show)：

```azurecli
az vm show \
    --resource-group myResourceGroup \
    --name myDockerVM \
    --show-details \
    --query [fqdns] \
    --output tsv
```

以 SSH 連線到新的 Docker 主機。 提供先前步驟中您自己的使用者名稱和 DNS 名稱：

```bash
ssh azureuser@mypublicdns.eastus.cloudapp.azure.com
```

一旦登入 Docker 主機後，請執行 NGINX 容器︰

```bash
sudo docker run -d -p 80:80 nginx
```

下載 NGINX 影像和啟動容器時，輸出類似下列的範例︰

```bash
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
efd26ecc9548: Pull complete
a3ed95caeb02: Pull complete
a48df1751a97: Pull complete
8ddc2d7beb91: Pull complete
Digest: sha256:2ca2638e55319b7bc0c7d028209ea69b1368e95b01383e66dfe7e4f43780926d
Status: Downloaded newer image for nginx:latest
b6ed109fb743a762ff21a4606dd38d3e5d35aff43fa7f12e8d4ed1d920b0cd74
```

檢查您的 Docker 主機上執行的容器狀態，如下所示︰

```bash
sudo docker ps
```

輸出類似下列的範例，會顯示 NGINX 容器正在執行，且正在轉送 TCP 連接埠 80 和 443︰

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                         NAMES
b6ed109fb743        nginx               "nginx -g 'daemon off"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, 443/tcp   adoring_payne
```

若要查看容器實際運作的情況，請開啟網頁瀏覽器，然後輸入 Docker 主機的 DNS 名稱︰

![執行 ngnix 容器](./media/dockerextension/nginxrunning.png)

## <a name="azure-docker-vm-extension-template-reference"></a>Azure Docker VM 延伸模組範本參考
前一個範例使用現有的快速入門範本。 您也可以使用您自己的 Resource Manager 範本來部署 Azure Docker VM 延伸模組。 若要這樣做，請將下列項目新增至 Resource Manager 範本，適當地定義 VM 的 `vmName`︰

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(variables('vmName'), '/DockerExtension'))]",
  "apiVersion": "2015-05-01-preview",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
  ],
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "DockerExtension",
    "typeHandlerVersion": "1.*",
    "autoUpgradeMinorVersion": true,
    "settings": {},
    "protectedSettings": {}
  }
}
```

如需更詳細的 Resource Manager 範本使用逐步解說，請參閱 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)。

## <a name="next-steps"></a>後續步驟
您可能會想要使用 [Docker Compose](https://docs.docker.com/compose/overview/) 來[設定 Docker 精靈 TCP 連接埠](https://docs.docker.com/engine/reference/commandline/dockerd/#/bind-docker-to-another-hostport-or-a-unix-socket)、了解[Docker 安全性](https://docs.docker.com/engine/security/security/)或部署容器。 如需有關 Azure Docker VM 延伸模組本身的詳細資訊，請參閱 [GitHub 專案](https://github.com/Azure/azure-docker-extension/)。

閱讀有關 Azure 中其他 Docker 部署選項的詳細資訊︰

* [使用 Docker 電腦搭配 Azure 驅動程式](docker-machine.md)  
* [在 Azure 虛擬機器上開始使用 Docker 和 Compose 定義並執行多容器應用程式](docker-compose-quickstart.md)。
* [部署 Azure 容器服務叢集](../../container-service/dcos-swarm/container-service-deployment.md)

