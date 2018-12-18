---
title: 根據 URL 路由網路流量 - Azure PowerShell
description: 了解如何使用 Azure PowerShell，根據 URL 將網路流量路由到伺服器的特定可調整集區。
services: application-gateway
author: vhorne
manager: jpconnock
ms.service: application-gateway
ms.topic: tutorial
ms.workload: infrastructure-services
ms.date: 7/13/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 66f79b68c003aa3605653b0decc091d22fbf3860
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39055180"
---
# <a name="route-web-traffic-based-on-the-url-using-azure-powershell"></a>使用 Azure PowerShell 根據 URL 路由網路流量

您可以使用 Azure PowerShell，根據用來存取您應用程式的 URL，將網路流量設定為路由到特定可調整的伺服器集區。 在本教學課程中，您可以使用[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)來建立具有 3 個後端集區的 [Azure 應用程式閘道](application-gateway-introduction.md)。 每個後端集區都有特定用途，例如處理常見資料、影像和影片。  將流量路由到個別集區，可確保您的客戶在需要時取得資訊。

若要啟用流量路由，請建立指派給接聽程式的[路由規則](application-gateway-url-route-overview.md)，接聽程式會接聽特定連接埠，以確保網路流量抵達集區中的適當伺服器。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立接聽程式、URL 路徑對應與規則
> * 建立可調整的後端集區

![URL 路由範例](./media/tutorial-url-route-powershell/scenario.png)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組版本 3.6 或更新版本。 若要尋找版本，請執行 ` Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。

由於建立資源需要時間，因此完成本教學課程可能需要最多 90 分鐘的時間。

## <a name="create-a-resource-group"></a>建立資源群組

您需要建立包含所有應用程式資源的資源群組。 

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。  

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>建立網路資源

無論您是使用現有虛擬網路，或是新建一個，您都必須確定其中包含只會用於應用程式閘道的子網路。 在本教學課程中，您會建立應用程式閘道的子網路和擴展集的子網路。 您會建立公用 IP 位址，以啟用應用程式閘道中的資源存取權。

使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 建立 myAGSubnet 和 myBackendSubnet 子網路設定。 使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 與子網路設定來建立名為 myVNet 的虛擬網路。 最後，使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 建立名為 myAGPublicIPAddress 的公用 IP 位址。 這些資源可用來為應用程式閘道及其相關聯的資源提供網路連線。

```azurepowershell-interactive
$backendSubnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24

$agSubnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.2.0/24

$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $backendSubnetConfig, $agSubnetConfig
$pip = New-AzureRmPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myAGPublicIPAddress `
  -AllocationMethod Dynamic
```

## <a name="create-an-application-gateway"></a>建立應用程式閘道

在本節中，您會建立支援應用程式閘道的資源，並在最後建立應用程式閘道。 您所建立的資源包括：

- IP 組態和前端連接埠 - 將您先前建立的子網路與應用程式閘道產生關聯，並且指派用來存取它的連接埠。
- 預設集區 - 所有應用程式閘道都必須具有至少一個伺服器後端集區。
- 預設接聽程式和規則 - 預設接聽程式會接聽所指派連接埠上的流量，而預設規則會將流量傳送到預設集區。

### <a name="create-the-ip-configurations-and-frontend-port"></a>建立 IP 設定與前端連接埠

使用 [New-AzureRmApplicationGatewayIPConfiguration](/powershell/module/azurerm.network/new-azurermapplicationgatewayipconfiguration) 將您先前建立的 myAGSubnet 與應用程式閘道產生關聯。 使用 [New-AzureRmApplicationGatewayFrontendIPConfig](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendipconfig) 將 myAGPublicIPAddress 指派至應用程式閘道。

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet

$subnet=$vnet.Subnets[0]

$pip = Get-AzureRmPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Name myAGPublicIPAddress

$gipconfig = New-AzureRmApplicationGatewayIPConfiguration `
  -Name myAGIPConfig `
  -Subnet $subnet

$fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -PublicIPAddress $pip

$frontendport = New-AzureRmApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -Port 80
```

### <a name="create-the-default-pool-and-settings"></a>建立預設集區和設定

使用 [New-AzureRmApplicationGatewayBackendAddressPool](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendaddresspool) 為應用程式閘道建立名為 appGatewayBackendPool 的預設後端集區。 使用 [New-AzureRmApplicationGatewayBackendHttpSettings](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendhttpsettings) 設定後端集區的設定。

```azurepowershell-interactive
$defaultPool = New-AzureRmApplicationGatewayBackendAddressPool `
  -Name appGatewayBackendPool

$poolSettings = New-AzureRmApplicationGatewayBackendHttpSettings `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 120
```

### <a name="create-the-default-listener-and-rule"></a>建立預設接聽程式和規則

需要接聽程式才能讓應用程式閘道將流量適當地路由到後端集區。 在本教學課程中，您將建立兩個接聽程式。 您所建立的第一個基本接聽程式會接聽根 URL 的流量。 您所建立的第二個接聽程式會接聽特定 URL 的流量。

使用 [New-AzureRmApplicationGatewayHttpListener](/powershell/module/azurerm.network/new-azurermapplicationgatewayhttplistener) 以及您先前建立的前端設定和前端連接埠，來建立名為 myDefaultListener 的預設接聽程式。 

接聽程式需要規則才能知道要將哪個後端集區用於連入流量。 使用 [New-AzureRmApplicationGatewayRequestRoutingRule](/powershell/module/azurerm.network/new-azurermapplicationgatewayrequestroutingrule) 建立名為 rule1 的基本規則。

```azurepowershell-interactive
$defaultlistener = New-AzureRmApplicationGatewayHttpListener `
  -Name myDefaultListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport

$frontendRule = New-AzureRmApplicationGatewayRequestRoutingRule `
  -Name rule1 `
  -RuleType Basic `
  -HttpListener $defaultlistener `
  -BackendAddressPool $defaultPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>建立應用程式閘道

現在您已建立必要的支援資源，請使用 [New-AzureRmApplicationGatewaySku](/powershell/module/azurerm.network/new-azurermapplicationgatewaysku) 指定名為 myAppGateway 之應用程式閘道的參數，然後使用 [New-AzureRmApplicationGateway](/powershell/module/azurerm.network/new-azurermapplicationgateway) 加以建立。

```azurepowershell-interactive
$sku = New-AzureRmApplicationGatewaySku `
  -Name Standard_Medium `
  -Tier Standard `
  -Capacity 2

$appgw = New-AzureRmApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $defaultPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendport `
  -HttpListeners $defaultlistener `
  -RequestRoutingRules $frontendRule `
  -Sku $sku
```

建立應用程式閘道最多需要 30 分鐘，請等候部署成功完成後，再繼續進行至下一節。 此時在教學課程中，您會有應用程式閘道可接聽通訊埠 80 上的流量，並將流量傳送給伺服器的預設集區。

### <a name="add-image-and-video-backend-pools-and-port"></a>新增映像和視訊後端集區及連接埠

將名為 imagesBackendPool 和 videoBackendPool 的後端集區新增至應用程式閘道 [Add-AzureRmApplicationGatewayBackendAddressPool](/powershell/module/azurerm.network/add-azurermapplicationgatewaybackendaddresspool)。 使用 [Add-AzureRmApplicationGatewayFrontendPort](/powershell/module/azurerm.network/add-azurermapplicationgatewayfrontendport) 新增集區的前端連接埠。 使用 [Set-AzureRmApplicationGateway](/powershell/module/azurerm.network/set-azurermapplicationgateway) 將變更提交至應用程式閘道。

```azurepowershell-interactive
$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

Add-AzureRmApplicationGatewayBackendAddressPool `
  -ApplicationGateway $appgw `
  -Name imagesBackendPool

Add-AzureRmApplicationGatewayBackendAddressPool `
  -ApplicationGateway $appgw `
  -Name videoBackendPool

Add-AzureRmApplicationGatewayFrontendPort `
  -ApplicationGateway $appgw `
  -Name bport `
  -Port 8080

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

更新應用程式閘道可能也需要 20 分鐘的時間才能完成。

### <a name="add-backend-listener"></a>新增後端接聽程式

使用 [Add-AzureRmApplicationGatewayHttpListener](/powershell/module/azurerm.network/add-azurermapplicationgatewayhttplistener) 新增名為 backendListener、路由流量所需的後端接聽程式。

```azurepowershell-interactive
$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$backendPort = Get-AzureRmApplicationGatewayFrontendPort `
  -ApplicationGateway $appgw `
  -Name bport

$fipconfig = Get-AzureRmApplicationGatewayFrontendIPConfig `
  -ApplicationGateway $appgw

Add-AzureRmApplicationGatewayHttpListener `
  -ApplicationGateway $appgw `
  -Name backendListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $backendPort

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

### <a name="add-url-path-map"></a>新增 URL 路徑對應

URL 路徑對應可確保傳送至應用程式的 URL，會路由到特定的後端集區。 使用 [New-AzureRmApplicationGatewayPathRuleConfig](/powershell/module/azurerm.network/new-azurermapplicationgatewaypathruleconfig) 及 [Add-AzureRmApplicationGatewayUrlPathMapConfig](/powershell/module/azurerm.network/add-azurermapplicationgatewayurlpathmapconfig) 建立名為 imagePathRule 和 videoPathRule 的 URL 路徑對應。

```azurepowershell-interactive
$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$poolSettings = Get-AzureRmApplicationGatewayBackendHttpSettings `
  -ApplicationGateway $appgw `
  -Name myPoolSettings

$imagePool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -ApplicationGateway $appgw `
  -Name imagesBackendPool

$videoPool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -ApplicationGateway $appgw `
  -Name videoBackendPool

$defaultPool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -ApplicationGateway $appgw `
  -Name appGatewayBackendPool

$imagePathRule = New-AzureRmApplicationGatewayPathRuleConfig `
  -Name imagePathRule `
  -Paths "/images/*" `
  -BackendAddressPool $imagePool `
  -BackendHttpSettings $poolSettings

$videoPathRule = New-AzureRmApplicationGatewayPathRuleConfig `
  -Name videoPathRule `
    -Paths "/video/*" `
    -BackendAddressPool $videoPool `
    -BackendHttpSettings $poolSettings

Add-AzureRmApplicationGatewayUrlPathMapConfig `
  -ApplicationGateway $appgw `
  -Name urlpathmap `
  -PathRules $imagePathRule, $videoPathRule `
  -DefaultBackendAddressPool $defaultPool `
  -DefaultBackendHttpSettings $poolSettings

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

### <a name="add-routing-rule"></a>新增路由規則

路由規則會將 URL 對應與您所建立的接聽程式產生關聯。 使用 [Add-AzureRmApplicationGatewayRequestRoutingRule](/powershell/module/azurerm.network/add-azurermapplicationgatewayrequestroutingrule) 新增名為 rule2 的規則。

```azurepowershell-interactive
$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$backendlistener = Get-AzureRmApplicationGatewayHttpListener `
  -ApplicationGateway $appgw `
  -Name backendListener

$urlPathMap = Get-AzureRmApplicationGatewayUrlPathMapConfig `
  -ApplicationGateway $appgw `
  -Name urlpathmap

Add-AzureRmApplicationGatewayRequestRoutingRule `
  -ApplicationGateway $appgw `
  -Name rule2 `
  -RuleType PathBasedRouting `
  -HttpListener $backendlistener `
  -UrlPathMap $urlPathMap

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

## <a name="create-virtual-machine-scale-sets"></a>建立虛擬機器擴展集

在此範例中，您要建立三個虛擬機器擴展集，以支援您所建立的三個後端集區。 您所建立的擴展集名為 myvmss1、myvmss2 和 myvmss3。 當您設定 IP 設定時，要將擴展集指派給後端集區。

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet

$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$backendPool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -Name appGatewayBackendPool `
  -ApplicationGateway $appgw

$imagesPool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -Name imagesBackendPool `
  -ApplicationGateway $appgw

$videoPool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -Name videoBackendPool `
  -ApplicationGateway $appgw

for ($i=1; $i -le 3; $i++)
{
  if ($i -eq 1)
  {
     $poolId = $backendPool.Id
  }
  if ($i -eq 2) 
  {
    $poolId = $imagesPool.Id
  }
  if ($i -eq 3)
  {
    $poolId = $videoPool.Id
  }

  $ipConfig = New-AzureRmVmssIpConfig `
    -Name myVmssIPConfig$i `
    -SubnetId $vnet.Subnets[1].Id `
    -ApplicationGatewayBackendAddressPoolsId $poolId

  $vmssConfig = New-AzureRmVmssConfig `
    -Location eastus `
    -SkuCapacity 2 `
    -SkuName Standard_DS2 `
    -UpgradePolicyMode Automatic

  Set-AzureRmVmssStorageProfile $vmssConfig `
    -ImageReferencePublisher MicrosoftWindowsServer `
    -ImageReferenceOffer WindowsServer `
    -ImageReferenceSku 2016-Datacenter `
    -ImageReferenceVersion latest
    -OsDiskCreateOption FromImage

  Set-AzureRmVmssOsProfile $vmssConfig `
    -AdminUsername azureuser `
    -AdminPassword "Azure123456!" `
    -ComputerNamePrefix myvmss$i

  Add-AzureRmVmssNetworkInterfaceConfiguration `
    -VirtualMachineScaleSet $vmssConfig `
    -Name myVmssNetConfig$i `
    -Primary $true `
    -IPConfiguration $ipConfig

  New-AzureRmVmss `
    -ResourceGroupName myResourceGroupAG `
    -Name myvmss$i `
    -VirtualMachineScaleSet $vmssConfig
}
```

### <a name="install-iis"></a>安裝 IIS

每個擴展集會包含兩個其中安裝 IIS 的虛擬機器執行個體，IIS 會執行範例頁面，以測試應用程式閘道是否可運作。

```azurepowershell-interactive
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }

for ($i=1; $i -le 3; $i++)
{
  $vmss = Get-AzureRmVmss -ResourceGroupName myResourceGroupAG -VMScaleSetName myvmss$i
  Add-AzureRmVmssExtension -VirtualMachineScaleSet $vmss `
    -Name "customScript" `
    -Publisher "Microsoft.Compute" `
    -Type "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -Setting $publicSettings

  Update-AzureRmVmss `
    -ResourceGroupName myResourceGroupAG `
    -Name myvmss$i `
    -VirtualMachineScaleSet $vmss
}
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得應用程式閘道的公用 IP 位址。 將公用 IP 位址複製並貼到您瀏覽器的網址列。 例如，*http://52.168.55.24*, *http://52.168.55.24:8080/images/test.htm*, 或 *http://52.168.55.24:8080/video/test.htm*。

```azurepowershell-interactive
Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![在應用程式閘道中測試基底 URL](./media/tutorial-url-route-powershell/application-gateway-iistest.png)

將 URL 變更為 http://&lt;ip-address&gt;:8080/images/test.html，使用您的 IP 位址取代 &lt;ip-address&gt;，然後您就應該會看到類似下列的範例：

![在應用程式閘道中測試影像 URL](./media/tutorial-url-route-powershell/application-gateway-iistest-images.png)

將 URL 變更為 http://&lt;ip-address&gt;:8080/video/test.htm、使用您的 IP 位址取代 &lt;ip-address&gt;，就應該會看到類似下列的範例：

![在應用程式閘道中測試影片 URL](./media/tutorial-url-route-powershell/application-gateway-iistest-video.png)

## <a name="clean-up-resources"></a>清除資源

不再需要時，使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除資源群組、應用程式閘道和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroupAG
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立接聽程式、URL 路徑對應與規則
> * 建立可調整的後端集區

> [!div class="nextstepaction"]
> [根據 URL 重新導向網路流量](./tutorial-url-redirect-powershell.md)
