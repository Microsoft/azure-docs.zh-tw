---
title: 管理解決方案中的 Azure 自動化資源 | Microsoft Docs
description: 管理解決方案通常會在 Azure 自動化中包含 Runbook，以便自動執行一些程序，例如收集及處理監視資料。  本文說明如何在解決方案中包含 Runbook 與其相關資源。
services: monitoring
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: 5281462e-f480-4e5e-9c19-022f36dce76d
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/24/2017
ms.author: bwren
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 95d5b2499f9e260e6ed134c4191b053325ca3f42
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37868816"
---
# <a name="adding-azure-automation-resources-to-a-management-solution-preview"></a>將 Azure 自動化資源新增至管理解決方案 (預覽)
> [!NOTE]
> 這是建立管理解決方案 (目前處於預覽狀態) 的預備文件。 以下所述的任何結構描述可能會有所變更。   


[管理解決方案]( monitoring-solutions.md)通常會在 Azure 自動化中包含 Runbook，以便自動執行一些程序，例如收集及處理監視資料。  除了 Runbook，自動化帳戶還包含一些資產，例如支援解決方案中所用 Runbook 的變數和排程。  本文說明如何在解決方案中包含 Runbook 與其相關資源。

> [!NOTE]
> 本文中的範例使用管理解決方案所需或通用的參數和變數，如[在 Azure 中設計和建置管理解決方案]( monitoring-solutions-creating.md)所述。 


## <a name="prerequisites"></a>先決條件
本文假設您已熟悉下列資訊。

- 如何[建立管理解決方案]( monitoring-solutions-creating.md)。
- [解決方案檔]( monitoring-solutions-solution-file.md)的結構。
- 如何[製作 Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)

## <a name="automation-account"></a>自動化帳戶
Azure 自動化中的所有資源都會包含在[自動化帳戶](../automation/automation-security-overview.md#automation-account-overview)中。  如 [Log Analytics 工作區和自動化帳戶]( monitoring-solutions.md#log-analytics-workspace-and-automation-account)所述，自動化帳戶不會包含於管理解決方案中，但在安裝解決方案前就必須存在。  如果無法使用，則解決方案會安裝失敗。

每個自動化資源的名稱皆包含其自動化帳戶的名稱。  這可在具有 **accountName** 參數的解決方案中完成，如下列 Runbook 資源範例所示。

    "name": "[concat(parameters('accountName'), '/MyRunbook'))]"


## <a name="runbooks"></a>Runbook
您應該在解決方案檔中包含解決方案所使用的任何 Runbook，以便系統會在安裝解決方案時建立這些 Runbook。  但您不能在範本中包含 Runbook 的主體，因此，您應該將 Runbook 發佈到公用位置，以供安裝了解決方案的使用者存取。

[Azure 自動化 Runbook](../automation/automation-runbook-types.md) 資源的類型為 **Microsoft.Automation/automationAccounts/runbooks**，且具有下列結構。 這包括一般變數和參數，因此您可以將此程式碼片段複製並貼到您的解決方案檔，然後變更參數名稱。 

    {
        "name": "[concat(parameters('accountName'), '/', variables('Runbook').Name)]",
        "type": "Microsoft.Automation/automationAccounts/runbooks",
        "apiVersion": "[variables('AutomationApiVersion')]",
        "dependsOn": [
        ],
        "location": "[parameters('regionId')]",
        "tags": { },
        "properties": {
            "runbookType": "[variables('Runbook').Type]",
            "logProgress": "true",
            "logVerbose": "true",
            "description": "[variables('Runbook').Description]",
            "publishContentLink": {
                "uri": "[variables('Runbook').Uri]",
                "version": [variables('Runbook').Version]"
            }
        }
    }


下表會說明 Runbook 的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| runbookType |指定 Runbook 的類型。 <br><br> Script - PowerShell 指令碼 <br>PowerShell - PowerShell 工作流程 <br> GraphPowerShell - 圖形化 PowerShell 指令碼 Runbook <br> GraphPowerShellWorkflow - 圖形化 PowerShell 工作流程 Runbook |
| logProgress |指定是否應針對 Runbook 產生[進度記錄](../automation/automation-runbook-output-and-messages.md)。 |
| logVerbose |指定是否應針對 Runbook 產生[詳細資訊記錄](../automation/automation-runbook-output-and-messages.md)。 |
| 說明 |Runbook 的選擇性說明。 |
| publishContentLink |指定 Runbook 的內容。 <br><br>uri - 指定 Runbook 的內容 Uri。  這會是 PowerShell 和 Script Runbook 的 .ps1 檔案，以及 Graph Runbook 的已匯出圖形化 Runbook 檔案。  <br> 版本 - 您自己追蹤的 Runbook 版本。 |


## <a name="automation-jobs"></a>自動化作業
當您在 Azure 自動化中啟動 Runbook 時，此 Runbook 便會建立自動化作業。  您可以在解決方案中新增自動化作業資源，以在安裝管理解決方案時自動啟動 Runbook。  這個方法通常會用來啟動可對解決方案進行初始設定的 Runbook。  若要定期啟動 Runbook，請建立[排程](#schedules)和[作業排程](#job-schedules)

作業資源的類型為 **Microsoft.Automation/automationAccounts/jobs**，且具有下列結構。  這包括一般變數和參數，因此您可以將此程式碼片段複製並貼到您的解決方案檔，然後變更參數名稱。 

    {
      "name": "[concat(parameters('accountName'), '/', parameters('Runbook').JobGuid)]",
      "type": "Microsoft.Automation/automationAccounts/jobs",
      "apiVersion": "[variables('AutomationApiVersion')]",
      "location": "[parameters('regionId')]",
      "dependsOn": [
        "[concat('Microsoft.Automation/automationAccounts/', parameters('accountName'), '/runbooks/', variables('Runbook').Name)]"
      ],
      "tags": { },
      "properties": {
        "runbook": {
          "name": "[variables('Runbook').Name]"
        },
        "parameters": {
          "Parameter1": "[[variables('Runbook').Parameter1]",
          "Parameter2": "[[variables('Runbook').Parameter2]"
        }
      }
    }

下表會說明自動化作業的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| Runbook |具有要啟動之 Runbook 名稱的單一名稱實體。 |
| parameters |Runbook 所需之每個參數值的實體。 |

作業包含 Runbook 名稱和任何要傳送至 Runbook 的參數值。  作業應該[相依於]( monitoring-solutions-solution-file.md#resources)正在啟動的 Runbook，因為 Runbook 必須建立於作業之前。  如果您有多個應該啟動的 Runbook，您可以藉由讓作業相依於其他任何應該先執行的作業，以定義這些 Runbook 的順序。

作業資源的名稱必須包含通常由參數所指派的 GUID。  [在 Azure 中建立管理解決方案]( monitoring-solutions-solution-file.md#parameters)可讓您深入了解 GUID 參數。  


## <a name="certificates"></a>憑證
[Azure 自動化憑證](../automation/automation-certificates.md)的類型為 **Microsoft.Automation/automationAccounts/certificates**，且具有下列結構。 這包括一般變數和參數，因此您可以將此程式碼片段複製並貼到您的解決方案檔，然後變更參數名稱。 

    {
      "name": "[concat(parameters('accountName'), '/', variables('Certificate').Name)]",
      "type": "Microsoft.Automation/automationAccounts/certificates",
      "apiVersion": "[variables('AutomationApiVersion')]",
      "location": "[parameters('regionId')]",
      "tags": { },
      "dependsOn": [
      ],
      "properties": {
        "base64Value": "[variables('Certificate').Base64Value]",
        "thumbprint": "[variables('Certificate').Thumbprint]"
      }
    }



下表會說明憑證資源的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| base64Value |憑證的 Base 64 值。 |
| thumbprint |憑證的指紋。 |



## <a name="credentials"></a>認證
[Azure 自動化認證](../automation/automation-credentials.md)的類型為 **Microsoft.Automation/automationAccounts/credentials**，且具有下列結構。  這包括一般變數和參數，因此您可以將此程式碼片段複製並貼到您的解決方案檔，然後變更參數名稱。 


    {
      "name": "[concat(parameters('accountName'), '/', variables('Credential').Name)]",
      "type": "Microsoft.Automation/automationAccounts/credentials",
      "apiVersion": "[variables('AutomationApiVersion')]",
      "location": "[parameters('regionId')]",
      "tags": { },
      "dependsOn": [
      ],
      "properties": {
        "userName": "[parameters('credentialUsername')]",
        "password": "[parameters('credentialPassword')]"
      }
    }

下表會說明認證資源的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| userName |認證的使用者名稱。 |
| password |認證的密碼。 |


## <a name="schedules"></a>排程
[Azure 自動化排程](../automation/automation-schedules.md)的類型為 **Microsoft.Automation/automationAccounts/schedules**，且具有下列結構。 這包括一般變數和參數，因此您可以將此程式碼片段複製並貼到您的解決方案檔，然後變更參數名稱。 

    {
      "name": "[concat(parameters('accountName'), '/', variables('Schedule').Name)]",
      "type": "microsoft.automation/automationAccounts/schedules",
      "apiVersion": "[variables('AutomationApiVersion')]",
      "tags": { },
      "dependsOn": [
      ],
      "properties": {
        "description": "[variables('Schedule').Description]",
        "startTime": "[parameters('scheduleStartTime')]",
        "timeZone": "[parameters('scheduleTimeZone')]",
        "isEnabled": "[variables('Schedule').IsEnabled]",
        "interval": "[variables('Schedule').Interval]",
        "frequency": "[variables('Schedule').Frequency]"
      }
    }

下表會說明排程資源的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| 說明 |排程的選擇性說明。 |
| startTime |將排程的開始時間指定為 DateTime 物件。 如果可以轉換成有效的 DateTime，即可提供字串。 |
| isEnabled |指定是否啟用排程。 |
| interval |排程的間隔類型。<br><br>day<br>hour |
| frequency |應觸發排程的頻率 (以天數或時數為單位)。 |

排程的開始時間值必須大於目前的時間。  您無法知道解決方案會在何時安裝，因此不能以變數來提供此值。

在解決方案中使用排程資源時，請使用下列兩種策略的其中一種。

- 使用參數來提供排程的開始時間。  這會提示使用者在他們安裝解決方案時提供值。  如果您有多個排程，您可以對它們使用單一參數值。
- 使用會在安裝解決方案時啟動的 Runbook 來建立排程。  這可讓使用者不必指定時間，但您不能在解決方案中包含排程，以便在移除解決方案時將該排程移除。


### <a name="job-schedules"></a>作業排程
作業排程資源會連結 Runbook 與排程。  作業排程資源的類型為 **Microsoft.Automation/automationAccounts/jobSchedules**，且具有下列結構。  這包括一般變數和參數，因此您可以將此程式碼片段複製並貼到您的解決方案檔，然後變更參數名稱。 

    {
      "name": "[concat(parameters('accountName'), '/', variables('Schedule').LinkGuid)]",
      "type": "microsoft.automation/automationAccounts/jobSchedules",
      "apiVersion": "[variables('AutomationApiVersion')]",
      "location": "[parameters('regionId')]",
      "dependsOn": [
        "[resourceId('Microsoft.Automation/automationAccounts/runbooks/', parameters('accountName'), variables('Runbook').Name)]",
        "[resourceId('Microsoft.Automation/automationAccounts/schedules/', parameters('accountName'), variables('Schedule').Name)]"
      ],
      "tags": {
      },
      "properties": {
        "schedule": {
          "name": "[variables('Schedule').Name]"
        },
        "runbook": {
          "name": "[variables('Runbook').Name]"
        }
      }
    }


下表會說明作業排程的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| 排程名稱 |包含排程名稱的單一**名稱**實體。 |
| Runbook 名稱  |包含 Runbook 名稱的單一**名稱**實體。  |



## <a name="variables"></a>變數
[Azure 自動化變數](../automation/automation-variables.md)的類型為 **Microsoft.Automation/automationAccounts/variables**，且具有下列結構。  這包括一般變數和參數，因此您可以將此程式碼片段複製並貼到您的解決方案檔，然後變更參數名稱。

    {
      "name": "[concat(parameters('accountName'), '/', variables('Variable').Name)]",
      "type": "microsoft.automation/automationAccounts/variables",
      "apiVersion": "[variables('AutomationApiVersion')]",
      "tags": { },
      "dependsOn": [
      ],
      "properties": {
        "description": "[variables('Variable').Description]",
        "isEncrypted": "[variables('Variable').Encrypted]",
        "type": "[variables('Variable').Type]",
        "value": "[variables('Variable').Value]"
      }
    }

下表會說明變數資源的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| 說明 | 變數的選擇性說明。 |
| isEncrypted | 指定是否應加密變數。 |
| type | 這個屬性目前沒有任何作用。  變數的資料類型將由初始值所決定。 |
| value | 變數的值。 |

> [!NOTE]
> **type** 屬性目前不會影響所建立的變數。  變數的資料類型將由 value 所決定。  

如果您設定變數的初始值，則必須將它設定為正確的資料類型。  下表提供允許的不同資料類型和其語法。  請注意，JSON 中的值應該一律要用引號括住，並以括號括住任何特殊字元。  例如，以括住字串的引號指定字串值 (使用逸出字元 (\\))，並以一組引號指定數值。

| 資料類型 | 說明 | 範例 | 解析成 |
|:--|:--|:--|:--|
| 字串   | 以雙引號括住值。  | "\"Hello world\"" | "Hello world" |
| numeric  | 以單引號括住數值。| "64" | 64 |
| 布林值  | 以引號括住 **true** 或 **false**。  請注意，此值必須是小寫。 | "true" | true |
| Datetime | 序列化的日期值。<br>您可以在 PowerShell 中使用 ConvertTo-Json Cmdlet，以針對特定日期產生這個值。<br>範例：get-date "5/24/2017 13:14:57" \| ConvertTo-Json | "\\/Date(1495656897378)\\/" | 2017-05-24 13:14:57 |

## <a name="modules"></a>模組
您的管理解決方案不需定義您的 Runbook 所用的[全域模組](../automation/automation-integration-modules.md)，因為您永遠可以在自動化帳戶中使用這些模組。  對於 Runbook 所使用的其他任何模組，您不需要包含其資源。

[整合模組](../automation/automation-integration-modules.md)的類型為 **Microsoft.Automation/automationAccounts/modules**，且具有下列結構。  這包括一般變數和參數，因此您可以將此程式碼片段複製並貼到您的解決方案檔，然後變更參數名稱。

    {
      "name": "[concat(parameters('accountName'), '/', variables('Module').Name)]",
      "type": "Microsoft.Automation/automationAccounts/modules",
      "apiVersion": "[variables('AutomationApiVersion')]",
      "dependsOn": [
      ],
      "properties": {
        "contentLink": {
          "uri": "[variables('Module').Uri]"
        }
      }
    }


下表會說明模組資源的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| contentLink |指定模組的內容。 <br><br>uri - 模組內容的 Uri。  這會是 PowerShell 和 Script Runbook 的 .ps1 檔案，以及 Graph Runbook 的已匯出圖形化 Runbook 檔案。  <br> 版本 - 自有追蹤的模組版本。 |

Runbook 應該相依於模組資源，以確保模組資源會比 Runbook 還早建立。

### <a name="updating-modules"></a>更新模組
如果您更新的管理解決方案包含使用排程的 Runbook，而且新版的解決方案具有該 Runbook 所用的新模組，則 Runbook 可能會使用舊版的模組。  您應該在您的解決方案中納入下列 Runbook，並建立一項作業，以在任何其他 Runbook 之前執行這些 Runbook。  這可確保在載入 Runbook 之前，會視需要更新任何模組。

* [Update-ModulesinAutomationToLatestVersion](https://www.powershellgallery.com/packages/Update-ModulesInAutomationToLatestVersion/1.03/DisplayScript) 可確保您解決方案中 Runbook 使用的所有模組都是最新版本。  
* [ReRegisterAutomationSchedule-MS-Mgmt](https://www.powershellgallery.com/packages/ReRegisterAutomationSchedule-MS-Mgmt/1.0/DisplayScript) 可重新註冊所有的排程資源，以確保已連結到這些資源的 Runbook 使用最新模組。




## <a name="sample"></a>範例
以下是解決方案的範例，其中包含下列資源：

- Runbook。  這是儲存在公用 GitHub 儲存機制的 Runbook 範例。
- 在安裝解決方案時會啟動 Runbook 的自動化作業。
- 用來定期啟動 Runbook 的排程和作業排程。
- 憑證。
- 認證。
- 變數。
- 模組。  這個 [OMSIngestionAPI 模組](https://www.powershellgallery.com/packages/OMSIngestionAPI/1.5)可用來將資料寫入 Log Analytics。 

此範例會使用[標準的解決方案參數]( monitoring-solutions-solution-file.md#parameters)變數，相對於資源定義中的硬式編碼值，這類變數常用於解決方案中。


    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "workspaceName": {
          "type": "string",
          "metadata": {
            "Description": "Name of Log Analytics workspace."
          }
        },
        "accountName": {
          "type": "string",
          "metadata": {
            "Description": "Name of Automation account."
          }
        },
        "workspaceregionId": {
          "type": "string",
          "metadata": {
            "Description": "Region of Log Analytics workspace."
          }
        },
        "regionId": {
          "type": "string",
          "metadata": {
            "Description": "Region of Automation account."
          }
        },
        "pricingTier": {
          "type": "string",
          "metadata": {
            "Description": "Pricing tier of both Log Analytics workspace and Azure Automation account."
          }
        },
        "certificateBase64Value": {
          "type": "string",
          "metadata": {
            "Description": "Base 64 value for certificate."
          }
        },
        "certificateThumbprint": {
          "type": "securestring",
          "metadata": {
            "Description": "Thumbprint for certificate."
          }
        },
        "credentialUsername": {
          "type": "string",
          "metadata": {
            "Description": "Username for credential."
          }
        },
        "credentialPassword": {
          "type": "securestring",
          "metadata": {
            "Description": "Password for credential."
          }
        },
        "scheduleStartTime": {
          "type": "string",
          "metadata": {
            "Description": "Start time for shedule."
          }
        },
        "scheduleTimeZone": {
          "type": "string",
          "metadata": {
            "Description": "Time zone for schedule."
          }
        },
        "scheduleLinkGuid": {
          "type": "string",
          "metadata": {
            "description": "GUID for the schedule link to runbook.",
            "control": "guid"
          }
        },
        "runbookJobGuid": {
          "type": "string",
          "metadata": {
            "description": "GUID for the runbook job.",
            "control": "guid"
          }
        }
      },
      "variables": {
        "SolutionName": "MySolution",
        "SolutionVersion": "1.0",
        "SolutionPublisher": "Contoso",
        "ProductName": "SampleSolution",
    
        "LogAnalyticsApiVersion": "2015-11-01-preview",
        "AutomationApiVersion": "2015-10-31",
    
        "Runbook": {
          "Name": "MyRunbook",
          "Description": "Sample runbook",
          "Type": "PowerShell",
          "Uri": "https://raw.githubusercontent.com/user/myrepo/master/samples/MyRunbook.ps1",
          "JobGuid": "[parameters('runbookJobGuid')]"
        },
    
        "Certificate": {
          "Name": "MyCertificate",
          "Base64Value": "[parameters('certificateBase64Value')]",
          "Thumbprint": "[parameters('certificateThumbprint')]"
        },
    
        "Credential": {
          "Name": "MyCredential",
          "UserName": "[parameters('credentialUsername')]",
          "Password": "[parameters('credentialPassword')]"
        },
    
        "Schedule": {
          "Name": "MySchedule",
          "Description": "Sample schedule",
          "IsEnabled": "true",
          "Interval": "1",
          "Frequency": "hour",
          "StartTime": "[parameters('scheduleStartTime')]",
          "TimeZone": "[parameters('scheduleTimeZone')]",
          "LinkGuid": "[parameters('scheduleLinkGuid')]"
        },
    
        "Variable": {
          "Name": "MyVariable",
          "Description": "Sample variable",
          "Encrypted": 0,
          "Type": "string",
          "Value": "'This is my string value.'"
        },
    
        "Module": {
          "Name": "OMSIngestionAPI",
          "Uri": "https://devopsgallerystorage.blob.core.windows.net/packages/omsingestionapi.1.3.0.nupkg"
        }
      },
      "resources": [
        {
          "name": "[concat(variables('SolutionName'), '[' ,parameters('workspacename'), ']')]",
          "location": "[parameters('workspaceRegionId')]",
          "tags": { },
          "type": "Microsoft.OperationsManagement/solutions",
          "apiVersion": "[variables('LogAnalyticsApiVersion')]",
          "dependsOn": [
            "[resourceId('Microsoft.Automation/automationAccounts/runbooks/', parameters('accountName'), variables('Runbook').Name)]",
            "[resourceId('Microsoft.Automation/automationAccounts/jobs/', parameters('accountName'), variables('Runbook').JobGuid)]",
            "[resourceId('Microsoft.Automation/automationAccounts/certificates/', parameters('accountName'), variables('Certificate').Name)]",
            "[resourceId('Microsoft.Automation/automationAccounts/credentials/', parameters('accountName'), variables('Credential').Name)]",
            "[resourceId('Microsoft.Automation/automationAccounts/schedules/', parameters('accountName'), variables('Schedule').Name)]",
            "[resourceId('Microsoft.Automation/automationAccounts/jobSchedules/', parameters('accountName'), variables('Schedule').LinkGuid)]",
            "[resourceId('Microsoft.Automation/automationAccounts/variables/', parameters('accountName'), variables('Variable').Name)]",
            "[resourceId('Microsoft.Automation/automationAccounts/modules/', parameters('accountName'), variables('Module').Name)]"
          ],
          "properties": {
            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces', parameters('workspacename'))]",
            "referencedResources": [
              "[resourceId('Microsoft.Automation/automationAccounts/modules/', parameters('accountName'), variables('Module').Name)]"
            ],
            "containedResources": [
              "[resourceId('Microsoft.Automation/automationAccounts/runbooks/', parameters('accountName'), variables('Runbook').Name)]",
              "[resourceId('Microsoft.Automation/automationAccounts/jobs/', parameters('accountName'), variables('Runbook').JobGuid)]",
              "[resourceId('Microsoft.Automation/automationAccounts/certificates/', parameters('accountName'), variables('Certificate').Name)]",
              "[resourceId('Microsoft.Automation/automationAccounts/credentials/', parameters('accountName'), variables('Credential').Name)]",
              "[resourceId('Microsoft.Automation/automationAccounts/schedules/', parameters('accountName'), variables('Schedule').Name)]",
              "[resourceId('Microsoft.Automation/automationAccounts/jobSchedules/', parameters('accountName'), variables('Schedule').LinkGuid)]",
              "[resourceId('Microsoft.Automation/automationAccounts/variables/', parameters('accountName'), variables('Variable').Name)]"
            ]
          },
          "plan": {
            "name": "[concat(variables('SolutionName'), '[' ,parameters('workspaceName'), ']')]",
            "Version": "[variables('SolutionVersion')]",
            "product": "[variables('ProductName')]",
            "publisher": "[variables('SolutionPublisher')]",
            "promotionCode": ""
          }
        },
        {
          "name": "[concat(parameters('accountName'), '/', variables('Runbook').Name)]",
          "type": "Microsoft.Automation/automationAccounts/runbooks",
          "apiVersion": "[variables('AutomationApiVersion')]",
          "dependsOn": [
          ],
          "location": "[parameters('regionId')]",
          "tags": { },
          "properties": {
            "runbookType": "[variables('Runbook').Type]",
            "logProgress": "true",
            "logVerbose": "true",
            "description": "[variables('Runbook').Description]",
            "publishContentLink": {
              "uri": "[variables('Runbook').Uri]",
              "version": "1.0.0.0"
            }
          }
        },
        {
          "name": "[concat(parameters('accountName'), '/', variables('Runbook').JobGuid)]",
          "type": "Microsoft.Automation/automationAccounts/jobs",
          "apiVersion": "[variables('AutomationApiVersion')]",
          "location": "[parameters('regionId')]",
          "dependsOn": [
            "[concat('Microsoft.Automation/automationAccounts/', parameters('accountName'), '/runbooks/', variables('Runbook').Name)]"
          ],
          "tags": { },
          "properties": {
            "runbook": {
              "name": "[variables('Runbook').Name]"
            },
            "parameters": {
              "targetSubscriptionId": "[subscription().subscriptionId]",
              "resourcegroup": "[resourceGroup().name]",
              "automationaccount": "[parameters('accountName')]"
            }
          }
        },
        {
          "name": "[concat(parameters('accountName'), '/', variables('Certificate').Name)]",
          "type": "Microsoft.Automation/automationAccounts/certificates",
          "apiVersion": "[variables('AutomationApiVersion')]",
          "location": "[parameters('regionId')]",
          "tags": { },
          "dependsOn": [
          ],
          "properties": {
            "Base64Value": "[variables('Certificate').Base64Value]",
            "Thumbprint": "[variables('Certificate').Thumbprint]"
          }
        },
        {
          "name": "[concat(parameters('accountName'), '/', variables('Credential').Name)]",
          "type": "Microsoft.Automation/automationAccounts/credentials",
          "apiVersion": "[variables('AutomationApiVersion')]",
          "location": "[parameters('regionId')]",
          "tags": { },
          "dependsOn": [
          ],
          "properties": {
            "userName": "[variables('Credential').UserName]",
            "password": "[variables('Credential').Password]"
          }
        },
        {
          "name": "[concat(parameters('accountName'), '/', variables('Schedule').Name)]",
          "type": "microsoft.automation/automationAccounts/schedules",
          "apiVersion": "[variables('AutomationApiVersion')]",
          "tags": { },
          "dependsOn": [
          ],
          "properties": {
            "description": "[variables('Schedule').Description]",
            "startTime": "[variables('Schedule').StartTime]",
            "timeZone": "[variables('Schedule').TimeZone]",
            "isEnabled": "[variables('Schedule').IsEnabled]",
            "interval": "[variables('Schedule').Interval]",
            "frequency": "[variables('Schedule').Frequency]"
          }
        },
        {
          "name": "[concat(parameters('accountName'), '/', variables('Schedule').LinkGuid)]",
          "type": "microsoft.automation/automationAccounts/jobSchedules",
          "apiVersion": "[variables('AutomationApiVersion')]",
          "location": "[parameters('regionId')]",
          "dependsOn": [
            "[resourceId('Microsoft.Automation/automationAccounts/runbooks/', parameters('accountName'), variables('Runbook').Name)]",
            "[resourceId('Microsoft.Automation/automationAccounts/schedules/', parameters('accountName'), variables('Schedule').Name)]"
          ],
          "tags": {
          },
          "properties": {
            "schedule": {
              "name": "[variables('Schedule').Name]"
            },
            "runbook": {
              "name": "[variables('Runbook').Name]"
            }
          }
        },
        {
          "name": "[concat(parameters('accountName'), '/', variables('Variable').Name)]",
          "type": "microsoft.automation/automationAccounts/variables",
          "apiVersion": "[variables('AutomationApiVersion')]",
          "tags": { },
          "dependsOn": [
          ],
          "properties": {
            "description": "[variables('Variable').Description]",
            "isEncrypted": "[variables('Variable').Encrypted]",
            "type": "[variables('Variable').Type]",
            "value": "[variables('Variable').Value]"
          }
        },
        {
          "name": "[concat(parameters('accountName'), '/', variables('Module').Name)]",
          "type": "Microsoft.Automation/automationAccounts/modules",
          "apiVersion": "[variables('AutomationApiVersion')]",
          "dependsOn": [
          ],
          "properties": {
            "contentLink": {
              "uri": "[variables('Module').Uri]"
            }
          }
        }
    
      ],
      "outputs": { }
    }




## <a name="next-steps"></a>後續步驟
* [將檢視新增至您的解決方案]( monitoring-solutions-resources-views.md)，以將所收集的資料視覺化。
