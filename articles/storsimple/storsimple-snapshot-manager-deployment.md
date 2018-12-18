---
title: 部署 StorSimple Snapshot Manager | Microsoft Docs
description: 了解如何下載及安裝 StorSimple Snapshot Manager MMC 嵌入式管理單元，來管理 StorSimple 資料保護和備份功能。
services: storsimple
documentationcenter: NA
author: SharS
manager: timlt
editor: ''
ms.assetid: f0128f57-519e-49ec-9187-23575809cdbe
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: v-sharos
ms.openlocfilehash: cde355381b0d726a1ab340bc4230b2dc8f6e2c56
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110666"
---
# <a name="deploy-the-storsimple-snapshot-manager-mmc-snap-in"></a>部署 StorSimple Snapshot Manager MMC 嵌入式管理單元

## <a name="overview"></a>概觀
StorSimple Snapshot Manager 是 Microsoft Management Console (MMC) 嵌入式管理單元，可在 Microsoft Azure StorSimple 環境中簡化資料保護和備份管理。 利用 StorSimple Snapshot Manager，您可以管理 Microsoft Azure StorSimple 內部部署和雲端儲存體，就好像是完全整合的儲存系統，並藉此簡化備份和還原程序並降低成本。 

本教學課程描述組態需求，以及安裝、移除及升級 StorSimple Snapshot Manager 的程序。

> [!NOTE]
> * 您無法使用 StorSimple Snapshot Manager 來管理 Microsoft Azure StorSimple Virtual Arrays (也稱為 StorSimple 內部部署虛擬裝置)。
> * 如果您打算在 StorSimple 裝置上安裝 StorSimple Update 2，在 **安裝 StorSimple Update 2 之前**，請務必下載最新版的 StorSimple Snapshot Manager 並安裝它。 最新版的 StorSimple Snapshot Manager 具回溯相容性，可搭配 Microsoft Azure StorSimple 的所有發行版本使用。 如果您使用的是舊版的 StorSimple Snapshot Manager，您必須更新它 (安裝新版本前不需解除安裝舊版)。


## <a name="storsimple-snapshot-manager-installation"></a>StorSimple Snapshot Manager 安裝
StorSimple Snapshot Manager 可以安裝在執行 Windows Server 2008 R2 SP1、Windows Server 2012 或 Windows Server 2012 R2 作業系統的電腦上。 在執行 Windows 2008 R2 的伺服器上，您也必須安裝 Windows Server 2008 SP1 和 Windows Management Framework 3.0。

在您安裝或升級 Microsoft Management Console (MMC) 的 StorSimple Snapshot Manager 嵌入式管理單元之前，請確定已正確設定 Microsoft Azure StorSimple 裝置及主機伺服器。

## <a name="configure-prerequisites"></a>設定必要條件
下列步驟提供在安裝 StorSimple Snapshot Manager 之前必須完成之組態工作的高階概觀。 如需完整的 Microsoft Azure StorSimple 組態和安裝資訊，包括系統需求和逐步指示，請參閱 [部署您的內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)。

> [!IMPORTANT]
> 在您開始之前，請檢閱[部署您的內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)中的[部署設定檢查清單](storsimple-8000-deployment-walkthrough-u2.md#deployment-configuration-checklist)和[部署必要條件](storsimple-8000-deployment-walkthrough-u2.md#deployment-prerequisites)。
> <br>
> 
> 

### <a name="before-you-install-storsimple-snapshot-manager"></a>安裝 StorSimple Snapshot Manager 之前
1. 打開包裝、掛接和連接 Microsoft Azure StorSimple 裝置，如[安裝 StorSimple 8100 裝置](storsimple-8100-hardware-installation.md)或[安裝 StorSimple 8600 裝置](storsimple-8600-hardware-installation.md)所述。
2. 請確定主機電腦正在執行下列其中一個作業系統：
   
   * Windows Server 2008 R2 (在執行 Windows 2008 R2 的伺服器上，您也必須安裝 Windows Server 2008 SP1 和 Windows Management Framework 3.0)
   * Windows Server 2012
   * Windows Server 2012 R2
     
     對於 StorSimple 虛擬裝置，主機必須是 Microsoft Azure 虛擬機器。
3. 請確定您符合所有的 Microsoft Azure StorSimple 組態需求。 如需詳細資料，請移至 [部署必要條件](storsimple-8000-deployment-walkthrough-u2.md#deployment-prerequisites)。
4. 將裝置連接至主機並執行初始組態。 如需詳細資料，請移至 [部署步驟](storsimple-8000-deployment-walkthrough-u2.md#deployment-steps)。
5. 在裝置上建立磁碟區，將它們指派給主機，並確認主機可以掛接和使用它們。 StorSimple Snapshot Manager 支援下列類型的磁碟區：
   
   * 基本磁碟區
   * 簡單磁碟區
   * 動態磁碟區
   * 鏡像動態磁碟區 (RAID 1)
   * 叢集共用磁碟區
     
     如需在 StorSimple 裝置或 StorSimple 虛擬裝置上建立磁碟區的詳細資訊，請移至[部署您的內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)中的[步驟 6：建立磁碟區](storsimple-8000-deployment-walkthrough-u2.md#step-6-create-a-volume)。

## <a name="install-a-new-storsimple-snapshot-manager"></a>安裝新的 StorSimple Snapshot Manager
安裝 StorSimple Snapshot Manager 之前，請確定您在 StorSimple 裝置或 StorSimple 虛擬裝置上建立的磁碟區會掛接、初始化和格式化，如 [設定必要條件](#configure-prerequisites)中所述。

> [!IMPORTANT]
> * 對於 StorSimple 虛擬裝置，主機必須是 Microsoft Azure 虛擬機器。 
> * 主機必須執行 Windows 2008 R2、Windows Server 2012 或 Windows Server 2012 R2。 如果您的伺服器執行 Windows Server 2008 R2，您也必須安裝 Windows Server 2008 SP1 和 Windows Management Framework 3.0。
> * 將裝置連接至 StorSimple Snapshot Manager 之前，您必須設定從主機到 StorSimple 裝置的 iSCSI 連接。

請遵循下列步驟來完成 StorSimple Snapshot Manager 的全新安裝。 如果您要安裝升級，請至 [升級或重新安裝 StorSimple Snapshot Manager](#upgrade-or-reinstall-storsimple-snapshot-manager)。

* 步驟 1：安裝 StorSimple Snapshot Manager 
* 步驟 2：連接 StorSimple Snapshot Manager 和裝置 
* 步驟 3：確認裝置的連接 

### <a name="step-1-install-storsimple-snapshot-manager"></a>步驟 1：安裝 StorSimple Snapshot Manager
使用下列步驟來安裝 StorSimple Snapshot Manager。

#### <a name="to-install-storsimple-snapshot-manager"></a>安裝 StorSimple Snapshot Manager
1. 下載 StorSimple Snapshot Manager 軟體 (移至 Microsoft 下載中心的 [StorSimple Snapshot Manager](https://www.microsoft.com/download/details.aspx?id=44220) ) 並將軟體儲存在本機主機上。
2. 在檔案總管中，以滑鼠右鍵按一下壓縮的資料夾，然後按一下 [ **全部解壓縮**]。
3. 在 [解壓縮壓縮 (壓縮) 資料夾] 視窗的 [選取目的地並解壓縮檔案] 方塊中，輸入或瀏覽至您想要解壓縮檔案的路徑。
   
    > [!IMPORTANT]
    > 您必須在 C: 磁碟機上安裝 StorSimple Snapshot Manager。
    
4. 選取 [完成時顯示解壓縮檔案] 核取方塊，然後再按一下 [解壓縮]。
   
    ![解壓縮檔案對話方塊](./media/storsimple-snapshot-manager-deployment/HCS_SSM_extract_files.png) 
5. 解壓縮完成時，目的地資料夾會隨即開啟。 按兩下出現在目的地資料夾中的應用程式安裝圖示。
6. 當**安裝成功**訊息出現時，按一下 [關閉]。 您應該會在桌面上看到 StorSimple Snapshot Manager 圖示。
   
    ![桌面圖示](./media/storsimple-snapshot-manager-deployment/HCS_SSM_desktop_icon.png) 

### <a name="step-2-connect-storsimple-snapshot-manager-to-a-device"></a>步驟 2：連接 StorSimple Snapshot Manager 和裝置
使用下列步驟連接 StorSimple Snapshot Manager 和 StorSimple 裝置。

#### <a name="to-connect-storsimple-snapshot-manager-to-a-device"></a>連接 StorSimple Snapshot Manager 和裝置
1. 按一下桌面上的 StorSimple Snapshot Manager 圖示。 StorSimple Snapshot Manager 視窗隨即出現。 此視窗包含 [範圍] 窗格、[結果] 窗格和 [動作] 窗格。 
   
    ![StorSimple Snapshot Manager 使用者介面](./media/storsimple-snapshot-manager-deployment/HCS_SSM_gui_panes.png)
   
   * [ **範圍** ] 窗格 (左窗格) 包含在樹狀結構中組織的節點清單。 您可以展開一些節點來選取檢視或與該節點相關的特定資料。 按一下箭頭圖示來展開或摺疊節點。 以滑鼠右鍵按一下 [ **範圍** ] 窗格中的項目，以查看該項目之可用動作的清單。
   * [結果] 窗格 (中央窗格) 包含您在 [範圍] 窗格中選取之節點、檢視或資料的相關詳細狀態資訊。
   * [動作] 窗格會列出您在 [範圍] 窗格中選取的節點、檢視或資料上可以執行的作業。
     
     如需 StorSimple Snapshot Manager 使用者介面的完整描述，請參閱 [StorSimple Snapshot Manager 使用者介面](storsimple-use-snapshot-manager.md)。
2. 在 [範圍] 窗格中，以滑鼠右鍵按一下 [裝置] 節點，然後按一下 [設定裝置]。 [ **設定裝置** ] 對話方塊隨即出現。
   
    ![設定裝置](./media/storsimple-snapshot-manager-deployment/HCS_SSM_config_device.png) 
3. 在 [ **裝置** ] 清單方塊中，選取 Microsoft Azure StorSimple 裝置或虛擬裝置的 IP 位址。 在 [密碼] 文字方塊中，輸入您在 Azure 入口網站中為裝置建立的 StorSimple Snapshot Manager 密碼。 按一下 [SERVICEPRINCIPAL] 。
4. StorSimple Snapshot Manager 會搜尋您所識別的裝置。 如果裝置可供使用，StorSimple Snapshot Manager 會新增連接。 您可以 [確認裝置連接](#to-verify-the-connection) 以確認連接已成功新增。
   
    如果由於任何原因而無法使用裝置，StorSimple Snapshot Manager 會傳回錯誤訊息。 按一下 [確定] 以關閉錯誤訊息，然後按一下 [取消] 以關閉 [設定裝置] 對話方塊。
5. 當它連接到裝置時，StorSimple Snapshot Manager 會匯入為該裝置設定的每個磁碟區群組，前提是磁碟區群組具有相關聯的備份。 不會匯入沒有相關聯備份的磁碟區群組。 此外，不會匯入為磁碟區群組建立的備份原則。 若要查看匯入的群組，請以滑鼠右鍵按一下 [範圍] 窗格中最上層的 [磁碟區群組] 節點，然後按一下 [切換匯入的群組]。

### <a name="step-3-verify-the-connection-to-the-device"></a>步驟 3：確認裝置的連接
使用下列步驟來確認 StorSimple Snapshot Manager 已連接至 StorSimple 裝置。

#### <a name="to-verify-the-connection"></a>確認連接
1. 在 [範圍] 窗格中，按一下 [裝置] 節點。
   
    ![StorSimple Snapshot Manager 裝置狀態](./media/storsimple-snapshot-manager-deployment/HCS_SSM_Device_status.png) 
2. 檢查 [ **結果** ] 窗格： 
   
   * 如果裝置圖示上出現綠色指標和且 [狀態] 資料行顯示為 [可用]，表示裝置已連接。 
   * 如果裝置圖示上出現紅色指標，且 [ **狀態** ] 資料行中顯示為無法使用，表示裝置未連接。 
   * 如果 [狀態] 資料行中顯示為 [正在重新整理]，表示 StorSimple Snapshot Manager 正在擷取磁碟區群組及連接裝置的相關聯備份。

## <a name="upgrade-or-reinstall-storsimple-snapshot-manager"></a>升級或重新安裝 StorSimple Snapshot Manager
您應該先完全解除安裝 StorSimple Snapshot Manager 之後再升級或重新安裝軟體。 

重新安裝 StorSimple Snapshot Manager 之前，請在主機電腦上備份現有的 StorSimple Snapshot Manager 資料庫。 這會儲存備份原則及組態資訊，以便您可以輕易地從備份還原這項資料。

如果您要升級或重新安裝 StorSimple Snapshot Manager，請遵循下列步驟：

* 步驟 1：解除安裝 StorSimple Snapshot Manager 
* 步驟 2：備份 StorSimple Snapshot Manager 資料庫 
* 步驟 3：重新安裝 StorSimple Snapshot Manager 並還原資料庫 

### <a name="step-1-uninstall-storsimple-snapshot-manager"></a>步驟 1：解除安裝 StorSimple Snapshot Manager
使用下列步驟來解除安裝 StorSimple Snapshot Manager。

#### <a name="to-uninstall-storsimple-snapshot-manager"></a>解除安裝 StorSimple Snapshot Manager
1. 在主機電腦上，開啟 [控制台]，按一下 [程式]，然後按一下 [程式和功能]。
2. 在左窗格中，按一下 [ **解除安裝或變更程式**]。
3. 以滑鼠右鍵按一下 [StorSimple Snapshot Manager]，然後按一下 [解除安裝]。
4. 這樣會啟動 StorSimple Snapshot Manager 安裝程式。 按一下 [修改安裝程式]，然後按一下 [解除安裝]。
   
   > [!NOTE]
   > 如果有任何 MMC 程序在背景執行，例如 StorSimple Snapshot Manager 或磁碟管理，解除安裝將會失敗，而且在您嘗試解除安裝程式之前，您會收到關閉所有 MMC 執行個體的訊息。 選取 [自動關閉應用程式，並嘗試在安裝完成後重新啟動]，然後按一下 [確定]。
   > 
   > 
5. 解除安裝程序完成時， **安裝成功** 訊息隨即出現。 按一下 [關閉] 。

### <a name="step-2-back-up-the-storsimple-snapshot-manager-database"></a>步驟 2：備份 StorSimple Snapshot Manager 資料庫
使用下列步驟建立和儲存 StorSimple Snapshot Manager 資料庫的複本。

#### <a name="to-back-up-the-database"></a>備份資料庫
1. 停止 Microsoft StorSimple 管理服務：
   
   1. 啟動伺服器管理員。
   2. 在伺服器管理員儀表板的 [工具] 功能表上，選取 [服務]。
   3. 在 [服務] 頁面上，選取 [Microsoft StorSimple 管理服務]。
   4. 在右窗格的 [Microsoft StorSimple 管理服務] 之下，按一下 [停止服務]。
      
        ![停止 StorSimple 裝置管理員服務](./media/storsimple-snapshot-manager-deployment/HCS_SSM_stop_service.png)
2. 瀏覽至 C:\ProgramData\Microsoft\StorSimple\BACatalog。 
   
   > [!NOTE]
   > ProgramData 是隱藏的資料夾。
  
3. 尋找目錄 XML 檔案、複製檔案，並將複本儲存在安全位置或雲端中。
   
    ![StorSimple 備份目錄檔案](./media/storsimple-snapshot-manager-deployment/HCS_SSM_bacatalog.png)
4. 重新啟動 Microsoft StorSimple 管理服務： 
   
   1. 在伺服器管理員儀表板的 [工具] 功能表上，選取 [服務]。
   2. 在 [服務] 頁面上，選取 [Microsoft StorSimple 管理服務]。
   3. 在右窗格的 [Microsoft StorSimple 管理服務] 之下，按一下 [重新啟動服務]。 

### <a name="step-3-reinstall-storsimple-snapshot-manager-and-restore-the-database"></a>步驟 3：重新安裝 StorSimple Snapshot Manager 並還原資料庫
如果要重新安裝 StorSimple Snapshot Manager，請遵循 [安裝新的 StorSimple Snapshot Manager](#install-a-new-storsimple-snapshot-manager)中的步驟。 然後，使用下列程序來還原 StorSimple Snapshot Manager 資料庫。

#### <a name="to-restore-the-database"></a>還原資料庫
1. 停止 Microsoft StorSimple 管理服務：
   
   1. 啟動伺服器管理員。
   2. 在伺服器管理員儀表板的 [工具] 功能表上，選取 [服務]。
   3. 在 [服務] 頁面上，選取 [Microsoft StorSimple 管理服務]。
   4. 在右窗格的 [Microsoft StorSimple 管理服務] 之下，按一下 [停止服務]。
2. 瀏覽至 C:\ProgramData\Microsoft\StorSimple\BACatalog。
   
   > [!NOTE]
   > ProgramData 是隱藏的資料夾。
   > 
   > 
3. 刪除目錄 XML 檔案，並以您稍早儲存的版本取代它。
4. 重新啟動 Microsoft StorSimple 管理服務： 
   
   1. 在伺服器管理員儀表板的 [工具] 功能表上，選取 [服務]。
   2. 在 [服務] 頁面上，選取 [Microsoft StorSimple 管理服務]。
   3. 在右窗格的 [Microsoft StorSimple 管理服務] 之下，按一下 [重新啟動服務]。

## <a name="next-steps"></a>後續步驟
* 若要深入了解 StorSimple Snapshot Manager，請移至 [什麼是 StorSimple Snapshot Manager？](storsimple-what-is-snapshot-manager.md)
* 若要深入了解 StorSimple Snapshot Manager 使用者介面，請移至 [StorSimple Snapshot Manager 使用者介面](storsimple-use-snapshot-manager.md)
* 若要深入了解如何使用 StorSimple Snapshot Manager，請移至 [使用 StorSimple Snapshot Manager 來管理您的 StorSimple 解決方案](storsimple-snapshot-manager-admin.md)。

