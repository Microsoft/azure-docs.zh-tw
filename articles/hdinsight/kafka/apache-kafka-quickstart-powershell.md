---
title: 開始使用 Apache Kafka - Azure HDInsight 快速入門
description: 在本快速入門中，您會了解如何使用 Azure PowerShell，在 Azure HDInsight 上建立 Apache Kafka 叢集。 您也會了解 Kafka 主題、訂閱者與取用者。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: mvc,hdinsightactive
ms.topic: quickstart
ms.date: 04/16/2018
ms.openlocfilehash: 756733ec98912685af438861d54db0d18e15a947
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046445"
---
# <a name="quickstart-create-a-kafka-on-hdinsight-cluster"></a>快速入門：建立 HDInsight 上的 Kafka 叢集

Kafka 是一個開放原始碼的分散式串流平台。 它通常會用來作為訊息代理程式，因為可以提供類似「發佈-訂閱」訊息佇列的功能。 

在本快速入門中，您會了解如何使用 Azure PowerShell 來建立 [Apache Kafka](https://kafka.apache.org) \(英文\) 叢集。 您也會了解如何使用內含的公用程式，使用 Kafka 來傳送和接收訊息。

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

> [!IMPORTANT]
> Kafka API 只能由同一個虛擬網路中的資源來存取。 在本快速入門中，您會使用 SSH 直接存取叢集。 若要將其他服務、網路或虛擬機器連線到 Kafka，您必須先建立虛擬網路，然後建立網路中的資源。
>
> 如需詳細資訊，請參閱[使用虛擬網路連線到 Kafka](apache-kafka-connect-vpn-gateway.md) 文件。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

* Azure PowerShell。 如需詳細資訊，請參閱[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps) \(英文\) 文件。

* SSH 用戶端。 本文件中的步驟會使用 SSH 來連線到叢集。

    Linux、Unix 及 macOS 系統上預設會提供 `ssh` 命令。 在 Windows 10 上，使用下列其中一種方法來安裝 `ssh` 命令：

    * 使用 [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/quickstart)。 Cloud Shell 會提供 `ssh` 命令，並可設定為使用 Bash 或 PowerShell 作為殼層環境。

    * 安裝[適用於 Linux 的 Windows 子系統](https://docs.microsoft.com/windows/wsl/install-win10) \(英文\)。 可透過 Microsoft Store 取得的 Linux 發行版本會提供 `ssh` 命令。

    > [!IMPORTANT]
    > 本文件中的步驟假設您使用上述其中一個 SSH 用戶端。 如果您使用不同的 SSH 用戶端並遇到問題，請參閱您的 SSH 用戶端文件。
    >
    > 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

## <a name="log-in-to-azure"></a>登入 Azure

使用 `Login-AzureRmAccount` Cmdlet 登入 Azure 訂用帳戶並遵循畫面上的指示。

```powershell
Login-AzureRmAccount
```

## <a name="create-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 下列範例會提示您提供名稱和位置，然後建立新的資源群組：

```powershell
$resourceGroup = Read-Input -Prompt "Enter the resource group name"
$location = Read-Input -Prompt "Enter the Azure region to use"

New-AzureRmResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶

儘管 HDInsight 上的 Kafka 會使用 Azure 受控磁碟來儲存 Kafka 資料，但叢集也會使用 Azure 儲存體來儲存像是記錄等資訊。 使用 [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/new-azurermstorageaccount) 來建立新的儲存體帳戶。

```powershell
$storageName = Read-Host -Prompt "Enter the storage account name"

New-AzureRmStorageAccount `
        -ResourceGroupName $resourceGroup `
        -Name $storageName `
        -Type Standard_LRS `
        -Location $location
```

HDInsight 會將儲存體帳戶的資料儲存於 Blob 容器中。 使用 [New-AzureStorageContainer](/powershell/module/Azure.Storage/New-AzureStorageContainer) 來建立新的容器。

```powershell
$containerName = Read-Host -Prompt "Enter the container name"

$storageKey = (Get-AzureRmStorageAccountKey `
                -ResourceGroupName $resourceGroup `
                -Name $storageName)[0].Value
$storageContext = New-AzureStorageContext `
                    -StorageAccountName $storageName `
                    -StorageAccountKey $storageKey
New-AzureStorageContainer -Name $containerName -Context $storageContext 
```

## <a name="create-a-kafka-cluster"></a>建立 Kafka 叢集

使用 [New-AzureRmHDInsightCluster](/powershell/module/AzureRM.HDInsight/New-AzureRmHDInsightCluster) 來建立 HDInsight 叢集上的 Kafka。

```powershell
# Create a Kafka 1.0 cluster
$clusterName = Read-Host -Prompt "Enter the name of the Kafka cluster"
$httpCredential = Get-Credential -Message "Enter the cluster login credentials" `
    -UserName "admin"
$sshCredentials = Get-Credential -Message "Enter the SSH user credentials"

$numberOfWorkerNodes = "4"
$clusterVersion = "3.6"
$clusterType="Kafka"
$clusterOS="Linux"
$disksPerNode=2

$kafkaConfig = New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"
$kafkaConfig.Add("kafka", "1.0")

New-AzureRmHDInsightCluster `
        -ResourceGroupName $resourceGroup `
        -ClusterName $clusterName `
        -Location $location `
        -ClusterSizeInNodes $numberOfWorkerNodes `
        -ClusterType $clusterType `
        -OSType $clusterOS `
        -Version $clusterVersion `
        -ComponentVersion $kafkaConfig `
        -HttpCredential $httpCredential `
        -DefaultStorageAccountName "$storageName.blob.core.windows.net" `
        -DefaultStorageAccountKey $storageKey `
        -DefaultStorageContainer $clusterName `
        -SshCredential $sshCredentials `
        -DisksPerWorkerNode $disksPerNode
```

> [!WARNING]
> 建立 HDInsight 叢集最多可能需要 20 分鐘。

> [!TIP]
> `-DisksPerWorkerNode` 參數會設定 HDInsight 上 Kafka 的延展性。 HDInsight 上的 Kafka 會在叢集中使用虛擬機器的本機磁碟來儲存資料。 Kafka 的 I/O 非常大量，因此會使用 [Azure 受控磁碟](../../virtual-machines/windows/managed-disks-overview.md)來提供高輸送量，並為每個節點提供更多儲存空間。 
>
> 受控磁碟的類型可以是__標準__ (HDD) 或__進階__ (SSD)。 磁碟類型取決於背景工作節點 (Kafka 代理程式) 所使用的 VM 大小。 進階磁碟會自動與 DS 和 GS 系列的 VM 搭配使用。 所有其他的 VM 類型是使用標準磁碟。 您可以使用 `-WorkerNodeSize` 參數來設定 VM 類型。 如需參數的詳細資訊，請參閱 [New-AzureRmHDInsightCluster](/powershell/module/AzureRM.HDInsight/New-AzureRmHDInsightCluster) 文件。


> [!IMPORTANT]
> 如果您規劃使用 32 個以上的背景工作節點 (在建立叢集時或在建立後調整叢集規模時)，則必須使用 `-HeadNodeSize` 參數來指定具有至少 8 個核心和 14 GB RAM 的 VM 大小。
>
> 如需節點大小和相關成本的詳細資訊，請參閱 [HDInsight 定價](https://azure.microsoft.com/pricing/details/hdinsight/)。

## <a name="connect-to-the-cluster"></a>連接到叢集

1. 若要連線到 Kafka 叢集的主要前端節點，請使用下列命令。 將 `sshuser` 取代為 SSH 使用者名稱。 將 `mykafka` 取代為您的 Kafka 叢集名稱。

    ```bash
    ssh sshuser@mykafka-ssh.azurehdinsight.net
    ```

2. 當您初次連線到叢集時，您的 SSH 用戶端可能會顯示警告，指出無法確認主機的真確性。 在系統提示時，輸入 __yes__，然後按 __Enter__ 鍵，以將主機新增至 SSH 用戶端信任的伺服器清單。

3. 出現提示時，請輸入 SSH 使用者的密碼。

連線之後，您會看到類似下列文字的資訊：

```text
Authorized uses only. All activity may be monitored and reported.
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-1011-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

83 packages can be updated.
37 updates are security updates.



Welcome to Kafka on HDInsight.

Last login: Thu Mar 29 13:25:27 2018 from 108.252.109.241
ssuhuser@hn0-mykafk:~$
```

## <a id="getkafkainfo"></a>取得 Zookeeper 和訊息代理程式主機資訊

使用 Kafka 時，您必須知道 *Zookeeper* 主機和「訊息代理程式」主機。 這些主機可搭配 Kafka API 以及 Kafka 隨附的許多公用程式使用。

在本節中，您會從叢集上的 Ambari REST API 取得主機資訊。

1. 從連往叢集的 SSH 連線中，使用下列命令來安裝 `jq` 公用程式。 此公用程式可用來剖析 JSON 文件，而且在擷取主機資訊時很有用：
   
    ```bash
    sudo apt -y install jq
    ```

2. 若要將環境變數設為叢集名稱，請使用下列命令：

    ```bash
    read -p "Enter the Kafka on HDInsight cluster name: " CLUSTERNAME
    ```

    出現提示時，輸入 Kafka 叢集的名稱。

3. 若要使用 Zookeeper 主機資訊設定環境變數，請使用下列命令：

    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

    出現提示時，輸入叢集登入帳戶 (不是 SSH 帳戶) 的密碼。

    > [!NOTE]
    > 此命令會擷取所有的 Zookeeper 主機，然後只傳回前兩個項目。 這是因為考量備援之故，以防某一部主機無法連線。

4. 若要確認是否已正確設定環境變數，請使用下列命令：

    ```bash
     echo '$KAFKAZKHOSTS='$KAFKAZKHOSTS
    ```

    此命令會傳回類似以下文字的資訊：

    `zk0-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181,zk2-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181`

5. 若要使用 Kafka 訊息代理程式主機資訊來設定環境變數，請使用下列命令：

    ```bash
    export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    ```

    出現提示時，輸入叢集登入帳戶 (不是 SSH 帳戶) 的密碼。

6. 若要確認是否已正確設定環境變數，請使用下列命令：

    ```bash   
    echo '$KAFKABROKERS='$KAFKABROKERS
    ```

    此命令會傳回類似以下文字的資訊：
   
    `wn1-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092,wn0-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092`

## <a name="manage-kafka-topics"></a>管理 Kafka 主題

Kafka 會將資料串流儲存於「主題」中。 您可以使用 `kafka-topics.sh` 公用程式來管理主題。

* **若要建立主題**，請在 SSH 連線中使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
    ```

    此命令會使用 `$KAFKAZKHOSTS` 中儲存的主機資訊連線到 Zookeeper。 然後建立名為 **test** 的 Kafka 主題。 

    * 此主題中所儲存的資料會分割到八個分割區。

    * 每個分割區都會在叢集中的三個背景工作節點之間進行複寫。

        > [!IMPORTANT]
        > 如果您已在 Azure 區域中建立叢集來提供三個容錯網域，請使用複寫因子 3。 否則，使用複寫因子 4。
        
        在具有三個容錯網域的區域中，複寫因子 3 可讓複本散佈於容錯網域中。 在具有兩個容錯網域的區域中，複寫因子 4 會在網域中平均散佈複本。
        
        如需區域中的容錯網域數目的資訊，請參閱 [Linux 虛擬機器的可用性](../../virtual-machines/windows/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set)文件。

        > [!IMPORTANT] 
        > Kafka 不知道 Azure 容錯網域。 為主題建立副本時，可能無法正確發散副本以實現高可用性。

        若要確保高可用性，請使用[Kafka 分割重新平衡工具](https://github.com/hdinsight/hdinsight-kafka-tools)。 您必須從連往 Kafka 叢集前端節點的 SSH 連線來執行此工具。

        針對 Kafka 資料的最高可用性，您應該在下列情況中重新平衡主題的分割區複本：

        * 建立新主題或磁碟分割時

        * 相應增加叢集時

* **若要列出主題**，請使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
    ```

    此命令會列出可在 Kafka 叢集上使用的主題。

* **若要刪除主題**，請使用下列命令：

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic topicname --zookeeper $KAFKAZKHOSTS
    ```

    此命令會刪除名為 `topicname` 的主題。

    > [!WARNING]
    > 如果您刪除先前建立的 `test` 主題，則必須加以重新建立。 本文件稍後的步驟會用到此主題。

如需 `kafka-topics.sh` 公用程式可用命令的詳細資訊，請使用下列命令：

```bash
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh
```

## <a name="produce-and-consume-records"></a>產生和取用記錄

Kafka 會在主題中儲存「記錄」。 記錄是由「產生者」產生，並由「取用者」取用。 產生者與取用者會與「Kafka 訊息代理程式」服務進行通訊。 HDInsight 叢集中的每個背景工作節點都是一部 Kafka 訊息代理程式主機。

若要將記錄儲存至您稍早建立的 test 主題，然後利用取用者進行讀取，請使用下列步驟：

1. 若要將記錄寫入主題，請從 SSH 連線使用 `kafka-console-producer.sh` 公用程式：
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic test
    ```
   
    在此命令之後，您會抵達空白行。

2. 在空白行中輸入文字訊息並按一下 enter 鍵。 如此輸入幾個訊息，然後使用 **Ctrl + C** 返回一般提示。 每一行都會以個別記錄傳送至 Kafka 主題。

3. 若要從主題讀取記錄，請從 SSH 連線使用 `kafka-console-consumer.sh` 公用程式：
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic test --from-beginning
    ```
   
    此命令會擷取主題中的記錄並加以顯示。 使用 `--from-beginning` 告知取用者從串流的開頭開始，所以會擷取所有的記錄。

    > [!NOTE]
    > 如果您使用舊版 Kafka，請以 `--zookeeper $KAFKAZKHOSTS` 取代 `--bootstrap-server $KAFKABROKERS`。

4. 使用 __Ctrl + C__ 來停止取用者。

您也可以利用程式設計方式建立產生者和取用者。 如需使用此 API 的範例，請參閱[採用 HDInsight 的 Kafka 產生者和取用者 API](apache-kafka-producer-consumer-api.md) 文件。

## <a name="clean-up-resources"></a>清除資源

不再需要時，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組、HDInsight 和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name $resourceGroup
```

> [!WARNING]
> HDInsight 叢集的計費起自叢集建立時，終至叢集刪除時。 計費是以每分鐘按比例計算，因此不再使用時，請一律刪除您的叢集。
> 
> 刪除 HDInsight 叢集上的 Kafka，也會刪除 Kafka 中儲存的任何資料。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Apache Spark 搭配 Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)
