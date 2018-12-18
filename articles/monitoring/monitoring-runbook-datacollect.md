---
title: 在 Azure 自動化中使用 Runbook 收集 Log Analytics 資料 | Microsoft Docs
description: 逐步教學課程，可逐步解說如何在 Azure 自動化中建立 Runbook 以將資料收集到 OMS 存放庫，供 Log Analytics 進行分析。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: a831fd90-3f55-423b-8b20-ccbaaac2ca75
ms.service: monitoring
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/27/2017
ms.author: bwren
ms.openlocfilehash: d3e8e876a6c01123d65c1e8df13328bdd5fad71f
ms.sourcegitcommit: 756f866be058a8223332d91c86139eb7edea80cc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37347608"
---
# <a name="collect-data-in-log-analytics-with-an-azure-automation-runbook"></a>使用 Azure 自動化 Runbook 在 Log Analytics 中收集資料
您可以在 Log Analytics 中收集來自各種來源 (包括代理程式上的[資料來源](../log-analytics/log-analytics-data-sources.md)) 的大量資料，以及[收集自 Azure 的資料](../log-analytics/log-analytics-azure-storage.md)。  在某些情況下，您需要收集無法透過這些標準來源存取到的資料。  在這些案例中，您可以使用 [HTTP 資料收集器 API](../log-analytics/log-analytics-data-collector-api.md) 將資料從任何 REST API 用戶端寫入 Log Analytics。  執行此資料收集的常見方法是在 Azure 自動化中使用 Runbook。   

本教學課程會逐步解說在 Azure 自動化中建立和排程 Runbook 以將資料寫入 Log Analytics 的程序。


## <a name="prerequisites"></a>先決條件
此案例要求您在 Azure 訂用帳戶中設定以下資源。  兩者都可以是免費帳戶。

- [Log Analytics 工作區](../log-analytics/log-analytics-get-started.md)。
- [Azure 自動化帳戶](../automation/automation-offering-get-started.md)。

## <a name="overview-of-scenario"></a>案例概觀
在本教學課程中，您將撰寫 Runbook 來收集自動化工作的相關資訊。  Azure 自動化中的 Runbook 是使用 PowerShell 所實作，因此您會先在 Azure 自動化編輯器中撰寫和測試指令碼。  驗證已收集所需的資訊後，請將該資料寫入 Log Analytics，並驗證自訂資料類型。  最後，您將建立定期啟動 Runbook 的排程。

> [!NOTE]
> 您可以設定 Azure 自動化將作業資訊傳送至 Log Analytics，而不需要使用此 Runbook。  此案例主要用來支援教學課程，並建議您將資料傳送至測試工作區。  


## <a name="1-install-data-collector-api-module"></a>1.安裝資料收集器 API 模組
[來自 HTTP 資料收集器 API 的每個要求](../log-analytics/log-analytics-data-collector-api.md#create-a-request)都必須適當地格式化，並包含授權標頭。  您可以在 Runbook 中執行這項作業，但可以減少使用可簡化此程序之模型所需的程式碼數量。  您可以使用的其中一個模組是 PowerShell 資源庫中的 [OMSIngestionAPI 模組](https://www.powershellgallery.com/packages/OMSIngestionAPI)。

若要在 Runbook 中使用[模組](../automation/automation-integration-modules.md)，則必須將它安裝在自動化帳戶中。  相同帳戶中的任何 Runbook 接著可以在模組中使用函式。  在自動化帳戶中選取 [資產] > [模組] > [新增模組]，即可安裝新的模組。  

PowerShell 資源庫提供快速選項讓您將模組直接部署至自動化帳戶，因此，您可以在本教學課程中使用該選項。  

![OMSIngestionAPI 模組](media/monitoring-runbook-datacollect/OMSIngestionAPI.png)

1. 前往 [PowerShell 資源庫](https://www.powershellgallery.com/)。
2. 搜尋 **OMSIngestionAPI**。
3. 按一下 [部署至 Azure 自動化] 按鈕。
4. 選取自動化帳戶，然後按一下 [確定] 安裝模組。


## <a name="2-create-automation-variables"></a>2.建立自動化變數
[自動化變數](..\automation\automation-variables.md)會保留值，以供自動化帳戶中的所有 Runbook 使用。  它們可讓您變更這些值，而不需要編輯實際 Runbook，讓 Runbook 更具彈性。 來自 HTTP 資料收集器 API 的每個要求都需要 OMS 工作區的識別碼和索引鍵，而且變數資產最適合用來儲存這項資訊。  

![變數](media/monitoring-runbook-datacollect/variables.png)

1. 在 Azure 入口網站中，瀏覽至您的自動化帳戶。
2. 選取 [共用資源] 下方的 [變數]。
2. 按一下 [新增變數]，然後建立下表中的兩個變數。

| 屬性 | 工作區識別碼值 | 工作區索引鍵值 |
|:--|:--|:--|
| Name | WorkspaceId | WorkspaceKey |
| 類型 | 字串 | 字串 |
| 值 | 貼入 Log Analytics 工作區的工作區識別碼。 | 貼入 Log Analytics 工作區的主要或次要索引鍵。 |
| 已加密 | 否 | yes |



## <a name="3-create-runbook"></a>3.建立 Runbook

Azure 自動化在入口網站中有一個編輯器，您可以在其中編輯和測試 Runbook。  您可以選擇使用指令碼編輯器來[直接使用 PowerShell](../automation/automation-edit-textual-runbook.md)，或[建立圖形化 Runbook](../automation/automation-graphical-authoring-intro.md)。  在本教學課程中，您將使用 PowerShell 指令碼。 

![編輯 Runbook](media/monitoring-runbook-datacollect/edit-runbook.png)

1. 瀏覽至您的自動化帳戶。  
2. 按一下 [Runbook] > [新增 Runbook] > [建立新的 Runbook]。
3. 針對 Runbook 名稱，鍵入 **Collect-Automation-jobs**。  針對 Runbook 類型，選取 [PowerShell]。
4. 按一下 [建立]  來建立 Runbook，並啟動編輯器。
5. 複製下列程式碼，並將其貼入 Runbook 中。  如需程式碼的說明，請參閱指令碼中的註解。
    
        # Get information required for the automation account from parameter values when the runbook is started.
        Param
        (
            [Parameter(Mandatory = $True)]
            [string]$resourceGroupName,
            [Parameter(Mandatory = $True)]
            [string]$automationAccountName
        )
        
        # Authenticate to the Automation account using the Azure connection created when the Automation account was created.
        # Code copied from the runbook AzureAutomationTutorial.
        $connectionName = "AzureRunAsConnection"
        $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName         
        Connect-AzureRmAccount `
            -ServicePrincipal `
            -TenantId $servicePrincipalConnection.TenantId `
            -ApplicationId $servicePrincipalConnection.ApplicationId `
            -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
        
        # Set the $VerbosePreference variable so that we get verbose output in test environment.
        $VerbosePreference = "Continue"
        
        # Get information required for Log Analytics workspace from Automation variables.
        $customerId = Get-AutomationVariable -Name 'WorkspaceID'
        $sharedKey = Get-AutomationVariable -Name 'WorkspaceKey'
        
        # Set the name of the record type.
        $logType = "AutomationJob"
        
        # Get the jobs from the past hour.
        $jobs = Get-AzureRmAutomationJob -ResourceGroupName $resourceGroupName -AutomationAccountName $automationAccountName -StartTime (Get-Date).AddHours(-1)
        
        if ($jobs -ne $null) {
            # Convert the job data to json
            $body = $jobs | ConvertTo-Json
        
            # Write the body to verbose output so we can inspect it if verbose logging is on for the runbook.
            Write-Verbose $body
        
            # Send the data to Log Analytics.
            Send-OMSAPIIngestionFile -customerId $customerId -sharedKey $sharedKey -body $body -logType $logType -TimeStampField CreationTime
        }


## <a name="4-test-runbook"></a>4.測試 Runbook
Azure 自動化包含可在發佈之前[測試 Runbook](../automation/automation-testing-runbook.md) 的環境。  您可以先檢查 Runbook 所收集的資料，並驗證它如預期寫入 Log Analytics 中，再將它發佈至生產環境。 
 
![測試 Runbook](media/monitoring-runbook-datacollect/test-runbook.png)

6. 按一下 [儲存] 以儲存 Runbook。
1. 按一下 [測試] 窗格，以在測試環境中開啟 Runbook。
3. 因為您的 Runbook 有參數，所以系統會提示您輸入值。  輸入您要從中收集作業資訊的資源群組和自動化帳戶名稱。
4. 按一下 [啟動] 以啟動 Runbook。
3. Runbook 會先以 [已排入佇列] 狀態啟動，再進入 [執行中]。  
3. Runbook 應該會以 json 格式顯示詳細資訊輸出以及收集到的作業。  如果未列出任何作業，則最後一個小時可能未在自動化帳戶中建立任何作業。  請嘗試在自動化帳戶中啟動任何 Runbook，然後重新執行測試。
4. 請確定輸出中未顯示 Log Analytics 的 post 命令中的任何錯誤。  您應該會看見類似下方的訊息。

    ![張貼輸出](media/monitoring-runbook-datacollect/post-output.png)

## <a name="5-verify-records-in-log-analytics"></a>5.確認 Log Analytics 中的記錄
在 Runbook 完成測試並確認成功收到輸出之後，即可確認已使用 [Log Analytics 中的記錄搜尋](../log-analytics/log-analytics-log-searches.md)來建立記錄。

![記錄輸出](media/monitoring-runbook-datacollect/log-output.png)

1. 在 Azure 入口網站中，選取 Log Analytics 工作區。
2. 按一下 [記錄搜尋]。
3. 鍵入下列命令 `Type=AutomationJob_CL`，然後按一下搜尋按鈕。 請注意，記錄類型會包括指令碼中未指定的 _CL。  該尾碼會自動附加至記錄類型，指出它是自訂記錄類型。
4. 您應該會看到傳回一或多筆記錄，指出 Runbook 已如預期運作。


## <a name="6-publish-the-runbook"></a>6.發佈 Runbook
驗證 Runbook 正確運作之後，必須將其發佈，以在生產環境中執行。  您可以繼續編輯和測試 Runbook，而不需要修改已發佈的版本。  

![發佈 Runbook](media/monitoring-runbook-datacollect/publish-runbook.png)

1. 返回自動化帳戶。
2. 按一下 [Runbook]，並選取 [收集自動化工作]。
3. 按一下 [編輯]，然後按一下 [發佈]。
4. 系統要求您驗證是否要覆寫先前發佈的版本時，請按一下 [是]。

## <a name="7-set-logging-options"></a>7.設定記錄選項 
在測試環境中，因為您在指令碼中設定 $VerbosePreference 變數，所以可以檢視[詳細資訊輸出](../automation/automation-runbook-output-and-messages.md#message-streams)。  在生產環境中，如果您想要檢視詳細資訊輸出，則需要設定 Runbook 的記錄內容。  針對本教學課程中所使用的 Runbook，這會顯示傳送至 Log Analytics 的 json 資料。

![記錄和追蹤](media/monitoring-runbook-datacollect/logging.png)

1. 在 Runbook 的內容中，選取 [Runbook 設定] 下的 [記錄和追蹤]。
2. 將 [記錄詳細記錄] 的設定變更為 [開啟]。
3. 按一下 [檔案] 。

## <a name="8-schedule-runbook"></a>8.排程 Runbook
啟動 Runbook 以收集監視資料的最常見方式，是將其排程為自動執行。  作法是[在 Azure 自動化中建立排程](../automation/automation-schedules.md)，並將它連結至 Runbook。

![排程 Runbook](media/monitoring-runbook-datacollect/schedule-runbook.png)

1. 在 Runbook 的內容中，選取 [資源] 下的 [排程]。
2. 按一下 [新增排程] > [將排程連結至您的 Runbook] > [建立新的排程]。
5. 鍵入排程的下列值，然後按一下 [建立]。

| 屬性 | 值 |
|:--|:--|
| Name | AutomationJobs-Hourly |
| 啟動 | 選取至少超過目前時間 5 分鐘的任何時間。 |
| 週期性 | 週期性 |
| 重複頻率 | 1 小時 |
| 設定到期日 | 否 |

建立排程之後，您需要設定將在每次此排程啟動 Runbook 時使用的參數值。

6. 按一下 [設定參數與回合設定]。
7. 填入 **ResourceGroupName** 和 **AutomationAccountName** 的值。
8. 按一下 [確定]。 

## <a name="9-verify-runbook-starts-on-schedule"></a>9.驗證 Runbook 會依排程啟動
每次啟動 Runbook 時，都會[建立作業](../automation/automation-runbook-execution.md)且會記錄任何輸出。  事實上，這些作業與 Runbook 所收集的作業相同。  在排程的開始時間過後，您可以檢查 Runbook 的作業來確認 Runbook 如預期啟動。

![工作](media/monitoring-runbook-datacollect/jobs.png)

1. 在 Runbook 的內容中，選取 [資源] 下的 [作業]。
2. 每次啟動 Runbook 時，您應該都會看到作業清單。
3. 按一下其中一個作業，即可檢視其詳細資料。
4. 按一下 [所有記錄]，以檢視來自 Runbook 的記錄和輸出。
5. 捲動至底部，找到與下圖類似的項目。<br>![詳細資訊](media/monitoring-runbook-datacollect/verbose.png)
6. 按一下此項目，檢視已傳送至 Log Analytics 的詳細 json 資料。



## <a name="next-steps"></a>後續步驟
- 使用[檢視表設計工具](../log-analytics/log-analytics-view-designer.md)來建立檢視，以顯示您收集到 Log Analytics 存放庫的資料。
- 將 Runbook 封裝到[管理解決方案](monitoring-solutions-creating.md)，以散發給客戶。
- 深入了解 [Log Analytics](https://docs.microsoft.com/azure/log-analytics/)。
- 深入了解 [Azure 自動化](https://docs.microsoft.com/azure/automation/)。
- 深入了解 [HTTP 資料收集器 API](../log-analytics/log-analytics-data-collector-api.md)。
