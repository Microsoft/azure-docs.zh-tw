---
title: VMware 到 Azure 的 Azure Site Recovery Deployment Planner | Microsoft Docs
description: 這是 Azure Site Recovery Deployment Planner 使用者指南。
services: site-recovery
documentationcenter: ''
author: nsoneji
manager: garavd
editor: ''
ms.assetid: ''
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 07/06/2018
ms.author: nisoneji
ms.openlocfilehash: 0311fed95adf7005512f0b675031ab90657c963a
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47159953"
---
# <a name="azure-site-recovery-deployment-planner-for-vmware-to-azure"></a>VMware 到 Azure 的 Azure Site Recovery 部署規劃工具
此文章是 VMware 到 Azure 生產部署的 Azure Site Recovery 部署規劃工具使用者指南。

## <a name="overview"></a>概觀

使用 Azure Site Recovery 開始保護任何 VMware 虛擬機器 (VM) 之前，請根據每日資料變化率來配置足夠的頻寬，以符合您所需的復原點目標 (RPO)。 務必在內部部署環境中部署適當的組態伺服器和處理序伺服器數目。

您也需要建立正確的目標 Azure 儲存體帳戶類型和數目。 您可建立標準或進階儲存體帳戶，將使用量隨時間增加所造成的來源生產伺服器成長納入考量。 您可以根據工作負載特性 (例如，每秒的讀/寫 I/O 作業 [IOPS]，或資料變換) 和 Site Recovery 限制，選擇每部 VM 的儲存體類型。

 Site Recovery 部署規劃工具是一個命令列工具，適用於 Hyper-V 到 Azure 與 VMware 到 Azure 的災害復原案例。 您可以使用此工具從遠端分析 VMware VM (完全不影響實際運作)，以了解成功複寫與測試容錯移轉的頻寬和儲存體需求。 您不需安裝任何 Site Recovery 內部部署元件，即可執行此工具。 若要取得精確的已達成輸送量結果，可在符合 Site Recovery 組態伺服器最低需求的 Windows Server 上執行規劃工具，而該伺服器是最後您必須在生產部署前幾個步驟中部署的。

此工具提供下列詳細資料︰

**相容性評估**

* 以磁碟數目、磁碟大小、IOPS、變換、開機類型 (EFI/BIOS) 和作業系統版本為基礎的 VM 合適性評估

**網路頻寬需求與 RPO 評估**

* 差異複寫所需的預估網路頻寬
* Site Recovery 可以從內部部署至 Azure 取得的輸送量
* 要劃分批次的 VM 數目 (根據在給定時間量內完成初始複寫所需的預估頻寬)
* 給定頻寬可達成的 RPO
* 對於所需 RPO 的影響 (如果佈建的頻寬比較低)：

**Azure 基礎結構需求**

* 每部 VM 的儲存體類型 (標準或進階儲存體帳戶) 需求
* 為了複寫所要設定的標準和進階儲存體帳戶總數
* 以儲存體指引為基礎的儲存體帳戶命名建議
* 所有 VM 的儲存體帳戶配置
* 在訂用帳戶上進行測試容錯移轉或容錯移轉之前所要設定的 Azure 核心數目
* 每個內部部署 VM 的 Azure VM 建議大小

**內部部署基礎結構需求**
* 要在內部部署環境中部署的必要組態伺服器和處理序伺服器數目

**向 Azure 進行災害復原的估計成本**
* 向 Azure 進行災害復原的總估計成本：計算、儲存體、網路和 Azure Site Recovery 授權成本
* 每一 VM 的詳細成本分析


>[!IMPORTANT]
>
>因為使用量可能會隨時間增加，所以執行所有上述工具計算時，假設工作負載特性中的成長因子為 30%。 計算也會使用所有分析計量 (讀/寫 IOPS 和變換等) 的第 95 個百分位數值。 成長因子和百分位數計算皆可設定。 若要深入了解成長因子，請參閱「成長因子考量」一節。 若要深入了解百分位數值，請參閱「用於計算的百分位數值」一節。
>

## <a name="support-matrix"></a>支援矩陣

| | **VMware 至 Azure** |**Hyper-V 至 Azure**|**Azure 至 Azure**|**Hyper-V 至次要網站**|**VMware 至次要網站**
--|--|--|--|--|--
支援的案例 |是|是|否|是*|否
支援的版本 | vCenter 6.5、6.0 或 5.5| Windows Server 2016、Windows Server 2012 R2 | NA |Windows Server 2016、Windows Server 2012 R2|NA
支援的設定|vCenter、ESXi| Hyper-V 叢集、Hyper-V 主機|NA|Hyper-V 叢集、Hyper-V 主機|NA|
每個執行中的 Site Recovery 部署規劃工具執行個體可以分析的伺服器數目 |單一 (屬於一個 vCenter Server 或一個 ESXi 伺服器的 VM 可同時加以分析)|多個 (跨多部主機或主機叢集的 VM 可同時加以分析)| NA |多個 (跨多部主機或主機叢集的 VM 可同時加以分析)| NA

*此工具主要用於 Hyper-V 到 Azure 的災害復原案例。 針對 Hyper-V 到次要站台的災害復原，此工具只能用來了解來源端建議，例如所需的網路頻寬、每個來源 Hyper-V 伺服器上所需的可用儲存體空間，以及初始複寫批次處理數目和批次定義。 請忽略報告中的 Azure 建議和成本。 此外，取得輸送量作業不適用於 Hyper-V 到次要站台的災害復原案例。

## <a name="prerequisites"></a>先決條件
此工具有兩個主要階段：分析和報告產生。 另外還有第三個選項：只計算輸送量。 下表列出起始分析 / 輸送量測量之伺服器的需求。

| 伺服器需求 | 說明|
|---|---|
|分析和輸送量測量| <ul><li>作業系統：Windows Server 2016 或 Windows Server 2012 R2<br>(最好至少符合[組態伺服器的大小建議](https://aka.ms/asr-v2a-on-prem-components))</li><li>機器組態︰8 個 vCPU、16 GB RAM、300 GB HDD</li><li>[.NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[VMware vSphere PowerCLI 6.0 R3](https://aka.ms/download_powercli)</li><li>[適用於 Visual Studio 2012 的 Visual C++ 可轉散發套件](https://aka.ms/vcplusplus-redistributable)</li><li>透過網際網路從這部伺服器存取 Azure</li><li>Azure 儲存體帳戶</li><li>伺服器的系統管理員存取權</li><li>100 GB 的可用磁碟空間下限 (假設分析平均各有 3 個磁碟的 1,000 部 VM 30 天)</li><li>VMware vCenter 統計資料層級設定可以是 1 或更高層級</li><li>允許 vCenter 連接埠 (預設為 443)：Site Recovery 部署規劃工具使用此連接埠來連線至 vCenter 伺服器/ESXi 主機</ul></ul>|
| 報告產生 | 具有 Excel 2013 或更新版本的 Windows 電腦或 Windows Server。<li>[.NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[適用於 Visual Studio 2012 的 Visual C++ 可轉散發套件](https://aka.ms/vcplusplus-redistributable)</li><li>只有在您會於報告產生命令中傳遞 -User 選項，以擷取 VM 之最新 VM 設定資訊的情況下，才需要 [VMware vSphere PowerCLI 6.0 R3](https://aka.ms/download_powercli) \(英文\)。 傳遞 -User 選項之後，部署規劃工具便會連線至 vCenter 伺服器。 允許 vCenter 連接埠 (預設為 443) 以連線至 vCenter 伺服器。</li>|
| 使用者權限 | 使用者帳戶的唯讀權限，在分析期間用來存取 VMware vCenter Server/VMware vSphere ESXi 主機 |

> [!NOTE]
>
>此工具只能剖析具有 VMDK 和 RDM 磁碟的 VM。 不能分析具有 iSCSI 或 NFS 磁碟的 VM。 Site Recovery 支援適用於 VMware 伺服器的 iSCSI 和 NFS 磁碟。 因為部署規劃工具不在客體內，而且只使用 vCenter 效能計數器進行剖析，所以此工具看不見這些磁碟類型。
>

## <a name="download-and-extract-the-deployment-planner-tool"></a>下載部署規劃工具並解壓縮
1. 下載最新版的 [Site Recovery 部署規劃工具](https://aka.ms/asr-deployment-planner)。
此工具封裝在 .zip 資料夾中。 目前的工具版本僅支援 VMware 到 Azure 案例。

2. 從您要執行工具的位置，將 .zip 資料夾複製到 Windows Server。
如果伺服器可存取網路以連線到保存要剖析之 VM 的 vCenter Server/vSphere ESXi 主機，您可以從 Windows Server 2012 R2 執行此工具。 不過，我們建議您在硬體設定符合[組態伺服器調整大小方針](https://aka.ms/asr-v2a-on-prem-components)的伺服器上執行此工具。 如果您已在內部部署環境部署 Site Recovery 元件，請從組態伺服器執行此工具。

    我們建議執行此工具的伺服器具有與組態伺服器 (具備內建處理序伺服器) 相同的硬體組態。 這種組態可確保工具報告的達成輸送量符合 Site Recovery 在複寫期間可達到的實際輸送量。 輸送量計算取決於伺服器可用的網路頻寬和伺服器的硬體組態 (CPU、儲存體等等)。 如果您從任何其他伺服器執行此工具，則會計算從該伺服器至 Azure 的輸送量。 此外，因為該伺服器的硬體組態可能會與組態伺服器不同，因為工具報告的達成輸送量可能不正確。

3. 將 .zip 資料夾解壓縮。
此資料夾包含多個檔案和子資料夾。 可執行檔是父資料夾中的 ASRDeploymentPlanner.exe。

    範例：將 .zip 檔案複製到 E:\ 磁碟機並將它解壓縮。
    E:\ASR Deployment Planner_v2.2.zip

    E:\ASR Deployment Planner_v2.2\ASRDeploymentPlanner.exe

### <a name="update-to-the-latest-version-of-deployment-planner"></a>更新至最新版的部署規劃工具
如果您的部署規劃工具是舊版的，請執行下列任一動作：
 * 如果最新版本不包含分析修正程式，並已在您目前的規劃工具版本上進行分析，則會繼續執行分析。
 * 如果最新版本包含分析修正程式，則建議您在目前的版本上停止分析，並使用新版本重新開始分析。


 >[!NOTE]
 >
 >當您開始使用新版本進行分析時，請傳遞相同的輸出目錄路徑，以便工具在現有檔案上附加分析資料。 將使用一組完整的分析資料來產生報告。 如果您傳遞不同的輸出目錄，則會建立新檔案，舊的分析資料不會用來產生報告。
 >
 >每個新的部署規劃工具版本都是 .zip 檔的累積更新。 您不需要將最新的檔案複製到先前的資料夾。 您可以建立及使用新的資料夾。


## <a name="version-history"></a>版本歷程記錄
Site Recovery 部署規劃工具的最新版本為 2.2。
請參閱 [部署規劃工具版本記錄](https://social.technet.microsoft.com/wiki/contents/articles/51049.asr-deployment-planner-version-history.aspx)頁面，了解每個更新中所新增的修正。

## <a name="next-steps"></a>後續步驟
[執行 Site Recovery 部署規劃工具](site-recovery-vmware-deployment-planner-run.md)
