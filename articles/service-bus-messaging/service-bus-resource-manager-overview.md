---
title: 使用 Resource Manager 範本建立 Azure 服務匯流排資源 | Microsoft Docs
description: 使用 Azure Resource Manager 範本自動建立服務匯流排資源
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: 24f6a207-0fa4-49cf-8a58-963f9e2fd655
ms.service: service-bus-messaging
ms.devlang: tbd
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 09/11/2018
ms.author: spelluru
ms.openlocfilehash: 5693307f9d032ad544a44eabd19126700c5a4099
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407156"
---
# <a name="create-service-bus-resources-using-azure-resource-manager-templates"></a>使用 Azure Resource Manager 範本建立服務匯流排資源

本文說明如何使用 Azure Resource Manager 範本、PowerShell 和服務匯流排資源提供者來建立和部署服務匯流排資源。

Azure Resource Manager 範本會協助您定義要部署給解決方案的資源，以及指定參數和變數，讓您可以針對不同的環境來輸入值。 範本以 JSON 編寫，由用於為部署建構值的運算式組成。 如需撰寫 Azure Resource Manager 範本和討論範本格式的詳細資訊，請參閱 [Azure Resource Manager 範本的結構和語法](../azure-resource-manager/resource-group-authoring-templates.md)。

> [!NOTE]
> 本文中的範例會示範如何使用 Azure Resource Manager 來建立服務匯流排命名空間和訊息實體 (佇列)。 如需其他範本範例，請造訪 [Azure 快速入門範本資源庫][Azure Quickstart Templates gallery]並搜尋**服務匯流排**。
>
>

## <a name="service-bus-resource-manager-templates"></a>服務匯流排 Resource Manager 範本

這些服務匯流排 Azure Resource Manager 範本可供下載和部署。 按一下下列連結可取得各自的詳細資料，並附有 GitHub 上的範本連結：

* [建立服務匯流排命名空間](service-bus-resource-manager-namespace.md)
* [建立服務匯流排命名空間與佇列](service-bus-resource-manager-namespace-queue.md)
* [建立服務匯流排命名空間與主題和訂用帳戶](service-bus-resource-manager-namespace-topic.md)
* [建立服務匯流排命名空間與佇列和授權規則](service-bus-resource-manager-namespace-auth-rule.md)
* [建立服務匯流排命名空間與主題、訂用帳戶和規則](service-bus-resource-manager-namespace-topic-with-rule.md)

## <a name="deploy-with-powershell"></a>使用 PowerShell 部署

下列程序描述如何使用 PowerShell 部署建立了標準層服務匯流排命名空間的 Azure Resource Manager 範本，以及如何在該命名空間內部署佇列。 這個範例是以[建立有佇列的服務匯流排命名空間](https://github.com/Azure/azure-quickstart-templates/tree/master/201-servicebus-create-queue)範本為基礎。 近似的工作流程如下︰

1. 安裝 PowerShell。
2. 建立範本和 (選擇性) 參數檔案。
3. 在 PowerShell 中登入您的 Azure 帳戶。
4. 如果沒有資源群組，請建立一個新的。
5. 測試部署。
6. 如有需要，請設定部署模式。
7. 部署範本。

如需部署 Azure Resource Manager 範本的完整資訊，請參閱[使用 Azure Resource Manager 範本部署資源][Deploy resources with Azure Resource Manager templates]。

### <a name="install-powershell"></a>安裝 PowerShell

依照下列指示安裝 Azure PowerShell：[開始使用 Azure PowerShell](/powershell/azure/get-started-azureps)。

### <a name="create-a-template"></a>建立範本

從 GitHub 複製儲存機制或複製 [201-servicebus-create-queue](https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-queue/azuredeploy.json) 範本：

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "serviceBusNamespaceName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Service Bus namespace"
      }
    },
    "serviceBusQueueName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Queue"
      }
    }
  },
  "variables": {
    "defaultSASKeyName": "RootManageSharedAccessKey",
    "authRuleResourceId": "[resourceId('Microsoft.ServiceBus/namespaces/authorizationRules', parameters('serviceBusNamespaceName'), variables('defaultSASKeyName'))]",
    "sbVersion": "2017-04-01"
  },
  "resources": [
    {
      "apiVersion": "2017-04-01",
      "name": "[parameters('serviceBusNamespaceName')]",
      "type": "Microsoft.ServiceBus/Namespaces",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {},
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('serviceBusQueueName')]",
          "type": "Queues",
          "dependsOn": [
            "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
          ],
          "properties": {
            "lockDuration": "PT5M",
            "maxSizeInMegabytes": "1024",
            "requiresDuplicateDetection": "false",
            "requiresSession": "false",
            "defaultMessageTimeToLive": "P10675199DT2H48M5.4775807S",
            "deadLetteringOnMessageExpiration": "false",
            "duplicateDetectionHistoryTimeWindow": "PT10M",
            "maxDeliveryCount": "10",
            "autoDeleteOnIdle": "P10675199DT2H48M5.4775807S",
            "enablePartitioning": "false",
            "enableExpress": "false"
          }
        }
      ]
    }
  ],
  "outputs": {
    "NamespaceConnectionString": {
      "type": "string",
      "value": "[listkeys(variables('authRuleResourceId'), variables('sbVersion')).primaryConnectionString]"
    },
    "SharedAccessPolicyPrimaryKey": {
      "type": "string",
      "value": "[listkeys(variables('authRuleResourceId'), variables('sbVersion')).primaryKey]"
    }
  }
}
```

### <a name="create-a-parameters-file-optional"></a>建立參數檔案 (選擇性)

若要使用選擇性的參數檔案，請複製 [201-servicebus-create-queue](https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-queue/azuredeploy.parameters.json) 檔案。 以您要在這個部署中建立的服務匯流排命名空間名稱取代 `serviceBusNamespaceName` 值，並以您要建立的佇列名稱取代 `serviceBusQueueName` 值。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "serviceBusNamespaceName": {
            "value": "<myNamespaceName>"
        },
        "serviceBusQueueName": {
            "value": "<myQueueName>"
        },
        "serviceBusApiVersion": {
            "value": "2017-04-01"
        }
    }
}
```

如需詳細資訊，請參閱[參數](../azure-resource-manager/resource-group-template-deploy.md#parameter-files)一文。

### <a name="log-in-to-azure-and-set-the-azure-subscription"></a>登入 Azure 並設定 Azure 訂用帳戶

從 PowerShell 提示字元中執行下列命令：

```powershell
Connect-AzureRmAccount
```

系統會提示您登入您的 Azure 帳戶。 登入之後，執行下列命令以檢視可用的訂用帳戶：

```powershell
Get-AzureRMSubscription
```

這個命令會傳回可用的 Azure 訂用帳戶清單。 執行下列命令為目前的工作階段選擇訂用帳戶。 以您要使用的 Azure 訂用帳戶 GUID 取代 `<YourSubscriptionId>`：

```powershell
Set-AzureRmContext -SubscriptionID <YourSubscriptionId>
```

### <a name="set-the-resource-group"></a>設定資源群組

如果沒有現成的資源群組，請使用 **New-AzureRmResourceGroup ** 命令建立新的資源群組。 提供您要使用的資源群組名稱和位置。 例如︰

```powershell
New-AzureRmResourceGroup -Name MyDemoRG -Location "West US"
```

如果成功，就會顯示新資源群組的摘要。

```powershell
ResourceGroupName : MyDemoRG
Location          : westus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/<GUID>/resourceGroups/MyDemoRG
```

### <a name="test-the-deployment"></a>測試部署

執行 `Test-AzureRmResourceGroupDeployment` Cmdlet 驗證部署。 測試部署時，請提供與執行部署時完全一致的參數。

```powershell
Test-AzureRmResourceGroupDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

### <a name="create-the-deployment"></a>建立部署

若要建立新的部署，請執行 `New-AzureRmResourceGroupDeployment` Cmdlet，並於提示出現時提供必要的參數。 參數會包含部署的名稱、資源群組的名稱，以及範本檔案的路徑或 URL。 如未指定 **Mode** 參數，即會使用預設值 **Incremental**。 如需詳細資訊，請參閱[累加部署與完整部署](../azure-resource-manager/deployment-modes.md)。

下列命令會提示您在 PowerShell 視窗中輸入三個參數︰

```powershell
New-AzureRmResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

若要改為指定參數檔案，使用下列命令：

```powershell
New-AzureRmResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
```

執行部署 Cmdlet 時，您也可以使用內嵌參數。 命令如下所示︰

```powershell
New-AzureRmResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -parameterName "parameterValue"
```

若要執行[完整](../azure-resource-manager/deployment-modes.md)部署，請將 **Mode** 參數設為 **Complete**：

```powershell
New-AzureRmResourceGroupDeployment -Name MyDemoDeployment -Mode Complete -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

### <a name="verify-the-deployment"></a>驗證部署
如果資源成功部署，PowerShell 視窗中就會顯示部署的摘要︰

```powershell
DeploymentName    : MyDemoDeployment
ResourceGroupName : MyDemoRG
ProvisioningState : Succeeded
Timestamp         : 4/19/2017 10:38:30 PM
Mode              : Incremental
TemplateLink      :
Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    serviceBusNamespaceName  String             <namespaceName>
                    serviceBusQueueName  String                 <queueName>
                    serviceBusApiVersion  String                2017-04-01

```

## <a name="next-steps"></a>後續步驟
您現在已了解部署 Azure Resource Manager 範本的基本工作流程和命令。 如需更多的詳細資訊，請瀏覽下列連結內容：

* [Azure Resource Manager 概觀][Azure Resource Manager overview]
* [使用 Resource Manager 範本與 Azure PowerShell 來部署資源][Deploy resources with Azure Resource Manager templates]
* [編寫 Azure 資源管理員範本](../azure-resource-manager/resource-group-authoring-templates.md)

[Azure Resource Manager overview]: ../azure-resource-manager/resource-group-overview.md
[Deploy resources with Azure Resource Manager templates]: ../azure-resource-manager/resource-group-template-deploy.md
[Azure Quickstart Templates gallery]: https://azure.microsoft.com/documentation/templates/?term=service+bus
