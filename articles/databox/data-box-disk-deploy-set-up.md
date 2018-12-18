---
title: 設定 Microsoft Azure 資料箱磁碟 | Microsoft Docs
description: 使用本教學課程以了解如何設定您的 Azure 資料箱磁碟
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: alkohli
Customer intent: As an IT admin, I need to be able to order Data Box Disk to upload on-premises data from my server onto Azure.
ms.openlocfilehash: 24de05a60c3cda47a1968bb1f83f9cb91e4e03de
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49091193"
---
# <a name="tutorial-unpack-connect-and-unlock-azure-data-box-disk"></a>教學課程：針對 Azure 資料箱磁碟打開包裝、連線然後解除鎖定

本教學課程說明如何針對 Azure 資料箱磁碟打開包裝、連線然後解除鎖定。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 打開資料箱磁碟的包裝
> * 連接到磁碟並取得通行金鑰
> * 將 Windows 用戶端上的磁碟解除鎖定
> * 將 Linux 用戶端上的磁碟解除鎖定

## <a name="prerequisites"></a>必要條件

在您開始前，請確定：

1. 您已完成[教學課程：訂購 Azure 資料箱磁碟](data-box-disk-deploy-ordered.md)。
2. 您已收到磁碟，入口網站中的作業狀態會更新為**已交付**。
3. 您有一部用戶端電腦，可以在上面安裝資料箱磁碟解除鎖定工具。 您的用戶端電腦必須：
    - 執行[支援的作業系統](data-box-disk-system-requirements.md#supported-operating-systems-for-clients)。
    - 安裝其他[必要軟體](data-box-disk-system-requirements.md#other-required-software-for-windows-clients) (如果是 Windows 用戶端)。  

## <a name="unpack-your-disks"></a>打開磁碟的包裝

 執行下列步驟以打開磁碟的包裝。

1. 資料箱磁碟是以小型包裹寄出。 請開啟包裹然後取出內容物。 檢查包裹裡每個磁碟是否有 1 到 5 個固態硬碟 (SSD) 和 USB 連接線。 檢查包裹是否有任何竄改的證據，或其他任何明顯的損毀。 

    ![資料箱磁碟寄送包裹](media/data-box-disk-deploy-set-up/data-box-disk-ship-package1.png)

2. 如果包裹已遭竄改或嚴重損毀，請勿開啟包裹。 請連絡 Microsoft 支援服務以協助您評估磁碟是否可正常運作，以及是否需要寄送替代品給您。
3. 確認包裹有明顯外盒，其中包含用來退貨的出貨標籤 (在目前的標籤底下)。 如果此標籤遺失或損毀，您隨時都可以從 Azure 入口網站下載並列印新標籤。 

    ![資料箱磁碟出貨標籤](media/data-box-disk-deploy-set-up/data-box-disk-package-ship-label.png)

4. 請保留包裹及包裝發泡材料，以便將磁碟退回。

## <a name="connect-to-disks-and-get-the-passkey"></a>連接到磁碟並取得通行金鑰 

1. 使用內含的纜線將磁碟連接到執行支援作業系統的用戶端電腦，如同必要條件中所述。 

    ![資料箱磁碟連線](media/data-box-disk-deploy-set-up/data-box-disk-connect-unlock.png)    
    
2. 在 Azure 入口網站中，移至 [一般] > [裝置詳細資料]。 使用複製圖示來複製通行金鑰。 此通行金鑰用來將磁碟解除鎖定。

    ![資料箱磁碟解除鎖定通行金鑰](media/data-box-disk-deploy-set-up/data-box-disk-get-passkey.png) 

視您連線至 Windows 或 Linux 用戶端，用來解除鎖定磁碟的步驟會有所不同。

## <a name="unlock-disks-on-windows-client"></a>將 Windows 用戶端上的磁碟解除鎖定

請執行下列步驟，以連接磁碟並解除鎖定。
     
1. 在 Azure 入口網站中，移至 [一般] > [裝置詳細資料]。 
2. 下載對應至 Windows 用戶端的資料箱磁碟工具組。 此工具組包含 3 個工具：資料箱磁碟解除鎖定工具、資料箱磁碟驗證工具及資料箱磁碟分割複製工具。 

    在此程序中，您將只使用資料箱磁碟解除鎖定工具。 其他兩個工具會在稍後使用。

    > [!div class="nextstepaction"]
    > [下載適用於 Windows 的資料箱磁碟工具組](http://aka.ms/databoxdisktoolswin)         

3. 在您用來複製資料的相同電腦上將工具組解壓縮。 
4. 在相同電腦上以系統管理員的身分，開啟 [命令提示字元] 視窗或執行 Windows PowerShell。
5. (選擇性) 若要確認您用來將磁碟解除鎖定的電腦符合作業系統需求，請執行系統檢查命令。 下方顯示一項範例輸出。 

    ```powershell
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.
    
    PS C:\DataBoxDiskUnlockTool\DiskUnlock> .\DataBoxDiskUnlock.exe /SystemCheck
    Successfully verified that the system can run the tool.
    PS C:\DataBoxDiskUnlockTool\DiskUnlock>
    ``` 

6. 執行 `DataBoxDiskUnlock.exe` 並提供您在[連接到磁碟並取得通行金鑰](#Connect-to-disks-and-get-the-passkey)中取得的通行金鑰。 指派給磁碟的磁碟機代號隨即顯示。 下方顯示一項範例輸出。

    ```powershell
    PS C:\WINDOWS\system32> cd C:\DataBoxDiskUnlockTool\DiskUnlock
    PS C:\DataBoxDiskUnlockTool\DiskUnlock> .\DataBoxDiskUnlock.exe
    Enter the passkey :
    testpasskey1
    
    Following volumes are unlocked and verified.
    Volume drive letters: D:
    
    PS C:\DataBoxDiskUnlockTool\DiskUnlock>
    ```

7. 請針對日後任何磁碟重新插入重複解除鎖定步驟。 如果您需要資料箱磁碟解除鎖定工具的說明，請使 `help` 命令。   

    ```powershell
    PS C:\DataBoxDiskUnlockTool\DiskUnlock> .\DataBoxDiskUnlock.exe /help
    USAGE:
    DataBoxUnlock /PassKey:<passkey_from_Azure_portal>
    
    Example: DataBoxUnlock /PassKey:<your passkey>
    Example: DataBoxUnlock /SystemCheck
    Example: DataBoxUnlock /Help
    
    /PassKey:        Get this passkey from Azure DataBox Disk order. The passkey unlocks your disks.
    /SystemCheck:    This option checks if your system meets the requirements to run the tool.
    /Help:           This option provides help on cmdlet usage and examples.
    
    PS C:\DataBoxDiskUnlockTool\DiskUnlock>
    ```  
8. 將磁碟解除鎖定之後，您就可以檢視磁碟的內容。    

    ![資料箱磁碟內容](media/data-box-disk-deploy-set-up/data-box-disk-content.png) 

## <a name="unlock-disks-on-linux-client"></a>將 Linux 用戶端上的磁碟解除鎖定

1. 在 Azure 入口網站中，移至 [一般] > [裝置詳細資料]。 
2. 下載對應至 Linux 用戶端的資料箱磁碟工具組。  

    > [!div class="nextstepaction"]
    > [下載適用於 Linux 的資料箱磁碟工具組](http://aka.ms/databoxdisktoolslinux) 

3. 在您的 Linux 用戶端上開啟終端機。 瀏覽至您下載軟體的資料夾。 變更檔案權限，以便執行這些檔案。 輸入以下命令： 

    `chmod +x DataBoxDiskUnlock_x86_64` 
    
    `chmod +x DataBoxDiskUnlock_Prep.sh` 
 
    下方顯示一項範例輸出。 執行 chmod 命令後，您可藉由執行 `ls` 命令來確認檔案權限已變更。 
 
    ```
        [user@localhost Downloads]$ chmod +x DataBoxDiskUnlock_x86_64  
        [user@localhost Downloads]$ chmod +x DataBoxDiskUnlock_Prep.sh   
        [user@localhost Downloads]$ ls -l  
        -rwxrwxr-x. 1 user user 1152664 Aug 10 17:26 DataBoxDiskUnlock_x86_64  
        -rwxrwxr-x. 1 user user 795 Aug 5 23:26 DataBoxDiskUnlock_Prep.sh
    ```
4. 執行指令碼，以便安裝資料箱磁碟解除鎖定軟體需要的所有二進位檔。 使用 `sudo` 以 root 身分執行命令。 成功安裝二進位檔後，您會在終端機上看到該該效果的附註。

    `sudo ./DataBoxDiskUnlock_Prep.sh`

    此指令碼會先檢查您的用戶端電腦是否執行支援的作業系統。 下方顯示一項範例輸出。 
 
    ```
    [user@localhost Documents]$ sudo ./DataBoxDiskUnlock_Prep.sh 
        OS = CentOS Version = 6.9 
        Release = CentOS release 6.9 (Final) 
        Architecture = x64 
    
        The script will install the following packages and dependencies. 
        epel-release 
        dislocker 
        ntfs-3g 
        fuse-dislocker 
        Do you wish to continue? y|n :|
    ```
    
 
5. 輸入 `y` 繼續安裝。 指令碼安裝的套件如下︰ 
    - **epel-release** - 包含下列三個套件的存放庫。 
    - **dislocker 和 fuse-dislocker** - 此公用程式可協助將 BitLocker 加密磁碟解密。 
    - **ntfs-3g** - 可協助掛接 NTFS 磁碟區的套件。 
 
    成功安裝套件後，終端機會顯示該效果的通知。     
    ```
    Dependency Installed: compat-readline5.x86 64 0:5.2-17.I.el6 dislocker-libs.x86 64 0:0.7.1-8.el6 mbedtls.x86 64 0:2.7.4-l.el6        ruby.x86 64 0:1.8.7.374-5.el6 
    ruby-libs.x86 64 0:1.8.7.374-5.el6 
    Complete! 
    Loaded plugins: fastestmirror, refresh-packagekit, security 
    Setting up Remove Process 
    Resolving Dependencies 
    --> Running transaction check 
    ---> Package epel-release.noarch 0:6-8 will be erased --> Finished Dependency Resolution 
    Dependencies Resolved 
    Package        Architecture        Version        Repository        Size 
    Removing:  epel-release        noarch         6-8        @extras        22 k 
    Transaction Summary                                 
    Remove        1 Package(s) 
    Installed size: 22 k 
    Downloading Packages: 
    Running rpmcheckdebug 
    Running Transaction Test 
    Transaction Test Succeeded 
    Running Transaction 
    Erasing : epel-release-6-8.noarch 
    Verifying : epel-release-6-8.noarch 
    Removed: 
    epel-release.noarch 0:6-8 
    Complete! 
    Dislocker is installed by the script. 
    OpenSSL is already installed.
    ```

6. 執行資料箱磁碟解除鎖定工具。 在 Azure 入口網站中提供您在[連接到磁碟並取得通行金鑰](#Connect-to-disks-and-get-the-passkey)中取得的通行金鑰。 選擇性地指定要解除鎖定的 BitLocker 加密磁碟區清單。 通行金鑰和磁碟區清單應該指定於單引號內。 

    輸入下列命令。
 
    `sudo ./DataBoxDiskUnlock_x86_64 /PassKey:’<Your passkey from Azure portal>’ /Volumes:’<list of volumes>’`         

    範例輸出如下所示。 
 
    ```
    [user@localhost Downloads]$ sudo ./DataBoxDiskUnlock_x86_64 /Passkey:’qwerqwerqwer’ /Volumes:’/dev/sdbl’ 
    
    START: Mon Aug 13 14:25:49 2018 
    Volumes: /dev/sdbl 
    Passkey: qwerqwerqwer 
    
    Volumes for data copy : 
    /dev/sdbl: /mnt/DataBoxDisk/mountVoll/ 
    END: Mon Aug 13 14:26:02 2018
    ```
    您可將資料複製到的磁碟區掛接點隨即顯示。

7. 請針對日後任何磁碟重新插入重複解除鎖定步驟。 如果您需要資料箱磁碟解除鎖定工具的說明，請使 `help` 命令。 
    
    `sudo ./DataBoxDiskUnlock_x86_64 /Help` 

    範例輸出如下所示。 
 
    ```
    [user@localhost Downloads]$ sudo ./DataBoxDiskUnlock_x86_64 /Help  
    START: Mon Aug 13 14:29:20 2018 
    USAGE: 
    sudo DataBoxDiskUnlock /PassKey:’<passkey from Azure_portal>’ 
    
    Example: sudo DataBoxDiskUnlock /PassKey:’passkey’ 
    Example: sudo DataBoxDiskUnlock /PassKey:’passkey’ /Volumes:’/dev/sdbl’ 
    Example: sudo DataBoxDiskUnlock /Help Example: sudo DataBoxDiskUnlock /Clean 
    
    /PassKey: This option takes a passkey as input and unlocks all of your disks. 
    Get the passkey from your Data Box Disk order in Azure portal. 
    /Volumes: This option is used to input a list of BitLocker encrypted volumes. 
    /Help: This option provides help on the tool usage and examples. 
    /Unmount: This option unmounts all the volumes mounted by this tool. 
   
    END: Mon Aug 13 14:29:20 2018 [user@localhost Downloads]$
    ```
    
8. 將磁碟解除鎖定之後，您可以移至掛接點並檢視磁碟的內容。 您現在已準備好將資料複製到 *BlockBlob* 或 *PageBlob* 資料夾。 

    ![資料箱磁碟內容](media/data-box-disk-deploy-set-up/data-box-disk-content-linux.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Azure 資料箱磁碟的相關主題，像是：

> [!div class="checklist"]
> * 打開資料箱磁碟的包裝
> * 連接到磁碟並取得通行金鑰
> * 將 Windows 用戶端上的磁碟解除鎖定
> * 將 Linux 用戶端上的磁碟解除鎖定


請前進到下一個教學課程，以了解如何複製資料箱磁碟上的資料。

> [!div class="nextstepaction"]
> [複製資料箱磁碟上的資料](./data-box-disk-deploy-copy-data.md)

