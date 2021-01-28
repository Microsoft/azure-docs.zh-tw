---
title: 鏡像 Apache Kafka 主題 - Azure HDInsight
description: 了解如何使用 Apache Kafka 的鏡像功能，藉由將主題鏡像處理至次要叢集來維護 HDInsight 叢集上的 Kafka 複本。
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/29/2019
ms.openlocfilehash: c2fce6d4ee95a56cc087d50184fcd69ac113620f
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98940852"
---
# <a name="use-mirrormaker-to-replicate-apache-kafka-topics-with-kafka-on-hdinsight"></a>使用 MirrorMaker，透過 HDInsight 上的 Kafka 來複寫 Apache Kafka 主題

了解如何使用 Apache Kafka 的鏡像功能，將主題複寫至次要叢集。 鏡像功能可以當作連續程序執行，或間歇地做為在叢集間移轉資料的方法。

> [!NOTE]
> 本文包含字詞「*允許清單*」的參考 (Microsoft 已不再使用該字詞)。 從軟體中移除該字詞時，我們也會將其從本文中移除。

在此範例中，會使用鏡像來複寫兩個 HDInsight 叢集之間的主題。 這兩個叢集都位於不同資料中心的不同虛擬網路中。

> [!WARNING]  
> 但不能將鏡像功能視為達成容錯的方法。 在主要和次要叢集之間，主題中專案的位移不同，因此用戶端無法交替使用這兩者。
>
> 如果您很擔心容錯，您應該為叢集內的主題設定複寫。 如需詳細資訊，請參閱 [開始使用 HDInsight 上的 Apache Kafka](apache-kafka-get-started.md)。

## <a name="how-apache-kafka-mirroring-works"></a>Apache Kafka 鏡像的運作方式

鏡像的運作方式是使用 [MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) 工具 (Apache Kafka) 的一部分，以取用主要叢集上主題中的記錄，然後在次要叢集上建立本機複本。 MirrorMaker 會使用一 (或多個讀取主要叢集的) 取用 *者* ，以及寫入本機 (次要) 叢集的 *生產者* 。

嚴重損壞修復最有用的鏡像設定會利用不同 Azure 區域中的 Kafka 叢集。 為了達成此目的，叢集所在的虛擬網路會對等互連在一起。

下圖說明鏡像程式和叢集之間的通訊流動方式：

![鏡像程序圖表](./media/apache-kafka-mirroring/kafka-mirroring-vnets2.png)

主要和次要叢集的節點和分割區數目可能不同，而主題內的位移也不同。 鏡像功能會維護用於資料分割的金鑰值，因此會根據每個金鑰保留記錄順序。

### <a name="mirroring-across-network-boundaries"></a>跨網路界限鏡像

如果您需要在不同網路中的 Kafka 叢集之間進行鏡像處理，有下列額外考量︰

* **閘道**：網路必須能夠在 tcp/ip 層級進行通訊。

* **伺服器定址**：您可以選擇使用 IP 位址或完整功能變數名稱來定址叢集節點。

    * **IP 位址**：如果您將 Kafka 叢集設定為使用 IP 位址廣告，您可以使用訊息代理程式節點和 zookeeper 節點的 IP 位址，繼續進行鏡像設定。
    
    * **功能變數名稱**：如果您未針對 IP 位址公告設定 Kafka 叢集，叢集必須能夠使用 (fqdn) 的完整功能變數名稱彼此連接。 這需要在設定為將要求轉送至其他網路的每個網路中， (DNS) 伺服器的網域名稱系統。 建立 Azure 虛擬網路 (而不是使用網路提供的自動 DNS) 時，您必須指定自訂 DNS 伺服器和伺服器的 IP 位址。 建立虛擬網路之後，您就必須建立使用該 IP 位址的 Azure 虛擬機器，然後在其上安裝和設定 DNS 軟體。

    > [!WARNING]  
    > 先建立和設定自訂 DNS 伺服器，然後再將 HDInsight 安裝到虛擬網路中。 HDInsight 不需要進行其他設定，即可使用針對虛擬網路設定的 DNS 伺服器。

如需有關如何連接兩個 Azure 虛擬網路的詳細資訊，請參閱[設定 VNet 對 VNet 連線](../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)。

## <a name="mirroring-architecture"></a>鏡像架構

此架構具有不同資源群組和虛擬網路中的兩個群集： **主要** 和 **次要**。

### <a name="creation-steps"></a>建立步驟

1. 建立兩個新的資源群組：

    |資源群組 | Location |
    |---|---|
    | kafka-主要-rg | 美國中部 |
    | kafka-次要-rg | 美國中北部 |

1. 在 **kafka** 中建立新的虛擬網路 **kafka-主要 vnet** 。 保留預設設定。
1. 在 kafka 中建立新的虛擬網路 **kafka-次要 vnet** - **次要-rg**，以及預設設定。

1. 建立兩個新的 Kafka 叢集：

    | 叢集名稱 | 資源群組 | 虛擬網路 | 儲存體帳戶 |
    |---|---|---|---|
    | kafka-主要叢集 | kafka-主要-rg | kafka-主要 vnet | kafkaprimarystorage |
    | kafka-次要叢集 | kafka-次要-rg | kafka-次要 vnet | kafkasecondarystorage |

1. 建立虛擬網路對等互連。 此步驟會建立兩個對等互連：一個從 **kafka-主要 vnet** 到 **kafka-次要 vnet** ，另一個則是從 **kafka-次要** vnet 再到 **kafka-主要 vnet**。
    1. 選取 [ **kafka-主要 vnet** 虛擬網路]。
    1. 在 [**設定**] 底下選取 [**對等互連**]。
    1. 選取 [新增]。
    1. 在 [ **新增對等互連** ] 畫面上，輸入詳細資料，如下列螢幕擷取畫面所示。

        ![HDInsight Kafka 新增 vnet 對等互連](./media/apache-kafka-mirroring/hdi-add-vnet-peering.png)

### <a name="configure-ip-advertising"></a>設定 IP 廣告

設定 IP 廣告，讓用戶端能夠使用訊息代理程式 IP 位址來連線，而不是使用功能變數名稱。

1. 移至主要叢集的 Ambari 儀表板： `https://PRIMARYCLUSTERNAME.azurehdinsight.net` 。
1. 選取 [**服務**  >  **Kafka**]。 CliSelectck [ **選項** ] 索引標籤。
1. 將下列設定行新增至底部的 **kafka 環境範本** 區段。 選取 [儲存]。

    ```
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

1. 在 [ **儲存** 設定] 畫面上輸入附注，然後按一下 [ **儲存**]。
1. 如果系統提示您設定警告，請按一下 [ **仍要繼續**]。
1. 選取 [儲存設定 **變更**] 上的 **[確定]** 。
1.   >  在 [**需要重新** 啟動] 通知中選取 [重新開機 **重新開機]** 。 選取 [ **確認全部重新開機**]。

    ![Apache Ambari 重新開機所有受影響的](./media/apache-kafka-mirroring/ambari-restart-notification.png)

### <a name="configure-kafka-to-listen-on-all-network-interfaces"></a>將 Kafka 設定為在所有網路介面上接聽。
    
1. 停留 **在 [** **服務** Kafka] 下的 [選項] 索引標籤  >  ****。 在 [ **Kafka Broker** ] 區段中 **，將 [接聽** 程式] 屬性設定為 `PLAINTEXT://0.0.0.0:9092` 。
1. 選取 [儲存]。
1. 選取 [ **重新開機**]，然後 **確認 [全部重新開機**]。

### <a name="record-broker-ip-addresses-and-zookeeper-addresses-for-primary-cluster"></a>記錄 Broker IP 位址和主要叢集的 Zookeeper 位址。

1. 在 Ambari 儀表板上選取 [ **主機** ]。
1. 記下訊息代理程式和 Zookeeper 的 IP 位址。 訊息代理程式節點的 **wn** 為主機名的前兩個字母，而 zookeeper 節點的 **zk** 是主機名稱的前兩個字母。

    ![Apache Ambari view node ip 位址](./media/apache-kafka-mirroring/view-node-ip-addresses2.png)

1. 針對第二個叢集 kafka 重複上述三個步驟 **-次要** 叢集：設定 IP 公告、設定接聽程式，並記下訊息代理程式和 Zookeeper IP 位址。

## <a name="create-topics"></a>建立主題

1. 使用 SSH 連接到 **主要** 叢集：

    ```bash
    ssh sshuser@PRIMARYCLUSTER-ssh.azurehdinsight.net
    ```

    將 **sshuser** 替換為建立叢集時所使用的 SSH 使用者名稱。 將 **PRIMARYCLUSTER** 取代為建立叢集時所使用的基底名稱。

    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

1. 使用下列命令，為主要叢集的 Apache Zookeeper 主機建立變數。 這類字串 `ZOOKEEPER_IP_ADDRESS1` 必須取代為先前記錄的實際 IP 位址，例如 `10.23.0.11` 和 `10.23.0.7` 。 如果您使用 FQDN 解析與自訂 DNS 伺服器，請遵循下列 [步驟](apache-kafka-get-started.md#getkafkainfo) 來取得訊息代理程式和 zookeeper 名稱。：

    ```bash
    # get the zookeeper hosts for the primary cluster
    export PRIMARY_ZKHOSTS='ZOOKEEPER_IP_ADDRESS1:2181, ZOOKEEPER_IP_ADDRESS2:2181, ZOOKEEPER_IP_ADDRESS3:2181'
    ```

1. 若要建立名為 `testtopic` 的主題，請使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 2 --partitions 8 --topic testtopic --zookeeper $PRIMARY_ZKHOSTS
    ```

1. 使用下列命令確認已建立主題：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $PRIMARY_ZKHOSTS
    ```

    回應包含 `testtopic`。

1. 使用下列程式來查看此 (**主要**) 叢集的 Zookeeper 主機資訊：

    ```bash
    echo $PRIMARY_ZKHOSTS
    ```

    此命令會傳回類似以下文字的資訊：

    `10.23.0.11:2181,10.23.0.7:2181,10.23.0.9:2181`

    請儲存此資訊。 它會在下一節中使用。

## <a name="configure-mirroring"></a>設定鏡像功能

1. 使用不同的 SSH 會話連接到 **次要** 叢集：

    ```bash
    ssh sshuser@SECONDARYCLUSTER-ssh.azurehdinsight.net
    ```

    將 **sshuser** 替換為建立叢集時所使用的 SSH 使用者名稱。 將 **SECONDARYCLUSTER** 取代為建立叢集時所使用的名稱。

    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

1. 檔案 `consumer.properties` 是用來設定與 **主要** 叢集的通訊。 若要建立檔案，請使用下列命令：

    ```bash
    nano consumer.properties
    ```

    使用下列文字做為 `consumer.properties` 檔案的內容：

    ```yaml
    zookeeper.connect=PRIMARY_ZKHOSTS
    group.id=mirrorgroup
    ```

    將 **PRIMARY_ZKHOSTS** 取代為來自 **主要** 叢集中的 Zookeeper IP 位址。

    此檔案描述從主要 Kafka 叢集讀取時所要使用的取用者資訊。 如需取用者組態詳細資訊，請參閱 kafka.apache.org 上的[取用者組態](https://kafka.apache.org/documentation#consumerconfigs)。

    若要儲存檔案，請使用 **Ctrl + X**、**Y** 和 **Enter** 鍵。

1. 設定要與次要叢集通訊的產生者之前，請先設定 **次要** 叢集的訊息代理程式 IP 位址的變數。 使用下列命令來建立此變數：

    ```bash
    export SECONDARY_BROKERHOSTS='BROKER_IP_ADDRESS1:9092,BROKER_IP_ADDRESS2:9092,BROKER_IP_ADDRESS2:9092'
    ```

    此命令 `echo $SECONDARY_BROKERHOSTS` 應該會傳回類似下列文字的資訊：

    `10.23.0.14:9092,10.23.0.4:9092,10.23.0.12:9092`

1. 檔案 `producer.properties` 是用來與 **次要** 叢集通訊。 若要建立檔案，請使用下列命令：

    ```bash
    nano producer.properties
    ```

    使用下列文字做為 `producer.properties` 檔案的內容：

    ```yaml
    bootstrap.servers=SECONDARY_BROKERHOSTS
    compression.type=none
    ```

    以上一個步驟中使用的 broker IP 位址取代 **SECONDARY_BROKERHOSTS** 。

    如需產生者組態詳細資訊，請參閱 kafka.apache.org 上的[者組態](https://kafka.apache.org/documentation#producerconfigs)。

1. 使用下列命令來建立具有次要叢集之 Zookeeper 主機 IP 位址的環境變數：

    ```bash
    # get the zookeeper hosts for the secondary cluster
    export SECONDARY_ZKHOSTS='ZOOKEEPER_IP_ADDRESS1:2181,ZOOKEEPER_IP_ADDRESS2:2181,ZOOKEEPER_IP_ADDRESS3:2181'
    ```

1. Kafka on HDInsight 的預設設定不允許自動建立主題。 您必須先使用下列其中一個選項，才能啟動鏡像程序：

    * 在 **次要叢集上建立主題**：此選項也可讓您設定分割區數目和複寫因數。

        您可以使用下列命令提前建立主題：

        ```bash
        /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 2 --partitions 8 --topic testtopic --zookeeper $SECONDARY_ZKHOSTS
        ```

        將 `testtopic` 取代為要建立的主題名稱。

    * **設定叢集以自動建立主題**：此選項可讓 MirrorMaker 自動建立主題，但它可能會使用與主要主題不同的分割區或複寫因數來建立它們。

        若要將次要叢集設定為自動建立主題，請執行下列步驟：

        1. 移至次要叢集的 Ambari 儀表板： `https://SECONDARYCLUSTERNAME.azurehdinsight.net` 。
        1. 按一下 [**服務**  >  **Kafka**]。 按一下 [Configs (設定)]  索引標籤。
        1. 在 [篩選] 欄位中，輸入 `auto.create` 的值。 這會篩選屬性清單並顯示 `auto.create.topics.enable` 設定。
        1. 將 `auto.create.topics.enable` 的值變更為 true，然後選取 [儲存]。 新增附註，然後再次選取 [儲存]。
        1. 依序選取 [Kafka] 服務、[重新啟動] 和 [重新啟動所有受影響的]。 出現提示時，選取 [ __確認全部重新開機__]。

        ![kafka 啟用自動建立主題](./media/apache-kafka-mirroring/kafka-enable-auto-create-topics.png)

## <a name="start-mirrormaker"></a>啟動 MirrorMaker

1. 從與 **次要** 叢集的 SSH 連線中，使用下列命令來啟動 MirrorMaker 流程：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-run-class.sh kafka.tools.MirrorMaker --consumer.config consumer.properties --producer.config producer.properties --whitelist testtopic --num.streams 4
    ```

    此範例中使用的參數：

    |參數 |描述 |
    |---|---|
    |--consumer.config|指定包含取用者屬性的檔案。 這些屬性是用來建立從 *主要* Kafka 叢集讀取的取用者。|
    |--producer.config|指定包含產生者屬性的檔案。 這些屬性是用來建立寫入 *次要* Kafka 叢集的生產者。|
    |--白名單|MirrorMaker 從主要叢集複寫至次要資料庫的主題清單。|
    |--num. 資料流程|要建立的取用者執行緒數目。|

    次要節點上的取用者現在正在等候接收訊息。

2. 從與 **主要** 叢集的 SSH 連線，使用下列命令來啟動產生者，並將訊息傳送至主題：

    ```bash
    export PRIMARY_BROKERHOSTS=BROKER_IP_ADDRESS1:9092,BROKER_IP_ADDRESS2:9092,BROKER_IP_ADDRESS2:9092
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $SOURCE_BROKERHOSTS --topic testtopic
    ```

     當您抵達有游標的空白行時，請輸入一些文字訊息。 這些訊息會傳送至 **主要** 叢集上的主題。 完成後，使用 **Ctrl + C** 結束產生者程序。

3. 從與 **次要** 叢集的 SSH 連線中，使用 **Ctrl + C** 來結束 MirrorMaker 進程。 這可能需要數秒鐘的時間才能完成程序。 若要確認訊息已複寫至次要資料庫，請使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $SECONDARY_ZKHOSTS --topic testtopic --from-beginning
    ```

    主題清單現在包含了 `testtopic` ，當 MirrorMaster 將主題從主要叢集鏡像至次要資料庫時，就會建立此清單。 從主題中取出的訊息，與您在主要叢集上輸入的訊息相同。

## <a name="delete-the-cluster"></a>選取叢集

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

本檔中的步驟已在不同的 Azure 資源群組中建立叢集。 若要刪除所建立的所有資源，您可以刪除建立的兩個資源群組： **kafka-primary-rg** 和 **kafka-secondary_rg**。 刪除資源群組會移除依照本檔建立的所有資源，包括叢集、虛擬網路和儲存體帳戶。

## <a name="next-steps"></a>後續步驟

在本文件中，您已學會如何使用 [MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) 建立 [Apache Kafka](https://kafka.apache.org/) 叢集的複本。 使用下列連結來探索使用 Kafka 的其他方式︰

* [Apache Kafka MirrorMaker 文件](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) (網址為 cwiki.apache.org)。
* [Kafka 鏡像製作者最佳做法](https://community.cloudera.com/t5/Community-Articles/Kafka-Mirror-Maker-Best-Practices/ta-p/249269)
* [開始使用 Apache Kafka on HDInsight](apache-kafka-get-started.md)
* [在 HDInsight 上搭配使用 Apache Spark 與 Apache Kafka](../hdinsight-apache-spark-with-kafka.md)
* [透過 Azure 虛擬網路連線到 Apache Kafka](apache-kafka-connect-vpn-gateway.md)
