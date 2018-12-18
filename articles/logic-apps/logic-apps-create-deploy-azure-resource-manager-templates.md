---
title: 使用 Azure Resource Manager 範本建立邏輯應用程式 - Azure Logic Apps | Microsoft Docs
description: 在 Azure Logic Apps 中使用 Azure Resource Manager 範本建立和部署邏輯應用程式工作流程
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.assetid: 7574cc7c-e5a1-4b7c-97f6-0cffb1a5d536
ms.date: 10/15/2017
ms.openlocfilehash: 70af92c2afd450d357bf9f30187ef200334698ba
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43124712"
---
# <a name="create-and-deploy-logic-apps-with-azure-resource-manager-templates"></a>使用 Azure Resource Manager 範本建立和部署邏輯應用程式

Azure Logic Apps 提供 Azure Resource Manager 範本，不僅可用來建立自動化工作流程的邏輯應用程式，也可以定義用於部署的資源和參數。 您可以在自己的商務案例中使用此範本，或自訂此範本以符合您的需求。 深入了解[邏輯應用程式的 Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/blob/master/101-logic-app-create/azuredeploy.json)和 [Azure Resource Manager 範本結構和語法](../azure-resource-manager/resource-group-authoring-templates.md)。

## <a name="define-the-logic-app"></a>定義邏輯應用程式

這個範例邏輯應用程式定義每小時執行一次，並 Ping `testUri` 參數中所指定的位置。
這個範本會使用邏輯應用程式名稱 (```logicAppName```) 以及要 Ping 以進行測試之位置 (```testUri```) 的參數值。 深入了解[在範本中定義這些參數](#define-parameters)。 這個範本也會將邏輯應用程式的位置設為與 Azure 資源群組相同的位置。 

``` json
{
   "type": "Microsoft.Logic/workflows",
   "apiVersion": "2016-06-01",
   "name": "[parameters('logicAppName')]",
   "location": "[resourceGroup().location]",
   "tags": {
      "displayName": "LogicApp"
   },
   "properties": {
      "definition": {
         "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "testURI": {
               "type": "string",
               "defaultValue": "[parameters('testUri')]"
            }
         },
         "triggers": {
            "Recurrence": {
               "type": "Recurrence",
               "recurrence": {
                  "frequency": "Hour",
                  "interval": 1
               }
            }
         },
         "actions": {
            "Http": {
              "type": "Http",
              "inputs": {
                  "method": "GET",
                  "uri": "@parameters('testUri')"
              },
              "runAfter": {}
           }
         },
         "outputs": {}
      },
      "parameters": {}
   }
}
``` 

<a name="define-parameters"></a>

### <a name="define-parameters"></a>定義參數

[!INCLUDE [app-service-logic-deploy-parameters](../../includes/app-service-logic-deploy-parameters.md)]

以下是範本中參數的描述：

| 參數 | 說明 | JSON 定義範例 | 
| --------- | ----------- | ----------------------- | 
| `logicAppName` | 定義該範本所建立之邏輯應用程式的名稱。 | "logicAppName": { "type": "string", "metadata": { "description": "myExampleLogicAppName" } } |
| `testUri` | 定義要 Ping 以進行測試的位置。 | "testUri": { "type": "string", "defaultValue": "http://azure.microsoft.com/status/feed/"} | 
||||

深入了解[邏輯應用程式工作流程定義和屬性的 REST API](https://docs.microsoft.com/rest/api/logic/workflows) 和[使用 JSON 根據邏輯應用程式定義建置](logic-apps-author-definitions.md)。

## <a name="deploy-logic-apps-automatically"></a>自動部署邏輯應用程式

若要建立邏輯應用程式，並自動將其部署至 Azure，請在這裡選擇 [部署至 Azure]：

[![部署至 Azure](./media/logic-apps-create-deploy-azure-resource-manager-templates/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-logic-app-create%2Fazuredeploy.json)

此動作會將您登入 Azure 入口網站，您可以在其中提供您邏輯應用程式的詳細資料，並對範本或參數進行任何變更。 例如，Azure 入口網站會提示您輸入這些詳細資料：

* Azure 訂用帳戶名稱
* 您想要使用的資源群組
* 邏輯應用程式位置
* 邏輯應用程式的名稱
* 測試 URI
* 接受指定的條款和條件

## <a name="deploy-logic-apps-with-commands"></a>使用命令部署邏輯應用程式

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### <a name="powershell"></a>PowerShell

```
New-AzureRmResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-logic-app-create/azuredeploy.json -ResourceGroupName ExampleDeployGroup
``` 

### <a name="azure-cli"></a>Azure CLI

```
azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-logic-app-create/azuredeploy.json -g ExampleDeployGroup
```

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [監視 Logic Apps](../logic-apps/logic-apps-monitor-your-logic-apps.md)