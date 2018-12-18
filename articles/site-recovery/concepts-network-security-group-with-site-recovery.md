---
title: 網路安全性群組搭配 Azure Site Recovery | Microsoft Docs
description: 說明如何使用網路安全性群組來搭配 Azure Site Recovery 以進行災害復原和移轉
services: site-recovery
documentationcenter: ''
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: mayg
ms.openlocfilehash: fc4beb9f664bb3be52907cb6cd746dad0aa37e64
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49352722"
---
# <a name="network-security-groups-with-azure-site-recovery"></a>網路安全性群組搭配 Azure Site Recovery

網路安全性群組可用來將網路流量限制為虛擬網路中的資源。 [網路安全性群組 (NSG)](../virtual-network/security-overview.md#network-security-groups) 包含一些安全性規則，可根據來源或目的地 IP 位址、連接埠和通訊協定允許或拒絕輸入或輸出網路流量。

在 Resource Manager 部署模式下，NSG 可與子網路或個別網路介面建立關聯。 當 NSG 與子網路相關聯時，系統會將規則套用至已連線至子網路的所有資源。 若某個子網路已有相關聯的 NSG，而您讓 NSG 也關聯至該子網路內的個別網路介面，就可以進一步限制流量。

本文說明如何使用網路安全性群組來搭配 Azure Site Recovery。

## <a name="using-network-security-groups"></a>使用網路安全性群組

個別子網路可以有零個或一個相關聯的 NSG。 個別網路介面也可以有零個或一個相關聯的 NSG。 因此，您實際上可以將某個 NSG 先關聯至子網路，再將另一個 NSG 關聯至 VM 的網路介面，來對虛擬機器施加雙重流量限制。 在此情況下，NSG 規則的應用取決於流量方向和所套用安全性規則的優先順序。

請設想具有一個虛擬機器的簡單範例，如下所示：
-   此虛擬機器放置於 **Contoso 子網路**內。
-   **Contoso 子網路**與**子網路 NSG** 相關聯。
-   VM 網路介面另外與 **VM NSG** 相關聯。

![NSG 搭配 Site Recovery](./media/concepts-network-security-group-with-site-recovery/site-recovery-with-network-security-group.png)

此範例中的輸入流量會先評估子網路 NSG。 VM NSG 接著會評估允許通過子網路 NSG 的任何流量。 程序反過來則適用於輸出流量，但會先評估 VM NSG。 子網路 NSG 接著會評估允許通過 VM NSG 的任何流量。

這可讓您進行更細微的安全性規則應用。 例如，您可以允許對子網路底下的一些應用程式 VM (例如，前端 VM) 進行輸入網際網路存取，但限制不得對其他 VM (例如，資料庫和其他後端 VM) 進行輸入網際網路存取。 在此情況下，您可以對子網路 NSG 實施較寬鬆的規則，允許網際網路流量，但藉由拒絕對 VM NSG 的存取來限制對特定 VM 的存取。 您也可以對輸出流量套用相同的規則。

在設定這類 NSG 組態時，請確定您對[安全性規則](../virtual-network/security-overview.md#security-rules)所套用的優先順序正確無誤。 系統會依照優先權順序處理規則，較低的數字會在較高的數字之前處理，因為較低的數字具有較高的優先順序。 一旦流量符合規則，處理就會停止。 因此，如果存在較低優先順序 (較高數字) 的規則具有與較高優先順序之規則相同的屬性，則不會進行處理。

您可能永遠不會注意網路安全性群組何時同時套用至網路介面和子網路。 藉由檢視網路介面的[有效安全性規則](../virtual-network/virtual-network-network-interface.md#view-effective-security-rules)，可以確認套用至網路介面的彙總規則。 您也可以使用 [Azure 網路監看員](../network-watcher/network-watcher-monitoring-overview.md)中的 [IP 流量確認](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md)功能來判斷是否允許網路介面的雙向通訊。 此工具會告訴您是否允許通訊，以及哪個網路安全性規則允許或拒絕流量。

## <a name="on-premises-to-azure-replication-with-nsg"></a>使用 NSG 進行內部部署至 Azure 的複寫

Azure Site Recovery 能夠針對內部部署的 [Hyper-V 虛擬機器](hyper-v-azure-architecture.md)、[VMware 虛擬機器](vmware-azure-architecture.md)和[實體伺服器](physical-azure-architecture.md)進行災害復原並移轉到 Azure。 針對所有內部部署到 Azure 的案例，會將複寫資料傳送到並儲存於 Azure 儲存體帳戶。 在複寫期間，您不需支付任何虛擬機器費用。 當您容錯移轉到 Azure 時，Site Recovery 會自動建立 Azure IaaS 虛擬機器。

若已在容錯移轉至 Azure 後建立 VM，則可使用 NSG 來限制虛擬網路和 VM 的網路流量。 Site Recovery 不會在容錯移轉作業進行期間建立 NSG。 建議您在起始容錯移轉之前先建立必要的 Azure NSG。 之後，您可以搭配使用自動化指令碼與 Site Recovery 強大的[復原計劃](site-recovery-create-recovery-plans.md)，在容錯移轉期間自動將 NSG 關聯至已容錯移轉的 VM。

例如，如果容錯移轉後的 VM 組態類似於上面詳述的[案例範例](concepts-network-security-group-with-site-recovery.md#using-network-security-groups)：
-   您可以在目標 Azure 區域上，建立 **Contoso VNet** 和 **Contoso 子網路**作為 DR 計劃的一部分。
-   您也可以建立及設定**子網路 NSG** 以及 **VM NSG** 作為同一個 DR 計劃的一部分。
-   **子網路 NSG** 接著可以立即與 **Contoso 子網路**建立關聯 (因為已有 NSG 和子網路可供使用)。
-   在容錯移轉期間，**VM NSG** 可以使用復原計劃來與 VM 建立關聯。

建立並設定 NSG 後，建議您執行[測試容錯移轉](site-recovery-test-failover-to-azure.md)以確認所編寫的 NSG 關聯和容錯移轉後的 VM 連線能力。

## <a name="azure-to-azure-replication-with-nsg"></a>使用 NSG 進行 Azure 到 Azure 的複寫

Azure Site Recovery 會啟用 [Azure 虛擬機器](azure-to-azure-architecture.md)的災害復原。 為 Azure VM 啟用複寫時，Site Recovery 可以在目標區域上建立複本虛擬網路 (包括子網路和閘道子網路)，並在來源和目標虛擬網路之間建立必要的對應。 您也可以預先建立目標端的網路和子網路，並且在啟用複寫時使用相同的網路和子網路。 Site Recovery 不會在[容錯移轉](azure-to-azure-tutorial-failover-failback.md)之前，於目標 Azure 區域上建立任何 VM。

對於 Azure VM 複寫，請確定來源 Azure 區域上的 NSG 規則允許使用[輸出連線能力](azure-to-azure-about-networking.md#outbound-connectivity-for-ip-address-ranges)來產生複寫流量。 您也可以透過此 [NSG 組態範例](azure-to-azure-about-networking.md#example-nsg-configuration)來測試並驗證這些必要規則。

Site Recovery 不會在容錯移轉作業進行期間建立或複寫 NSG。 建議您在起始容錯移轉之前先在目標 Azure 區域上建立必要的 NSG。 之後，您可以搭配使用自動化指令碼與 Site Recovery 強大的[復原計劃](site-recovery-create-recovery-plans.md)，在容錯移轉期間自動將 NSG 關聯至已容錯移轉的 VM。

請設想稍早所述的[案例範例](concepts-network-security-group-with-site-recovery.md#using-network-security-groups)：
-   若已為 VM 啟用複寫，Site Recovery 就可以在目標 Azure 區域上建立 **Contoso VNet** 和 **Contoso 子網路**的複本。
-   您也可以在目標 Azure 區域上建立所需的**子網路 NSG** 和 **VM NSG** (例如，分別名為**目標子網路 NSG** 和**目標 VM NSG**) 複本，而能在目標區域上建立所需的其他規則。
-   **目標子網路 NSG** 接著可以立即與目標區域子網路建立關聯 (因為已有 NSG 和子網路可供使用)。
-   在容錯移轉期間，**目標 VM NSG** 可以使用復原計劃來與 VM 建立關聯。

建立並設定 NSG 後，建議您執行[測試容錯移轉](azure-to-azure-tutorial-dr-drill.md)以確認所編寫的 NSG 關聯和容錯移轉後的 VM 連線能力。

## <a name="next-steps"></a>後續步驟
-   深入了解[網路安全性群組](../virtual-network/security-overview.md#network-security-groups)。
-   深入了解 NSG [安全性規則](../virtual-network/security-overview.md#security-rules)。
-   深入了解 NSG 的[有效安全性規則](../virtual-network/diagnose-network-traffic-filter-problem.md)。
-   深入了解自動執行應用程式容錯移轉的[復原計劃](site-recovery-create-recovery-plans.md)。
