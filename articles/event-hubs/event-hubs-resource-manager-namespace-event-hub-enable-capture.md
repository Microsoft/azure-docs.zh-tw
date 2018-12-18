---
title: 使用範本來建立 Azure 事件中樞命名空間並啟用擷取 | Microsoft Docs
description: 使用 Azure Resource Manager 範本建立含有一個事件中樞的 Azure 事件中樞命名空間並啟用擷取
services: event-hubs
documentationcenter: .net
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: 8bdda6a2-5ff1-45e3-b696-c553768f1090
ms.service: event-hubs
ms.devlang: tbd
ms.topic: get-started-article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 08/16/2018
ms.author: shvija
ms.openlocfilehash: c8341b40ba7616add1415178a2f0775fbbc66ec1
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "41920268"
---
# <a name="create-a-namespace-with-event-hub-and-enable-capture-using-a-template"></a>使用範本來建立含有事件中樞的命名空間並啟用擷取

本文說明如何使用 Azure Resource Manager 範本來建立含有一個事件中樞執行個體的[事件中樞](event-hubs-what-is-event-hubs.md)命名空間，也可在該事件中樞上啟用[擷取功能](event-hubs-capture-overview.md)。 此文章說明如何定義要部署哪些資源，以及如何定義執行部署時所指定的參數。 您可以直接在自己的部署中使用此範本，或自訂此範本以符合您的需求。

本文也會示範如何根據您選擇的目的地，指定將事件擷取到 Azure 儲存體 Blob 或 Azure Data Lake Store 中。

如需關於建立範本的詳細資訊，請參閱[編寫 Azure Resource Manager 範本][Authoring Azure Resource Manager templates]。

如需 Azure 資源命名慣例相關模式和實務的詳細資訊，請參閱 [Azure 資源命名慣例][Azure Resources naming conventions]。

對於所有範本，按一下下列 GitHub 連結：

- [事件中樞和啟用擷取至儲存體範本][Event Hub and enable Capture to Storage template] 
- [事件中樞和啟用擷取至 Azure Data Lake Store 範本][Event Hub and enable Capture to Azure Data Lake Store template]

> [!NOTE]
> 若要檢查最新的範本，請造訪 [Azure 快速入門範本][Azure Quickstart Templates] 資源庫並搜尋事件中樞。
> 
> 

## <a name="what-will-you-deploy"></a>您將部署什麼？

使用此範本，您可部署含有事件中樞的事件中樞命名空間，也可啟用[事件中樞擷取](event-hubs-capture-overview.md)。 事件中樞擷取功能讓您能夠在指定時間或大小間隔內，自動將事件中樞的串流資料傳遞到 Azure Blob 儲存體或 Azure Data Lake Store。 按一下以下按鈕來啟用「事件中心擷取到 Azure 儲存體中」：

[![部署至 Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-capture%2Fazuredeploy.json)

按一下以下按鈕來啟用「事件中心擷取到 Azure Data Lake Store 中」：

[![部署至 Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-capture-for-adls%2Fazuredeploy.json)

## <a name="parameters"></a>參數

透過 Azure 資源管理員，您可以定義在部署範本時想要指定之值的參數。 此範本有一個 `Parameters` 區段，內含所有參數值。 您應該為會隨著要部署的專案或要部署到的環境而變化的值定義參數。 請不要為永遠保持不變的值定義參數。 每個參數值都可在範本中用來定義所部署的資源。

範本會定義下列參數。

### <a name="eventhubnamespacename"></a>eventHubNamespaceName

要建立的事件中樞命名空間名稱。

```json
"eventHubNamespaceName":{  
     "type":"string",
     "metadata":{  
         "description":"Name of the EventHub namespace"
      }
}
```

### <a name="eventhubname"></a>eventHubName

在「事件中樞」命名空間中建立的事件中樞名稱。

```json
"eventHubName":{  
    "type":"string",
    "metadata":{  
        "description":"Name of the event hub"
    }
}
```

### <a name="messageretentionindays"></a>messageRetentionInDays

要在事件中樞中保留訊息的天數。 

```json
"messageRetentionInDays":{
    "type":"int",
    "defaultValue": 1,
    "minValue":"1",
    "maxValue":"7",
    "metadata":{
       "description":"How long to retain the data in event hub"
     }
 }
```

### <a name="partitioncount"></a>partitionCount

要在事件中樞中建立的資料分割數目。

```json
"partitionCount":{
    "type":"int",
    "defaultValue":2,
    "minValue":2,
    "maxValue":32,
    "metadata":{
        "description":"Number of partitions chosen"
    }
 }
```

### <a name="captureenabled"></a>captureEnabled

在事件中樞上啟用封存擷取功能。

```json
"captureEnabled":{
    "type":"string",
    "defaultValue":"true",
    "allowedValues": [
    "false",
    "true"],
    "metadata":{
        "description":"Enable or disable the Capture for your event hub"
    }
 }
```
### <a name="captureencodingformat"></a>captureEncodingFormat

您指定用來將事件資料序列化的編碼格式。

```json
"captureEncodingFormat":{
    "type":"string",
    "defaultValue":"Avro",
    "allowedValues":[
    "Avro"],
    "metadata":{
        "description":"The encoding format in which Capture serializes the EventData"
    }
}
```

### <a name="capturetime"></a>captureTime

事件中樞擷取功能開始擷取資料的時間間隔。

```json
"captureTime":{
    "type":"int",
    "defaultValue":300,
    "minValue":60,
    "maxValue":900,
    "metadata":{
         "description":"The time window in seconds for the capture"
    }
}
```

### <a name="capturesize"></a>captureSize
擷取功能開始擷取資料的大小間隔。

```json
"captureSize":{
    "type":"int",
    "defaultValue":314572800,
    "minValue":10485760,
    "maxValue":524288000,
    "metadata":{
        "description":"The size window in bytes for capture"
    }
}
```

### <a name="capturenameformat"></a>captureNameFormat

事件中樞擷取功能用來寫入 Avro 檔案的名稱格式。 請注意，擷取名稱格式必須包含 `{Namespace}`、`{EventHub}`、`{PartitionId}`、`{Year}`、`{Month}`、`{Day}`、`{Hour}`、`{Minute}` 和 `{Second}` 欄位。 這些欄位可以依任何順序排列 (不論是否有分隔符號)。
 
```json
"captureNameFormat": {
      "type": "string",
      "defaultValue": "{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}",
      "metadata": {
        "description": "A Capture Name Format must contain {Namespace}, {EventHub}, {PartitionId}, {Year}, {Month}, {Day}, {Hour}, {Minute} and {Second} fields. These can be arranged in any order with or without delimeters. E.g.  Prod_{EventHub}/{Namespace}\\{PartitionId}_{Year}_{Month}/{Day}/{Hour}/{Minute}/{Second}"
      }
    }
  
```

### <a name="apiversion"></a>apiVersion

範本的 API 版本。

```json
 "apiVersion":{  
    "type":"string",
    "defaultValue":"2017-04-01",
    "metadata":{  
        "description":"ApiVersion used by the template"
    }
 }
```

如果您選擇 Azure 儲存體作為目的地，請使用下列參數。

### <a name="destinationstorageaccountresourceid"></a>destinationStorageAccountResourceId

擷取功能需要有 Azure 儲存體帳戶資源識別碼，才能為您所需的儲存體帳戶啟用擷取功能。

```json
 "destinationStorageAccountResourceId":{
    "type":"string",
    "metadata":{
        "description":"Your existing Storage account resource ID where you want the blobs be captured"
    }
 }
```

### <a name="blobcontainername"></a>blobContainerName

用來擷取存事件資料的 blob 容器。

```json
 "blobContainerName":{
    "type":"string",
    "metadata":{
        "description":"Your existing storage container in which you want the blobs captured"
    }
}
```

如果您選擇 Azure Data Lake Store 作為目的地，請使用下列參數。 您必須在您想要擷取事件的 Data Lake Store 路徑上設定權限。 若要設定權限，請參閱[本文](event-hubs-capture-enable-through-portal.md#capture-data-to-an-azure-data-lake-store-account)。

### <a name="subscriptionid"></a>subscriptionId

事件中樞命名空間和 Azure Data Lake Store 的訂用帳戶 ID。 這兩個資源都必須屬於同一個訂用帳戶識別碼。

```json
"subscriptionId": {
    "type": "string",
    "metadata": {
        "description": "Subscription ID of both Azure Data Lake Store and Event Hubs namespace"
     }
 }
```

### <a name="datalakeaccountname"></a>dataLakeAccountName

已擷取事件的 Azure Data Lake Store 名稱。

```json
"dataLakeAccountName": {
    "type": "string",
    "metadata": {
        "description": "Azure Data Lake Store name"
    }
}
```

### <a name="datalakefolderpath"></a>dataLakeFolderPath

已擷取事件的目的地資料夾路徑。 這是 Data Lake Store 中的資料夾，而事件會在擷取作業期間推送至該資料夾。 若要在此資料夾上設定權限，請參閱[使用 Azure Data Lake Store 從事件中樞擷取資料](../data-lake-store/data-lake-store-archive-eventhub-capture.md)。

```json
"dataLakeFolderPath": {
    "type": "string",
    "metadata": {
        "description": "Destination capture folder path"
    }
}
```

## <a name="resources-to-deploy-for-azure-storage-as-destination-to-captured-events"></a>要針對 Azure 儲存體部署為已擷取事件之目的地的資源

建立類型為 **EventHub**、含有一個事件中樞，而且也會啟用「擷取至 Azure Blob 儲存體」功能的命名空間。

```json
"resources":[  
      {  
         "apiVersion":"[variables('ehVersion')]",
         "name":"[parameters('eventHubNamespaceName')]",
         "type":"Microsoft.EventHub/Namespaces",
         "location":"[variables('location')]",
         "sku":{  
            "name":"Standard",
            "tier":"Standard"
         },
         "resources": [
    {
      "apiVersion": "2017-04-01",
      "name": "[parameters('eventHubNamespaceName')]",
      "type": "Microsoft.EventHub/Namespaces",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "isAutoInflateEnabled": "true",
        "maximumThroughputUnits": "7"
      },
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('eventHubName')]",
          "type": "EventHubs",
          "dependsOn": [
            "[concat('Microsoft.EventHub/namespaces/', parameters('eventHubNamespaceName'))]"
          ],
          "properties": {
            "messageRetentionInDays": "[parameters('messageRetentionInDays')]",
            "partitionCount": "[parameters('partitionCount')]",
            "captureDescription": {
              "enabled": "true",
              "encoding": "[parameters('captureEncodingFormat')]",
              "intervalInSeconds": "[parameters('captureTime')]",
              "sizeLimitInBytes": "[parameters('captureSize')]",
              "destination": {
                "name": "EventHubArchive.AzureBlockBlob",
                "properties": {
                  "storageAccountResourceId": "[parameters('destinationStorageAccountResourceId')]",
                  "blobContainer": "[parameters('blobContainerName')]",
                  "archiveNameFormat": "[parameters('captureNameFormat')]"
                }
              }
            }
          }

        }
      ]
    }
  ]
```

## <a name="resources-to-deploy-for-azure-data-lake-store-as-destination"></a>要針對 Azure Data Lake Store 部署為目的地的資源

建立類型為 **EventHub**、含有一個事件中樞，而且也會啟用「擷取至 Azure Data Lake Store」功能的命名空間。

```json
 "resources": [
        {
            "apiVersion": "2017-04-01",
            "name": "[parameters('namespaceName')]",
            "type": "Microsoft.EventHub/Namespaces",
            "location": "[variables('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard"
            },
            "resources": [
                {
                    "apiVersion": "2017-04-01",
                    "name": "[parameters('eventHubName')]",
                    "type": "EventHubs",
                    "dependsOn": [
                        "[concat('Microsoft.EventHub/namespaces/', parameters('namespaceName'))]"
                    ],
                    "properties": {
                        "path": "[parameters('eventHubName')]",
                        "captureDescription": {
                            "enabled": "true",
                            "encoding": "[parameters('archiveEncodingFormat')]",
                            "intervalInSeconds": "[parameters('captureTime')]",
                            "sizeLimitInBytes": "[parameters('captureSize')]",
                            "destination": {
                                "name": "EventHubArchive.AzureDataLake",
                                "properties": {
                                    "DataLakeSubscriptionId": "[parameters('subscriptionId')]",
                                    "DataLakeAccountName": "[parameters('dataLakeAccountName')]",
                                    "DataLakeFolderPath": "[parameters('dataLakeFolderPath')]",
                                    "ArchiveNameFormat": "[parameters('captureNameFormat')]"
                                }
                            }
                        }
                    }
                }
            ]
        }
    ]
```

## <a name="commands-to-run-deployment"></a>執行部署的命令

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell

部署您的範本，以啟用「事件中樞擷取至 Azure 儲存體」功能：
 
```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture/azuredeploy.json
```

部署您的範本，以啟用「事件中樞擷取至 Azure Data Lake Store」功能：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture-for-adls/azuredeploy.json
```

## <a name="azure-cli"></a>Azure CLI

Azure Blob 儲存體作為目的地：

```azurecli
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture/azuredeploy.json][]
```

Azure Data Lake Store 作為目的地：

```azurecli
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture-for-adls/azuredeploy.json][]
```

## <a name="next-steps"></a>後續步驟

您也可以透過 [Azure 入口網站](https://portal.azure.com)設定事件中樞擷取功能。 如需詳細資訊，請參閱[使用 Azure 入口網站啟用事件中樞擷取功能](event-hubs-capture-enable-through-portal.md)。

您可以造訪下列連結以深入了解事件中樞︰

* [事件中心概觀](event-hubs-what-is-event-hubs.md)
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/resource-group-authoring-templates.md
[Azure Quickstart Templates]:  https://azure.microsoft.com/documentation/templates/?term=event+hubs
[Azure Resources naming conventions]: https://azure.microsoft.com/documentation/articles/guidance-naming-conventions/
[Event hub and enable Capture to Storage template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-capture
[Event hub and enable Capture to Azure Data Lake Store template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-capture-for-adls