---
title: 在 Azure Stack 中執行驗證測試 | Microsoft Docs
description: 如何在 Azure Stack 中收集記錄檔以進行診斷。
services: azure-stack
author: mattbriggs
manager: femila
cloud: azure-stack
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: article
ms.date: 10/15/2018
ms.author: mabrigg
ms.reviewer: hectorl
ms.openlocfilehash: 3f4dc6e4136d8d2e3eb1ca5e822306aae2217e3b
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340846"
---
# <a name="run-a-validation-test-for-azure-stack"></a>執行 Azure Stack 的驗證測試

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*
 
您可以驗證 Azure Stack 的狀態。 如有任何問題，請連絡 Microsoft 客戶服務支援。 支援人員會要求您從管理節點執行 **Test-AzureStack**。 驗證測試會將失敗區域隔離出來。 支援人員才可分析詳細的記錄、將焦點放在發生錯誤的區域，並與您一起解決問題。

## <a name="run-test-azurestack"></a>執行 Test-AzureStack

如有任何問題，請連絡 Microsoft 客戶服務支援，然後執行 **Test-AzureStack**。

1. 您有一個問題。
2. 請連絡 Microsoft 客戶服務支援。
3. 從具有特殊權限的端點執行 **Test-AzureStack**。
    1. 存取具有特殊權限的端點。 如需相關指示，請參閱[使用 Azure Stack 中具有特殊權限的端點](azure-stack-privileged-endpoint.md)。 
    2. 在 ASDK 上，以 **AzureStack\CloudAdmin** 身分登入管理主機。  
    您必須在整合的系統上使用具特殊權限端點的 IP 位址，以進行 OEM 硬體廠商所提供給您的管理。
    3. 以系統管理員身分啟動 PowerShell。
    4. 執行： `Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint`
    5. 執行： `Test-AzureStack`
4. 如有任何測試報告失敗，請執行：`Get-AzureStackLog -FilterByRole SeedRing -OutputPath <Log output path>` 此 Cmdlet 會從 Test-AzureStack 收集記錄。 如需診斷記錄的詳細資訊，請參閱 [Azure Stack 診斷工具](azure-stack-diagnostics.md)。
5. 將 **SeedRing** 記錄傳送給 Microsoft 客戶服務支援。 Microsoft 客戶服務支援人員會與您一起解決問題。

## <a name="reference-for-test-azurestack"></a>Test-AzureStack 的參考

這一節包含 Test-AzureStack Cmdlet 的概觀及驗證報告的摘要。

### <a name="test-azurestack"></a>Test-AzureStack

驗證 Azure Stack 的狀態。 此 Cmdlet 會報告 Azure Stack 硬體和軟體的狀態。 支援人員可以使用此報告來減少解決 Azure Stack 支援案例所需的時間。

> [!Note]  
> **Test-AzureStack** 可能會偵測到不會導致雲端中斷的錯誤，例如單一失敗磁碟或單一實體主機節點錯誤。

#### <a name="syntax"></a>語法

````PowerShell
  Test-AzureStack
````

#### <a name="parameters"></a>參數

| 參數               | 值           | 必要 | 預設值 |
| ---                     | ---             | ---      | ---     |
| ServiceAdminCredentials | 字串    | 否       | FALSE   |
| DoNotDeployTenantVm     | SwitchParameter | 否       | FALSE   |
| AdminCredential         | PSCredential    | 否       | NA      |
| 列出                    | SwitchParameter | 否       | FALSE   |
| 略過                  | 字串          | 否       | NA      |
| 包含                 | 字串          | 否       | NA      |
| BackupSharePath         | 字串          | 否       | NA      |
| BackupShareCredential   | PSCredential    | 否       | NA      |


Test-AzureStack Cmdlet 支援一般參數：Verbose、Debug、ErrorAction、ErrorVariable、WarningAction、WarningVariable、OutBuffer、PipelineVariable 和 OutVariable。 如需詳細資訊，請參閱[關於一般參數](http://go.microsoft.com/fwlink/?LinkID=113216)。 

### <a name="examples-of-test-azurestack"></a>Test-AzureStack 的範例

下列範例假設您以 **CloudAdmin** 身分進行登入並存取具有特殊權限的端點 (PEP)。 如需相關指示，請參閱[使用 Azure Stack 中具有特殊權限的端點](azure-stack-privileged-endpoint.md)。 

#### <a name="run-test-azurestack-interactively-without-cloud-scenarios"></a>以互動方式執行 Test-AzureStack (沒有雲端案例)

在 PEP 工作階段中執行：

````PowerShell
    Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint -Credential $localcred
    Test-AzureStack
````

#### <a name="run-test-azurestack-with-cloud-scenarios"></a>執行 Test-AzureStack (具有雲端案例)

您可以使用 **Test-AzureStack** 對您的 Azure Stack 執行雲端案例。 這些案例包括：

 - 建立資源群組
 - 建立方案
 - 建立供應項目
 - 建立儲存體帳戶
 - 建立虛擬機器
 - 使用在測試案例中建立的儲存體帳戶執行 blob 作業
 - 使用在測試案例中建立的儲存體帳戶執行佇列作業
 - 使用在測試案例中建立的儲存體帳戶執行資料表作業

雲端案例需要雲端管理員認證。 
> [!Note]  
> 您無法使用 Active Directory 同盟服務 (AD FS) 認證執行雲端案例。 **Test-AzureStack** Cmdlet 只能透過 PEP 存取。 但是，PEP 不支援 AD FS 認證。

以 UPN 格式 serviceadmin@contoso.onmicrosoft.com (Azure AD) 輸入雲端管理員使用者名稱。 出現提示時，輸入雲端管理員帳戶的密碼。

在 PEP 工作階段中執行：

````PowerShell
  Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint -Credential $localcred
  Test-AzureStack -ServiceAdminCredentials <Cloud administrator user name>
````

#### <a name="run-test-azurestack-without-cloud-scenarios"></a>執行 Test-AzureStack (沒有雲端案例)

在 PEP 工作階段中執行：

````PowerShell
  $session = New-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint -Credential $localcred
  Invoke-Command -Session $session -ScriptBlock {Test-AzureStack}
````

#### <a name="list-available-test-scenarios"></a>列出可用的測試案例：

在 PEP 工作階段中執行：

````PowerShell
  Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint -Credential $localcred
  Test-AzureStack -List
````

#### <a name="run-a-specified-test"></a>執行指定的測試

在 PEP 工作階段中執行：

````PowerShell
  Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint -Credential $localcred
  Test-AzureStack -Include AzsSFRoleSummary, AzsInfraCapacity
````

若要排除特定測試：

````PowerShell
    Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint  -Credential $localcred
    Test-AzureStack -Ignore AzsInfraPerformance
````

### <a name="run-test-azurestack-to-test-infrastructure-backup-settings"></a>執行 Test-AzureStack 以測試基礎結構備份設定

設定基礎結構備份之前，您可以使用 **AzsBackupShareAccessibility** 測試來測試備份共用路徑和認證。

在 PEP 工作階段中執行：

````PowerShell
    Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint -Credential $localcred
    Test-AzureStack -Include AzsBackupShareAccessibility -BackupSharePath "\\<fileserver>\<fileshare>" -BackupShareCredential <PSCredentials-for-backup-share>
````
設定好備份之後，您可以從 PEP 工作階段中執行 AzsBackupShareAccessibility，來確認共用是否可以從 ERCS 進行存取：

````PowerShell
    Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint  -Credential $localcred
    Test-AzureStack -Include AzsBackupShareAccessibility
````

若要對設定的備份共用測試新認證，請從 PEP 工作階段中執行：

````PowerShell
    Enter-PSSession -ComputerName <ERCS-VM-name> -ConfigurationName PrivilegedEndpoint -Credential $localcred
    Test-AzureStack -Include AzsBackupShareAccessibility -BackupShareCredential <PSCredential for backup share>
````

### <a name="validation-test"></a>驗證測試

下表摘要說明 **Test-AzureStack** 所執行的驗證測試。

| 名稱                                                                                                                              |
|-----------------------------------------------------------------------------------------------------------------------------------|-----------------------|
| Azure Stack 雲端裝載基礎結構摘要                                                                                  |
| Azure Stack 儲存體服務摘要                                                                                              |
| Azure Stack 基礎結構角色執行個體摘要                                                                                  |
| Azure Stack 雲端裝載基礎結構使用率                                                                              |
| Azure Stack 基礎結構容量                                                                                               |
| Azure Stack 入口網站和 API 摘要                                                                                                |
| Azure Stack 的 Azure Resource Manager 憑證摘要                                                                                               |
| 基礎結構管理控制器、網路控制器、儲存體服務和具有特殊權限的端點基礎結構角色          |
| 基礎結構管理控制器、網路控制器、儲存體服務和具有特殊權限的端點基礎結構角色執行個體 |
| Azure Stack 基礎結構角色摘要                                                                                           |
| Azure Stack 雲端 Service Fabric 服務                                                                                         |
| Azure Stack 基礎結構角色執行個體效能                                                                              |
| Azure Stack 雲端主機效能摘要                                                                                        |
| Azure Stack 服務資源耗用量摘要                                                                                  |
| Azure Stack 縮放單位重大事件 (過去 8 小時)                                                                             |
| Azure Stack 儲存體服務實體磁碟摘要                                                                               |
|Azure Stack 備份共用的可及性摘要                                                                                     |

## <a name="next-steps"></a>後續步驟

 - 若要深入了解 Azure Stack 診斷工具和問題記錄的詳細資訊，請參閱 [Azure Stack 診斷工具](azure-stack-diagnostics.md)。
 - 若要深入了解疑難排解，請參閱 [Microsoft Azure Stack 疑難排解](azure-stack-troubleshooting.md)
