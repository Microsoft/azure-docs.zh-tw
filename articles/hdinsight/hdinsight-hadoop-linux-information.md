---
title: 在 Linux 架構的 HDInsight 上使用 Hadoop 的秘訣 - Azure
description: 取得在 Azure 雲端中執行的熟悉 Linux 環境上使用 Linux 架構的 HDInsight (Hadoop) 叢集的實作秘訣。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 08/09/2018
ms.openlocfilehash: d725bbe31de0f93d1b741ffd008bf39086904b61
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997500"
---
# <a name="information-about-using-hdinsight-on-linux"></a>在 Linux 上使用 HDInsight 的相關資訊

Azure HDInsight 叢集可在您熟悉的 Linux 環境中提供於 Azure 雲端中執行的 Hadoop。 其操作大多與 Linux 安裝上的任何其他 Hadoop 相同。 本文件會指出其中應注意的特殊不同之處。

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="prerequisites"></a>必要條件

本文件中的許多步驟都使用下列公用程式，可能需要安裝在您的系統上。

* [cURL](https://curl.haxx.se/) - 用來與 Web 型服務通訊
* [jq](https://stedolan.github.io/jq/) - 用來剖析 JSON 文件
* [Azure CLI](https://docs.microsoft.com/cli/azure/install-az-cli2) - 用來從遠端管理 Azure 服務

## <a name="users"></a>使用者

除非[已加入網域](./domain-joined/apache-domain-joined-introduction.md)，否則應將 HDInsight 視為**單一使用者**系統。 叢集中會建立一個具有系統管理員層級權限的 SSH 使用者帳戶。 您可以建立其他 SSH 帳戶，但這些帳戶也會擁有叢集的系統管理員權限。

已加入網域的 HDInsight 可支援多個使用者和更細微的權限和角色設定。 如需詳細資訊，請參閱[管理已加入網域的 HDInsight 叢集](./domain-joined/apache-domain-joined-manage.md)。

## <a name="domain-names"></a>網域名稱

從網際網路連接到叢集時所要使用的完整網域名稱 (FQDN) 是 **&lt;clustername>.azurehdinsight.net** 或 (僅適用於 SSH) **&lt;clustername-ssh>.azurehdinsight.net**。

就內部而言，叢集中的每個節點都具有在叢集組態期間指派的名稱。 若要尋找叢集名稱，請參閱 Ambari Web UI 上的 [主機] 頁面。 您也可以使用下列命令從 Ambari REST API 傳回主機清單︰

    curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts" | jq '.items[].Hosts.host_name'

將 **CLUSTERNAME** 取代為您叢集的名稱。 出現提示時，請輸入系統管理員帳戶的密碼。 此命令會傳回包含叢集中主機清單的 JSON 文件。 Jq 用來擷取每部主機的 `host_name` 元素值。

如果您需要針對特定服務尋找節點的名稱，您可以查詢 Ambari 有無該元件。 例如，若要尋找主機有無 HDFS 名稱節點，請使用下列命令：

    curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/HDFS/components/NAMENODE" | jq '.host_components[].HostRoles.host_name'

此命令會傳回描述服務的 JSON 文件，然後 jq 只會針對主機提取 `host_name` 值。

## <a name="remote-access-to-services"></a>遠端存取服務

* **Ambari (web)** - https://&lt;clustername>.azurehdinsight.net

    使用叢集系統管理員使用者和密碼進行驗證，然後登入 Ambari 。

    驗證是純文字的 - 請一律使用 HTTPS 來協助確保連線的安全性。

    > [!IMPORTANT]
    > 透過 Ambari 使用的一些 Web UI 會使用內部網域名稱存取節點。 內部網域名稱無法透過網際網路公開存取。 在嘗試透過網際網路存取某些功能時可能會收到「找不到伺服器」的錯誤。
    >
    > 若要使用 Ambari Web UI 的完整功能，請使用 SSH 通道將 Web 流量以 Proxy 處理傳輸到叢集前端節點。 請參閱[使用 SSH 通道來存取 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 及其他 Web UI](hdinsight-linux-ambari-ssh-tunnel.md)

* **Ambari (REST)** - https://&lt;clustername>.azurehdinsight.net/ambari

    > [!NOTE]
    > 使用叢集系統管理員使用者和密碼進行驗證。
    >
    > 驗證是純文字的 - 請一律使用 HTTPS 來協助確保連線的安全性。

* **WebHCat (Templeton)** - https://&lt;clustername>.azurehdinsight.net/templeton

    > [!NOTE]
    > 使用叢集系統管理員使用者和密碼進行驗證。
    >
    > 驗證是純文字的 - 請一律使用 HTTPS 來協助確保連線的安全性。

* **SSH** - 連接埠 22 或 23 上的 &lt;clustername>-ssh.azurehdinsight.net。 連接埠 22 用來連接至主要前端節點，而 23 用來連接至次要前端節點。 如需前端節點的詳細資訊，請參閱 [HDInsight 上 Hadoop 叢集的可用性和可靠性](hdinsight-high-availability-linux.md)。

    > [!NOTE]
    > 您只能從用戶端電腦透過 SSH 存取叢集前端節點。 然後在連線後，再從前端節點使用 SSH 存取背景工作角色節點。

如需詳細資訊，請參閱 [HDInsight 上 Hadoop 服務所使用的連接埠](hdinsight-hadoop-port-settings-for-services.md)文件。

## <a name="file-locations"></a>檔案位置

Hadoop 相關檔案可以在叢集節點的 `/usr/hdp`上找到。 此目錄包含下列子目錄：

* **2.2.4.9-1**︰目錄名稱是 HDInsight 所使用的 Hortonworks Data Platform 版本。 叢集上的數字可能不同於此處所列的數字。
* **current**︰此目錄包含 **2.2.4.9-1** 目錄下的子目錄連結。 因為有此目錄，您就不必記住版本號碼。

在 Hadoop 分散式檔案系統的 `/example` 和 `/HdiSamples` 可取得範例資料和 JAR 檔案。

## <a name="hdfs-azure-storage-and-data-lake-store"></a>HDFS、Azure 儲存體和 Data Lake Store

在大部分的 Hadoop 散發套件中，資料會儲存在叢集機器上的本機儲存體所支援的 HDFS 中。 針對雲端解決方案使用本機儲存體的成本可能相當高，因為計算資源是以每小時或每分鐘為單位來計費。

使用 HDInsight 時，會使用 Azure Blob 儲存體 (並選擇性地使用 Azure Data Lake Store) 以可調整的彈性方式將資料檔案儲存在雲端中。 這些服務提供下列優點：

* 長期儲存成本低廉
* 可從各種外部服務進行存取，例如網站、檔案上傳/下載公用程式、各種語言的 SDK 和網頁瀏覽器
* 大型檔案容量和大型可調整儲存體

如需詳細資訊，請參閱[了解 Blob](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) 和[Data Lake Store](https://azure.microsoft.com/services/data-lake-store/)。

使用 Azure 儲存體或 Data Lake Store 時，您不需要從 HDInsight 進行任何特殊動作就能存取資料。 例如，下列命令列出 `/example/data` 資料夾中的檔案，不論其儲存在 Azure 儲存體或 Data Lake Store 中：

    hdfs dfs -ls /example/data

在 HDInsight 中，資料儲存體資源 (Azure Blob 儲存體和 Azure Data Lake Store) 會與計算資源分離。 因此，您可以在必要時建立 HDInsight 叢集以執行計算，並在後續完成工作後刪除該叢集，同時將您的資料檔案安全地保存在雲端儲存體中，沒有時間限制。

### <a name="uri-and-scheme"></a>URI 和配置

有些命令可能需要您在存取檔案時於 URI 中指定配置。 例如，Storm-HDFS 元件就需要您指定配置。 在使用非預設儲存體 (新增為叢集「其他」儲存體的儲存體) 時，您一律必須在 URI 中使用配置。

使用 __Azure 儲存體__時，可使用下列其中一種 UI 配置︰

* `wasb:///`︰使用未加密通訊存取預設儲存體。

* `wasbs:///`︰使用加密通訊存取預設儲存體。  HDInsight 3.6 版和更新版本才會支援 wasbs 配置。

* `wasb://<container-name>@<account-name>.blob.core.windows.net/`︰以非預設儲存體帳戶進行通訊時使用。 例如，當您有其他儲存體帳戶，或在可公開存取的儲存體帳戶中存取儲存的資料時。

使用 __Data Lake Store__ 時，可使用下列其中一種 UI 配置︰

* `adl:///`︰存取叢集的預設 Data Lake Store。

* `adl://<storage-name>.azuredatalakestore.net/`：用來與非預設 Data Lake Store 進行通訊。 也用來存取 HDInsight 叢集根目錄之外的資料。

> [!IMPORTANT]
> 使用 Data Lake Store 做為 HDInsight 的預設存放區時，您必須在存放區內指定要做為 HDInsight 儲存體根目錄的路徑。 預設路徑為 `/clusters/<cluster-name>/`。
>
> 使用 `/` 或 `adl:///` 存取資料時，您只可以存取儲存在叢集根目錄 (例如，`/clusters/<cluster-name>/`) 中的資料。 若要存取存放區中任何位置的資料，請使用 `adl://<storage-name>.azuredatalakestore.net/` 格式。

### <a name="what-storage-is-the-cluster-using"></a>叢集使用什麼儲存體

您可以使用 Ambari 來擷取叢集的預設儲存體組態。 請使用下列命令和 CURL 來擷取 HDFS 組態資訊，並使用 [jq](https://stedolan.github.io/jq/)加以篩選：

```curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'```

> [!NOTE]
> 這個命令會傳回套用至伺服器的第一個組態 (`service_config_version=1`)，其中包含這項資訊。 您可能需要列出所有組態版本來找出最新的版本。

此命令會傳回類似下列 URI 的值：

* `wasb://<container-name>@<account-name>.blob.core.windows.net`，若使用 Azure 儲存體帳戶。

    帳戶名稱是「Azure 儲存體」帳戶的名稱。 容器名稱是作為叢集儲存體根目錄的 Blob 容器。

* `adl://home`，若使用 Azure Data Lake Store。 若要取得 Data Lake Store 名稱，請使用下列 REST 呼叫︰

    ```curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["dfs.adls.home.hostname"] | select(. != null)'```

    此命令會傳回下列主機名稱：`<data-lake-store-account-name>.azuredatalakestore.net`。

    若要取得 HDInsight 根目錄存放區內的目錄，請使用下列 REST 呼叫︰

    ```curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["dfs.adls.home.mountpoint"] | select(. != null)'```

    此命令會傳回類似下列路徑的路徑：`/clusters/<hdinsight-cluster-name>/`。

您也可以使用 Azure 入口網站，以下列步驟尋找儲存體資訊︰

1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 HDInsight 叢集。

2. 從 [屬性] 區段中，選取 [儲存體帳戶]。 隨即會顯示叢集的儲存體資訊。

### <a name="how-do-i-access-files-from-outside-hdinsight"></a>如何從 HDInsight 外部存取檔案

有各種不同的方式可從 HDInsight 叢集之外存取資料。 以下是幾個可用來處理資料之公用程式和 SDK 的連結︰

如果使用 __Azure 儲存體__，請參閱下列連結，以取得可供存取資料的方式︰

* [Azure CLI](https://docs.microsoft.com/cli/azure/install-az-cli2)：適用於 Azure 的命令列介面命令。 安裝好後，請使用 `az storage` 命令以協助使用儲存體，或是針對 Blob 特有命令使用 `az storage blob`。
* [blobxfer.py](https://github.com/Azure/azure-batch-samples/tree/master/Python/Storage)：python 指令碼，用於 Azure 儲存體中的 Blob。
* 各種 SDK：

    * [Java](https://github.com/Azure/azure-sdk-for-java)
    * [Node.js](https://github.com/Azure/azure-sdk-for-node)
    * [PHP](https://github.com/Azure/azure-sdk-for-php)
    * [Python](https://github.com/Azure/azure-sdk-for-python)
    * [Ruby](https://github.com/Azure/azure-sdk-for-ruby)
    * [.NET](https://github.com/Azure/azure-sdk-for-net)
    * [儲存體 REST API](https://msdn.microsoft.com/library/azure/dd135733.aspx)

如果使用 __Azure Data Lake Store__，請參閱下列連結，以取得可供存取資料的方式︰

* [Web 瀏覽器](../data-lake-store/data-lake-store-get-started-portal.md)
* [PowerShell](../data-lake-store/data-lake-store-get-started-powershell.md)
* [Azure CLI](../data-lake-store/data-lake-store-get-started-cli-2.0.md)
* [WebHDFS REST API](../data-lake-store/data-lake-store-get-started-rest-api.md)
* [Visual Studio 適用的 Data Lake 工具](https://www.microsoft.com/download/details.aspx?id=49504)
* [.NET](../data-lake-store/data-lake-store-get-started-net-sdk.md)
* [Java](../data-lake-store/data-lake-store-get-started-java-sdk.md)
* [Python](../data-lake-store/data-lake-store-get-started-python.md)

## <a name="scaling"></a>調整您的叢集規模

叢集調整功能可讓您動態變更叢集所用的資料節點數目。 正在叢集上執行其他工作或處理序時，您可以執行調整作業。

不同的叢集類型會受調整影響，如下所示：

* **Hadoop**︰相應減少叢集中的節點數目時，會重新啟動叢集中的部分服務。 調整作業可能會導致執行中或擱置的工作在調整作業完成時失敗。 您可以在作業完成後重新提交這些工作。
* **HBase**︰區域伺服器會在調整作業完成後的幾分鐘內自動取得平衡。 若要手動平衡區域伺服器，請使用下列步驟：

    1. 使用 SSH 連線到 HDInsight 叢集。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

    2. 使用下列命令來啟動 HBase Shell：

            hbase shell

    3. 載入 HBase Shell 後，使用下列命令來手動平衡區域伺服器︰

            balancer

* **Storm**︰執行調整作業之後，您應該重新平衡任何執行中的 Storm 拓撲。 重新平衡可讓拓撲根據叢集中的新節點數目，重新調整平行處理原則設定。 若要重新平衡執行中的拓撲，請使用下列其中一個選項：

    * **SSH**︰連接到伺服器並使用下列命令來重新平衡拓撲：

            storm rebalance TOPOLOGYNAME

        您也可以指定參數來覆寫拓撲原先提供的平行處理原則提示。 例如，`storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10` 會將拓撲重新設定為 5 個背景工作角色處理序、適用於 blue-spout 元件的 3 個執行程式，以及適用於 yellow-bolt 元件的 10 個執行程式。

    * **Storm UI**︰使用下列步驟來重新平衡使用 Storm UI 的拓撲。

        1. 在您的網頁瀏覽器中開啟 **https://CLUSTERNAME.azurehdinsight.net/stormui**，其中 CLUSTERNAME 是 Storm 叢集的名稱。 出現提示時，輸入建立叢集時所指定的 HDInsight 叢集系統管理員 (管理員) 名稱和密碼。
        2. 選取您要重新平衡的拓撲，然後選取 [重新平衡] 按鈕。 在執行重新平衡作業之前輸入延遲。

* **Kafka**：您應該在調整作業完成後重新平衡磁碟分割複本。 如需詳細資訊，請參閱[使用 HDInsight 上的 Kafka 確保資料的高可用性](./kafka/apache-kafka-high-availability.md)文件。

如需有關調整 HDInsight 叢集的特定資訊，請參閱：

* [使用 Azure 入口網站管理 HDInsight 中的 Hadoop 叢集](hdinsight-administer-use-portal-linux.md#scale-clusters)
* [使用 Azure PowerShell 管理 HDInsight 上的 Hadoop 叢集](hdinsight-administer-use-command-line.md#scale-clusters)

## <a name="how-do-i-install-hue-or-other-hadoop-component"></a>如何安裝 Hue (或其他 Hadoop 元件)？

HDInsight 是受控服務。 如果 Azure 偵測到叢集問題，它可能會刪除失敗節點並建立要取代它的節點。 如果您以手動方式在叢集上安裝項目，此作業發生時，不會保存這些項目。 請改用 [HDInsight 指令碼動作](hdinsight-hadoop-customize-cluster.md)。 指令碼動作可用來進行下列變更︰

* 安裝及設定服務或網站。
* 安裝及設定需要在叢集中的多個節點上進行組態變更的元件。

指令碼動作是 Bash 指令碼。 指令碼會在叢集建立期間執行，而且可用來安裝及設定其他元件。 範例指令碼可供安裝下列元件：

* [Giraph](hdinsight-hadoop-giraph-install-linux.md)
* [Solr](hdinsight-hadoop-solr-install-linux.md)

如需開發您自己的指令碼動作相關資訊，請參閱 [使用 HDInsight 開發指令碼動作](hdinsight-hadoop-script-actions-linux.md)。

### <a name="jar-files"></a>JAR 檔案

獨立的 jar 檔案中提供了一些 Hadoop 技術，其中包含可用來做為 MapReduce 工作一部分的函式，或者來自 Pig 或 Hive 內部的函式。 它們通常不需要任何設定，而且可在佈建後上傳到叢集並直接使用。 如果您想要確定元件會在重新安裝叢集的映像後保存下來，則可將 jar 檔案儲存於叢集的預設儲存體 (WASB 或 ADL) 中。

例如，如果您想要使用最新版本的 [DataFu](http://datafu.incubator.apache.org/)，則可下載包含專案的 jar，並將它上傳至 HDInsight 叢集。 接著遵循 DataFu 文件中，如何從 Pig 或 Hive 中使用它的指示進行。

> [!IMPORTANT]
> 有一些屬於獨立 jar 檔案的元件是透過 HDInsight 來提供，但它們不在路徑中。 如果您正在尋找特定元件，可在叢集上使用下列內容來搜尋：
>
> ```find / -name *componentname*.jar 2>/dev/null```
>
> 此命令會傳回任何相符 jar 檔案的路徑。

若要使用不同版本的元件，請上傳您需要的版本並在工作中使用。

> [!WARNING]
> 透過 HDInsight 叢集提供的元件會受到完整支援，且 Microsoft 支援服務會協助釐清與解決這些元件的相關問題。
>
> 自訂元件則獲得商務上合理的支援，協助您進一步疑難排解問題。 如此可能會進而解決問題，或要求您利用可用管道，以找出開放原始碼技術，從中了解該技術的深度專業知識。 例如，有許多社群網站可供使用，例如：[MSDN 的 HDInsight 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=hdinsight) \(英文\)、[http://stackoverflow.com](http://stackoverflow.com) \(英文\)。 此外，Apache 專案在 [http://apache.org](http://apache.org) 上也有專案網站，例如 [Hadoop](http://hadoop.apache.org/)、[Spark](http://spark.apache.org/)。

## <a name="next-steps"></a>後續步驟

* [從以 Windows 為基礎的 HDInsight 移轉至以 Linux 為基礎的 HDInsight](hdinsight-migrate-from-windows-to-linux.md)
* [〈搭配 HDInsight 使用 Hivet〉](hadoop/hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hadoop/hdinsight-use-pig.md)
* [搭配 HDInsight 使用 MapReduce 工作](hadoop/hdinsight-use-mapreduce.md)
