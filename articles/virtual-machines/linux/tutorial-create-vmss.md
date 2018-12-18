---
title: 教學課程 - 在 Azure 中建立 Linux 的虛擬機器擴展集 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure CLI 在 Linux VM 上使用虛擬機器擴展集，建立及部署高可用性應用程式
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 06/01/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: e3354abb400530bc5aa18288408b1052cd3575c4
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992230"
---
# <a name="tutorial-create-a-virtual-machine-scale-set-and-deploy-a-highly-available-app-on-linux-with-the-azure-cli"></a>教學課程：使用 Azure CLI 在 Linux 上建立虛擬機器擴展集及部署高可用性應用程式

虛擬機器擴展集可讓您部署和管理一組相同、自動調整的虛擬機器。 您可以手動調整擴展集中的 VM 數目，或定義規則以根據如 CPU、記憶體需求或網路流量的資源使用量來自動調整。 在本教學課程中，您將會在 Azure 部署虛擬機器擴展集。 您會了解如何：

> [!div class="checklist"]
> * 使用 Cloud-init 來建立要調整的應用程式
> * 建立虛擬機器擴展集
> * 增加或減少擴展集內的執行個體數目
> * 建立自動調整規則
> * 檢視擴展集執行個體的連線資訊
> * 在擴展集內使用資料磁碟

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="scale-set-overview"></a>擴展集概觀
虛擬機器擴展集可讓您部署和管理一組相同、自動調整的虛擬機器。 擴展集中的 VM 會分散於一或多個放置群組中的邏輯容錯網域和更新網域。 這些是類似設定 VM 的群組，類似於[可用性設定組](tutorial-availability-sets.md)。

VM 會視需要建立於擴展集中。 您將定義自動調整規則，以控制如何及何時新增或移除擴展集中的 VM。 您可以根據計量 (例如 CPU 負載、記憶體使用量或網路流量) 觸發這些規則。

當您使用 Azure 平台映像時，擴展集可支援多達 1,000 部 VM。 對於包含重要安裝作業或 VM 自訂要求的工作負載，您可能想要[建立自訂的 VM 映像](tutorial-custom-images.md)。 使用自訂映像時，您可以在擴展集中建立多達 300 部 VM。


## <a name="create-an-app-to-scale"></a>建立要調整的應用程式
為了提供生產環境使用，您可以[建立自訂 VM 映像](tutorial-custom-images.md)，其中包含已安裝和設定的應用程式。 在本教學課程中，我們會在首次開機時自訂 VM，以便快速查看作用中擴展集。

在先前的教學課程中，您已了解使用 cloud-init [如何在首次開機時自訂 Linux 虛擬機器](tutorial-automate-vm-deployment.md)。 您可以使用相同的 cloud-init 組態檔來安裝 NGINX 和執行簡單的 'Hello World' Node.js 應用程式。

您目前的殼層中，建立名為 cloud-init.txt 的檔案，並貼上下列組態。 例如，在 Cloud Shell 中建立不在本機電腦上的檔案。 輸入 `sensible-editor cloud-init.txt` 可建立檔案，並查看可用的編輯器清單。 請確定已正確複製整個 cloud-init 檔案，特別是第一行：

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```


## <a name="create-a-scale-set"></a>建立擴展集
請先使用 [az group create](/cli/azure/group#az-group-create) 建立資源群組，才可以建立擴展集。 下列範例會在 eastus 位置建立名為 myResourceGroupScaleSet 的資源群組：

```azurecli-interactive
az group create --name myResourceGroupScaleSet --location eastus
```

現在使用 [az vmss create](/cli/azure/vmss#az-vmss-create) 建立虛擬機器擴展集。 下列範例會建立名為 myScaleSet 的擴展集，使用 cloud-int 檔案來自訂 VM，以及產生 SSH 金鑰 (如果不存在)︰

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroupScaleSet \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.txt \
  --admin-username azureuser \
  --generate-ssh-keys
```

建立及設定所有擴展集資源和 VM 需要幾分鐘的時間。 在 Azure CLI 將您返回提示字元之後，背景工作會繼續執行。 可能需要再等候幾分鐘，才能存取應用程式。


## <a name="allow-web-traffic"></a>允許 Web 流量
負載平衡器會自動建立，作為虛擬機器擴展集的一部分。 負載平衡器會使用負載平衡器規則，將流量分散於一組定義的 VM。 您可以在下一個教學課程[如何平衡 Azure 中虛擬機器的負載](tutorial-load-balancer.md)中，深入了解負載平衡器的概念和設定。

若要允許流量觸達 Web 應用程式，請使用 [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create) 建立規則。 下列範例會建立名為 myLoadBalancerRuleWeb 的規則：

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroupScaleSet \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

## <a name="test-your-app"></a>測試應用程式
若要查看 Web 上的 Node.js 應用程式，請使用 [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) 取得負載平衡器的公用 IP 位址。 下列範例會取得建立作為擴展集一部分的 myScaleSetLBPublicIP IP 位址︰

```azurecli-interactive
az network public-ip show \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSetLBPublicIP \
    --query [ipAddress] \
    --output tsv
```

在 Web 瀏覽器中輸入公用 IP 位址。 應用程式隨即顯示，包括負載平衡器分散流量之 VM 的主機名稱︰

![執行 Node.js 應用程式](./media/tutorial-create-vmss/running-nodejs-app.png)

若要查看作用中的擴展集，請強制重新整理您的 Web 瀏覽器，以查看負載平衡器如何將流量分散於執行應用程式的所有 VM。


## <a name="management-tasks"></a>管理工作
在擴展集生命週期中，您可能需要執行一或多個管理工作。 此外，您可以建立指令碼來自動化各種生命週期工作。 Azure CLI 提供快速的方式來執行這些工作。 以下是一些常見工作。

### <a name="view-vms-in-a-scale-set"></a>檢視擴展集中的 VM
若要檢視在擴展集中執行的 VM 清單，請使用 [az vmss list-instances](/cli/azure/vmss#az-vmss-list-instances)，如下所示︰

```azurecli-interactive
az vmss list-instances \
  --resource-group myResourceGroupScaleSet \
  --name myScaleSet \
  --output table
```

輸出類似於下列範例：

```bash
  InstanceId  LatestModelApplied    Location    Name          ProvisioningState    ResourceGroup            VmId
------------  --------------------  ----------  ------------  -------------------  -----------------------  ------------------------------------
           1  True                  eastus      myScaleSet_1  Succeeded            MYRESOURCEGROUPSCALESET  c72ddc34-6c41-4a53-b89e-dd24f27b30ab
           3  True                  eastus      myScaleSet_3  Succeeded            MYRESOURCEGROUPSCALESET  44266022-65c3-49c5-92dd-88ffa64f95da
```


### <a name="manually-increase-or-decrease-vm-instances"></a>手動增加或減少 VM 執行個體
若要查看擴展集中目前擁有的執行個體數目，請使用 [az vmss show](/cli/azure/vmss#az-vmss-show)並查詢 sku.capacity：

```azurecli-interactive
az vmss show \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --query [sku.capacity] \
    --output table
```

然後，您可以使用 [az vmss scale](/cli/azure/vmss#az-vmss-scale)，手動增加或減少擴展集中的虛擬機器數目。 下列範例會將擴展集中的 VM 數目設定為 *3*：

```azurecli-interactive
az vmss scale \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --new-capacity 3
```

### <a name="get-connection-info"></a>取得連線資訊
若要取得擴展集中 VM 的相關連線資訊，請使用 [az vmss list-instance-connection-info](/cli/azure/vmss#az-vmss-list-instance-connection-info)。 此命令會輸出每部 VM 的公用 IP 位址和連接埠，可讓您與 SSH 連線︰

```azurecli-interactive
az vmss list-instance-connection-info \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet
```


## <a name="use-data-disks-with-scale-sets"></a>使用資料磁碟搭配擴展集
您可以建立及使用資料磁碟搭配擴展集。 在先前的教學課程中，您已了解如何[管理 Azure 磁碟](tutorial-manage-disks.md)，其中概述了在資料磁碟上 (不是在 OS 磁碟上) 建置應用程式的最佳做法和效能改進。

### <a name="create-scale-set-with-data-disks"></a>使用資料磁碟建立擴展集
若要建立擴展集並連結資料磁碟，可將 `--data-disk-sizes-gb` 參數新增到 [az vmss create](/cli/azure/vmss#az-vmss-create)命令。 下列範例會建立有 50GB 資料磁碟且連結到每個執行個體的擴展集︰

```azurecli-interactive
az vmss create \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSetDisks \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --custom-data cloud-init.txt \
    --admin-username azureuser \
    --generate-ssh-keys \
    --data-disk-sizes-gb 50
```

當執行個體從擴展集移除時，所有連結的資料磁碟也會一併移除。

### <a name="add-data-disks"></a>新增資料磁碟
若要將資料磁碟新增到擴展集中的執行個體，請使用 [az vmss disk attach](/cli/azure/vmss/disk#az-vmss-disk-attach)。 下列範例會將 50GB 的磁碟新增到每個執行個體：

```azurecli-interactive
az vmss disk attach \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --size-gb 50 \
    --lun 2
```

### <a name="detach-data-disks"></a>卸離資料磁碟
若要將資料磁碟從擴展集中的執行個體上移除，請使用 [az vmss disk detach](/cli/azure/vmss/disk#az-vmss-disk-detach)。 下列範例會從每個執行個體移除在 LUN 2 的資料磁碟︰

```azurecli-interactive
az vmss disk detach \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --lun 2
```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已建立虛擬機器擴展集。 您已了解如何︰

> [!div class="checklist"]
> * 使用 Cloud-init 來建立要調整的應用程式
> * 建立虛擬機器擴展集
> * 增加或減少擴展集內的執行個體數目
> * 建立自動調整規則
> * 檢視擴展集執行個體的連線資訊
> * 在擴展集內使用資料磁碟

請前進到下一個教學課程，以深入了解虛擬機器的負載平衡概念。

> [!div class="nextstepaction"]
> [平衡虛擬機器的負載](tutorial-load-balancer.md)
