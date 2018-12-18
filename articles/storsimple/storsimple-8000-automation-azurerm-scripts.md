---
title: 使用 Azure Resource Manager 指令碼管理 StorSimple 裝置 | Microsoft Docs
description: 了解如何使用 Azure Resource Manager 指令碼將 StorSimple 作業自動化
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
ms.date: 10/03/2017
ms.author: alkohli
ms.openlocfilehash: f4456200d6f497a87424f12a23034dbff00c75aa
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/08/2017
ms.locfileid: "26373936"
---
# <a name="use-azure-resource-manager-sdk-based-scripts-to-manage-storsimple-devices"></a>使用 Azure Resource Manager SDK 型指令碼管理 StorSimple 裝置

本文說明如何使用 Azure Resource Manager SDK 型指令碼管理您的 StorSimple 8000 系列裝置。 其中也包含一個範例指令碼，可引導您逐步完成設定環境以執行這些指令碼的步驟。

本文僅適用於 Azure 入口網站中執行的 StorSimple 8000 系列裝置。

## <a name="sample-scripts"></a>範例指令碼

下列範例指令碼可用來將各種 StorSimple 作業自動化。

#### <a name="table-of-azure-resource-manager-sdk-based-sample-scripts"></a>Azure Resource Manager SDK 型範例指令碼表格

| Azure Resource Manager 指令碼                    | 說明                                                                                                                                                                                                       |
|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Authorize-ServiceEncryptionRollover.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Authorize-ServiceEncryptionRollover.ps1)          | 此指令碼可讓您授權 StorSimple 裝置以變更服務資料加密金鑰。                                                                                                           |
| [Create-StorSimpleCloudAppliance.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Create-StorSimpleCloudAppliance.ps1)              | 此指令碼會建立 8010 或 8020 StorSimple 雲端設備。 接著可以設定雲端設備，然後向您的 StorSimple Data Manager 服務註冊。                                                       |
| [CreateOrUpdate-Volume.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/CreateOrUpdate-Volume.ps1)                        | 此指令碼會建立或修改 StorSimple 磁碟區。                                                                                                                                                             |
| [Get-DeviceBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceBackup.ps1)                             | 此指令碼會針對向 StorSimple Device Manager 服務註冊的裝置，列出所有備份。                                                                                                          |
| [Get-DeviceBackupPolicy.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceBackupPolicy.ps1)                       | 此指令碼會為您的 StorSimple 裝置取得所有備份原則。                                                                                                                                                 |
| [Get-DeviceJobs.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1)                               | 此指令碼會取得您 StorSimple Device Manager 服務上執行的所有 StorSimple 作業。                                                                                                                     |
| [Get-DeviceUpdateAvailability.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceUpdateAvailability.ps1)                 | 此指令碼會掃描更新伺服器，並讓您知道是否有可安裝在您 StorSimple 裝置上的更新。                                                                                          |
| [Install-DeviceUpdate.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Install-DeviceUpdate.ps1)                         | 此指令碼會在您的 StorSimple 裝置上安裝可用的更新。                                                                                                                                           |
| [Manage-CloudSnapshots.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Manage-CloudSnapshots.ps1)                        | 此指令碼會啟動手動雲端快照集，並刪除比指定之保留天數還舊的雲端快照集。                                                                                                   |
| [Monitor-Backups.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1)                              | 此 Azure Automation Runbook PowerShell 指令碼會報告所有備份工作的狀態。                                                                                                              |
| [Remove-DeviceBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Remove-DeviceBackup.ps1)                          | 此指令碼會刪除單一備份物件。                                                                                                                                                           |
| [Start-DeviceBackupJob.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Start-DeviceBackupJob.ps1)                        | 此指令碼會在您的 StorSimple 裝置上啟動手動備份。                                                                                                                                       |
| [Update-CloudApplianceServiceEncryptionKey.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Update-CloudApplianceServiceEncryptionKey.ps1)    | 此指令碼會針對已經向 StorSimple Device Manager 服務註冊的所有 8010/8020 StorSimple 雲端設備，更新服務資料加密金鑰。                                     |
| [Verify-BackupScheduleAndBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Verify-BackupScheduleAndBackup.ps1)               | 此指令碼會在分析與備份原則相關聯的所有排程之後，反白顯示遺漏的備份。 它也會使用可用備份的清單，驗證備份目錄。             |




## <a name="configure-and-run-a-sample-script"></a>設定並執行範例指令碼

本節會採用範例指令碼，並詳細說明執行指令碼所需的各種步驟。

### <a name="prerequisites"></a>先決條件

開始之前，請確定您有︰

*   已安裝 Azure PowerShell。 若要安裝 Azure PowerShell 模組：
    * 在 Windows 環境中，依照[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-4.4.0) \(英文\) 中的步驟進行。 如果有使用 Azure PowerShell，可以將其安裝在 StorSimple 的 Windows Server 主機上。
    * 在 Linux 或 MacOS 環境中，依照[在 MacOS 或 Linux 上安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurermps-maclinux?view=azurermps-4.4.0) \(英文\) 中的步驟進行。

如需有關使用 Azure PowerShell 的詳細資訊，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps?view=azurermps-4.4.0)。

### <a name="run-azure-powershell-script"></a>執行 Azure PowerShell 指令碼

此範例中使用的指令碼會列出 StorSimple 裝置上的所有作業。 這包括成功、失敗或進行中的作業。 執行下列步驟，下載並執行指令碼。

1. 啟動 Azure PowerShell。 建立一個新資料夾，並將目錄切換到新資料夾。

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
        wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1 -Out Get-DeviceJobs.ps1

    ```

5. 執行指令碼。 收到進行驗證的提示時，請提供您的 Azure 認證。 此指令碼應該會針對您 StorSimple 裝置上的所有作業，輸出經過篩選的清單。
           
    ```           
        .\Get-StorSimpleJob.ps1 -SubscriptionId [subid] -TenantId [tenant id] -DeviceName [name of device] -ResourceGroupName [name of resource group] -ManagerName[name of device manager] -FilterByStatus [Filter for job status] -FilterByJobType [Filter for job type] -FilterByStartTime [Filter for start date time] -FilterByEndTime [Filter for end date time]

    ```

### <a name="sample-output"></a>範例輸出

下列輸出會在執行範例指令碼時顯示。 輸出會包含在 2017 年 9 月 25 日至 2017 年 10 月 2 日期間註冊之裝置上執行的所有作業。

```
-----------------------------------------
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Scripts\StorSimpleSDKTools> wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1 -Out Get-DeviceJobs.ps1
PS C:\Scripts\StorSimpleSDKTools> .\Get-DeviceJobs.ps1 -SubscriptionId 1234ab5c-678d-910e-9fc4-0accc9c0166e -TenantId 12a345bc-67d8-91ef-01ab-2c7cd123ef45 -DeviceName 8600-ABC1234567D89EF -ResourceGroupName Contoso -ManagerName ContosoDeviceMgr -FilterByStartTime "09/25/2017 08:10:02" -FilterByEndTime "10/02/2017 08:10:02"


Status            : Succeeded
StartTime         : 10/2/2017 10:30:02 PM
EndTime           : 10/2/2017 10:31:36 PM
PercentComplete   : 100
Error             :
JobType           : ScheduledBackup
DataStats         : Microsoft.Azure.Management.StorSimple8000Series.Models.DataStatistics
EntityLabel       : ss-asr-policy1
EntityType        : Microsoft.StorSimple/managers/devices/backupPolicies
JobStages         :
DeviceId          : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-SHG0997877L71FC
IsCancellable     : True
BackupType        : CloudSnapshot
SourceDeviceId    :
BackupPointInTime : 1/1/0001 12:00:00 AM
Id                : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-SHG0997877L71FC/jobs/75905c48-b153-4af1-8b21-4b9a2ff9
                    825b
Name              : 75905c48-b153-4af1-8b21-4b9a2ff9825b
Type              : Microsoft.StorSimple/managers/devices/jobs
Kind              : Series8000
-------------------------------------------
CUT              CUT  
-------------------------------------------
Status            : Succeeded
StartTime         : 9/26/2017 5:00:02 PM
EndTime           : 9/26/2017 5:01:20 PM
PercentComplete   : 100
Error             :
JobType           : ScheduledBackup
DataStats         : Microsoft.Azure.Management.StorSimple8000Series.Models.DataStatistics
EntityLabel       : 8010 policy
EntityType        : Microsoft.StorSimple/managers/devices/backupPolicies
JobStages         :
DeviceId          : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-ABC1234567D89EF
IsCancellable     : True
BackupType        : CloudSnapshot
SourceDeviceId    :
BackupPointInTime : 1/1/0001 12:00:00 AM
Id                : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-ABC1234567D89EF/jobs/3cfd8108-db60-4e9a-a8da-6d8fe457
                    8d2b
Name              : 3cfd8108-db60-4e9a-a8da-6d8fe4578d2b
Type              : Microsoft.StorSimple/managers/devices/jobs
Kind              : Series8000



PS C:\Scripts\StorSimpleSDKTools>
--------------------------------------------

```


## <a name="next-steps"></a>後續步驟

[使用 StorSimple Device Manager 服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。