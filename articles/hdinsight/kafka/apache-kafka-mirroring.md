---
title: 鏡像 Apache Kafka 主題 - Azure HDInsight
description: 了解如何使用 Apache Kafka 的鏡像功能，藉由將主題鏡像處理至次要叢集來維護 HDInsight 叢集上的 Kafka 複本。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/01/2018
ms.openlocfilehash: 9f8204eb29566aaae9170c21b8dde33aa3178384
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042698"
---
# <a name="use-mirrormaker-to-replicate-apache-kafka-topics-with-kafka-on-hdinsight"></a>使用 MirrorMaker，透過 HDInsight 上的 Kafka 來複寫 Apache Kafka 主題

了解如何使用 Apache Kafka 的鏡像功能，將主題複寫至次要叢集。 鏡像功能可以當作連續程序執行，或間歇地做為在叢集間移轉資料的方法。

在此範例中，會使用鏡像來複寫兩個 HDInsight 叢集之間的主題。 這兩個叢集是位於相同區域中的 Azure 虛擬網路。

> [!WARNING]
> 但不能將鏡像功能視為達成容錯的方法。 主題中的項目位移在來源與目的地叢集之間有所不同，所以用戶端無法交替使用這兩者。
>
> 如果您很擔心容錯，您應該為叢集內的主題設定複寫。 如需詳細資訊，請參閱[開始使用 Kafka on HDInsight](apache-kafka-get-started.md)。

## <a name="how-kafka-mirroring-works"></a>Kafka 鏡像的運作方式

鏡像功能的運作方式是使用 MirrorMaker 工具 (Apache Kafka 的一部分)，取用來源叢集上主題中的記錄，然後在目的地叢集上建立本機複本。 MirrorMaker 會使用一個 (或多個) *取用者*從來源叢集讀取資料，以及使用一個*產生者*來將資料寫入本機 (目的地) 叢集。

下圖說明鏡像程序：

![鏡像程序圖表](./media/apache-kafka-mirroring/kafka-mirroring.png)

Apache Kafka on HDInsight 不提供透過公用網際網路存取 Kafka 服務。 Kafka 產生者和取用者必須與 Kafka 叢集中之節點位於相同的 Azure 虛擬網路。 例如，Kafka 來源和目的地叢集均位於 Azure 虛擬網路中。 下圖顯示叢集之間的通訊流動方式︰

![Azure 虛擬網路中的來源和目的地 Kafka 叢集圖表](./media/apache-kafka-mirroring/spark-kafka-vnet.png)

來源與目的地叢集的節點與磁碟分割數目可能有所不同，且主題中的位移也會不同。 鏡像功能會維護用於資料分割的金鑰值，因此會根據每個金鑰保留記錄順序。

### <a name="mirroring-across-network-boundaries"></a>跨網路界限鏡像

如果您需要在不同網路中的 Kafka 叢集之間進行鏡像處理，有下列額外考量︰

* **閘道**：網路必須能夠在 TCPIP 層級進行通訊。

* **名稱解析**︰每個網路中的 Kafka 叢集必須能夠使用主機名稱彼此連接。 這可能會要求每個網路中的網域名稱系統 (DNS) 伺服器設定成將要求轉送到其他網路。

    建立 Azure 虛擬網路 (而不是使用網路提供的自動 DNS) 時，您必須指定自訂 DNS 伺服器和伺服器的 IP 位址。 建立虛擬網路之後，您就必須建立使用該 IP 位址的 Azure 虛擬機器，然後在其上安裝和設定 DNS 軟體。

    > [!WARNING]
    > 先建立和設定自訂 DNS 伺服器，然後再將 HDInsight 安裝到虛擬網路中。 HDInsight 不需要進行其他設定，即可使用針對虛擬網路設定的 DNS 伺服器。

如需有關如何連接兩個 Azure 虛擬網路的詳細資訊，請參閱[設定 VNet 對 VNet 連線](../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)。

## <a name="create-kafka-clusters"></a>建立 Kafka 叢集

雖然您可以手動建立 Azure 虛擬網路和 Kafka 叢集，但使用 Azure Resource Manager 範本更輕鬆。 使用下列步驟將 Azure 虛擬網路和兩個 Kafka 叢集部署到 Azure 訂用帳戶。

1. 使用以下按鈕，在 Azure 入口網站中登入 Azure 並開啟範本。
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Farmtemplates%2Fcreate-linux-based-kafka-mirror-cluster-in-vnet-v2.1.json" target="_blank"><img src="./media/apache-kafka-mirroring/deploy-to-azure.png" alt="Deploy to Azure"></a>
   
    Azure Resource Manager 範本位於 **https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-kafka-mirror-cluster-in-vnet-v2.1.json**。

    > [!WARNING]
    > 若要保證 Kafka 在 HDInsight 上的可用性，您的叢集必須包含至少三個背景工作角色節點。 此範本會建立包含三個背景工作角色節點的 Kafka 叢集。

2. 使用下列資訊來填入 [自訂部署] 刀鋒視窗上的項目︰
    
    ![HDInsight 自訂部署](./media/apache-kafka-mirroring/parameters.png)
    
    * **資源群組**：建立群組或選取現有的群組。 此群組包含 HDInsight 叢集。

    * **位置**：選取在地理上靠近您的位置。
     
    * **基底叢集名稱**︰此值會做為 Kafka 叢集的基底名稱。 例如，輸入 **hdi** 可建立名為 **source-hdi** 和 **dest-hdi** 的叢集。

    * **叢集登入使用者名稱**：來源和目的地 Kafka 叢集的系統管理員使用者名稱。

    * **叢集登入密碼**：來源和目的地 Kafka 叢集的系統管理員使用者密碼。

    * **SSH 使用者名稱**：建立來源和目的地 Kafka 叢集的 SSH 使用者。

    * **SSH 密碼**：來源和目的地 Kafka 叢集的 SSH 使用者密碼。

3. 讀取**條款及條件**，然後選取 [我同意上方所述的條款及條件]。

4. 最後，核取 [釘選到儀表板]，然後選取 [購買]。 大約需要 20 分鐘的時間來建立叢集。

> [!IMPORTANT]
> HDInsight 叢集的名稱是 **source-BASENAME** 和 **dest-BASENAME**，其中 BASENAME 是您提供給範本的名稱。 連接到叢集時，您會在稍後步驟中使用這些名稱。

## <a name="create-topics"></a>建立主題

1. 使用 SSH 連接到**來源**叢集：

    ```bash
    ssh sshuser@source-BASENAME-ssh.azurehdinsight.net
    ```

    將 **sshuser** 替換為建立叢集時所使用的 SSH 使用者名稱。 將 **BASENAME** 替換為建立叢集時使用的基底名稱。

    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 使用下列命令為來源叢集尋找 Zookeeper 主機：

    ```bash
    # Install jq if it is not installed
    sudo apt -y install jq
    # get the zookeeper hosts for the source cluster
    export SOURCE_ZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

    將 `$CLUSTERNAME` 取代為來源叢集的名稱。 出現提示時，輸入叢集登入 (admin) 帳戶的密碼。

3. 若要建立名為 `testtopic` 的主題，請使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 2 --partitions 8 --topic testtopic --zookeeper $SOURCE_ZKHOSTS
    ```

3. 使用下列命令確認已建立主題：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $SOURCE_ZKHOSTS
    ```

    回應包含 `testtopic`。

4. 使用下列命令來檢視這個 (**來源**) 叢集的 Zookeeper 主機資訊︰

    ```bash
    echo $SOURCE_ZKHOSTS
    ```

    此命令會傳回類似以下文字的資訊：

    `zk0-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:2181,zk1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:2181`

    請儲存此資訊。 此資訊使用於下一節。

## <a name="configure-mirroring"></a>設定鏡像功能

1. 使用不同的 SSH 工作階段連接到**目的地**叢集：

    ```bash
    ssh sshuser@dest-BASENAME-ssh.azurehdinsight.net
    ```

    將 **sshuser** 替換為建立叢集時所使用的 SSH 使用者名稱。 將 **BASENAME** 替換為建立叢集時使用的基底名稱。

    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. `consumer.properties` 檔案是用來設定與**來源**叢集通訊。 若要建立檔案，請使用下列命令：

    ```bash
    nano consumer.properties
    ```

    使用下列文字做為 `consumer.properties` 檔案的內容：

    ```yaml
    zookeeper.connect=SOURCE_ZKHOSTS
    group.id=mirrorgroup
    ```

    以**來源**叢集中的 Zookeeper 主機資訊取代 **SOURCE_ZKHOSTS**。

    此檔案描述從來源 Kafka 叢集讀取資料時所要使用的取用者資訊。 如需取用者組態詳細資訊，請參閱 kafka.apache.org 上的[取用者組態](https://kafka.apache.org/documentation#consumerconfigs)。

    若要儲存檔案，請使用 **Ctrl + X**、**Y** 和 **Enter** 鍵。

3. 在設定可與目的地叢集通訊的產生者之前，您必須尋找**目的地**叢集的訊息代理程式主機。 請使用下列命令來擷取此資訊：

    ```bash
    sudo apt -y install jq
    DEST_BROKERHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    echo $DEST_BROKERHOSTS
    ```

    將 `$CLUSTERNAME` 取代為目的地叢集的名稱。 出現提示時，輸入叢集登入 (admin) 帳戶的密碼。

    `echo` 命令會傳回類似以下文字的資訊：

        wn0-dest.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn1-dest.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092

4. `producer.properties` 檔案是用來與__目的地__叢集通訊。 若要建立檔案，請使用下列命令：

    ```bash
    nano producer.properties
    ```

    使用下列文字做為 `producer.properties` 檔案的內容：

    ```yaml
    bootstrap.servers=DEST_BROKERS
    compression.type=none
    ```

    以上一個步驟中的訊息代理程式資訊取代 **DEST_BROKERS**。

    如需產生者組態詳細資訊，請參閱 kafka.apache.org 上的[者組態](https://kafka.apache.org/documentation#producerconfigs)。

5. 使用下列命令為目的地叢集尋找 Zookeeper 主機：

    ```bash
    # Install jq if it is not installed
    sudo apt -y install jq
    # get the zookeeper hosts for the source cluster
    export DEST_ZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

    將 `$CLUSTERNAME` 取代為目的地叢集的名稱。 出現提示時，輸入叢集登入 (admin) 帳戶的密碼。

7. HDInsight 上 Kafka 的預設組態不允許自動建立主題。 您必須先使用下列其中一個選項，才能啟動鏡像程序：

    * **在目的地叢集上建立主題**：這個選項也可讓您設定分割數目和複寫因子。

        您可以使用下列命令提前建立主題：

        ```bash
        /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 2 --partitions 8 --topic testtopic --zookeeper $DEST_ZKHOSTS
        ```

        將 `testtopic` 取代為要建立的主題名稱。

    * **設定可供自動建立主題的叢集**：此選項可讓 MirrorMaker 自動建立主題，但它可能使用與來源主題不同的分割數目或複寫因子建立主題。

        若要設定目的地叢集來自動建立主題，請執行下列步驟：

        1. 從 [Azure 入口網站](https://portal.azure.com)，選取 Kafka 目的地叢集。
        2. 從叢集概觀，選取 [叢集儀表板]。 然後選取 [HDInsight 叢集儀表板]。 出現提示時，使用叢集的登入 (系統管理員) 認證進行驗證。
        3. 從頁面左邊的清單中，選取 [Kafka] 服務。
        4. 選取頁面中間的 [設定]。
        5. 在 [篩選] 欄位中，輸入 `auto.create` 的值。 這會篩選屬性清單並顯示 `auto.create.topics.enable` 設定。
        6. 將 `auto.create.topics.enable` 的值變更為 true，然後選取 [儲存]。 新增附註，然後再次選取 [儲存]。
        7. 依序選取 [Kafka] 服務、[重新啟動] 和 [重新啟動所有受影響的]。 出現提示時，選取 [確認全部重新啟動]。

## <a name="start-mirrormaker"></a>啟動 MirrorMaker

1. 在連往**目的地**叢集的 SSH 連線中，使用下列命令來啟動 MirrorMaker 程序：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-run-class.sh kafka.tools.MirrorMaker --consumer.config consumer.properties --producer.config producer.properties --whitelist testtopic --num.streams 4
    ```

    此範例中使用的參數：

    * **--consumer.config**︰指定包含取用者屬性的檔案。 這些屬性用來建立可從*來源* Kafka 叢集讀取資料的取用者。

    * **--producer.config**︰指定包含產生者屬性的檔案。 這些屬性用來建立可寫入*目的地* Kafka 叢集的產生者。

    * **--whitelist**：MirrorMaker 從來源叢集複寫至目的地的主題清單。

    * **--num.streams**︰要建立的取用者執行緒數目。

 啟動時，MirrorMaker 會傳回類似以下文字的資訊：

    ```json
    {metadata.broker.list=wn1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn0-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092, request.timeout.ms=30000, client.id=mirror-group-3, security.protocol=PLAINTEXT}{metadata.broker.list=wn1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn0-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092, request.timeout.ms=30000, client.id=mirror-group-0, security.protocol=PLAINTEXT}
    metadata.broker.list=wn1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn0-kafka.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092, request.timeout.ms=30000, client.id=mirror-group-2, security.protocol=PLAINTEXT}
    metadata.broker.list=wn1-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092,wn0-source.aazwc2onlofevkbof0cuixrp5h.gx.internal.cloudapp.net:9092, request.timeout.ms=30000, client.id=mirror-group-1, security.protocol=PLAINTEXT}
    ```

2. 從連往**來源**叢集的 SSH 連線中，使用下列命令來啟動產生者，然後傳送訊息到主題：

    ```bash
    SOURCE_BROKERHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $SOURCE_BROKERHOSTS --topic testtopic
    ```

    將 `$CLUSTERNAME` 取代為來源叢集的名稱。 出現提示時，輸入叢集登入 (admin) 帳戶的密碼。

     當您抵達有游標的空白行時，請輸入一些文字訊息。 訊息會傳送到**來源**叢集上的主題。 完成後，使用 **Ctrl + C** 結束產生者程序。

3. 在連往**目的地**叢集的 SSH 連線中，使用 **Ctrl + C** 來結束 MirrorMaker 程序。 這可能需要數秒鐘的時間才能完成程序。 若要確認訊息已複寫到目的地，請使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --zookeeper $DEST_ZKHOSTS --topic testtopic --from-beginning
    ```

    將 `$CLUSTERNAME` 取代為目的地叢集的名稱。 出現提示時，輸入叢集登入 (admin) 帳戶的密碼。

    主題清單現在包含 `testtopic`，它是在 MirrorMaster 將主題從來源叢集鏡射至目的地時所建立。 從主題中擷取的訊息與在來源叢集上所輸入的相同。

## <a name="delete-the-cluster"></a>刪除叢集

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

因為本文件中的步驟會在相同的 Azure 資源群組中建立兩個叢集，您可以在 Azure 入口網站中刪除資源群組。 刪除資源群組，即可移除依循本文件建立的所有資源、Azure 虛擬網路，以及叢集所使用的儲存體帳戶。

## <a name="next-steps"></a>後續步驟

在本文件中，您已學會如何使用 MirrorMaker 建立 Kafka 叢集的複本。 使用下列連結來探索使用 Kafka 的其他方式︰

* [Apache Kafka MirrorMaker 文件](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) (網址為 cwiki.apache.org)。
* [開始使用 Apache Kafka on HDInsight](apache-kafka-get-started.md)
* [使用 Apache Spark 搭配 Kafka on HDInsight](../hdinsight-apache-spark-with-kafka.md)
* [使用 Apache Storm 搭配 HDInsight 上的 Kafka](../hdinsight-apache-storm-with-kafka.md)
* [透過 Azure 虛擬網路連線到 Kafka](apache-kafka-connect-vpn-gateway.md)
