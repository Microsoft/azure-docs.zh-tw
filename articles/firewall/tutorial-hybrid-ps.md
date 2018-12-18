---
title: 使用 Azure PowerShell 在混合式網路中部署及設定 Azure 防火牆
description: 在本教學課程中，您將了解如何使用 Azure 入口網站部署及設定 Azure 防火牆。
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 10/2/2018
ms.author: victorh
ms.openlocfilehash: 27221ac4b23f52dd6976a959e6e5529eb0cc89fa
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48856066"
---
# <a name="tutorial-deploy-and-configure-azure-firewall-in-a-hybrid-network-using-azure-powershell"></a>教學課程：使用 Azure PowerShell 在混合式網路中部署及設定 Azure 防火牆


在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定網路環境
> * 設定及部署防火牆
> * 建立路由
> * 建立虛擬機器
> * 測試防火牆

在本教學課程中，您會建立三個 VNet：
- **VNet-Hub** - 防火牆位於此 VNet 中。
- **VNet-Spoke** - 輪輻 VNet 代表位於 Azure 的工作負載。
- **VNet-Onprem** - 內部部署 VNet 代表內部部署網路。 在實際部署中，它可經由 VPN 或 Express Route 連線來連線。 為了簡單起見，本教學課程使用 VPN 閘道連線，而位於 Azure 的 VNet 用來代表內部部署網路。

![混合式網路中的防火牆](media/tutorial-hybrid-ps/hybrid-network-firewall.png)

## <a name="key-requirements"></a>重要需求

要讓此案例正常運作有三項重要需求：

- 輪輻子網路上使用者定義的路由會指向 Azure 防火牆 IP 位址，作為預設閘道。 此路由表上的 BGP 路由傳播必須 [已停用]。
- 中樞閘道子網路上使用者定義的路由必須指向防火牆 IP 位址，作為輪輻網路的下一個躍點。
- Azure 防火牆子網路不需要任何使用者定義的路由，因為它從 BGP 得知路由。
- 請務必在將 VNet-Hub 對等互連至 VNet-Spoke 時設定 **AllowGatewayTransit**，以及在將 VNet-Spoke 對等互連至 VNet-Hub 時設定 **UseRemoteGateways**。

請參閱本教學課程中的[建立路由](#create-routes)一節，了解如何建立這些路由。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

## <a name="declare-the-variables"></a>宣告變數
下列範例會使用本教學課程中的值來宣告變數。 在大部分的情況下，您應將值取代為您自己的值。 不過，若您執行這些步驟是為了熟悉此類型的設定，則可以使用這些變數。 視需要修改變數，然後將其複製並貼到您的 PowerShell 主控台中。

```azurepowershell
$RG1 = "FW-Hybrid-Test"
$Location1 = "East US"

# Variables for the firewall hub VNet

$VNetnameHub = "VNet-hub"
$SNnameHub = "AzureFirewallSubnet"
$VNetHubPrefix = "10.5.0.0/16"
$SNHubPrefix = "10.5.0.0/24"
$SNGWHubPrefix = "10.5.1.0/24"
$GWHubName = "GW-hub"
$GWHubpipName = "VNet-hub-GW-pip"
$GWIPconfNameHub = "GW-ipconf-hub"
$ConnectionNameHub = "hub-to-Onprem"

# Variables for the spoke VNet

$VnetNameSpoke = "VNet-Spoke"
$SNnameSpoke = "SN-Workload"
$VNetSpokePrefix = "10.6.0.0/16"
$SNSpokePrefix = "10.6.0.0/24"
$SNSpokeGWPrefix = "10.6.1.0/24"

# Variables for the OnPrem VNet

$VNetnameOnprem = "Vnet-Onprem"
$SNNameOnprem = "SN-Corp"
$VNetOnpremPrefix = "192.168.0.0/16"
$SNOnpremPrefix = "192.168.1.0/24"
$SNGWOnpremPrefix = "192.168.2.0/24"
$GWOnpremName = "GW-Onprem"
$GWIPconfNameOnprem = "GW-ipconf-Onprem"
$ConnectionNameOnprem = "Onprem-to-hub"
$GWOnprempipName = "VNet-Onprem-GW-pip"

$SNnameGW = "GatewaySubnet"
```

## <a name="create-a-resource-group"></a>建立資源群組
建立資源群組，以包含本教學課程中需要的所有資源：

```azurepowershell
  New-AzureRmResourceGroup -Name $RG1 -Location $Location1
  ```

## <a name="create-and-configure-the-firewall-hub-vnet"></a>建立及設定防火牆中樞 Vnet

定義要包含在 VNet 中的子網路：

```azurepowershell
$FWsub = New-AzureRmVirtualNetworkSubnetConfig -Name $SNnameHub -AddressPrefix $SNHubPrefix
$GWsub = New-AzureRmVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNGWHubPrefix
```

現在，建立防火牆中樞 VNet：

```azurepowershell
$VNetHub = New-AzureRmVirtualNetwork -Name $VNetnameHub -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetHubPrefix -Subnet $FWsub,$GWsub
```
要求一個公用 IP 位址，以配置給您將建立給 VNet 使用的 VPN 閘道。 請注意，AllocationMethod 為 [動態]。 您無法指定想要使用的 IP 位址。 該 IP 位址會以動態方式配置給您的 VPN 閘道。 

  ```azurepowershell
  $gwpip1 = New-AzureRmPublicIpAddress -Name $GWHubpipName -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
```
## <a name="create-and-configure-the-spoke-vnet"></a>建立及設定輪輻 Vnet

定義要包含在輪輻 VNet 中的子網路：

```azurepowershell
$Spokesub = New-AzureRmVirtualNetworkSubnetConfig -Name $SNnameSpoke -AddressPrefix $SNSpokePrefix
$GWsubSpoke = New-AzureRmVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNSpokeGWPrefix
```

建立輪輻 VNet：

```azurepowershell
$VNetSpoke = New-AzureRmVirtualNetwork -Name $VnetNameSpoke -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetSpokePrefix -Subnet $Spokesub,$GWsubSpoke
```

## <a name="create-and-configure-the-onprem-vnet"></a>建立及設定內部部署 VNet

定義要包含在 VNet 中的子網路：

```azurepowershell
$Onpremsub = New-AzureRmVirtualNetworkSubnetConfig -Name $SNNameOnprem -AddressPrefix $SNOnpremPrefix
$GWOnpremsub = New-AzureRmVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNGWOnpremPrefix
```

現在建立內部部署 VNet：

```azurepowershell
$VNetOnprem = New-AzureRmVirtualNetwork -Name $VNetnameOnprem -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetOnpremPrefix -Subnet $Onpremsub,$GWOnpremsub
```
要求一個公用 IP 位址，以配置給您將建立給 VNet 使用的閘道。 請注意，AllocationMethod 為 [動態]。 您無法指定想要使用的 IP 位址。 該 IP 位址會以動態方式配置給您的閘道。 

  ```azurepowershell
  $gwOnprempip = New-AzureRmPublicIpAddress -Name $GWOnprempipName -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
```

## <a name="configure-and-deploy-the-firewall"></a>設定及部署防火牆

現在將防火牆部署到中樞 VNet 中。

```azurepowershell
# Get a Public IP for the firewall
$FWpip = New-AzureRmPublicIpAddress -Name "fw-pip" -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Static -Sku Standard
# Create the firewall
$Azfw = New-AzureRmFirewall -Name AzFW01 -ResourceGroupName $RG1 -Location $Location1 -VirtualNetworkName $VNetnameHub -PublicIpName fw-pip

#Save the firewall private IP address for future use

$AzfwPrivateIP = $Azfw.IpConfigurations.privateipaddress
$AzfwPrivateIP

```

### <a name="configure-network-rules"></a>設定網路規則

<!--- $Rule2 = New-AzureRmFirewallNetworkRule -Name "AllowPing" -Protocol ICMP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort *--->

```azurepowershell
$Rule1 = New-AzureRmFirewallNetworkRule -Name "AllowWeb" -Protocol TCP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort 80

$Rule3 = New-AzureRmFirewallNetworkRule -Name "AllowRDP" -Protocol TCP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort 3389

$NetRuleCollection = New-AzureRmFirewallNetworkRuleCollection -Name RCNet01 -Priority 100 `
   -Rule $Rule1,$Rule2,$Rule3 -ActionType "Allow"
$Azfw.NetworkRuleCollections = $NetRuleCollection
Set-AzureRmFirewall -AzureFirewall $Azfw
```
### <a name="configure-an-application-rule"></a>設定應用程式規則

```azurepowershell
$Rule4 = New-AzureRmFirewallApplicationRule -Name "AllowBing" -Protocol "Http:80","Https:443" `
   -SourceAddress $SNOnpremPrefix -TargetFqdn "bing.com"

$AppRuleCollection = New-AzureRmFirewallApplicationRuleCollection -Name RCApp01 -Priority 100 `
   -Rule $Rule4 -ActionType "Allow"
$Azfw.ApplicationRuleCollections = $AppRuleCollection
Set-AzureRmFirewall -AzureFirewall $Azfw

```

## <a name="create-and-connect-the-vpn-gateways"></a>建立及連線 VPN 閘道

中樞和內部部署 VNet 都是透過 VPN 閘道連線的。

### <a name="create-a-vpn-gateway-for-the-hub-vnet"></a>建立中樞 VNet 的 VPN 閘道

建立 VPN 閘道組態。 VPN 閘道器組態定義要使用的子網路和公用 IP 位址。

  ```azurepowershell
  $vnet1 = Get-AzureRmVirtualNetwork -Name $VNetnameHub -ResourceGroupName $RG1
  $subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
  $gwipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfNameHub `
  -Subnet $subnet1 -PublicIpAddress $gwpip1
  ```

現在建立中樞 VNet 的 VPN 閘道。 VNet 對 VNet 組態需要 RouteBased VpnType。 建立 VPN 閘道通常可能需要 45 分鐘或更久，視選取的 VPN 閘道 SKU 而定。

```azurepowershell
New-AzureRmVirtualNetworkGateway -Name $GWHubName -ResourceGroupName $RG1 `
-Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
-VpnType RouteBased -GatewaySku basic
```

### <a name="create-a-vpn-gateway-for-the-onprem-vnet"></a>建立內部部署 VNet 的 VPN 閘道

建立 VPN 閘道組態。 VPN 閘道器組態定義要使用的子網路和公用 IP 位址。

  ```azurepowershell
$vnet2 = Get-AzureRmVirtualNetwork -Name $VNetnameOnprem -ResourceGroupName $RG1
$subnet2 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gwipconf2 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfNameOnprem `
  -Subnet $subnet2 -PublicIpAddress $gwOnprempip
  ```

現在建立內部部署 VNet 的 VPN 閘道。 VNet 對 VNet 組態需要 RouteBased VpnType。 建立 VPN 閘道通常可能需要 45 分鐘或更久，視選取的 VPN 閘道 SKU 而定。

```azurepowershell
New-AzureRmVirtualNetworkGateway -Name $GWOnpremName -ResourceGroupName $RG1 `
-Location $Location1 -IpConfigurations $gwipconf2 -GatewayType Vpn `
-VpnType RouteBased -GatewaySku basic
```
### <a name="create-the-vpn-connections"></a>建立 VPN 連線
您現在可以建立中樞與內部部署閘道之間的 VPN 連線

#### <a name="get-the-vpn-gateways"></a>取得 VPN 閘道
```azurepowershell
$vnetHubgw = Get-AzureRmVirtualNetworkGateway -Name $GWHubName -ResourceGroupName $RG1
$vnetOnpremgw = Get-AzureRmVirtualNetworkGateway -Name $GWOnpremName -ResourceGroupName $RG1
```

#### <a name="create-the-connections"></a>建立連線

在此步驟中，您會建立從中樞 VNet 到內部部署 VNet 的連線。 您會看到範例使用共用金鑰。 您可以使用自己的值，作為共用金鑰。 但請務必確認該共用金鑰必須適用於這兩個連線。 建立連線可能需要一段時間才能完成。

```azurepowershell
New-AzureRmVirtualNetworkGatewayConnection -Name $ConnectionNameHub -ResourceGroupName $RG1 `
-VirtualNetworkGateway1 $vnetHubgw -VirtualNetworkGateway2 $vnetOnpremgw -Location $Location1 `
-ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
```
建立內部部署到中樞 VNet 的連線。 此步驟類似前一個步驟，只不過您是建立 Vnet-Onprem 到 VNet-hub 的連線。 請確認共用的金鑰相符。 稍候幾分鐘就會建立連線。

  ```azurepowershell
  New-AzureRmVirtualNetworkGatewayConnection -Name $ConnectionNameOnprem -ResourceGroupName $RG1 `
  -VirtualNetworkGateway1 $vnetOnpremgw -VirtualNetworkGateway2 $vnetHubgw -Location $Location1 `
  -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```
#### <a name="verify-the-connection"></a>驗證連線
 
您可以使用 Get-AzureRmVirtualNetworkGatewayConnection Cmdlet，並在搭配或不搭配 -Debug 的情況下驗證連線是否成功。 請使用下列 Cmdlet 範例，並將值設定為與您狀況相符的值。 出現提示時，請選取 **A** 以執行 [全部]。 在此範例中，-Name 是指您想要測試的連線名稱。

```azurepowershell
Get-AzureRmVirtualNetworkGatewayConnection -Name $ConnectionNameHub -ResourceGroupName $RG1
```

完成 Cmdlet 之後，請檢視值。 在下列範例中，連線狀態會顯示為 [已連線]，且您可以看見輸入和輸出位元組。

```
"connectionStatus": "Connected",
"ingressBytesTransferred": 33509044,
"egressBytesTransferred": 4142431
```



## <a name="peer-the-hub-and-spoke-vnets"></a>對等互連中樞與輪輻 VNet

現在對等互連輪輻與中樞 VNet。

```azurepowershell
# Peer hub to spoke
Add-AzureRmVirtualNetworkPeering -Name HubtoSpoke -VirtualNetwork $VNetHub -RemoteVirtualNetworkId $VNetSpoke.Id -AllowGatewayTransit

# Peer spoke to hub
Add-AzureRmVirtualNetworkPeering -Name SpoketoHub -VirtualNetwork $VNetSpoke -RemoteVirtualNetworkId $VNetHub.Id -AllowForwardedTraffic -UseRemoteGateways
```
## <a name="create-routes"></a>建立路由

接下來，建立幾個路由： 
- 透過防火牆 IP 位址，從中樞閘道子網路到輪輻子網路的路由
- 透過防火牆 IP 位址，從輪輻子網路開始的預設路由

> [!NOTE]
> Azure 防火牆會使用 BGP 學習您的內部部署網路。 其中可能包括會將網際網路流量路由回內部部署網路的預設路由。 如果您想要讓網際網路流量直接從防火牆傳送到網際網路，請在 AzureFirewallSubnet 上新增使用者定義的預設路由 (0.0.0.0/0)，並以**網際網路**作為下一個躍點類型。 您以內部部署為目的地的流量仍會使用從 BGP 學習到且更明確的路由，透過 VPN/ExpressRoute 閘道強制輸送。

```azurepowershell
#Create a route table
$routeTableHubSpoke = New-AzureRmRouteTable `
  -Name 'UDR-Hub-Spoke' `
  -ResourceGroupName $RG1 `
  -location $Location1

#Create a route
Get-AzureRmRouteTable `
  -ResourceGroupName $RG1 `
  -Name UDR-Hub-Spoke `
  | Add-AzureRmRouteConfig `
  -Name "ToSpoke" `
  -AddressPrefix $VNetSpokePrefix `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzureRmRouteTable

#Associate the route table to the subnet

Set-AzureRmVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNetHub `
  -Name $SNnameGW `
  -AddressPrefix $SNGWHubPrefix `
  -RouteTable $routeTableHubSpoke | `
Set-AzureRmVirtualNetwork

#Now create the default route

#Create a table, with BGP route propagation disabled
$routeTableSpokeDG = New-AzureRmRouteTable `
  -Name 'UDR-DG' `
  -ResourceGroupName $RG1 `
  -location $Location1 `
  -DisableBgpRoutePropagation

#Create a route
Get-AzureRmRouteTable `
  -ResourceGroupName $RG1 `
  -Name UDR-DG `
  | Add-AzureRmRouteConfig `
  -Name "ToSpoke" `
  -AddressPrefix 0.0.0.0/0 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzureRmRouteTable

#Associate the route table to the subnet

Set-AzureRmVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNetSpoke `
  -Name $SNnameSpoke `
  -AddressPrefix $SNSpokePrefix `
  -RouteTable $routeTableSpokeDG | `
Set-AzureRmVirtualNetwork
```
## <a name="create-virtual-machines"></a>建立虛擬機器

現在建立輪輻工作負載和內部部署虛擬機器，並將它們放在適當的子網路中。

### <a name="create-the-workload-virtual-machine"></a>建立工作負載虛擬機器
在輪輻 VNet 中建立虛擬機器，該虛擬機器會執行 IIS、沒有公用 IP 位址，而且允許 Ping 傳入。
出現提示時，請輸入虛擬機器的使用者名稱和密碼。

```azurepowershell
# Create an inbound network security group rule for port 3389
$nsgRuleRDP = New-AzureRmNetworkSecurityRuleConfig -Name Allow-RDP  -Protocol Tcp `
  -Direction Inbound -Priority 200 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix $SNSpokePrefix -DestinationPortRange 3389 -Access Allow
$nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig -Name Allow-web  -Protocol Tcp `
  -Direction Inbound -Priority 202 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix $SNSpokePrefix -DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $RG1 -Location $Location1 -Name NSG-Spoke02 -SecurityRules $nsgRuleRDP,$nsgRuleWeb

#Create the NIC
$NIC = New-AzureRmNetworkInterface -Name spoke-01 -ResourceGroupName $RG1 -Location $Location1 -SubnetId $VnetSpoke.Subnets[0].Id -NetworkSecurityGroupId $nsg.Id

#Define the virtual machine
$VirtualMachine = New-AzureRmVMConfig -VMName VM-Spoke-01 -VMSize "Standard_DS2"
$VirtualMachine = Set-AzureRmVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName Spoke-01 -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzureRmVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzureRmVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2016-Datacenter' -Version latest

#Create the virtual machine
New-AzureRmVM -ResourceGroupName $RG1 -Location $Location1 -VM $VirtualMachine -Verbose

#Install IIS on the VM
Set-AzureRmVMExtension `
    -ResourceGroupName $RG1 `
    -ExtensionName IIS `
    -VMName VM-Spoke-01 `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server"}' `
    -Location $Location1
```

<!---#Create a host firewall rule to allow ping in
Set-AzureRmVMExtension `
    -ResourceGroupName $RG1 `
    -ExtensionName IIS `
    -VMName VM-Spoke-01 `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4"}' `
    -Location $Location1--->


### <a name="create-the-onprem-virtual-machine"></a>建立內部部署虛擬機器
這是一部簡單的虛擬機器，您可以使用遠端桌面連線到公用 IP 位址。 從該處，您可以透過防火牆接著連線到內部部署伺服器。 出現提示時，請輸入虛擬機器的使用者名稱和密碼。
```azurepowershell
New-AzureRmVm `
    -ResourceGroupName $RG1 `
    -Name "VM-Onprem" `
    -Location $Location1 `
    -VirtualNetworkName $VNetnameOnprem `
    -SubnetName $SNNameOnprem `
    -OpenPorts 3389 `
    -Size "Standard_DS2"
```

## <a name="test-the-firewall"></a>測試防火牆
首先，取得並記下 **VM-spoke-01** 虛擬機器的私人 IP 位址。

```azurepowershell
$NIC.IpConfigurations.privateipaddress
```

1. 從 Azure 入口網站，連線到 **VM-Onprem** 虛擬機器。
<!---2. Open a Windows PowerShell command prompt on **VM-Onprem**, and ping the private IP for **VM-spoke-01**.

   You should get a reply.--->
2. 在 **VM-Onprem** 上開啟網頁瀏覽器，並瀏覽至 http://\<VM-spoke-01 private IP\>

   您應該會看到 Internet Information Services 預設頁面。

3. 從 **VM-Onprem**，針對位於私人 IP 位址的 **VM-spoke-01** 開啟遠端桌面。

   您的連線應會成功，而且應該能夠使用您所選的使用者名稱和密碼登入。

因此，現在您已確認防火牆規則正在運作：

<!---- You can ping the server on the spoke VNet.--->
- 您可以瀏覽輪輻 VNet 上的網頁伺服器。
- 您可以使用 RDP 連線到輪輻 VNet 上的伺服器。

接下來，將防火牆網路規則集合動作變更為 [拒絕]，確認防火牆規則會如預期般運作。 請執行下列指令碼，將規則集合動作變更為 [拒絕]。

```azurepowershell
$rcNet = $azfw.GetNetworkRuleCollectionByName("RCNet01")
$rcNet.action.type = "Deny"

$rcApp = $azfw.GetApplicationRuleCollectionByName("RCApp01")
$rcApp.action.type = "Deny"

Set-AzureRmFirewall -AzureFirewall $azfw
```
現在再次執行測試。 這次所有測試應該都會失敗。 在測試已變更的規則之前，請關閉任何現有的遠端桌面。

## <a name="clean-up-resources"></a>清除資源

您可以保留防火牆資源供下一個教學課程使用，若不再需要，則可刪除 **FW-Hybrid-Test** 資源群組來刪除所有防火牆相關資源。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定網路環境
> * 設定及部署防火牆
> * 建立路由
> * 建立虛擬機器
> * 測試防火牆

接下來，您可以監視 Azure 防火牆記錄。

> [!div class="nextstepaction"]
> [教學課程：監視 Azure 防火牆記錄](./tutorial-diagnostics.md)
