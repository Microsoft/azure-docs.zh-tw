---
title: Azure 監視 REST API 逐步解說
description: 如何驗證要求，以及使用 Azure 監視器 REST API 來取出可用的計量定義和計量值。
author: mcollier
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 03/19/2018
ms.author: mcollier
ms.component: ''
ms.openlocfilehash: 9524d471388e69166191b6197fb295532b068092
ms.sourcegitcommit: e3d5de6d784eb6a8268bd6d51f10b265e0619e47
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39390549"
---
# <a name="azure-monitoring-rest-api-walkthrough"></a>Azure 監視 REST API 逐步解說
本文將說明如何執行驗證，讓您的程式碼可以使用 [Microsoft Azure 監視器 REST API 參考](https://msdn.microsoft.com/library/azure/dn931943.aspx)。         

Azure 監視器 API 可讓您以程式設計方式取出可用的預設計量定義、細微性和計量值。 資料可以儲存在另一個資料存放區，例如 Azure SQL Database、Azure Cosmos DB 或 Azure Data Lake。 其他分析可視需要從該處執行。

除了使用各種計量資料點，監視器 API 還可讓您列出警示規則、檢視活動記錄等等。 如需可用作業的完整清單，請參閱 [Microsoft Azure 監視器 REST API 參考](https://msdn.microsoft.com/library/azure/dn931943.aspx)。

## <a name="authenticating-azure-monitor-requests"></a>驗證 Azure 監視器要求
第一步是驗證要求。

針對 Azure 監視器 API 執行的所有工作都會使用 Azure Resource Manager 驗證模型。 因此，所有要求都必須使用 Azure Active Directory (Azure AD) 進行驗證。 驗證用戶端應用程式的其中一個方法是建立 Azure AD 服務主體，並擷取驗證 (JWT) 權杖。 下列範例指令碼會示範透過 PowerShell 建立 Azure AD 服務主體的方法。 如需更詳細的逐步解說中，請參閱 [使用 Azure PowerShell 建立用來存取資源的服務主體](https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps)上的文件。 它也可以[透過 Azure 入口網站建立服務原則](../azure-resource-manager/resource-group-create-service-principal-portal.md)。

```PowerShell
$subscriptionId = "{azure-subscription-id}"
$resourceGroupName = "{resource-group-name}"

# Authenticate to a specific Azure subscription.
Connect-AzureRmAccount -SubscriptionId $subscriptionId

# Password for the service principal
$pwd = "{service-principal-password}"
$secureStringPassword = ConvertTo-SecureString -String $pwd -AsPlainText -Force

# Create a new Azure AD application
$azureAdApplication = New-AzureRmADApplication `
                        -DisplayName "My Azure Monitor" `
                        -HomePage "https://localhost/azure-monitor" `
                        -IdentifierUris "https://localhost/azure-monitor" `
                        -Password $secureStringPassword

# Create a new service principal associated with the designated application
New-AzureRmADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

# Assign Reader role to the newly created service principal
New-AzureRmRoleAssignment -RoleDefinitionName Reader `
                          -ServicePrincipalName $azureAdApplication.ApplicationId.Guid

```

若要查詢 Azure 監視器 API，用戶端應用程式應使用先前建立的服務主體來進行驗證。 下列範例 PowerShell 指令碼示範一種方法：使用 [Active Directory Authentication Library](../active-directory/active-directory-authentication-libraries.md) (ADAL) 取得 JWT 驗證權杖。 JWT 權杖會做為要求中 HTTP 授權參數的一部分傳遞至 Azure 監視器 API。

```PowerShell
$azureAdApplication = Get-AzureRmADApplication -IdentifierUri "https://localhost/azure-monitor"

$subscription = Get-AzureRmSubscription -SubscriptionId $subscriptionId

$clientId = $azureAdApplication.ApplicationId.Guid
$tenantId = $subscription.TenantId
$authUrl = "https://login.microsoftonline.com/${tenantId}"

$AuthContext = [Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext]$authUrl
$cred = New-Object -TypeName Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential -ArgumentList ($clientId, $secureStringPassword)

$result = $AuthContext.AcquireToken("https://management.core.windows.net/", $cred)

# Build an array of HTTP header values
$authHeader = @{
'Content-Type'='application/json'
'Accept'='application/json'
'Authorization'=$result.CreateAuthorizationHeader()
}
```

驗證之後，即可針對 Azure 監視器 REST API 執行查詢。 有兩個實用的查詢︰

1. 列出資源的度量定義
2. 擷取度量值

## <a name="retrieve-metric-definitions-multi-dimensional-api"></a>取出計量定義 (多維度 API)

使用 [Azure 監視器度量定義 REST API](https://docs.microsoft.com/rest/api/monitor/metricdefinitions) 來存取可供服務使用的度量清單。

**方法**：GET

**要求 URI**：https://management.azure.com/subscriptions/*{subscriptionId}*/resourceGroups/*{resourceGroupName}*/providers/*{resourceProviderNamespace}*/*{resourceType}*/*{resourceName}*/providers/microsoft.insights/metricDefinitions?api-version=*{apiVersion}*

例如，若要取出 Azure 儲存體帳戶的計量定義，則要求看起來如下所示：

```PowerShell
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricDefinitions?api-version=2018-01-01"

Invoke-RestMethod -Uri $request `
                  -Headers $authHeader `
                  -Method Get `
                  -OutFile ".\contosostorage-metricdef-results.json" `
                  -Verbose

```
> [!NOTE]
> 若要使用多維度 Azure 監視器計量 REST API 取出計量定義，請使用 "2018-01-01"作為 API 版本。
>
>

產生的 JSON 回應本文看起來會像下列範例：(請注意，第二個計量具有維度)

```JSON
{
    "value": [
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricdefinitions/UsedCapacity",
            "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage",
            "namespace": "Microsoft.Storage/storageAccounts",
            "category": "Capacity",
            "name": {
                "value": "UsedCapacity",
                "localizedValue": "Used capacity"
            },
            "isDimensionRequired": false,
            "unit": "Bytes",
            "primaryAggregationType": "Average",
            "supportedAggregationTypes": [
                "Total",
                "Average",
                "Minimum",
                "Maximum"
            ],
            "metricAvailabilities": [
                {
                    "timeGrain": "PT1H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT6H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT12H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "P1D",
                    "retention": "P93D"
                }
            ]
        },
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricdefinitions/Transactions",
            "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage",
            "namespace": "Microsoft.Storage/storageAccounts",
            "category": "Transaction",
            "name": {
                "value": "Transactions",
                "localizedValue": "Transactions"
            },
            "isDimensionRequired": false,
            "unit": "Count",
            "primaryAggregationType": "Total",
            "supportedAggregationTypes": [
                "Total"
            ],
            "metricAvailabilities": [
                {
                    "timeGrain": "PT1M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT5M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT15M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT30M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT1H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT6H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT12H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "P1D",
                    "retention": "P93D"
                }
            ],
            "dimensions": [
                {
                    "value": "ResponseType",
                    "localizedValue": "Response type"
                },
                {
                    "value": "GeoType",
                    "localizedValue": "Geo type"
                },
                {
                    "value": "ApiName",
                    "localizedValue": "API name"
                }
            ]
        },
        ...
    ]
}
```

## <a name="retrieve-dimension-values-multi-dimensional-api"></a>取出維度值 (多維度 API)
知道可用的計量定義後，有一些計量可能是有維度的。 查詢計量之前，建議您先探索維度的值範圍。 在查詢計量時，您就可以選擇根據這些維度值來篩選或分類計量。  使用 [Azure 監視器度量 REST API](https://docs.microsoft.com/rest/api/monitor/metrics) 來完成此作業。

在任何篩選要求中，請使用計量的名稱 ‘value’ (不是 ‘localizedValue’)。 如果未指定篩選器，則會傳回預設度量。 此 API 的使用方式只允許一個維度有萬用字元篩選條件。

> [!NOTE]
> 若要使用 Azure 監視器 REST API 擷取維度值，請使用 "2018-01-01" 作為 API 版本。
>
>

**方法**：GET

**要求 URI**：https://management.azure.com/subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/*{resource-provider-namespace}*/*{resource-type}*/*{resource-name}*/providers/microsoft.insights/metrics?metricnames=*{metric}*&timespan=*{starttime/endtime}*&$filter=*{filter}*&resultType=metadata&api-version=*{apiVersion}*

例如，若要擷取 'Transactions' 計量之 'API Name dimension' 中發出的維度值清單，其中指定時間範圍內的 GeoType 維度 = 'Primary'，要求將如下所示：

```PowerShell
$filter = "APIName eq '*' and GeoType eq 'Primary'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metrics?metricnames=Transactions&timespan=2018-03-01T00:00:00Z/2018-03-02T00:00:00Z&resultType=metadata&`$filter=${filter}&api-version=2018-01-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosostorage-dimension-values.json" `
    -Verbose
```
產生的 JSON 回應本文看起來會像下列範例：

```JSON
{
  "timespan": "2018-03-01T00:00:00Z/2018-03-02T00:00:00Z",
  "value": [
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/Microsoft.Insights/metrics/Transactions",
      "type": "Microsoft.Insights/metrics",
      "name": {
        "value": "Transactions",
        "localizedValue": "Transactions"
      },
      "unit": "Count",
      "timeseries": [
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "DeleteBlob"
            }
          ]
        },
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "SetBlobProperties"
            }
          ]
        },
        ...
      ]    
    }
  ],
  "namespace": "Microsoft.Storage/storageAccounts",
  "resourceregion": "eastus"
}
```

## <a name="retrieve-metric-values-multi-dimensional-api"></a>取出計量值 (多維度 API)
知道可用的計量定義和可能的維度值之後，就可以取出相關的計量值。  使用 [Azure 監視器度量 REST API](https://docs.microsoft.com/rest/api/monitor/metrics) 來完成此作業。

在任何篩選要求中，請使用計量的名稱 ‘value’ (不是 ‘localizedValue’)。 如果未指定維度篩選條件，則會傳回積存的彙總計量。 如果計量查詢傳回多個時間序列，您可以使用 'Top' 和 'OrderBy' 查詢參數傳回時間序列的有限排序清單。

> [!NOTE]
> 若要使用 Azure 監視器 REST API 擷取多維度計量值，請使用 "2018-01-01" 作為 API 版本。
>
>

**方法**：GET

**要求 URI**：https://management.azure.com/subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/*{resource-provider-namespace}*/*{resource-type}*/*{resource-name}*/providers/microsoft.insights/metrics?metricnames=*{metric}*&timespan=*{starttime/endtime}*&$filter=*{filter}*&interval=*{timeGrain}*&aggregation=*{aggreation}*&api-version=*{apiVersion}*

例如，若要根據 5 分鐘範圍內的 'Transactions' 數量 (其中 GeotType 是 'Primary')，擷取排名前 3 個 API (依遞減值排列)，要求如下所示：

```PowerShell
$filter = "APIName eq '*' and GeoType eq 'Primary'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metrics?metricnames=Transactions&timespan=2018-03-01T02:00:00Z/2018-03-01T02:05:00Z&`$filter=${filter}&interval=PT1M&aggregation=Total&top=3&orderby=Total desc&api-version=2018-01-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosostorage-metric-values.json" `
    -Verbose
```
產生的 JSON 回應本文看起來會像下列範例：

```JSON
{
  "cost": 0,
  "timespan": "2018-03-01T02:00:00Z/2018-03-01T02:05:00Z",
  "interval": "PT1M",
  "value": [
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/Microsoft.Insights/metrics/Transactions",
      "type": "Microsoft.Insights/metrics",
      "name": {
        "value": "Transactions",
        "localizedValue": "Transactions"
      },
      "unit": "Count",
      "timeseries": [
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "GetBlobProperties"
            }
          ],
          "data": [
            {
              "timeStamp": "2017-09-19T02:00:00Z",
              "total": 2
            },
            {
              "timeStamp": "2017-09-19T02:01:00Z",
              "total": 1
            },
            {
              "timeStamp": "2017-09-19T02:02:00Z",
              "total": 3
            },
            {
              "timeStamp": "2017-09-19T02:03:00Z",
              "total": 7
            },
            {
              "timeStamp": "2017-09-19T02:04:00Z",
              "total": 2
            }
          ]
        },
        ...
      ]
    }
  ],
  "namespace": "Microsoft.Storage/storageAccounts",
  "resourceregion": "eastus"
}
```

## <a name="retrieve-metric-definitions"></a>取出計量定義
使用 [Azure 監視器度量定義 REST API](https://msdn.microsoft.com/library/mt743621.aspx) 來存取可供服務使用的度量清單。

**方法**：GET

**要求 URI**：https://management.azure.com/subscriptions/*{subscriptionId}*/resourceGroups/*{resourceGroupName}*/providers/*{resourceProviderNamespace}*/*{resourceType}*/*{resourceName}*/providers/microsoft.insights/metricDefinitions?api-version=*{apiVersion}*

例如，若要取出 Azure Logic App 的計量定義，則要求看起來如下所示：

```PowerShell
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricDefinitions?api-version=2016-03-01"

Invoke-RestMethod -Uri $request `
                  -Headers $authHeader `
                  -Method Get `
                  -OutFile ".\contostweets-metricdef-results.json" `
                  -Verbose
```
> [!NOTE]
> 若要使用 Azure 監視器 REST API 擷取度量定義，請使用 "2016-03-01" 做為 API 版本。
>
>

產生的 JSON 回應本文看起來會像下列範例：
```JSON
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricdefinitions",
  "value": [
    {
      "name": {
        "value": "RunsStarted",
        "localizedValue": "Runs Started"
      },
      "category": "AllMetrics",
      "startTime": "0001-01-01T00:00:00Z",
      "endTime": "0001-01-01T00:00:00Z",
      "unit": "Count",
      "primaryAggregationType": "Total",
      "resourceUri": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets",
      "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets",
      "metricAvailabilities": [
        {
          "timeGrain": "PT1M",
          "retention": "P30D",
          "location": null,
          "blobLocation": null
        },
        {
          "timeGrain": "PT1H",
          "retention": "P30D",
          "location": null,
          "blobLocation": null
        }
      ],
      "properties": null,
      "dimensions": null,
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricdefinitions/RunsStarted",
      "supportedAggregationTypes": [ "None", "Average", "Minimum", "Maximum", "Total", "Count" ]
    }
  ]
}
```

如需詳細資訊，請參閱 [列出 Azure 監視器 REST API 中的資源度量定義](https://msdn.microsoft.com/library/azure/mt743621.aspx) 文件。

## <a name="retrieve-metric-values"></a>取出計量值
一旦已知可用的度量定義後，便可擷取相關的度量值。 將度量名稱 ‘value’ (而非 ‘localizedValue’) 用於任何篩選要求 (例如，擷取 ‘CpuTime’ 和 ‘Requests’ 度量資料點)。 如果未指定篩選器，則會傳回預設度量。

> [!NOTE]
> 若要使用 Azure 監視器 REST API 取出計量值，請使用 "2016-09-01" 作為 API 版本。
>
>

**方法**：GET

**要求 URI**：https://management.azure.com/subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/*{resource-provider-namespace}*/*{resource-type}*/*{resource-name}*/providers/microsoft.insights/metrics?$filter=*{filter}*&api-version=*{apiVersion}*

例如，若要擷取給定時間範圍及時間粒紋為 1 小時的 RunsSucceeded 度量資料點，要求如下所示︰

```PowerShell
$filter = "(name.value eq 'RunsSucceeded') and aggregationType eq 'Total' and startTime eq 2017-08-18T19:00:00 and endTime eq 2017-08-18T23:00:00 and timeGrain eq duration'PT1H'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metrics?`$filter=${filter}&api-version=2016-09-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contostweets-metrics-results.json" `
    -Verbose
```

產生的 JSON 回應本文看起來會像下列範例：

```JSON
{
  "value": [
    {
      "data": [
        {
          "timeStamp": "2017-08-18T19:00:00Z",
          "total": 0.0
        },
        {
          "timeStamp": "2017-08-18T20:00:00Z",
          "total": 159.0
        },
        {
          "timeStamp": "2017-08-18T21:00:00Z",
          "total": 174.0
        },
        {
          "timeStamp": "2017-08-18T22:00:00Z",
          "total": 97.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/RunsSucceeded",
      "name": {
        "value": "RunsSucceeded",
        "localizedValue": "Runs Succeeded"
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    }
  ]
}
```

若要擷取多個資料或彙總點，將度量定義名稱和彙總類型新增至篩選器，如下列範例所示︰

```PowerShell
$filter = "(name.value eq 'ActionsCompleted' or name.value eq 'RunsSucceeded') and (aggregationType eq 'Total' or aggregationType eq 'Average') and startTime eq 2017-08-18T21:00:00 and endTime eq 2017-08-18T21:30:00 and timeGrain eq duration'PT1M'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metrics?`$filter=${filter}&api-version=2016-09-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contostweets-metrics-multiple-results.json" `
    -Verbose
```
產生的 JSON 回應本文看起來會像下列範例：

```JSON
{
  "value": [
    {
      "data": [
        {
          "timeStamp": "2017-08-18T21:03:00Z",
          "total": 5.0,
          "average": 1.0
        },
        {
          "timeStamp": "2017-08-18T21:04:00Z",
          "total": 7.0,
          "average": 1.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/ActionsCompleted",
      "name": {
        "value": "ActionsCompleted",
        "localizedValue": "Actions Completed "
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    },
    {
      "data": [
        {
          "timeStamp": "2017-08-18T21:03:00Z",
          "total": 5.0,
          "average": 1.0
        },
        {
          "timeStamp": "2017-08-18T21:04:00Z",
          "total": 7.0,
          "average": 1.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/RunsSucceeded",
      "name": {
        "value": "RunsSucceeded",
        "localizedValue": "Runs Succeeded"
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    }
  ]
}
```

### <a name="use-armclient"></a>使用 ARMClient
另一種方法是在 Windows 機器上使用 [ARMClient](https://github.com/projectkudu/armclient)。 ARMClient 會自動處理 Azure AD 驗證 (以及產生的 JWT 權杖)。 下列步驟概述使用 ARMClient 來取出計量資料︰

1. 安裝 [Chocolatey](https://chocolatey.org/) 和 [ARMClient](https://github.com/projectkudu/armclient)。
2. 在終端機視窗中，輸入 armclient.exe login 。 這樣做會提示您登入 Azure。
3. 輸入 *armclient GET [your_resource_id]/providers/microsoft.insights/metricdefinitions?api-version=2016-03-01*
4. 輸入 *armclient GET [your_resource_id]/providers/microsoft.insights/metrics?api-version=2016-09-01*

例如，若要取出特定邏輯應用程式的計量定義，請發出下列命令：
```
armclient GET /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricDefinitions?api-version=2016-03-01
```


## <a name="retrieve-the-resource-id"></a>取出資源識別碼
使用 REST API 可實際協助了解可用的度量定義、細微度以及相關值。 使用 [Azure Management Library](https://msdn.microsoft.com/library/azure/mt417623.aspx)時，該資訊會很實用。

上述程式碼中，要使用的資源識別碼是所需 Azure 資源的完整路徑。 例如，若要查詢 Azure Web 應用程式，資源識別碼為︰

<bpt id="p1">*</bpt>/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{site-name}/<ept id="p1">*</ept>

下列範例清單包含各種 Azure 資源的資源識別碼格式︰

* **IoT 中樞** - /subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/Microsoft.Devices/IotHubs/*{iot-hub-name}*
* **SQL 彈性集區** - /subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/Microsoft.Sql/servers/*{pool-db}*/elasticpools/*{sql-pool-name}*
* **SQL Database (v12)** - /subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/Microsoft.Sql/servers/*{server-name}*/databases/*{database-name}*
* **服務匯流排** - /subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/Microsoft.ServiceBus/*{namespace}*/*{servicebus-name}*
* **虛擬機器擴展集** - /subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/Microsoft.Compute/virtualMachineScaleSets/*{vm-name}*
* **VM** - /subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/Microsoft.Compute/virtualMachines/*{vm-name}*
* **事件中樞** - /subscriptions/*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/Microsoft.EventHub/namespaces/*{eventhub-namespace}*

還有其他方法可擷取資源識別碼，包括使用 Azure 資源總管、在 Azure 入口網站中以及透過 PowerShell 或 Azure CLI 檢視所需的資源。

### <a name="azure-resource-explorer"></a>Azure 資源總管
若要尋找所需資源的資源識別碼，其中一個實用的方法是使用 [Azure 資源總管](https://resources.azure.com) 工具。 瀏覽至所需的資源，然後查看如下列螢幕擷取畫面所示的識別碼︰

![替代「Azure 資源總管」](./media/monitoring-rest-api-walkthrough/azure_resource_explorer.png)

### <a name="azure-portal"></a>Azure 入口網站
也可以從 Azure 入口網站取得資源識別碼。 若要這樣做，請瀏覽至所需的資源，然後選取 [內容]。 「資源識別碼」會顯示在 [屬性] 區段中，如下列螢幕擷取畫面所示︰

![替代「Azure 入口網站的 [屬性] 刀鋒視窗中顯示的資源識別碼」](./media/monitoring-rest-api-walkthrough/resourceid_azure_portal.png)

### <a name="azure-powershell"></a>Azure PowerShell
可以使用 Azure PowerShell Cmdlet 來擷取資源識別碼。 例如，若要取得 Azure Logic App 的資源識別碼，請執行 Get-AzureLogicApp Cmdlet，如下列範例所示︰

```PowerShell
Get-AzureRmLogicApp -ResourceGroupName azmon-rest-api-walkthrough -Name contosotweets
```

結果應該類似於下列範例：
```
Id             : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets
Name           : ContosoTweets
Type           : Microsoft.Logic/workflows
Location       : centralus
ChangedTime    : 8/21/2017 6:58:57 PM
CreatedTime    : 8/18/2017 7:54:21 PM
AccessEndpoint : https://prod-08.centralus.logic.azure.com:443/workflows/f3a91b352fcc47e6bff989b85446c5db
State          : Enabled
Definition     : {$schema, contentVersion, parameters, triggers...}
Parameters     : {[$connections, Microsoft.Azure.Management.Logic.Models.WorkflowParameter]}
SkuName        :
AppServicePlan :
PlanType       :
PlanId         :
Version        : 08586982649483762729
```


### <a name="azure-cli"></a>Azure CLI
若要使用 Azure CLI 取出 Azure 儲存體帳戶的資源識別碼，請執行 'az storage account show' 命令，如下列範例所示：

```
az storage account show -g azmon-rest-api-walkthrough -n contosotweets2017
```

結果應該類似於下列範例：
```JSON
{
  "accessTier": null,
  "creationTime": "2017-08-18T19:58:41.840552+00:00",
  "customDomain": null,
  "enableHttpsTrafficOnly": false,
  "encryption": null,
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/contosotweets2017",
  "identity": null,
  "kind": "Storage",
  "lastGeoFailoverTime": null,
  "location": "centralus",
  "name": "contosotweets2017",
  "networkAcls": null,
  "primaryEndpoints": {
    "blob": "https://contosotweets2017.blob.core.windows.net/",
    "file": "https://contosotweets2017.file.core.windows.net/",
    "queue": "https://contosotweets2017.queue.core.windows.net/",
    "table": "https://contosotweets2017.table.core.windows.net/"
  },
  "primaryLocation": "centralus",
  "provisioningState": "Succeeded",
  "resourceGroup": "azmon-rest-api-walkthrough",
  "secondaryEndpoints": null,
  "secondaryLocation": "eastus2",
  "sku": {
    "name": "Standard_GRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": "available",
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}
```

> [!NOTE]
> 由於還無法透過 Azure CLI 使用 Azure Logic Apps，因此，上述範例中顯示 Azure 儲存體帳戶。
>
>

## <a name="retrieve-activity-log-data"></a>取出活動記錄資料
除了計量定義及相關值，也可以使用 Azure Monitor REST API 取出關於 Azure 資源的其他有用的深入解析。 例如，可以查詢 [活動記錄檔](https://msdn.microsoft.com/library/azure/dn931934.aspx) 資料。 下列範例示範使用 Azure 監視器 REST API 查詢 Azure 訂用帳戶特定日期範圍內的活動記錄檔資料︰

```PowerShell
$apiVersion = "2015-04-01"
$filter = "eventTimestamp ge '2017-08-18' and eventTimestamp le '2017-08-19'and eventChannels eq 'Admin, Operation'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/microsoft.insights/eventtypes/management/values?api-version=${apiVersion}&`$filter=${filter}"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -Verbose
```

## <a name="next-steps"></a>後續步驟
* 檢閱 [監視的概觀](monitoring-overview.md)。
* 檢視 [支援 Azure 監視器的度量](monitoring-supported-metrics.md)。
* 檢閱 [Microsoft Azure 監視器 REST API 參考](https://msdn.microsoft.com/library/azure/dn931943.aspx)。
* 檢閱 [Azure Management Library](https://msdn.microsoft.com/library/azure/mt417623.aspx)。
