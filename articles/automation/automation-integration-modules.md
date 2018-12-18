---
title: 建立 Azure 自動化整合模組
description: 引導您在 Azure 自動化中建立、測試和舉例使用整合模組的逐步解說教學課程。
services: automation
ms.service: automation
ms.component: shared-capabilities
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 93c61f0b9b923f84b2c84d2db4456442e2f9fb27
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39444499"
---
# <a name="azure-automation-integration-modules"></a>Azure 自動化整合模組
PowerShell 是 Azure 自動化背後的基本技術。 由於 Azure 自動化的基礎是 PowerShell，PowerShell 模組會是 Azure 自動化的擴充性關鍵。 本文會引導您了解 Azure 自動化在使用 PowerShell 模組 (稱為「整合模組」) 方面的細節以及用來建立自有 PowerShell 模組的最佳做法，以確保 PowerShell 模組可作為 Azure 自動化內的整合模組。 

## <a name="what-is-a-powershell-module"></a>什麼是 PowerShell 模組？
PowerShell 模組是一組 PowerShell Cmdlet (例如 **Get-Date** 或 **Copy-Item**，可透過 PowerShell 主控台、指令碼、工作流程、Runbook 來使用) 和 PowerShell DSC 資源 (例如 WindowsFeature 或檔案，可透過 PowerShell DSC 組態來使用)。 所有 PowerShell 功能都是透過 Cmdlet 和 DSC 資源來公開，而每個 Cmdlet/DSC 資源都會受到 PowerShell 模組所支援，其中有許多模組會隨附在 PowerShell 本身之中。 例如，**Get-Date** Cmdlet 是 Microsoft.PowerShell.Utility PowerShell 模組的一部分、**Copy-Item** Cmdlet 是 Microsoft.PowerShell.Management PowerShell 模組的一部分，而「套件 DSC」資源是 PSDesiredStateConfiguration PowerShell 模組的一部分。 以上這兩個模組隨附在 PowerShell 之中。 但有許多 PowerShell 模組並未隨附為 PowerShell 的一部分，而是會隨第一方或第三方產品 (例如 System Center 2012 Configuration Manager) 或是透過廣大的 PowerShell 社群 (PowerShell 資源庫之類的地方) 來散發。 模組非常實用，因為它們能透過封裝起來的功能讓複雜的工作變得簡單。  您可以在 MSDN 上深入了解 [PowerShell 模組](https://msdn.microsoft.com/library/dd878324%28v=vs.85%29.aspx)。 

## <a name="what-is-an-azure-automation-integration-module"></a>什麼是 Azure 自動化整合模組？
整合模組與 PowerShell 模組不同。 它就是選擇性地多包含一個檔案的 PowerShell 模組，該檔案是一個中繼資料檔案，會指定要與模組在 Runbook 中的 Cmdlet 搭配使用的 Azure 自動化連線類型。 不論是否包含選擇性檔案，這些 PowerShell 模組都可以匯入到 Azure 自動化，以使其 Cmdlet 可供在 Runbook 內使用，以及使其 DSC 資源可供在 DSC 組態內使用。 Azure 自動化會在幕後儲存這些模組，並在執行 Runbook 作業和 DSC 編譯作業時將其載入 Azure 自動化沙箱，以在其中執行 Runbook 和編譯 DSC 組態。 模組中的任何 DSC 資源也會自動放在 Automation DSC 提取伺服器上，以供嘗試套用 DSC 組態的機器提取。  

我們在 Azure 自動化中隨附了許多現成可用的 Azure PowerShell 模組供您使用，因此您可以立即開始將 Azure 管理自動化，但您也可以針對任何您想要整合的系統、服務或工具匯入 PowerShell 模組。 

> [!NOTE]
> 某些模組會以自動化服務中的「全域模組 」隨附。 這些全域模組可在您建立自動化帳戶時使用，而我們有時會更新這些模組，自動將其推送到您的自動化帳戶。 如果您不想要進行自動更新，則一律可以自行匯入相同的模組，而這會優先隨附在服務中該模組的全域模組版本。 

您匯入整合模組封裝時所用的格式為，和模組同名的且副檔名為 .zip 的壓縮檔。 封裝中含有 Windows PowerShell 模組和任何支援檔案，包括資訊清單檔 (.psd1) (如果模組有此檔的話)。

如果模組應該包含 Azure 自動化連線類型，則它也必須包含名稱為 `<ModuleName>-Automation.json` 的檔案，以指定連線類型屬性。 這是放在 .zip 壓縮檔的模組資料夾內的 json 檔案，並包含要連線到模組所代表的系統或服務所必須具有之「連線」的欄位。 這最終會在 Azure 自動化中建立連線類型。 使用此檔案，您就可以為模組的連線類型設定欄位名稱、類型以及欄位是否應該加密和/或選用的。 以下是使用 json 檔案格式的範本︰

```
{ 
   "ConnectionFields": [
   {
      "IsEncrypted":  false,
      "IsOptional":  false,
      "Name":  "ComputerName",
      "TypeName":  "System.String"
   },
   {
      "IsEncrypted":  false,
      "IsOptional":  true,
      "Name":  "Username",
      "TypeName":  "System.String"
   },
   {
      "IsEncrypted":  true,
      "IsOptional":  false,
      "Name":  "Password",
   "TypeName":  "System.String"
   }],
   "ConnectionTypeName":  "DataProtectionManager",
   "IntegrationModuleName":  "DataProtectionManager"
}
```

如果您已部署 Service Management Automation 並為自動化 Runbook 建立整合模組套件，則應該熟悉此範本。 

## <a name="authoring-best-practices"></a>撰寫最佳做法
即使整合模組本來是 PowerShell 模組，當您在撰寫 PowerShell 模組時，我們仍建議您考慮一些事項，以便讓它在 Azure 自動化中發揮最大效用。 其中有些考慮事項是 Azure 自動化特有的，有些則只適用於讓模組能在 PowerShell 工作流程中良好運作，而不論您是否使用自動化。 

1. 在模組中加入每個 Cmdlet 的概要、描述和說明 URI。 在 PowerShell 中，您可以為 Cmdlet 定義特定說明資訊，以讓使用者透過 **Get-Help** Cmdlet 獲得其使用說明。 例如，以下是如何為 .psm1 檔案中所撰寫的 PowerShell 模組定義概要和說明 URI。<br>  
   
    ```
    <#
        .SYNOPSIS
         Gets all outgoing phone numbers for this Twilio account 
    #>
    function Get-TwilioPhoneNumbers {
    [CmdletBinding(DefaultParameterSetName='SpecifyConnectionFields', `
    HelpUri='http://www.twilio.com/docs/api/rest/outgoing-caller-ids')]
    param(
       [Parameter(ParameterSetName='SpecifyConnectionFields', Mandatory=$true)]
       [ValidateNotNullOrEmpty()]
       [string]
       $AccountSid,
   
       [Parameter(ParameterSetName='SpecifyConnectionFields', Mandatory=$true)]
       [ValidateNotNullOrEmpty()]
       [string]
       $AuthToken,
   
       [Parameter(ParameterSetName='UseConnectionObject', Mandatory=$true)]
       [ValidateNotNullOrEmpty()]
       [Hashtable]
       $Connection
    )
   
    $cred = CreateTwilioCredential -Connection $Connection -AccountSid $AccountSid -AuthToken $AuthToken
   
    $uri = "$TWILIO_BASE_URL/Accounts/" + $cred.UserName + "/IncomingPhoneNumbers"
   
    $response = Invoke-RestMethod -Method Get -Uri $uri -Credential $cred
   
    $response.TwilioResponse.IncomingPhoneNumbers.IncomingPhoneNumber
    }
    ```
   <br> 提供此資訊不只能在 PowerShell 主控台中透過 **Get-Help** Cmdlet 來顯示此說明，也會在 Azure 自動化內公開這個說明功能。  例如，在製作 Runbook 期間插入活動時。 按一下 [檢視詳細說明]，就會在用來存取 Azure 自動化之 Web 瀏覽器的另一個索引標籤中，開啟說明 URI。<br>![整合模組說明](media/automation-integration-modules/automation-integration-module-activitydesc.png)
1. 如果針對遠端系統來執行模組，

    a. 它應該包含整合模組中繼資料檔案，以定義用來連線到該遠端系統所需的資訊，也就是連線類型。  
    b. 模組中的每個 Cmdlet 應該要能夠採用連線物件 (該連線類型的執行個體) 來做為參數。  

    如果您允許將具有連線類型之欄位的物件，當做參數來傳遞至 Cmdlet，則模組中的 Cmdlet 會變得更容易使用 Azure 自動化。 如此一來，使用者就不必在每次呼叫 Cmdlet 時，將連線資產的參數對應至 Cmdlet 的相對應參數。 根據上述的 Runbook 範例，它使用稱為 CorpTwilio 的 Twilio 連線資產來存取 Twilio，並傳回帳戶中的所有電話號碼。  請注意它如何將連線的欄位對應至 Cmdlet 的參數？<br>
   
    ```
    workflow Get-CorpTwilioPhones
    {
      $CorpTwilio = Get-AutomationConnection -Name 'CorpTwilio'
   
      Get-TwilioPhoneNumbers 
        -AccountSid $CorpTwilio.AccountSid  
        -AuthToken $CorptTwilio.AuthToken
    }
    ```
  
    更容易且更好的處理方法是直接將連線物件傳遞給 Cmdlet -
   
    ```
    workflow Get-CorpTwilioPhones
    {
      $CorpTwilio = Get-AutomationConnection -Name 'CorpTwilio'
   
      Get-TwilioPhoneNumbers -Connection $CorpTwilio
    }
    ```
   
    您可以透過讓 Cmdlet 接受直接將連線物件當做參數，而非只是參數的連線欄位，來讓 Cmdlet 具有這樣的行為。 通常您會想讓每個 Cmdlet 都設定參數，讓未使用 Azure 自動化的使用者可以直接呼叫 Cmdlet，而不必建構雜湊表來作為連線物件。 下面的參數集 **SpecifyConnectionFields** 可用來逐一傳遞連線欄位屬性。 **UseConnectionObject** 則可讓您直接傳遞連線。 如您所見， [Twilio PowerShell 模組](https://gallery.technet.microsoft.com/scriptcenter/Twilio-PowerShell-Module-8a8bfef8) 中的 Send-TwilioSMS Cmdlet 允許以任一方式傳遞︰ 
   
    ```
    function Send-TwilioSMS {
      [CmdletBinding(DefaultParameterSetName='SpecifyConnectionFields', `
      HelpUri='http://www.twilio.com/docs/api/rest/sending-sms')]
      param(
         [Parameter(ParameterSetName='SpecifyConnectionFields', Mandatory=$true)]
         [ValidateNotNullOrEmpty()]
         [string]
         $AccountSid,
   
         [Parameter(ParameterSetName='SpecifyConnectionFields', Mandatory=$true)]
         [ValidateNotNullOrEmpty()]
         [string]
         $AuthToken,
   
         [Parameter(ParameterSetName='UseConnectionObject', Mandatory=$true)]
         [ValidateNotNullOrEmpty()]
         [Hashtable]
         $Connection
   
       )
    }
    ```
   <br>
1. 為模組中的所有 Cmdlet 定義輸出類型。 為 Cmdlet 定義輸出類型，可讓設計階段 IntelliSense 協助您判斷 Cmdlet 的輸出屬性，以供在撰寫期間使用。 在圖形化撰寫自動化 Runbook 期間，它會特別有幫助，因為設計階段的知識是讓模組的使用者獲得容易使用體驗的關鍵。<br><br> ![圖形化 Runbook 輸出類型](media/automation-integration-modules/runbook-graphical-module-output-type.png)<br> 這類似於 Cmdlet 在 PowerShell ISE 中的輸出的「自動提示」功能，但不需要加以執行。<br><br> ![POSH IntelliSense](media/automation-integration-modules/automation-posh-ise-intellisense.png)<br>
1. 模組中的 Cmdlet 不應該採用複雜物件類型來做為參數。 PowerShell 工作流程與 PowerShell 的不同之處在於，它會以還原序列化的形式儲存複雜類型。 基本類型會保持基本，但複雜類型則會轉換為已還原序列化的版本，基本上來說也就是屬性包。 例如，如果您在 Runbook 使用 **Get-Process** Cmdlet (或任何類似用途的 PowerShell 工作流程)，它會傳回類型為 [Deserialized.System.Diagnostic.Process] 的物件，而非預期的 [System.Diagnostic.Process] 類型。 這個類型擁有和非還原序列化類型相同的屬性，但沒有任何方法。 而且如果您嘗試將此值作為參數傳遞至 Cmdlet，而此 Cmdlet 預期此參數要有 [System.Diagnostic.Process] 值，則您會收到下列錯誤︰*「無法處理參數 'process' 的引數轉換。錯誤：「無法將類型為 "Deserialized.System.Diagnostics.Process" 的 "System.Diagnostics.Process (CcmExec)" 值，轉換為 "System.Diagnostics.Process" 類型。」*   這是因為預期的 [System.Diagnostic.Process] 類型和給定的 [Deserialized.System.Diagnostic.Process] 類型不相符。 此問題的解決方式是確保模組的 Cmdlet 不會採用複雜類型來做為參數。 以下是錯誤的處理方式。
   
    ```
    function Get-ProcessDescription {
      param (
            [System.Diagnostic.Process] $process
      )
      $process.Description
    }
    ``` 
    <br>
    以下則是正確的方式，採用可由 Cmdlet 在內部使用的原始物件，來捕捉複雜物件並加以使用。 既然 Cmdlet 會在 PowerShell 環境中執行，而非 PowerShell 工作流程中，因此在 Cmdlet 內，$process 會變成正確的 [System.Diagnostic.Process] 類型。  
   
    ```
    function Get-ProcessDescription {
      param (
            [String] $processName
      )
      $process = Get-Process -Name $processName
   
      $process.Description
    }
    ```
   <br>
   Runbook 中的連線資產是雜湊表，其為複雜類型，然而這些雜湊表似乎可以傳遞給 Cmdlet 供 –Connection 參數完美使用，而不會發生轉換例外狀況。 技術上來說，某些 PowerShell 類型能夠從其序列化形式正確轉換成還原序列化形式，因此可以傳遞給 Cmdlet 以供接受非還原序列化類型的參數使用。 雜湊表便是其中之一。 模組作者所定義的類型也有可能以可正確還原序列化的方式來實作，但必須考量一些取捨。 此類型需要有預設建構函式、其所有公用的屬性和 PSTypeConverter。 不過，對於模組作者未擁有的已定義類型，則沒有辦法加以「修正」，因此才會建議所有參數全都避免使用複雜類型。 Runbook 製作提示︰如果 Cmdlet 因為某些原因需要採用複雜類型的參數，或要使用他人需要複雜類型參數的模組，則在 PowerShell 工作流程 Runbook 中和本機 PowerShell 內的 PowerShell 工作流程中，因應措施是將會產生複雜類型的 Cmdlet 和在相同 InlineScript 活動中使用複雜類型的 Cmdlet 包裝起來。 由於 InlineScript 會以 PowerShell 形式而非 PowerShell 工作流程形式來執行其內容，產生複雜類型 Cmdlet 會產生該正確類型，而不會產生還原序列化的複雜類型。
1. 讓模組中的所有 Cmdlet 變成無狀態。 PowerShell 工作流程會在不同工作階段執行工作流程中所呼叫的每個 Cmdlet。 這表示任何相依於同一模組中其他 Cmdlet 所建立/修改的工作階段狀態的 Cmdlet，將不會在 PowerShell 工作流程 Runbook 中運作。  以下是不該做之事情的範例：
   
    ```
    $globalNum = 0
    function Set-GlobalNum {
       param(
           [int] $num
       )
   
       $globalNum = $num
    }
    function Get-GlobalNumTimesTwo {
       $output = $globalNum * 2
   
       $output
    }
    ```
   <br>
1. 模組應該完全包含在可 Xcopy 的封裝中。 當 Runbook 需要執行時，Azure 自動化模組會散發到自動化沙箱中，因此它們需要獨立於其執行所在的主機之外單獨運作。 其所代表的意義是，您應該能夠壓縮模組封裝，將它移至擁有相同或較新 PowerShell 版本的任何其他主機，並讓它在匯入到該主機的 PowerShell 環境時正常運作。 為了達成此一情況，模組不應該相依於模組資料夾 (匯入至 Azure 自動化時遭到壓縮的資料夾) 以外的任何檔案，或相依於主機上的任何唯一的登錄設定，例如產品安裝時所設定的登錄設定。 若未遵循此最佳做法，模組在 Azure 自動化中將無法使用。  

## <a name="next-steps"></a>後續步驟

* 若要開始使用 PowerShell 工作流程 Runbook，請參閱 [我的第一個 PowerShell 工作流程 Runbook](automation-first-runbook-textual.md)
* 若要深入了解如何建立 PowerShell 模組，請參閱 [撰寫 Windows PowerShell 模組](https://msdn.microsoft.com/library/dd878310%28v=vs.85%29.aspx)

