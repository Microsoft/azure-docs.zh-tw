---
title: 使用範本來建立 Azure 事件中樞命名空間和取用者群組 | Microsoft Docs
description: 使用 Azure Resource Manager 範本來建立含有事件中樞和取用者群組的事件中樞命名空間
services: event-hubs
documentationcenter: .net
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: 28bb4591-1fd7-444f-a327-4e67e8878798
ms.service: event-hubs
ms.devlang: tbd
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 04/30/2018
ms.author: shvija
ms.openlocfilehash: 2edcb89c6468896c2e878f61ed8330917037c3eb
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40002913"
---
# <a name="create-an-event-hubs-namespace-with-event-hub-and-consumer-group-using-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本，以事件中樞和取用者群組來建立事件中樞命名空間

本文說明如何使用 Azure Resource Manager 範本來建立一個類型為[事件中樞](event-hubs-what-is-event-hubs.md)、含有一個事件中樞和一個取用者群組的的命名空間。 本文說明如何定義要部署哪些資源，以及如何定義執行部署時所指定的參數。 您可以直接在自己的部署中使用此範本，或自訂此範本以符合您的需求。

如需關於建立範本的資訊，請參閱[編寫 Azure Resource Manager 範本][Authoring Azure Resource Manager templates]。

如需完整的範本，請參閱 GitHub 上的[事件中樞和取用者群組範本][Event Hub and consumer group template]。

> [!NOTE]
> 若要檢查最新的範本，請造訪 [Azure 快速入門範本][Azure Quickstart Templates] 資源庫並搜尋事件中樞。
> 
> 

## <a name="what-will-you-deploy"></a>您將部署什麼？

藉由此範本，您會部署含有事件中樞和取用者群組的「事件中樞」命名空間。

若要自動執行部署，請按一下下列按鈕：

[![部署至 Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-event-hubs-create-event-hub-and-consumer-group%2Fazuredeploy.json)

## <a name="parameters"></a>參數

透過 Azure 資源管理員，您可以定義在部署範本時想要指定之值的參數。 此範本有一個 `Parameters` 區段，內含所有參數值。 針對會隨著所要部署專案或部署環境而變化的值，您應該為這些值定義參數。 請不要為永遠保持不變的值定義參數。 範本中的每個參數值會定義所部署的資源。

範本會定義下列參數：

### <a name="eventhubnamespacename"></a>eventHubNamespaceName

要建立的事件中樞命名空間名稱。

```json
"eventHubNamespaceName": {
"type": "string"
}
```

### <a name="eventhubname"></a>eventHubName

在「事件中樞」命名空間中建立的事件中樞名稱。

```json
"eventHubName": {
"type": "string"
}
```

### <a name="eventhubconsumergroupname"></a>eventHubConsumerGroupName

為事件中樞建立的取用者群組名稱。

```json
"eventHubConsumerGroupName": {
"type": "string"
}
```

### <a name="apiversion"></a>apiVersion

範本的 API 版本。

```json
"apiVersion": {
"type": "string"
}
```

## <a name="resources-to-deploy"></a>要部署的資源

建立類型為 **EventHubs**、含有事件中樞和取用者群組的命名空間：

```json
"resources":[  
      {  
         "apiVersion":"[variables('ehVersion')]",
         "name":"[parameters('namespaceName')]",
         "type":"Microsoft.EventHub/namespaces",
         "location":"[variables('location')]",
         "sku":{  
            "name":"Standard",
            "tier":"Standard"
         },
         "resources":[  
            {  
               "apiVersion":"[variables('ehVersion')]",
               "name":"[parameters('eventHubName')]",
               "type":"EventHubs",
               "dependsOn":[  
                  "[concat('Microsoft.EventHub/namespaces/', parameters('namespaceName'))]"
               ],
               "properties":{  
                  "path":"[parameters('eventHubName')]"
               },
               "resources":[  
                  {  
                     "apiVersion":"[variables('ehVersion')]",
                     "name":"[parameters('consumerGroupName')]",
                     "type":"ConsumerGroups",
                     "dependsOn":[  
                        "[parameters('eventHubName')]"
                     ],
                     "properties":{  

                     }
                  }
               ]
            }
         ]
      }
   ],
```

## <a name="commands-to-run-deployment"></a>執行部署的命令

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell

```azurepowershell
New-AzureRmResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-event-hubs-create-event-hub-and-consumer-group/azuredeploy.json
```

## <a name="azure-cli"></a>Azure CLI

```azurecli
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-event-hubs-create-event-hub-and-consumer-group/azuredeploy.json][]
```

## <a name="next-steps"></a>後續步驟

您可以造訪下列連結以深入了解事件中樞︰

* [事件中心概觀](event-hubs-what-is-event-hubs.md)
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/resource-group-authoring-templates.md
[Azure Quickstart Templates]:  https://azure.microsoft.com/documentation/templates/?term=event+hubs
[Using Azure PowerShell with Azure Resource Manager]: ../powershell-azure-resource-manager.md
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../xplat-cli-azure-resource-manager.md
[Event hub and consumer group template]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-event-hubs-create-event-hub-and-consumer-group/
