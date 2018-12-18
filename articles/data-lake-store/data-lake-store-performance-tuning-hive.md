---
title: Azure Data Lake Store Hive 效能微調方針 | Microsoft Docs
description: Azure Data Lake Store Hive 效能微調方針
services: data-lake-store
documentationcenter: ''
author: stewu
manager: amitkul
editor: stewu
ms.assetid: ebde7b9f-2e51-4d43-b7ab-566417221335
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 12/19/2016
ms.author: stewu
ms.openlocfilehash: c46eb1b2da62d70337e60066ed0706c3a4fdedcf
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34198964"
---
# <a name="performance-tuning-guidance-for-hive-on-hdinsight-and-azure-data-lake-store"></a>HDInsight 和 Azure Data Lake Store 上的 Hive 效能微調方針

預設設定已設定好，以便在許多不同的使用案例中提供良好的效能。  針對 I/O 密集的查詢，Hive 可進行微調，以在 ADLS 取得更好的效能。  

## <a name="prerequisites"></a>先決條件

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure 資料湖儲存區帳戶**。 如需有關如何建立帳戶的詳細指示，請參閱 [開始使用 Azure Data Lake Store](data-lake-store-get-started-portal.md)
* **Azure HDInsight 叢集** 。 請參閱 [建立具有 Data Lake Store 的 HDInsight 叢集](data-lake-store-hdinsight-hadoop-use-portal.md)。 請確實為叢集啟用遠端桌面。
* **在 HDInsight 上執行 Hive**。  若要深入了解在 HDInsight 上執行 Hive 作業，請參閱[在 HDInsight 上使用 Hive] (https://docs.microsoft.com/azure/hdinsight/hdinsight-use-hive)
* **ADLS 的效能微調指導方針**。  如需一般的效能概念，請參閱 [Data Lake Store 效能微調指導方針](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-performance-tuning-guidance)

## <a name="parameters"></a>參數

以下是要改善 ADLS 效能所應微調的最重要設定︰

* **hive.tez.container.size** – 每個工作所使用的記憶體數量

* **tez.grouping.min-size** – 每個對應器的大小下限

* **tez.grouping.max-size** – 每個對應器的大小上限

* **hive.exec.reducer.bytes.per.reducer** – 每個歸納器的大小

**hive.tez.container.size** - 容器大小會決定每個工作可以使用多少記憶體。  這是用來控制 Hive 中之並行能力的主要輸入。  

**tez.grouping.min-size** – 此參數可讓您設定每個對應器的大小下限。  如果 Tez 選擇的對應器數目小於此參數的值，Tez 會使用此處設定的值。

**tez.grouping.max-size** – 此參數可讓您設定每個對應器的大小上限。  如果 Tez 選擇的對應器數目大於此參數的值，Tez 會使用此處設定的值。

**hive.exec.reducer.bytes.per.reducer** – 此參數會設定每個歸納器的大小。  根據預設，每個歸納器為 256 MB。  

## <a name="guidance"></a>指引

**Set hive.exec.reducer.bytes.per.reducer** – 資料若未壓縮，預設值就很適用。  資料若有壓縮，則應縮減歸納器的大小。  

**Set hive.tez.container.size** – 在每個節點中，記憶體會由 yarn.nodemanager.resource.memory-mb 指定，且預設應該會在 HDI 叢集上正確設定。  如需在 YARN 中設定適當記憶體的詳細資訊，請參閱這篇[文章](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-hive-out-of-memory-error-oom)。

I/O 密集工作負載可以透過減少 Tez 容器大小，而從更符合平行處理原則受益。 這會讓使用者獲得更多容器，而增加並行能力。  不過，某些 Hive 查詢需要大量的記憶體 (例如 MapJoin)。  如果工作沒有足夠的記憶體，您會在執行階段期間遇到記憶體不足的例外狀況。  如果您遇到記憶體不足的例外狀況，則應增加記憶體。   

並行執行的工作數或平行處理原則會受到 YARN 記憶體總數的限制。  YARN 容器數目會決定可以執行多少並行工作。  若要尋找每個節點的 YARN 記憶體，您可以前往 Ambari。  瀏覽至 YARN，然後檢視 [設定] 索引標籤。YARN 記憶體會顯示在此視窗中。  

        Total YARN memory = nodes * YARN memory per node
        # of YARN containers = Total YARN memory / Tez container size
使用 ADLS 來改善效能的關鍵是盡可能地增加並行能力。  Tez 會自動計算應該建立的工作數目，因此您並不需要設定。   

## <a name="example-calculation"></a>範例計算

假設您有 8 節點的 D14 叢集。  

    Total YARN memory = nodes * YARN memory per node
    Total YARN memory = 8 nodes * 96GB = 768GB
    # of YARN containers = 768GB / 3072MB = 256

## <a name="limitations"></a>限制

**ADLS 節流** 

如果您達到 ADLS 所提供的頻寬限制，您會開始看到工作失敗。 透過觀察工作記錄檔中的節流錯誤即可加以識別。  您可以藉由增加 Tez 容器大小來減少平行處理原則。  如果您的作業需要更多並行能力，請與我們連絡。

若要檢查您是否遭到節流，您必須在用戶端啟用偵錯記錄。 做法如下：

1. 將下列屬性放在 Hive 組態中的 log4j 屬性內。做法是從 Ambari 檢視︰log4j.logger.com.microsoft.azure.datalake.store=DEBUG 重新啟動所有節點/服務以便讓組態生效。

2. 如果您遭到節流，您會看到 Hive 記錄檔中有 HTTP 429 錯誤碼。 Hive 記錄檔位於 /tmp/&lt;user&gt;/hive.log

## <a name="further-information-on-hive-tuning"></a>關於微調 Hive 的進一步資訊

以下是一些有助於微調 Hive 查詢的部落格︰
* [在 Hdinsight 中最佳化 Hadoop 的 Hive 查詢](https://azure.microsoft.com/documentation/articles/hdinsight-hadoop-optimize-hive-query/)
* [針對 Hive 查詢的效能進行疑難排解](https://blogs.msdn.microsoft.com/bigdatasupport/2015/08/13/troubleshooting-hive-query-performance-in-hdinsight-hadoop-cluster/)
* [Ignite 講解如何將 HDInsight 上的 Hive 最佳化](https://channel9.msdn.com/events/Machine-Learning-and-Data-Sciences-Conference/Data-Science-Summit-2016/MSDSS25)
