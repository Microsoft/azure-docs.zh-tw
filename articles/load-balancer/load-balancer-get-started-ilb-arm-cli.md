---
title: 建立內部基本負載平衡器 - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure CLI 來建立內部負載平衡器
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/27/2018
ms.author: kumud
ms.openlocfilehash: 96c52beadb9424bda31726b2fa6da8b4c5b94fbf
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48830135"
---
# <a name="create-an-internal-load-balancer-to-load-balance-vms-using-azure-cli"></a>使用 Azure CLI 來建立內部負載平衡器以平衡 VM 的負載

本文說明如何建立內部負載平衡器以平衡 VM 的負載。 若要測試負載平衡器，您必須部署兩個執行 Ubuntu 伺服器的虛擬機器 (VM)，以平衡 Web 應用程式的負載。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

如果您選擇在本機安裝和使用 CLI，在執行本教學課程時，您必須執行 Azure CLI 2.0.28 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

下列範例會在 *eastus* 位置中建立名為 *myResourceGroupILB* 的資源群組：

```azurecli-interactive
  az group create \
    --name myResourceGroupILB \
    --location eastus
```
## <a name="create-a-virtual-network"></a>建立虛擬網路

使用 [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#create)，在 *myResourceGroup* 中建立名為 *myVNet*、具有子網路 *mySubnet* 的虛擬網路。

```azurecli-interactive
  az network vnet create \
    --name myVnet \
    --resource-group myResourceGroupILB \
    --location eastus \
    --subnet-name mySubnet
```
## <a name="create-basic-load-balancer"></a>建立基本負載平衡器

本節將詳細說明如何建立及設定下列負載平衡器元件：
  - 前端 IP 組態，可接收負載平衡器上的連入網路流量。
  - 後端 IP 集區，前端集區在其中傳送負載平衡網路流量。
  - 健康狀態探查，可判斷後端 VM 執行個體的健康狀態。
  - 負載平衡器規則，可定義如何將流量分散至 VM。

### <a name="create-the-load-balancer"></a>建立負載平衡器

使用 [az network lb create](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest#create) 建立名為 **myLoadBalancer** 的公用基本負載平衡器，包含名為 **myFrontEnd** 的前端 IP 組態、名為 **myBackEndPool** 的後端集區，與私人 IP 位址 **10.0.0.7 相關聯。

```azurecli-interactive
  az network lb create \
    --resource-group myResourceGroupILB \
    --name myLoadBalancer \
    --frontend-ip-name myFrontEnd \
    --private-ip-address 10.0.0.7 \
    --backend-pool-name myBackEndPool \
    --vnet-name myVnet \
    --subnet mySubnet      
  ```
### <a name="create-the-health-probe"></a>建立健康情況探查

健全狀況探查會檢查所有虛擬機器執行個體，確認它們可以接收網路流量。 探查檢查失敗的虛擬機器執行個體會從負載平衡器上移除，直到其恢復正常運作且探查判斷其健全狀況良好為止。 使用 [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe?view=azure-cli-latest#create) 建立健康狀態探查，以檢視虛擬機器的健康狀態。 

```azurecli-interactive
  az network lb probe create \
    --resource-group myResourceGroupILB \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-the-load-balancer-rule"></a>建立負載平衡器規則

負載平衡器規則可定義連入流量的前端 IP 組態及接收流量的後端 IP 集區，以及所需的來源和目的地連接埠。 使用 [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#create) 建立負載平衡器規則 myHTTPRule，用來接聽前端集區 myFrontEnd 中的連接埠 80，以及用來將負載平衡的網路流量傳送到後端位址集區 myBackEndPool (也是使用連接埠 80)。 

```azurecli-interactive
  az network lb rule create \
    --resource-group myResourceGroupILB \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe  
```

## <a name="create-servers-for-the-backend-address-pool"></a>建立後端位址集區的伺服器

請先建立支援的虛擬網路資源，才可部署一些 VM 及測試您的負載平衡器。

### <a name="create-nics"></a>建立 NIC

使用 [az network nic create](/cli/azure/network/nic#az-network-nic-create) 建立兩個網路介面，並使其與私人 IP 位址產生關聯。 

```azurecli-interactive
for i in `seq 1 2`; do
  az network nic create \
    --resource-group myResourceGroupILB \
    --name myNic$i \
    --vnet-name myVnet \
    --subnet mySubnet \
    --lb-name myLoadBalancer \
    --lb-address-pools myBackEndPool
done
```

## <a name="create-backend-servers"></a>建立後端伺服器

在此範例中，您會建立兩個虛擬機器，作為負載平衡器的後端伺服器。 若要確認已成功建立負載平衡器，您也可在虛擬機器上安裝 NGINX。

### <a name="create-an-availability-set"></a>建立可用性設定組

使用 [az vm availabilityset create](/cli/azure/network/nic#az-network-availabilityset-create) 建立可用性設定組

 ```azurecli-interactive
  az vm availability-set create \
    --resource-group myResourceGroupILB \
    --name myAvailabilitySet
```

### <a name="create-two-virtual-machines"></a>建立兩部虛擬機器

您可以使用 cloud-init 組態檔來安裝 NGINX，並在 Linux 虛擬機器上執行 'Hello World' Node.js 應用程式。 您目前的殼層中，建立名為 cloud-init.txt 的檔案，並將下列設定複製及貼到殼層中。 請確定您正確複製整個 cloud-init 檔案，特別是第一行：

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
 
使用 [az vm create](/cli/azure/vm#az-vm-create) 建立虛擬機器。

 ```azurecli-interactive
for i in `seq 1 2`; do
  az vm create \
    --resource-group myResourceGroupILB \
    --name myVM$i \
    --availability-set myAvailabilitySet \
    --nics myNic$i \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
    done
```
部署 VM 可能需要幾分鐘的時間。

### <a name="create-a-vm-for-testing-the-load-balancer"></a>建立用來測試負載平衡器的 VM

若要測試負載平衡器，請建立虛擬機器 *myVMTest*，並將它與 *myNic3* 產生關聯。

```azurecli-interactive
 az vm create \
    --resource-group myResourceGroupILB \
    --name myVMTest \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password myPassword123456!
```

## <a name="test-the-internal-load-balancer"></a>測試內部負載平衡器

若要測試負載平衡器，您必須先取得負載平衡器的私人 IP 位址。 接著，登入虛擬機器 myVMTest，並在其網頁瀏覽器的網址列中輸入私人 IP 位址。

若要取得負載平衡器的私人 IP 位址，請使用 [az network lb show](/cli/azure/network/public-ip##az-network-lb-show)。 複製私人 IP 位址，並將其貼到您虛擬機器之網頁瀏覽器的網址列中 - *myVMTest*。

```azurecli-interactive
  az network lb show \
    --name myLoadBalancer \
    --resource-group myResourceGroupILB
``` 
![測試負載平衡器](./media/load-balancer-get-started-ilb-arm-cli/load-balancer-test.png)

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令來移除資源群組、負載平衡器和所有相關資源。

```azurecli-interactive 
  az group delete --name myResourceGroupILB
```


## <a name="next-steps"></a>後續步驟
在本文中，您已建立內部基本負載平衡器、將 VM 連結至該平衡器、設定負載平衡器流量規則、健康情況探查，然後測試負載平衡器。 若要深入了解負載平衡器及其相關聯的資源，請繼續閱讀操作說明文章。
