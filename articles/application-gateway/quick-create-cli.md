---
title: 快速入門 - 使用 Azure 應用程式閘道引導網路流量 - Azure CLI | Microsoft Docs
description: 深入了解如何使用 Azure CLI 建立 Azure 應用程式閘道，以將網路流量導向後端集區中的虛擬機器。
services: application-gateway
author: vhorne
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.service: application-gateway
ms.devlang: azurecli
ms.topic: quickstart
ms.workload: infrastructure-services
ms.date: 02/14/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 62c4e51cd160ed7830eb42943225847857dc4963
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963623"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway---azure-cli"></a>快速入門：使用 Azure 應用程式閘道引導網路流量 - Azure CLI

您可以使用 Azure 應用程式閘道，透過將接聽程式指派給連接埠、建立規則以及將資源新增至後端集區，將您的應用程式網路流量導向至特定資源。

本快速入門示範如何使用 Azure CLI，快速建立在其後端集區中具有兩部虛擬機器的應用程式閘道。 然後進行測試以確定它正常運作。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

您必須一律在資源群組中建立資源。 使用 [az group create](/cli/azure/group#az-group-create) 建立資源群組。 

下列範例會在 eastus 位置建立名為 myResourceGroupAG 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>建立網路資源 

您必須為應用程式閘道建立虛擬網路，應用程式閘道才能與其他資源通訊。 您建立應用程式閘道時，可以同時建立虛擬網路。 這個範例中會建立兩個子網路：一個用於應用程式閘道，另一個用於虛擬機器。 

使用 [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create) 建立虛擬網路和子網路。 使用 [az network public-ip create](/cli/azure/network/public-ip#az-public-ip-create) 建立公用 IP 位址。

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
az network vnet subnet create \
  --name myBackendSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet   \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-backend-servers"></a>建立後端伺服器

在此範例中，您要建立兩個虛擬機器，作為應用程式閘道的後端伺服器。 

### <a name="create-two-virtual-machines"></a>建立兩部虛擬機器

您也可以在虛擬機器上安裝 NGINX，以確認成功建立應用程式閘道。 您可以使用 cloud-init 組態檔來安裝 NGINX，並在 Linux 虛擬機器上執行 'Hello World' Node.js 應用程式。 

您目前的殼層中，建立名為 cloud-init.txt 的檔案，並將下列設定複製及貼到殼層中。 請確定您正確複製整個 cloud-init 檔案，特別是第一行：

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

使用 [az network nic create](/cli/azure/network/nic#az-network-nic-create) 建立網路介面。 使用 [az vm create](/cli/azure/vm#az-vm-create) 建立虛擬機器。

```azurecli-interactive
for i in `seq 1 2`; do
  az network nic create \
    --resource-group myResourceGroupAG \
    --name myNic$i \
    --vnet-name myVNet \
    --subnet myBackendSubnet
  az vm create \
    --resource-group myResourceGroupAG \
    --name myVM$i \
    --nics myNic$i \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
done
```

## <a name="create-the-application-gateway"></a>建立應用程式閘道

使用 [az network application-gateway create](/cli/azure/network/application-gateway#az-application-gateway-create) 建立應用程式閘道。 當您使用 Azure CLI 建立應用程式閘道時，需要指定設定資訊，例如容量、SKU 和 HTTP 設定。 系統會新增網路介面的私人 IP 位址，作為應用程式閘道後端集區中的伺服器。

```azurecli-interactive
address1=$(az network nic show --name myNic1 --resource-group myResourceGroupAG | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
address2=$(az network nic show --name myNic2 --resource-group myResourceGroupAG | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --capacity 2 \
  --sku Standard_Medium \
  --http-settings-cookie-based-affinity Enabled \
  --public-ip-address myAGPublicIPAddress \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --servers "$address1" "$address2"
```

最多可能需要 30 分鐘的時間來建立應用程式閘道。 建立應用程式閘道後，您可以看到它的這些功能：

- appGatewayBackendPool - 應用程式閘道必須至少有一個後端位址集區。
- appGatewayBackendHttpSettings - 指定以連接埠 80 和 HTTP 通訊協定來進行通訊。
- appGatewayHttpListener - 與 appGatewayBackendPool 相關聯的預設接聽程式。
- appGatewayFrontendIP - 將 myAGPublicIPAddress 指派給 appGatewayHttpListener。
- rule1 - 與 appGatewayHttpListener 相關聯的預設路由規則。

## <a name="test-the-application-gateway"></a>測試應用程式閘道

建立應用程式閘道時不需要安裝 NGINX，但是您會在本快速入門中安裝它，以確認是否已成功建立應用程式閘道。 若要取得應用程式閘道的公用 IP 位址，請使用 [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show)。 將公用 IP 位址複製並貼到您瀏覽器的網址列。

```azurepowershell-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
``` 

![測試應用程式閘道](./media/quick-create-cli/application-gateway-nginxtest.png)

當您重新整理瀏覽器時，您應該會看到其他虛擬機器的名稱出現。

## <a name="clean-up-resources"></a>清除資源

首先探索使用應用程式閘道建立的資源，若不再需要，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令來移除資源群組、應用程式閘道和所有相關資源。

```azurecli-interactive 
az group delete --name myResourceGroupAG
```
 
## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Azure CLI 以應用程式閘道管理網路流量](./tutorial-manage-web-traffic-cli.md)

