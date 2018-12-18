---
title: 使用 Azure 自動化 Runbook 管理 StorSimple 裝置 | Microsoft Docs
description: 了解如何使用 Azure 自動化 Runbook 將 StorSimple 作業自動化
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 10/23/2017
ms.author: alkohli
ms.openlocfilehash: cfd0e4dbb6a4f24df5ba42cd45f9c16fbe5b493c
ms.sourcegitcommit: e6029b2994fa5ba82d0ac72b264879c3484e3dd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2017
ms.locfileid: "23493128"
---
# <a name="use-azure-automation-runbooks-to-manage-storsimple-devices"></a>使用 Azure 自動化 Runbook 管理 StorSimple 裝置

本文說明如何使用 Azure 自動化 Runbook 在 Azure 入口網站中管理您的 StorSimple 8000 系列裝置。 其中也包含一個範例 Runbook，逐步引導您完成設定環境以執行此 Runbook。


## <a name="configure-add-and-run-azure-runbook"></a>設定、新增和執行 Azure Runbook

本節採用 StorSimple 的範例 Windows PowerShell 指令碼，並詳細說明各種必要步驟將指令碼匯入 Runbook，然後發佈及執行 Runbook。

### <a name="prerequisites"></a>先決條件

開始之前，請確定您有︰

* 以 StorSimple 8000 系列裝置註冊，與您的 StorSimple 裝置管理員服務建立關聯的作用中 Azure 訂用帳戶。


* Windows PowerShell 5.0 已安裝在您的電腦上 (或者，如果使用，則為 StorSimple 的 Windows Server 主機)。


### <a name="create-automation-runbook-module-in-windows-powershell"></a>在 Windows PowerShell 中建立自動化 Runbook 模組

若要建立 StorSimple 8000 系列裝置管理的自動化模組，請執行下列步驟：

1. 啟動 Windows PowerShell。 建立一個新資料夾，並將目錄切換到新資料夾。

    ```
        mkdir C:\scripts\StorSimpleSDKTools
        cd C:\scripts\StorSimpleSDKTools
    ```    
2. 在上一個步驟中建立的資料夾底下，[下載 NuGet CLI](http://www.nuget.org/downloads)。 有各種版本的 _nuget.exe_。 選擇對應至您 SDK 的版本。 每個下載連結都會直接指向一個 _.exe_ 檔案。 請務必按一下滑鼠右鍵，並將檔案儲存到您的電腦，而不是直接從瀏覽器中執行該檔案。

    您也可以執行下列命令來下載指令碼，並將其儲存在您稍早建立的相同資料夾中。
    
    ```
        wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
    ```
3. 下載相依的 SDK。

    ```
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
    ```    
4. 從範例 GitHub 專案下載指令碼。

    ```
        wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1 -Out Monitor-Backups.ps1

    ```
5. 建立 StorSimple 8000 系列裝置管理的 Azure 自動化 Runbook 模組。 在 Windows PowerShell 視窗中鍵入下列命令：

    ```
            # set path variables
            $downloadDir = "C:\scripts\StorSimpleSDKTools"
            $moduleDir = "$downloadDir\AutomationModule\Microsoft.Azure.Management.StorSimple8000Series"

            #don't change the folder name "Microsoft.Azure.Management.StorSimple8000Series"
            mkdir "$moduleDir"
            copy "$downloadDir\Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\Microsoft.IdentityModel.Clients.ActiveDirectory*.dll" $moduleDir
            copy "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\Microsoft.Rest.ClientRuntime.Azure*.dll" $moduleDir
            copy "$downloadDir\Microsoft.Rest.ClientRuntime.2.3.8\lib\net452\Microsoft.Rest.ClientRuntime*.dll" $moduleDir
            copy "$downloadDir\Newtonsoft.Json.6.0.8\lib\net45\Newtonsoft.Json*.dll" $moduleDir
            copy "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.Authentication.2.2.9-preview\lib\net45\Microsoft.Rest.ClientRuntime.Azure.Authentication*.dll" $moduleDir
            copy "$downloadDir\Microsoft.Azure.Management.Storsimple8000series.1.0.0\lib\net452\Microsoft.Azure.Management.Storsimple8000series*.dll" $moduleDir

            #Don't change the name of the Archive
            compress-Archive -Path "$moduleDir" -DestinationPath Microsoft.Azure.Management.StorSimple8000Series.zip

    ```

6. 確認已在 `C:\scripts\StorSimpleSDKTools` 中建立自動化模組 ZIP 檔案。

    ![verify-automation-module](./media/storsimple-8000-automation-azurerm-runbook/verify-automation-module.png)

7. 透過 Windows PowerShell 建立自動化模組後，會顯示下列輸出。 

    ```
    -----------------------------------------
    Windows PowerShell
    Copyright (C) 2016 Microsoft Corporation. All rights reserved.

    PS C:\WINDOWS\system32> mkdir C:\scripts\StorSimpleSDKTools

        Directory: C:\scripts

    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-----       10/18/2017   8:43 AM                StorSimpleSDKTools


    PS C:\WINDOWS\system32> cd c:\scripts\StorSimpleSDKTools
    PS C:\scripts\StorSimpleSDKTools> wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorS
    impleSDKTools\nuget.exe
    PS C:\scripts\StorSimpleSDKTools> C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8
    000series

    -------------------------------------------
    CUT              CUT  
    -------------------------------------------
    Successfully installed 'Microsoft.Azure.Management.Storsimple8000series 1.0.0' to C:\scripts\StorSimpleSDKTools
    Executing nuget actions took 1.77 sec
    PS C:\scripts\StorSimpleSDKTools> C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.Active
    Directory -Version 2.28.3

    -------------------------------------------
    CUT              CUT  
    -------------------------------------------
    Successfully installed 'Microsoft.IdentityModel.Clients.ActiveDirectory 2.28.3' to C:\scripts\StorSimpleSDKTools
    Executing nuget actions took 927.64 ms
    PS C:\scripts\StorSimpleSDKTools> C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Aut
    hentication -Version 2.2.9-preview

    -------------------------------------------
    CUT              CUT  
    -------------------------------------------
    Successfully installed 'Microsoft.Rest.ClientRuntime.Azure.Authentication 2.2.9-preview' to C:\scripts\StorSimpleSDKTool
    s
    Executing nuget actions took 717.48 ms
    PS C:\scripts\StorSimpleSDKTools> wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Mo
    nitor-Backups.ps1 -Out Monitor-Backups.ps1
    PS C:\scripts\StorSimpleSDKTools> # set path variables
    PS C:\scripts\StorSimpleSDKTools>             $downloadDir = "C:\scripts\StorSimpleSDKTools"
    PS C:\scripts\StorSimpleSDKTools>             $moduleDir = "$downloadDir\AutomationModule\Microsoft.Azure.Management.Sto
    rSimple8000Series"
    PS C:\scripts\StorSimpleSDKTools>
    PS C:\scripts\StorSimpleSDKTools>             #don't change the folder name "Microsoft.Azure.Management.StorSimple8000Se
    ries"
    PS C:\scripts\StorSimpleSDKTools>             mkdir "$moduleDir"

        Directory: C:\scripts\StorSimpleSDKTools\AutomationModule


    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-----       10/18/2017   8:48 AM                Microsoft.Azure.Management.StorSimple8000Series

    PS C:\scripts\StorSimpleSDKTools>             copy "$downloadDir\Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\
    lib\net45\Microsoft.IdentityModel.Clients.ActiveDirectory*.dll" $moduleDir
    PS C:\scripts\StorSimpleSDKTools>             copy "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\Mic
    rosoft.Rest.ClientRuntime.Azure*.dll" $moduleDir
    PS C:\scripts\StorSimpleSDKTools>             copy "$downloadDir\Microsoft.Rest.ClientRuntime.2.3.8\lib\net452\Microsoft
    .Rest.ClientRuntime*.dll" $moduleDir
    PS C:\scripts\StorSimpleSDKTools>             copy "$downloadDir\Newtonsoft.Json.6.0.8\lib\net45\Newtonsoft.Json*.dll" $
    moduleDir
    PS C:\scripts\StorSimpleSDKTools>             copy "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.Authentication.2.2.9
    -preview\lib\net45\Microsoft.Rest.ClientRuntime.Azure.Authentication*.dll" $moduleDir
    PS C:\scripts\StorSimpleSDKTools>             copy "$downloadDir\Microsoft.Azure.Management.Storsimple8000series.1.0.0\l
    ib\net452\Microsoft.Azure.Management.Storsimple8000series*.dll" $moduleDir
    PS C:\scripts\StorSimpleSDKTools>
    PS C:\scripts\StorSimpleSDKTools>             #Don't change the name of the Archive
    PS C:\scripts\StorSimpleSDKTools>             compress-Archive -Path "$moduleDir" -DestinationPath Microsoft.Azure.Manag
    ement.StorSimple8000Series.zip
    PS C:\scripts\StorSimpleSDKTools>

    -------------------------------------------

    ```

### <a name="import-publish-and-run-automation-runbook"></a>匯入、發佈和執行自動化 Runbook

1. 在 Azure 入口網站中建立 Azure 執行身分自動化帳戶。 若要這樣做，請移至 [Azure Marketplace] > [所有項目]，然後搜尋**自動化**。 選取 [自動化帳戶]。

    ![search-automation](./media/storsimple-8000-automation-azurerm-runbook/automation1.png)

2. 在 [新增自動化帳戶] 刀鋒視窗中：

    1. 提供自動化帳戶的**名稱**。
    2. 選取連結至您 StorSimple 裝置管理員服務的**訂用帳戶**。
    3. 建立新的資源群組，或從現有的資源群組中選取。
    4. 選取 [位置] \(如果可能，請選取服務執行所在位置)。
    5. 保持選取預設值 [建立執行身分帳戶] 選項。
    5. 或者勾選 [釘選到儀表板]。 按一下頁面底部的 [新增] 。

        ![create-automation-account](./media/storsimple-8000-automation-azurerm-runbook/create-automation-account.png)

    成功建立自動化帳戶之後會通知您。 如需如何建立自動化帳戶的詳細資訊，請移至[建立 [執行身分] 帳戶](https://docs.microsoft.com/azure/automation/automation-create-runas-account)。

3. 為確保建立的自動化帳戶可以存取 StorSimple 裝置管理員服務，您需要指派適當的權限給自動化帳戶。 請移至 StorSimple 裝置管理員服務中的 [存取控制]。 按一下 [+ 新增] 並提供您的 Azure 自動化帳戶名稱。 **儲存**設定。

    ![add-permissions-automation-account](./media/storsimple-8000-automation-azurerm-runbook/goto-add-roles.png)

4. 在新建立的帳戶中，移至 [共用資源] > [模組] 並按一下 [+ 新增模組]。

5. 在 [新增模組] 刀鋒視窗中，瀏覽至 ZIP 壓縮模組的位置，然後選取並開啟模組。 按一下 [SERVICEPRINCIPAL] 。

    ![add-module](./media/storsimple-8000-automation-azurerm-runbook/add-module.png)

6. 移至 [程序自動化] > [Runbook] 並按一下 [+ 新增 Runbook]。 在 [新增 Runbook] 刀鋒視窗中，按一下 [匯入現有 Runbook]。 指向 **Runbook 檔案** 的 Windows PowerShell 指令碼檔案。 會自動選取 Runbook 類型。 提供 Runbook 的名稱和選擇性描述。 按一下頁面底部的 [新增] 。

    ![add-module](./media/storsimple-8000-automation-azurerm-runbook/import-runbook.png)

7. Runbook 會新增至 Runbook 清單。 選取並按一下此 Runbook。

    ![click-new-runbook](./media/storsimple-8000-automation-azurerm-runbook/verify-runbook-created.png)

8. 編輯 Runbook 並按一下 [測試] 窗格。 提供參數，例如 StorSimple 裝置管理員服務的名稱、StorSimple 裝置的名稱和訂用帳戶。 **開始**測試。 執行完成後即會產生報告。 如需詳細資訊，請移至[如何測試 Runbook](../automation/automation-first-runbook-textual-powershell.md#step-3---test-the-runbook)。

    ![test-runbook](./media/storsimple-8000-automation-azurerm-runbook/test-runbook.png)

9. 在 [測試] 窗格中檢查 Runbook 的輸出。 如果滿意測試結果，請關閉窗格。 按一下 [發佈] 並在提示要求確認時予以確認，並發佈 Runbook。

    ![publish-runbook](./media/storsimple-8000-automation-azurerm-runbook/publish-runbook.png)


## <a name="next-steps"></a>後續步驟

[使用 StorSimple Device Manager 服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。