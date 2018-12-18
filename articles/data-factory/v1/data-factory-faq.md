---
title: Azure 資料處理站-常見問題集
description: 關於 Azure Data Factory 的常見問題。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 532dec5a-7261-4770-8f54-bfe527918058
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 414a6042d064a1dece7025dc277533d40ef2aa64
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43781908"
---
# <a name="azure-data-factory---frequently-asked-questions"></a>Azure 資料處理站-常見問題集
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[常見問題集：Data Factory](../frequently-asked-questions.md)。

## <a name="general-questions"></a>一般問題
### <a name="what-is-azure-data-factory"></a>Azure 資料處理站是什麼？
Data Factory 是雲端架構資料整合服務，用來 **自動移動和轉換資料**。 就像會運轉設備以將原物料轉換成成品的工廠一樣，Data Factory 會協調現有的服務來收集未經處理資料，並將之轉換成隨時可用的資訊。

Data Factory 可讓您建立資料導向工作流程，不僅可透過計算服務 (例如 Azure HDInsight 和 Azure Data Lake Analytics) 來處理/轉換資料，還能在內部部署與雲端資料存放區之間移動資料。 建立可執行您所需動作的管線之後，您可以排定讓它定期執行 (每小時、每天、每週等)。   

如需詳細資訊，請參閱[概觀與重要概念](data-factory-introduction.md)。

### <a name="where-can-i-find-pricing-details-for-azure-data-factory"></a>哪裡可以找到 Azure 資料處理站的定價詳細資料？
請參閱 [Data Factory 定價詳細資料頁面][adf-pricing-details]，以了解 Azure Data Factory 的定價詳細資料。  

### <a name="how-do-i-get-started-with-azure-data-factory"></a>如何開始使用 Azure Data Factory？
* 如需 Azure Data Factory 的概觀，請參閱 [Azure Data Factory 簡介](data-factory-introduction.md)。
* 如需說明如何使用複製活動 **複製/移動資料** 的教學課程，請參閱 [將資料從 Azure Blob 儲存體複製到 Azure SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
* 如需說明如何使用 HDInsight Hive 活動 **轉換資料** 的教學課程。 請參閱 [在 Hadoop 叢集上執行 Hive 指令碼來處理資料](data-factory-build-your-first-pipeline.md)

### <a name="what-is-the-data-factorys-region-availability"></a>什麼是 Data Factory 的區域可用性？
Data Factory 可在**美國西部**和**北歐**地區使用。 資料處理站所使用的計算服務和儲存體服務可以在其他區域使用。 請參閱 [支援的區域](data-factory-introduction.md#supported-regions)。

### <a name="what-are-the-limits-on-number-of-data-factoriespipelinesactivitiesdatasets"></a>資料處理站/管線/活動/資料集的數量有什麼限制？
請參閱〈 **Azure 訂用帳戶和服務限制、配額及條件約束** 〉中的〈 [Azure Data Factory 限制](../../azure-subscription-service-limits.md#data-factory-limits) 〉章節。

### <a name="what-is-the-authoringdeveloper-experience-with-azure-data-factory-service"></a>Azure Data Factory 服務的撰寫/開發人員經驗為何？
您可以使用下列其中一個工具/SDK 來製作/建立資料處理站：

* **Azure 入口網站** Azure 入口網站中的 Data Factory 刀鋒視窗提供豐富的使用者介面，可讓您建立 Data Factory 和連結的服務。 **Data Factory 編輯器**也是入口網站的一部分，讓您透過指定成品的 JSON 定義，輕鬆建立連結服務、資料表、資料集和管線。 如需使用入口網站/編輯器來建立和部署 Data Factory 的範例，請參閱 [使用 Azure 入口網站建置您的第一個資料管線](data-factory-build-your-first-pipeline-using-editor.md) 。
* **Visual Studio** 您可以使用 Visual Studio 建立 Azure Data Factory。 如需詳細資料，請參閱 [使用 Visual Studio 建置您的第一個資料管線](data-factory-build-your-first-pipeline-using-vs.md) 。
* **Azure PowerShell** 如需使用 PowerShell 來建立 Data Factory 的教學課程/逐步解說，請參閱 [使用 Azure PowerShell 建立和監視 Azure Data Factory](data-factory-build-your-first-pipeline-using-powershell.md) 。 如需 Data Factory Cmdlet 的完整文件，請參閱 MSDN Library 上的 [Data Factory Cmdlet 參考][adf-powershell-reference]內容。
* **.NET 類別庫** 您可以使用 Data Factory .NET SDK，透過程式設計方式建立 Data Factory。 如需使用 .NET SDK 建立 Data Factory 的逐步解說，請參閱 [使用 .NET SDK 建立、監視和管理 Data Factory](data-factory-create-data-factories-programmatically.md) 。 如需 Data Factory .NET SDK 的完整文件，請參閱 [Data Factory 類別庫參考][msdn-class-library-reference]。
* **REST API** 您也可以使用 Azure Data Factory 服務所公開的 REST API 來建立和部署 Data Factory。 如需 Data Factory REST API 的完整文件，請參閱 [Data Factory REST API 參考][msdn-rest-api-reference]。
* **Azure Resource Manager 範本** 請參閱 [教學課程：使用 Azure Resource Manager 範本建置您的第一個 Azure Data Factory](data-factory-build-your-first-pipeline-using-arm.md) 以取得詳細資訊。

### <a name="can-i-rename-a-data-factory"></a>我是否可以重新命名資料處理站？
否。 和其他 Azure 資源一樣，您無法變更 Azure Data Factory 的名稱。

### <a name="can-i-move-a-data-factory-from-one-azure-subscription-to-another"></a>我是否可以將 Data Factory 從一個 Azure 訂用帳戶移至另一個訂用帳戶？
是。 請使用您資料處理站刀鋒視窗上的 [移動] 按鈕，如下圖所示：

![移動 Data Factory](media/data-factory-faq/move-data-factory.png)

### <a name="what-are-the-compute-environments-supported-by-data-factory"></a>Data Factory 支援什麼計算環境?
下表列出 Data Factory 支援的計算環境以及可在環境上執行的活動。

| 計算環境 | 活動 |
| --- | --- |
| [隨選 HDInsight 叢集](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)或[您自己的 HDInsight 叢集](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) |[DotNet](data-factory-use-custom-activities.md)、[Hive](data-factory-hive-activity.md)、[Pig](data-factory-pig-activity.md)、[MapReduce](data-factory-map-reduce.md)、[Hadoop 串流](data-factory-hadoop-streaming-activity.md) |
| [Azure Batch](data-factory-compute-linked-services.md#azure-batch-linked-service) |[DotNet](data-factory-use-custom-activities.md) |
| [Azure Machine Learning](data-factory-compute-linked-services.md#azure-machine-learning-linked-service) |[Machine Learning 活動︰批次執行和更新資源](data-factory-azure-ml-batch-execution-activity.md) |
| [Azure 資料湖分析](data-factory-compute-linked-services.md#azure-data-lake-analytics-linked-service) |[Data Lake Analytics U-SQL](data-factory-usql-activity.md) |
| [Azure SQL](data-factory-compute-linked-services.md#azure-sql-linked-service)、[Azure SQL 資料倉儲](data-factory-compute-linked-services.md#azure-sql-data-warehouse-linked-service)、[SQL Server](data-factory-compute-linked-services.md#sql-server-linked-service) |[預存程序](data-factory-stored-proc-activity.md) |

### <a name="how-does-azure-data-factory-compare-with-sql-server-integration-services-ssis"></a>Azure Data Factory 相較於 SQL Server Integration Services (SSIS) 有何異同？ 
請參閱由我們的 MVP (最有價值專家) Reza Rad 所撰寫的 [Azure Data Factory 與SSIS 的比較 (英文)](http://www.sqlbits.com/Sessions/Event15/Azure_Data_Factory_vs_SSIS)。 Data Factory 中的某些最近變更可能未列於投影片組中。 我們會持續新增更多功能到 Azure Data Factory。 我們會持續新增更多功能到 Azure Data Factory。 我們會在今年將這些更新整合到 Microsoft 的資料整合技術比較中。   

## <a name="activities---faq"></a>活動 - 常見問題集
### <a name="what-are-the-different-types-of-activities-you-can-use-in-a-data-factory-pipeline"></a>您可以在 Data Factory 管線中使用的不同類型活動有哪些？
* [資料移動活動](data-factory-data-movement-activities.md) 以移動資料。
* [資料轉換活動](data-factory-data-transformation-activities.md) 以處理/轉換資料。

### <a name="when-does-an-activity-run"></a>何時執行活動？
輸出資料表中的 **可用性** 組態設定決定何時執行活動。 如果已指定輸入資料集，活動會在開始執行之前，先檢查是否滿足所有輸入資料相依性 (即「就緒」  狀態)。

## <a name="copy-activity---faq"></a>複製活動 - 常見問題集
### <a name="is-it-better-to-have-a-pipeline-with-multiple-activities-or-a-separate-pipeline-for-each-activity"></a>最好是一個管線有多個活動，還是每個活動都有不同的管線？
管線依例應該有配套的相關活動。 如果管線外的任何其他活動都未使用連接它們的資料集，則您可以將活動保留在一個管線中。 如此一來，您就不需要鏈結管線作用期間，使其彼此一致。 此外，更新管線時，也會更適當地保留管線內部資料表中的資料完整性。 管線更新基本上會停止、移除並重新建立管線內的所有活動。 從撰寫觀點來看，可能也較容易看出管線的某個 JSON 檔案中相關活動內的資料流程。

### <a name="what-are-the-supported-data-stores"></a>支援哪些資料存放區？
Data Factory 中的複製活動會將資料從來源資料存放區複製到接收資料存放區。 Data Factory 支援下列資料存放區。 可將來自任何來源的資料寫入任何接收器。 按一下資料存放區，即可了解如何將資料複製到該存放區，以及從該存放區複製資料。

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> 具有 * 的資料存放區可以在內部部署環境或 Azure IaaS 上，並且需要您在內部部署/Azure IaaS 機器上安裝 [資料管理閘道](data-factory-data-management-gateway.md) 。

### <a name="what-are-the-supported-file-formats"></a>支援什麼檔案格式？
[!INCLUDE [data-factory-file-format](../../../includes/data-factory-file-format.md)]

### <a name="where-is-the-copy-operation-performed"></a>會在哪裡執行複製作業？
如需詳細資料，請參閱 [全域可用的資料移動](data-factory-data-movement-activities.md#global) 一節。 簡單地說，當涉及內部部署資料存放區時，複製作業是由「資料管理閘道」在內部部署環境中執行。 而在兩個雲端存放區之間移動資料時，複製作業是在最接近相同地理位置內接收位置的區域中執行。

## <a name="hdinsight-activity---faq"></a>HDInsight 活動 - 常見問題集
### <a name="what-regions-are-supported-by-hdinsight"></a>HDInsight 支援哪些區域？
請參閱下列文章中的＜各地區上市情況＞一節：或 [HDInsight 定價詳細資料][hdinsight-supported-regions]。

### <a name="what-region-is-used-by-an-on-demand-hdinsight-cluster"></a>隨選 HDInsight 叢集使用哪一個區域？
隨選 HDInsight 叢集會建立在存有您指定用來使用叢集之儲存體的位置。    

### <a name="how-to-associate-additional-storage-accounts-to-your-hdinsight-cluster"></a>如何讓其他儲存體帳戶與 HDInsight 叢集產生關聯？
如果您使用的是自己的「HDInsight 叢集」(BYOC - 自攜叢集)，請參閱下列主題：

* [搭配使用 HDInsight 叢集與替代儲存體帳戶和中繼存放區][hdinsight-alternate-storage]
* [搭配使用其他儲存體帳戶與 HDInsight Hive][hdinsight-alternate-storage-2]

如果您使用的是 Data Factory 服務所建立的隨選叢集，請為 HDInsight 連結服務指定額外的儲存體帳戶，以便讓 Data Factory 服務代表您註冊它們。 在隨選連結服務的 JSON 定義中，請使用 **additionalLinkedServiceNames** 屬性指定替代的儲存體帳戶，如下列 JSON 片段所示：

```JSON
{
    "name": "MyHDInsightOnDemandLinkedService",
    "properties":
    {
        "type": "HDInsightOnDemandLinkedService",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "LinkedService-SampleData",
            "additionalLinkedServiceNames": [ "otherLinkedServiceName1", "otherLinkedServiceName2" ]
        }
    }
}
```
在上述範例中，otherLinkedServiceName1 和 otherLinkedServiceName2 連結服務的定義，包含 HDInsight 叢集存取替代儲存體帳戶所需的認證。

## <a name="slices---faq"></a>配量 - 常見問題集
### <a name="why-are-my-input-slices-not-in-ready-state"></a>為什麼我的輸入配量不是處於「就緒」狀態？
常見的錯誤是當輸入資料是 Data Factory 的外部資料 (不是由 Data Factory 產生) 時，未將輸入資料集上的 **external** 屬性設定為 **true**。

在下列範例中，您只需要將 **dataset1** 上的 **external** 設定為 true 即可。  

**DataFactory1** Pipeline 1: dataset1 -> activity1 -> dataset2 -> activity2 -> dataset3 Pipeline 2: dataset3-> activity3 -> dataset4

如果您有另一個 Data Factory，其管線會接受 dataset4 (由 Data Factory 1 中的 pipeline 2 產生)，請將 dataset4 標示為外部資料集，因為該資料集是由不同的 Data Factory (DataFactory1，而非 DataFactory2) 產生。  

**DataFactory2**    
Pipeline 1: dataset4->activity4->dataset5

如果 external 屬性的設定正確，請確認輸入資料是否存在於輸入資料集定義中指定的位置。

### <a name="how-to-run-a-slice-at-another-time-than-midnight-when-the-slice-is-being-produced-daily"></a>每日產生配量時，如何在午夜以外的其他時間執行配量？
請使用 **offset** 屬性來指定要產生配量的時間。 如需有關此屬性的詳細資料，請參閱 [資料集可用性](data-factory-create-datasets.md#dataset-availability) 一節。 以下是一個簡短的範例：

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
每日配量於 **上午 6 點** (而非預設的午夜) 開始。     

### <a name="how-can-i-rerun-a-slice"></a>如何重新執行配量？
您可以利用下列方式之一來重新執行配量：

* 使用「監視及管理應用程式」來重新執行活動時段或配量。 如需相關指示，請參閱 [重新執行已選取的活動時段](data-factory-monitor-manage-app.md#perform-batch-actions) 。   
* 在 Azure 入口網站中，於該配量的 [資料配量] 刀鋒視窗上，按一下命令列中的 [執行]。
* 在配量的 Status 設定為 **Waiting** 的情況下，執行 **Set-AzureRmDataFactorySliceStatus** Cmdlet。   

    ```PowerShell
    Set-AzureRmDataFactorySliceStatus -Status Waiting -ResourceGroupName $ResourceGroup -DataFactoryName $df -TableName $table -StartDateTime "02/26/2015 19:00:00" -EndDateTime "02/26/2015 20:00:00"
    ```
如需 Cmdlet 的詳細資料，請參閱 [Set-AzureRmDataFactorySliceStatus][set-azure-datafactory-slice-status]。

### <a name="how-long-did-it-take-to-process-a-slice"></a>處理配量需要多久的時間？
使用「監視及管理應用程式」中的「活動時段總管」來了解處理一個資料配量所花費的時間。 如需詳細資料，請參閱 [活動時段總管](data-factory-monitor-manage-app.md#activity-window-explorer) 。

您也可以在 Azure 入口網站中執行下列操作：  

1. 在您 Data Factory 的 [DATA FACTORY] 刀鋒視窗中，按一下 [資料集] 圖格。
2. 在 [ **資料集** ] 刀鋒視窗中，按一下特定資料集。
3. 從 [資料表] 刀鋒視窗的 [最近配量] 清單中，選取您感興趣的配量。
4. 從 [資料配量] 刀鋒視窗的 [活動執行] 清單中，按一下活動執行。
5. 在 [活動執行詳細資料] 刀鋒視窗中，按一下 [屬性] 圖格。
6. 您應該會看到 [持續時間]  欄位與值。 這個值是處理配量所花費的時間。   

### <a name="how-to-stop-a-running-slice"></a>如何停止執行中配量？
如果需要停止執行管線，可以使用 [Suspend-AzureRmDataFactoryPipeline](/powershell/module/azurerm.datafactories/suspend-azurermdatafactorypipeline) Cmdlet。 目前，擱置管線並不會停止正在進行的配量執行。 一旦進行中的執行完成，就不會再挑選任何額外的配量。

如果真的想要立即停止所有執行作業，唯一的方法就是刪除管線，然後再重新建立。 如果您選擇刪除管線，則「不」需要刪除管線所使用的資料表和連結服務。

[create-factory-using-dotnet-sdk]: data-factory-create-data-factories-programmatically.md
[msdn-class-library-reference]: /dotnet/api/microsoft.azure.management.datafactories.models
[msdn-rest-api-reference]: /rest/api/datafactory/

[adf-powershell-reference]: /powershell/module/azurerm.datafactories/
[azure-portal]: http://portal.azure.com
[set-azure-datafactory-slice-status]: /powershell/module/azurerm.datafactories/set-azurermdatafactoryslicestatus

[adf-pricing-details]: http://go.microsoft.com/fwlink/?LinkId=517777
[hdinsight-supported-regions]: http://azure.microsoft.com/pricing/details/hdinsight/
[hdinsight-alternate-storage]: http://social.technet.microsoft.com/wiki/contents/articles/23256.using-an-hdinsight-cluster-with-alternate-storage-accounts-and-metastores.aspx
[hdinsight-alternate-storage-2]: http://blogs.msdn.com/b/cindygross/archive/2014/05/05/use-additional-storage-accounts-with-hdinsight-hive.aspx
