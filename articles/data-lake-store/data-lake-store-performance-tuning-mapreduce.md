---
title: Azure Data Lake Storage Gen1 MapReduce 效能微調方針 | Microsoft Docs
description: Azure Data Lake Storage Gen1 MapReduce 效能微調方針
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
ms.openlocfilehash: b661499786057a3083f79684dfd12c85266b7b5c
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46128786"
---
# <a name="performance-tuning-guidance-for-mapreduce-on-hdinsight-and-azure-data-lake-storage-gen1"></a>HDInsight 和 Azure Data Lake Storage Gen1 上的 MapReduce 效能微調方針

## <a name="prerequisites"></a>必要條件

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure Data Lake Storage Gen1 帳戶**。 如需建立帳戶的指示，請參閱[開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* 可存取 Data Lake Storage Gen1 帳戶的 **Azure HDInsight 叢集**。 請參閱[建立具有 Data Lake Storage Gen1 的 HDInsight 叢集](data-lake-store-hdinsight-hadoop-use-portal.md)。 請確實為叢集啟用遠端桌面。
* **在 HDInsight 上使用 MapReduce**。  如需詳細資訊，請參閱[在 HDInsight 上的 Hadoop 中使用 MapReduce](https://docs.microsoft.com/azure/hdinsight/hdinsight-use-mapreduce)
* **Data Lake Storage Gen1 的效能微調方針**。  如需一般的效能概念，請參閱 [Data Lake Storage Gen1 效能微調指導方針](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-performance-tuning-guidance)

## <a name="parameters"></a>參數

在執行 MapReduce 作業時，您可以設定以下最重要的參數，以在 Data Lake Storage Gen1 上增加效能︰

* **Mapreduce.map.memory.mb** – 要配置給每個對應器的記憶體數量
* **Mapreduce.job.maps** – 每個作業的對應工作數目
* **Mapreduce.reduce.memory.mb** – 要配置給每個歸納器的記憶體數量
* **Mapreduce.job.reduces** – 每個作業的縮減工作數目

**Mapreduce.map.memory / Mapreduce.reduce.memory** 這個數字應根據對應和/或縮減工作需要多少記憶體來進行調整。  mapreduce.map.memory 和 mapreduce.reduce.memory 的預設值可以在 Ambari 中透過 Yarn 組態來檢視。  在 Ambari 中，瀏覽至 YARN，然後檢視 [設定] 索引標籤。YARN 記憶體將會顯示。  

**Mapreduce.job.maps / Mapreduce.job.reduces** 這會決定要建立的對應器或歸納器數目上限。  分割數會決定要為 MapReduce 作業建立多少對應器。  因此，如果分割數比要求的對應器數目少，您所得到的對應器可能會比您要求的少。       

## <a name="guidance"></a>指引

**步驟 1︰決定執行的作業數目** - 根據預設，MapReduce 會為您的作業使用整個叢集。  您可以使用比可用容器還少的對應器，來使用較少的叢集。  本文中的指導方針假設您的應用程式是叢集上唯一執行的應用程式。      

**步驟 2︰設定 mapreduce.map.memory/mapreduce.reduce.memory** – 對應和縮減工作的記憶體大小會取決於您的特定作業。  如果您想要增加並行能力，您可以減少記憶體大小。  並行執行工作的數目取決於容器數目。  藉由降低每個對應器或歸納器的記憶體數量，即可建立更多容器，而讓更多的對應器或歸納器並行執行。  減少太多的記憶體數量可能會導致某些處理程序耗盡記憶體。  如果您在執行作業時遇到堆積錯誤，您應該增加每個對應器或歸納器的記憶體。  您應該考慮到，新增更多容器會對每個額外的容器新增額外的負擔，而可能降低效能。  另一個方式是使用擁有較高數量記憶體的叢集，或增加叢集中的節點數目，以獲得更多的記憶體。  更多的記憶體就能使用更多的容器，亦即會有更多並行能力。  

**步驟 3︰決定 YARN 記憶體總數** - 若要調整 mapreduce.job.maps/mapreduce.job.reduces，您應該考慮可供使用的 YARN 記憶體總數。  這項資訊可在 Ambari 中取得。  瀏覽至 YARN，然後檢視 [設定] 索引標籤。YARN 記憶體會顯示在此視窗中。  您應該將 YARN 記憶體乘上叢集中的節點數目，以算出 YARN 記憶體總數。

    Total YARN memory = nodes * YARN memory per node
如果您使用空白叢集，則記憶體會是叢集的 YARN 記憶體總數。  如果有其他應用程式使用記憶體，則您可以選擇僅使用部分的叢集記憶體，方法是將對應器或歸納器的數量減少為您想要使用的容器數量。  

**步驟 4︰計算 YARN 容器的數量** – YARN 容器會決定作業可用的並行數量。  取得 YARN 記憶體總數，然後除以 mapreduce.map.memory。  

    # of YARN containers = total YARN memory / mapreduce.map.memory

**步驟 5：設定 mapreduce.job.maps/mapreduce.job.reduces** 將 mapreduce.job.maps/mapreduce.job.reduces 設定為至少是可用容器的數目。  您可以進一步試驗，將對應器和歸納器的數目增加，看看是否能獲得更好的效能。  請記住，更多的對應器會產生額外的負擔，因此對應器過多可能會降低效能。  

CPU 排程和 CPU 隔離預設會關閉，因此 YARN 容器的數目會受記憶體所限制。

## <a name="example-calculation"></a>範例計算

設想您目前有由 8 個 D14 節點所組成的叢集，而且您想要執行 I/O 密集作業。  以下是您應該進行的計算︰

**步驟 1︰決定所執行作業的數目** - 在我們的範例中，我們假設我們的作業是唯一在執行的作業。  

**步驟 2︰設定 mapreduce.map.memory/mapreduce.reduce.memory** – 在我們的範例中，您將會執行 I/O 密集作業，並決定 3 GB 的記憶體已足夠對應工作使用。

    mapreduce.map.memory = 3GB
**步驟 3︰確定 YARN 記憶體總數**

    total memory from the cluster is 8 nodes * 96GB of YARN memory for a D14 = 768GB
**步驟 4︰計算 YARN 容器的數目**

    # of YARN containers = 768GB of available memory / 3 GB of memory =   256

**步驟 5：設定 mapreduce.job.maps/mapreduce.job.reduces**

    mapreduce.map.jobs = 256

## <a name="limitations"></a>限制

**Data Lake Storage Gen1 節流**

Data Lake Storage Gen1 為多租用戶服務，因此會設定帳戶層級的頻寬限制。  如果您達到這些限制，您將會開始看到工作失敗。 透過觀察工作記錄檔中的節流錯誤即可加以識別。  如果您的作業需要更多頻寬，請與我們連絡。   

若要檢查您是否遭到節流，您必須在用戶端啟用偵錯記錄。 做法如下：

1. 將下列屬性放在 [Ambari] > [YARN] > [設定] > [進階 yarn-log4j] 的 log4j 屬性中：log4j.logger.com.microsoft.azure.datalake.store=DEBUG

2. 重新啟動所有節點/服務，以便讓設定生效。

3. 如果您遭到節流，您會看到 YARN 記錄檔中有 HTTP 429 錯誤碼。 YARN 記錄檔位於 /tmp/&lt;user&gt;/yarn.log 中

## <a name="examples-to-run"></a>要執行的範例

為示範 MapReduce 在 Data Lake Storage Gen1 中的執行方式，以下提供一些使用下列設定在叢集上執行的範例程式碼︰

* 16 節點 D14v2
* 執行 HDI 3.6 的 Hadoop 叢集

以下是一些用來執行 MapReduce Teragen、Terasort 和 Teravalidate 的範例命令，您可以從這邊來著手。  您可以根據您的資源調整這些命令。

**Teragen**

    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 10000000000 adl://example/data/1TB-sort-input

**Terasort**

    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 -Dmapreduce.job.reduces=512 -Dmapreduce.reduce.memory.mb=3072 adl://example/data/1TB-sort-input adl://example/data/1TB-sort-output

**Teravalidate**

    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapreduce.job.maps=512 -Dmapreduce.map.memory.mb=3072 adl://example/data/1TB-sort-output adl://example/data/1TB-sort-validate
