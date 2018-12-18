---
title: Apache Kafka 提高級別 - Azure HDInsight
description: 了解如何設定在 Azure HDInsight 上 Apache Kafka 叢集的受控磁碟來提高延展性。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/30/2018
ms.openlocfilehash: f74bc5732bbcf05423698603113998204b7514a9
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046669"
---
# <a name="configure-storage-and-scalability-for-apache-kafka-on-hdinsight"></a>在 HDInsight 上設定 Apache Kafka 的儲存體和延展性

了解如何設定 Apache Kafka 在 HDInsight 上所使用的受控磁碟數目。

HDInsight 上的 Kafka 會在 HDInsight 叢集中使用虛擬機器的本機磁碟。 由於 Kafka 的 I/O 非常大量，因此會使用 [Azure 受控磁碟](../../virtual-machines/windows/managed-disks-overview.md)來提供高輸送量，並提供每個節點更多儲存空間。 如果將傳統的虛擬硬碟 (VHD) 用於 Kafka，每個節點就會限制為 1 TB。 使用受控磁碟時，您可以利用多個磁碟在叢集中的每個節點達到 16 TB。

下圖提供 HDInsight 上的 Kafka 採用受控磁碟之前與 HDInsight 上的 Kafka 採用受控磁碟之間的比較：

![圖表顯示 HDInsight 上的 Kafka 每個 VM 使用單一 VHD 與每個 VM 使用多個受控磁碟](./media/apache-kafka-scalability/kafka-with-managed-disks-architecture.png)

## <a name="configure-managed-disks-azure-portal"></a>設定受控磁碟：Azure 入口網站

1. 請遵循[建立 HDInsight 叢集](../hdinsight-hadoop-create-linux-clusters-portal.md)中的步驟，了解使用入口網站建立叢集的一般步驟。 請勿完成入口網站建立程序。

2. 從 [叢集大小] 區段，使用 [每個背景工作角色節點的磁碟] 欄位來設定磁碟的數目。

    > [!NOTE]
    > 受控磁碟的類型可以是__標準__ (HDD) 或__進階__ (SSD)。 進階磁碟會與 DS 和 GS 系列搭配使用。 所有其他的 VM 類型是使用標準磁碟。

    ![叢集大小區段的映像，其中每個背景工作角色節點的磁碟都會醒目提示](./media/apache-kafka-scalability/set-managed-disks-portal.png)

## <a name="configure-managed-disks-resource-manager-template"></a>設定受控磁碟：Resource Manager 範本

若要控制背景工作角色節點在 Kafka 叢集中所使用的磁碟數目，請使用下列區段的範本：

```json
"dataDisksGroups": [
    {
        "disksPerNode": "[variables('disksPerWorkerNode')]"
    }
    ],
```

您可以在 [https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-kafka-mirror-cluster-in-vnet-v2.1.json](https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-kafka-mirror-cluster-in-vnet-v2.1.json) 中找到示範如何設定受控磁碟的完整範本。

## <a name="next-steps"></a>後續步驟

如需使用 HDInsight 上 Kafka 的詳細資訊，請參閱下列文件：

* [使用 MirrorMaker 建立 Apache Kafka on HDInsight 複本](apache-kafka-mirroring.md)
* [使用 Apache Storm 搭配 HDInsight 上的 Kafka](../hdinsight-apache-storm-with-kafka.md)
* [使用 Apache Spark 搭配 Kafka on HDInsight](../hdinsight-apache-spark-with-kafka.md)
* [透過 Azure 虛擬網路連線至 Kafka](apache-kafka-connect-vpn-gateway.md)

* [使用 Kafka 之受控磁碟的 HDInsight 部落格](https://azure.microsoft.com/blog/announcing-public-preview-of-apache-kafka-on-hdinsight-with-azure-managed-disks/)
