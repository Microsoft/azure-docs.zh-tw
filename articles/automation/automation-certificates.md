---
title: Azure 自動化中的憑證資產
description: 憑證可以安全地儲存在 Azure 自動化中，使得 Runbook 或 DSC 組態可以存取憑證，以向 Azure 和協力廠商資源進行驗證。  這篇文章說明憑證的詳細資料，以及如何以文字和圖形化編寫形式加以使用。
services: automation
ms.service: automation
ms.component: shared-capabilities
author: georgewallace
ms.author: gwallace
ms.date: 03/15/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: c13da6ff7c864ffa365dbad33d6eb0cf2e35fa42
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42141887"
---
# <a name="certificate-assets-in-azure-automation"></a>Azure 自動化中的憑證資產

憑證可以安全地儲存在 Azure 自動化中，使得 Runbook 或 DSC 組態可以透過使用 Azure Resource Manager 資源的 **Get-AzureRmAutomationCertificate** 活動來存取憑證。 這個功能可讓您建立使用憑證進行驗證的 Runbook 和 DSC 設定，或將它們新增至 Azure 或協力廠商資源。

>[!NOTE]
>Azure 自動化中的安全資產包括認證、憑證、連接和加密的變數。 這些資產都會經過加密，並使用為每個自動化帳戶產生的唯一金鑰儲存在 Azure 自動化中。 此金鑰會儲存在 Key Vault 中。 在儲存安全資產之前，系統會從 Key Vault 載入金鑰，然後用來加密資產。

## <a name="azurerm-powershell-cmdlets"></a>AzureRM PowerShell Cmdlet
針對 AzureRM，下表中的 Cmdlet 可透過 Windows PowerShell 來建立和管理自動化認證資產。 它們附屬於 [AzureRM.Automation 模組](/powershell/azure/overview)，而此模組可供在自動化 Runbook 和 DSC 設定中使用。

|Cmdlet|說明|
|:---|:---|
|[Get-AzureRmAutomationCertificate](https://docs.microsoft.com/powershell/module/azurerm.automation/get-azurermautomationcertificate)|擷取要在 Runbook 或 DSC 組態中使用的憑證相關資訊。 您只能從 Get-AutomationCertificate 活動擷取憑證本身。|
|[New-AzureRmAutomationCertificate](https://docs.microsoft.com/powershell/module/azurerm.automation/new-azurermautomationcertificate)|建立新的憑證到 Azure 自動化。|
[Remove-AzureRmAutomationCertificate](https://docs.microsoft.com/powershell/module/azurerm.automation/remove-azurermautomationcertificate)|從 Azure 自動化中移除憑證。|建立新的憑證到 Azure 自動化。
|[Set-AzureRmAutomationCertificate](https://docs.microsoft.com/powershell/module/azurerm.automation/set-azurermautomationcertificate)|設定現有的憑證，包括上傳憑證檔案和設定 .pfx 的密碼屬性。|
|[Add-AzureCertificate](https://msdn.microsoft.com/library/azure/dn495214.aspx)|上傳指定雲端服務的服務憑證。|

## <a name="activities"></a>活動
下表中的活動是用來存取 Runbook 和 DSC 設定中的憑證。

| 活動 | 說明 |
|:---|:---|
|Get-AutomationCertificate|取得要在 Runbook 或 DSC 組態中使用的憑證。 傳回 [System.Security.Cryptography.X509Certificates.X509Certificate2](https://msdn.microsoft.com/library/system.security.cryptography.x509certificates.x509certificate2.aspx) 物件。|

> [!NOTE] 
> 您應該避免在 Runbook 或 DSC 設定中 **Get-AutomationCertificate** 的 –Name 參數中使用變數，因為它會使在設計階段中探索 Runbook 或 DSC 設定與自動化變數之間的相依性變得複雜。

## <a name="python2-functions"></a>Python2 函式

下表中的函式用於存取 Python2 Runbook 中的憑證。

| 函式 | 說明 |
|:---|:---|
| automationassets.get_automation_certificate | 擷取憑證資產的相關資訊。 |

> [!NOTE]
> 您必須在 Python Runbook 的頂端匯入 **automationassets** 模組，才能存取資產函式。

## <a name="creating-a-new-certificate"></a>建立新憑證

建立新憑證時，您會將 cer 或 pfx 檔案上傳到 Azure 自動化。 如果您將憑證標示為可匯出，那麼您可以將它傳送到 Azure 自動化憑證存放區外部。 如果無法匯出，則只可以將它用在 Runbook 或 DSC 組態內的簽署。 Azure 自動化需要讓憑證擁有提供者：**Microsoft Enhanced RSA 與 AES 密碼編譯提供者**。

### <a name="to-create-a-new-certificate-with-the-azure-portal"></a>使用 Azure 入口網站建立新憑證

1. 從您的自動化帳戶，按一下 [資產] 圖格，以開啟 [資產] 刀鋒視窗。
1. 按一下 [憑證] 憑證，以開啟 [憑證] 刀鋒視窗。
1. 在分頁的頂端按一下 [ **加入憑證** ]。
1. 在 [ **名稱** ] 方塊中輸入憑證的名稱。
1. 若要瀏覽 .cer 或 .pfx 檔案，請按一下 [上傳憑證檔案] 下方的 [選取檔案]。 如果您選取 .pfx 檔案，請指定密碼，以及是否允許匯出。
1. 按一下 [ **建立** ] 以儲存新的憑證資產。

### <a name="to-create-a-new-certificate-with-windows-powershell"></a>使用 Windows PowerShell 建立新憑證

下列範例示範如何建立新的自動化憑證，並將其標示為可匯出。 這樣會匯入現有的 pfx 檔案。

```powershell-interactive
$certName = 'MyCertificate'
$certPath = '.\MyCert.pfx'
$certPwd = ConvertTo-SecureString -String 'P@$$w0rd' -AsPlainText -Force
$ResourceGroup = "ResourceGroup01"

New-AzureRmAutomationCertificate -AutomationAccountName "MyAutomationAccount" -Name $certName -Path $certPath –Password $certPwd -Exportable -ResourceGroupName $ResourceGroup
```

## <a name="using-a-certificate"></a>使用憑證

若要使用憑證，請使用 **Get-AutomationCertificate** 活動。 您不能使用 [Get-AzureRmAutomationCertificate](https://docs.microsoft.com/powershell/module/azurerm.automation/get-azurermautomationcertificate?view=azurermps-6.6.0) Cmdlet，因為它會傳回憑證資產的相關資訊，而不是憑證本身。

### <a name="textual-runbook-sample"></a>文字式 Runbook 範例

下列範例程式碼示範如何將憑證新增到 Runbook 中的雲端服務。 在此範例中，密碼是擷取自加密的自動化變數。

```powershell-interactive
$serviceName = 'MyCloudService'
$cert = Get-AutomationCertificate -Name 'MyCertificate'
$certPwd = Get-AzureRmAutomationVariable -ResourceGroupName "ResouceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyCertPassword'
Add-AzureCertificate -ServiceName $serviceName -CertToDeploy $cert
```

### <a name="graphical-runbook-sample"></a>圖形化 Runbook 範例

透過在圖形化編輯器 [文件庫] 窗格的憑證上按一下滑鼠右鍵，然後選取 [加入至畫布]，即可加入 **Get-AutomationCertificate** 至圖形化 Runbook。

![將憑證新增至畫布](media/automation-certificates/automation-certificate-add-to-canvas.png)

下圖顯示在圖形化 Runbook 中使用憑證的範例。 這與先前用來從文字式 Runbook 將憑證新增至雲端服務的範例相同。

![範例圖形化編寫 ](media/automation-certificates/graphical-runbook-add-certificate.png)

### <a name="python2-sample"></a>Python2 範例
下列範例示範如何存取 Python2 Runbook 中的憑證。

```python
# get a reference to the Azure Automation certificate
cert = automationassets.get_automation_certificate("AzureRunAsCertificate")

# returns the binary cert content  
print cert 
```

## <a name="next-steps"></a>後續步驟

- 若要深入了解使用連結控制您的 Runbook 設計用來執行的活動之邏輯流程，請參閱[圖形化編寫中的連結](automation-graphical-authoring-intro.md#links-and-workflow)。 
