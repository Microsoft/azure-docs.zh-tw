---
title: 使用企業安全性套件在 HDInsight 中設定 Kafka 原則 - Azure
description: 了解如何使用企業安全性套件在 Azure HDInsight 中設定 Kafka 的 Apache Ranger 原則。
services: hdinsight
ms.service: hdinsight
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.topic: tutorial
ms.date: 09/24/2018
ms.openlocfilehash: 1a8f04f39568816252175fc9e0893f1ab3e2cdc6
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224812"
---
# <a name="tutorial-configure-kafka-policies-in-hdinsight-with-enterprise-security-package-preview"></a>教學課程：使用企業安全性套件在 HDInsight 中設定 Kafka 原則 (預覽)

了解如何對企業安全性套件 (ESP) Kafka 叢集設定 Apache Ranger 原則。 ESP 叢集連線到網域，讓使用者使用網域認證進行驗證。 在此教學課程中，您會建立兩個 Ranger 原則來限制 `sales*` 和 `marketingspend` 主題的存取權。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立網域使用者
> * 建立 Ranger 原則
> * 在 Kafka 叢集中建立主題
> * 測試 Ranger 原則

## <a name="before-you-begin"></a>開始之前

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/)。

* 登入 [Azure 入口網站](https://portal.azure.com/)。

* 使用企業安全性套件建立 [HDInsight Kafka 叢集](https://docs.microsoft.com/azure/hdinsight/domain-joined/apache-domain-joined-run-hive)。

## <a name="connect-to-apache-ranger-admin-ui"></a>連線到 Apache Ranger 系統管理 UI

1. 從瀏覽器中，使用 URL `https://<ClusterName>.azurehdinsight.net/Ranger/` 連線到 Ranger 管理員使用者介面。 請記得將 `<ClusterName>` 變更為 Kafka 叢集的名稱。

    > [!NOTE] 
    > Ranger 認證與 Hadoop 叢集認證並非相同。 若要避免瀏覽器使用快取的 Hadoop 認證，請使用新的 InPrivate 瀏覽器視窗連線至 Ranger 管理員 UI。

2. 使用您的 Azure Active Directory (AD) 管理員認證登入。 Azure AD 管理員認證與 HDInsight 叢集認證或 Linux HDInsight 節點 SSH 認證並非相同。

   ![Apache Ranger 管理員 UI](./media/apache-domain-joined-run-kafka/apache-ranger-admin-login.png)

## <a name="create-domain-users"></a>建立網域使用者

請參閱[使用企業安全性套件建立 HDInsight 叢集](https://docs.microsoft.com/azure/hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds#create-a-domain-joined-hdinsight-cluster)，以了解如何建立 **sales_user** 和 **marketing_user** 網域使用者。 在生產情節中，網域使用者來自 Active Directory 租用戶。

## <a name="create-ranger-policy"></a>建立 Ranger 原則 

為 **sales_user** 和 **marketing_user** 建立 Ranger 原則。

1. 開啟 **Ranger 管理員 UI**。

2. 按一下 [Kafka] 底下的 [\<ClusterName>_kafka]。 可能會列出其中一個預先設定的原則。

3. 按一下 [新增原則]，並輸入下列值︰

   |**設定**  |**建議的值**  |
   |---------|---------|
   |原則名稱  |  hdi 銷售* 原則   |
   |話題   |  銷售* |
   |選取使用者  |  sales_user1 |
   |權限  | 發佈、取用、建立 |

   主題名稱可以包含下列萬用字元：

   * 「*」表示出現零次以上的字元。
   * 「?」表示單一字元。

   ![Apache Ranger 管理員 UI 建立原則](./media/apache-domain-joined-run-kafka/apache-ranger-admin-create-policy.png)   

   >[!NOTE] 
   >如果 [選取使用者] 未自動填入網域使用者，請稍候 Ranger 與 Azure AD 同步處理。

4. 按一下 [新增] 以儲存規則。

5. 按一下 [新增原則]，然後輸入下列值︰

   |**設定**  |**建議的值**  |
   |---------|---------|
   |原則名稱  |  hdi 行銷原則   |
   |話題   |  marketingspend |
   |選取使用者  |  marketing_user1 |
   |權限  | 發佈、取用、建立 |

   ![Apache Ranger 管理員 UI 建立原則](./media/apache-domain-joined-run-kafka/apache-ranger-admin-create-policy-2.png)  

6. 按一下 [新增] 以儲存規則。

## <a name="create-topics-in-a-kafka-cluster-with-esp"></a>在 Kafka 叢集中使用 ESP 建立主題

若要建立 **salesevents** 和 **marketingspend** 這兩個主題：

1. 下列命令可用來開啟與叢集的 SSH 連線：

   ```bash
   ssh SSHUSER@CLUSTERNAME-ssh.azurehdinsight.net
   ```

   將 `SSHUSER` 取代為叢集的 SSH 使用者，並將 `CLUSTERNAME` 取代為叢集的名稱。 出現提示時，請輸入 SSH 使用者帳戶的密碼。 如需搭配 HDInsight 使用 `scp` 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix)。

   在生產情節中，在叢集建立期間設定的網域使用者可以將 SSH 連線到叢集。

2. 下列命令可用來將叢集名稱儲存到變數，並安裝 JSON 剖析公用程式 `jq`。 出現提示時，請輸入 Kafka 叢集名稱。

   ```bash
   sudo apt -y install jq
   read -p 'Enter your Kafka cluster name:' CLUSTERNAME
   ```

3. 下列命令可用來取得 Kafka 訊息代理程式主機和 Zookeeper 主機。 出現提示時，輸入叢集管理員帳戶的密碼。

   ```bash
   export KAFKAZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`; \
   
   export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`; \
   ```

4. 執行下列命令： 

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/config/kafka_client_jaas.conf kafka-producer-consumer.jar create salesevents $KAFKABROKERS
   
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/config/kafka_client_jaas.conf kafka-producer-consumer.jar create marketingspend $KAFKABROKERS
   ```

   >[!NOTE] 
   >只有 Kafka 服務的程序所有人 (例如根使用者) 才能寫入 Zookeeper znode `/config/topics`。 非特殊權限使用者建立主題時，不會強制執行Ranger 原則。 這是因為 `kafka-topics.sh` 指令碼會直接與 Zookeeper 進行通訊來建立主題。 雖然訊息代理程式端上的監看員會監看並據此建立主題，但是項目仍會加入到 Zookeeper 節點。 無法透過 Ranger 外掛程式進行授權，而且會透過 Kafka 訊息代理程式使用 `sudo` 執行上述命令。


## <a name="test-the-ranger-policies"></a>測試 Ranger 原則

根據設定的 Ranger 原則，**sales_user** 可產生/取用主題 **salesevents**，但是無法產生/取用主題 **marketingspend**。 相反地，**marketing_user** 可產生/取用主題 **marketingspend**，而無法產生/取用主題 **salesevents**。

1. 對於叢集開啟新的 SSH 連線。 下列命令可用來以 **sales_user1** 身分登入：

   ```bash
   ssh sales_user1@CLUSTERNAME-ssh.azurehdinsight.net
   ```

2. 執行以下命令：

   ```bash
   export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/config/kafka_client_jaas.conf"
   ```

3. 上一節中的訊息代理程式和 Zookeeper 名稱可用來設定下列環境變數：

   ```bash
   export KAFKABROKERS=<brokerlist>:9092 
   ```

   範例： `export KAFKABROKERS=wn0-khdicl.contoso.com:9092,wn1-khdicl.contoso.com:9092`

   ```bash
   export KAFKAZKHOSTS=<zklist>:2181
   ```

   範例： `export KAFKAZKHOSTS=zk1-khdicl.contoso.com:2181,zk2-khdicl.contoso.com:2181`

4. 確認 **sales_user1** 可產生至主題 **salesevents**。
   
   下列命令可用來啟動主題 **salesevents** 的主控台產生者：

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic salesevents --security-protocol SASL_PLAINTEXT
   ```

   然後，在主控台上輸入一些訊息。 按下 **Ctrl + C** 可結束主控台產生者。

5. 下列命令可用來取用主題 **salesevents**：

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --zookeeper $KAFKAZKHOSTS --topic salesevents --security-protocol PLAINTEXTSASL --from-beginning
   ```
 
6. 確認您在上一個步驟中輸入的訊息會出現，而且 **sales_user1** 不會產生至主題 **marketingspend**。

   從如上所示的同一個 SSH 視窗，執行下列命令來產生主題 **marketingspend**：

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic marketingspend --security-protocol SASL_PLAINTEXT
   ```

   發生授權錯誤，可予以忽略。 

7. 請注意，無法從主題 **salesevents** 取用 **marketing_user1**。

   重複執行上述步驟 1-3，不過這次是以 **marketing_user1** 身分執行。

   下列命令可用來取用主題 **salesevents**：

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --zookeeper $KAFKAZKHOSTS --topic marketingspend --security-protocol PLAINTEXTSASL --from-beginning
   ```

   先前的訊息不會出現。

8. 從 Ranger UI 檢視稽核存取事件。

   ![Ranger UI 原則稽核](./media/apache-domain-joined-run-kafka/apache-ranger-admin-audit.png)

## <a name="next-steps"></a>後續步驟

* [自備 Kafka 的金鑰](https://docs.microsoft.com/azure/hdinsight/kafka/apache-kafka-byok)
* [使用企業安全性套件維護 Hadoop 安全性的簡介](https://docs.microsoft.com/azure/hdinsight/domain-joined/apache-domain-joined-introduction)
