---
title: 具備 Site Recovery 的公用 IP 位址 | Microsoft Docs
description: 說明如何使用 Azure Site Recovery 和 Azure 流量管理員設定公用 IP 位址，以進行災害復原和移轉
services: site-recovery
documentationcenter: ''
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 09/28/2018
ms.author: mayg
ms.openlocfilehash: fd17c786515a6db1f8aa5c1aff7c0dcd42ed7dc6
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47435819"
---
# <a name="public-ip-address-with-site-recovery"></a>具備 Site Recovery 的公用 IP 位址

公用 IP 位址可讓網際網路資源向 Azure 資源進行輸入通訊。 公用 IP 位址也可透過指派給資源的 IP 位址，讓 Azure 資源向網際網路和公眾對應的 Azure 服務進行輸出通訊。
- 啟用從網路網路到資源的輸入通訊；資源包括 Azure 虛擬機器 (VM)、Azure 應用程式閘道、Azure Load Balancer、Azure VPN 閘道等等。 即使未將公用 IP 位址指派給該虛擬機器，只要虛擬機器是負載平衡器後端集區的一部分，且已將公用 IP 位址指派給負載平衡器，就可以讓您與來自網際網路的虛擬機器等資源進行通訊。
- 使用可預測 IP 位址對網際網路進行輸出連線。 例如，虛擬機器不需有指派的公用 IP 位址，即可對網際網路進行輸出通訊，但其位址是由 Azure 轉譯而成的網路位址 (依預設是無法預測的公用位址)。 指派公用 IP 位址給資源，可讓您知道輸出連線所使用的 IP 位址。 雖然可預測，但位址可能根據選擇的指派方法而有所變更。 如需詳細資訊，請參閱＜[建立公用 IP 位址](../virtual-network/virtual-network-public-ip-address.md#create-a-public-ip-address)＞。 若要深入了解 Azure 資源的輸出連線，請參閱[了解輸出連線](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

在 Azure Resource Manager 中，公用 IP 位址是有自己屬性的資源。 可供公用 IP 位址資源與之建立關聯的部分資源如下：

* 虛擬機器網路介面
* 網際網路對應負載平衡器
* VPN 閘道
* 應用程式閘道

此文章說明如何搭配 Site Recovery 使用公用 IP 位址。

## <a name="public-ip-address-assignment-using-recovery-plan"></a>使用復原計劃的公用 IP 位址指派

**容錯移轉時無法保留**實際執行應用程式的公用 IP 位址。 目標區域中必須為容錯移轉程序中產生的工作負載指派一個可用的 Azure 公用 IP 資源。 此步驟可以手動進行或使用復原方案自動完成。 復原方案會將機器收集至復原群組中。 它可協助您定義系統復原程序。 您可以使用復原方案強制加上順序，並使用 Azure 自動化 Runbook 容錯移轉至 Azure 或使用指令碼，自動執行每個步驟所需的動作。

設定方式如下︰
- 建立[復原方案](../site-recovery/site-recovery-create-recovery-plans.md#create-a-recovery-plan)，並視需要，在方案中為工作負載分組。
- 若要自訂方案，請使用 [Azure 自動化 Runbook](../site-recovery/site-recovery-runbook-automation.md#customize-the-recovery-plan) 指令碼新增步驟，以便將公用 IP 位址連結至容錯移轉的 VM。

 
## <a name="public-endpoint-switching-with-dns-level-routing"></a>使用 DNS 層級路由切換公用端點

Azure 流量管理員允許在端點之間切換 DNS 層級路由，並可針對 DR 案例，協助[降低您的 RTO](../site-recovery/concepts-traffic-manager-with-site-recovery.md#recovery-time-objective-rto-considerations)。 

深入了解流量管理員的容錯移轉案例：
1. 使用流量管理員[從內部部署容錯移轉至 Azure](../site-recovery/concepts-traffic-manager-with-site-recovery.md#on-premises-to-azure-failover) 
2. 使用流量管理員[在 Azure 之間容錯移轉](../site-recovery/concepts-traffic-manager-with-site-recovery.md#azure-to-azure-failover) 

設定方式如下︰
- 建立[流量管理員設定檔](../traffic-manager/traffic-manager-create-profile.md)。
- 利用**優先順序**路由方法建立兩個端點 – **主要**用於來源，**容錯移轉**用於 Azure。 **主要**會被指派優先順序 1，**容錯移轉**會被指派優先順序 2。
- **主要**端點可以是 [Azure](../traffic-manager/traffic-manager-endpoint-types.md#azure-endpoints) 或[外部](../traffic-manager/traffic-manager-endpoint-types.md#external-endpoints)，端視您的來源環境是在 Azure 內部還是外部而定。
- **容錯移轉**端點會建立為 **Azure** 端點。 使用**靜態公用 IP 位址**，因為這是災害事件中流量管理員的外部端點。

## <a name="next-steps"></a>後續步驟
- 深入了解[流量管理員搭配 Azure Site Recovery](../site-recovery/concepts-traffic-manager-with-site-recovery.md)
- 深入了解流量管理員的[路由方法](../traffic-manager/traffic-manager-routing-methods.md)。
- 深入了解自動執行應用程式容錯移轉的[復原計劃](site-recovery-create-recovery-plans.md)。
