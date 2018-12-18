---
title: VMware 到 Azure 的 Azure Site Recovery Deployment Planner | Microsoft Docs
description: 這是 Azure Site Recovery Deployment Planner 使用者指南。
services: site-recovery
author: nsoneji
manager: garavd
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/11/2018
ms.author: nisoneji
ms.openlocfilehash: 7470dab537a8ab248228b9c445e1124e7ec9c444
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387798"
---
# <a name="cost-estimation-report-of-azure-site-recovery-deployment-planner"></a>Azure Site Recovery 部署規劃工具的成本估計報告  

部署規劃工具報告在[建議](site-recovery-vmware-deployment-planner-analyze-report.md#recommendations)工作表中提供成本估計摘要，以及在成本估計工作表中提供詳細的成本分析。 其中包含每部 VM 的詳細成本分析。 

### <a name="cost-estimation-summary"></a>成本估計摘要 
此圖形會就您所選的目標區域和您針對報告產生所指定的貨幣，顯示災害復原 (DR) 至 Azure 之估計總成本的摘要檢視。
成本估計摘要

![成本估計摘要](media/site-recovery-vmware-deployment-planner-analyze-report/cost-estimation-summary-v2a.png)

當您使用 Azure Site Recovery 在 Azure 中保護所有相容 VM 時，此摘要可協助您了解您需要支付的儲存體、計算、網路和授權成本。 成本是針對相容的 VM，而不是針對所有已分析的 VM 計算。  
 
您可以檢視每月或每年的成本。 深入了解[支援的目標區域](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-target-regions)和[支援的貨幣](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-currencies)。

**依元件的成本** DR 總成本分成四個元件：計算、儲存體、網路和 Azure Site Recovery 授權成本。 此成本是根據在複寫期間和在 DR 演練時產生的耗用量來計算，其中涵蓋計算、儲存體 (進階和標準)、在內部部署網站與 Azure 之間設定的 ExpressRoute/VPN，以及 Azure Site Recovery 授權。

**依狀態的成本** 災害復原 (DR) 的總成本是根據兩個不同的狀態來分類 - 複寫和 DR 演練。 

**複寫成本**：在複寫期間產生的成本。 其中涵蓋儲存體、網路和 Azure Site Recovery 授權的成本。 

**DR 演練成本**：在測試容錯移轉期間產生的成本。 Azure Site Recovery 會在測試容錯移轉期間啟動 VM。 DR 演練成本涵蓋執行 VM 的計算和儲存成本。 

**每月/年的 Azure 儲存成本** 它會顯示將會針對複寫及 DR 演練的進階和標準儲存體產生的總儲存成本。

## <a name="detailed-cost-analysis"></a>詳細成本分析
計算、儲存體、網路等的 Azure 價格因 Azure 地區而有所不同。 您可以產生成本估計報告，其中包含以您的訂用帳戶為基礎的最新 Azure 價格 (以指定的貨幣計價)，與您的訂用帳戶相關聯的供應項目，以及適用於指定之目標 Azure 區域的供應項目。 根據預設，此工具會使用美國西部 2 Azure 區域和美元 (USD) 貨幣。 如果您已使用其他區域和貨幣，當您下次產生不具訂用帳戶 ID、供應項目識別碼、目標區域和貨幣的報告時，，它會使用上次使用之目標區域和上次使用之貨幣的價格進行成本估計。
此區段會顯示您用於報告產生的訂用帳戶識別碼和供應項目識別碼。  如果未使用，則為空白。

在整份報告中，以灰色標示的資料格為唯讀。 您可以根據自己的需求修改白色資料格。

![成本估計詳細資料 1](media/site-recovery-hyper-v-deployment-planner-cost-estimation/cost-estimation1-h2a.png)

### <a name="overall-dr-cost-by-components"></a>依元件的整體 DR 成本
第一個區段顯示依元件的整體 DR 成本和依狀態的 DR 成本。 

**計算**：針對 DR 需求在 Azure 上執行的 IaaS VM 成本。 其中包含 Azure Site Recovery 在 DR 演練 (測試容錯移轉) 期間建立的 VM，以及在 Azure 上執行的 VM，例如具有 Always On 可用性群組和網域控制站 / 網域名稱伺服器的 SQL Server。

**儲存體**：針對 DR 需求的 Azure 儲存體耗用量成本。 其中包含針對複寫以及在 DR 演練期間的儲存體耗用量。
網路：針對 DR 需求的 ExpressRoute 和站對站 VPN 成本。 

**ASR 授權**：所有相容 VM 的 Azure Site Recovery 授權成本。 如果您已在詳細的成本分析表格中手動輸入 VM，則 Azure Site Recovery 授權成本也包含該 VM。

### <a name="overall-dr-cost-by-states"></a>依狀態的整體 DR 成本
DR 總成本是根據兩個不同的狀態來分類 - 複寫和 DR 演練。

**複寫成本**：在複寫時發生的成本。 其中涵蓋儲存體、網路和 Azure Site Recovery 授權的成本。 

**DR 演練成本**：DR 演練時發生的成本。 Azure Site Recovery 會在 DR 演練期間啟動 VM。 DR 演練成本涵蓋執行 VM 的計算和儲存成本。
一年中的總計 DR 演練持續時間 = DR 演練的數目 x 每個 DR 演練持續時間 (天) 平均 DR 演練成本 (每月) = 總計 DR 演練成本 / 12

### <a name="storage-cost-table"></a>儲存成本表格：
下表顯示針對複寫與 DR 演練的進階和標準儲存成本 (不論是否有折扣)。

### <a name="site-to-azure-network"></a>站對 Azure 網路
根據您的需求，選取適當的設定。 

**ExpressRoute**：根據預設，此工具會選取符合差異複寫所需網路頻寬的最接近 ExpressRoute 計劃。 您可以根據您的需求變更計劃。

**VPN 閘道**：如果您的環境中有任何，請選取 VPN 閘道。 根據預設，這是 NA。

**目標區域**：針對 DR 指定的 Azure 區域。 報告中針對計算、儲存體、網路和授權使用的價格是以該地區的 Azure 價格為基礎。 

### <a name="vm-running-on-azure"></a>在 Azure 上執行的 VM
如果您有任何網域控制站或 DNS VM 或 SQL Server VM 具有在 Azure 上針對 DR 執行的 Always On 可用性群組，您可以提供 VM 數目和大小，以在 DR 總成本中考慮其計算成本。 

### <a name="apply-overall-discount-if-applicable"></a>套用整體折扣 (適用的話)
如果您是 Azure 夥伴或客戶，而且有資格享有整體 Azure 價格的任何折扣，您可以使用此欄位。 此工具會對所有元件套用折扣 (%)。

### <a name="number-of-virtual-machines-type-and-compute-cost-per-year"></a>虛擬機器的數目和計算成本 (每年)
此表顯示 Windows 和非 Windows VM 的數目，以及其 DR 演練計算成本。

### <a name="settings"></a>設定 
**使用受控磁碟**：它會指定在 DR 演練時是否正在使用受控磁碟。 預設值為 [是]。 如果您已將 -UseManagedDisks 設為 [否]，它會使用非受控磁碟價格進行成本計算。

**貨幣**：用於產生報告的貨幣。 成本持續時間：您可以檢視該月份或整年的所有成本。 

## <a name="detailed-cost-analysis-table"></a>詳細的成本分析表格
![詳細的成本分析](media/site-recovery-hyper-v-deployment-planner-cost-estimation/detailed-cost-analysis-h2a.png) 此表格列出每個相容 VM 的成本細目。 您也可以使用此表格，手動新增 VM 來取得未剖析 VM 的估計 Azure DR 成本。 在您需要評估新災害復原部署的 Azure 成本，而不需進行詳細分析的情況下，這很有用。
若要手動新增 VM： 
1.  按一下 [插入資料列] 按鈕，在 [開始] 與 [結束] 資料列之間插入新資料列。

2.  根據符合此組態的的大約 VM 大小和 VM 數目，填入下列資料行： 

* VM 數目、IaaS 大小 (您的選擇)
* 儲存體類型 (標準/進階)
* VM 總儲存體大小 (GB)
* 一年中的 DR 演練次數 
* 每次 DR 演練持續時間 (天) 
* OS 類型
* 資料備援 
* Azure Hybrid Benefit

3.  針對 [一年中的 DR 演練次數]、[每次 DR 演練持續時間 (天)]、[資料備援] 及 [Azure Hybrid Use Benefit]，按一下 [套用到全部] 按鈕，即可將同一個值套用至表格中的所有 VM。

4.  按一下 [重新計算成本] 來更新成本。

**VM 名稱**：VM 的名稱。

**VM 數目**：符合組態的 VM 數目。 如果類似組態的 VM 未經過剖析，但是將會受到保護，您可以更新現有的 VM 數目。

**IaaS 大小 (建議)**：這是工具建議之相容 VM 的 VM 角色大小。 

**IaaS 大小 (您的選擇)**：根據預設，這與建議的 VM 角色大小一樣。 您可以根據您的需求變更角色。 計算成本是以您選取的 VM 角色大小為基礎。

**儲存體類型**：VM 所使用的儲存體類型。 這是標準或進階儲存體。

**VM 總儲存體大小 (GB)**：VM 的總儲存體。

**一年中的 DR 演練次數**：您在一年中執行 DR 演練的次數。 根據預設，一年中有 4 次。 在頂端列輸入新值，然後按一下 [套用到全部] 按鈕，即可修改特定 VM 的期間，或將新值套用至所有 VM。 根據一年中的 DR 演練次數和每次 DR 演練持續期間，計算 DR 演練總成本。  

**每次 DR 演練持續時間 (天)**：每次 DR 演練的持續時間。 根據預設，依照[災害復原軟體保證權益](https://azure.microsoft.com/pricing/details/site-recovery)，這是每隔 90 天持續 7 天。 在頂端列輸入新值，然後按一下 [套用到全部] 按鈕，即可修改特定 VM 的期間，或將新值套用至所有 VM。 根據一年中的 DR 演練次數和每次 DR 演練持續期間，計算 DR 演練總成本。
  
**OS 類型**：VM 的 OS 類型。 這是 Windows 或 Linux。 如果 OS 類型是 Windows，則 Azure Hybrid Use Benefit 可以套用到該 VM。 

**資料備援**：可以是下列任何一項 - 本地備援儲存體 (LRS)、異地備援儲存體 (GRS) 和讀取權限異地備援儲存體 (RA-GRS)。 預設值為 LRS。 變更頂端列的類型，然後按一下 [套用到全部] 按鈕，即可根據您的儲存體帳戶來變更特定 VM 的類型，也可以將新類型套用至所有 VM。  根據您選取的資料備援價格，計算複寫的儲存成本。 

**Azure Hybrid Benefit**：您可以將 Azure Hybrid Benefit 套用至 Windows VM (適用的話)。  預設值為 [是]。 按一下 [套用到全部] 按鈕，即可變更特定 VM 的設定，或更新所有 VM。

**Azure 總耗用量**：其中包含您的 DR 的計算、儲存及 Azure Site Recovery 授權成本。 根據您的選取項目，顯示每月或每年的成本。

**穩定狀態複寫成本**：包含複寫的儲存成本。

**DR 演練總成本 (平均)**：其中包含 DR 演練的計算和儲存成本。

**ASR 授權成本**：Azure Site Recovery 授權成本。

## <a name="supported-target-regions"></a>支援的目標區域
Azure Site Recovery 部署規劃工具會提供下列 Azure 區域的成本估計。 如果您的區域未列在下方，您可以使用價格最接近您的區域的下列任何區域。

eastus、eastus2、uswest、centralus、northcentralus、southcentralus、northeurope、westeurope、eastasia、southeastasia、japaneast、japanwest、australiaeast、australiasoutheast、brazilsouth、southindia、centralindia、westindia、canadacentral、canadaeast、westus2、westcentralus、uksouth、ukwest、koreacentral、koreasouth 

## <a name="supported-currencies"></a>支援的貨幣
Azure Site Recovery 部署規劃工具可以使用下列任何貨幣來產生成本報告。

|貨幣|名稱||貨幣|名稱||貨幣|名稱|
|---|---|---|---|---|---|---|---|
|ARS 披索|阿根廷披索 ($)||澳幣|澳洲元 ($)||巴西里耳|巴西里耳 (R$)|
|加拿大幣|加幣 ($)||瑞士法郎|瑞士法郎 (chf)||丹麥幣|丹麥克朗 (kr)|
|歐元|歐元 (€)||英鎊|英鎊 (£)||港幣|港幣 (HK$)|
|印尼盾|印尼盾 (Rp)||印度幣|印度盧比 (₹)||日圓|日圓 (¥)|
|韓元|Korean Won (₩)||墨西哥批索|Mexican Peso (MX$)||馬來西亞林吉特|馬來西亞林吉特 (RM$)|
|挪威克朗|挪威克朗 (kr)||紐西蘭幣|紐西蘭元 ($)||盧布|俄羅斯盧布 (руб)|
|SAR 里亞爾|沙烏地里亞爾 (SR)||瑞典克朗|瑞典克朗 (kr)||新台幣|新台幣 (NT$)|
|土耳其里拉|土耳其里拉 (TL)||美元| 美元 ($)||南非幣|南非蘭特 (R)|

## <a name="next-steps"></a>後續步驟
深入了解[使用 Azure Site Recovery 在 Azure 中保護 VMware](https://docs.microsoft.com/azure/site-recovery/tutorial-vmware-to-azure)。
