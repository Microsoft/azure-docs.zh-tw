---
title: 估計 Azure 中的複寫容量 | Microsoft Docs
description: 使用本文章來估計使用 Azure Site Recovery 進行複寫時的容量
services: site-recovery
documentationcenter: ''
author: rayne-wiselman
manager: jwhit
editor: ''
ms.assetid: 0a1cd8eb-a8f7-4228-ab84-9449e0b2887b
ms.service: site-recovery
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 10/11/2018
ms.author: nisoneji
ms.openlocfilehash: 3df7bd5ed44bdf514d48e451468329bd11fdf596
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49094032"
---
# <a name="plan-capacity-for-protecting-hyper-v-vms-with-site-recovery"></a>規劃使用 Site Recovery 保護 Hyper-V VM 的容量

[適用於 Hyper-V 對 Azure 部署的 Azure Site Recovery 部署規劃工具](site-recovery-hyper-v-deployment-planner.md)的新增強版本現已可用。 它會取代舊版工具。 使用新版工具進行部署規劃。
此工具提供下列指導方針：

* 以磁碟數目、磁碟大小、IOPS、變換和數個 VM 特性為基礎的 VM 適用性評估
* 網路頻寬需求與 RPO 評估
* Azure 基礎結構需求
* 內部部署基礎結構需求
* 初始複寫批次處理指引
* 向 Azure 進行災害復原的總估計成本


Azure Site Recovery Capacity Planner 可協助您判斷使用 Azure Site Recovery 複寫 Hyper-V VM 時的容量需求。

使用 Site Recovery Capacity Planner 來分析來源環境和工作負載。 它可協助您預估頻寬需求、來源位置所需的伺服器資源，以及目標位置中所需的資源 (例如 VM 和儲存體)。

有兩種模式可讓您執行工具：

* **快速規劃**：以 VM、磁碟、儲存體及變更率的平均數作為基礎，提供網路與伺服器的預測。
* **詳細的規劃**：提供在 VM 層級上每個工作負載的詳細資料。 分析 VM 相容性，並取得網路和伺服器預測。

## <a name="before-you-start"></a>開始之前

* 收集您的環境的資訊，包括 VM、每個 VM 的磁碟、每個磁碟的儲存體。
* 識別複寫資料的每日變更 (流失) 率。 請下載 [Hyper-V 容量規劃工具](https://www.microsoft.com/download/details.aspx?id=39057)來取得變更率。 [深入了解](site-recovery-capacity-planning-for-hyper-v-replication.md) 此工具。 我們建議您執行此工具一週以上的時間來擷取平均值。


## <a name="run-the-quick-planner"></a>執行快速規劃
1. 下載 [Site Recovery Capacity Planner](http://aka.ms/asr-capacity-planner-excel) 並開啟。 您需要執行巨集。 出現提示時，選擇啟用編輯與內容。

2. 在 [選取規劃類型] 清單方塊中，選取 [快速規劃]。

   ![開始使用](./media/site-recovery-capacity-planner/getting-started.png)

3. 在 [Capacity Planner] 工作表中，輸入必要資訊。 填妥以下螢幕擷取畫面中以紅色圈出來的所有欄位：

   a. 在 [選取您的案例] 中，選擇 [Hyper-V 到 Azure] 或 [VMware/實體到 Azure]。

   b. 在 [平均每日資料變更率 (%)] 中，輸入使用 [Hyper-V 容量規劃工具](site-recovery-capacity-planning-for-hyper-v-replication.md)或 [Site Recovery 部署規劃工具](./site-recovery-deployment-planner.md)收集的資訊。

   c. 將 Hyper-V VM 複寫到 Azure 時，不使用 [壓縮] 設定。 如需壓縮，請使用第三方設備，例如 Riverbed。

   d. 在 [保留天數] 中，指定保留複本的天數。

   e. 在 [應該完成虛擬機器批次初始複寫的時數] 與 [每一初始複寫批次的虛擬機器數] 中，輸入的設定是用於計算初始複寫需求。 部署 Site Recovery 時，應該上傳整個初始資料集。

   ![輸入](./media/site-recovery-capacity-planner/inputs.png)

4. 輸入來源環境的值之後，顯示的輸出會包含：

   * **差異複寫所需的頻寬 (MB/秒)**：根據平均每日資料變更率計算差異複寫的網路頻寬。
   * **初始複寫所需的頻寬 (MB/秒)**：根據您輸入的初始複寫值計算初始複寫的網路頻寬。
   * **所需的儲存體 (以 GB 為單位)**：所需的 Azure 儲存體總計。
   * **標準儲存體上的 IOPS 總數**：根據標準儲存體帳戶總計上的 8K IOPS 單位大小計算的數目。 針對快速規劃，數目是根據來源 VM 磁碟和每日資料變更率計算。 針對詳細規劃，數目是根據對應到標準 Azure VM 的 VM 數，以及那些 VM 上的資料變更率計算。
   * **所需的標準儲存體帳戶數目**：保護 VM 所需的標準儲存體帳戶總數。 標準儲存體帳戶可對標準儲存體中的所有 VM 支援多達 20,000 個 IOPS。 每個磁碟最多支援 500 個 IOPS。
   * **所需的 Blob 磁碟數目**：在 Azure 儲存體上建立的磁碟數目。
   * **所需的進階帳戶數目**：保護 VM 所需的進階儲存體帳戶總數。 具有高 IOPS (超過 20,000) 的來源 VM 需要進階儲存體帳戶。 進階儲存體帳戶可以支援多達 80,000 個 IOPS。
   * **進階儲存體上的 IOPS 總數**：根據進階儲存體帳戶總計上的 256K IOPS 單位大小計算的數目。 針對快速規劃，數目是根據來源 VM 磁碟和每日資料變更率計算。 針對詳細規劃，數目是根據對應到進階 Azure VM (DS 和 GS 系列) 的 VM 數，以及那些 VM 上的資料變更率計算。
   * **所需的組態伺服器數目**：顯示需要多少組態伺服器以進行部署。
   * **所需的額外處理序伺服器數目**：顯示除了預設在組態伺服器上執行的處理序伺服器，是否需要額外的處理序伺服器。
   * **來源上的 100% 其他儲存體**：顯示來源位置中是否需要其他儲存體。

      ![輸出](./media/site-recovery-capacity-planner/output.png)

## <a name="run-the-detailed-planner"></a>執行詳細規劃

1. 下載 [Site Recovery Capacity Planner](http://aka.ms/asr-capacity-planner-excel) 並開啟。 您需要執行巨集。 出現提示時，選擇啟用編輯與內容。

2. 在 [選取規劃類型] 中，從清單方塊中選取 [詳細規劃]。

   ![使用者入門指南](./media/site-recovery-capacity-planner/getting-started-2.png)

3. 在 [工作負載限定性條件] 工作表中，輸入必要資訊。 您必須填寫所有標示的欄位。

   a. 在 [處理器核心] 中，指定來源伺服器的核心總數。

   b. 在 [記憶體配置 (MB)] 中，指定來源伺服器的 RAM 大小。

   c. 在 [NIC 數] 中，指定來源伺服器上的網路介面卡數目。

   d. 在 [總儲存體 (GB)] 中，指定 VM 儲存體的總大小。 例如，如果來源伺服器有 3 個磁碟，每一個磁碟有 500 GB，則總儲存體大小為 1,500 GB。

   e. 在 [連接的磁碟數] 中，指定來源伺服器的磁碟總數。

   f. 在 [磁碟容量使用率 (%)] 中，指定平均使用率。

   g. 在 [每日資料變更率 (%)] 中，指定來源伺服器的每日資料變更率。

   h. 在 [對應 Azure VM 大小] 中，輸入您想要對應的 Azure VM 大小。 如果您不想要手動執行此動作，則選取 [計算 IaaS VM]。 如果您輸入手動設定，然後選取 [計算 IaaS VM]，則手動設定可能會遭到覆寫。 計算程序會自動識別最符合的 Azure VM 大小。

   ![工作負載限定性條件工作表](./media/site-recovery-capacity-planner/workload-qualification.png)

4. 如果您選取 [計算 IaaS VM] ，它會執行動作如下：

   * 驗證必要的輸入。
   * 計算 IOPS，並針對每部可供複寫至 Azure 的 VM 建議最佳的 Azure VM 大小。 如果無法偵測適當的 Azure VM 大小，則會顯示錯誤。 例如，如果連接的磁碟數為 65 就會顯示錯誤，因為 Azure VM 的最大大小為 64。
   * 建議可用於 Azure VM 的儲存體帳戶。
   * 計算工作負載所需的標準儲存體帳戶與進階儲存體帳戶總數。 向下捲動以檢視可用於來源伺服器的 Azure 儲存體類型與儲存體帳戶。
   * 根據為 VM 指派的必要儲存體類型 (標準或進階)，以及連接的磁碟數，完成並排序資料表的其餘部分。 對於符合 Azure 需求的所有 VM，資料行 [VM 是否合格？] 會顯示 [是]。 如果 VM 無法備份到 Azure，則會顯示錯誤。

資料行 AA 至 AE 為輸出，並提供每部 VM 的資訊。

![輸出資料行 AA 至 AE](./media/site-recovery-capacity-planner/workload-qualification-2.png)

### <a name="example"></a>範例
例如，對於具有資料表中顯示之值的六個 VM，此工具會計算並指派最佳的 Azure VM 相符項目，以及 Azure 儲存體需求。

![工作負載限定性條件指派](./media/site-recovery-capacity-planner/workload-qualification-3.png)

* 在範例輸出中，請注意下列項目：

  * 第一個資料行是 VM、磁碟及變換的驗證資料行。
  * 五個 VM 需要兩個標準儲存體帳戶和一個進階儲存體帳戶。
  * VM3 不適合保護，因為一或多個磁碟超過 1 TB。
  * VM1 和 VM2 可以使用第一個標準儲存體帳戶。
  * VM4 可以使用第二個標準儲存體帳戶。
  * VM5 和 VM6 需要進階儲存體帳戶，兩者都可以使用單一帳戶。

    > [!NOTE]
    > 在標準及進階儲存體上的 IOPS 是在 VM 層級計算，而不是在磁碟層級。 標準虛擬機器可以處理每個磁碟最多 500 個 IOPS。 如果磁碟的 IOPS 大於 500，則您需要進階儲存體。 如果磁碟的 IOPS 超過 500，但 VM 磁碟總數的 IOPS 在標準 Azure VM 支援的限制中，那麼規劃工具會挑選標準 VM，而不是 DS 或 GS 系列。 (Azure VM 的限制有 VM 大小、磁碟數目、介面卡數目、CPU 和記憶體。)您必須以適當的 DS 或 GS 系列 VM 手動更新對應的 Azure 大小儲存格。


輸入所有資訊之後，請選取 [將資料提交至規劃工具] 來開啟 Capacity Planner。 工作負載會醒目提示，以顯示是否可以進行保護。

### <a name="submit-data-in-capacity-planner"></a>提交 Capacity Planner 中的資料
1. 當您開啟 [Capacity Planner]  工作表時，它會根據您指定的設定填入。 「工作負載」這個字出現在 [基礎輸入來源] 儲存格，以顯示輸入 [工作負載限定性條件] 工作表。

2. 如果您想要進行變更，您必須修改 [工作負載限定性條件] 工作表。 然後再次選取 [將資料提交至規劃工具]。

   ![容量規劃](./media/site-recovery-capacity-planner/capacity-planner.png)

## <a name="next-steps"></a>後續步驟
[了解如何執行](site-recovery-capacity-planning-for-hyper-v-replication.md)容量規劃工具。
