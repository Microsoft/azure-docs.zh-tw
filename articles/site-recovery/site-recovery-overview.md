---
title: 關於 Azure Site Recovery | Microsoft Docs
description: 提供 Azure Site Recovery 服務的概觀，並摘要說明部署案例。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: overview
ms.date: 10/10/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 442e8d425e60566df5ce919e3463235ef1d6e4f0
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078091"
---
# <a name="about-site-recovery"></a>關於 Site Recovery

歡迎使用 Azure Site Recovery 服務！ 本文提供快速服務概觀。

以一個組織而言，您必須採用商務持續性與災害復原 (BCDR) 策略，以便在計劃或未計劃中斷發生時，可以保護您資料的安全，並使您的應用程式和工作負載保持運作。

Azure 復原服務會以下列方式參與您的 BCDR 策略：

- **Site Recovery 服務**：Site Recovery 可協助確保商務持續性，方法是在中斷期間讓商務應用程式和工作負載保持執行狀態。 Site Recovery 會將實體和虛擬機器 (VM) 上執行的工作負載從主要站台複寫到次要位置。 當您的主要站台發生中斷時，您會容錯移轉到次要位置，並且從該處存取應用程式。 在主要位置再次執行之後，您就可以容錯回復。  
- **備份服務**：[Azure 備份](https://docs.microsoft.com/azure/backup/)服務會將您的資料備份至 Azure，維持資料安全且更容易復原。

Site Recovery 可以管理複寫：

- 在 Azure 區域間複寫 Azure VM。
- 將內部部署 VM 和實體伺服器複寫至 Azure 或次要站台。


## <a name="what-does-site-recovery-provide"></a>Site Recovery 可以提供什麼功能？


**功能** | **詳細資料**
--- | ---
**簡單的 BCDR 解決方案** | 您可以使用 Site Recovery 從 Azure 入口網站中的單一位置設定及管理複寫、容錯移轉和容錯回復。
**Azure VM 複寫** | 您可以設定從主要區域到次要區域的 Azure VM 災害復原。
**內部部署 VM 複寫** | 您可以將內部部署 VM 和實體伺服器複寫至 Azure 或次要內部部署資料中心。 複寫至 Azure 可排除維護次要資料中心的成本和複雜度。
**工作負載複寫** | 複寫在支援的 Azure VM、內部部署 Hyper-V 和VMware VM，以及 Windows/Linux 實體伺服器上執行的任何工作負載。
**資料恢復功能** | Site Recovery 會協調複寫，且無須攔截應用程式資料。 當您複寫到 Azure 時，資料會儲存在 Azure 儲存體中，並具備所提供的恢復功能。 進行容錯移轉時，會根據複寫的資料來建立 Azuer VM。
**RTO 和 RPO 目標** | 將復原時間目標 (RTO) 和復原點目標 (RPO) 保持在組織的限制範圍內。 Site Recovery 為 Azure VM 和 VMware VM 提供了連續複寫功能，並為 Hyper-V 提供最低 30 秒的複寫頻率。 您可以藉由與 [Azure 流量管理員](https://azure.microsoft.com/blog/reduce-rto-by-using-azure-traffic-manager-with-azure-site-recovery/)整合，進一步縮減 RTO。
**透過容錯移轉讓應用程式保持一致** | 您可以使用具有與應用程式一致的快照集復原點來複寫。 這些快照集會擷取磁碟資料、記憶體中的所有資料和處理序中的所有交易。
**測試不中斷** | 您可以輕鬆執行災害復原演練，不會影響進行中的複寫。
**彈性容錯移轉** | 您可以執行計劃性容錯移轉，因為是預期中的中斷，所以不會遺失任何資料；也可以執行非計劃性容錯移轉，以在發生未預期的災害時將資料損失減到最少 (取決於複寫頻率)。 您可以在主要站台再次可用時，輕鬆地容錯回復到該站台。
**自訂復原方案** | 使用復原方案，可以自訂及排序多部 VM 上執行之多層應用程式的容錯移轉和復原。 您在復原計劃中將機器群組在一起，並且選擇性地新增指令碼和手動動作。 復原計劃可以與 Azure 自動化 Runbook 整合。
**BCDR 整合** | Site Recovery 與其他 BCDR 技術整合。 例如，您可以使用 Site Recovery 保護公司工作負載的 SQL Server 後端，具有原生支援 SQL Server AlwaysOn 以便管理可用性群組的容錯移轉。
**Azure 自動化整合** | 豐富的 Azure Automation 文件庫，提供已可用於生產環境，且可下載並已經與 Site Recovery 整合的應用程式特定指令碼。
**網路整合** | Site Recovery 與 Azure 整合，提供簡單的應用程式網路管理，包括保留 IP 位址、設定負載平衡器和整合 Azure 流量管理員以進行有效率的網路轉換。


## <a name="what-can-i-replicate"></a>我可以複寫哪些項目？

**支援** | **詳細資料**
--- | ---
**複寫案例** | 將 Azure VM 從一個 Azure 區域複寫至另一個 Azure 區域。<br/><br/>  將內部部署 VMware VM、Hyper-V VM、實體伺服器 (Windows 和 Linux) 複寫至 Azure。<br/><br/> 將 System Center VMM 管理的內部部署 VMware VM、Hyper-V VM 和實體伺服器複寫至次要站台。
**區域** | 檢閱 Site Recovery 的[支援區域](https://azure.microsoft.com/regions/services/)。 |
**複寫的機器** | 檢閱 [Azure VM](azure-to-azure-support-matrix.md#support-for-replicated-machine-os-versions) 複寫、[內部部署 VM 和實體伺服器](vmware-physical-azure-support-matrix.md#replicated-machines)及[內部部署 Hyper-V VM](hyper-v-azure-support-matrix.md#replicated-vms) 的複寫需求。
**VMware 伺服器/主機** | 您想要複寫的 VMware VM 可以位於[支援的主機和虛擬伺服器](vmware-physical-azure-support-matrix.md)。
**工作負載** | 您可以複寫在支援複寫之機器上執行的所有工作負載。 此外，Site Recovery 小組已經針對[應用程式數目](site-recovery-workload.md#workload-summary)執行應用程式專屬測試。



## <a name="next-steps"></a>後續步驟
* 深入了解[工作負載支援](site-recovery-workload.md)。
* 開始[不同區域間的 Azure VM 複寫](azure-to-azure-quickstart.md)。 
