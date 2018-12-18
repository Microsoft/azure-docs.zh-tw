---
title: 使用 Azure Databricks Python 來轉換資料 | Microsoft Docs
description: 了解如何藉由執行 Databricks Python 來處理或轉換資料。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.assetid: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/15/2018
ms.author: douglasl
ms.openlocfilehash: 17a8e6f6d6d374c6f8620ecb525727e6fee8c4b9
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39501872"
---
# <a name="transform-data-by-running-a-python-activity-in-azure-databricks"></a>在 Azure Databricks 中執行 Python 活動來轉換資料

[Data Factory 管線](concepts-pipelines-activities.md)中的 Azure Databricks Python 活動會在 Azure Databricks 叢集中執行 Python 檔案。 本文是根據 [資料轉換活動](transform-data.md) 一文，它呈現資料轉換和支援的轉換活動的一般概觀。 Azure Databricks 是用於執行 Apache Spark 的受控平台。

如需此功能的簡介與示範，請觀看下列 11 分鐘長的影片：

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Execute-Jars-and-Python-scripts-on-Azure-Databricks-using-Data-Factory/player]

## <a name="databricks-python-activity-definition"></a>Databricks Python 活動定義

以下是 Databricks Python 活動的 JSON 定義範例：

```json
{
    "activity": {
        "name": "MyActivity",
        "description": "MyActivity description",
        "type": "DatabricksSparkPython",
        "linkedServiceName": {
            "referenceName": "MyDatabricksLinkedservice",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "pythonFile": "dbfs:/docs/pi.py",
            "parameters": [
                "10"
            ],
            "libraries": [
                {
                    "pypi": {
                        "package": "tensorflow"
                    }
                }
            ]
        }
    }
}
```

## <a name="databricks-python-activity-properties"></a>Databricks Python 活動屬性

下表說明 JSON 定義中使用的 JSON 屬性：

|屬性|說明|必要|
|---|---|---|
|name|管線中的活動名稱。|是|
|說明|說明活動用途的文字。|否|
|type|若是 Databricks Python 活動，則活動類型是 DatabricksSparkPython。|是|
|預設容器|Python 活動執行所在之 Databricks 連結服務的名稱。 若要深入了解此已連結的服務，請參閱[計算已連結的服務](compute-linked-services.md)一文。|是|
|pythonFile|要執行之 Python 檔案的 URI。 只支援 DBFS 路徑。|是|
|parameters|將傳遞至 Python 檔案的命令列參數。 這是字串陣列。|否|
|程式庫|要在負責執行工作的叢集上，即將安裝的程式庫清單。 可以是 < 字串, 物件 > 陣列|否|

## <a name="supported-libraries-for-databricks-activities"></a>Databricks 活動支援的程式庫

在上述的 Databricks 活動定義中，您指定以下的程式庫類型： *jar*、*egg*、 *maven*、 *pypi*、 *cran*。

```json
{
    "libraries": [
        {
            "jar": "dbfs:/mnt/libraries/library.jar"
        },
        {
            "egg": "dbfs:/mnt/libraries/library.egg"
        },
        {
            "maven": {
                "coordinates": "org.jsoup:jsoup:1.7.2",
                "exclusions": [ "slf4j:slf4j" ]
            }
        },
        {
            "pypi": {
                "package": "simplejson",
                "repo": "http://my-pypi-mirror.com"
            }
        },
        {
            "cran": {
                "package": "ada",
                "repo": "http://cran.us.r-project.org"
            }
        }
    ]
}

```

如需詳細資訊，請參閱 [Databricks 文件](https://docs.azuredatabricks.net/api/latest/libraries.html#managedlibrarieslibrary) \(英文\)，了解程式庫類型。

## <a name="how-to-upload-a-library-in-databricks"></a>如何在 Databricks 中上傳程式庫

#### <a name="using-databricks-workspace-uihttpsdocsazuredatabricksnetuser-guidelibrarieshtmlcreate-a-library"></a>[使用 Databricks 工作區 UI](https://docs.azuredatabricks.net/user-guide/libraries.html#create-a-library) \(英文\)

若要取得利用 UI 新增之程式庫的 dbfs 路徑，您可以使用 [Databricks CLI (安裝)](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#install-the-cli) \(英文\)。 

使用 UI 時，Jar 程式庫通常會儲存在 dbfs: FileStore/jar。 您可以透過 CLI 來列出一切MM： *databricks fs ls dbfs: / FileStore/jars* 



#### <a name="copy-library-using-databricks-clihttpsdocsazuredatabricksnetuser-guidedev-toolsdatabricks-clihtmlcopy-a-file-to-dbfs"></a>[使用 Databricks CLI 來複製程式庫](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#copy-a-file-to-dbfs) \(英文\)

範例：*databricks fs cp SparkPi-assembly-0.1.jar dbfs:/FileStore/jars*