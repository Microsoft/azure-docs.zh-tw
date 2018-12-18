---
title: 在 Azure 中的多個 IP 組態上進行負載平衡 | Microsoft Docs
description: 在主要和次要 IP 組態間進行負載平衡。
services: load-balancer
documentationcenter: na
author: anavinahar
manager: narayan
editor: na
ms.assetid: 244907cd-b275-4494-aaf7-dcfc4d93edfe
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: annahar
ms.openlocfilehash: 12a978fc85d9502ce484859b436575b67364c9c4
ms.sourcegitcommit: c3d53d8901622f93efcd13a31863161019325216
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/29/2018
ms.locfileid: "30261716"
---
# <a name="load-balancing-on-multiple-ip-configurations-using-powershell"></a>使用 PowerShell 在多個 IP 組態上進行負載平衡

> [!div class="op_single_selector"]
> * [入口網站](load-balancer-multiple-ip.md)
> * [CLI](load-balancer-multiple-ip-cli.md)
> * [PowerShell](load-balancer-multiple-ip-powershell.md)


本文說明如何對次要網路介面 (NIC) 上的多個 IP 位址使用 Azure Load Balancer。 在此案例中，我們有兩部執行 Windows 的 VM，每部各有一個主要和次要 NIC。 每個次要 NIC 都有兩個 IP 組態。 每部 VM 都裝載 contoso.com 和 fabrikam.com 兩個網站。每個網站繫結到次要 NIC 上的其中一個 IP 組態。 我們使用 Azure Load Balancer 來公開兩個前端 IP 位址，每個網站各使用其中一個，以將流量分散給網站的個別 IP 組態。 此案例會在兩個前端以及兩個後端集區 IP 位址使用相同的連接埠號碼。

![LB 案例影像](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

## <a name="steps-to-load-balance-on-multiple-ip-configurations"></a>在多個 IP 組態上進行負載平衡的步驟

請遵循下列步驟來達成本文所述案例︰

1. 安裝 Azure PowerShell。 如需如何安裝最新版 Azure PowerShell、選取訂用帳戶，以及登入帳戶的相關資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。
2. 使用下列設定建立資源群組：

    ```powershell
    $location = "westcentralus".
    $myResourceGroup = "contosofabrikam"
    ```

    如需詳細資訊，請參閱[建立資源群組](../virtual-machines/virtual-machines-windows-ps-create.md?toc=%2fazure%2fload-balancer%2ftoc.json)的步驟 2。

3. [建立可用性設定組](../virtual-machines/windows/tutorial-availability-sets.md?toc=%2fazure%2fload-balancer%2ftoc.json)以容納 VM。 在此案例中，請使用下列命令：

    ```powershell
    New-AzureRmAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset" -Location "West Central US"
    ```

4. 遵循[建立 Windows VM](../virtual-machines/virtual-machines-windows-ps-create.md?toc=%2fazure%2fload-balancer%2ftoc.json)一文中的指示步驟 3 至 5，以準備建立具有單一 NIC 的 VM。 執行步驟 6.1，並使用下列項目，而不是步驟 6.2︰

    ```powershell
    $availset = Get-AzureRmAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset"
    New-AzureRmVMConfig -VMName "VM1" -VMSize "Standard_DS1_v2" -AvailabilitySetId $availset.Id
    ```

    然後完成[建立 Windows VM](../virtual-machines/virtual-machines-windows-ps-create.md?toc=%2fazure%2fload-balancer%2ftoc.json)的步驟 6.3 至 6.8。

5. 在每個 VM 中新增第二個 IP 組態。 遵循[對虛擬機器指派多個 IP 位址](../virtual-network/virtual-network-multiple-ip-addresses-powershell.md#add)一文中的指示。 使用下列組態設定：

    ```powershell
    $NicName = "VM1-NIC2"
    $RgName = "contosofabrikam"
    $NicLocation = "West Central US"
    $IPConfigName4 = "VM1-ipconfig2"
    $Subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $myVnet
    ```

    在進行本教學課程時，並不需要讓次要 IP 組態與公用 IP 建立關聯。 請編輯命令以移除公用 IP 關聯部分。

6. 針對 VM2 再次完成本文的步驟 4 到 6。 在進行這項操作時，請務必將 VM 名稱改為 VM2。 請注意，您不需要為第二個 VM 建立虛擬網路。 視您的使用案例而定，您不一定需要建立新的子網路。

7. 建立兩個公用 IP 位址，並將它們儲存在適當的變數中，如下所示︰

    ```powershell
    $publicIP1 = New-AzureRmPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel contoso
    $publicIP2 = New-AzureRmPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel fabrikam

    $publicIP1 = Get-AzureRmPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam
    $publicIP2 = Get-AzureRmPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam
    ```

8. 建立兩個前端 IP 組態：

    ```powershell
    $frontendIP1 = New-AzureRmLoadBalancerFrontendIpConfig -Name contosofe -PublicIpAddress $publicIP1
    $frontendIP2 = New-AzureRmLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2
    ```

9. 建立後端位址集區、探查和負載平衡規則︰

    ```powershell
    $beaddresspool1 = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name contosopool
    $beaddresspool2 = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name fabrikampool

    $healthProbe = New-AzureRmLoadBalancerProbeConfig -Name HTTP -RequestPath 'index.html' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

    $lbrule1 = New-AzureRmLoadBalancerRuleConfig -Name HTTPc -FrontendIpConfiguration $frontendIP1 -BackendAddressPool $beaddresspool1 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    $lbrule2 = New-AzureRmLoadBalancerRuleConfig -Name HTTPf -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    ```

10. 建立好這些資源後，請建立負載平衡器︰

    ```powershell
    $mylb = New-AzureRmLoadBalancer -ResourceGroupName contosofabrikam -Name mylb -Location 'West Central US' -FrontendIpConfiguration $frontendIP1 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe
    ```

11. 在新建立的負載平衡器中新增第二個後端位址集區和前端 IP 組態︰

    ```powershell
    $mylb = Get-AzureRmLoadBalancer -Name "mylb" -ResourceGroupName $myResourceGroup | Add-AzureRmLoadBalancerBackendAddressPoolConfig -Name fabrikampool | Set-AzureRmLoadBalancer

    $mylb | Add-AzureRmLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2 | Set-AzureRmLoadBalancer
    
    Add-AzureRmLoadBalancerRuleConfig -Name HTTP -LoadBalancer $mylb -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80 | Set-AzureRmLoadBalancer
    ```

12. 下列命令會取得 NIC，然後將每個次要 NIC 的這兩個 IP 組態新增至負載平衡器的後端位址集區︰

    ```powershell
    $nic1 = Get-AzureRmNetworkInterface -Name "VM1-NIC2" -ResourceGroupName "MyResourcegroup";
    $nic2 = Get-AzureRmNetworkInterface -Name "VM2-NIC2" -ResourceGroupName "MyResourcegroup";

    $nic1.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic1.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);
    $nic2.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic2.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);

    $mylb = $mylb | Set-AzureRmLoadBalancer

    $nic1 | Set-AzureRmNetworkInterface
    $nic2 | Set-AzureRmNetworkInterface
    ```

13. 最後，您必須將 DNS 資源記錄設定為指向 Load Balancer 的個別前端 IP 位址。 您可以在 Azure DNS 中裝載網域。 如需搭配使用 Azure DNS 與 Load Balancer 的詳細資訊，請參閱[使用 Azure DNS 搭配其他 Azure 服務](../dns/dns-for-azure-services.md)。

## <a name="next-steps"></a>後續步驟
- 若要深入了解如何在 Azure 中合併負載平衡服務，請參閱[在 Azure 中使用負載平衡服務](../traffic-manager/traffic-manager-load-balancing-azure.md)。
- 若要深入了解如何在 Azure 中使用不同類型的 記錄檔來管理負載平衡器和針對其問題進行疑難排解，請參閱 [Azure Load Balancer 的 Log Analytics](../load-balancer/load-balancer-monitor-log.md)。
