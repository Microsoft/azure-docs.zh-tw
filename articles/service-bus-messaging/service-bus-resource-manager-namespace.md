---
title: 使用 Azure Resource Manager 範本建立服務匯流排傳訊命名空間 | Microsoft Docs
description: 使用 Azure Resource Manager 範本來建立服務匯流排傳訊命名空間
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: dc0d6482-6344-4cef-8644-d4573639f5e4
ms.service: service-bus-messaging
ms.devlang: tbd
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 09/11/2018
ms.author: spelluru
ms.openlocfilehash: 6f3f44394ab11c1b66be3af976dbd1f7d23de96e
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405779"
---
# <a name="create-a-service-bus-namespace-using-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本來建立服務匯流排命名空間

本文說明如何使用 Azure Resource Manager 範本，建立**訊息**類型的服務匯流排命名空間與標準 SKU。 本文也會定義針對部署執行而指定的參數。 您可以直接在自己的部署中使用此範本，或自訂此範本以符合您的需求。

如需關於建立範本的詳細資訊，請參閱[編寫 Azure Resource Manager 範本][Authoring Azure Resource Manager templates]。

如需完整的範本，請參閱 GitHub 上的 [服務匯流排命名空間範本][Service Bus namespace template]。

> [!NOTE]
> 下列 Azure Resource Manager 範本可供下載和部署。 
> 
> * [建立服務匯流排命名空間與佇列](service-bus-resource-manager-namespace-queue.md)
> * [建立服務匯流排命名空間與主題和訂用帳戶](service-bus-resource-manager-namespace-topic.md)
> * [建立服務匯流排命名空間與佇列和授權規則](service-bus-resource-manager-namespace-auth-rule.md)
> * [建立服務匯流排命名空間與主題、訂用帳戶和規則](service-bus-resource-manager-namespace-topic-with-rule.md)
> 
> 若要檢查最新的範本，請造訪 [Azure 快速入門範本][Azure Quickstart Templates]資源庫並搜尋「服務匯流排」。
> 
> 

## <a name="what-will-you-deploy"></a>您將部署什麼？

使用此範本時，您會部署具有[標準或進階](https://azure.microsoft.com/pricing/details/service-bus/) SKU 的服務匯流排命名空間。

若要自動執行部署，請按一下下列按鈕：

[![部署至 Azure](./media/service-bus-resource-manager-namespace/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-servicebus-create-namespace%2Fazuredeploy.json)

## <a name="parameters"></a>參數

透過 Azure 資源管理員，您可以定義在部署範本時想要指定之值的參數。 此範本有一個 `Parameters` 區段，內含所有參數值。 您應該為會隨著要部署的專案或要部署到的環境而變化的值定義參數。 請不要為永遠保持不變的值定義參數。 每個參數值都可在範本中用來定義所部署的資源。

此範本會定義下列參數：

### <a name="servicebusnamespacename"></a>serviceBusNamespaceName

要建立的服務匯流排命名空間名稱。

```json
"serviceBusNamespaceName": {
"type": "string",
"metadata": { 
    "description": "Name of the Service Bus namespace" 
    }
}
```

### <a name="servicebussku"></a>serviceBusSKU

要建立的服務匯流 [SKU](https://azure.microsoft.com/pricing/details/service-bus/) 名稱。

```json
"serviceBusSku": { 
    "type": "string", 
    "allowedValues": [ 
        "Standard",
        "Premium" 
    ], 
    "defaultValue": "Standard", 
    "metadata": { 
        "description": "The messaging tier for service Bus namespace" 
    } 

```

此範本會定義此參數允許使用的值 (標準或進階)。 如果未指定任何值，資源管理員會指定預設值 (標準)。

如需服務匯流排定價的詳細資訊，請參閱[服務匯流排定價與計費][Service Bus pricing and billing]。

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

### <a name="service-bus-namespace"></a>服務匯流排命名空間

建立 **訊息**類型的標準服務匯流排命名空間。

```json
"resources": [
    {
        "apiVersion": "[parameters('serviceBusApiVersion')]",
        "name": "[parameters('serviceBusNamespaceName')]",
        "type": "Microsoft.ServiceBus/Namespaces",
        "location": "[variables('location')]",
        "kind": "Messaging",
        "sku": {
            "name": "Standard",
        },
        "properties": {
        }
    }
]
```

## <a name="commands-to-run-deployment"></a>執行部署的命令

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### <a name="powershell"></a>PowerShell

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-servicebus-create-namespace/azuredeploy.json
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
azure config mode arm

azure group deployment create <my-resource-group> <my-deployment-name> --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-servicebus-create-namespace/azuredeploy.json
```

## <a name="next-steps"></a>後續步驟
現在您已使用 Azure Resource Manager 建立並部署資源，請參閱這些文件，了解如何管理這些資源︰

* [使用 PowerShell 管理服務匯流排](service-bus-manage-with-ps.md)
* [使用服務匯流排總管管理服務匯流排資源](https://github.com/paolosalvatori/ServiceBusExplorer/releases)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/resource-group-authoring-templates.md
[Service Bus namespace template]: https://github.com/Azure/azure-quickstart-templates/blob/master/101-servicebus-create-namespace/
[Azure Quickstart Templates]: https://azure.microsoft.com/documentation/templates/?term=service+bus
[Service Bus pricing and billing]: service-bus-pricing-billing.md
[Using Azure PowerShell with Azure Resource Manager]: ../azure-resource-manager/powershell-azure-resource-manager.md
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: ../azure-resource-manager/xplat-cli-azure-resource-manager.md
