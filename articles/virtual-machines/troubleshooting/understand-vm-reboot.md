---
title: 了解 Azure VM 的系統重新啟動 | Microsoft Docs
description: 列出可能導致 VM 重新啟動的事件
services: virtual-machines
documentationcenter: ''
author: genlin
manager: willchen
editor: ''
tags: ''
ms.service: virtual-machines
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: dd9a2c6a5b2183d4909b6bcfd24dc1fae401c785
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411757"
---
# <a name="understand-a-system-reboot-for-azure-vm"></a>了解 Azure VM 的系統重新啟動

Azure 虛擬機器 (VM) 可能會因不明原因而重新開機，且無法證明您已起始重新開機作業。 本文列出可能導致 VM 重新開機的動作和事件，並提供如何避免發生非預期重新開機問題或降低這類問題影響的深入解析。

## <a name="configure-the-vms-for-high-availability"></a>設定高可用性的 VM
保護在 Azure 上執行的應用程式對抗 VM 重新開機和停機的最佳方法，就是設定為高可用性的 VM。

若要為應用程式提供此層級的備援，建議您在可用性設定組中，將兩部以上的 VM 組成群組。 這項組態可以確保在規劃或未規劃的維護事件發生期間，至少有一部 VM 可以使用，且符合 99.95% 的 [Azure SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_5/)。

如需可用性設定組的詳細資訊，請參閱下列文章：

- [管理 VM 的可用性](../windows/manage-availability.md)
- [設定 VM 的可用性](../windows/classic/configure-availability.md)

## <a name="resource-health-information"></a>資源健全狀況資訊 
Azure 資源健康狀態是一項服務，會揭露個別 Azure 資源的健康狀態，並提供可採取動作的指導以便針對問題進行疑難排解。 在無法直接存取伺服器或基礎結構元素的雲端環境中，資源健全狀況旨在減少您花在疑難排解上的時間。 尤其旨在減少花在判斷問題根源是出自應用程式本身還是 Azure 平台內事件的時間。 如需詳細資訊，請參閱[了解和使用資源健全狀況](../../resource-health/resource-health-overview.md)。

## <a name="actions-and-events-that-can-cause-the-vm-to-reboot"></a>可能導致 VM 重新開機的動作和事件

### <a name="planned-maintenance"></a>預定的維修
為提升 VM 之基礎主機基礎結構的可靠性、效能和安全性，Microsoft Azure 會全球定期執行更新。 許多這些更新 (包括記憶體保留的更新) 在執行時並不會對 VM 或雲端服務造成任何影響。

不過，有些更新的確需要重新開機。 在這類情況下，當我們修補基礎結構時，VM 會關閉，而後再重新開機。

若要了解什麼是 Azure 預定進行的維修，以及其對 Linux VM 可用性有何影響，請參閱下列文章。 這些文章提供有關 Azure 背景計劃性維護程序的背景，以及如何排定計劃性維護以進一步降低影響。

- [Azure 中 VM 的計劃性維護](../windows/planned-maintenance.md)
- [如何在 Azure VM 上排定計劃性維護](../windows/classic/planned-maintenance-schedule.md)

### <a name="memory-preserving-updates"></a>記憶體保留的更新   
對於 Microsoft Azure 中的這類更新，使用者覺得其執行中的 VM 不受任何影響。 許多這些更新都是針對元件或服務，更新時不會干擾執行中的執行個體。 有些更新是主機作業系統上的平台基礎結構更新，不需要將 VM 重新開機就可以套用。

這些記憶體保留的更新是透過可進行就地即時移轉的技術來完成。 更新時，VM 會進入「暫停」狀態。 此狀態會在 RAM 中保留記憶體，而基礎主機作業系統會收到必要的更新和修補程式。 VM 會在暫停後 30 秒內繼續執行。 在 VM 繼續執行後，系統將會自動同步處理其時鐘。

因為短暫的暫停期間，所以透過此機制部署更新可大幅減少對 VM 的影響。 不過，並非所有更新都可以此方式部署。 

多重執行個體更新 (可用性設定組中的 VM) 一次只會套用到一個更新網域。

> [!NOTE]
> 在此更新方法期間，舊核心版本的 Linux 機器會受到核心異常影響。 若要避免這個問題，請更新為核心版本 3.10.0-327.10.1 或更新版本。 如需詳細資訊，請參閱[主機節點升級後以 3.10 為基礎的核心異常上的 Azure Linux VM](https://support.microsoft.com/help/3212236)。     
    
### <a name="user-initiated-reboot-or-shutdown-actions"></a>使用者起始的重新開機或關機動作
 
如果從 Azure 入口網站、Azure PowerShell、命令列介面或重設 API 執行重新開機，可以在 [Azure 活動記錄](../../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)中找到此事件。

如果您從 VM 的作業系統執行動作，您可以在系統記錄中找到此事件。

常會導致 VM 重新啟動的其他案例包括多重組態變更動作。 您通常會看到一則警告訊息，指出執行特定動作將會導致 VM 重新開機。 範例包括任何 VM 大小調整作業、變更系統管理帳戶的密碼，以及設定靜態 IP 位址。

### <a name="azure-security-center-and-windows-update"></a>Azure 資訊安全中心與 Windows Update
Azure 資訊安全中心每日監視 Windows 和 Linux VM 是否有遺漏的作業系統更新。 資訊安全中心會根據 Windows VM 上設定的服務，從 Windows Update 或 Windows Server Update Services (WSUS) 擷取可用的安全性和重大更新清單。 資訊安全中心也會檢查 Linux 系統的最新更新。 如果您的 VM 遺漏系統更新，資訊安全中心建議您套用系統更新。 這些系統更新的應用程式是透過 Azure 入口網站中的資訊安全中心控制。 套用某些更新後，可能需要 VM 重新開機。 如需詳細資訊，請參閱[在 Azure 資訊安全中心套用系統更新](../../security-center/security-center-apply-system-updates.md)。

如同內部部署伺服器，Azure 不會將 Windows Update 的更新推送至 Microsoft Azure VM，因為這些機器預計是由使用者管理。 不過，建議您讓自動安裝 Windows Update 設定保持啟用狀態。 自動安裝 Windows Update 的更新也可能導致在套用更新後發生重新開機。 如需詳細資訊，請參閱 [ Windows 更新常見問題集](https://support.microsoft.com/help/12373/windows-update-faq)。

### <a name="other-situations-affecting-the-availability-of-your-vm"></a>其他會影響 VM 可用性的情況
在其他情況下，Azure 可能會主動暫停使用 VM。 您會在系統採取此動作前收到電子郵件通知，如此才有機會解決此基礎問題。 影響 VM 可用性的問題範例包括安全性違規和付款方式過期。

### <a name="host-server-faults"></a>主機伺服器錯誤 
VM 會裝載於在 Azure 資料中心內執行的實體伺服器。 除了少數幾個其他 Azure 元件，此實體伺服器會執行稱為「主機代理程式」的代理程式。 當實體伺服器上的這些 Azure 軟體元件都沒有回應時，監視系統會觸發主機伺服器重新開機以嘗試復原。 VM 通常在五分鐘內再度可用，並繼續存留在與之前相同的主機上。

伺服器錯誤通常是由硬體故障造成，例如硬碟或固態硬碟故障。 Azure 會持續監視這些狀況、找出基礎錯誤，並且在實作及測試緩和措施之後推出更新。

因為有些主機伺服器錯誤可能專屬於該伺服器，所以手動將 VM 重新部署到另一部主機伺服器可改善重複的 VM 重新開機情況。 使用 VM [詳細資料] 頁面上的 [重新部署] 選項，或在 Azure 入口網站中停止並重新啟動 VM，即可觸發這項作業。

### <a name="auto-recovery"></a>自動復原
在主機伺服器因為任何原因而無法重新開機的情況下，Azure 平台會起始自動復原動作，讓錯誤的主機伺服器脫離輪替循環，以便進一步調查。 

該主機上的所有 VM 都會自動重新安置到狀況良好的不同主機伺服器。 此程序通常在 15 分鐘內完成。 若要深入了解自動復原程序，請參閱[自動復原 VM](https://azure.microsoft.com/blog/service-healing-auto-recovery-of-virtual-machines)。

### <a name="unplanned-maintenance"></a>非計劃性維護
在少數情況下，Azure 作業小組可能需要執行維護活動，以確保 Azure 平台的整體健全狀況。 此行為可能會影響 VM 可用性，且通常會導致如先前所述的相同自動復原動作。  

非計劃性維護包括下列各項：

- 緊急節點磁碟重組
- 緊急網路交換器更新

### <a name="vm-crashes"></a>VM 損毀
VM 可能會因為 VM 本身的問題而重新啟動。 在 VM 上執行的工作負載或角色可能會觸發客體作業系統內的錯誤檢查。 如需判斷損毀原因的協助，請檢視 Windows VM 的系統和應用程式記錄以及 Linux VM 的序列記錄。

### <a name="storage-related-forced-shutdowns"></a>儲存體相關的強制關機
Azure 中的 VM 依賴作業系統的虛擬磁碟以及 Azure 儲存體基礎結構上裝載的資料儲存體。 每當 VM 和相關聯虛擬磁碟之間的可用性或連線受影響超過 120 秒，Azure 平台會執行 VM 強制關機，以避免資料損毀。 VM 會在儲存體恢復連線之後自動啟動。 

關機持續時間可能只有短短五分鐘，但也可能會更長。 以下是與儲存體相關強制關閉相關聯的特定案例之一： 

**超出 IO 限制**

當 I/O 要求因為每秒 I/O 作業次數 (IOPS) 超出磁碟的 I/O 限制而一直節流時，VM 可能會暫時關閉。 (標準磁碟儲存體受限於 500 IOPS)。若要緩和這個問題，請使用磁碟等量化或在客體 VM 內設定儲存體空間 (視工作負載而定)。 如需詳細資訊，請參閱[設定 Azure VM 以達到最佳的儲存體效能](http://blogs.msdn.com/b/mast/archive/2014/10/14/configuring-azure-virtual-machines-for-optimal-storage-performance.aspx)。

透過有高達 80,000 IOPS 的 Azure 進階儲存體可取得較高的 IOPS 限制。 如需詳細資訊，請參閱[高效能進階儲存體](../windows/premium-storage.md)。

### <a name="other-incidents"></a>其他事件
在少數情況下，廣泛的問題可能會影響 Azure 資料中心內的多部伺服器。 如果發生此問題，Azure 團隊會傳送電子郵件通知給受影響的訂用帳戶。 您可以在 [Azure 服務健全狀況儀表板](https://azure.microsoft.com/status/)和 Azure 入口網站中，檢查目前中斷和過去事件的狀態。
