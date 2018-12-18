---
title: 使用 Azure Resource Manager 佈建 Redis Cache | Microsoft Docs
description: 使用 Azure 資源管理員範本來部署 Azure Redis 快取。
services: app-service
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: ce6f5372-7038-4655-b1c5-108f7c148282
ms.service: cache
ms.workload: web
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: wesmc
ms.openlocfilehash: b26116b974abbfe410b0a6ebc0186d73f4eea1bf
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38452578"
---
# <a name="create-a-redis-cache-using-a-template"></a>使用範本建立 Redis 快取
在本主題中，您將學習如何建立 Azure Resource Manager 範本，以部署 Azure Redis 快取。 快取可以搭配現有的儲存體帳戶以保留診斷資料。 您將學習如何定義要部署哪些資源，以及如何定義執行部署時所指定的參數。 您可以直接在自己的部署中使用此範本，或自訂此範本以符合您的需求。

目前對於訂用帳戶，同一區域中所有快取的診斷設定是共用的。 更新區域中的一個快取將會影響區域中的所有其他快取。

如需關於建立範本的詳細資訊，請參閱 [編寫 Azure 資源管理員範本](../azure-resource-manager/resource-group-authoring-templates.md)。

如需完整範本，請參閱 [Redis 快取範本](https://github.com/Azure/azure-quickstart-templates/blob/master/101-redis-cache/azuredeploy.json)。

> [!NOTE]
> 新 [Premium 層](cache-premium-tier-intro.md) 中有可用的 Resource Manager 範本。 
> 
> * [建立具有叢集的 Premium Redis 快取](https://azure.microsoft.com/documentation/templates/201-redis-premium-cluster-diagnostics/)
> * [建立具有資料永續性的 Premium Redis 快取](https://azure.microsoft.com/documentation/templates/201-redis-premium-persistence/)
> * [建立具有 VNet 和選擇性叢集的 Premium Redis 快取](https://azure.microsoft.com/documentation/templates/201-redis-premium-vnet-cluster-diagnostics/)
> 
> 若要查看最新的範本，請參閱 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)並搜尋 `Redis Cache`。
> 
> 

## <a name="what-you-will-deploy"></a>部署內容
在此範本中，您將部署使用現有儲存體帳戶的 Azure Redis 快取來診斷資料。

若要自動執行部署，請按一下下列按鈕：

[![部署至 Azure](./media/cache-redis-cache-arm-provision/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-redis-cache%2Fazuredeploy.json)

## <a name="parameters"></a>參數
透過 Azure 資源管理員，您可以定義在部署範本時想要指定之值的參數。 此範本有一個 Parameters 區段，內含所有參數值。
您應該為會隨著要部署的專案或要部署到的環境而變化的值定義參數。 請不要為永遠保持不變的值定義參數。 每個參數值都可在範本中用來定義所部署的資源。 

[!INCLUDE [app-service-web-deploy-redis-parameters](../../includes/cache-deploy-parameters.md)]

### <a name="rediscachelocation"></a>redisCacheLocation
Redis 快取的位置。 針對最佳效能，使用要與快取搭配使用之應用程式相同的位置。

    "redisCacheLocation": {
      "type": "string"
    }

### <a name="existingdiagnosticsstorageaccountname"></a>existingDiagnosticsStorageAccountName
要用於診斷的現有儲存體帳戶名稱。 

    "existingDiagnosticsStorageAccountName": {
      "type": "string"
    }

### <a name="enablenonsslport"></a>enableNonSslPort
指出是否允許透過非 SSL 連接埠存取的布林值。

    "enableNonSslPort": {
      "type": "bool"
    }

### <a name="diagnosticsstatus"></a>diagnosticsStatus
指出診斷是否啟用的值。 使用 ON 或 OFF。

    "diagnosticsStatus": {
      "type": "string",
      "defaultValue": "ON",
      "allowedValues": [
            "ON",
            "OFF"
        ]
    }

## <a name="resources-to-deploy"></a>要部署的資源
### <a name="redis-cache"></a>Redis 快取
建立 Azure Redis 快取。

    {
      "apiVersion": "2015-08-01",
      "name": "[parameters('redisCacheName')]",
      "type": "Microsoft.Cache/Redis",
      "location": "[parameters('redisCacheLocation')]",
      "properties": {
        "enableNonSslPort": "[parameters('enableNonSslPort')]",
        "sku": {
          "capacity": "[parameters('redisCacheCapacity')]",
          "family": "[parameters('redisCacheFamily')]",
          "name": "[parameters('redisCacheSKU')]"
        }
      },
      "resources": [
        {
          "apiVersion": "2015-07-01",
          "type": "Microsoft.Cache/redis/providers/diagnosticsettings",
          "name": "[concat(parameters('redisCacheName'), '/Microsoft.Insights/service')]",
          "location": "[parameters('redisCacheLocation')]",
          "dependsOn": [
            "[concat('Microsoft.Cache/Redis/', parameters('redisCacheName'))]"
          ],
          "properties": {
            "status": "[parameters('diagnosticsStatus')]",
            "storageAccountName": "[parameters('existingDiagnosticsStorageAccountName')]"
          }
        }
      ]
    }



## <a name="commands-to-run-deployment"></a>執行部署的命令
[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### <a name="powershell"></a>PowerShell
    New-AzureRmResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-redis-cache/azuredeploy.json -ResourceGroupName ExampleDeployGroup -redisCacheName ExampleCache

### <a name="azure-cli"></a>Azure CLI
    azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-redis-cache/azuredeploy.json -g ExampleDeployGroup


