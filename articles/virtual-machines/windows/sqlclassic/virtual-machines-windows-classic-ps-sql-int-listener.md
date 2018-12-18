---
title: 在 Azure 中設定 Always On 可用性群組的 ILB 接聽程式 | Microsoft Docs
description: 本教學課程使用以傳統部署模型建立的資源，並在 Azure 中建立使用內部負載平衡器的 Always On 可用性群組接聽程式。
services: virtual-machines-windows
documentationcenter: na
author: MikeRayMSFT
manager: craigg
editor: ''
tags: azure-service-management
ms.assetid: 291288a0-740b-4cfa-af62-053218beba77
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/02/2017
ms.author: mikeray
ms.openlocfilehash: 0466265ad5a24e8ea6dc5079e2b4006d74e7dde0
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38452527"
---
# <a name="configure-an-ilb-listener-for-always-on-availability-groups-in-azure"></a>在 Azure 中設定 Always On 可用性群組的 ILB 接聽程式
> [!div class="op_single_selector"]
> * [內部接聽程式](../classic/ps-sql-int-listener.md)
> * [外部接聽程式](../classic/ps-sql-ext-listener.md)
>
>

## <a name="overview"></a>概觀

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有兩種：[Azure Resource Manager](../../../azure-resource-manager/resource-manager-deployment-model.md) 和傳統。 本文涵蓋傳統部署模型的使用。 我們建議讓大部分的新部署使用 Resource Manager 模型。

若要在 Resource Manager 模型中設定 Always On 可用性群組的接聽程式，請參閱[在 Azure 中設定 Always On 可用性群組的負載平衡器](../sql/virtual-machines-windows-portal-sql-alwayson-int-listener.md)。

您的可用性群組可包含的複本為僅限內部部署或僅限 Azure，或同時跨內部部署和 Azure 的混合式組態。 Azure 複本可位於相同區域內，或跨越使用多個虛擬網路的多個區域。 本文中的程序假設您已經[設定可用性群組](../classic/portal-sql-alwayson-availability-groups.md)，但尚未設定接聽程式。

## <a name="guidelines-and-limitations-for-internal-listeners"></a>內部接聽程式指導方針和限制
在 Azure 中使用內部負載平衡器 (ILB) 搭配可用性群組接聽程式時，必須遵循下列指導方針：

* 可用性群組接聽程式支援 Windows Server 2008 R2、Windows Server 2012 和 Windows Server 2012 R2。
* 每個雲端服務僅支援一個內部可用性群組接聽程式，因為接聽程式被設定為 ILB，且每個雲端服務僅有一個 ILB； 但是可以建立多個外部接聽程式。 如需詳細資訊，請參閱[在 Azure 中設定 Always On 可用性群組的外部接聽程式](../classic/ps-sql-ext-listener.md)。

## <a name="determine-the-accessibility-of-the-listener"></a>判斷接聽程式的存取性
[!INCLUDE [ag-listener-accessibility](../../../../includes/virtual-machines-ag-listener-determine-accessibility.md)]

本文著重於建立使用 ILB 的接聽程式。 如果您需要公用或外部接聽程式，請參閱本文討論設定[外部接聽程式](../classic/ps-sql-ext-listener.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)的版本。

## <a name="create-load-balanced-vm-endpoints-with-direct-server-return"></a>使用伺服器直接回傳建立負載平衡 VM 端點
藉由執行本節後面的指令碼，先建立 ILB。

為每部主控 Azure 複本的 VM 建立負載平衡端點。 如果您的複本位於多個區域，則該區域的每個複本必須位於相同 Azure 虛擬網路中的相同雲端服務。 建立跨越多個 Azure 區域的可用性群組複本需要設定多個虛擬網路。 如需設定跨虛擬網路連線的詳細資訊，請參閱[設定虛擬網路對虛擬網路連線](../../../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md)。

1. 在 Azure 入口網站中，移至每部主控複本的 VM 以檢視詳細資料。

2. 按一下每部 VM 的 [端點] 索引標籤。

3. 對於您想要使用的接聽程式端點，確認其 [名稱] 和 [公用連接埠] 並未使用中。 在本節的範例中，名稱是 MyEndpoint，而通訊埠為 1433。

4. 在您的本機用戶端上，下載並安裝最新的 [PowerShell 模組](https://azure.microsoft.com/downloads/)。

5. 啟動 Azure PowerShell。  
    新的 PowerShell 工作階段隨即開啟並載入 Azure 系統管理模組。

6. 執行 `Get-AzurePublishSettingsFile`。 這個 Cmdlet 會將您導向瀏覽器，以便將發佈設定檔案下載至本機目錄。 系統可能會提示您輸入 Azure 訂用帳戶的登入認證。

7. 使用您所下載發佈設定檔案的路徑來執行下列 `Import-AzurePublishSettingsFile` 命令：

        Import-AzurePublishSettingsFile -PublishSettingsFile <PublishSettingsFilePath>

    匯入發佈設定檔案之後，您便可以在 PowerShell 工作階段中管理 Azure 訂用帳戶。

8. 針對 *ILB*，指派靜態 IP 位址。 執行下列命令來檢查目前的虛擬網路組態：

        (Get-AzureVNetConfig).XMLConfiguration
9. 請記下子網路 (其中包含主控複本的 VM) 的 *Subnet* 名稱。 此名稱使用於指令碼中的 $SubnetName 參數。

10. 記下子網路 (其中包含主控複本的 VM) 的 VirtualNetworkSite 名稱和起始的 AddressPrefix。 將這兩個值傳遞至 `Test-AzureStaticVNetIP` 命令並檢查 AvailableAddresses 以尋找可用的 IP 位址。 例如，如果虛擬網路被命名為 MyVNet 且具有以 172.16.0.128 開始的子網路位址範圍，下列命令便會列出可用的位址：

        (Test-AzureStaticVNetIP -VNetName "MyVNet"-IPAddress 172.16.0.128).AvailableAddresses
11. 選取其中一個可用的位址，並將其用於下一個步驟中指令碼的 $ILBStaticIP 參數。

12. 將下列 PowerShell 指令碼複製到文字編輯器，並設定變數值以符合您的環境。 某些參數已提供預設值。  

    使用同質群組的現有部署無法新增 ILB。 如需有關 ILB 需求的詳細資訊，請參閱[內部負載平衡器概觀](../../../load-balancer/load-balancer-internal-overview.md)。

    此外，如果您的可用性群組跨越 Azure 區域，您必須針對雲端服務和位於該資料中心的節點，在每個資料中心執行一次指令碼。

        # Define variables
        $ServiceName = "<MyCloudService>" # the name of the cloud service that contains the availability group nodes
        $AGNodes = "<VM1>","<VM2>","<VM3>" # all availability group nodes containing replicas in the same cloud service, separated by commas
        $SubnetName = "<MySubnetName>" # subnet name that the replicas use in the virtual network
        $ILBStaticIP = "<MyILBStaticIPAddress>" # static IP address for the ILB in the subnet
        $ILBName = "AGListenerLB" # customize the ILB name or use this default value

        # Create the ILB
        Add-AzureInternalLoadBalancer -InternalLoadBalancerName $ILBName -SubnetName $SubnetName -ServiceName $ServiceName -StaticVNetIPAddress $ILBStaticIP

        # Configure a load-balanced endpoint for each node in $AGNodes by using ILB
        ForEach ($node in $AGNodes)
        {
            Get-AzureVM -ServiceName $ServiceName -Name $node | Add-AzureEndpoint -Name "ListenerEndpoint" -LBSetName "ListenerEndpointLB" -Protocol tcp -LocalPort 1433 -PublicPort 1433 -ProbePort 59999 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -InternalLoadBalancerName $ILBName -DirectServerReturn $true | Update-AzureVM
        }

13. 設定變數之後，請從文字編輯器將指令碼複製到您的 PowerShell 工作階段來執行它。 如果提示依然顯示 **>>**，請再次按 ENTER 鍵以確定指令碼開始執行。

## <a name="verify-that-kb2854082-is-installed-if-necessary"></a>必要時，請確認已安裝 KB2854082
[!INCLUDE [kb2854082](../../../../includes/virtual-machines-ag-listener-kb2854082.md)]

## <a name="open-the-firewall-ports-in-availability-group-nodes"></a>在可用性群組節點中開啟防火牆連接埠
[!INCLUDE [firewall](../../../../includes/virtual-machines-ag-listener-open-firewall.md)]

## <a name="create-the-availability-group-listener"></a>建立可用性群組接聽程式

透過兩個步驟建立可用性群組接聽程式。 首先，建立用戶端存取點叢集資源，並設定相依性。 接著，在 PowerShell 中設定叢集資源。

### <a name="create-the-client-access-point-and-configure-the-cluster-dependencies"></a>建立用戶端存取點並設定叢集的相依性
[!INCLUDE [firewall](../../../../includes/virtual-machines-ag-listener-create-listener.md)]

### <a name="configure-the-cluster-resources-in-powershell"></a>在 PowerShell 中設定叢集資源
1. 對於 ILB，您必須使用之前所建立 ILB 的 IP 位址。 若要取得 PowerShell 中的這個 IP 位址，請使用下列指令碼：

        # Define variables
        $ServiceName="<MyServiceName>" # the name of the cloud service that contains the AG nodes
        (Get-AzureInternalLoadBalancer -ServiceName $ServiceName).IPAddress

2. 在其中一部 VM 上，將適用於您作業系統的 PowerShell 指令碼複製到文字編輯器，然後將變數設定為您先前記下的值。

    針對 Windows Server 2012 或更新版本，請使用下列指令碼︰

        # Define variables
        $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
        $IPResourceName = "<IPResourceName>" # the IP address resource name
        $ILBIP = "<X.X.X.X>" # the IP address of the ILB

        Import-Module FailoverClusters

        Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"="59999";"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}

    針對 Windows Server 2008 R2，請使用下列指令碼︰

        # Define variables
        $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
        $IPResourceName = "<IPResourceName>" # the IP address resource name
        $ILBIP = "<X.X.X.X>" # the IP address of the ILB

        Import-Module FailoverClusters

        cluster res $IPResourceName /priv enabledhcp=0 address=$ILBIP probeport=59999  subnetmask=255.255.255.255

3. 設定變數之後，開啟提升權限的 Windows PowerShell 視窗，然後將指令碼從文字編輯器貼到您的 PowerShell 工作階段中來執行它。 如果提示依然顯示 **>>**，請再次按 ENTER 鍵以確定指令碼開始執行。

4. 對每部 VM 重複上述步驟。  
    此指令碼會使用雲端服務的 IP 位址來設定 IP 位址資源，並設定其他參數 (例如探查連接埠)。 當 IP 位址資源處於線上時，它會從您稍早所建立的負載平衡端點，回應探查連接埠上的輪詢。

## <a name="bring-the-listener-online"></a>使接聽程式上線
[!INCLUDE [Bring-Listener-Online](../../../../includes/virtual-machines-ag-listener-bring-online.md)]

## <a name="follow-up-items"></a>待處理項目
[!INCLUDE [Follow-up](../../../../includes/virtual-machines-ag-listener-follow-up.md)]

## <a name="test-the-availability-group-listener-within-the-same-virtual-network"></a>測試可用性群組接聽程式 (位於相同的虛擬網路中)
[!INCLUDE [Test-Listener-Within-VNET](../../../../includes/virtual-machines-ag-listener-test.md)]

## <a name="next-steps"></a>後續步驟
[!INCLUDE [Listener-Next-Steps](../../../../includes/virtual-machines-ag-listener-next-steps.md)]
