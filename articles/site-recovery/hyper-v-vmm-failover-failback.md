---
title: 使用 Site Recovery 將複寫的 Hyper-V VM 容錯移轉及容錯回復到次要資料中心 | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 將 Hyper-V VM 容錯移轉至次要內部部署網站，以及容錯回復至主要網站
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/10/2018
ms.author: raynew
ms.openlocfilehash: 782ddc3ebb7b3c533d0eb5c4b7735a13cbaf268c
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077675"
---
# <a name="fail-over-and-fail-back-hyper-v-vms-replicated-to-your-secondary-on-premises-site"></a>將複寫的 Hyper-V VM 容錯移轉及容錯回至次要內部部署網站

[Azure Site Recovery](site-recovery-overview.md) 服務會管理並協調內部部署機器和 Azure 虛擬機器 (VM) 的複寫、容錯移轉和容錯回復。

本文說明如何將 System Center Virtual Machine Manager (VMM) 雲端中管理的 Hyper-V VM 容錯移轉至次要 VMM 網站。 在容錯移轉之後，您可以容錯回復至可用的內部部署網站。 在本文中，您將了解：

> [!div class="checklist"]
> * 將 Hyper-V VM 從主要 VMM 雲端容錯移轉至次要 VMM 雲端
> * 從次要網站放到主要網站重新保護，然後容錯回復
> * 選擇性再次開始從主要網站複寫至次要網站

## <a name="failover-and-failback"></a>容錯移轉和容錯回復

容錯移轉和容錯回復有三個階段：

1. **容錯移轉至次要網站**：將機器從主要網站容錯移轉至次要網站。
2. **從次要網站容錯回復**：將 VM 從次要網站複寫至主要網站，然後執行計劃性容錯移轉來容錯回復。
3. 在計劃性容錯移轉後，選擇性地再次開始從主要網站複寫至次要網站。


## <a name="prerequisites"></a>必要條件

- 確定您已完成[災害復原演練](hyper-v-vmm-test-failover.md)，檢查一切是否如預期般運作。
- 若要完成容錯回復，請確定主要和次要 VMM 伺服器已連線至 Site Recovery。



## <a name="run-a-failover-from-primary-to-secondary"></a>執行從主要網站到次要網站的容錯移轉

您可以對 Hyper-V VM 執行定期或計劃性容錯移轉。

- 對非預期的中斷情形使用定期容錯移轉。 當您執行此容錯移轉時，Site Recovery 會在次要網站中建立 VM，然後將它啟動。 視尚未同步處理的擱置中資料而定，可能會發生資料遺失。
- 規劃的容錯移轉可用於維護，或在預期的中斷期間使用。 此選項會提供資料零遺失。 觸發規劃的容錯移轉時，來源 VM 會關閉。 系統會同步處理未同步處理的資料，並觸發容錯移轉。 
- 
此程序說明如何執行定期容錯移轉。


1. 在 [設定] > [複寫的項目] 中，按一下 VM > [容錯移轉]。
1. 如果想在觸發容錯移轉之前，讓 Site Recovery 嘗試將來源 VM 關機，請選取 [先將機器關機再開始容錯移轉]。 Site Recovery 也會在觸發容錯移轉之前，嘗試同步處理尚未傳送至次要網站的內部部署資料。 請注意，即使關機失敗，仍會繼續容錯移轉。 您可以 [作業] 頁面上追蹤容錯移轉進度。
2. 您現在應該能夠在次要 VMM 雲端中看到此 VM。
3. 驗證 VM 之後，請 [認可] 容錯移轉。 這會刪除所有可用的復原點。

> [!WARNING]
> **不要取消正在進行的容錯移轉**：在啟動容錯移轉之前，已停止 VM 複寫。 如果您取消正在進行的容錯移轉，容錯移轉會停止，但 VM 不會再次複寫。  


## <a name="reverse-replicate-and-failover"></a>反向複寫和容錯移轉

開始從次要網站複寫至主要網站，然後容錯回復至主要網站。 當 VM 重新在主要網站中執行之後，您可以將它們複寫至次要站台。  

 
1. 按一下 [VM] > 按一下 [反向複寫]。
2. 一旦工作完成，請按一下 [VM] >在 [容錯移轉] 中，確認容錯移轉方向 (從次要 VMM 雲端)，並選取來源和目標位置。 
4. 起始容錯移轉。 您可以在 [工作]  索引標籤上追蹤容錯移轉進度。
5. 在主要 VMM 雲端中，檢查 VM 是否可用。
6. 如果您想要再次開始將主要 VM 複寫回到次要站台，請按一下 [反向複寫]。

## <a name="next-steps"></a>後續步驟
[檢閱步驟](hyper-v-vmm-disaster-recovery.md)，了解如何將 Hyper-V VM 複寫至次要網站。
