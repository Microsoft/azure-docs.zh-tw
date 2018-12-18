---
title: 將 Azure 中的資料還原至 Windows Server 或 Windows 電腦
description: 了解如何將儲存於 Azure 中的資料還原至 Windows Server 或 Windows 電腦。
services: backup
author: saurabhsensharma
manager: shivamg
ms.service: backup
ms.topic: conceptual
ms.date: 9/7/2018
ms.author: saurse
ms.openlocfilehash: 20d2f289f4d40d773fde9f6b770dc49b87c34804
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297242"
---
# <a name="restore-files-to-a-windows-server-or-windows-client-machine-using-resource-manager-deployment-model"></a>使用 Resource Manager 部署模型將檔案還原到 Windows Server 或 Windows 用戶端電腦

此文章說明如何從備份保存庫還原資料。 若要還原資料，您可以使用 Microsoft Azure 復原服務 (MARS) 代理程式中的 [復原資料精靈]。 當您還原資料時，您可以︰

* 將資料還原到進行備份的相同電腦。
* 將資料還原至其他電腦。

2017 年 1 月，Microsoft 推出 MARS 代理程式預覽更新。 這項更新除了錯誤修正之外，還啟用了「立即還原」，允許您掛接可寫入的復原點快照來做為復原磁碟區。 您接著可以探索復磁碟區並將檔案複製至本機電腦，藉此選擇性地還原檔案。

> [!NOTE]
> 如果您要使用「立即還原」來還原資料，必須要有 [2017 年 1 月 Azure 備份更新](https://support.microsoft.com/en-us/help/3216528?preview)。 另外，您也必須在支援文章列出之地區設定的保存庫中，保護備份資料。 請參閱 [2017 年 1 月 Azure 備份更新](https://support.microsoft.com/en-us/help/3216528?preview)，了解支援「立即還原」的最新地區設定清單。 「立即還原」目前**無法**在所有地區設定中使用。
>

在 Azure 入口網站中搭配復原服務保存庫使用立即還原。 如果您已在備份保存庫中儲存資料，則已將它們轉換至復原服務保存庫。 如果您想要使用「立即還原」，請下載 MARS 更新，並依照提及「立即還原」部分中的程序進行操作。

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="use-instant-restore-to-recover-data-to-the-same-machine"></a>使用「立即還原」將資料還原至同一台電腦

如果您不小心刪除檔案，而您想要將它還原到相同的電腦 (備份進行處)，下列步驟可協助您復原資料。

1. 開啟 **Microsoft Azure 備份** 嵌入式管理單元。 如果您不知道快照安裝的位置，請在電腦或伺服器上搜尋 **Microsoft Azure 備份**。

    傳統型應用程式應該會出現在搜尋結果中。

2. 按一下 [復原資料] 以啟動精靈。

    ![復原資料](./media/backup-azure-restore-windows-server/recover.png)

3. 在 [開始使用] 窗格中，若要將資料還原至同一台伺服器或電腦，請選取 [這台伺服器 (`<server name>`)] 並按一下 [下一步]。

    ![選擇 [這台伺服器] 選項以將資料還原到同一台電腦](./media/backup-azure-restore-windows-server/samemachine_gettingstarted_instantrestore.png)

4. 在 [選取復原模式] 窗格上，選擇 [個別檔案與資料夾]，然後按一下 [下一步]。

    ![瀏覽檔案](./media/backup-azure-restore-windows-server/samemachine_selectrecoverymode_instantrestore.png)
> [!IMPORTANT]
> 若要執行還原*個別檔案與資料夾*的選項，必須具備 .NET Framework 4.5.2 或更高版本。 如果您未看見 [個別檔案與資料夾] 選項，您必須先將 .NET Framework 升級至 4.5.2 版到或更高版本，再重新嘗試。

> [!TIP]
> [個別檔案及資料夾] 選項可讓您快速存取復原點資料。 這很適合用來復原個別檔案 (大小總計不得超過 80 GB)，並在復原期間提供最多 6 MBps 的傳輸/複製速度。 [磁碟區] 選項會復原指定磁碟區中的所有備份資料。 此選項可提供更快的傳輸速度 (最多 60 MBps)，適合用來復原大型資料或整個磁碟區。

5. 在 [選取磁碟區和日期] 窗格上，選取包含您要還原之檔案和/或資料夾的磁碟區。

    在日曆上，選取復原點。 您可以從任何時間的復原點還原。 **粗體**的日期表示至少有一個復原點可用。 一旦您選取一個日期，如果有多個復原點可用，您可以從 [時間] 下拉式功能表選擇特定的復原點。

    ![磁碟區和日期](./media/backup-azure-restore-windows-server/samemachine_selectvolumedate_instantrestore.png)

6. 選擇要還原的復原點之後，按一下 [掛接]。

    Azure 備份會掛接本機復原點，並且使用它做為復原磁碟區。

7. 在 [瀏覽及復原檔案] 窗格上，按一下 [瀏覽] 以開啟 [Windows 檔案總管]，然後尋找所需的檔案和資料夾。

    ![修復選項](./media/backup-azure-restore-windows-server/samemachine_browserecover_instantrestore.png)


8. 在 [Windows 檔案總管] 中，複製要還原的檔案和/或資料夾，並將它們貼上至伺服器或電腦的任意本機位置。 您可以直接從復原磁碟區開啟或串流檔案，並確認您是否復原正確的版本。

    ![複製掛接磁碟區的檔案和資料夾，並貼上至本機位置](./media/backup-azure-restore-windows-server/samemachine_copy_instantrestore.png)


9. 當您完成還原檔案和/或資料夾後，請在 [瀏覽及復原檔案] 窗格上，按一下 [卸載]。 按一下 [是] 以確認要卸載磁碟區。

    ![卸載磁碟區並確認](./media/backup-azure-restore-windows-server/samemachine_unmount_instantrestore.png)

    > [!Important]
    > 如果您並未按一下 [卸載]，復原磁碟區會保持掛接 6 個小時 (從掛接後開始計算)。 不過，如果是進行中的檔案複製，掛接時間可擴充至高達 24 小時。 當磁碟區處於掛接狀態時，不會執行任何備份作業。 當掛接磁碟區時，任何排定要執行的備份作業會在復原磁碟區卸載之後才執行。
    >


## <a name="use-instant-restore-to-restore-data-to-an-alternate-machine"></a>使用「立即還原」將資料還原至其他電腦
若您遺失整個伺服器，您仍然可從 Azure 備份將資料還原到其他電腦。 下列步驟說明工作流程。


這些步驟中所使用的術語包含：

*  – 用來進行備份且目前無法使用的的原始電腦。
*  – 復原資料時的目標電腦。
* 「範例保存庫」–「來源電腦」和「目標電腦」註冊的復原服務保存庫。 <br/>

> [!NOTE]
> 備份無法還原到執行舊版作業系統的目標電腦。 例如，從 Windows 7 電腦建立的備份可以還原至 Windows 8 或更新版本的電腦。 從 Windows 8 電腦建立的備份無法還原至 Windows 7 電腦。
>
>

1. 在「目標電腦」上開啟 [Microsoft Azure 備份] 嵌入式管理單元。

2. 確定 [目標電腦] 和 [來源電腦] 已註冊到同一個復原服務保存庫。

3. 按一下 [復原資料] 以開啟 [復原資料精靈]。

    ![復原資料](./media/backup-azure-restore-windows-server/recover.png)

4. 在 [開始使用] 窗格上，選取 [其他伺服器]

    ![其他伺服器](./media/backup-azure-restore-windows-server/alternatemachine_gettingstarted_instantrestore.png)

5. 提供與「範例保存庫」相對應的保存庫認證檔，然後按一下 [下一步]。

    如果保存庫認證檔無效 (或已過期)，請從 Azure 入口網站中的「範例保存庫」下載新的保存庫認證檔。 一旦您提供有效的保存庫認證檔之後，就會顯示對應的備份保存庫名稱。


6. 在 [選取備份伺服器] 窗格上，從顯示的電腦清單選取 [來源電腦]，並提供複雜密碼。 然後按 [下一步] 。

    ![電腦清單](./media/backup-azure-restore-windows-server/alternatemachine_selectmachine_instantrestore.png)

7. 在 [選取復原模式] 窗格上，選取 [個別檔案和資料夾] 並按一下 [下一步]。

    ![Search](./media/backup-azure-restore-windows-server/alternatemachine_selectrecoverymode_instantrestore.png)

8. 在 [選取磁碟區和日期] 窗格上，選取包含您要還原之檔案和/或資料夾的磁碟區。

    在日曆上，選取復原點。 您可以從任何時間的復原點還原。 **粗體**的日期表示至少有一個復原點可用。 一旦您選取一個日期，如果有多個復原點可用，您可以從 [時間] 下拉式功能表選擇特定的復原點。

    ![搜尋項目](./media/backup-azure-restore-windows-server/alternatemachine_selectvolumedate_instantrestore.png)

9. 按一下 [掛接] 以掛接本機的復原點，做為 [目標電腦] 的復原磁碟區。

10. 在 [瀏覽及復原檔案] 窗格上，按一下 [瀏覽] 以開啟 [Windows 檔案總管]，然後尋找所需的檔案和資料夾。

    ![加密](./media/backup-azure-restore-windows-server/alternatemachine_browserecover_instantrestore.png)

11. 在 [Windows 檔案總管] 中，從復原磁碟區複製檔案和/或資料夾，然後貼上至 [目標電腦] 位置。 您可以直接從復原磁碟區開啟或串流檔案，並確認是否復原正確的版本。

    ![加密](./media/backup-azure-restore-windows-server/alternatemachine_copy_instantrestore.png)

12. 當您完成還原檔案和/或資料夾後，請在 [瀏覽及復原檔案] 窗格上，按一下 [卸載]。 按一下 [是] 以確認要卸載磁碟區。

    ![加密](./media/backup-azure-restore-windows-server/alternatemachine_unmount_instantrestore.png)

    > [!Important]
    > 如果您並未按一下 [卸載]，復原磁碟區會保持掛接 6 個小時 (從掛接後開始計算)。 不過，如果是進行中的檔案複製，掛接時間可擴充至高達 24 小時。 當磁碟區處於掛接狀態時，不會執行任何備份作業。 當掛接磁碟區時，任何排定要執行的備份作業會在復原磁碟區卸載之後才執行。
    >

## <a name="next-steps"></a>後續步驟
* 現在您已復原檔案和資料夾，接下來您可以 [管理您的備份](backup-azure-manage-windows-server.md)。
