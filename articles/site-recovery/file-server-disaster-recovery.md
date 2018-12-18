---
title: 使用 Azure Site Recovery 保護檔案伺服器
description: 本文說明如何使用 Azure Site Recovery 保護檔案伺服器
services: site-recovery
author: rajani-janaki-ram
manager: gauravd
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2018
ms.author: rajanaki
ms.custom: mvc
ms.openlocfilehash: 0b6d5dccbce30c55e259e4bb3f8ae4194a02b646
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37916878"
---
# <a name="protect-a-file-server-by-using-azure-site-recovery"></a>使用 Azure Site Recovery 保護檔案伺服器 

[Azure Site Recovery](site-recovery-overview.md) 藉由確保您的商務應用程式可在計劃性與非計劃性中斷期間持續啟動並執行，來提供商務持續性和災害復原 (BCDR) 策略。 Site Recovery 會管理並協調內部部署機器和 Azure 虛擬機器 (VM) 的災害復原。 災害復原包括各種工作負載的複寫、容錯移轉和復原。

本文說明如何使用 Site Recovery 保護檔案伺服器，並提供因應不同環境的其他建議。 

- [複寫 Azure IaaS 檔案伺服器機器](#disaster-recovery-recommendation-for-azure-iaas-virtual-machines)
- [使用 Site Recovery 複寫內部部署檔案伺服器](#replicate-an-on-premises-file-server-by-using-site-recovery)

## <a name="file-server-architecture"></a>檔案伺服器架構
開放分散式檔案共用系統的目的是要提供一種環境，讓一群地理位置分散的使用者可以共同作業以有效率地處理檔案，並確保能夠達到檔案完整性的需求。 典型的內部部署檔案伺服器生態系統，支援大量並行使用者和大量內容項目，使用分散式檔案系統複寫 (DFSR) 來進行複寫排程和頻寬節流。 

DFSR 使用名為「遠端差異壓縮 (RDC)」的壓縮演算法，可用來在頻寬有限的網路上有效率地更新檔案。 它可在檔案中的資料遭到插入、移除與重新排列時偵測出來。 啟用的 DFSR 只會在檔案更新時複寫已變更的檔案區塊。 另外還有檔案伺服器環境，在非尖峰時間才進行每日備份，可因應發生災害時的需求。 此時不會實作 DFSR。

下圖說明實作 DFSR 的檔案伺服器環境。
                
![DFSR 架構](media/site-recovery-file-server/dfsr-architecture.JPG)

在上圖中，有多個名為成員的檔案伺服器主動參與整個複寫群組中的檔案複寫。 即使一個成員離線，向其中一個成員傳送要求的所有用戶端還是可以取得複寫資料夾的內容。

## <a name="disaster-recovery-recommendations-for-file-servers"></a>檔案伺服器的災害復原建議

* **使用 Site Recovery 複寫檔案伺服器**：可使用 Site Recovery 將檔案伺服器複寫至 Azure。 當一或多個內部部署檔案伺服器無法存取時，可以在 Azure 中執行復原 VM。 後續只要有站對站的 VPN 連線能力，且在 Azure 中設定了 Active Directory，VM 就可以處理內部部署用戶端的要求。 您可以在設定了 DFSR 的環境或沒有 DFSR 的簡單檔案伺服器環境中使用此方法。 

* **將 DFSR 延伸至 Azure IaaS VM**：在實作 DFSR 的叢集檔案伺服器環境中，您可以將內部部署 DFSR 延伸至 Azure。 然後啟用 Azure VM，以執行檔案伺服器角色。 

    * 在處理好站對站 VPN 連線能力與 Active Directory 的相依性，且 DFSR 已準備就緒後，當一或多個內部部署的檔案伺服器無法存取時，用戶端將可連線至可處理要求的 Azure VM。

    * 如果您的 VM 所使用的組態不受 Site Recovery 的支援，您可以使用此方法。 範例之一是常用於檔案伺服器環境中的共用叢集磁碟。 DFSR 也適用於具中等變換率的低頻寬環境。 您必須考量讓 Azure VM 隨時啟動並執行的額外成本。 

* **使用 Azure 檔案同步複寫您的檔案**：如果您想要使用雲端，或已經使用 Azure VM，您可以使用「Azure 檔案同步」。「Azure 檔案同步」可讓您同步雲端中受到完整管理、並且可透過業界標準[伺服器訊息區](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx) (SMB) 通訊協定來存取的檔案共用。 然後 Windows、Linux 和 macOS 的雲端部署或內部部署就可同時掛接 Azure 檔案共用。 

下圖可協助您決定檔案伺服器環境所適用的策略。

![決策樹](media/site-recovery-file-server/decisiontree.png)


### <a name="factors-to-consider-in-your-decisions-about-disaster-recovery-to-azure"></a>您在 Azure 的災害復原相關決策中所應考量的因素

|環境  |建議  |考慮事項 |
|---------|---------|---------|
|有或沒有 DFSR 的檔案伺服器環境|   [使用 Site Recovery 進行複寫](#replicate-an-on-premises-file-server-by-using-site-recovery)   |    Site Recovery 不支援共用磁碟叢集或網路連接儲存裝置 (NAS)。 如果您的環境使用這些組態，請在適當時使用任何其他方法。 <br> Site Recovery 不支援 SMB 3.0。 只有在對檔案所做的變更已在檔案的原始位置中更新時，複寫的 VM 才會納入變更。
|有 DFSR 的檔案伺服器環境     |  [將 DFSR 延伸到 Azure IaaS 虛擬機器](#extend-dfsr-to-an-azure-iaas-virtual-machine)  |      DFSR 適用於頻寬極端變換的環境。 要使用此方法，Azure VM 必須隨時處於啟動並執行中的狀態。 您必須在規劃時計算 VM 的成本。         |
|Azure IaaS VM     |     [檔案同步](#use-azure-file-sync-service-to-replicate-your-files)   |     如果您在災害復原案例中使用檔案同步，在容錯移轉期間，您必須手動執行動作，以確保用戶端機器可以透明的方式存取檔案共用。 要使用「檔案同步」，必須從用戶端機器開啟連接埠 445。     |


### <a name="site-recovery-support"></a>Site Recovery 支援
由於 Site Recovery 複寫無法以應用程式驗證，下列建議應該適用於下列案例。
| 來源    |至次要網站    |至 Azure
|---------|---------|---------|
|Azure| -|yes|
|Hyper-V|   yes |yes
|VMware |yes|   yes
|實體伺服器|   yes |yes
 

> [!IMPORTANT]
> 在您進行以下三種方法的任何一種之前，請先確定已處理下列相依性。

**站對站連線能力**：必須在內部部署網站與 Azure 網路之間建立直接連線，以允許伺服器之間的通訊。 請使用安全的站對站 VPN 連線，連線到作為災害復原網站的 Azure 虛擬網路。 如需詳細資訊，請參閱[在內部部署網站與 Azure 網路之間建立站對站 VPN 連線](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal)。

**Active Directory**：DFSR 依存於 Active Directory。 這表示，具有本機網域控制站的 Active Directory 樹系會延伸至 Azure 中的災害復原網站。 即使您未使用 DFSR，如果需要授與預定使用者的存取權或加以驗證，您就必須執行這些步驟。 如需詳細資訊，請參閱[將內部部署 Active Directory 延伸至 Azure](https://docs.microsoft.com/azure/site-recovery/site-recovery-active-directory)。

## <a name="disaster-recovery-recommendation-for-azure-iaas-virtual-machines"></a>Azure IaaS 虛擬機器的災害復原建議

如果您要設定及管理在 Azure IaaS VM 上裝載之檔案伺服器的災害復原，您可以根據是否要移至 [Azure 檔案](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)，在兩個選項當中作選擇：

* [使用檔案同步](#use-file-sync-to-replicate-files-hosted-on-an-iaas-virtual-machine)
* [使用 Site Recovery](#replicate-an-iaas-file-server-virtual-machine-by-using-site-recovery)

## <a name="use-file-sync-to-replicate-files-hosted-on-an-iaas-virtual-machine"></a>使用檔案同步服務複寫裝載在 IaaS 虛擬機器上的檔案

Azure 檔案服務可用來完全取代或補充傳統內部部署檔案伺服器或 NAS 裝置。 Azure 檔案共用也可透過檔案同步複寫至 Windows 伺服器 (在內部部署環境或雲端)，從而在資料的使用位置提高效能和進行分散式快取。 下列步驟說明與傳統檔案伺服器執行相同功能的 Azure VM 的災害復原建議：
* 使用 Site Recovery 保護機器。 請遵循[將 Azure VM 複寫到另一個 Azure 區域](azure-to-azure-quickstart.md)中的步驟。
* 使用檔案同步將作為檔案伺服器的 VM 中的檔案複寫到雲端。
* 使用 Site Recovery 的[復原計畫](site-recovery-create-recovery-plans.md)功能新增指令碼，以[掛接 Azure 檔案共用](https://docs.microsoft.com/azure/storage/files/storage-how-to-use-files-windows)並存取虛擬機器中的共用。

下列各節將簡短說明如何使用檔案同步：

1. [在 Azure 中建立儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-create-storage-account?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。 如果您為儲存體帳戶選擇了讀取權限異地備援儲存體，則會獲得在發生災害時從次要區域讀取資料的權限。 如需詳細資訊，請參閱 [Azure 檔案共用災害復原策略](https://docs.microsoft.com/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。
2. [建立檔案共用](https://docs.microsoft.com/azure/storage/files/storage-how-to-create-file-share)。
3. 在 Azure 檔案伺服器上[啟動檔案同步](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide)。
4. 建立同步群組。 同步群組內的端點會與彼此保持同步。 同步群組必須包含至少一個雲端端點，代表 Azure 檔案共用。 同步群組也必須包含一個伺服器端點，代表 Windows 伺服器上的路徑。
5. 您的檔案此時會在 Azure 檔案共用和內部部署伺服器之間保持同步。
6. 您的內部部署環境中發生災害時，您可以使用[復原計畫](site-recovery-create-recovery-plans.md)執行容錯移轉。 新增指令碼，以[掛接 Azure 檔案共用](https://docs.microsoft.com/azure/storage/files/storage-how-to-use-files-windows)並存取虛擬機器中的共用。

### <a name="replicate-an-iaas-file-server-virtual-machine-by-using-site-recovery"></a>使用 Site Recovery 複寫 IaaS 檔案伺服器虛擬機器

如果您有會存取 IaaS 檔案伺服器虛擬機器的內部部署用戶端，請執行下列所有步驟。 否則，請跳到步驟 3。

1. 在內部部署網站與 Azure 網路之間建立站對站 VPN 連線。
2. 延伸內部部署 Active Directory。
3. 為 IaaS 檔案伺服器機器[設定災害復原](azure-to-azure-tutorial-enable-replication.md)到次要地區。


若想進一步了解復原到次要區域的災害復原，請參閱[這篇文章](concepts-azure-to-azure-architecture.md)。


## <a name="replicate-an-on-premises-file-server-by-using-site-recovery"></a>使用 Site Recovery 複寫內部部署檔案伺服器

下列步驟說明 VMware VM 的複寫。 如需複寫 HYPER-V VM 的步驟，請參閱[本教學課程](tutorial-hyper-v-to-azure.md)。

1. [準備 Azure 資源](tutorial-prepare-azure.md)以進行內部部署機器的複寫。
2. 在內部部署網站與 Azure 網路之間建立站對站 VPN 連線。 
3. 延伸內部部署 Active Directory。
4. [準備內部部署 VMware 伺服器](tutorial-prepare-on-premises-vmware.md)。
5. 為內部部署 VM [設定災害復原](tutorial-vmware-to-azure.md)到 Azure。

## <a name="extend-dfsr-to-an-azure-iaas-virtual-machine"></a>將 DFSR 延伸至 Azure IaaS 虛擬機器

1. 在內部部署網站與 Azure 網路之間建立站對站 VPN 連線。 
2. 延伸內部部署 Active Directory。
3. 在 Azure 虛擬網路上[建立及佈建檔案伺服器 VM](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)。
請確定虛擬機器已新增至相同的 Azure 虛擬網路，其具備與內部部署環境交叉連線的能力。 
4. 在 Windows Server 上安裝和[設定 DFSR](https://blogs.technet.microsoft.com/b/filecab/archive/2013/08/21/dfs-replication-initial-sync-in-windows-server-2012-r2-attack-of-the-clones.aspx)。
5. [實作 DFS 命名空間](https://docs.microsoft.com/windows-server/storage/dfs-namespaces/deploying-dfs-namespaces)。
6. 若實作 DFS 命名空間，只要更新 DFS 命名空間資料夾目標，就可以將生產網站的共用資料夾容錯移轉至災害復原網站。 透過 Active Directory 複寫這些 DFS 命名空間變更後，使用者就會以透明的方式連線到適當的資料夾目標。

## <a name="use-file-sync-to-replicate-your-on-premises-files"></a>使用檔案同步複寫內部部署檔案
您可以使用檔案同步將檔案複寫到雲端。 當發生災害且無法使用您的內部部署檔案伺服器時，您就可以掛接雲端上的適當檔案位置，繼續處理來自用戶端機器的要求。
若要整合檔案同步與 Site Recovery：

* 使用 Site Recovery 保護檔案伺服器機器。 遵循[本教學課程](tutorial-vmware-to-azure.md)中的步驟。
* 使用檔案同步，將作為檔案伺服器的機器中的檔案複寫到雲端。
* 使用 Site Recovery 中的復原計畫功能新增指令碼，以在 Azure 中已容錯移轉的檔案伺服器 VM 上掛接 Azure 檔案共用。

請依照下列步驟使用檔案同步：

1. [在 Azure 中建立儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-create-storage-account?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。 如果您為儲存體帳戶選擇了「讀取權限異地備援儲存體」(建議)，在發生災害時，您將可從次要區域讀取資料。 如需詳細資訊，請參閱 [Azure 檔案共用災害復原策略](https://docs.microsoft.com/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。
2. [建立檔案共用](https://docs.microsoft.com/azure/storage/files/storage-how-to-create-file-share)。
3. 在您在內部部署檔案伺服器中[部署檔案同步](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide)。
4. 建立同步群組。 同步群組內的端點會與彼此保持同步。 同步群組必須包含至少一個雲端端點，代表 Azure 檔案共用。 同步群組也必須包含一個伺服器端點，代表內部部署 Windows 伺服器上的路徑。
5. 您的檔案此時會在 Azure 檔案共用和內部部署伺服器之間保持同步。
6. 您的內部部署環境中發生災害時，您可以使用[復原計畫](site-recovery-create-recovery-plans.md)執行容錯移轉。 新增指令碼，以掛接 Azure 檔案共用並存取虛擬機器中的共用。

> [!NOTE]
> 請確定連接埠 445 已開啟。 Azure 檔案會服務使用 SMB 通訊協定。 SMB 會透過 TCP 連接埠 445 進行通訊。 請確認您的防火牆不會從用戶端機器封鎖 TCP 連接埠 445。


## <a name="do-a-test-failover"></a>執行測試容錯移轉

1. 請移至 Azure 入口網站，然後選取您的復原服務保存庫。
2. 選取為檔案伺服器環境建立的復原計畫。
3. 選取 [測試容錯移轉]。
4. 選取復原點和 Azure 虛擬網路，以啟動測試容錯移轉程序。
5. 次要環境啟動後，請執行驗證。
6. 驗證完成後，請選取復原計畫上的 [清除測試容錯移轉]，測試容錯移轉環境隨即清除。

如需關於如何執行測試容錯移轉的詳細資訊，請參閱 [Site Recovery 的測試容錯移轉](site-recovery-test-failover-to-azure.md)。

如需為 Active Directory 和 DNS 進行測試容錯移轉的指引，請參閱 [Active Directory 和 DNS 的測試容錯移轉考量](site-recovery-active-directory.md)。

## <a name="do-a-failover"></a>執行容錯移轉

1. 移至 Azure 入口網站，然後選取您的復原服務保存庫。
2. 選取為檔案伺服器環境建立的復原計畫。
3. 選取 [容錯移轉]。
4. 選取復原點，以啟動容錯移轉程序。

如需關於如何執行容錯移轉的詳細資訊，請參閱[在 Site Recovery 中容錯移轉](site-recovery-failover.md)。
