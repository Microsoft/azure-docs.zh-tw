---
title: 將資料複製到您的 Microsoft Azure 資料箱磁碟 | Microsoft Docs
description: 使用本教學課程以了解如何將資料複製到您的 Azure 資料箱磁碟
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: tutorial
ms.date: 10/09/2018
ms.author: alkohli
Customer intent: As an IT admin, I need to be able to order Data Box Disk to upload on-premises data from my server onto Azure.
ms.openlocfilehash: 7bc8b3ba415f8fe701098a9fa7e51d60ffb9df4e
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092448"
---
# <a name="tutorial-copy-data-to-azure-data-box-disk-and-verify"></a>教學課程：將資料複製到 Azure 資料箱磁碟並確認

本教學課程說明如何從主機電腦複製資料，然後產生總和檢查碼來確認資料完整性。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 將資料複製到資料箱磁碟
> * 驗證資料

## <a name="prerequisites"></a>必要條件

在您開始前，請確定：
- 您已完成[教學課程：安裝及設定您的 Azure 資料箱磁碟](data-box-disk-deploy-set-up.md)。
- 您的磁碟已解除鎖定，並連線到用戶端電腦。
- 您用來將資料複製到磁碟的用戶端電腦必須執行[支援的作業系統](data-box-disk-system-requirements.md)。


## <a name="copy-data-to-disks"></a>將資料複製到磁碟

執行下列步驟以從電腦連線到資料箱磁碟，並且複製資料。

1. 檢視已解除鎖定磁碟機的內容。 

    ![檢視磁碟機內容](media/data-box-disk-deploy-copy-data/data-box-disk-content.png)
 
2. 將需要以區塊 Blob 形式匯入的資料複製到 BlockBlob 資料夾。 同樣地，將 VHD/VHDX 等資料複製到 PageBlob 資料夾。 

    系統會在 Azure 儲存體帳戶中為 BlockBlob 和 PageBlob 資料夾下的每個子資料夾建立容器。 BlockBlob 和 PageBlob 資料夾下的所有檔案，都會複製到 Azure 儲存體帳戶下的預設容器 `$root`。 `$root` 容器中的任何檔案一律會上傳為區塊 Blob。

    如果根目錄中有檔案和資料夾存在，則您必須在開始複製資料之前，將這些項目移到不同的資料夾。

    針對容器和 Blob 名稱請遵循 Azure 命名需求。

    #### <a name="azure-naming-conventions-for-container-and-blob-names"></a>容器和 Blob 名稱的 Azure 命名慣例
    |實體   |慣例  |
    |---------|---------|
    |容器會為區塊 Blob 和分頁 Blob 命名     |必須以字母或數字開頭，且只能包含小寫字母、數字和連字號 (-)。 每個連字號 (-) 字元的前面和後面必須緊接著字母或數字。 名稱中不允許連續的連字號。 <br>必須是有效的 DNS 名稱，也就是 3 到 63 個字元的長度。          |
    |區塊 Blob 和分頁 Blob 的 Blob 名稱    |Blob 名稱會區分大小寫，而且可以包含字元的任意組合。 <br>Blob 名稱長度必須介於 1 到 1,024 個字元之間。<br>保留的 URL 字元必須正確逸出。<br>構成 Blob 名稱的路徑區段數目不可超過 254 個。 路徑線段是連續分隔符號字元 (例如，正斜線 '/') 之間的字串，會對應到虛擬目錄的名稱。         |

    > [!IMPORTANT] 
    > 所有容器和 Blob 都應符合 [Azure 命名慣例](data-box-disk-limits.md#azure-block-blob-and-page-blob-naming-conventions)。 如果未遵循這些規則，則將資料上傳至 Azure 會失敗。

3. 複製檔案時，請確定檔案的區塊 Blob 不超過約 4.7 TB，而分頁 Blob 不超過約 8 TB。 
4. 您可以使用檔案總管的拖放功能來複製資料。 您也可以使用任何 SMB 相容的檔案複製工具 (例如 Robocopy) 來複製資料。 可以使用下列 Robocopy 命令來起始多個複製作業：

    `Robocopy <source> <destination>  * /MT:64 /E /R:1 /W:1 /NFL /NDL /FFT /Log:c:\RobocopyLog.txt` 
    
    命令的參數和選項會以表格形式顯示，如下所示：
    
    |參數/選項  |說明 |
    |--------------------|------------|
    |來源            | 指定來源目錄的路徑。        |
    |目的地       | 指定目的地目錄的路徑。        |
    |/E                  | 複製子目錄，包含空的目錄。 |
    |/MT[:N]             | 建立具有 N 個執行緒的多執行緒複製，其中 N 是介於 1 到 128 之間的整數。 <br>N 的預設值為 8。        |
    |/R: <N>             | 指定失敗複製的重試次數。 N 的預設值為 1000000 (1 百萬次重試)。        |
    |/W: <N>             | 指定重試之間的等待時間 (以秒為單位)。 N 的預設值為 30 (等候時間 30 秒)。        |
    |/NFL                | 指定不會記錄檔案名稱。        |
    |/NDL                | 指定不會記錄目錄名稱。        |
    |/FFT                | 假設 FAT 檔案時間 (兩秒精確度)。        |
    |/Log:<Log File>     | 將狀態輸出寫入至記錄檔 (覆寫現有記錄檔)。         |

    您可採平行方式使用多個磁碟，在每個磁碟上執行多項作業。 

6. 當作業正在進行中時，檢查複製狀態。 下列範例顯示將檔案複製到資料箱磁碟的 robocopy 命令輸出。

    ```
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
    
    C:\Users>Robocopy C:\Git\azure-docs-pr\contributor-guide \\10.126.76.172\devicemanagertest1_AzFile\templates /MT:64 /E /R:1 /W:1 /FFT 
    -------------------------------------------------------------------------------
       ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
    
      Started : Thursday, March 8, 2018 2:34:58 PM
       Source : C:\Git\azure-docs-pr\contributor-guide\
         Dest : \\10.126.76.172\devicemanagertest1_AzFile\templates\
    
        Files : *.*
    
      Options : *.* /DCOPY:DA /COPY:DAT /MT:8 /R:1000000 /W:30
    
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
       Times :   0:00:05   0:00:00                       0:00:00   0:00:00
        
       Speed :                5620 Bytes/sec.
       Speed :               0.321 MegaBytes/min.
       Ended : Thursday, March 8, 2018 2:34:59 PM
        
    C:\Users>
    ```
 
    
7. 開啟目標資料夾，以檢視並確認已複製的檔案。 如果您在複製程序期間遇到任何錯誤，請下載記錄檔以進行疑難排解。 記錄檔位於 robobopy 命令所指定的位置。
 


> [!IMPORTANT]
> - 您必須負責確保將資料複製到資料夾中，該資料夾對應至適當的資料格式。 例如，將區塊 Blob 資料複製到區塊 Blob 的資料夾。 如果資料格式與適當資料夾 (儲存體類型) 不相符，則在稍後步驟中，資料上傳至 Azure 會失敗。
> -  複製資料時，請確定資料大小符合 [Azure 儲存體和資料箱磁碟限制](data-box-disk-limits.md)中所述的大小限制。 
> - 如果資料 (由資料箱磁碟上傳) 同時由資料箱磁碟以外的其他應用程式上傳，則可能導致上傳作業失敗和資料損毀。

### <a name="split-and-copy-data-to-disks"></a>將資料分割並複製到磁碟

如果您使用多個磁碟，而且具有必須加以分割並複製到所有磁碟上的大型資料集，則您可能會用到此選擇性程序。 「資料箱分割複製」工具可協助您分割和複製 Windows 電腦上的資料。

1. 在您的 Windows 電腦上，請確定已下載「資料箱分割複製」工具，並且已在本機資料夾中解壓縮。 此工具會在您下載適用於 Windows 的資料箱磁碟工具組時一併下載。
2. 開啟檔案總管。 請記下指派給資料箱磁碟的資料來源磁碟機和磁碟機代號。 

     ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-1.png)
 
3. 識別要複製的來源資料。 例如，在此案例中是：
    - 已識別到下列區塊 Blob 資料。

         ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-2.png)    

    - 已識別到下列分頁 Blob 資料。

         ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-3.png)
 
4. 請移至軟體解壓縮後所在的資料夾。 在該資料夾中，找出 SampleConfig.json 檔案。 這是您可以修改並儲存的唯讀檔案。

   ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-4.png)
 
5. 修改 SampleConfig.json 檔案。
 
    - 提供作業名稱。 這會在資料箱磁碟中建立資料夾，而這最後會在與這些磁碟相關聯的 Azure 儲存體帳戶中成為容器。 作業名稱必須遵循 Azure 容器的命名慣例。 
    - 提供來源路徑並記下 SampleConfigFile.json 中的路徑格式。 
    - 輸入對應到目標磁碟的磁碟機代號。 資料會取自來源路徑，並複製到多個磁碟。
    - 提供記錄檔的路徑。 根據預設，這會傳送到 .exe 目前所在的目錄。

     ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-5.png)

6. 若要驗證檔案格式，請移至 JSONlint。 將檔案儲存為 ConfigFile.json。 

     ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-6.png)
 
7. 開啟命令提示字元視窗。 

8. 執行 DataBoxDiskSplitCopy.exe。 類型

    `DataBoxDiskSplitCopy.exe PrepImport /config:<Your-config-file-name.json>`

     ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-7.png)
 
9. 按 Entert 後繼續執行指令碼。

    ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-8.png)
  
10. 分割和複製資料集後，複製工作階段的分割複製工具摘要會隨即顯示。 下方顯示一項範例輸出。

    ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-9.png)
 
11. 請確認資料分散在目標磁碟上。 
 
    ![分割複製資料](media/data-box-disk-deploy-copy-data/split-copy-10.png)
    ![分割複製資料](media/data-box-disk-deploy-copy-data/split-copy-11.png)
     
    如果您進一步檢查 n:磁碟機的內容，您會看到對應至區塊 Blob 和分頁 Blob 格式資料的兩個子資料夾已建立。
    
     ![分割複製資料 ](media/data-box-disk-deploy-copy-data/split-copy-12.png)

12. 如果複製工作階段動作失敗，則使用下列命令來復原並繼續：

    `DataBoxDiskSplitCopy.exe PrepImport /config:<configFile.json> /ResumeSession`


資料複製完成之後，下一個步驟是驗證資料。 


## <a name="validate-data"></a>驗證資料 

若要確認資料，請執行下列步驟。

1. 在磁碟機的 AzureImportExport 資料夾中執行 `DataBoxDiskValidation.cmd` 以驗證總和檢查碼。 
    
    ![資料箱磁碟驗證工具的輸出](media/data-box-disk-deploy-copy-data/data-box-disk-validation-tool-output.png)

2. 請選擇適當的選項。 **我們建議您選取選項 2，一律驗證檔案並產生總和檢查碼**。 視您的資料大小而定，此步驟可能需要一段時間。 指令碼完成之後，結束命令視窗。 如果在驗證和產生總和檢查碼期間發生任何錯誤，您會收到通知及錯誤記錄檔的連結。

    ![總和檢查碼輸出](media/data-box-disk-deploy-copy-data/data-box-disk-checksum-output.png)

    > [!TIP]
    > - 請在兩次的執行之間重設工具。
    > - 只在處理包含小檔案 (~KB) 的大型資料集時，使用選項 1 來驗證檔案。 在這些情況下，產生總和檢查碼可能需要很長的時間，而且效能可能會變得非常慢。

3. 如果使用了多個磁碟，請對每個磁碟執行命令。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Azure 資料箱磁碟的相關主題，像是：

> [!div class="checklist"]
> * 將資料複製到資料箱磁碟
> * 確認資料完整性

請前進到下一個教學課程，以了解如何寄回資料箱磁碟，並確認資料上傳至 Azure。

> [!div class="nextstepaction"]
> [將您的 Azure 資料箱寄回給 Microsoft](./data-box-disk-deploy-picked-up.md)

