---
title: Azure Migrate 中的評量計算 | Microsoft Docs
description: 概括介紹 Azure Migrate 服務中的評量計算。
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 09/25/2018
ms.author: raynew
ms.openlocfilehash: f7f06636e025eda604caa65ca82d4dd7eb909d3f
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47165682"
---
# <a name="assessment-calculations"></a>評量計算

[Azure Migrate](migrate-overview.md) 會評估要移轉至 Azure 的內部部署工作負載。 此文章提供評量計算方式的相關資訊。


## <a name="overview"></a>概觀

Azure Migrate 評量有三個階段。 評量會從適用性分析開始，然後是調整大小，最後是每月成本估計。 機器必須先通過前一個評量，才可前往下一階段。 舉例而言，如果機器未通過 Azure 適用性檢查，就會標示為不適合 Azure，而不會再去計算大小和成本。


## <a name="azure-suitability-analysis"></a>Azure 適用性分析

並非所有機器都適用於雲端上執行，因為雲端有自己的限制和需求。 Azure Migrate 會評估每個內部部署機器移轉 Azure 的適用性，並將機器分類成下列類別之一：
- **可供 Azure 使用** - 機器不進行任何變更，即可依原樣移轉至 Azure。 它會在 Azure 中開機，並具有完整的 Azure 支援。
- **可有條件地供 Azure 使用** - 機器可在 Azure 中開機，但無法獲得完整的 Azure 支援。 例如，Azure 中不支援使用較舊版本 Windows Server 作業系統的機器。 將這些機器移轉至 Azure 之前需要特別注意，並請在移轉之前，遵循評量中建議的補救指導方針來修正整備程度問題。
- **無法供 Azure 使用** - 機器在 Azure 中將無法開機。 例如，如果內部部署機器有大小超過 4 TB、與其連接的磁碟，則無法將它裝載在 Azure 上。 在移轉至 Azure 之前，您需要遵循評量中建議的補救指導方針來修正整備程度問題。 對於標示為未備妥供 Azure 使用的機器，未完成適當的大小調整和成本估計。
- **整備程度未知** - 由於 vCenter Server 中的可用資料不足，Azure Migrate 找不到機器的整備程度。

Azure Migrate 會檢閱機器內容和客體作業系統，以識別內部部署機器的 Azure 移轉整備程度。

### <a name="machine-properties"></a>機器屬性
Azure Migrate 檢閱內部部署 VM 的下列屬性，以識別 VM 是否可以在 Azure 上執行。

**屬性** | **詳細資料** | **Azure 移轉整備程度狀態**
--- | --- | ---
**開機類型** | Azure 支援開機類型為 BIOS (而非 UEFI) 的 VM。 | 如果開機類型為 UEFI，便已有條件地就緒。
**核心** | 機器的核心數目必須等於或小於 Azure VM 支援的核心數目上限 (128 個核心)。<br/><br/> 如果有提供效能記錄，則 Azure Migrate 會將已使用的核心納入考量，進行比較。 如果評量設定已指定緩和因數，則會將使用的核心數目乘以緩和因數。<br/><br/> 如果沒有效能記錄，Azure Migrate 會使用配置的核心，而不套用緩和因數。 | 小於或等於限制便就緒。
**記憶體** | 機器的記憶體大小必須等於或小於 Azure VM 允許的記憶體上限 (Azure M 系列 Standard_M128m 上 3892 GB &nbsp;<sup>2</sup>)。 [深入了解](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)。<br/><br/> 如果有提供效能記錄，Azure Migrate 會將已使用的記憶體納入考量，進行比較。 如果已指定緩和因數，則會將使用的記憶體乘以緩和因數。<br/><br/> 如果沒有記錄，則會使用配置的記憶體，而不套用緩和因數。<br/><br/> | 在限制內便就緒。
**存放磁碟** | 已配置的磁碟大小必須小於或等於 4 TB (4096 GB)。<br/><br/> 附加至機器的磁碟數目必須小於或等於 65，作業系統磁碟亦包含在內。 | 在限制內便就緒。
**網路功能** | 機器必須附加 32 個以內的網路介面卡。 | 在限制內便就緒。

### <a name="guest-operating-system"></a>客體作業系統
隨著 VM 屬性，Azure Migrate 也會查看在內部部署 VM 的客體作業系統，以識別 VM 是否可以在 Azure 上執行。

> [!NOTE]
> Azure Migrate 會考量 vCenter Server 中指定的作業系統以進行下列分析。 由 Azure Migrate 探索是應用裝置為基礎，因此無法驗證 VM 內執行的作業系統是否與 vCenter Server 中所指定的相同。

Azure Migrate 以作業系統為基礎，使用下列邏輯來識別 VM 的 Azure 整備程度。

**作業系統** | **詳細資料** | **Azure 移轉整備程度狀態**
--- | --- | ---
Windows Server 2016 與所有 SP | Azure 提供完整支援。 | 可供 Azure 使用
Windows Server 2012 R2 與所有 SP | Azure 提供完整支援。 | 可供 Azure 使用
Windows Server 2012 與所有 SP | Azure 提供完整支援。 | 可供 Azure 使用
Windows Server 2008 R2，含所有 SP | Azure 提供完整支援。| 可供 Azure 使用
Windows Server 2008 (32 位元和 64 位元) | Azure 提供完整支援。 | 可供 Azure 使用
Windows Server 2003、2003 R2 | 這些作業系統已經過其支援結束的日期，需要[自訂支援合約 (CSA)](https://aka.ms/WSosstatement) 才能在 Azure 中獲得支援。 | 可有條件地供 Azure 使用，在移轉到 Azure 之前，請考慮升級作業系統。
Windows 2000、98、95、NT、3.1、MS-DOS | 這些作業系統已經過其支援結束的日期，機器可能可以在 Azure 中開機，但 Azure 不會提供作業系統支援。 | 可有條件地供 Azure 使用，建議在移轉到 Azure 之前升級作業系統。
Windows Client 7、8 及 10 | Azure [僅對 Visual Studio 訂用帳戶](https://docs.microsoft.com/azure/virtual-machines/windows/client-images)提供支援。 | 可有條件地供 Azure 使用
Windows 10 專業版桌面 | Azure 對[多租用戶主機權限](https://docs.microsoft.com/azure/virtual-machines/windows/windows-desktop-multitenant-hosting-deployment)提供支援。 | 可有條件地供 Azure 使用
Windows Vista、XP Professional | 這些作業系統已經過其支援結束的日期，機器可能可以在 Azure 中開機，但 Azure 不會提供作業系統支援。 | 可有條件地供 Azure 使用，建議在移轉到 Azure 之前升級作業系統。
Linux | Azure 認同這些 [Linux 作業系統](../virtual-machines/linux/endorsed-distros.md)。 其他 Linux 作業系統可能可以在 Azure 中開機，但建議您在移轉至 Azure 之前，將作業系統升級為認同的版本。 | 如果版本受到認同，則可供 Azure 使用。<br/><br/>如果版本未受到認同，則有條件地可供使用。
其他作業系統<br/><br/> 例如，Oracle Solaris、Apple Mac OS 等、FreeBSD 等。 | Azure 並未認同這些作業系統。 機器可能可以在 Azure 中開機，但 Azure 不會提供作業系統支援。 | 可有條件地供 Azure 使用，建議在移轉到 Azure 之前安裝支援的作業系統。  
在 vCenter Server 中作業系統指定為**其他** | Azure Migrate 無法在此情況下識別作業系統。 | 整備程度未知。 請確保 Azure 支援 VM 內執行的作業系統。
32 位元作業系統 | 機器可能可以在 Azure 中開機，但 Azure 可能不會提供完整支援。 | 可有條件地供 Azure 使用，在移轉到 Azure 之前，請考慮將機器的作業系統從 32 位元作業系統升級至 64 位元作業系統。

## <a name="sizing"></a>調整大小

將機器標示為可供 Azure 使用之後，Azure Migrate 會為 Azure 調整 VM 大小和其磁碟。 如果評估屬性中指定的大小調整準則是要執行以效能為基礎的大小調整，Azure Migrate 會考慮機器的效能歷程記錄以識別 Azure 中的 VM 大小和磁碟類型。 在您超額配置了內部部署 VM 但使用率很低，而您想要讓 Azure 中的 VM 有適當的大小來節省成本，則此方法會很有幫助。

如果不想要考慮 VM 調整大小的效能歷程記錄，並想要為將 VM 依原樣帶至 Azure，則可以將大小準則指定為*作為內部部署*，Azure Migrate 將會在不考慮使用量資料的情況下，以內部部署組態為基礎調整 VM 的大小。 在此情況下，將根據您在評估內容中指定的儲存類型 (標準磁碟或進階磁碟) 調整磁碟大小

### <a name="performance-based-sizing"></a>以效能為基礎調整大小

針對以效能為基礎的調整大小，Azure Migrate 一開始會先將磁碟附加至 VM，接著是網路介面卡，然後根據內部部署 VM 的運算需求來對應 Azure VM。

- **儲存體**：Azure Migrate 嘗試將連接至機器的每個磁碟對應至 Azure 中的磁碟。

    > [!NOTE]
    > Azure Migrate 僅支援受控磁碟的評量。

    - 為了獲得有效的每秒磁碟 I/O (IOPS) 和輸送量 (MBps)，Azure Migrate 會將磁碟 IOPS 和輸送量乘上緩和因數。 根據有效的 IOPS 及輸送量值，Azure Migrate 會識別磁碟應該對應至 Azure 中的標準或進階磁碟。
    - 如果 Azure Migrate 找不到 IOPS 及輸送量符合需求的磁碟，便會將該機器標示為不適合 Azure。 [進一步了解](../azure-subscription-service-limits.md#storage-limits)關於每個磁碟和 VM 的 Azure 限制。
    - 如果找到多個適合的磁碟，Azure Migrate 會挑選可支援儲存體備援方法的磁碟，以及可支援評量設定的指定位置的磁碟。
    - 如果有多個符合資格的磁碟，則會選取成本最低的磁碟。
    - 如果磁碟的效能資料無法使用，則所有磁碟都會對應至 Azure 中的標準磁碟。

- **網路**：Azure Migrate 會嘗試尋找可支援附加至內部部署機器的網路介面卡數量以及這些網路介面卡所需效能的 Azure VM。
    - 為了獲得內部部署 VM 的有效網路效能，Azure Migrate 會彙總在所有的網路介面卡間每秒從機器傳出的資料 (MBps) (網路對外)，並套用緩和因數。 會使用此數字來尋找可支援所要求的網路效能的 Azure VM。
    - 隨著網路效能，它也會考慮 Azure VM 是否可支援所需的網路介面卡數目。
    - 如果沒有可用的網路效能資料，調整虛擬機器大小時只會僅會考量網路介面卡的數量。

- **計算**：計算儲存體和網路需求之後，Azure Migrate 會考慮在 Azure 中尋找適當的 VM 大小的 CPU 和記憶體需求。
    - Azure Migrate 查看使用的核心和記憶體，並套用緩和因數以獲得有效的核心和記憶體。 Azure Migrate 會根據該數字，嘗試在 Azure 中尋找適當的虛擬機器大小。
    - 如果找不到合適的大小，便會將機器標示為不適合 Azure。
    - 如果找到合適的大小，Azure Migrate 會套用儲存體與網路計算。 然後再套用位置及定價層設定，對虛擬機器大小提出最終建議。
    - 如果有多個符合資格的 Azure VM 大小，建議使用成本最低的那個。

### <a name="as-on-premises-sizing"></a>作為內部部署調整大小
如果調整大小準則是*作為內部部署調整大小*，Azure Migrate 不會考慮 VM 和磁碟的效能歷程記錄，並且會根據內部部署所配置的大小在 Azure 中配置 VM SKU。 與調整磁碟大小類似，這會查看評估內容中指定的儲存類型 (標準/進階)，並據此建議磁碟類型。 預設儲存體類型為進階磁碟。

### <a name="confidence-rating"></a>信賴評等
Azure Migrate 中的每個效能型評量會與信賴評等相關聯，信賴評等的範圍從 1 顆星到 5 顆星 (1 顆星最低，5 顆星最高)。 根據計算評量所需的資料點可用性，每個評量都會指派信賴評等。 評量的信賴評等可協助您評估 Azure Migrate 提供的大小建議之可靠性。 信賴評等不適用內部部署評估。

對於以效能為基礎的大小調整，Azure Migrate 需要 VM 的 CPU、記憶體使用量資料。 此外，對於連結至 VM 的每個磁碟，它都需要磁碟 IOPS 和輸送量資料。 同樣地，對於連接至虛擬機器的每個網路介面卡，Azure Migrate 需要輸入/輸出網路以進行以效能為基礎的大小調整。 如果上述的任何使用量數字在 vCenter Server 中無法取得，則 Azure Migrate 所完成的大小建議可能不可靠。 根據可用資料點的百分比提供評量的信賴評等如下：

   **資料點的可用性** | **信賴評等**
   --- | ---
   0%-20% | 1 顆星
   21%-40% | 2 顆星
   41%-60% | 3 顆星
   61%-80% | 4 顆星
   81%-100% | 5 顆星

   以下是關於評量會獲得低信賴評等的原因：

   **單次探索**

   - vCenter Server 中統計資料設定未設為等級 3。 由於單次探索模型取決於 vCenter Server 的統計資料設定，如果 vCenter Server 中的統計資料設定低於等級 3，磁碟和網路的效能資料並未從 vCenter Server 收集。 在此情況下，Azure Migrate 針對磁碟和網路所提供的建議是以使用率為基礎。 若不考量磁碟的 IOPS/輸送量，Azure Migrate 便無法識別磁碟在 Azure 中是否需要進階磁碟，因此在此情況下，Azure Migrate 會針對所有磁碟建議標準磁碟。
   - vCenter Server 中的統計資料設定會在開始探索前短期設定為等級 3。 例如，讓我們考慮以下案例：您今天將統計資料設定變更為等級 3，並在明天 (24 小時後) 使用收集器設備開始探索。 如果您要建立一天的評量，您可擁有所有資料點，且評量的信賴評等會是 5 顆星。 但如果您將評量屬性中的效能持續時間變更為一個月，信賴評等會關閉，因無法使用過去一個月的磁碟和網路效能資料。 如果您想考量過去一個月的效能資料，建議您將 vCenter Server 統計資料設定保留在等級 3 一個月，再開始進行探索。

   **持續探索**

   - 您未針對正在建立評量的持續時間剖析環境。 例如，如果您要建立的評量將效能持續時間設定為 1 天，您需要至少等待一天後，再開始探索才能收集到所有資料點。

   **常見原因**  

   - 少數虛擬機器在評量計算期間關閉。 如果任何虛擬機器在某段期間內電源關閉，則無法收集這段時間的效能資料。
   - 少數虛擬機器在評量計算期間才建立。 例如，如果您要建立過去一個月的效能記錄評量，但是少數虛擬機器在一週前才建立在環境中。 在這種情況下，新虛擬機器的效能記錄不會在整段期間中出現。

   > [!NOTE]
   > 對於單次探索模型，如果任何評量的信賴評等低於 4 顆星，建議您將 vCenter Server 的統計資料設定等級變更為 3，等候您要評量的持續時間 (1 天/1 週/1 個月)，然後再執行探索及評量。 對於持續探索模型，至少等待一天後，再讓設備剖析環境，然後「重新計算」評量。 如果無法完成上述內容，則以效能為基礎的大小調整可能不可靠，建議您變更評量屬性以切換至「如內部部署大小調整」。

## <a name="monthly-cost-estimation"></a>每月成本預估

調整大小的建議完成之後，Azure Migrate 會計算移轉後的計算和儲存成本。

- **計算成本**：使用建議的 Azure VM 大小時，Azure Migrate 會使用計費 API 來計算虛擬機器的每月成本。 計算時會將作業系統、軟體保證、保留執行個體、VM 運作時間、位置和貨幣設定納入考量。 其會彙總所有機器的成本，以計算每月計算總成本。
- **儲存成本**：將機器附加的所有磁碟的每月成本彙總起來，計算出機器的每月儲存成本。 Azure Migrate 會將所有機器的儲存成本彙總起來，計算出每月的總儲存成本。 目前在計算時不會將評量設定中指定的供應項目列入考量。

成本會以評量設定中指定的貨幣顯示。


## <a name="next-steps"></a>後續步驟

[為內部部署 VMware VM 建立評量](tutorial-assessment-vmware.md)
