---
title: 使用 Azure Site Recovery 複寫多層式 Citrix XenDesktop 和 XenApp 部署 | Microsoft Docs
description: 本文說明如何使用 Azure Site Recovery 保護和復原 Citrix XenDesktop 和 XenApp 部署。
services: site-recovery
documentationcenter: ''
author: ponatara
manager: abhemraj
editor: ''
ms.assetid: 9126f5e8-e9ed-4c31-b6b4-bf969c12c184
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2018
ms.author: ponatara
ms.openlocfilehash: 45d366842416ddfa7b0153a1d075ee6de58e45a1
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39213628"
---
# <a name="replicate-a-multi-tier-citrix-xenapp-and-xendesktop-deployment-using-azure-site-recovery"></a>使用 Azure Site Recovery 複寫多層式 Citrix XenApp 和 XenDesktop 部署

## <a name="overview"></a>概觀

Citrix XenDesktop 是桌面虛擬化解決方案，能夠為任何地方的任何使用者提供桌面與應用程式的隨需服務。 藉由 FlexCast 傳遞技術，XenDesktop 能夠以快速且安全的方式為使用者傳遞應用程式和桌面。
現在，Citrix XenApp 不提供任何災害復原功能。

理想的災害復原解決方案應該允許上述複雜應用程式架構的相關復原計劃模型化，並且也能夠新增自訂步驟以處理各層之間的應用程式對應，因此在導致較低 RTO 的災難發生時能提供單鍵擷取方案。

對於為 Hyper-V 和 VMware vSphere 平台上的內部部署 Citrix XenApp 部署建置災害復原方案，本文提供逐步指引。 本文同時也會說明如何使用復原計劃、支援的組態和必要條件，執行測試容錯移轉 (災害復原訓練) 和未計劃的 Azure 容錯移轉。


## <a name="prerequisites"></a>必要條件

開始之前，請確定您瞭解下列項目︰

1. [將虛擬機器複寫至 Azure](site-recovery-vmware-to-azure.md)
1. 如何[設計修復網路](site-recovery-network-design.md)
1. [執行測試容錯移轉至 Azure](site-recovery-test-failover-to-azure.md)
1. [執行容錯移轉至 Azure](site-recovery-failover.md)
1. 如何[複寫網域控制站](site-recovery-active-directory.md)
1. 如何[複寫 SQL Server](site-recovery-sql.md)

## <a name="deployment-patterns"></a>部署模式

Citrix XenApp 和 XenDesktop 伺服器陣列通常有下列部署模式：

**部署模式**

AD DNS 伺服器、SQL 資料庫伺服器、Citrix 傳遞控制站、StoreFront 伺服器，XenApp Master (VDA)、Citrix XenApp 授權伺服器的 Citrix XenApp 和 XenDesktop 部署

![部署模式 1](./media/site-recovery-citrix-xenapp-and-xendesktop/citrix-deployment.png)


## <a name="site-recovery-support"></a>Site Recovery 支援

在本文中，vSphere 6.0 / System Center VMM 2012 R2 所管理 VMware 虛擬機器上的 Citrix 部署是用來安裝 DR。

### <a name="source-and-target"></a>來源與目標

**案例** | **至次要網站** | **至 Azure**
--- | --- | ---
**Hyper-V** | 不在範圍中 | 是
**VMware** | 不在範圍中 | 是
**實體伺服器** | 不在範圍中 | 是

### <a name="versions"></a>版本
客戶可以部署 XenApp 元件成為 Hyper-V 或 VMware 上執行的虛擬機器，或成為實體伺服器。 Azure Site Recovery 可以保護 Azure 的實體與虛擬部署。
由於 Azure 支援 XenApp 7.7 或更新版本，因此只有這些版本的部署才能容錯移轉到 Azure 進行災害復原或移轉。

### <a name="things-to-keep-in-mind"></a>要牢記在心的事項

1. 支援使用伺服器作業系統機器保護和復原內部部署來傳遞 XenApp 發佈的應用程式和 XenApp 發佈的桌面。

2. 不支援使用桌面作業系統機器保護和復原內部部署來傳遞虛擬桌面的桌面 VDI，包括 Windows 10 用戶端。 這是因為 ASR 不支援使用復原使用桌面作業系統的機器。  此外，某些用戶端虛擬桌面作業系統 (例如 Windows 7) 在 Azure 中尚不支援授權。 [深入了解](https://azure.microsoft.com/pricing/licensing-faq/)如何在 Azure 中進行用戶端/伺服器桌面的授權。

3.  Azure Site Recovery 無法複寫和保護現有的內部部署 MCS 或 PV 複製品。
您需要使用傳遞控制站的 Azure RM 佈建重新建立這些複製品。

4. 無法使用 Azure Site Recovery 保護 NetScaler，因為 NetScaler 是以 FreeBSD 為基礎，而且 Azure Site Recovery 不支援 FreeBSD 作業系統的保護。 容錯移轉到 Azure 之後，您需要從 Azure Market Place 部署和設定新的 NetScaler 裝置。


## <a name="replicating-virtual-machines"></a>複寫虛擬機器

需要保護 Citrix XenApp 部署的下列元件，才能啟用複寫和復原。

* AD DNS 伺服器的保護
* SQL 資料庫伺服器的保護
* Citrix 傳遞控制站的保護
* StoreFront 伺服器的保護。
* XenApp Master (VDA) 的保護
* Citrix XenApp 授權伺服器的保護


**AD DNS 伺服器複寫**

關於複寫和設定 Azure 中的網域控制站所需的指引，請參閱[以 Azure Site Recovery 保護 Active Directory 和 DNS](site-recovery-active-directory.md)。

**SQL Database 伺服器複寫**

如需保護 SQL Server 的建議選項有關的詳細技術指引，請參閱[使用 SQL Server 災害復原與 Azure Site Recovery 保護 SQL Server](site-recovery-sql.md)。

請依照[本指引](site-recovery-vmware-to-azure.md)開始將其他元件虛擬機器複寫至 Azure。

![XenApp 元件的保護](./media/site-recovery-citrix-xenapp-and-xendesktop/citrix-enablereplication.png)

**計算和網路設定**

機器受到保護之後 (在 [複寫項目] 下顯示為「受保護」狀態)，需要設定計算和網路設定。
在 [計算和網路] > [計算屬性] 中，您可以指定 Azure VM 名稱和目標大小。
視需要修改名稱以符合 Azure 需求。 您也可以檢視和加入目標網路、子網路的相關資訊，以及將指派給 Azure VM 的 IP 位址。

請注意：

* 您可以設定目標 IP 位址。 如果您未提供地址，則容錯移轉的機器會使用 DHCP。 如果您設定的位址在容錯移轉時無法使用，則容錯移轉會失敗。 如果位址可用於測試容錯移轉網路，則相同的目標 IP 位址可用於測試容錯移轉。

* 對於 AD/DNS 伺服器，保留內部部署位址可讓您為 Azure 虛擬網路指定與 DNS 伺服器相同的位址。

網路介面卡的數目會視您指定給目標虛擬機器的大小而有所不同，如下所示：

*   如果來源電腦上的網路介面卡數目小於或等於針對目標機器大小所允許的介面卡數目，則目標將具備與來源相同的介面卡數目。
*   如果來源虛擬機器的介面卡數目超過針對目標大小所允許的數目，則將使用目標大小的最大值。
* 例如，如果來源機器具有兩張網路介面卡，而目標機器大小支援四張，則目標機器將會有兩張介面卡。 如果來源機器具有兩張介面卡，但支援的目標大小僅支援一張，則目標機器將只會有一張介面卡。
*   如果虛擬機器有多張網路介面卡，則全部會連接至相同的網路。
*   如果虛擬機器具有多個網路介面卡，則清單中顯示的第一個介面卡會變成 Azure 虛擬機器中的「預設」網路介面卡。


## <a name="creating-a-recovery-plan"></a>建立復原計劃

啟用 XenApp 元件 VM 的複寫之後，下一個步驟是建立復原計畫。
復原計畫群組會將有類似需求的虛擬機器分組，以便進行容錯移轉和復原。  

**建立復原計畫的步驟**

1. 在復原計畫中加入 XenApp 元件虛擬機器。
2. 按一下 [復原計畫] -> [+ 復原計畫]。 為復原計畫取個直覺式名稱。
3. 對於 VMware 虛擬機器：選取 VMware 處理序伺服器做為來源，選取 Microsoft Azure 做為目標，並選取資源管理員做為部署模型，然後按一下 [選取項目]。
4. 對於 Hyper-V 虛擬機器：選取 VMM 伺服器做為來源，選取 Microsoft Azure 做為目標，選取資源管理員做為部署模型，並按一下 [選取項目]，然後選取 XenApp 部署 VM。

### <a name="adding-virtual-machines-to-failover-groups"></a>將虛擬機器新增至容錯移轉群組

可以自訂復原計畫，以新增特定啟動順序、指令碼或手動動作的容錯移轉群組。 您必須將下列群組加入至復原計畫。

1. 容錯移轉群組 1：AD DNS
2. 容錯移轉群組 2：SQL Server VM
2. 容錯移轉群組 3：VDA Master Image VM
3. 容錯移轉群組 4：傳遞控制站和 StoreFront 伺服器 VM


### <a name="adding-scripts-to-the-recovery-plan"></a>將指令碼新增至復原計畫

可以在復原計畫中的特定群組之前或之後執行指令碼。 也可以在容錯移轉期間加入並執行手動動作。

自訂的復原計畫如下所示：

1. 容錯移轉群組 1：AD DNS
2. 容錯移轉群組 2：SQL Server VM
3. 容錯移轉群組 3：VDA Master Image VM

   >[!NOTE]     
   >包含手動或指令碼動作的步驟 4、6 和 7 僅適用於有 MCS/PV 目錄的內部部署 XenApp 環境。

4. 群組 3 手動或指令碼動作：關閉主要 VDA VM。主要 VDA VM 在容錯移轉至 Azure 時將處於執行中狀態。 若要使用 Azure ARM 裝載建立新的 MCS 目錄，主要 VDA VM 需要處於已停止 (已解除配置) 狀態。 從 Azure 入口網站關閉 VM。

5. 容錯移轉群組 4：傳遞控制站和 StoreFront 伺服器 VM
6. 群組 3 手動或指令碼動作 1：

    ***新增 Azure RM 主機連線***

    在傳遞控制站機器中建立 Azure ARM 主機連線，以便在 Azure 中佈建新的 MCS 目錄。 請遵循本[文章](https://www.citrix.com/blogs/2016/07/21/connecting-to-azure-resource-manager-in-xenapp-xendesktop/)中所述的步驟。

7. 群組 3 手動或指令碼動作 2：

    ***在 Azure 中重新建立 MCS 目錄***

    主要網站上的現有 MCS 或 PVS 複製品不會複寫到 Azure。 您需要使用複寫的主要 VDA 和 Azure ARM 佈建，從傳遞控制站重新建立這些複製品。請遵循本[文章](https://www.citrix.com/blogs/2016/09/12/using-xenapp-xendesktop-in-azure-resource-manager/)中所述的步驟在 Azure 中建立 MCS 目錄。

![XenApp 元件的復原計畫](./media/site-recovery-citrix-xenapp-and-xendesktop/citrix-recoveryplan.png)


   >[!NOTE]
   >您可以在[位置](https://github.com/Azure/azure-quickstart-templates/tree/master/asr-automation-recovery/scripts)使用指令碼，以容錯移轉虛擬機器的新 IP 更新 DNS，或在有需要時連接容錯移轉機器上的負載平衡器。


## <a name="doing-a-test-failover"></a>執行測試容錯移轉

請依照[本指引](site-recovery-test-failover-to-azure.md)來執行測試容錯移轉。

![復原方案](./media/site-recovery-citrix-xenapp-and-xendesktop/citrix-tfo.png)


## <a name="doing-a-failover"></a>執行容錯移轉

當您在進行容錯移轉時，請依照[本指引](site-recovery-failover.md)。

## <a name="next-steps"></a>後續步驟

您可以在這份白皮書中[深入了解](https://aka.ms/citrix-xenapp-xendesktop-with-asr)複寫 Citrix XenApp 和 XenDesktop 部署。 參閱使用 Site Recovery [複寫其他應用程式](site-recovery-workload.md)的指引。
