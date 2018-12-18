---
title: 管理 Resource Manager 部署的虛擬機器備份
description: 了解如何管理和監視 Resource Manager 部署的虛擬機器備份
services: backup
author: trinadhk
manager: shreeshd
ms.service: backup
ms.topic: conceptual
ms.date: 11/28/2016
ms.author: trinadhk
ms.openlocfilehash: 4d45db6ba6354f85c3ed67561751720b6f6f4b77
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38465715"
---
# <a name="manage-azure-virtual-machine-backups"></a>管理 Azure 虛擬機器備份

本文提供如何管理 VM 備份的指引，並說明入口網站儀表板中提供的備份警示資訊。 本文中的指引適用於 VM 與復原服務保存庫搭配使用。 本文並未涵蓋建立虛擬機器，也不說明如何保護虛擬機器。 有關在 Azure 中以復原服務保存庫來保護 Azure Resource Manager 部署的 VM 的初步解說，請參閱 [搶先目睹︰將 VM 備份至復原服務保存庫](backup-azure-vms-first-look-arm.md)。

## <a name="manage-vaults-and-protected-virtual-machines"></a>管理保存庫與受保護的虛擬機器
在 Azure 入口網站中，復原服務保存庫儀表板可讓您存取保存庫的相關資訊，包括︰

* 最近的備份快照，也就是最新的還原點
* 備份原則
* 所有備份快照的大小總計
* 使用保存庫保護的虛擬機器數目

虛擬機器備份的許多管理工作都是從儀表板中開啟保存庫開始。 不過，因為保存庫可以用來保護多個項目 (或多個 VM)，若要檢視特定 VM 的詳細資料，請開啟保存庫項目儀表板。 下列程序示範如何開啟「保存庫儀表板」，然後繼續開啟「保存庫項目儀表板」。 兩個程序都有「提示」，指出如何使用「釘選到儀表板」命令，將保存庫和保存庫項目新增至 Azure 儀表板。 「釘選到儀表板」是建立保存庫捷徑或項目捷徑的方法。 您也可以從捷徑執行常用的命令。

> [!TIP]
> 如果您開啟了多個儀表板和刀鋒視窗，請使用視窗底部的深藍色滑桿，來左右滑動 Azure 儀表板。
>
>

![使用滑桿以完整檢視](./media/backup-azure-manage-vms/bottom-slider.png)

### <a name="open-a-recovery-services-vault-in-the-dashboard"></a>在儀表板中開啟復原服務保存庫︰
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在 [中樞] 功能表上按一下 [瀏覽]，然後在資源清單中輸入**復原服務**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 按一下 [復原服務保存庫] 。

    ![建立復原服務保存庫的步驟 1](./media/backup-azure-manage-vms/browse-to-rs-vaults.png)

    隨即會顯示 [復原服務保存庫] 清單。

    ![復原服務保存庫清單 ](./media/backup-azure-manage-vms/list-o-vaults.png)

   > [!TIP]
   > 如果您將保存庫釘選到 Azure 儀表板，則一開啟 Azure 入口網站就可立即存取該保存庫。 若要將保存庫釘選到儀表板，請在保存庫清單中以滑鼠右鍵按一下保存庫，然後選取 [釘選到儀表板] 。
   >
   >
3. 從保存庫清單中，選取保存庫以開啟其儀表板。 選取保存庫時，保存庫儀表板和 [設定] 刀鋒視窗將會開啟。 下圖中反白顯示 **Contoso-vault** 儀表板。

    ![開啟保存庫儀表板和設定刀鋒視窗](./media/backup-azure-manage-vms/full-view-rs-vault.png)

### <a name="open-a-vault-item-dashboard"></a>開啟保存庫項目儀表板
上一個程序中，您開啟保存庫儀表板。 若要開啟保存庫項目儀表板︰

1. 在保存庫儀表板的 [備份項目] 圖格上，按一下 [Azure 虛擬機器]。

    ![開啟備份項目備份圖格](./media/backup-azure-manage-vms/contoso-vault-1606.png)

    [備份項目]  刀鋒視窗會列出每個項目的最後一個備份作業。 在此範例中，有一個受此保存庫保護的虛擬機器：demovm markgal。  

    ![備份項目圖格](./media/backup-azure-manage-vms/backup-items-blade.png)

   > [!TIP]
   > 為了方便存取，您可以將保存庫項目釘選到 Azure 儀表板。 若要釘選保存庫項目，請以滑鼠右鍵按一下項目，然後選取 [釘選到儀表板] 。
   >
   >
2. 在 [備份項目]  刀鋒視窗中，按一下項目以開啟保存庫項目儀表板。

    ![備份項目圖格](./media/backup-azure-manage-vms/backup-items-blade-select-item.png)

    保存庫項目儀表板及其 [設定]  刀鋒視窗隨即開啟。

    ![搭配設定刀鋒視窗的備份項目儀表板](./media/backup-azure-manage-vms/item-dashboard-settings.png)

    從保存庫項目儀表板，您可以完成許多重要的管理工作，例如︰

   * 變更原則或建立新的備份原則
   * 檢視還原點並查看其一致性狀態
   * 虛擬機器的隨選備份
   * 停止保護虛擬機器
   * 繼續保護虛擬機器
   * 刪除備份資料 (或復原點)
   * [還原備份磁碟](backup-azure-arm-restore-vms.md#restore-backed-up-disks)

在以下的程序中，起始點是保存庫項目儀表板。

## <a name="manage-backup-policies"></a>管理備份原則
1. 在[保存庫項目儀表板](backup-azure-manage-vms.md#open-a-vault-item-dashboard)中，按一下 [所有設定] 以開啟 [設定] 刀鋒視窗。

    ![備份原則刀鋒視窗](./media/backup-azure-manage-vms/all-settings-button.png)
2. 在 [設定] 刀鋒視窗上，按一下 [備份原則] 以開啟該刀鋒視窗。

    刀鋒視窗上會顯示備份頻率和保留範圍詳細資料。

    ![備份原則刀鋒視窗](./media/backup-azure-manage-vms/backup-policy-blade.png)
3. 從 [選擇備份原則]  功能表︰

   * 若要變更原則，請選取不同的原則，然後按一下 [儲存] 。 新原則時會立即套用至保存庫。
   * 若要建立原則，請選取 [建立新項目] 。

     ![虛擬機器備份](./media/backup-azure-manage-vms/backup-policy-create-new.png)

     如需建立備份原則的指示，請參閱 [定義備份原則](backup-azure-manage-vms.md#defining-a-backup-policy)。

[!INCLUDE [backup-create-backup-policy-for-vm](../../includes/backup-create-backup-policy-for-vm.md)]

> [!NOTE]
> 在管理備份原則時，請務必遵循[最佳做法](backup-azure-vms-introduction.md#best-practices)以便獲得最佳備份效能
>
>

## <a name="on-demand-backup-of-a-virtual-machine"></a>虛擬機器的隨選備份
設定保護後，您可以執行虛擬機器的隨選備份。 如果初始備份已暫止，則隨選備份會在復原服務保存庫中建立虛擬機器的完整複本。 如果已完成初始備份，隨選備份只會將前一個備份快照的變更傳送到復原服務保存庫。 亦即，後續備份一律是增量備份。

> [!NOTE]
> 隨選備份的保留範圍是原則中為「每日」備份點所指定的保留值。 如果未選取任何「每日」備份點，則會使用每週備份點。
>
>

若要觸發虛擬機器的隨選備份：

* 在[保存庫項目儀表板](backup-azure-manage-vms.md#open-a-vault-item-dashboard)上，按一下 [立即備份]。

    ![立即備份按鈕](./media/backup-azure-manage-vms/backup-now-button.png)

    入口網站會確認您想要啟動隨選備份作業。 按一下 [是]  啟動備份作業。

    ![立即備份按鈕](./media/backup-azure-manage-vms/backup-now-check.png)

    備份作業會建立復原點。 復原點的保留範圍與在虛擬機器相關的原則中指定的保留範圍相同。 若要追蹤作業的進度，請在保存庫儀表板中按一下 [備份作業]  圖格。  

## <a name="stop-protecting-virtual-machines"></a>停止保護虛擬機器
如果您選擇停止保護虛擬機器，系統會詢問您是否要保留復原點。 有兩種方式可停止保護虛擬機器︰

* 停止所有未來的備份作業並刪除所有復原點，或
* 停止所有未來的備份作業但保留復原點 

在儲存體中保留復原點需要付出相關的成本。 不過，保留復原點的好處是以後可以在需要時還原虛擬機器。 如需復原點保留成本的相關資訊，請參閱[價格詳細資料](https://azure.microsoft.com/pricing/details/backup/)。 如果您選擇刪除所有復原點，則無法還原虛擬機器。

若要停止保護虛擬機器：

1. 在[保存庫項目儀表板](backup-azure-manage-vms.md#open-a-vault-item-dashboard)上，按一下 [停止備份]。

    ![停止備份按鈕](./media/backup-azure-manage-vms/stop-backup-button.png)

    [停止備份] 刀鋒視窗隨即開啟。

    ![停止備份刀鋒視窗](./media/backup-azure-manage-vms/stop-backup-blade.png)
2. 在 [停止備份]  刀鋒視窗上，選擇要保留還是刪除備份資料。 資訊方塊會針對您的選擇提供詳細資料。

    ![停止保護](./media/backup-azure-manage-vms/retain-or-delete-option.png)
3. 如果您選擇要保留備份資料，請跳至步驟 4。 如果您選擇要刪除備份資料，請確認您想要停止備份作業，並刪除復原點 - 輸入項目的名稱。

    ![停止驗證](./media/backup-azure-manage-vms/item-verification-box.png)

    如果您不確定項目名稱，請以滑鼠暫留在驚嘆號來檢視名稱。 項目的名稱也會出現在刀鋒視窗頂端的 [停止備份]  下。
4. 選擇性地提供 [原因] 或 [註解]。
5. 若要停止目前項目的備份作業，請按一下![停止備份按鈕](./media/backup-azure-manage-vms/stop-backup-button-blue.png)

    通知訊息可讓您知道備份作業已停止。

    ![確認停止保護](./media/backup-azure-manage-vms/stop-message.png)

## <a name="resume-protection-of-a-virtual-machine"></a>繼續保護虛擬機器
如果在停止保護虛擬機器時已選擇 [保留備份資料]  選項，則可以繼續保護。 如果已選擇 [刪除備份資料]  選項，則無法繼續保護虛擬機器。

繼續保護虛擬機器

1. 在[保存庫項目儀表板](backup-azure-manage-vms.md#open-a-vault-item-dashboard)上，按一下 [繼續備份]。

    ![繼續保護](./media/backup-azure-manage-vms/resume-backup-button.png)

    [備份原則] 刀鋒視窗隨即開啟。

   > [!NOTE]
   > 重新保護虛擬機器時，您可以選擇與最初用於保護虛擬機器不同的原則。
   >
   >
2. 請依照[管理備份原則](backup-azure-manage-vms.md#manage-backup-policies)中的步驟，指派虛擬機器的原則。

    備份原則套用至虛擬機器後，您會看到下列訊息。

    ![已成功保護 VM](./media/backup-azure-manage-vms/success-message.png)

## <a name="delete-backup-data"></a>刪除備份資料
您可以在 **停止備份** 作業期間，或在備份作業完成之後的任何時間，刪除與虛擬機器相關聯的備份資料。 先等候幾天或幾週再刪除復原點甚至可能會讓您受益。 不同於還原復原點，當您刪除備份資料時，您無法選擇刪除特定的復原點。 如果您選擇刪除備份資料，則會刪除與項目相關聯的所有復原點。

下列程序假設虛擬機器的備份作業已停止或停用。 一旦備份作業停用，保存庫項目儀表板便會提供 [繼續備份] 和 [刪除備份] 選項。

![繼續和刪除按鈕](./media/backup-azure-manage-vms/resume-delete-buttons.png)

若要在「備份已停用」 的情況下刪除虛擬機器上的備份資料：

1. 在[保存庫項目儀表板](backup-azure-manage-vms.md#open-a-vault-item-dashboard)上，按一下 [刪除備份]。

    ![VM 類型](./media/backup-azure-manage-vms/delete-backup-buttom.png)

    [刪除備份資料]  刀鋒視窗隨即開啟。

    ![VM 類型](./media/backup-azure-manage-vms/delete-backup-blade.png)
2. 輸入項目的名稱，以確認您想要刪除復原點。

    ![停止驗證](./media/backup-azure-manage-vms/item-verification-box.png)

    如果您不確定項目名稱，請以滑鼠暫留在驚嘆號來檢視名稱。 項目的名稱也出現刀鋒視窗頂端的 [刪除備份資料]  下。
3. 選擇性地提供 [原因] 或 [註解]。
4. 若要刪除目前項目的備份資料，請按一下![停止備份按鈕](./media/backup-azure-manage-vms/delete-button.png)

    通知訊息可讓您知道備份資料已刪除。

## <a name="next-steps"></a>後續步驟
如需從復原點重新建立虛擬機器的詳細資訊，請參閱 [還原 Azure VM](backup-azure-arm-restore-vms.md)。 如需保護虛擬機器的詳細資訊，請參閱 [搶先目睹︰將 VM 備份至復原服務保存庫](backup-azure-vms-first-look-arm.md)。 如需監視事件的相關資訊，請參閱 [監視 Azure 虛擬機器備份的警示](backup-azure-monitor-vms.md)。
