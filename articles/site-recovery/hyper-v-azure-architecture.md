---
title: Azure Site Recovery 中的 Hyper-V 至 Azure 複寫架構 | Microsoft Docs
description: 本文提供使用 Azure Site Recovery 服務將內部部署 Hyper-V VM (不含 VMM) 複寫至 Azure 時所用元件和架構的概觀。
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/10/2018
ms.author: raynew
ms.openlocfilehash: 49059415c5f96eeb4dd871e7d2c8ae4ab9b2c12e
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078353"
---
# <a name="hyper-v-to-azure-replication-architecture"></a>Hyper-V 至 Azure 複寫架構


本文說明透過 [Azure Site Recovery](site-recovery-overview.md) 服務，在內部部署 Hyper-V 主機與 Azure 之間複寫、容錯移轉和復原 Hyper-V 虛擬機器 (VM) 時所使用的架構和程序。

Hyper-V 主機可選擇性第在 System Center Virtual Machine Manager (VMM) 私人雲端中管理。



## <a name="architectural-components---hyper-v-without-vmm"></a>架構元件 - 不含 VMM 的 Hyper-V

下表和圖形提供當 Hyper-V 主機不受 VMM 管理時，將 Hyper-V 複寫至 Azure 時所用元件的高層級檢視。

**元件** | **需求** | **詳細資料**
--- | --- | ---
**Azure** | Azure 訂用帳戶、Azure 儲存體帳戶及 Azure 網路。 | 從內部部署 VM 工作負載複寫的資料會儲存在儲存體帳戶中。 從內部部署網站進行容錯移轉時，便會以複寫的工作負載資料建立 Azure VM。<br/><br/> Azure VM 在建立後會連線到 Azure 虛擬網路。
**Hyper-V** | 在 Site Recovery 部署期間，您會將 Hyper-V 主機和叢集蒐集到 Hyper-V 網站中。 您可在每部獨立的 Hyper-V 主機上，或在每個 Hyper-V 叢集節點上，安裝 Azure Site Recovery Provider 和復原服務代理程式。 | 此提供者會透過網際網路與 Site Recovery 協調複寫作業。 復原服務代理程式會處理資料複寫。<br/><br/> 來自提供者和代理程式的通訊都是安全且加密的。 Azure 儲存體中的複寫的資料也會加密。
**Hyper-V VM** | 一或多部在 Hyper-V 上執行的 VM。 | 不需要在 VM 上明確安裝任何項目。


**Hyper-V 至 Azure 架構 (不含 VMM)**

![架構](./media/hyper-v-azure-architecture/arch-onprem-azure-hypervsite.png)



## <a name="architectural-components---hyper-v-with-vmm"></a>架構元件 - 包含 VMM 的 Hyper-V

下表和圖形提供當 Hyper-V 主機在 VMM 中管理時，將 Hyper-V 複寫至 Azure 時所用元件的高層級檢視。

**元件** | **需求** | **詳細資料**
--- | --- | ---
**Azure** | Azure 訂用帳戶、Azure 儲存體帳戶及 Azure 網路。 | 從內部部署 VM 工作負載複寫的資料會儲存在儲存體帳戶中。 從內部部署站台進行容錯移轉時，便會以複寫的資料建立 Azure VM。<br/><br/> Azure VM 在建立後會連線到 Azure 虛擬網路。
**VMM 伺服器** | VMM 伺服器有一或多個包含 Hyper-V 主機的雲端。 | 您可在 VMM 伺服器上安裝 Site Recovery Provider 來協調 Site Recovery 進行複寫，並在復原服務保存庫中註冊伺服器。
**Hyper-V 主機** | 一或多個由 VMM 管理的 Hyper-V 主機/叢集。 |  您在每個 Hyper-V 主機或叢集節點上安裝復原服務代理程式。
**Hyper-V VM** | 一或多個在 Hyper-V 主機伺服器上執行的 VM。 | 不需要在 VM 上明確安裝任何項目。
**網路功能** | 在 VMM 伺服器上設定的邏輯和 VM 網路。 VM 網路應該連結到與雲端相關聯的邏輯網路。 | VM 網路會對應至 Azure 虛擬網路。 在容錯移轉後建立 Azure VM 時，Azure VM 會新增到對應至 VM 網路的 Azure 網路。

**Hyper-V 至 Azure 架構 (包含 VMM)**

![元件](./media/hyper-v-azure-architecture/arch-onprem-onprem-azure-vmm.png)



## <a name="replication-process"></a>複寫程序

![Hyper-V 至 Azure 複寫](./media/hyper-v-azure-architecture/arch-hyperv-azure-workflow.png)

**複寫和復原程序**


### <a name="enable-protection"></a>啟用保護。

1. 您在 Azure 入口網站或內部部署針對 Hyper-V VM 啟用保護之後，**啟用保護**隨即啟動。
2. 作業會檢查符合必要條件的機器，然後叫用 [CreateReplicationRelationship](https://msdn.microsoft.com/library/hh850036.aspx) 方法，以使用您進行的設定來設定複寫。
3. 作業會啟動初始複寫，方法是叫用 [StartReplication](https://msdn.microsoft.com/library/hh850303.aspx) 方法，以初始化完整的 VM 複寫，並且將 VM 的虛擬磁碟傳送至 Azure。
4. 您可以在 [作業] 索引標籤中監視作業。    ![作業清單](media/hyper-v-azure-architecture/image1.png) ![啟用保護向下鑽研](media/hyper-v-azure-architecture/image2.png)


### <a name="initial-data-replication"></a>初始資料複寫

1. 觸發初始複寫後，就會建立 [Hyper-V VM 快照集](https://technet.microsoft.com/library/dd560637.aspx)。
2. VM 上的虛擬硬碟會逐一複寫，直到它們全部複製到 Azure 為止。 視 VM 大小和網路頻寬而定，這可能需要一些時間。 [了解如何](https://support.microsoft.com/kb/3056159)增加網路頻寬。
3. 如果在初始複寫進行時發生磁碟變更，Hyper-V 複本複寫追蹤器會以 Hyper-V 複寫記錄 (.hrl) 的形式追蹤變更。 這些記錄檔位於與磁碟相同的資料夾中。 每個磁碟都有一個相關聯的.hrl 檔案會傳送至次要儲存體。 當初始複寫正在進行時，快照和記錄檔會取用磁碟資源。
4. 初始複寫完成時，就會刪除 VM 快照集。
5. 記錄中的差異磁碟變更會同步處理，並合併到父磁碟。


### <a name="finalize-protection-process"></a>完成保護程序

1. 當初始複寫完成之後，就會執行 [完成虛擬機器上的保護] 作業。 這會設定網路和其他複寫後設定，讓 VM 受到保護。
2. 在這個階段，您可以檢查 VM 設定以確定它已準備好進行容錯移轉。 您可以執行 VM 的災害復原演練 (測試容錯移轉)，以檢查它是否如預期般容錯移轉。 


## <a name="delta-replication"></a>差異複寫

1. 在初始複寫之後，系統會根據複寫原則，開始進行差異複寫。
2. Hyper-V 複本複寫追蹤器會以 .hrl 檔案格式追蹤虛擬硬碟的變更。 每個設定用於複寫的磁碟都會有相關聯的 .hrl 檔案。
3. 此記錄會傳送至客戶的儲存體帳戶。 當記錄傳輸至 Azure 時，將會在同一資料夾內的另一個記錄檔中追蹤主要磁碟中的變更。
4. 在初始和差異複寫期間，您可以在 Azure 入口網站中監視 VM。

### <a name="resynchronization-process"></a>重新同步處理程序

1. 如果差異複寫失敗且完整複寫因為頻寬或時間需要大量成本，就會將 VM 標示為重新同步處理。
    - 例如，如果 .hrl 檔案達到磁碟大小的 50%，系統就會標示 VM 以便重新同步處理。
    -  根據預設，重新同步處理會排程在上班時間以外的時間自動執行。
1.  重新同步處理只會傳送差異資料。
    - 它會計算來源和目標 VM 的總和檢查碼，將傳送的資料量降至最低。
    - 它會使用固定區塊的區塊處理演算法，將來源和目標檔案分成固定區塊。
    - 系統會產生每個區塊的總和檢查碼。 這些總和檢查碼會相互比較，以決定來源中的哪些區塊必須套用至目標。
2. 重新同步處理完成之後，應會繼續進行正常的差異複寫。
3. 如果您不想等候預設外部重新同步處理時數，您可以手動重新同步處理 VM。 例如，如果發生中斷。 若要這樣做，請在入口網站中選取 VM > [重新同步處理]。

    ![手動重新同步處理](./media/hyper-v-azure-architecture/image4-site.png)


### <a name="retry-process"></a>重試程序

如果發生複寫錯誤，會有內建的重試。 重試的分類如下表所述。

**類別** | **詳細資料**
--- | ---
**無法復原的錯誤** | 不嘗試重試。 VM 狀態為**重大**，需要管理員介入處理。<br/><br/> 這些錯誤的範例包括中斷 VHD 鏈結、複本 VM 的狀態無效、網路驗證錯誤、授權錯誤，以及找不到 VM 錯誤 (適用於獨立 Hyper-V 伺服器)。
**可復原的錯誤** | 在每個複寫間隔中進行重試，並採用指數倒退法，從第一次嘗試開始增加重試間隔 (1、2、4、8、10 分鐘)。 如果錯誤持續發生，會每隔 30 分鐘重試一次。 範例包括網路錯誤、磁碟空間不足錯誤，以及記憶體不足情況。



## <a name="failover-and-failback-process"></a>容錯移轉和容錯回復程序

1. 您可以執行從內部部署 Hyper-V VM 至 Azure 的計劃性或非計劃性容錯移轉。 如果您執行計劃性容錯移轉，則來源 VM 會關閉以確保不會遺失資料。 如果無法存取主要網站，請執行非計劃性容錯移轉。
2. 您可以容錯移轉單一機器，或建立復原方案來協調多部機器的容錯移轉。
3. 您可執行容錯移轉。 完成第一階段的容錯移轉之後，您應會在 Azure 中看到所建立的複本 VM。 如有必要，您可以對 VM 指派公用 IP 位址。
4. 然後，您要認可讓容錯移轉開始存取來自複本 Azure VM 的工作負載。

您的內部部署基礎結構再次啟動並執行之後，您可以進行容錯回復。 容錯回復分為三個階段：

1. 開始執行從 Azure 至內部部署網站的計劃性容錯移轉：
    - **將停機時間最小化**：如果您使用這個選項，Site Recovery 會在容錯移轉前同步處理資料。 它會檢查變更的資料區塊並將其下載到內部部署網站，而 Azure VM 會保持執行，以將停機時間最小化。 當您手動指定應該完成的容錯移轉時，系統會關閉 Azure VM，複製任何最終的差異變更，而容錯移轉會開始。
    - **完整下載**：使用此選項，資料會在容錯移轉期間進行同步處理。 此選項會下載整個磁碟。 它的速度更快，因為未計算任何總和檢查碼，但有更多的停機時間。 如果您已執行複本 Azure VM 一段時間，或已刪除內部部署 VM，請使用這個選項。
    - **建立 VM**：您可以選擇容錯回復到相同的 VM 或替代 VM。 您可指定 Site Recovery 應該建立 VM (如果尚未存在)。

2. 初始同步處理完成之後，您可選擇完成容錯移轉。 完成之後，您可登入內部部署 VM 來檢查一切是否如預期般運作。 在 Azure 入口網站中，您可以看到 Azure VM 已停止。
3.  然後，您要認可讓容錯移轉結束，並再次開始存取來自內部部署 VM 的工作負載。
4. 容錯回復工作負載之後，您會啟用反向複寫，以便內部部署 VM 再次複寫至 Azure。



## <a name="next-steps"></a>後續步驟


遵循[此教學課程](tutorial-prepare-azure.md)以開始使用 Hyper-V 至 Azure 的複寫。


