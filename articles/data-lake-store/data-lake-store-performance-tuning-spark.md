---
title: Azure Data Lake Store Spark 效能微調方針 | Microsoft Docs
description: Azure Data Lake Store Spark 效能微調方針
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
ms.openlocfilehash: a807bea13063d2a0b3c1c71ddb6c98aa2d2568d3
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34197064"
---
# <a name="performance-tuning-guidance-for-spark-on-hdinsight-and-azure-data-lake-store"></a>HDInsight 和 Azure Data Lake Store 上的 Spark 效能微調方針

在微調 Spark 的效能時，您必須考慮叢集上會執行的應用程式數目。  根據預設，您可以在 HDI 叢集上並行執行 4 個應用程式 (附註︰預設設定有可能變更)。  您可能會決定使用較少的應用程式，因此您可以覆寫預設設定，並使用更多的叢集來執行這些應用程式。  

## <a name="prerequisites"></a>先決條件

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure 資料湖儲存區帳戶**。 如需有關如何建立帳戶的詳細指示，請參閱 [開始使用 Azure Data Lake Store](data-lake-store-get-started-portal.md)
* **Azure HDInsight 叢集** 。 請參閱 [建立具有 Data Lake Store 的 HDInsight 叢集](data-lake-store-hdinsight-hadoop-use-portal.md)。 請確實為叢集啟用遠端桌面。
* **在 Azure Data Lake Store 上執行 Spark 叢集**。  如需詳細資訊，請參閱[使用 HDInsight Spark 叢集來分析 Data Lake Store 中的資料](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-use-with-data-lake-store)
* **ADLS 的效能微調指導方針**。  如需一般的效能概念，請參閱 [Data Lake Store 效能微調指導方針](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-performance-tuning-guidance) 

## <a name="parameters"></a>參數

在執行 Spark 作業時，以下是要增進 ADLS 效能所能微調的最重要設定︰

* **Num-executors** - 可執行的並行工作數目。

* **Executor-memory** - 配置給每個執行程式的記憶體數量。

* **Executor-cores** - 配置給每個執行程式的核心數目。                     

**Num-executors** Num-executors 會設定可平行執行的工作數目上限。  可平行執行的實際工作數目會受限於叢集中可用的記憶體和 CPU 資源。

**Executor-memory** - 這是要配置給每個執行程式的記憶體數量。  每個執行程式所需的記憶體取決於作業。  複雜的作業需要較高的記憶體。  讀取和寫入等簡單作業的記憶體需求會較低。  您可以在 Ambari 中檢視每個執行程式的記憶體數量。  在 Ambari 中，瀏覽至 Spark，然後檢視 [設定] 索引標籤。  

**Executor-cores** 這會設定每個執行程式所使用的核心數量，進而決定每個執行程式可以執行的平行執行緒數目。  例如，如果 executor-cores = 2，則每個執行程式可以在執行程式中執行 2 個平行工作。  所需的 executor-cores 取決於該作業。  需要大量 I/O 的作業所需的每一工作記憶體並不需要太多，因此每個執行程式可以處理更多的平行工作。

依預設，在 HDInsight 上執行 Spark 時，會為每個實體核心定義兩個虛擬 YARN 核心。  這個數量能在並行能力與從多個執行緒切換而來的內容數量取得良好平衡。  

## <a name="guidance"></a>指引

針對 Data Lake Store 中的資料執行 Spark 分析工作負載時，我們建議您使用最新的 HDInsight 版本以取得最佳的 Data Lake Store 效能。 當您的作業較為 I/O 密集時，您可以設定某些參數以提升效能。  Azure Data Lake Store 是具有高擴充性的儲存體平台，可處理高輸送量。  如果作業主要包含讀取和寫入，則為 I/O 提升針對 Azure Data Lake Store 的並行將會提升效能。

有幾種基本的方法可以提升 I/O 密集作業的並行。

**步驟 1︰確定叢集上所執行的應用程式數目** – 您應該要知道有多少個應用程式在叢集上執行，包括目前的應用程式。  每個 Spark 設定的預設值會假設有 4 個應用程式在並行執行。  因此，每個應用程式只能使用 25% 的叢集。  若要獲得更好的效能，您可以變更執行程式的數目來覆寫預設值。  

**步驟 2︰設定 executor-memory** – 首先要設定的是 executor-memory。  記憶體會取決於您要執行的作業。  您可以為每個執行程式配置較少的記憶體，以增加並行數量。  如果您在執行作業時看到記憶體不足的例外狀況，則應該增加此參數的值。  另一個方式是使用擁有較高數量記憶體的叢集，或增加叢集大小，以獲得更多的記憶體。  更多的記憶體就能使用更多的執行程式，亦即會有更多並行能力。

**步驟 3︰設定 executor-cores** – 對於沒有複雜作業的 I/O 密集工作負載，最好是先設定較高的 executor-cores 數值，以增加每個執行程式的平行工作數目。  將 executor-cores 設定為 4 是不錯的開始。   

    executor-cores = 4
增加 executor-cores 的數字可讓您更符合平行處理原則，因此請試試不同的 executor-cores。  對於具有較複雜作業的工作，您應該減少每個執行程式的核心數目。  如果 executor-cores 設定為高於 4，則記憶體回收可能會變得沒有效率而降低效能。

**步驟 4︰決定叢集中的 YARN 記憶體數量** – 這項資訊可在 Ambari 中取得。  瀏覽至 YARN，然後檢視 [設定] 索引標籤。YARN 記憶體會顯示在此視窗中。  
注意︰當您位於此視窗時，您也可以查看預設的 YARN 容器大小。  YARN 容器大小和每個執行程式參數的記憶體相同。

    Total YARN memory = nodes * YARN memory per node
**步驟 5︰計算 num-executors**

**計算記憶體限制** - num-executors 參數會受到記憶體或 CPU 所限制。  記憶體限制取決於應用程式的可用 YARN 記憶體數量。  您應該取得 YARN 記憶體總數，然後除以 executor-memory。  應用程式數目的限制必須取消調整，因此我們除以應用程式數目。

    Memory constraint = (total YARN memory / executor memory) / # of apps   
**計算 CPU 限制**-CPU 限制的計算方式為虛擬核心總數除以每個執行程式的核心數目。  每個實體核心有 2 個虛擬核心。  和記憶體限制類似，我們除以應用程式數目。

    virtual cores = (nodes in cluster * # of physical cores in node * 2)
    CPU constraint = (total virtual cores / # of cores per executor) / # of apps
**設定 num-executors**– num-executors 參數是由記憶體限制和 CPU 限制較小者來決定。 

    num-executors = Min (total virtual Cores / # of cores per executor, available YARN memory / executor-memory)   
設定較高的 num-executors 數值未必能提升效能。  您應該考慮到，新增更多執行程式會對每個額外的執行程式新增額外的負擔，而可能降低效能。  Num-executors 受到叢集資源的限制。    

## <a name="example-calculation"></a>範例計算

假設您目前有由 8 個 D4v2 節點所組成的叢集，其中執行了 2 個應用程式 (包括您要執行的應用程式在內)。  

**步驟 1︰確定叢集上所執行的應用程式數目** – 您知道您的叢集上有 2 個應用程式 (包括您要執行的應用程式在內)。  

**步驟 2︰設定 executor-memory** – 在此範例中，我們判斷 6 GB 的 executor-memory 就足夠 I/O 密集作業使用。  

    executor-memory = 6GB
**步驟 3︰設定 executor-cores** – 這是 I/O 密集作業，因此我們可以將每個執行程式的核心數目設為 4。  將每個執行程式的核心設為大於 4，可能會造成記憶體回收問題。  

    executor-cores = 4
**步驟 4︰決定叢集中的 YARN 記憶體數量** – 我們瀏覽至 Ambari，發現每個 D4v2 有 25 GB 的 YARN 記憶體。  由於有 8 個節點，可用的 YARN 記憶體會乘以 8。

    Total YARN memory = nodes * YARN memory* per node
    Total YARN memory = 8 nodes * 25GB = 200GB
**步驟 5︰計算 num-executors** – num-executors 參數是由記憶體限制和 CPU 限制較小者除以 Spark 上執行的應用程式數目來決定。    

**計算記憶體限制** – 記憶體限制的計算方式為 YARN 記憶體總數除以每個執行程式的記憶體。

    Memory constraint = (total YARN memory / executor memory) / # of apps   
    Memory constraint = (200GB / 6GB) / 2   
    Memory constraint = 16 (rounded)
**計算 CPU 限制**-CPU 限制的計算方式為 YARN 核心總數除以每個執行程式的核心數目。
    
    YARN cores = nodes in cluster * # of cores per node * 2   
    YARN cores = 8 nodes * 8 cores per D14 * 2 = 128
    CPU constraint = (total YARN cores / # of cores per executor) / # of apps
    CPU constraint = (128 / 4) / 2
    CPU constraint = 16
**設定 num-executors**

    num-executors = Min (memory constraint, CPU constraint)
    num-executors = Min (16, 16)
    num-executors = 16    

