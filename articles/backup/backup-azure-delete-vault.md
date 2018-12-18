---
title: 刪除 Azure 中的復原服務保存庫
description: 本文說明如何刪除復原服務保存庫。 本文包含當您嘗試刪除保存庫卻無法做到時的疑難排解步驟。
services: backup
author: markgalioto
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 7/6/2018
ms.author: markgal
ms.openlocfilehash: 4dc5b006be8599177fb908fe022a3a821b137e12
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39422938"
---
# <a name="delete-a-recovery-services-vault"></a>刪除復原服務保存庫

本文將說明如何從復原服務保存庫中移除所有項目，然後刪除該保存庫。 如果復原服務保存庫已向伺服器註冊並保留備份資料，則無法刪除該保存庫。 如果您嘗試刪除保存庫，但是無法刪除，則保存庫仍會設定為接收備份資料。

若要了解如何刪除保存庫，請參閱[從 Azure 入口網站刪除保存庫](backup-azure-delete-vault.md#delete-a-vault-from-azure-portal)一節。 如果您不要保留復原服務保存庫中的任何資料，並且想刪除保存庫，請參閱[強制刪除保存庫](backup-azure-delete-vault.md#delete-the-recovery-services-vault-by-force)一節。 如果您不確定保存庫中有什麼，但必須確認您可以刪除保存庫，請參閱[移除保存庫相依性並刪除保存庫](backup-azure-delete-vault.md#remove-vault-dependencies-and-delete-vault)。

## <a name="delete-a-vault-from-azure-portal"></a>從 Azure 入口網站刪除保存庫

如果您已開啟復原服務保存庫，請跳至第 2 步驟。

1. 開啟 Azure 入口網站，從儀表板中開啟您要刪除的保存庫。

   如果儀表板中尚未釘選復原服務保存庫，請按一下 [中樞] 功能表上的 [所有服務]，然後在資源清單中輸入**復原服務**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 若要檢視訂用帳戶中的保存庫清單，按一下 [復原服務保存庫]。

   ![建立復原服務保存庫的步驟 1](./media/backup-azure-delete-vault/open-recovery-services-vault.png) <br/>

   隨即會顯示 [復原服務保存庫] 清單。 

   ![從清單中選擇保存庫](./media/backup-azure-delete-vault/choose-vault-to-delete-.png)

1. 從清單中，選取您要刪除的保存庫。 當您選取保存庫時，其保存庫儀表板隨即開啟。

    ![選取您的保存庫以開啟其儀表板](./media/backup-azure-delete-vault/contoso-bkpvault-settings.png)

1. 若要刪除保存庫，在保存庫儀表板中按一下 [刪除]。 系統會向您確認是否要刪除保存庫。

    ![選取您的保存庫以開啟其儀表板](./media/backup-azure-delete-vault/click-delete-button-to-delete-vault.png)

    如果出現**保存庫刪除錯誤**，您可以從保存庫中移除相依性，或者可以使用 PowerShell 強制刪除保存庫。 下列各節將說明如何完成這些工作。

    ![保存庫刪除錯誤](./media/backup-azure-delete-vault/vault-delete-error.png)


## <a name="delete-the-recovery-services-vault-by-force"></a>強制刪除復原服務保存庫

您可以使用 PowerShell 來強制刪除復原服務保存庫。 強制刪除表示會永久刪除復原服務保存庫和所有相關聯的備份資料。 

> [!Warning]
> 使用 PowerShell 來刪除復原服務保存庫時，請先確定您要永久刪除保存庫中的所有備份資料。
>

刪除復原服務保存庫：

1. 登入您的 Azure 帳戶。

   使用 `Connect-AzureRmAccount` 命令登入 Azure 訂用帳戶並遵循畫面上的指示。

   ```powershell
    Connect-AzureRmAccount
   ```
   第一次使用 Azure 備份時，您必須使用 [Register-AzureRmResourceProvider](/powershell/module/AzureRM.Resources/Register-AzureRmResourceProvider) 在您的訂用帳戶中註冊 Azure 復原服務提供者。

   ```powershell
    Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
   ```

1. 以系統管理權限開啟 PowerShell 視窗。

1. 使用 `Set-ExecutionPolicy Unrestricted` 移除任何限制。

1. 執行下列命令，從 chocolately.org 下載 Azure Resource Manager 用戶端套件。

    `iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`

1. 使用下列命令來安裝 Azure Resource Manager API 用戶端。

   `choco.exe install armclient`

1. 在 Azure 入口網站中，針對您要刪除的復原服務保存庫，收集其訂用帳戶識別碼和相關聯的資源群組名稱。

1. 在 PowerShell 中，使用您的訂用帳戶識別碼、資源群組名稱和復原服務保存庫名稱來執行下列命令。 當您執行命令時，此命令會刪除保存庫和所有相依性。

   ```powershell
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<recovery services vault name>?api-version=2015-03-15
   ```
1. 在 Azure 入口網站中登入您的訂用帳戶，並確認保存庫已刪除。


## <a name="remove-vault-dependencies-and-delete-vault"></a>移除保存庫相依性並刪除保存庫

若要手動移除保存庫相依性，請刪除每個項目或伺服器與復原服務保存庫之間的組態。 當您進行下列程序時，請使用 [備份項目] 功能表 (請參閱圖片) 來找出下列項目：

* Azure 儲存體 (Azure 檔案服務) 的備份
* Azure VM 中 SQL Server 的備份
* Azure 虛擬機器的備份
* Microsoft Azure 復原服務代理程式服務的備份

使用 [備份基礎結構]功能表 (請參閱影像) 找出下列項目：

* Azure 備份伺服器的備份
* System Center DPM 的備份

    ![選取您的保存庫以開啟其儀表板](./media/backup-azure-delete-vault/backup-items-backup-infrastructure.png)

1. 在保存庫儀表板功能表中，向下捲動到 [受保護項目] 區段，然後按一下 [備份項目]。 在此功能表中，您可以停止和刪除 Azure 檔案伺服器、Azure VM 中的 SQL Server 和 Azure 虛擬機器。 在此範例中，我們將會移除 Azure 檔案伺服器中的備份資料。

    ![選取您的保存庫以開啟其儀表板](./media/backup-azure-delete-vault/selected-backup-items.png)

1. 選取備份類型，即可檢視該類型的所有項目。

    ![選取備份類型](./media/backup-azure-delete-vault/azure-storage-selected-list.png)

1. 針對清單中的所有項目，以滑鼠右鍵按一下項目，然後從捷徑功能表中選取 [停止備份]。

    ![選取備份類型](./media/backup-azure-delete-vault/stop-backup-item.png) 

    [停止備份] 功能表隨即開啟。

1. 在 [停止備份] 功能表中，從 [選擇選項] 功能表中選取 [刪除備份資料]，輸入項目的名稱，然後按一下 [停止備份]。

    輸入項目的名稱，以確認您想要刪除它。 一旦您確認項目之後，[停止備份] 按鈕會生效。 如果您有保留資料，則會無法刪除保存庫。

    ![刪除備份資料](./media/backup-azure-delete-vault/stop-backup-blade-delete-backup-data.png)

    您想要的話也可以提供為什麼要刪除資料的原因，並加入註解。 若要確認作業已完成，請檢查 Azure 訊息 ![刪除備份資料](./media/backup-azure-delete-vault/messages.png)。 <br/>
    作業完成之後，服務會傳送一則訊息：*備份程序已停止，並已刪除備份資料*。

1. 刪除清單中的項目後，在 [備份項目] 功能表上按一下 [重新整理]，以查看保存庫中的項目。

      ![刪除備份資料](./media/backup-azure-delete-vault/empty-items-list.png)

      當清單中沒有任何項目時，捲動到復原服務保存庫功能表中的 [基本資訊] 窗格。 清單中不應該有任何 [備份項目]、[備份管理伺服器] 或 [複寫的項目]。 如果保存庫中仍有項目，請返回步驟三，並選擇其他項目類型清單。  

1. 當保存庫工具列中沒有其他項目，按一下 [刪除] 。

    ![刪除備份資料](./media/backup-azure-delete-vault/vault-ready-to-delete.png)

1. 若要確認您想要刪除保存庫，請按一下 [是]。

    保存庫便會被刪除，入口網站會回到 [新增]  服務功能表。

## <a name="removing-azure-backup-server-or-dpm"></a>移除 Azure 備份伺服器或 DPM

1. 在保存庫儀表板功能表中，向下捲動到 [管理] 區段，然後按一下 [備份基礎結構]。 

1. 在子功能表中按一下 [備份管理伺服器]，即可檢視 Azure 備份伺服器和 System Center DPM 伺服器。 您可以停止和刪除 Azure 檔案伺服器、Azure VM 中的 SQL Server 和 Azure 虛擬機器。 

    ![選取您的保存庫以開啟其儀表板](./media/backup-azure-delete-vault/delete-backup-management-servers.png)

1. 以滑鼠右鍵按一下您要刪除的項目，然後從子功能表中選取 [刪除]。

    ![選取備份類型](./media/backup-azure-delete-vault/azure-storage-selected-list.png)

    [停止備份] 功能表隨即開啟。

1. 在 [停止備份] 功能表中，從 [選擇選項] 功能表中選取 [刪除備份資料]，輸入項目的名稱，然後按一下 [停止備份]。

    請輸入項目名稱以確認要將其刪除。 一旦您確認項目之後，[停止備份] 按鈕會生效。 如果您有保留資料，則會無法刪除保存庫。

    ![刪除備份資料](./media/backup-azure-delete-vault/stop-backup-blade-delete-backup-data.png)

    或者，您可以提供您為什麼要刪除資料的原因，並加入註解。 若要確認作業已完成，請檢查 Azure 訊息 ![delete backup data](./media/backup-azure-delete-vault/messages.png)。 <br/>
    作業完成之後，服務會傳送一則訊息：備份程序已停止，並已刪除備份資料。

1. 刪除清單中的項目後，在 [備份項目] 功能表上按一下 [重新整理]，以查看保存庫中的其餘項目。

      ![刪除備份資料](./media/backup-azure-delete-vault/empty-items-list.png)

      當清單中沒有任何項目時，捲動到復原服務保存庫功能表中的 [基本資訊] 窗格。 清單中不應該有任何 [備份項目]、[備份管理伺服器] 或 [複寫的項目]。 如果保存庫中仍有項目，請返回步驟三，並選擇其他項目類型清單。  
1. 當保存庫中沒有其他項目時，在保存庫儀表板中按一下 [刪除] 。

    ![刪除備份資料](./media/backup-azure-delete-vault/vault-ready-to-delete.png)

1. 若要確認您想要刪除保存庫，請按一下 [是]。

    保存庫便會被刪除，入口網站會回到 [新增]  服務功能表。


## <a name="removing-azure-backup-agent-recovery-points"></a>移除 Azure 備份代理程式復原點

1. 在保存庫儀表板功能表中，向下捲動到 [管理] 區段，然後按一下 [備份基礎結構]。

1. 在子功能表中按一下 [受保護的伺服器]，即可檢視受保護的伺服器類型清單，包括 Azure 備份代理程式。

    ![選取您的保存庫以開啟其儀表板](./media/backup-azure-delete-vault/identify-protected-servers.png)

1. 在 [受保護的伺服器] 清單中，按一下 [Azure 備份代理程式]。

    ![選取備份類型](./media/backup-azure-delete-vault/list-of-protected-server-types.png)

    使用 Azure 備份代理程式保護的伺服器清單隨即開啟。

    ![選取特定受保護的伺服器](./media/backup-azure-delete-vault/azure-backup-agent-protected-servers.png)

1. 在伺服器清單中，按一下任一伺服器以開啟其功能表。

    ![檢視所選伺服器的儀表板](./media/backup-azure-delete-vault/selected-protected-server.png)

1. 在選取的伺服器儀表板功能表上，按一下 [刪除]。

    ![刪除選取的伺服器](./media/backup-azure-delete-vault/selected-protected-server-click-delete.png)

1. 在 [刪除] 功能表中，輸入項目的名稱，然後按一下 [刪除]。

    輸入項目的名稱，以確認您想要刪除它。 一旦您確認項目之後，[刪除] 按鈕會生效。

    ![刪除備份資料](./media/backup-azure-delete-vault/delete-protected-server-dialog.png)

    或者，您可以提供您為什麼要刪除資料的原因，並加入註解。 若要確認作業已完成，請檢查 Azure 訊息 ![delete backup data](./media/backup-azure-delete-vault/messages.png)。 <br/>
    作業完成之後，服務會傳送一則訊息：備份程序已停止，並已刪除備份資料。

1. 刪除清單中的項目後，在 [備份項目] 功能表上按一下 [重新整理]，以查看保存庫中的其餘項目。

      ![刪除備份資料](./media/backup-azure-delete-vault/empty-items-list.png)

      當清單中沒有任何項目時，捲動到復原服務保存庫功能表中的 [基本資訊] 窗格。 清單中不應該有任何 [備份項目]、[備份管理伺服器] 或 [複寫的項目]。 如果保存庫中仍有項目，請返回步驟三，並選擇其他項目類型清單。  
1. 當保存庫中沒有其他項目時，在保存庫儀表板中按一下 [刪除] 。

    ![刪除備份資料](./media/backup-azure-delete-vault/vault-ready-to-delete.png)

1. 若要確認您想要刪除保存庫，請按一下 [是]。

    保存庫便會被刪除，入口網站會回到 [新增]  服務功能表。

## <a name="what-if-i-stop-the-backup-process-but-retain-the-data"></a>如果我停止備份程序但卻保留了資料，會怎麼樣？

如果您停止備份程序，但是不小心「保留」  資料，您必須先刪除備份資料，才能刪除保存庫。 刪除備份資料：

1. 在 [備份項目] 功能表中，以滑鼠右鍵按一下項目，然後在內容功能表中按一下 [刪除備份資料]。

    ![刪除備份資料](./media/backup-azure-delete-vault/delete-backup-data-menu.png)

    [刪除備份資料] 功能表隨即開啟。
1. 在 [刪除備份資料] 功能表中，輸入項目的名稱，然後按一下 [刪除]。

    ![刪除備份資料](./media/backup-azure-delete-vault/delete-retained-vault.png)

    刪除資料之後，請返回步驟 4c，並繼續進行程序。
