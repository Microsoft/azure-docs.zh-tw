---
title: 使用警示來觸發 Azure 自動化 Runbook
description: 了解如何在引發 Azure 警示時觸發 Runbook 執行。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 09/18/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: ac117994140f96ec993e4fed739626f736ad7efc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965274"
---
# <a name="use-an-alert-to-trigger-an-azure-automation-runbook"></a>使用警示來觸發 Azure 自動化 Runbook

您可以使用 [Azure 監視器](../azure-monitor/overview.md?toc=%2fazure%2fautomation%2ftoc.json)來監視 Azure 中大多數服務的基準層級計量和記錄。 您可以使用[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md?toc=%2fazure%2fautomation%2ftoc.json)或使用傳統警示來呼叫 Azure 自動化 Runbook，以根據警示來自動執行工作。 本文示範如何使用警示來設定及執行 Runbook。

## <a name="alert-types"></a>警示類型

您可以將自動化 Runbook 與三種警示類型搭配使用：
* 傳統計量警示
* 活動記錄警示
* 近乎即時的計量警示

當警示呼叫 Runbook 時，實際的呼叫是對 Webhook 的 HTTP POST 要求。 POST 要求的本文包含 JSON 格式的物件，其中具有與警示相關的實用屬性。 下表列出每種警示類型的承載結構描述連結：

|警示  |說明|承載結構描述  |
|---------|---------|---------|
|[傳統計量警示](../monitoring-and-diagnostics/insights-alerts-portal.md?toc=%2fazure%2fautomation%2ftoc.json)    |當任何平台層級計量符合特定條件時，便傳送通知。 例如，當 VM 上 [CPU %] 的值在過去 5 分鐘大於 **90**時。| [類別計量警示承載結構描述](../monitoring-and-diagnostics/insights-webhooks-alerts.md?toc=%2fazure%2fautomation%2ftoc.json#payload-schema)         |
|[活動記錄警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts.md?toc=%2fazure%2fautomation%2ftoc.json)    |當 Azure 活動記錄中的任何新事件符合特定條件時，便傳送通知。 例如，當 **myProductionResourceGroup** 中發生 `Delete VM` 作業時，或出現狀態為 [作用中] 的新「Azure 服務健康狀態」事件時。| [活動記錄警示承載結構描述](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md)        |
|[近乎即時計量警示](../monitoring-and-diagnostics/monitoring-near-real-time-metric-alerts.md?toc=%2fazure%2fautomation%2ftoc.json)    |當一或多個平台層級的計量符合指定的條件時，便以比計量警示快的速度傳送通知。 例如，當在過去 5 分鐘 VM 上 [CPU %] 的值大於 **90**，且 [網路輸入] 的值大於 **500 MB** 時。| [近乎即時計量警示承載結構描述](../monitoring-and-diagnostics/insights-webhooks-alerts.md?toc=%2fazure%2fautomation%2ftoc.json#payload-schema)          |

由於每種類型的警示所提供的資料不同，因此每種警示類型的處理方式也不同。 在下一節中，您將了解如何建立 Runbook 來處理不同類型的警示。

## <a name="create-a-runbook-to-handle-alerts"></a>建立 Runbook 來處理警示

若要搭配警示使用「自動化」，您需要具有邏輯的 Runbook，此邏輯會管理傳遞給 Runbook 的警示 JSON 承載。 下列範例 Runbook 必須從 Azure 警示呼叫。

如上一節所述，每種類型的警示都有不同的結構描述。 指令碼會從警示採用 `WebhookData` Runbook 輸入參數中的 Webhook 資料。 接著，指令碼會評估 JSON 承載，以判斷使用的是哪種警示類型。

此範例使用來自 VM 的警示。 它會從承載擷取 VM 資料，然後使用該資訊來停止 VM。 您必須在執行 Runbook 的「自動化」帳戶中設定連線。 使用警示來觸發 Runbook 時，請務必檢查所觸發 Runbook 中的警示狀態。 每一次警示變更狀態時，Runbook 都會觸發。 警示有多個狀態，最常見的兩個狀態為 `Activated` 和 `Resolved`。 在 Runbook 邏輯中檢查這個狀態，以確保 Runbook 不會執行多次。 本文中的範例只會示範如何尋找 `Activated` 警示。

Runbook 會使用**AzureRunAsConnection** [執行身分帳戶](automation-create-runas-account.md)向 Azure 進行驗證，以針對 VM 執行管理動作。

請使用此範例來建立名為 **Stop-AzureVmInResponsetoVMAlert** 的 Runbook。 您可以修改 PowerShell 指令碼，然後將它與許多不同的資源搭配使用。

1. 移至您的「Azure 自動化」帳戶。
1. 在 [程序自動化] 底下，選取 [Runbook]。
1. 在 Runbook 清單頂端，選取 [新增 Runbook]。 
1. 在 [新增 Runbook] 頁面上，選取 [快速建立]。
1. 針對 Runbook 名稱，輸入 **Stop-AzureVmInResponsetoVMAlert**。 針對 Runbook 類型，選取 [PowerShell]。 然後，選取 [Create] \(建立\)。  
1. 將下列 PowerShell 範例複製到 [編輯] 窗格中。 

    ```powershell-interactive
    <#
    .SYNOPSIS
    This runbook stops a resource management VM in response to an Azure alert trigger.

    .DESCRIPTION
    This runbook stops a resource management VM in response to an Azure alert trigger.
    The input is alert data that has the information required to identify which VM to stop.
    
    DEPENDENCIES
    - The runbook must be called from an Azure alert via a webhook.
    
    REQUIRED AUTOMATION ASSETS
    - An Automation connection asset called "AzureRunAsConnection" that is of type AzureRunAsConnection.
    - An Automation certificate asset called "AzureRunAsCertificate".

    .PARAMETER WebhookData
    Optional. (The user doesn't need to enter anything, but the service always passes an object.)
    This is the data that's sent in the webhook that's triggered from the alert.

    .NOTES
    AUTHOR: Azure Automation Team
    LASTEDIT: 2017-11-22
    #>

    [OutputType("PSAzureOperationResponse")]

    param
    (
        [Parameter (Mandatory=$false)]
        [object] $WebhookData
    )

    $ErrorActionPreference = "stop"

    if ($WebhookData)
    {
        # Get the data object from WebhookData.
        $WebhookBody = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)

        # Get the info needed to identify the VM (depends on the payload schema).
        $schemaId = $WebhookBody.schemaId
        Write-Verbose "schemaId: $schemaId" -Verbose
        if ($schemaId -eq "AzureMonitorMetricAlert") {
            # This is the near-real-time Metric Alert schema
            $AlertContext = [object] ($WebhookBody.data).context
            $ResourceName = $AlertContext.resourceName
            $status = ($WebhookBody.data).status
        }
        elseif ($schemaId -eq "Microsoft.Insights/activityLogs") {
            # This is the Activity Log Alert schema
            $AlertContext = [object] (($WebhookBody.data).context).activityLog
            $ResourceName = (($AlertContext.resourceId).Split("/"))[-1]
            $status = ($WebhookBody.data).status
        }
        elseif ($schemaId -eq $null) {
            # This is the original Metric Alert schema
            $AlertContext = [object] $WebhookBody.context
            $ResourceName = $AlertContext.resourceName
            $status = $WebhookBody.status
        }
        else {
            # The schema isn't supported.
            Write-Error "The alert data schema - $schemaId - is not supported."
        }

        Write-Verbose "status: $status" -Verbose
        if ($status -eq "Activated")
        {
            $ResourceType = $AlertContext.resourceType
            $ResourceGroupName = $AlertContext.resourceGroupName
            $SubId = $AlertContext.subscriptionId
            Write-Verbose "resourceType: $ResourceType" -Verbose
            Write-Verbose "resourceName: $ResourceName" -Verbose
            Write-Verbose "resourceGroupName: $ResourceGroupName" -Verbose
            Write-Verbose "subscriptionId: $SubId" -Verbose

            # Use this only if this is a resource management VM.
            if ($ResourceType -eq "Microsoft.Compute/virtualMachines")
            {
                # This is the VM.
                Write-Verbose "This is a resource management VM." -Verbose

                # Authenticate to Azure by using the service principal and certificate. Then, set the subscription.
                Write-Verbose "Authenticating to Azure with service principal and certificate" -Verbose
                $ConnectionAssetName = "AzureRunAsConnection"
                Write-Verbose "Get connection asset: $ConnectionAssetName" -Verbose
                $Conn = Get-AutomationConnection -Name $ConnectionAssetName
                if ($Conn -eq $null)
                {
                    throw "Could not retrieve connection asset: $ConnectionAssetName. Check that this asset exists in the Automation account."
                }
                Write-Verbose "Authenticating to Azure with service principal." -Verbose
                Connect-AzureRmAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint | Write-Verbose
                Write-Verbose "Setting subscription to work against: $SubId" -Verbose
                Set-AzureRmContext -SubscriptionId $SubId -ErrorAction Stop | Write-Verbose

                # Stop the VM.
                Write-Verbose "Stopping the VM - $ResourceName - in resource group - $ResourceGroupName -" -Verbose
                Stop-AzureRmVM -Name $ResourceName -ResourceGroupName $ResourceGroupName -Force
                # [OutputType(PSAzureOperationResponse")]
            }
            else {
                # ResourceType isn't supported.
                Write-Error "$ResourceType is not a supported resource type for this runbook."
            }
        }
        else {
            # The alert status was not 'Activated', so no action taken.
            Write-Verbose ("No action taken. Alert status: " + $status) -Verbose
        }
    }
    else {
        # Error
        Write-Error "This runbook is meant to be started from an Azure alert webhook only."
    }
    ```
1. 選取 [發佈] 來儲存並發佈 Runbook。

## <a name="create-an-action-group"></a>建立動作群組

動作群組是警示所觸發的動作集合。 Runbook 只是可與動作群組搭配使用的眾多動作之一。

1. 在 Azure 入口網站中，選取 [監視] > [設定] > [動作群組]。
1. 選取 [新增動作群組]，然後輸入必要的資訊：  
    1. 在 [動作群組名稱] 方塊中，輸入名稱。
    1. 在 [簡短名稱] 方塊中，輸入名稱。 使用此動作群組來傳送通知時，會使用簡短名稱來取代完整的動作群組名稱。
    1. [訂用帳戶] 方塊中會自動填入您目前的訂用帳戶。 這是要在其中儲存動作群組的訂用帳戶。
    1. 選取要在其中儲存動作群組的資源群組。

針對此範例，您會建立兩個動作：Runbook 動作和通知動作。

### <a name="runbook-action"></a>Runbook 動作

在動作群組中建立 Runbook 動作：

1. 在 [動作] 底下，針對 [動作名稱] 輸入動作的名稱。 針對 [動作類型]，選取 [自動化 Runbook]。
1. 在 [詳細資料] 底下，選取 [編輯詳細資料]。  
1. 在 [設定 Runbook] 頁面上的 [Runbook 來源] 底下，選取 [使用者]。  
1. 選取您的 [訂用帳戶] 和 [自動化帳戶]，然後選取 [Stop-AzureVmInResponsetoVMAlert] Runbook。  
1. 完成時，選取 [確定]。

### <a name="notification-action"></a>通知動作

在動作群組中建立通知動作：

1. 在 [動作] 底下，針對 [動作名稱] 輸入動作的名稱。 針對 [動作類型]，選取 [電子郵件]。  
1. 在 [詳細資料] 底下，選取 [編輯詳細資料]。  
1. 在 [電子郵件] 頁面上，輸入用於通知的電子郵件地址，然後選取 [確定]。 除了 Runbook 之外，再新增電子郵件地址作為動作，會相當有幫助。 當 Runbook 啟動時，您會收到通知。  

    您的動作群組看起來應該如下圖所示：

   ![新增動作群組分頁](./media/automation-create-alert-triggered-runbook/add-action-group.png)
1. 若要建立動作群組，請選取 [確定]。

您可以在所建立的[活動記錄警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts.md?toc=%2fazure%2fautomation%2ftoc.json)和[近乎即時警示](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md?toc=%2fazure%2fautomation%2ftoc.json#create-an-alert-rule-with-the-azure-portal)中使用此動作群組。

## <a name="classic-alert"></a>傳統警示

傳統警示會根據計量，而不使用動作群組。 不過，您可以根據傳統警示來設定 Runbook 動作。 

建立傳統警示：

1. 選取 [新增計量警示]。
1. 將您的計量警示命名為 **myVMCPUAlert**。 輸入簡短的警示描述。
1. 針對計量警示條件，選取 [大於]。 針對 [閾值] 值，選取 [10]。 針對 [期間] 值，選取 [過去 5 分鐘]。
1. 在 [採取動作] 底下，選取 [從此警示執行 Runbook]。
1. 在 [設定 Runbook] 頁面上，針對 [Runbook 來源]，選取 [使用者]。 選擇您的自動化帳戶，然後選取 [Stop-AzureVmInResponsetoVMAlert] Runbook。 選取 [確定] 。
1. 若要儲存警示規則，請選取 [確定]。

## <a name="next-steps"></a>後續步驟

* 如需有關使用 Webhook 來啟動「自動化」Runbook 的詳細資訊，請參閱[從 Webhook 啟動 Runbook](automation-webhooks.md)。
* 如需以不同方式啟動 Runbook 的詳細資訊，請參閱 [啟動 Runbook](automation-starting-a-runbook.md)。
* 若要了解如何建立活動記錄警示，請參閱[建立活動記錄警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts.md?toc=%2fazure%2fautomation%2ftoc.json)。
* 若要了解如何建立近乎即時計量警示，請參閱[在 Azure 入口網站中建立警示規則](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md?toc=%2fazure%2fautomation%2ftoc.json#create-an-alert-rule-with-the-azure-portal)。
