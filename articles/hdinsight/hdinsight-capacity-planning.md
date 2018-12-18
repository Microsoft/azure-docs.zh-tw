---
title: Azure HDInsight 中的叢集容量規劃
description: 如何指定 HDInsight 叢集的容量和效能。
services: hdinsight
author: maxluk
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/22/2017
ms.author: maxluk
ms.openlocfilehash: 4438cff0dcf5e896f39729d9871d4deb3207b4b8
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43107990"
---
# <a name="capacity-planning-for-hdinsight-clusters"></a>HDInsight 叢集的容量規劃

在部署 HDInsight 叢集之前，請規劃所需的叢集容量，方法為判斷所需的效能和縮放比例。 此計劃可協助將可用性和成本最佳化。 部署之後，就無法變更某些叢集容量決策。 如果效能參數變更，就可以拆卸並重新建立叢集，而不會遺失預存的資料。

容量規劃的主要問題是：

* 您應該在哪些地理區域中部署叢集？
* 您需要多少儲存空間？
* 您應該部署哪種叢集類型？
* 您的叢集節點應該使用什麼尺寸和類型的虛擬機器 (VM)？
* 您的叢集應該有幾個背景工作角色？

## <a name="choose-an-azure-region"></a>選擇 Azure 區域

Azure 地區決定您叢集實際佈建的位置。 若要將讀取和寫入的延遲降至最低，叢集就應該接近您的資料。

HDInsight 可在多個 Azure 區域中使用。 若要尋找最接近的區域，請參閱[依區域提供的 Azure 產品](https://azure.microsoft.com/regions/services/)中資料 + 分析下的 HDInsight Linux 項目。

## <a name="choose-storage-location-and-size"></a>選擇存放位置和大小

### <a name="location-of-default-storage"></a>預設儲存體的位置

預設儲存體 (無論是 Azure 儲存體帳戶或 Azure Data Lake Store) 必須與您的叢集在相同的位置。 Azure 儲存體可在所有位置使用。 Data Lake Store 可在某些區域中使用 - 請在[依區域提供的 Azure 產品](https://azure.microsoft.com/regions/services/)中的儲存體下參閱目前的 Data Lake Store 可用性。

### <a name="location-of-existing-data"></a>現有資料的位置

如果您已經有包含資料的儲存體帳戶或 Data Lake Store，並需要使用這個儲存體作為叢集的預設儲存體，就必須在該相同的位置部署您的叢集。

### <a name="storage-size"></a>儲存體大小

部署 HDInsight 叢集之後，您可以連接額外的 Azure 儲存體帳戶，或存取其他 Data Lake Store。 所有的儲存體帳戶都必須與您的叢集位於相同的位置。 Data Lake Store 可位於不同的位置，雖然這樣可能會導致某些資料的讀取/寫入延遲。

Azure 儲存體有某些[容量限制](../azure-subscription-service-limits.md#storage-limits)，但 Data Lake Store 幾乎不受限制。

叢集可以存取不同儲存體帳戶的組合。 典型的範例包括：

* 當資料量很可能超過單一 blob 儲存體容器的儲存體容量。
* 當 blob 容器的存取速率可能會超過節流發生的閾值。
* 當您需要製作資料時，已上傳到可供叢集使用的 blob 容器。
* 當您基於安全性的原因或要簡化系統管理而需要將儲存體的不同部分進行隔離。

針對 48 節點叢集，建議使用 4 到 8 個儲存體帳戶。 雖然可能已經有足夠的儲存體總計，每個儲存體帳戶都會提供額外的網路頻寬供計算節點使用。 當您有多個儲存體帳戶時，請對每個儲存體帳戶使用隨機名稱，不含前置詞。 隨機命名的目的在於減少儲存體瓶頸 (節流) 或所有帳戶發生一般模式失敗的機會。 為提升效能，每個儲存體帳戶僅使用一個容器。

## <a name="choose-a-cluster-type"></a>選擇叢集類型

叢集類型會決定您 HDInsight 叢集設定要執行的工作負載，例如 Hadoop、Storm、Kafka 或 Spark。 如需可用叢集類型的詳細說明，請參閱 [Azure HDInsight 簡介](hadoop/apache-hadoop-introduction.md#cluster-types-in-hdinsight)。 每個叢集類型都有特定的部署拓撲，其中包含節點數目和大小的需求。

## <a name="choose-the-vm-size-and-type"></a>選擇 VM 大小與類型

每個叢集類型都具有一組節點類型，且每個節點類型都有其 VM 大小和類型的特定選項。

若要判斷您應用程式的最佳叢集大小，您可以效能評定叢集容量，並依指示增加大小。 例如，您可以使用模擬的工作負載或 Canary 查詢。 使用模擬工作負載時，您要在不同大小的叢集上執行您預期的工作負載，逐漸增加大小直到觸達所需的效能為止。 可以在其他生產查詢之間定期插入 Canary 查詢，以說明叢集是否有足夠的資源。

VM 大小與類型是由 CPU 處理能力、RAM 大小和網路延遲所決定：

* CPU：VM 大小會規定核心的數目。 核心越多，每個節點可達到的平行計算程度就越大。 此外，某些 VM 類型具有更快的核心。

* RAM：VM 大小也會決定 VM 可用的 RAM 數量。 針對將資料儲存在記憶體以進行處理的工作負載，並非從磁碟讀取，請確保背景工作節點有足夠的記憶體來容納資料。

* 網路：對於大部分的叢集類型而言，由叢集處理的資料不是在本機磁碟上，而是在諸如 Data Lake Store 或 Azure 儲存體等外部儲存體服務中。 請考慮節點 VM 與儲存體服務之間的網路頻寬和輸送量。 可供 VM 使用的網路頻寬通常會隨著較大的大小而增加。 如需詳細資訊，請參閱 [VM 大小概觀](https://docs.microsoft.com/azure/virtual-machines/linux/sizes)。

## <a name="choose-the-cluster-scale"></a>選擇叢集縮放比例

叢集的縮放比例取決於其 VM 節點的數量。 針對所有的叢集類型，有些叢集類型具有特定的縮放比例，以及支援相應放大的節點類型。例如，叢集可能需要正好三個 ZooKeeper 節點或兩個 Head 節點。 以分散式方式進行資料處理的背景工作節點可受益於相應放大，方法是新增其他背景工作節點。

根據您的叢集類型，增加背景工作節點數可新增額外的計算容量 (例如更多核心)，但也可以新增至整個叢集所需的記憶體總數，以支援記憶體內正在進行處理的資料儲存體。 如同 VM 大小和類型的選擇，通常會使用模擬的工作負載或 Canary 查詢，以實證方式達到選取正確的叢集縮放比例。

您可以相應放大叢集以符合尖峰負載需求，然後當不再需要這些額外的節點時，將其相應縮小。 如需詳細資訊，請參閱[調整 HDInsight 叢集](hdinsight-scaling-best-practices.md)。

### <a name="cluster-lifecycle"></a>叢集生命週期

叢集的存留期需要收費。 如果您只需要叢集在特定時間開機及執行，可以[使用 Azure Data Factory 來建立隨需叢集](hdinsight-hadoop-create-linux-clusters-adf.md)。 您也可以建立 PowerShell 指令碼來佈建和刪除您的叢集，並使用 [Azure 自動化](https://azure.microsoft.com/services/automation/)來排程這些指令碼。

> [!NOTE]
> 刪除叢集時，也會刪除其預設的 Hive 中繼存放區。 若要保存中繼存放區以進行下一次重新建立叢集，請使用外部中繼資料存放區，例如 Azure 資料庫或 Oozie。
<!-- see [Using external metadata stores](hdinsight-using-external-metadata-stores.md). -->

### <a name="isolate-cluster-job-errors"></a>找出叢集作業錯誤

有時會因多個對應平行執行而發生錯誤，從而減少多節點叢集上的元件。 若要協助隔離問題，請嘗試在單一節點叢集上同時執行多個作業的分散式測試，然後延伸這種方法，在包含多個節點的叢集上同時執行多個作業。 若要在 Azure 中建立單一節點 HDInsight 叢集，請使用進階選項。

您也可以在本機電腦上安裝單一節點開發環境，並在該處測試解決方案。 Hortonworks 提供的單一節點本機開發環境適用於以 Hadoop 作為基礎的解決方案，適合初期開發、概念證明及測試。 如需詳細資訊，請參閱 [Hortonworks 沙箱](http://hortonworks.com/products/hortonworks-sandbox/)。

若要識別單一節點本機叢集上的問題，您可以將失敗的作業重新執行及調整輸入資料，或使用較小的資料集。 執行這些作業的方式取決於平台和應用程式類型。

## <a name="quotas"></a>配額

決定您的目標叢集 VM 大小、縮放比例和類型之後，請檢查您訂用帳戶的目前配額容量限制。 當您觸達配額限制時，可能無法部署新的叢集，或新增更多背景工作節點來將現有的叢集相應放大。 已觸達之最常見配額限制是 CPU 核心配額，存在於訂用帳戶、區域和 VM 系列層級。 例如，您的訂用帳戶可能有 200 個核心的總計限制，且在您的區域為 30 個核心的限制，以及 VM 執行個體上 30 個核心的限制。 您可以[連絡支援人員以要求增加配額](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request)。

不過，有一些固定的配額限制，例如單一 Azure 訂用帳戶最多可以有 10,000 個核心。 如需這些限制的詳細資料，請參閱 [Azure 訂用帳戶和服務限制、配額及條件約束](https://docs.microsoft.com/azure/azure-subscription-service-limits#limits-and-the-azure-resource-manager)。

## <a name="next-steps"></a>後續步驟

* [使用 Hadoop、Spark 及 Kafka 等在 HDInsight 中設定叢集](hdinsight-hadoop-provision-linux-clusters.md)：了解如何在 HDInsight 中使用 Hadoop、Spark、Kafka、Interactive Hive、HBase、ML 服務或 Storm 安裝並設定叢集。
* [監視叢集效能](hdinsight-key-scenarios-to-monitor.md)：了解重要情節，以監視可能會影響叢集容量的 HDInsight 叢集。
