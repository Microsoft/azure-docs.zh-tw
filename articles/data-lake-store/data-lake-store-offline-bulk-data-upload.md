---
title: 使用離線方法將大量資料上傳到 Azure Data Lake Storage Gen1 | Microsoft Docs
description: 使用 AdlCopy 工具將資料從 Azure 儲存體 Blob 複製到 Azure Data Lake Storage Gen1
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: 45321f6a-179f-4ee4-b8aa-efa7745b8eb6
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 6430bf524ac81af242bf7afb4c2c8196309806ab
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391670"
---
# <a name="use-the-azure-importexport-service-for-offline-copy-of-data-to-azure-data-lake-storage-gen1"></a>使用 Azure 匯入/匯出服務將資料離線複製到 Azure Data Lake Storage Gen1
在本文中，您將深入了解如何使用離線複製方法 (例如 [Azure 匯入/匯出服務](../storage/common/storage-import-export-service.md)) 將大型資料集 (>200 GB) 複製到 Azure Data Lake Storage Gen1 中。 具體來說，作為本文中範例的檔案是 339,420,860,416 個位元組，或在磁碟上大約是 319 GB。 讓我將此檔案稱為 319GB.tsv。

Azure 匯入/匯出服務可讓您將硬碟運送到 Azure 資料中心，更安全地傳輸大量資料至 Azure Blob 儲存體。

## <a name="prerequisites"></a>必要條件
開始之前，您必須具備下列條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure 儲存體帳戶**。
* **Azure Data Lake Storage Gen1 帳戶**。 如需如何建立帳戶的指示，請參閱[開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)

## <a name="preparing-the-data"></a>準備資料

開始使用「匯入/匯出服務」之前，請將要傳輸的資料檔分割成大小**小於 200 GB 的複本**。 匯入工具不適用於大於 200 GB 的檔案。 在本教學課程中，我們將檔案分割成每個 100 GB 的區塊。 您可以使用 [Cygwin](https://cygwin.com/install.html) 來達成此目的。 Cygwin 支援 Linux 命令。 在此情況下，請使用下列命令：

    split -b 100m 319GB.tsv

分割作業會建立帶有以下名稱的檔案。

    319GB.tsv-part-aa

    319GB.tsv-part-ab

    319GB.tsv-part-ac

    319GB.tsv-part-ad

## <a name="get-disks-ready-with-data"></a>備妥資料磁碟
依照[使用 Azure 匯入/匯出服務](../storage/common/storage-import-export-service.md)的指示 (在**準備磁碟機**一節底下) 來準備您的硬碟。 以下是整體順序︰

1. 取得符合 Auzre 匯入/匯出服務使用需求的硬碟。
2. 識別當資料被送至 Azure 資料中心時，將用來複製資料的 Azure 儲存體帳戶。
3. 使用 [Azure 匯入/匯出工具](http://go.microsoft.com/fwlink/?LinkID=301900&clcid=0x409)，這是一個命令列公用程式。 以下是一個有關如何使用該工具的簡單程式碼片段。

    ````
    WAImportExport PrepImport /sk:<StorageAccountKey> /t: <TargetDriveLetter> /format /encrypt /logdir:e:\myexportimportjob\logdir /j:e:\myexportimportjob\journal1.jrn /id:myexportimportjob /srcdir:F:\demo\ExImContainer /dstdir:importcontainer/vf1/
    ````
    如需更多範例程式碼片段，請參閱[使用 Azure 匯入/匯出服務](../storage/common/storage-import-export-service.md)。
4. 前述命令會在指定的位置建立日誌檔案。 使用此日誌檔案從 [Azure 入口網站](https://portal.azure.com)建立匯入作業。

## <a name="create-an-import-job"></a>建立匯入作業
您現在可以依照[使用 Azure 匯入/匯出服務](../storage/common/storage-import-export-service.md)的指示 (在**準備磁碟機**一節底下) 來建立匯入作業。 針對此匯入作業，除了其他詳細資料之外，也請提供在準備磁碟機時建立的日誌檔。

## <a name="physically-ship-the-disks"></a>實際寄送磁碟
您現在可以將磁碟實際寄送至 Azure 資料中心。 在此，資料將在這裡複製到您建立匯入作業時所提供的 Azure 儲存體 Blob。 此外，如果您在建立作業時選擇了稍後提供追蹤資訊，您現在便可返回您的匯入作業並更新追蹤號碼。

## <a name="copy-data-from-azure-storage-blobs-to-azure-data-lake-storage-gen1"></a>將資料從 Azure 儲存體 Blob 複製到 Azure Data Lake Storage Gen1
匯入作業的狀態顯示為完成之後，您可以確認您指定的 Azure 儲存體 Blob 中是否有該資料。 接下來，您可以使用各種方法將該資料從 Blob 移至 Azure Data Lake Storage Gen1。 如需所有可用的上傳資料選項，請參閱[將資料內嵌到 Azure Data Lake Storage Gen1](data-lake-store-data-scenarios.md#ingest-data-into-data-lake-storage-gen1)。

在本節中，我們會提供您可用來建立 Azure Data Factory 管線以供複製資料的 JSON 定義。 您可以從 [Azure 入口網站](../data-factory/tutorial-copy-data-portal.md)或 [Visual Studio](../data-factory/tutorial-copy-data-dot-net.md) 使用這些 JSON 定義。

### <a name="source-linked-service-azure-storage-blob"></a>來源連結服務 (Azure 儲存體 Blob)
````
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "description": "",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
````

### <a name="target-linked-service-azure-data-lake-storage-gen1"></a>目標連結服務 (Azure Data Lake Storage Gen1)
````
{
    "name": "AzureDataLakeStorageGen1LinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "description": "",
        "typeProperties": {
            "authorization": "<Click 'Authorize' to allow this data factory and the activities it runs to access this Data Lake Storage Gen1 account with your access rights>",
            "dataLakeStoreUri": "https://<adlsg1_account_name>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<OAuth session id from the OAuth authorization session. Each session id is unique and may only be used once>"
        }
    }
}
````
### <a name="input-data-set"></a>輸入資料集
````
{
    "name": "InputDataSet",
    "properties": {
        "published": false,
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "importcontainer/vf1/"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
````
### <a name="output-data-set"></a>輸出資料集
````
{
"name": "OutputDataSet",
"properties": {
  "published": false,
  "type": "AzureDataLakeStore",
  "linkedServiceName": "AzureDataLakeStorageGen1LinkedService",
  "typeProperties": {
    "folderPath": "/importeddatafeb8job/"
    },
  "availability": {
    "frequency": "Hour",
    "interval": 1
    }
  }
}
````
### <a name="pipeline-copy-activity"></a>管線 (複製活動)
````
{
    "name": "CopyImportedData",
    "properties": {
        "description": "Pipeline with copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "AzureDataLakeStoreSink",
                        "copyBehavior": "PreserveHierarchy",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity"
            }
        ],
        "start": "2016-02-08T22:00:00Z",
        "end": "2016-02-08T23:00:00Z",
        "isPaused": false,
        "pipelineMode": "Scheduled"
    }
}
````
如需詳細資訊，請參閱[使用 Azure Data Factory 從 Azure Data Lake Storage Gen1 來回複製資料](../data-factory/connector-azure-data-lake-store.md)。

## <a name="reconstruct-the-data-files-in-azure-data-lake-storage-gen1"></a>在 Azure Data Lake Storage Gen1 中重新建構資料檔
我們從 319 GB 的檔案開始著手並將它分割成數個較小的檔案，以便使用 Azure 匯入/匯出服務來傳輸它。 現在，該資料在 Azure Data Lake Storage Gen1 中，我們可以將檔案重新建構成其原始大小。 您可以使用下列 Azure PowerShell Cmdlet 來進行。

```
# Login to our account
Connect-AzureRmAccount

# List your subscriptions
Get-AzureRmSubscription

# Switch to the subscription you want to work with
Set-AzureRmContext -SubscriptionId
Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"

# Join  the files
Join-AzureRmDataLakeStoreItem -AccountName "<adlsg1_account_name" -Paths "/importeddatafeb8job/319GB.tsv-part-aa","/importeddatafeb8job/319GB.tsv-part-ab", "/importeddatafeb8job/319GB.tsv-part-ac", "/importeddatafeb8job/319GB.tsv-part-ad" -Destination "/importeddatafeb8job/MergedFile.csv"
````

## <a name="next-steps"></a>後續步驟
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
* [搭配 Data Lake Storage Gen1 使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [搭配 Data Lake Storage Gen1 使用 Azure HDInsight](data-lake-store-hdinsight-hadoop-use-portal.md)
