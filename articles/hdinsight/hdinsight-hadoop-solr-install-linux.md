---
title: 使用指令碼動作在以 Linux 為基礎的 HDInsight 上安裝 Solr - Azure
description: 在本主題中，您將學習如何使用指令碼動作在以 Linux 為基礎的 HDInsight Hadoop 叢集上安裝 Solr。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: 205983344be8ae5bbe566a208ceb862b2e93cb8d
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43093092"
---
# <a name="install-and-use-solr-on-hdinsight-hadoop-clusters"></a>在 HDInsight Hadoop 叢集上安裝和使用 Solr

了解如何使用指令碼動作，在 Azure HDInsight 上安裝 Solr。 Solr 是強大的搜尋平台，可對 Hadoop 管理的資料執行企業級搜尋功能。

> [!IMPORTANT]
    > 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

> [!IMPORTANT]
> 本文件中使用的範例指令碼會以特定組態安裝 Solr 4.9。 如果您想要以不同的集合、分區、結構描述和複本等項目設定 Solr 叢集，則必須修改指令碼和 Solr 二進位檔。

## <a name="whatis"></a>什麼是 Solr

[Apache Solr](http://lucene.apache.org/solr/features.html) 是可對資料執行強大全文搜尋作業的企業搜尋平台。 Hadoop 可儲存和管理大量資料，而 Apache Solr 則是提供搜尋功能以便快速擷取資料。

> [!WARNING]
> Microsoft 對隨 HDInsight 叢集提供的元件提供完整支援。
>
> 自訂元件 (例如 Solr) 則獲得商務上合理的支援，協助您進一步對問題進行疑難排解。 Microsoft 支援服務可能無法解決自訂元件的問題。 您可能需要加入開放原始碼社群以取得協助。 例如，有許多社群網站可供使用，例如：[MSDN 的 HDInsight 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=hdinsight) \(英文\)、[http://stackoverflow.com](http://stackoverflow.com) \(英文\)。 此外，Apache 專案在 [http://apache.org](http://apache.org) 上也有專案網站，例如 [Hadoop](http://hadoop.apache.org/)。

## <a name="what-the-script-does"></a>指令碼會執行哪些作業

此指令碼可以對 HDInsight 叢集進行下列變更：

* 將 Solr 4.9 安裝至 `/usr/hdp/current/solr`
* 建立用來執行 Solr 服務的使用者 **solrusr**
* 將 **solruser** 設為 `/usr/hdp/current/solr` 的擁有者
* 加入會自動啟動 Solr 的 [Upstart](http://upstart.ubuntu.com/) 組態。

## <a name="install"></a>使用指令碼動作安裝 Solr

在 HDInsight 叢集上安裝 Solr 的範例指令碼位於下列位置：

    https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh

若要建立已安裝 Solr 的叢集，請使用[建立 HDInsight 叢集](hdinsight-hadoop-create-linux-clusters-portal.md)文件中的步驟。 在建立過程中，使用下列步驟來安裝 Solr：

1. 從 [叢集摘要] 區段，依序選取 [進階設定] 和 [指令碼動作]。 使用下列資訊來填入表單：

   * **名稱**：輸入指令碼動作的易記名稱。
   * **指令碼 URI**： https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh
   * **HEAD**：勾選此選項
   * **WORKER**：勾選此選項
   * **ZOOKEEPER**：勾選此選項以在 Zookeeper 節點上安裝
   * **參數**：將此欄位保留空白

2. 在 [指令碼動作] 區段底部，使用 [選取] 按鈕以儲存組態。 最後，使用 [下一步] 按鈕回到 [叢集摘要]。

3. 從 [叢集摘要] 頁面，選取 [建立] 以建立叢集。

## <a name="usesolr"></a>如何在 HDInsight 中使用 Solr

> [!IMPORTANT]
> 本節中的步驟示範基本的 Solr 功能。 如需有關使用 Solr 的詳細資訊，請參閱 [Apache Solr 網站](http://lucene.apache.org/solr/)。

### <a name="index-data"></a>索引資料

使用下列步驟將範例資料新增至 Solr，然後查詢它：

1. 使用 SSH 連線到 HDInsight 叢集

    > [!NOTE]
    > 將 `sshuser` 取代為叢集的 SSH 使用者。 將 `clustername` 取代為叢集的名稱。

    ```bash
    ssh sshuser@clustername-ssh.azurehdinsight.net
    ```

    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

     > [!IMPORTANT]
     > 本文件中稍後的步驟會使用 SSH 通道以連線至 Solr Web UI。 您必須先建立 SSH 通道，將您的瀏覽器設定為可以使用此通道，然後才能使用這些步驟。
     >
     > 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH 通道](hdinsight-linux-ambari-ssh-tunnel.md)文件。

2. 使用下列命令以具備 Solr 索引範例資料：

    ```bash
    cd /usr/hdp/current/solr/example/exampledocs
    java -jar post.jar solr.xml monitor.xml
    ```

    下列輸出會傳回到主控台：

        POSTing file solr.xml
        POSTing file monitor.xml
        2 files indexed.
        COMMITting Solr index changes to http://localhost:8983/solr/update..
        Time spent: 0:00:01.624

    `post.jar` 公用程式會將 **solr.xml** 和 **monitor.xml** 文件加入至索引。
  
3. 使用下列命令來查詢 Solr REST API：

    ```bash
    curl "http://localhost:8983/solr/collection1/select?q=*%3A*&wt=json&indent=true"
    ```

    此命令會搜尋 **collection1** 中是否有文件符合 **\*:\*** (在查詢字串中編碼為 \*%3A\*)。 下列 JSON 文件是回應的範例：

            "response": {
                "numFound": 2,
                "start": 0,
                "maxScore": 1,
                "docs": [
                  {
                    "id": "SOLR1000",
                    "name": "Solr, the Enterprise Search Server",
                    "manu": "Apache Software Foundation",
                    "cat": [
                      "software",
                      "search"
                    ],
                    "features": [
                      "Advanced Full-Text Search Capabilities using Lucene",
                      "Optimized for High Volume Web Traffic",
                      "Standards Based Open Interfaces - XML and HTTP",
                      "Comprehensive HTML Administration Interfaces",
                      "Scalability - Efficient Replication to other Solr Search Servers",
                      "Flexible and Adaptable with XML configuration and Schema",
                      "Good unicode support: héllo (hello with an accent over the e)"
                    ],
                    "price": 0,
                    "price_c": "0,USD",
                    "popularity": 10,
                    "inStock": true,
                    "incubationdate_dt": "2006-01-17T00:00:00Z",
                    "_version_": 1486960636996878300
                  },
                  {
                    "id": "3007WFP",
                    "name": "Dell Widescreen UltraSharp 3007WFP",
                    "manu": "Dell, Inc.",
                    "manu_id_s": "dell",
                    "cat": [
                      "electronics and computer1"
                    ],
                    "features": [
                      "30\" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
                    ],
                    "includes": "USB cable",
                    "weight": 401.6,
                    "price": 2199,
                    "price_c": "2199,USD",
                    "popularity": 6,
                    "inStock": true,
                    "store": "43.17614,-90.57341",
                    "_version_": 1486960637584081000
                  }
                ]
              }

### <a name="using-the-solr-dashboard"></a>使用 Solr 儀表板

Solr 儀表板是 Web UI，可讓您透過網頁瀏覽器使用 Solr。 Solr 儀表板不會直接從您的 HDInsight 叢集公開至網際網路上。 您可以使用 SSH 通道來存取。 如需使用 SSH 通道的詳細資訊，請參閱[搭配 HDInsight 使用 SSH 通道](hdinsight-linux-ambari-ssh-tunnel.md)文件。

一旦您建立 SSH 通道，請使用下列步驟以使用 Solr 儀表板：

1. 決定主要前端節點的主機名稱：

   1. 使用 SSH 連線到叢集前端節點。 例如： `ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net`。

       如需使用 SSH 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

   2. 使用下列命令以取得完整主機名稱︰

        ```bash
        hostname -f
        ```

        此命令會傳回類似下列主機名稱的值：

            hn0-myhdi-nfebtpfdv1nubcidphpap2eq2b.ex.internal.cloudapp.net

        儲存傳回的值，因為稍後會使用它。

2. 在瀏覽器中，連線到 **http://HOSTNAME:8983/solr/#/**，其中的 HOSTNAME 是您在先前步驟中決定的名稱。

    此要求是透過 SSH 通道路由到您叢集上的 Solr Web UI。 此頁面看起來類似下圖：

    ![Solr 儀表板的映像](./media/hdinsight-hadoop-solr-install-linux/solrdashboard.png)

3. 從左窗格使用 [核心選取器] 下拉式清單，以選取 [collection1]。 數個項目應該會出現在 **collection1** 底下。

4. 從 **collection1** 底下的項目中，選取 [查詢]。 使用下列值來填入搜尋頁面：

   * 在 [q] 文字方塊中輸入 **:**\*\*。 此查詢會傳回已在 Solr 中編製索引的所有文件。 如果您想要搜尋文件內的特定字串，您可以在此輸入該字串。
   * 在 [ **wt** ] 文字方塊中，選取輸出格式。 預設值是 [ **json**]。

     最後，選取搜尋頁面底部的 [執行查詢]  按鈕。

     ![使用指令碼動作以自訂叢集](./media/hdinsight-hadoop-solr-install-linux/hdi-solr-dashboard-query.png)

     輸出會傳回您先前加入至索引的兩個文件。 輸出與下列 JSON 文件類似：

           "response": {
               "numFound": 2,
               "start": 0,
               "maxScore": 1,
               "docs": [
                 {
                   "id": "SOLR1000",
                   "name": "Solr, the Enterprise Search Server",
                   "manu": "Apache Software Foundation",
                   "cat": [
                     "software",
                     "search"
                   ],
                   "features": [
                     "Advanced Full-Text Search Capabilities using Lucene",
                     "Optimized for High Volume Web Traffic",
                     "Standards Based Open Interfaces - XML and HTTP",
                     "Comprehensive HTML Administration Interfaces",
                     "Scalability - Efficient Replication to other Solr Search Servers",
                     "Flexible and Adaptable with XML configuration and Schema",
                     "Good unicode support: héllo (hello with an accent over the e)"
                   ],
                   "price": 0,
                   "price_c": "0,USD",
                   "popularity": 10,
                   "inStock": true,
                   "incubationdate_dt": "2006-01-17T00:00:00Z",
                   "_version_": 1486960636996878300
                 },
                 {
                   "id": "3007WFP",
                   "name": "Dell Widescreen UltraSharp 3007WFP",
                   "manu": "Dell, Inc.",
                   "manu_id_s": "dell",
                   "cat": [
                     "electronics and computer1"
                   ],
                   "features": [
                     "30\" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
                   ],
                   "includes": "USB cable",
                   "weight": 401.6,
                   "price": 2199,
                   "price_c": "2199,USD",
                   "popularity": 6,
                   "inStock": true,
                   "store": "43.17614,-90.57341",
                   "_version_": 1486960637584081000
                 }
               ]
             }

### <a name="starting-and-stopping-solr"></a>啟動和停止 Solr

使用下列命令手動停止和啟動 Solar：

```bash
sudo stop solr
sudo start solr
```

## <a name="backup-indexed-data"></a>備份已編製索引的資料

使用下列步驟將 Solr 資料備份到叢集的預設儲存體︰

1. 使用 SSH 連線到叢集，然後使用下列命令來取得前端節點的主機名稱：

    ```bash
    hostname -f
    ```

2. 使用下列命令來建立已編製索引資料的快照。 以上一個命令傳回的名稱取代 **HOSTNAME**：

    ```bash
    curl http://HOSTNAME:8983/solr/replication?command=backup
    ```

    回應如下列 XML 所示：

        <?xml version="1.0" encoding="UTF-8"?>
        <response>
          <lst name="responseHeader">
            <int name="status">0</int>
            <int name="QTime">9</int>
          </lst>
          <str name="status">OK</str>
        </response>

3. 將目錄變更為 `/usr/hdp/current/solr/example/solr`。 在這裡每個集合有一個子目錄。 每個集合目錄均包含一個 `data` 目錄，其中含有集合的快照。

4. 若要建立快照資料夾的壓縮封存，請使用下列命令：

    ```bash
    tar -zcf snapshot.20150806185338855.tgz snapshot.20150806185338855
    ```

    將 `snapshot.20150806185338855` 值取代為集合快照的名稱。

    此命令會建立名為 **snapshot.20150806185338855.tgz** 的封存，其中包含 **snapshot.20150806185338855** 目錄的內容。

5. 然後您可以使用下列命令，將封存儲存至叢集的主要儲存體：

    ```bash
    hdfs dfs -put snapshot.20150806185338855.tgz /example/data
    ```

如需有關使用 Solr 備份和還原的詳細資訊，請參閱 [https://cwiki.apache.org/confluence/display/solr/Making+and+Restoring+Backups](https://cwiki.apache.org/confluence/display/solr/Making+and+Restoring+Backups)。

## <a name="next-steps"></a>後續步驟

* [在 HDInsight 叢集上安裝 Giraph](hdinsight-hadoop-giraph-install-linux.md)。 在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Giraph。 Giraph 可讓您利用 Hadoop 執行圖形處理，且可以搭配 Azure HDInsight 一起使用。

* [在 HDInsight 叢集上安裝 Hue](hdinsight-hadoop-hue-linux.md)。 在 HDInsight Hadoop 叢集上使用叢集自訂安裝色調。 色調是一組 Web 應用程式，用來與 Hadoop 叢集互動。

[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster-linux.md
