---
title: 建立裝載多個網站的應用程式閘道 - Azure PowerShell
description: 了解如何使用 Azure PowerShell 來建立裝載多個網站的應用程式閘道。
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: tutorial
ms.workload: infrastructure-services
ms.date: 7/13/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 7d376ab958a1fb56753e033129ece84bc1c439c2
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39044691"
---
# <a name="create-an-application-gateway-that-hosts-multiple-web-sites-using-azure-powershell"></a>使用 Azure PowerShell 建立裝載多個網站的應用程式閘道

您可以使用 Azure PowerShell，在建立[應用程式閘道](overview.md)時[設定裝載多個網站](multiple-site-overview.md)。 在本教學課程中，您可以使用虛擬機器擴展集定義後端位址集區。 接著，您可以根據擁有的網域來設定接聽程式和規則，確保網路流量會抵達集區中的適當伺服器。 本教學課程假設您擁有多個網域，並使用 *www.contoso.com* 和 *www.fabrikam.com* 的範例。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立應用程式閘道
> * 建立後端接聽程式
> * 建立路由規則
> * 建立包含後端集區的虛擬機器擴展集
> * 在網域中建立 CNAME 記錄

![多站台路由範例](./media/tutorial-multiple-sites-powershell/scenario.png)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組版本 3.6 或更新版本。 若要尋找版本，請執行 ` Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。  

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>建立網路資源

使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 建立子網路設定。 使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 與子網路設定建立虛擬網路。 最後，使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 建立公用 IP 位址。 這些資源可用來為應用程式閘道及其相關聯的資源提供網路連線。

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

### <a name="create-the-ip-configurations-and-frontend-port"></a>建立 IP 設定與前端連接埠

使用 [New-AzureRmApplicationGatewayIPConfiguration](/powershell/module/azurerm.network/new-azurermapplicationgatewayipconfiguration) 讓您先前建立的子網路與應用程式閘道產生關聯。 使用 [New-AzureRmApplicationGatewayFrontendIPConfig](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendipconfig) 將公用 IP 位址指派給應用程式閘道。

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet

$subnet=$vnet.Subnets[0]

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

### <a name="create-the-backend-pools-and-settings"></a>建立後端集區和設定

使用 [New-AzureRmApplicationGatewayBackendAddressPool](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendaddresspool) 為應用程式閘道建立第一個後端位址集區。 使用 [New-AzureRmApplicationGatewayBackendHttpSettings](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendhttpsettings) 設定集區的設定。

```azurepowershell-interactive
$contosoPool = New-AzureRmApplicationGatewayBackendAddressPool `
  -Name contosoPool

$fabrikamPool = New-AzureRmApplicationGatewayBackendAddressPool `
  -Name fabrikamPool

$poolSettings = New-AzureRmApplicationGatewayBackendHttpSettings `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 120
```

### <a name="create-the-listeners-and-rules"></a>建立接聽程式和規則

需要接聽程式才能讓應用程式閘道將流量適當地路由傳送到後端位址集區。 在本教學課程中，您會為兩個網域建立兩個接聽程式。 在此範例中，會為 *www.contoso.com* 和 *www.fabrikam.com* 網域建立接聽程式。

使用 [New-AzureRmApplicationGatewayHttpListener](/powershell/module/azurerm.network/new-azurermapplicationgatewayhttplistener) 以及您先前建立的前端組態和前端連接埠，來建立第一個接聽程式。 接聽程式需要規則以便知道要針對連入流量使用哪個後端集區。 使用 [New-AzureRmApplicationGatewayRequestRoutingRule](/powershell/module/azurerm.network/new-azurermapplicationgatewayrequestroutingrule) 以建立名為 contosoRule 的基本規則。

```azurepowershell-interactive
$contosolistener = New-AzureRmApplicationGatewayHttpListener `
  -Name contosoListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport `
  -HostName "www.contoso.com"

$fabrikamlistener = New-AzureRmApplicationGatewayHttpListener `
  -Name fabrikamListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport `
  -HostName "www.fabrikam.com"

$contosoRule = New-AzureRmApplicationGatewayRequestRoutingRule `
  -Name contosoRule `
  -RuleType Basic `
  -HttpListener $contosoListener `
  -BackendAddressPool $contosoPool `
  -BackendHttpSettings $poolSettings

$fabrikamRule = New-AzureRmApplicationGatewayRequestRoutingRule `
  -Name fabrikamRule `
  -RuleType Basic `
  -HttpListener $fabrikamListener `
  -BackendAddressPool $fabrikamPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>建立應用程式閘道

您已經建立必要的支援資源，接著請使用 [New-AzureRmApplicationGatewaySku](/powershell/module/azurerm.network/new-azurermapplicationgatewaysku) 指定應用程式閘道的參數，然後使用 [New-AzureRmApplicationGateway](/powershell/module/azurerm.network/new-azurermapplicationgateway) 加以建立。

```azurepowershell-interactive
$sku = New-AzureRmApplicationGatewaySku `
  -Name Standard_Medium `
  -Tier Standard `
  -Capacity 2

$appgw = New-AzureRmApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $contosoPool, $fabrikamPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendport `
  -HttpListeners $contosoListener, $fabrikamListener `
  -RequestRoutingRules $contosoRule, $fabrikamRule `
  -Sku $sku
```

## <a name="create-virtual-machine-scale-sets"></a>建立虛擬機器擴展集

在此範例中，您要建立兩個虛擬機器擴展集，以支援您所建立的兩個後端集區。 您所建立的擴展集名為 myvmss1 和 myvmss2。 每個擴展集都會包含兩個您安裝 IIS 的虛擬機器執行個體。 當您設定 IP 設定時，要將擴展集指派給後端集區。

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet

$appgw = Get-AzureRmApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$contosoPool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -Name contosoPool `
  -ApplicationGateway $appgw

$fabrikamPool = Get-AzureRmApplicationGatewayBackendAddressPool `
  -Name fabrikamPool `
  -ApplicationGateway $appgw

for ($i=1; $i -le 2; $i++)
{
  if ($i -eq 1) 
  {
    $poolId = $contosoPool.Id
  }
  if ($i -eq 2)
  {
    $poolId = $fabrikamPool.Id
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

```azurepowershell-interactive
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }

for ($i=1; $i -le 2; $i++)
{
  $vmss = Get-AzureRmVmss `
    -ResourceGroupName myResourceGroupAG `
    -VMScaleSetName myvmss$i

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

## <a name="create-cname-record-in-your-domain"></a>在網域中建立 CNAME 記錄

在以公用 IP 位址建立應用程式閘道之後，您可以取得 DNS 位址並用以在網域中建立 CNAME 記錄。 您可以使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得應用程式閘道的 DNS 位址。 複製 DNSSettings 的 *fqdn* 值，並用來作為所建立 CNAME 記錄的值。 不建議使用 A-records，因為重新啟動應用程式閘道時，可能會變更 VIP。

```azurepowershell-interactive
Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

在瀏覽器的網址列中輸入您的網域名稱。 例如，http://www.contoso.com。

![在應用程式閘道中測試 contoso 網站](./media/tutorial-multiple-sites-powershell/application-gateway-iistest.png)

將位址變更為您的另一個網域，您應該會看到類似下列的範例：

![測試應用程式閘道中的 fabrikam 網站](./media/tutorial-multiple-sites-powershell/application-gateway-iistest2.png)

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
> * 建立後端接聽程式
> * 建立路由規則
> * 建立包含後端集區的虛擬機器擴展集
> * 在網域中建立 CNAME 記錄

> [!div class="nextstepaction"]
> [建立包含 URL 路徑型路由規則的應用程式閘道](./tutorial-url-route-powershell.md)