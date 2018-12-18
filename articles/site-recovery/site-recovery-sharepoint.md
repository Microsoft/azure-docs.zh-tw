---
title: 使用 Azure Site Recovery 複寫多層式 SharePoint 應用程式 | Microsoft Docs
description: 本文說明如何使用 Azure Site Recovery 功能複寫多層式 SharePoint 應用程式。
services: site-recovery
documentationcenter: ''
author: sujayt
manager: rochakm
editor: ''
ms.assetid: ''
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2018
ms.author: sutalasi
ms.openlocfilehash: eb5801988b6fa966a0326c39be4a267ea08500a8
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37916780"
---
# <a name="replicate-a-multi-tier-sharepoint-application-for-disaster-recovery-using-azure-site-recovery"></a>使用 Azure Site Recovery 複寫多層式 SharePoint 應用程式以便進行災害復原

本文詳細說明如何使用 [Azure Site Recovery](site-recovery-overview.md) 保護 SharePoint 應用程式。


## <a name="overview"></a>概觀

Microsoft SharePoint 是功能強大的應用程式，可協助群組或部門組織、共同作業及共用資訊。 SharePoint 可提供內部網路入口網站、文件和檔案管理、共同作業、社交網路、外部網路、網站、企業搜尋與商業智慧。 它也具有系統整合、程序整合和工作流程自動化功能。 一般而言，組織會將其視為對停機時間和資料遺失很敏感的第 1 層應用程式。

現今，Microsoft SharePoint 並未提供任何現成的災害復原功能。 不論災害的類型和規模為何，復原都會涉及使用您可以將伺服器陣列復原至的待命資料中心。 在本機備援系統和備份無法自主要資料中心中斷修復的情況下，需要待命資料中心。

理想的災害復原解決方案應允許以複雜應用程式架構 (例如 SharePoint) 為主的復原方案模型。 它也應該能夠新增自訂步驟，以處理各層之間的應用程式對應，因而提供在發生災害時具有較低 RTO 的單鍵容錯移轉。

本文詳細說明如何使用 [Azure Site Recovery](site-recovery-overview.md) 保護 SharePoint 應用程式。 本文將介紹最佳做法來將三層 SharePoint 應用程式複寫至 Azure、如何進行災害復原訓練，以及如何將應用程式容錯移轉至 Azure。

您可以觀賞以下有關將多層式應用程式復原至 Azure 的影片。

> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/Disaster-Recovery-of-load-balanced-multi-tier-applications-using-Azure-Site-Recovery/player]


## <a name="prerequisites"></a>先決條件

開始之前，請確定您瞭解下列項目︰

1. [將虛擬機器複寫至 Azure](site-recovery-vmware-to-azure.md)
2. 如何[設計修復網路](site-recovery-network-design.md)
3. [執行測試容錯移轉至 Azure](site-recovery-test-failover-to-azure.md)
4. [執行容錯移轉至 Azure](site-recovery-failover.md)
5. 如何[複寫網域控制站](site-recovery-active-directory.md)
6. 如何[複寫 SQL Server](site-recovery-sql.md)

## <a name="sharepoint-architecture"></a>SharePoint 架構

可以使用分層式拓撲和伺服器角色在一或多部伺服器上部署 SharePoint，以實作符合特定目標的伺服器陣列設計。 支援大量並行使用者和大量內容項目的典型大型、高需求 SharePoint 伺服器陣列，會使用服務群組作為其延展性策略的一部分。 這種方法涉及在專用伺服器上執行服務、將這些服務群組在一起，然後將伺服器相應放大為群組。 下列拓撲說明三層式 SharePoint 伺服器陣列的服務與伺服器群組。 如需不同 SharePoint 拓撲的詳細指引，請參閱 SharePoint 文件集和產品線架構。 您可以在[這份文件](https://technet.microsoft.com/library/cc303422.aspx)中找到有關 SharePoint 2013 部署的詳細資訊。



![部署模式 1](./media/site-recovery-sharepoint/sharepointarch.png)


## <a name="site-recovery-support"></a>Site Recovery 支援

為了建立這篇文章，使用了 VMware 虛擬機器搭配 Windows Server 2012 R2 Enterprise。 還使用 SharePoint 2013 Enterprise 版本和 SQL Server 2014 Enterprise 版本。 因為站台復原複寫應用程式無從驗證，此處提供的建議也都必須對下列案例保留。

### <a name="source-and-target"></a>來源與目標

**案例** | **至次要網站** | **至 Azure**
--- | --- | ---
**Hyper-V** | yes | yes
**VMware** | yes | yes
**實體伺服器** | yes | yes
**Azure** | NA | yes

### <a name="sharepoint-versions"></a>SharePoint 版本
支援下列 SharePoint Server 版本。

* SharePoint Server 2013 Standard
* SharePoint Server 2013 Enterprise
* SharePoint Server 2016 Standard
* SharePoint Server 2016 Enterprise

### <a name="things-to-keep-in-mind"></a>要牢記在心的事項

如果您使用以磁碟為基礎的共用叢集作為您應用程式中的任何層，然後您將無法使用 Site Recovery 複寫來複寫這些虛擬機器。 您可以使用應用程式所提供的原生複寫，然後使用[復原方案](site-recovery-create-recovery-plans.md)來容錯移轉所有層。

## <a name="replicating-virtual-machines"></a>複寫虛擬機器

請依照[本指引](site-recovery-vmware-to-azure.md)開始將虛擬機器複寫至 Azure。

* 複寫完成後，請務必移至每層的每部虛擬機器，然後在 [複寫項目 > 設定 > 屬性 > 計算和網路] 中選取相同的可用性設定組。 例如，如果您的 Web 層有 3 部 VM，請確定這 3 部 VM 全會設定為 Azure 中相同可用性設定組的一部分。

    ![Set-Availability-Set](./media/site-recovery-sharepoint/select-av-set.png)

* 如需保護 Active Directory 和 DNS 的指引，請參閱[保護 Active Directory 和 DNS](site-recovery-active-directory.md) 文件。

* 如需保護在 SQL Server 上執行之資料庫層的指引，請參閱[保護 SQL Server](site-recovery-active-directory.md) 文件。

## <a name="networking-configuration"></a>網路設定

### <a name="network-properties"></a>網路屬性

* 對於應用程式和 Web 層 VM，在 Azure 入口網站中設定網路設定，讓 VM 能在容錯移轉之後連結到正確的 DR 網路。

    ![選取網路](./media/site-recovery-sharepoint/select-network.png)


* 如果您是使用靜態 IP 位址，然後在 [目標 IP] 欄位中指定您希望虛擬機器採用的 IP

    ![設定靜態 IP](./media/site-recovery-sharepoint/set-static-ip.png)

### <a name="dns-and-traffic-routing"></a>DNS 和流量路由

針對網際網路面向網站，在 Azure 訂用帳戶中[建立「優先順序」類型的流量管理員設定檔](../traffic-manager/traffic-manager-create-profile.md)。 然後以下列方式設定您的 DNS 和流量管理員設定檔。


| **其中** | **來源** | **目標**|
| --- | --- | --- |
| 公用 DNS | SharePoint 網站的 公用 DNS <br/><br/> 例如︰sharepoint.contoso.com | 流量管理員 <br/><br/> contososharepoint.trafficmanager.net |
| 內部部署 DNS | sharepointonprem.contoso.com | 內部部署伺服器陣列上的公用 IP |


在流量管理員設定檔中，[建立主要和復原端點](../traffic-manager/traffic-manager-configure-priority-routing-method.md)。 使用內部部署端點的外部端點和 Azure 端點的公用 IP。 確保內部部署端點的優先順序設定為較高。

在 SharePoint Web 層中的特定通訊埠 (例如 800) 上裝載測試頁，以便流量管理員自動偵測容錯移轉後的可用性。 如果您無法在任何 SharePoint 網站上啟用匿名驗證，這是個解決辦法。

使用下列設定來[設定流量管理員設定檔](../traffic-manager/traffic-manager-configure-priority-routing-method.md)。

* 路由方法 -「優先順序」
* DNS 存留時間 (TTL) -「30 秒」
* 端點監視器設定 - 如果您可以啟用匿名驗證，即可提供特定的網站端點。 或者，您可以在特定連接埠 (例如 800) 上使用測試頁。

## <a name="creating-a-recovery-plan"></a>建立復原計劃

復原計劃可讓您在多層式應用程式中排序各層的容錯移轉，因此可維護應用程式一致性。 建立多層式 web 應用程式的復原計畫時請，遵循下列步驟。 [深入了解如何建立復原計劃](site-recovery-runbook-automation.md#customize-the-recovery-plan)。

### <a name="adding-virtual-machines-to-failover-groups"></a>將虛擬機器新增至容錯移轉群組

1. 新增應用程式和 Web 層 VM，以建立復原方案。
2. 按一下 [自訂] 將 VM 分組。 根據預設，所有 VM 都是「群組 1」的一部分。

    ![自訂 RP](./media/site-recovery-sharepoint/rp-groups.png)

3. 建立另一個群組 (群組 2)，並將 Web 層 VM 移到新群組中。 您的應用程式層 VM 應該是「群組 1」的一部分，而 Web 層 VM 應該是「群組 2」的一部分。 這是為了確保應用程式層 VM 先啟動，Web 層 VM 再接著啟動。


### <a name="adding-scripts-to-the-recovery-plan"></a>將指令碼新增至復原計畫

您可以按一下底下的 [部署至 Azure] 按鈕，將最常用的 Azure Site Recovery 指令碼部署至您的自動化帳戶。 當您使用任何已發佈的指令碼時，請務必遵循指令碼中的指引。

[![部署至 Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

1. 將動作前指令碼新增至「群組 1」，以容錯移轉 SQL 可用性群組。 使用在範例指令碼中發佈的 'ASR-SQL-FailoverAG' 指令碼。 請務必遵循指令碼中的指引，並在指令碼中適當地進行必要的變更。

    ![Add-AG-Script-Step-1](./media/site-recovery-sharepoint/add-ag-script-step1.png)

    ![Add-AG-Script-Step-2](./media/site-recovery-sharepoint/add-ag-script-step2.png)

2. 新增動作後指令碼，以連結 Web 層 (群組 2) 之已容錯移轉虛擬機器上的負載平衡器。 使用在範例指令碼中發佈的 'ASR-AddSingleLoadBalancer' 指令碼。 請務必遵循指令碼中的指引，並在指令碼中適當地進行必要的變更。

    ![Add-LB-Script-Step-1](./media/site-recovery-sharepoint/add-lb-script-step1.png)

    ![Add-LB-Script-Step-2](./media/site-recovery-sharepoint/add-lb-script-step2.png)

3. 新增手動步驟來更新 DNS 記錄，以指向 Azure 中新的伺服器陣列。

    * 若為網際網路面向網站，容錯移轉後不需要更新 DNS。 遵循「網路指引」一節所述的步驟來設定流量管理員。 如果已如上一節所述設定流量管理員設定檔，請新增一個指令碼，以開啟 Azure VM 上的虛擬連接埠 (在此範例中為 800)。

    * 針對內部面向網站，新增一個手動步驟來更新 DNS 記錄，以指向新 Web 層 VM 的負載平衡器 IP。

4. 新增一個手動步驟，以從備份還原搜尋應用程式或啟動新的搜尋服務。

5. 如需從備份還原搜尋服務應用程式，請遵循下列步驟。

    * 此方法假設搜尋服務應用程式的備份已在災難性事件之前執行，而且可在 DR 網站上取得該備份。
    * 排定備份 (例如，每天一次) 並使用複製程序將備份置於 DR 網站上，即可輕鬆達成。 複製程序可能包含指令碼程式，例如 AzCopy (Azure 複製) 或設定 DFSR (分散式檔案服務複寫)。
    * 既然 SharePoint 伺服器陣列正在執行中，請瀏覽 [管理中心]、[備份及還原]，然後選取 [還原]。 還原會質詢所指定的備份位置 (您可能需要更新此值)。 選取您想要還原的搜尋服務應用程式備份。
    * 已還原搜尋。 請記住，還原應會發現相同的拓撲 (相同的伺服器數目) 以及指派給這些伺服器的相同硬碟磁碟機代號。 如需詳細資訊，請參閱[在 SharePoint 2013 中還原搜尋服務應用程式](https://technet.microsoft.com/library/ee748654.aspx)文件。


6. 如需從新的搜尋服務應用程式著手，請遵循下列步驟。

    * 此方法假設可在 DR 網站上取得「搜尋管理」資料庫的備份。
    * 由於不會複寫其他搜尋服務應用程式資料庫，因此必須加以重建。 若要這麼做，請瀏覽至管理中心並刪除搜尋服務應用程式。 在任何裝載搜尋索引的伺服器上，刪除索引檔案。
    * 重建搜尋服務應用程式，而這會重建資料庫。 建議備妥一個指令碼來重建此服務應用程式，因為不可能透過 GUI 執行所有動作。 例如，唯有使用 SharePoint PowerShell cmdlet，才可能設定索引磁碟機位置及設定搜尋拓撲。 使用 Windows PowerShell Cmdlet Restore-SPEnterpriseSearchServiceApplication 並指定記錄隨附和複寫的搜尋管理資料庫 Search_Service__DB。 此 Cmdlet 可提供搜尋組態、結構描述、受控屬性、規則和來源，並建立一組預設的其他元件。
    * 一旦重建搜尋服務應用程式，您就必須為每個內容來源啟動完整編目，以還原搜尋服務。 您會從內部部署伺服器陣列遺失一些分析資訊，例如搜尋建議。

7. 完成所有步驟後，儲存復原方案，而最終的復原方案如下所示。

    ![已儲存的 RP](./media/site-recovery-sharepoint/saved-rp.png)

## <a name="doing-a-test-failover"></a>執行測試容錯移轉
請依照[本指引](site-recovery-test-failover-to-azure.md)來執行測試容錯移轉。

1.  請移至 Azure 入口網站，然後選取您的復原服務保存庫。
2.  按一下為 SharePoint 應用程式建立的復原方案。
3.  按一下 [測試容錯移轉]。
4.  選取復原點和 Azure 虛擬網路來開始測試容錯移轉程序。
5.  次要環境啟動後，您就可以執行您的驗證。
6.  完成驗證後，您可以按一下復原方案上的 [清除測試容錯移轉]，而測試容錯移轉環境會清除。

如需有關進行 AD 和 DNS 之測試容錯移轉的指引，請參閱 [AD 和 DNS 的測試容錯移轉考量](site-recovery-active-directory.md#test-failover-considerations)文件。

如需有關進行 SQL Always ON 可用性群組之測試容錯移轉的指引，請參閱[進行 SQL Server Always On 的測試容錯移轉](site-recovery-sql.md#steps-to-do-a-test-failover)文件。

## <a name="doing-a-failover"></a>執行容錯移轉
請依照[本指引](site-recovery-failover.md)來進行容錯移轉。

1.  請移至 Azure 入口網站，然後選取您的復原服務保存庫。
2.  按一下為 SharePoint 應用程式建立的復原方案。
3.  按一下 [容錯移轉]。
4.  選取復原點以啟動容錯移轉程序。

## <a name="next-steps"></a>後續步驟
您可以深入了解如何使用 Site Recovery [複寫其他應用程式](site-recovery-workload.md)。
