---
title: 在 Azure 中建立管理解決方案檔 | Microsoft Docs
description: 管理解決方案提供客戶可新增至其 Azure 環境的已封裝管理案例。  這篇文章提供詳細資料，說明如何建立要用於自己的環境中或可供客戶使用的管理解決方案。
services: monitoring
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: 1915e204-ba7e-431b-9718-9eb6b4213ad8
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/09/2018
ms.author: bwren
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 46e6ea791752045b0f1afbf1e83e43f498415e54
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33887458"
---
# <a name="creating-a-management-solution-file-in-azure-preview"></a>在 Azure 中建立管理解決方案檔 (預覽)
> [!NOTE]
> 這是在 Azure 中建立管理解決方案 (目前處於預覽狀態) 的預備文件。 以下所述的任何結構描述可能會有所變更。  

Azure 中的管理解決方案會實作為 [Resource Manager 範本](../azure-resource-manager/resource-manager-template-walkthrough.md)。  學習如何撰寫管理解決方案的主要工作，是學習如何[撰寫範本](../azure-resource-manager/resource-group-authoring-templates.md)。  本文提供用於解決方案的範本獨特詳細資料，以及設定一般解決方案資源的方式。


## <a name="tools"></a>工具

您可以使用任何文字編輯器來處理解決方案檔，但建議您利用 Visual Studio 或 Visual Studio Code 中提供的功能，如下列文章所述。

- [透過 Visual Studio 建立與部署 Azure 資源群組](../azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy.md)
- [在 Visual Studio Code 中使用 Azure Resource Manager 範本](../azure-resource-manager/resource-manager-vs-code.md)




## <a name="structure"></a>Structure
管理解決方案檔案與[資源管理員範本](../azure-resource-manager/resource-group-authoring-templates.md#template-format)的基本結構相同，如下所示。  下列各節說明最上層元素及其在解決方案中的內容。  

    {
       "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0",
       "parameters": {  },
       "variables": {  },
       "resources": [  ],
       "outputs": {  }
    }

## <a name="parameters"></a>參數
[Parameters](../azure-resource-manager/resource-group-authoring-templates.md#parameters) 是您在使用者安裝解決方案時向他們要求的值。  所有解決方案都會有標準參數，而您可以視需要針對特定解決方案新增額外的參數。  使用者在安裝解決方案時提供參數值的方式，將取決於特定參數以及解決方案的安裝方式。

當使用者透過 Azure Marketplace 或 Azure 快速入門範本[安裝您的管理解決方案](monitoring-solutions.md#install-a-management-solution)時，系統會提示他們選取 [Log Analytics 工作區和自動化帳戶](monitoring-solutions.md#log-analytics-workspace-and-automation-account)。  這些用來填入每個標準參數的值。  系統不會提示使用者直接提供標準參數的值，但會提示他們提供任何其他參數的值。


範例參數如下所示。  

    "startTime": {
        "type": "string",
        "metadata": {
            "description": "Enter time for starting VMs by resource group.",
            "control": "datetime",
            "category": "Schedule"
        }

下表說明參數的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| type |變數的資料類型。 針對使用者顯示的輸入控制項視資料類型而定。<br><br>bool - 下拉式方塊<br>string - 文字方塊<br>int - 文字方塊<br>securestring - 密碼欄位<br> |
| category |參數的選擇性類別。  相同類別中的參數會群組在一起。 |
| control |string 參數的其他功能。<br><br>datetime - Datetime 控制項隨即顯示。<br>guid - 會自動產生 Guid 值，但未顯示此參數。 |
| 說明 |參數的選擇性說明。  顯示於參數旁邊的資訊球形文字說明。 |

### <a name="standard-parameters"></a>標準參數
下表列出所有管理解決方案的標準參數。  系統會為使用者填入這些值，而不會在他們從 Azure Marketplace 或快速入門範本安裝解決方案時提示他們輸入這些值。  如果以其他方法安裝解決方案，使用者必須提供這些值。

> [!NOTE]
> Azure Marketplace 和快速入門範本中的使用者介面需有表格中的參數名稱。  如果您使用不同的參數名稱，則會提示使用者輸入其值，而不會自動填入。
>
>

| 參數 | 類型 | 說明 |
|:--- |:--- |:--- |
| accountName |字串 |Azure 自動化帳戶名稱。 |
| pricingTier |字串 |Log Analytics 工作區和 Azure 自動化帳戶的定價層。 |
| regionId |字串 |Azure 自動化帳戶的區域。 |
| solutionName |字串 |解決方案的名稱。  如果您是透過快速入門範本部署解決方案，則您應該將 solutionName 定義為參數，如此您就可以定義字串，而不需要使用者來指定。 |
| workspaceName |字串 |Log Analytics 工作區名稱。 |
| workspaceRegionId |字串 |Log Analytics 工作區的區域。 |


以下是您可以複製並貼到您的方案檔案中的標準參數結構。  

    "parameters": {
        "workspaceName": {
            "type": "string",
            "metadata": {
                "description": "A valid Log Analytics workspace name"
            }
        },
        "accountName": {
               "type": "string",
               "metadata": {
                   "description": "A valid Azure Automation account name"
               }
        },
        "workspaceRegionId": {
               "type": "string",
               "metadata": {
                   "description": "Region of the Log Analytics workspace"
            }
        },
        "regionId": {
            "type": "string",
            "metadata": {
                "description": "Region of the Azure Automation account"
            }
        },
        "pricingTier": {
            "type": "string",
            "metadata": {
                "description": "Pricing tier of both Log Analytics workspace and Azure Automation account"
            }
        }
    }


參考解決方案的其他項目中使用 **parameters('parameter name')** 語法的參數值。  例如，若要存取工作區名稱，您會使用 **parameters('workspaceName')**

## <a name="variables"></a>變數
[變數](../azure-resource-manager/resource-group-authoring-templates.md#variables)是您會在管理解決方案的其餘部分中使用的值。  這些值不會公開給安裝解決方案的使用者。  它們旨在為作者提供單一位置，讓他們可以管理在整個解決方案可能會重複使用的值。 您應該將解決方案特有的值放在變數中，而非將這些值硬式編碼在 **resources** 元素中。  這會讓程式碼更容易閱讀，並可讓您輕鬆地在後續的版本中變更這些值。

下列範例為 **variables** 項目，包含解決方案中所使用的一般參數。

    "variables": {
        "SolutionVersion": "1.1",
        "SolutionPublisher": "Contoso",
        "SolutionName": "My Solution",
        "LogAnalyticsApiVersion": "2015-11-01-preview",
        "AutomationApiVersion": "2015-10-31"
    },

參考透過使用 **variables('variable name')** 語法的解決方案參數值。  例如，若要存取 SolutionName 變數，您會使用 **variables('SolutionName')**。

您也可以定義有多組值的複雜變數。  這在您會針對不同的資源類型定義多個屬性的管理解決方案中，這特別實用。  例如，您可以將上述的解決方案變數重建為下列所示的狀態。

    "variables": {
        "Solution": {
          "Version": "1.1",
          "Publisher": "Contoso",
          "Name": "My Solution"
        },
        "LogAnalyticsApiVersion": "2015-11-01-preview",
        "AutomationApiVersion": "2015-10-31"
    },

在此情況下，您會以 **variables('variable name').property** 語法透過解決方案參考變數。  例如，若要存取 Solution Name 變數，您會使用 **variables('Solution').Name**。

## <a name="resources"></a>資源
[資源](../azure-resource-manager/resource-group-authoring-templates.md#resources)會定義您的管理解決方案將會安裝並設定的不同資源。  這是範本最大且最複雜的部分。  您可以在[編寫 Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md#resources)中取得 resource 元素的架構和完整描述。  本文件中的其他文章會詳述您經常定義的其他資源。 


### <a name="dependencies"></a>相依性
**dependsOn** 元素指定對另一個資源的[相依性](../azure-resource-manager/resource-group-define-dependencies.md)。  安裝解決方案時，直到所有相依性建立後才會建立資源。  例如，解決方案可能會在使用[作業資源](monitoring-solutions-resources-automation.md#automation-jobs)安裝時[啟動 Runbook](monitoring-solutions-resources-automation.md#runbooks)。  作業資源會相依於 Runbook 資源，以確保在建立作業前建立 Runbook。

### <a name="log-analytics-workspace-and-automation-account"></a>Log Analytics 工作區和自動化帳戶
管理解決方案需要 [Log Analytics 工作區](../log-analytics/log-analytics-manage-access.md)來包含檢視，以及[自動化帳戶](../automation/automation-security-overview.md#automation-account-overview)來包含 Runbook 和相關資源。  這些項目必須在建立解決方案中的資源前取得，且不得定義於解決方案本身。  使用者將會在部署解決方案時[指定工作區和帳戶](monitoring-solutions.md#log-analytics-workspace-and-automation-account)，但身為作者，您應該考慮下列幾點。


## <a name="solution-resource"></a>解決方案資源
每個解決方案需要**資源**項目中定義解決方案本身的資源項目。  這會有一種 **Microsoft.OperationsManagement/solutions** 並有下列結構。 這包括經常用來定義解決方案屬性的[標準參數](#parameters)和[變數](#variables)。


    {
      "name": "[concat(variables('Solution').Name, '[' ,parameters('workspaceName'), ']')]",
      "location": "[parameters('workspaceRegionId')]",
      "tags": { },
      "type": "Microsoft.OperationsManagement/solutions",
      "apiVersion": "[variables('LogAnalyticsApiVersion')]",
      "dependsOn": [
        <list-of-resources>
      ],
      "properties": {
        "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces', parameters('workspaceName'))]",
        "referencedResources": [
            <list-of-referenced-resources>
        ],
        "containedResources": [
            <list-of-contained-resources>
        ]
      },
      "plan": {
        "name": "[concat(variables('Solution').Name, '[' ,parameters('workspaceName'), ']')]",
        "Version": "[variables('Solution').Version]",
        "product": "[variables('ProductName')]",
        "publisher": "[variables('Solution').Publisher]",
        "promotionCode": ""
      }
    }




### <a name="dependencies"></a>相依性
解決方案資源在解決方案中的每隔一個資源上須有[相依性](../azure-resource-manager/resource-group-define-dependencies.md)，因為必須先存在相依性，才能建立解決方案。  您可以在 **dependsOn** 項目中針對每個資源新增一個項目。

### <a name="properties"></a>properties
解決方案資源具有下表中的屬性。  這包括由定義解決方案安裝後如何管理資源的解決方案所參考及包含的資源。  解決方案中的每個資源應列在 **referencedResources** 或 **containedResources** 屬性中。

| 屬性 | 說明 |
|:--- |:--- |
| workspaceResourceId |具有以下形式的 Log Analytics 工作區識別碼：*<Resource Group ID>/providers/Microsoft.OperationalInsights/workspaces/\<工作區名稱\>*。 |
| referencedResources |解決方案移除時不應移除的解決方案資源清單。 |
| containedResources |解決方案移除時應移除的解決方案資源清單。 |

上述範例適用於具有 Runbook、排程和檢視的解決方案。  **properties** 元素會「參考」排程和 Runbook，因此在移除解決方案時不會移除它們。  會*包含*檢視，因此當移除解決方案時會移除它。

### <a name="plan"></a>規劃
解決方案資源的**計劃**實體具有下表中的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| name |解決方案的名稱。 |
| version |作者所決定的解決方案版本。 |
| product |識別解決方案的唯一字串。 |
| publisher |解決方案的發佈者。 |



## <a name="sample"></a>範例
您可以在以下位置檢視具有解決方案資源的解決方案檔範例。

- [自動化資源](monitoring-solutions-resources-automation.md#sample)
- [搜尋和警示資源](monitoring-solutions-resources-searches-alerts.md#sample)


## <a name="next-steps"></a>後續步驟
* 在您的管理解決方案中[新增儲存的搜尋和警示](monitoring-solutions-resources-searches-alerts.md)。
* 在您的管理解決方案中[新增檢視](monitoring-solutions-resources-views.md)。
* 在您的管理解決方案中[新增 Runbook 及其他自動化資源](monitoring-solutions-resources-automation.md)。
* 了解[編寫 Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)的詳細資料。
* 搜尋 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates)不同 Resource Manager 範本的範例。
