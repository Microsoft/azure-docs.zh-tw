---
title: 使用 Azure Resource Manager 範本建立 Azure 服務匯流排主題、訂用帳戶和規則 | Microsoft Docs
description: 使用 Azure Resource Manager 範本建立服務匯流排命名空間與主題、訂用帳戶和規則
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: 9e0aaf58-0214-4bca-bd00-d29c08f9b1bc
ms.service: service-bus-messaging
ms.devlang: tbd
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 09/11/2018
ms.author: spelluru
ms.openlocfilehash: 7774d67d02b2b0f0080cbff2ffc3606b99450f26
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47404956"
---
# <a name="create-a-service-bus-namespace-with-topic-subscription-and-rule-using-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本建立服務匯流排命名空間與主題、訂用帳戶和規則

此文章說明如何使用 Azure Resource Manager 範本，建立服務匯流排命名空間與主題、訂用帳戶和規則 (篩選器)。 此文章說明如何指定要部署哪些資源，以及如何定義執行部署時所指定的參數。 您可以直接在自己的部署中使用此範本，或自訂此範本以符合您的需求

如需關於建立範本的詳細資訊，請參閱[編寫 Azure Resource Manager 範本][Authoring Azure Resource Manager templates]。

如需 Azure 資源命名慣例相關實務和模式的詳細資訊，請參閱 [Azure 資源的建議命名慣例][Recommended naming conventions for Azure resources]。

如需完整的範本，請參閱[服務匯流排命名空間與主題、訂用帳戶和規則][Service Bus namespace with topic, subscription, and rule]範本。

> [!NOTE]
> 下列 Azure Resource Manager 範本可供下載和部署。
> 
> * [建立服務匯流排命名空間與佇列和授權規則](service-bus-resource-manager-namespace-auth-rule.md)
> * [建立服務匯流排命名空間與佇列](service-bus-resource-manager-namespace-queue.md)
> * [建立服務匯流排命名空間](service-bus-resource-manager-namespace.md)
> * [建立服務匯流排命名空間與主題和訂用帳戶](service-bus-resource-manager-namespace-topic.md)
> 
> 若要檢查最新的範本，請造訪 [Azure 快速入門範本][Azure Quickstart Templates]資源庫並搜尋「服務匯流排」。
> 
> 

## <a name="what-do-you-deploy"></a>您要部署什麼？

使用此範本，您將部署具有主題、訂用帳戶和規則 (篩選器) 的服務匯流排命名空間。

[服務匯流排主題和訂用帳戶](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions)使用「發佈/訂閱」模式，提供一對多的通訊形式。 使用主題和訂用帳戶時，分散式應用程式的元件彼此不直接通訊，相反的，它們會透過扮演中繼角色的主題來交換訊息。主題的訂用帳戶類似於虛擬佇列，同樣可接收已傳送到主題的訊息複本。 訂用帳戶上的篩選器可讓您指定傳送至主題的哪些訊息應出現在特定主題訂用帳戶中。

## <a name="what-are-rules-filters"></a>什麼是規則 (篩選器)？

在許多情況下，必須以不同的方式處理具有特定特性的訊息。 若要這樣自訂處理，您可以設定訂用帳戶以尋找具有特定屬性的訊息，然後對這些屬性進行修改。 雖然服務匯流排訂用帳戶可看見所有傳送至主題的訊息，但您只可以將部分的訊息複製到虛擬訂用帳戶佇列。 使用訂用帳戶篩選器即可達成。 若要深入了解規則 (篩選條件)，請參閱 [規則和動作](service-bus-queues-topics-subscriptions.md#rules-and-actions)。

若要自動執行部署，請按一下下列按鈕：

[![部署至 Azure](./media/service-bus-resource-manager-namespace-topic/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-servicebus-create-topic-subscription-rule%2Fazuredeploy.json)

## <a name="parameters"></a>參數

透過 Azure Resource Manager，定義在部署範本時想要指定之值的參數。 此範本有一個 `Parameters` 區段，內含所有參數值。 針對隨您要部署的專案或要部署到的環境而變化的值定義參數。 請不要為永遠保持不變的值定義參數。 每個參數值都可在範本中用來定義所部署的資源。

範本會定義下列參數：

### <a name="servicebusnamespacename"></a>serviceBusNamespaceName
要建立的服務匯流排命名空間名稱。

```json
"serviceBusNamespaceName": {
"type": "string"
}
```

### <a name="servicebustopicname"></a>serviceBusTopicName
在服務匯流排命名空間中建立的主題名稱。

```json
"serviceBusTopicName": {
"type": "string"
}
```

### <a name="servicebussubscriptionname"></a>serviceBusSubscriptionName
在服務匯流排命名空間中建立的訂用帳戶名稱。

```json
"serviceBusSubscriptionName": {
"type": "string"
}
```
### <a name="servicebusrulename"></a>serviceBusRuleName
在服務匯流排命名空間中建立的規則 (篩選器) 名稱。

```json
   "serviceBusRuleName": {
   "type": "string",
  }
```
### <a name="servicebusapiversion"></a>serviceBusApiVersion
範本的服務匯流排 API 版本。

```json
"serviceBusApiVersion": { 
       "type": "string", 
       "defaultValue": "2017-04-01", 
       "metadata": { 
           "description": "Service Bus ApiVersion used by the template" 
       }
```
## <a name="resources-to-deploy"></a>要部署的資源
建立**訊息**類型的標準服務匯流排命名空間與主題、訂用帳戶和規則。

```json
 "resources": [{
        "apiVersion": "[variables('sbVersion')]",
        "name": "[parameters('serviceBusNamespaceName')]",
        "type": "Microsoft.ServiceBus/Namespaces",
        "location": "[variables('location')]",
        "sku": {
            "name": "Standard",
        },
        "resources": [{
            "apiVersion": "[variables('sbVersion')]",
            "name": "[parameters('serviceBusTopicName')]",
            "type": "Topics",
            "dependsOn": [
                "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
            ],
            "properties": {
                "path": "[parameters('serviceBusTopicName')]"
            },
            "resources": [{
                "apiVersion": "[variables('sbVersion')]",
                "name": "[parameters('serviceBusSubscriptionName')]",
                "type": "Subscriptions",
                "dependsOn": [
                    "[parameters('serviceBusTopicName')]"
                ],
                "properties": {},
                "resources": [{
                    "apiVersion": "[variables('sbVersion')]",
                    "name": "[parameters('serviceBusRuleName')]",
                    "type": "Rules",
                    "dependsOn": [
                        "[parameters('serviceBusSubscriptionName')]"
                    ],
                    "properties": {
                        "filterType": "SqlFilter",
                        "sqlFilter": {
                            "sqlExpression": "StoreName = 'Store1'",
                            "requiresPreprocessing": "false"
                        },
                        "action": {
                            "sqlExpression": "set FilterTag = 'true'"
                        }
                    }
                }]
            }]
        }]
    }]
```

## <a name="commands-to-run-deployment"></a>執行部署的命令
[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell
```powershell
New-AzureResourceGroupDeployment -Name \<deployment-name\> -ResourceGroupName \<resource-group-name\> -TemplateUri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-subscription-rule/azuredeploy.json>
```

## <a name="azure-cli"></a>Azure CLI
```azurecli
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-subscription-rule/azuredeploy.json>
```

## <a name="next-steps"></a>後續步驟
現在您已使用 Azure Resource Manager 建立並部署資源，請檢視這些文件，了解如何管理這些資源︰

* [管理 Azure 服務匯流排](service-bus-management-libraries.md)
* [使用 PowerShell 管理服務匯流排](service-bus-manage-with-ps.md)
* [使用服務匯流排總管管理服務匯流排資源](https://github.com/paolosalvatori/ServiceBusExplorer/releases)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/resource-group-authoring-templates.md
[Azure Quickstart Templates]: https://azure.microsoft.com/documentation/templates/?term=service+bus
[Learn more about Service Bus topics and subscriptions]: service-bus-queues-topics-subscriptions.md
[Using Azure PowerShell with Azure Resource Manager]: ../azure-resource-manager/powershell-azure-resource-manager.md
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../azure-resource-manager/xplat-cli-azure-resource-manager.md
[Recommended naming conventions for Azure resources]: ../guidance/guidance-naming-conventions.md
[Service Bus namespace with topic, subscription, and rule]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-topic-subscription-rule/
[Service Bus queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md

