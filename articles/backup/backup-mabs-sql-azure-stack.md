---
title: 備份 Azure Stack 中的 SQL Server 工作負載
description: 使用 Azure 備份伺服器來保護 Azure Stack 中的 SQL Server 工作負載。
services: backup
author: pvrk
manager: Shivamg
ms.service: backup
ms.topic: conceptual
ms.date: 6/8/2018
ms.author: pullabhk
ms.openlocfilehash: ca7da7ab048b6f7bfdba81aac9bc7702b20ff967
ms.sourcegitcommit: 6eb14a2c7ffb1afa4d502f5162f7283d4aceb9e2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/25/2018
ms.locfileid: "36751792"
---
# <a name="back-up-sql-server-on-stack"></a>備份 Stack 上的 SQL Server
運用本文資訊來設定 Microsoft Azure 備份伺服器 (MABS)，以保護 Azure Stack 中的 SQL Server 資料庫。

將 SQL Server 資料庫備份至 Azure 及從 Azure 復原的管理作業包含三個步驟：

1. 建立備份原則以保護 SQL Server 資料庫
2. 建立隨選備份複本
3. 從磁碟和從 Azure 中復原資料庫

## <a name="before-you-start"></a>開始之前

[安裝及準備 Azure 備份伺服器](backup-mabs-install-azure-stack.md)。

## <a name="create-a-backup-policy-to-protect-sql-server-databases-to-azure"></a>建立備份原則以在 Azure 保護 SQL Server 資料庫
1. 在 Azure 備份伺服器 UI 上，按一下 [保護] 工作區。

2. 在工具功能區中，按一下 [新增]  以建立新的保護群組。

    ![建立保護群組](./media/backup-azure-backup-sql/protection-group.png)

    Azure 備份伺服器會啟動 [保護群組] 精靈，引導您建立**保護群組**。 按 [下一步] 。

3. 在 [選取保護群組類型] 畫面中，選取 [伺服器]。

    ![選取保護群組類型 - 伺服器](./media/backup-azure-backup-sql/pg-servers.png)

4. 在 [選取群組成員] 畫面中，[可用成員] 清單會顯示各種資料來源。 按一下 **+**，以展開資料夾，並顯示子資料夾。 按一下核取方塊以選取項目。

    ![選取 SQL DB](./media/backup-azure-backup-sql/pg-databases.png)

    所有選取的項目會出現在 [選取的成員] 清單。 選取您要保護的伺服器或資料庫之後，按 [下一步]。

5. 在 [選擇資料保護方式] 畫面中，提供保護群組的名稱，然後選取 [我想要線上保護] 核取方塊。

    ![資料保護方式 - 短期磁碟和線上 Azure](./media/backup-azure-backup-sql/pg-name.png)

6. 在 [指定短期目標] 畫面中，輸入建立磁碟備份點的必要資訊，然後按 [下一步]。

    在此範例中，[保留範圍] 是 **5 天**，[同步處理頻率] 是每 **15 分鐘**一次，即是備份的頻率。 [快速完整備份] 設定為 [下午 8:00]。

    ![短期目標](./media/backup-azure-backup-sql/pg-shortterm.png)

   > [!NOTE]
   > 在範例中，每天晚上 8:00 會傳輸前一天晚上 8:00 備份點過後所修改的資料，以此建立備份點。 這個程序稱為 [快速完整備份] 。 交易記錄每隔 15 分鐘會同步處理。 如果在晚上 9:00 時需要復原資料庫，則會用最新的快速完整備份點來建立復原點，在此例中即是晚上 8 點。
   >
   >

7. 在 [檢閱磁碟配置] 畫面中，確認可用的整體儲存空間和潛在的磁碟空間。 按 [下一步] 。

8. 在 [選擇複本的建立方式] 中，選擇建立第一個復原點的方式。 您可以手動 (關閉網路) 傳輸此初始備份，以避免頻寬壅塞，或透過網路進行傳輸。 如果您選擇等候以傳送第一個備份，您可以指定初始傳輸的時間。 按 [下一步] 。

    ![初始複寫方法](./media/backup-azure-backup-sql/pg-manual.png)

    初始備份副本要求將整個資料來源 (SQL Server 資料庫) 從生產伺服器 (SQL Server 電腦) 傳輸到 Azure備份伺服器。 這些資料可能會很大，因此透過網路傳輸資料可能會超出頻寬。 基於這個理由，您可以選擇傳送初始備份的方式︰[手動] (使用卸除式媒體) 可避免頻寬壅塞，也可選擇 [自動透過網路] 在指定時間傳輸。

    一旦完成初始備份，其餘的備份會是初始備份複本的增量備份。 增量備份通常都非常小，因此有利於透過網路傳輸。

9. 選擇是否想要執行一致性檢查，然後按一下 [下一步] 。

    ![一致性檢查](./media/backup-azure-backup-sql/pg-consistent.png)

    Azure 備份伺服器可以執行一致性檢查來檢查備份點的完整性。 Azure 備份伺服器會計算生產伺服器 (在此案例中為 SQL Server 電腦) 上備份檔案的總和檢查碼以及該檔案的備份資料。 如果有衝突，則會假設 Azure 備份伺服器上的備份檔案已損毀。 Azure 備份伺服器會傳送對應至總和檢查碼不符的區塊，藉此改正已備份的資料。 由於一致性檢查會耗用大量效能，您可以安排要在何時進行一致性檢查，也可以自動執行檢查。

10. 若要指定資料來源的線上保護，請選取要送至 Azure 保護的資料庫，然後按一下 [下一步] 。

    ![選取資料來源](./media/backup-azure-backup-sql/pg-sqldatabases.png)

11. 選擇備份排程以及符合其組織原則的保留原則。

    ![排程和保留](./media/backup-azure-backup-sql/pg-schedule.png)

    在此範例中，每天下午 12:00 和下午 8:00 會進行一次備份 (螢幕的下半部)

    > [!NOTE]
    > 建議您在磁碟上保留幾個短期復原點，以便快速復原。 這些復原點可用於操作復原。 Azure 可做為具有較高 SLA 和保證可用性的良好離站位置。
    >
    >

    **最佳做法**：如果您排定在本機磁碟備份完成後，再開始備份到 Azure，則最新的磁碟備份定會複製到 Azure。

12. 選擇保留原則排程。 如需保留原則運作方式的詳細資訊，請參閱 [使用 Azure 備份來取代您的磁帶基礎結構](backup-azure-backup-cloud-as-tape.md)一文。

    ![保留原則](./media/backup-azure-backup-sql/pg-retentionschedule.png)

    在此範例中：

    * 每天下午 12:00 和下午 8:00 會進行一次備份 (螢幕的下半部) 並保留 180 天。
    * 星期六下午 12:00 的備份 會保留 104 週
    * 上星期六下午 12:00 的備份 會保留 60 週
    * 三月份最後一個星期六下午 12:00 的備份 會保留 10 年
13. 按一下 [下一步]  並選取適當的選項，以便將初始備份複本傳輸至 Azure。 您可以選擇 [自動透過網路]

14. 在 [摘要] 畫面中檢閱原則詳細資料後，按一下 [建立群組] 以完成工作流程。 您可以按一下 [關閉]，並在 [監視] 工作區中監視工作進度。

    ![保護群組的建立正在進行中](./media/backup-azure-backup-sql/pg-summary.png)

## <a name="on-demand-backup-of-a-sql-server-database"></a>SQL Server 資料庫的隨選備份
雖然先前的步驟建立了備份原則，但只有在第一次備份發生時，才會建立「復原點」。 不需等待排程器開始作用，下列步驟會手動觸發復原點的建立。

1. 建立復原點之前，請等到資料庫的保護群組狀態顯示 [確定]  。

    ![保護群組成員](./media/backup-azure-backup-sql/sqlbackup-recoverypoint.png)
2. 以滑鼠右鍵按一下資料庫，然後選取 [建立復原點] 。

    ![建立線上復原點](./media/backup-azure-backup-sql/sqlbackup-createrp.png)
3. 在下拉式功能表中選擇 [線上保護]，然後按一下 [確定]，開始在 Azure 中建立復原點。

    ![建立復原點](./media/backup-azure-backup-sql/sqlbackup-azure.png)
4. 在 [監視] 工作區中檢視工作進度。

    ![監視主控台](./media/backup-azure-backup-sql/sqlbackup-monitoring.png)

## <a name="recover-a-sql-server-database-from-azure"></a>從 Azure 復原 SQL Server 資料庫
以下是從 Azure 復原受保護的實體 (SQL Server 資料庫) 所需的步驟。

1. 開啟 [Azure 備份伺服器管理主控台]。 前往 [復原] 工作區，您可以在此處查看受到保護的伺服器。 瀏覽所需的資料庫 (在此例中為 ReportServer$MSDPM2012)。 選取 [復原開始時間]，該時間會指定為**線上**點。

    ![選取復原點](./media/backup-azure-backup-sql/sqlbackup-restorepoint.png)
2. 以滑鼠右鍵按一下資料庫名稱，然後按一下 [復原]。

    ![從 Azure 復原](./media/backup-azure-backup-sql/sqlbackup-recover.png)
3. MABS 會顯示復原點的詳細資料。 按 [下一步] 。 若要覆寫資料庫，請選取復原類型 [復原到原始的 SQL Server 執行個體] 。 按 [下一步] 。

    ![復原到原始位置](./media/backup-azure-backup-sql/sqlbackup-recoveroriginal.png)

    在本例中，MABS 會將資料庫復原至另一個 SQL Server 執行個體或獨立的網路資料夾。

4. 在 [指定復原選項]  畫面上，您可以選取 [網路頻寬使用節流設定] 等復原選項來進行復原所用頻寬的節流。 按 [下一步] 。

5. 在 [摘要]  畫面中，您會看到目前為止提供的所有復原組態。 按一下 [復原] 。

    [復原狀態] 會顯示正在復原的資料庫。 您可以按一下 [關閉] 關閉精靈，並在 [監視] 工作區中檢視進度。

    ![起始復原程序](./media/backup-azure-backup-sql/sqlbackup-recoverying.png)

    一旦完成復原，還原的資料庫會是應用程式一致複本。

## <a name="next-steps"></a>後續步驟

請參閱[備份檔案和應用程式](backup-mabs-files-applications-azure-stack.md)一文。
請參閱 [在 Azure Stack 上備份 SharePoint (英文)](backup-mabs-sharepoint-azure-stack.md) 一文。
