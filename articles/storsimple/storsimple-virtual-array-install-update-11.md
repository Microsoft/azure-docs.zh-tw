---
title: 在 StorSimple Virtual Array 上安裝 Update 1.1 | Microsoft Docs
description: 說明如何使用 Azure 入口網站與 StorSimple Virtual Array 的本機 Web UI 來套用更新
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/18/2018
ms.author: alkohli
ms.openlocfilehash: e10bd04f37951f93db8af083692b7a2fe25ac9b7
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39347761"
---
# <a name="install-update-11-on-your-storsimple-virtual-array"></a>在 StorSimple Virtual Array 上安裝 Update 1.1

## <a name="overview"></a>概觀

本文說明透過本機 Web UI 和透過 Azure 入口網站在 StorSimple Virtual Array 上安裝 Update 1.1 時所需的步驟。

您套用軟體更新或 Hotfix，以便讓您的 StorSimple Virtual Array 保持在最新狀態。 套用更新之前，建議您先讓主機上的磁碟區或共用離線，再讓裝置離線。 這樣可以讓資料損毀的可能性降至最低。 磁碟區或共用離線之後，您也應該手動備份裝置。

> [!IMPORTANT]
> - Update 1.1 對應至您裝置上的 **10.0.10307.0** 軟體版本。 如需此更新中新功能的資訊，請移至 [Update 1.1 的版本資訊](storsimple-virtual-array-update-11-release-notes.md)。
>
> - 請記住，安裝更新或 Hotfix 會重新啟動您的裝置。 假設 StorSimple Virtual Array 是單一節點裝置，就會中斷任何進行中的 I/O，裝置也會停機。
>
> - 只有在虛擬陣列執行 Update 1 時，Azure 入口網站中才會提供 Update 1.1。 針對執行 Update 0.6 版本的虛擬陣列，您必須先安裝 Update 1.0，然後再套用 Update 1.1。

## <a name="use-the-azure-portal"></a>使用 Azure 入口網站

如果執行 Update 0.2 或更新版本，建議您透過 Azure 入口網站安裝更新。 入口網站的程序需要使用者掃描、下載然後安裝更新。 透過 Azure 入口網站套用的更新 會有所差異，視虛擬陣列值型的軟體版本而定。

 - 如果您的虛擬陣列執行的是 Update 1，Azure 入口網站會直接在裝置上安裝 Update 1.1 (10.0.10307.0)。 此程序需要大約 10 分鐘才能完成。
 - 如果您的虛擬陣列執行的是 Update 0.6，則會分兩個階段完成更新。 Azure 入口網站會在您的裝置上先安裝 Update 1.0 (10.0.10296.0)。 虛擬陣列會重新啟動，然後入口網站會在裝置上安裝 Update 1.1 (10.0.10307.0)。 此程序需要大約 15 分鐘才能完成。


[!INCLUDE [storsimple-virtual-array-install-update-via-portal](../../includes/storsimple-virtual-array-install-update-via-portal-11.md)]

安裝完成後，請移至 StorSimple 裝置管理員服務。 選取 [裝置]，然後選取並按一下您剛剛更新的裝置。 移至 [設定] > [管理] > [裝置更新]。 顯示的軟體版本應該是 **10.0.10307.0**。

![更新之後的軟體版本](./media/storsimple-virtual-array-install-update-11/azupdate17m2.png)

## <a name="use-the-local-web-ui"></a>使用本機 Web UI

使用本機 Web UI 時有兩個步驟︰

* 下載更新或 Hotfix
* 安裝更新或 Hotfix

> [!IMPORTANT] 
> **如果執行的是 Update 1 (10.0.10296.0)，才要繼續進行此更新。如果執行的是 Update 0.6，請先在裝置上[安裝 Update 1](storsimple-virtual-array-install-update-1.md)，再套用 Update 1.1。**

### <a name="download-the-update-or-the-hotfix"></a>下載更新或 Hotfix

請執行下列步驟，從 Microsoft Update Catalog 下載 Update 1.1。

#### <a name="to-download-the-update-or-the-hotfix"></a>下載更新或 Hotfix

1. 啟動 Internet Explorer 並瀏覽至 [http://catalog.update.microsoft.com](http://catalog.update.microsoft.com)。

2. 如果這是您第一次在此電腦上使用 Microsoft Update Catalog，請在系統提示您安裝 Microsoft Update Catalog 附加元件時，按一下 [安裝]。

3. 在 Microsoft Update Catalog 的搜尋方塊中，輸入您要下載的 Hotfix 知識庫 (KB) 編號。 輸入 **4337628** (適用於 Update 1.1)，然後按一下 [搜尋]。
   
    此時會顯示 Hotfix 清單，例如 **StorSimple Virtual Array Update 1.1**。
   
    ![搜尋目錄](./media/storsimple-virtual-array-install-update-11/download1.png)

4. 按一下 [下載]。

5. 將兩個檔案下載至資料夾。 您也可以將資料夾複製到裝置可連線的網路共用位置。

6. 開啟檔案所在的資料夾。

    ![套件中的檔案](./media/storsimple-virtual-array-install-update-11/update01folder.png)

    您會看到兩個檔案：
    -  Microsoft Update 獨立套件檔案`WindowsTH-KB3011067-x64`。 此檔案用來更新裝置軟體。
    - 包含今年 6 月份累積更新的檔案 `Windows8.1-KB4284815-x64`。 如需有關此彙總套件中包含之內容的詳細資訊，請移至 [6 月份每月安全性彙總套件](https://support.microsoft.com/help/4284815/windows-81-update-kb4284815)。

### <a name="install-the-update-or-the-hotfix"></a>安裝更新或 Hotfix

安裝更新或 Hotfix 之前，請確定：

 - 已有更新檔，或 Hotfix 已下載至您主機的本機上，或是可透過網路共用存取。
 - 您的虛擬陣列執行的是 Update 1 (10.0.10296.0)。 如果執行的是 Update 0.6，請先[安裝 Update 1](storsimple-virtual-array-install-update-1.md)，然後再安裝 Update 1.1。

此程序需要大約 4 分鐘才能完成。 請執行下列步驟以安裝更新或 Hotfix。

#### <a name="to-install-the-update-or-the-hotfix"></a>安裝更新或 Hotfix

1. 在本機 Web UI 中，移至 [維護]  >  [軟體更新]。 記下您在執行的軟體版本。 **如果執行的是 Update 1 (10.0.10296.0)，才要繼續進行此更新。如果執行的是 Update 0.6，請先在裝置上[安裝 Update 1](storsimple-virtual-array-install-update-1.md)，再套用 Update 1.1。**
   
    ![更新裝置](./media/storsimple-virtual-array-install-update-11/update1m.png)

2. 在 [更新檔案路徑] 中，輸入更新或 Hotfix 的檔案名稱。 如果更新或 Hotfix 的安裝檔案是放在網路共用上，您也可以瀏覽至該檔案。 按一下 [套用]。
   
    ![更新裝置](./media/storsimple-virtual-array-install-update-11/update2m.png)

3. 此時會顯示警告。 如果虛擬陣列是單一節點裝置，在套用更新後，裝置就會重新啟動而會有停機時間。 按一下核取圖示。
   
   ![更新裝置](./media/storsimple-virtual-array-install-update-11/update3m.png)

4. 更新會開始進行。 成功更新裝置之後，裝置就會重新啟動。 在這段持續時間會無法存取本機 UI。
   
    ![更新裝置](./media/storsimple-virtual-array-install-update-11/update5m.png)

5. 重新啟動完成後，您就會進入 [登入] 頁面。 若要確認裝置軟體是否已更新，請在本機 Web UI 中，移至 [維護]  >  [軟體更新]。 顯示的軟體版本應該是 **10.0.0.0.0.10307** (適用於 Update 1.1)。
   
   > [!NOTE]
   > 我們在本機 Web UI 和 Azure 入口網站中回報軟體版本的方式略有不同。 例如，本機 Web UI 會回報 **10.0.0.0.0.10307**，而相同版本在 Azure 入口網站則會回報為 **10.0.10307.0**。
   
    ![更新裝置](./media/storsimple-virtual-array-install-update-11/update6m.png)

6. 使用檔案 `Windows8.1-KB4284815-x64`，重複步驟 2-4 安裝 Windows 安全性修正。 安裝後會重新啟動虛擬陣列，而您需要登入本機的 Web UI。


## <a name="next-steps"></a>後續步驟

深入了解 [administering your StorSimple Virtual Array (管理 StorSimple Virtual Array)](storsimple-ova-web-ui-admin.md)。
