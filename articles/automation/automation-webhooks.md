---
title: 使用 Webhook 啟動 Azure 自動化 Runbook
description: 可讓用戶端透過 HTTP 呼叫在 Azure 自動化中啟動 Runbook 的 Webhook。  本文說明如何建立 Webhook，以及如何進行呼叫以啟動 Runbook。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 06/04/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: a65a0b8e054b1d0bb6cd4cbeb2daf9be2b132a9e
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304523"
---
# <a name="starting-an-azure-automation-runbook-with-a-webhook"></a>使用 Webhook 啟動 Azure 自動化 Runbook

*Webhook* 可讓您在 Azure 自動化中透過單一 HTTP 要求啟動特定的 Runbook。 這可讓 Azure DevOps、GitHub、Azure Log Analytics 或自訂應用程式等外部服務不需使用 Azure 自動化 API 實作完整的解決方案，即可啟動 Runbook。  
![WebhooksOverview](media/automation-webhooks/webhook-overview-image.png)

您可以透過 [在 Azure 自動化中啟動 Runbook](automation-starting-a-runbook.md)

## <a name="details-of-a-webhook"></a>Webhook 的詳細資料

下表描述您必須為 Webhook 設定的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| 名稱 |您可以為 Webhook 提供任何想要的名稱，因為這並不會向用戶端公開。 該名稱僅供您用來識別 Azure 自動化中的 Runbook。 <br> 最佳做法是您給予 Webhook 的名稱應該與要使用它的用戶端相關。 |
| URL |Webhook 的 URL 是一種唯一性的位址，即用戶端用來呼叫 HTTP POST 以啟動連結至 Webhook的 Runbook。 當您建立 Webhook 時其會自動產生。 您無法指定自訂 URL。 <br> <br> URL 包含可讓協力廠商系統不需進一步驗證即可叫用 Runbook 的安全性權杖。 基於這個原因，應該將其視為一種密碼。 基於安全性原因，您僅能於 Webhook 建立時在 Azure 入口網站中檢視 URL。 請在安全的位置記下 URL 以供日後使用。 |
| 到期日期 |例如憑證，每個 Webhook 都會有一個到期日期，到期後便無法再使用。 此到期日期可在 Webhook 建立後加以修改。 |
| 已啟用 |建立 Runbook 時 Webhook 會預設為啟用。 如果您將其設定為 [停用]，則任何用戶端皆無法使用。 當您建立 Webhook 時或在建立後的任何時候，您可以設定 [ **啟用** ] 屬性。 |

### <a name="parameters"></a>參數

Webhook 可以定義由該 Webhook 啟動 Runbook 時所使用的 Runbook 參數值。 Webhook 必須包含任何 Runbook 的強制參數值，且可能包含選擇性的參數值。 您可以修改設定至 Webhoook 的參數值，甚至是在建立 Webhook 之後也可以進行修改。 多個 Webhook 連結至單一 Runbook 時，可以個別使用不同的參數值。

當用戶端使用 Webhook 啟動 Runbook 時，其無法覆寫 Webhook 中定義的參數值。 若要從用戶端接收資料，Runbook 可以接受 [object] 類型且稱為 **$WebhookData** 的單一參數，其中包含用戶端在 POST 要求中所包含的資料。

![Webhookdata 屬性](media/automation-webhooks/webhook-data-properties.png)

**$WebhookData** 物件具有下列屬性：

| 屬性 | 說明 |
|:--- |:--- |
| WebhookName |Webhook 的名稱。 |
| RequestHeader |雜湊表包含傳入 POST 要求的標頭。 |
| RequestBody |傳入 POST 要求的本文。 這會保留任何格式 (例如字串、JSON、XML 或表單) 編碼的資料。 Runbook 必須撰寫用來搭配預期的資料格式。 |

支援 **$WebhookData** 參數不需要 Webhook 的組態，且 Runbook 不需要接受其。 若 Runbook 並未定義參數，則會忽略從用戶端所傳送要求的任何詳細資料。

若您在建立 Webhook 時指定 $WebhookData 的值，即使用戶端在要求本文中並未包含任何資料，該值也會在 Webhook 使用來自用戶端 POST 要求的資料來啟動 Runbook 時遭到覆寫。 若您使用 Webhook 以外的方式啟動具有 $WebhookData 的 Runbook，則可以提供可讓 Runbook 辨識的 $WebhookData 值。 這個值應該是 [屬性](#details-of-a-webhook) 相同皆為 $Webhookdata 的物件，Runbook 如此即可正確地與其一起運作，就像是其使用 webhook 所傳遞的實際 WebhookData 一般。

例如，如果要從 Azure 入口網站啟動下列 Runbook，並想要傳遞一些 WebhookData 範例進行測試，由於 WebhookData 是一個物件，因此應在 UI 中以 JSON 傳遞。

![來自 UI 的 WebhookData 參數](media/automation-webhooks/WebhookData-parameter-from-UI.png)

針對下述 Runbook，如果您具有下列適用於 WebhookData 參數的屬性：

* WebhookName: *MyWebhook*
* RequestBody: *[{'ResourceGroup': 'myResourceGroup','Name': 'vm01'},{'ResourceGroup': 'myResourceGroup','Name': 'vm02'}]*

則您應在 UI 中針對 WebhookData 參數傳遞下列 JSON 值。 下列具有歸位字元與新行字元的範例會符合從 Webhook 傳入的格式。

```json
{"WebhookName":"mywebhook","RequestBody":"[\r\n {\r\n \"ResourceGroup\": \"vm01\",\r\n \"Name\": \"vm01\"\r\n },\r\n {\r\n \"ResourceGroup\": \"vm02\",\r\n \"Name\": \"vm02\"\r\n }\r\n]"}
```

![啟動來自 UI 的 WebhookData 參數](media/automation-webhooks/Start-WebhookData-parameter-from-UI.png)

> [!NOTE]
> 所有輸入參數的值會使用 Runbook 工作進行記錄。 這表示，任何由用戶端在 Webhook 要求中提供的輸入將會進行記錄，並可讓任何有自動化工作存取權的人使用。  基於這個原因，您在 Webhook 呼叫中包含機密資訊時應該特別謹慎。

## <a name="security"></a>安全性

Webhook 的安全性仰賴其 URL 的隱私權，其中包含允許叫用它的安全性權杖。 只要針對正確的 URL 進行要求，Azure 自動化就不會對要求執行任何驗證。 基於這個原因，若不使用驗證要求的替代方式時，Webhooks 不應用於執行高度機密功能的 Runbook。

您可以透過檢查 $WebhookData 參數的 **WebhookName** 屬性，在 Runbook 中包含邏輯來判斷其是否由 Webhook 所呼叫。 Runbook 可以透過尋找 **RequestHeader** 或 **RequestBody** 屬性中的特定資訊來執行進一步的驗證。

另一個策略是當 Runbook 收到 Webhook 要求時，使其執行某些外部條件的驗證。 例如，當 GitHub 儲存機制出現新的認可時，考慮使用由 GitHub 呼叫的 Runbook。 Runbook 可能會連線到 GitHub，以驗證已在繼續之前出現新的認可。

## <a name="creating-a-webhook"></a>建立 Webhook

使用下列程序在 Azure 入口網站中建立連結至 Runbook 的全新 Webhook。

1. 從 Azure 入口網站的 [Runbook] 頁面中，按一下 Webhook 要開始檢視其詳細資料頁面的 Runbook。 請確定 Runbook 的 [狀態] 為 [已發佈]。
2. 按一下頁面頂端的 [Webhook] 以開啟 [新增 Webhook] 頁面。
3. 按一下 [建立新的 Webhook] 以開啟 [建立 Webhook] 頁面。
4. 指定 Webhook 的 [名稱]、[到期日期] 以及是否應該啟用。 請參閱 [Webhook 的詳細資料](#details-of-a-webhook) 以取得這些屬性的詳細資訊。
5. 按一下複製圖示，然後按 Ctrl + C 以複製 Webhook 的 URL。 然後將其記錄在安全的地方。 **一旦您建立 Webhook，即無法再次擷取 URL。**

   ![Webhook URL](media/automation-webhooks/copy-webhook-url.png)

1. 按一下 [參數]  來提供 Runbook 的參數值。 如果 Runbook 有強制參數，除非提供值，否則您無法建立 Webhook。
1. 按一下 [ **建立** ] 來建立 Webhook。

## <a name="using-a-webhook"></a>使用 Webhook

建立 Webhook 之後若要使用它，您的用戶端應用程式必須使用 Webhook 的 URL 來發出 HTTP POST。 Webhook 的語法為下列格式：

```http
http://<Webhook Server>/token?=<Token Value>
```

用戶端會從 POST 要求中接收下列其中一個傳回碼。

| 代碼 | 文字 | 說明 |
|:--- |:--- |:--- |
| 202 |已接受 |已接受要求，且 Runbook 已經成功排入佇列。 |
| 400 |不正確的要求 |基於下列其中一個因素而不接受此要求： <ul> <li>Webhook 已過期。</li> <li>Webhook 已停用。</li> <li>URL 中的權杖無效。</li>  </ul> |
| 404 |找不到 |基於下列其中一個因素而不接受此要求： <ul> <li>找不到該 Webhook。</li> <li>找不到該 Runbook。</li> <li>找不到帳戶。</li>  </ul> |
| 500 |內部伺服器錯誤 |URL 有效，但發生錯誤。 請重新提交要求。 |

假設要求成功，Webhook 會回應包含 JSON 格式的工作識別碼，如下所示。 它會包含單一的工作識別碼，但是 JSON 格式允許未來可能的增強功能。

```json
{"JobIds":["<JobId>"]}
```

用戶端無法從 Webhook 判斷 Runbook 的工作何時完成或完成狀態。 這項資訊可使用工作識別碼搭配其他方法 (例如 [Windows PowerShell](https://docs.microsoft.com/powershell/module/servicemanagement/azure/get-azureautomationjob) 或 [Azure 自動化 API](/rest/api/automation/job)) 來判斷。

## <a name="sample-runbook"></a>範例 Runbook

下列範例 Runbook 會接受 Webhook 資料，並啟動要求本文中指定的虛擬機器。 若要測試此 Runbook，請在您自動化帳戶的 [Runbook] 下，按一下 [+ 加入 Runbook]。 如果您不知道如何建立 Runbook，請參閱[建立 Runbook](automation-quickstart-create-runbook.md)。

```powershell
param
(
    [Parameter (Mandatory = $false)]
    [object] $WebhookData
)

# If runbook was called from Webhook, WebhookData will not be null.
if ($WebhookData) {

    # Retrieve VM's from Webhook request body
    $vms = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)

    # Authenticate to Azure by using the service principal and certificate. Then, set the subscription.

    Write-Output "Authenticating to Azure with service principal and certificate"
    $ConnectionAssetName = "AzureRunAsConnection"
    Write-Output "Get connection asset: $ConnectionAssetName" 

    $Conn = Get-AutomationConnection -Name $ConnectionAssetName
            if ($Conn -eq $null)
            {
                throw "Could not retrieve connection asset: $ConnectionAssetName. Check that this asset exists in the Automation account."
            }
            Write-Output "Authenticating to Azure with service principal." 
            Add-AzureRmAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint | Write-Output

        # Start each virtual machine
        foreach ($vm in $vms)
        {
            $vmName = $vm.Name
            Write-Output "Starting $vmName"
            Start-AzureRMVM -Name $vm.Name -ResourceGroup $vm.ResourceGroup
        }
}
else {
    # Error
    write-Error "This runbook is meant to be started from an Azure alert webhook only."
}
```

## <a name="test-the-example"></a>測試範例

下列範例使用 Windows PowerShell 搭配 Webhook 來啟動 Runbook。 任何可發出 HTTP 要求的語言都可以使用 Webhook；Windows PowerShell 在這裡用來作為範例。

Runbook 預期在要求的主體中有 JSON 格式的虛擬機器清單。

```azurepowershell-interactive
$uri = "<webHook Uri>"

$vms  = @(
            @{ Name="vm01";ResourceGroup="vm01"},
            @{ Name="vm02";ResourceGroup="vm02"}
        )
$body = ConvertTo-Json -InputObject $vms

$response = Invoke-RestMethod -Method Post -Uri $uri -Body $body
$jobid = (ConvertFrom-Json ($response.Content)).jobids[0]
```

下列範例顯示的要求本文可在 **WebhookData** 的 **RequestBody** 屬性中提供給 Runbook 使用。 格式化為 JSON 是因為其格式是要求的主體中所包含之格式。

```json
[
    {
        "Name":  "vm01",
        "ResourceGroup":  "myResourceGroup"
    },
    {
        "Name":  "vm02",
        "ResourceGroup":  "myResourceGroup"
    }
]
```

下圖顯示由 Windows PowerShell 送出的要求以及產生的回應。 工作識別碼從回應中擷取，再轉換為字串。

![Webhook 按鈕](media/automation-webhooks/webhook-request-response.png)

## <a name="next-steps"></a>後續步驟

* 若要了解如何使用 Azure 自動化來對 Azure 警示採取動作，請參閱[使用警示來觸發 Azure 自動化 Runbook](automation-create-alert-triggered-runbook.md)。
