---
title: 建立有多站台裝載的應用程式閘道 - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure CLI，以建立裝載多個站台的應用程式閘道。
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 7/14/2018
ms.author: victorh
ms.openlocfilehash: 472c8666d8c57f56113d00e60d8de92e636dc7d7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995851"
---
# <a name="create-an-application-gateway-with-multiple-site-hosting-using-the-azure-cli"></a>使用 Azure CLI，以建立有多站台裝載的應用程式閘道

您可以使用 Azure CLI，在建立[應用程式閘道](application-gateway-introduction.md)時設定[裝載多個網站](application-gateway-multi-site-overview.md)。 在本教學課程中，您可以使用虛擬機器擴展集建立後端集區。 接著，您可以根據擁有的網域來設定接聽程式和規則，確保網路流量會抵達集區中的適當伺服器。 本教學課程假設您擁有多個網域，並使用 *www.contoso.com* 和 *www.fabrikam.com* 的範例。

在本文中，您將了解：

> [!div class="checklist"]
> * 設定網路
> * 建立應用程式閘道
> * 建立接聽程式和路由規則
> * 建立包含後端集區的虛擬機器擴展集
> * 在網域中建立 CNAME 記錄

![多站台路由範例](./media/tutorial-multisite-cli/scenario.png)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

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

您可以使用 [az network application-gateway create](/cli/azure/network/application-gateway#create) 來建立名為 myAppGateway 的應用程式閘道。 當您使用 Azure CLI 建立應用程式閘道時，需要指定設定資訊，例如容量、SKU 和 HTTP 設定。 應用程式閘道會指派給您先前建立的 myAGSubnet 和 myAGPublicIPAddress。 

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
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress
```

可能需要幾分鐘的時間來建立應用程式閘道。 建立應用程式閘道後，您可以看到它的這些新功能：

- appGatewayBackendPool - 應用程式閘道必須至少有一個後端位址集區。
- appGatewayBackendHttpSettings - 指定以連接埠 80 和 HTTP 通訊協定來進行通訊。
- appGatewayHttpListener - 與 appGatewayBackendPool 相關聯的預設接聽程式。
- appGatewayFrontendIP - 將 myAGPublicIPAddress 指派給 appGatewayHttpListener。
- rule1 - 與 appGatewayHttpListener 相關聯的預設路由規則。

### <a name="add-the-backend-pools"></a>新增後端集區

使用 [az network application-gateway address-pool create](/cli/azure/network/application-gateway#az-network_application_gateway_address_pool_create)，以新增包含後端伺服器所需且名為 *contosoPool* 和 *fabrikamPool* 的後端集區。

```azurecli-interactive
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name contosoPool
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name fabrikamPool
```

### <a name="add-listeners"></a>新增接聽程式

需要接聽程式才能讓應用程式閘道將流量適當地路由到後端集區。 在本教學課程中，您會為兩個網域建立兩個接聽程式。 在此範例中，會為 *www.contoso.com* 和 *www.fabrikam.com* 網域建立接聽程式。 

使用 [az network application-gateway http-listener create](/cli/azure/network/application-gateway#az-network_application_gateway_http_listener_create)，以新增路由流量時所需且名為 *contosoListener* 和 *fabrikamListener* 的接聽程式。

```azurecli-interactive
az network application-gateway http-listener create \
  --name contosoListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.com
az network application-gateway http-listener create \
  --name fabrikamListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.fabrikam.com   
  ```

### <a name="add-routing-rules"></a>新增路由規則

系統會依據規則的建立順序來處理規則，而且會根據傳送至應用程式閘道的 URL，使用符合 URL 的第一條規則將流量導向。 例如，如果您在相同的連接埠上同時使用基本接聽程式的規則和多站台接聽程式的規則，則必須將多站台接聽程式的規則列於基本接聽程式的規則之前，多站台規則才能如預期般運作。 

在此範例中，您會建立兩個新規則，並刪除您在建立應用程式閘道時所建立的預設規則。 您可以使用 [az network application-gateway rule create](/cli/azure/network/application-gateway#az-network_application_gateway_rule_create) 來新增規則。

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoListener \
  --rule-type Basic \
  --address-pool contosoPool
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name fabrikamRule \
  --resource-group myResourceGroupAG \
  --http-listener fabrikamListener \
  --rule-type Basic \
  --address-pool fabrikamPool
az network application-gateway rule delete \
  --gateway-name myAppGateway \
  --name rule1 \
  --resource-group myResourceGroupAG
```

## <a name="create-virtual-machine-scale-sets"></a>建立虛擬機器擴展集

在此範例中，您會建立三個虛擬機器擴展集，以支援應用程式閘道中的三個後端集區。 您所建立的擴展集名為 myvmss1、myvmss2 和 myvmss3。 每個擴展集都會包含兩個您安裝 NGINX 的虛擬機器執行個體。

```azurecli-interactive
for i in `seq 1 2`; do
  if [ $i -eq 1 ]
  then
    poolName="contosoPool"
  fi
  if [ $i -eq 2 ]
  then
    poolName="fabrikamPool"
  fi
  az vmss create \
    --name myvmss$i \
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
    --backend-pool-name $poolName
done
```

### <a name="install-nginx"></a>安裝 NGINX

```azurecli-interactive
for i in `seq 1 2`; do
  az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroupAG \
    --vmss-name myvmss$i \
    --settings '{
  "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
done
```

## <a name="create-a-cname-record-in-your-domain"></a>在網域中建立 CNAME 記錄

在以公用 IP 位址建立應用程式閘道之後，您可以取得 DNS 位址並用以在網域中建立 CNAME 記錄。 您可以使用 [az network public-ip show](/cli/azure/network/public-ip#az-network_public_ip_show) 來取得應用程式閘道的 DNS 位址。 複製 DNSSettings 的 *fqdn* 值，並用來作為所建立 CNAME 記錄的值。 

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [dnsSettings.fqdn] \
  --output tsv
```

不建議使用 A-records，因為重新啟動應用程式閘道時，可能會變更 VIP。

## <a name="test-the-application-gateway"></a>測試應用程式閘道

在瀏覽器的網址列中輸入您的網域名稱。 例如， http://www.contoso.com。

![在應用程式閘道中測試 contoso 網站](./media/tutorial-multisite-cli/application-gateway-nginxtest1.png)

將位址變更為您的另一個網域，您應該會看到類似下列的範例：

![測試應用程式閘道中的 fabrikam 網站](./media/tutorial-multisite-cli/application-gateway-nginxtest2.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立應用程式閘道
> * 建立接聽程式和路由規則
> * 建立包含後端集區的虛擬機器擴展集
> * 在網域中建立 CNAME 記錄

> [!div class="nextstepaction"]
> [深入了解應用程式閘道的用途](application-gateway-introduction.md)