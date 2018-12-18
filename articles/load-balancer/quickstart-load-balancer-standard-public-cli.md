---
title: 快速入門：建立公用標準負載平衡器 - Azure CLI | Microsoft Docs
description: 本快速入門說明如何使用 Azure CLI 建立公用負載平衡器
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to create a Standard Load balancer so that I can load balance internet traffic to VMs.
ms.assetid: a8bcdd88-f94c-4537-8143-c710eaa86818
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/20/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: e4429c5ec808b561360f5088236b16a9fa6a276a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46955124"
---
# <a name="quickstart-create-a-standard-load-balancer-to-load-balance-vms-using-azure-cli"></a>快速入門：使用 Azure CLI 來建立標準負載平衡器以平衡 VM 的負載

本快速入門示範如何建立標準負載平衡器。 若要測試負載平衡器，您要部署兩部執行 Ubuntu 伺服器的虛擬機器 (VM)，並平衡兩個 VM 間 Web 應用程式的負載。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

如果您選擇在本機安裝和使用 CLI，在執行本教學課程時，您必須執行 Azure CLI 2.0.28 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

下列範例會在 eastus 位置建立名為 myResourceGroupSLB 的資源群組：

```azurecli-interactive
  az group create \
    --name myResourceGroupSLB \
    --location eastus
```

## <a name="create-a-public-standard-ip-address"></a>建立公用標準 IP 位址

若要在網際網路上存取您的 Web 應用程式，您需要負載平衡器的公用 IP 位址。 標準負載平衡器只支援標準公用 IP 位址。 使用 [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#create)，在 myResourceGroupSLB 中建立名為 myPublicIP 的標準公用 IP 位址。

```azurecli-interactive
  az network public-ip create --resource-group myResourceGroupSLB --name myPublicIP --sku standard
```

## <a name="create-azure-load-balancer"></a>建立 Azure Load Balancer

本節將詳細說明如何建立及設定下列負載平衡器元件：
  - 前端 IP 集區，可接收負載平衡器上的連入網路流量。
  - 後端 IP 集區，前端集區在其中傳送負載平衡網路流量。
  - 健康狀態探查，可判斷後端 VM 執行個體的健康狀態。
  - 負載平衡器規則，可定義如何將流量分散至 VM。

### <a name="create-the-load-balancer"></a>建立負載平衡器

使用 [az network lb create](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest#create) 建立名為 **myLoadBalancer** 的公用 Azure Load Balancer，包含名為 **myFrontEnd** 的前端集區、名為 **myBackEndPool** 的後端集區，與您在前一個步驟中建立的公用 IP 位址 **myPublicIP** 相關聯。

```azurecli-interactive
  az network lb create \
    --resource-group myResourceGroupSLB \
    --name myLoadBalancer \
    --sku standard
    --public-ip-address myPublicIP \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool       
  ```

### <a name="create-the-health-probe"></a>建立健康狀態探查

健全狀況探查會檢查所有虛擬機器執行個體，確認它們可以傳送網路流量。 探查檢查失敗的虛擬機器執行個體會從負載平衡器上移除，直到其恢復正常運作且探查判斷其健全狀況良好為止。 使用 [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe?view=azure-cli-latest#create) 建立健康狀態探查，以檢視虛擬機器的健康狀態。 

```azurecli-interactive
  az network lb probe create \
    --resource-group myResourceGroupSLB \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-the-load-balancer-rule"></a>建立負載平衡器規則

負載平衡器規則可定義連入流量的前端 IP 組態及接收流量的後端 IP 集區，以及所需的來源和目的地連接埠。 使用 [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#create) 建立負載平衡器規則 myLoadBalancerRuleWeb，用來接聽前端集區 myFrontEnd 中的連接埠 80，以及用來將負載平衡的網路流量傳送到後端位址集區 myBackEndPool (也是使用連接埠 80)。 

```azurecli-interactive
  az network lb rule create \
    --resource-group myResourceGroupSLB \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe  
```

## <a name="configure-virtual-network"></a>設定虛擬網路

請先建立支援的虛擬網路資源，才可部署一些 VM 及測試您的負載平衡器。

### <a name="create-a-virtual-network"></a>建立虛擬網路

使用 [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#create)，在 *myResourceGroup* 中建立名為 *myVNet*、具有子網路 *mySubnet* 的虛擬網路。

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupSLB \
    --location eastus \
    --name myVnet \
    --subnet-name mySubnet
```
###  <a name="create-a-network-security-group"></a>建立網路安全性群組

如果是標準負載平衡器，後端位址中的 VM 都需要有屬於網路安全性群組的 NIC。 建立網路安全性群組，以定義虛擬網路的輸入連線。

```azurecli-interactive
  az network nsg create \
    --resource-group myResourceGroupSLB \
    --name myNetworkSecurityGroup
```

### <a name="create-a-network-security-group-rule"></a>建立網路安全性群組規則

建立網路安全性群組規則，以允許透過連接埠 80 的輸入流量。

```azurecli-interactive
  az network nsg rule create \
    --resource-group myResourceGroupSLB \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleHTTP \
    --protocol tcp \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```
### <a name="create-nics"></a>建立 NIC

使用 [az network nic create](/cli/azure/network/nic#az-network-nic-create) 建立三個網路介面，並使其與公用 IP 位址和網路安全性群組產生關聯。 

```azurecli-interactive
for i in `seq 1 2`; do
  az network nic create \
    --resource-group myResourceGroupSLB \
    --name myNic$i \
    --vnet-name myVnet \
    --subnet mySubnet \
    --network-security-group myNetworkSecurityGroup \
    --lb-name myLoadBalancer \
    --lb-address-pools myBackEndPool
done
```


## <a name="create-backend-servers"></a>建立後端伺服器

在此範例中，您要建立三個虛擬機器，作為負載平衡器的後端伺服器。 若要確認已成功建立負載平衡器，您也可在虛擬機器上安裝 NGINX。

### <a name="create-an-availability-set"></a>建立可用性設定組

使用 [az vm availabilityset create](/cli/azure/network/nic#az-network-availabilityset-create) 建立可用性設定組

 ```azurecli-interactive
  az vm availability-set create \
    --resource-group myResourceGroupSLB \
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
    --resource-group myResourceGroupSLB \
    --name myVM$i \
    --availability-set myAvailabilitySet \
    --nics myNic$i \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
    --no-wait
    done
```
部署 VM 可能需要幾分鐘的時間。

## <a name="test-the-load-balancer"></a>測試負載平衡器

若要取得負載平衡器的公用 IP 位址，請使用 [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show)。 將公用 IP 位址複製並貼到您瀏覽器的網址列。

```azurecli-interactive
  az network public-ip show \
    --resource-group myResourceGroupSLB \
    --name myPublicIP \
    --query [ipAddress] \
    --output tsv
``` 
   ![測試負載平衡器](./media/load-balancer-standard-public-cli/running-nodejs-app.png)

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令來移除資源群組、負載平衡器和所有相關資源。

```azurecli-interactive 
  az group delete --name myResourceGroupSLB
```
## <a name="next-step"></a>後續步驟
本快速入門中，您已建立標準 Load Balancer、將 VM 加以連結、設定負載平衡器流量規則、健康情況探查，接著測試負載平衡器。 若要深入了解 Azure Load Balancer，請繼續 Azure Load Balancer 的教學課程。

> [!div class="nextstepaction"]
> [Azure Load Balancer 教學課程](tutorial-load-balancer-standard-public-zone-redundant-portal.md)

