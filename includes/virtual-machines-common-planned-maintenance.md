---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: shants123
ms.service: virtual-machines
ms.topic: include
ms.date: 07/05/2018
ms.author: shants
ms.custom: include file
ms.openlocfilehash: f203e056df00fb1a9b1e7e43930955040dfce4aa
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39029992"
---
為提升虛擬機器之主機基礎結構的可靠性、效能和安全性，Azure 會定期執行更新。 這些更新的範圍，從在主控環境 (像是作業系統、Hypervisor 和各種在主機上部署的代理程式) 中修補軟體元件、升級網路元件，以至硬體解除委任。 這些更新大多數都會在不影響託管虛擬機器的情況下執行。 不過，更新在某些情況下確實會造成影響：

- 如果能夠進行不需重新開機的更新，在主機更新或 VM 移到已更新的主機時，Azure 會使用記憶體來保留維護，以暫停 VM。

- 如果維護需要重新開機，您會在規劃維護時收到通知。 在這些情況下，您也將獲得一個時間範圍，您可以在適合您的時間自行開始維護。

本頁面說明 Microsoft Azure 如何執行這兩種類型的維護。 如需非計劃性事件 (中斷) 的詳細資訊，請參閱管理 [Windows](../articles/virtual-machines/windows/manage-availability.md) 或 [Linux](../articles/virtual-machines/linux/manage-availability.md) 虛擬機器的可用性。

在虛擬機器中執行的應用程式可以使用適用於 [Windows](../articles/virtual-machines/windows/instance-metadata-service.md)或 [Linux](../articles/virtual-machines/linux/instance-metadata-service.md) 的 Azure Metadata Service，收集近期更新的相關資訊。

如需管理計劃性維護的「作法」資訊，請參閱適用於 [Linux](../articles/virtual-machines/linux/maintenance-notifications.md) 或 [Windows](../articles/virtual-machines/windows/maintenance-notifications.md) 的「處理計劃性維護通知」。

## <a name="memory-preserving-maintenance"></a>記憶體保留維護

更新不需要完整的重新開機時，記憶體保留維護機制可用來限制對虛擬機器的影響。 虛擬機器會暫停 30 秒並將記憶體保留在 RAM 中，而主控環境會套用必要的更新和修補程式，或將 VM 移到已經更新的主機。 虛擬機器會接著繼續執行，而系統會自動同步虛擬機器的時鐘。 

這些不需要重新開機的維護作業會逐個容錯網域套用，而且如果收到任何警告健康情況訊號，進度就會停止。

這些類型的更新會對某些應用程式造成影響。 執行即時事件處理 (如媒體串流處理或轉碼，或高輸送量網路服務案例) 的應用程式，其設計可能不會容許暫停 30 秒。 <!-- sooooo, what should they do? --> 萬一正在將 VM 移到不同的主機，某些重要的工作負載可能會有幾分鐘出現些微的效能降低，導致虛擬機器暫停。 


## <a name="maintenance-requiring-a-reboot"></a>維護需要重新開機

如果 VM 需要針對計劃性維護重新啟動，您會事先收到通知。 計劃性維護有兩個階段：自助式期間和排程維護期間。

**自助式期間**可讓您在您的 VM 上起始維護。 在此期間，您可以查詢每個 VM 來查看其狀態，並檢查上次維護要求的結果。

當您開始自助式維護時，VM 會移至已更新的節點，然後重新開機。 因為 VM 會重新開機，所以暫存磁碟會遺失，而系統會更新與虛擬網路介面關聯的動態 IP 位址。

如果您開始自助維護且在此過程中發生錯誤，此作業就會停止，系統不會更新 VM，而且您可以選擇重試自助維護。 

當自助式期間過後，**排程的維護期間**隨即開始。 在這段期間內，您仍然可以查詢維護期間，但是無法再自行開始維護。

如需管理需要重新開機的維護作業的相關資訊，請參閱 [Linux](../articles/virtual-machines/linux/maintenance-notifications.md) 或 [Windows](../articles/virtual-machines/windows/maintenance-notifications.md) 的「處理預定進行的維修作業通知」。 

### <a name="availability-considerations-during-scheduled-maintenance"></a>排程維護期間的可用性考量 

如果您決定等到排程維護期間，為了維持 VM 的最高可用性，需考慮下列事項。 

#### <a name="paired-regions"></a>配對的區域

每個 Azure 區域都會與相同地理位置內的另一個區域配對，以共同形成區域配對。 在排程維護期間，Azure 只會更新區域配對中單一區域的 VM。 舉例來說，Azure 在更新美國中北部的虛擬機器時，就不會同時更新任何在美國中南部的虛擬機器。 不過，像是北歐等其他區域可以和美國東部一同進行維護。 了解區域配對的運作方式可協助您將 VM 更妥善地分散於各區域。 如需詳細資訊，請參閱 [Azure 區域配對](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。

#### <a name="availability-sets-and-scale-sets"></a>可用性設定組和擴展集

在 Azure VM 上部署工作負載時，您可以在一個可用性設定組內建立 VM，以便為應用程式提供高可用性。 這可確保在中斷或需要重新開機的維護事件期間，至少有一部虛擬機器可供使用。

在可用性設定組內，個別 VM 會散佈在多達 20 個更新網域 (UD)。 在排程維護期間，在任何指定時間只有單一更新網域受影響。 請注意，受影響的更新網域順序不一定會循序發生。 

虛擬機器擴展集是一個可讓您以單一資源的形式，部署和管理一組相同 VM 的 Azure 計算資源。 擴展集會自動部署於更新網域，如可用性設定組中的 VM。 就如同可用性設定組，於排程維護期間使用擴展集，在任何指定時間只有單一更新網域受影響。

如需設定虛擬機器以取得高可用性的詳細資訊，請參閱管理 [Windows](../articles/virtual-machines/windows/manage-availability.md) 或 [Linux](../articles/virtual-machines/linux/manage-availability.md) 虛擬機器的可用性。
