---
title: 資料轉換︰處理和轉換資料 | Microsoft Docs
description: 了解如何使用 Hadoop、Machine Learning 或 Azure Data Lake Analytics 在 Azure Data Factory 中轉換資料或處理資料。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 39786731-1e4b-40a4-81b7-d06e127427aa
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: afa00a81738563ec3c5b4b168f533b732c263825
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045614"
---
# <a name="transform-data-in-azure-data-factory"></a>Azure Data Factory 中的資料轉換
> [!div class="op_single_selector"]
> * [Hive](data-factory-hive-activity.md)  
> * [Pig](data-factory-pig-activity.md)  
> * [MapReduce](data-factory-map-reduce.md)  
> * [Hadoop 串流](data-factory-hadoop-streaming-activity.md)
> * [機器學習服務](data-factory-azure-ml-batch-execution-activity.md) 
> * [預存程序](data-factory-stored-proc-activity.md)
> * [資料湖分析 U-SQL](data-factory-usql-activity.md)
> * [.NET 自訂](data-factory-use-custom-activities.md)

## <a name="overview"></a>概觀
> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱 [Data Factory 中的資料轉換活動](../transform-data.md)。

本文說明 Azure Data Factory 中的資料轉換活動，您可用來轉換未經處理資料，並將其處理為預測和見解。 轉換活動會在計算環境中執行，例如 Azure HDInsight 叢集或 Azure Batch。 它會提供每個轉換活動的詳細資訊文章連結。

Data Factory 支援下列可個別或與其他活動鏈結而加入至 [管線](data-factory-create-pipelines.md) 的資料轉換活動。

> [!NOTE]
> 如需逐步解說，請參閱 [建立可 Hive 轉換的管線](data-factory-build-your-first-pipeline.md) 。  
> 
> 

## <a name="hdinsight-hive-activity"></a>HDInsight Hive 活動
Data Factory 管線中的 HDInsight Hive 活動會在您自己或隨選的 Windows/Linux 架構 HDInsight 叢集上執行 Hive 查詢。 如需此活動的詳細資訊，請參閱 [Hive 活動](data-factory-hive-activity.md) 。 

## <a name="hdinsight-pig-activity"></a>HDInsight Pig 活動
Data Factory 管線中的 HDInsight Pig 活動會在您自己或隨選的 Windows/Linux 架構 HDInsight 叢集上執行 Pig 查詢。 如需此活動的詳細資訊，請參閱 [Pig 活動](data-factory-pig-activity.md) 。 

## <a name="hdinsight-mapreduce-activity"></a>HDInsight MapReduce 活動
Data Factory 管線中的 HDInsight MapReduce 活動會在您自己或隨選的 Windows/Linux 架構 HDInsight 叢集上執行 MapReduce 程式。 如需此活動的詳細資訊，請參閱 [MapReduce 活動](data-factory-map-reduce.md) 。

## <a name="hdinsight-streaming-activity"></a>HDInsight 串流活動
Data Factory 管線中的 HDInsight 串流活動會在您自己或隨選的 Windows/Linux 架構 HDInsight 叢集上執行 Hadoop 串流程式。 如需此活動的詳細資訊，請參閱 [HDInsight 串流活動](data-factory-hadoop-streaming-activity.md) 。

## <a name="hdinsight-spark-activity"></a>HDInsight Spark 活動
Data Factory 管線中的 HDInsight Spark 活動會在您自己的 HDInsight 叢集上執行 Spark 程式。 如需詳細資訊，請參閱[從 Azure Data Factory 叫用 Spark 程式](data-factory-spark.md)。 

## <a name="machine-learning-activities"></a>Machine Learning 活動
Azure Data Factory 可讓您輕鬆地建立管線，使用已發佈的 Azure Machine Learning Web 服務進行預測性分析。 在 Azure Data Factory 管線中使用 [批次執行活動](data-factory-azure-ml-batch-execution-activity.md#invoking-a-web-service-using-batch-execution-activity) ，您可以叫用 Machine Learning Web 服務來對批次中的資料進行預測。

經過一段時間，必須使用新的輸入資料集重新訓練 Machine Learning 評分實驗中的預測模型。 完成重新訓練之後，您想要使用已重新訓練的 Machine Learning 模型來更新評分 Web 服務。 您可以使用 [更新資源活動](data-factory-azure-ml-batch-execution-activity.md#updating-models-using-update-resource-activity) ，以新訓練的模型更新 Web 服務。  

如需這些機器學習活動的詳細資料，請參閱 [使用 Machine Learning 活動](data-factory-azure-ml-batch-execution-activity.md) 。 

## <a name="stored-procedure-activity"></a>預存程序活動
您可以在 Data Factory 管線中使用 SQL Server 的預存程序活動，以叫用下列其中一個資料存放區中的預存程序：您的企業或 Azure VM 中的 Azure SQL Database、Azure SQL 資料倉儲、SQL Server 資料庫。 如需詳細資料，請參閱 [預存程序活動](data-factory-stored-proc-activity.md) 。  

## <a name="data-lake-analytics-u-sql-activity"></a>Data Lake Analytics U-SQL 活動
Data Lake Analytics U-SQL 活動會在 Azure Data Lake Analytics 叢集上執行 U-SQL 指令碼。 如需詳細資料，請參閱 [U-SQL 活動](data-factory-usql-activity.md) 。 

## <a name="net-custom-activity"></a>.NET 自訂活動
如果您需要以 Data Factory 不支援的方法轉換資料，可以利用自己的資料處理邏輯建立自訂活動，然後在管線中使用活動。 您可以將自訂 .NET 活動設定為使用 Azure Batch 服務或 Azure HDInsight 叢集來執行。 如需詳細資訊請參閱 [使用自訂活動](data-factory-use-custom-activities.md) 。 

您可以建立自訂活動，以便在已安裝 R 的 HDInsight 叢集上執行 R 指令碼。 請參閱 [使用 Azure Data Factory 執行 R 指令碼](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/RunRScriptUsingADFSample)。 

## <a name="compute-environments"></a>計算環境
您需要為計算環境建立連結服務，然後在定義轉換活動時使用該連結服務。 Data Factory 支援兩種類型的資計算環境。 

1. **隨選**：在此情況下，運算環境完全由 Data Factory 管理。 Data Factory 服務會在工作提交前自動建立運算環境以處理資料，而在工作完成時予以移除。 您可以針對工作執行、叢集管理及啟動載入動作，設定和控制隨選計算環境的細微設定。 
2. **攜帶您自己的裝置**：在此情況下，您可以註冊自己的運算環境 (例如 HDInsight 叢集)，做為 Data Factory 中的連結服務。 運算環境由您自行管理，而 Data Factory 會使用它來執行活動。 

如需了解 Data Factory 所支援的計算服務，請參閱 [計算連結服務](data-factory-compute-linked-services.md) 一文。 

## <a name="summary"></a>總結
Azure Data Factory 支援下列資料轉換活動和活動計算環境。 可將轉換活動個別加入管線，或先與其他活動鏈結再加入管線。

| 資料轉換活動 | 計算環境 |
|:--- |:--- |
| [Hive](data-factory-hive-activity.md) |HDInsight [Hadoop] |
| [Pig](data-factory-pig-activity.md) |HDInsight [Hadoop] |
| [MapReduce](data-factory-map-reduce.md) |HDInsight [Hadoop] |
| [Hadoop 串流](data-factory-hadoop-streaming-activity.md) |HDInsight [Hadoop] |
| [Machine Learning 活動︰批次執行和更新資源](data-factory-azure-ml-batch-execution-activity.md) |Azure VM |
| [預存程序](data-factory-stored-proc-activity.md) |Azure SQL、Azure SQL 資料倉儲或 SQL Server |
| [Data Lake Analytics U-SQL](data-factory-usql-activity.md) |Azure Data Lake Analytics |
| [DotNet](data-factory-use-custom-activities.md) |HDInsight [Hadoop] 或 Azure Batch |

