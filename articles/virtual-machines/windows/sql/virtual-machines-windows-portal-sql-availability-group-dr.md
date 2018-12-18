---
title: SQL Server 可用性群組 - Azure 虛擬機器 - 災害復原 | Microsoft Docs
description: 本文說明如何在於不同區域中有複本的 Azure 虛擬機器上設定 SQL Server 可用性群組。
services: virtual-machines
documentationCenter: na
authors: MikeRayMSFT
manager: craigg
editor: monicar
tags: azure-service-management
ms.assetid: 388c464e-a16e-4c9d-a0d5-bb7cf5974689
ms.service: virtual-machines-sql
ms.devlang: na
ms.custom: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/02/2017
ms.author: mikeray
ms.openlocfilehash: 84fa2e051c46e178e3e72709886babc8c3db7b9d
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29852824"
---
# <a name="configure-an-always-on-availability-group-on-azure-virtual-machines-in-different-regions"></a>在不同區域的 Azure 虛擬機器上設定 Always On 可用性群組

本文說明如何在遠端 Azure 位置的「Azure 虛擬機器」上設定「SQL Server Always On 可用性群組」複本。 您可以使用此組態來支援災害復原。

本文適用於 Resource Manager 模式的「Azure 虛擬機器」。

下圖顯示 Azure 虛擬機器上常見的可用性群組部署：

   ![可用性群組](./media/virtual-machines-windows-portal-sql-availability-group-dr/00-availability-group-basic.png)

在此部署中，所有虛擬機器都位於一個 Azure 區域中。 可用性群組複本可以使用同步認可搭配在 SQL-1 和 SQL-2 上自動容錯移轉。 若要建置此架構，請參閱[可用性群組範本或教學課程](virtual-machines-windows-portal-sql-availability-group-overview.md)。

此架構很容易因 Azure 區域變成無法存取而停機。 若要克服這個弱點，請在不同的 Azure 區域中新增複本。 下圖顯示新架構看起來的樣子：

   ![可用性群組 DR](./media/virtual-machines-windows-portal-sql-availability-group-dr/00-availability-group-basic-dr.png)

上圖顯示一個名為 SQL-3 的新虛擬機器。 SQL-3 位於不同的 Azure 區域中。 SQL-3 被新增到「Windows Server 容錯移轉叢集」中。 SQL-3 可以裝載可用性群組複本。 最後，請注意，SQL-3 的 Azure 區域有一個新的 Azure Load Balancer。

>[!NOTE]
> 當相同區域中有多部虛擬機器時，必須要有 Azure 可用性設定組。 如果區域中只有一部虛擬機器，則不需要有可用性設定組。 您只能在建立虛擬機器時將其置於可用性設定組中。 如果虛擬機器已經在可用性設定組中，則您可以稍後新增虛擬機器作為額外複本。

在此架構中，通常會以非同步認可可用性模式和手動容錯移轉模式設定遠端區域中的複本。

當可用性群組複本位於不同 Azure 區域中的 Azure 虛擬機器上時，每個區域需要：

* 虛擬網路閘道
* 虛擬網路閘道連線

下圖顯示資料中心之間的網路通訊方式。

   ![可用性群組](./media/virtual-machines-windows-portal-sql-availability-group-dr/01-vpngateway-example.png)

>[!IMPORTANT]
>此架構會針對 Azure 區域之間複寫的資料產生輸出資料費用。 請參閱[頻寬定價](http://azure.microsoft.com/pricing/details/bandwidth/)。  

## <a name="create-remote-replica"></a>建立遠端複本

若要在遠端資料中心建立複本，請執行下列步驟：

1. [在新區域中建立虛擬網路](../../../virtual-network/manage-virtual-network.md#create-a-virtual-network)。

1. [使用 Azure 入口網站設定 VNet 對 VNet 連接](../../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)。

   >[!NOTE]
   >在某些情況下，您可能需要使用 PowerShell 來建立 VNet 對 VNet 連接。 例如，如果您使用不同的 Azure 帳戶，您就無法在入口網站中設定該連線。 在此情況下，請參閱[使用 Azure 入口網站設定 VNet 對 VNet 連接](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)。

1. [在新區域中建立網域控制站](../../../active-directory/active-directory-new-forest-virtual-machine.md)。

   此網域控制站可在主要站台中的網域控制站無法供使用時，提供驗證。

1. [在新區域中建立 SQL Server 虛擬機器](virtual-machines-windows-portal-sql-server-provision.md)。

1. [在新區域的網路中建立 Azure Load Balancer](virtual-machines-windows-portal-sql-availability-group-tutorial.md#configure-internal-load-balancer)。

   此負載平衡器必須：

   - 與新虛擬機器位於相同的網路和子網路。
   - 擁有「可用性群組」接聽程式的靜態 IP 位址。
   - 包含一個僅由相同區域中的虛擬機器所組成的後端集區作為負載平衡器。
   - 使用 IP 位址特定的 TCP 連接埠探查。
   - 擁有相同區域中 SQL Server 特定的負載平衡規則。  

1. [將容錯移轉叢集功能新增到新的 SQL Server](virtual-machines-windows-portal-sql-availability-group-prereq.md#add-failover-clustering-features-to-both-sql-server-vms)。

1. [將新的 SQL Server 加入網域](virtual-machines-windows-portal-sql-availability-group-prereq.md#joinDomain)。

1. [設定新的 SQL Server 服務帳戶以使用網域帳戶](virtual-machines-windows-portal-sql-availability-group-prereq.md#setServiceAccount)。

1. [將新的 SQL Server 新增到 Windows Server 容錯移轉叢集](virtual-machines-windows-portal-sql-availability-group-tutorial.md#addNode)。

1. 在叢集上建立 IP 位址資源。

   您可以在「容錯移轉叢集管理員」中建立 IP 位址資源。 在可用性群組角色上按一下滑鼠右鍵，然後依序按一下 [加入資源]、[其他資源]、[IP 位址]。

   ![建立 IP 位址](./media/virtual-machines-windows-portal-sql-availability-group-dr/20-add-ip-resource.png)

   請依照下列方式設定此 IP 位址：

   - 使用來自遠端資料中心的網路。
   - 指派來自新 Azure Load Balancer 的 IP 位址。 

1. 在「SQL Server 組態管理員」中的新 SQL Server 上，[啟用 Always On 可用性群組](http://msdn.microsoft.com/library/ff878259.aspx)。

1. [在新的 SQL Server 上開啟防火牆連接埠](virtual-machines-windows-portal-sql-availability-group-prereq.md#endpoint-firewall)。

   您需要開啟的連接埠號碼取決於您的環境。 請開啟要供鏡像端點及 Azure Load Balancer 健康情況探查使用的連接埠。

1. [將複本新增到新 SQL Server 上的可用性群組](http://msdn.microsoft.com/library/hh213239.aspx)。

   針對遠端 Azure 區域中的複本，請設定它來進行非同步複寫搭配手動容錯移轉。  

1. 將 IP 位址資源新增為接聽程式用戶端存取點 (網路名稱) 叢集的相依性。

   以下螢幕擷取畫面顯示的是已正確設定的 IP 位址叢集資源：

   ![可用性群組](./media/virtual-machines-windows-portal-sql-availability-group-dr/50-configure-dependency-multiple-ip.png)

   >[!IMPORTANT]
   >叢集資源群組同時包含兩個 IP位址。 這兩個 IP 位址都是接聽程式用戶端存取點的相依性。 在叢集相依性組態中，請使用 **OR** 運算子。

1. [在 PowerShell 中設定叢集參數](virtual-machines-windows-portal-sql-availability-group-tutorial.md#setparam)。

使用您在新區域中負載平衡器上設定的叢集網路名稱、IP 位址及探查連接埠來執行 PowerShell 指令碼。

   ```PowerShell
   $ClusterNetworkName = "<MyClusterNetworkName>" # The cluster name for the network in the new region (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name).
   $IPResourceName = "<IPResourceName>" # The cluster name for the new IP Address resource.
   $ILBIP = “<n.n.n.n>” # The IP Address of the Internal Load Balancer (ILB) in the new region. This is the static IP address for the load balancer you configured in the Azure portal.
   [int]$ProbePort = <nnnnn> # The probe port you set on the ILB.

   Import-Module FailoverClusters

   Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   ```

## <a name="set-connection-for-multiple-subnets"></a>設定多個子網路的連線

遠端資料中心內的複本是可用性群組的一部份，但是位於不同的子網路。 如果此複本變成主要複本，可能會發生應用程式連線逾時。 此行為與多子網路部署中的內部部署可用性群組相同。 若要允許來自用戶端應用程式的連線，請更新用戶端連線，或在叢集網路名稱資源上設定名稱解析快取功能。

建議您更新用戶端連接字串以設定 `MultiSubnetFailover=Yes`。 請參閱[使用 MultiSubnetFailover 進行連接](http://msdn.microsoft.com/library/gg471494#Anchor_0)。

如果您無法修改連接字串，您可以設定名稱解析快取功能。 請參閱[多子網路可用性群組中的連線逾時 (英文)](http://blogs.msdn.microsoft.com/alwaysonpro/2014/06/03/connection-timeouts-in-multi-subnet-availability-group/)。

## <a name="fail-over-to-remote-region"></a>容錯移轉至遠端區域

若要測試對遠端區域的接聽程式連線能力，您可以將複本容錯移轉至遠端區域。 當複本是非同步複本時，容錯移轉容易導致潛在的資料遺失。 若要容錯移轉又不遺失資料，請將可用性模式變更為同步，並將容錯移轉模式設定為自動。 請使用下列步驟：

1. 在 [物件總管] 中，連接到裝載主要複本的 SQL Server 執行個體。
1. 在 [AlwaysOn 可用性群組]、[可用性群組] 底下，於您的可用性群組上按一下滑鼠右鍵，然後按一下 [屬性]。
1. 在 [一般] 頁面上的 [可用性複本] 底下，將 DR 站台中的次要複本設定成使用 [同步認可] 模式和 [自動] 容錯移轉模式。
1. 如果您在與主要複本相同的站台中有次要複本以支援高可用性，請將此複本設定為 [非同步認可] 和 [手動]。
1. 按一下 [確定]。
1. 在 [物件總管] 中，於可用性群組上按一下滑鼠右鍵，然後按一下 [顯示儀表板]。
1. 在儀表板上，確認 DR 站台上的複本已同步。
1. 在 [物件總管] 中，於可用性群組上按一下滑鼠右鍵，然後按一下 [容錯移轉]。SQL Server Management Studio 會開啟精靈來容錯移轉 SQL Server。  
1. 按 [下一步]，然後選取 DR 站台中的 SQL Server 執行個體。 再按一下 [下一步]  。
1. 連接到 DR 站台中的 SQL Server 執行個體，然後按 [下一步]。
1. 在 [摘要] 頁面上確認設定，然後按一下 [完成]。

測試完連線之後，請將主要複本移回到您的主要資料中心，並將可用性模式設定回其一般作業設定。 下表顯示本文件所述架構的一般作業設定：

| 位置 | 伺服器執行個體 | 角色 | 可用性模式 | 容錯移轉模式
| ----- | ----- | ----- | ----- | -----
| 主要資料中心 | SQL-1 | 主要 | 同步 | 自動
| 主要資料中心 | SQL-2 | 次要 | 同步 | 自動
| 次要或遠端資料中心 | SQL-3 | 次要 | 非同步 | 手動


### <a name="more-information-about-planned-and-forced-manual-failover"></a>有關計劃性和強制性容錯移轉的更多詳細資訊

如需詳細資訊，請參閱下列主題：

- [執行可用性群組的已規劃手動容錯移轉 (SQL Server)](http://msdn.microsoft.com/library/hh231018.aspx)
- [執行可用性群組的強制手動容錯移轉 (SQL Server)](http://msdn.microsoft.com/library/ff877957.aspx)

## <a name="additional-links"></a>其他連結

* [Always On 可用性群組](http://msdn.microsoft.com/library/hh510230.aspx)
* [Azure 虛擬機器](http://docs.microsoft.com/azure/virtual-machines/windows/)
* [Azure Load Balancer](virtual-machines-windows-portal-sql-availability-group-tutorial.md#configure-internal-load-balancer)
* [Azure 可用性設定組](../manage-availability.md)
