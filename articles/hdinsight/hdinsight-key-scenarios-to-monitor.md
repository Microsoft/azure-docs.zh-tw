---
title: 監視叢集效能 - Azure HDInsight
description: 如何監視 HDInsight 叢集的容量和效能。
services: hdinsight
author: maxluk
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/27/2017
ms.author: maxluk
ms.openlocfilehash: 434b3ecf65aaa5ecea81f5a9773f1bc6e8f6f2be
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43092322"
---
# <a name="monitor-cluster-performance"></a>監視叢集效能

監視 HDInsight 叢集的健康情況和效能對於維護最大效能和資源使用率而言是不可或缺的。 監視也可以協助您解決可能發生的編碼或叢集設定錯誤。

下列各節說明如何將叢集負載最佳化、YARN 佇列效率，以及儲存體的存取能力。

## <a name="cluster-loading"></a>叢集負載

Hadoop 叢集應該在叢集的節點之間平衡負載。 此負載平衡可避免處理工作受限於 RAM、CPU 或磁碟資源。

若要高階查看您叢集的節點及其負載，請登入 [Ambari Web UI](hdinsight-hadoop-manage-ambari.md)，然後選取 [主機]。會依主機的完整網域名稱加以列出。 每個主機的操作狀態是依彩色的健康情況指示器來顯示：

| 色彩 | 說明 |
| --- | --- |
| 紅色 | 主機上至少有一個主要元件已關閉。 暫留以查看列出受影響元件的工具提示。 |
| 橙色 | 主機上至少有一個從屬元件已關閉。 暫留以查看列出受影響元件的工具提示。 |
| 黃色 | Ambari 伺服器超過 3 分鐘未收到主機的活動訊號。 |
| 綠色 | 一般執行狀態。 |

您也會看到資料行顯示每個主機的核心數及 RAM 數量，以及磁碟使用量和負載平均。

![主機索引標籤](./media/hdinsight-key-scenarios-to-monitor/hosts-tab.png)

選取任何主機名稱，以詳細查看該主機和其計量上執行的元件。 計量會顯示為可選取的 CPU 使用量時間軸、負載、磁碟使用量、記憶體使用量、網路使用量和程序的數目。

![主機詳細資料](./media/hdinsight-key-scenarios-to-monitor/host-details.png)

如需設定警示和檢視計量的詳細資料，請參閱[使用 Ambari Web UI 來管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari.md)。

## <a name="yarn-queue-configuration"></a>YARN 佇列組態

Hadoop 有各種服務在其分散式平台之間執行。 YARN (Yet Another Resource Negotiator) 會協調這些服務、配置叢集資源，並管理一般資料集的存取。

YARN 會將 JobTracker、資源管理及作業排程/監視的兩個責任分割為兩個精靈：全域 ResourceManager 和每個應用程式 ApplicationMaster (AM)。

ResourceManager 是純排程器，且會單獨仲裁所有競爭應用程式之間的可用資源。 ResourceManager 可確保所有資源一律在使用中、最佳化各種常數，例如 SLA、容量保證等等。 ApplicationMaster 會交涉 ResourceManager 的資源，並使用 NodeManager(s) 來執行及監視容器和其資源耗用量。

當多個租用戶共用大型叢集時，叢集資源會進行競爭。 CapacityScheduler 是隨插即用的排程器，可藉由將要求排入佇列來協助資源共用。 CapacityScheduler 也支援階層式佇列，以確保在允許其他應用程式的佇列使用可用資源之前，在組織的子佇列之間共用資源。

YARN 可讓我們將資源配置給這些佇列，並顯示是否已指派所有可用的資源。 若要檢視您佇列的相關資訊，請登入 Ambari Web UI，然後從頂端功能表中選取 [YARN 佇列管理員]。

![YARN 佇列管理員](./media/hdinsight-key-scenarios-to-monitor/yarn-queue-manager.png)

[YARN 佇列管理員] 頁面左側會顯示您的佇列清單，以及指派給每個佇列的容量百分比。

![YARN 佇列管理員詳細資料頁面](./media/hdinsight-key-scenarios-to-monitor/yarn-queue-manager-details.png)

若要更詳細查看您的佇列，請從 Ambari 儀表板中的左側清單選取 [YARN] 服務。 然後在 [快速連結] 下拉式功能表中，選取作用中節點下的 [ResourceManager UI]。

![ResourceManager UI 功能表連結](./media/hdinsight-key-scenarios-to-monitor/resource-manager-ui-menu.png)

在 ResourceManager UI 中，從左側功能表選取 [排程器]。 您會在 [應用程式佇列] 下方看到您的佇列清單。 您可以在這裡查看每個佇列使用的容量，作業在它們之間散發的情況，以及是否有任何作業為有限資源。

![ResourceManager UI 功能表連結](./media/hdinsight-key-scenarios-to-monitor/resource-manager-ui.png)

## <a name="storage-throttling"></a>儲存體節流

叢集的效能瓶頸可能會發生在儲存層級中。 這種類型的瓶頸最常因為封鎖輸入/輸出 (IO) 作業，在您執行的工作傳送之 IO 超過儲存體服務可以處理的範圍時就會發生這個狀況。 此封鎖會建立 IO 要求的佇列，等候目前 IO 處理完成後才會予以處理。 區塊是由於儲存體節流，這並不是實體的限制，而是服務等級協定 (SLA) 的儲存體服務所加諸的限制。 這項限制可確保沒有任何單一用戶端或租用戶可以獨佔服務。 SLA 會限制 Azure 儲存體每秒的 IO 數 (IOPS) - 如需詳細資訊，請參閱 [Azure 儲存體的擴充和效能目標](https://docs.microsoft.com/azure/storage/storage-scalability-targets)。

如果您是使用 Azure 儲存體，如需監視儲存體相關問題的資訊 (包括節流)，請參閱[監視、診斷 Microsoft Azure 儲存體，及對其進行疑難排解](https://docs.microsoft.com/azure/storage/storage-monitoring-diagnosing-troubleshooting)。

如果您叢集的備份存放區是 Azure Data Lake Store (ADLS)，您的節流很有可能是因為頻寬限制。 在此情況下，透過觀察工作記錄中的節流錯誤即可識別節流。 如需 ADLS，請參閱這些文章中的節流一節以了解適當服務：

* [HDInsight 和 Azure Data Lake Store 上的 Hive 效能微調方針](../data-lake-store/data-lake-store-performance-tuning-hive.md)
* [HDInsight 和 Azure Data Lake Store 上的 MapReduce 效能微調方針](../data-lake-store/data-lake-store-performance-tuning-mapreduce.md)
* [HDInsight 和 Azure Data Lake Store 上的 Storm 效能微調方針](../data-lake-store/data-lake-store-performance-tuning-storm.md)

## <a name="next-steps"></a>後續步驟

請造訪下列連結以取得關於疑難排解和監視您叢集的詳細資訊：

* [分析 HDInsight 記錄檔](hdinsight-debug-jobs.md)
* [利用 YARN 記錄為應用程式偵錯](hdinsight-hadoop-access-yarn-app-logs-linux.md)
* [在以 Linux 為基礎的 HDInsight 上啟用 Hadoop 服務的堆積傾印](hdinsight-hadoop-collect-debug-heap-dump-linux.md)
