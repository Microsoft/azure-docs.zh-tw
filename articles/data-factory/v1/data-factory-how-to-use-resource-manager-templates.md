---
title: 在 Data Factory 中使用 Resource Manager 範本 | Microsoft Docs
description: 了解如何建立及使用 Azure Resource Manager 範本來建立 Data Factory 實體。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.assetid: 37724021-f55f-4e85-9206-6d4a48bda3d8
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 3419f8718396bfb4ec894310e545f6a8a5b8f718
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37049259"
---
# <a name="use-templates-to-create-azure-data-factory-entities"></a>使用範本來建立 Azure Data Factory 實體
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 

## <a name="overview"></a>概觀
基於資料整合需求使用 Azure Data Factory 時，您可能會發現自己跨不同環境重複使用相同的模式，或在相同的解決方案內反覆地實作相同的工作。 範本可協助您輕鬆地實作和管理這些案例。 Azure Data Factory 中的範本最適用於涉及重複使用和重複時。

假設某個組織在全球各地有 10 個製造工廠。 每個工廠中的記錄都會儲存在不同的內部部署 SQL Server 資料庫中。 公司想要在雲端中建置單一資料倉儲，以進行臨機操作分析。 它也想要具有相同的邏輯，但開發、測試和生產環境的組態不同。

在此情況下，必須在相同的環境內重複執行工作，但每個製造工廠的 10 個資料處理站各有不同的值。 實際上，具有**重複**情況。 範本化允許任意使用這個泛形流程 (即在每個資料處理站中具有相同活動的管線)，但會針對每個製造工廠使用不同的參數檔案。

此外，組織想要跨不同環境部署這 10 個資料處理站多次時，範本可以針對開發、測試和生產環境使用不同的參數檔案，來使用這個**重複使用性**。

## <a name="templating-with-azure-resource-manager"></a>使用 Azure Resource Manager 範本化
[Azure Resource Manager 範本](../../azure-resource-manager/resource-group-overview.md#template-deployment)是達成 Azure Data Factory 中範本化的不錯方式。 Resource Manager 範本透過 JSON 檔案來定義 Azure 解決方案的基礎結構和組態。 因為 Azure Resource Manager 範本是與所有/大部分 Azure 服務搭配運作，所以可以廣泛用來輕鬆地管理 Azure 資產的所有資源。 若要深入了解 Resource Manager 範本的一般資訊，請參閱[撰寫 Azure Resource Manager 範本](../../azure-resource-manager/resource-group-authoring-templates.md) 。

## <a name="tutorials"></a>教學課程
如需使用 Resource Manager 範本建立 Data Factory 實體的逐步指示，請參閱下列教學課程︰

* [教學課程：使用 Azure Resource Manager 範本建立管線以複製資料](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
* [教學課程：使用 Azure Resource Manager 範本建立管線以處理資料](data-factory-build-your-first-pipeline.md)

## <a name="data-factory-templates-on-github"></a>GitHub 上的 Data Factory 範本
請參考 GitHub 上的下列 Azure 快速入門範本：

* [建立 Data Factory 以將資料從 Azure Blob 儲存體複製到 Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-blob-to-sql-copy)
* [在 Azure HDInsight 叢集上使用 Hive 活動建立 Data Factory](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-hive-transformation)
* [建立 Data Factory 以將資料從 Salesforce 複製到 Azure Blob](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-salesforce-to-blob-copy)
* [建立 Data factory 以鏈結活動︰將資料從 FTP 伺服器複製到 Azure Blob、叫用隨選 HDInsight 叢集上的 hive 指令碼來轉換資料，並將結果複製到 Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-data-factory-ftp-hive-blob)

在 [Azure 快速啟動](https://azure.microsoft.com/documentation/templates/)上自由共用 Azure Data Factory 範本。 開發可透過這個存放庫共用的範本時，請參閱[參與指南](https://github.com/Azure/azure-quickstart-templates/tree/master/1-CONTRIBUTION-GUIDE)。

下列各節提供在 Resource Manager 範本中定義 Data Factory 資源的詳細資料。

## <a name="defining-data-factory-resources-in-templates"></a>在範本中定義 Data Factory 資源
用於定義資料處理站的最上層範本是︰

```JSON
"$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
"contentVersion": "1.0.0.0",
"parameters": { ...
},
"variables": { ...
},
"resources": [
{
    "name": "[parameters('dataFactoryName')]",
    "apiVersion": "[variables('apiVersion')]",
    "type": "Microsoft.DataFactory/datafactories",
    "location": "westus",
    "resources": [
    { "type": "linkedservices",
        ...
    },
    {"type": "datasets",
        ...
    },
    {"type": "dataPipelines",
        ...
    }
}
```

### <a name="define-data-factory"></a>定義資料處理站
您可以在 Resource Manager 範本中定義資料處理站，如下列範例所示︰

```JSON
"resources": [
{
    "name": "[variables('<mydataFactoryName>')]",
    "apiVersion": "2015-10-01",
    "type": "Microsoft.DataFactory/datafactories",
    "location": "East US"
}
```
dataFactoryName 在 “variables” 中定義為：

```JSON
"dataFactoryName": "[concat('<myDataFactoryName>', uniqueString(resourceGroup().id))]",
```

### <a name="define-linked-services"></a>定義連結服務

```JSON
"type": "linkedservices",
"name": "[variables('<LinkedServiceName>')]",
"apiVersion": "2015-10-01",
"dependsOn": [ "[variables('<dataFactoryName>')]" ],
"properties": {
    ...
}
```

如需所要部署之特定連結服務的 JSON 屬性詳細資料，請參閱[儲存體連結服務](data-factory-azure-blob-connector.md#azure-storage-linked-service)或[算連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)。 “dependsOn” 參數指定對應資料處理站的名稱。 下列 JSON 定義中會顯示針對 Azure 儲存體定義連結服務的範例：

### <a name="define-datasets"></a>定義資料集

```JSON
"type": "datasets",
"name": "[variables('<myDatasetName>')]",
"dependsOn": [
    "[variables('<dataFactoryName>')]",
    "[variables('<myDatasetLinkedServiceName>')]"
],
"apiVersion": "2015-10-01",
"properties": {
    ...
}
```
如需您想要部署之特定資料集類型的 JSON 屬性詳細資料，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 請注意，“dependsOn” 參數指定對應資料處理站和儲存體連結服務的名稱。 下列 JSON 定義中會顯示針對 Azure Blob 儲存體定義資料集類型的範例：

```JSON
"type": "datasets",
"name": "[variables('storageDataset')]",
"dependsOn": [
    "[variables('dataFactoryName')]",
    "[variables('storageLinkedServiceName')]"
],
"apiVersion": "2015-10-01",
"properties": {
"type": "AzureBlob",
"linkedServiceName": "[variables('storageLinkedServiceName')]",
"typeProperties": {
    "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
    "fileName": "[parameters('sourceBlobName')]",
    "format": {
        "type": "TextFormat"
    }
},
"availability": {
    "frequency": "Hour",
    "interval": 1
}
```

### <a name="define-pipelines"></a>定義管線

```JSON
"type": "dataPipelines",
"name": "[variables('<mypipelineName>')]",
"dependsOn": [
    "[variables('<dataFactoryName>')]",
    "[variables('<inputDatasetLinkedServiceName>')]",
    "[variables('<outputDatasetLinkedServiceName>')]",
    "[variables('<inputDataset>')]",
    "[variables('<outputDataset>')]"
],
"apiVersion": "2015-10-01",
"properties": {
    activities: {
        ...
    }
}
```

如需定義您想要部署之特定管線和活動的 JSON 屬性詳細資料，請參閱[定義管線](data-factory-create-pipelines.md#pipeline-json)。 請注意，“dependsOn” 參數指定資料處理站的名稱，以及任何對應連結服務或資料集。 下列 JSON 片段顯示將資料從 Azure Blob 儲存體複製至 Azure SQL Database 的管線範例：

```JSON
"type": "datapipelines",
"name": "[variables('pipelineName')]",
"dependsOn": [
    "[variables('dataFactoryName')]",
    "[variables('azureStorageLinkedServiceName')]",
    "[variables('azureSqlLinkedServiceName')]",
    "[variables('blobInputDatasetName')]",
    "[variables('sqlOutputDatasetName')]"
],
"apiVersion": "2015-10-01",
"properties": {
    "activities": [
    {
        "name": "CopyFromAzureBlobToAzureSQL",
        "description": "Copy data frm Azure blob to Azure SQL",
        "type": "Copy",
        "inputs": [
            {
                "name": "[variables('blobInputDatasetName')]"
            }
        ],
        "outputs": [
            {
                "name": "[variables('sqlOutputDatasetName')]"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "BlobSource"
            },
            "sink": {
                "type": "SqlSink",
                "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM {0}', 'emp')"
            },
            "translator": {
                "type": "TabularTranslator",
                "columnMappings": "Column0:FirstName,Column1:LastName"
            }
        },
        "Policy": {
            "concurrency": 1,
            "executionPriorityOrder": "NewestFirst",
            "retry": 3,
            "timeout": "01:00:00"
        }
    }
    ],
    "start": "2016-10-03T00:00:00Z",
    "end": "2016-10-04T00:00:00Z"
}
```
## <a name="parameterizing-data-factory-template"></a>參數化 Data Factory 範本
如需參數化的最佳做法，請參閱[建立 Azure Resource Manager 範本的最佳做法](../../azure-resource-manager/resource-manager-template-best-practices.md)。 一般而言，參數的使用應該降到最低，特別是改為使用變數時。 在下列情況中，僅提供參數：

* 設定會因環境 (範例︰開發、測試和生產) 而不同
* 機密資料 (例如密碼)

如果您在使用範本部署 Azure Data Factory 實體時需要從 [Azure 金鑰保存庫](../../key-vault/key-vault-get-started.md)提取密碼，請指定 **金鑰保存庫**和**密碼名稱**，如下列範例所示︰

```JSON
"parameters": {
    "storageAccountKey": {
        "reference": {
            "keyVault": {
                "id":"/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.KeyVault/vaults/<keyVaultName>",
             },
            "secretName": "<secretName>"
           },
       },
       ...
}
```

> [!NOTE]
> 雖然目前尚未支援匯出現有資料處理站的範本，但是目前正在努力中。
>
>
