---
title: StorSimple 8000 系列做為 Backup Exec 的備份目標 | Microsoft Docs
description: 描述搭配 Veritas Backup Exec 的 StorSimple 備份目標組態。
services: storsimple
documentationcenter: ''
author: harshakirank
manager: matd
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/05/2016
ms.author: hkanna
ms.openlocfilehash: 1326e4a84938c46a9e7acc10dd8ed94db708f62a
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37113080"
---
# <a name="storsimple-as-a-backup-target-with-backup-exec"></a>使用 StorSimple 做為 Backup Exec 的備份目標

## <a name="overview"></a>概觀

Azure StorSimple 是 Microsoft 提供的混合式雲端儲存體解決方案。 StorSimple 使用 Azure 儲存體帳戶做為內部部署解決方案的擴充功能，跨內部部署儲存體和雲端儲存體自動將資料分層，解決資料暴增的複雜性問題。

在本文中，我們會討論 StorSimple 與 Veritas Backup Exec 的整合，以及整合這兩種解決方案的最佳作法。 我們也會建議如何設定 Backup Exec 來與 StorSimple 進行最佳的整合。 我們遵循 Veritas 的最佳作法，並聽從備份架構設計人員和管理員關於如何設定 Backup Exec 以符合個人備份需求和服務等級協定 (SLA) 的意見。

雖然我們會說明設定步驟和重要概念，但本文不是逐步的設定或安裝指南。 我們假設基本元件和基礎結構都已正常運作，並準備好支援我們所描述的概念。

### <a name="who-should-read-this"></a>哪些人應閱讀本文？

本文中的資訊最適合於了解儲存體、Windows Server 2012 R2、乙太網路、雲端服務和 Backup Exec 等知識的備份管理員、儲存體管理員及儲存體架構設計人員。

## <a name="supported-versions"></a>支援的版本

-   [備份 Exec 16 及更新版本](http://backupexec.com/compatibility)
-   [StorSimple Update 3 及更新版本](storsimple-overview.md#storsimple-workload-summary)


## <a name="why-storsimple-as-a-backup-target"></a>為什麼使用 StorSimple 做為備份目標？

StorSimple 是不錯的備份目標選擇，因為︰

-   為備份應用程式提供標準的本機儲存體，以做為快速備份目的地，而不需要做任何變更。 您也可以將 StorSimple 用來快速還原最新的備份。
-   其雲端分層可與 Azure 雲端儲存體帳戶緊密整合，以使用經濟實惠的 Azure 儲存體。
-   自動提供可供災害復原的異地儲存體。

## <a name="key-concepts"></a>重要概念

如同任何儲存體解決方案，仔細評估解決方案的儲存體效能、SLA、變更速率及容量成長需求是成功與否的關鍵。 主要概念是藉由引進雲端層，對雲端的存取時間和輸送量就能對 StorSimple 執行其工作的能力扮演重要角色。

StorSimple 的設計目的是針對運作定義完善的使用中資料集 (熱資料) 的應用程式提供儲存體。 在此模型中，使用中的資料集儲存在本機層，其餘非使用中/冷/封存的資料集則分層儲存到雲端。 下圖說明這個模型。 幾乎是平坦的綠線表示儲存在 StorSimple 裝置本機層的資料。 紅線則表示儲存在 StorSimple 解決方案所有各層的資料總量。 平坦綠線與呈指數形的紅色曲線之間的空間代表儲存在雲端的資料總量。

**StorSimple 分層**
![StorSimple 分層圖](./media/storsimple-configure-backup-target-using-backup-exec/image1.jpg)

記住此架構，您會發現 StorSimple 非常適合做為備份目標。 您可以使用 StorSimple 進行下列作業：
-   從本機使用中的資料集執行最頻繁的還原。
-   對於異地災害復原和較少進行還原的舊資料使用雲端。

## <a name="storsimple-benefits"></a>StorSimple 的優點

StorSimple 提供的內部部署解決方案，可以不間斷存取內部部署和雲端儲存體，與 Microsoft Azure 緊密整合。

StorSimple 在具有固態裝置 (SSD) 和序列連接 SCSI (SAS) 儲存體的內部部署裝置與 Azure 儲存體之間使用自動分層。 自動分層將經常存取的資料保存在本機的 SSD 和 SAS 層。 而將不常存取的資料移到 Azure 儲存體。

StorSimple 提供下列優點︰

-   獨家的重複資料刪除和壓縮演算法，使用雲端以達到前所未有的重複資料刪除層級
-   高可用性
-   使用 Azure 異地複寫進行異地複寫
-   與 Azure 整合
-   雲端中的資料會進行加密
-   改進災害復原和提高合規性

雖然 StorSimple 展示兩個主要的部署案例 (主要備份目標和次要備份目標)，但基本上只是一般的區塊存放裝置。 StorSimple 會進行所有的壓縮及重複資料刪除。 它會在雲端、應用程式及檔案系統之間順暢地傳送和擷取資料。

如需 StorSimple 的詳細資訊，請參閱 [StorSimple 8000 系列︰混合式雲端儲存體解決方案](storsimple-overview.md)。 此外，您可以檢閱 [StorSimple 8000 系列技術規格](storsimple-technical-specifications-and-compliance.md)。

> [!IMPORTANT]
> 只有 StorSimple 8000 Update 3 或更新版本才支援使用 StorSimple 裝置做為備份目標。

## <a name="architecture-overview"></a>架構概觀

下表顯示裝置機型與架構的初始指導方針。

**StorSimple 的本機和雲端儲存體容量**

| 儲存體容量       | 8100          | 8600            |
|------------------------|---------------|-----------------|
| 本機儲存體容量 | &lt; 10 TiB\*  | &lt; 20 TiB\*  |
| 雲端儲存體容量 | &gt; 200 TiB\* | &gt; 500 TiB\* |
\* 儲存體大小假設沒有重複資料刪除或壓縮。

**StorSimple 的主要和次要備份容量**

| 備份案例  | 本機儲存體容量  | 雲端儲存體容量  |
|---|---|---|
| 主要備份  | 最近的備份會儲存在本機儲存體供快速復原，以符合復原點目標 (RPO) | 備份歷程記錄 (RPO) 可放入雲端容量 |
| 次要備份 | 備份資料的次要複本可儲存在雲端容量  | N/A  |

## <a name="storsimple-as-a-primary-backup-target"></a>使用 StorSimple 做為主要備份目標

在此案例中，備份應用程式會使用 StorSimple 磁碟區，做為備份的唯一儲存機制。 下圖說明解決方案架構，其中所有備份都使用 StorSimple 分層磁碟區進行備份和還原。

![使用 StorSimple 做為主要備份目標的邏輯圖](./media/storsimple-configure-backup-target-using-backup-exec/primarybackuptargetlogicaldiagram.png)

### <a name="primary-target-backup-logical-steps"></a>主要目標備份的邏輯步驟

1.  備份伺服器連絡目標備份代理程式，然後備份代理程式將資料傳輸到備份伺服器。
2.  備份伺服器將資料寫入 StorSimple 分層磁碟區。
3.  備份伺服器更新類別目錄資料庫，然後完成備份作業。
4.  快照集指令碼可觸發 StorSimple 雲端快照集管理員 (啟動或刪除)。
5.  備份伺服器會根據保留原則刪除過期的備份。


### <a name="primary-target-restore-logical-steps"></a>主要目標還原的邏輯步驟

1.  備份伺服器開始從儲存機制還原適當的資料。
2.  備份代理程式接收來自備份伺服器的資料。
3.  備份伺服器完成還原作業。

## <a name="storsimple-as-a-secondary-backup-target"></a>使用 StorSimple 做為次要備份目標

在此案例中，StorSimple 磁碟區主要用於長期保留或封存。

下圖說明初始備份和還原以高效能磁碟區做為目標的架構。 這些備份會依照設定好的排程複製並封存至 StorSimple 分層磁碟區。

請務必調整您的高效能磁碟區大小，使其能夠處理保留原則、容量和效能需求。

![使用 StorSimple 做為次要備份目標的邏輯圖](./media/storsimple-configure-backup-target-using-backup-exec/secondarybackuptargetlogicaldiagram.png)

### <a name="secondary-target-backup-logical-steps"></a>次要目標備份的邏輯步驟

1.  備份伺服器連絡目標備份代理程式，然後備份代理程式將資料傳輸到備份伺服器。
2.  備份伺服器將資料寫入高效能儲存體。
3.  備份伺服器更新類別目錄資料庫，然後完成備份作業。
4.  備份伺服器會根據保留原則將備份複製到 StorSimple。
5.  快照集指令碼可觸發 StorSimple 雲端快照集管理員 (啟動或刪除)。
6.  備份伺服器會根據保留原則刪除過期的備份。

### <a name="secondary-target-restore-logical-steps"></a>次要目標還原的邏輯步驟

1.  備份伺服器開始從儲存機制還原適當的資料。
2.  備份代理程式接收來自備份伺服器的資料。
3.  備份伺服器完成還原作業。

## <a name="deploy-the-solution"></a>部署解決方案

部署解決方案需要三個步驟：
1. 準備網路基礎結構。
2. 部署您的 StorSimple 裝置做為備份目標。
3. 部署 Backup Exec。

下列各節會詳細討論每個步驟。

### <a name="set-up-the-network"></a>設定網路

因為 StorSimple 是與 Azure 雲端整合的解決方案，所以需要作用中並正常運作的 Azure 雲端連線。 此連線供雲端快照集、管理、中繼資料傳輸等操作使用，以及將較舊、較少存取的資料分層儲存到 Azure 雲端儲存體。

為了能以最佳方式執行解決方案，我們建議您遵循下列網路最佳作法︰

-   將 StorSimple 分層連接至 Azure 的連結必須符合您的頻寬需求。 若要達到此目的，將必要的服務品質 (QoS) 層級套用至您的基礎結構參數，以符合您的 RPO 和復原時間目標 (RTO) SLA。
-   最大的 Azure Blob 儲存體存取延遲時間應該大約 80 毫秒。

### <a name="deploy-storsimple"></a>部署 StorSimple

如需 StorSimple 部署的逐步指引，請參閱[部署內部部署 StorSimple 裝置](storsimple-deployment-walkthrough-u2.md)。

### <a name="deploy-backup-exec"></a>部署 Backup Exec

如需 Backup Exec 安裝最佳作法，請參閱 [Backup Exec 安裝的最佳作法](https://www.veritas.com/content/support/en_US/doc/72686287-131623464-0/v70444238-131623464)。

## <a name="set-up-the-solution"></a>設定解決方案

在本節中，我們會示範一些組態範例。 下列範例和建議說明最基本也最重要的實作。 此實作可能無法直接套用到您特定的備份需求。

### <a name="set-up-storsimple"></a>設定 StorSimple

| StorSimple 部署工作  | 其他註解 |
|---|---|
| 部署您的內部部署 StorSimple 裝置。 | 支援的版本：Update 3 及更新版本。 |
| 開啟備份目標。 | 使用下列命令來開啟或關閉備份目標模式，以及取得狀態。 如需詳細資訊，請參閱[從遠端連接至 StorSimple 裝置](storsimple-remote-connect.md)。</br> 若要開啟備份模式︰`Set-HCSBackupApplianceMode -enable`。 </br> 若要關閉備份模式︰`Set-HCSBackupApplianceMode -disable`。 </br> 若要取得備份模式設定的目前狀態：`Get-HCSBackupApplianceMode`。 |
| 為儲存備份資料的磁碟區建立一般的磁碟區容器。 磁碟區容器中的所有資料都已刪除重複資料。 | StorSimple 磁碟區容器定義重複資料刪除網域。  |
| 建立 StorSimple 磁碟區。 | 建立大小盡可能接近預期使用量的磁碟區，因為磁碟區大小會影響雲端快照集的持續時間。 如需有關如何調整磁碟區大小的資訊，請參閱[保留原則](#retention-policies)。</br> </br> 使用 StorSimple 分層磁碟區，並選取 [使用此磁碟區存放不常存取的封存資料] 核取方塊。 </br> 不支援只使用固定在本機的磁碟區。 |
| 為所有備份目標磁碟區建立唯一的 StorSimple 備份原則。 | StorSimple 備份原則定義磁碟區一致性群組。 |
| 在快照集到期時停用排程。 | 後處理作業會觸發快照集。 |

### <a name="set-up-the-host-backup-server-storage"></a>設定主機備份伺服器儲存體

根據下列指導方針，設定主機備份伺服器儲存體︰  

- 不要使用合併磁碟區 (由 Windows 磁碟管理所建立)。 不支援合併磁碟。
- 使用 64 KB 配置大小的 NTFS 格式化磁碟區。
- 將 StorSimple 磁碟區直接對應到 Backup Exec 伺服器。
    - 為實體伺服器使用 iSCSI。
    - 虛擬伺服器請使用傳遞磁碟。

## <a name="best-practices-for-storsimple-and-backup-exec"></a>StorSimple 和 Backup Exec 的最佳作法

根據下列各節中的指導方針來設定您的解決方案。

### <a name="operating-system-best-practices"></a>作業系統最佳作法

-   停用 NTFS 檔案系統的 Windows Server 加密和重複資料刪除。
-   停用 StorSimple 磁碟區的 Windows Server 磁碟重組。
-   停用 StorSimple 磁碟區的 Windows Server 索引。
-   在來源主機 (不是針對 StorSimple 磁碟區) 執行防毒軟體掃描。
-   在工作管理員中關閉預設 [Windows Server 維護](https://msdn.microsoft.com/library/windows/desktop/hh848037.aspx)。 利用下列其中一種方式來執行此作業：
   - 在 Windows 工作排程器中關閉維護設定程式。
   - 從 Windows Sysinternals 下載 [PsExec](https://technet.microsoft.com/sysinternals/bb897553.aspx)。 下載 PsExec 之後，請以系統管理員身分執行 Azure PowerShell：
      ```powershell
      psexec \\%computername% -s schtasks /change /tn “MicrosoftWindowsTaskSchedulerMaintenance Configurator" /disable
      ```

### <a name="storsimple-best-practices"></a>StorSimple 最佳作法

  -   確定 StorSimple 裝置已更新為 [Update 3 或更新版本](storsimple-install-update-3.md)。
  -   隔離 iSCSI 和雲端流量。 針對 StorSimple 和備份伺服器之間的流量使用專用的 iSCSI 連線。
  -   確定 StorSimple 裝置是專用的備份目標。 不支援混合的工作負載，因為它們會影響 RTO 和 RPO。

### <a name="backup-exec-best-practices"></a>Backup Exec 最佳作法

-   Backup Exec 必須安裝在伺服器的本機磁碟機，而不是在 StorSimple 磁碟區。
-   將 Backup Exec 儲存體的**並行寫入作業**設定為允許的最大值。
    -   將 Backup Exec 儲存體的**區塊和緩衝區大小**設定為 512 KB。
    -   開啟 Backup Exec 儲存體的**緩衝讀取和寫入**。
-   StorSimple 支援 Backup Exec 完整和增量備份。 建議您不要使用綜合和差異備份。
-   備份資料檔案最好只包含特定作業的資料。 例如，不允許在不同作業之間附加媒體。
-   停用作業驗證。 如有必要，應該將驗證安排在最新的備份作業之後進行。 請務必了解這項作業會影響您的備份時間範圍。
-   選取 [儲存體] > 您的磁碟 > [詳細資料] > [屬性]。 關閉 [預先配置磁碟空間]。

如需實作這些需求的最新 Backup Exec 設定和最佳作法，請參閱 [Veritas 網站](https://www.veritas.com)。

## <a name="retention-policies"></a>保留原則

其中一種最常用的備份保留原則類型是三代循環 (GFS) 原則。 在 GFS 原則中，增量備份會每日執行，而完整備份會每週和每月進行。 此原則會產生六個 StorSimple 分層磁碟區。 一個磁碟區包含每週、每個月和每年的完整備份。 其他五個磁碟區則儲存每日的增量備份。

在下列範例中，我們使用 GFS 循環。 範例的假設如下：

-   使用非重複資料刪除或壓縮的資料。
-   每個完整備份為 1 TiB。
-   每個每日增量備份為 500 GiB。
-   四個每週備份會保留一個月。
-   十二個每月備份會保留一年。
-   一個每年備份會保留 10 年。

根據先前的假設，為每個月和每年的完整備份建立 26 TiB 的 StorSimple 分層磁碟區。 為每個每日增量備份建立 5 TiB 的 StorSimple 分層磁碟區。

| 備份類型保留期 | 大小 (TiB) | GFS 乘數\* | 總容量 (TiB)  |
|---|---|---|---|
| 每週完整 | 1 | 4  | 4 |
| 每日增量 | 0.5 | 20 (循環數等於每個月的週數) | 12 (2 為其他配額) |
| 每月完整 | 1 | 12 | 12 |
| 每年完整 | 1  | 10 | 10 |
| GFS 需求 |   | 38 |   |
| 其他配額  | 4  |   | 42 (總計 GFS 需求)  |
\*GFS 乘數是您為了符合備份原則需求所需保護和保留的複本數目。

## <a name="set-up-backup-exec-storage"></a>設定 Backup Exec 儲存體

### <a name="to-set-up-backup-exec-storage"></a>若要設定 Backup Exec 儲存體

1.  在 Backup Exec 管理主控台中，選取 [儲存體] > [設定儲存體] > [磁碟型儲存體] > [下一步]。

    ![Backup Exec 管理主控台，設定儲存體頁面](./media/storsimple-configure-backup-target-using-backup-exec/image4.png)

2.  選取 [磁碟儲存體]，然後選取 [下一步]。

    ![Backup Exec 管理主控台，選取儲存體頁面](./media/storsimple-configure-backup-target-using-backup-exec/image5.png)

3.  輸入一個代表性名稱，例如，「星期六完整」和描述。 選取 [下一步] 。

    ![Backup Exec 管理主控台，名稱和描述頁面](./media/storsimple-configure-backup-target-using-backup-exec/image7.png)

4.  選取您要建立磁碟存放裝置的磁碟，然後選取 [下一步]。

    ![Backup Exec 管理主控台，儲存體磁碟選取頁面](./media/storsimple-configure-backup-target-using-backup-exec/image9.png)

5.  將寫入作業數目增加為 **16**，然後選取 [下一步]。

    ![Backup Exec 管理主控台，並行寫入作業設定頁面](./media/storsimple-configure-backup-target-using-backup-exec/image10.png)

6.  檢閱設定，然後選取 [完成]。

    ![Backup Exec 管理主控台，儲存體組態摘要頁面](./media/storsimple-configure-backup-target-using-backup-exec/image11.png)

7.  在每項磁碟區指派作業結束時，變更存放裝置設定，以符合 [StorSimple 和 Backup Exec 的最佳作法](#best-practices-for-storsimple-and-backup-exec)所建議的設定。

    ![Backup Exec 管理主控台，存放裝置設定頁面](./media/storsimple-configure-backup-target-using-backup-exec/image12.png)

8.  重複步驟 1-7，直到完成將 StorSimple 磁碟區指派給 Backup Exec 為止。

## <a name="set-up-storsimple-as-a-primary-backup-target"></a>設定 StorSimple 做為主要備份目標

> [!NOTE]
> 已分層儲存到雲端的備份資料，會以雲端速度進行還原。

下圖說明如何將典型的磁碟區對應到備份作業。 在此情況下，所有的每週備份會對應到星期六完整的磁碟，增量備份則會對應到星期一至星期五的增量磁碟。 所有備份和還原都是從 StorSimple 分層磁碟區進行。

![主要備份目標組態的邏輯圖](./media/storsimple-configure-backup-target-using-backup-exec/primarybackuptargetdiagram.png)

### <a name="storsimple-as-a-primary-backup-target-gfs-schedule-example"></a>使用 StorSimple 做為主要備份目標的 GFS 排程範例

以下是四週、每月和每年的 GFS 循環排程範例：

| 頻率/備份類型 | 完整 | 增量 (第 1-5 天)  |   
|---|---|---|
| 每週 (第 1 - 4 週) | 星期六 | 星期一至星期五 |
| 每月  | 星期六  |   |
| 每年 | 星期六  |   |   |


### <a name="assign-storsimple-volumes-to-a-backup-exec-backup-job"></a>將 StorSimple 磁碟區指派給 Backup Exec 備份作業

下列程序假設 Backup Exec 和目標主機已依據 Backup Exec 代理程式指導方針進行設定。

#### <a name="to-assign-storsimple-volumes-to-a-backup-exec-backup-job"></a>若要將 StorSimple 磁碟區指派給 Backup Exec 備份作業

1.  在 Backup Exec 管理主控台中，選取 [主機] > [備份] > [備份到磁碟]。

    ![Backup Exec 管理主控台，選取主機、備份及備份到磁碟](./media/storsimple-configure-backup-target-using-backup-exec/image14.png)

2.  在 [備份定義屬性] 對話方塊的 [備份] 之下，選取 [編輯]。

    ![Backup Exec 管理主控台，備份定義屬性對話方塊](./media/storsimple-configure-backup-target-using-backup-exec/image15.png)

3.  設定您的完整和增量備份，讓這些備份符合 RPO 和 RTO 需求並遵循 Veritas 的最佳作法。

4.  在 [備份選項] 對話方塊中，選取 [儲存體]。

    ![Backup Exec 管理主控台，備份選項、儲存體對話方塊](./media/storsimple-configure-backup-target-using-backup-exec/image16.png)

5.  將對應的 StorSimple 磁碟區指派到您的備份排程。

    > [!NOTE]
    > [Compression (壓縮)] 和 [Encryption type (加密類型)] 設定為 [None (無)]。

6.  在 [確認] 之下，選取 [不要驗證這個作業的資料] 核取方塊。 使用此選項可能會影響 StorSimple 分層。

    > [!NOTE]
    > 磁碟重組、索引和背景驗證會對 StorSimple 分層造成負面影響。

    ![Backup Exec 管理主控台，備份選項、驗證設定](./media/storsimple-configure-backup-target-using-backup-exec/image17.png)

7.  設定完其餘的備份選項以符合需求之後，選取 [確定] 即可完成。

## <a name="set-up-storsimple-as-a-secondary-backup-target"></a>設定 StorSimple 做為次要備份目標

> [!NOTE]
>已分層儲存到雲端的備份資料，會以雲端速度進行還原。

在此模型中，您必須擁有儲存媒體 (而非 StorSimple) 做為暫時快取。 例如，您可以使用獨立磁碟容錯陣列 (RAID) 磁碟區來容納空間、輸入/輸出 (I/O) 和頻寬。 我們建議使用 RAID 5、50 和 10。

下圖說明一般短期保留的 (相對於伺服器) 本機磁碟區，以及長期保留的封存磁碟區。 在此案例中，所有備份都會在 (相對於伺服器) 本機的 RAID 磁碟區上執行。 這些備份會定期複製並封存到封存磁碟區。 請務必設定您 (相對於伺服器) 本機的 RAID 磁碟區，以便處理短期保留容量和效能需求。

### <a name="storsimple-as-a-secondary-backup-target-gfs-example"></a>使用 StorSimple 做為次要備份目標 GFS 範例

![使用 StorSimple 做為次要備份目標的邏輯圖](./media/storsimple-configure-backup-target-using-backup-exec/secondarybackuptargetdiagram.png)

下表顯示如何設定備份以在本機和 StorSimple 磁碟上執行。 其中包含個別和總容量需求。

### <a name="backup-configuration-and-capacity-requirements"></a>備份設定和容量需求

| 備份類型和保留期 | 設定的儲存體 | 大小 (TiB) | GFS 乘數 | 總容量\* (TiB) |
|---|---|---|---|---|
| 第 1 週 (完整和增量) |本機磁碟 (短期)| 1 | 1 | 1 |
| StorSimple 第 2-4 週 |StorSimple 磁碟 (長期) | 1 | 4 | 4 |
| 每月完整 |StorSimple 磁碟 (長期) | 1 | 12 | 12 |
| 每年完整 |StorSimple 磁碟 (長期) | 1 | 1 | 1 |
|GFS 磁碟區大小需求 |  |  |  | 18*|
\* 總容量包含 17 TiB 的 StorSimple 磁碟和 1 TiB 的本機 RAID 磁碟區。


### <a name="gfs-example-schedule-gfs-rotation-weekly-monthly-and-yearly-schedule"></a>GFS 範例排程：每週、每月和每年排程的 GFS 循環

| 週 | 完整 | 增量 (第 1 天) | 增量 (第 2 天) | 增量 (第 3 天) | 增量 (第 4 天) | 增量 (第 5 天) |
|---|---|---|---|---|---|---|
| 第 1 週 | 本機 RAID 磁碟區  | 本機 RAID 磁碟區 | 本機 RAID 磁碟區 | 本機 RAID 磁碟區 | 本機 RAID 磁碟區 | 本機 RAID 磁碟區 |
| 第 2 週 | StorSimple 第 2-4 週 |   |   |   |   |   |
| 第 3 週 | StorSimple 第 2-4 週 |   |   |   |   |   |
| 第 4 週 | StorSimple 第 2-4 週 |   |   |   |   |   |
| 每月 | StorSimple 每月 |   |   |   |   |   |
| 每年 | StorSimple 每年  |   |   |   |   |   |   |


### <a name="assign-storsimple-volumes-to-a-backup-exec-archive-and-deduplication-job"></a>將 StorSimple 磁碟區指派給 Backup Exec 封存和/重複資料刪除作業

#### <a name="to-assign-storsimple-volumes-to-a-backup-exec-archive-and-duplication-job"></a>若要將 StorSimple 磁碟區指派給 Backup Exec 封存和重複資料刪除作業

1.  在 Backup Exec 管理主控台中，以滑鼠右鍵按一下您想要封存至 StorSimple 磁碟區的作業，然後選取 [備份定義屬性] > **編輯**。

    ![Backup Exec 管理主控台，備份定義屬性索引標籤](./media/storsimple-configure-backup-target-using-backup-exec/image19.png)

2.  選取 [新增階段] > [複製到磁碟] > [編輯]。

    ![Backup Exec 管理主控台，新增階段](./media/storsimple-configure-backup-target-using-backup-exec/image20.png)

3.  在 [複製選項] 對話方塊中，選取您要用於 [來源] 和 [排程] 的值。

    ![Backup Exec 管理主控台，備份定義屬性和複製選項](./media/storsimple-configure-backup-target-using-backup-exec/image21.png)

4.  在 [儲存體] 下拉式清單中，選取您想要封存作業以儲存資料的 StorSimple 磁碟區。

    ![Backup Exec 管理主控台，備份定義屬性和複製選項](./media/storsimple-configure-backup-target-using-backup-exec/image22.png)

5.  選取 [確認]，然後選取 [不要驗證這個作業的資料] 核取方塊。

    ![Backup Exec 管理主控台，備份定義屬性和複製選項](./media/storsimple-configure-backup-target-using-backup-exec/image23.png)

6.  選取 [確定] 。

    ![Backup Exec 管理主控台，備份定義屬性和複製選項](./media/storsimple-configure-backup-target-using-backup-exec/image24.png)

7.  在 [Backup (備份)] 欄中，新增新的階段。 針對來源，使用 [增量]。 針對目標，選擇其中封存增量備份作業的 StorSimple 磁碟區。 重複步驟 1 至 6。

## <a name="storsimple-cloud-snapshots"></a>StorSimple 雲端快照集

StorSimple 雲端快照集可保護位於 StorSimple 裝置中的資料。 建立雲端快照集相當於將本機備份磁碟送到異地場所。 如果您使用 Azure 異地備援儲存體，建立雲端快照集相當於將備份磁帶送到多個站台。 如果您需要在災害後還原裝置，可以讓另一個 StorSimple 裝置上線並執行容錯移轉。 容錯移轉之後，您就可以從最新的雲端快照集 (以雲端速度) 存取資料。

下一節說明如何建立簡短指令碼，以在備份後處理期間啟動和刪除 StorSimple 雲端快照集。

> [!NOTE]
> 以手動方式或以程式設計方式建立的快照集不會遵循 StorSimple 快照集到期原則。 這些快照集必須以手動方式或以程式設計方式刪除。

### <a name="start-and-delete-cloud-snapshots-by-using-a-script"></a>使用指令碼啟動和刪除雲端快照集

> [!NOTE]
> 在刪除 StorSimple 快照集之前，請先仔細評估合規性和資料保留的影響。 如需有關如何執行備份後指令碼的詳細資訊，請參閱 [Backup Exec 文件](https://www.veritas.com/support/en_US/15047.html)。

### <a name="backup-lifecycle"></a>備份生命週期

![備份生命週期圖表](./media/storsimple-configure-backup-target-using-backup-exec/backuplifecycle.png)

### <a name="requirements"></a>需求

-   執行指令碼的伺服器必須能夠存取 Azure 雲端資源。
-   使用者帳戶必須擁有必要的權限。
-   必須設定但不要啟用與 StorSimple 磁碟區相關聯的 StorSimple 備份原則。
-   您需要 StorSimple 資源名稱、註冊金鑰、裝置名稱和備份原則識別碼。

### <a name="to-start-or-delete-a-cloud-snapshot"></a>若要啟動或刪除雲端快照集

1.  [安裝 Azure PowerShell](/powershell/azure/overview)。
2. 下載及安裝 [Manage-CloudSnapshots.ps1](https://github.com/anoobbacker/storsimpledevicemgmttools/blob/master/Manage-CloudSnapshots.ps1) PowerShell 指令碼。
3. 在執行指令碼的伺服器上，以系統管理員身分執行 PowerShell。 請確定您搭配 `-WhatIf $true` 執行指令碼，以查看指令碼會執行哪些變更。 完成驗證之後，傳遞 `-WhatIf $false`。 執行下列命令：
```powershell
.\Manage-CloudSnapshots.ps1 -SubscriptionId [Subscription Id] -TenantId [Tenant ID] -ResourceGroupName [Resource Group Name] -ManagerName [StorSimple Device Manager Name] -DeviceName [device name] -BackupPolicyName [backup policyname] -RetentionInDays [Retention days] -WhatIf [$true or $false]
```
4.  編輯 Backup Exec 作業選項的前處理和後處理命令，以在 Backup Exec 的備份作業中新增指令碼。

    ![Backup Exec 主控台，備份選項，前處理和後處理命令索引標籤](./media/storsimple-configure-backup-target-using-backup-exec/image25.png)

> [!NOTE]
> 我們建議您在每日備份作業結束時執行 StorSimple 雲端快照集集備份原則，做為後處理指令碼。 如需有關如何備份和還原備份應用程式環境以符合 RPO 和 RTO 的詳細資訊，請洽詢您的備份架構設計人員。

## <a name="storsimple-as-a-restore-source"></a>使用 StorSimple 做為還原來源

從 StorSimple 裝置還原的運作方式就如同從任何區塊存放裝置還原。 還原已分層儲存到雲端的資料時，會以雲端速度進行。 如果是本機資料，則會以裝置的本機磁碟速度進行還原。 如需有關如何執行還原的詳細資訊，請參閱 Backup Exec 文件。 我們建議您遵守 Backup Exec 還原的最佳作法。

## <a name="storsimple-failover-and-disaster-recovery"></a>StorSimple 容錯移轉和災害復原

> [!NOTE]
> 對於備份目標的案例，不支援使用 StorSimple Cloud Appliance 做為還原目標。

各種因素都可能造成災害。 下表列出常見的災害復原案例。

| 案例 | 影響 | 如何復原 | 注意 |
|---|---|---|---|
| StorSimple 裝置故障 | 備份和還原作業會中斷。 | 更換故障的裝置，並執行 [StorSimple 容錯移轉和災害復原](storsimple-device-failover-disaster-recovery.md)。 | 如果您需要在裝置復原後執行還原，則會從雲端擷取完整的使用中資料集到新裝置。 所有作業都會以雲端速度進行。 編製索引及編製目錄的重新掃描程序可能會造成所有備份集都要進行掃描並從雲端層提取到本機裝置層，而這可能會非常耗時。 |
| Backup Exec 伺服器故障 | 備份和還原作業會中斷。 | 重建備份伺服器，並依[如何執行手動備份和還原 Backup Exec (BEDB) 資料庫 (英文)](http://www.veritas.com/docs/000041083) 中所述執行資料庫還原。 | 您必須在災害復原站台重建或還原 Backup Exec 伺服器。 將資料庫還原到最新的點。 如果還原的 Backup Exec 資料庫沒有與您最新的備份作業同步，就必須編製索引及編製目錄。 重新掃描索引和目錄的程序可能會造成所有備份集都要進行掃描並從雲端層提取到本機裝置層。 這會更耗費時間。 |
| 站台故障造成備份伺服器和 StorSimple 都遺失 | 備份和還原作業會中斷。 | 先還原 StorSimple，然後再還原 Backup Exec。 | 先還原 StorSimple，然後再還原 Backup Exec。 如果您需要在裝置復原後執行還原，則會從雲端擷取完整的使用中資料集到新裝置。 所有作業都會以雲端速度進行。 |

## <a name="references"></a>參考

本文中參考下列文件︰

- [StorSimple 多重路徑 I/O 設定](storsimple-configure-mpio-windows-server.md)
- [儲存體案例︰精簡佈建 (英文)](http://msdn.microsoft.com/library/windows/hardware/dn265487.aspx)
- [使用 GPT 磁碟機 (英文)](http://msdn.microsoft.com/windows/hardware/gg463524.aspx#EHD)
- [設定共用資料夾的陰影複製](http://technet.microsoft.com/library/cc771893.aspx)

## <a name="next-steps"></a>後續步驟

- 深入了解如何[從備份集還原](storsimple-restore-from-backup-set-u2.md)。
- 深入了解如何執行[裝置容錯移轉和災害復原](storsimple-device-failover-disaster-recovery.md)。
