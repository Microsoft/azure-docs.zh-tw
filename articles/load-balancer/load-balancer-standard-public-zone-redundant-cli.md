---
title: 使用 Azure CLI 來進行區域備援 VM 負載平衡 | Microsoft Docs
description: 了解如何使用 Azure CLI 來建立具有區域備援前端的公用 Load Balancer Standard
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/09/2018
ms.author: kumud
ms.openlocfilehash: dc8a0228c9b2a0c2ab54bc14dc911698593ba691
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948531"
---
#  <a name="load-balance-vms-across-all-availability-zones-using-azure-cli"></a>使用 Azure CLI 來進行跨所有可用性區域的 VM 負載平衡

本文會逐步說明如何在不倚賴多個 DNS 記錄的情況下，建立具有區域備援前端的公用 [Load Balancer Standard](https://aka.ms/azureloadbalancerstandard) 來達到區域備援的目的。 單一前端 IP 位址會自動具備區域備援。  現在為負載平衡器使用區域備援前端時，只需使用單一 IP 位址即可在單一地區內橫跨所有可用性區域，並連線至該地區內某虛擬網路中的任何 VM。 萬一整個資料中心失敗或遺失，使用可用性區域可保護您的應用程式和資料免於受害。

如需有關搭配標準 Load Balancer 使用可用性區域的詳細資訊，請參閱[標準 Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.17 版或更新版本。  若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

> [!NOTE]
> 針對精選的 Azure 資源和區域及 VM 大小系列有提供「可用性區域」支援。 如需如何開始使用，以及有哪些 Azure 資源、區域和 VM 大小系列可供用來試用可用性區域，請參閱[可用性區域概觀](https://docs.microsoft.com/azure/availability-zones/az-overview)。 如需支援，您可以透過 [StackOverflow](https://stackoverflow.com/questions/tagged/azure-availability-zones) 與我們聯繫或[開啟 Azure 支援票證](../azure-supportability/how-to-create-azure-support-request.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。  

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

下列範例會在 *westeurope* 位置建立名為 *myResourceGroupSLB* 的資源群組：

```azurecli-interactive
az group create \
--name myResourceGroupSLB \
--location westeurope
```

## <a name="create-a-zone-redundant-public-ip-standard"></a>建立區域備援標準公用 IP
若要存取網際網路上您的應用程式，您需要負載平衡器的公用 IP 位址。 區域中的所有可用性區域會同時為區域備援前端提供服務。 請使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 來建立區域備援公用 IP 位址。 當您建立標準公用 IP 位址時，它預設會具備區域備援能力。

下列範例會在 *myResourceGroupLoadBalancer* 資源群組中建立名為 *myPublicIP* 的公用 IP 位址。

```azurecli-interactive
az network public-ip create \
--resource-group myResourceGroupSLB \
--name myPublicIP \
--sku Standard
```

## <a name="create-azure-load-balancer-standard"></a>建立 Azure Load Balancer Standard
本節將詳細說明如何建立及設定下列負載平衡器元件：
- 前端 IP 集區，可接收負載平衡器上的連入網路流量。
- 後端 IP 集區，前端集區在其中傳送負載平衡網路流量。
- 健康狀態探查，可判斷後端 VM 執行個體的健康狀態。
- 負載平衡器規則，可定義如何將流量分散至 VM。

### <a name="create-the-load-balancer"></a>建立負載平衡器
使用 [az network lb create](/cli/azure/network/lb#az-network-lb-create) 來建立標準負載平衡器。 下列範例會建立名為 *myLoadBalancer* 的負載平衡器，並將 *myPublicIP* 位址指派給前端 IP 設定。

```azurecli-interactive
az network lb create \
--resource-group myResourceGroupSLB \
--name myLoadBalancer \
--public-ip-address myPublicIP \
--frontend-ip-name myFrontEnd \
--backend-pool-name myBackEndPool \
--sku Standard
```

## <a name="create-health-probe-on-port-80"></a>在連接埠 80 上建立健康狀態探查

健全狀況探查會檢查所有虛擬機器執行個體，確認它們可以傳送網路流量。 探查檢查失敗的虛擬機器執行個體會從負載平衡器上移除，直到其恢復正常運作且探查判斷其健全狀況良好為止。 請使用 az network lb probe create 來建立健康狀態探查，以監視虛擬機器的健康狀態。 若要建立 TCP 健康狀態探查，請使用 [az network lb probe create](/cli/azure/network/lb/probe#az-network-lb-probe-create)。 下列範例會建立名為 myHealthProbe 的健康狀態探查：

```azurecli-interactive
az network lb probe create \
--resource-group myResourceGroupSLB \
--lb-name myLoadBalancer \
--name myHealthProbe \
--protocol tcp \
--port 80
```

## <a name="create-load-balancer-rule-for-port-80"></a>建立用於連接埠 80 的負載平衡器規則
負載平衡器規則可定義連入流量的前端 IP 組態及接收流量的後端 IP 集區，以及所需的來源和目的地連接埠。 使用 [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create) 建立負載平衡器規則 *myLoadBalancerRuleWeb*，用來接聽前端集區 *myFrontEndPool* 中的連接埠 80，以及用來將負載平衡的網路流量傳送到後端位址集區 *myBackEndPool* (也是使用連接埠 80)。

```azurecli-interactive
az network lb rule create \
--resource-group myResourceGroupSLB \
--lb-name myLoadBalancer \
--name myLoadBalancerRuleWeb \
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

使用 [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create)，在 myResourceGroup 中建立名為 *myVnet*且子網路名為 *mySubnet* 的虛擬網路。


```azurecli-interactive
az network vnet create \
--resource-group myResourceGroupSLB \
--location westeurope \
--name myVnet \
--subnet-name mySubnet
```

### <a name="create-a-network-security-group"></a>建立網路安全性群組

使用 [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create).來建立名為 *myNetworkSecurityGroup* 的網路安全性群組，以定義對您虛擬網路的輸入連線。

```azurecli-interactive
az network nsg create \
--resource-group myResourceGroupSLB \
--name myNetworkSecurityGroup
```

使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) 來針對連接埠 80 建立名為 *myNetworkSecurityGroupRule* 的安全性群組規則。

```azurecli-interactive
az network nsg rule create \
--resource-group myResourceGroupSLB \
--nsg-name myNetworkSecurityGroup \
--name myNetworkSecurityGroupRule \
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
使用 [az network nic create](/cli/azure/network/nic#az-network-nic-create) 來建立三個網路 NIC，並將它們與公用 IP 位址和網路安全性群組建立關聯。 下列範例會建立六個虛擬 NIC。 (您在下列步驟中針對應用程式建立的每部 VM 都有一個虛擬 NIC)。 您可以隨時建立其他虛擬 NIC 和 VM，並將它們新增至負載平衡器︰

```azurecli-interactive
for i in `seq 1 3`; do
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
在此範例中，您會建立三個位於區域 1、區域 2 及區域 3 的虛擬機器，用來作為負載平衡器的後端伺服器。 此外，您還會在虛擬機器上安裝 NGINX 來確認已成功建立該負載平衡器。

### <a name="create-cloud-init-config"></a>建立 Cloud-init 組態

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

### <a name="create-the-zonal-virtual-machines"></a>建立區域虛擬機器
使用 [az vm create](/cli/azure/vm#az-vm-create) 在區域 1、區域 2 及區域 3 建立 VM。 下列範例會在每個區域建立 VM 並產生 SSH 金鑰 (如果尚未存在的話)︰

在 westeurope 位置的每個區域 (區域 1、區域 2 和區域 3) 中建立 VM。

```azurecli-interactive
for i in `seq 1 3`; do
  az vm create \
    --resource-group myResourceGroupSLB \
    --name myVM$i \
    --nics myNic$i \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --zone $i \
    --custom-data cloud-init.txt
done
```
## <a name="test-the-load-balancer"></a>測試負載平衡器

使用 [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) 來取得負載平衡器的公用 IP 位址。 

```azurecli-interactive
  az network public-ip show \
    --resource-group myResourceGroupSLB \
    --name myPublicIP \
    --query [ipAddress] \
    --output tsv
``` 
接著，您可以在 Web 瀏覽器中輸入公用 IP 位址。 請記住，負載平衡器開始將流量分散給 VM 之前，VM 需要花幾分鐘的時間才會就緒。 應用程式隨即顯示，包括負載平衡器分散流量之 VM 的主機名稱，如下列範例所示：

![執行 Node.js 應用程式](./media/load-balancer-standard-public-zone-redundant-cli/running-nodejs-app.png)

若要查看負載平衡器如何將流量分散到所有三個可用性區域中執行您應用程式的各個 VM，您可以停止某個特定區域中的 VM，然後重新整理瀏覽器。

## <a name="next-steps"></a>後續步驟
- 深入了解[標準 Load Balancer](./load-balancer-standard-overview.md)



