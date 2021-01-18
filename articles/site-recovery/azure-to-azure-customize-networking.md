---
title: 自訂容錯移轉 VM 的網路設定 |Microsoft Docs
description: 概述如何使用 Azure Site Recovery 在 Azure Vm 的複寫中自訂容錯移轉 VM 的網路設定。
services: site-recovery
author: sideeksh
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 10/21/2019
ms.author: sideeksh
ms.openlocfilehash: f63021275574e294fa372357d6e62724f5efe0f7
ms.sourcegitcommit: 949c0a2b832d55491e03531f4ced15405a7e92e3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/18/2021
ms.locfileid: "98541185"
---
# <a name="customize-networking-configurations-of-the-target-azure-vm"></a>自訂目標 Azure 虛擬機器的網路設定

本文提供在使用 [Azure Site Recovery](site-recovery-overview.md)將 azure vm 從一個區域複寫及復原到另一個區域時，自訂目標 Azure 虛擬機器上的網路設定 (VM) 的指引。

## <a name="before-you-start"></a>在您開始使用 Intune 之前

了解 Site Recovery 如何針對[這種情況](azure-to-azure-architecture.md)提供災害復原。

## <a name="supported-networking-resources"></a>支援的網路資源

您可以在複寫 Azure Vm 時，為容錯移轉 VM 提供下列金鑰資源設定：

- [內部負載平衡器](../load-balancer/load-balancer-overview.md)
- [公用 IP](../virtual-network/public-ip-addresses.md)
- 子網和 NIC 的[網路安全性群組](../virtual-network/manage-network-security-group.md)

## <a name="prerequisites"></a>先決條件

- 確定您事先規劃復原端設定。
- 事先建立網路資源。 提供它做為輸入，讓 Azure Site Recovery 服務可以接受這些設定，並確定容錯移轉 VM 遵守這些設定。

## <a name="customize-failover-and-test-failover-networking-configurations"></a>自訂容錯移轉和測試容錯移轉網路設定

1. 移至 [已複寫的 **專案**]。 
2. 選取所需的 Azure VM。
3. 選取 [ **計算和網路** ]，然後選取 [ **編輯**]。 請注意，NIC 設定包含來源的對應資源。 

     ![自訂容錯移轉網路設定](media/azure-to-azure-customize-networking/edit-networking-properties.png)

4. 選取測試容錯移轉虛擬網路。
5. 在您想要設定的 **NIC 附近，** 選取 [容錯移轉網路]。 在開啟的下一個分頁中，在測試容錯移轉和容錯移轉位置中選取對應的預先建立資源。

    ![編輯 NIC 設定](media/azure-to-azure-customize-networking/nic-drilldown.png) 

6. 選取 [確定]。

Site Recovery 現在會接受這些設定，並確保容錯移轉上的 VM 會透過對應的 NIC 連線到選取的資源。

當您透過復原方案觸發測試容錯移轉時，它一律會詢問 Azure 虛擬網路。 針對未預先設定測試容錯移轉設定的機器，此虛擬網路將用於測試容錯移轉。

## <a name="troubleshooting"></a>疑難排解

### <a name="unable-to-view-or-select-a-resource"></a>無法查看或選取資源

如果您無法選取或查看網路資源，請完成下列檢查和條件：

- 只有當來源 VM 具有對應的輸入時，才會啟用網路資源的目標欄位。 這是以嚴重損壞修復案例的原則為基礎，您需要來源的精確或相應縮小版本。
- 針對每個網路資源，會在下拉式清單中套用某些篩選準則，以確保容錯移轉 VM 可以將自己附加至選取的資源，並維護容錯移轉的可靠性。 這些篩選器會以您設定來源 VM 時已驗證的相同網路狀況為基礎。

內部負載平衡器驗證：

- 負載平衡器和目標 VM 的訂用帳戶和區域應該相同。
- 與內部負載平衡器相關聯的虛擬網路，以及目標 VM 的虛擬網路應該相同。
- 目標 VM 的公用 IP SKU 和內部負載平衡器的 SKU 應該相同。
- 如果目標 VM 設定為放置於可用性區域，請檢查負載平衡器是否為區域多餘或任何可用性區域的一部分。  (基本 SKU 負載平衡器不支援區域，在此情況下不會顯示在下拉式清單中。 ) 
- 確定內部負載平衡器具有預先建立的後端集區和前端設定。

公用 IP 位址：

- 公用 IP 和目標 VM 的訂用帳戶和區域應該相同。
- 目標 VM 的公用 IP SKU 和內部負載平衡器的 SKU 應該相同。

網路安全性群組：
- 網路安全性群組和目標 VM 的訂用帳戶和區域應該相同。


> [!WARNING]
> 如果目標 VM 與可用性設定組相關聯，則您必須將相同 SKU 的公用 IP 和內部負載平衡器與可用性設定組中其他 VM 的公用 IP 和內部負載平衡器的公用 IP 和內部負載平衡器產生關聯。 如果您沒有這麼做，則容錯移轉可能不會成功。
