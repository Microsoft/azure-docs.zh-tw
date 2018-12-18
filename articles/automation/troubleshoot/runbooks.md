---
title: 針對 Azure 自動化 Runbook 的錯誤進行疑難排解
description: 了解如何針對 Azure 自動化 Runbook 的錯誤進行疑難排解
services: automation
author: georgewallace
ms.author: gwallace
ms.date: 07/13/2018
ms.topic: conceptual
ms.service: automation
manager: carmonm
ms.openlocfilehash: b02f1b04756f1e3f01426e58c5f8c625cb746f05
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47163897"
---
# <a name="troubleshoot-errors-with-runbooks"></a>針對 Runbook 的錯誤進行疑難排解

## <a name="authentication-errors-when-working-with-azure-automation-runbooks"></a>使用 Azure 自動化 Runbook 時的驗證錯誤

### <a name="sign-in-failed"></a>案例：登入 Azure 帳戶失敗

#### <a name="issue"></a>問題

使用 `Add-AzureAccount` 或 `Connect-AzureRmAccount` Cmdlet 時，收到下列錯誤
：

```
Unknown_user_type: Unknown User Type
```

#### <a name="cause"></a>原因

如果認證資產名稱無效，或您用來設定自動化認證資產的使用者名稱和密碼無效，就會發生此錯誤。

#### <a name="resolution"></a>解決方案

為了判斷錯誤原因，請執行下列步驟：  

1. 請確定您沒有任何特殊字元，包括用來連接到 Azure 的「自動化」認證資產名稱中的 **@** 符號。  
2. 請檢查您可以使用儲存在本機 PowerShell ISE 編輯器中的 Azure 自動化認證的使用者名稱和密碼。 您可以藉由在 PowerShell ISE 中執行下列 Cmdlet 來完成這項操作：  

   ```powershell
   $Cred = Get-Credential  
   #Using Azure Service Management
   Add-AzureAccount –Credential $Cred  
   #Using Azure Resource Manager  
   Connect-AzureRmAccount –Credential $Cred
   ```

3. 如果您的驗證在本機失敗，這表示您尚未正確設定 Azure Active Directory 認證。 請參閱 [Authenticating to Azure using Azure Active Directory (使用 Azure Active Directory 對 Azure 進行驗證)](https://azure.microsoft.com/blog/azure-automation-authenticating-to-azure-using-azure-active-directory/) 部落格文章，以正確設定 Azure Active Directory 帳戶。  

4. 如果這似乎為暫時性錯誤，請嘗試將重試邏輯新增至您的驗證常式，讓驗證變得更強固。

   ```powershell
   # Get the connection "AzureRunAsConnection"
   $connectionName = "AzureRunAsConnection"
   $servicePrincipalConnection = Get-AutomationConnection -Name $connectionName

   $logonAttempt = 0
   $logonResult = $False

   while(!($connectionResult) -And ($logonAttempt -le 10))
   {
   $LogonAttempt++
   # Logging in to Azure...
   $connectionResult = Connect-AzureRmAccount `
      -ServicePrincipal `
      -TenantId $servicePrincipalConnection.TenantId `
      -ApplicationId $servicePrincipalConnection.ApplicationId `
      -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint

   Start-Sleep -Seconds 30
   }
   ```

### <a name="unable-to-find-subscription"></a>案例：找不到 Azure 訂用帳戶

#### <a name="issue"></a>問題

使用 `Select-AzureSubscription` 或 `Select-AzureRmSubscription` Cmdlet 時，收到下列錯誤：

```
The subscription named <subscription name> cannot be found.
```

#### <a name="error"></a>Error

如果訂用帳戶名稱無效，或嘗試取得訂用帳戶詳細資料的 Azure Active Directory 使用者未被設定為訂用帳戶的系統管理員，就會發生此錯誤。

#### <a name="resolution"></a>解決方案

如要判斷您是否已正確地向 Azure 驗證自己的身分，以及您是否有嘗試選取之訂用帳戶的存取權，請執行下列步驟：  

1. 確認您在執行 **Select-AzureSubscription** Cmdlet 之前，已經執行 **Add-AzureAccount** Cmdlet。  
2. 如果您仍然看到此錯誤訊息，請在 **Add-AzureAccount** Cmdlet 之後新增 **-AzureRmContext** 參數，以修改您的程式碼，然後執行此程式碼。

   ```powershell
   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzureRmAccount -ServicePrincipal -Tenant $Conn.TenantID `
-ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $context = Get-AzureRmContext

   Get-AzureRmVM -ResourceGroupName myResourceGroup -AzureRmContext $context
   ```

### <a name="auth-failed-mfa"></a>案例：對 Azure 的驗證失敗，因為已啟用多重要素驗證

#### <a name="issue"></a>問題

使用 Azure 使用者名稱與密碼向 Azure 進行驗證時，收到下列錯誤：

```
Add-AzureAccount: AADSTS50079: Strong authentication enrollment (proof-up) is required
```

#### <a name="cause"></a>原因

如果您的 Azure 帳戶使用多重要素驗證，您就無法使用 Azure Active Directory 使用者來向 Azure 進行驗證。 相反地，您必須使用憑證或服務主體來向 Azure 驗證。

#### <a name="resolution"></a>解決方案

若要搭配 Azure 傳統部署模型 Cmdlet 使用憑證，請參閱 [creating and adding a certificate to manage Azure services](http://blogs.technet.com/b/orchestrator/archive/2014/04/11/managing-azure-services-with-the-microsoft-azure-automation-preview-service.aspx) (建立及新增憑證來管理 Azure 服務)。 若要搭配 Azure Resource Manager Cmdlet 來使用服務主體，請參閱[使用 Azure 入口網站來建立服務主體](../../azure-resource-manager/resource-group-create-service-principal-portal.md)和[使用 Azure Resource Manager 驗證服務主體](../../azure-resource-manager/resource-group-authenticate-service-principal.md)

## <a name="common-errors-when-working-with-runbooks"></a>使用 Runbook 時的常見錯誤

### <a name="task-was-cancelled"></a>案例：Runbook 失敗，錯誤為：工作已取消

#### <a name="issue"></a>問題

您的 Runbook 失敗，並具有類似下列範例的錯誤：

```
Exception: A task was canceled.
```

#### <a name="cause"></a>原因

此錯誤可能是因為使用過期的 Azure 模組所造成。

#### <a name="resolution"></a>解決方案

此錯誤可以透過將 Azure 模組更新至最新版本來解決。

在您的自動化帳戶中，按一下 [模組]，然後按一下 [更新 Azure 模組]。 更新需要花費大約 15 分鐘的時間。完成之後，請重新執行先前失敗的 Runbook。 若要深入了解如何更新模組，請參閱[更新 Azure 自動化中的 Azure 模組](../automation-update-azure-modules.md)。

### <a name="child-runbook-auth-failure"></a>案例：處理多個訂用帳戶時，子 Runbook 失敗

#### <a name="issue"></a>問題

使用 `Start-AzureRmRunbook` 執行子 Runbook 時，子 Runbook 無法管理 Azure 資源。

#### <a name="cause"></a>原因

子 Runbook 在執行時未使用正確的內容。

#### <a name="resolution"></a>解決方案

在使用多個訂用帳戶時，若叫用子 Runbook，可能會遺失訂用帳戶內容。 若要確保訂用帳戶的內容會傳遞至子 Runbook，請將 `AzureRmContext` 參數新增至 Cmdlet，並將內容傳遞給它。

```azurepowershell-interactive
# Connect to Azure with RunAs account
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Add-AzureRmAccount `
    -ServicePrincipal `
    -TenantId $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzureContext = Select-AzureRmSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID

$params = @{"VMName"="MyVM";"RepeatCount"=2;"Restart"=$true}

Start-AzureRmAutomationRunbook `
    –AutomationAccountName 'MyAutomationAccount' `
    –Name 'Test-ChildRunbook' `
    -ResourceGroupName 'LabRG' `
    -AzureRmContext $AzureContext `
    –Parameters $params –wait
```

### <a name="not-recognized-as-cmdlet"></a>案例：因為遺失 Cmdlet，所以 Runbook 失敗

#### <a name="issue"></a>問題

您的 Runbook 失敗，並具有類似下列範例的錯誤：

```
The term 'Connect-AzureRmAccount' is not recognized as the name of a cmdlet, function, script file, or operable program.  Check the spelling of the name, or if the path was included verify that the path is correct and try again.
```

#### <a name="cause"></a>原因

這個錯誤可能是下列原因所造成：

1. 包含 Cmdlet 的模組未匯入到自動化帳戶
2. 包含 Cmdlet 的模組已匯入，但已過期

#### <a name="resolution"></a>解決方案

藉由完成下列其中一項工作，就可以解決此錯誤：

如果模組是 Azure 模組，請參閱[如何更新 Azure 自動化中的 Azure PowerShell 模組](../automation-update-azure-modules.md)以了解如何在您的自動化帳戶中更新模組。

如果是個別的模組，請確定模組已匯入到您的自動化帳戶中。

### <a name="job-attempted-3-times"></a>案例：已嘗試啟動 Runbook 作業三次，但無法每次都順利啟動

#### <a name="issue"></a>問題

您的 Runbook 失敗，錯誤為：

```
The job was tried three times but it failed
```

#### <a name="cause"></a>原因

這個錯誤可能是下列原因所造成：

1. 記憶體限制。 配置給沙箱[自動化服務限制](../../azure-subscription-service-limits.md#automation-limits)的記憶體數量具有已知限制，因此如果使用超過 400 MB 的記憶體，作業可能就會失敗。

1. 網路通訊端。 Azure 沙箱受限於如[自動化服務限制](../../azure-subscription-service-limits.md#automation-limits)所述的 1000 個並存網路通訊端。

1. 模組不相容。 如果模組相依性不正確，可能會發生這個錯誤。不正確時，您的 Runbook 通常會傳回「找不到命令」或「無法繫結參數」訊息。

#### <a name="resolution"></a>解決方案

下列任何一個解決方案都可以修正此問題：

* 在記憶體限制內運作的建議方法是分割多個 Runbook 之間的工作負載、不要處理記憶體中的太多資料、不要寫入來自 Runbook 的不必要輸出，或考慮您寫入 PowerShell 工作流程 Runbook 的檢查點數目。 您可以使用 clear 方法 (例如 `$myVar.clear()`) 來清除變數，並使用 `[GC]::Collect()` 立即執行記憶體回收，這會減少您的 Runbook 在執行階段的記憶體使用量。

* 遵循[如何更新 Azure 自動化中的 Azure PowerShell 模組](../automation-update-azure-modules.md)步驟來更新 Azure 模組。  

* 另一個解決方案是在[混合式 Runbook 背景工作角色](../automation-hrw-run-runbooks.md)上執行 Runbook。 混合式背景工作角色未受限於 Azure 沙箱的記憶體和網路限制。

### <a name="fails-deserialized-object"></a>案例：Runbook 因還原序列化物件而失敗

#### <a name="issue"></a>問題

您的 Runbook 失敗，錯誤為：

```
Cannot bind parameter <ParameterName>.

Cannot convert the <ParameterType> value of type Deserialized <ParameterType> to type <ParameterType>.
```

#### <a name="cause"></a>原因

如果您的 Runbook 是 PowerShell 工作流程，它會以還原序列化格式儲存複雜物件，以在暫止工作流程時保存 Runbook 狀態。

#### <a name="resolution"></a>解決方案

下列三個解決方案的任何一個可以修正此問題：

1. 如果您要將複雜物件從某個 Cmdlet 傳送到另一個 Cmdlet，請將這些 Cmdlet 包裝在 InlineScript 中。
2. 從複雜物件傳遞您需要的名稱或值，而非傳遞整個物件。
3. 使用 PowerShell Runbook，而不是 PowerShell 工作流程 Runbook。

### <a name="quota-exceeded"></a>案例：Runbook 作業失敗，因為超過已配置的配額

#### <a name="issue"></a>問題

您的 Runbook 作業失敗，錯誤為：

```
The quota for the monthly total job run time has been reached for this subscription
```

#### <a name="cause"></a>原因

當作業執行超過您的帳戶的 500 分鐘免費配額時，會發生此錯誤。 這個配額會套用至所有類型的作業執行工作，例如，測試作業、從入口網站中開始作業、使用 Webhook 執行作業，使用 Azure 入口網站或您的資料中心排程作業以執行。 若要深入了解自動化的價格，請參閱[自動化價格](https://azure.microsoft.com/pricing/details/automation/)。

#### <a name="resolution"></a>解決方案

如果您每個月想要使用超過 500 分鐘的處理時間，您必須將您的訂用帳戶從免費層變更為基本層。 您可以採取下列步驟，升級至基本層：  

1. 登入您的 Azure 訂用帳戶：  
2. 選取您想要升級的自動化帳戶  
3. 按一下 [設定]  >  [定價]。
4. 按一下頁面底端的 [啟用]，將您的帳戶升級至**基本**層。

### <a name="cmdlet-not-recognized"></a>案例：執行 Runbook 時，無法辨識 Cmdlet

#### <a name="issue"></a>問題

您的 Runbook 作業失敗，錯誤為：

```
<cmdlet name>: The term <cmdlet name> is not recognized as the name of a cmdlet, function, script file, or operable program.
```

#### <a name="cause"></a>原因

當 PowerShell 引擎找不到您在 Runbook 中使用的 Cmdlet 時，就會發生此錯誤。 這可能是因為包含 Cmdlet 的模組不在此帳戶、與 Runbook 名稱有名稱衝突，或 Cmdlet 也存在於另一個模組且自動化無法解析名稱。

#### <a name="resolution"></a>解決方案

下列任何一個解決方案都可以修正此問題：  

* 請檢查是否輸入正確的 Cmdlet 名稱。  
* 請確定 Cmdlet 存在於您的自動化帳戶中且沒有衝突。 如要確認 Cmdlet 是否存在，請以編輯模式開啟 Runbook，然後搜尋您想要在程式庫中尋找的 Cmdlet，或執行 `Get-Command <CommandName>`。 一旦驗證 Cmdlet 可用於帳戶，且與其他 Cmdlet 或 Runbook 沒有名稱衝突，將其加入至畫布，並確保您在 Runbook 中使用有效的參數集。  
* 如果有名稱衝突，而且 Cmdlet 可以在兩個不同的模組中使用，您可以使用 Cmdlet 的完整名稱來解決此問題。 例如，您可以使用 **ModuleName\CmdletName**。  
* 如果您是以混合式背景工作角色群組身分在內部部署環境中執行 Runbook，則請確定 module/cmdlet 是安裝在裝載混合式背景工作角色的電腦上。

### <a name="evicted-from-checkpoint"></a>案例：長時間執行的 Runbook 不斷失敗，且伴隨例外狀況：「作業無法繼續執行，因為它不斷在相同的檢查點遭到撤銷」

#### <a name="issue"></a>問題

這是故意設計的行為，原因是針對 Azure 自動化中程序的「公平共用」監視，也就是如果 Runbook 的執行時間超過三小時，就會自動暫止。 不過，傳回的錯誤訊息不會提供「接下來該怎麼辦」的選項。

#### <a name="cause"></a>原因

有許多原因能造成 Runbook 暫停， 而大多數是因為發生錯誤。 舉例來說，Runbook 中未攔截到的例外狀況、網路失敗，或是執行 Runbook 的 Runbook 背景工作角色當機，都會造成 Runbook 暫停，並讓 Runbook 在恢復運作時從上一個檢查點開始。

#### <a name="resolution"></a>解決方案

能避免此問題的已記載解決方案，就是在工作流程中使用檢查點。 若要深入了解，請參閱[了解 PowerShell 工作流程](../automation-powershell-workflow.md#checkpoints)。 如需對於「公平共用」及檢查點更詳細的說明，請參閱這篇部落格文章：[在 Runbook 中使用檢查點](https://azure.microsoft.com/blog/azure-automation-reliable-fault-tolerant-runbook-execution-using-checkpoints/)。

### <a name="long-running-runbook"></a>情節：長時間執行的 Runbook 無法完成

#### <a name="issue"></a>問題

這是 Azure 沙箱中故意設計的行為，原因是針對 Azure 自動化中程序的「公平共用」監視，也就是如果 Runbook 的執行時間超過三小時，就會自動暫止。

#### <a name="cause"></a>原因

Runbook 已執行超過 Azure 沙箱中公平共用所允許的 3 小時限制

#### <a name="resolution"></a>解決方案

建議的解決方案是在[混合式 Runbook 背景工作角色](../automation-hrw-run-runbooks.md)上執行 Runbook。 混合式背景工作角色未受限於 Azure 沙箱的[公平共用](../automation-runbook-execution.md#fair-share) 3 小時 Runbook 限制。

## <a name="common-errors-when-importing-modules"></a>匯入模組時的常見錯誤

### <a name="module-fails-to-import"></a>案例：無法匯入模組，或無法在匯入之後執行 Cmdlet

#### <a name="issue"></a>問題

模組無法匯入或匯入成功，但沒有擷取到任何 Cmdlet。

#### <a name="cause"></a>原因

模組可能無法成功匯入 Azure 自動化的部分常見原因如下：

* 結構不符合自動化所需要的結構。
* 此模組是相依於尚未部署到您的自動化帳戶的另一個模組。
* 模組在資料夾中遺失其相依性。
* 您使用 `New-AzureRmAutomationModule` Cmdlet 來上傳模組，但您尚未提供完整的儲存路徑，或是尚未使用可公開存取的 URL 來載入模組。

#### <a name="resolution"></a>解決方案

下列任何一個解決方案都可以修正此問題：

* 確定模組遵循下列格式：ModuleName.Zip **->** 模組名稱或版本號碼 **->** (ModuleName.psm1、ModuleName.psd1)
* 開啟 .psd1 檔案，並且查看該模組是否有任何相依性。 如果有，請將這些模組上傳至自動化帳戶。
* 請確定任何參考的 .dll 會出現在模組資料夾。

## <a name="next-steps"></a>後續步驟

如果您看不到問題或無法解決問題，請瀏覽下列其中一個通道以取得更多支援：

* 透過 [Azure 論壇](https://azure.microsoft.com/support/forums/)獲得由 Azure 專家所提供的解答
* 與 [@AzureSupport](https://twitter.com/azuresupport) 連繫－專為改善客戶體驗而設的官方 Microsoft Azure 帳戶，協助 Azure 社群連接至適當的資源，像是解答、支援及專家等。
* 如果需要更多協助，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。