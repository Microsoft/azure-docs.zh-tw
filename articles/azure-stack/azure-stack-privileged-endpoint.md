---
title: 使用 Azure Stack 中具有特殊權限的端點 | Microsoft Docs
description: 說明如何使用 Azure Stack 中具有特殊權限的端點 (PEP) (適用於 Azure Stack 操作員)。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: e94775d5-d473-4c03-9f4e-ae2eada67c6c
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/27/2018
ms.author: mabrigg
ms.reviewer: fiseraci
ms.openlocfilehash: 9fb928b7cb8e1a83734b64a8b9c19bc3cf3203ba
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32153179"
---
# <a name="using-the-privileged-endpoint-in-azure-stack"></a>使用 Azure Stack 中具有特殊權限的端點

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

作為 Azure Stack 操作員，您應該在大部分的日常管理工作使用系統管理員入口網站、PowerShell 或 Azure Resource Manager API。 不過，針對較非一般的作業，您必須使用「具有特殊權限的端點」(PEP)。 PEP 是預先設定的遠端 PowerShell 主控台，能提供恰到好處的功能來協助您執行必要的工作。 此端點使用 [PowerShell JEA (Just Enough Administration)](https://docs.microsoft.com/powershell/jea/overview) 來只公開一組有限的 Cmdlet。 若要存取 PEP 並叫用一組有限的 Cmdlet，可使用低權限的帳戶。 無須系統管理員帳戶。 為了增加安全性，不允許使用指令碼。

您可以使用 PEP 來執行如以下的工作：

- 執行低階工作，例如[收集診斷記錄](https://docs.microsoft.com/azure/azure-stack/azure-stack-diagnostics#log-collection-tool)。
- 執行整合式系統的許多部署後資料中心整合工作，例如在部署後新增「網域名稱系統」(DNS) 轉寄站、設定 Microsoft Graph 整合、「Active Directory 同盟服務」(AD FS) 整合、憑證輪用等等。
- 與支援人員共同合作，以取得整合系統深入疑難排解的暫時、高階存取權。

PEP 會記錄您在 PowerShell 工作階段中執行的每個動作 (和其對應的輸出)。 這會提供完全透明化和完整稽核的作業。 您可以保留這些記錄檔以供日後稽核。

> [!NOTE]
> 在「Azure Stack 開發套件」(ASDK) 中，您可以在開發套件主機上，直接從 PowerShell 工作階段執行 PEP 中的一些可用命令。 不過，您可能會想要使用 PEP 來測試一些作業 (例如記錄收集)，因為這是在整合式系統環境中執行特定作業的唯一可用方法。

## <a name="access-the-privileged-endpoint"></a>存取具有特殊權限的端點

您可在裝載 PEP 的虛擬機器上，透過遠端 PowerShell 工作階段來存取 PEP。 在 ASDK 中，此虛擬機器名為 **AzS-ERCS01**。 如果您使用整合式系統，則會有三個 PEP 執行個體，每個都在不同主機上的虛擬機器內執行 (*Prefix*-ERCS01、*Prefix*-ERCS02 或 *Prefix*-ERCS03) 以提供復原能力。 

在開始針對整合式系統進行此程序之前，請確定您可以透過 IP 位址或 DNS 來存取 PEP。 在初始部署 Azure Stack 之後，您只能透過 IP 位址來存取 PEP，因為尚未設定 DNS 整合。 您的 OEM 硬體廠商會提供名為 **AzureStackStampDeploymentInfo** 的 JSON 檔案，其中包含 PEP IP 位址。


> [!NOTE]
> 為了確保安全性，建議您只從硬體生命週期主機以外的主機上執行的強化虛擬機器連線至 PEP，或者從專用、安全的電腦 (例如[特殊權限存取工作站](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)) 來連線到 PEP。 請不要修改硬體生命週期主機的原始組態，包括安裝新軟體；也請不要用來連線至 PEP。

1. 建立信任關係。

    - 在整合的系統中，從提升權限的 Windows PowerShell 工作階段執行下列命令，將 PEP 新增為在硬體生命週期主機或特殊權限存取工作站上執行的強化虛擬機器的受信任主機。

      ````PowerShell
        winrm s winrm/config/client '@{TrustedHosts="<IP Address of Privileged Endpoint>"}'
      ````
    - 如果您是執行 ADSK，請登入開發套件主機。

2. 在硬體生命週期主機或特殊權限工作站中執行的強化虛擬機器上，開啟 Windows PowerShell 工作階段。 執行下列命令，以在裝載 PEP 的虛擬機器上建立遠端工作階段：
 
    - 在整合系統上：
      ````PowerShell
        $cred = Get-Credential

        Enter-PSSession -ComputerName <IP_address_of_ERCS> `
          -ConfigurationName PrivilegedEndpoint -Credential $cred
      ````
      `ComputerName` 參數可以是其中一部裝載 PEP 之虛擬機器的 IP 位址或 DNS 名稱。 
    - 如果您是執行 ADSK：
     
      ````PowerShell
        $cred = Get-Credential

        Enter-PSSession -ComputerName azs-ercs01 `
          -ConfigurationName PrivilegedEndpoint -Credential $cred
      ```` 
   出現輸入提示時，使用下列認證：

      - **使用者名稱**：指定 CloudAdmin 帳戶，格式為 **&lt;*Azure Stack 網域*&gt;\cloudadmin**。 (若為 ASDK，使用者名稱是 **azurestack\cloudadmin**。)
      - **密碼**：輸入與 AzureStackAdmin 網域系統管理員帳戶安裝期間所提供的相同密碼。

    > [!NOTE]
    > 如果您無法連線到 ERCS 端點，請用 ERCS VM 的 IP 位址，再試一次步驟一和二與尚未連線的端點連線。

3.  連線之後，視環境而定，提示會變更為 **[*IP 位址或 ERCS VM 名稱*]: PS>** 或變更為 **[azs-ercs01]: PS>**。 從這裡執行 `Get-Command` 可檢視可用的 Cmdlet 清單。

    這些 Cmdlet 有許多僅供整合系統環境 (例如與資料中心整合相關的 cmdlet) 使用。 在 ASDK 中，下列 Cmdlet 已經過驗證：

    - Clear-Host
    - Close-PrivilegedEndpoint
    - Exit-PSSession
    - Get-AzureStackLog
    - Get-AzureStackStampInformation
    - Get-Command
    - Get-FormatData
    - Get-Help
    - Get-ThirdPartyNotices
    - Measure-Object
    - New-CloudAdminUser
    - Out-Default
    - Remove-CloudAdminUser
    - Select-Object
    - Set-CloudAdminUserPassword
    - Test-AzureStack
    - Stop-AzureStack
    - Get-ClusterLog

## <a name="tips-for-using-the-privileged-endpoint"></a>使用具有特殊權限端點的秘訣 

如以上所述，PEP 是 [PowerShell JEA](https://docs.microsoft.com/powershell/jea/overview) 端點。 雖然 JEA 端點提供強大的安全性階層，但也縮減了一些 PowerShell 功能，例如指令碼處理或 TAB 鍵自動完成功能。 如果您嘗試任何一種指令碼作業，作業就會失敗，並出現錯誤 **ScriptsNotAllowed**。 這是預期行為。

因此，例如，若要取得特定 Cmdlet 的參數清單，您需執行下列命令：

```PowerShell
    Get-Command <cmdlet_name> -Syntax
```

或者，您也可以使用 [Import-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Utility/Import-PSSession?view=powershell-5.1) Cmdlet，將所有 PEP Cmdlet 匯入到您本機電腦上的目前工作階段中。 如此一來，現在在本機電腦上就可使用 PEP 的所有 Cmdlet 和函式，連同 TAB 鍵自動完成功能，以及更廣泛來說，還有指令碼處理。 

若要在本機電腦上匯入 PEP 工作階段，請執行下列步驟：

1. 建立信任關係。

    -在整合的系統中，從提升權限的 Windows PowerShell 工作階段執行下列命令，將 PEP 新增為在硬體生命週期主機或特殊權限存取工作站上執行的強化虛擬機器的受信任主機。

      ````PowerShell
        winrm s winrm/config/client '@{TrustedHosts="<IP Address of Privileged Endpoint>"}'
      ````
    - 如果您是執行 ADSK，請登入開發套件主機。

2. 在硬體生命週期主機或特殊權限工作站中執行的強化虛擬機器上，開啟 Windows PowerShell 工作階段。 執行下列命令，以在裝載 PEP 的虛擬機器上建立遠端工作階段：
 
    - 在整合系統上：
      ````PowerShell
        $cred = Get-Credential

        $session = New-PSSession -ComputerName <IP_address_of_ERCS> `
          -ConfigurationName PrivilegedEndpoint -Credential $cred
      ````
      `ComputerName` 參數可以是其中一部裝載 PEP 之虛擬機器的 IP 位址或 DNS 名稱。 
    - 如果您是執行 ADSK：
     
      ````PowerShell
       $cred = Get-Credential

       $session = New-PSSession -ComputerName azs-ercs01 `
          -ConfigurationName PrivilegedEndpoint -Credential $cred
      ```` 
   出現輸入提示時，使用下列認證：

      - **使用者名稱**：指定 CloudAdmin 帳戶，格式為 **&lt;*Azure Stack 網域*&gt;\cloudadmin**。 (若為 ASDK，使用者名稱是 **azurestack\cloudadmin**。)
      - **密碼**：輸入與 AzureStackAdmin 網域系統管理員帳戶安裝期間所提供的相同密碼。

3. 將 PEP 工作階段匯入到本機電腦
    ````PowerShell 
        Import-PSSession $session
    ````
4. 現在，您可以在本機 PowerShell 工作階段上，搭配 PEP 的所有函式和 Cmdlet 如常使用 TAB 鍵自動完成功能和進行指令碼處理，而無須降低 Azure Stack 的安全性狀態。 盡情享受！


## <a name="close-the-privileged-endpoint-session"></a>關閉具有特殊權限的端點工作階段

 如先前所述，PEP 會記錄您在 PowerShell 工作階段中執行的每個動作 (和其對應的輸出)。 您必須使用 `Close-PrivilegedEndpoint` Cmdlet 來關閉工作階段。 此 Cmdlet 會正確關閉端點，並將記錄檔傳送至外部檔案共用作為保留。

若要關閉端點工作階段：

1. 建立 PEP 可存取的外部檔案共用。 在開發套件環境中，您只能在開發套件主機上建立檔案共用。
2. 執行 `Close-PrivilegedEndpoint` Cmdlet。 
3. 系統會提示您要存放文字記錄記錄檔的路徑。 指定在稍早建立的檔案共用，格式為 &#92;&#92;servername&#92;sharename。 如果您未指定路徑，Cmdlet 就會失敗，且工作階段維持開啟狀態。 

    ![會顯示您指定文字記錄目的地路徑的 Close-PrivilegedEndpoint Cmdlet 輸出](media/azure-stack-privileged-endpoint/closeendpoint.png)

文字記錄記錄檔在成功傳輸至檔案共用之後，會自動從 PEP 中刪除。 

> [!NOTE]
> 如果您使用 `Exit-PSSession` 或 `Exit` Cmdlet 來關閉 PEP 工作階段，或您直接關閉 PowerShell 主控台，該文字記錄記錄檔就不會傳輸至檔案共用。 它們會存留在 PEP 中。 下次您執行 `Close-PrivilegedEndpoint` 並包含檔案共用時，也將會傳送上一個工作階段的文字記錄記錄。 請勿使用 `Exit-PSSession` 或 `Exit` 關閉 PEP 工作階段：改為使用 `Close-PrivilegedEndpoint`。


## <a name="next-steps"></a>後續步驟
[Azure Stack 診斷工具](azure-stack-diagnostics.md)
