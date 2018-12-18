---
title: 教學課程 - 在 Azure 入口網站中備份 Linux 虛擬機器 | Microsoft Docs
description: 在本教學課程中，您會學習如何使用 Azure 入口網站，以 Azure 備份保護 Linux 虛擬機器。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/27/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: c91e2b1380e5048fa1dfb7a0e028c88e589cbaa4
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32190551"
---
# <a name="tutorial-back-up-and-restore-files-for-linux-virtual-machines-in-azure"></a>教學課程：在 Azure 中備份和還原 Linux 虛擬機器的檔案

您可以定期建立備份以保護您的資料。 Azure 備份會建立復原點，並儲存在異地備援復原保存庫。 當您從復原點還原時，可以還原整個 VM 或特定檔案。 本文說明如何將單一檔案還原成執行 nginx 的 Linux VM。 如果您還沒有 VM 可以使用，請用 [Linux 快速入門](quick-create-cli.md)中的步驟建立一個。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立 VM 的備份
> * 排定每日備份
> * 從備份還原檔案

## <a name="backup-overview"></a>備份概觀

Azure 備份服務開始備份時，會觸發備份擴充功能以建立時間點快照集。 Azure 備份服務在 Linux 中使用 _VMSnapshotLinux_ 擴充功能。 如果 VM 正在執行，會在第一次 VM 備份期間安裝此擴充功能。 如果 VM 未在執行中，則備份服務會擷取基礎儲存體的快照集 (因為 VM 停止時不會發生任何應用程式寫入)。

根據預設，Azure 備份會為 Linux VM 建立檔案系統一致備份，但可以將它設定成建立[應用程式一致備份 (使用備份前指令碼和備份後指令碼架構)](https://docs.microsoft.com/azure/backup/backup-azure-linux-app-consistent)。 Azure 備份服務擷取快照集之後，資料會傳輸至保存庫。 為了能更有效率，服務只會找出並傳輸自上次備份之後有變更的資料區塊。

資料傳輸完畢後，系統會移除快照集並建立復原點。


## <a name="create-a-backup"></a>建立備份
建立復原服務保存庫的排程每日備份：

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

## <a name="restore-a-file"></a>還原檔案

如果您不小心刪除或變更某個檔案，可以使用「檔案復原」從您的備份保存庫復原檔案。 檔案復原使用在虛擬機器上執行的指令碼，將復原點掛接為本機磁碟機。 這些磁碟機會保持掛接達 12 小時，讓您可以從復原點複製檔案，並將它們還原至 VM。  

在此範例中，我們會示範如何復原預設的 nginx 網頁 /var/www/html/index.nginx-debian.html。 我們的 VM 在範例中的公用 IP 位址是 13.69.75.209。 您可以使用以下命令找到您的 VM 的 IP 位址︰

 ```bash 
 az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --o tsv
 ```

 
1. 在您的本機電腦上開啟瀏覽器，輸入在您的 VM 公用 IP 位址，應該可以看到預設的 nginx 網頁。

    ![預設的 nginx 網頁](./media/tutorial-backup-vms/nginx-working.png)

1. 透過 SSH 連線到您的 VM。

    ```bash
    ssh 13.69.75.209
    ```
2. 刪除 /var/www/html/index.nginx-debian.html。

    ```bash
    sudo rm /var/www/html/index.nginx-debian.html
    ```
    
4. 在您的本機電腦上，按 CTRL + F5 重新整理瀏覽器，應該會看到該預設 nginx 網頁已不存在。

    ![預設的 nginx 網頁](./media/tutorial-backup-vms/nginx-broken.png)
    
1. 在您的本機電腦上，登入 [Azure 入口網站](https://portal.azure.com/)。
6. 在左邊的功能表上，選取 [虛擬機器]。 
7. 從清單中選取 VM。
8. 在 VM 刀鋒視窗的 [設定] 區段中，按一下 [備份]。 [備份] 刀鋒視窗隨即開啟。 
9. 在刀鋒視窗頂端的功能表中，選取 [檔案復原]。 [檔案復原] 刀鋒視窗隨即開啟。
10. 在 [步驟 1︰選取復原點] 中，從下拉式清單選取復原點。
11. 在 [步驟 2：下載指令碼以瀏覽及復原檔案] 中，按一下 [下載執行檔] 按鈕。 將下載的檔案儲存到您的本機電腦。
7. 按一下 [下載指令碼] 將指令碼檔案下載至本機。
8. 開啟 Bash 提示字元並輸入下列命令，將其中的 Linux_myVM_05-05-2017.sh 取代為您下載之指令碼的正確路徑和檔名，將 azureuser 取代為 VM 的使用者名稱，將 13.69.75.209 取代為您的 VM 公用 IP 位址。
    
    ```bash
    scp Linux_myVM_05-05-2017.sh azureuser@13.69.75.209:
    ```
    
9. 在您的本機電腦上，開啟與 VM 的 SSH 連線。

    ```bash
    ssh 13.69.75.209
    ```
    
10. 在您的 VM 上，新增指令碼檔案的執行權限。

    ```bash
    chmod +x Linux_myVM_05-05-2017.sh
    ```
    
11. 您的 VM 上，執行指令碼將復原點掛接為檔案系統。

    ```bash
    ./Linux_myVM_05-05-2017.sh
    ```
    
12. 指令碼的輸出會提供掛接點的路徑。 輸出大致如下：

    ```bash
    Microsoft Azure VM Backup - File Recovery
    ______________________________________________
                          
    Connecting to recovery point using ISCSI service...
    
    Connection succeeded!
    
    Please wait while we attach volumes of the recovery point to this machine...
                         
    ************ Volumes of the recovery point and their mount paths on this machine ************

    Sr.No.  |  Disk  |  Volume  |  MountPath 

    1)  | /dev/sdc  |  /dev/sdc1  |  /home/azureuser/myVM-20170505191055/Volume1

    ************ Open File Explorer to browse for files. ************

    After recovery, to remove the disks and close the connection to the recovery point, please click 'Unmount Disks' in step 3 of the portal.

    Please enter 'q/Q' to exit...
    ```

12. 在您的 VM 上，從刪除檔案之際的掛接點複製 nginx 預設網頁。

    ```bash
    sudo cp ~/myVM-20170505191055/Volume1/var/www/html/index.nginx-debian.html /var/www/html/
    ```
    
17. 在您的本機電腦上開啟瀏覽器索引標籤，您已連線到顯示 nginx 預設網頁之 VM 的 IP 位址。 按 CTRL + F5 重新整理瀏覽器頁面。 您現在應該會看到預設網頁再次運作。

    ![預設的 nginx 網頁](./media/tutorial-backup-vms/nginx-working.png)

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

