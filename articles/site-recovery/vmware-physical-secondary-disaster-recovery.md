---
title: 使用 Azure Site Recovery 設定 VMware VM 或實體伺服器至次要網站的災害復原 | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 設定 VMware VM 或 Windows 和 Linux 實體伺服器至次要網站的災害復原。
services: site-recovery
author: nsoneji
manager: gauarvd
ms.service: site-recovery
ms.workload: backup-recovery
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 08/01/2018
ms.author: raynew
ms.openlocfilehash: 94abdd30dc9cd279ab791541250787a111f80d30
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39618983"
---
# <a name="set-up-disaster-recovery-of-on-premises-vmware-virtual-machines-or-physical-servers-to-a-secondary-site"></a>設定內部部署 VMware 虛擬機器或實體伺服器至次要網站的災害復原

[Azure Site Recovery](site-recovery-overview.md) 中的 InMage Scout 可提供內部部署 VMware 網站之間的即時複寫。 InMage Scout 包含在 Azure Site Recovery 服務訂用帳戶中。

## <a name="end-of-support-announcement"></a>終止支援公告

內部部署 VMware 或實體資料中心之間的複寫適用的 Azure Site Recovery 案例已達終止支援日期。

-   自 2018 年 8 月起，即無法在復原服務保存庫中設定此案例，且無法從保存庫下載 InMage Scout 軟體。 現有的部署仍受支援。 
-   自 2020 年 12 月 31 日起，將不再支援此案例。
- 現有的合作夥伴可在支援終止前將新客戶登入至案例。

在 2018 年和 2019 年期間，將會發行兩項更新： 

-   更新 7：修正網路組態和合規性問題，並提供 TLS 1.2 支援。
-   更新 8：新增 Linux 作業系統 RHEL/CentOS 7.3/7.4/7.5 和 SUSE 12 的支援

在更新 8 之後，將不再發行其他更新。 更新 8 將為作業系統新增有限的 Hotfix 支援，以及目前能力所及的最佳 Bug 修正程式。

Azure Site Recovery 將持續創新，以 Azure 作為災害復原站台，為 VMware 和 Hyper-V 客戶提供順暢且頂級的 DRaaS 解決方案。 Microsoft 建議現有的 InMage/ASR Scout 客戶應考慮使用 Azure Site Recovery 的「VMware 至 Azure」案例，以符合其商務持續性需求。 Azure Site Recovery 的「VMware 至 Azure」案例是適用於 VMware 應用程式的企業級 DR 解決方案，提供可在數分鐘內完成 RPO 和 RTO 的功能、多重 VM 應用程式複寫與復原的支援、順暢的登入體驗、全方位的監視，以及顯著的 TCO 優勢。

### <a name="scenario-migration"></a>案例移轉
在替代方案方面，我們建議將內部部署 VMware VM 和實體機器複寫至 Azure，藉以設定其災害復原功能。 以下列方式來執行此動作：

1.  檢閱下列快速比較。 您必須先確認內部部署機器符合複寫至 Azure 的[需求](./vmware-physical-azure-support-matrix.md#replicated-machines)，才能複寫這些機器。 如果您要複寫 VMware VM，建議您檢閱[容量規劃準則](./site-recovery-plan-capacity-vmware.md)，並執行[部署規劃工具](./site-recovery-deployment-planner.md)以識別容量需求，然後驗證合規性。
2.  執行部署規劃工具之後，您可以設定複寫：針對 VMware VM，請依照這些教學課程的指示[準備 Azure](./tutorial-prepare-azure.md)、[準備內部部署 VMware 環境](./vmware-azure-tutorial-prepare-on-premises.md)，並[設定災害復原](./vmware-azure-tutorial-prepare-on-premises.md)。
針對實體機器，請依照本[教學課程](./physical-azure-disaster-recovery.md)操作。
3.  在機器複寫至 Azure 之後，您可以執行[災害復原演練](./site-recovery-test-failover-to-azure.md)，以確保一切都如預期般運作。

### <a name="quick-comparison"></a>快速比較

**功能** | **複寫至 Azure** |**在 VMware 資料中心之間複寫**
--|--|--
**必要的元件** |複寫機器上的行動服務。 內部部署組態伺服器、處理伺服器、主要目標伺服器。Azure 中用於容錯回復的臨時處理伺服器。|行動服務、處理伺服器、組態伺服器和主要目標
**組態和協調流程** |Azure 入口網站中的復原服務保存庫 | 使用 vContinuum 
**複寫**|磁碟 (Windows 和 Linux) |磁碟區 - Windows<br> 磁碟 - Linux
**共用磁碟叢集**|不支援|支援
**資料變換限制 (平均值)** |每個磁碟 10 MB/s 的資料<br> 每個 VM 25 MB/s 的資料<br> [深入了解](./site-recovery-vmware-deployment-planner-analyze-report.md#azure-site-recovery-limits) | 每個磁碟 > 10 MB/s 的資料  <br> 每個 VM > 25 MB/s 的資料
**監視** |從 Azure 入口網站|從 CX (組態伺服器)
**支援矩陣**| [按一下這裡以取得詳細資訊](./vmware-physical-azure-support-matrix.md)|[下載 ASR Scout 相容矩陣](https://aka.ms/asr-scout-cm)


## <a name="prerequisites"></a>必要條件
若要完成本教學課程：

- [檢閱](vmware-physical-secondary-support-matrix.md)所有元件的支援需求。
- 確定您要複寫的機器符合[支援的複寫機器](vmware-physical-secondary-support-matrix.md#replicated-vm-support)。


## <a name="create-a-vault"></a>建立保存庫

[!INCLUDE [site-recovery-create-vault](../../includes/site-recovery-create-vault.md)]

## <a name="choose-a-protection-goal"></a>選擇保護目標

選取要複寫的內容及複寫目的地。

1. 按一下 [Site Recovery] > [準備基礎結構] > [保護目標]。
2. 選取 [到復原站台] > [Yes, with VMware vSphere Hypervisor] \(是，使用 VMware vSphere Hypervisor)。 然後按一下 [確定] 。
3. 在 [Scout 設定] 中，下載 InMage Scout 8.0.1 GA 軟體和註冊金鑰。 所有元件的安裝程式檔案都包含在下載的 .zip 檔中。

## <a name="download-and-install-component-updates"></a>下載及安裝元件更新

 檢閱並安裝最新[更新](#updates)。 請依下列順序在伺服器上安裝更新：

1. RX 伺服器 (如果適用)
2. 設定伺服器
3. 處理序伺服器
4. 主要目標伺服器
5. vContinuum 伺服器
6. 來源伺服器 (Windows 和 Linux 伺服器)

安裝更新，如下所示：

> [!NOTE]
>並非所有 Scout 元件的檔案更新版本在更新 .zip 檔案中都相同。 舊版表示自上次更新此更新以來，元件沒有任何變更。

下載 [update](https://aka.ms/asr-scout-update6) .zip 檔案。 此檔案包含下列元件： 
  - RX_8.0.4.0_GA_Update_4_8725872_16Sep16.tar.gz
  - CX_Windows_8.0.6.0_GA_Update_6_13746667_18Sep17.exe
  - UA_Windows_8.0.5.0_GA_Update_5_11525802_20Apr17.exe
  - UA_RHEL6-64_8.0.4.0_GA_Update_4_9035261_26Sep16.tar.gz
  - vCon_Windows_8.0.6.0_GA_Update_6_11525767_21Sep17.exe
  - 適用於 RHEL5、OL5、OL6、SUSE 10、SUSE 11 的 UA update4 位元：UA_<Linux OS>_8.0.4.0_GA_Update_4_9035261_26Sep16.tar.gz
1. 解壓縮 .zip 檔。
2. **RX 伺服器**：將 **RX_8.0.4.0_GA_Update_4_8725872_16Sep16.tar.gz** 複製到 RX 伺服器並將其解壓縮。 在解壓縮的資料夾中執行 **/Install**。
3. **設定伺服器和處理伺服器**：將 **CX_Windows_8.0.6.0_GA_Update_6_13746667_18Sep17.exe** 複製到設定伺服器和處理伺服器。 連按兩下加以執行。<br>
4. **Windows 主要目標伺服器**：若要更新整合代理程式，請將 **UA_Windows_8.0.5.0_GA_Update_5_11525802_20Apr17.exe** 複製到伺服器。 連按兩下加以執行。 相同的整合代理程式更新也適用於來源伺服器。 如果來源尚未更新為 Update 4，您應該更新整合代理程式。
  更新不需要在使用 **InMage_Scout_vContinuum_MT_8.0.1.0_Windows_GA_10Oct2017_release.exe** 備妥的主要目標上套用，因為這是包含所有最新變更的新 GA 安裝程式。
5. **vContinuum 伺服器**：將 **vCon_Windows_8.0.6.0_GA_Update_6_11525767_21Sep17.exe** 複製到伺服器。  確定您已經關閉 vContinuum 精靈。 連按兩下檔案加以執行。
    更新不需要在使用 **InMage_Scout_vContinuum_MT_8.0.1.0_Windows_GA_10Oct2017_release.exe** 備妥的主要目標上套用，因為這是包含所有最新變更的新 GA 安裝程式。
6. **Linux 主要目標伺服器**：若要更新整合代理程式，請將 **UA_RHEL6-64_8.0.4.0_GA_Update_4_9035261_26Sep16.tar.gz** 複製到主要目標伺服器並將它解壓縮。 在解壓縮的資料夾中執行 **/Install**。
7. **Windows 來源伺服器**：若要更新整合代理程式，請將 **UA_Windows_8.0.5.0_GA_Update_5_11525802_20Apr17.exe** 複製到來源伺服器。 連按兩下檔案加以執行。 
    如果來源伺服器已經更新為 Update 4，或者來源代理程式已經與最新的基底安裝程式 **InMage_UA_8.0.1.0_Windows_GA_28Sep2017_release.exe** 一起安裝，則您不需要在來源伺服器上安裝 Update 5 代理程式。
8. **Linux 來源伺服器**：若要更新整合代理程式，請將對應的整合代理程式版本複製到 Linux 伺服器並將它解壓縮。 在解壓縮的資料夾中執行 **/Install**。  範例：針對 RHEL 6.7 64 位元伺服器，請將 **UA_RHEL6-64_8.0.4.0_GA_Update_4_9035261_26Sep16.tar.gz** 複製到伺服器，並將它解壓縮。 在解壓縮的資料夾中執行 **/Install**。

## <a name="enable-replication"></a>啟用複寫

1. 設定來源與目標 VMware 網站之間的複寫。
2. 請參閱下列文件，以深入了解安裝 保護及復原：

   * [版本資訊](https://aka.ms/asr-scout-release-notes)
   * [相容性矩陣](https://aka.ms/asr-scout-cm)
   * [使用者指南](https://aka.ms/asr-scout-user-guide)
   * [RX 使用者指南](https://aka.ms/asr-scout-rx-user-guide)
   * [快速安裝指南](https://aka.ms/asr-scout-quick-install-guide)

## <a name="updates"></a>更新

### <a name="site-recovery-scout-801-update-6"></a>Site Recovery Scout 8.0.1 Update 6 
更新日期：2017 年 10 月 12 日

下載 [Scout Update 6](https://aka.ms/asr-scout-update6)。

Scout Update 6 是累積更新。 其中包含 Update 1 到 Update 5 的所有修正及以下描述的新修正和增強功能。 

#### <a name="new-platform-support"></a>新平台支援
* 新增對來源 Windows Server 2016 的支援
* 新增對下列 Linux 作業系統的支援：
    - Red Hat Enterprise Linux (RHEL) 6.9
    - CentOS 6.9
    - Oracle Linux 5.11
    - Oracle Linux 6.8
* 新增對VMware Center 6.5 的支援

> [!NOTE]
> * 已更新 Windows 適用的基底整合代理程式 (UA) 安裝程式，以支援 Windows Server 2016。 新的安裝程式 **InMage_UA_8.0.1.0_Windows_GA_28Sep2017_release.exe** 會與基底 Scout GA 封裝 (**InMage_Scout_Standard_8.0.1 GA-Oct17.zip**) 一起封裝。 相同的安裝程式將用於所有支援的 Windows 版本。 
> * 已更新基底 Windows vContinuum 和主要目標安裝程式，以支援 Windows Server 2016。 新的安裝程式 **InMage_Scout_vContinuum_MT_8.0.1.0_Windows_GA_10Oct2017_release.exe** 會與基底 Scout GA 封裝 (**InMage_Scout_Standard_8.0.1 GA-Oct17.zip**) 一起封裝。 相同的安裝程式將用來部署 Windows 2016 主要目標與 Windows 2012R2 主要目標。
> * 請遵循[建立保存庫](#create-a-vault)中所述，從入口網站下載 GA 套件。
>

#### <a name="bug-fixes-and-enhancements"></a>Bug 修正和增強功能
- Linux VM 的容錯回復保護失敗，且要複寫之磁碟的清單在組態結尾是空的。

### <a name="site-recovery-scout-801-update-5"></a>Site Recovery Scout 8.0.1 Update 5
Scout Update 5 是累積更新， 包含 Update 1 到 Update 4 的所有修正及以下描述的新修正。
- 從 Site Recovery Scout Update 4 到 Update 5 的修正是針對主要目標和 vContinuum 元件。
- 如果來源伺服器、主要目標、設定、處理和 RX 伺服器已在執行 Update 4，請只在主要目標伺服器上套用。 

#### <a name="new-platform-support"></a>新平台支援
* SUSE Linux Enterprise Server 11 Service Pack 4 (SP4)
* SLES 11 SP4 64 位元 **InMage_UA_8.0.1.0_SLES11-SP4-64_GA_13Apr2017_release.tar.gz** 與基底 Scout GA 套件 (**InMage_Scout_Standard_8.0.1 GA.zip**) 封裝在一起。 請遵循[建立保存庫](#create-a-vault)中所述，從入口網站下載 GA 套件。


#### <a name="bug-fixes-and-enhancements"></a>Bug 修正和增強功能

* 增加 Windows 叢集支援可靠性的修正：
    * 已修正 - 某些 P2V MSCS 叢集磁碟會在復原後變成 RAW。
    * 已修正 - P2V MSCS 叢集的復原因為磁碟順序不符而失敗。
    * 已修正 - MSCS 叢集的新增磁碟作業因為磁碟大小不符而失敗。
    * 已修正 - 來源 MSCS 叢集與 RDM LUN 對應的整備檢查在大小確認步驟失敗。
    * 已修正 - 單一節點叢集保護因為 SCSI 不符的問題而失敗。 
    * 已修正 - 如果有目標叢集磁碟，重新保護 P2V Windows 叢集伺服器會失敗。 
    
* 已修正 - 在容錯回復保護期間，如果選取的主要目標伺服器不是在和受保護來源機器相同的 ESXi 伺服器上 (在正向保護期間)，vContinuum 會在容錯回復復原期間挑出錯誤的主要目標伺服器，而且復原作業會失敗。

> [!NOTE]
> * P2V 叢集修正僅適用於剛以 Site Recovery Scout Update 5 加以保護的實體 MSCS 叢集。 若要在以舊版更新保護的 P2V MSCS 叢集上安裝叢集修正，請遵循 [Site Recovery Scout 版本資訊](https://aka.ms/asr-scout-release-notes)第 12 節中所述的升級步驟。
> * 重新保護時，只有當在每個叢集節點上使用中的幾個磁碟與最初受保護時相同，重新保護實體 MSCS 叢集才可以重複使用現有的目標磁碟。 如果沒有，則使用 [Site Recovery Scout 版本資訊](https://aka.ms/asr-scout-release-notes)第 12 節中的手動步驟，將目標端磁碟移至正確的資料存放區路徑，以便在重新保護時重複使用。 如果未遵循升級步驟重新保護 P2V 模式中的 MSCS 叢集，則會在目標 ESXi 伺服器上建立新的磁碟。 您必須手動刪除資料存放區中的舊磁碟。
> * 當來源 SLES11 或 SLES11 (含任何 Service Pack) 伺服器正常重新開機時，請手動標記**根**磁碟複寫配對以利重新同步處理。 CX 介面中不會有任何通知。 如果您未標記要重新同步處理的根磁碟，可能會注意到資料完整性問題。


### <a name="azure-site-recovery-scout-801-update-4"></a>Azure Site Recovery Scout 8.0.1 Update 4
Scout Update 4 是累積更新。 包含 Update 1 到 Update 3 的所有修正及以下描述的新修正。

#### <a name="new-platform-support"></a>新平台支援

* 新增對 vCenter/vSphere 6.0、6.1 及 6.2 的支援
* 新增對下列 Linux 作業系統的支援：
  * Red Hat Enterprise Linux (RHEL) 7.0、7.1 及 7.2
  * CentOS 7.0、7.1 及 7.2
  * Red Hat Enterprise Linux (RHEL) 6.8
  * CentOS 6.8

> [!NOTE]
> RHEL/CentOS 7 64 位元 **InMage_UA_8.0.1.0_RHEL7-64_GA_06Oct2016_release.tar.gz** 與基底 Scout GA 套件 **InMage_Scout_Standard_8.0.1 GA.zip** 封裝在一起。 請遵循[建立保存庫](#create-a-vault)中所述，從入口網站下載 Scout GA 套件。

#### <a name="bug-fixes-and-enhancements"></a>Bug 修正和增強功能

* 提升下列 Linux 作業系統和複製項的關機處理，以防止多餘的重新同步處理問題：
    * Red Hat Enterprise Linux (RHEL) 6.x
    * Oracle Linux (OL) 6.x
* 針對 Linux，現在僅限本機使用者可以擁有整合代理程式安裝目錄中的所有資料夾存取權限。
* 在 Windows 上，修正在負載繁重的分散式應用程式 (例如 SQL Server 和 Share Point 叢集) 上發出通用分散式一致性書籤時，所發生的逾時問題。
* 設定伺服器基底安裝程式中的記錄相關修正。
* 在 Windows 主要目標基底安裝程式中新增 VMware vCLI 6.0 的下載連結。
* 針對容錯移轉和災害復原演練期間的網路設定變更，新增更多檢查和記錄。
* 修正導致不會向設定伺服器回報保留資訊的問題。  
* 針對實體叢集，修正當壓縮來源磁碟區時，導致無法在 vContinuum 精靈中調整磁碟區大小的問題。
* 修正當叢集磁碟是 PRDM 磁碟時，叢集保護會因發生「找不到磁碟簽章」錯誤而失敗的問題。
* 修正超出範圍例外狀況所造成的 cxps 傳輸伺服器當機。
* 現在在 vContinuum 精靈的 [推送安裝] 頁面中可以調整 [伺服器名稱] 和 [IP 位址] 欄的大小。
* RX API 增強功能：
  * 現已推出五個最新可用的共同一致性點 (僅限保證標記)。
  * 顯示所有受保護裝置的容量和可用空間詳細資料。
  * 提供來源伺服器上的 Scout 驅動程式狀態。

> [!NOTE]
> * **InMage_Scout_Standard_8.0.1_GA.zip** 基底套件具有：
    * 更新的設定伺服器基底安裝程式 (**InMage_CX_8.0.1.0_Windows_GA_26Feb2015_release.exe**)
    * Windows 主要目標基底安裝程式 (**InMage_Scout_vContinuum_MT_8.0.1.0_Windows_GA_26Feb2015_release.exe**)。
    * 針對所有新的安裝，請使用新的設定伺服器和 Windows 主要目標 GA 位元。
> * Update 4 可以直接套用於 8.0.1 GA。
> * 設定伺服器和 RX 更新在套用後便無法復原。


### <a name="azure-site-recovery-scout-801-update-3"></a>Azure Site Recovery Scout 8.0.1 Update 3

所有的 Site Recovery 更新都是累計的。 Update 3 包含 Update 1 和 Update 2 的所有修正。 Update 3 可以直接套用於 8.0.1 GA。 設定伺服器和 RX 更新在套用後便無法復原。

#### <a name="bug-fixes-and-enhancements"></a>Bug 修正和增強功能
Update 3 修正下列問題：

* 設定伺服器和 RX 位於 Proxy 後方時未在保存庫中註冊。
* 健康狀態報告中未更新沒有達到復原點目標 (RPO) 的時數。
* 當 ESX 硬體詳細資料或網路詳細資料包含任何 UTF-8 字元時，設定伺服器不會與 RX 同步處理。
* Windows Server 2008 R2 網域控制站在復原後未啟動。
* 離線同步處理未如預期般運作。
* 在 VM 容錯移轉後，複寫配對刪除有很長的一段時間在設定伺服器主控台中沒有進展。 使用者無法完成容錯回復或繼續作業。
* 一致性作業的整體快照作業已進行最佳化，有助於減少應用程式中斷連接 SQL Server 用戶端等。
* 已提升一致性工具 (VACP.exe) 的效能。 已減少在 Windows 上建立快照所需的記憶體使用量。
* 密碼超過 16 個字元時，推送安裝服務會當機。
* vContinuum 不會在認證修改時檢查和提示輸入新的 vCenter 認證。
* 在 Linux 上，主要目標快取管理員 (cachemgr) 並未從處理伺服器下載檔案。 這會導致複寫配對節流。
* 當所有節點上的實體容錯移轉叢集 (MSCS) 磁碟順序不同時，某些叢集磁碟區不會設定複寫。 必須重新保護叢集才能利用此修正。  
* 在 RX 從 Scout 7.1 升級至 Scout 8.0.1 之後，SMTP 功能並未如預期般運作。
* 已在記錄中新增更多統計資料，以便復原作業追蹤完成此作業所需的時間。
* 已加入來源伺服器上 Linux 作業系統的支援：
  * Red Hat Enterprise Linux (RHEL) 6 Update 7
  * CentOS 6 Update 7
* 設定伺服器和 RX 主控台現在可以針對進入點陣圖模式的配對顯示通知。
* RX 中已加入下列安全性修正：
    * 透過竄改參數略過授權：對不適用使用者的存取權限制。
    * 跨網站偽造要求：實作針對每一頁隨機產生的頁面權杖概念。 這表示相同使用者只有一個單一登入執行個體，而且頁面重新整理無法運作。 相反地，它會重新導向到儀表板。
    * 惡意檔案上傳：將檔案限制於特定副檔名︰z、aiff、asf、avi、bmp、csv、doc、docx、fla、flv、gif、gz、gzip、jpeg、jpg、log、mid、mov、mp3、mp4、mpc、mpeg、mpg、ods、odt、pdf、png、ppt、pptx、pxd、qt、ram、rar、rm、rmi、rmvb、rtf、sdc、sitd、swf、sxc、sxw、tar、tgz、tif、tiff、txt、vsd、wav、wma、wmv、xls、xlsx、xml、zip。
    * 持續性跨網站指令碼：已新增輸入驗證。

### <a name="azure-site-recovery-scout-801-update-2-update-03dec15"></a>Azure Site Recovery Scout 8.0.1 Update 2 (Update 03Dec15)

Update 2 中的修正包括：

* **組態伺服器**：當組態伺服器註冊到 Azure Site Recovery 保存庫時，會發生使 31 天免費計量功能無法正常運作的問題。
* **整合代理程式**：修正 Update 1 中導致 8.0 版升級至 8.0.1 版期間，更新未安裝在主要目標伺服器上的問題。

### <a name="azure-site-recovery-scout-801-update-1"></a>Azure Site Recovery Scout 8.0.1 Update 1
Update 1 包含下列錯誤修正和新功能：

* 每個伺服器執行個體享有 31 天的免費保護。 這可讓您測試功能或設定概念證明。
* 伺服器上的所有作業 (包括容錯移轉和容錯回復) 在前 31 天都是免費的。 此時間是從第一次使用 Site Recovery Scout 保護伺服器開始計算。 從第 32 天起，每部受保護的伺服器都會依照標準執行個體費率，向客戶擁有的網站收取 Site Recovery 保護費用。
* 在保存庫的 [儀表板] 上，隨時可取得目前計費的受保護伺服器數目。
* 新增對 vSphere 命令列介面 (vCLI) 5.5 Update 2 的支援。
* 新增對來源伺服器上下列 Linux 作業系統的支援：
    * RHEL 6 Update 6
    * RHEL 5 Update 11
    * CentOS 6 Update 6
    * CentOS 5 Update 11
* 用以處理下列問題的錯誤修正：
  * 設定伺服器或 RX 伺服器的保存庫註冊失敗。
  * 當叢集 VM 在恢復期間重新受到保護時，叢集磁碟區就不會如預期般出現。
  * 當主要目標伺服器裝載於與內部部署實際執行 VM 不同的 ESXi 伺服器時，容錯回復會失敗。
  * 升級至 8.0.1 時，設定檔權限會變更。 這項變更會影響保護和作業。
  * 不會如預期般強制執行重新同步處理閾值，因而導致不一致的複寫行為。
  * RPO 設定並未正確地顯示在設定伺服器主控台中。 未壓縮的資料值未正確地顯示壓縮的值。
  * 未如預期在 vContinuum 精靈中刪除移除作業，而且未從設定伺服器主控台中刪除複寫作業。
  * 在 vContinuum 精靈中，於 MSCS VM 的保護期間，按一下磁碟檢視中的 [詳細資料]，磁碟會自動取消選取。
  * 在實體到虛擬 (P2V) 案例中，必要的 HP 服務 (例如 CIMnotify 和 CqMgHost) 不會移至 VM 復原中的 [手動]。 此問題會導致額外的開機時間。
  * 主要目標伺服器上有超過 26 個磁碟時，Linux VM 的保護就會失敗。

