---
title: 在 Azure Site Recovery 中測試容錯移轉到 Azure | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 服務執行從內部部署到 Azure 的測試容錯移轉。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 09/11/2018
ms.author: raynew
ms.openlocfilehash: 4c72a58cdc6082a40fe80b7a3cf8cf964199371e
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391771"
---
# <a name="test-failover-to-azure-in-site-recovery"></a>在 Site Recovery 中測試容錯移轉到 Azure


本文說明如何使用 Site Recovery 測試容錯移轉，將災害復原深入執行至 Azure。  

您可以執行測試容錯移轉，以驗證您的複寫和災害復原策略不會有資料遺失或停機時間。 測試容錯移轉不會影響正在進行的複寫或生產環境。 您可以在特定虛擬機器 (VM) 上執行測試容錯移轉，或在包含多部 VM 的[復原計劃](site-recovery-create-recovery-plans.md)上執行測試容錯移轉。


## <a name="run-a-test-failover"></a>執行測試容錯移轉
此程序說明如何針對復原方案執行測試容錯移轉。 如果想要執行單一 VM 的測試容錯移轉，請依照[這裡](tutorial-dr-drill-azure.md#run-a-test-failover-for-a-single-vm)所述的步驟進行

![Test Failover](./media/site-recovery-test-failover-to-azure/TestFailover.png)


1. 在 Azure 入口網站的 Site Recovery 中，按一下 [復原計劃] > recoveryplan_name > [測試容錯移轉]。
2. 選取要對其進行容錯移轉的 [復原點]。 您可以使用下列其中一個選項：
    - **最近處理**：此選項會將計劃中所有 VM 容錯移轉到 Site Recovery 所處理的最新復原點。 若要查看特定 VM 的最新復原點，請檢查 VM 設定中的 [最新復原點]。 此選項提供低 RTO (復原時間目標)，因為無須花費時間處理未處理的資料。
    - **最新應用程式一致**：此選項會將計劃中所有 VM 容錯移轉到 Site Recovery 所處理的最新應用程式一致復原點。 若要查看特定 VM 的最新復原點，請檢查 VM 設定中的 [最新復原點]。
    - **最新**︰此選項會先處理已傳送到 Site Recovery 服務的所有資料，先為每部 VM 建立復原點後再進行容錯移轉。 此選項會提供最低的 RPO (復原點目標)，因為在容錯移轉後建立的 VM 具有在觸發容錯移轉時複寫到 Site Recovery 的所有資料。
    - **已處理最新多部 VM**：此選項適用於一或多部 VM 已啟用多部 VM 一致性的復原計劃。 啟用這項設定的 VM 會容錯移轉至最新的一般多部 VM 一致復原點。 其他 VM 會容錯移轉至最新的已處理復原點。  
    - **最新多部 VM 應用程式一致**：此選項適用於一或多部 VM 已啟用多部 VM 一致性的復原計劃。 屬於最新一般多部 VM 應用程式一致復原點的複寫群組容錯移轉一部分的 VM。 對於最新應用程式一致復原點進行的其他 VM 容錯移轉。
    - **自訂**：使用這個選項來將特定 VM 容錯移轉至特定復原點。
3. 選取要在其中建立測試 VM 的 Azure 虛擬網路。

    - Site Recovery 會嘗試在名稱相同的子網路中建立測試 VM，並使用 VM 的 [計算與網路] 設定中提供的 IP 位址。
    - 如果用於測試容錯移轉的 Azure 虛擬網路中無法使用名稱相同的子網路，則會在依字母順序的第一個子網路中建立測試 VM。
    - 如果在子網路中無法使用相同的 IP 位址，則 VM 會接收子網路中另一個可用的 IP 位址。 [深入了解](#create-a-network-for-test-failover)。
4. 如果您正在容錯移轉到 Azure 且已啟用資料加密，請在 [加密金鑰] 中，選取當您在提供者安裝期間啟用加密時所發出的憑證。 您可以忽略這個步驟加密未啟用的情況。
5. 在 [工作]  索引標籤上追蹤容錯移轉進度。您應該可以在 Azure 入口網站中看到測試複本機器。
6. 若要在 Azure VM 上初始化 RDP 連線，您必須在容錯移轉之 VM 的網路介面上[新增公用 IP 位址](https://aka.ms/addpublicip)。
7. 當一切都如預期般運作時，請按一下 [清除測試容錯移轉]。 這樣會刪除在測試容錯移轉期間所建立的 VM。
8. 在 [記事] 中，記錄並儲存關於測試容錯移轉的任何觀察。


![Test Failover](./media/site-recovery-test-failover-to-azure/TestFailoverJob.png)

觸發測試容錯移轉時，會發生下列情況︰

1. **必要條件**：程式會進行必要條件檢查，以確保所有容錯移轉所需之條件都已符合。
2. **容錯移轉**：容錯移轉會處理並且準備資料，以便從中建立 Azure VM。
3. **最新**：如果您已選擇最新復原點，則會從已傳送至服務的資料建立復原點。
4. **開始**︰這個步驟會使用上一個步驟中所處理的資料建立 Azure 虛擬機器。

### <a name="failover-timing"></a>容錯移轉時間

在下列案例中，容錯移轉需要其他中繼步驟，通常會費時大約 8 到 10 分鐘才能完成：

* VMware VM 執行的版本比行動服務版本 9.8 更早
* 實體伺服器
* VMware Linux VM
* 如同實體伺服器般受到保護的 Hyper-V VM
* 下列驅動程式不是開機驅動程式的 VMware VM：
    * storvsc
    * vmbus
    * storflt
    * intelide
    * atapi
* 未啟用 DHCP 的 VMware 虛擬機器，無論其是否正在使用 DHCP 或靜態 IP 位址。

在其他所有情況下則不需要任何中繼步驟，且容錯移轉所花費的時間非常少。


## <a name="create-a-network-for-test-failover"></a>建立測試容錯移轉的網路

我們建議針對測試容錯移轉所選擇的網路，是與每部 VM [計算與網路] 設定中特定實際執行復原網站網路隔離的網路。 根據預設，當您建立 Azure 虛擬網路時，它會與其他網路隔離。 此測試網路應模擬您的實際執行網路︰

- 測試網路應該具有與實際執行網路相同數目的子網路。 子網路應具有相同的名稱。
- 測試網路應使用相同的 IP 位址範圍。
- 使用針對 [計算與網路] 設定中 DNS VM 指定的 IP 位址，來更新測試網路的 DNS。 如需詳細資訊，請參閱 [Active Directory 測試容錯移轉考量](site-recovery-active-directory.md#test-failover-considerations)。


## <a name="test-failover-to-a-production-network-in-the-recovery-site"></a>在復原網站中測試容錯移轉到實際執行網路

雖然我們建議您使用與實際執行網路不同的測試網路，如果您想要對實際執行網路進行測試災害復原訓練，請注意下列事項：

- 執行測試容錯移轉時，請確定主要 VM 已關機。 否則，相同的網路中將同時有兩部 VM 以相同的身分識別執行。 這可能會導致非預期的結果。
- 當您清除容錯移轉時，為測試容錯移轉建立之 VM 的任何變更都會遺失。 這些變更不會複寫回主要 VM。
- 在實際執行環境中測試會導致實際執行應用程式的停機時間。 當測試容錯移轉進行時，使用者不應該使用在 VM 上執行的應用程式。  



## <a name="prepare-active-directory-and-dns"></a>準備 Active Directory 和 DNS

若要執行測試容錯移轉以進行應用程式測試，您需要在測試環境中具有 Active Directory 生產環境的複本。 如需詳細資訊，請參閱 [Active Directory 測試容錯移轉考量](site-recovery-active-directory.md#test-failover-considerations)。

## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

如果您想要在容錯移轉後使用 RDP/SSH 連線到 Azure VM，請遵循資料表中摘要說明的需求。

**容錯移轉** | **位置** | **動作**
--- | --- | ---
**執行 Windows 的 Azure VM** | 在容錯移轉前的內部部署機器 | 若要透過網際網路存取 Azure VM，請啟用 RDP，並確定已針對 [公用] 新增 TCP 和 UDP 規則，且在 [Windows 防火牆] > [允許的應用程式] 中已針對所有設定檔允許 RDP。<br/><br/> 若要透過站對站連線存取 Azure VM，請在機器上啟用 RDP，並確定在 [Windows 防火牆]  ->  [允許的應用程式和功能] 中已針對 [網域] 和 [私人] 網路允許 RDP。<br/><br/>  確定作業系統的 SAN 原則已設為 **OnlineAll**。 [深入了解](https://support.microsoft.com/kb/3031135)。<br/><br/> 觸發容錯移轉時，請確定 VM 上沒有任何暫止的 Windows 更新。 容錯移轉時，可能會啟動 Windows 更新，必須等到更新完成，才能登入 VM。
**執行 Windows 的 Azure VM** | 容錯移轉後的 Azure VM |  [新增 VM 的公用 IP 位址](https://aka.ms/addpublicip)。<br/><br/> 已容錯移轉的 VM 上的網路安全性群組規則 (以及它所連線的 Azure 子網路) 必須允許 RDP 連接埠的連入連線。<br/><br/> 勾選 [開機診斷] 以確認 VM 的螢幕擷取畫面。<br/><br/> 如果您無法連線，請檢查 VM 是否正在執行，並檢閱這些[疑難排解祕訣](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。
**執行 Linux 的 Azure VM** | 在容錯移轉前的內部部署機器 | 確定 VM 上的安全殼層服務已設定為在系統開機時自動啟動。<br/><br/> 請檢查防火牆規則是否允許 SSH 連線。
**執行 Linux 的 Azure VM** | 容錯移轉後的 Azure VM | 已容錯移轉的 VM 上的網路安全性群組規則 (以及它所連線的 Azure 子網路) 必須允許 SSH 連接埠的連入連線。<br/><br/> [新增 VM 的公用 IP 位址](https://aka.ms/addpublicip)。<br/><br/> 勾選 [開機診斷] 以檢視 VM 的螢幕擷取畫面。<br/><br/>

請依照[這裡](site-recovery-failover-to-azure-troubleshoot.md)所述的步驟，對容錯移轉後的連線問題進行疑難排解。

## <a name="next-steps"></a>後續步驟
完成災害復原訓練之後，深入了解其他類型的[容錯移轉](site-recovery-failover.md)。
