---
title: Azure 服務的虛擬網路 | Microsoft Docs
description: 了解將資源部署到擬網路的優點。 虛擬網路中的資源可以彼此通訊，並且可以內部部署資源，而不會產生周遊網際網路的流量。
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: jdial
ms.openlocfilehash: 163a8f318608ba980d98209277371fb4d148c4ff
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48902768"
---
# <a name="virtual-network-integration-for-azure-services"></a>Azure 服務的虛擬網路整合

透過將 Azure 服務整合到 Azure 虛擬網路，可以啟用從虛擬機器或虛擬網路中的計算資源對服務進行私下存取。
您可以使用下列選項將 Azure 服務整合到您的虛擬網路：將服務的專用執行個體直接部署到虛擬網路中。 然後，在虛擬網路和內部部署網路中，可以私下存取這些服務。
透過服務端點將虛擬網路擴充至服務。 服務端點可保障虛擬網路使用得到個別服務資源。

若要將多個 Azure 服務整合到您的虛擬網路，您可以結合上述一或多種模式。 例如，您可以將 HDInsight 部署到您的虛擬網路，並透過服務端點保護 HDInsight 子網路的儲存體帳戶。
 
## <a name="deploy-azure-services-into-virtual-networks"></a>將 Azure 服務部署至虛擬網路

若在[虛擬網路](virtual-networks-overview.md)中部署專用 Azure 服務，可以使用私人 IP 位址，私下與該服務資源通訊。

![部署在虛擬網路中的服務](./media/virtual-network-for-azure-services/deploy-service-into-vnet.png)

部署在虛擬網路中的服務有下列功能：

- 虛擬網路中的資源可以透過私人 IP 位址，私下互相通訊。 例如，在虛擬網路中，直接在於虛擬機器上執行的 HDInsight 和 SQL Server 之間傳輸資料。
- 內部部署資源可以透過[站對站 VPN (VPN 閘道)](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json#s2smulti) 或 [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，使用私人 IP 位址，存取虛擬網路中的資源。
- 虛擬網路則可以使用私人 IP 位址進行[對等互連](virtual-network-peering-overview.md)，來讓虛擬網路中的資源彼此通訊。
- 虛擬網路中的服務執行個體完全由 Azure 服務管理，以監控執行個體的健康情況，及根據負載提供需要的規模。
- 服務執行個體已部署至虛擬網路中的子網路。 根據服務提供的指導，必須透過子網路的[網路安全性群組](security-overview.md#network-security-groups)，開放輸入和輸出網路存取。
- (選擇性) 服務可能需要[委派的子網路](virtual-network-manage-subnet.md#add-a-subnet)，作為子網路可以裝載特定服務的明確識別項。 子網路委派提供明確的權限給服務，以在子網路中建立服務特定資源。

### <a name="services-that-can-be-deployed-into-a-virtual-network"></a>可以部署至虛擬網路的服務

|類別|服務|
|-|-|
| 計算 | 虛擬機器：[Linux](../virtual-machines/linux/infrastructure-networking-guidelines.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Windows](../virtual-machines/windows/infrastructure-networking-guidelines.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-mvss-existing-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[雲端服務](https://msdn.microsoft.com/library/azure/jj156091)：僅限虛擬網路 (傳統)<br/> [Azure Batch](../batch/batch-api-basics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#virtual-network-vnet-and-firewall-configuration)  |
| 網路 | [應用程式閘道 - WAF](../application-gateway/application-gateway-ilb-arm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[VPN 閘道](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure 防火牆](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) <br/>[網路虛擬設備](/windowsserverdocs/WindowsServerDocs/networking/sdn/manage/Use-Network-Virtual-Appliances-on-a-VN.md) 
|資料|[RedisCache](../redis-cache/cache-how-to-premium-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure SQL Database 受控執行個體](../sql-database/sql-database-managed-instance-vnet-configuration.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|
分析 | [Azure HDInsight](../hdinsight/hdinsight-extend-hadoop-virtual-network.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure Databricks](../azure-databricks/what-is-azure-databricks.md?toc=%2fazure%2fvirtual-network%2ftoc.json) |
| 身分識別 | [Azure Active Directory 網域服務](../active-directory-domain-services/active-directory-ds-getting-started-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json) |
| 容器 | [Azure Kubernetes Service (AKS)](../aks/networking-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure 容器執行個體 (ACI)](http://www.aka.ms/acivnet)<br/>[Azure Container Service 引擎](https://github.com/Azure/acs-engine)搭配 Azure 虛擬網路 CNI [外掛程式](https://github.com/Azure/acs-engine/tree/master/examples/vnet)||
| Web | [API 管理](../api-management/api-management-using-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[App Service 環境](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>
<br/>


## <a name="service-endpoints-for-azure-services"></a>Azure 服務的服務端點

某些 Azure 服務無法部署在虛擬網路中。 您可以選擇將部分的服務資源只開放給特定虛擬網路子網路存取，只要啟用虛擬網路服務端點即可。  深入了解[虛擬網路服務端點](virtual-network-service-endpoints-overview.md)，以及可啟用端點的服務。
