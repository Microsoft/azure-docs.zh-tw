---
title: 包含檔案
description: 包含檔案
services: backup
author: markgalioto
ms.service: backup
ms.topic: include
ms.date: 5/14/2018
ms.author: markgal
ms.custom: include file
ms.openlocfilehash: 0ff9bf829e2dbe1bca078360ccded94bad63d9a6
ms.sourcegitcommit: 156364c3363f651509a17d1d61cf8480aaf72d1a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39249456"
---
## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫
復原服務保存庫是一個實體，可儲存一段時間以來建立的備份和復原點。 復原服務保存庫也包含與受保護虛擬機器相關聯的備份原則。

若要建立復原服務保存庫：

1. 在 [Azure 入口網站](https://portal.azure.com/)中登入您的訂用帳戶。

2. 在左側功能表上，選取 [所有服務]。

    ![選取所有服務](./media/backup-create-rs-vault/click-all-services.png)

3. 在 [所有服務] 對話方塊中，輸入 **Recovery Services**。 資源清單會根據您的輸入進行篩選。 在資源清單中，選取 [復原服務保存庫]。

    ![輸入並選擇復原服務保存庫](./media/backup-create-rs-vault/all-services.png)

    隨即會在訂用帳戶中出現 [復原服務保存庫] 清單。
    
4. 在 [復原服務保存庫] 儀表板上，選取 [新增]。

    ![新增復原服務保存庫](./media/backup-create-rs-vault/add-button-create-vault.png)

    [復原服務保存庫] 對話方塊隨即開啟。 提供 [名稱]、[訂用帳戶]、[資源群組] 和 [位置] 的值。

    ![設定復原服務保存庫](./media/backup-create-rs-vault/create-new-vault-dialog.png)

   - **名稱**：輸入可識別保存庫的易記名稱。 該名稱必須是 Azure 訂用帳戶中唯一的名稱。 指定的名稱至少要有兩個字元，但不能超過 50 個字元。 名稱開頭必須是字母，且只能包含字母、數字和連字號。
   - **訂用帳戶**：選擇要使用的訂用帳戶。 如果您是唯一一個訂用帳戶的成員，就會看到該名稱。 如果您不確定要使用哪個訂用帳戶，請使用預設 (建議) 的訂用帳戶。 只有在您的公司或學校帳戶與多個 Azure 訂用帳戶相關聯時，才會有多個選擇。
   - **資源群組**：使用現有資源群組，或建立新的群組。 若要查看您訂用帳戶中可用的資源群組清單，請選取 [使用現有的]﹐然後從下拉式清單方塊中選取資源。 若要建立新的資源群組，請選取 [新建]，然後輸入名稱。 如需資源群組的完整資訊，請參閱 [Azure Resource Manager 概觀](../articles/azure-resource-manager/resource-group-overview.md)。
   - **位置**：選取保存庫的地理區域。 若要要建立保存庫來保護虛擬機器，則保存庫**必須**與虛擬機器位於相同區域。

      > [!IMPORTANT]
      > 如果不確定您 VM 的位置，請關閉對話方塊。 移至入口網站中的虛擬機器清單。 如果您在數個區域中有虛擬機器，請在每個區域中建立復原服務保存庫。 請在第一個位置建立保存庫，然後再建立另一個位置的保存庫。 不需要指定用來儲存備份資料的儲存體帳戶。 復原服務保存庫和 Azure 備份服務會自動處理該事宜。
      >
      >

5. 當您準備好建立復原服務保存庫時，選取 [建立]。

    ![建立復原服務保存庫](./media/backup-create-rs-vault/click-create-button.png)

    建立復原服務保存庫可能需要一些時間。 請監視入口網站右上角 [通知] 區域中的狀態通知。 保存庫建立之後，就可以在 [復原服務保存庫] 的清單中看到。 如果您沒有看到保存庫，請選取 [重新整理]。

     ![重新整理備份保存庫的清單](./media/backup-create-rs-vault/refresh-button.png)
