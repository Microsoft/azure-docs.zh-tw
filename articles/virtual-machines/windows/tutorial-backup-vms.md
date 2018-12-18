---
title: 教學課程 - 在 Azure 入口網站中備份 Windows 虛擬機器 | Microsoft Docs
description: 在本教學課程中，您會學習如何使用 Azure 入口網站，以 Azure 備份保護 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/27/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 6ae014597a89c75e4426715227bbb19f1e98a438
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32189284"
---
# <a name="tutorial-back-up-and-restore-files-for-windows-virtual-machines-in-azure"></a>教學課程：在 Azure 中備份和還原 Windows 虛擬機器的檔案

您可以定期建立備份以保護您的資料。 Azure 備份會建立復原點，並儲存在異地備援復原保存庫。 當您從復原點還原時，可以還原整個 VM 或特定檔案。 本文說明如何將單一檔案還原成執行 Windows Server 和 IIS 的 VM。 如果您還沒有 VM 可以使用，請用 [Windows 快速入門](quick-create-portal.md)中的步驟建立一個。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立 VM 的備份
> * 排定每日備份
> * 從備份還原檔案

## <a name="backup-overview"></a>備份概觀

Azure 備份服務開始備份作業時，會觸發備份擴充功能以建立時間點快照集。 Azure 備份服務使用 VMSnapshot 擴充功能。 如果 VM 正在執行，會在第一次 VM 備份期間安裝此擴充功能。 如果 VM 未在執行中，則備份服務會擷取基礎儲存體的快照集 (因為 VM 停止時不會發生任何應用程式寫入)。

當擷取 Windows VM 的快照集時，備份服務會與磁碟區陰影複製服務 (VSS) 協調，以取得虛擬機器磁碟一致的快照集。 Azure 備份服務擷取快照集之後，資料會傳輸至保存庫。 為了能更有效率，服務只會找出並傳輸自上次備份之後有變更的資料區塊。

資料傳輸完畢後，系統會移除快照集並建立復原點。


## <a name="create-a-backup"></a>建立備份
建立復原服務保存庫的簡單排程每日備份。 

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在左邊的功能表上，選取 [虛擬機器]。 
3. 從清單中選取要備份的 VM。
4. 在 VM 刀鋒視窗的 [設定] 區段中，按一下 [備份]。 [啟用備份] 刀鋒視窗隨即開啟。
5. 在 [復原服務保存庫] 中，按一下 [新建] 並提供新保存庫的名稱。 新保存庫已在與虛擬機器相同的資源群組和位置中建立。
6. 按一下 [備份原則]。 在此範例中請保留預設值，按一下 [確定]。
7. 在 [啟用備份] 刀鋒視窗上，按一下 [啟用備份]。 這會根據預設排程建立每日備份。
10. 若要建立初始復原點，在 [備份] 刀鋒視窗中按一下 [立即備份]。
11. 在 [立即備份] 刀鋒視窗上，按一下日曆圖示，使用日曆控制項選取此復原點保留的最後一天，然後按一下 [備份]。
12. 在 VM 的 [備份] 刀鋒視窗中，您會看到已完成的復原點數目。

    ![復原點](./media/tutorial-backup-vms/backup-complete.png)
    
第一次備份大約需要 20 分鐘的時間。 在備份完成之後，繼續進行本教學課程的下一個部分。

## <a name="recover-a-file"></a>復原檔案

如果您不小心刪除或變更某個檔案，可以使用「檔案復原」從您的備份保存庫復原檔案。 檔案復原使用在 VM 上執行的指令碼，將復原點掛接為本機磁碟機。 這些磁碟機會保持掛接達 12 小時，讓您可以從復原點複製檔案，並將它們還原至 VM。  

在此範例中，我們會示範如何復原用於 IIS 預設網頁的影像檔案。 

1. 開啟瀏覽器，連線到 VM 的 IP 位址，以顯示預設的 IIS 網頁。

    ![預設的 IIS 網頁](./media/tutorial-backup-vms/iis-working.png)

2. 連線至 VM。
3. 在 VM 上開啟 [檔案總管]，瀏覽至 \inetpub\wwwroot，刪除案 iisstart.png 檔。
4. 在您的本機電腦上，重新整理瀏覽器，應該會看到預設 IIS 網頁上的影像已不存在。

    ![預設的 IIS 網頁](./media/tutorial-backup-vms/iis-broken.png)

5. 在您的本機電腦上，開啟新的索引標籤並前往 [Azure 入口網站](https://portal.azure.com)。
6. 在左邊的功能表上，選取 [虛擬機器]，然後從清單中選取虛擬機器。
8. 在 VM 刀鋒視窗的 [設定] 區段中，按一下 [備份]。 [備份] 刀鋒視窗隨即開啟。 
9. 在刀鋒視窗頂端的功能表中，選取 [檔案復原]。 [檔案復原] 刀鋒視窗隨即開啟。
10. 在 [步驟 1︰選取復原點] 中，從下拉式清單選取復原點。
11. 在 [步驟 2：下載指令碼以瀏覽及復原檔案] 中，按一下 [下載執行檔] 按鈕。 將檔案儲存至您的 **Downloads** 資料夾。
12. 在您的本機電腦上，開啟 [檔案總管] 並瀏覽至 **Downloads** 資料夾，複製下載的 .exe 檔案。 檔案名稱前面會加上您的 VM 名稱。 
13. 在您的 VM 上 (透過 RDP 連線)，將 .exe 檔案貼到 VM 桌面上。 
14. 瀏覽至 VM 桌面，按兩下 .exe 檔案。 這會啟動命令提示字元，並將復原點掛接為您可以存取的檔案共用。 當共用建立完成時，輸入 **q** 關閉命令提示字元。
15. 在您的 VM 上，開啟[檔案總管] 並瀏覽至此檔案共用所使用的磁碟機代號。
16. 瀏覽到 \inetpub\wwwroot，複製檔案共用中的 **iisstart.png**，並將它貼到 \inetpub\wwwroot。 例如，複製 F:\inetpub\wwwroot\iisstart.png，並將其貼到 c:\inetpub\wwwroot 以復原檔案。
17. 在您的本機電腦上開啟瀏覽器索引標籤，您已連線到顯示 IIS 預設網頁之 VM 的 IP 位址。 按 CTRL + F5 重新整理瀏覽器頁面。 您現在應該會看到已還原的影像。
18. 在您的本機電腦上，回到 Azure 入口網站的瀏覽器索引標籤，並在 [步驟 3︰在復原後取消掛接磁碟] 按一下 [取消掛接磁碟] 按鈕。 如果您忘記執行此步驟，與此掛接點的連線會在 12 小時後自動關閉。 在這 12 小時後，您必須下載新的指令碼來建立新的掛接點。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立 VM 的備份
> * 排定每日備份
> * 從備份還原檔案

請前進到下一個教學課程，了解如何監視虛擬機器。

> [!div class="nextstepaction"]
> [管理虛擬機器](tutorial-govern-resources.md)









