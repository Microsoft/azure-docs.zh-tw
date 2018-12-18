---
title: Hadoop 高可用性 - Azure HDInsight
description: 了解如何使用額外的前端節點，讓 HDInsight 叢集可以提高可靠性和可用性。 了解這會如何影響例如 Ambari 和 Hive 等 Hadoop 服務，以及如何使用 SSH 分別連線到每個前端節點。
services: hdinsight
ms.reviewer: jasonh
author: jasonwhowell
keywords: hadoop 高可用性
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 03/22/2018
ms.author: jasonh
ms.openlocfilehash: 33458794ad74b367f1278364d7b4ace30f7d13a8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953578"
---
# <a name="availability-and-reliability-of-hadoop-clusters-in-hdinsight"></a>HDInsight 上 Hadoop 叢集的可用性和可靠性

HDInsight 叢集提供兩個前端節點，以提升執行中 Hadoop 服務和作業的可用性和可靠性。

Hadoop 藉由在叢集中的多個節點間複寫服務和資料，以達到高可用性和可靠性。 不過 Hadoop 的標準散佈功能通常只能有一個前端節點。 任何單一前端節點的中斷情況都可能導致叢集停止運作。 HDInsight 提供兩個前端節點，以改善 Hadoop 的可用性和可靠性。

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="availability-and-reliability-of-nodes"></a>節點的可用性和可靠性

使用 Azure 虛擬機器在 HDInsight 叢集中的節點實作。 下列各節會討論個別的節點類型所搭配使用的 HDInsight。 

> [!NOTE]
> 並非所有的節點類型都可用於某個叢集類型。 例如，Hadoop 叢集類型就不會有任何 Nimbus 節點。 如需 HDInsight 叢集類型所使用的節點詳細資訊，請參閱[在 HDInsight 建立以 Linux 為基礎的 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md#cluster-types)文件的＜叢集類型＞一節。

### <a name="head-nodes"></a>前端節點

為了確保 Hadoop 服務的高可用性，HDInsight 提供兩個前端節點。 這兩個前端節點會同時為作用中和在 HDInsight 叢集中執行。 某些服務，例如 HDFS 或 YARN，在任何給定的時間僅能在其中一個前端節點上為「作用中」。 HiveServer2 或 Hive MetaStore 等其他服務可同時在這兩個前端節點上作用。

前端節點 (和 HDInsight 中的其他節點) 以數值作為節點主機名稱的一部分。 例如，`hn0-CLUSTERNAME` 或 `hn4-CLUSTERNAME`。

> [!IMPORTANT]
> 請勿將數值與節點 (不論是主要或次要) 相關聯。 數值只是用來為每個節點提供唯一名稱。

### <a name="nimbus-nodes"></a>Nimbus 節點

Nimbus 節點可用於 Storm 叢集。 Nimbus 節點會透過在背景工作節點之間散佈並監視處理，來提供 Hadoop jobtracker 類似的功能。 HDInsight 為 Storm 叢集提供兩個 Nimbus 節點

### <a name="zookeeper-nodes"></a>Zookeeper 節點

[ZooKeeper](http://zookeeper.apache.org/) 節點用於前端節點上主要服務的前置選擇。 也可以用來確保服務、資料 (背景工作角色) 節點和閘道知道主要服務在哪一個前端節點上為作用中。 根據預設，HDInsight 會提供三個 ZooKeeper 節點。

### <a name="worker-nodes"></a>背景工作節點

當作業提交至叢集時，背景工作節點會執行實際的資料分析。 如果背景工作節點失敗，它所執行的工作將會提交至另一個背景工作節點。 根據預設，HDInsight 會建立四個背景工作節點。 不過，您可以視需要在叢集建立期間和之後變更該數字。

### <a name="edge-node"></a>邊緣節點

邊緣節點並不積極地參與叢集內的資料分析， 而是由開發人員或資料科學家在使用 Hadoop 時搭配使用。 邊緣節點和叢集中的其他節點一樣，存在於相同的 Azure 虛擬網路中，也可直接存取其他所有節點。 使用邊緣節點時，不需從關鍵 Hadoop 服務或分析作業取走資源。

目前，HDInsight 上的 ML 服務是唯一可依預設提供邊緣節點的叢集類型。 對於 HDInsight 上的 ML 服務來說，邊緣節點是用來在提交至叢集進行分散式處理之前，在本機節點上測試 R 程式碼使用。

如需有關將邊緣節點搭配其他叢集類型使用的相關資訊，請參閱[在 HDInsight 中使用邊緣節點](hdinsight-apps-use-edge-node.md)文件。

## <a name="accessing-the-nodes"></a>存取節點

您可以透過公用閘道，經由網際網路存取叢集。 您只能連線至前端節點，以及 (如果有的話) 邊緣節點。 具有多個前端節點並不會影響在前端節點上執行之服務的存取。 公用閘道會將要求路由至託管已要求服務的前端節點。 比方說，如果 Ambari 目前託管在次要前端節點上，閘道會將 Ambari 收到的要求路由至該節點。

只有連接埠 443 (HTTPS)、22 和 23 才能透過公用閘道進行存取。

* 連接埠 __443__ 用來存取 Ambari 和裝載在前端節點上的其他 Web UI 或 REST API。

* 連接埠 __22__ 用來使用 SSH 存取主要前端節點或邊緣節點。

* 連接埠 __23__ 用來使用 SSH 存取次要前端節點。 例如，`ssh username@mycluster-ssh.azurehdinsight.net` 會連線到名為 **mycluster** 之叢集的主要前端節點。

如需使用 SSH 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

### <a name="internal-fully-qualified-domain-names-fqdn"></a>內部完整網域名稱 (FQDN)

HDInsight 叢集中的節點具有只能自叢集存取的內部 IP 位址和 FQDN。 使用內部 FQDN 或 IP 位址存取叢集上的服務時，您應該使用 Ambari 來驗證要存取服務時所使用的 IP 或 FQDN。

例如 Oozie 服務只能在一個前端節點上執行，而且從 SSH 工作階段使用 `oozie` 命令需要有該服務的 URL。 這個 URL 可以使用下列命令從 Ambari 擷取：

    curl -u admin:PASSWORD "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations?type=oozie-site&tag=TOPOLOGY_RESOLVED" | grep oozie.base.url

此命令會傳回類似以下的值，其中包含要使用 `oozie` 命令的內部 URL：

    "oozie.base.url": "http://hn0-CLUSTERNAME-randomcharacters.cx.internal.cloudapp.net:11000/oozie"

如需使用 Ambari REST API 的詳細資訊，請參閱[使用 Ambari REST API 監視和管理 HDInsight](hdinsight-hadoop-manage-ambari-rest-api.md)。

### <a name="accessing-other-node-types"></a>存取其他節點類型

您可以使用下列方法連線到並非直接透過網際網路存取的節點：

* **SSH**︰一旦使用 SSH 連接到前端節點，您便可以使用來自前端節點的 SSH 來連接至叢集中的其他節點。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

* **SSH 通道**：如果您需要存取裝載在其中一個節點上的 Web 服務，且該服務不會在網際網路上公開，您就必須使用 SSH 通道。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH 通道](hdinsight-linux-ambari-ssh-tunnel.md)文件。

* **Azure 虛擬網路**：如果您的 HDInsight 叢集是 Azure 虛擬網路的一部分，則任何在相同虛擬網路上的資源都可以直接存取叢集內所有的節點。 如需詳細資訊，請參閱[使用 Azure 虛擬網路延伸 HDInsight](hdinsight-extend-hadoop-virtual-network.md) 文件。

## <a name="how-to-check-on-a-service-status"></a>如何檢查服務狀態

若要檢查在前端節點執行的服務狀態，請使用 Ambari Web UI 或 Ambari REST API。

### <a name="ambari-web-ui"></a>Ambari Web UI

可以在 https://CLUSTERNAME.azurehdinsight.net 檢視 Ambari Web UI。 將 **CLUSTERNAME** 取代為您叢集的名稱。 如果出現提示，請輸入叢集的 HTTP 使用者認證。 預設 HTTP 使用者名稱為 **admin** ，密碼是您在建立叢集時輸入的密碼。

當您來到 Ambari 頁面上時，會在該頁面的左邊列出已安裝的服務。

![已安裝的服務](./media/hdinsight-high-availability-linux/services.png)

服務旁可能會出現一系列圖示以表示狀態。 可以使用在頁面頂端的 [警示]  連結，檢視與服務相關的任何警示。 您可以選取每個服務來檢視其詳細資訊。

雖然 [服務] 頁面會提供資訊之狀態和每個服務的組態，但並不提供該服務正在哪些前端節點上執行的資訊。 若要檢視這項資訊，請使用在頁面頂端的 [主機]  連結。 此頁面會顯示叢集內的主機，包括前端節點在內。

![主機清單](./media/hdinsight-high-availability-linux/hosts.png)

選取其中一個前端節點的連結會顯示該節點上執行的服務與元件。

![元件狀態](./media/hdinsight-high-availability-linux/nodeservices.png)

如需使用 Ambari 的詳細資訊，請參閱[使用 Ambari Web UI 監視和管理 HDInsight](hdinsight-hadoop-manage-ambari.md)。

### <a name="ambari-rest-api"></a>Ambari REST API

Ambari REST API 可透過網際網路提供。 HDInsight 公用閘道器會處理路由要求，將其傳送到目前裝載 REST API 的前端節點。

您可以使用下列命令透過 Ambari REST API 來檢查服務的狀態：

    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICENAME?fields=ServiceInfo/state

* 將 **PASSWORD** 取代為 HTTP 使用者 (系統管理員) 帳戶密碼。
* 將 **CLUSTERNAME** 取代為叢集的名稱。
* 將 **SERVICENAME** 取代為您要檢查其狀態的服務名稱。

例如，若要檢查在名為 **mycluster** 的叢集上的 **HDFS** 服務狀態，其中密碼是 **password**，您可使用下列命令：

    curl -u admin:password https://mycluster.azurehdinsight.net/api/v1/clusters/mycluster/services/HDFS?fields=ServiceInfo/state

回應如下列 JSON 所示：

    {
      "href" : "http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:8080/api/v1/clusters/mycluster/services/HDFS?fields=ServiceInfo/state",
      "ServiceInfo" : {
        "cluster_name" : "mycluster",
        "service_name" : "HDFS",
        "state" : "STARTED"
      }
    }

該 URL 會告訴我們服務目前正在名為 **hn0-CLUSTERNAME** 的前端節點上執行。

該狀態會告訴我們此服務目前正在執行，或 **已啟動**。

如果您不知道有哪些服務安裝在該叢集上，您可以使用下列命令擷取清單：

    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services

如需使用 Ambari REST API 的詳細資訊，請參閱[使用 Ambari REST API 監視和管理 HDInsight](hdinsight-hadoop-manage-ambari-rest-api.md)。

#### <a name="service-components"></a>服務元件

服務可能包含您想要個別檢查狀態的元件。 例如，HDFS 包含 NameNode 元件。 若要檢視在元件上的資訊，該命令為：

    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICE/components/component

如果您不知道服務提供了哪些元件，您可以使用下列命令擷取清單：

    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICE/components/component

## <a name="how-to-access-log-files-on-the-head-nodes"></a>如何存取前端節點上的記錄檔

### <a name="ssh"></a>SSH

透過 SSH 連接至前端節點時，可以在 **/var/log**找到記錄檔。 例如， **/var/log/hadoop-yarn/yarn** 包含 YARN 的記錄檔。

每個前端節點可以有唯一的記錄項目，所以您應該檢查兩者的記錄檔。

### <a name="sftp"></a>SFTP

您也可以使用 SSH 檔案傳輸通訊協定或安全檔案傳輸通訊協定 (SFTP)，來連線至前端節點，並直接下載記錄檔。

類似於使用 SSH 用戶端，當連接到叢集時，您必須提供 SSH 的使用者帳戶名稱和叢集的 SSH 位址。 例如： `sftp username@mycluster-ssh.azurehdinsight.net`。 出現提示時，請提供帳戶密碼或使用 `-i` 參數提供公開金鑰。

連線之後，您會看到 `sftp>` 提示。 您可以從該提示變更目錄、上傳和下載檔案。 例如：下列命令會將目錄變更至 **/var/log/hadoop/hdfs** 目錄，然後在目錄中下載所有檔案。

    cd /var/log/hadoop/hdfs
    get *

如需可用命令清單，請在 `sftp>` 提示中輸入 `help`。

> [!NOTE]
> 使用 SFTP 連線時，也提供圖形化介面可讓您以視覺化方式檢視檔案系統。 例如： [MobaXTerm](http://mobaxterm.mobatek.net/) 可讓您使用類似於「Windows 檔案總管」的介面瀏覽檔案系統。

### <a name="ambari"></a>Ambari

> [!NOTE]
> 若要使用 Ambari 存取記錄檔，您必須使用 SSH 通道。 個別服務的 Web 介面不會在網際網路上公開。 如需使用 SSH 通道的詳細資訊，請參閱[使用 SSH 通道](hdinsight-linux-ambari-ssh-tunnel.md)文件。

從 Ambari Web UI 中，選取您想要檢視記錄的服務 (例如，YARN)。 然後使用 [快速連結] 來選取要檢視記錄的前端節點。

![使用快速連結檢視記錄檔](./media/hdinsight-high-availability-linux/viewlogs.png)

## <a name="how-to-configure-the-node-size"></a>如何設定節點大小

只能在叢集建立期間選取節點的大小。 您可以在 [HDInsight 價格頁面](https://azure.microsoft.com/pricing/details/hdinsight/)找到 HDInsight 可用之不同 VM 大小的清單。

建立叢集時，您可以指定節點的大小。 以下資訊提供如何使用 [Azure 入口網站][preview-portal]、[Azure PowerShell][azure-powershell] 和 [Azure Classic CLI][azure-cli] 指定大小的指引：

* **Azure 入口網站**：建立叢集時，您可以設定叢集所使用的節點大小：

    ![可選取節點大小的 [叢集映像建立精靈]](./media/hdinsight-high-availability-linux/headnodesize.png)

* **Azure Classic CLI**︰當使用 `azure hdinsight cluster create` 命令時，您可以使用 `--headNodeSize`、`--workerNodeSize` 和 `--zookeeperNodeSize` 參數設定前端、 背景工作與 ZooKeeper 節點的大小。

* **Azure PowerShell**︰當使用 `New-AzureRmHDInsightCluster` Cmdlet 時，您可以使用 `-HeadNodeVMSize`、`-WorkerNodeSize` 和 `-ZookeeperNodeSize` 參數設定前端、背景工作與 ZooKeeper 節點的大小。

## <a name="next-steps"></a>後續步驟

若要深入了解這份文件中所敘述的項目，請使用下列連結。

* [Ambari REST 參考](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)
* [安裝和設定 Azure CLI](../cli-install-nodejs.md)
* [安裝並設定 Azure PowerShell](/powershell/azure/overview)
* [使用 Ambari 管理 HDInsight](hdinsight-hadoop-manage-ambari.md)
* [佈建以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)

[preview-portal]: https://portal.azure.com/
[azure-powershell]: /powershell/azureps-cmdlets-docs
[azure-cli]: ../cli-install-nodejs.md
