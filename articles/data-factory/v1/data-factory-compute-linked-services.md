---
title: Azure Data Factory 支援的計算環境 | Microsoft Docs
description: 了解您可以在 Azure Data Factory 管線中 (如 Azure HDInsight) 用來轉換/處理資料的計算環境。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 6877a7e8-1a58-4cfb-bbd3-252ac72e4145
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 59631ee0115c817da1b0588c1ad37d2f8b34db67
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37053645"
---
# <a name="compute-environments-supported-by-azure-data-factory"></a>Azure Data Factory 支援的計算環境
> [!NOTE]
> 本文適用於第 1 版的 Azure Data Factory。 如果您使用目前版本的 Data Factory 服務，請參閱[計算連結服務](../compute-linked-services.md)。

本文說明您可用來處理或轉換資料的計算環境。 本文還提供在設定連結服務以將這些計算環境連結至 Azure Data Factory 時，Data Factory 所支援之不同設定 (隨選與自備) 的詳細資料。

下表列出 Data Factory 所支援的計算環境，以及可在這些環境上執行的活動。 

| 計算環境                      | 活動                               |
| ---------------------------------------- | ---------------------------------------- |
| [隨選 Azure HDInsight 叢集](#azure-hdinsight-on-demand-linked-service)或[您自己的 HDInsight 叢集](#azure-hdinsight-linked-service) | [DotNet](data-factory-use-custom-activities.md)、[Hive](data-factory-hive-activity.md)、[Pig](data-factory-pig-activity.md)、[MapReduce](data-factory-map-reduce.md)、[Hadoop 串流](data-factory-hadoop-streaming-activity.md) |
| [Azure Batch](#azure-batch-linked-service) | [DotNet](data-factory-use-custom-activities.md) |
| [Azure Machine Learning](#azure-machine-learning-linked-service) | [Machine Learning 活動︰批次執行和更新資源](data-factory-azure-ml-batch-execution-activity.md) |
| [Azure 資料湖分析](#azure-data-lake-analytics-linked-service) | [Data Lake Analytics U-SQL](data-factory-usql-activity.md) |
| [Azure SQL](#azure-sql-linked-service)、[Azure SQL 資料倉儲](#azure-sql-data-warehouse-linked-service)、[SQL Server](#sql-server-linked-service) | [預存程序活動](data-factory-stored-proc-activity.md) |

## <a name="supported-hdinsight-versions-in-azure-data-factory"></a>Data Factory 中支援的 HDInsight 版本
Azure HDInsight 支援多個可供您隨時部署的 Hadoop 叢集版本。 每一個支援版本都會建立特定版本的 Hortonworks Data Platform (HDP) 散發套件，以及該散發套件的一組元件。 

Microsoft 會更新具有最新 Hadoop 生態系統元件和修正程式的 HDInsight 支援版本清單。 如需詳細資訊，請參閱[支援的 HDInsight 版本](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions)。

> [!IMPORTANT]
> 以 Linux 為基礎的 HDInsight 3.3 版已於 2017 年 7 月 31 日淘汰。 我們已讓 Data Factory 第 1 版隨選 HDInsight 連結服務客戶在 2017 年 12 月 15 日之前測試並升級到較新版的 HDInsight。 以 Windows 為基礎的 HDInsight 則將於 2018 年 7 月 31 日淘汰。
>
> 

### <a name="after-the-retirement-date"></a>過了淘汰日期之後 

在 2017 年 12 月 15 日之後：

- 您無法再使用 Data Factory 第 1 版中的隨選 HDInsight 連結服務，建立以 Linux 為基礎的 HDInsight 3.3 版 (或較舊版本) 叢集。 
- 如果沒有在現有 Data Factory 第 1 版隨選 HDInsight 連結服務之 JSON 定義中明確指定 [**osType** 和 **Version** 屬性 ](https://docs.microsoft.com/azure/data-factory/v1/data-factory-compute-linked-services#azure-hdinsight-on-demand-linked-service)，預設值將會從 **Version=3.1, osType=Windows** 變更為 **Version=\<最新 HDI 預設版本\>(https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#hadoop-components-available-with-different-hdinsight-versions), osType=Linux**。

在 2018 年 7 月 31 日之後：

- 您無法再使用 Data Factory 第 1 版中的隨選 HDInsight 連結服務，建立任何版本的 Windows 架構 HDInsight 叢集。 

### <a name="recommended-actions"></a>建議動作 

- 為確保您可以使用最新的 Hadoop 生態系統元件和修正程式，請將受影響之 Data Factory 第 1 版隨選 HDInsight 連結服務定義的 [**osType** 和 **Version** 屬性](https://docs.microsoft.com/azure/data-factory/v1/data-factory-compute-linked-services#azure-hdinsight-on-demand-linked-service)，更新為較新版之以 Linux 為基礎的 HDInsight (HDInsight 3.6)。 
- 在 2017 年 12 月 15 日之前，對參考受影響之連結服務的 Data Factory 第 1 版 Hive、Pig、MapReduce 和 Hadoop 串流活動進行測試。 請確定這些活動能與新的 **osType** 和 **Version** 預設值 (**Version=3.6**, **osType=Linux**) 或您要升級到的明確 HDInsight 版本和 OS 類型相容。 
  若要深入了解相容性，請參閱[從以 Windows 為基礎的 HDInsight 叢集移轉至以 Linux 為基礎的叢集](https://docs.microsoft.com/azure/hdinsight/hdinsight-migrate-from-windows-to-linux)和[可以搭配 HDInsight 使用的 Hadoop 元件和版本有哪些？](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#hortonworks-release-notes-associated-with-hdinsight-versions)。 
- 若要繼續使用 Data Factory 第 1 版隨選 HDInsight 連結服務來建立以 Windows 為基礎的 HDInsight 叢集，請於 2017 年 12 月 15 日之前將 **osType** 明確設定為 **Windows**。 我們的建議是在 2018 年 7 月 31 日之前移轉至以 Linux 為基礎的 HDInsight 叢集。 
- 如果您要使用隨選 HDInsight 連結服務來執行 Data Factory 第 1 版 DotNet 自訂活動，請將 DotNet 自訂活動 JSON 定義更新成改為使用 Azure Batch 連結服務。 如需詳細資訊，請參閱[在 Data Factory 管線中使用自訂活動](https://docs.microsoft.com/azure/data-factory/v1/data-factory-use-custom-activities)。 

> [!Note]
> 如果您使用現有的 Data Factory 第 1 版自備叢集 HDInsight 連結裝置，或 Azure Data Factory 自備和隨選 HDInsight 連結服務，則不必採取任何動作。 這些案例已強制執行最新版的 HDInsight 叢集支援原則。 
>
> 


## <a name="on-demand-compute-environment"></a>隨選計算環境
在隨選設定中，Data Factory 會對計算環境進行完全的管理。 Data Factory 會先自動建立計算環境，再提交作業來處理資料。 當作業完成時，Data Factory 便會移除計算環境。 

您可以對隨選計算環境建立連結服務。 請使用連結服務來設定計算環境，以及控制作業執行、叢集管理和啟動動作的細微設定。

> [!NOTE]
> 隨選設定目前只支援供 HDInsight 叢集使用。
> 

## <a name="azure-hdinsight-on-demand-linked-service"></a>Azure HDInsight 隨選連結服務
Data Factory 可自動建立以 Windows 為基礎或以 Linux 為基礎的隨選 HDInsight 叢集來處理資料。 叢集建立所在的區域，就是與叢集相關聯之儲存體帳戶所在的區域。 請使用 JSON **linkedServiceName** 屬性來建立叢集。

請注意下列有關隨選 HDInsight 連結服務的重點：

* Azure 訂用帳戶中不會出現隨選 HDInsight 叢集。 Data Factory 服務會代您管理隨選 HDInsight 叢集。
* 系統會將隨選 HDInsight 叢集上所執行之作業的記錄，複製到與 HDInsight 叢集相關聯的儲存體帳戶。 若要存取這些記錄，請在 Azure 入口網站中移至 [活動執行詳細資料] 窗格。 如需詳細資訊，請參閱[監視與管理管線](data-factory-monitor-manage-pipelines.md)。
* 您只需要就 HDInsight 叢集啟動並執行作業的時間來付費。

> [!IMPORTANT]
> 隨選 HDInsight 叢集通常需要 *20 分鐘*以上的時間才能佈建完成。
>
> 

### <a name="example"></a>範例
下列 JSON 會定義以 Linux 為基礎的隨選 HDInsight 連結服務。 Data Factory 會在處理資料配量時自動建立*以 Linux 為基礎的* HDInsight 叢集。 

```json
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.6",
            "osType": "Linux",
            "clusterSize": 1,
            "timeToLive": "00:05:00",            
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

> [!IMPORTANT]
> HDInsight 叢集會在您於 JSON **linkedServiceName** 屬性中指定的 Azure Blob 儲存體內建立「預設容器」。 根據設計，HDInsight 不會在刪除叢集時刪除此容器。 除非隨選 HDInsight 連結服務中有現有的即時叢集 (**timeToLive**)，否則每當有需要處理的配量時，系統就會建立 HDInsight 叢集。 當處理完成時，系統就會刪除此叢集。 
>
> 隨著處理的配量越來越多，您會在 Blob 儲存體中看到許多容器。 如果在對作業進行疑難排解時不需要這些容器，建議您加以刪除以降低儲存成本。 這些容器的名稱會遵循模式︰`adf<your Data Factory name>-<linked service name>-<date and time>`。 您可以使用 [Microsoft 儲存體總管](http://storageexplorer.com/) 之類的工具來刪除 Blob 儲存體中的容器。
>
> 

### <a name="properties"></a>properties
| 屬性                     | 說明                              | 必要 |
| ---------------------------- | ---------------------------------------- | -------- |
| type                         | 將 type 屬性設定為 **HDInsightOnDemand**。 | yes      |
| clusterSize                  | 叢集中背景工作角色和資料節點的數目。 除了您為此屬性指定的背景工作節點數目外，所建立的 HDInsight 叢集還會有 2 個前端節點。 節點的大小為 Standard_D3，具有 4 個核心。 4 個背景工作節點的叢集會使用 24 個核心 (4\*4 = 16 個核心用於背景工作節點，加上 2\*4 = 8 個核心用於前端節點)。 如需 Standard_D3 層的詳細資料，請參閱[在 HDInsight 中建立以 Linux 為基礎的 Hadoop 叢集](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md)。 | yes      |
| timeToLive                   | 隨選 HDInsight 叢集允許的閒置時間。 指定當活動執行完成時，如果叢集中沒有其他作用中的作業，隨選 HDInsight 叢集要保持運作多久。<br /><br />例如，如果活動執行花費 6 分鐘，而 **timeToLive** 設為 5 分鐘，叢集會在花費 6 分鐘處理活動執行之後保持運作 5 分鐘。 如果 6 分鐘期間內有另一個活動執行需要執行，則會由同一個叢集來處理。<br /><br />建立隨選 HDInsight 叢集的作業成本很高 (因為可能需要一些時間)。 請視需要使用此設定，藉由重複使用隨選 HDInsight 叢集來提升資料處理站的效能。<br /><br />如果您將 **timeToLive** 值設定為 **0**，則系統會在活動執行完成後立即刪除叢集。 不過，如果您設定較高的值，叢集可能會有不必要的閒置而導致成本提高。 請務必根據您的需要設定適當的值。<br /><br />如果 **timeToLive** 值有適當設定，多個管線便可以共用隨選 HDInsight 叢集的執行個體。 | yes      |
| version                      | HDInsight 叢集的版本。 如需允許的 HDInsight 版本，請參閱[支援的 HDInsight 版本](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#supported-hdinsight-versions)。 如果未指定此值，則會使用[最新的 HDI 預設版本](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#hadoop-components-available-with-different-hdinsight-versions)。 | 否       |
| 預設容器            | 隨選叢集用於儲存及處理資料的 Azure 儲存體連結服務。 HDInsight 叢集會和這個儲存體帳戶建立在相同的區域。<p>您目前無法建立以 Azure Data Lake Store 作為儲存體的隨選 HDInsight 叢集。 如果您想要在 Data Lake Store 中儲存 HDInsight 處理的結果資料，請使用複製活動將 Blob 儲存體的資料複製到 Data Lake Store。 </p> | yes      |
| additionalLinkedServiceNames | 指定 HDInsight 連結服務的其他儲存體帳戶。 Data Factory 會代您註冊儲存體帳戶。 這些儲存體帳戶必須和 HDInsight 叢集位於同一個區域。 HDInsight 叢集建立所在的區域，和 **linkedServiceName** 屬性所指定之儲存體帳戶所在的區域相同。 | 否       |
| osType                       | 作業系統的類型。 允許的值為 **Linux** 和 **Windows**。 如果未指定此值，則會使用 **Linux**。  <br /><br />強烈建議您使用以 Linux 為基礎的 HDInsight 叢集。 Windows 上 HDInsight 的停用日期是 2018 年 7 月 31 日。 | 否       |
| hcatalogLinkedServiceName    | 指向 HCatalog 資料庫的 Azure SQL 連結服務名稱。 系統會以 SQL 資料庫作為中繼存放區來建立隨選 HDInsight 叢集。 | 否       |

#### <a name="example-linkedservicenames-json"></a>範例：LinkedServiceNames JSON

```json
"additionalLinkedServiceNames": [
    "otherLinkedServiceName1",
    "otherLinkedServiceName2"
  ]
```

### <a name="advanced-properties"></a>進階屬性
如需隨選 HDInsight 叢集的細微設定，請指定下列屬性：

| 屬性               | 說明                              | 必要 |
| :--------------------- | :--------------------------------------- | :------- |
| coreConfiguration      | 指定要用來建立 HDInsight 叢集的核心設定參數 (core-site.xml)。 | 否       |
| hBaseConfiguration     | 指定 HDInsight 叢集的 HBase 組態參數 (hbase-site.xml)。 | 否       |
| hdfsConfiguration      | 指定 HDInsight 叢集的 HDFS 組態參數 (hdfs-site.xml)。 | 否       |
| hiveConfiguration      | 指定 HDInsight 叢集的 Hive 設定參數 (hive-site.xml)。 | 否       |
| mapReduceConfiguration | 指定 HDInsight 叢集的 MapReduce 組態參數 (mapred-site.xml)。 | 否       |
| oozieConfiguration     | 指定 HDInsight 叢集的 Oozie 組態參數 (oozie-site.xml)。 | 否       |
| stormConfiguration     | 指定 HDInsight 叢集的 Storm 組態參數 (storm-site.xml)。 | 否       |
| yarnConfiguration      | 指定 HDInsight 叢集的 YARN 設定參數 (yarn-site.xml)。 | 否       |

#### <a name="example-on-demand-hdinsight-cluster-configuration-with-advanced-properties"></a>範例：含有進階屬性的隨選 HDInsight 叢集設定

```json
{
  "name": " HDInsightOnDemandLinkedService",
  "properties": {
    "type": "HDInsightOnDemand",
    "typeProperties": {
      "version": "3.6",
      "osType": "Linux",
      "clusterSize": 16,
      "timeToLive": "01:30:00",
      "linkedServiceName": "adfods1",
      "coreConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "hiveConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "mapReduceConfiguration": {
        "mapreduce.reduce.java.opts": "-Xmx4000m",
        "mapreduce.map.java.opts": "-Xmx4000m",
        "mapreduce.map.memory.mb": "5000",
        "mapreduce.reduce.memory.mb": "5000",
        "mapreduce.job.reduce.slowstart.completedmaps": "0.8"
      },
      "yarnConfiguration": {
        "yarn.app.mapreduce.am.resource.mb": "5000",
        "mapreduce.map.memory.mb": "5000"
      },
      "additionalLinkedServiceNames": [
        "datafeeds",
        "adobedatafeed"
      ]
    }
  }
}
```

### <a name="node-sizes"></a>節點大小
若要指定前端、資料和 ZooKeeper 節點的大小，請使用下列屬性： 

| 屬性          | 說明                              | 必要 |
| :---------------- | :--------------------------------------- | :------- |
| headNodeSize      | 設定前端節點的大小。 預設值為 **Standard_D3**。 如需詳細資料，請參閱[指定節點大小](#specify-node-sizes)。 | 否       |
| dataNodeSize      | 設定資料節點的大小。 預設值為 **Standard_D3**。 | 否       |
| zookeeperNodeSize | 設定 ZooKeeper 節點的大小。 預設值為 **Standard_D3**。 | 否       |

#### <a name="specify-node-sizes"></a>指定節點大小
如需必須為上一節所述屬性指定的字串值，請參閱[虛擬機器大小](../../virtual-machines/linux/sizes.md)。 這些值必須符合[虛擬機器大小](../../virtual-machines/linux/sizes.md)中所指出的 Cmdlet 和 API。 大型 (預設) 資料節點大小有 7 GB 記憶體。 這可能不足以供您的案例使用。 

如果您要建立 D4 大小的前端節點與背景工作節點，請指定 **Standard_D4** 作為 **headNodeSize** 與 **dataNodeSize** 屬性的值： 

```json
"headNodeSize": "Standard_D4",    
"dataNodeSize": "Standard_D4",
```

如果為這些屬性設定的值不正確，您可能會看到下列訊息：

  無法建立叢集。 例外狀況：無法完成叢集建立作業。 作業失敗，錯誤碼為 '400'。 叢集剩餘狀態：「錯誤」。 訊息：「PreClusterCreationValidationFailure」。 
  
如果您看到此訊息，請確定您所使用的 Cmdlet 和 API 名稱是來自[虛擬機器大小](../../virtual-machines/linux/sizes.md)中的資料表。  

> [!NOTE]
> Data Factory 目前不支援以 Data Lake Store 作為主要存放區的 HDInsight 叢集。 請以 Azure 儲存體作為 HDInsight 叢集的主要存放區。 
>
> 


## <a name="bring-your-own-compute-environment"></a>自備計算環境
您可以在 Data Factory 中將現有計算環境註冊為連結服務。 您必須管理計算環境。 Data Factory 服務會使用計算環境來執行活動。

下列計算環境可支援這類型的組態：

* Azure HDInsight
* Azure Batch
* Azure Machine Learning
* Azure Data Lake Analytics
* Azure SQL Database、Azure SQL 資料倉儲、SQL Server

## <a name="azure-hdinsight-linked-service"></a>Azure HDInsight 連結服務
您可以建立 HDInsight 連結服務，以向 Data Factory 註冊自己的 HDInsight 叢集。

### <a name="example"></a>範例

```json
{
  "name": "HDInsightLinkedService",
  "properties": {
    "type": "HDInsight",
    "typeProperties": {
      "clusterUri": " https://<hdinsightclustername>.azurehdinsight.net/",
      "userName": "admin",
      "password": "<password>",
      "linkedServiceName": "MyHDInsightStoragelinkedService"
    }
  }
}
```

### <a name="properties"></a>properties
| 屬性          | 說明                              | 必要 |
| ----------------- | ---------------------------------------- | -------- |
| type              | 將 type 屬性設定為 **HDInsight**。 | yes      |
| clusterUri        | HDInsight 叢集的 URI。        | yes      |
| username          | 要用來連線到現有 HDInsight 叢集的使用者帳戶名稱。 | yes      |
| password          | 使用者帳戶的密碼。   | yes      |
| 預設容器 | 參照 HDInsight 叢集所使用之 Blob 儲存體的儲存體連結服務名稱。 <p>您目前無法針對此屬性指定 Data Lake Store 連結服務。 如果 HDInsight 叢集可存取 Data Lake Store，您可以透過 Hive 或 Pig 指令碼存取 Data Lake Store 中的資料。 </p> | yes      |

## <a name="azure-batch-linked-service"></a>Azure Batch 已連結的服務
您可以建立 Batch 連結服務，以向資料處理站註冊虛擬機器 (VM) 的 Batch 集區。 您可以使用 Batch 或 HDInsight 來執行 Microsoft .NET 自訂活動。

如果您不熟悉如何使用 Batch 服務：

* 了解 [Azure Batch 基本概念](../../batch/batch-technical-overview.md)。
* 了解 [New-AzureBatchAccount](https://msdn.microsoft.com/library/mt125880.aspx) Cmdlet。 請使用這個 Cmdlet 建立 Batch 帳戶。 您也可以使用 [Azure 入口網站](../../batch/batch-account-create-portal.md)建立 Batch 帳戶。 如需如何使用此 Cmdlet 的詳細資訊，請參閱[使用 PowerShell 管理 Batch 帳戶](http://blogs.technet.com/b/windowshpc/archive/2014/10/28/using-azure-powershell-to-manage-azure-batch-account.aspx)。
* 了解 [New-AzureBatchPool](https://msdn.microsoft.com/library/mt125936.aspx) Cmdlet。 請使用這個 Cmdlet 建立 Batch 集區。

### <a name="example"></a>範例

```json
{
  "name": "AzureBatchLinkedService",
  "properties": {
    "type": "AzureBatch",
    "typeProperties": {
      "accountName": "<Azure Batch account name>",
      "accessKey": "<Azure Batch account key>",
      "poolName": "<Azure Batch pool name>",
      "linkedServiceName": "<Specify associated storage linked service reference here>"
    }
  }
}
```

至於 **accountName** 屬性，請於批次帳戶名稱後面附加 **.\<區域名稱\>**。 例如︰

```json
"accountName": "mybatchaccount.eastus"
```

另一個選項是提供 **batchUri** 端點。 例如︰

```json
"accountName": "adfteam",
"batchUri": "https://eastus.batch.azure.com",
```

### <a name="properties"></a>properties
| 屬性          | 說明                              | 必要 |
| ----------------- | ---------------------------------------- | -------- |
| type              | 將 type 屬性設定為 **AzureBatch**。 | yes      |
| accountName       | Batch 帳戶的名稱。         | yes      |
| accessKey         | Batch 帳戶的存取金鑰。  | yes      |
| poolName          | VM 集區的名稱。    | yes      |
| 預設容器 | 與此 Batch 連結服務相關聯之儲存體服務連結的名稱。 此連結服務會用於執行活動及儲存活動執行記錄所需的暫存檔案。 | yes      |

## <a name="azure-machine-learning-linked-service"></a>Azure Machine Learning 連結服務
您可以建立 Machine Learning 連結服務，以向資料處理站註冊 Machine Learning 批次評分端點。

### <a name="example"></a>範例

```json
{
  "name": "AzureMLLinkedService",
  "properties": {
    "type": "AzureML",
    "typeProperties": {
      "mlEndpoint": "https://[batch scoring endpoint]/jobs",
      "apiKey": "<apikey>"
    }
  }
}
```

### <a name="properties"></a>properties
| 屬性   | 說明                              | 必要 |
| ---------- | ---------------------------------------- | -------- |
| 類型       | 將 type 屬性設定為 **AzureML**。 | yes      |
| mlEndpoint | 批次評分 URL。                   | yes      |
| apiKey     | 已發佈的工作區模型的 API。     | yes      |

## <a name="azure-data-lake-analytics-linked-service"></a>Azure Data Lake Analytics 已連結的服務
您可以建立 Data Lake Analytics 連結服務，將 Data Lake Analytics 計算服務連結到 Azure Data Factory。 管線中的 Data Lake Analytics U-SQL 活動會參考此連結服務。 

下表說明 JSON 定義中所使用的一般屬性：

| 屬性                 | 說明                              | 必要                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| type                 | 將 type 屬性設定為 **AzureDataLakeAnalytics**。 | yes                                      |
| accountName          | Data Lake Analytics 帳戶名稱。  | yes                                      |
| dataLakeAnalyticsUri | Data Lake Analytics URI。           | 否                                       |
| subscriptionId       | Azure 訂用帳戶識別碼。                    | 否<br /><br />(如果未指定，便會使用資料處理站的訂用帳戶。) |
| resourceGroupName    | Azure 資源群組名稱。                | 否<br /><br /> (如果未指定，便會使用資料處理站的資源群組。) |

### <a name="authentication-options"></a>驗證選項
您可以針對 Data Lake Analytics 連結服務，選擇使用服務主體驗證或使用者認證驗證。

#### <a name="service-principal-authentication-recommended"></a>服務主體驗證 (建議)
若要使用服務主體驗證，請在 Azure Active Directory (Azure AD) 中註冊應用程式實體。 然後，將 Azure AD 存取權授與 Data Lake Store。 如需詳細的步驟，請參閱[服務對服務驗證](../../data-lake-store/data-lake-store-authenticate-using-active-directory.md)。 請記下以下的值，您可以使用這些值來定義連結服務：
* 應用程式識別碼
* 應用程式金鑰 
* 租用戶識別碼

指定下列屬性以使用服務主體驗證：

| 屬性                | 說明                              | 必要 |
| :---------------------- | :--------------------------------------- | :------- |
| servicePrincipalId  | 應用程式的用戶端識別碼。     | yes      |
| servicePrincipalKey | 應用程式的金鑰。           | yes      |
| tenant              | 應用程式所在租用戶的資訊 (網域名稱或租用戶識別碼)。 若要獲得此資訊，請將滑鼠游標暫留在 Azure 入口網站右上角。 | yes      |

**範例：服務主體驗證**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="user-credential-authentication"></a>使用者認證驗證
若為 Data Lake Analytics 的使用者認證驗證，請指定下列屬性：

| 屬性          | 說明                              | 必要 |
| :---------------- | :--------------------------------------- | :------- |
| 授權 | 在 Data Factory 編輯器中，選取 [授權] 按鈕。 輸入會將自動產生的授權 URL 指派給這個屬性的認證。 | yes      |
| sessionId     | OAuth 授權工作階段的 OAuth 工作階段識別碼。 每個工作階段識別碼都是唯一的，只能使用一次。 當您使用 Data Factory 編輯器時便會自動產生此設定。 | yes      |

**範例：使用者認證授權**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>權杖到期
您選取 [授權] 按鈕所產生的授權碼會在一定時間後到期。 

當驗證權杖到期時，您可能會看見下列錯誤訊息： 

  認證作業錯誤：invalid_grant - AADSTS70002：驗證認證時發生錯誤。 AADSTS70008：提供的存取授權已過期或撤銷。 追蹤識別碼：d18629e8-af88-43c5-88e3-d8419eb1fca1 相互關連識別碼：fac30a0c-6be6-4e02-8d69-a776d2ffefd7 時間戳記：2015-12-15 21:09:31Z

下表顯示依使用者帳戶類型的到期時間： 

| 使用者類型                                | 到期時間                            |
| :--------------------------------------- | :--------------------------------------- |
| 「不受」Azure AD 管理的使用者帳戶 (Hotmail、Live 等等) | 12 小時。                                 |
| 「受」Azure AD 管理的使用者帳戶 | 最後一次執行配量後的 14 天。 <br /><br />如果以 OAuth 式連結服務為基礎的配量至少每 14 天執行一次，則為 90 天。 |

若要避免或解決此錯誤，請在權杖到期時選取 [授權] 按鈕來重新授權。 然後，重新部署連結服務。 您也可以使用下列程式碼，以程式設計方式產生 sessionId  和 authorization 屬性的值：

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```

如需有關此程式碼範例所用 Data Factory 類別的詳細資料，請參閱：
* [AzureDataLakeStoreLinkedService 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx)
* [AzureDataLakeAnalyticsLinkedService 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx)
* [AuthorizationSessionGetResponse 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx)

請針對 **WindowsFormsWebAuthenticationDialog** 類別，新增對 Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll 的參考。 

## <a name="azure-sql-linked-service"></a>Azure SQL 連結服務
您可以建立 SQL 連結服務並與[預存程序活動](data-factory-stored-proc-activity.md)搭配使用，以從 Data Factory 管線叫用預存程序。 如需詳細資訊，請參閱 [Azure SQL 連接器](data-factory-azure-sql-connector.md#linked-service-properties)。

## <a name="azure-sql-data-warehouse-linked-service"></a>Azure SQL 資料倉儲連結服務
您可以建立 SQL 資料倉儲連結服務並與[預存程序活動](data-factory-stored-proc-activity.md)搭配使用，以從 Data Factory 管線叫用預存程序。 如需詳細資訊，請參閱 [Azure SQL 資料倉儲連接器](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties)。

## <a name="sql-server-linked-service"></a>SQL Server 連結服務
您可以建立 SQL Server 連結服務並與[預存程序活動](data-factory-stored-proc-activity.md)搭配使用，以從 Data Factory 管線叫用預存程序。 如需詳細資訊，請參閱 [SQL Server 連接器](data-factory-sqlserver-connector.md#linked-service-properties)。

