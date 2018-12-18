---
title: VMware 到 Azure 的 Azure Site Recovery Deployment Planner | Microsoft Docs
description: 本文針對 VMware 到 Azure 案例說明 Azure Site Recovery 部署規劃工具所產生報告的分析。
services: site-recovery
author: nsoneji
manager: garavd
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/11/2018
ms.author: nisoneji
ms.openlocfilehash: e6aa3a8c788fe06a528a9c363a28faffa0a9ca88
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49094083"
---
# <a name="azure-site-recovery-deployment-planner-report"></a>Azure Site Recovery 部署規劃工具報告
產生的 Microsoft Excel 報告包含下列工作表：
## <a name="on-premises-summary"></a>內部部署摘要
內部部署摘要工作表提供已剖析的 VMware 環境概觀。

![VMware 環境的內部部署摘要](media/site-recovery-vmware-deployment-planner-analyze-report/on-premises-summary-v2a.png)

[開始日期] 和 [結束日期]：考慮用於產生報告之分析資料的開始和結束日期。 根據預設，開始日期是分析開始的日期，而結束日期是分析停止的日期。 如果使用這些參數產生報告，這可以是 'StartDate' 和 'EndDate' 值。

**分析總天數**：為其產生報告之開始和結束日期之間的分析總天數。

**相同的虛擬機器數目**：相容的 VM 總數，系統會計算其所需的網路頻寬、所需的儲存體帳戶、Microsoft Azure 核心及組態伺服器和額外處理序伺服器的數目。

**所有相容虛擬機器的磁碟總數**：此數字可做為其中一個輸入值，以決定部署中所要使用的組態伺服器與額外處理序伺服器數目。

**每部相容虛擬機器的平均磁碟數**：在所有相容 VM 中計算的平均磁碟數。

**平均磁碟大小 (GB)**：在所有相容 VM 中計算的平均磁碟大小。

**所需的 RPO (分鐘)**：預設復原點目標，或在產生報告時針對 'DesiredRPO' 參數所傳遞的值，用來估計所需的頻寬。

**所需的頻寬 (Mbps)**：在產生報告時針對 ‘Bandwidth’ 參數所傳遞的值，用來估計可用的 RPO。

**每日觀察到的典型資料變換 (GB)**：在所有分析天數中觀察到的平均資料變換。 此數字可做為其中一個輸入值，以決定部署中所要使用的組態伺服器與額外處理序伺服器數目。

## <a name="recommendations"></a>建議

根據選取的所需 RPO，VMware 到 Azure 報告的建議工作表包含下列詳細資料：

![VMware 至 Azure 報告的建議](media/site-recovery-vmware-deployment-planner-analyze-report/Recommendations-v2a.png)

### <a name="profiled-data"></a>剖析的資料
![Deployment Planner 中已剖析的資料檢視](media/site-recovery-vmware-deployment-planner-analyze-report/profiled-data-v2a.png)

**分析的資料期間**：執行分析的期間。 根據預設，此工具會在計算中包含所有剖析的資料，除非在報告產生期間使用 StartDate 和 EndDate 選項來針對特定期間產生此報告。

**伺服器名稱**：為其產生 VM 報告之 VMware vCenter 或 ESXi 主機的名稱或 IP 位址。

**所需的 RPO**：您部署的復原點目標。 預設會計算 RPO 值 15、30 和 60 分鐘所需的網路頻寬。 根據選取項目，在工作表上更新受影響的值。 如果您在產生報告時使用了 DesiredRPOinMin 參數，該值會顯示於所需 RPO 結果中。

### <a name="profiling-overview"></a>分析概觀

![Deployment Planner 中的分析結果](media/site-recovery-vmware-deployment-planner-analyze-report/profiling-overview-v2a.png)

**剖析的虛擬機器總數**：可取得其剖析資料的 VM 總數。 如果 VMListFile 包含任何未剖析的 VM 名稱，則這些 VM 不會納入報告產生考量並從剖析的 VM 總計數中排除。

**相容的虛擬機器**：可以使用 Site Recovery 受 Azure 保護的 VM 數目。 這是相容的 VM 總數，系統會計算其所需的網路頻寬、儲存體帳戶數目、Azure 核心數目以及組態伺服器和額外處理序伺服器數目。 在 [相容 VM] 區段中可取得每部相容 VM 的詳細資料。

**不相容的虛擬機器**：可使用 Site recovery 保護之不相容的剖析 VM 數目。 「不相容的 VM」一節會提到不相容的原因。 如果 VMListFile 包含任何未剖析 VM 的名稱，則這些 VM 會從不相容的 VM 計數中排除。 這些 VM 會在「不相容的 VM」區段結尾列為「找不到資料」。

**所需 RPO**：以分鐘為單位的所需復原點目標。 此報告會針對三個 RPO 值而產生：15 (預設值)、30 和 60 分鐘。 報告中的頻寬建議會隨著您在工作表右上方的 [所需的 RPO] 下拉式清單中選取的項目變更。 如果您已使用 -DesiredRPO 參數以自訂值產生報告，這個自訂值將會顯示為 [所需的 RPO] 下拉式清單中的預設值。

### <a name="required-network-bandwidth-mbps"></a>所需的網路頻寬 (Mbps)

![Deployment Planner 中所需的網路頻寬](media/site-recovery-vmware-deployment-planner-analyze-report/required-network-bandwidth-v2a.png)

**符合當時的 100% RPO︰** 所要配置的建議頻寬 (以 Mbps 為單位)，以符合當時所需的 100% RPO。 此頻寬量必須專用於您所有相容 VM 的穩定狀態差異複寫，以避免任何 RPO 違規。

**符合當時的 90% RPO**︰如果因為寬頻價格或任何其他原因，您無法設定符合當時所需 100% RPO 所需的頻寬，您可以選擇設定較低的頻寬，以符合當時所需的 90% RPO。 若要了解設定此低頻寬的含意，報告提供了可預期之 RPO 違規次數和持續時間的假設分析。

**達成的輸送量**：來自您已對儲存體帳戶所在 Microsoft Azure 區域執行 GetThroughput 命令之伺服器的輸送量。 此輸送量指出當您使用 Site Recovery 保護相容 VM 時可達成的預估等級，前提是您的組態伺服器或處理序伺服器儲存體和網路特性仍與您執行此工具的伺服器相同。

如需複寫，您應該設定建議的頻寬，以符合當時的 100% RPO。 在設定頻寬之後，如果您未看到此工具所報告的達成輸送量有任何增加，請執行下列作業︰

1. 查看是否有任何網路服務品質 (QoS) 會限制 Site Recovery 輸送量。

2. 查看 Site Recovery 保存庫是否位於支援的最近實體 Microsoft Azure 區域，以縮小網路延遲。

3. 檢查本機儲存體特性，以判斷是否可以改善硬體 (例如 HDD 變成 SSD)。

4. 變更處理序伺服器中的 Site Recovery 設定，以[增加用於複寫的網路頻寬](./site-recovery-plan-capacity-vmware.md#control-network-bandwidth)。

如果您在已有受保護 VM 的組態伺服器或處理序伺服器上執行此工具，請執行此工具數次。 視當時正在處理的變換數量而定，達成的輸送量數字會隨之改變。

對於所有企業 Site Recovery 部署，建議使用 [ExpressRoute](https://aka.ms/expressroute)。

### <a name="required-storage-accounts"></a>所需的儲存體帳戶
下列圖表顯示保護所有相容 VM 所需的儲存體帳戶 (標準和進階) 總數。 若要了解每個 VM 所要使用的儲存體帳戶，請參閱「VM 儲存體放置」一節。

![Deployment Planner 中所需的儲存體帳戶](media/site-recovery-vmware-deployment-planner-analyze-report/required-storage-accounts-v2a.png)

### <a name="required-number-of-azure-cores"></a>所需的 Azure 核心數目
此結果是在所有相容 VM 的容錯移轉或測試容錯移轉之前所要設定的核心總數。 如果訂用帳戶中可用的核心太少，Site Recovery 便無法在測試容錯移轉或容錯移轉時建立 VM。

![Deployment Planner 中所需的 Azure 核心數目](media/site-recovery-vmware-deployment-planner-analyze-report/required-cores-v2a.png)

### <a name="required-on-premises-infrastructure"></a>所需的內部部署基礎結構
此數字是為了足以保護所有相容的 VM，所要設定的組態伺服器與額外處理序伺服器總數。 視支援的[組態伺服器大小建議](https://aka.ms/asr-v2a-on-prem-components)而定，此工具可能會建議額外的伺服器。 建議是以每日變換或受保護 VM 數目上限中的較大者為基礎 (假設每部 VM 平均有三個磁碟)，無論在組態伺服器或額外處理序伺服器上何者先達到。 您可在「內部部署摘要」一節中找到每日變換總計和受保護磁碟總數的詳細資料。

![Deployment Planner 中所需的內部部署基礎結構](media/site-recovery-vmware-deployment-planner-analyze-report/required-on-premises-components-v2a.png)

### <a name="what-if-analysis"></a>假設分析
此分析概述當您設定較低的頻寬，讓所需的 RPO 只符合當時的 90% 時，在剖析期間可發生多少次違規。 任何指定的日期都可能發生一或多個 RPO 違規。 此圖會顯示當天的尖峰 RPO。
根據這項分析，您可以判斷指定的較低頻寬是否可接受所有天數的 RPO 違規次數和每日的尖峰 RPO 目標達成。 如果可接受，您可以配置較低的頻寬進行複寫，否則依照建議配置較高的頻寬，以符合當時所需的 100% RPO。

![Deployment Planner 中的假設分析](media/site-recovery-vmware-deployment-planner-analyze-report/what-if-analysis-v2a.png)

### <a name="recommended-vm-batch-size-for-initial-replication"></a>初始複寫的建議 VM 批次大小
在這一節中，我們會建議可平行保護的 VM 數目，以使用建議的頻寬在 72 小時內完成初始複寫，進而符合設定階段所需的 100% RPO。 此值是可設定的值。 若要在產生報告階段變更此值，請使用 *GoalToCompleteIR* 參數。

下面的圖形顯示各種頻寬值和計算的 VM 批次大小計數，以根據在所有相容 VM 中偵測到的平均 VM 大小，在 72 小時內完成初始複寫。

在公開預覽版本中，報告不會指定哪些 VM 應包含在某個批次中。 您可以使用「相容的 VM」區段中顯示的磁碟大小，找出每部 VM 的大小並選取某個批次的 VM，或根據已知的工作負載特性選取 VM。 初始複寫的完成時間會根據實際 VM 磁碟大小、已用的磁碟空間和可用的網路輸送量按比例變更。

![建議的 VM 批次大小](media/site-recovery-vmware-deployment-planner-analyze-report/ir-batching-v2a.png)

### <a name="cost-estimation"></a>成本估計
此圖形會就您所選的目標區域和您針對報告產生所指定的貨幣，顯示災害復原 (DR) 至 Azure 之估計總成本的摘要檢視。

![成本估計摘要](media/site-recovery-vmware-deployment-planner-analyze-report/cost-estimation-summary-v2a.png)

當您使用 Azure Site Recovery 在 Azure 中保護所有相容 VM 時，此摘要可協助您了解您需要支付的儲存體、計算、網路和授權成本。 成本是針對相容的 VM，而不是針對所有已分析的 VM 計算。  

您可以檢視每月或每年的成本。 深入了解[支援的目標區域](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-target-regions)和[支援的貨幣](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-currencies)。

**依元件的成本** DR 總成本分成四個元件：計算、儲存體、網路和 Azure Site Recovery 授權成本。 此成本是根據在複寫期間和在 DR 演練時產生的耗用量來計算，其中涵蓋計算、儲存體 (進階和標準)、在內部部署網站與 Azure 之間設定的 ExpressRoute/VPN，以及 Azure Site Recovery 授權。

**依狀態的成本** 災害復原 (DR) 的總成本是根據兩個不同的狀態來分類 - 複寫和 DR 演練。

**複寫成本**：在複寫期間產生的成本。 其中涵蓋儲存體、網路和 Azure Site Recovery 授權的成本。

**DR 演練成本**：在測試容錯移轉期間產生的成本。 Azure Site Recovery 會在測試容錯移轉期間啟動 VM。 DR 演練成本涵蓋執行 VM 的計算和儲存成本。

**每月/年的 Azure 儲存成本** 它會顯示將會針對複寫及 DR 演練的進階和標準儲存體產生的總儲存成本。
您可以在[成本估計](site-recovery-vmware-deployment-planner-cost-estimation.md)工作表中檢視每個 VM 的詳細成本分析。

### <a name="growth-factor-and-percentile-values-used"></a>使用的成長因子和百分位數值
位於工作表底部的這個區段會顯示用於剖析 VM 之所有效能計數器的百分位數值 (預設值為第 95 個百分位數)，以及所有計算中使用的成長因子 (預設值為 30%)。

![使用的成長因子和百分位數值](media/site-recovery-vmware-deployment-planner-analyze-report/growth-factor-v2a.png)

## <a name="recommendations-with-available-bandwidth-as-input"></a>以可用頻寬做為輸入的建議

![以可用頻寬做為輸入的建議](media/site-recovery-vmware-deployment-planner-analyze-report/profiling-overview-bandwidth-input-v2a.png)

您可能會遇到一下情況：您知道無法針對 Site Recovery 複寫設定超過 x Mbps 的頻寬。 此工具可讓您輸入可用的頻寬 (在報告產生期間使用 -Bandwidth 參數)，並取得可達成的 RPO (以分鐘為單位)。 使用此可達成的 RPO 值，您可以決定是否需要設定額外的頻寬，或可接受具有此 RPO 的災害復原解決方案。

![500 Mbps 頻寬可達成的 RPO](media/site-recovery-vmware-deployment-planner-analyze-report/achievable-rpo-v2a.png)

## <a name="vm-storage-placement"></a>VM 儲存體放置

![VM 儲存體放置](media/site-recovery-vmware-deployment-planner-analyze-report/vm-storage-placement-v2a.png)

**磁碟儲存體類型**：標準或進階儲存體帳戶，用來複寫 [要放置的 VM] 資料行中提到的所有對應 VM。

**建議的前置詞**：建議的三個字元前置詞，可用來命名儲存體帳戶。 您可以使用自己的前置詞，但工具的建議會遵循[儲存體帳戶的磁碟分割命名慣例](https://aka.ms/storage-performance-checklist)。

**建議的帳戶名稱**：在您包含建議前置詞後的儲存體帳戶名稱。 以您的自訂輸入取代括弧內的名稱 (< 和 >)。

**記錄儲存體帳戶**︰所有複寫記錄都會儲存在標準儲存體帳戶中。 對於複寫至進階儲存體帳戶的 VM，請針對記錄儲存體設定額外的標準儲存體帳戶。 多個進階複寫儲存體帳戶可以使用單一標準記錄儲存體帳戶。 複寫到標準儲存體帳戶的 VM 會對記錄使用相同的儲存體帳戶。

**建議的記錄帳戶名稱**：在您包含建議前置詞後的儲存體記錄帳戶名稱。 以您的自訂輸入取代括弧內的名稱 (< 和 >)。

**放置摘要**：提供在複寫和測試容錯移轉或容錯移轉時，儲存體帳戶上載入的 VM 總數摘要。 其中包含對應到儲存體帳戶的 VM 總數、置於此儲存體帳戶中所有 VM 的讀/寫 IOPS 總計、寫入 (複寫) IOPS 總計、所有磁碟的設定大小總計，以及磁碟總數。

**要放置的虛擬機器**：應置於指定之儲存體帳戶的所有 VM 清單，以提供最佳效能和使用情況。

## <a name="compatible-vms"></a>相容的 VM
![相容 VM 的 Excel 試算表](media/site-recovery-vmware-deployment-planner-analyze-report/compatible-vms-v2a.png)

**VM 名稱**：產生報告時，使用於 VMListFile 的 VM 名稱或 IP 位址。 此資料行也會列出連結至 VM 的磁碟 (VMDK)。 為了區分具有重複名稱或 IP 位址的 vCenter VM，名稱中包含 ESXi 主機名稱。 所列的 ESXi 主機是此工具在分析期間探索到 VM 時其所在位置的主機。

**VM 相容性**：值為 [是] 和 [是\*]。 **[是\*]** 適用於 VM 適合於 [Azure 進階儲存體](https://aka.ms/premium-storage-workload)的情況。 在此情況下，剖析的高變換 / IOPS 磁碟符合 P20 或 P30 類別，但磁碟大小會導致它向下對應至 P10 或 P20。 儲存體帳戶會根據磁碟大小，決定磁碟所要對應至的進階儲存體大小類型。 例如︰
* <128 GB 為 P10。
* 128 GB 至 256 GB 為 P15。
* 256 GB 至 512 GB 為 P20。
* 512 GB 至 1024 GB 為 P30。
* 1025 GB 至 2048 GB 為 P40。
* 2049 GB 至 4095 GB 為 P50。

比方說，如果磁碟的工作負載特性讓它放在 P20 或 P30 類別中，但大小會將它對應到較低的進階儲存體磁碟類型，此工具會將該 VM 標示為 **[是**]\*。 此工具也建議您變更來源磁碟大小，以符合建議的進階儲存體磁碟類型，或變更容錯移轉後的目標磁碟類型。

**儲存體類型**：標準或進階。

**建議的前置詞**：三個字元的儲存體帳戶前置詞。

**儲存體帳戶**︰使用建議儲存體帳戶前置詞的名稱。

**尖峰 R/W IOPS (含成長因子)**：磁碟上的尖峰工作負載讀/寫 IOPS (預設值為第 95 個百分位數)，包括未來的成長因子 (預設值為 30%)。 請注意，VM 的讀/寫 IOPS 總計不一定是 VM 個別磁碟的讀/寫 IOPS 總和，因為 VM 的尖峰讀/寫 IOPS 是其個別磁碟在分析期間內每一分鐘之讀/寫 IOPS 總和的尖峰。

**尖峰資料變換 (Mbps) (含成長因子)**：磁碟上的尖峰變換率 (預設值為第 95 個百分位數)，包括未來的成長因子 (預設值為 30%)。 請注意，VM 的資料變換總計不一定是 VM 個別磁碟的資料變換總和，因為 VM 的尖峰資料變換是其個別磁碟在分析期間每一分鐘之變換總和的尖峰。

**Azure VM 大小**：此內部部署 VM 理想的對應 Azure 雲端服務虛擬機器大小。 對應是以內部部署 VM 的記憶體、磁碟/核心/NIC 數目和讀/寫 IOPS 為基礎。 建議一律是符合所有內部部署 VM 特性的最低 Azure VM 大小。

**磁碟數目**：VM 上的虛擬機器磁碟 (VMDK) 總數。

**磁碟大小 (GB)**：VM 之所有磁碟的總設定大小。 此工具也會顯示 VM 中個別磁碟的磁碟大小。

**核心**：VM 上的 CPU 核心數目。

**記憶體 (MB)**：VM 上的 RAM。

**NIC**：VM 上的 NIC 數目。

**開機類型**︰VM 的開機類型。 可以是 BIOS 或 EFI。  Azure Site Recovery 目前支援 Windows Server EFI VM (Windows Server 2012、2012 R2 和 2016)，前提是開機磁碟中的磁碟分割數目小於 4 個，而且開機磁區大小為 512 個位元組。 若要保護 EFI VM，Azure Site Recovery 行動服務版本必須是 9.13 或更新版本。 EFI VM 僅支援容錯移轉。 不支援容錯回復。  

**OS 類型**：這是 VM 的 OS 類型。 根據在建立 VM 時從 VMware vSphere 所選的範本，可以是 Windows 或 Linux 或其他類型。  

## <a name="incompatible-vms"></a>不相容的 VM

![不相容 VM 的 Excel 試算表
](media/site-recovery-vmware-deployment-planner-analyze-report/incompatible-vms-v2a.png)

**VM 名稱**：產生報告時，使用於 VMListFile 的 VM 名稱或 IP 位址。 此資料行也會列出連結至 VM 的磁 VMDK。 為了區分具有重複名稱或 IP 位址的 vCenter VM，名稱中包含 ESXi 主機名稱。 所列的 ESXi 主機是此工具在分析期間探索到 VM 時其所在位置的主機。

**VM 相容性**：指出為何指定的 VM 不適合與 Site Recovery 搭配使用。 相關原因會針對 VM 的每個不相容磁碟進行說明，且根據發佈的[儲存體限制](https://aka.ms/azure-storage-scalbility-performance)，原因可能是下列其中一項：

* 磁碟大小 >4095 GB。 Azure 儲存體目前不支援大於 4095 GB 的資料磁碟大小。

* OS 磁碟 >2048 GB。 Azure 儲存體目前不支援大於 2048 GB 的 OS 磁碟大小。

* VM 大小總計 (複寫 + TFO) 超過支援的儲存體帳戶大小限制 (35 TB)。 當 VM 中單一磁碟的效能特性超過標準儲存體支援的最大 Azure 或 Site Recovery 限制，通常會發生此不相容情況。 這類情況會將 VM 推送到進階儲存體區域中。 不過，進階儲存體帳戶支援的大小上限為 35 TB，而單一的受保護 VM 無法跨多個儲存體帳戶受到保護。 也請注意，在受保護的 VM 上執行測試容錯移轉時，它會在正在進行複寫的相同儲存體帳戶中執行。 在此例中，設定 2 倍的磁碟大小，以便進行複寫並以平行方式繼續進行測試容錯移轉。

* 來源 IOPS 超過支援的儲存體 IOPS 限制 (每個磁碟 7500)。

* 來源 IOPS 超過支援的儲存體 IOPS 限制 (每個 VM 80,000)。

* 平均資料變換超出磁碟平均 IO 大小支援的 Site Recovery 資料變換限制 10 MB/s。

* 平均資料變換超出 VM 平均 IO 大小支援的 Site Recovery 資料變換限制 25 MB/秒 (所有磁碟變換的總和)。

* VM 上所有磁碟的尖峰資料變換超過每個 VM 支援的 Site Recovery 尖峰資料變換限制 54 MB/秒。

* 磁碟的平均有效寫入 IOPS 超過支援的 Site Recovery IOPS 限制 840。

* 計算的快照集儲存體超過支援的快照集儲存體限制 10 TB。

* 每日資料變換總計超出處理序伺服器支援的每日變換限制 2 TB。


**尖峰 R/W IOPS (含成長因子)**：磁碟上的尖峰工作負載 IOPS (預設值為第 95 個百分位數)，包括未來的成長因子 (預設值為 30%)。 請注意，VM 的讀/寫 IOPS 總計不一定是 VM 個別磁碟的讀/寫 IOPS 總和，因為 VM 的尖峰讀/寫 IOPS 是其個別磁碟在分析期間內每一分鐘之讀/寫 IOPS 總和的尖峰。

**尖峰資料變換 (Mbps) (含成長因子)**：磁碟上的尖峰變換率 (預設值為第 95 個百分位數)，包括未來的成長因子 (預設值為 30%)。 請注意，VM 的資料變換總計不一定是 VM 個別磁碟的資料變換總和，因為 VM 的尖峰資料變換是其個別磁碟在分析期間每一分鐘之變換總和的尖峰。

**磁碟數目**：VM 上的 VMDK 總數。

**磁碟大小 (GB)**：VM 之所有磁碟的總設定大小。 此工具也會顯示 VM 中個別磁碟的磁碟大小。

**核心**：VM 上的 CPU 核心數目。

**記憶體 (MB)**：VM 上的 RAM 數量。

**NIC**：VM 上的 NIC 數目。

**開機類型**︰VM 的開機類型。 可以是 BIOS 或 EFI。  Azure Site Recovery 目前支援 Windows Server EFI VM (Windows Server 2012、2012 R2 和 2016)，前提是開機磁碟中的磁碟分割數目小於 4 個，而且開機磁區大小為 512 個位元組。 若要保護 EFI VM，Azure Site Recovery 行動服務版本必須是 9.13 或更新版本。 EFI VM 僅支援容錯移轉。 不支援容錯回復。

**OS 類型**：這是 VM 的 OS 類型。 根據在建立 VM 時從 VMware vSphere 所選的範本，可以是 Windows 或 Linux 或其他類型。

## <a name="azure-site-recovery-limits"></a>Azure Site Recovery 限制
下表提供 Azure Site Recovery 限制。 上述限制是以我們的測試為基礎，但無法涵蓋所有可能的應用程式 I/O 組合。 實際的結果會隨著您的應用程式 I/O 混合而有所不同。 為了獲得最佳結果，即使在部署規劃之後，仍一律建議發出測試容錯移轉來執行廣泛的應用程式測試，以了解應用程式真正的效能情況。

**複寫儲存體目標** | **平均來源磁碟 I/O 大小** |**平均來源磁碟資料變換** | **每日的來源磁碟資料變換總計**
---|---|---|---
標準儲存體 | 8 KB | 2 MB/秒 | 每個磁碟 168 GB
進階 P10 或 P15 磁碟 | 8 KB  | 2 MB/秒 | 每個磁碟 168 GB
進階 P10 或 P15 磁碟 | 16 KB | 4 MB/秒 |  每個磁碟 336 GB
進階 P10 或 P15 磁碟 | 32 KB 或更大 | 8 MB/秒 | 每個磁碟 672 GB
進階 P20、P30、P40 或 P50 磁碟 | 8 KB    | 5 MB/秒 | 每個磁碟 421 GB
進階 P20、P30、P40 或 P50 磁碟 | 16 KB 或更大 |10 MB/秒 | 每個磁碟 842 GB

**來源資料變換** | **最大限制**
---|---
每個 VM 的平均資料變換| 25 MB/秒
VM 上所有磁碟的尖峰資料變換 | 54 MB/秒
處理序伺服器支援的每日資料變換上限 | 2 TB

以上是採用 30% I/O 重疊時的平均數字。 Site Recovery 能夠處理更高的輸送量 (以重疊比為基礎)、較大的寫入大小和實際工作負載 I/O 行為。 先前數字採用大約五分鐘的典型積壓。 也就是說，資料上傳之後，便會進行處理並在五分鐘內建立復原點。


## <a name="cost-estimation"></a>成本估計
深入了解[成本估計](site-recovery-vmware-deployment-planner-cost-estimation.md)。


## <a name="next-steps"></a>後續步驟
深入了解[成本估計](site-recovery-vmware-deployment-planner-cost-estimation.md)。
