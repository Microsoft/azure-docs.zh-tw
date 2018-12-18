---
title: 在 Azure 中建立 SAP 多 SID 組態 | Microsoft Docs
description: Windows 虛擬機器上的 SAP NetWeaver 多 SID 組態的高可用性指南
services: virtual-machines-windows, virtual-network, storage
documentationcenter: saponazure
author: goraco
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 0b89b4f8-6d6c-45d7-8d20-fe93430217ca
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 570c427166c07e93d3c8a7d3be079d9007238243
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34656817"
---
# <a name="create-an-sap-netweaver-multi-sid-configuration"></a>建立 SAP NetWeaver 多 SID 組態

[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md
[sap-ha-guide]:sap-high-availability-guide.md 
[sap-ha-guide-figure-6001]:./media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:./media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:./media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:./media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:./media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png
[networking-limits-azure-resource-manager]:../../../azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[sap-ha-guide-9.1.1]:sap-high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097 
[sap-ha-guide-8.8]:sap-high-availability-guide.md#f19bd997-154d-4583-a46e-7f5a69d0153c
[sap-ha-guide-8.12.3.3]:sap-high-availability-guide.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006 
[sap-ha-guide-9]:sap-high-availability-guide.md#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:sap-high-availability-guide.md#31c6bd4f-51df-4057-9fdf-3fcbc619c170 
[sap-ha-guide-9.1.2]:sap-high-availability-guide.md#eb5af918-b42f-4803-bb50-eff41f84b0b0 
[sap-ha-guide-9.1.3]:sap-high-availability-guide.md#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556 
[sap-ha-guide-9.1.4]:sap-high-availability-guide.md#10822f4f-32e7-4871-b63a-9b86c76ce761 
[sap-ha-guide-9.4]:sap-high-availability-guide.md#094bc895-31d4-4471-91cc-1513b64e406a 
[sap-ha-guide-9.5]:sap-high-availability-guide.md#2477e58f-c5a7-4a5d-9ae3-7b91022cafb5 
[sap-ha-guide-9.6]:sap-high-availability-guide.md#0ba4a6c1-cc37-4bcf-a8dc-025de4263772 
[sap-ha-guide-10]:sap-high-availability-guide.md#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9

Microsoft 在 2016 年 9 月發行的功能，可讓您使用 Azure 內部負載平衡器管理多個虛擬 IP 位址。 這項功能已存在 Azure 外部負載平衡器。

如果您有 SAP 部署，可以使用內部負載平衡器針對 SAP 的 ASCS/SCS 建立 Windows 叢集組態，如 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide]所述。

本文將著重於如何將單一 ASCS/SCS 安裝移至 SAP 多 SID 組態，方法是將其他 SAP ASCS/SCS 叢集執行個體安裝至現有 Windows Server 容錯移轉叢集 (WSFC) 叢集。 完成此程序之後，您將已設定 SAP 多 SID 叢集。


## <a name="prerequisites"></a>先決條件
您已經設定用於一個 SAP ASCS/SCS 執行個體的 WSFC 叢集，如 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide]所述及如下圖所示。

![高可用性的 SAP ASCS/SCS 執行個體][sap-ha-guide-figure-6001]

## <a name="target-architecture"></a>目標架構

目標是在相同 WSAFC 叢集中安裝多個 SAP ABAP ASCS 或 SAP Java SCS 叢集執行個體，如這裡所詳述：

![Azure 中多個 SAP ASCS/SCS 叢集執行個體][sap-ha-guide-figure-6002]

> [!NOTE]
>每個 Azure 內部負載平衡器的私人前端 IP 有數量限制。
>
>一個 WSFC 叢集中 SAP ASCS/SCS 執行個體數目上限等於每個 Azure 內部負載平衡器的私人前端 IP 數目上限。
>

如需更多有關負載平衡器限制的資訊，請參閱[網路限制：Azure Resource Manager][networking-limits-azure-resource-manager] 中的「每個負載平衡器的私人前端 IP」。

具有兩個高度可用 SAP 系統的完整配置畫面如下所示：

![具有兩個 SAP 系統 SID 的 SAP 高可用性多 SID 設定][sap-ha-guide-figure-6003]

> [!IMPORTANT]
> 安裝程式必須符合下列條件︰
> - SAP ASCS / SCS 執行個體必須共用相同的 WSFC 叢集。
> - 每個 DBMS SID 必須有其自己專用的 WSFC 叢集。
> - SAP 應用程式伺服器屬於必須擁有自己專用 VM 的一個 SAP 系統 SID。


## <a name="prepare-the-infrastructure"></a>準備基礎結構
若要準備您的基礎結構，您可以安裝額外的 SAP ASCS/SCS 執行個體，並使用下列參數︰

| 參數名稱 | 值 |
| --- | --- |
| SAP ASCS/SCS SID |pr1-lb-ascs |
| SAP DBMS 內部負載平衡器 | PR5 |
| SAP 虛擬主機名稱 | pr5-sap-cl |
| SAP ASCS/SCS 虛擬主機 IP 位址 (其他 Azure Load Balancer IP 位址) | 10.0.0.50 |
| SAP ASCS/SCS 執行個體號碼 | 50 |
| 其他 SAP ASCS/SCS 執行個體的 ILB 探查連接埠 | 62350 |

> [!NOTE]
> 對於 SAP ASCS/SCS 叢集執行個體，每個 IP 位址需要唯一的探查連接埠。 例如，如果 Azure 內部負載平衡器上有一個 IP 位址使用探查連接埠 62300，該負載平衡器上的任何其他 IP 位址就不能使用探查連接埠 62300。
>
>針對本文目的，因為已保留探查連接埠 62300，我們會使用探查連接埠 62350。

您可以在具有兩個節點的現有 WSFC 叢集中安裝額外 SAP ASCS/SCS 執行個體︰

| 虛擬機器角色 | 虛擬機器主機名稱 | 靜態 IP 位址 |
| --- | --- | --- |
| ASCS/SCS 執行個體的第 1 個叢集節點 |pr1-ascs-0 |10.0.0.10 |
| ASCS/SCS 執行個體的第 2 個叢集節點 |pr1-ascs-1 |10.0.0.9 |

### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance-on-the-dns-server"></a>在 DNS 伺服器上建立叢集 SAP ASCS/SCS 執行個體的虛擬主機名稱

您可以使用下列參數為 ASCS/SCS 執行個體的虛擬主機名稱建立 DNS 項目：

| 新的 SAP ASCS/SCS 虛擬主機名稱 | 相關聯的 IP 位址 |
| --- | --- | --- |
|pr5-sap-cl |10.0.0.50 |

新的主機名稱和 IP 位址會顯示在 DNS 管理員中，如下列螢幕擷取畫面所示︰

![DNS 管理員清單反白顯示已定義之新的 SAP ASCS/SCS 叢集虛擬名稱和 TCP/IP 位址的 DNS 項目][sap-ha-guide-figure-6004]

建立 DNS 項目的程序在主要的 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide-9.1.1]中也有詳細說明。

> [!NOTE]
> 您指派給 ASCS/SCS 執行個體之虛擬主機名稱的新 IP 位址必須與指派給 SAP Azure Load Balancer 的新 IP 位址相同。
>
>在我們的案例中，IP 位址是 10.0.0.50。

### <a name="add-an-ip-address-to-an-existing-azure-internal-load-balancer-by-using-powershell"></a>使用 PowerShell 將 IP 位址新增至現有的 Azure 內部負載平衡器

若要在相同的 WSFC 叢集中建立多個 SAP ASCS/SCS 執行個體，請使用 PowerShell 將 IP 位址新增至現有的 Azure 內部負載平衡器。 每個 IP 位址都需要有自己的負載平衡規則、探查連接埠、前端 IP 集區和後端集區。

下列指令碼會將新的 IP 位址新增至現有的負載平衡器。 請更新您環境的 PowerShell 變數。 指令碼會為所有 SAP ASCS/SCS 通訊埠建立所有必要的負載平衡規則。

```powershell

# Select-AzureRmSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>
Clear-Host
$ResourceGroupName = "SAP-MULTI-SID-Landscape"      # Existing resource group name
$VNetName = "pr2-vnet"                        # Existing virtual network name
$SubnetName = "Subnet"                        # Existing subnet name
$ILBName = "pr2-lb-ascs"                      # Existing ILB name                      
$ILBIP = "10.0.0.50"                          # New IP address
$VMNames = "pr2-ascs-0","pr2-ascs-1"          # Existing cluster virtual machine names
$SAPInstanceNumber = 50                       # SAP ASCS/SCS instance number: must be a unique value for each cluster
[int]$ProbePort = "623$SAPInstanceNumber"     # Probe port: must be a unique value for each IP and load balancer

$ILB = Get-AzureRmLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName

$count = $ILB.FrontendIpConfigurations.Count + 1
$FrontEndConfigurationName ="lbFrontendASCS$count"
$LBProbeName = "lbProbeASCS$count"

# Get the Azure VNet and subnet
$VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName
$Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

# Add second front-end and probe configuration
Write-Host "Adding new front end IP Pool '$FrontEndConfigurationName' ..." -ForegroundColor Green
$ILB | Add-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id
$ILB | Add-AzureRmLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 10  | Set-AzureRmLoadBalancer

# Get new updated configuration
$ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
# Get new updated LP FrontendIP COnfig
$FEConfig = Get-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

# Add new back-end configuration into existing ILB
$BackEndConfigurationName  = "backendPoolASCS$count"
Write-Host "Adding new backend Pool '$BackEndConfigurationName' ..." -ForegroundColor Green
$BEConfig = Add-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB | Set-AzureRmLoadBalancer

# Get new updated config
$ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

# Assign VM NICs to backend pool
$BEPool = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
foreach($VMName in $VMNames){
        $VM = Get-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VMName
        $NICName = ($VM.NetworkInterfaceIDs[0].Split('/') | select -last 1)        
        $NIC = Get-AzureRmNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName                
        $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools += $BEPool
        Write-Host "Assigning network card '$NICName' of the '$VMName' VM to the backend pool '$BackEndConfigurationName' ..." -ForegroundColor Green
        Set-AzureRmNetworkInterface -NetworkInterface $NIC
        #start-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VM.Name
}


# Create load-balancing rules
$Ports = "445","32$SAPInstanceNumber","33$SAPInstanceNumber","36$SAPInstanceNumber","39$SAPInstanceNumber","5985","81$SAPInstanceNumber","5$SAPInstanceNumber`13","5$SAPInstanceNumber`14","5$SAPInstanceNumber`16"
$ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
$FEConfig = get-AzureRMLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$BEConfig = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

Write-Host "Creating load balancing rules for the ports: '$Ports' ... " -ForegroundColor Green

foreach ($Port in $Ports) {

        $LBConfigrulename = "lbrule$Port" + "_$count"
        Write-Host "Creating load balancing rule '$LBConfigrulename' for the port '$Port' ..." -ForegroundColor Green

        $ILB | Add-AzureRmLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $HealthProbe -Protocol tcp -FrontendPort  $Port -BackendPort $Port -IdleTimeoutInMinutes 30 -LoadDistribution Default -EnableFloatingIP   
}

$ILB | Set-AzureRmLoadBalancer

Write-Host "Succesfully added new IP '$ILBIP' to the internal load balancer '$ILBName'!" -ForegroundColor Green

```
在執行指令碼之後，結果會顯示在 Azure 入口網站中，如下列螢幕擷取畫面所示︰

![Azure 入口網站中新的前端 IP 集區][sap-ha-guide-figure-6005]

### <a name="add-disks-to-cluster-machines-and-configure-the-sios-cluster-share-disk"></a>將磁碟新增至叢集機器，並設定 SIOS 叢集共用磁碟

對於每個額外 SAP ASCS/SCS 執行個體，您必須新增叢集共用磁碟。 針對 Windows Server 2012 R2，目前使用的 WSFC 叢集共用磁碟是 SIOS DataKeeper 軟體解決方案。

執行下列動作：
1. 將額外磁碟或大小相同的磁碟 (您需要等量的磁碟) 新增至每個叢集節點中，並將其格式化。
2. 使用 SIOS DataKeeper 設定儲存體複寫。

此程序假設您已在 WSFC 叢集機器上安裝了 SIOS DataKeeper。 如果已經安裝，您現在必須在電腦之間設定複寫。 如需此程序的詳細說明，請參閱主要的 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide-8.12.3.3]。  

![新的 SAP ASCS/SCS 共用磁碟的 DataKeeper 同步鏡像][sap-ha-guide-figure-6006]

### <a name="deploy-vms-for-sap-application-servers-and-dbms-cluster"></a>針對 SAP 應用程式伺服器和 DBMS 叢集部署 VM

若要完成第二個 SAP 系統的基礎結構準備，執行下列作業︰

1. 為 SAP 應用程式伺服器部署專用 VM，並將它們放在其自己專用的可用性群組。
2. 為 DBMS 叢集部署專用 VM，並將它們放在其自己專用的可用性群組。


## <a name="install-the-second-sap-sid2-netweaver-system"></a>安裝第二個 SAP SID2 NetWeaver 系統

如需有關安裝第二個 SAP SID2 系統的完整程序說明，請參閱主要的 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide-9]。

高階程序如下所示︰

1. [安裝 SAP 的第一個叢集節點][sap-ha-guide-9.1.2]。  
 在此步驟中，您要在**現有 WSFC 叢集節點 1** 上使用高可用性 ASCS/SCS 執行個體安裝 SAP。

2. [修改 ASCS/SCS 執行個體的 SAP 設定檔][sap-ha-guide-9.1.3]。

3. [設定探查連接埠][sap-ha-guide-9.1.4]。  
 在此步驟中，您要使用 PowerShell 設定 SAP 叢集資源 SAP SID2 IP 探查連接埠。 在其中一個 SAP ASCS/SCS 叢集節點上執行此組態。

4. [安裝資料庫執行個體][sap-ha-guide-9.2]。  
 在此步驟中，您要在專用的 WSFC 叢集上安裝 DBMS。

5. [安裝第二個叢集節點][sap-ha-guide-9.3]。  
 在此步驟中，您要在現有 WSFC 叢集節點 2 上使用高可用性 ASCS/SCS 執行個體安裝 SAP。

6. 開啟 SAP ASCS/SCS 執行個體和 ProbePort 的 Windows 防火牆連接埠。  
 在用於 SAP ASCS/SCS 執行個體的兩個叢集節點上，您要開啟 SAP ASCS/SCS 所使用的所有 Windows 防火牆連接埠。 如需這些連接埠的清單，請參閱 [Windows VM 上的高可用性 SAP NetWeaver 指南][sap-ha-guide-8.8]。  
 此外，開啟 Azure 內部負載平衡器探查連接埠，在我們的案例中為 62350。

7. [變更 SAP ERS Windows 服務執行個體的啟動類型][sap-ha-guide-9.4]。

8. 在新的專用 VM 上[安裝 SAP 主要應用程式伺服器][sap-ha-guide-9.5]。

9. 在新的專用 VM 上[安裝 SAP 額外應用程式伺服器][sap-ha-guide-9.6]。

10. [測試 SAP ASCS/SCS 執行個體容錯移轉和 SIOS 複寫][sap-ha-guide-10]。

## <a name="next-steps"></a>後續步驟

- [網路限制：Azure Resource Manager][networking-limits-azure-resource-manager]
- [Azure Load Balancer 的多個 VIP][load-balancer-multivip-overview]
- [Windows VM 上的 SAP NetWeaver 高可用性指南][sap-ha-guide]
