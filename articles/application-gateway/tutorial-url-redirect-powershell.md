---
title: 建立包含 URL 路徑型重新導向的應用程式閘道 - Azure PowerShell
description: 瞭解如何使用 Azure PowerShell，以建立包含 URL 路徑型重新導向流量功能的應用程式閘道。
services: application-gateway
author: vhorne
manager: jpconnock
ms.service: application-gateway
ms.topic: tutorial
ms.workload: infrastructure-services
ms.date: 7/13/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 11cd052ca07e074ce8303891f7a372890fdf7291
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45580594"
---
# <a name="create-an-application-gateway-with-url-path-based-redirection-using-azure-powershell"></a>使用 Azure PowerShell 以建立包含 URL 路徑型重新導向的應用程式閘道

您可以使用 Azure PowerShell，在建立[應用程式閘道](application-gateway-introduction.md)時設定 [URL 型路由規則](application-gateway-url-route-overview.md)。 在本教學課程中，您可以使用[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)來建立後端集區。 然後，您可以建立 URL 路由規則，確保 Web 流量會重新導向到適當的後端集區。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立應用程式閘道
> * 新增接聽程式和路由規則
> * 為後端集區建立虛擬機器擴展集

以下範例會顯示來自連接埠 8080 和 8081 並導向至相同後端集區的網站流量：

![URL 路由範例](./media/tutorial-url-redirect-powershell/scenario.png)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組版本 3.6 或更新版本。 若要尋找版本，請執行 ` Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。  

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>建立網路資源

使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 來建立 *myBackendSubnet* 和 *myAGSubnet* 的子網路設定。 使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 與子網路設定來建立名為 myVNet 的虛擬網路。 最後，使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 建立名為 myAGPublicIPAddress 的公用 IP 位址。 這些資源可用來為應用程式閘道及其相關聯的資源提供網路連線。

```azurepowershell-interactive
$backendSubnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24

$agSubnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.2.0/24

New-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $backendSubnetConfig, $agSubnetConfig

New-AzureRmPublicIpAddress `
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

使用 [New-AzureRmApplicationGatewayIPConfiguration](/powershell/module/azurerm.network/new-azurermapplicationgatewayipconfiguration) 將您先前建立的 myAGSubnet 與應用程式閘道產生關聯。 使用 [New-AzureRmApplicationGatewayFrontendIPConfig](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendipconfig) 將 myAGPublicIPAddress 指派至應用程式閘道。 然後，您可以使用 [New-AzureRmApplicationGatewayFrontendPort](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendport) 建立 HTTP 連接埠。

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

需要有接聽程式，才能讓應用程式閘道將流量適當地路由到後端集區。 在本教學課程中，您將建立多個接聽程式。 第一個基本的接聽程式會預期位於根 URL 的流量。 其他接聽程式則預期位於特定 URL 的流量，例如 *http://52.168.55.24:8080/images/* 或 *http://52.168.55.24:8081/video/*。

使用 [New-AzureRmApplicationGatewayHttpListener](/powershell/module/azurerm.network/new-azurermapplicationgatewayhttplistener) 以及您先前建立的前端設定和前端連接埠，以建立名為 *defaultListener* 的接聽程式。 接聽程式需要規則以便知道要針對連入流量使用哪個後端集區。 使用 [New-AzureRmApplicationGatewayRequestRoutingRule](/powershell/module/azurerm.network/new-azurermapplicationgatewayrequestroutingrule) 建立名為 rule1 的基本規則。

```azurepowershell-interactive
$defaultlistener = New-AzureRmApplicationGatewayHttpListener `
  -Name defaultListener `
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

New-AzureRmApplicationGateway `
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

### <a name="add-backend-pools-and-ports"></a>新增後端集區和連接埠

您可以使用 [Add-AzureRmApplicationGatewayBackendAddressPool](/powershell/module/azurerm.network/add-azurermapplicationgatewaybackendaddresspool)，將後端集區新增至應用程式閘道。 在此範例中，會建立 *imagesBackendPool* 和 *videoBackendPool*。 使用 [Add-AzureRmApplicationGatewayFrontendPort](/powershell/module/azurerm.network/add-azurermapplicationgatewayfrontendport) 新增集區的前端連接埠。 然後您要使用 [Set-AzureRmApplicationGateway](/powershell/module/azurerm.network/set-azurermapplicationgateway) 將變更提交至應用程式閘道。

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

Add-AzureRmApplicationGatewayFrontendPort `
  -ApplicationGateway $appgw `
  -Name rport `
  -Port 8081

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

## <a name="add-listeners-and-rules"></a>新增接聽程式和規則

### <a name="add-listeners"></a>新增接聽程式

使用 [Add-AzureRmApplicationGatewayHttpListener](/powershell/module/azurerm.network/add-azurermapplicationgatewayhttplistener)，以新增路由流量時所需且名為 *backendListener* 和 *redirectedListener* 的接聽程式。

```azurepowershell-interactive
$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$backendPort = Get-AzureRmApplicationGatewayFrontendPort `
  -ApplicationGateway $appgw `
  -Name bport

$redirectPort = Get-AzureRmApplicationGatewayFrontendPort `
  -ApplicationGateway $appgw `
  -Name rport

$fipconfig = Get-AzureRmApplicationGatewayFrontendIPConfig `
  -ApplicationGateway $appgw

Add-AzureRmApplicationGatewayHttpListener `
  -ApplicationGateway $appgw `
  -Name backendListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $backendPort

Add-AzureRmApplicationGatewayHttpListener `
  -ApplicationGateway $appgw `
  -Name redirectedListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $redirectPort

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

### <a name="add-the-default-url-path-map"></a>新增預設 URL 路徑對應

URL 路徑對應可確保特定 URL 會路由傳送到特定的後端集區。 您可以使用 [New-AzureRmApplicationGatewayPathRuleConfig](/powershell/module/azurerm.network/new-azurermapplicationgatewaypathruleconfig) 及 [Add-AzureRmApplicationGatewayUrlPathMapConfig](/powershell/module/azurerm.network/add-azurermapplicationgatewayurlpathmapconfig)，以建立名為 *imagePathRule* 和 *videoPathRule* 的 URL 路徑對應。

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

### <a name="add-redirection-configuration"></a>新增重新導向設定

您可以使用 [Add-AzureRmApplicationGatewayRedirectConfiguration](/powershell/module/azurerm.network/add-azurermapplicationgatewayredirectconfiguration)，為接聽程式設定重新導向。

```azurepowershell-interactive
$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$backendListener = Get-AzureRmApplicationGatewayHttpListener `
  -ApplicationGateway $appgw `
  -Name backendListener

$redirectConfig = Add-AzureRmApplicationGatewayRedirectConfiguration `
  -ApplicationGateway $appgw `
  -Name redirectConfig `
  -RedirectType Found `
  -TargetListener $backendListener `
  -IncludePath $true `
  -IncludeQueryString $true

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

### <a name="add-the-redirection-url-path-map"></a>新增重新導向 URL 路徑對應

```azurepowershell-interactive
$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$poolSettings = Get-AzureRmApplicationGatewayBackendHttpSettings `
  -ApplicationGateway $appgw `
  -Name myPoolSettings

$defaultPool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -ApplicationGateway $appgw `
  -Name appGatewayBackendPool

$redirectConfig = Get-AzureRmApplicationGatewayRedirectConfiguration `
  -ApplicationGateway $appgw `
  -Name redirectConfig

$redirectPathRule = New-AzureRmApplicationGatewayPathRuleConfig `
  -Name redirectPathRule `
  -Paths "/images/*" `
  -RedirectConfiguration $redirectConfig

Add-AzureRmApplicationGatewayUrlPathMapConfig `
  -ApplicationGateway $appgw `
  -Name redirectpathmap `
  -PathRules $redirectPathRule `
  -DefaultBackendAddressPool $defaultPool `
  -DefaultBackendHttpSettings $poolSettings

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

### <a name="add-routing-rules"></a>新增路由規則

路由規則會讓 URL 對應與您所建立的接聽程式產生關聯。 您可以使用 [Add-AzureRmApplicationGatewayRequestRoutingRule](/powershell/module/azurerm.network/add-azurermapplicationgatewayrequestroutingrule)，以新增名為 *defaultRule* 和 *redirectedRule* 的規則。


```azurepowershell-interactive
$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$backendlistener = Get-AzureRmApplicationGatewayHttpListener `
  -ApplicationGateway $appgw `
  -Name backendListener

$redirectlistener = Get-AzureRmApplicationGatewayHttpListener `
  -ApplicationGateway $appgw `
  -Name redirectedListener

$urlPathMap = Get-AzureRmApplicationGatewayUrlPathMapConfig `
  -ApplicationGateway $appgw `
  -Name urlpathmap

$redirectPathMap = Get-AzureRmApplicationGatewayUrlPathMapConfig `
  -ApplicationGateway $appgw `
  -Name redirectpathmap

Add-AzureRmApplicationGatewayRequestRoutingRule `
  -ApplicationGateway $appgw `
  -Name defaultRule `
  -RuleType PathBasedRouting `
  -HttpListener $backendlistener `
  -UrlPathMap $urlPathMap

Add-AzureRmApplicationGatewayRequestRoutingRule `
  -ApplicationGateway $appgw `
  -Name redirectedRule `
  -RuleType PathBasedRouting `
  -HttpListener $redirectlistener `
  -UrlPathMap $redirectPathMap

Set-AzureRmApplicationGateway -ApplicationGateway $appgw
```

## <a name="create-virtual-machine-scale-sets"></a>建立虛擬機器擴展集

在此範例中，您要建立三個虛擬機器擴展集，以支援您所建立的三個後端集區。 您所建立的擴展集名為 myvmss1、myvmss2 和 myvmss3。 每個擴展集都會包含兩個您安裝 IIS 的虛擬機器執行個體。 當您設定 IP 設定時，要將擴展集指派給後端集區。

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
    -ImageReferenceVersion latest `
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

您可以使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得應用程式閘道的公用 IP 位址。 將公用 IP 位址複製並貼到您瀏覽器的網址列。 例如，*http://52.168.55.24*、*http://52.168.55.24:8080/images/test.htm*、*http://52.168.55.24:8080/video/test.htm* 或 *http://52.168.55.24:8081/images/test.htm*。

```azurepowershell-interactive
Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![在應用程式閘道中測試基底 URL](./media/tutorial-url-redirect-powershell/application-gateway-iistest.png)

將 URL 變更為 http://&lt;ip-address&gt;:8080/video/test.htm、使用您的 IP 位址取代 &lt;ip-address&gt;，就應該會看到類似下列的範例：

![在應用程式閘道中測試影像 URL](./media/tutorial-url-redirect-powershell/application-gateway-iistest-images.png)

將 URL 變更為 http://&lt;ip-address&gt;:8080/video/test.htm、使用您的 IP 位址取代 &lt;ip-address&gt;，就應該會看到類似下列的範例：

![在應用程式閘道中測試影片 URL](./media/tutorial-url-redirect-powershell/application-gateway-iistest-video.png)

現在，請將 URL 變更為 http://&lt;ip-address&gt;:8081/images/test.htm、使用您的 IP 位址取代 &lt;ip-address&gt;，然後就會看到流量重新導向回位於 http://&lt;ip-address&gt;:8080/images 的影像後端集區。

## <a name="clean-up-resources"></a>清除資源

不再需要時，使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除資源群組、應用程式閘道和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroupAG
```
## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立應用程式閘道
> * 新增接聽程式和路由規則
> * 為後端集區建立虛擬機器擴展集

> [!div class="nextstepaction"]
> [深入了解應用程式閘道的用途](application-gateway-introduction.md)
