---
title: HDInsight 上 Hadoop 服務所使用的連接埠 - Azure
description: 在 HDInsight 上執行的 Hadoop 服務所使用的連接埠清單。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: jasonh
ms.openlocfilehash: 3dea97d674c2027307ac7eb7cae34b6664e40b0b
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43108115"
---
# <a name="ports-used-by-hadoop-services-on-hdinsight"></a>HDInsight 上 Hadoop 服務所使用的連接埠

本文件提供在以 Linux 為基礎的 HDInsight 叢集上執行的 Hadoop 服務所使用的連接埠清單。 此外，也提供用來連線到使用 SSH 之叢集的連接埠相關資訊。

## <a name="public-ports-vs-non-public-ports"></a>公用連接埠與非公用連接埠

以 Linux 為基礎的 HDInsight 叢集只會在網際網路上公開三個公開連接埠：22、23 和 443。 這些連接埠用來使用 SSH 以及透過安全 HTTPS 通訊協定公開的服務安全地存取叢集。

在內部，HDInsight 由數個在 Azure 虛擬網路上執行的 Azure 虛擬機器 (叢集內的節點) 實作。 您可以從虛擬網路存取不是透過網際網路公開的連接埠。 例如，如果您使用 SSH 連接到其中一個前端節點，您可以從此前端節點直接存取在叢集節點上執行的服務。

> [!IMPORTANT]
> 如果您沒有將 Azure 虛擬網路指定為 HDInsight 的設定選項，則會自動建立一個。 不過，您無法將其他電腦 (例如其他 Azure 虛擬機器或用戶端開發電腦) 加入此虛擬網路。


若要將其他電腦加入至虛擬網路，您必須先建立虛擬網路，然後在建立 HDInsight 叢集時進行指定。 如需詳細資訊，請參閱 [使用 Azure 虛擬網路延伸 HDInsight 功能](hdinsight-extend-hadoop-virtual-network.md)

## <a name="public-ports"></a>公用連接埠

HDInsight 叢集中的所有節點都位於 Azure 虛擬網路中，無法直接從網際網路存取。 公用閘道提供下列連接埠 (常見於所有的 HDInsight 叢集類型) 的網際網路存取。

| 服務 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| sshd |22 |SSH |將用戶端連接到主要前端節點上的 sshd。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。 |
| sshd |22 |SSH |將用戶端連接至邊緣節點上的 sshd。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。 |
| sshd |23 |SSH |將用戶端連接到次要前端節點上的 sshd。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。 |
| Ambari |443 |HTTPS |Ambari Web UI。 請參閱 [使用 Ambari Web UI 管理 HDInsight](hdinsight-hadoop-manage-ambari.md) |
| Ambari |443 |HTTPS |Ambari REST API。 請參閱 [使用 Ambari REST API 管理 HDInsight](hdinsight-hadoop-manage-ambari-rest-api.md) |
| WebHCat |443 |HTTPS |HCatalog REST API。 請參閱[搭配使用 Hive 與 Curl](hadoop/apache-hadoop-use-pig-curl.md)、[搭配使用 Pig 與 Curl](hadoop/apache-hadoop-use-pig-curl.md)、[搭配使用 MapReduce 與 Curl](hadoop/apache-hadoop-use-mapreduce-curl.md) |
| HiveServer2 |443 |ODBC |使用 ODBC 連接至 Hive。 請參閱 [使用 Microsoft ODBC 驅動程式將 Excel 連接到 HDInsight](hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)。 |
| HiveServer2 |443 |JDBC |使用 JDBC 連接至 Hive。 請參閱 [使用 Hive JDBC 驅動程式連接到 HDInsight 上的 Hive](hadoop/apache-hadoop-connect-hive-jdbc-driver.md) |

下列各項適用於特定叢集類型︰

| 服務 | Port | 通訊協定 | 叢集類型 | 說明 |
| --- | --- | --- | --- | --- |
| Stargate |443 |HTTPS |hbase |HBase REST API。 請參閱 [開始使用 HBase](hbase/apache-hbase-tutorial-get-started-linux.md) |
| Livy |443 |HTTPS |Spark |Spark REST API。 請參閱 [使用 Livy 從遠端提交 Spark 作業](spark/apache-spark-livy-rest-interface.md) |
| Spark Thrift 伺服器 |443 |HTTPS |Spark |用來提交 Hive 查詢的 Spark Thrift 伺服器。 請參閱[搭配使用 Beeline 與 HDInsight 上的 Hive](hadoop/apache-hadoop-use-hive-beeline.md) |
| Storm |443 |HTTPS |Storm |Storm Web UI。 請參閱 [部署和管理 HDInsight 上的 Storm 拓撲](storm/apache-storm-deploy-monitor-topology-linux.md) |

### <a name="authentication"></a>驗證

在網際網路上公開的所有服務都必須經過驗證︰

| Port | 認證 |
| --- | --- |
| 22 或 23 |在叢集建立期間指定的 SSH 使用者認證 |
| 443 |在叢集建立期間設定的登入名稱 (預設值：admin) 和密碼 |

## <a name="non-public-ports"></a>非公用連接埠

> [!NOTE]
> 部分服務只能在特定叢集類型上使用。 例如，HBase 只能在 HBase 叢集類型上使用。

> [!IMPORTANT]
> 某些服務一次只能在一個前端節點上執行。 如果您嘗試連接到主要前端節點上的服務並收到錯誤，請使用次要前端節點重試。

### <a name="ambari"></a>Ambari

| 服務 | 節點 | Port | URL 路徑 | 通訊協定 | 
| --- | --- | --- | --- | --- |
| Ambari Web UI | 前端節點 | 8080 | / | HTTP |
| Ambari REST API | 前端節點 | 8080 | /api/v1 | HTTP |

範例：

* Ambari REST API：`curl -u admin "http://10.0.0.11:8080/api/v1/clusters"`

### <a name="hdfs-ports"></a>HDFS 連接埠

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| NameNode Web UI |前端節點 |30070 |HTTPS |用以檢視狀態的 Web UI |
| NameNode 中繼資料服務 |前端節點 |8020 |IPC |檔案系統中繼資料 |
| DataNode |所有背景工作節點 |30075 |HTTPS |用以檢視狀態、記錄檔等的 Web UI |
| DataNode |所有背景工作節點 |30010 |&nbsp; |資料傳輸 |
| DataNode |所有背景工作節點 |30020 |IPC |中繼資料作業 |
| 次要 NameNode |前端節點 |50090 |HTTP |NameNode 中繼資料的檢查點 |

### <a name="yarn-ports"></a>YARN 連接埠

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| Resource Manager |前端節點 |8088 |HTTP |適用於 Resource Manager 的 Web UI |
| Resource Manager |前端節點 |8090 |HTTPS |適用於 Resource Manager 的 Web UI |
| Resource Manager 系統管理介面 |前端節點 |8141 |IPC |適用於應用程式提交 (Hive、Hive 伺服器、Pig 等) |
| Resource Manager 排程器 |前端節點 |8030 |HTTP |系統管理介面 |
| Resource Manager 應用程式介面 |前端節點 |8050 |HTTP |應用程式管理員介面的位址 |
| NodeManager |所有背景工作節點 |30050 |&nbsp; |容器管理員的位址 |
| NodeManager Web UI |所有背景工作節點 |30060 |HTTP |Resource Manager 介面 |
| Timeline 位址 |前端節點 |10200 |RPC |Timeline 服務 RPC 服務。 |
| Timeline Web UI |前端節點 |8181 |HTTP |Timeline 服務 Web UI |

### <a name="hive-ports"></a>Hive 連接埠

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| HiveServer2 |前端節點 |10001 |Thrift |要連接到 Hive 的服務 (Thrift/JDBC) |
| Hive 中繼存放區 |前端節點 |9083 |Thrift |要連接到 Hive 中繼資料的服務 (Thrift/JDBC) |

### <a name="webhcat-ports"></a>WebHCat 連接埠

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| WebHCat 伺服器 |前端節點 |30111 |HTTP |以 HCatalog 和其他 Hadoop 服務為基礎的 Web API |

### <a name="mapreduce-ports"></a>MapReduce 連接埠

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| JobHistory |前端節點 |19888 |HTTP |MapReduce JobHistory Web UI |
| JobHistory |前端節點 |10020 |&nbsp; |MapReduce JobHistory 伺服器 |
| ShuffleHandler |&nbsp; |13562 |&nbsp; |將中繼對應輸出傳輸至要求的歸納器 |

### <a name="oozie"></a>Oozie

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| Oozie 伺服器 |前端節點 |11000 |HTTP |Oozie 服務的 URL |
| Oozie 伺服器 |前端節點 |11001 |HTTP |Oozie 系統管理的連接埠 |

### <a name="ambari-metrics"></a>Ambari 度量

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| TimeLine (應用程式歷程記錄) |前端節點 |6188 |HTTP |Timeline 服務 Web UI |
| TimeLine (應用程式歷程記錄) |前端節點 |30200 |RPC |Timeline 服務 Web UI |

### <a name="hbase-ports"></a>HBase 連接埠

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| HMaster |前端節點 |16000 |&nbsp; |&nbsp; |
| HMaster 資訊 Web UI |前端節點 |16010 |HTTP |HBase 主要 Web UI 的連接埠 |
| 區域伺服器 |所有背景工作節點 |16020 |&nbsp; |&nbsp; |
| &nbsp; |&nbsp; |2181 |&nbsp; |用戶端用於連接至 ZooKeeper 的連接埠 |

### <a name="kafka-ports"></a>Kafka 連接埠

| 服務 | 節點 | Port | 通訊協定 | 說明 |
| --- | --- | --- | --- | --- |
| Broker |背景工作節點 |9092 |[Kafka Wire Protocol (Kafka 有線通訊協定)](http://kafka.apache.org/protocol.html) |用於用戶端通訊 |
| &nbsp; |Zookeeper 節點 |2181 |&nbsp; |用戶端用於連接至 ZooKeeper 的連接埠 |

### <a name="spark-ports"></a>Spark 連接埠

| 服務 | 節點 | Port | 通訊協定 | URL 路徑 | 說明 |
| --- | --- | --- | --- | --- | --- |
| Spark Thrift 伺服器 |前端節點 |10002 |Thrift | &nbsp; | 要連接到 Spark SQL 的服務 (Thrift/JDBC) |
| Livy 伺服器 | 前端節點 | 8998 | HTTP | &nbsp; | 要執行陳述式、作業和應用程式的服務 |
| Jupyter 筆記本 | 前端節點 | 8001 | HTTP | &nbsp; | Jupyter Notebook 網站 |

範例：

* Livy：`curl -u admin -G "http://10.0.0.11:8998/"`。 在此範例中，`10.0.0.11` 是託管 Livy 服務之前端節點的 IP 位址。
