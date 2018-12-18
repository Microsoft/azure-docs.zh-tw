---
title: 在 Microsoft Azure StorSimple Virtual Array 上安裝更新 | Microsoft Docs
description: 描述如何透過 StorSimple Virtual Array Web UI，使用入口網站和 Hotfix 方法套用更新
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 9997a97b-9382-43ed-b56e-61369335c987
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c2d081604c0ca01f47c3ff2aab7477859d280963
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110166"
---
# <a name="install-updates-on-your-storsimple-virtual-array---azure-portal"></a>在 StorSimple Virtual Array 上安裝更新 - Azure 入口網站

## <a name="overview"></a>概觀

本文說明透過本機 Web UI 和透過 Azure 入口網站，在 StorSimple Virtual Array 上安裝更新時所需的步驟。 您需要套用軟體更新或 Hotfix，以便讓您的 StorSimple Virtual Array 保持在最新狀態。 

請記住，安裝更新或 Hotfix 會重新啟動您的裝置。 假設 StorSimple Virtual Array 是單一節點裝置，就會中斷任何進行中的 I/O，裝置也會停機。 

套用更新之前，建議您先讓主機上的磁碟區或共用離線，再讓裝置離線。 這樣可以讓資料損毀的可能性降至最低。

> [!IMPORTANT]
> 如果您正在執行 Update 0.1 或 GA 軟體版本，您就必須透過本機 Web UI 使用 Hotfix 方法來安裝 Update 0.3。 如果您執行的是 Update 0.2，建議您透過 Azure 傳統入口網站安裝更新。
 

## <a name="use-the-local-web-ui"></a>使用本機 Web UI

使用本機 Web UI 時有兩個步驟︰

* 下載更新或 Hotfix
* 安裝更新或 Hotfix

### <a name="download-the-update-or-the-hotfix"></a>下載更新或 Hotfix

請執行下列步驟，從 Microsoft Update Catalog 下載軟體更新。

#### <a name="to-download-the-update-or-the-hotfix"></a>下載更新或 Hotfix

1. 啟動 Internet Explorer 並瀏覽至 [http://catalog.update.microsoft.com](http://catalog.update.microsoft.com)。

2. 如果這是您第一次在此電腦上使用 Microsoft Update Catalog，請在系統提示您安裝 Microsoft Update Catalog 附加元件時，按一下 [安裝]  。

3. 在 Microsoft Update Catalog 的搜尋方塊中，輸入您要下載的 Hotfix 知識庫 (KB) 編號。 輸入 **3182061** (適用於 Update 0.3)，然後按一下 [搜尋]。
   
    此時會顯示 Hotfix 清單，例如 **StorSimple Virtual Array Update 0.3**。
   
    ![搜尋目錄](./media/storsimple-virtual-array-install-update/download1.png)

4. 按一下 [新增] 。 更新便會新增到購物籃中。

5. 按一下 [ **檢視購物籃**]。

6. 按一下 [下載] 。 指定或「瀏覽」  至您想要儲存下載項目的本機位置。 更新便會下載到指定的位置，並放在與更新名稱相同的子資料夾中。 資料夾也可以複製到裝置可連線的網路共用位置。

7. 開啟已複製的資料夾，您應該會看到 Microsoft Update 獨立封裝檔案 `WindowsTH-KB3011067-x64`。 此檔案是用來安裝更新或 Hotfix。

### <a name="install-the-update-or-the-hotfix"></a>安裝更新或 Hotfix

安裝更新或 Hotfix 之前，請確定更新或 Hotfix 已下載至您主機的本機上，或是可透過網路共用存取。 

請使用此方法在執行 GA 或 Update 0.1 軟體版本的裝置上安裝更新。 此程序需時不到 2 分鐘。 請執行下列步驟以安裝更新或 Hotfix。

#### <a name="to-install-the-update-or-the-hotfix"></a>安裝更新或 Hotfix

1. 在本機 Web UI 中，移至 [維護]  >  [軟體更新]。
   
    ![更新裝置](./media/storsimple-virtual-array-install-update/update1m.png)

2. 在 [更新檔案路徑] 中，輸入更新或 Hotfix 的檔案名稱。 如果更新或 Hotfix 的安裝檔案是放在網路共用上，您也可以瀏覽至該檔案。 按一下 [套用]。
   
    ![更新裝置](./media/storsimple-virtual-array-install-update/update2m.png)

3. 此時會顯示警告。 如果這是單一節點裝置，在套用更新後，裝置就會重新啟動而會有停機時間。 按一下核取圖示。
   
   ![更新裝置](./media/storsimple-virtual-array-install-update/update3m.png)

4. 更新會開始進行。 成功更新裝置之後，裝置就會重新啟動。 在這段持續時間會無法存取本機 UI。
   
    ![更新裝置](./media/storsimple-virtual-array-install-update/update5m.png)

5. 重新啟動完成後，您就會進入 [登入] 頁面。 若要確認裝置軟體是否已更新，請在本機 Web UI 中，移至 [維護]  >  [軟體更新]。 顯示的軟體版本應該是 **10.0.0.0.0.10288.0** (適用於 Update 0.3)。
   
   > [!NOTE]
   > 我們在本機 Web UI 和 Azure 入口網站中回報軟體版本的方式略有不同。 例如，本機 Web UI 會回報 **10.0.0.0.0.10288**，而相同版本在 Azure 入口網站則會回報為 **10.0.10288.0**。
   
    ![更新裝置](./media/storsimple-virtual-array-install-update/update6m.png)

## <a name="use-the-azure-portal"></a>使用 Azure 入口網站

如果執行 Update 0.2，建議您透過 Azure 入口網站安裝更新。 入口網站的程序需要使用者掃描、下載然後安裝更新。 此程序需要大約 7 分鐘才能完成。 請執行下列步驟以安裝更新或 Hotfix。

[!INCLUDE [storsimple-virtual-array-install-update-via-portal](../../includes/storsimple-virtual-array-install-update-via-portal.md)]

安裝完成後 (以作業狀態 100% 表示)，請移至 StorSimple 裝置管理員服務。 選取 [裝置]，然後從連接至此服務的裝置清單中，選取並按一下您想要更新的裝置。 在 [設定] 刀鋒視窗中，移至 [管理] 區段，然後選取 [裝置更新]。 顯示的軟體版本應該是 **10.0.10288.0**。


## <a name="next-steps"></a>後續步驟

深入了解 [administering your StorSimple Virtual Array (管理 StorSimple Virtual Array)](storsimple-ova-web-ui-admin.md)。

