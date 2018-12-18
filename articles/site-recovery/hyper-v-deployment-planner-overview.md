---
title: Hyper-V 到 Azure 的 Azure Site Recovery 部署規劃工具 | Microsoft Docs
description: 這是 Hyper-V 到 Azure 之案例的 Azure Site Recovery 部署規劃工具使用者指南。
services: site-recovery
author: nsoneji
manager: garavd
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.topic: conceptual
ms.date: 10/11/2018
ms.author: nisoneji
ms.openlocfilehash: 7a91217e35ae7a32714478171dcebc002e384fcb
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093585"
---
# <a name="site-recovery-deployment-planner-for-hyper-v-to-azure"></a>Hyper-V 到 Azure 的 Site Recovery 部署規劃工具

本文是 Hyper-V 到 Azure 生產部署的 Azure Site Recovery Deployment Planner 使用者指南。

開始使用 Site Recovery 保護任何 Hyper-V 虛擬機器 (VM) 之前，請根據每日資料變化率來配置足夠的頻寬，以符合您所需的復原點目標 (RPO)，並在內部部署 Hyper-V 儲存體的每個磁碟區上配置足夠的可用儲存體空間。

您也需要建立正確的目標 Azure 儲存體帳戶類型和數目。 您可建立標準或進階儲存體帳戶，將使用量隨時間增加所造成的來源生產伺服器成長納入考量。 您可以根據工作負載特性 (例如，每秒的讀/寫 I/O 作業 (IOPS)，或資料變換) 和 Azure Site Recovery 限制，選擇每部 VM 的儲存體類型。 

Azure Site Recovery 部署規劃工具是一項命令列工具，適用於 Hyper-V 到 Azure 與 VMware 到 Azure 的災害復原案例。 您可以使用此工具從遠端分析多個 Hyper-V 主機上存在的 Hyper-V VM (完全不影響生產作業)，以了解成功複寫與測試容錯移轉/容錯移轉的頻寬和 Azure 儲存體需求。 您不需安裝任何 Azure Site Recovery 內部部署元件，即可執行此工具。 不過，若要獲得準確達到的輸送量結果，建議您用來執行規劃工具的 Windows Server，和您要用來對 Azure 啟用災害復原保護的其中一個 Hyper-V 伺服器，應該要有相同的硬體組態。 

此工具提供下列詳細資料︰

**相容性評估**

* 以磁碟數目、磁碟大小、IOPS、變換和幾個 VM 特性為基礎的 VM 合適性評估。

**網路頻寬需求與 RPO 評估**

* 差異複寫所需的預估網路頻寬
* Azure Site Recovery 可以從內部部署至 Azure 取得的輸送量
* 給定頻寬可達成的 RPO
* 對於所需 RPO 的影響 (如果佈建的頻寬比較低)。

    
**Azure 基礎結構需求**

* 每部 VM 的儲存體類型 (標準或進階儲存體帳戶) 需求
* 為了複寫所要設定的標準和進階儲存體帳戶總數
* 以 Azure 儲存體指引為基礎的儲存體帳戶命名建議
* 所有 VM 的儲存體帳戶放置
* 在訂用帳戶上進行測試容錯移轉或容錯移轉之前所要設定的 Azure 核心數目
* 每個內部部署 VM 的 Azure VM 建議大小

**內部部署基礎結構需求**
* 要成功進行初始複寫和差異複寫，Hyper-V 儲存體的每個磁碟區所需擁有的可用儲存體空間，以便確保 VM 複寫不會對生產應用程式造成任何不想要的停機時間
* 要針對 Hyper-V 複寫設定的最大複製頻率

**初始複寫批次處理指引** 
* 要用於保護的 VM 批次數目
* 每個批次中之 VM 的清單
* 每個批次的保護順序
* 每個批次完成初始複寫的估計時間

**Azure DR 的估計成本**
* Azure DR 的總估計成本：計算、儲存體、網路和 Azure Site Recovery 授權成本
* 每一 VM 的詳細成本分析



>[!IMPORTANT]
>
>因為使用量可能會隨時間增加，所以執行所有上述工具計算時，假設工作負載特性中的成長因子為 30%，並使用所有分析計量 (讀/寫 IOPS、變換等等) 的第 95 個百分位數值。 這兩個元素 (成長因子和百分位數計算) 皆可設定。 若要深入了解成長因子，請參閱「成長因子考量」一節。 若要深入了解百分位數值，請參閱「用於計算的百分位數值」一節。
>

## <a name="support-matrix"></a>支援矩陣

| | **VMware 至 Azure** |**Hyper-V 至 Azure**|**Azure 至 Azure**|**Hyper-V 至次要網站**|**VMware 至次要網站**
--|--|--|--|--|--
支援的案例 |是|yes|否|是*|否
支援的版本 | vCenter 6.5、6.0 或 5.5| Windows Server 2016、Windows Server 2012 R2 | NA |Windows Server 2016、Windows Server 2012 R2|NA
支援的設定|vCenter、ESXi| Hyper-V 叢集、Hyper-V 主機|NA|Hyper-V 叢集、Hyper-V 主機|NA|
每個執行中的 Azure Site Recovery 部署規劃工具執行個體可以分析的伺服器數目 |單一 (屬於一個 vCenter Server 或一個 ESXi 伺服器的 VM 可同時加以分析)|多個 (跨多部主機或主機叢集的 VM 可同時加以分析)| NA |多個 (跨多部主機或主機叢集的 VM 可同時加以分析)| NA

*此工具主要用於 Hyper-V 到 Azure 的災害復原案例。 若為 Hyper-V 到次要站台的災害復原，此工具只能用來了解來源端建議，例如所需的網路頻寬、每個來源 Hyper-V 伺服器上所需的可用儲存體空間，以及初始複寫批次處理數目和批次定義。  請忽略報告中的 Azure 建議和成本。 此外，取得輸送量作業不適用於 Hyper-V 到次要站台的災害復原案例。

## <a name="prerequisites"></a>必要條件
對於 Hyper-V，此工具有三個主要階段：取得 VM 清單、分析，以及產生報告。 另外還有第四個選項：只計算輸送量。 下表顯示必須用來執行不同階段之伺服器的需求：

| 伺服器需求 | 說明 |
|---|---|
|取得 VM 清單、分析，以及測量輸送量 |<ul><li>作業系統：Microsoft Windows Server 2016 或 Microsoft Windows Server 2012 R2 </li><li>機器組態︰8 個 vCPU、16 GB RAM、300 GB HDD</li><li>[Microsoft .NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[適用於 Visual Studio 2012 的 Microsoft Visual C++ 可轉散發套件](https://aka.ms/vcplusplus-redistributable)</li><li>透過網際網路從這部伺服器存取 Azure</li><li>Azure 儲存體帳戶</li><li>伺服器的系統管理員存取權</li><li>100 GB 的可用磁碟空間下限 (假設剖析平均各有 3 個磁碟的 1000 部 VM 30 天)</li><li>用來執行 Azure Site Recovery 部署規劃工具的 VM 必須新增到所有 Hyper-V 伺服器的 TrustedHosts 清單。</li><li>要分析的所有 Hyper-V 伺服器 VM 必須新增到要用來執行此工具之用戶端 VM 的 TrustedHosts 清單。 [進一步了解如何將伺服器新增到 TrustedHosts 清單](#steps-to-add-servers-into-trustedhosts-list)。 </li><li> 請從用戶端上的 PowerShell 或命令列主控台，以系統管理權限執行此工具</ul></ul>|
| 報告產生 | 具有 Microsoft Excel 2013 和更新版本的 Windows PC 或 Windows Server |
| 使用者權限 | 系統管理員帳戶，可在取得 VM 清單和分析作業期間，用來存取 Hyper-V 叢集/Hyper-v 主機。<br>所有必須進行分析之主機所擁有的網域系統管理員帳戶，都應該具有相同認證 (也就是使用者名稱和密碼)
 |

## <a name="steps-to-add-servers-into-trustedhosts-list"></a>將伺服器新增到 TrustedHosts 清單的步驟
1.  要用來從中部署工具的 VM 應該在其 TrustedHosts 清單中擁有所有要進行分析的主機。 若要將用戶端新增到 Trustedhosts 清單，請在 VM 上透過提高權限的 PowerShell 執行下列命令。 此 VM 可以是 Windows Server 2012 R2 或 Windows Server 2016。 

            set-item wsman:\localhost\Client\TrustedHosts -value '<ComputerName>[,<ComputerName>]' -Concatenate

1.  每個必須進行分析的 Hyper-V 主機應該具有：

    a. 其 TrustedHosts 清單中要用來執行此工具的 VM。 在 Hyper-V 主機上透過提高權限的 PowerShell 執行下列命令。

            set-item wsman:\localhost\Client\TrustedHosts -value '<ComputerName>[,<ComputerName>]' -Concatenate

    b. 啟用 PowerShell 遠端處理。

            Enable-PSRemoting -Force

## <a name="download-and-extract-the-deployment-planner-tool"></a>下載部署規劃工具並解壓縮

1.  下載最新版的 [Azure Site Recovery Deployment Planner](https://aka.ms/asr-deployment-planner)。
此工具封裝在 .zip 資料夾中。 同一個工具可同時支援 VMware 到 Azure 和 Hyper-V 到 Azure 災害復原案例。 您也可以將此工具用於 Hyper-V 到次要站台的災害復原案例，但請忽略報告中的 Azure 基礎結構建議。

1.  將 .zip 資料夾複製到您要用來執行此工具的 Windows Server。 您可以在 Windows Server 2012 R2 或 Windows Server 2016 上執行此工具。 伺服器必須能存取網路，以便連線到保有要分析之 VM 的 Hyper-V 叢集或 Hyper-V 主機。 建議您 VM (將用來執行此工具的所在位置) 的硬體設定，要和您想要保護之 Hyper-V 伺服器的硬體設定相同。 這種組態可確保工具報告的達成輸送量符合 Azure Site Recovery 在複寫期間可達到的實際輸送量。 輸送量計算取決於伺服器可用的網路頻寬和伺服器的硬體組態 (CPU、儲存體等等)。 系統會計算從工具執行所在之伺服器到 Azure 的輸送量。 如果該伺服器的硬體設定與 Hyper-V 伺服器的設定不同，工具所報告的達成輸送量不會正確。
建議的 VM 設定：8 個 vCPU、16 GB RAM、300 GB HDD。

1.  將 .zip 資料夾解壓縮。
此資料夾包含多個檔案和子資料夾。 可執行檔是父資料夾中的 ASRDeploymentPlanner.exe。

範例：將 .zip 檔案複製到 E:\ 磁碟機並將它解壓縮。 E:\ASR Deployment Planner_v2.2.zip

E:\ASR Deployment Planner_v2.2\ASRDeploymentPlanner.exe

### <a name="updating-to-the-latest-version-of-deployment-planner"></a>更新至最新版的部署規劃工具
如果您的部署規劃工具是舊版的，請執行下列任一動作：
 * 如果最新版本不包含分析修正程式，並已在您目前的規劃工具版本上進行分析，則會繼續執行分析。
 * 如果最新版本包含分析修正程式，則建議您在目前的版本上停止分析，並使用新版本重新開始分析。


  >[!NOTE]
  >
  >當您開始使用新版本進行分析時，請傳遞相同的輸出目錄路徑，以便工具在現有檔案上附加分析資料。 將使用一組完整的剖析資料來產生報告。 如果您傳遞不同的輸出目錄，則會建立新檔案，舊的剖析資料不會用來產生報告。
  >
  >每個新的 Deployment Planner 都是 .zip 檔的累積更新。 您不需要將最新的檔案複製到先前的資料夾。 您可以建立及使用新的資料夾。

## <a name="version-history"></a>版本歷程記錄
ASR 部署規劃工具的最新版本為 2.2。
請參閱 [ASR 部署規劃工具版本記錄](https://social.technet.microsoft.com/wiki/contents/articles/51049.asr-deployment-planner-version-history.aspx)頁面，了解每個更新中所新增的修正。


## <a name="next-steps"></a>後續步驟
* [執行部署規劃工具](site-recovery-hyper-v-deployment-planner-run.md)。
