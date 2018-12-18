---
title: 啟用 Web 應用程式防火牆 - Azure CLI
description: 了解如何使用 Azure CLI，在應用程式閘道上使用 Web 應用程式防火牆來限制網路流量。
services: application-gateway
author: vhorne
manager: jpconnock
ms.service: application-gateway
ms.topic: tutorial
ms.workload: infrastructure-services
ms.date: 7/14/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 92e4f93f3bf384ef8ec75a6f65d49a7b81e0925b
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432319"
---
# <a name="tutorial-enable-web-application-firewall-using-the-azure-cli"></a>教學課程：使用 Azure CLI 啟用 Web 應用程式防火牆

您可以使用 [Web 應用程式防火牆](waf-overview.md) (WAF)，限制[應用程式閘道](overview.md)上的流量。 WAF 會使用 [OWASP](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 規則來保護您的應用程式。 這些規則包括防禦諸如 SQL 插入攻擊、跨網站指令碼攻擊，以及工作階段劫持等攻擊。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立已啟用 WAF 的應用程式閘道
> * 建立虛擬機器擴展集
> * 建立儲存體帳戶並設定診斷

![Web 應用程式防火牆範例](./media/tutorial-restrict-web-traffic-cli/scenario-waf.png)

如果您想要的話，可以使用 [Azure PowerShell](tutorial-restrict-web-traffic-powershell.md) 完成本教學課程。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.4 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用 [az group create](/cli/azure/group#az-group-create) 建立名為 myResourceGroupAG 的 Azure 資源群組。

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>建立網路資源

這些虛擬網路和子網路可用來為應用程式閘道及其相關聯的資源提供網路連線。 使用 [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create) 和 [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create) 建立名為 myVNet 的虛擬網路，以及名為 myAGSubnet 的子網路。 使用 [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) 建立名為 myAGPublicIPAddress 的公用 IP 位址。

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myBackendSubnet \
  --subnet-prefix 10.0.1.0/24

az network vnet subnet create \
  --name myAGSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24

az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-an-application-gateway-with-a-waf"></a>建立包含 WAF 的應用程式閘道

您可以使用 [az network application-gateway create](/cli/azure/network/application-gateway#az-application-gateway-create) 來建立名為 myAppGateway 的應用程式閘道。 當您使用 Azure CLI 建立應用程式閘道時，需要指定設定資訊，例如容量、SKU 和 HTTP 設定。 應用程式閘道會指派給您先前建立的 myAGSubnet 和 myAGPublicIPAddress。

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --capacity 2 \
  --sku WAF_Medium \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress

az network application-gateway waf-config set \
  --enabled true \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --firewall-mode Detection \
  --rule-set-version 3.0
```

可能需要幾分鐘的時間來建立應用程式閘道。 建立應用程式閘道後，您可以看到它的這些新功能：

- appGatewayBackendPool - 應用程式閘道必須至少有一個後端位址集區。
- appGatewayBackendHttpSettings - 指定以連接埠 80 和 HTTP 通訊協定來進行通訊。
- appGatewayHttpListener - 與 appGatewayBackendPool 相關聯的預設接聽程式。
- appGatewayFrontendIP - 將 myAGPublicIPAddress 指派給 appGatewayHttpListener。
- rule1 - 與 appGatewayHttpListener 相關聯的預設路由規則。

## <a name="create-a-virtual-machine-scale-set"></a>建立虛擬機器擴展集

在此範例中，您會建立虛擬機器擴展集，為應用程式閘道中的後端集區提供兩部伺服器。 擴展集中的虛擬機器會與 myBackendSubnet 子網路相關聯。 若要建立擴展集，您可以使用 [az vmss create](/cli/azure/vmss#az-vmss-create)。

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
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],"commandToExecute": "./install_nginx.sh" }'
```

## <a name="create-a-storage-account-and-configure-diagnostics"></a>建立儲存體帳戶並設定診斷

在本教學課程中，應用程式閘道會使用儲存體帳戶來儲存資料，以達到偵測與預防的目的。 您也可以使用 Log Analytics 或事件中樞來記錄資料。 

### <a name="create-a-storage-account"></a>建立儲存體帳戶

使用 [az storage account create](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create) 建立名為 myagstore1 的儲存體帳戶。

```azurecli-interactive
az storage account create \
  --name myagstore1 \
  --resource-group myResourceGroupAG \
  --location eastus \
  --sku Standard_LRS \
  --encryption blob
```

### <a name="configure-diagnostics"></a>設定診斷

設定診斷以將資料記錄到 ApplicationGatewayAccessLog、ApplicationGatewayPerformanceLog 和 ApplicationGatewayFirewallLog 記錄。 使用您的訂用帳戶識別碼來取代 `<subscriptionId>`，然後使用 [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings?view=azure-cli-latest#az-monitor-diagnostic-settings-create) 來設定診斷。

```azurecli-interactive
appgwid=$(az network application-gateway show --name myAppGateway --resource-group myResourceGroupAG --query id -o tsv)

storeid=$(az storage account show --name myagstore1 --resource-group myResourceGroupAG --query id -o tsv)

az monitor diagnostic-settings create --name appgwdiag --resource $appgwid \
  --logs '[ { "category": "ApplicationGatewayAccessLog", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "ApplicationGatewayPerformanceLog", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "ApplicationGatewayFirewallLog", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } } ]' \
  --storage-account $storeid
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

若要取得應用程式閘道的公用 IP 位址，請使用 [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show)。 將公用 IP 位址複製並貼到您瀏覽器的網址列。

```azurepowershell-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![在應用程式閘道中測試基底 URL](./media/tutorial-restrict-web-traffic-cli/application-gateway-nginxtest.png)

## <a name="clean-up-resources"></a>清除資源

若不再需要，可移除資源群組、應用程式閘道和所有相關資源。

```azurecli-interactive
az group delete --name myResourceGroupAG --location eastus
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立已啟用 WAF 的應用程式閘道
> * 建立虛擬機器擴展集
> * 建立儲存體帳戶並設定診斷

> [!div class="nextstepaction"]
> [建立具有 SSL 終止的應用程式閘道](./tutorial-ssl-cli.md)
