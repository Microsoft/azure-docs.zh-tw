---
title: 準備內部部署 Hyper-V 伺服器以進行 Hyper-V VM 至 Azure 的災害復原 | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 服務，準備不由 System Center VMM 管理的內部部署 Hyper-V VM 來進行至 Azure 的災害復原。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 09/12/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: f1899817ee2d0efec4ab561a64f24e49cb173c29
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44720764"
---
# <a name="prepare-on-premises-hyper-v-servers-for-disaster-recovery-to-azure"></a>準備內部部署 Hyper-V 伺服器以進行至 Azure 的災害復原

此教學課程說明如何基於災害復原的目的，在您想要將 Hyper-V VM 複寫至 Azure 時，準備內部部署 Hyper-V 基礎結構。 Hyper-V 主機可由 System Center Virtual Machine Manager (VMM) 管理，但並非必要。  在此教學課程中，您將了解如何：

> [!div class="checklist"]
> * 檢閱 Hyper-V 需求和 VMM 需求 (如果適用)。
> * 準備 VMM (如果適用)
> * 確認對 Azure 位置的網際網路存取
> * 準備 VM，讓您可以在容錯移轉至 Azure 之後存取它們

這是系列中的第二個教學課程。 確定您已如上一個教學課程中所述[設定 Azure 元件](tutorial-prepare-azure.md)。



## <a name="review-requirements-and-prerequisites"></a>檢閱需求和先決條件

確定 Hyper-V 主機和 VM 符合下列需求。

1. [確認](hyper-v-azure-support-matrix.md#on-premises-servers)內部部署伺服器的需求。
2. [檢查](hyper-v-azure-support-matrix.md#replicated-vms)您要複寫至 Azure 的 HYPER-V VM 有何需求。
3. 請檢查 HYPER-V 主機[網路](hyper-v-azure-support-matrix.md#hyper-v-network-configuration)，以及內部部署 Hyper-V 主機的主機和客體[儲存體](hyper-v-azure-support-matrix.md#hyper-v-host-storage)支援。
4. 檢查在容錯移轉之後，[Azure 網路](hyper-v-azure-support-matrix.md#azure-vm-network-configuration-after-failover)、[儲存體](hyper-v-azure-support-matrix.md#azure-storage)和[計算](hyper-v-azure-support-matrix.md#azure-compute-features)支援的項目。
5. 您複寫到 Azure 的內部部署 VM 必須符合 [Azure VM 需求](hyper-v-azure-support-matrix.md#azure-vm-requirements)。


## <a name="prepare-vmm-optional"></a>準備 VMM (選擇性)

如果 Hyper-V 主機由 VMM 管理，您需要準備內部部署 VMM 伺服器。 

- 請確定 VMM 伺服器至少具有一個雲端，以及一或多個主機群組。 執行 VM 的 Hyper-V 主機應該位於雲端。
- 準備 VMM 伺服器以進行網路對應。

### <a name="prepare-vmm-for-network-mapping"></a>準備 VMM 以進行網路對應

如果您使用 VMM，[網路對應](site-recovery-network-mapping.md)會在內部部署 VMM VM 網路與 Azure 虛擬網路之間進行對應。 對應可確保容錯移轉之後建立的 Azure VM 會連線到正確的網路。

準備 VMM 以進行網路對應，如下所示：

1. 請確定 Hyper-V 主機所在雲端有相關聯的 [VMM 邏輯網路](https://docs.microsoft.com/system-center/vmm/network-logical)。
2. 請確定您有 [VM 網路](https://docs.microsoft.com/system-center/vmm/network-virtual)連結至邏輯網路。
3. 在 VMM 中，將 VM 連線至 VM 網路。

## <a name="verify-internet-access"></a>確認網際網路存取

1. 對此教學課程來說，最簡單的設定是讓 Hyper-V 主機和 VMM 伺服器可直接存取網際網路，而不需使用 Proxy。 
2. 確定 Hyper-V 主機和 VMM 伺服器 (如果相關) 可以存取以下必要的 URL。   
3. 如果您要依據 IP 位址來控制存取，請確定：
    - 以 IP 位址為基礎的防火牆規則可以連線至 [Azure Datacenter IP 範圍](https://www.microsoft.com/download/confirmation.aspx?id=41653)，以及 HTTPS (443) 連接埠。
    - 允許訂用帳戶之 Azure 區域的 IP 位址範圍。
    
### <a name="required-urls"></a>必要的 URL


[!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]


## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

在容錯移轉案例期間，您可能想要連線到已複寫的內部部署網路。

若要在容錯移轉後使用 RDP 連線到 Windows VM，請依照下列方式允許存取權：

1. 若要透過網際網路存取，請在內部部署 VM 上啟用 RDP，再進行容錯移轉。 確定已針對 [公用] 設定檔新增 TCP 和 UDP 規則，且在 [Windows 防火牆] > [允許的應用程式] 中已針對所有設定檔允許 RDP。
2. 若要透過站對站 VPN 存取，請在內部部署機器上啟用 RDP。 您應該在 [Windows 防火牆] -> [允許的應用程式與功能] 中，針對 [網域] 和 [私人] 網路允許 RDP。
   確認作業系統的 SAN 原則已設為 [OnlineAll]。 [深入了解](https://support.microsoft.com/kb/3031135)。 觸發容錯移轉時，VM 上不應該有任何擱置的 Windows 更新。 如果有，在更新完成之前，您將無法登入虛擬機器。
3. 在容錯移轉之後，於 Windows Azure VM 上，勾選 [開機診斷] 以檢視 VM 的螢幕擷取畫面。 如果您無法連線，請檢查 VM 是否正在執行，並檢閱這些[疑難排解祕訣](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。

容錯移轉之後，您可以使用與複寫的內部部署 VM 相同的 IP 位址或不同 IP 位址來存取 Azure VM。 [深入了解](concepts-on-premises-to-azure-networking.md)如何設定 IP 位址以進行容錯移轉。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [設定 Hyper-V VM 至 Azure 的災害復原](tutorial-hyper-v-to-azure.md)
> [在 VMM 雲端設定 Hyper-V VM 至 Azure 的災害復原](tutorial-hyper-v-vmm-to-azure.md)
