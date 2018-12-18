---
title: 建立包含 SSL 終止的應用程式閘道 - Azure PowerShell
description: 了解如何使用 Azure PowerShell 建立應用程式閘道，並新增 SSL 終止的憑證。
services: application-gateway
author: vhorne
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: tutorial
ms.workload: infrastructure-services
ms.date: 7/13/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: a11aa119e46a01b964776ca9352da41d1ce2991a
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39040262"
---
# <a name="create-an-application-gateway-with-ssl-termination-using-azure-powershell"></a>使用 Azure PowerShell 建立包含 SSL 終止的應用程式閘道

您可以使用 Azure PowerShell 來建立[應用程式閘道](overview.md)，當中包含使用[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)作為後端伺服器的 [SSL 終止](ssl-overview.md)憑證。 在此範例中，該擴展集包含兩個虛擬機器執行個體，這些執行個體會新增至應用程式閘道的預設後端集區。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立自我簽署憑證
> * 設定網路
> * 建立包含憑證的應用程式閘道
> * 建立包含預設後端集區的虛擬機器擴展集

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

本教學課程需要 Azure PowerShell 模組 3.6 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-self-signed-certificate"></a>建立自我簽署憑證

若要在生產環境中使用，您應該匯入由受信任提供者所簽署的有效憑證。 在此教學課程中，您要使用 [New-selfsignedcertificate](https://docs.microsoft.com/powershell/module/pkiclient/new-selfsignedcertificate) 來建立自我簽署的憑證。 您可以使用 [Export-PfxCertificate](https://docs.microsoft.com/powershell/module/pkiclient/export-pfxcertificate) 搭配系統所傳回的指紋，以將 pfx 檔案從憑證匯出。

```powershell
New-SelfSignedCertificate `
  -certstorelocation cert:\localmachine\my `
  -dnsname www.contoso.com
```

您應該會看到類似這個結果的內容：

```
PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\my

Thumbprint                                Subject
----------                                -------
E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630  CN=www.contoso.com
```

使用指紋來建立 pfx 檔案：

```powershell
$pwd = ConvertTo-SecureString -String "Azure123456!" -Force -AsPlainText

Export-PfxCertificate `
  -cert cert:\localMachine\my\E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630 `
  -FilePath c:\appgwcert.pfx `
  -Password $pwd
```

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立名為 myResourceGroupAG 的 Azure 資源群組。 

```powershell
New-AzureRmResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>建立網路資源

使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 設定名為 myBackendSubnet 和 myAGSubnet 的子網路。 使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 與子網路設定來建立名為 myVNet 的虛擬網路。 最後，使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 建立名為 myAGPublicIPAddress 的公用 IP 位址。 這些資源可用來為應用程式閘道及其相關聯的資源提供網路連線。

```powershell
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

使用 [New-AzureRmApplicationGatewayIPConfiguration](/powershell/module/azurerm.network/new-azurermapplicationgatewayipconfiguration) 將您先前建立的 myAGSubnet 與應用程式閘道產生關聯。 使用 [New-AzureRmApplicationGatewayFrontendIPConfig](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendipconfig) 將 myAGPublicIPAddress 指派至應用程式閘道。

```powershell
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
  -Port 443
```

### <a name="create-the-backend-pool-and-settings"></a>建立後端集區和設定

使用 [New-AzureRmApplicationGatewayBackendAddressPool](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendaddresspool) 為應用程式閘道建立名為 appGatewayBackendPool 的後端集區。 使用 [New-AzureRmApplicationGatewayBackendHttpSettings](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendhttpsettings) 設定後端集區的設定。

```powershell
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

需要接聽程式才能讓應用程式閘道將流量適當地路由到後端集區。 在此範例中，您會建立基本接聽程式，接聽根 URL 的 HTTPS 流量。 

使用 [New-AzureRmApplicationGatewaySslCertificate](/powershell/module/azurerm.network/new-azurermapplicationgatewaysslcertificate) 建立憑證物件，然後使用 [New-AzureRmApplicationGatewayHttpListener](/powershell/module/azurerm.network/new-azurermapplicationgatewayhttplistener) 搭配前端設定、前端連接埠和您先前建立的憑證，建立名為 mydefaultListener 的接聽程式。 接聽程式需要規則才能知道要將哪個後端集區用於連入流量。 使用 [New-AzureRmApplicationGatewayRequestRoutingRule](/powershell/module/azurerm.network/new-azurermapplicationgatewayrequestroutingrule) 建立名為 rule1 的基本規則。

```powershell
$pwd = ConvertTo-SecureString `
  -String "Azure123456!" `
  -Force `
  -AsPlainText

$cert = New-AzureRmApplicationGatewaySslCertificate `
  -Name "appgwcert" `
  -CertificateFile "c:\appgwcert.pfx" `
  -Password $pwd

$defaultlistener = New-AzureRmApplicationGatewayHttpListener `
  -Name mydefaultListener `
  -Protocol Https `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport `
  -SslCertificate $cert

$frontendRule = New-AzureRmApplicationGatewayRequestRoutingRule `
  -Name rule1 `
  -RuleType Basic `
  -HttpListener $defaultlistener `
  -BackendAddressPool $defaultPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway-with-the-certificate"></a>建立包含憑證的應用程式閘道

現在您已建立必要的支援資源，請使用 [New-AzureRmApplicationGatewaySku](/powershell/module/azurerm.network/new-azurermapplicationgatewaysku) 指定名為 myAppGateway 之應用程式閘道的參數，然後使用 [New-AzureRmApplicationGateway](/powershell/module/azurerm.network/new-azurermapplicationgateway) 搭配憑證加以建立。

### <a name="create-the-application-gateway"></a>建立應用程式閘道

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
  -Sku $sku `
  -SslCertificates $cert
```

## <a name="create-a-virtual-machine-scale-set"></a>建立虛擬機器擴展集

在此範例中，您會建立虛擬機器擴展集，以在應用程式閘道中提供後端集區的伺服器。 當您設定 IP 設定時，要將擴展集指派給後端集區。

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

$ipConfig = New-AzureRmVmssIpConfig `
  -Name myVmssIPConfig `
  -SubnetId $vnet.Subnets[1].Id `
  -ApplicationGatewayBackendAddressPoolsId $backendPool.Id

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
  -ComputerNamePrefix myvmss

Add-AzureRmVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name myVmssNetConfig `
  -Primary $true `
  -IPConfiguration $ipConfig

New-AzureRmVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmssConfig
```

### <a name="install-iis"></a>安裝 IIS

```azurepowershell-interactive
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }

$vmss = Get-AzureRmVmss -ResourceGroupName myResourceGroupAG -VMScaleSetName myvmss

Add-AzureRmVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings

Update-AzureRmVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmss
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

您可以使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得應用程式閘道的公用 IP 位址。 將公用 IP 位址複製並貼到您瀏覽器的網址列。

```azurepowershell-interactive
Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![安全警告](./media/tutorial-ssl-powershell/application-gateway-secure.png)

若要在使用自我簽署憑證時接受安全性警告，請依序按一下 [詳細資料] 與 [繼續瀏覽網頁]。 接著會顯示受保護的 IIS 網站，如下列範例所示：

![在應用程式閘道中測試基底 URL](./media/tutorial-ssl-powershell/application-gateway-iistest.png)

## <a name="clean-up-resources"></a>清除資源

不再需要時，使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除資源群組、應用程式閘道和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroupAG
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立自我簽署憑證
> * 設定網路
> * 建立包含憑證的應用程式閘道
> * 建立包含預設後端集區的虛擬機器擴展集

> [!div class="nextstepaction"]
> [建立裝載多個網站的應用程式閘道](./tutorial-multiple-sites-powershell.md)