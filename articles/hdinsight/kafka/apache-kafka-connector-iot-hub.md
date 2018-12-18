---
title: 搭配使用 HDInsight 上的 Apache Kafka 與 Azure IoT 中樞
description: 了解如何搭配使用 HDInsight 上的 Apache Kafka 與 Azure IoT 中樞。 Kafka Connect Azure IoT 中樞專案提供 Kafka 的來源和接收連接器。 來源連接器可從 IoT 中樞讀取資料，接收連接器則可寫入 IoT 中樞。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/15/2018
ms.openlocfilehash: 282fc6a1525238fba05c4f472b74d7eb55a49130
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042876"
---
# <a name="use-apache-kafka-on-hdinsight-with-azure-iot-hub"></a>搭配使用 HDInsight 上的 Apache Kafka 與 Azure IoT 中樞

了解如何使用 [Kafka Connect Azure IoT 中樞](https://github.com/Azure/toketi-kafka-connect-iothub)連接器在 HDInsight 上的 Apache Kafka 與 Azure IoT 中樞之間移動資料。 在本文件中，您將了解如何從叢集中的邊緣節點執行 IoT 中樞連接器。

Kafka Connect API 可讓您實作持續將資料提取到 Kafka 中，或將資料從 Kafka 推送至另一個系統的連接器。 [Kafka Connect Azure IoT 中樞](https://github.com/Azure/toketi-kafka-connect-iothub)是會將資料從 Azure IoT 中樞提取到 Kafka 中的連接器。 它也可以將資料從 Kafka 推送到 IoT 中樞。 

從 IoT 中樞提取時，您會使用__來源__連接器。 推送至 IoT 中樞時，您會使用__接收__連接器。 IoT 中樞連接器同時提供來源和接收連接器。

下圖顯示在使用連接器時，Azure IoT 中樞與 HDInsight 上的 Kafka 之間的資料流程。

![顯示資料透過連接器從 IoT 中樞流入 Kafka 的影像](./media/apache-kafka-connector-iot-hub/iot-hub-kafka-connector-hdinsight.png)

如需關於 Connect API 的詳細資訊，請參閱 [https://kafka.apache.org/documentation/#connect](https://kafka.apache.org/documentation/#connect)。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

* Kafka on HDInsight 叢集。 如需詳細資訊，請參閱 [HDInsight 上的 Kafka 快速入門](apache-kafka-get-started.md)文件。

* Kafka 叢集中的邊緣節點。 如需詳細資訊，請參閱[搭配使用邊緣節點與 HDInsight](../hdinsight-apps-use-edge-node.md) 文件。

* Azure IoT 中樞。 使用本教學課程時，建議您參閱[將 Raspberry Pi 線上模擬器連線至 Azure IoT 中樞](https://docs.microsoft.com/azure/iot-hub/iot-hub-raspberry-pi-web-simulator-get-started)文件。

* SSH 用戶端。 本文件中的步驟會使用 SSH 來連線到叢集。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

## <a name="install-the-connector"></a>安裝連接器

1. 從 [https://github.com/Azure/toketi-kafka-connect-iothub/releases/](https://github.com/Azure/toketi-kafka-connect-iothub/releases) 下載最新版本的 Kafka Connect for Azure IoT 中樞。

2. 若要將 .jar 檔案上傳至「HDInsight 上的 Kafka」叢集的邊緣節點，請使用下列命令：

    > [!NOTE]
    > 將 `sshuser` 取代為 HDInsight 叢集的 SSH 使用者帳戶。 將 `new-edgenode` 取代為邊緣節點名稱。 將 `clustername` 取代為叢集名稱。 如需與邊緣節點的 SSH 端點有關的詳細資訊，請參閱[搭配使用邊緣節點與 HDInsight](../hdinsight-apps-use-edge-node.md#access-an-edge-node) 文件。

    ```bash
    scp kafka-connect-iothub-assembly*.jar sshuser@new-edgenode.clustername-ssh.azurehdinsight.net:
    ```

3. 檔案複製完成後，請使用 SSH 連線至邊緣節點：

    ```bash
    ssh sshuser@new-edgenode.clustername-ssh.azurehdinsight.net
    ```

    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

4. 若要將連接器安裝到 Kafka`libs` 目錄中，請使用下列命令：

    ```bash
    sudo mv kafka-connect-iothub-assembly*.jar /usr/hdp/current/kafka-broker/libs/
    ```

> [!TIP]
> 如果您在本文件的其餘步驟中使用預先建置的 .jar 檔案時遇到問題，請嘗試從來源建置套件。
>
> 若要建立連接器，您必須具有 Scala 開發環境，並使用 [Scala 建置工具](http://www.scala-sbt.org/)。
>
> 1. 從 [https://github.com/Azure/toketi-kafka-connect-iothub/](https://github.com/Azure/toketi-kafka-connect-iothub/) 將連接器的來源下載到您的部署環境。
>
> 2. 從專案目錄中的命令提示字元，使用下列命令建置並封裝專案：
>
>    ```bash
>    sbt assembly
>    ```
>
>    此命令會在專案的 `target/scala-2.11` 目錄中建立名為 `kafka-connect-iothub-assembly_2.11-0.6.jar` 的檔案。

## <a name="configure-kafka"></a>設定 Kafka

透過邊緣節點的 SSH 連線，使用下列步驟設定在獨立模式中執行連接器的 Kafka：

1. 將叢集名稱儲存到變數。 使用變數可讓您更輕鬆地複製/貼上本節中的其他命令：

    ```bash
    read -p "Enter the Kafka on HDInsight cluster name: " CLUSTERNAME
    ```

2. 安裝 `jq` 公用程式。 此公用程式有助於處理 Ambari 查詢所傳回的 JSON 文件：

    ```bash
    sudo apt -y install jq
    ```

3. 取得 Kafka 訊息代理程式的位址。 您的叢集中可能會有許多訊息代理程式，但您只需要參考其中一或兩個。 若要取得兩個訊息代理程式主機的位址，請使用下列命令：

    ```bash
    export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    echo $KAFKABROKERS
    ```

    出現提示時，輸入叢集登入 (admin) 帳戶的密碼。 傳回的值類似下列文字︰

    `wn0-kafka.w5ijyohcxt5uvdhhuaz5ra4u5f.ex.internal.cloudapp.net:9092,wn1-kafka.w5ijyohcxt5uvdhhuaz5ra4u5f.ex.internal.cloudapp.net:9092`

4. 取得 Zookeeper 節點的位址。 叢集中可能會有數個 Zookeeper 節點，但您只需要參考其中一或兩個。 若要取得兩個 Zookeeper 節點的位址，請使用下列命令：

    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

5. 在獨立模式中執行連接器時，會使用 `/usr/hdp/current/kafka-broker/config/connect-standalone.properties` 檔案與 Kafka 訊息代理程式通訊。 若要編輯 `connect-standalone.properties` 檔案，請使用下列命令：

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-standalone.properties
    ```

    * 若要設定讓邊緣節點找出 Kafka 訊息代理程式的獨立組態，請找出開頭為 `bootstrap.servers=` 的那一行。 請將 `localhost:9092` 值取代為先前步驟中的訊息代理程式主機。

    * 將 `key.converter=` 和 `value.converter=` 這兩行變更為下列值：

        ```text
        key.converter=org.apache.kafka.connect.storage.StringConverter
        value.converter=org.apache.kafka.connect.storage.StringConverter
        ```

        > [!IMPORTANT]
        > 此變更可讓您使用隨附於 Kafka 的主控台產生者進行測試。 對於其他產生者和取用者，您可能需要不同的轉換器。 如需使用其他轉換器值的相關資訊，請參閱 [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md)。

    * 在檔案結尾處加入包含下列文字的一行︰

        ```text
        consumer.max.poll.records=10
        ```

        > [!TIP]
        > 此變更可將接收連接器限定為一次 10 筆記錄以內，以防止連接器逾時。 如需詳細資訊，請參閱 [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md)。

6. 若要儲存檔案，請使用 __Ctrl + X__、__Y__ 和 __Enter__ 鍵。

7. 若要建立連接器所使用的主題，請使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic iotin --zookeeper $KAFKAZKHOSTS

    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic iotout --zookeeper $KAFKAZKHOSTS
    ```

    若要確認 `iotin` 和 `iotout` 主題存在，請使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
    ```

    `iotin` 主題可用來接收來自 IoT 中樞的訊息。 `iotout` 主題可用來將訊息傳送至 IoT 中樞。

## <a name="get-iot-hub-connection-information"></a>取得 IoT 中樞連線資訊

若要擷取連接器所使用 IoT 中樞資訊，請使用下列步驟：

1. 取得 IoT 中樞的事件中樞相容端點和事件中樞相容端點名稱。 若要取得這項資訊，請使用下列其中一個方法：

    * __在 [Azure 入口網站](https://portal.azure.com/)中__，使用下列步驟：

        1. 瀏覽至您的 IoT 中樞，並選取 [端點]。
        2. 在 [內建端點] 中，選取 [事件]。
        3. 在 [屬性] 中，複製下列欄位的值：

            * __事件中樞相容名稱__
            * __事件中樞相容端點__
            * __分割數__

        > [!IMPORTANT]
        > 入口網站中的端點值可能包含在此範例中不需要的多餘文字。 請擷取符合 `sb://<randomnamespace>.servicebus.windows.net/` 模式的文字。

    * __在 [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) 中__，使用下列命令：

        ```azure-cli
        az iot hub show --name myhubname --query "{EventHubCompatibleName:properties.eventHubEndpoints.events.path,EventHubCompatibleEndpoint:properties.eventHubEndpoints.events.endpoint,Partitions:properties.eventHubEndpoints.events.partitionCount}"
        ```

        請將 `myhubname` 取代為您的 IoT 中樞名稱。 回應會類似於下列文字：

        ```text
        "EventHubCompatibleEndpoint": "sb://ihsuprodbnres006dednamespace.servicebus.windows.net/",
        "EventHubCompatibleName": "iothub-ehub-myhub08-207673-d44b2a856e",
        "Partitions": 2
        ```

2. 取得__共用存取原則__和__金鑰__。 在此範例中，請使用__服務__金鑰。 若要取得這項資訊，請使用下列其中一個方法：

    * __在 [Azure 入口網站](https://portal.azure.com/)中__，使用下列步驟：

        1. 選取 [共用存取原則]，然後選取 [服務]。
        2. 複製 [主要金鑰] 值。
        3. 複製 [連接字串 – 主要金鑰] 的值。

    * __在 [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) 中__，使用下列命令：

        1. 若要取得主要金鑰值，請使用下列命令：

            ```azure-cli
            az iot hub policy show --hub-name myhubname --name service --query "primaryKey"
            ```

            請將 `myhubname` 取代為您的 IoT 中樞名稱。 回應是此中樞之 `service` 原則的主要金鑰。

        2. 若要取得 `service` 原則的連接字串，請使用下列命令：

            ```azure-cli
            az iot hub show-connection-string --name myhubname --policy-name service --query "connectionString"
            ```

            請將 `myhubname` 取代為您的 IoT 中樞名稱。 回應是 `service` 原則的連接字串。

## <a name="configure-the-source-connection"></a>設定來源連線

若要設定使用 IoT 中樞時的來源，請透過邊緣節點的 SSH 連線執行下列動作：

1. 在 `/usr/hdp/current/kafka-broker/config/` 目錄中建立 `connect-iot-source.properties` 檔案的複本。 若要從 toketi-kafka-connect-iothub 專案下載檔案，請使用下列命令：

    ```bash
    sudo wget -P /usr/hdp/current/kafka-broker/config/ https://raw.githubusercontent.com/Azure/toketi-kafka-connect-iothub/master/connect-iothub-source.properties
    ```

2. 若要編輯 `connect-iot-source.properties` 檔案並新增 IoT 中樞資訊，請使用下列命令：

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-iothub-source.properties
    ```

    在編輯器中找出下列項目，並加以變更：

    * `Kafka.Topic=PLACEHOLDER`：將 `PLACEHOLDER` 取代為 `iotin`。 從 IoT 中樞接收到的訊息會放在 `iotin` 主題中。
    * `IotHub.EventHubCompatibleName=PLACEHOLDER`：將 `PLACEHOLDER` 取代為事件中樞相容名稱。
    * `IotHub.EventHubCompatibleEndpoint=PLACEHOLDER`：將 `PLACEHOLDER` 取代為事件中樞相容端點。
    * `IotHub.Partitions=PLACEHOLDER`：將 `PLACEHOLDER` 取代為前述步驟中的分割區數目。
    * `IotHub.AccessKeyName=PLACEHOLDER`：將 `PLACEHOLDER` 取代為 `service`。
    * `IotHub.AccessKeyValue=PLACEHOLDER`：將 `PLACEHOLDER` 取代為 `service` 原則的主要金鑰。
    * `IotHub.StartType=PLACEHOLDER`：將 `PLACEHOLDER` 取代為 UTC 日期。 此日期是連接器開始檢查訊息的時間。 日期格式為 `yyyy-mm-ddThh:mm:ssZ`。
    * `BatchSize=100`：將 `100` 取代為 `5`。 此變更會使連接器在 IoT 中樞內有五個新訊息之後，將訊息讀取到 Kafka 中。

    如需範例組態，請參閱 [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md)。

3. 若要儲存變更，請使用 __Ctrl + X__、__Y__ 和 __Enter__ 鍵。

如需關於設定連接器來源的詳細資訊，請參閱 [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md)。

## <a name="configure-the-sink-connection"></a>設定接收連線

若要設定使用 IoT 中樞時的接收連線，請透過邊緣節點的 SSH 連線執行下列動作：

1. 在 `/usr/hdp/current/kafka-broker/config/` 目錄中建立 `connect-iothub-sink.properties` 檔案的複本。 若要從 toketi-kafka-connect-iothub 專案下載檔案，請使用下列命令：

    ```bash
    sudo wget -P /usr/hdp/current/kafka-broker/config/ https://raw.githubusercontent.com/Azure/toketi-kafka-connect-iothub/master/connect-iothub-sink.properties
    ```

2. 若要編輯 `connect-iothub-sink.properties` 檔案並新增 IoT 中樞資訊，請使用下列命令：

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-iothub-sink.properties
    ```

    在編輯器中找出下列項目，並加以變更：

    * `topics=PLACEHOLDER`：將 `PLACEHOLDER` 取代為 `iotout`。 寫入 `iotout` 主題的訊息會轉送至 IoT 中樞。
    * `IotHub.ConnectionString=PLACEHOLDER`：將 `PLACEHOLDER` 取代為 `service` 原則的連接字串。

    如需範例組態，請參閱 [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md)。

3. 若要儲存變更，請使用 __Ctrl + X__、__Y__ 和 __Enter__ 鍵。

如需關於設定連接器接收端的詳細資訊，請參閱 [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md)。

## <a name="start-the-source-connector"></a>啟動來源連接器

若要啟動來源連接器，請透過邊緣節點的 SSH 連線使用下列命令：

```bash
/usr/hdp/current/kafka-broker/bin/connect-standalone.sh /usr/hdp/current/kafka-broker/config/connect-standalone.properties /usr/hdp/current/kafka-broker/config/connect-iothub-source.properties
```

連接器啟動後，請從您的裝置將訊息傳送至 IoT 中樞。 連接器從 IoT 中樞讀取訊息並將其儲存於 Kafka 主題時，會將資訊記錄至主控台：

```text
[2017-08-29 20:15:46,112] INFO Polling for data - Obtained 5 SourceRecords from IotHub (com.microsoft.azure.iot.kafka.co
nnect.IotHubSourceTask:39)
[2017-08-29 20:15:54,106] INFO Finished WorkerSourceTask{id=AzureIotHubConnector-0} commitOffsets successfully in 4 ms (
org.apache.kafka.connect.runtime.WorkerSourceTask:356)
```

> [!NOTE]
> 當連接器啟動時，您可能會看到幾則警告。 這些警告不會導致從 IoT 中樞接收訊息的問題。

若要停止連接器，請使用 __Ctrl + C__。

## <a name="start-the-sink-connector"></a>啟動接收連接器

透過邊緣節點的 SSH 連線，使用下列命令在獨立模式中啟動接收連接器：

```bash
/usr/hdp/current/kafka-broker/bin/connect-standalone.sh /usr/hdp/current/kafka-broker/config/connect-standalone.properties /usr/hdp/current/kafka-broker/config/connect-iothub-sink.properties
```

連接器執行時，會顯示類似下列文字的資訊：

```text
[2017-08-30 17:49:16,150] INFO Started tasks to send 1 messages to devices. (com.microsoft.azure.iot.kafka.connect.sink.
IotHubSinkTask:47)
[2017-08-30 17:49:16,150] INFO WorkerSinkTask{id=AzureIotHubSinkConnector-0} Committing offsets (org.apache.kafka.connec
t.runtime.WorkerSinkTask:262)
```

> [!NOTE]
> 當連接器啟動時，您可能會看到幾則警告。 您可以放心地忽略這些警告。

若要透過連接器傳送訊息，請使用下列步驟：

1. 開啟 Kafka 叢集的另一個 SSH 工作階段：

    ```bash
    ssh sshuser@new-edgenode.clustername-ssh.azurehdinsight.net
    ```
2. 若要將訊息傳送至 `iotout` 主題，請使用下列命令：

    > [!WARNING]
    > 由於這是新的 SSH 連線，`$KAFKABROKERS` 變數並未包含任何資訊。 若要加以設定，請使用下列其中一個方法：
    >
    > * 使用[設定 Kafka](#configure-kafka) 一節中的前三個步驟。
    > * 使用先前 SSH 連線中的 `echo $KAFKABROKERS` 以取得值，並在下列命令中將 `$KAFKABROKERS` 取代為實際值。

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic iotout
    ```

    此命令不會使您返回一般 Bash 提示字元。 它會將鍵盤輸入傳送至 `iotout` 主題。

3. 若要將訊息傳送至您的裝置，請將 JSON 文件貼到 `kafka-console-producer` 的 SSH 工作階段中。

    > [!IMPORTANT]
    > 您必須將 `"deviceId"` 項目的值設定為您的裝置識別碼。 在下列範例中，裝置會命名為 `fakepi`：

    ```text
    {"messageId":"msg1","message":"Turn On","deviceId":"fakepi"}
    ```

    此 JSON 文件的結構描述在 [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md) 有更詳細的說明。

    如果您使用模擬 Raspberry Pi 裝置，且正在執行中，裝置將會記錄下列訊息：

    ```text
    Receive message: Turn On
    ```

    重新傳送 JSON 文件，但變更 `"message"` 項目的值。 裝置會記錄新的值。

如需使用接收連接器的詳細資訊，請參閱 [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md)。

## <a name="next-steps"></a>後續步驟

在本文件中，您已了解如何使用 Kafka Connect API 在 HDInsight 上啟動 IoT Kafka Connector。 使用下列連結來探索使用 Kafka 的其他方式︰

* [使用 Apache Spark 搭配 Kafka on HDInsight](../hdinsight-apache-spark-with-kafka.md)
* [使用 Apache Storm 搭配 HDInsight 上的 Kafka](../hdinsight-apache-storm-with-kafka.md)
