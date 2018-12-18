---
title: 使用具有特殊權限的端點來監視 Azure Stack 中的更新 | Microsoft Docs
description: 了解如何使用具有特殊權限的端點來監視 Azure Stack 整合系統的更新狀態。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 449ae53e-b951-401a-b2c9-17fee2f491f1
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/17/2018
ms.author: mabrigg
ms.openlocfilehash: 8f384a79811c9a9b104acb98c8f6b6e162946ab8
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "41947933"
---
# <a name="monitor-updates-in-azure-stack-using-the-privileged-endpoint"></a>使用具有特殊權限的端點來監視 Azure Stack 中的更新

「適用於：Azure Stack 整合系統」

您可以使用具有特殊權限的端點來監視 Azure Stack 更新執行的進度，以及從 Azure Stack 入口網站應變成無法使用的最後一個成功步驟中，繼續進行失敗的更新執行。  建議使用 Azure Stack 入口網站在 Azure Stack 中管理更新。

下列適用於更新管理的新 PowerShell Cmdlet 隨附於適用於 Azure Stack 整合系統的 1710 更新中。

| Cmdlet  | 說明  |
|---------|---------|
| `Get-AzureStackUpdateStatus` | 傳回目前正在執行、已完成或失敗之更新的狀態。 提供更新作業的高階狀態，以及說明目前步驟和對應狀態的 XML 文件。 |
| `Resume-AzureStackUpdate` | 從更新失敗的地方繼續執行失敗的更新。 在某些情況下，您可能必須先完成風險降低步驟，然後才能繼續執行更新。         |
| | |

## <a name="verify-the-cmdlets-are-available"></a>確認 Cmdlet 可供使用
因為 Cmdlet 是適用於 Azure Stack 之 1710 更新程式封裝中的新功能，所以 1710 更新程序需要先到達某個點，才能使用監視功能。 一般而言，如果系統管理員入口網站中的狀態表示 1710 更新位於**重新啟動儲存體主機**步驟中，則可使用 Cmdlet。 具體來說，Cmdlet 更新會在**步驟：正在執行步驟 2.6 - 更新 PrivilegedEndpoint 白名單**期間發生。

您也可以藉由從具有特殊權限的端點查詢命令清單，來判斷 Cmdlet 是否可以程式設計方式使用。 若要這樣做，請從硬體生命週期主機或特殊權限存取工作站執行下列命令。 此外，確定具有特殊權限的端點是信任的主機。 如需詳細資訊，請參閱[存取具有特殊權限的端點](azure-stack-privileged-endpoint.md#access-the-privileged-endpoint)的步驟 1。 

1. 在您的 Azure Stack 環境中的任何 ERCS 虛擬機器上建立 PowerShell 工作階段 (*Prefix*-ERCS01、*Prefix*-ERCS02 或 *Prefix*-ERCS03)。 使用您環境專屬的虛擬機器前置字串來取代 *Prefix*。

   ```powershell
   $cred = Get-Credential

   $pepSession = New-PSSession -ComputerName <Prefix>-ercs01 -Credential $cred -ConfigurationName PrivilegedEndpoint 
   ```
   當系統提示您輸入認證時，請使用 &lt;Azure Stack 網域&gt;\cloudadmin 帳戶或屬於 CloudAdmins 群組成員的帳戶。 針對 CloudAdmin 帳戶，輸入 AzureStackAdmin 網域系統管理員帳戶安裝期間所提供的相同密碼。

2. 取得具有特殊權限之端點中可用的完整命令清單。 

   ```powershell
   $commands = Invoke-Command -Session $pepSession -ScriptBlock { Get-Command } 
   ```
3. 判斷具有特殊權限的端點是否已更新。

   ```powershell
   $updateManagementModuleName = "Microsoft.Azurestack.UpdateManagement"
    if (($commands | ? Source -eq $updateManagementModuleName)) {
   Write-Host "Privileged endpoint was updated to support update monitoring tools."
    } else {
   Write-Host "Privileged endpoint has not been updated yet. Please try again later."
    } 
   ```

4. 列出 Microsoft.AzureStack.UpdateManagement 模組特有的命令。

   ```powershell
   $commands | ? Source -eq $updateManagementModuleName 
   ```
   例如︰
   ```powershell
   $commands | ? Source -eq $updateManagementModuleName
   
   CommandType     Name                                               Version    Source                                                  PSComputerName
    -----------     ----                                               -------    ------                                                  --------------
   Function        Get-AzureStackUpdateStatus                         0.0        Microsoft.Azurestack.UpdateManagement                   Contoso-ercs01
   Function        Resume-AzureStackUpdate                            0.0        Microsoft.Azurestack.UpdateManagement                   Contoso-ercs01
   ``` 

## <a name="use-the-update-management-cmdlets"></a>使用更新管理 Cmdlet

> [!NOTE]
> 從硬體生命週期主機或特殊權限存取工作站執行下列命令。 此外，確定具有特殊權限的端點是信任的主機。 如需詳細資訊，請參閱[存取具有特殊權限的端點](azure-stack-privileged-endpoint.md#access-the-privileged-endpoint)的步驟 1。

### <a name="connect-to-the-privileged-endpoint-and-assign-session-variable"></a>連接到具有特殊權限的端點並指派工作階段變數

執行下列命令，在您的 Azure Stack 環境中的任何 ERCS 虛擬機器上建立 PowerShell 工作階段 (*Prefix*-ERCS01、*Prefix*-ERCS02 或 *Prefix*-ERCS03)，以及指派工作階段變數。

```powershell
$cred = Get-Credential

$pepSession = New-PSSession -ComputerName <Prefix>-ercs01 -Credential $cred -ConfigurationName PrivilegedEndpoint 
```
 當系統提示您輸入認證時，請使用 &lt;Azure Stack 網域&gt;\cloudadmin 帳戶或屬於 CloudAdmins 群組成員的帳戶。 針對 CloudAdmin 帳戶，輸入 AzureStackAdmin 網域系統管理員帳戶安裝期間所提供的相同密碼。

### <a name="get-high-level-status-of-the-current-update-run"></a>取得目前更新執行的高階狀態 

若要取得目前更新執行的高階狀態，請執行下列命令： 

```powershell
$statusString = Invoke-Command -Session $pepSession -ScriptBlock { Get-AzureStackUpdateStatus -StatusOnly }

$statusString.Value 
```

可能的值包括：

- 執行中
- Completed
- Failed 
- Canceled

您可以重複執行這些命令來查看最新狀態。 您不需要重新建立連線來再次檢查。

### <a name="get-the-full-update-run-status-with-details"></a>取得完整更新執行狀態以及詳細資料 

您可以 XML 字串形式取得完整的更新執行摘要。 您可以將字串寫入檔案，以供檢查之用，或將它轉換成 XML 文件，並使用 PowerShell 進行剖析。 下列命令會剖析 XML，以取得目前執行中步驟的階層式清單。

```powershell
[xml]$updateStatus = Invoke-Command -Session $pepSession -ScriptBlock { Get-AzureStackUpdateStatus }

$updateStatus.SelectNodes("//Step[@Status='InProgress']")
```

在下列範例中，最上層步驟 (更新雲端) 具有更新並重新啟動儲存體主機的子方案。 它會顯示「重新啟動儲存體主機」方案正在其中一部主機上更新 Blob 儲存體服務。

```powershell
[xml]$updateStatus = Invoke-Command -Session $pepSession -ScriptBlock { Get-AzureStackUpdateStatus }

$updateStatus.SelectNodes("//Step[@Status='InProgress']") 

    FullStepIndex : 2
    Index         : 2
    Name          : Cloud Update
    Description   : Perform cloud update.
    StartTimeUtc  : 2017-10-13T12:50:39.9020351Z
    Status        : InProgress
    Task          : Task
    
    FullStepIndex  : 2.9
    Index          : 9
    Name           : Restart Storage Hosts
    Description    : Restart Storage Hosts.
    EceErrorAction : Stop
    StartTimeUtc   : 2017-10-13T15:44:06.7431447Z
    Status         : InProgress
    Task           : Task
    
    FullStepIndex : 2.9.2
    Index         : 2
    Name          : PreUpdate ACS Blob Service
    Description   : Check function level, update deployment artifacts, configure Blob service settings
    StartTimeUtc  : 2017-10-13T15:44:26.0708525Z
    Status        : InProgress
    Task          : Task
```

### <a name="resume-a-failed-update-operation"></a>繼續執行失敗的更新作業

如果更新失敗，您可以從更新中斷的位置繼續進行更新執行。

```powershell
Invoke-Command -Session $pepSession -ScriptBlock { Resume-AzureStackUpdate } 
```

## <a name="troubleshoot"></a>疑難排解

具有特殊權限的端點適用於 Azure Stack 環境中的所有 ERCS 虛擬機器上。 由於並未與高度可用的端點建立連線，因此，您可能會遇到偶發性的中斷、警告或錯誤訊息。 這些訊息可能表示工作階段已中斷連線，或者與 ECE 服務通訊時發生錯誤。 這是預期的行為。 您可以在幾分鐘後重試此作業，或者在其他 ERCS 虛擬機器的其中一部上新建具有特殊權限的端點工作階段。 

## <a name="next-steps"></a>後續步驟

- [在 Azure Stack 中管理更新](azure-stack-updates.md) 


