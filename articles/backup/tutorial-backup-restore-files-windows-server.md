---
title: 將檔案從 Azure 復原至 Windows Server
description: 本教學課程會詳細說明從 Azure 復原到 Windows Server 的項目。
services: backup
author: saurabhsensharma
manager: shivamg
keywords: Windows Server 備份; 還原檔案 Windows Server; 備份與災害復原
ms.service: backup
ms.topic: tutorial
ms.date: 2/14/2018
ms.author: saurse
ms.custom: mvc
ms.openlocfilehash: e05c80e52605e051bdd6815608ca8c12e1393727
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34607016"
---
# <a name="recover-files-from-azure-to-a-windows-server"></a>將檔案從 Azure 復原至 Windows Server

Azure Backup 可以復原 Windows Server 備份中的個別項目。 如果必須迅速還原意外刪除的檔案，復原個別檔案很有幫助。 本教學課程涵蓋如何使用 Microsoft Azure 復原服務 (MARS) 代理程式，復原 Azure 已執行備份中的項目。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 起始復原個別的項目 
> * 選取復原點 
> * 從復原點還原項目

本教學課程假設您已執行[將 Windows Server 備份至 Azure](backup-configure-vault.md) 的步驟，在 Azure 中至少有一份 Windows Server 檔案的備份。

## <a name="initiate-recovery-of-individual-items"></a>起始復原個別的項目

名為 Microsoft Azure 備份的使用者介面精靈很有幫助，會與 Microsoft Azure 復原服務 (MARS) 代理程式一起安裝。 Microsoft Azure 備份精靈搭配 Microsoft Azure 復原服務 (MARS) 代理程式，擷取儲存在 Azure 復原點中的備份資料。 使用 Microsoft Azure 備份精靈找出您想要還原到 Windows Server 的檔案或資料夾。 

1. 開啟 **Microsoft Azure 備份**嵌入式管理單元。 您可以透過在您的電腦中搜尋 **Microsoft Azure 備份**來找出備份。

    ![待備份](./media/tutorial-backup-restore-files-windows-server/mars.png)

2. 在精靈中，按一下代理程式主控台 [動作] 窗格的 [復原資料]，啟動 [復原資料精靈]。

    ![待備份](./media/tutorial-backup-restore-files-windows-server/mars-recover-data.png)

3. 在 [入門] 頁面上，選取 [This server (server name)] \(這部伺服器 (伺服器名稱))，然後按一下 [下一步]。

4. 在 [選取復原模式] 頁面上，選取 [個別檔案與資料夾]，然後按一下 [下一步] 開始復原點選取程序。
 
5. 在 [選取磁碟區和日期] 頁面上，選取包含要還原之檔案或資料夾的磁碟區，然後按一下 [掛接]。 選取日期，再從對應到復原點的下拉式功能表中選取時間。 **粗體**的日期表示當日至少有一個復原點可用。

    ![待備份](./media/tutorial-backup-restore-files-windows-server/mars-select-date.png)
 
    當您按一下 [掛接]，Azure 備份就會讓復原點能像磁碟一樣使用。 瀏覽並復原磁碟的檔案。

## <a name="restore-items-from-a-recovery-point"></a>從復原點還原項目

1. 掛接復原磁碟區之後，按一下 [瀏覽] 開啟 Windows 檔案總管，找到您希望復原的檔案和資料夾。 

    ![待備份](./media/tutorial-backup-restore-files-windows-server/mars-browse-recover.png)

    您可以直接從復原磁碟區開啟檔案，予以確認。

2. 在 Windows 檔案總管 中，複製要還原的檔案及/或資料夾，並將它們貼至伺服器的任意位置。

    ![待備份](./media/tutorial-backup-restore-files-windows-server/mars-final.png)

3. 當您完成還原檔案及/或資料夾後，請在 [復原資料精靈] 的 [瀏覽及復原檔案] 頁面上，按一下 [卸載]。 

    ![待備份](./media/tutorial-backup-restore-files-windows-server/unmount-and-confirm.png)

4.  按一下 [是] 確認卸載磁碟區。

    快照集卸載後，代理程式主控台的 [作業] 窗格就會顯示 [作業已完成]。

## <a name="next-steps"></a>後續步驟

如此即完成將 Windows Server 資料備份及還原到 Azure 上的教學課程。 若要深入了解 Azure 備份，請參閱備份加密虛擬機器的 PowerShell 範例。

> [!div class="nextstepaction"]
> [備份加密的 VM](./scripts/backup-powershell-sample-backup-encrypted-vm.md)
