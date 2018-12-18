---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 08/14/2018
ms.author: cynthn;kareni
ms.custom: include file
ms.openlocfilehash: 130cc66831b25621cb022eb19005c624fcd71b9e
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "40105501"
---
**上次文件更新**：太平洋標準時間 2018 年 8 月 14 日上午 10:00。

揭露的[新 CPU 漏洞類別](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002) (也稱為推測性執行旁路攻擊)，讓許多客戶不知所措。  

Microsoft 已在我們所有的雲端服務上部署防護功能。 執行 Azure 的基礎結構，以及將客戶的工作負載互相隔離這方面已受到保護。 這表示，使用相同基礎結構的潛在攻擊者不能利用這些漏洞來攻擊您的應用程式。

Azure 會盡可能使用[記憶體保留維護](https://docs.microsoft.com/azure/virtual-machines/windows/maintenance-and-updates#memory-preserving-maintenance)，將客戶的影響降到最低，以及避免重新開機。 Azure 會在對主機進行全系統更新時，繼續使用這些方法，並且保護我們的客戶。

如何將安全性整合至 Azure 各個層面的詳細資訊位於 [Azure 安全性文件](https://docs.microsoft.com/azure/security/)網站。 

> [!NOTE] 
> 自從這份文件首次發行以來，已揭露此弱點類別的多個變體。 Microsoft 持續致力於保護客戶和提供指引。 我們會持續發行進一步的修正，而此頁面也會隨之更新。 
> 
> 在 2018 年 8 月 14 日，業界揭露了新的推測性執行旁路弱點，亦稱為 [L1 終端故障](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180018) (L1TF)，其已獲派多個 CVE ([CVE-2018-3615、CVE-2018-3620 和 CVE-2018-3646](https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00161.html))。 此弱點會影響 Intel® Core® 處理器和 Intel® Xeon® 處理器。 Microsoft 已在我們的雲端服務上部署防護功能，可加強客戶之間的隔離。 請往下閱讀，以了解防禦 L1TF 和先前弱點 ([Spectre Variant 2 CVE-2017-5715 和 Meltdown Variant 3 CVE-2017-5754](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution)) 的其他指引。
>  






## <a name="keeping-your-operating-systems-up-to-date"></a>讓您的作業系統維持最新狀態

雖然無需更新作業系統，就能讓 Azure 上執行的應用程式與其他 Azure 客戶隔離，但最佳做法是讓軟體維持在最新狀態。 Windows 最新的安全性彙總套件包含數個推測性執行旁路弱點的防護功能。 同樣地，Linux 散發套件已發行多個更新來解決這些弱點。 以下是我們對更新作業系統提供的建議動作：

| 供應項目 | 建議的動作  |
|----------|---------------------|
| Azure 雲端服務  | 啟用[自動更新](https://docs.microsoft.com/azure/cloud-services/cloud-services-how-to-configure-portal)，或確定您執行最新的客體作業系統。 |
| Azure Linux 虛擬機器 | 安裝來自作業系統提供者的更新。 如需詳細資訊，請參閱本文後面的 [Linux](#linux)。 |
| Azure Windows 虛擬機器  | 安裝最新的安全性彙總套件。
| 其他 Azure PaaS 服務 | 使用這些服務的客戶無需採取任何行動。 Azure 會自動將您的作業系統版本保持在最新狀態。 |

## <a name="additional-guidance-if-you-are-running-untrusted-code"></a>如果您執行的是不受信任的程式碼的其他指引 

如果客戶允許不受信任的使用者執行任意程式碼，則可以建議他們在 Azure 虛擬機器或雲端服務內實作一些額外的安全性功能。 這些功能可抵禦多個推測性執行弱點描述的程序內洩漏向量。

建議使用額外安全性功能的案例：

- 您允許不受信任的程式碼在 VM 內執行。  
    - 例如，您允許其中一個客戶上傳二進位檔或指令碼，然後在您的應用程式內執行。 
- 您允許不受信任的使用者以低權限帳戶登入 VM。   
    - 例如，您允許低權限的使用者透過遠端桌面或 SSH 登入您其中一個 VM。  
- 允許不受信任的使用者存取透過巢狀虛擬化實作的虛擬機器。  
    - 例如，您控制 HYPER-V 主機，但將 VM 配置給不受信任的使用者。 

如果客戶實作的案例並未包含不受信任的程式碼，則無須啟用這些額外的安全性功能。 

## <a name="enabling-additional-security"></a>啟用額外的安全性 

您可以在 VM 或雲端服務內啟用額外的安全性功能。

### <a name="windows"></a>Windows 

目標作業系統必須是最新版本，才能啟用這些額外的安全性功能。 雖然許多推測性執行旁路攻擊的防護功能會預設為啟用，但此處描述的其他功能必須手動啟用，並可能會對效能造成影響。 

**步驟 1**：[連絡 Azure 支援](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical)，將更新的韌體 (微碼) 公開到您的虛擬機器中。 

**步驟 2**：啟用核心虛擬位址遮蔽功能 (Kernel Virtual Address Shadowing，KVAS) 和分支目標注入 (Branch Target Injection，BTI) 作業系統支援。 請依照 [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) 中的指示，透過 `Session Manager` 登錄機碼啟用保護功能。 需要重新開機。 

**步驟 3**：針對使用[巢狀虛擬化](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization)的部署 (僅限 D3 和 E3)：這些指示適用於作為 HYPER-V 主機的 VM。 

1. 請依照 [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) 中的指示，透過 `MinVmVersionForCpuBasedMitigations` 登錄機碼啟用保護功能。  
 
1. 請遵循[這裡](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/manage-hyper-v-scheduler-types)的指示，將 Hypervisor 排程器類型設定為**核心**。 

**步驟 4**：遵循 [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) 中的指示，使用 [SpeculationControl](https://aka.ms/SpeculationControlPS) PowerShell 模組確認保護功能已啟用。 

> [!NOTE]
> 如果您先前已下載此模組，則必須安裝最新版本。
>

所有 VM 應該會顯示：

```
branch target injection mitigation is enabled: True

kernel VA shadow is enabled: True  

L1TFWindowsSupportEnabled: True
```


### <a name="linux"></a>Linux

<a name="linux"></a>若要在內部啟用一組額外安全性功能，目標作業系統必須完全是最新版本。 有些防護功能會預設為啟用。 下節會描述預設為關閉的功能，以及 (或) 相依於硬體支援 (微碼) 的功能。 啟用這些功能可能會對效能造成影響。 請參閱作業系統提供者的文件，以取得進一步的指示
 
**步驟 1**：[連絡 Azure 支援](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical)，將更新的韌體 (微碼) 公開到您的虛擬機器中。
 
**步驟 2**：遵循作業系統提供者的文件，啟用分支目標注入 (BTI) 作業系統支援來減輕 CVE-2017-5715 (Spectre Variant 2) 的影響。 
 
**步驟 3**：遵循作業系統提供者的文件，啟用核心分頁表隔離 (KPTI) 來減輕 CVE-2017-5754 (Meltdown Variant 3) 的影響。 
 
作業系統提供者可提供的更多資訊：  
 
- [Redhat 和 CentOS](https://access.redhat.com/security/vulnerabilities/speculativeexecution) 
- [Suse](https://www.suse.com/support/kb/doc/?id=7022512) 
- [Ubuntu](https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/SpectreAndMeltdown) 


## <a name="next-steps"></a>後續步驟

若要進一步了解，請參閱[保護 Azure 客戶免於受到 CPU 漏洞攻擊](https://azure.microsoft.com/blog/securing-azure-customers-from-cpu-vulnerability/)。
