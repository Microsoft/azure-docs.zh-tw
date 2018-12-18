---
title: 使用 PowerShell 將 Azure Application Insights 自動化 | Microsoft Docs
description: 在 PowerShell 中使用 Azure Resource Manager 範本自動建立資源、警示及可用性測試。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 9f73b87f-be63-4847-88c8-368543acad8b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 04/02/2017
ms.author: mbullwin
ms.openlocfilehash: cfed1636bf27279b8a391559d3e88b823036f703
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35632983"
---
#  <a name="create-application-insights-resources-using-powershell"></a>使用 PowerShell 建立 Application Insights 資源
本文說明如何使用 Azure 資源管理，自動將 [Application Insights](app-insights-overview.md) 資源的建立和更新自動化。 例如，您可能建置程序中這麼做。 除了基本的 Application Insights 資源外，您可以建立[可用性 Web 測試](app-insights-monitor-web-app-availability.md)、設定[警示](app-insights-alerts.md)、設定[價格配置](app-insights-pricing.md)和建立其他 Azure 資源。

建立這些資源的關鍵是 [Azure 資源管理員](../azure-resource-manager/powershell-azure-resource-manager.md)適用的 JSON 範本。 簡單地說，此程序是：下載現有資源的 JSON 定義；參數化某些值 (例如名稱)；然後每當您想建立新的資源時再執行範本。 您可以一起封裝幾項資源一次全部建立，例如一個包含可用性測試、警示和連續匯出儲存體的應用程式監視器。 部分參數化有一些微妙之處，我們會在這裡說明。

## <a name="one-time-setup"></a>單次設定
若您未曾將 PowerShell 與 Azure 訂用帳戶搭配使用：

在您要執行指令碼的電腦上安裝 Azure Powershell 模組：

1. 安裝 [Microsoft Web Platform Installer (v5 或更新版本)](http://www.microsoft.com/web/downloads/platform.aspx)。
2. 請使用它來安裝 Microsoft Azure Powershell。

## <a name="create-an-azure-resource-manager-template"></a>建立 Azure Resource Manager 範本
建立新的 .json 檔案 - 在此範例中稱為 `template1.json` 。 將此內容複製到其中：

```JSON
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "appName": {
                "type": "string",
                "metadata": {
                    "description": "Enter the application name."
                }
            },
            "appType": {
                "type": "string",
                "defaultValue": "web",
                "allowedValues": [
                    "web",
                    "java",
                    "other"
                ],
                "metadata": {
                    "description": "Enter the application type."
                }
            },
            "appLocation": {
                "type": "string",
                "defaultValue": "East US",
                "allowedValues": [
                    "South Central US",
                    "West Europe",
                    "East US",
                    "North Europe"
                ],
                "metadata": {
                    "description": "Enter the application location."
                }
            },
            "priceCode": {
                "type": "int",
                "defaultValue": 1,
                "allowedValues": [
                    1,
                    2
                ],
                "metadata": {
                    "description": "1 = Basic, 2 = Enterprise"
                }
            },
            "dailyQuota": {
                "type": "int",
                "defaultValue": 100,
                "minValue": 1,
                "metadata": {
                    "description": "Enter daily quota in GB."
                }
            },
            "dailyQuotaResetTime": {
                "type": "int",
                "defaultValue": 24,
                "metadata": {
                    "description": "Enter daily quota reset hour in UTC (0 to 23). Values outside the range will get a random reset hour."
                }
            },
            "warningThreshold": {
                "type": "int",
                "defaultValue": 90,
                "minValue": 1,
                "maxValue": 100,
                "metadata": {
                    "description": "Enter the % value of daily quota after which warning mail to be sent. "
                }
            }
        },
        "variables": {
            "priceArray": [
                "Basic",
                "Application Insights Enterprise"
            ],
            "pricePlan": "[take(variables('priceArray'),parameters('priceCode'))]",
            "billingplan": "[concat(parameters('appName'),'/', variables('pricePlan')[0])]"
        },
        "resources": [
            {
                "type": "microsoft.insights/components",
                "kind": "[parameters('appType')]",
                "name": "[parameters('appName')]",
                "apiVersion": "2014-04-01",
                "location": "[parameters('appLocation')]",
                "tags": {},
                "properties": {
                    "ApplicationId": "[parameters('appName')]"
                },
                "dependsOn": []
            },
            {
                "name": "[variables('billingplan')]",
                "type": "microsoft.insights/components/CurrentBillingFeatures",
                "location": "[parameters('appLocation')]",
                "apiVersion": "2015-05-01",
                "dependsOn": [
                    "[resourceId('microsoft.insights/components', parameters('appName'))]"
                ],
                "properties": {
                    "CurrentBillingFeatures": "[variables('pricePlan')]",
                    "DataVolumeCap": {
                        "Cap": "[parameters('dailyQuota')]",
                        "WarningThreshold": "[parameters('warningThreshold')]",
                        "ResetTime": "[parameters('dailyQuotaResetTime')]"
                    }
                }
            }
        ]
    }
```



## <a name="create-application-insights-resources"></a>建立 Application Insights 資源
1. 在 PowerShell 中，登入 Azure：
   
    `Connect-AzureRmAccount`
2. 執行如下命令：
   
    ```PS
   
        New-AzureRmResourceGroupDeployment -ResourceGroupName Fabrikam `
               -TemplateFile .\template1.json `
               -appName myNewApp

    ``` 
   
   * `-ResourceGroupName` 是您要在其中建立新資源的群組。
   * `-TemplateFile` 必須出現在自訂參數之前。
   * `-appName` 是要建立的資源的名稱。

您可以新增其他參數 - 可在範本的參數區段中找到其描述。

## <a name="to-get-the-instrumentation-key"></a>取得檢測金鑰
建立應用程式資源之後，您會想要檢測金鑰： 

```PS
    $resource = Find-AzureRmResource -ResourceNameEquals "<YOUR APP NAME>" -ResourceType "Microsoft.Insights/components"
    $details = Get-AzureRmResource -ResourceId $resource.ResourceId
    $ikey = $details.Properties.InstrumentationKey
```


<a id="price"></a>
## <a name="set-the-price-plan"></a>設定價格方案

您可以設定[價格方案](app-insights-pricing.md)。

若要使用企業價格計劃建立應用程式資源，請使用上述的範本︰

```PS
        New-AzureRmResourceGroupDeployment -ResourceGroupName Fabrikam `
               -TemplateFile .\template1.json `
               -priceCode 2 `
               -appName myNewApp
```

|priceCode|計劃|
|---|---|
|1|基本|
|2|Enterprise|

* 如果您只想要使用預設基本價格方案，您可以從範本中省略 CurrentBillingFeatures 資源。
* 如果您想在建立元件資源之後變更價格方案，可以使用省略 "microsoft.insights/components" 資源的範本。 此外，也從計費資源省略 `dependsOn` 節點。 

若要驗證更新的定價方案，請在瀏覽器中查看 [使用量和估計成本頁面] 刀鋒視窗。 「重新整理瀏覽器檢視」以確保您看到的是最新的狀態。



## <a name="add-a-metric-alert"></a>新增度量警示

若要與您的應用程式資源同時設定度量警示，請將類似的程式碼合併至範本檔案︰

```JSON
{
    parameters: { ... // existing parameters ...
            "responseTime": {
              "type": "int",
              "defaultValue": 3,
              "minValue": 1,
              "metadata": {
                "description": "Enter response time threshold in seconds."
              }
    },
    variables: { ... // existing variables ...
      // Alert names must be unique within resource group.
      "responseAlertName": "[concat('ResponseTime-', toLower(parameters('appName')))]"
    }, 
    resources: { ... // existing resources ...
     {
      //
      // Metric alert on response time
      //
      "name": "[variables('responseAlertName')]",
      "type": "Microsoft.Insights/alertrules",
      "apiVersion": "2014-04-01",
      "location": "[parameters('appLocation')]",
      // Ensure this resource is created after the app resource:
      "dependsOn": [
        "[resourceId('Microsoft.Insights/components', parameters('appName'))]"
      ],
      "tags": {
        "[concat('hidden-link:', resourceId('Microsoft.Insights/components', parameters('appName')))]": "Resource"
      },
      "properties": {
        "name": "[variables('responseAlertName')]",
        "description": "response time alert",
        "isEnabled": true,
        "condition": {
          "$type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.ThresholdRuleCondition, Microsoft.WindowsAzure.Management.Mon.Client",
          "odata.type": "Microsoft.Azure.Management.Insights.Models.ThresholdRuleCondition",
          "dataSource": {
            "$type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.RuleMetricDataSource, Microsoft.WindowsAzure.Management.Mon.Client",
            "odata.type": "Microsoft.Azure.Management.Insights.Models.RuleMetricDataSource",
            "resourceUri": "[resourceId('microsoft.insights/components', parameters('appName'))]",
            "metricName": "request.duration"
          },
          "threshold": "[parameters('responseTime')]", //seconds
          "windowSize": "PT15M" // Take action if changed state for 15 minutes
        },
        "actions": [
          {
            "$type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.RuleEmailAction, Microsoft.WindowsAzure.Management.Mon.Client",
            "odata.type": "Microsoft.Azure.Management.Insights.Models.RuleEmailAction",
            "sendToServiceOwners": true,
            "customEmails": []
          }
        ]
      }
    }
}
```

當您叫用範本時，您可以選擇性地新增此參數︰

    `-responseTime 2`

您當然可以參數化其他欄位。 

若要尋找其他警示規則的類型名稱和組態詳細資料，請以手動方式建立規則，然後在 [Azure Resource Manager](https://resources.azure.com/) 檢查。 


## <a name="add-an-availability-test"></a>新增可用性測試

這個範例是 ping 測試 (以測試單一網頁)。  

可用性測試中**有兩個部分**︰測試本身，以及通知您失敗的警示。

將下列程式碼合併至建立應用程式的範本檔案。

```JSON
{
    parameters: { ... // existing parameters here ...
      "pingURL": { "type": "string" },
      "pingText": { "type": "string" , defaultValue: ""}
    },
    variables: { ... // existing variables here ...
      "pingTestName":"[concat('PingTest-', toLower(parameters('appName')))]",
      "pingAlertRuleName": "[concat('PingAlert-', toLower(parameters('appName')), '-', subscription().subscriptionId)]"
    },
    resources: { ... // existing resources here ...
    { //
      // Availability test: part 1 configures the test
      //
      "name": "[variables('pingTestName')]",
      "type": "Microsoft.Insights/webtests",
      "apiVersion": "2014-04-01",
      "location": "[parameters('appLocation')]",
      // Ensure this is created after the app resource:
      "dependsOn": [
        "[resourceId('Microsoft.Insights/components', parameters('appName'))]"
      ],
      "tags": {
        "[concat('hidden-link:', resourceId('Microsoft.Insights/components', parameters('appName')))]": "Resource"
      },
      "properties": {
        "Name": "[variables('pingTestName')]",
        "Description": "Basic ping test",
        "Enabled": true,
        "Frequency": 900, // 15 minutes
        "Timeout": 120, // 2 minutes
        "Kind": "ping", // single URL test
        "RetryEnabled": true,
        "Locations": [
          {
            "Id": "us-va-ash-azr"
          },
          {
            "Id": "emea-nl-ams-azr"
          },
          {
            "Id": "apac-jp-kaw-edge"
          }
        ],
        "Configuration": {
          "WebTest": "[concat('<WebTest   Name=\"', variables('pingTestName'), '\"   Enabled=\"True\"         CssProjectStructure=\"\"    CssIteration=\"\"  Timeout=\"120\"  WorkItemIds=\"\"         xmlns=\"http://microsoft.com/schemas/VisualStudio/TeamTest/2010\"         Description=\"\"  CredentialUserName=\"\"  CredentialPassword=\"\"         PreAuthenticate=\"True\"  Proxy=\"default\"  StopOnError=\"False\"         RecordedResultFile=\"\"  ResultsLocale=\"\">  <Items>  <Request Method=\"GET\"    Version=\"1.1\"  Url=\"', parameters('Url'),   '\" ThinkTime=\"0\"  Timeout=\"300\" ParseDependentRequests=\"True\"         FollowRedirects=\"True\" RecordResult=\"True\" Cache=\"False\"         ResponseTimeGoal=\"0\"  Encoding=\"utf-8\"  ExpectedHttpStatusCode=\"200\"         ExpectedResponseUrl=\"\" ReportingName=\"\" IgnoreHttpStatusCode=\"False\" />        </Items>  <ValidationRules> <ValidationRule  Classname=\"Microsoft.VisualStudio.TestTools.WebTesting.Rules.ValidationRuleFindText, Microsoft.VisualStudio.QualityTools.WebTestFramework, Version=10.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a\" DisplayName=\"Find Text\"         Description=\"Verifies the existence of the specified text in the response.\"         Level=\"High\"  ExectuionOrder=\"BeforeDependents\">  <RuleParameters>        <RuleParameter Name=\"FindText\" Value=\"',   parameters('pingText'), '\" />  <RuleParameter Name=\"IgnoreCase\" Value=\"False\" />  <RuleParameter Name=\"UseRegularExpression\" Value=\"False\" />  <RuleParameter Name=\"PassIfTextFound\" Value=\"True\" />  </RuleParameters> </ValidationRule>  </ValidationRules>  </WebTest>')]"
        },
        "SyntheticMonitorId": "[variables('pingTestName')]"
      }
    },

    {
      //
      // Availability test: part 2, the alert rule
      //
      "name": "[variables('pingAlertRuleName')]",
      "type": "Microsoft.Insights/alertrules",
      "apiVersion": "2014-04-01",
      "location": "[parameters('appLocation')]", 
      "dependsOn": [
        "[resourceId('Microsoft.Insights/webtests', variables('pingTestName'))]"
      ],
      "tags": {
        "[concat('hidden-link:', resourceId('Microsoft.Insights/components', parameters('appName')))]": "Resource",
        "[concat('hidden-link:', resourceId('Microsoft.Insights/webtests', variables('pingTestName')))]": "Resource"
      },
      "properties": {
        "name": "[variables('pingAlertRuleName')]",
        "description": "alert for web test",
        "isEnabled": true,
        "condition": {
          "$type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.LocationThresholdRuleCondition, Microsoft.WindowsAzure.Management.Mon.Client",
          "odata.type": "Microsoft.Azure.Management.Insights.Models.LocationThresholdRuleCondition",
          "dataSource": {
            "$type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.RuleMetricDataSource, Microsoft.WindowsAzure.Management.Mon.Client",
            "odata.type": "Microsoft.Azure.Management.Insights.Models.RuleMetricDataSource",
            "resourceUri": "[resourceId('microsoft.insights/webtests', variables('pingTestName'))]",
            "metricName": "GSMT_AvRaW"
          },
          "windowSize": "PT15M", // Take action if changed state for 15 minutes
          "failedLocationCount": 2
        },
        "actions": [
          {
            "$type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.RuleEmailAction, Microsoft.WindowsAzure.Management.Mon.Client",
            "odata.type": "Microsoft.Azure.Management.Insights.Models.RuleEmailAction",
            "sendToServiceOwners": true,
            "customEmails": []
          }
        ]
      }
    }
}
```

若要探索其他測試位置的程式碼，或自動建立更複雜的 web 測試，請手動建立範例，然後從 [Azure Resource Manager](https://resources.azure.com/) 參數化程式碼。

## <a name="add-more-resources"></a>新增其他資源

若要自動建立任何類型的任何其他資源，手動建立範例，然後從 [Azure Resource Manager](https://resources.azure.com/) 複製並參數化其程式碼。 

1. 開啟 [Azure 資源管理員](https://resources.azure.com/)。 向下導覽 `subscriptions/resourceGroups/<your resource group>/providers/Microsoft.Insights/components` 直到應用程式資源。 
   
    ![Azure 資源總管中的瀏覽](./media/app-insights-powershell/01.png)
   
    *元件* 是用來顯示應用程式的基本 Application Insights 資源。 相關聯的警示規則和可用性 Web 測試有個別的資源。
2. 將元件的 JSON 複製到 `template1.json`的適當位置。
3. 刪除這些屬性：
   
   * `id`
   * `InstrumentationKey`
   * `CreationDate`
   * `TenantId`
4. 開啟 webtests 和 alertrules 區段，將個別項目的 JSON 複製到您的範本。 (請勿從 webtests 或 alertrules 節點複製：移到其下的項目。)
   
    每個 Web 測試都有一個關聯的警示規則，您必須同時複製這兩者。
   
    您也可以包含計量的警示。 [計量名稱](app-insights-powershell-alerts.md#metric-names)。
5. 在每個資源中插入下面這行：
   
    `"apiVersion": "2015-05-01",`

### <a name="parameterize-the-template"></a>參數化範本
現在您必須以參數取代特定的名稱。 若要[參數化範本](../azure-resource-manager/resource-group-authoring-templates.md)，您要使用[一組協助程式函式](../azure-resource-manager/resource-group-template-functions.md)撰寫表示式。 

您無法將參數化字串的一部分，因此請使用 `concat()` 建置字串。

以下是您會想要進行的替換的範例。 每個替換各出現數次。 您的範本中可能需要其他替換。 這些範例使用我們在範本頂端定義的參數和變數。

| find | 取代為 |
| --- | --- |
| `"hidden-link:/subscriptions/.../components/MyAppName"` |`"[concat('hidden-link:',`<br/>` resourceId('microsoft.insights/components',` <br/> ` parameters('appName')))]"` |
| `"/subscriptions/.../alertrules/myAlertName-myAppName-subsId",` |`"[resourceId('Microsoft.Insights/alertrules', variables('alertRuleName'))]",` |
| `"/subscriptions/.../webtests/myTestName-myAppName",` |`"[resourceId('Microsoft.Insights/webtests', parameters('webTestName'))]",` |
| `"myWebTest-myAppName"` |`"[variables(testName)]"'` |
| `"myTestName-myAppName-subsId"` |`"[variables('alertRuleName')]"` |
| `"myAppName"` |`"[parameters('appName')]"` |
| `"myappname"` (小寫) |`"[toLower(parameters('appName'))]"` |
| `"<WebTest Name=\"myWebTest\" ...`<br/>` Url=\"http://fabrikam.com/home\" ...>"` |`[concat('<WebTest Name=\"',` <br/> `parameters('webTestName'),` <br/> `'\" ... Url=\"', parameters('Url'),` <br/> `'\"...>')]"`<br/>刪除 Guid 和識別碼。 |

### <a name="set-dependencies-between-the-resources"></a>設定資源間的相依性
Azure 應以嚴格的順序設定資源。 為確保一項設定完成後再開始下一項設定，請加入相依性命令行：

* 在可用性測試資源中︰
  
    `"dependsOn": ["[resourceId('Microsoft.Insights/components', parameters('appName'))]"],`
* 可用性測試的警示資源中︰
  
    `"dependsOn": ["[resourceId('Microsoft.Insights/webtests', variables('testName'))]"],`



## <a name="next-steps"></a>後續步驟
其他自動化文件：

* [建立 Application Insights 資源](app-insights-powershell-script-create-resource.md) - 快速方法 (不使用範本)
* [設定警示](app-insights-powershell-alerts.md)
* [建立 Web 測試](https://azure.microsoft.com/blog/creating-a-web-test-alert-programmatically-with-application-insights/)
* [將 Azure 診斷傳送至 Application Insights](app-insights-powershell-azure-diagnostics.md)
* [從 GitHub 部署至 Azure (英文)](http://blogs.msdn.com/b/webdev/archive/2015/09/16/deploy-to-azure-from-github-with-application-insights.aspx)
* [建立版本附註](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/API/CreateReleaseAnnotation.ps1)

