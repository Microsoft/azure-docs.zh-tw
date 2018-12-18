---
title: 建立具有外部流量重新導向的應用程式閘道 - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure CLI，建立會將內部 Web 流量重新導向至適當集區的應用程式閘道。
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/24/2018
ms.author: victorh
ms.openlocfilehash: fa3548affade23e134860fb8e347c7813094aa12
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996871"
---
# <a name="create-an-application-gateway-with-external-redirection-using-the-azure-cli"></a>使用 Azure CLI 建立具有外部重新導向的應用程式閘道

您可以使用 Azure CLI，在建立[應用程式閘道](application-gateway-introduction.md)時設定 [Web 流量重新導向](application-gateway-multi-site-overview.md)。 在本教學課程中，您會設定接聽程式和規則，以將抵達應用程式閘道的 Web 流量重新導向至外部網站。

在本文中，您將了解：

> [!div class="checklist"]
> * 設定網路
> * 建立接聽程式和重新導向規則
> * 建立應用程式閘道

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

使用 [az network vnet create](/cli/azure/network/vnet#az-net) 建立名為 myVNet 的虛擬網路，以及名為 myAGSubnet 的子網路。 使用 [az network public-ip create](/cli/azure/network/public-ip#az-network_public_ip_create) 建立名為 myAGPublicIPAddress 的公用 IP 位址。 這些資源可用來為應用程式閘道及其相關聯的資源提供網路連線。

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-an-application-gateway"></a>建立應用程式閘道

您可以使用 [az network application-gateway create](/cli/azure/network/application-gateway#create) 來建立名為 myAppGateway 的應用程式閘道。 當您使用 Azure CLI 建立應用程式閘道時，需要指定設定資訊，例如容量、SKU 和 HTTP 設定。 應用程式閘道會指派給您先前建立的 myAGSubnet 和 myPublicIPSddress。 

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
  --frontend-port 8080 \
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

### <a name="add-the-redirection-configuration"></a>新增重新導向設定

使用 [az network application-gateway redirect-config create](/cli/azure/network/application-gateway/redirect-config#az-network_application_gateway_redirect_config_create) 新增重新導向設定，以將流量從應用程式閘道傳送至 *bing.com*。

```azurecli-interactive
az network application-gateway redirect-config create \
  --name myredirect \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --type Temporary \
  --target-url "http://bing.com"
```

### <a name="add-a-listener-and-routing-rule"></a>新增接聽程式和路由規則

需要接聽程式才能讓應用程式閘道適當地路由傳送流量。 使用 [az network application-gateway http-listener create](/cli/azure/network/application-gateway#az-network_application_gateway_http_listener_create) 建立接聽程式，並讓其具有使用 [az network application-gateway frontend-port create](/cli/azure/network/application-gateway#az-network_application_gateway_frontend_port_create) 所建立的前端連接埠。 接聽程式需要規則才能知道要將傳入流量傳送到何處。 使用 [az network application-gateway rule create](/cli/azure/network/application-gateway#az-network_application_gateway_rule_create) 建立名為「redirectRule」且具有重新導向設定的基本規則。

```azurecli-interactive
az network application-gateway frontend-port create \
  --port 80 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name redirectPort
az network application-gateway http-listener create \
  --name redirectListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port redirectPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name redirectRule \
  --resource-group myResourceGroupAG \
  --http-listener redirectListener \
  --rule-type Basic \
  --redirect-config myredirect
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

若要取得應用程式閘道的公用 IP 位址，您可以使用 [az network public-ip show](/cli/azure/network/public-ip#az-network_public_ip_show)。 將公用 IP 位址複製並貼到您瀏覽器的網址列。

您應該會看到瀏覽器中出現 bing.com。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> * 設定網路
> * 建立接聽程式和重新導向規則
> * 建立應用程式閘道

> [!div class="nextstepaction"]
> [深入了解應用程式閘道的用途](./application-gateway-introduction.md)