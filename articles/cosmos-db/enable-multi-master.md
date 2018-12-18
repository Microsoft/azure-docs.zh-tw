---
title: 為 Azure Cosmos DB 帳戶啟用多重主機功能
description: 本文說明如何在使用 Azure 入口網站、PowerShell、CLI 或 Azure Resource Manager 範本建立 Azure Cosmos DB 帳戶時，啟用多重主機支援。
services: cosmos-db
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: mjbrown
ms.reviewer: sngun
ms.openlocfilehash: b6fc04fc728f753dc8a3900b26c6c01f03cc7710
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077233"
---
# <a name="enable-multi-master-for-azure-cosmos-db-accounts"></a>為 Azure Cosmos DB 帳戶啟用多重主機功能

多重主機支援會在建立 Azure Cosmos DB 帳戶時啟用。 您可以使用 Azure 入口網站、PowerShell、CLI 或 an Azure Resource Manager 範本，建立 Azure Cosmos DB 帳戶。

> [!IMPORTANT]
> 目前僅能針對新的 Azure Cosmos DB 帳戶啟用多重主機支援。 現有的 Azure Cosmos DB 帳戶無法使用此功能。 我們一直在努力為現有的帳戶提供支援，而且我們將會在提供時，宣佈這項支援。

建立具有多重主機支援的 Azure Cosmos DB 帳戶之後，您可以建立資料庫、容器、上傳文件，以及指派衝突解決原則。 若要解決多重主機和程式碼範例中的衝突，請參閱[多重主機程式碼範例](multi-master-conflict-resolution.md#code-samples)一文。

## <a name="enable-multi-master-using-azure-portal"></a>使用 Azure 入口網站啟用多重主機

1. 登入 [Azure 入口網站](https://portal.azure.com/)

2. 按一下可選取 [建立資源] > [資料庫] > [Azure Cosmos DB]。

3. 在 [新增帳戶] 窗格中，輸入新 Azure Cosmos DB 帳戶的下列設定：

   |**設定**  |**建議的值** |**說明**|
   |---------|---------|---------|
   |訂用帳戶   | {您的訂用帳戶}  |選取用於此 Azure Cosmos DB 帳戶的 Azure 訂用帳戶。  |
   |資源群組  |   {您的資源群組名稱}    |  選取現有的資源群組，或選取  **[新建]** ，然後為您的帳戶輸入新的資源群組名稱。 |
   |帳戶名稱 | {您的帳戶名稱}   |  輸入唯一名稱來識別您的 Azure Cosmos DB 帳戶。        |
   |API  |   任意   |  選取 API 類型。   |
   |位置  | 選取任何區域   | 選取用來主控 Azure Cosmos DB 帳戶的地理位置。 此帳戶將會在多個區域中，因此您可以選擇任何區域。  |
   |啟用異地備援   |  啟用  |  選取可啟用要在下方選取的多重主機。   |
   |啟用多重主機 | 啟用  | 選取可啟用此帳戶的多重主機。 |


## <a name="using-multi-master-in-sdks"></a>在 SDK 中使用多重主機

在應用程式中透過啟用多重主機的帳戶，您可以運用 ConnectionPolicy 善用多重主機功能，如下所示。

```csharp
ConnectionPolicy policy = new ConnectionPolicy
{
   ConnectionMode = ConnectionMode.Direct,
   ConnectionProtocol = Protocol.Tcp,
   UseMultipleWriteLocations = true,
};
policy.PreferredLocations.Add(LocationNames.WestUS);
policy.PreferredLocations.Add(LocationNames.NorthEurope);
policy.PreferredLocations.Add(LocationNames.SoutheastAsia);
```

## <a name="enable-multi-master-using-powershell"></a>使用 PowerShell 啟用多重主機

您也可以將 `enableMultipleWriteLocations` 參數設定為 "true"，藉此建立啟用多重主機的 Cosmos DB 帳戶。 若要在已啟用多重主機的情況下建立 Azure Cosmos DB 帳戶，請開啟 PowerShell 視窗，然後執行下列指令碼：

```azurepowershell-interactive
$locations = @(@{"locationName"="East US"; "failoverPriority"=0},
             @{"locationName"="West US"; "failoverPriority"=1})

$iprangefilter = "<ip-range-filter>"

$consistencyPolicy = @{"defaultConsistencyLevel"="Session";
                       "maxIntervalInSeconds"= "10";
                       "maxStalenessPrefix"="200"}

$CosmosDBProperties = @{"databaseAccountOfferType"="Standard";
                        "locations"=$locations;
                        "consistencyPolicy"=$consistencyPolicy;
                        "ipRangeFilter"=$iprangefilter;
                        "enableMultipleWriteLocations"="true"}

New-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
  -ApiVersion "2015-04-08" `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -Name "myCosmosDbAccount" `
  -Properties $CosmosDBProperties
```

## <a name="enable-multi-master-using-cli"></a>使用 CLI 啟用多重主機

您可以將 enable-multiple-write-locations 參數設定為 "true"，藉此啟用多重主機。 若要在已啟用多重主機的情況下建立 Azure Cosmos DB 帳戶，請開啟 Azure CLI 或 Cloud Shell，然後執行下列指令碼：

```azurecli-interactive
az cosmosdb create \
   –-name "myCosmosDbAccount" \
   --resource-group "myResourceGroup" \
   --default-consistency-level "Session" \
   --enable-automatic-failover "true" \
   --locations "EastUS=0" "WestUS=1" \
   --enable-multiple-write-locations true \
```

## <a name="enable-multi-master-using-resource-manager-template"></a>使用 Resource Manager 範本啟用多重主機

下列 JSON 程式碼是您可用來部署 Azure Cosmos DB 帳戶的範例 Resource Manager 範本。 若要了解 Resource Manager 範本格式和語法，請參閱 [Resource Manager](../azure-resource-manager/resource-group-authoring-templates.md) 文件。 請注意，此範本中的主要參數是 "enableMultipleWriteLocations": true。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "String"
        },
        "location": {
            "type": "String"
        },
        "locationName": {
            "type": "String"
        },
        "defaultExperience": {
            "type": "String"
        }
    },
    "resources": [
        {
            "type": "Microsoft.DocumentDb/databaseAccounts",
            "kind": "GlobalDocumentDB",
            "name": "[parameters('name')]",
            "apiVersion": "2015-04-08",
            "location": "[parameters('location')]",
            "tags": {
                "defaultExperience": "[parameters('defaultExperience')]"
            },
            "properties": {
                "databaseAccountOfferType": "Standard",
                "locations": [
                    {
                        "id": "[concat(parameters('name'), '-', parameters('location'))]",
                        "failoverPriority": 0,
                        "locationName": "[parameters('locationName')]"
                    }
                ],
                "isVirtualNetworkFilterEnabled": false,
                "enableMultipleWriteLocations": true,
                "virtualNetworkRules": [],
                "dependsOn": []
            }
        }
    ]
}
```

## <a name="next-steps"></a>後續步驟

在本文中，您已經了解如何為 Azure Cosmos DB 帳戶啟用多重主機支援。 接著，請參閱下列資源：

* [搭配開放原始碼 NoSQL 資料庫使用多重主機](multi-master-oss-nosql.md)

* [了解 Azure Cosmos DB 中的衝突解決方法](multi-master-conflict-resolution.md)
