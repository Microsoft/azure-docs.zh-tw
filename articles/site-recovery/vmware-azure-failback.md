---
title: 使用 Azure Site Recovery 從 Azure 容錯回復至 VMware | Microsoft Docs
description: 將虛擬機器容錯移轉到 Azure 之後，您可以開始將虛擬機器容錯回復到內部部署的作業。 了解如何容錯回復的步驟。
author: nsoneji
ms.service: site-recovery
ms.date: 10/11/2018
ms.topic: conceptual
ms.author: nisoneji
ms.openlocfilehash: 841e036555723051dfeaf946f31eaf9eedaee798
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093505"
---
# <a name="fail-back-from-azure-to-an-on-premises-site"></a>從 Azure 容錯回復至內部部署網站

本文說明如何將虛擬機器從 Azure 虛擬機器容錯回復到內部部署 VMware 環境。 遵循本文中的指示，使用[在 Azure Site Recovery 中容錯移轉](site-recovery-failover.md)教學課程，將已從內部部署網站容錯移轉至 Azure 的 VMware 虛擬機器或 Windows/Linux 實體伺服器容錯回復。

## <a name="prerequisites"></a>必要條件
- 請確定您已閱讀[不同類型容錯回復](concepts-types-of-failback.md)的相關詳細資料和對應的須知。

> [!WARNING]
> 如果您已[完成移轉](migrate-overview.md#what-do-we-mean-by-migration)、將虛擬機器移至另一個資源群組，或刪除 Azure 虛擬機器，則您無法在之後執行容錯回復。 如果您停用虛擬機器的保護，便無法進行容錯回復。

> [!WARNING]
> Windows Server 2008 R2 SP1 實體伺服器如果受保護且容錯移轉至 Azure，則無法容錯回復。

> [!NOTE]
> 如果您已對 VMware 虛擬機器進行容錯移轉，便無法容錯回復到 Hyper-V 主機。


- 繼續之前，請先完成重新保護步驟，使虛擬機器處於已複寫狀態，這樣才能開始容錯移轉回到內部部署網站。 如需詳細資訊，請參閱[如何從 Azure 移至內部部署來重新保護](vmware-azure-reprotect.md)。

- 請先確定 vCenter 處於已連線狀態，再進行容錯回復。 否則，將磁碟中斷連線並將它們重新連結至虛擬機器的作業會失敗。

- 在對 Azure 進行容錯移轉期間，可能會無法存取內部部署網站，而且組態伺服器可能會無法使用或關機。 在重新保護和容錯回復期間，內部部署設定伺服器應該會執行並處於連線正常狀態。 

- 在容錯回復期間，虛擬機器必須存在於組態伺服器資料庫中，否則容錯回復會失敗。 請確定您有排定來定期備份伺服器。 發生災害時，您需要使用原始的 IP 位址來還原伺服器，以便容錯回復能夠運作。

- 觸發重新保護/容錯回復之前，主要目標伺服器不能有任何快照集。

## <a name="overview-of-failback"></a>容錯回復的概觀
在容錯移轉至 Azure 之後，您可以透過執行下列步驟來容錯回復至內部部署網站：

1. [重新保護](vmware-azure-reprotect.md) Azure 上的虛擬機器，讓它們開始複寫到內部部署網站中的 VMware 虛擬機器。 在此程序進行期間，您還需要︰

    * 設定內部部署主要目標。 使用 Windows 主要目標 (若為 Windows 虛擬機器) 和 [Linux 主要目標](vmware-azure-install-linux-master-target.md) (若為 Linux 虛擬機器)。
    * 設定[處理序伺服器](vmware-azure-set-up-process-server-azure.md)。
    * 請動[重新保護](vmware-azure-reprotect.md)以關閉內部部署虛擬機器，並使 Azure 虛擬機器的資料與內部部署磁碟同步。

2. 在 Azure 上的虛擬機器複寫至內部部署網站後，您可以啟動從 Azure 到內部部署網站的容錯回復。

3. 容錯回復資料之後，您必須再次重新保護內部部署虛擬機器，使其開始複寫至 Azure。

如需快速概觀，請觀看下列有關如何容錯回復到內部部署網站的影片：
> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/VMware-to-Azure-with-ASR-Video5-Failback-from-Azure-to-On-premises/player]


## <a name="steps-to-fail-back"></a>容錯回復的步驟

> [!IMPORTANT]
> 啟動容錯回復之前，請確定您已完成虛擬機器的重新保護。 虛擬機器應該處於受保護狀態，且其健康狀態應該是**良好**。 若要重新保護虛擬機器，請參閱[如何重新保護](vmware-azure-reprotect.md)。

1. 在複寫的項目頁面上，選取虛擬機器。 以滑鼠右鍵按一下以選取 [未規劃的容錯移轉]。
2. 在 [確認容錯移轉] 中，確認容錯移轉方向 (以 Azure 為來源)。 然後選取您想要用來進行容錯移轉的復原點 (最近的復原點或最近的應用程式一致復原點)。 應用程式一致復原點會在最近的復原點之後，並造成部分資料遺失。
3. 在容錯移轉期間，Site Recovery 會將 Azure 上的虛擬機器關閉。 在確認容錯回復已如預期完成後，您可以確認 Azure 上的虛擬機器是否已關閉。
4. 需要**認可**，以便從 Azure 移除容錯移轉的虛擬機器。 請以滑鼠右鍵按一下受保護的項目，然後選取 [認可]。 此時會有作業移除 Azure 中已容錯移轉的虛擬機器。


## <a name="to-what-recovery-point-can-i-fail-back-the-virtual-machines"></a>我可以將虛擬機器容錯回復至哪個復原點？

在容錯回復期間，有兩個選項讓您容錯回復虛擬機器/復原方案。

- 如果您選取最近處理的時間點，所有虛擬機器會容錯移轉至其最新可用的時間點。 如果復原計劃內有複寫群組，則複寫群組的每個虛擬機器會容錯移轉至其獨自的最新時間點。

  虛擬機器擁有至少一個復原點之後，才能予以容錯回復。 復原方案的所有虛擬機器至少要有一個復原點，您才能容錯回復此復原方案。

  > [!NOTE]
  > 最新復原點是損毀一致復原點。

- 如果您選取應用程式一致復原點，單一虛擬機器容錯回復會復原至其最新可用的應用程式一致復原點。 如果復原方案有複寫群組，每個複寫群組會復原至其一般可用的復原點。
應用程式一致復原點的時間會落後，可能會遺失資料。

## <a name="what-happens-to-vmware-tools-post-failback"></a>VMware 工具在容錯回復後會發生什麼狀況？

萬一遇上 Windows 虛擬機器，Site Recovery 會在容錯移轉期間停用 VMware 工具。 在 Windows 虛擬機器容錯回復期間，VMware 工具會重新啟用。 


## <a name="reprotect-from-on-premises-to-azure"></a>從內部部署移至 Azure 來重新保護
在容錯回復完成且您已啟動認可之後，會刪除 Azure 中復原的虛擬機器。 現在，虛擬機器會回到內部部署網站上，但不會受到保護。 若要再次開始複寫至 Azure，請執行下列動作：

1. 選取 [保存庫] > [設定] > [已複寫的項目] 後，選取已容錯回復的虛擬機器，然後選取 [重新保護]。
2. 輸入必須用來將資料傳送回 Azure 之處理伺服器的值。
3. 選取 [確定] 以開始重新保護作業。

> [!NOTE]
> 在內部部署虛擬機器啟動之後，需要經過一些時間，代理程式才會註冊回到組態伺服器中 (最多 15 分鐘)。 在這段時間，重新保護會失敗，並傳回錯誤訊息以指出未安裝代理程式。 請等候幾分鐘，然後再次嘗試重新保護。

## <a name="next-steps"></a>後續步驟

重新保護作業完成之後，虛擬機器會複寫回到 Azure，而且您可以執行[容錯移轉](site-recovery-failover.md)以將虛擬機器再次移至 Azure。


