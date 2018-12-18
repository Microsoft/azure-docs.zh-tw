---
title: 將資料複製到您的 Microsoft Azure 資料箱 | Microsoft Docs
description: 了解如何將資料複製到您的 Azure 資料箱
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 10/10/2018
ms.author: alkohli
ms.openlocfilehash: b59830677ac8c07c6b7adbab24c82ca25d71f5a0
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093454"
---
# <a name="tutorial-copy-data-to-azure-data-box"></a>教學課程：將資料複製到 Azure 資料箱 

本教學課程說明如何使用本機 Web UI 連接主機電腦並從中複製資料，然後進行寄送資料箱的準備工作。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 連線至資料箱
> * 將資料複製到資料箱
> * 準備寄送資料箱。

## <a name="prerequisites"></a>必要條件

在您開始前，請確定：

1. 您已完成[教學課程：設定 Azure 資料箱](data-box-deploy-set-up.md)。
2. 您已收到資料箱，且入口網站中的訂購狀態為 [已交付]。
3. 您有一部主機電腦，其中包含要複製到資料箱的資料。 您的主機電腦必須符合下列條件：
    - 執行[支援的作業系統](data-box-system-requirements.md)。
    - 連線至高速網路。 強烈建議您具有至少一個 10 GbE 的連線。 如果無法使用 10 GbE 連線，也可以使用 1 GbE 資料連結，但是複製速度會受到影響。 

## <a name="connect-to-data-box"></a>連線至資料箱

根據選取的儲存體帳戶，資料箱最多會建立：
- 每個相關聯的儲存體帳戶三個共用 (針對 GPv1 與 GPv2)。
- 進階或 Blob 儲存體帳戶的一個共用。 

在區塊 Blob 與分頁 Blob 共用底下，第一層實體是容器，而第二層實體是 Blob。 在 Azure 檔案共用底下，第一層實體是共用，第二層實體是檔案。

請思考一下下列範例。 

- 儲存體帳戶：*Mystoracct*
- 區塊 Blob 的共用：*Mystoracct_BlockBlob/my-container/blob*
- 分頁 Blob 的共用：*Mystoracct_PageBlob/my-container/blob*
- 檔案的共用：*Mystoracct_AzFile/my-share*

根據您的資料箱是連線至 Windows Server 主機電腦還是 Linux 主機，連線和複製的步驟可能會不同。

### <a name="connect-via-smb"></a>透過 SMB 連線 

如果您使用 Windows Server 主機電腦，請執行下列步驟來連線至資料箱。

1. 第一個步驟是驗證並啟動工作階段。 移至 [連線並複製]。 按一下 [取得認證]，以取得您儲存體帳戶相關共用的存取認證。 

    ![取得共用認證 1](media/data-box-deploy-copy-data/get-share-credentials1.png)

2. 在 [存取共用及複製資料] 對話方塊中，複製與共用相對應的 [使用者名稱] 和 [密碼]。 按一下 [確定]。
    
    ![取得共用認證 1](media/data-box-deploy-copy-data/get-share-credentials2.png)

3. 存取與您的儲存體帳戶相關聯的共用 (在下列範例中為 Mystoracct)。 使用 `\\<IP of the device>\ShareName` 路徑存取共用。 根據您的資料格式，連線至位於下列位址的共用 (使用共用名稱)： 
    - *\\<IP address of the device>\Mystoracct_Blob*
    - *\\<IP address of the device>\Mystoracct_Page*
    - *\\<IP address of the device>\Mystoracct_AzFile*
    
    若要從主機電腦連線至共用，請開啟命令視窗。 在命令提示字元中，輸入：

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

    出現提示時，請輸入共用的密碼。 下列範例說明如何透過前述命令連線至共用。

    ```
    C:\Users\Databoxuser>net use \\10.126.76.172\devicemanagertest1_BlockBlob /u:devicemanagertest1
    Enter the password for 'devicemanagertest1' to connect to '10.126.76.172':
    The command completed successfully.
    ```

4. 按 Windows + R。在 [執行] 視窗中，指定 `\\<device IP address>`。 按一下 [確定]。 這會開啟 [檔案總管]。 此時，您應可看到以資料夾形式呈現的共用。
    
    ![透過檔案總管 2 連線至共用](media/data-box-deploy-copy-data/connect-shares-file-explorer1.png)

5.  **一律針對您想複製的檔案，在共用之下建立一個資料夾，然後將檔案複製到該資料夾**。 這些資料夾有時候可能會顯示灰色叉號。 此叉號不代表錯誤狀況。 應用程式為這些資料夾加上要追蹤狀態的旗標。
    
    ![透過檔案總管 2 連線至共用](media/data-box-deploy-copy-data/connect-shares-file-explorer2.png) ![透過檔案總管 2 連線至共用](media/data-box-deploy-copy-data/connect-shares-file-explorer2.png) 

### <a name="connect-via-nfs"></a>透過 NFS 連線 

如果您使用 Linux 主機電腦，請執行下列步驟設定資料箱，以允許 NFS 用戶端的存取。

1. 針對允許存取共用的用戶端提供其 IP 位址。 在本機 Web UI 中，移至 [連線並複製] 頁面。 在 [NFS 設定] 下方，按一下 [NFS 用戶端存取]。 

    ![設定 NFS 用戶端存取 1](media/data-box-deploy-copy-data/nfs-client-access.png)

2. 提供 NFS 用戶端的 IP 位址，然後按一下 [新增]。 您可以重複此步驟，以設定多個 NFS 用戶端的存取。 按一下 [確定]。

    ![設定 NFS 用戶端存取 2](media/data-box-deploy-copy-data/nfs-client-access2.png)

2. 確定 Linux 主機電腦已安裝NFS 用戶端的[支援版本](data-box-system-requirements.md)。 針對您的 Linux 發行版本，請使用特定版本。 

3. 安裝 NFS 用戶端之後，請使用下列命令將 NFS 共用掛接在您的資料箱裝置上：

    `sudo mount <Data Box device IP>:/<NFS share on Data Box device> <Path to the folder on local Linux computer>`

    下列範例說明如何透過 NFS 連線至資料箱共用。 資料箱裝置 IP 為 `10.161.23.130`，共用 `Mystoracct_Blob` 會掛接在 ubuntuVM 上，掛接點為 `/home/databoxubuntuhost/databox`。

    `sudo mount -t nfs 10.161.23.130:/Mystoracct_Blob /home/databoxubuntuhost/databox`


## <a name="copy-data-to-data-box"></a>將資料複製到資料箱

連線至資料箱共用後，下一個步驟是複製資料。 在複製資料之前，請務必檢閱下列注意事項：

- 確實將資料複製到與適當資料格式相對應的共用。 例如，將區塊 Blob 資料複製到區塊 Blob 的共用。 如果資料格式不符合適當的共用類型，則在稍後步驟中，資料上傳至 Azure 的作業將會失敗。
-  複製資料時，請確定資料大小符合 [Azure 儲存體和資料箱限制](data-box-limits.md)中所述的大小限制。 
- 如果資料 (由資料箱上傳) 同時由資料箱以外的其他應用程式上傳，則可能導致上傳作業失敗和資料損毀。
- 建議您不要同時使用 SMB 與 NFS，或將相同的資料複製到 Azure 上的相同最終目的地。 在這種情況下，無法判斷最後的結果。

### <a name="copy-data-via-smb"></a>透過 SMB 複製資料

在連線至 SMB 共用後，請起始資料複製。 

您可以使用任何與 SMB 相容的檔案複製工具 (例如 Robocopy) 來複製資料。 使用 Robocopy 可起始多個複製作業。 使用下列命令：
    
    robocopy <Source> <Target> * /e /r:3 /w:60 /is /nfl /ndl /np /MT:32 or 64 /fft /Log+:<LogFile> 
  
 下表說明這些屬性。
    
|屬性  |說明  |
|---------|---------|
|/e     |複製子目錄，包含空的目錄。         |
|/r:     |指定失敗複製的重試次數。         |
|/w:     |指定重試之間的等待時間 (以秒為單位)。         |
|/is     |包含相同的檔案。         |
|/nfl     |指定不會記錄檔案名稱。         |
|/ndl    |指定不會記錄目錄名稱。        |
|/np     |指定將不會顯示複製作業的進度 (到目前為止複製的檔案或目錄數目)。 顯示進度會大幅降低效能。         |
|/MT     | 使用多執行緒，建議使用 32 或 64 個執行緒。 此選項無法用於加密檔案。 您可能需要區分加密和未加密的檔案。 不過，單一執行緒複製會大幅降低效能。           |
|/fft     | 用以降低任何檔案系統的時間戳記細微性。        |
|/b     | 以備份模式複製檔案。        |
|/z    | 以重新啟動模式複製檔案；如果環境不穩定，請使用此選項。 此選項會因為額外的記錄而降低輸送量。      |
| /zb     | 使用重新啟動模式。 如果存取遭拒，此選項會使用備份模式。 此選項會因為執行檢查點而降低輸送量。         |
|/efsraw     | 以 EFS 原始模式複製所有加密檔案。 這只適用於加密檔案。         |
|log+:<LogFile>| 將輸出附加至現有的記錄檔。|    
 
下列範例顯示將檔案複製到資料箱的 robocopy 命令輸出。
    
    C:\Users>robocopy
        -------------------------------------------------------------------------------
        ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
    
        Started : Thursday, March 8, 2018 2:34:53 PM
            Simple Usage :: ROBOCOPY source destination /MIR
    
                    source :: Source Directory (drive:\path or \\server\share\path).
            destination :: Destination Dir  (drive:\path or \\server\share\path).
                    /MIR :: Mirror a complete directory tree.
    
        For more usage information run ROBOCOPY /?    
    
    ****  /MIR can DELETE files as well as copy them !
    
    C:\Users>Robocopy C:\Git\azure-docs-pr\contributor-guide \\10.126.76.172\devicemanagertest1_AzFile\templates /MT:32
    -------------------------------------------------------------------------------
        ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
    
        Started : Thursday, March 8, 2018 2:34:58 PM
        Source : C:\Git\azure-docs-pr\contributor-guide\
            Dest : \\10.126.76.172\devicemanagertest1_AzFile\templates\
    
        Files : *.*
    
        Options : *.* /DCOPY:DA /COPY:DAT /MT:32 /R:5 /W:60
    
    ------------------------------------------------------------------------------
    
    100%        New File                 206        C:\Git\azure-docs-pr\contributor-guide\article-metadata.md
    100%        New File                 209        C:\Git\azure-docs-pr\contributor-guide\content-channel-guidance.md
    100%        New File                 732        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-index.md
    100%        New File                 199        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pr-criteria.md
                New File                 178        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pull-request-co100%  .md
                New File                 250        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pull-request-et100%  e.md
    100%        New File                 174        C:\Git\azure-docs-pr\contributor-guide\create-images-markdown.md
    100%        New File                 197        C:\Git\azure-docs-pr\contributor-guide\create-links-markdown.md
    100%        New File                 184        C:\Git\azure-docs-pr\contributor-guide\create-tables-markdown.md
    100%        New File                 208        C:\Git\azure-docs-pr\contributor-guide\custom-markdown-extensions.md
    100%        New File                 210        C:\Git\azure-docs-pr\contributor-guide\file-names-and-locations.md
    100%        New File                 234        C:\Git\azure-docs-pr\contributor-guide\git-commands-for-master.md
    100%        New File                 186        C:\Git\azure-docs-pr\contributor-guide\release-branches.md
    100%        New File                 240        C:\Git\azure-docs-pr\contributor-guide\retire-or-rename-an-article.md
    100%        New File                 215        C:\Git\azure-docs-pr\contributor-guide\style-and-voice.md
    100%        New File                 212        C:\Git\azure-docs-pr\contributor-guide\syntax-highlighting-markdown.md
    100%        New File                 207        C:\Git\azure-docs-pr\contributor-guide\tools-and-setup.md
    ------------------------------------------------------------------------------
    
                    Total    Copied   Skipped  Mismatch    FAILED    Extras
        Dirs :         1         1         1         0         0         0
        Files :        17        17         0         0         0         0
        Bytes :     3.9 k     3.9 k         0         0         0         0          
    C:\Users>
       

若要將效能最佳化，複製資料時請使用下列 robocopy 參數。

|    平台    |    大部分是 < 512 KB 的小檔案                           |    大部分是 512 KB - 1 MB 的中型檔案                      |    大部分是 > 1 MB 的大檔案                             |   
|----------------|--------------------------------------------------------|--------------------------------------------------------|--------------------------------------------------------|---|
|    資料箱         |    2 個 Robocopy 工作階段 <br> 每個工作階段 16 個執行緒    |    3 個 Robocopy 工作階段 <br> 每個工作階段 16 個執行緒    |    2 個 Robocopy 工作階段 <br> 每個工作階段 24 個執行緒    |  |


如需 Robocopy 命令的詳細資訊，請移至 [Robocopy 和數個範例](https://social.technet.microsoft.com/wiki/contents/articles/1073.robocopy-and-a-few-examples.aspx)。

開啟目標資料夾，以檢視並確認已複製的檔案。 如果您在複製程序期間遇到任何錯誤，請下載錯誤檔以進行疑難排解。

為了確保資料完整性，在複製資料時會計算內嵌的總和檢查碼。 複製完成後，請確認您的裝置上已使用的空間和可用空間。
    
   ![確認儀表板上的可用空間和已使用的空間](media/data-box-deploy-copy-data/verify-used-space-dashboard.png)

### <a name="copy-data-via-nfs"></a>透過 NFS 複製資料

如果您使用 Linux 主機電腦，請使用類似於 Robocopy 的複製公用程式。 在 Linux 中有替代工具可供使用，包括 [rsync](https://rsync.samba.org/)、[FreeFileSync](https://www.freefilesync.org/)、[Unison](https://www.cis.upenn.edu/~bcpierce/unison/) 或 [Ultracopier](https://ultracopier.first-world.info/)。  

`cp` 命令是複製目錄的最佳選項之一。 如需其使用方式的詳細資訊，請移至 [cp 手冊頁](http://man7.org/linux/man-pages/man1/cp.1.html)。

如果使用 rsync 選項進行多執行緒複製，請遵循下列指導方針：

 - 根據您的 Linux 用戶端所使用的檔案系統，安裝 **CIFS Utils** 或 **NFS Utils** 套件。

    `sudo apt-get install cifs-utils`

    `sudo apt-get install nfs-utils`

 -  安裝 **Rsync** 和 **Parallel** (依 Linux 發行的版本而有所不同)。

    `sudo apt-get install rsync`
   
    `sudo apt-get install parallel` 

 - 建立掛接點。

    `sudo mkdir /mnt/databox`

 - 掛接磁碟區。

    `sudo mount -t NFS4  //Databox IP Address/share_name /mnt/databox` 

 - 鏡像資料夾目錄結構。  

    `rsync -za --include='*/' --exclude='*' /local_path/ /mnt/databox`

 - 複製檔案。 

    `cd /local_path/; find -L . -type f | parallel -j X rsync -za {} /mnt/databox/{}`

     其中，j 會指定平行處理數目，X = 平行複製數目

     建議您一開始先使用 16 個平行複製，然後再根據可用的資源增加執行緒數目。

## <a name="prepare-to-ship"></a>準備寄送

最後一個步驟是準備寄送裝置。 在此步驟中，裝置的所有共用都會離線。 在您開始準備寄送裝置後，即無法存取共用。
1. 移至 [準備寄送]，然後按一下 [開始準備]。 
   
    ![準備寄送 1](media/data-box-deploy-copy-data/prepare-to-ship1.png)

2. 根據預設，在準備寄送期間會計算內嵌的總和檢查碼。 視您的資料大小而定，計算總和檢查碼可能需要一些時間。 按一下 [開始準備]。
    1. 在我們準備寄送時，裝置共用會離線，且裝置會鎖定。
        
        ![準備寄送 1](media/data-box-deploy-copy-data/prepare-to-ship2.png) 
   
    2. 當裝置準備工作完成後，裝置狀態即會更新為 [已準備好寄送]。 
        
        ![準備寄送 1](media/data-box-deploy-copy-data/prepare-to-ship3.png)

    3. 下載在此程序中複製的檔案清單 (資訊清單)。 您稍後可以使用這份清單來確認上傳至 Azure 的檔案。
        
        ![準備寄送 1](media/data-box-deploy-copy-data/prepare-to-ship4.png)

3. 關閉裝置。 移至 [關機或重新啟動] 頁面，然後按一下 [關閉]。 系統提示您進行確認時，請按一下 [確定] 繼續作業。
4. 拔除纜線。 下一個步驟是將裝置寄回給 Microsoft。

 
<!--## Appendix - robocopy parameters

This section describes the robocopy parameters used when copying the data to optimize the performance.

|    Platform    |    Mostly small files < 512 KB                           |    Mostly medium  files 512 KB-1 MB                      |    Mostly large files > 1 MB                             |   
|----------------|--------------------------------------------------------|--------------------------------------------------------|--------------------------------------------------------|---|
|    Data Box         |    2 Robocopy sessions <br> 16 threads per sessions    |    3 Robocopy sessions <br> 16 threads per sessions    |    2 Robocopy sessions <br> 24 threads per sessions    |  |
|    Data Box Heavy     |    6 Robocopy sessions <br> 24 threads per sessions    |    6 Robocopy sessions <br> 16 threads per sessions    |    6 Robocopy sessions <br> 16 threads per sessions    |   
|    Data Box Disk         |    4 Robocopy sessions <br> 16 threads per sessions             |    2 Robocopy sessions <br> 16 threads per sessions    |    2 Robocopy sessions <br> 16 threads per sessions    |   
-->

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Azure 資料箱的相關主題，像是：

> [!div class="checklist"]
> * 連線至資料箱
> * 將資料複製到資料箱
> * 準備寄送資料箱

請繼續進行下一個教學課程，以了解如何在資料箱上進行設定和複製資料。

> [!div class="nextstepaction"]
> [將您的 Azure 資料箱寄送給 Microsoft](./data-box-deploy-picked-up.md)

