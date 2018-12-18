---
title: Apache Storm 寫入儲存體/Data Lake Store - Azure HDInsight
description: 了解如何使用 Apache Storm 寫入 HDInsight 的 HDFS 相容儲存體。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.openlocfilehash: 670d465f1481592862f5e653a508460c19be07d8
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049686"
---
# <a name="write-to-hdfs-from-apache-storm-on-hdinsight"></a>從 Apache Storm on HDInsight 寫入 HDFS

了解如何使用 Storm 將資料寫入 Apache Storm on HDInsight 所使用的 HDFS 相容儲存體。 HDInsight 可以同時使用 Azure 儲存體以及 Azure Data Lake Store 作為 HDFS 相容儲存體。 Storm 提供了將資料寫入 HDFS 的 [HdfsBolt](http://storm.apache.org/releases/current/javadocs/org/apache/storm/hdfs/bolt/HdfsBolt.html) 元件。 本文件提供從 HdfsBolt 寫入任一類型儲存體的資訊。 

> [!IMPORTANT]
> 本文件使用的範例拓撲依賴 Storm on HDInsight 隨附的元件。 它可能需要進行修改，才能在與其他 Apache Storm 叢集搭配使用時使用 Azure Data Lake Store。

## <a name="get-the-code"></a>取得程式碼

包含此拓撲的專案可從 [https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store](https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store) 下載。

若要編譯此專案，您需要開發環境的下列設定：

* [Java JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 或更新版本。 HDInsight 3.5 或更新版本需要 Java 8。

* [Maven 3.x](https://maven.apache.org/download.cgi)

當您在開發工作站上安裝 Java 和 JDK 時可能會設定下列環境變數。 不過，您應該檢查它們是否存在，以及它們是否包含您系統的正確值。

* `JAVA_HOME` - 應指向已安裝 JDK 的目錄。
* `PATH` - 應該包含下列路徑：
  
    * `JAVA_HOME` (或對等的路徑)。
    * `JAVA_HOME\bin` (或對等的路徑)。
    * 已安裝 Maven 的目錄。

## <a name="how-to-use-the-hdfsbolt-with-hdinsight"></a>如何搭配 HDInsight 使用 HdfsBolt

> [!IMPORTANT]
> 在搭配 Storm on HDInsight 使用 HdfsBolt 之前，您必須先使用指令碼動作，將所需的 Jar 檔案複製到 Storm 的 `extpath`。 如需詳細資訊，請參閱[設定叢集](#configure)一節。

HdfsBolt 會使用您提供的檔案配置來了解如何寫入 HDFS。 利用 HDInsight 時，請使用下列其中一項配置：

* `wasb://`：搭配使用 Azure 儲存體帳戶。
* `adl://`：搭配使用 Azure Data Lake Store。

下表提供針對不同案例使用檔案配置的範例：

| 配置 | 注意 |
| ----- | ----- |
| `wasb:///` | 預設儲存體帳戶是 Azure 儲存體帳戶中的 Blob 容器 |
| `adl:///` | 預設儲存體帳戶是 Azure Data Lake Store 中的目錄。 叢集建立期間，您可以指定 Data Lake Store 中其為叢集 HDFS 根目錄的目錄。 例如，`/clusters/myclustername/` 目錄。 |
| `wasb://CONTAINER@ACCOUNT.blob.core.windows.net/` | 與叢集建立關聯的非預設 (其他) Azure 儲存體帳戶。 |
| `adl://STORENAME/` | 叢集所使用的 Data Lake Store 根目錄。 此配置可讓您存取位於叢集檔案系統所在目錄外部的資料。 |

如需詳細資訊，請參閱 Apache.org 上的 [HdfsBolt](http://storm.apache.org/releases/current/javadocs/org/apache/storm/hdfs/bolt/HdfsBolt.html) 參考。

### <a name="example-configuration"></a>設定範例

下列 YAML 是摘錄自範例中隨附的 `resources/writetohdfs.yaml` 檔案。 此檔案會使用 Apache Storm 的 [Flux](https://storm.apache.org/releases/1.1.2/flux.html) 架構定義 Storm 拓撲。

```yaml
components:
  - id: "syncPolicy"
    className: "org.apache.storm.hdfs.bolt.sync.CountSyncPolicy"
    constructorArgs:
      - 1000

  # Rotate files when they hit 5 MB
  - id: "rotationPolicy"
    className: "org.apache.storm.hdfs.bolt.rotation.FileSizeRotationPolicy"
    constructorArgs:
      - 5
      - "MB"

  - id: "fileNameFormat"
    className: "org.apache.storm.hdfs.bolt.format.DefaultFileNameFormat"
    configMethods:
      - name: "withPath"
        args: ["${hdfs.write.dir}"]
      - name: "withExtension"
        args: [".txt"]

  - id: "recordFormat"
    className: "org.apache.storm.hdfs.bolt.format.DelimitedRecordFormat"
    configMethods:
      - name: "withFieldDelimiter"
        args: ["|"]

# spout definitions
spouts:
  - id: "tick-spout"
    className: "com.microsoft.example.TickSpout"
    parallelism: 1


# bolt definitions
bolts:
  - id: "hdfs-bolt"
    className: "org.apache.storm.hdfs.bolt.HdfsBolt"
    configMethods:
      - name: "withConfigKey"
        args: ["hdfs.config"]
      - name: "withFsUrl"
        args: ["${hdfs.url}"]
      - name: "withFileNameFormat"
        args: [ref: "fileNameFormat"]
      - name: "withRecordFormat"
        args: [ref: "recordFormat"]
      - name: "withRotationPolicy"
        args: [ref: "rotationPolicy"]
      - name: "withSyncPolicy"
        args: [ref: "syncPolicy"]
```

此 YAML 會定義下列項目：

* `syncPolicy`：定義檔案何時同步處理/排清到檔案系統。 在此範例中，為每 1000 個 Tuple。
* `fileNameFormat`：定義寫入檔案時要使用的路徑和檔案名稱模式。 在此範例中，會使用篩選條件在執行階段提供路徑，而檔案的副檔名為 `.txt`。
* `recordFormat`：定義所寫入檔案的內部格式。 在此範例中，會以 `|` 字元分隔欄位。
* `rotationPolicy`：定義何時輪替檔案。 在此範例中，不會執行輪替。
* `hdfs-bolt`：使用舊版元件作為 `HdfsBolt` 類別的設定參數。

如需有關 Flux 架構的詳細資訊，請參閱 [https://storm.apache.org/releases/1.1.2/flux.html](https://storm.apache.org/releases/1.1.2/flux.html)。

## <a name="configure-the-cluster"></a>設定叢集

根據預設，Storm on HDInsight 未包含 HdfsBolt 用來與 Storm Classpath 中出現的 Azure 儲存體或 Data Lake Store 進行通訊的元件。 請使用下列指令碼動作，將這些元件新增至叢集上 Storm 的 `extlib` 目錄：

* 指令碼 URI：`https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh`
* 要套用的節點：Nimbus、Supervisor
* 參數：無

如需搭配叢集使用此指令碼的資訊，請參閱[使用指令碼動作自訂 HDInsight 叢集](./../hdinsight-hadoop-customize-cluster-linux.md)文件。

## <a name="build-and-package-the-topology"></a>建置和封裝拓撲

1. 從 [https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store ](https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store) 下載範例專案到您的部署環境。

2. 從命令提示字元、終端機或 Shell 工作階段，將目錄變更為所下載專案的根目錄。 若要建置和封裝拓撲，請使用下列命令：
   
        mvn compile package
   
    建置和封裝完成之後，會有名為 `target` 的新目錄，其中包含名為 `StormToHdfs-1.0-SNAPSHOT.jar` 的檔案。 此檔案包含已編譯的拓撲。

## <a name="deploy-and-run-the-topology"></a>部署並執行拓撲

1. 使用下列命令將拓撲複製到 HDInsight 叢集。 將 **USER** 取代為建立叢集時所使用的 SSH 使用者名稱。 將 **CLUSTERNAME** 取代為叢集的名稱。
   
        scp target\StormToHdfs-1.0-SNAPSHOT.jar USER@CLUSTERNAME-ssh.azurehdinsight.net:StormToHdfs-1.0-SNAPSHOT.jar
   
    出現提示時，請輸入建立叢集的 SSH 使用者時所使用的密碼。 如果您使用公開金鑰而非密碼，則可能必須使用 `-i` 參數來指定相對應私密金鑰的路徑。
   
   > [!NOTE]
   > 如需搭配 HDInsight 使用 `scp` 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 上傳完成後，使用下列命令來透過 SSH 連接到 HDInsight 叢集。 將 **USER** 取代為建立叢集時所使用的 SSH 使用者名稱。 將 **CLUSTERNAME** 取代為叢集的名稱。
   
        ssh USER@CLUSTERNAME-ssh.azurehdinsight.net
   
    出現提示時，請輸入建立叢集的 SSH 使用者時所使用的密碼。 如果您使用公開金鑰而非密碼，則可能必須使用 `-i` 參數來指定相對應私密金鑰的路徑。
   
   如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

3. 連接之後，使用下列命令來建立名為 `dev.properties` 的檔案：

        nano dev.properties

4. 使用下列文字做為 `dev.properties` 檔案的內容：

        hdfs.write.dir: /stormdata/
        hdfs.url: wasb:///

    > [!IMPORTANT]
    > 這個範例假設您的叢集使用 Azure 儲存體帳戶作為預設儲存體。 如果叢集使用 Azure Data Lake Store，請改為使用 `hdfs.url: adl:///`。
    
    若要儲存檔案，使用 __Ctrl + X__，然後是 __Y__，最後按 __Enter__。 此檔案中的值會設定 Data Lake Store URL，以及要寫入資料的目錄名稱。

3. 使用下列命令來啟動拓撲：
   
        storm jar StormToHdfs-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /writetohdfs.yaml --filter dev.properties

    此命令藉由將拓撲提交給叢集的 Nimbus 節點，以使用 Flux 架構來啟動拓撲。 拓撲是由 jar 中包含的 `writetohdfs.yaml` 檔案所定義。 `dev.properties` 檔案會傳遞來做為篩選，而檔案中包含的值是透過拓撲來讀取。

## <a name="view-output-data"></a>檢視輸出資料

若要檢視資料，請使用下列命令：

    hdfs dfs -ls /stormdata/

隨即會顯示此拓撲所建立的檔案清單。

下列清單是前一個命令所傳回的資料範例：

    Found 30 items
    -rw-r-----+  1 sshuser sshuser       488000 2017-03-03 19:13 /stormdata/hdfs-bolt-3-0-1488568403092.txt
    -rw-r-----+  1 sshuser sshuser       444000 2017-03-03 19:13 /stormdata/hdfs-bolt-3-1-1488568404567.txt
    -rw-r-----+  1 sshuser sshuser       502000 2017-03-03 19:13 /stormdata/hdfs-bolt-3-10-1488568408678.txt
    -rw-r-----+  1 sshuser sshuser       582000 2017-03-03 19:13 /stormdata/hdfs-bolt-3-11-1488568411636.txt
    -rw-r-----+  1 sshuser sshuser       464000 2017-03-03 19:13 /stormdata/hdfs-bolt-3-12-1488568411884.txt

## <a name="stop-the-topology"></a>停止拓撲

Storm 拓撲會一直執行，直到其停止或叢集遭到刪除為止。 若要停止拓撲，請使用下列命令：

    storm kill hdfswriter

## <a name="delete-your-cluster"></a>刪除叢集

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="next-steps"></a>後續步驟

現在，您已了解如何使用 Storm 來寫入至 Azure 儲存體和 Azure Data Lake Store，接下來請探索其他 [HDInsight 的 Storm 範例](apache-storm-example-topology.md)。

