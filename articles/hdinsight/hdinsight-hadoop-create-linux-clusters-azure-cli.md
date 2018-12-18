---
title: 使用 Azure 傳統 CLI 建立 Hadoop 叢集 - Azure HDInsight
description: 了解如何使用跨平台 Azure 傳統 CLI 建立 HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: jasonh
ms.openlocfilehash: 84b352fea0c5b9c98cd3b4e814e448cf8b706402
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992808"
---
# <a name="create-hdinsight-clusters-using-the-azure-classic-cli"></a>使用 Azure 傳統 CLI 建立 HDInsight 叢集

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

此文件中的步驟詳細說明如何使用 Azure 傳統 CLI 建立 HDInsight 3.5 叢集。

[!INCLUDE [classic-cli-warning](../../includes/requires-classic-cli.md)]

## <a name="prerequisites"></a>必要條件

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。

* **Azure 傳統 CLI**。 此文件中的步驟最近一次是以 Azure 傳統 CLI 版本 0.10.14 來測試。

## <a name="log-in-to-your-azure-subscription"></a>登入您的 Azure 訂用帳戶

依照 [從 Azure 命令列介面連線到 Azure 訂用帳戶](/cli/azure/authenticate-azure-cli)中記載的步驟，使用 **login** 方法連線到您的訂用帳戶。

## <a name="create-a-cluster"></a>建立叢集

下列步驟應透過命令列 (如 PowerShell 或 Bash) 執行。

1. 使用下列命令來驗證您的 Azure 訂用帳戶：

        azure login

    系統會提示您提供使用者名稱與密碼。 如果您有多個 Azure 訂用帳戶，則可以使用 `azure account set <subscriptionname>` 來設定傳統 CLI 命令所使用的訂用帳戶。

2. 使用下列命令來切換至 Azure 資源管理員模式︰

        azure config mode arm

3. 建立資源群組。 此資源群組包含 HDInsight 叢集和關聯的儲存體帳戶。

        azure group create groupname location

    * 以群組的唯一名稱取代 `groupname`。

    * 以您想要在其中建立群組的地理區域取代 `location`。

       如需有效位置的清單，請使用 `azure location list` 命令，然後使用 [`Name`] \(名稱\) 欄中的其中一個位置。

4. 建立儲存體帳戶。 此儲存體帳戶會用來作為 HDInsight 叢集的預設儲存體。

        azure storage account create -g groupname --sku-name RAGRS -l location --kind Storage storagename

    * 以上一個步驟中建立的群組名稱取代 `groupname`。

    * 以與上一個步驟中使用的相同位置取代 `location`。

    * 以儲存體帳戶的唯一名稱取代 `storagename`。

        > [!NOTE]
        > 如需有關此命令中所使用參數的詳細資訊，請使用 `azure storage account create -h` 來檢視此命令的說明。

5. 擷取用來存取儲存體帳戶的金鑰。

        azure storage account keys list -g groupname storagename

    * 以資源群組名稱取代 `groupname`。
    * 使用儲存體帳戶名稱取代 `storagename`。

     在傳回的資料中，儲存 `key1` 的 `key` 值。

6. 建立 HDInsight 叢集。

        azure hdinsight cluster create -g groupname -l location -y Linux --clusterType Hadoop --defaultStorageAccountName storagename.blob.core.windows.net --defaultStorageAccountKey storagekey --defaultStorageContainer clustername --workerNodeCount 3 --userName admin --password httppassword --sshUserName sshuser --sshPassword sshuserpassword clustername

    * 以資源群組名稱取代 `groupname`。

    * 將 `Hadoop` 取代為您想要建立的叢集類型。 例如，`Hadoop`、`HBase`、`Kafka`、`Spark` 或 `Storm`。

     > [!IMPORTANT]
     > HDInsight 叢集有各種不同類型，這些類型各自對應到叢集微調時所針對的工作負載或技術。 沒有任何支援方法可建立結合多個類型的叢集，例如在一個叢集上並存 Storm 和 HBase。

    * 以與前述步驟中使用的相同位置取代 `location`。

    * 使用儲存體帳戶名稱取代 `storagename`。

    * 以在上一個步驟中取得的金鑰取代 `storagekey`。

    * 針對 `--defaultStorageContainer` 參數，使用與您用於叢集的相同名稱。

    * 以當您透過 HTTPS 存取叢集時所要使用的名稱和密碼取代 `admin` 和 `httppassword`。

    * 以當您使用 SSH 存取叢集時所要使用的使用者名稱和密碼取代 `sshuser` 和 `sshuserpassword`。

    > [!IMPORTANT]
    > 此範例使用兩個背景工作角色節點建立叢集。 您也可以在叢集建立後執行調整規模作業，以變更背景工作角色節點數。 如果您規劃使用 32 個以上的背景工作角色節點，則必須選取具有至少 8 個核心和 14 GB RAM 的前端節點大小。 建立叢集期間，您可以使用 `--headNodeSize` 參數來設定前端節點大小。
    >
    > 如需節點大小和相關成本的詳細資訊，請參閱 [HDInsight 定價](https://azure.microsoft.com/pricing/details/hdinsight/)。

    可能需要數分鐘的時間，才能完成叢集建立程序。 通常大約 15 分鐘。

## <a name="troubleshoot"></a>疑難排解

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](hdinsight-administer-use-portal-linux.md#create-clusters)。

## <a name="next-steps"></a>後續步驟

既然您已使用傳統 CLI 順利建立 HDInsight 叢集，請使用下列內容來了解如何使用您的叢集：

### <a name="hadoop-clusters"></a>Hadoop 叢集

* [搭配 HDInsight 使用 Hivet](hadoop/hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hadoop/hdinsight-use-pig.md)
* [搭配 HDInsight 使用 MapReduce](hadoop/hdinsight-use-mapreduce.md)

### <a name="hbase-clusters"></a>HBase 叢集

* [開始在 HDInsight 上使用 HBase](hbase/apache-hbase-tutorial-get-started-linux.md)
* [在 HDInsight 上開發適用於 HBase 的 Java 應用程式](hbase/apache-hbase-build-java-maven-linux.md)

### <a name="storm-clusters"></a>Storm 叢集

* [在 HDInsight 上開發適用於 Storm 的 Java 拓撲](storm/apache-storm-develop-java-topology.md)
* [在 HDInsight 上的 Storm 中使用 Python 元件](storm/apache-storm-develop-python-topology.md)
* [在 HDInsight 上使用 Storm 部署和監視拓撲](storm/apache-storm-deploy-monitor-topology-linux.md)
