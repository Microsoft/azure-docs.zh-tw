---
title: 建立包含 SSL 終止的應用程式閘道 - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure CLI 建立應用程式閘道，並新增 SSL 終止的憑證。
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.service: application-gateway
ms.topic: article
ms.workload: infrastructure-services
ms.date: 7/14/2018
ms.author: victorh
ms.openlocfilehash: 4ca97271608fda447cbda35c7edfa7b55d266fe5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46985345"
---
# <a name="create-an-application-gateway-with-ssl-termination-using-the-azure-cli"></a>使用 Azure CLI 建立包含 SSL 終止的應用程式閘道

您可以使用 Azure CLI 來建立[應用程式閘道](application-gateway-introduction.md)，當中包含使用[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)作為後端伺服器的 [SSL 終止](application-gateway-backend-ssl.md)憑證。 在此範例中，該擴展集包含兩個虛擬機器執行個體，這些執行個體會新增至應用程式閘道的預設後端集區。

在本文中，您將了解：

> [!div class="checklist"]
> * 建立自我簽署憑證
> * 設定網路
> * 建立包含憑證的應用程式閘道
> * 建立包含預設後端集區的虛擬機器擴展集

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="create-a-self-signed-certificate"></a>建立自我簽署憑證

若要在生產環境中使用，您應該匯入由受信任提供者所簽署的有效憑證。 對於此教學課程，您會使用 openssl 命令建立自我簽署的憑證和 pfx 檔案。

```azurecli-interactive
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out appgwcert.crt
```

輸入對憑證有意義的值。 您可以接受預設值。

```azurecli-interactive
openssl pkcs12 -export -out appgwcert.pfx -inkey privateKey.key -in appgwcert.crt
```

輸入憑證的密碼。 在此範例中會使用 Azure123456! 。

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用 [az group create](/cli/azure/group#create) 建立資源群組。

下列範例會在 eastus 位置建立名為 myResourceGroupAG 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>建立網路資源

使用 [az network vnet create](/cli/azure/network/vnet#az-net) 建立名為 myVNet 的虛擬網路，以及名為 myAGSubnet 的子網路。 然後您可以使用 [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network_vnet_subnet_create) 新增名為 myBackendSubnet 的子網路，後端伺服器需要該子網路。 使用 [az network public-ip create](/cli/azure/network/public-ip#az-network_public_ip_create) 建立名為 myAGPublicIPAddress 的公用 IP 位址。

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
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-the-application-gateway"></a>建立應用程式閘道

您可以使用 [az network application-gateway create](/cli/azure/network/application-gateway#create) 來建立應用程式閘道。 當您使用 Azure CLI 建立應用程式閘道時，需要指定設定資訊，例如容量、SKU 和 HTTP 設定。 

應用程式閘道會指派給您先前建立的 myAGSubnet 和 myAGPublicIPAddress。 在此範例中，您會在建立應用程式閘道時讓您建立的憑證與其密碼產生關聯。 

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_Medium \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 443 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress \
  --cert-file appgwcert.pfx \
  --cert-password "Azure123456!"

```

 可能需要幾分鐘的時間來建立應用程式閘道。 建立應用程式閘道後，您可以看到它的這些新功能：

- appGatewayBackendPool - 應用程式閘道必須至少有一個後端位址集區。
- appGatewayBackendHttpSettings - 指定以連接埠 80 和 HTTP 通訊協定來進行通訊。
- appGatewayHttpListener - 與 appGatewayBackendPool 相關聯的預設接聽程式。
- appGatewayFrontendIP - 將 myAGPublicIPAddress 指派給 appGatewayHttpListener。
- rule1 - 與 appGatewayHttpListener 相關聯的預設路由規則。

## <a name="create-a-virtual-machine-scale-set"></a>建立虛擬機器擴展集

在此範例中，您會建立虛擬機器擴展集，以在應用程式閘道中提供預設後端集區的伺服器。 擴展集中的虛擬機器會與 myBackendSubnet 和 appGatewayBackendPool 相關聯。 若要建立擴展集，您可以使用 [az vmss create](/cli/azure/vmss#az-vmss-create)。

```azurecli-interactive
az vmss create \
  --name myvmss \
  --resource-group myResourceGroupAG \
  --image UbuntuLTS \
  --admin-username azureuser \
  --admin-password Azure123456! \
  --instance-count 2 \
  --vnet-name myVNet \
  --subnet myBackendSubnet \
  --vm-sku Standard_DS2 \
  --upgrade-policy-mode Automatic \
  --app-gateway myAppGateway \
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>安裝 NGINX

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vmss-name myvmss \
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

若要取得應用程式閘道的公用 IP 位址，您可以使用 [az network public-ip show](/cli/azure/network/public-ip#az-network_public_ip_show)。 將公用 IP 位址複製並貼到您瀏覽器的網址列。

```azurepowershell-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![安全警告](./media/application-gateway-ssl-cli/application-gateway-secure.png)

若要在使用自我簽署憑證時接受安全性警告，請依序按一下 [詳細資料] 與 [繼續瀏覽網頁]。 接著會顯示受保護的 NGINX 網站，如下列範例所示：

![在應用程式閘道中測試基底 URL](./media/application-gateway-ssl-cli/application-gateway-nginx.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立自我簽署憑證
> * 設定網路
> * 建立包含憑證的應用程式閘道
> * 建立包含預設後端集區的虛擬機器擴展集

若要深入了解應用程式閘道和其相關聯的資源，請繼續進行操作說明文章。
