---
title: 使用 Azure Site Recovery 執行從 Hyper-V VM 到次要網站的 DR 演練 | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 執行從 VMM 雲端中的 Hyper-V VM 到次要資料中心的 DR 演練。
services: site-recovery
author: ponatara
manager: abhemraj
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: ponatara
ms.openlocfilehash: 03533af27ac6fd406b4639c31c3add0015a76f45
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37918793"
---
# <a name="run-a-dr-drill-for-hyper-v-vms-to-a-secondary-site"></a>執行從 Hyper-V VM 到次要網站的 DR 演練


本文說明如何使用 [Azure Site Recovery](site-recovery-overview.md) 執行從 System Center Virtual Machine Manager V(MM) 雲端中管理的 Hyper-V VM 到次要內部部署網站的災害復原 (DR) 演練。

您可以執行測試容錯移轉來驗證您的複寫策略，以及在沒有資料遺失或停機時間的情況下執行 DR 演練。 測試容錯移轉不會影響任何正在進行的複寫或生產環境。 

## <a name="how-do-test-failovers-work"></a>測試容錯移轉如何運作？

您可以執行從主要到次要網站的測試容錯移轉。 如果您只想要確認某個 VM 可容錯移轉，您可以直接執行測試容錯移轉，而無需在次要網站上進行任何設定。 如果您想要確認應用程式容錯移轉可正常運作，您必須在次要位置設定網路功能和基礎結構。
- 您可以對單一 VM 執行測試容錯移轉，或是對[復原計劃](site-recovery-create-recovery-plans.md)執行。
- 您可以在沒有網路、連線至現有網路，或已自動建立網路的情況下執行測試容錯移轉。 以上選項的相關詳細資料提供於下表中。
    - 您可以在沒有網路的情況下執行測試容錯移轉。 如果您只是要確認某個 VM 能夠進行容錯移轉，此選項會很有用，但您將無法驗證任何網路組態。
    - 以現有的網路執行容錯移轉。 建議您不要使用生產環境網路。
    - 執行容錯移轉，並且讓 Site Recovery 自動建立測試網路。 在此情況下，Site Recovery 會自動建立網路，並在測試容錯移轉完成後加以清除。
- 您必須選取測試容錯移轉的復原點： 
    - **最近處理**：此選項會將 VM 容錯移轉至 Site Recovery 所處理的最新復原點。 此選項提供低 RTO (復原時間目標)，因為無須花費時間處理未處理的資料。
    - **最新應用程式一致**：此選項會將 VM 容錯移轉至 Site Recovery 所處理的最新應用程式一致復原點。 
    - **最新**︰此選項會先處理已傳送到 Site Recovery 服務的所有資料，先為每部 VM 建立復原點後再進行容錯移轉。 此選項會提供最低的 RPO (復原點目標)，因為在容錯移轉後建立的 VM 具有在觸發容錯移轉時複寫到 Site Recovery 的所有資料。
    - **已處理最新多個 VM**：適用於有一或多個 VM 已啟用多個 VM 一致性的復原計劃。 啟用這項設定的 VM 會容錯移轉至最新的一般多部 VM 一致復原點。 其他 VM 會容錯移轉至最新的已處理復原點。
    - **最新多部 VM 應用程式一致**：此選項適用於一或多部 VM 已啟用多部 VM 一致性的復原計劃。 屬於最新一般多部 VM 應用程式一致復原點的複寫群組容錯移轉一部分的 VM。 對於最新應用程式一致復原點進行的其他 VM 容錯移轉。
    - **自訂**：使用這個選項來將特定 VM 容錯移轉至特定復原點。



## <a name="prepare-networking"></a>準備網路功能

當您執行測試容錯移轉時，系統會要求您選取用以測試複本機器的網路設定，如下表所彙總。

**選項** | **詳細資料** 
--- | --- 
**None** | 會在複本 VM 所在的主機上建立測試 VM。 此機器不會新增至雲端，且不會連線至任何網路。<br/><br/> 您可以在機器建立後將其連線至 VM 網路。
**使用現有的** | 會在複本 VM 所在的主機上建立測試 VM。 此機器不會新增至雲端。<br/><br/>建立與您的生產網路隔離的 VM 網路。<br/><br/>如果您是使用 VLAN 網路，建議您在 VMM 中另外建立未用於生產網路的測試專用邏輯網路。 此邏輯網路是用來建立測試容錯移轉的 VM 網路。<br/><br/>邏輯網路應該至少要與所有裝載虛擬機器之 Hyper-V 伺服器的其中一個網路介面卡相關聯。<br/><br/>若使用 VLAN 邏輯網路，應該要隔離您新增至邏輯網路的網路站台。<br/><br/>如果您是使用 Windows 網路虛擬化型邏輯網路，Azure Site Recovery 會自動建立隔離的 VM 網路。 
**建立網路** | 暫時性測試網路將根據您在 [邏輯網路] 及其相關網路站台中指定的設定自動建立。<br/><br/> 容錯移轉會確認 VM 是否已建立。 |如果復原計劃使用多個 VM 網路，您應使用此選項。<br/><br/> 如果您使用的是 Windows 網路虛擬化網路，這個選項會使用複本虛擬機器網路中的相同設定 (子網路和 IP 位址集區) 自動建立 VM 網路。 在測試容錯移轉完成後，這些 VM 網路將自動清除。<br/><br/> 將會在複本虛擬機器所在的主機上建立測試 VM。 此機器不會新增至雲端。

### <a name="best-practices"></a>最佳作法

- 測試生產網路會導致生產工作停滯。 請要求您的使用者不要在進行災害復原測試時使用相關應用程式。

- 測試網路不需要符合用於測試容錯移轉的 VMM 邏輯網路類型。 但是，某些組合將不會運作：

     - 如果複本會使用 DHCP 和 VLAN 架構的隔離，則複本的 VM 網路不需要靜態 IP 位址集區。 因此，使用 Windows 網路虛擬化來測試容錯移轉將不會運作，因為沒有任何位址集區可供使用。 
        
     - 如果複本網路不使用隔離，而且測試網路使用 Windows 網路虛擬化，則測試容錯移轉將不會運作。 這是因為「未隔離」的網路沒有建立 Windows 網路虛擬化網路所需的子網路。
        
- 我們建議您不要針對測試容錯移轉使用您用來進行網路對應的網路。

- 容錯移轉之後，用來將複本虛擬機器連線至對應 VM 網路的方式，取決於 VM 網路在 VMM 主控台中的設定方式。


### <a name="vm-network-configured-with-no-isolation-or-vlan-isolation"></a>設定為未隔離或 VLAN 隔離的 VM 網路

如果在 VMM 中設定未隔離或 VLAN 隔離的 VM 網路，請注意下列事項：

- 如果 VM 網路已定義 DHCP，則複本虛擬機器將透過針對相關聯邏輯網路中網路站台所指定的設定連線到 VLAN 識別碼。 虛擬機器從可用的 DHCP 伺服器接收其 IP 位址。
- 您不需要為目標 VM 網路定義靜態 IP 位址集區。 如果 VM 網路使用靜態 IP位址集區，則複本虛擬機器將透過針對相關聯邏輯網路中網路站台所指定的設定連線到 VLAN 識別碼。
- 虛擬機器從 VM 網路所定義的集區接收其 IP 位址。 如果靜態 IP 位址集區未定義於目標 VM 網路上，則 IP 位址配置將會失敗。 在您即將用來保護與復原的來源和目標 VMM 伺服器上，建立 IP 位址集區。

### <a name="vm-network-with-windows-network-virtualization"></a>採用 Windows 網路虛擬化的 VM 網路

如果在 VMM 中設定 Windows 網路虛擬化的 VM 網路，請注意下列事項：

- 您應為目標 VM 網路定義靜態集區，而不論來源 VM 網路是設定為使用 DHCP 或靜態 IP 位址集區。 
- 如果您定義 DHCP，目標 VMM 伺服器將作為 DHCP 伺服器，並提供為目標 VM 網路定義之集區中的 IP 位址。
- 如果已定義來源伺服器使用靜態 IP 位址集區，則目標 VMM 伺服器會配置集區中的 IP 位址。 在這兩種情況下，如果未定義靜態 IP 位址集區，IP 位址配置將會失敗。



## <a name="prepare-the-infrastructure"></a>準備基礎結構

如果您只想要確認某個 VM 可容錯移轉，您可以直接執行測試容錯移轉，而不需要基礎結構。 如果您想要對測試應用程式容錯移轉執行完整的 DR 演練，則必須在次要網站準備基礎結構：

- 如果您使用現有的網路來執行測試容錯移轉，請在該網路中準備 Active Directory、DHCP 和 DNS。
- 如果您使用會自動建立 VM 網路的選項執行測試容錯移轉，您必須先將基礎結構資源新增至自動建立的網路，然後再執行測試容錯移轉。 在復原計劃中，您可以在將用於測試容錯移轉之復原計劃中的群組 1 之前新增手動步驟，以達到此目的。 然後，將基礎結構資源新增到自動建立的網路，接著再執行測試容錯移轉。


### <a name="prepare-dhcp"></a>準備 DHCP
如果參與測試容錯移轉的虛擬機器使用 DHCP，請在基於測試容錯移轉目的而建立的隔離網路中建立測試 DHCP 伺服器。


### <a name="prepare-active-directory"></a>準備 Active Directory
若要執行測試容錯移轉以進行應用程式測試，您需要在測試環境中具有 Active Directory 生產環境的複本。 如需詳細資訊，請參閱 [Active Directory 的測試容錯移轉考量](site-recovery-active-directory.md#test-failover-considerations)。

### <a name="prepare-dns"></a>準備 DNS
準備 DNS 伺服器來進行測試容錯移轉，如下所示：

* **DHCP**：如果虛擬機器使用 DHCP，就應該在測試 DHCP 伺服器上更新測試 DNS 的 IP 位址。 如果使用的是 Windows 網路虛擬化的網路類型，VMM 伺服器就會當做 DHCP 伺服器使用。 因此，應該在測試容錯移轉中更新 DNS 的 IP 位址。 在此情況下，虛擬機器會向相關的 DNS 伺服器註冊其本身。
* **靜態位址**：如果虛擬機器使用靜態 IP 位址，就應該在測試容錯移轉中更新測試 DNS 伺服器的 IP 位址。 您可能需要以測試虛擬機器的 IP 位址來更新 DNS。 您可以基於此目的來使用下列指令碼範例：

        Param(
        [string]$Zone,
        [string]$name,
        [string]$IP
        )
        $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
        $newrecord = $record.clone()
        $newrecord.RecordData[0].IPv4Address  =  $IP
        Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord



## <a name="run-a-test-failover"></a>執行測試容錯移轉

此程序說明如何針對復原方案執行測試容錯移轉。 或者，您可以在 [虛擬機器] 索引標籤上針對單一虛擬機器執行容錯移轉。

1. 選取 [復原方案]  >  *recoveryplan_name*。 按一下 [容錯移轉] **容錯移轉** > **Test 容錯移轉**中張貼意見或問題。
2. 在 [測試容錯移轉] 刀鋒視窗上，指定複本 VM 在測試容錯移轉後應如何連線至網路。
3. 在 [工作]  索引標籤上追蹤容錯移轉進度。
4. 完成容錯移轉之後，請確認 VM 已成功啟動。
5. 完成後，按一下復原方案上的 [清除測試容錯移轉]。 在 [記事] 中，記錄並儲存關於測試容錯移轉的任何觀察。 此步驟會刪除 Site Recovery 在測試容錯移轉期間所建立的任何 VM 和網路。 

![測試容錯移轉](./media/hyper-v-vmm-test-failover/TestFailover.png)
 


> [!TIP]
> 測試容錯移轉期間提供給虛擬機器的 IP 位址，與計劃性或非計劃性容錯移轉時虛擬機器收到的 IP 位址相同 (假設該 IP 位址可用於測試容錯移轉網路)。 如果在測試容錯移轉網路中無法使用相同的 IP 位址，虛擬機器會收到另一個可用於測試容錯移轉網路的 IP 位址。



### <a name="run-a-test-failover-to-a-production-network"></a>執行對生產網路的測試容錯移轉

建議您不要對您在網路對應期間指定的生產復原網站網路執行測試容錯移轉。 但是，如果您真的想驗證容錯移轉的 VM 中的端對端網路連線，請注意下列幾點︰

* 執行測試容錯移轉時，請確定主要 VM 已關機。 如果不這麼做，相同的網路中將同時有兩部虛擬機器以相同的身分識別執行。 這可能造成不良的後果。
* 當您清除測試容錯移轉 VM 時，您對測試容錯移轉虛擬機器所做的任何變更都將遺失。 這些變更不會複寫回主要 VM。
* 這種測試方式會導致生產應用程式的停機時間。 進行 DR 訓練時，要求應用程式的使用者不要使用應用程式。  


## <a name="next-steps"></a>後續步驟
成功執行 DR 演練之後，您就可以[執行完整容錯移轉](site-recovery-failover.md)。



