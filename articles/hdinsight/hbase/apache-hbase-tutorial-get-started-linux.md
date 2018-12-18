---
title: 開始使用 HDInsight 上的 HBase 範例 - Azure
description: 遵循此 Apache HBase 範例開始在 HDInsight 上使用 Hadoop。 使用 Hive 從 HBase Shell 建立資料表並加以查詢。
keywords: hbasecommand,hbase 範例
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 02/22/2018
ms.author: jasonh
ms.openlocfilehash: e31d42cc8ec117e00d83a076d1690cd37c2fc4ef
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042406"
---
# <a name="get-started-with-an-apache-hbase-example-in-hdinsight"></a>開始使用 HDInsight 中的 Apache HBase 範例

了解如何使用 Hive 在 HDInsight 中建立 HBase 叢集、建立 HBase 資料表，以及查詢資料表。 如需一般 HBase 資訊，請參閱 [HDInsight HBase 概觀][hdinsight-hbase-overview]。

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="prerequisites"></a>必要條件
開始嘗試進行本 HBase 範例之前，您必須具備下列項目：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* [安全殼層 (SSH)](../hdinsight-hadoop-linux-use-ssh-unix.md)。 
* [cURL](http://curl.haxx.se/download.html)。

## <a name="create-hbase-cluster"></a>建立 HBase 叢集
下列程序會使用 Azure Resource Manager 範本建立 HBase 叢集和相依的預設 Azure 儲存體帳戶。 若要了解此程序與其他叢集建立方法中使用的參數，請參閱 [在 HDInsight 中建立以 Linux 為基礎的 Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。 如需有關如何使用 Data Lake Storage Gen2 的詳細資訊，請參閱[快速入門：在 HDInsight 中設定叢集](../../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)。

1. 按一下以下影像，在 Azure 入口網站中開啟範本。 範本位在 [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/)。
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-hbase-linux%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-tutorial-get-started-linux/deploy-to-azure.png" alt="Deploy to Azure"></a>
2. 從 [自訂部署] 刀鋒視窗，輸入下列值：
   
   * **訂用帳戶**：選取用來建立叢集的 Azure 訂用帳戶。
   * **資源群組**：建立 Azure 資源管理群組，或使用現有的資源管理群組。
   * **位置**：指定資源群組的位置。 
   * **叢集名稱**：輸入 HBase 叢集的名稱。
   * **叢集登入名稱和密碼**：預設登入名稱是 **admin**。
   * **SSH 使用者名稱和密碼**：預設使用者名稱是 **sshuser**。  您可以將它重新命名。
     
     其他參數都是選擇性的。  
     
     每個叢集都具備 Azure 儲存體帳戶相依性。 刪除叢集之後，資料會保留在儲存體帳戶中。 叢集預設儲存體帳戶名稱是附加 "store" 的叢集名稱。 它會硬式編碼在範本變數區段中。
3. 選取 [我同意上方所述的條款及條件]，然後按一下 [購買]。 大約需要 20 分鐘的時間來建立叢集。

> [!NOTE]
> 刪除 HBase 叢集之後，您可以使用相同的預設 Blob 容器建立另一個 HBase 叢集。 這個新叢集會挑選您在原始叢集中建立的 HBase 資料表。 為了避免不一致，建議您在刪除叢集之前，先停用 HBase 資料表。
> 
> 

## <a name="create-tables-and-insert-data"></a>建立資料表和插入資料
您可以使用 SSH 來連線到 HBase 叢集，然後使用 HBase Shell 來建立 HBase 資料表、插入及查詢資料。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

對大多數人而言，資料會以表格形式出現：

![HDInsight HBase 表格式資料][img-hbase-sample-data-tabular]

在 HBase (實作 BigTable) 中，相同的資料看起來如下：

![HDInsight HBase BigTable 資料][img-hbase-sample-data-bigtable]


**使用 HBase Shell**

1. 從 SSH，執行下列 HBase 命令：
   
    ```bash
    hbase shell
    ```

2. 使用兩個資料行系列建立 HBase：

    ```hbaseshell   
    create 'Contacts', 'Personal', 'Office'
    list
    ```
3. 插入一些資料：
    
    ```hbaseshell   
    put 'Contacts', '1000', 'Personal:Name', 'John Dole'
    put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
    put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
    put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
    scan 'Contacts'
    ```
   
    ![HDInsight Hadoop HBase 殼層][img-hbase-shell]
4. 取得單一資料列
   
    ```hbaseshell
    get 'Contacts', '1000'
    ```
   
    您會看到與使用掃描命令相同的結果，因為只有一個資料列。
   
    如需 HBase 資料表結構描述的詳細資訊，請參閱 [HBase 結構描述設計簡介][hbase-schema]。 如需其他 HBase 命令，請參閱 [Apache HBase 參考指南][hbase-quick-start]。
5. 結束 Shell
   
    ```hbaseshell
    exit
    ```

**將資料大量載入連絡人 HBase 資料表中**

HBase 包含數個將資料載入資料表的方法。  如需詳細資訊，請參閱 [大量載入](http://hbase.apache.org/book.html#arch.bulk.load)。

您可以在公用 Blob 容器中找到資料檔案範例 (*wasb://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt*)。  資料檔案的內容：

    8396    Calvin Raji      230-555-0191    230-555-0191    5415 San Gabriel Dr.
    16600   Karen Wu         646-555-0113    230-555-0192    9265 La Paz
    4324    Karl Xie         508-555-0163    230-555-0193    4912 La Vuelta
    16891   Jonn Jackson     674-555-0110    230-555-0194    40 Ellis St.
    3273    Miguel Miller    397-555-0155    230-555-0195    6696 Anchor Drive
    3588    Osa Agbonile     592-555-0152    230-555-0196    1873 Lion Circle
    10272   Julia Lee        870-555-0110    230-555-0197    3148 Rose Street
    4868    Jose Hayes       599-555-0171    230-555-0198    793 Crawford Street
    4761    Caleb Alexander  670-555-0141    230-555-0199    4775 Kentucky Dr.
    16443   Terry Chander    998-555-0171    230-555-0200    771 Northridge Drive

您可以選擇性地建立文字檔，並將檔案上載至自己的儲存體帳戶。 如需指示，請參閱[在 HDInsight 中將 Hadoop 工作的資料上傳][hdinsight-upload-data]。

> [!NOTE]
> 此程序會使用您在上一個程序中建立的連絡人 HBase 資料表。
> 

1. 從 SSH，執行下列命令，將資料檔案轉換成 StoreFiles 並存放在 Dimporttsv.bulk.output 所指定的相對路徑。  如果您在 HBase Shell 中，請使用 exit 命令來結束。

    ```bash   
    hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns="HBASE_ROW_KEY,Personal:Name,Personal:Phone,Office:Phone,Office:Address" -Dimporttsv.bulk.output="/example/data/storeDataFileOutput" Contacts wasb://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt
    ```

2. 執行下列命令，將資料從 /example/data/storeDataFileOutput 上傳至 HBase 資料表：
   
    ```bash
    hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /example/data/storeDataFileOutput Contacts
    ```

3. 您可以開啟 HBase Shell，並使用掃描命令來列出資料表內容。

## <a name="use-hive-to-query-hbase"></a>使用 Hive 查詢 HBase

您可以使用 Hive 查詢 HBase 資料表中的資料。 在本節中，您會建立對應至 HBase 資料表的 Hive 資料表，並用以查詢您 HBase 資料表中的資料。

1. 開啟 **PuTTY**，然後連線到叢集。  請參閱先前程序中的指示。
2. 在 SSH 工作階段中，使用以下命令啟動 Beeline：

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin
    ```

    如需有關 Beeline 的詳細資訊，請參閱[利用 Beeline 搭配使用 Hive 與 HDInsight 中的 Hadoop](../hadoop/apache-hadoop-use-hive-beeline.md)。
       
3. 執行下列 HiveQL 指令碼以建立對應到 HBase 資料表的 Hive 資料表。 在執行此陳述式前，請確定您已使用 HBase Shell 建立參考先前本教學課程的範例資料表。

    ```hiveql   
    CREATE EXTERNAL TABLE hbasecontacts(rowkey STRING, name STRING, homephone STRING, officephone STRING, officeaddress STRING)
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES ('hbase.columns.mapping' = ':key,Personal:Name,Personal:Phone,Office:Phone,Office:Address')
    TBLPROPERTIES ('hbase.table.name' = 'Contacts');
    ```

4. 執行下列 HiveQL 指令碼，以查詢 HBase 資料表中的資料：

    ```hiveql   
    SELECT count(rowkey) FROM hbasecontacts;
    ```

## <a name="use-hbase-rest-apis-using-curl"></a>使用 Curl 來使用 HBase REST API

透過 [基本驗證](http://en.wikipedia.org/wiki/Basic_access_authentication)來保護 REST API 的安全。 您應該一律使用安全 HTTP (HTTPS) 提出要求，確保認證安全地傳送至伺服器。

2. 使用下列命令列出現有的 HBase 資料表：

    ```bash
    curl -u <UserName>:<Password> \
    -G https://<ClusterName>.azurehdinsight.net/hbaserest/
    ```

3. 使用下列命令建立含兩個資料欄系列的新 HBase 資料表：

    ```bash   
    curl -u <UserName>:<Password> \
    -X PUT "https://<ClusterName>.azurehdinsight.net/hbaserest/Contacts1/schema" \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -d "{\"@name\":\"Contact1\",\"ColumnSchema\":[{\"name\":\"Personal\"},{\"name\":\"Office\"}]}" \
    -v
    ```

    結構描述是以 JSON 格式提供。
4. 使用下列命令插入一些資料：

    ```bash   
    curl -u <UserName>:<Password> \
    -X PUT "https://<ClusterName>.azurehdinsight.net/hbaserest/Contacts1/false-row-key" \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -d "{\"Row\":[{\"key\":\"MTAwMA==\",\"Cell\": [{\"column\":\"UGVyc29uYWw6TmFtZQ==\", \"$\":\"Sm9obiBEb2xl\"}]}]}" \
    -v
    ```
   
    您必須使用 base64 編碼 -d 參數中指定的值。 在範例中︰
   
   * MTAwMA==: 1000
   * UGVyc29uYWw6TmFtZQ==: Personal:Name
   * Sm9obiBEb2xl: John Dole
     
     [false-row-key](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/rest/package-summary.html#operation_cell_store_single) 可讓您插入多個 (批次) 值。
5. 使用下列命令取得資料列：
   
    ```bash 
    curl -u <UserName>:<Password> \
    -X GET "https://<ClusterName>.azurehdinsight.net/hbaserest/Contacts1/1000" \
    -H "Accept: application/json" \
    -v
    ```

如需 HBase Rest 的詳細資訊，請參閱 [Apache HBase 參考指南](https://hbase.apache.org/book.html#_rest)。

> [!NOTE]
> Thrift 不受 HDInsight 中的 HBase 所支援。
>
> 在使用 Curl 或與 WebHCat 進行任何其他 REST 通訊時，您必須提供 HDInsight 叢集系統管理員的使用者名稱和密碼來驗證要求。 您也必須在用來將要求傳送至伺服器的統一資源識別項 (URI) 中使用叢集名稱：
> 
>   
>        curl -u <UserName>:<Password> \
>        -G https://<ClusterName>.azurehdinsight.net/templeton/v1/status
>   
>    您應該會收到類似下列的回應：
>   
>        {"status":"ok","version":"v1"}
   


## <a name="check-cluster-status"></a>檢查叢集狀態
HDInsight 中的 HBase 隨附於 Web UI，以供監視叢集。 使用 Web UI，您可要求關於區域的統計資料或資訊。

**存取 HBase 主要 UI**

1. 登入 Ambari Web UI (https://&lt;Clustername>.azurehdinsight.net)。
2. 按一下左側功能表的 [HBase]。
3. 按一下頁面頂端的 [快速連結]，指向使用中的 Zookeeper 節點連結，然後按一下 [HBase 主要 UI]。  UI 會在另一個瀏覽器索引標籤中開啟：

  ![HDInsight HBase HMaster UI](./media/apache-hbase-tutorial-get-started-linux/hdinsight-hbase-hmaster-ui.png)

  HBase 主要 UI 包含下列區段：

  - 區域伺服器
  - 備份主機
  - tables
  - 工作
  - 軟體屬性

## <a name="delete-the-cluster"></a>刪除叢集
為了避免不一致，建議您在刪除叢集之前，先停用 HBase 資料表。

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="troubleshoot"></a>疑難排解

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](../hdinsight-administer-use-portal-linux.md#create-clusters)。

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何建立 HBase 叢集，以及如何建立資料表，並從 HBase Shell 檢視這些資料表中的資料。 您同時也了解到如何使用 Hive 查詢 HBase 資料表中的資料，以及如何使用 HBase C# REST API 建立 HBase 資料表，並擷取其資料表中的資料。

若要深入了解，請參閱：

* [HDInsight HBase 概觀][hdinsight-hbase-overview]：HBase 是建置於 Hadoop 上的 Apache 開放原始碼 NoSQL 資料庫，可針對大量非結構化及半結構化資料，提供隨機存取功能和強大一致性。

[hdinsight-manage-portal]: hdinsight-administer-use-management-portal.md
[hdinsight-upload-data]: ../hdinsight-upload-data.md
[hbase-reference]: http://hbase.apache.org/book.html#importtsv
[hbase-schema]: http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf
[hbase-quick-start]: http://hbase.apache.org/book.html#quickstart





[hdinsight-hbase-overview]:apache-hbase-overview.md
[hdinsight-hbase-provision-vnet]:apache-hbase-provision-vnet.md
[hdinsight-versions]: hdinsight-component-versioning.md
[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[azure-portal]: https://portal.azure.com/
[azure-create-storageaccount]: http://azure.microsoft.com/documentation/articles/storage-create-storage-account/

[img-hbase-shell]: ./media/apache-hbase-tutorial-get-started-linux/hdinsight-hbase-shell.png
[img-hbase-sample-data-tabular]: ./media/apache-hbase-tutorial-get-started-linux/hdinsight-hbase-contacts-tabular.png
[img-hbase-sample-data-bigtable]: ./media/apache-hbase-tutorial-get-started-linux/hdinsight-hbase-contacts-bigtable.png
