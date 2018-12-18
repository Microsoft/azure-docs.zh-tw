---
title: 快速入門：建立標準負載平衡器 - Azure PowerShell | Microsoft Docs
description: 本快速入門說明如何使用 PowerShell 建立標準負載平衡器
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
tags: azure-resource-manager
Customer intent: I want to create a Standard Load balancer so that I can load balance internet traffic to VMs.
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/22/2018
ms.author: kumud
ms:custom: mvc
ms.openlocfilehash: e197d5306d0df93c243b765ba9154dd8661c6908
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319518"
---
# <a name="get-started"></a>快速入門：使用 Azure PowerShell 建立標準負載平衡器
本快速入門說明如何使用 Azure PowerShell 建立標準負載平衡器。 若要測試負載平衡器，您要部署兩部執行 Windows Server 的虛擬機器 (VM)，並平衡兩部 VM 間 Web 應用程式的負載。 若要深入了解標準負載平衡器，請參閱[什麼是標準負載平衡器](load-balancer-standard-overview.md)。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本文會要求使用 Azure PowerShell 模組版本 5.4.1 或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。 

## <a name="create-a-resource-group"></a>建立資源群組

您必須使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立資源群組，才可建立負載平衡器。 下列範例會在 EastUS 位置建立名為 myResourceGroupLB 的資源群組：

```azurepowershell-interactive
New-AzureRmResourceGroup `
  -ResourceGroupName "myResourceGroupLB" `
  -Location "EastUS"
```
## <a name="create-a-public-ip-address"></a>建立公用 IP 位址
若要存取網際網路上您的應用程式，您需要負載平衡器的公用 IP 位址。 使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 建立公用 IP 位址。 下列範例會在 myResourceGroupLB 資源群組中建立名為 myPublicIP 的公用 IP 位址：

```azurepowershell-interactive
$publicIP = New-AzureRmPublicIpAddress `
-Name "myPublicIP" `
-ResourceGroupName "myResourceGroupLB" `
-Location "EastUS" `
-Sku "Standard" `
-AllocationMethod "Static"
  
```
## <a name="create-standard-load-balancer"></a>建立標準負載平衡器
 在本節中，您會設定負載平衡器的前端 IP 和後端位址集區，然後建立基本負載平衡器。
 
### <a name="create-frontend-ip"></a>建立前端 IP
使用 [New-AzureRmLoadBalancerFrontendIpConfig](/powershell/module/azurerm.network/new-azurermloadbalancerfrontendipconfig) 建立前端 IP。 下列範例會建立名為 myFrontEnd 的前端 IP 組態，並連結 myPublicIP 位址： 

```azurepowershell-interactive
$frontendIP = New-AzureRmLoadBalancerFrontendIpConfig `
  -Name "myFrontEnd" `
  -PublicIpAddress $publicIP
```

### <a name="configure-backend-address-pool"></a>設定後端位址集區

使用 [New-AzureRmLoadBalancerBackendAddressPoolConfig](/powershell/module/azurerm.network/new-azurermloadbalancerbackendaddresspoolconfig) 建立後端位址集區。 在其餘步驟中，VM 會連結至此後端集區。 下列範例會建立名為 myBackEndPool 的後端位址集區：

```azurepowershell-interactive
$backendPool = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name "myBackEndPool"
```
### <a name="create-a-health-probe"></a>建立健康狀態探查
若要讓負載平衡器監視您應用程式的狀態，請使用健康狀態探查。 健康狀態探查會根據 VM 對健康狀態檢查的回應，以動態方式從負載平衡器輪替中新增或移除 VM。 根據預設，在 15 秒的間隔內連續發生兩次失敗後，VM 就會從負載平衡器分配中移除。 您可根據通訊協定或您應用程式的特定健康狀態檢查頁面，建立健康狀態探查。 

下列範例會建立 TCP 探查。 您也可以建立自訂 HTTP 探查，以進行更精細的健康狀態檢查。 使用自訂 HTTP 探查時，您必須建立健康狀態檢查頁面，例如 healthcheck.aspx。 此探查必須對負載平衡器傳回 **HTTP 200 OK** 回應，以將主機保留在輪替中。

若要建立 TCP 健康狀態探查，請使用 [Add-AzureRmLoadBalancerProbeConfig](/powershell/module/azurerm.network/add-azurermloadbalancerprobeconfig)。 下列範例會建立名為 *myHealthProbe* 的健康狀態探查，以在 HTTP 連接埠 80 上監視每部 VM：

```azurepowershell-interactive
$probe = New-AzureRmLoadBalancerProbeConfig `
  -Name "myHealthProbe" `
  -RequestPath healthcheck2.aspx `
  -Protocol http `
  -Port 80 `
  -IntervalInSeconds 16 `
  -ProbeCount 2
  ```

### <a name="create-a-load-balancer-rule"></a>建立負載平衡器規則
負載平衡器規則用來定義如何將流量分散至 VM。 您可定義連入流量的前端 IP 組態及後端 IP 集區來接收流量，以及所需的來源和目的地連接埠。 若要確定只有狀況良好的 VM 可接收流量，您也可定義要使用的健康狀態探查。

使用 [Add-AzureRmLoadBalancerRuleConfig](/powershell/module/azurerm.network/add-azurermloadbalancerruleconfig) 建立負載平衡器規則。 下列範例會建立名為 *myLoadBalancerRule* 的負載平衡器規則，並平衡 *TCP* 連接埠 *80* 上的流量：

```azurepowershell-interactive
$lbrule = New-AzureRmLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe $probe
```

### <a name="create-the-nat-rules"></a>建立 NAT 規則

使用 [Add-AzureRmLoadBalancerRuleConfig](/powershell/module/azurerm.network/new-azurermloadbalancerinboundnatruleconfig) 建立 NAT 規則。 下列範例會建立名為 myLoadBalancerRDP1 和 myLoadBalancerRDP2 的 NAT 規則，以允許透過 4221 和 4222 連接埠與後端伺服器進行 RDP 連線：

```azurepowershell-interactive
$natrule1 = New-AzureRmLoadBalancerInboundNatRuleConfig `
-Name 'myLoadBalancerRDP1' `
-FrontendIpConfiguration $frontendIP `
-Protocol tcp `
-FrontendPort 4221 `
-BackendPort 3389

$natrule2 = New-AzureRmLoadBalancerInboundNatRuleConfig `
-Name 'myLoadBalancerRDP2' `
-FrontendIpConfiguration $frontendIP `
-Protocol tcp `
-FrontendPort 4222 `
-BackendPort 3389
```

### <a name="create-load-balancer"></a>建立負載平衡器

使用 [New-AzureRmLoadBalancer](/powershell/module/azurerm.network/new-azurermloadbalancer) 建立標準負載平衡器。 下列範例會使用您在先前步驟中建立的前端 IP 組態、後端集區、健康狀態探查、負載平衡規則和 NAT 規則，建立名為 myLoadBalancer 的公用基本負載平衡器：

```azurepowershell-interactive
$lb = New-AzureRmLoadBalancer `
-ResourceGroupName 'myResourceGroupLB' `
-Name 'MyLoadBalancer' `
-Location 'eastus' `
-FrontendIpConfiguration $frontendIP `
-BackendAddressPool $backendPool `
-Probe $probe `
-LoadBalancingRule $lbrule `
-InboundNatRule $natrule1,$natrule2 `
-sku Standard
```

## <a name="create-network-resources"></a>建立網路資源
您必須先建立支援的網路資源 - 虛擬網路和虛擬 NIC，才可部署一些 VM 及測試您的平衡器。 

### <a name="create-a-virtual-network"></a>建立虛擬網路
使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 建立虛擬網路。 下列範例會建立名為 myVnet 的虛擬網路和 mySubnet：

```azurepowershell-interactive
# Create subnet config
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.2.0/24

# Create the virtual network
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName "myResourceGroupLB" `
  -Location "EastUS" `
  -Name "myVnet" `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnetConfig
```
### <a name="create-network-security-group"></a>建立網路安全性群組
建立網路安全性群組，以定義虛擬網路的輸入連線。

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>建立連接埠 3389 的網路安全性群組規則
使用 [New-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/new-azurermnetworksecurityruleconfig) 建立網路安全性群組規則，以允許透過連接埠 3389 的 RDP 連線。

```azurepowershell-interactive

$rule1 = New-AzureRmNetworkSecurityRuleConfig `
-Name 'myNetworkSecurityGroupRuleRDP' `
-Description 'Allow RDP' `
-Access Allow `
-Protocol Tcp `
-Direction Inbound `
-Priority 1000 `
-SourceAddressPrefix Internet `
-SourcePortRange * `
-DestinationAddressPrefix * `
-DestinationPortRange 3389
```

#### <a name="create-a-network-security-group-rule-for-port-80"></a>建立連接埠 80 的網路安全性群組規則
使用 [New-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/new-azurermnetworksecurityruleconfig) 建立網路安全性群組規則，以允許透過連接埠 80 的輸入連線。

```azurepowershell-interactive
$rule2 = New-AzureRmNetworkSecurityRuleConfig `
-Name 'myNetworkSecurityGroupRuleHTTP' `
-Description 'Allow HTTP' `
-Access Allow `
-Protocol Tcp `
-Direction Inbound `
-Priority 2000 `
-SourceAddressPrefix Internet `
-SourcePortRange * `
-DestinationAddressPrefix * `
-DestinationPortRange 80
```
#### <a name="create-a-network-security-group"></a>建立網路安全性群組

使用 [New-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/new-azurermnetworksecuritygroup) 建立網路安全性群組。

```azurepowershell-interactive
$nsg = New-AzureRmNetworkSecurityGroup`
-ResourceGroupName 'myResourceGroupLB' `
-Location 'EastUS' `
-Name 'myNetworkSecurityGroup'`
-SecurityRules $rule1,$rule2
```

### <a name="create-nics"></a>建立 NIC
使用 [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) 建立虛擬 NIC。 下列範例會建立兩個虛擬 NIC。 (您在下列步驟中針對應用程式建立的每部 VM 都有一個虛擬 NIC)。 您可以隨時建立其他虛擬 NIC 和 VM，並將它們新增至負載平衡器︰

```azurepowershell-interactive
# Create NIC for VM1
$nicVM1 = New-AzureRmNetworkInterface `
-ResourceGroupName 'myResourceGroupLB' `
-Location 'EastUS' `
-Name 'MyNic1' `
-LoadBalancerBackendAddressPool $backendPool `
-NetworkSecurityGroup $nsg `
-LoadBalancerInboundNatRule $natrule1 `
-Subnet $vnet.Subnets[0]

# Create NIC for VM2
$nicVM2 = New-AzureRmNetworkInterface `
-ResourceGroupName 'myResourceGroupLB' `
-Location 'EastUS' `
-Name 'MyNic2' `
-LoadBalancerBackendAddressPool $backendPool `
-NetworkSecurityGroup $nsg `
-LoadBalancerInboundNatRule $natrule2 `
-Subnet $vnet.Subnets[0]

```

### <a name="create-virtual-machines"></a>建立虛擬機器
若要改善您應用程式的高可用性，請將 VM 放在可用性設定組中。

使用 [New-AzureRmAvailabilitySet](/powershell/module/azurerm.compute/new-azurermavailabilityset) 建立可用性設定組。 下列範例會建立名為 myAvailabilitySet 的可用性設定組：

```azurepowershell-interactive
$availabilitySet = New-AzureRmAvailabilitySet `
  -ResourceGroupName "myResourceGroupLB" `
  -Name "myAvailabilitySet" `
  -Location "EastUS" `
  -Sku aligned `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2
```

使用 [Get-credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 來設定 VM 的系統管理員使用者名稱和密碼：

```azurepowershell-interactive
$cred = Get-Credential
```

現在您可以使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM。 下列範例會建立兩個 VM 及必要的虛擬網路元件 (如果尚未存在)︰

```azurepowershell-interactive
for ($i=1; $i -le 2; $i++)
{
    New-AzureRmVm `
        -ResourceGroupName "myResourceGroupLB" `
        -Name "myVM$i" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -OpenPorts 80 `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred `
        -AsJob
}
```

`-AsJob` 參數會以背景工作建立 VM，因此會傳回 PowerShell 提示。 您可以使用 `Job` Cmdlet 檢視背景作業的詳細資料。 建立及設定這兩部 VM 需要幾分鐘的時間。

### <a name="install-iis-with-custom-web-page"></a>使用自訂網頁安裝 IIS
 
使用自訂網頁在兩部後端 VM 上安裝 IIS，如下所示：

1. 取得負載平衡器的公用 IP 位址。 使用 `Get-AzureRmPublicIPAdress` 取得負載平衡器的公用 IP 位址。

  ```azurepowershell-interactive
    Get-AzureRmPublicIPAddress `
    -ResourceGroupName "myResourceGroupLB" `
    -Name "myPublicIP" | select IpAddress
  ```
2. 使用您從前一個步驟取得的公用 IP 位址，建立 VM1 的遠端桌面連線。 

  ```azurepowershell-interactive

      mstsc /v:PublicIpAddress:4221  
  
  ```
3. 輸入 VM1 的認證以啟動 RDP 工作階段。
4. 啟動 VM1 上的 Windows PowerShell 上，並使用下列命令來安裝 IIS 伺服器及更新預設 htm 檔。
    ```azurepowershell-interactive
    # Install IIS
      Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from host" + $env:computername)
    ```
5. 關閉與 myVM1 的 RDP 連線。
6. 執行 `mstsc /v:PublicIpAddress:4222` 命令，並針對 VM2 重複步驟 4，以建立與 myVM2 的 RDP 連線。

## <a name="test-load-balancer"></a>測試負載平衡器
使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得負載平衡器的公用 IP 位址。 下列範例會取得稍早建立的 myPublicIP IP 位址︰

```azurepowershell-interactive
Get-AzureRmPublicIPAddress `
  -ResourceGroupName "myResourceGroupLB" `
  -Name "myPublicIP" | select IpAddress
```

接著，您可以在 Web 瀏覽器中輸入公用 IP 位址。 網站隨即顯示，包括負載平衡器分散流量之 VM 的主機名稱，如下列範例所示：

![測試負載平衡器](media/quickstart-create-basic-load-balancer-powershell/load-balancer-test.png)

若要查看負載平衡器如何將流量分散於執行您應用程式的這兩部 VM，您可以強制重新整理您的網頁瀏覽器。 

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組、VM 和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroupLB
```

## <a name="next-steps"></a>後續步驟

本快速入門中，您已建立基本 Load Balancer、並將 VM 加以連結、設定負載平衡器流量規則、健康情況探查，接著測試負載平衡器。 若要深入了解 Azure Load Balancer，請繼續 Azure Load Balancer 的教學課程。

> [!div class="nextstepaction"]
> [Azure Load Balancer 教學課程](tutorial-load-balancer-basic-internal-portal.md)
