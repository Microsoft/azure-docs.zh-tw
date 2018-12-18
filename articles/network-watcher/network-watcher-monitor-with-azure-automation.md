---
title: 使用 Azure 網路監看員疑難排解來監視 VPN 閘道 | Microsoft Docs
description: 本文說明如何使用 Azure 自動化和網路監看員診斷內部部署連線
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: a102916bb0626f5b110fb134a8a25c902cfaefe7
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31598127"
---
# <a name="monitor-vpn-gateways-with-network-watcher-troubleshooting"></a>使用網路監看員疑難排解來監視 VPN 閘道

深入了解網路效能對於為客戶提供可靠的服務相當重要。 因此務必快速偵測網路中斷狀況，並採取更正動作以降低中斷條件。 Azure 自動化可讓您實作，並透過 runbook 以程式設計方式執行工作。 使用 Azure 自動化會建立一個完美的配方，讓您執行連續且主動的網路監視和警示。

## <a name="scenario"></a>案例

下圖中的案例是多層式應用程式，並具有使用 VPN 閘道與通道建立的內部部署連線。 確保 VPN 閘道已啟動且執行對於應用程式效能很重要。

Runbook 會使用資源疑難排解 API 檢查連線狀態，利用指令碼檢查 VPN 通道連線狀態來建立。 如果狀態不是狀況良好，會傳送電子郵件觸發程序給系統管理員。

![案例範例][scenario]

此案例將會：

- 建立 runbook 呼叫 `Start-AzureRmNetworkWatcherResourceTroubleshooting` cmdlet 來疑難排解連線狀態
- 將排程連結至 runbook

## <a name="before-you-begin"></a>開始之前

開始進行本案例之前，您必須具備下列條件：

- 在 Azure 中使用 Azure 自動化帳戶。 確認自動化帳戶有最新模組，並且也有 AzureRM.Network 模組。 如果您需要將 AzureRM.Network 模組新增至自動化帳戶，可從模組庫取得。
- 您必須在 Azure 自動化中設定一組認證。 在 [Azure 自動化安全性](../automation/automation-security-overview.md)深入了解
- 有效的 SMTP 伺服器 (Office 365、您的內部部署電子郵件或其他) 和 Azure 自動化中定義的認證
- 在 Azure 中已設定的虛擬網路閘道。
- 具有現有容器的儲存體帳戶，用於儲存記錄。

> [!NOTE]
> 上圖所示的基礎結構供說明用途，並不會使用本文中所包含的步驟建立。

### <a name="create-the-runbook"></a>建立 runbook

設定範例的第一個步驟是建立 runbook。 這個範例會使用「執行身分」帳戶。 若要深入了解執行身分帳戶，請造訪[使用 Azure 執行身分帳戶驗證 Runbook](../automation/automation-create-runas-account.md)

### <a name="step-1"></a>步驟 1

在 [Azure 入口網站](https://portal.azure.com)中瀏覽至 Azure 自動化中並按一下 **Runbook**

![自動化帳戶概觀][1]

### <a name="step-2"></a>步驟 2

按一下 [新增 runbook] 來啟動 runbook 的建立程序。

![runbook 刀鋒視窗][2]

### <a name="step-3"></a>步驟 3

在 [快速建立] 下，按一下 [建立新的 runbook] 來建立 runbook。

![新增 runbook 刀鋒視窗][3]

### <a name="step-4"></a>步驟 4

在此步驟中，我們為 runbook 命名，在範例中它稱為 **Get-VPNGatewayStatus**。 為 runbook 提供描述性的名稱很重要，且建議以遵循標準 PowerShell 命名標準來命名。 此範例中的 runbook 類型是 **PowerShell**，其他選項包括圖形、PowerShell 工作流程和圖形化的 PowerShell 工作流程。

![runbook 刀鋒視窗][4]

### <a name="step-5"></a>步驟 5

runbook 會在此步驟中建立，下列程式碼範例會提供範例所需的所有程式碼。 在程式碼中包含\<值\>的項目必須換成您訂用帳戶中的值。

按一下 [儲存] 使用下列程式碼

```PowerShell
# Set these variables to the proper values for your environment
$o365AutomationCredential = "<Office 365 account>"
$fromEmail = "<from email address>"
$toEmail = "<to email address>"
$smtpServer = "<smtp.office365.com>"
$smtpPort = 587
$runAsConnectionName = "<AzureRunAsConnection>"
$subscriptionId = "<subscription id>"
$region = "<Azure region>"
$vpnConnectionName = "<vpn connection name>"
$vpnConnectionResourceGroup = "<resource group name>"
$storageAccountName = "<storage account name>"
$storageAccountResourceGroup = "<resource group name>"
$storageAccountContainer = "<container name>"

# Get credentials for Office 365 account
$cred = Get-AutomationPSCredential -Name $o365AutomationCredential

# Get the connection "AzureRunAsConnection "
$servicePrincipalConnection=Get-AutomationConnection -Name $runAsConnectionName

"Logging in to Azure..."
Connect-AzureRmAccount `
    -ServicePrincipal `
    -TenantId $servicePrincipalConnection.TenantId `
    -ApplicationId $servicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint
"Setting context to a specific subscription"
Set-AzureRmContext -SubscriptionId $subscriptionId

$nw = Get-AzurermResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq $region }
$networkWatcher = Get-AzureRmNetworkWatcher -Name $nw.Name -ResourceGroupName $nw.ResourceGroupName
$connection = Get-AzureRmVirtualNetworkGatewayConnection -Name $vpnConnectionName -ResourceGroupName $vpnConnectionResourceGroup
$sa = Get-AzureRmStorageAccount -Name $storageAccountName -ResourceGroupName $storageAccountResourceGroup 
$storagePath = "$($sa.PrimaryEndpoints.Blob)$($storageAccountContainer)"
$result = Start-AzureRmNetworkWatcherResourceTroubleshooting -NetworkWatcher $networkWatcher -TargetResourceId $connection.Id -StorageId $sa.Id -StoragePath $storagePath

if($result.code -ne "Healthy")
    {
        $body = "Connection for $($connection.name) is: $($result.code) `n$($result.results[0].summary) `nView the logs at $($storagePath) to learn more."
        Write-Output $body
        $subject = "$($connection.name) Status"
        Send-MailMessage `
        -To $toEmail `
        -Subject $subject `
        -Body $body `
        -UseSsl `
        -Port $smtpPort `
        -SmtpServer $smtpServer `
        -From $fromEmail `
        -BodyAsHtml `
        -Credential $cred
    }
else
    {
    Write-Output ("Connection Status is: $($result.code)")
    }
```

### <a name="step-6"></a>步驟 6

儲存 runbook 後，排程必須連結到它以自動啟動 runbook。 若要啟動處理程序，請按一下 [排程]。

![步驟 6][6]

## <a name="link-a-schedule-to-the-runbook"></a>將排程連結至 runbook

您必須建立新的排程。 按一下 [將排程連結至您的 runbook]。

![步驟 7][7]

### <a name="step-1"></a>步驟 1

在 [排程] 刀鋒視窗中，按一下 [建立新的排程]

![步驟 8][8]

### <a name="step-2"></a>步驟 2

在 [新增排程] 刀鋒視窗中填寫排程資訊。 下列清單中，是可以設定的值︰

- **名稱** - 排程的易記名稱。
- **說明** - 排程的說明。
- **啟動** - 這個值是組成排程所觸發之時間的日期、時間和時區的組合。
- **循環** - 這個值會決定排程重複。  有效值為 [一次] 或 [重複]。
- **重複頻率** - 小時、天、週或月的排程循環間隔。
- **設定到期日** - 值會決定排程是否應過期。 可以設定為 [是] 或 [否]。 如果選擇 [是]，則會提供有效的日期和時間。

> [!NOTE]
> 如果您需要一個 runbook 的執行頻率超過一小時，必須在不同的時間間隔建立多個排程 (也就是在每小時後 15、 30、45 分鐘)

![步驟 9][9]

### <a name="step-3"></a>步驟 3

按一下 [儲存] 以將排程儲存至 runbook。

![步驟 10][10]

## <a name="next-steps"></a>後續步驟

現在您已了解如何將網路監看員疑難排解與 Azure 自動化整合，請造訪[使用 Azure 網路監看員建立觸發封包擷取的警示](network-watcher-alert-triggered-packet-capture.md)，了解如何在 VM 警示上觸發封包擷取。

<!-- images -->
[scenario]: ./media/network-watcher-monitor-with-azure-automation/scenario.png
[1]: ./media/network-watcher-monitor-with-azure-automation/figure1.png
[2]: ./media/network-watcher-monitor-with-azure-automation/figure2.png
[3]: ./media/network-watcher-monitor-with-azure-automation/figure3.png
[4]: ./media/network-watcher-monitor-with-azure-automation/figure4.png
[5]: ./media/network-watcher-monitor-with-azure-automation/figure5.png
[6]: ./media/network-watcher-monitor-with-azure-automation/figure6.png
[7]: ./media/network-watcher-monitor-with-azure-automation/figure7.png
[8]: ./media/network-watcher-monitor-with-azure-automation/figure8.png
[9]: ./media/network-watcher-monitor-with-azure-automation/figure9.png
[10]: ./media/network-watcher-monitor-with-azure-automation/figure10.png
