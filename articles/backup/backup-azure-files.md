---
title: 備份 Azure 檔案共用
description: 本文詳細說明如何備份及還原 Azure 檔案共用，並且說明管理工作。
services: backup
author: markgalioto
ms.author: markgal
ms.date: 3/23/2018
ms.topic: tutorial
ms.service: backup
manager: carmonm
ms.openlocfilehash: d21a235602c425cef77b26d8c60f1e3562411095
ms.sourcegitcommit: 0408c7d1b6dd7ffd376a2241936167cc95cfe10f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/26/2018
ms.locfileid: "36961667"
---
# <a name="back-up-azure-file-shares"></a>備份 Azure 檔案共用
本文說明如何使用 Azure 入口網站來備份和還原 [Azure 檔案共用](../storage/files/storage-files-introduction.md)。

在本指南中，您將了解如何：
> [!div class="checklist"]
> * 設定復原服務保存庫以備份 Azure 檔案
> * 執行隨選備份作業以建立還原點
> * 從還原點還原一或多個檔案
> * 管理備份作業
> * 停止保護 Azure 檔案
> * 刪除備份資料

## <a name="prerequisites"></a>先決條件
請先確定 Azure 檔案共用存在於其中一種[支援的儲存體帳戶類型](backup-azure-files.md#limitations-for-azure-file-share-backup-during-preview)中，您才可以備份 Azure 檔案共用。 一旦確認這點，您即可保護您的檔案共用。

## <a name="limitations-for-azure-file-share-backup-during-preview"></a>預覽期間的 Azure 檔案共用備份限制
Azure 檔案共用的備份處於預覽狀態。 Azure 檔案共用不支援下列備份案例︰
- 您無法使用[讀取權限異地備援儲存體](../storage/common/storage-redundancy-grs.md) (RA-GRS) 複寫功能，保護儲存體帳戶中的 Azure 檔案共用*。
- 您無法在已啟用虛擬網路或防火牆的儲存體帳戶中保護 Azure 檔案共用。
- 無法透過 PowerShell 或 CLI 使用 Azure 備份來保護 Azure 檔案。
- 每天的排程備份次數上限為 1 次。
- 每天的隨選備份次數上限為 4 次。
- 在儲存體帳戶上使用[資源鎖定](https://docs.microsoft.com/cli/azure/resource/lock?view=azure-cli-latest)，以防止在您的復原服務保存庫中意外刪除備份。
- 請勿刪除 Azure 備份所建立的快照集。 刪除快照集可能會導致遺失復原點和/或還原失敗。

\*儲存體帳戶中的 Azure 檔案共用以[讀取權限異地備援儲存體](../storage/common/storage-redundancy-grs.md) (RA-GRS) 複寫功能作為 GRS，並以 GRS 價格計費。

在儲存體帳戶中使用[區域備援儲存體](../storage/common/storage-redundancy-zrs.md) (ZRS) 複寫功能備份 Azure 檔案共用，目前僅適用於美國中部 (CUS)、美國東部 2 (EUS2)、北歐 (NE)、東南亞 (SEA) 和西歐 (WE)。

## <a name="configuring-backup-for-an-azure-file-share"></a>設定 Azure 檔案共用備份
所有備份資料都儲存在復原服務保存庫中。 本教學課程假設您已經建立了 Azure 檔案共用。 若要備份 Azure 檔案共用：

1. 在與您的檔案共用相同的區域中建立復原服務保存庫。 如果您已經有保存庫，請開啟保存庫的 [概觀] 頁面，然後按一下 [備份]。

    ![選擇 Azure 檔案共用作為備份目標](./media/backup-file-shares/overview-backup-page.png)

2. 在 [備份目標] 功能表中，從 [您想要備份什麼？]，選擇 Azure 檔案共用。

    ![選擇 Azure 檔案共用作為備份目標](./media/backup-file-shares/choose-azure-fileshare-from-backup-goal.png)

3. 按一下 [備份]，將 Azure 檔案共用設定為備份至復原服務保存庫。 

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/set-backup-goal.png)

    一旦保存庫與 Azure 檔案共用建立關聯，[備份] 功能表隨即開啟並提示您選取儲存體帳戶。 此功能表會顯示您的保存庫所在區域中所有支援的儲存體帳戶，而且這些儲存體帳戶尚未與復原服務保存庫建立關聯。

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/list-of-storage-accounts.png)

4. 在 [儲存體帳戶] 清單中，選取帳戶並按一下 [確定]。 Azure 會搜尋儲存體帳戶中可以備份的檔案共用。 如果您最近新增了檔案共用，但並未在清單中看到它們，請稍後片刻讓檔案共用出現。

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/discover-file-shares.png)

5. 從 [檔案共用] 清單中，選取您要備份的一或多個檔案，然後按一下 [確定]。

6. 選擇您的檔案共用之後，[備份] 功能表會切換至 [備份原則]。 從這個功能表中選取現有的備份原則，或建立新的備份原則，然後按一下 [啟用備份]。 

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/apply-backup-policy.png)

    建立備份原則之後，系統會在排定的時間產生檔案共用的快照集，並針對所選的期間保留復原點。

## <a name="create-an-on-demand-backup"></a>建立隨選備份
您偶爾可以在備份原則中排定的時間以外，產生備份快照集或復原點。 產生隨選備份的常見時間是在您設定備份原則之後。 根據備份原則中的排程，在產生快照集之前，可能需要幾小時或幾天的時間。 若要在備份原則參與之前保護您的資料，請起始隨選備份。 在您對檔案共用進行計劃性變更之前，通常需要建立隨選備份。 

### <a name="to-create-an-on-demand-backup"></a>若要建立隨選備份：

1. 開啟包含檔案共用復原點的復原服務保存庫，然後按一下 [備份項目]。 [備份項目] 類型清單隨即出現。

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/list-of-backup-items.png)

2. 從清單中，選取 [Azure 儲存體 (Azure 檔案服務)]。 Azure 檔案共用的清單隨即出現。

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/list-of-azure-files-backup-items.png)

3. 從 Azure 檔案共用的清單中，選取所需的檔案共用。 所選檔案共用的 [備份項目] 功能表隨即開啟。

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/backup-item-menu.png)

4. 從 [備份項目] 功能表，按一下 [立即備份]。 因為這是隨選備份作業，所以沒有與復原點相關聯的保留原則。 [立即備份] 對話方塊隨即開啟。 指定您想要保留復原點的最後一天。 
  
   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/backup-now-menu.png)

## <a name="restore-from-backup-of-azure-file-share"></a>從 Azure 檔案共用的備份進行還原
如果您需要從還原點還原整個檔案共用或個別的檔案或資料夾，請前往上一節中詳述的 [備份項目]。 選擇 [還原共用]，以從所需的時間點還原整個檔案共用。 從顯示的 [還原點] 清單中，選取一個還原點，其可覆寫您目前的檔案共用或將它還原至相同區域中的替代檔案共用。

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/select-restore-location.png)

## <a name="restore-individual-files-or-folders-from-backup-of-azure-file-shares"></a>從 Azure 檔案共用的備份還原個別的檔案或資料夾
Azure 備份提供在 Azure 入口網站中瀏覽還原點的功能。 若要還原您所選的檔案或資料夾，請按一下 [備份項目] 頁面中的 [檔案復原]，並從 [還原點] 清單中進行選擇。 選取 [復原目的地]，然後按一下 [選取檔案] 來瀏覽還原點。 選取您所選的檔案或資料夾，然後選取 [還原]。

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/restore-individual-files-folders.png)

## <a name="manage-azure-file-share-backups"></a>管理 Azure 檔案共用備份

您可以在 [備份作業] 頁面上對檔案共用備份執行幾項管理工作，包括：
- [監視工作](backup-azure-files.md#monitor-jobs)
- [建立新的原則](backup-azure-files.md#create-a-new-policy)
- [停止保護檔案共用](backup-azure-files.md#stop-protecting-an-azure-file-share)
- [繼續保護檔案共用](backup-azure-files.md#resume-protection-for-azure-file-share)
- [刪除備份資料](backup-azure-files.md#delete-backup-data)

### <a name="monitor-jobs"></a>監視工作

您可以在 [備份作業] 頁面上監視所有作業的進度。

若要開啟 [備份作業] 頁面：

- 開啟您要監視的復原服務保存庫，然後在 [復原服務保存庫] 功能表中，按一下 [作業]，然後按一下 [備份作業]。
   ![選取您要監視的作業](./media/backup-file-shares/open-backup-jobs.png)

    備份作業清單和這些作業的狀態隨即出現。
   ![選取您要監視的作業](./media/backup-file-shares/backup-jobs-progress-list.png)

### <a name="create-a-new-policy"></a>建立新的原則

您可以建立新的原則，以從復原服務保存庫的 [備份原則] 備份 Azure 檔案共用。 當您針對檔案共用設定備份時建立的所有原則都會顯示，其 [原則類型] 為 Azure 檔案共用。

若要檢視現有的備份原則：

- 開啟您想要的復原服務保存庫，然後在 [復原服務保存庫] 功能表中，按一下 [備份原則]。 系統會列出所有的備份原則。

   ![選取您要監視的作業](./media/backup-file-shares/list-of-backup-policies.png)

若要建立新的備份原則：

1. 在 [復原服務保存庫] 功能表中，按一下 [備份原則]。
2. 在備份原則清單中，按一下 [新增]。

   ![選取您要監視的作業](./media/backup-file-shares/new-backup-policy.png)

3. 在 [新增] 功能表中，選取 [Azure 檔案共用]。 Azure 檔案共用的 [備份原則] 功能表隨即開啟。 提供原則名稱、備份頻率，以及復原點的保留期範圍。 當您定義好原則時，按一下 [確定]。

   ![選取您要監視的作業](./media/backup-file-shares/create-new-policy.png)

### <a name="stop-protecting-an-azure-file-share"></a>停止保護 Azure 檔案共用

如果您選擇停止保護 Azure 檔案共用，系統會詢問您是否要保留復原點。 有兩種方式可停止保護 Azure 檔案共用︰

- 停止所有未來的備份作業並刪除所有復原點，或
- 停止所有未來的備份作業但保留復原點

將復原點保留在儲存體中可能會產生相關費用，因為系統將會保留 Azure 備份所建立的基礎快照集。 不過，保留復原點的好處是以後可以在需要時還原檔案共用。 如需復原點保留成本的相關資訊，請參閱定價詳細資料。 如果您選擇刪除所有復原點，則無法還原檔案共用。

若要停止保護 Azure 檔案共用：

1. 開啟包含檔案共用復原點的復原服務保存庫，然後按一下 [備份項目]。 [備份項目] 類型清單隨即出現。

   ![按一下 [備份] 讓 Azure 檔案共用與保存庫建立關聯](./media/backup-file-shares/list-of-backup-items.png) 

2. 在 [備份管理類型] 清單中，選取 [Azure 儲存體 (Azure 檔案服務)]。 Azure 儲存體 (Azure 檔案服務) 的 [備份項目] 清單隨即出現。

   ![按一下項目以開啟其他功能表](./media/backup-file-shares/azure-file-share-backup-items.png) 

3. 在 [備份項目 (Azure 儲存體 (Azure 檔案服務))] 的清單中，選取您要停止的備份項目。

4. 在 Azure 檔案共用項目中，按一下 [更多] 功能表，然後選取 [停止備份]。 

   ![按一下項目以開啟其他功能表](./media/backup-file-shares/stop-backup.png)

5. 從 [停止備份] 功能表，選擇 [保留] 或 [刪除備份資料]，然後按一下 [停止備份]。

   ![按一下項目以開啟其他功能表](./media/backup-file-shares/retain-data.png)

### <a name="resume-protection-for-azure-file-share"></a>繼續保護 Azure 檔案共用

如果在停止保護檔案共用時選擇了 [保留備份資料] 選項，則可以繼續保護。 如果已選擇 [刪除備份資料] 選項，則無法繼續保護檔案共用。

若要繼續保護檔案共用，請移至 [備份項目] 並按一下 [繼續備份]。 [備份原則] 隨即開啟，而您可以選擇您所選的原則來繼續備份。

   ![選取您要監視的作業](./media/backup-file-shares/resume-backup-job.png)

### <a name="delete-backup-data"></a>刪除備份資料 

您可以在停止備份作業期間，或在您已停止保護後的任何時間，刪除檔案共用的備份。 先等候幾天或幾週再刪除復原點甚至可能會讓您受益。 不同於還原復原點，當您刪除備份資料時，您無法選擇刪除特定的復原點。 如果您選擇刪除備份資料，則會刪除與項目相關聯的所有復原點。

下列程序假設虛擬機器的備份作業已停止。 一旦備份作業停止，備份項目儀表板便會提供 [繼續備份] 和 [刪除備份資料] 選項。 按一下 [刪除備份資料] 並輸入檔案共用的名稱，以確認刪除。 選擇性地提供刪除 [原因] 或 [註解]。

## <a name="see-also"></a>另請參閱
如需 Azure 檔案共用的詳細資訊，請參閱
- [Azure 檔案共用備份的常見問題](backup-azure-files-faq.md)
- [Azure 檔案共用備份的移難排解](troubleshoot-azure-files.md)
