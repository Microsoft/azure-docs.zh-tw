---
title: 設定 IP 位址以在使用 Azure Site Recovery 容錯移轉至 Azure 之後進行連線 | Microsoft Docs
description: 說明如何設定 IP 位址以在使用 Azure Site Recovery 從內部部署容錯移轉之後連線至 Azure VM
services: site-recovery
documentationcenter: ''
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: mayg
ms.openlocfilehash: 9b7797ba58a44e64460f4dbc58d562b4c470c043
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353082"
---
# <a name="set-up-ip-addressing-to-connect-after-failover-to-azure"></a>設定 IP 位址以在容錯移轉至 Azure 之後進行連線

本文說明使用 [Azure Site Recovery](site-recovery-overview.md) 服務來複寫和容錯移轉至 Azure之後，連線至 Azure VM 的網路服務需求。

在本文中，您將了解：

> [!div class="checklist"]
> * 您可以使用的連線方法
> * 如何將不同的 IP 位址用於複寫的 Azure VM
> * 如何在容錯移轉之後保留 Azure VM 的 IP 位址

## <a name="connecting-to-replica-vms"></a>連線到複本 VM

規劃您的複寫和容錯移轉策略時，其中一個重要的問題，就是如何在容錯移轉之後連線至 Azure VM。 設計複本 Azure VM 的網路策略時，有兩種選擇：

- **使用不同的 IP 位址**：您可以選擇針對複寫的 Azure VM 網路使用不同的 IP 位址範圍。 在此情況下，VM 會在容錯移轉之後取得新的 IP 位址，因此需要更新 DNS。
- **保留相同的 IP 位址**：您可能會想要在容錯移轉之後，將與您主要內部部署站台中相同的 IP 位址範圍用於 Azure 網路。 保留相同的 IP 位址可在容錯移轉之後減少網路相關問題，藉以簡化復原。 不過，當您複寫到 Azure 時，將需要在容錯移轉後，以新的 IP 位址位置更新路由。

## <a name="retaining-ip-addresses"></a>保留 IP 位址

Site Recovery 提供可在容錯移轉至 Azure (藉由子網路容錯移轉) 時保留固定 IP 位址的功能。

- 藉由子網路容錯移轉，特定的子網路會存在於站台 1 或站台 2 中，但絕不會同時存在於這兩個站台中。
- 為了在發生容錯移轉時維持 IP 位址空間，您需以程式設計方式為路由器基礎結構進行安排，將子網路從一個站台移到另一個站台。
- 在容錯移轉期間，子網路會隨著關聯的受保護 VM 一起移動。 主要的缺點在於，當發生失敗時，您必須移動整個子網路。


### <a name="failover-example"></a>容錯移轉範例

讓我們看看使用虛構公司 Woodgrove Bank 容錯移轉至 Azure 的範例。

- Woodgrove Bank 將商務應用程式裝載在內部部署網站中。 他們在 Azure 上裝載其行動應用程式。
- 其內部部署邊緣網路與 Azure 虛擬網路之間有 VPN 站對站連線。 因為此 VPN 連線，Azure 中的虛擬網路會顯示為內部部署網路的延伸。
- Woodgrove 想要使用 Site Recovery 將內部部署工作負載複寫到 Azure。
 - Woodgrove 具有相依於硬式編碼 IP 位址的應用程式，因此在容錯移轉至 Azure 之後，需要保留應用程式的 IP 位址。
 - 在 Azure 中執行的資源會使用 IP 位址範圍 172.16.1.0/24、172.16.2.0/24。

![子網路容錯移轉之前](./media/site-recovery-network-design/network-design7.png)

**容錯移轉之前的基礎結構**


Woodgrove 若要既能將其 VM 複寫至 Azure，同時又保留 IP 位址，就必須執行下列操作：


1. 建立 Azure 虛擬網路，以便在內部部署機器容錯移轉之後，在其中建立 Azure VM。 這應該為內部部署網路的延伸，以便讓應用程式可以順暢地容錯移轉。
2. 在 Site Recovery 中容錯移轉之前，他們會在機器屬性中指派相同的 IP 位址。 容錯移轉之後，Site Recovery 會將此位址指派給 Azure VM。
3. 執行容錯移轉並以相同的 IP 位址建立 Azure VM 之後，他們會使用 [Vnet 對 Vnet 連線](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md)來連線到網路。 此動作可以編寫指令碼。
4. 他們需要修改路由，以反映 192.168.1.0/24 現已移至 Azure。


**容錯移轉之後的基礎結構**

![子網路容錯移轉之後](./media/site-recovery-network-design/network-design9.png)

#### <a name="site-to-site-connection"></a>站對站連線

在容錯移轉之後，除了 vnet 對 vnet 連線，Woodgrove 可以設定站對站 VPN 連線：
- 當您設定站對站連線時，如果 IP 位址範圍不同於內部部署 IP 位址範圍，則在 Azure 網路中您只能將流量路由傳送到內部部署位置 (區域網路)。 這是因為 Azure 不支援延伸的子網路。 因此，如果您的內部部署環境中有子網路 192.168.1.0/24，您便無法在 Azure 網路中新增本機網路 192.168.1.0/24。 這是預期行為，因為 Azure 並不知道子網路中沒有任何作用中的 VM，也不知道子網路是僅針對災害復原而建立的。
- 若要能夠從 Azure 網路正確地路由傳送網路流量，網路和本機網路中的子網路便不得發生衝突。




## <a name="assigning-new-ip-addresses"></a>指派新的 IP 位址

這篇[部落格文章](http://azure.microsoft.com/blog/2014/09/04/networking-infrastructure-setup-for-microsoft-azure-as-a-disaster-recovery-site/)說明當您不需要在容錯移轉之後保留 IP 位址時，如何設定 Azure 網路基礎結構。 它是以應用程式描述作為開頭、研究如何在內部部署環境及 Azure 中設定網路，然後以關於執行容錯移轉的資訊作為總結。

## <a name="next-steps"></a>後續步驟
[執行容錯移轉](site-recovery-failover.md)
