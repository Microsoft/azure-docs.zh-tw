---
title: 使用 Azure Site Recovery 將 Linux VM 移轉到 Azure 進階儲存體 | Microsoft Docs
description: 使用 Site Recovery 將您現有的虛擬機器移轉到 Azure 進階儲存體。 「進階儲存體」可針對在「Azure 虛擬機器」上執行且需要大量 I/O 的工作負載，提供高效能、低延遲的磁碟支援。
services: virtual-machines-linux,storage
cloud: Azure
author: luywang
ms.service: virtual-machines-linux
ms.tgt_pltfrm: linux
ms.topic: article
ms.date: 08/15/2017
ms.author: luywang
ms.component: disks
ms.openlocfilehash: 5c7a136149bac2fb5b6368bf891e5d8021e12bdd
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39715347"
---
# <a name="migrate-to-premium-storage-by-using-azure-site-recovery"></a>使用 Azure Site Recovery 移轉到進階儲存體

針對執行時需要大量 I/O 之工作負載的虛擬機器 (VM)，[Azure 進階儲存體](premium-storage.md)可提供高效能、低延遲的磁碟支援。 本指南會協助您使用 [Azure Site Recovery](../../site-recovery/site-recovery-overview.md) 將 VM 磁碟從標準儲存體帳戶移轉到進階儲存體帳戶。

Site Recovery 是一項 Azure 服務，可藉由將內部部署實體伺服器和 VM 的複寫協調至雲端 (Azure) 或次要資料中心，協助您建立商務持續性和災害復原的策略。 當您的主要位置發生故障時，您容錯移轉至次要位置，讓應用程式和工作負載保持可用。 當它恢復正常作業時，容錯回復至您的主要位置。 

Site Recovery 可提供測試用容錯移轉，既能支援災害復原演練，又不會影響生產環境。 在發生未預期的災害時，您可以執行容錯移轉，讓資料損失量降到最低 (取決於複寫頻率)。 在移轉到進階儲存體的案例中，您可以使用 [Site Recovery 中的容錯移轉](../../site-recovery/site-recovery-failover.md)，將目標磁碟移轉到進階儲存體帳戶。

我們的建議是使用 Site Recovery 來移轉到進階儲存體，因為此選項的停機時間最短， 也不需要手動執行複製磁碟和建立新 VM 的作業。 在容錯移轉期間，Site Recovery 會有系統地複製您的磁碟並建立新的 VM。 

Site Recovery 支援數種類型的容錯移轉，且停機時間最短或甚至不用停機。 若要規劃停機時間並預估資料損失情形，請參閱 Site Recovery 中的[容錯移轉的類型](../../site-recovery/site-recovery-failover.md)。 如果您[準備在容錯移轉後連接到 Azure VM](../../site-recovery/vmware-walkthrough-overview.md)，您應該可以在容錯移轉後使用 RDP 連接到 Azure VM。

![災害復原圖表][1]

## <a name="azure-site-recovery-components"></a>Azure Site Recovery 元件

以下 Site Recovery 元件與此移轉案例有關：

* **組態伺服器**是協調通訊以及管理資料複寫與復原程序的 Azure VM。 在此 VM 上，您會執行單一安裝程式檔案，以安裝組態伺服器和其他元件 (稱為處理序伺服器) 來做為複寫閘道。 請參閱[組態伺服器必要條件](../../site-recovery/vmware-walkthrough-overview.md)。 您只需設定組態伺服器一次，即可用於所有移轉到相同區域的作業。

* **處理序伺服器**是複寫閘道器，能夠： 

  1. 從來源 VM 接收複寫資料。
  2. 運用快取、壓縮和加密將該資料最佳化。
  3. 將資料傳送至儲存體帳戶。 

  它還會處理對來源 VM 進行的行動服務推送安裝作業，並執行來源 VM 的自動探索。 組態伺服器上會安裝預設的處理序伺服器。 您可以部署額外的獨立處理序伺服器來調整您的部署。 請參閱[處理序伺服器部署的最佳作法](https://azure.microsoft.com/blog/best-practices-for-process-server-deployment-when-protecting-vmware-and-physical-workloads-with-azure-site-recovery/)和[部署額外的處理序伺服器](../../site-recovery/site-recovery-plan-capacity-vmware.md#deploy-additional-process-servers)。 您只需設定處理序伺服器一次，即可用於所有移轉到相同區域的作業。

* **行動服務**是部署在每個您想要複寫之標準 VM 上的元件。 它會擷取在標準 VM 上寫入的資料，並將它們轉送到處理序伺服器。 請參閱[複寫之機器的必要條件](../../site-recovery/vmware-walkthrough-overview.md)。

此圖顯示這些元件如何互動：

![Site Recovery 元件的互動][15]

> [!NOTE]
> Site Recovery 不支援儲存空間磁碟的移轉。

如需適用於其他案例的其他元件，請參閱[案例架構](../../site-recovery/vmware-walkthrough-overview.md)。

## <a name="azure-essentials"></a>Azure 基本元件

以下是適用於此移轉案例的 Azure 需求：

* Azure 訂用帳戶。
* 用來儲存所複寫資料的 Azure 進階儲存體帳戶。
* 在容錯移轉時建立 VM 時，VM 將連接的 Azure 虛擬網路。 Azure 虛擬網路所在的區域必須與 Site Recovery 執行的區域相同。
* 用來儲存複寫記錄的 Azure 標準儲存體帳戶。 這可以是和所要移轉之 VM 磁碟相同的儲存體帳戶。

## <a name="prerequisites"></a>必要條件

* 了解上一節的相關移轉案例元件。
* 了解 [Site Recovery 中的容錯移轉](../../site-recovery/site-recovery-failover.md)中的容錯移轉，以規劃停機時間。

## <a name="setup-and-migration-steps"></a>設定和移轉步驟

您可以使用 Site Recovery 在區域間或相同區域內移轉 Azure IaaS VM。 下列指示專用於[將 VMware VM 或實體伺服器複寫至 Azure](../../site-recovery/vmware-walkthrough-overview.md) 一文中的移轉案例。 除了這篇文章中的指示外，請遵循連結以取得詳細步驟。

### <a name="step-1-create-a-recovery-services-vault"></a>步驟 1：建立復原服務保存庫

1. 開啟 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [建立群組] > [管理] > [備份] 和 [Site Recovery (OMS)]。 或者，您也可以選取 [瀏覽] > [復原服務保存庫] > [加入]。 
3. 指定 VM 將複寫到的地區。 若要在相同區域中移轉，請選取來源 VM 和來源儲存體帳戶所在的區域。 

### <a name="step-2-choose-your-protection-goals"></a>步驟 2︰選擇您的保護目標 

1. 在您要安裝組態伺服器的 VM 上，開啟 [Azure 入口網站](https://portal.azure.com)。
2. 前往 [復原服務保存庫] > [設定] > [Site Recovery] > [步驟 1︰準備基礎結構] > [保護目標]。

   ![瀏覽至 [保護目標] 窗格][2]

3. 在 [保護目標] 底下的第一個下拉式清單中，選取 [至 Azure]。 在第二個下拉式清單中，選取 [未虛擬化/其他]，然後選取 [確定]。

   ![選項已填妥的 [保護目標] 窗格][3]

### <a name="step-3-set-up-the-source-environment-configuration-server"></a>步驟 3：設定來源環境 (組態伺服器)

1. 下載 **Azure Site Recovery 整合安裝**和保存庫註冊金鑰，方法是前往 [準備基礎結構][準備來源][新增伺服器] >  >  窗格。 
 
   必須有保存庫註冊金鑰才能執行整合安裝。 該金鑰在產生後會維持 5 天有效。

   ![瀏覽至 [新增伺服器] 窗格][4]

2. 在 [新增伺服器] 窗格中新增組態伺服器。

   ![已選取組態伺服器的 [新增伺服器] 窗格][5]

3. 在您用來作為組態伺服器的 VM 上，執行統一安裝以安裝組態伺服器和處理序伺服器。 您可以按照[螢幕擷取畫面逐步進行](../../site-recovery/vmware-walkthrough-overview.md)，以完成安裝。 您可以參考下列螢幕擷取畫面，來了解此移轉案例的指定步驟。

   1. 在 [開始之前] 選取 [安裝設定伺服器和處理序伺服器]。

      ![[開始之前] 頁面][6]

   2. 在 [註冊] 中，瀏覽並選取您從保存庫下載的註冊金鑰。

      ![[註冊] 窗格][7]

   3. 在 [環境詳細資料] 中，選取您是否要複寫 VMware VM。 針對這個移轉案例，請選擇 [否]。

      ![[環境詳細資料] 頁面][8]

4. 安裝完成後，請在 [Microsoft Azure Site Recovery 組態伺服器] 視窗中執行下列動作：
 
   1. 使用 [管理帳戶] 索引標籤來建立可供 Site Recovery 用於自動探索的帳戶  (在有關保護實體機器的案例中，設定帳戶並非我們的討論範圍，但您至少需要一個帳戶才能啟用下列其中一個步驟。 在此案例中，您可以任意命名帳戶和密碼)。 
   2. 使用 [保存庫註冊] 索引標籤上傳保存庫認證檔。

      ![[保存庫註冊] 索引標籤][9]

### <a name="step-4-set-up-the-target-environment"></a>步驟 4︰設定目標環境

選取 [準備基礎結構] > [目標]，然後指定您要在容錯移轉後用於 VM 的部署模型。 視您的案例而定，您可以選擇 [傳統] 或 [Resource Manager]。

![[目標] 窗格][10]

Site Recovery 會檢查您是否有一或多個相容的 Azure 儲存體帳戶和網路。 

> [!NOTE]
> 如果您使用進階儲存體帳戶來存放複寫的資料，則必須設定其他標準儲存體帳戶來儲存複寫記錄。

### <a name="step-5-set-up-replication-settings"></a>步驟 5：設定複寫設定

若要確認組態伺服器已順利與您所建立的複寫原則相關聯，請遵循[設定複寫設定](../../site-recovery/vmware-walkthrough-overview.md)。

### <a name="step-6-plan-capacity"></a>步驟 6：規劃產能

1. 使用 [Capacity Planner](../../site-recovery/site-recovery-capacity-planner.md) 準確地估計網路頻寬、儲存體和其他需求，以符合複寫的需求。 
2. 當您完成時，在 [是否已完成容量規劃？] 中選取 [是，已完成]。

   ![確認已完成容量規劃的方塊][11]

### <a name="step-7-install-the-mobility-service-and-enable-replication"></a>步驟 7：安裝行動服務並啟用複寫

1. 您可以選擇[推送安裝](../../site-recovery/vmware-walkthrough-overview.md)到來源 VM，或在來源 VM 上[手動安裝行動服務](../../site-recovery/site-recovery-vmware-to-azure-install-mob-svc.md)。 您可以在提供的連結中找到推送安裝的需求和手動安裝程式的路徑。 如果您要手動安裝，您可能需要使用內部 IP 位址來找到組態伺服器。

   ![[組態伺服器詳細資料] 頁面][12]

   容錯移轉的 VM 會有兩個暫存磁碟︰一個來自主要 VM，另一個則是在 VM 於復原區域中佈建期間所建立。 若要在複寫前排除暫存磁碟，請先安裝行動服務再啟用複寫。 若要深入了解如何排除暫存磁碟，請參閱[從複寫排除磁碟](../../site-recovery/vmware-walkthrough-overview.md)。

2. 啟用複寫，如下所示︰
   1. 選取 [複寫應用程式] > [來源]。 第一次啟用複寫之後，請選取保存庫中的 [+複寫]，以對其他機器啟用複寫。
   2. 在步驟 1 中，設定**來源**做為您的處理序伺服器。
   3. 在步驟 2 中，指定容錯移轉後部署模型、要移轉到的進階儲存體帳戶、要用來儲存記錄的標準儲存體帳戶，以及要容錯移轉到的虛擬網路。
   4. 在步驟 3 中，依 IP 位址新增受保護的 VM。 (您可能需要內部 IP 位址才能找到這些 VM。)
   5. 在步驟 4 中，選取您先前在處理序伺服器上設定的帳戶來設定屬性。
   6. 在步驟 5 中，選擇您先前在「步驟 5：設定複寫設定」中建立的複寫原則。
   7. 選取 [確定] 。

   > [!NOTE]
   > Azure VM 在取消配置後再重新啟動時，不一定會取得相同的 IP 位址。 如果組態伺服器/處理序伺服器或受保護 Azure VM 的 IP 位址變更，此案例中的複寫作業可能不會正確運作。

   ![啟用已選取來源的複寫窗格][13]

在設計 Azure 儲存體環境時，建議您為可用性設定組中的每個 VM 使用不同的儲存體帳戶。 我們建議您遵循儲存體層中的最佳做法，[針對每個可用性設定組使用多個儲存體帳戶](../linux/manage-availability.md)。 將 VM 磁碟分散到多個儲存體帳戶有助於增進儲存體可用性，並將 I/O 分散到整 Azure 儲存體基礎結構。

如果您的 VM 位於可用性設定組，而非將所有 VM 的磁碟複寫到單一儲存體帳戶，強烈建議您將多個 VM 分數次移轉。 如此一來，相同可用性設定組中的 VM 便不會共用單一儲存體帳戶。 使用 [啟用複寫] 窗格，設定每個 VM 的目的地儲存體帳戶，一次一個。
 
您可以根據需求來選擇容錯移轉後部署模型。 如果您選擇 Azure Resource Manager 做為容錯移轉後部署模型，即可將 VM (Resource Manager) 容錯移轉到 VM (Resource Manager)，也可將 VM (傳統) 容錯移轉到 VM (Resource Manager)。

### <a name="step-8-run-a-test-failover"></a>步驟 8：執行測試容錯移轉

若要檢查複寫是否已完成，請選取 Site Recovery 執行個體，然後選取 [設定] > [複寫的項目]。 您將會看到複寫程序的狀態和百分比。 

在初始複寫完成之後，請執行測試容錯移轉來驗證複寫策略。 如需測試容錯移轉的詳細步驟，請參閱[在 Site Recovery 中執行測試容錯移轉](../../site-recovery/vmware-walkthrough-overview.md)。 

> [!NOTE]
> 執行任何容錯移轉前，請確定 VM 和複寫策略符合需求。 如需執行測試容錯移轉的詳細資訊，請參閱[在 Site Recovery 中測試容錯移轉到 Azure](../../site-recovery/site-recovery-test-failover-to-azure.md)。

您可以在 [設定] > [作業] > [YOUR_FAILOVER_PLAN_NAME] 中看到測試容錯移轉的狀態。 在窗格中，您會看到分解步驟和成功/失敗的結果。 如果測試容錯移轉在任一步驟失敗，請選取該步驟來檢查錯誤訊息。 

### <a name="step-9-run-a-failover"></a>步驟 9：執行容錯移轉

在測試容錯移轉完成之後，請執行容錯移轉，將磁碟移轉到進階儲存體，並複寫 VM 執行個體。 依照[執行容錯移轉](../../site-recovery/site-recovery-failover.md#run-a-failover)中的詳細步驟來執行。 

請務必選取 [關閉 VM 並同步處理最新資料]。 此選項會指定 Site Recovery 應嘗試關閉受保護的 VM 並同步處理資料，以便為最新的資料版本進行容錯移轉。 如果您未選取此選項或嘗試失敗，容錯移轉將會來自 VM 的最新可用復原點。 

Site Recovery 會建立類型與可支援進階儲存體之 VM 相同或類似的 VM 執行個體。 您可以前往 [Windows 虛擬機器定價](https://azure.microsoft.com/pricing/details/virtual-machines/windows/)或 [Linux 虛擬機器定價](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)來查看各種 VM 執行個體的效能和價格。

## <a name="post-migration-steps"></a>移轉後步驟

1. **在情況允許時將複寫的 VM 設定至可用性設定組**。 Site Recovery 不支援移轉 VM 以及可用性設定組。 根據複寫 VM 的部署，執行下列其中一項︰
   * 透過傳統部署模型建立的 VM︰在 Azure 入口網站中將 VM 新增至可用性設定組。 如需詳細步驟，請移至[將現有虛擬機器加入至可用性設定組](../linux/classic/configure-availability-classic.md)。
   * 對於透過 Resource Manager 部署模型建立的 VM︰儲存 VM 的組態，然後在可用性設定組中刪除再重新建立 VM。 若要這樣做，請使用[設定 Azure Resource Manager VM 可用性設定組](https://gallery.technet.microsoft.com/Set-Azure-Resource-Manager-f7509ec4)中的指令碼。 執行此指令碼之前，請先檢查其限制並規劃您的停機時間。

2. **刪除舊 VM 和磁碟**。 請確定進階磁碟與來源磁碟一致，而且新的 VM 執行的是與來源 VM 相同的函式。 在 Azure 入口網站中，從來源儲存體帳戶刪除 VM 和磁碟。 如果有即使已刪除 VM，其中的磁碟卻未刪除的問題，請參閱[針對儲存體資源刪除錯誤進行疑難排解](storage-resource-deletion-errors.md)。

3. **清除 Azure Site Recovery 基礎結構**。 如果不再需要 Site Recovery，您可以清除它的基礎結構。 請刪除複寫的項目、組態伺服器和復原原則，然後刪除 Azure Site Recovery 保存庫。

## <a name="troubleshooting"></a>疑難排解

* [監視和疑難排解虛擬機器與實體伺服器的保護](../../site-recovery/site-recovery-monitoring-and-troubleshooting.md)
* [Microsoft Azure Site Recovery 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=hypervrecovmgr)

## <a name="next-steps"></a>後續步驟

如需移轉虛擬機器的特定案例，請參閱下列資源：

* [在儲存體帳戶之間移轉 Azure 虛擬機器](https://azure.microsoft.com/blog/2014/10/22/migrate-azure-virtual-machines-between-storage-accounts/)
* [上傳 Linux 虛擬硬碟](upload-vhd.md)
* [將虛擬機器從 Amazon AWS 移轉至 Microsoft Azure](http://channel9.msdn.com/Series/Migrating-Virtual-Machines-from-Amazon-AWS-to-Microsoft-Azure)

若要深入了解 Azure 儲存體和 Azure 虛擬機器，也請參閱下列資源：

* [Azure 儲存體](https://azure.microsoft.com/documentation/services/storage/)
* [Azure 虛擬機器](https://azure.microsoft.com/documentation/services/virtual-machines/)
* [進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](premium-storage.md)

[1]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-1.png
[2]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-2.png
[3]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-3.png
[4]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-4.png
[5]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-5.png
[6]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-6.PNG
[7]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-7.PNG
[8]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-8.PNG
[9]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-9.PNG
[10]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-10.png
[11]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-11.PNG
[12]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-12.PNG
[13]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-13.png
[14]:../site-recovery/media/site-recovery-vmware-to-azure/v2a-architecture-henry.png
[15]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-14.png
