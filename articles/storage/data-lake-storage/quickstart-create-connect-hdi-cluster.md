---
title: Hadoop、Spark、Kafka、HBase 或 R 伺服器的叢集設定 - Azure HDInsight
description: 從瀏覽器、Azure CLI、Azure PowerShell、REST 或 SDK 設定 HDInsight 的 Hadoop、Kafka、Spark、HBase、R 伺服器或 Storm 叢集。
keywords: hadoop 叢集設定、kafka 叢集設定、spark 叢集設定、什麼是 hadoop 中的叢集
services: storage
author: jamesbak
tags: azure-portal
ms.component: data-lake-storage-gen2
ms.service: storage
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: jamesbak
ms.openlocfilehash: e0816e8609ba1ab0ef1b4f685731339378fee844
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525581"
---
# <a name="quickstart-set-up-clusters-in-hdinsight"></a>快速入門：在 HDInsight 中設定叢集

在本快速入門中，您會了解如何在 HDInsight 中使用 Hadoop、Spark、Kafka、互動式查詢、HBase、R 伺服器或 Storm 來安裝並設定叢集。 您也會了解如何自訂叢集、將它們加入網域，並附加至 [Azure Data Lake Storage Gen2 預覽](introduction.md)。

Hadoop 叢集由數個虛擬機器 (節點) 組成，可用於分散處理作業。 Azure HDInsight 會處理個別節點所安裝和設定的實作細節，您只需要提供一般設定資訊即可。

> [!IMPORTANT]
>HDInsight 叢集的計費起自叢集建立時，終至叢集刪除時。 計費是以每分鐘按比例計算，因此不再使用時，請一律刪除您的叢集。 了解如何[刪除叢集。](../../hdinsight/hdinsight-delete-cluster.md)

在本快速入門中，Data Lake Storage 會作為資料層。 Data Lake Storage 利用其階層式命名空間服務和 [Hadoop 驅動程式](abfs-driver.md)，針對分散式處理和分析進行最佳化。 即使將 HDInsight 叢集刪除，Data Lake Storage 中儲存的資料仍然存在。

## <a name="cluster-setup-methods"></a>叢集設定方法

下表顯示可用來設定 HDInsight 叢集的不同方法。

| 叢集建立方法 | Web 瀏覽器 | 命令列 | REST API | SDK | 
| --- |:---:|:---:|:---:|:---:|
| [Azure 入口網站](../../hdinsight/hdinsight-hadoop-create-linux-clusters-portal.md) |✔ |&nbsp; |&nbsp; |&nbsp; |
| [Azure Data Factory](../../hdinsight/hdinsight-hadoop-create-linux-clusters-adf.md) |✔ |✔ |✔ |✔ |
| [Azure CLI (1.0 版)](../../hdinsight/hdinsight-hadoop-create-linux-clusters-azure-cli.md) |&nbsp; |✔ |&nbsp; |&nbsp; |
| [Azure PowerShell](../../hdinsight/hdinsight-hadoop-create-linux-clusters-azure-powershell.md) |&nbsp; |✔ |&nbsp; |&nbsp; |
| [cURL](../../hdinsight/hdinsight-hadoop-create-linux-clusters-curl-rest.md) |&nbsp; |✔ |✔ |&nbsp; |
| [Azure 資源管理員範本](../../hdinsight/hdinsight-hadoop-create-linux-clusters-arm-templates.md) |&nbsp; |✔ |&nbsp; |&nbsp; |

## <a name="quick-create-basic-cluster-setup"></a>快速建立：基本的叢集設定

本文會逐步引導您完成 [Azure 入口網站](https://portal.azure.com)中的設定，您可以在此入口網站中使用 [Quick create] \(快速建立\) 或 [Custom] \(自訂\) 建立 HDInsight 叢集。 

![hdinsight 建立選項自訂快速建立](media/quickstart-create-connect-hdi-cluster/hdinsight-creation-options.png)

依照畫面上指示執行基本的叢集設定。 以下是下列各項的詳細資訊：

* [資源群組名稱](#resource-group-name)
* [叢集類型和設定](#cluster-types) 
* [叢集登入和 SSH 使用者名稱](#cluster-login-and-ssh-user-name)
* [位置](#location)

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [HDInsight 3.3 淘汰](../../hdinsight/hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="resource-group-name"></a>資源群組名稱

[Azure Resource Manager](../../azure-resource-manager/resource-group-overview.md) 可協助您將應用程式中的資源做為群組使用，稱為 Azure 資源群組。 您可以在單一、協調的作業中，將應用程式的所有資源進行部署、更新、監視或刪除。

## <a name="cluster-types"></a>叢集類型和設定
Azure HDInsight 目前提供下列的叢集類型，每種都有一組提供特定功能的元件。

> [!IMPORTANT]
> HDInsight 叢集有多種類型，每種類型各適合單一工作負載或技術。 沒有任何支援方法可建立結合多個類型的叢集，例如在一個叢集上並存 Storm 和 HBase。 如果您的解決方案需要會分散到多個 HDInsight 叢集類型的技術，[Azure 虛擬網路](https://docs.microsoft.com/azure/virtual-network)可以連接必要的叢集類型。 
>
>

| 叢集類型 | 功能 |
| --- | --- |
| [Hadoop](../../hdinsight/hadoop/apache-hadoop-introduction.md) |批次查詢和分析儲存的資料 |
| [HBase](../../hdinsight/hbase/apache-hbase-overview.md) |處理大量無綱要的 NoSQL 資料 |
| [互動式查詢](../../hdinsight/interactive-query/apache-interactive-query-get-started.md) |更快速之互動式 Hive 查詢的記憶體內快取 |
| [Kafka](../../hdinsight/kafka/apache-kafka-introduction.md) | 可用來建置即時串流資料管線和應用程式的分散式串流平台 |
| [R 伺服器](../../hdinsight/r-server/r-server-overview.md) |各種巨量資料統計資料、預測模型和機器學習功能 |
| [Spark](../../hdinsight/spark/apache-spark-overview.md) |記憶體內處理、互動式查詢、微批次串流處理 |
| [Storm](../../hdinsight/storm/apache-storm-overview.md) |即時事件處理 |

### <a name="hdinsight-version"></a>HDInsight 版本

選擇此叢集的 HDInsight 版本。 如需詳細資訊，請參閱[支援的 HDInsight 版本](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions)。

### <a name="enterprise-security-package"></a>企業安全性套件

對於 Hadoop、Spark 和互動式查詢叢集類型，您可以選擇啟用 [企業安全性套件]。 此套件使用 Apache Ranger 並與 Azure Active Directory 整合，讓您可選擇更安全的叢集設定。 如需詳細資訊，請參閱 [Azure HDInsight 中的企業安全性套件](../../hdinsight/domain-joined/apache-domain-joined-introduction.md)。

![hdinsight 建立選擇企業安全性套件的選項](./media/quickstart-create-connect-hdi-cluster/hdinsight-creation-enterprise-security-package.png)

如需建立已加入網域之 HDInsight 叢集的詳細資訊，請參閱[建立已加入網域的 HDInsight 沙箱環境](../../hdinsight/domain-joined/apache-domain-joined-configure.md)。

## <a name="cluster-login-and-ssh-user-name"></a>叢集登入和 SSH 使用者名稱

使用 HDInsight 叢集，您可以在建立叢集期間設定兩個使用者帳戶：

* HTTP 使用者：預設使用者名稱為 *admin*。使用 Azure 入口網站上的基本組態。 有時稱之為「叢集使用者」。
* SSH 使用者 (Linux 叢集)：用來透過 SSH 連線到叢集。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md)。

企業安全性套件可讓您整合 HDInsight 與 Active Directory 及 Apache Ranger。 使用企業安全性套件可以建立多個使用者。

## <a name="location"></a>叢集與儲存體的位置 (區域)

不需明確指定叢集位置；叢集與預設儲存體位於相同的位置。 如需支援的區域清單，請按一下 [HDInsight 價格](https://go.microsoft.com/fwLink/?LinkID=282635&clcid=0x409)中的 [區域] 下拉式清單。

## <a name="storage-endpoints-for-clusters"></a>叢集的儲存體端點

內部部署安裝的 Hadoop 叢集使用 Hadoop 分散式檔案系統 (HDFS) 作為叢集上的儲存體，但在雲端中，您可以使用已連接到叢集的儲存體端點。 HDInsight 叢集會使用 [Data Lake Storage Gen2](abfs-driver.md) 或 [Azure 儲存體中的 Blob](../../hdinsight/hdinsight-hadoop-use-blob-storage.md)。 使用 Azure 儲存體或 Data Lake Storage 表示您可以放心地刪除用於計算的 HDInsight 叢集，同時仍會留存您的資料。

> [!WARNING]
> 不支援在與 HDInsight 叢集不同的位置中使用其他儲存體帳戶。

在設定期間，您要針對預設儲存體端點指定 Data Lake Storage。 預設儲存體包含應用程式與系統記錄檔。 您也可以選擇指定叢集可存取的其他已連結 Azure Data Lake Storage 帳戶。 HDInsight 叢集與相依的儲存體帳戶必須位於相同的 Azure 位置。

![叢集儲存體設定：HDFS 相容的儲存體端點](media/quickstart-create-connect-hdi-cluster/hdinsight-cluster-creation-storage2.png)

> [!IMPORTANT]
> 請務必**停用 Data Lake Store 存取**。 此設定是指舊的 Data Lake Store 功能，且必須加以停用，Data Lake Storage 功能才可正常運作。

[!INCLUDE [secure-transfer-enabled-storage-account](../../../includes/hdinsight-secure-transfer.md)]

### <a name="optional-metastores"></a>選擇性中繼存放區
您可以建立選擇性的 Hive 或 Oozie 中繼存放區。 不過，並非所有叢集類型都支援中繼存放區，且 Azure SQL 資料倉儲不相容於中繼存放區。 

如需詳細資訊，請參閱[在 Azure HDInsight 中使用外部中繼資料存放區](../../hdinsight/hdinsight-use-external-metadata-stores.md)。

> [!IMPORTANT]
> 在建立自訂中繼存放區時，資料庫名稱請勿使用破折號、連字號或空格。 這可能會導致叢集建立程序失敗。

### <a name="use-hiveoozie-metastore"></a>Hive 中繼存放區

如果想要在刪除 HDInsight 叢集之後保留 Hive 資料表，請使用自訂的中繼存放區。 您可以接著將中繼存放區附加至另一個 HDInsight 叢集。

針對某個 HDInsight 叢集版本建立的 HDInsight 中繼存放區，不能在不同的 HDInsight 叢集版本之間共用。 如需 HDInsight 版本清單，請參閱[支援的 HDInsight 版本](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions)。

### <a name="oozie-metastore"></a>Oozie 中繼存放區

為提升使用 Oozie 時的效能，請使用自訂的中繼存放區。 在您刪除叢集後，中繼存放區也可提供 Oozie 作業資料的存取。 

> [!IMPORTANT]
> 您無法重複使用自訂的 Oozie 中繼存放區。 若要使用自訂的 Oozie 中繼存放區，您必須在建立 HDInsight 叢集時提供空的 Azure SQL Database。

## <a name="configure-cluster-size"></a>設定叢集大小

只要叢集存在，就會針對您的節點使用量收費。 建立叢集後就開始計費，並在叢集刪除後停止計費。 無法取消配置或保留叢集。


### <a name="number-of-nodes-for-each-cluster-type"></a>每個叢集類型的節點數目
每個叢集類型都有自己的節點數目、節點術語和預設 VM 大小。 下表中各節點類型的節點數目位於括號中。

| 類型 | 節點 | 圖表 |
| --- | --- | --- |
| Hadoop |前端節點 (2)、資料節點 (1+) |![HDInsight Hadoop 叢集節點](media/quickstart-create-connect-hdi-cluster/hdinsight-hadoop-cluster-type-nodes.png) |
| hbase |前端伺服器 (2)、區域伺服器 (1+)、主要/Zookeeper 節點 (3) |![HDInsight HBase 叢集節點](media/quickstart-create-connect-hdi-cluster/hdinsight-hbase-cluster-type-setup.png) |
| Storm |Nimbus 節點 (2)、監督員伺服器 (1+)、Zookeeper 節點 (3) |![HDInsight Storm 叢集節點](media/quickstart-create-connect-hdi-cluster/hdinsight-storm-cluster-type-setup.png) |
| Spark |前端節點 (2)、背景工作角色節點 (1+)、Zookeeper 節點 (3) (A1 Zookeeper VM 大小不限) |![HDInsight Spark 叢集節點](media/quickstart-create-connect-hdi-cluster/hdinsight-spark-cluster-type-setup.png) |

如需詳細資訊，請參閱＜HDInsight 中的 Hadoop 元件和版本是什麼？＞中的[叢集的預設節點設定和虛擬機器大小](../../hdinsight/hdinsight-component-versioning.md#default-node-configuration-and-virtual-machine-sizes-for-clusters)。

HDInsight 叢集的成本是由節點數和節點的虛擬機器大小來決定。 

不同的叢集類型具有不同的節點類型、節點數目和節點大小：
* Hadoop 叢集類型的預設值： 
    * 兩個*前端節點*  
    * 四個*資料節點*
* Storm 叢集類型的預設值： 
    * 兩個 *Nimbus 節點*
    * 三個 *ZooKeeper 節點*
    * 四個*監督員節點* 

如果您只是在試用 HDInsight，建議您使用一個資料節點。 如需關於 HDInsight 定價的詳細資訊，請參閱 [HDInsight 定價](https://go.microsoft.com/fwLink/?LinkID=282635&clcid=0x409)。

> [!NOTE]
> 叢集大小限制會隨著 Azure 訂用帳戶而有所不同。 若要提高限制，請與 [Azure 帳務支援人員](https://docs.microsoft.com/azure/azure-supportability/how-to-create-azure-support-request)連絡。
>

使用 Azure 入口網站設定叢集時，節點大小會透過 [節點定價層] 刀鋒視窗公開。 在入口網站中，您也可以查看與不同節點大小相關聯的成本。 

![HDinsight VM 節點大小](media/quickstart-create-connect-hdi-cluster/hdinsight-node-sizes.png)

### <a name="virtual-machine-sizes"></a>虛擬機器大小 
當您部署叢集時，請依據計劃部署的解決方案選擇計算資源。 下列 VM 可用於 HDInsight 叢集：
* A 和 D1-4 系列 VM：[一般用途 Linux VM 大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-general)
* D11-14 系列 VM：[記憶體最佳化 Linux VM 大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-memory)

如需使用不同的 SDK 或使用 Azure PowerShell 建立叢集時應用來指定 VM 大小的值，請參閱[使用於 HDInsight 叢集的 VM 大小](../../cloud-services/cloud-services-sizes-specs.md#size-tables)。 在此連結的文件中，請使用資料表中 **Size (大小)** 資料行的值。

> [!IMPORTANT]
> 如果您的叢集需要 32 個以上的背景工作角色節點，則必須選取具有至少 8 個核心和 14 GB RAM 的前端節點大小。
>
>

如需相關資訊，請參閱[虛擬機器的大小](../../virtual-machines/windows/sizes.md)。 如需各式大小的價格資訊，請參閱 [HDInsight 價格](https://azure.microsoft.com/pricing/details/hdinsight)。   

## <a name="custom-cluster-setup"></a>自訂叢集設定
自訂叢集設定是以 [Quick create] \(快速建立\) 的設定為基礎，並加入下列選項：
- [HDInsight 應用程式](#hdinsight-applications)
- [叢集大小](#cluster-size)
- 進階設定
  - [指令碼動作](#customize-clusters-using-script-action)
  - [虛擬網路](#use-virtual-network)

## <a name="install-hdinsight-applications-on-clusters"></a>在叢集上安裝 HDInsight 應用程式

HDInsight 應用程式是使用者可以在以 Linux 為基礎的 HDInsight 叢集上安裝的應用程式。 您可以使用由 Microsoft、協力廠商所提供或您自己開發的應用程式。 如需詳細資訊，請參閱[在 Azure HDInsight 上安裝第三方 Hadoop 應用程式](../../hdinsight/hdinsight-apps-install-applications.md)。

大部分的 HDInsight 應用程式會安裝在空白的邊緣節點。  空白的邊緣節點是一部 Linux 虛擬機器，其中已安裝及設定和前端節點相同的用戶端工具。 您可以使用邊緣節點來存取叢集、測試用戶端應用程式，以及裝載用戶端應用程式。 如需詳細資訊，請參閱 [Use empty edge nodes in HDInsight (在 HDInsight 中使用空白的邊緣節點)](../../hdinsight/hdinsight-apps-use-edge-node.md)。

## <a name="advanced-settings-script-actions"></a>進階設定：指令碼動作

您可以在建立期間使用指令碼來安裝其他元件或自訂組態。 這類指令碼可透過 **指令碼動作**叫用，指令碼動作是一個組態選項，其可從 Azure 入口網站、HDInsight Windows PowerShell Cmdlet 或 HDInsight .NET SDK 使用。 如需詳細資訊，請參閱 [使用指令碼動作自訂 HDInsight 叢集](../../hdinsight/hdinsight-hadoop-customize-cluster-linux.md)。

您可以使用 Java 封存 (JAR) 檔案形式在叢集上執行一些原生 Java 元件 (例如 Mahout 和 Cascading)。 這些 JAR 檔案可以配送至 Azure 儲存體或 Azure Data Lake Storage，並透過 Hadoop 作業提交機制提交至 HDInsight 叢集。 如需詳細資訊，請參閱 [以程式設計方式提交 Hadoop 工作](../../hdinsight/hadoop/submit-apache-hadoop-jobs-programmatically.md)。

> [!NOTE]
> 如果您在將 JAR 檔案部署至 HDInsight 叢集，或在 HDInsight 叢集上呼叫 JAR 檔案時發生問題，請連絡 [Microsoft 支援](https://azure.microsoft.com/support/options/)。
>
> Cascading 不受 HDInsight 支援，而且不符合「Microsoft 支援」的資格。 如需所支援元件的清單，請參閱 [HDInsight 所提供叢集版本的新功能](../../hdinsight/hdinsight-component-versioning.md)。
>
>

有時候，您可能要在建立程序期間設定下列組態檔：

* clusterIdentity.xml
* core-site.xml
* gateway.xml
* hbase-env.xml
* hbase-site.xml
* hdfs-site.xml
* hive-env.xml
* hive-site.xml
* mapred-site
* oozie-site.xml
* oozie-env.xml
* storm-site.xml
* tez-site.xml
* webhcat-site.xml
* yarn-site.xml

如需詳細資訊，請參閱 [使用 Bootstrap 自訂 HDInsight 叢集](../../hdinsight/hdinsight-hadoop-customize-cluster-bootstrap.md)。

## <a name="advanced-settings-extend-clusters-with-a-virtual-network"></a>進階設定：使用虛擬網路擴充叢集
如果您的解決方案需要會分散到多個 HDInsight 叢集類型的技術，[Azure 虛擬網路](../../hdinsight/https://docs.microsoft.com/azure/virtual-network)可以連接必要的叢集類型。 此組態可讓叢集以及其中部署的任何程式碼直接彼此通訊。

如需如何搭配使用 Azure 虛擬網路與 HDInsight 的詳細資訊，請參閱[使用 Azure 虛擬網路擴充 HDInsight](../../hdinsight/hdinsight-extend-hadoop-virtual-network.md)。

如需在 Azure 虛擬網路內使用兩個叢集類型的範例，請參閱[使用 Spark 結構化串流搭配 Kafka](../../hdinsight/hdinsight-apache-kafka-spark-structured-streaming.md)。 如需搭配虛擬網路使用 HDInsight 的詳細資訊 (包含虛擬網路的特定組態需求)，請參閱 [使用 Azure 虛擬網路延伸 HDInsight 功能](../../hdinsight/hdinsight-extend-hadoop-virtual-network.md)。

## <a name="troubleshoot-access-control-issues"></a>針對存取控制問題進行疑難排解

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](../../hdinsight/hdinsight-administer-use-portal-linux.md#create-clusters)。

## <a name="next-steps"></a>後續步驟

- [Azure Data Lake Storage Gen2 的 ABFS Hadoop Filesystem 驅動程式](abfs-driver.md)
- [教學課程：使用 Azure HDInsight 上的 Apache Hive 來擷取、轉換和載入資料](tutorial-extract-transform-load-hive.md)
- [什麼是 HDInsight、Hadoop 生態系統以及 Hadoop 叢集？](../../hdinsight/hadoop/apache-hadoop-introduction.md)
- [開始在 HDInsight 中使用 Hadoop](../../hdinsight/hadoop/apache-hadoop-linux-tutorial-get-started.md)
- [從 Windows PC 在 HDInsight 上的 Hadoop 中作業](../../hdinsight/hdinsight-hadoop-windows-tools.md)
