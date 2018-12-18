---
title: 使用 Azure Site Recovery 複寫多層式 Dynamics AX 部署 | Microsoft Docs
description: 本文說明如何使用 Azure Site Recovery 複寫和保護 Dynamics AX
services: site-recovery
documentationcenter: ''
author: asgang
manager: rochakm
editor: ''
ms.assetid: 9126f5e8-e9ed-4c31-b6b4-bf969c12c184
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2018
ms.author: asgang
ms.openlocfilehash: eb6f7d9b34e00ce1efd8c871439c2504e5f550d5
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669437"
---
# <a name="replicate-a-multitier-dynamics-ax-application-by-using-azure-site-recovery"></a>使用 Azure Site Recovery 複寫多層式 Dynamics AX 應用程式

## <a name="overview"></a>概觀


 Dynamics AX 是企業愛用的 ERP 解決方案之一，可橫跨位置將程序標準化、管理資源及簡化合規性。 因為應用程式攸關組織的運作，在發生災害時，應用程式應該要在最短的時間內啟動並執行。

Dynamics AX 目前並未提供任何現成的災害復原功能。 Dynamics AX 包含許多伺服器元件，例如 Windows Application Object Server、Azure Active Directory、Azure SQL Database、SharePoint Server、Reporting Service。 手動管理上述每個元件的災害復原，不僅代價昂貴，而且也容易發生錯誤。

本文說明如何使用 [Azure Site Recovery](site-recovery-overview.md) 為 Dynamics AX 應用程式建立災害復原解決方案。 文中也探討使用單鍵復原方案、支援的組態和必要條件進行計劃性/非計劃性的測試容錯移轉。



## <a name="prerequisites"></a>必要條件

您需要完成下列必要條件，才能使用 Site Recovery 實作 Dynamics AX 應用程式的災害復原：

• 安裝內部部署 Dynamics AX 部署。

• 在 Azure 訂用帳戶中建立 Site Recovery 保存庫。

• 如果 Azure 是您的復原網站，在虛擬機器上執行 Azure 虛擬機器整備性評估工具。 它們必須與 Azure 虛擬機器及 Site Recovery 服務相容。

## <a name="site-recovery-support"></a>Site Recovery 支援

我們為了編寫本文而使用的設備是 Windows Server 2012 R2 Enterprise 上的 VMware 虛擬機器與 Dynamics AX 2012R3。 因為站台復原複寫應用程式無從驗證，我們預期此處提供的建議可以運用在下列案例。

### <a name="source-and-target"></a>來源與目標

**案例** | **至次要網站** | **至 Azure**
--- | --- | ---
**Hyper-V** | 是 | 是
**VMware** | 是 | 是
**實體伺服器** | 是 | 是

## <a name="enable-disaster-recovery-of-the-dynamics-ax-application-by-using-site-recovery"></a>使用 Site Recovery 讓 Dynamics AX 應用程式可以災害復原
### <a name="protect-your-dynamics-ax-application"></a>保護 Dynamics AX 應用程式
為了做到完整的應用程式複寫與復原，Dynamics AX 的每個元件都需要受到保護。

### <a name="1-set-up-active-directory-and-dns-replication"></a>1.設定 Active Directory 和 DNS 複寫

災害復原網站上需要有 Active Directory，Dynamics AX 應用程式才能運作。 根據客戶內部部署環境的複雜度，我們建議以下兩個的選項。

**選項 1**

客戶有少量應用程式和一個用於整個內部部署網站的網域控制站，並計劃將整個網站一起容錯移轉。 我們建議您使用 Site Recovery 複寫將網域控制站機器複寫到次要網站 (適用於網站至網站和網站至 Azure 兩種案例)。

**選項 2**

如果客戶有大量應用程式、且執行 Active Directory 樹系，並計劃一次容錯移轉少數應用程式。 我們建議在災害復原站台 (次要網站或在 Azure 中) 另外設定一個網域控制站。

 如需詳細資訊，請參閱[設定使用災害復原站台上的網域控制站](site-recovery-active-directory.md)。 本文之後的內容假設災害復原站台上有可使用的網域控制站。

### <a name="2-set-up-sql-server-replication"></a>2.設定 SQL Server 複寫
如需有關保護 SQL 層的建議選項技術指引，請參閱[以 SQL Server 及 Azure Site Recovery 複寫應用程式](site-recovery-sql.md)。

### <a name="3-enable-protection-for-the-dynamics-ax-client-and-application-object-server-vms"></a>3.啟用 Dynamics AX 用戶端和應用程式物件伺服器 VM 的保護
根據 VM 是部署於 [Hyper-V](site-recovery-hyper-v-site-to-azure.md) 還是 [VMware](site-recovery-vmware-to-azure.md)，執行相關的 Site Recovery 設定。

> [!TIP]
> 建議您將損毀一致頻率設定為 15 分鐘。
>

下列快照集顯示 Dynamics 元件 VM 在「從 VMware 網站到 Azure」保護案例中的保護狀態。

![受保護項目](./media/site-recovery-dynamics-ax/protecteditems.png)

### <a name="4-configure-networking"></a>4.設定網路功能
**設定 VM 計算和網路設定**

在 AX 用戶端和應用程式物件伺服器 VM 上，設定 Site Recovery 中的網路設定，讓 VM 網路能在容錯移轉之後連線到正確的災害復原網路。 確定這些層的災害復原網路可路由傳送到 SQL 層。

您可以在複寫的項目中選取 VM 來進行網路設定，如以下快照集所示：

* 在「應用程式物件伺服器」伺服器上，選取正確的可用性設定組。

* 如果您是使用靜態 IP 位址，在 [目標 IP] 文字方塊中指定您希望 VM 採用的 IP。

    ![網路設定 ](./media/site-recovery-dynamics-ax/vmpropertiesaos1.png)


### <a name="5-create-a-recovery-plan"></a>5.建立復原計畫

您可以在 Site Recovery 中建立復原方案，將容錯移轉程序自動化。 在復原方案中新增應用程式層和 Web 層。 將它們歸入順序不同的群組中，讓前端關機發生在應用程式層之前。

1. 在您的訂用帳戶中選取 Site Recovery 保存庫，然後按一下 [復原方案] 圖格。

2. 按一下 [+ 復原方案] 並指定名稱。

3. 選取 [來源] 和 [目標]。 目標可以是 Azure 或次要網站。 如果您選擇 Azure，則必須指定部署模型。

    ![建立復原計畫](./media/site-recovery-dynamics-ax/recoveryplancreation1.png)

4. 選取復原方案的應用程式物件伺服器和用戶端 VM，然後選取 ✓。

    ![選取項目](./media/site-recovery-dynamics-ax/selectvms.png)

    復原方案範例：

    ![復原方案詳細資料](./media/site-recovery-dynamics-ax/recoveryplan.png)

新增下列步驟，即可自訂 Dynamics AX 應用程式的復原方案。 前述快照集會顯示新增所有步驟之後的完整復原方案。


* **SQL Server 容錯移轉步驟**：如需 SQL Server 專屬復原步驟的詳細資訊，請參閱[以 SQL Server 及 Azure Site Recovery 複寫應用程式](site-recovery-sql.md)。

* **容錯移轉群組 1**︰容錯移轉應用程式物件伺服器 VM。
確定所選取的復原點儘可能接近資料庫 PIT，但不在它之前。

* **指令碼**：新增負載平衡器 (僅限 E-A)。
在應用程式物件伺服器 VM 群組開始新增負載平衡器之後，新增指令碼 (透過 Azure 自動化)。 您可以使用指令碼來執行此工作。 如需詳細資訊，請參閱[如何新增多層式應用程式災害復原的負載平衡器](https://azure.microsoft.com/blog/cloud-migration-and-disaster-recovery-of-load-balanced-multi-tier-applications-using-azure-site-recovery/)。

* **容錯移轉群組 2**︰容錯移轉 Dynamics AX 用戶端 VM。 容錯移轉 Web 層 VM 作為復原方案的一部份。


### <a name="perform-a-test-failover"></a>執行測試容錯移轉

如需測試容錯移轉期間 Active Directory 專屬的詳細資訊，請參閱＜Active Directory 災害復原解決方案＞附屬指南。

如需測試容錯移轉期間 SQL Server 專屬的詳細資訊，請參閱[以 SQL Server 及 Azure Site Recovery 複寫應用程式](site-recovery-sql.md)。

1. 移至 Azure 入口網站，選取您的 Site Recovery 保存庫。

2. 選取為 Dynamics AX 建立的復原方案。

3. 選取 [測試容錯移轉]。

4. 選取虛擬網路來開始測試容錯移轉程序。

5. 次要環境啟動後，您就可以執行驗證。

6. 驗證完成後，選取 [驗證完成]，便會清除測試容錯移轉環境。

如需執行測試容錯移轉的詳細資訊，請參閱[在 Site Recovery 中測試容錯移轉到 Azure](site-recovery-test-failover-to-azure.md)。

### <a name="perform-a-failover"></a>執行容錯移轉

1. 移至 Azure 入口網站，選取您的 Site Recovery 保存庫。

2. 選取為 Dynamics AX 建立的復原方案。

3. 按一下 [容錯移轉]，然後選取 [容錯移轉]。

4. 選取目標網路，然後按一下 [✓] 啟動容錯移轉程序。

如需進行容錯移轉的詳細資訊，請參閱[在 Site Recovery 中容錯移轉](site-recovery-failover.md)。

### <a name="perform-a-failback"></a>執行容錯移轉

如需容錯回復期間 SQL Server 專屬的考量事項，請參閱[以 SQL Server 及 Azure Site Recovery 複寫應用程式](site-recovery-sql.md)。

1. 移至 Azure 入口網站，選取您的 Site Recovery 保存庫。

2. 選取為 Dynamics AX 建立的復原方案。

3. 按一下 [容錯移轉]，然後選取 [容錯移轉]。

4. 按一下 [變更方向]。

5. 選取適當的選項：資料同步處理和 VM 建立。

6. 選取 [✓] 開始容錯回復程序。


如需執行容錯回復的詳細資訊，請參閱[將 VMware VM 從 Azure 容錯回復到內部部署](site-recovery-failback-azure-to-vmware.md)。

## <a name="summary"></a>總結
使用 Site Recovery，可以為 Dynamics AX 應用程式建立一個完整的自動化災害復原方案。 當發生中斷時，可以在幾秒鐘內從任何地方起始容錯移轉，並且在數分鐘內啟動並執行應用程式。

## <a name="next-steps"></a>後續步驟
如需了解如何以 Azure Site Recovery 保護企業工作負載，請參閱[我可以保護哪些工作負載?](site-recovery-workload.md)。
