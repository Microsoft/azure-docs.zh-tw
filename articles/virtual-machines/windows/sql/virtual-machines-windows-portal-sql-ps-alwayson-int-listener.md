---
title: 設定 Always On 可用性群組接聽程式 - Microsoft Azure | Microsoft Docs
description: 使用具有一個或多個 IP 位址的內部負載平衡器，在 Azure Resource Manager 模型上設定可用性群組接聽程式。
services: virtual-machines
documentationcenter: na
author: MikeRayMSFT
manager: craigg
editor: monicar
ms.assetid: 14b39cde-311c-4ddf-98f3-8694e01a7d3b
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/22/2017
ms.author: mikeray
ms.openlocfilehash: b4641c847db817df905f056847a26d003ac25fd1
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43381790"
---
# <a name="configure-one-or-more-always-on-availability-group-listeners---resource-manager"></a>設定一或多個 Always On 可用性群組接聽程式 - Resource Manager
本主題說明如何：

* 使用 PowerShell Cmdlet 為 SQL Server 可用性群組建立內部負載平衡器。
* 將其他 IP 位址新增至多個可用性群組的負載平衡器。 

可用性群組接聽程式是用戶端連接以進行資料庫存取的虛擬網路名稱。 在 Azure 虛擬機器上，負載平衡器會保有接聽程式的 IP 位址。 負載平衡器會將流量路由傳送至在探查連接埠上進行接聽的 SQL Server 執行個體。 通常，可用性群組會使用內部負載平衡器。 Azure 內部負載平衡器可以裝載一或多個 IP 位址。 每個 IP 位址皆使用特定的探查連接埠。 本文件說明如何使用 PowerShell 來建立負載平衡器，或將 IP 位址新增至 SQL Server 可用性群組的現有負載平衡器。 

將多個 IP 位址指派給內部負載平衡器是一項新的 Azure 功能，而且只有在 Resource Manager 模型中才可使用。 若要完成這項工作，您必須在 Resource Manager 模型中的 Azure 虛擬機器上部署 SQL Server 可用性群組。 這兩部 SQL Server 虛擬機器必須屬於相同的可用性設定組。 您可以使用 [Microsoft 範本](virtual-machines-windows-portal-sql-alwayson-availability-groups.md) 在 Azure Resource Manager 中自動建立可用性群組。 此範本會自動為您建立可用性群組，包括內部負載平衡器。 如果您想要的話，也可以[手動設定 Always On 可用性群組](virtual-machines-windows-portal-sql-availability-group-tutorial.md)。

本主題會要求您已經設定可用性群組。  

相關主題包括：

* [在 Azure VM (GUI) 中設定 AlwaysOn 可用性群組](virtual-machines-windows-portal-sql-availability-group-tutorial.md)   
* [使用 Azure Resource Manager 和 PowerShell 來設定 VNet 對 VNet 連線](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

[!INCLUDE [Start your PowerShell session](../../../../includes/sql-vm-powershell.md)]

## <a name="configure-the-windows-firewall"></a>設定 Windows 防火牆
設定 Windows 防火牆以允許 SQL Server 存取。 防火牆規則可允許透過 TCP 連線至 SQL Server 執行個體及接聽程式探查所使用的連接埠。 如需詳細的指示，請參閱[設定用於 Database Engine 存取的 Windows 防火牆](http://msdn.microsoft.com/library/ms175043.aspx#Anchor_1)。 為 SQL Server 連接埠和探查連接埠建立輸入規則。

如果您使用 Azure 網路安全性群組來限制存取，請確定允許規則包含後端 SQL Server VM IP 位址和 AG 接聽程式的負載平衡器浮動 IP 位址，以及叢集核心 IP 位址 (如果適用的話)。

## <a name="example-script-create-an-internal-load-balancer-with-powershell"></a>範例指令碼：使用 PowerShell 來建立內部負載平衡器
> [!NOTE]
> 如果您使用了 [Microsoft 範本](virtual-machines-windows-portal-sql-alwayson-availability-groups.md)來建立可用性群組，則已經建立內部負載平衡器。 
> 
> 

下列 PowerShell 指令碼會建立內部負載平衡器、設定負載平衡規則，以及設定負載平衡器的 IP 位址。 若要執行此指令碼，請開啟 Windows PowerShell ISE，然後將指令碼貼到 [指令碼] 窗格中。 請使用 `Connect-AzureRmAccount` 來登入 PowerShell。 如果您有多個 Azure 訂用帳戶，請使用 `Select-AzureRmSubscription ` 來設定訂用帳戶。 

```powershell
# Connect-AzureRmAccount
# Select-AzureRmSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>

$ResourceGroupName = "<Resource Group Name>" # Resource group name
$VNetName = "<Virtual Network Name>"         # Virtual network name
$SubnetName = "<Subnet Name>"                # Subnet name
$ILBName = "<Load Balancer Name>"            # ILB name
$Location = "<Azure Region>"                 # Azure location
$VMNames = "<VM1>","<VM2>"                   # Virtual machine names

$ILBIP = "<n.n.n.n>"                         # IP address
[int]$ListenerPort = "<nnnn>"                # AG listener port
[int]$ProbePort = "<nnnn>"                   # Probe port

$LBProbeName ="ILBPROBE_$ListenerPort"       # The Load balancer Probe Object Name              
$LBConfigRuleName = "ILBCR_$ListenerPort"    # The Load Balancer Rule Object Name

$FrontEndConfigurationName = "FE_SQLAGILB_1" # Object name for the front-end configuration 
$BackEndConfigurationName ="BE_SQLAGILB_1"   # Object name for the back-end configuration

$VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName 

$Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName 

$FEConfig = New-AzureRMLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.id

$BEConfig = New-AzureRMLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName 

$SQLHealthProbe = New-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -Protocol tcp -Port $ProbePort -IntervalInSeconds 15 -ProbeCount 2

$ILBRule = New-AzureRmLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -Probe $SQLHealthProbe -Protocol tcp -FrontendPort $ListenerPort -BackendPort $ListenerPort -LoadDistribution Default -EnableFloatingIP 

$ILB= New-AzureRmLoadBalancer -Location $Location -Name $ILBName -ResourceGroupName $ResourceGroupName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -LoadBalancingRule $ILBRule -Probe $SQLHealthProbe 

$bepool = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB 

foreach($VMName in $VMNames)
    {
        $VM = Get-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VMName 
        $NICName = ($vm.NetworkProfile.NetworkInterfaces.Id.split('/') | select -last 1)
        $NIC = Get-AzureRmNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName
        $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools = $BEPool
        Set-AzureRmNetworkInterface -NetworkInterface $NIC
        start-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VM.Name 
    }
```

## <a name="Add-IP"></a> 範例指令碼︰使用 PowerShell 將 IP 位址新增至現有的負載平衡器
若要使用多個可用性群組，請將額外的 IP 位址新增至負載平衡器。 每個 IP 位址都需要有自己的負載平衡規則、探查連接埠及前端連接埠。

前端連接埠是應用程式用來連線到 SQL Server 執行個體的連接埠。 不同可用性群組的 IP 位址可以使用相同的前端連接埠。

> [!NOTE]
> 就 SQL Server 可用性群組而言，每個 IP 位址都需要一個特定的探查連接埠。 例如，如果負載平衡器上有一個 IP 位址使用探查連接埠 59999，該負載平衡器上的任何其他 IP 位址就不能使用探查連接埠 59999。

* 如需有關負載平衡器限制的資訊，請參閱[網路限制 - Azure Resource Manager](../../../azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits) 底下的「每個負載平衡器的私人前端 IP」。
* 如需有關可用性群組限制的資訊，請參閱[限制 (可用性群組)](http://msdn.microsoft.com/library/ff878487.aspx#RestrictionsAG)。

下列指令碼會將新的 IP 位址新增至現有的負載平衡器。 ILB 會使用接聽程式連接埠作為負載平衡前端連接埠。 此連接埠可以是 SQL Server 正在接聽的連接埠。 就預設的 SQL Server 執行個體而言，連接埠是 1433。 可用性群組的負載平衡規則需要浮動 IP (伺服器直接回傳)，因此後端連接埠與前端連接埠相同。 請更新您環境的變數。 

```powershell
# Connect-AzureRmAccount
# Select-AzureRmSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>

$ResourceGroupName = "<ResourceGroup>"          # Resource group name
$VNetName = "<VirtualNetwork>"                  # Virtual network name
$SubnetName = "<Subnet>"                        # Subnet name
$ILBName = "<ILBName>"                          # ILB name                      

$ILBIP = "<n.n.n.n>"                            # IP address
[int]$ListenerPort = "<nnnn>"                   # AG listener port
[int]$ProbePort = "<nnnnn>"                     # Probe port 

$ILB = Get-AzureRmLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName 

$count = $ILB.FrontendIpConfigurations.Count+1
$FrontEndConfigurationName ="FE_SQLAGILB_$count"  

$LBProbeName = "ILBPROBE_$count"
$LBConfigrulename = "ILBCR_$count"

$VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName 
$Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

$ILB | Add-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id 

$ILB | Add-AzureRmLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 15  | Set-AzureRmLoadBalancer 

$ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

$FEConfig = get-AzureRMLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB

$SQLHealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

$BEConfig = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $ILB.BackendAddressPools[0].Name -LoadBalancer $ILB 

$ILB | Add-AzureRmLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $SQLHealthProbe -Protocol tcp -FrontendPort  $ListenerPort -BackendPort $ListenerPort -LoadDistribution Default -EnableFloatingIP | Set-AzureRmLoadBalancer   
```

## <a name="configure-the-listener"></a>設定接聽程式

[!INCLUDE [ag-listener-configure](../../../../includes/virtual-machines-ag-listener-configure.md)]

## <a name="set-the-listener-port-in-sql-server-management-studio"></a>在 SQL Server Management Studio 中設定接聽程式連接埠

1. 啟動 SQL Server Management Studio，然後連接到主要複本。

1. 瀏覽至 [AlwaysOn 高可用性] | [可用性群組] | [可用性群組接聽程式]。 

1. 您現在應該會看到在容錯移轉叢集管理員中建立的接聽程式名稱。 以滑鼠右鍵按一下接聽程式名稱，然後按一下 [屬性] 。

1. 在 [連接埠] 方塊中，使用您稍早所用的 $EndpointPort (預設值是 1433) 來指定可用性群組接聽程式的連接埠號碼，然後按一下 [確定]。

## <a name="test-the-connection-to-the-listener"></a>測試接聽程式的連線

若要測試連線︰

1. 透過 RDP 連接到相同虛擬網路中不擁有複本的 SQL Server。 這可以是叢集中的其他 SQL Server。

1. 使用 **sqlcmd** 公用程式來測試連線。 例如，下列指令碼會透過接聽程式搭配 Windows 驗證，建立與主要複本的 **sqlcmd** 連線︰
   
    ```
    sqlmd -S <listenerName> -E
    ```
   
    如果接聽程式使用預設連接埠 (1433) 以外的連接埠，請在連接字串中指定該連接埠。 例如，下列 sqlcmd 命令會連線到位於連接埠 1435 的接聽程式︰ 
   
    ```
    sqlcmd -S <listenerName>,1435 -E
    ```

SQLCMD 連線會自動連線到任何一個裝載主要複本的 SQL Server 執行個體。 

> [!NOTE]
> 確定您指定的連接埠在兩部 SQL Server 在防火牆上開啟。 這兩部伺服器需要您使用的 TCP 連接埠的輸入規則。 如需詳細資訊，請參閱 [新增或編輯防火牆規則](http://technet.microsoft.com/library/cc753558.aspx) 。 
> 
> 

## <a name="guidelines-and-limitations"></a>指導方針和限制
請注意，下列關於 Azure 中使用內部負載平衡器之可用性群組接聽程式的指導方針：

* 使用內部負載平衡器時，您只會從相同的虛擬網路內存取接聽程式。

* 如果您使用 Azure 網路安全性群組來限制存取，請確定允許規則包含後端 SQL Server VM IP 位址和 AG 接聽程式的負載平衡器浮動 IP 位址，以及叢集核心 IP 位址 (如果適用的話)。

## <a name="for-more-information"></a>取得詳細資訊
如需詳細資訊，請參閱[在 Azure VM 中手動設定 Always On 可用性群組](virtual-machines-windows-portal-sql-availability-group-tutorial.md)。

## <a name="powershell-cmdlets"></a>PowerShell Cmdlet
請使用下列 PowerShell Cmdlet 為 Azure 虛擬機器建立內部負載平衡器。

* [New-AzureRmLoadBalancer](http://msdn.microsoft.com/library/mt619450.aspx) 會建立負載平衡器。 
* [New-AzureRMLoadBalancerFrontendIpConfig](http://msdn.microsoft.com/library/mt603510.aspx) 會建立負載平衡器的前端 IP 組態。 
* [New-AzureRmLoadBalancerRuleConfig](http://msdn.microsoft.com/library/mt619391.aspx) 會建立負載平衡器的規則組態。 
* [New-AzureRmLoadBalancerBackendAddressPoolConfig](http://msdn.microsoft.com/library/mt603791.aspx) 會建立負載平衡器的後端位址集區組態。 
* [New-AzureRmLoadBalancerProbeConfig](http://msdn.microsoft.com/library/mt603847.aspx) 會建立負載平衡器的探查組態。
* [Remove-AzureRmLoadBalancer](http://msdn.microsoft.com/library/mt603862.aspx) 會從 Azure 資源群組中移除負載平衡器。
