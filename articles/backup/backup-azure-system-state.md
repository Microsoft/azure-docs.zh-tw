---
title: 將 Windows 系統狀態備份至 Azure
description: 了解如何將 Windows Server 和/或 Windows 電腦的系統狀態備份至 Azure。
services: backup
author: saurabhsensharma
manager: shivamg
keywords: 如何備份; 如何備份; 備份檔案和資料夾
ms.service: backup
ms.topic: conceptual
ms.date: 05/23/2018
ms.author: saurse
ms.openlocfilehash: 61ee1ce7d5cc6dc2aa4b7a8b02c2e5ba77539725
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34606064"
---
# <a name="back-up-windows-system-state-in-resource-manager-deployment"></a>在 Resource Manager 部署中備份 Windows 系統狀態
本文說明如何將 Windows Server 的系統狀態備份至 Azure。 本文中的教學課程是要讓您逐步了解基本概念。

如果您想要深入了解 Azure 備份，請閱讀此 [概觀](backup-introduction-to-azure-backup.md)。

如果您沒有 Azure 訂用帳戶，請建立 [免費帳戶](https://azure.microsoft.com/free/) ，以便存取任何 Azure 服務。

## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫。
若要備份 Windows Server 系統狀態，您必須在要儲存這些資料的區域中建立復原服務保存庫。 您也需要決定儲存體的複寫方式。

### <a name="to-create-a-recovery-services-vault"></a>建立復原服務保存庫
1. 如果您尚未這麼做，請使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com/) 。
2. 在 [中樞] 功能表上按一下 [所有服務]，在資源清單中輸入**復原服務**，然後按一下 [復原服務保存庫]。

    ![建立復原服務保存庫的步驟 1](./media/backup-azure-system-state/open-rs-vault-list.png) <br/>

    如果訂用帳戶中有復原服務保存庫，則會列出保存庫。
3. 在 [復原服務保存庫] 功能表上，按一下 [新增]。

    ![建立復原服務保存庫的步驟 2](./media/backup-try-azure-backup-in-10-mins/rs-vault-menu.png)

    [復原服務保存庫] 刀鋒視窗隨即開啟，並提示您提供 [名稱]、[訂用帳戶]、[資源群組] 和 [位置]。

    ![建立復原服務保存庫的步驟 3](./media/backup-try-azure-backup-in-10-mins/rs-vault-step-3.png)

4. 在 [名稱] 中，輸入易記名稱來識別保存庫。 必須是 Azure 訂用帳戶中唯一的名稱。 輸入包含 2 到 50 個字元的名稱。 該名稱必須以字母開頭，而且只可以包含字母、數字和連字號。

5. 在 [訂用帳戶] 區段中，使用下拉式功能表來選擇 Azure 訂用帳戶。 如果您只使用一個訂用帳戶，該訂用帳戶會出現，您可以跳到下一個步驟。 如果您不確定要使用哪個訂用帳戶，請使用預設 (或建議) 的訂用帳戶。 只有在您的組織帳戶與多個 Azure 訂用帳戶相關聯時，才會有多個選擇。

6. 在 [資源群組] 區段中︰

    * 如果您想建立新的資源群組，請選取 [新建]。
    或
    * 選取 [使用現有的]﹐然後按一下下拉式功能表，以查看可用的資源群組清單。

  如需資源群組的完整資訊，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)。

7. 按一下 [位置]  以選取保存庫的地理區域。 此選項會決定您的備份資料要傳送到哪個地理區域。

8. 按一下 [復原服務保存庫] 刀鋒視窗底部的 [建立]。

    建立復原服務保存庫可能需要一些時間。 請監視入口網站右上方區域中的狀態通知。 保存庫一旦建立好，就會出現在 [復原服務保存庫] 的清單中。 在數分鐘之後﹐如果您沒有看到您的保存庫，請按一下 [重新整理]。

    ![按一下 [重新整理] 按鈕。](./media/backup-try-azure-backup-in-10-mins/refresh-button.png)</br>

    一旦在復原服務保存庫清單中看到您的保存庫，您即可開始設定儲存體備援。

### <a name="set-storage-redundancy-for-the-vault"></a>設定保存庫的儲存體備援
當您建立復原服務保存庫時，務必以您想要的方式設定儲存體備援。

1. 從 [復原服務保存庫] 刀鋒視窗，按一下 [新增保存庫]。

    ![從復原服務保存庫清單中選取新的保存庫](./media/backup-try-azure-backup-in-10-mins/rs-vault-list.png)

    當您選取保存庫時，[復原服務保存庫] 刀鋒視窗會縮小﹐而 [設定] 刀鋒視窗 (頂端有保存庫名稱) 和 [保存庫詳細資料] 刀鋒視窗隨即開啟。

    ![檢視新保存庫的儲存體組態](./media/backup-try-azure-backup-in-10-mins/set-storage-configuration-2.png)
2. 在新保存庫的 [設定] 刀鋒視窗中，使用垂直滑桿捲動至 [管理] 區段，然後按一下 [備份基礎結構]。
    [備份基礎結構] 刀鋒視窗隨即開啟。
3. 在 [備份基礎結構] 刀鋒視窗中，按一下 [備份設定]開啟 [備份設定] 刀鋒視窗。

    ![為新保存庫設定儲存體組態](./media/backup-try-azure-backup-in-10-mins/set-storage-configuration.png)
4. 為保存庫選擇適當的儲存體複寫選項。

    ![儲存體組態選項](./media/backup-try-azure-backup-in-10-mins/choose-storage-configuration.png)

    根據預設，保存庫具有異地備援儲存體。 如果您使用 Azure 做為主要的備份儲存體端點，請繼續使用 [異地備援]。 如果您未使用 Azure 做為主要的備份儲存體端點，則選擇 [本地備援]，以減少 Azure 儲存體成本。 在此[儲存體備援概觀](../storage/common/storage-redundancy.md)中，深入了解[異地備援](../storage/common/storage-redundancy-grs.md)和[本地備援](../storage/common/storage-redundancy-lrs.md)儲存體選項。

您已建立了保存庫，接著請設定它來備份 Windows 系統狀態。

## <a name="configure-the-vault"></a>設定保存庫
1. 在 [復原服務保存庫] 刀鋒視窗上 (針對剛建立的保存庫)，在 [開始使用] 區段中按一下 [備份]，然後在 [開始使用備份功能] 刀鋒視窗上，選取 [備份目標]。

    ![開啟備份目標刀鋒視窗](./media/backup-try-azure-backup-in-10-mins/open-backup-settings.png)

    [備份目標] 刀鋒視窗隨即開啟。

    ![開啟備份目標刀鋒視窗](./media/backup-try-azure-backup-in-10-mins/backup-goal-blade.png)

2. 從 [您的工作負載在何處執行?] 下拉式功能表中，選取 [內部部署]。

    因為您的 Windows Server 或 Windows 電腦是不在 Azure 中的實體電腦，所以您選擇 [內部部署]。

3. 從 [您要備份什麼?] 功能表中，選取 [系統狀態]，然後按一下 [確定]。

    ![設定檔案和資料夾](./media/backup-azure-system-state/backup-goal-system-state.png)

    按一下 [確定] 後，[備份目標] 旁會出現勾選記號，且 [準備基礎結構] 刀鋒視窗隨即開啟。

    ![已設定備份目標，接下來是準備基礎結構](./media/backup-try-azure-backup-in-10-mins/backup-goal-configed.png)

4. 在 [準備基礎結構] 刀鋒視窗上，按 [下載 Windows Server 或 Windows Client 的代理程式]。

    ![準備基礎結構](./media/backup-try-azure-backup-in-10-mins/choose-agent-for-server-client.png)

    如果您使用 Windows Server Essential，請選擇下載 Windows Server Essential 的代理程式。 快顯功能表會提示您執行或儲存 MARSAgentInstaller.exe。

    ![MARSAgentInstaller dialog](./media/backup-try-azure-backup-in-10-mins/mars-installer-run-save.png)

5. 在下載快顯功能表中，按一下 [儲存]。

    根據預設，**MARSagentinstaller.exe** 檔案會儲存至 [下載] 資料夾。 安裝程式完成時，您會看到快顯視窗，詢問您是否要執行安裝程式，或開啟資料夾。

    ![準備基礎結構](./media/backup-try-azure-backup-in-10-mins/mars-installer-complete.png)

    您還不需要安裝代理程式。 您可以在下載保存庫認證之後﹐安裝代理程式。

6. 在 [準備基礎結構] 刀鋒視窗上，按 [下載]。

    ![下載保存庫認證](./media/backup-try-azure-backup-in-10-mins/download-vault-credentials.png)

    保存庫認證會下載至「下載」資料夾。 保存庫認證下載完成之後，您會看到快顯視窗，詢問您是否要開啟或儲存認證。 按一下 [檔案] 。 如果您不小心按到 [開啟]，請讓嘗試開啟保存庫認證的對話方塊失敗。 您無法開啟保存庫認證。 請繼續進行下一個步驟。 保存庫認證位於 [下載] 資料夾中。   

    ![保存庫認證下載完成](./media/backup-try-azure-backup-in-10-mins/vault-credentials-downloaded.png)
> [!NOTE]
> 保存庫認證只能儲存至您要使用代理程式的 Windows Server 的本機位置。 
>

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="install-and-register-the-agent"></a>安裝和註冊代理程式

> [!NOTE]
> 透過 Azure 入口網站啟用備份的功能尚未推出。 使用 Microsoft Azure 復原服務代理程式來備份 Windows Server 系統狀態。
>

1. 在 [下載] 資料夾 (或其他儲存位置) 中找到 **MARSagentinstaller.exe** 並對其按兩下。

    安裝程式在擷取、安裝和註冊復原服務代理程式時會提供一系列訊息。

    ![執行復原服務代理安裝程式的認證](./media/backup-try-azure-backup-in-10-mins/mars-installer-registration.png)

2. 完成 Microsoft Azure 復原服務代理程式安裝精靈。 若要完成精靈，您需要︰

   * 選擇安裝和快取資料夾的位置。
   * 如果您使用 Proxy 伺服器來連線到網際網路，請提供您的 Proxy 伺服器資訊。
   * 如果您使用已驗證的 Proxy，請提供您的使用者名稱和密碼詳細資料。
   * 提供下載的保存庫認證
   * 將加密複雜密碼存放在安全的位置。

     > [!NOTE]
     > 如果遺失或忘記複雜密碼，Microsoft 將無法協助您復原備份資料。 將檔案存放在安全的位置。 必須有此檔案才能還原備份。
     >
     >

現已安裝代理程式，且已向保存庫註冊您的電腦。 您已準備好可以設定及排程備份。

## <a name="back-up-windows-server-system-state"></a>備份 Windows Server 系統狀態 
初始備份包括兩項工作：

* 排程備份
* 第一次備份系統狀態

若要完成初始備份，請使用 Microsoft Azure 復原服務代理程式。

> [!NOTE]
> 您可以透過 Windows Server 2016，在 Windows Server 2008 R2 上備份系統狀態。 用戶端 SKU 不支援系統狀態備份。 [系統狀態] 不會顯示為 Windows 用戶端或 Windows Server 2008 SP2 電腦的選項。
>
>

### <a name="to-schedule-the-backup-job"></a>排程備份作業

1. 開啟 Microsoft Azure 復原服務代理程式。 您可以透過在您的電腦中搜尋 **Microsoft Azure 備份**來找出備份。

    ![啟動 Azure 復原服務代理程式](./media/backup-try-azure-backup-in-10-mins/snap-in-search.png)

2. 在復原服務代理程式中，按一下 [排程備份] 。

    ![Windows Server 備份排程](./media/backup-try-azure-backup-in-10-mins/schedule-first-backup.png)

3. 在排程備份精靈的 [開始使用] 頁面上，按 [下一步] 。

4. 在 [選取要備份的項目] 頁面上，按一下 [新增項目] 。

5. 選取 [系統狀態]，然後按一下 [確定]。

6. 按 [下一步] 。

7. 在後續頁面中針對系統狀態備份選取所需的備份頻率和保留原則。 

8. 在 [確認] 頁面上檢閱資訊，然後按一下 [完成] 。

9. 當精靈建立好備份排程時，請按一下 [關閉] 。

### <a name="to-back-up-windows-server-system-state-for-the-first-time"></a>第一次備份 Windows Server 系統狀態

1. 確定 Windows Server 沒有需要重新啟動的擱置中更新。

2. 在復原服務代理程式中按一下 [立即備份]  ，以透過網路完成初始植入。

    ![立即備份 Windows Server](./media/backup-try-azure-backup-in-10-mins/backup-now.png)

3. 在出現的 [選取備份項目] 畫面上選取 [系統狀態]，然後按一下 [下一步]。

4. 在 [確認] 頁面上，檢閱立即備份精靈將用於備份電腦的設定。 然後按一下 [備份] 。

4. 按一下 [關閉]  即可關閉精靈。 如果您在備份程序完成之前關閉精靈，精靈會繼續在背景中執行。


完成初始備份之後，備份主控台中會顯示 [作業已完成]  狀態。

  ![IR 已完成](./media/backup-try-azure-backup-in-10-mins/ircomplete.png)

## <a name="questions"></a>有疑問嗎？
如果您有問題，或希望我們加入任何功能，請 [傳送意見反應給我們](http://aka.ms/azurebackup_feedback)。

## <a name="next-steps"></a>後續步驟
* 詳細了解如何 [備份 Windows 電腦](backup-configure-vault.md)。
* 現在您已備份好 Windows Server 系統狀態，接下來您可以[管理您的保存庫和伺服器](backup-azure-manage-windows-server.md)。
* 如果您需要還原備份，請使用本文來 [還原檔案到 Windows 電腦](backup-azure-restore-windows-server.md)。
