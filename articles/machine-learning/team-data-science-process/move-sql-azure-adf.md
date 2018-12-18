---
title: 使用 Azure Data Factory 從內部部署 SQL Server 將資料移至 SQL Azure | Microsoft Docs
description: 請設定 ADF 管線來編寫兩個資料移轉活動，這兩個活動會每天在內部部署及雲端中的資料庫之間一同移動資料。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 36837c83-2015-48be-b850-c4346aa5ae8a
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/04/2017
ms.author: deguhath
ms.openlocfilehash: f67bab33413b0048fc93d5f62e88984acaa25fd6
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393230"
---
# <a name="move-data-from-an-on-premises-sql-server-to-sql-azure-with-azure-data-factory"></a>使用 Azure Data Factory 從內部部署 SQL Server 將資料移至 SQL Azure
本主題說明如何使用 Azure Data Factory (ADF)，透過 Azure Blob 儲存體，將資料從內部部署的 SQL Server 資料庫移動至 SQL Azure 資料庫。

針對將資料移至 Azure SQL Database 的各種選項，如需摘要說明的資料表，請參閱[將資料移至 Azure Machine Learning 的 Azure SQL Database](move-sql-azure.md)。

## <a name="intro"></a>簡介：什麼是 ADF ，以及其應該用來移轉資料的時機？
Azure Data Factory 是完全受控的雲端架構資料整合服務，用來協調及自動化資料的移動和轉換。 ADF 模型中的重要概念是管線。 管線是活動的邏輯群組，各個群組都會定義包含在資料集中的資料上要執行的動作。 連結的服務是用來定義 Data Factory 所需的資訊，以便連接到資料資源。

使用 ADF，您可將現有的資料處理服務組合成具高可用性的資料管線，並在雲端中管理。 這些資料管線可排程來內嵌、準備、轉換、分析和發佈資料，而 ADF 會管理並協調複雜的資料和處理中的相依項目。 您可以快速地在雲端中建置和部署解決方案，藉此連接逐漸增加的內部部署和雲端資料來源。

請考慮使用 ADF：

* 若同時存取內部部署和雲端資源的混合式案例需要持續移轉資料
* 當資料進行交易、需要加以修改，或是在移轉期間新增了商務邏輯時。

ADF 允許使用定期管理資料移動的簡易 JSON 指令碼，來進行排程和監視的工作。 ADF 也有其他功能，例如支援複雜作業。 如需 ADF 的詳細資訊，請參閱 [Azure Data Factory (ADF)](https://azure.microsoft.com/services/data-factory/)上的文件。

## <a name="scenario"></a>案例
我們設定了 ADF 管線來組成兩個資料移轉活動。 它們每天都會一起在內部部署 SQL 資料庫和雲端 Azure SQL Database 之間移動資料。 這兩個活動為：

* 將資料從內部部署 SQL Server 資料庫複製到 Azure Blob 儲存體帳戶
* 將資料從 Azure Blob 儲存體帳戶複製至 Azure SQL Database

> [!NOTE]
> 這裡顯示的步驟已根據 ADF 團隊所提供的更詳細教學課程進行改編：[利用資料管理閘道在內部部署來源和雲端之間移動資料](../../data-factory/tutorial-hybrid-copy-portal.md)。該主題相關章節的參考資料也會在必要時提供。
>
>

## <a name="prereqs"></a>必要條件
本教學課程假設您有：

* **Azure 訂用帳戶**。 如果您沒有訂用帳戶，可以註冊 [免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure 儲存體帳戶**。 在本教學課程中，您會使用 Azure 儲存體帳戶來儲存資料。 如果您沒有 Azure 儲存體帳戶，請參閱 [建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md) 一文。 建立儲存體帳戶之後，您必須取得用來存取儲存體的帳戶金鑰。 請參閱[管理儲存體存取金鑰](../../storage/common/storage-account-manage.md#access-keys)。
* 存取 **Azure SQL Database**。 如果您必須設定 Azure SQL Database，[開始使用 Microsoft Azure SQL Database](../../sql-database/sql-database-get-started.md) 主題會提供如何佈建 Azure SQL Database 之新執行個體的相關資訊。
* 已在本機上安裝和設定 **Azure PowerShell** 。 如需指示，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

> [!NOTE]
> 此程序會使用 [Azure 入口網站](https://portal.azure.com/)。
>
>

## <a name="upload-data"></a> 將資料上傳至您的內部部署 SQL Server
我們會使用 [NYC 計程車資料集](http://chriswhong.com/open-data/foil_nyc_taxi/) 示範移轉程序。 NYC 計程車資料集可在 Azure Blob 儲存體 [NYC 計程車資料](http://www.andresmh.com/nyctaxitrips/)中取得 (如該文章所述)。 該資料有兩個檔案：包含路線詳細資料的 trip_data.csv 檔案，以及包含每次車程支付車資之詳細資料的 trip_far.csv 檔案。 這些檔案的範例和說明都會在 [NYC 計程車車程資料集說明](sql-walkthrough.md#dataset)中提供。

您可以將這裡提供的程序調整為自己的資料集，或者遵循上述步驟使用 NYC 計程車資料集。 若要將 NYC 計程車資料集上傳至您的內部部署 SQL Server 資料庫，請遵循[大量匯入資料至 SQL Server 資料庫](sql-walkthrough.md#dbload)中概述的程序進行。 這些指示適用於 Azure 虛擬機器上的 SQL Server，但將資料上傳至內部部署 SQL Server 的程序是相同的。

## <a name="create-adf"></a> 建立 Azure Data Factory
用於建立新 Azure Data Factory 的指示及 [Azure 入口網站](https://portal.azure.com/)中的資源群組，已在[建立 Azure Data Factory](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-data-factory) 提供。 將新的 ADF 執行個體命名為 *adfdsp*，並將建立的資源群組命名為 *adfdsprg*。

## <a name="install-and-configure-up-the-data-management-gateway"></a>安裝和設定資料管理閘道
若要在 Azure 資料處理站中啟用管線以使用內部部署的 SQL Server，您必須將其以連結服務形式新增至資料處理站。 若要建立內部部署 SQL Server 的連結服務，您必須︰

* 將 Microsoft 資料管理閘道下載並安裝到內部部署電腦。
* 設定內部部署資料來源的連結服務以使用閘道。

資料管理閘道器會序列化和還原序列化託管之電腦上的來源與接收資料。

如需關於資料管理閘道的設定指示及詳細資料，請參閱 [利用資料管理閘道在內部部署來源和雲端之間移動資料](../../data-factory/tutorial-hybrid-copy-portal.md)

## <a name="adflinkedservices"></a>建立連結服務以連接至資料資源
連結服務定義會定義 Azure Data Factory 所需的資訊，以便連接到資料資源。 此案例中的三個資源都必須使用連結服務：

1. 內部部署 SQL Server
2. Azure Blob 儲存體
3. Azure SQL Database

用於建立連結服務的逐步程序，已在[建立連結服務](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline)中提供。


## <a name="adf-tables"></a>定義和建立資料表以指定存取資料集的方式
使用下列指令碼型程序，建立指定資料集結構、位置及可用性的資料表。 JSON 檔案可用來定義資料表。 如需這些檔案結構的詳細資訊，請參閱 [資料集](../../data-factory/concepts-datasets-linked-services.md)。

> [!NOTE]
> 您應該先執行 `Add-AzureAccount` Cmdlet，再執行 [New-AzureDataFactoryTable](https://msdn.microsoft.com/library/azure/dn835096.aspx) Cmdlet，以確認已選取正確的 Azure 訂用帳戶來執行命令。 如需此 Cmdlet 的文件，請參閱 [Add-AzureAccount](/powershell/module/servicemanagement/azure/add-azureaccount?view=azuresmps-3.7.0)。
>
>

資料表中的 JSON 型定義使用下列名稱：

* 內部部署 SQL Server 中的**資料表名稱**為 *nyctaxi_data*
* Azure Blob 儲存體帳戶中的「容器名稱」  為 *containername*  

此 ADF 管線所需的三個資料表定義為：

1. [SQL 內部部署資料表](#adf-table-onprem-sql)
2. [Blob 資料表 ](#adf-table-blob-store)
3. [SQL Azure 資料表](#adf-table-azure-sql)

> [!NOTE]
> 這些程序使用 Azure PowerShell 來定義和建立 ADF 活動。 但是，這些工作也可以透過 Azure 入口網站來完成。 如需詳細資訊，請參閱[建立資料集](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline)。
>
>

### <a name="adf-table-onprem-sql"></a>SQL 內部部署資料表
內部部署 SQL Server 的資料表定義已在下列 JSON 檔案中指定：

        {
            "name": "OnPremSQLTable",
            "properties":
            {
                "location":
                {
                "type": "OnPremisesSqlServerTableLocation",
                "tableName": "nyctaxi_data",
                "linkedServiceName": "adfonpremsql"
                },
                "availability":
                {
                "frequency": "Day",
                "interval": 1,   
                "waitOnExternal":
                {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
                }

                }
            }
        }

這裡未包含資料行名稱。 您可以子選取資料行名稱，方法是將其包含在此 (如需詳細資料，請參閱 [ADF 文件](../../data-factory/copy-activity-overview.md) 主題)。

將資料表的 JSON 定義複製到名為 *onpremtabledef.json* 的檔案，並將其儲存至已知位置 (此處假設為 *C:\temp\onpremtabledef.json*)。 使用下列 Azure PowerShell Cmdlet，在 ADF 中建立資料表：

    New-AzureDataFactoryTable -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp –File C:\temp\onpremtabledef.json


### <a name="adf-table-blob-store"></a>Blob 資料表
輸出 Blob 位置的資料表定義如下 (此說明會將內嵌資料從內部部署對應至 Azure Blob)：

        {
            "name": "OutputBlobTable",
            "properties":
            {
                "location":
                {
                "type": "AzureBlobLocation",
                "folderPath": "containername",
                "format":
                {
                "type": "TextFormat",
                "columnDelimiter": "\t"
                },
                "linkedServiceName": "adfds"
                },
                "availability":
                {
                "frequency": "Day",
                "interval": 1
                }
            }
        }

將資料表的 JSON 定義複製到名為 *bloboutputtabledef.json* 的檔案，並將其儲存至已知位置 (此處假設為 *C:\temp\bloboutputtabledef.json*)。 使用下列 Azure PowerShell Cmdlet，在 ADF 中建立資料表：

    New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\bloboutputtabledef.json  

### <a name="adf-table-azure-sql"></a>SQL Azure 資料表
SQL Azure 輸出的資料表定義如下 (此結構描述會對應來自 Blob 的資料)：

    {
        "name": "OutputSQLAzureTable",
        "properties":
        {
            "structure":
            [
                { "name": "column1", type": "String"},
                { "name": "column2", type": "String"}                
            ],
            "location":
            {
                "type": "AzureSqlTableLocation",
                "tableName": "your_db_name",
                "linkedServiceName": "adfdssqlazure_linked_servicename"
            },
            "availability":
            {
                "frequency": "Day",
                "interval": 1            
            }
        }
    }

將資料表的 JSON 定義複製到名為 *AzureSqlTable.json* 的檔案，並將其儲存至已知位置 (此處假設為 *C:\temp\AzureSqlTable.json*)。 使用下列 Azure PowerShell Cmdlet，在 ADF 中建立資料表：

    New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\AzureSqlTable.json  


## <a name="adf-pipeline"></a>定義和建立管線
指定屬於管線的活動，並使用下列指令碼型程序建立管線。 JSON 檔案是用來定義管線屬性。

* 指令碼假設 **管線名稱** 為 *AMLDSProcessPipeline*。
* 此外請注意，我們會設定管線週期以每日執行，並使用預設的作業執行時間 (12 am UTC)。

> [!NOTE]
> 下列程序會使用 Azure PowerShell 來定義和建立 ADF 管線。 但是，此工作也可以透過 Azure 入口網站來完成。 如需詳細資訊，請參閱[建立管線](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline)。
>
>

使用先前提供的資料表定義，將 ADF 的管線定義指定為：

        {
            "name": "AMLDSProcessPipeline",
            "properties":
            {
                "description" : "This pipeline has one Copy activity that copies data from an on-premises SQL to Azure blob",
                 "activities":
                [
                    {
                        "name": "CopyFromSQLtoBlob",
                        "description": "Copy data from on-premises SQL server to blob",     
                        "type": "CopyActivity",
                        "inputs": [ {"name": "OnPremSQLTable"} ],
                        "outputs": [ {"name": "OutputBlobTable"} ],
                        "transformation":
                        {
                            "source":
                            {                               
                                "type": "SqlSource",
                                "sqlReaderQuery": "select * from nyctaxi_data"
                            },
                            "sink":
                            {
                                "type": "BlobSink"
                            }   
                        },
                        "Policy":
                        {
                            "concurrency": 3,
                            "executionPriorityOrder": "NewestFirst",
                            "style": "StartOfInterval",
                            "retry": 0,
                            "timeout": "01:00:00"
                        }       

                     },

                    {
                        "name": "CopyFromBlobtoSQLAzure",
                        "description": "Push data to Sql Azure",        
                        "type": "CopyActivity",
                        "inputs": [ {"name": "OutputBlobTable"} ],
                        "outputs": [ {"name": "OutputSQLAzureTable"} ],
                        "transformation":
                        {
                            "source":
                            {                               
                                "type": "BlobSource"
                            },
                            "sink":
                            {
                                "type": "SqlSink",
                                "WriteBatchTimeout": "00:5:00",                
                            }            
                        },
                        "Policy":
                        {
                            "concurrency": 3,
                            "executionPriorityOrder": "NewestFirst",
                            "style": "StartOfInterval",
                            "retry": 2,
                            "timeout": "02:00:00"
                        }
                     }
                ]
            }
        }

將該管線的 JSON 定義複製到名為 *pipelinedef.json* 的檔案，並將其儲存至已知位置 (此處假設為 *C:\temp\pipelinedef.json*)。 使用下列 Azure PowerShell Cmdlet，在 ADF 中建立管線：

    New-AzureDataFactoryPipeline  -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\pipelinedef.json


## <a name="adf-pipeline-start"></a>啟動管線
現在可使用下列命令來執行管線：

    Set-AzureDataFactoryPipelineActivePeriod -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp -StartDateTime startdateZ –EndDateTime enddateZ –Name AMLDSProcessPipeline

*startdate* 和 *enddate* 參數值必須替換為您想要執行管線的實際日期。

當管線執行時，您應該可以看到資料在選取用於 Blob 的容器中顯示 (每天一個檔案)。

請注意，我們尚未運用 ADF 提供的功能，以遞增方式輸送資料。 如需如何執行此功能和 ADF 提供之其他功能的詳細資訊，請參閱 [ADF 文件](https://azure.microsoft.com/services/data-factory/)。
