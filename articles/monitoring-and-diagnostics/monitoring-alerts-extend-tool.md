---
title: 將警示從 Log Analytics 延伸至 Azure
description: 本文說明可用來將警示從 Log Analytics 延伸至 Azure 警示的工具和 API。
author: msvijayn
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 06/04/2018
ms.author: vinagara
ms.component: alerts
ms.openlocfilehash: d70eecb6a5d6bafbfa6507dbe8b1bcb1cad67191
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990230"
---
# <a name="extend-alerts-from-log-analytics-into-azure-alerts"></a>將警示從 Log Analytics 延伸至 Azure 警示
Azure Log Analytics 中的警示功能已由 Azure 警示取代。 作為此轉換的一部分，您最初在 Log Analytics 中設定的警示將會延伸至 Azure。 如果不想等警示自動移動到 Azure 中，您可以起始此程序：

- 從 Operations Management Suite 入口網站手動執行。 
- 使用 AlertsVersion API 以程式設計方式執行。  

> [!NOTE]
> Microsoft 自 2018 年 5 月 14 日起，會定期自動將在 Log Analytics 公用雲端執行個體中建立的警示擴充至 Azure 警示，直到完成作業。 如果您建立[動作群組](monitoring-action-groups.md)時遇到問題，請使用[這些補救步驟](monitoring-alerts-extend-tool.md#troubleshooting)讓系統自動建立動作群組。 2018 年 7 月 5 日之前，您可以使用這些步驟。 *不適用於 Log Analytics 的 Azure Goverment and Soveriegn 雲端使用者*。 

## <a name="option-1-initiate-from-the-operations-management-suite-portal"></a>選項 1：從 Operations Management Suite 入口網站起始
下列步驟說明如何從 Operations Management Suite 入口網站延伸工作區的警示。  

1. 在 Azure 入口網站中，選取 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
2. 在 [Log Analytics 訂用帳戶] 窗格中，選取工作區，然後選取 [OMS 入口網站] 圖格。
![Log Analytics 訂用帳戶窗格螢幕擷取畫面，其中反白顯示 OMS 入口網站圖格](./media/monitor-alerts-extend/azure-portal-01.png) 
3. 將您重新導向至 Operations Management Suite 入口網站之後，請選取 [設定] 圖示。
![Operations Management Suite 入口網站的螢幕擷取畫面，其中反白顯示 [設定] 圖示](./media/monitor-alerts-extend/oms-portal-settings-option.png) 
4. 在 [設定] 頁面中，選取 [警示]。  
5. 選取 [延伸至 Azure]。
![Operations Management Suite 入口網站 [警示設定] 頁面的螢幕擷取畫面，其中反白顯示 [延伸至 Azure]](./media/monitor-alerts-extend/ExtendInto.png)
6. 三步驟精靈會出現在 [警示] 窗格中。 閱讀概觀，並選取 [下一步]。
![精靈的步驟 1 螢幕擷取畫面](./media/monitor-alerts-extend/ExtendStep1.png)  
7. 在第二個步驟中，您會看到建議變更的摘要，其中列出適用於警示的[動作群組](monitoring-action-groups.md)。 如果在多個警示中看到類似動作，精靈會提議將單一動作群組與這些動作建立關聯。  命名慣例如下：*WorkspaceName_AG_#Number*。 若要繼續，請選取 [下一步]。
![精靈的步驟 2 螢幕擷取畫面](./media/monitor-alerts-extend/ExtendStep2.png)  
8. 在精靈的最後一個步驟中，選取 [完成]，並在系統提示您時確認以起始程序。 或者，您可以提供電子郵件地址，以便在程序完成且所有警示都已成功移至 Azure 警示時收到通知。
![精靈的步驟 3 螢幕擷取畫面](./media/monitor-alerts-extend/ExtendStep3.png)

精靈完成時，在 [警示設定] 頁面上，將警示延伸至 Azure 的選項會移除。 在背景中，警示會移動到 Azure，這可能需要一些時間。 在作業期間，您無法從 Operations Management Suite 入口網站對警示進行變更。 您可以在入口網站頂端的橫幅中看到目前狀態。 如果您先前已提供電子郵件地址，則程序順利完成時，您會收到電子郵件。  


警示會持續列在 Operations Management Suite 入口網站中，即使在成功移至 Azure 之後也是。
![Operations Management Suite 入口網站警示設定頁面的螢幕擷取畫面](./media/monitor-alerts-extend/PostExtendList.png)


## <a name="option-2-use-the-alertsversion-api"></a>選項 2：使用 AlertsVersion API
您可以從可呼叫 REST API 的任何用戶端使用 Log Analytics AlertsVersion API，將警示從 Log Analytics 延伸至 Azure 警示。 您可以使用開放原始碼命令列工具 [ARMClient](https://github.com/projectkudu/ARMClient) 從 PowerShell 存取 API。 您可以輸出 JSON 的結果。  

若要使用 API，首先要建立 GET 要求。 這會評估並傳回建議變更的摘要，然後再使用 POST 要求嘗試實際延伸至 Azure。 結果會以 JSON 格式列出警示和建議的[動作群組](monitoring-action-groups.md)清單。 如果在多個警示中看到類似動作，服務會提議將這些動作與單一動作群組建立關聯。 命名慣例如下：*WorkspaceName_AG_#Number*。

```
armclient GET  /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

如果 GET 要求成功，會傳回 HTTP 狀態碼 200，以及 JSON 資料中的警示清單和建議動作群組。 以下是範例回應：

```json
{
    "version": 1,
    "migrationSummary": {
        "alertsCount": 2,
        "actionGroupsCount": 2,
        "alerts": [
            {
                "alertName": "DemoAlert_1",
                "alertId": " /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/savedSearches/<savedSearchId>/schedules/<scheduleId>/actions/<actionId>",
                "actionGroupName": "<workspaceName>_AG_1"
            },
            {
                "alertName": "DemoAlert_2",
                "alertId": " /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/savedSearches/<savedSearchId>/schedules/<scheduleId>/actions/<actionId>",
                "actionGroupName": "<workspaceName>_AG_2"
            }
        ],
        "actionGroups": [
            {
                "actionGroupName": "<workspaceName>_AG_1",
                "actionGroupResourceId": "/subscriptions/<subscriptionid>/resourceGroups/<resourceGroupName>/providers/microsoft.insights/actionGroups/<workspaceName>_AG_1",
                "actions": {
                    "emailIds": [
                        "JohnDoe@mail.com"
                    ],
                    "webhookActions": [
                        {
                            "name": "Webhook_1",
                            "serviceUri": "http://test.com"
                        }
                    ],
                    "itsmAction": {}
                }
            },
            {
                "actionGroupName": "<workspaceName>_AG_1",
                "actionGroupResourceId": "/subscriptions/<subscriptionid>/resourceGroups/<resourceGroupName>/providers/microsoft.insights/actionGroups/<workspaceName>_AG_1",
                 "actions": {
                    "emailIds": [
                        "test1@mail.com",
                          "test2@mail.com"
                    ],
                    "webhookActions": [],
                    "itsmAction": {
                        "connectionId": "<Guid>",
                        "templateInfo":"{\"PayloadRevision\":0,\"WorkItemType\":\"Incident\",\"UseTemplate\":false,\"WorkItemData\":\"{\\\"contact_type\\\":\\\"email\\\",\\\"impact\\\":\\\"3\\\",\\\"urgency\\\":\\\"2\\\",\\\"category\\\":\\\"request\\\",\\\"subcategory\\\":\\\"password\\\"}\",\"CreateOneWIPerCI\":false}"
                    }
                }
            }
        ]
    }
}

```
如果指定的工作區並未定義任何警示規則，則 JSON 資料會傳回下列結果：

```json
{
    "version": 1,
    "Message": "No Alerts found in the workspace for migration."
}
```

如果指定工作區中的所有警示規則都已延伸至 Azure，GET 要求的回應會是：

```json
{
    "version": 2
}
```

若要起始將警示移轉至 Azure，請起始 POST 回應。 POST 回應會確認您的目的並接受，以讓警示從 Log Analytics 延伸至 Azure 警示。 活動已排程，且警示已根據先前執行 GET 回應的結果進行處理。 或者，您可以提供電子郵件地址清單，在移轉警示的排程背景程序成功完成時，讓 Log Analytics 傳送報告。 您可以使用下列要求範例：

```
$emailJSON = “{‘Recipients’: [‘a@b.com’, ‘b@a.com’]}”
armclient POST  /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview $emailJSON
```

> [!NOTE]
> 將警示移轉至 Azure 警示的結果可能會根據 GET 回應所提供的摘要而有不同。 排程時，會暫時無法在 Operations Management Suite 入口網站中修改 Log Analytics 中的警示。 不過，您可以建立新的警示。 

如果 POST 要求成功，會傳回 HTTP 200 OK 狀態以及下列回應：

```json
{
    "version": 2
}
```

此回應指出警示已成功延伸至 Azure 警示。 版本屬性僅供檢查警示是否已延伸至 Azure，與 [Log Analytics 搜尋 API](../log-analytics/log-analytics-api-alerts.md) 並無任何關聯。 當警示成功延伸至 Azure，會將報告傳送到與 POST 要求一起提供的所有電子郵件地址。 如果指定工作區中的所有警示都已排程要進行延伸，則對 POST 要求的回應會是嘗試已禁止 (403 狀態碼)。 若要檢視任何錯誤訊息，或了解程序是否卡住，您可以提交 GET 要求。 如果有錯誤訊息，它會傳回，並附上摘要資訊。

```json
{
    "version": 1,
    "message": "OMS was unable to extend your alerts into Azure, Error: The subscription is not registered to use the namespace 'microsoft.insights'. OMS will schedule extending your alerts, once remediation steps illustrated in the troubleshooting guide are done.",
    "recipients": [
       "john.doe@email.com",
       "jane.doe@email.com"
     ],
    "migrationSummary": {
        "alertsCount": 2,
        "actionGroupsCount": 2,
        "alerts": [
            {
                "alertName": "DemoAlert_1",
                "alertId": " /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/savedSearches/<savedSearchId>/schedules/<scheduleId>/actions/<actionId>",
                "actionGroupName": "<workspaceName>_AG_1"
            },
            {
                "alertName": "DemoAlert_2",
                "alertId": " /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/savedSearches/<savedSearchId>/schedules/<scheduleId>/actions/<actionId>",
                "actionGroupName": "<workspaceName>_AG_2"
            }
        ],
        "actionGroups": [
            {
                "actionGroupName": "<workspaceName>_AG_1",
                "actionGroupResourceId": "/subscriptions/<subscriptionid>/resourceGroups/<resourceGroupName>/providers/microsoft.insights/actionGroups/<workspaceName>_AG_1",
                "actions": {
                    "emailIds": [
                        "JohnDoe@mail.com"
                    ],
                    "webhookActions": [
                        {
                            "name": "Webhook_1",
                            "serviceUri": "http://test.com"
                        }
                    ],
                    "itsmAction": {}
                }
            },
            {
                "actionGroupName": "<workspaceName>_AG_1",
                "actionGroupResourceId": "/subscriptions/<subscriptionid>/resourceGroups/<resourceGroupName>/providers/microsoft.insights/actionGroups/<workspaceName>_AG_1",
                 "actions": {
                    "emailIds": [
                        "test1@mail.com",
                          "test2@mail.com"
                    ],
                    "webhookActions": [],
                    "itsmAction": {
                        "connectionId": "<Guid>",
                        "templateInfo":"{\"PayloadRevision\":0,\"WorkItemType\":\"Incident\",\"UseTemplate\":false,\"WorkItemData\":\"{\\\"contact_type\\\":\\\"email\\\",\\\"impact\\\":\\\"3\\\",\\\"urgency\\\":\\\"2\\\",\\\"category\\\":\\\"request\\\",\\\"subcategory\\\":\\\"password\\\"}\",\"CreateOneWIPerCI\":false}"
                    }
                }
            }
        ]
    }
}              

```


## <a name="option-3-use-a-custom-powershell-script"></a>選項 3：使用自訂 PowerShell 指令碼
 如果 Microsoft 未成功將警示從 Operations Management Suite 入口網站延伸至 Azure，在 2018 年 7 月 5 日前您可以手動執行。 手動延伸的兩個選項已涵蓋在前兩節中。

在 2018 年 7 月 5 日之後，Operations Management Suite 入口網站的所有警示都會延伸至 Azure。 未採取[建議必要修復步驟](#troubleshooting)的使用者，其警示會執行但無法觸發動作或通知，因為缺少相關聯的[動作群組](monitoring-action-groups.md)。 

若要在 Log Analytics 中手動建立警示的[動作群組](monitoring-action-groups.md)，請使用下列範例指令碼：
```PowerShell
########## Input Parameters Begin ###########


$subscriptionId = ""
$resourceGroup = ""
$workspaceName = "" 


########## Input Parameters End ###########

armclient login

try
{
    $workspace = armclient get /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.OperationalInsights/workspaces/"$workspaceName"?api-version=2015-03-20 | ConvertFrom-Json
    $workspaceId = $workspace.properties.customerId
    $resourceLocation = $workspace.location
}
catch
{
    "Please enter valid input parameters i.e. Subscription Id, Resource Group and Workspace Name !!"
    exit
}

# Get Extend Summary of the Alerts
"`nGetting Extend Summary of Alerts for the workspace...`n"
try
{

    $value = armclient get /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.OperationalInsights/workspaces/$workspaceName/alertsversion?api-version=2017-04-26-preview

    "Extend preview summary"
    "=========================`n"

    $value

    $result = $value | ConvertFrom-Json
}
catch
{

    $ErrorMessage = $_.Exception.Message
    "Error occured while fetching/parsing Extend summary: $ErrorMessage"
    exit 
}

if ($result.version -eq 2)
{
    "`nThe alerts in this workspace have already been extended to Azure."
    exit
}

$in = Read-Host -Prompt "`nDo you want to continue extending the alerts to Azure? (Y/N)"

if ($in.ToLower() -ne "y")
{
    exit
} 


# Check for resource provider registration
try
{
    $val = armclient get subscriptions/$subscriptionId/providers/microsoft.insights/?api-version=2017-05-10 | ConvertFrom-Json
    if ($val.registrationState -eq "NotRegistered")
    {
        $val = armclient post subscriptions/$subscriptionId/providers/microsoft.insights/register/?api-version=2017-05-10
    }
}
catch
{
    "`nThe user does not have required access to register the resource provider. Please try with user having Contributor/Owner role in the subscription"
    exit
}

$actionGroupsMap = @{}
try
{
    "`nCreating new action groups for alerts extension...`n"
    foreach ($actionGroup in $result.migrationSummary.actionGroups)
    {
        $actionGroupName = $actionGroup.actionGroupName
        $actions = $actionGroup.actions
        if ($actionGroupsMap.ContainsKey($actionGroupName))
        {
            continue
        } 
        
        # Create action group payload
        $shortName = $actionGroupName.Substring($actionGroupName.LastIndexOf("AG_"))
        $properties = @{"groupShortName"= $shortName; "enabled" = $true}
        $emailReceivers = New-Object Object[] $actions.emailIds.Count
        $webhookReceivers = New-Object Object[] $actions.webhookActions.Count
        
        $count = 0
        foreach ($email in $actions.emailIds)
        {
            $emailReceivers[$count] = @{"name" = "Email$($count+1)"; "emailAddress" = "$email"}
            $count++
        }

        $count = 0
        foreach ($webhook in $actions.webhookActions)
        {
            $webhookReceivers[$count] = @{"name" = "$($webhook.name)"; "serviceUri" = "$($webhook.serviceUri)"}
            $count++
        }

        $itsmAction = $actions.itsmAction
        if ($itsmAction.connectionId -ne $null)
        {
            $val = @{
            "name" = "ITSM"
            "workspaceId" = "$subscriptionId|$workspaceId"
            "connectionId" = "$($itsmAction.connectionId)"
            "ticketConfiguration" = $itsmAction.templateInfo
            "region" = "$resourceLocation"
            }
            $properties["itsmReceivers"] = @($val)  
        }

        $properties["emailReceivers"] = @($emailReceivers)
        $properties["webhookReceivers"] = @($webhookReceivers)
        $armPayload = @{"properties" = $properties; "location" = "Global"} | ConvertTo-Json -Compress -Depth 4

    
        # Azure Resource Manager call to create action group
        $response = $armPayload | armclient put /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.insights/actionGroups/$actionGroupName/?api-version=2017-04-01

        "Created Action Group with name $actionGroupName" 
        $actionGroupsMap[$actionGroupName] = $actionGroup.actionGroupResourceId.ToLower()
        $index++
    }

    "`nSuccessfully created all action groups!!"
}
catch
{
    $ErrorMessage = $_.Exception.Message

    #Delete all action groups in case of failure
    "`nDeleting newly created action groups if any as some error happened..."
    
    foreach ($actionGroup in $actionGroupsMap.Keys)
    {
        $response = armclient delete /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.insights/actionGroups/$actionGroup/?api-version=2017-04-01      
    }

    "`nError: $ErrorMessage"
    "`nExiting..."
    exit
}

# Update all alerts configuration to the new version
"`nExtending OMS alerts to Azure...`n"

try
{
    $index = 1
    foreach ($alert in $result.migrationSummary.alerts)
    {
        $uri = $alert.alertId + "?api-version=2015-03-20"
        $config = armclient get $uri | ConvertFrom-Json
        $aznsNotification = @{
            "GroupIds" = @($actionGroupsMap[$alert.actionGroupName])
        }
        if ($alert.customWebhookPayload)
        {
            $aznsNotification.Add("CustomWebhookPayload", $alert.customWebhookPayload)
        }
        if ($alert.customEmailSubject)
        {
            $aznsNotification.Add("CustomEmailSubject", $alert.customEmailSubject)
        }      

        # Update alert version
        $config.properties.Version = 2

        $config.properties | Add-Member -MemberType NoteProperty -Name "AzNsNotification" -Value $aznsNotification
        $payload = $config | ConvertTo-Json -Depth 4
        $response = $payload | armclient put $uri
    
        "Extended alert with name $($alert.alertName)"
        $index++
    }
}
catch
{
    $ErrorMessage = $_.Exception.Message   
    if ($index -eq 1)
    {
        "`nDeleting all newly created action groups as no alerts got extended..."
        foreach ($actionGroup in $actionGroupsMap.Keys)
        {
            $response = armclient delete /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.insights/actionGroups/$actionGroup/?api-version=2017-04-01      
        }
        "`nDeleted all action groups."  
    }
    
    "`nError: $ErrorMessage"
    "`nPlease resolve the issue and try extending again!!"
    "`nExiting..."
    exit
}

"`nSuccessfully extended all OMS alerts to Azure!!" 

# Update version of workspace to indicate extension
"`nUpdating alert version information in OMS workspace..." 

$response = armclient post "/subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.OperationalInsights/workspaces/$workspaceName/alertsversion?api-version=2017-04-26-preview&iversion=2"

"`nExtension complete!!"
```


### <a name="about-the-custom-powershell-script"></a>關於自訂 PowerShell 指令碼 
以下是使用指令碼的重要資訊：
- 必要條件是安裝 [ARMClient](https://github.com/projectkudu/ARMClient)，這是可簡化叫用 Azure Resource Manager API 的開放原始碼命令列工具。
- 若要執行指令碼，您必須在 Azure 訂用帳戶中有參與者或擁有者角色。
- 您必須提供下列參數：
    - $subscriptionId：與 Operations Management Suite Log Analytics 工作區相關聯的 Azure 訂用帳戶 ID。
    - $resourceGroup：Operations Management Suite Log Analytics 工作區的 Azure 資源群組。
    - $workspaceName：Operations Management Suite Log Analytics 工作區的名稱。

### <a name="output-of-the-custom-powershell-script"></a>自訂 PowerShell 指令碼的輸出
指令碼是詳細資訊，並會在執行時輸出步驟： 
- 它會顯示摘要，其中包含工作區中現有 Operations Management Suite Log Analytics 警示的相關資訊。 摘要也包含與 Azure 動作群組相關的資訊，系統會依與其相關聯的動作而建立這些 Azure 動作群組。 
- 在檢視摘要之後，系統會提示您繼續進行延伸或結束。
- 如果您要繼續進行延伸，會建立新的 Azure 動作群組，所有現有警示會與這些群組建立關聯。 
- 指令碼會顯示訊息「延伸完成！」並結束。 如果發生任何中繼失敗，指令碼會顯示後續的錯誤。

## <a name="troubleshooting"></a>疑難排解 
在延伸警示的程序中，問題可能導致系統無法建立所需的[動作群組](monitoring-action-groups.md)。 在這種情況下，您會在 Operations Management Suite 入口網站 [警示] 區段的橫幅中看到錯誤訊息，或在對 API 執行的 GET 呼叫中看到。

> [!IMPORTANT]
> 若 Azure 公用雲端的 Log Analytics 使用者未在 2018 年 7 月 5 日之前採取下列修復步驟，警示會在 Azure 中執行，但不會觸發任何動作或通知。 若要取得警示通知，您必須手動編輯並新增[動作群組](monitoring-action-groups.md)，或使用先前的[自訂 PowerShell 指令碼](#option-3---using-custom-powershell-script)。

以下是每個錯誤的修復步驟：
- **錯誤：寫入作業的訂用帳戶/資源群組層級出現範圍鎖定**：![Operations Management Suite 入口網站 [警示設定] 頁面的螢幕擷取畫面，其中反白顯示範圍鎖定錯誤訊息](./media/monitor-alerts-extend/ErrorScopeLock.png)

    啟用範圍鎖定時，此功能會限制包含 Log Analytics (Operations Management Suite) 工作區的訂用帳戶或資源群組中的任何新變更。 系統無法將警示延伸至 Azure，並建立必要的動作群組。
    
    若要解決，請刪除含有工作區的訂用帳戶或資源群組上的 *ReadOnly* 鎖定。 您可以使用 Azure 入口網站、PowerShell、Azure CLI 或 API 來執行這項工作。 若要深入了解，請參閱[資源鎖定使用方式](../azure-resource-manager/resource-group-lock-resources.md)。 
    
    在您使用文章所述步驟來解決錯誤後，Operations Management Suite 會在隔日的排定執行內將警示延伸到 Azure。 您不需要採取任何進一步的動作或起始任何項目。

- **錯誤：訂用帳戶/資源群組層級出現原則**：![Operations Management Suite 入口網站 [警示設定] 頁面的螢幕擷取畫面，其中反白顯示原則錯誤訊息](./media/monitor-alerts-extend/ErrorPolicy.png)

    套用 [Azure 原則](../azure-policy/azure-policy-introduction.md)時，它會限制包含 Log Analytics (Operations Management Suite) 工作區的訂用帳戶或資源群組中的任何新資源。 系統無法將警示延伸至 Azure，並建立必要的動作群組。
    
    若要解決，請編輯引起 *[RequestDisallowedByPolicy](../azure-resource-manager/resource-manager-policy-requestdisallowedbypolicy-error.md)* 錯誤的原則，該錯誤會妨礙在您的訂用帳戶或資源群組上，建立包含工作區的新資源。 您可以使用 Azure 入口網站、PowerShell、Azure CLI 或 API 來執行這項工作。 您可以稽核動作以尋找造成失敗的原則。 若要深入了解，請參閱[檢視活動記錄以稽核動作](../azure-resource-manager/resource-group-audit.md)。 
    
    在您使用文章所述步驟來解決錯誤後，Operations Management Suite 會在隔日的排定執行內將警示延伸到 Azure。 您不需要採取任何進一步的動作或起始任何項目。


## <a name="next-steps"></a>後續步驟

* 深入了解新的 [Azure 警示體驗](monitoring-overview-unified-alerts.md)。
* 了解 [Azure 警示中的記錄警示](monitor-alerts-unified-log.md)。
