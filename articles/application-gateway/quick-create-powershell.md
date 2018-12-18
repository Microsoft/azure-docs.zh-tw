---
title: 快速入門 - 使用 Azure 應用程式閘道引導網路流量 - Azure PowerShell | Microsoft Docs
description: 深入了解如何使用 Azure PowerShell 建立 Azure 應用程式閘道，該應用程式閘道會將網路流量導向至後端集區中的虛擬機器。
services: application-gateway
author: vhorne
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.service: application-gateway
ms.devlang: azurepowershell
ms.topic: quickstart
ms.workload: infrastructure-services
ms.date: 01/25/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 95b806eaabaf0ba93b1e8b6823340e82842b0f1c
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38591380"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway---azure-powershell"></a>快速入門：使用 Azure 應用程式閘道引導網路流量 - Azure PowerShell

您可以使用 Azure 應用程式閘道，將接聽程式指派給連接埠、建立規則以及將資源新增至後端集區，來將應用程式網路流量導向至特定資源。

本快速入門示範如何使用 Azure 入口網站，快速建立在其後端集區中具有兩部虛擬機器的應用程式閘道。 然後進行測試以確定它正常運作。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組版本 3.6 或更新版本。 若要尋找版本，請執行 `Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-resource-group"></a>建立資源群組

您必須一律在資源群組中建立資源。 使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>建立網路資源 

您必須為應用程式閘道建立虛擬網路，應用程式閘道才能與其他資源通訊。 這個範例中會建立兩個子網路：一個用於應用程式閘道，另一個用於後端伺服器。 

使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 建立子網路設定。 使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 與子網路設定建立虛擬網路。 最後，使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 建立公用 IP 位址。

```azurepowershell-interactive
$backendSubnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.1.0/24
$agSubnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
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
## <a name="create-backend-servers"></a>建立後端伺服器

在此範例中，您要建立兩個虛擬機器，作為應用程式閘道的後端伺服器。 

您也可以在虛擬機器上安裝 IIS，以確認成功建立應用程式閘道。

### <a name="create-two-virtual-machines"></a>建立兩部虛擬機器

使用 [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) 建立網路介面。 使用 [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) 建立虛擬機器設定。 當您執行下列命令時，系統會提示您輸入認證。 輸入「azureuser」作為使用者名稱，並且輸入「Azure123456!」 作為密碼。 使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立虛擬機器。

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName myResourceGroupAG -Name myVNet
$cred = Get-Credential
for ($i=1; $i -le 2; $i++)
{
  $nic = New-AzureRmNetworkInterface `
    -Name myNic$i `
    -ResourceGroupName myResourceGroupAG `
    -Location EastUS `
    -SubnetId $vnet.Subnets[1].Id
  $vm = New-AzureRmVMConfig `
    -VMName myVM$i `
    -VMSize Standard_DS2
  $vm = Set-AzureRmVMOperatingSystem `
    -VM $vm `
    -Windows `
    -ComputerName myVM$i `
    -Credential $cred
  $vm = Set-AzureRmVMSourceImage `
    -VM $vm `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest
  $vm = Add-AzureRmVMNetworkInterface `
    -VM $vm `
    -Id $nic.Id
  $vm = Set-AzureRmVMBootDiagnostics `
    -VM $vm `
    -Disable
  New-AzureRmVM -ResourceGroupName myResourceGroupAG -Location EastUS -VM $vm
  Set-AzureRmVMExtension `
    -ResourceGroupName myResourceGroupAG `
    -ExtensionName IIS `
    -VMName myVM$i `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
    -Location EastUS
}
```

## <a name="create-an-application-gateway"></a>建立應用程式閘道

### <a name="create-the-ip-configurations-and-frontend-port"></a>建立 IP 設定與前端連接埠

使用 [New-AzureRmApplicationGatewayIPConfiguration](/powershell/module/azurerm.network/new-azurermapplicationgatewayipconfiguration) 建立設定，該設定會讓您先前建立的子網路與應用程式閘道產生關聯。 使用 [New-AzureRmApplicationGatewayFrontendIPConfig](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendipconfig) 建立設定，該設定會將也是在先前建立的公用 IP 位址指派至應用程式閘道。 使用 [New-AzureRmApplicationGatewayFrontendPort](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendport) 指派連接埠 80 以用於存取應用程式閘道。

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName myResourceGroupAG -Name myVNet
$pip = Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress 
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

### <a name="create-the-backend-pool"></a>建立後端集區

使用 [New-AzureRmApplicationGatewayBackendAddressPool](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendaddresspool) 建立應用程式閘道的後端集區。 使用 [New-AzureRmApplicationGatewayBackendHttpSettings](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendhttpsettings) 設定後端集區的設定。

```azurepowershell-interactive
$address1 = Get-AzureRmNetworkInterface -ResourceGroupName myResourceGroupAG -Name myNic1
$address2 = Get-AzureRmNetworkInterface -ResourceGroupName myResourceGroupAG -Name myNic2
$backendPool = New-AzureRmApplicationGatewayBackendAddressPool `
  -Name myAGBackendPool `
  -BackendIPAddresses $address1.ipconfigurations[0].privateipaddress, $address2.ipconfigurations[0].privateipaddress
$poolSettings = New-AzureRmApplicationGatewayBackendHttpSettings `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 120
```

### <a name="create-the-listener-and-add-a-rule"></a>建立接聽程式並新增規則

需要接聽程式才能讓應用程式閘道將流量適當地路由到後端集區。 使用 [New-AzureRmApplicationGatewayHttpListener](/powershell/module/azurerm.network/new-azurermapplicationgatewayhttplistener) 以及您先前建立的前端設定和前端連接埠，來建立接聽程式。 接聽程式需要規則以便知道要針對連入流量使用哪個後端集區。 使用 [New-AzureRmApplicationGatewayRequestRoutingRule](/powershell/module/azurerm.network/new-azurermapplicationgatewayrequestroutingrule) 建立名為 rule1 的規則。

```azurepowershell-interactive
$defaultlistener = New-AzureRmApplicationGatewayHttpListener `
  -Name myAGListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport
$frontendRule = New-AzureRmApplicationGatewayRequestRoutingRule `
  -Name rule1 `
  -RuleType Basic `
  -HttpListener $defaultlistener `
  -BackendAddressPool $backendPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>建立應用程式閘道

您已經建立必要的支援資源，接著請使用 [New-AzureRmApplicationGatewaySku](/powershell/module/azurerm.network/new-azurermapplicationgatewaysku) 指定應用程式閘道的參數，然後使用 [New-AzureRmApplicationGateway](/powershell/module/azurerm.network/new-azurermapplicationgateway) 加以建立。

```azurepowershell-interactive
$sku = New-AzureRmApplicationGatewaySku `
  -Name Standard_Medium `
  -Tier Standard `
  -Capacity 2
New-AzureRmApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $backendPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendport `
  -HttpListeners $defaultlistener `
  -RequestRoutingRules $frontendRule `
  -Sku $sku
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

安裝 IIS 不是建立應用程式閘道的必要條件，但是在本快速入門中，您已安裝 IIS 來確認應用程式閘道是否已成功建立。 使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得應用程式閘道的公用 IP 位址。 將公用 IP 位址複製並貼到您瀏覽器的網址列。

```azurepowershell-interactive
Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![測試應用程式閘道](./media/quick-create-powershell/application-gateway-iistest.png)

當您重新整理瀏覽器時，您應該會看到其他虛擬機器的名稱出現。

## <a name="clean-up-resources"></a>清除資源

首先探索使用應用程式閘道建立的資源，之後若不再需要，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組、應用程式閘道和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroupAG
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Azure PowerShell 以應用程式閘道管理網路流量](./tutorial-manage-web-traffic-powershell.md)

