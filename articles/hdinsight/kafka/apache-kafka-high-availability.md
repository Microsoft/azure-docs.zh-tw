---
title: 使用 Apache Kafka 確保高可用性 - Azure HDInsight
description: 了解如何使用 Azure HDInsight 上的 Apache Kafka 確保高可用性。 了解如何重新平衡 Kafka 中的磁碟分割複本，使其位於包含 HDInsight 的 Azure 區域內的不同容錯網域上。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/01/2018
ms.openlocfilehash: bd3b02d54e0a65411e45f0422a0d245645d59096
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049966"
---
# <a name="high-availability-of-your-data-with-apache-kafka-on-hdinsight"></a>使用 HDInsight 上的 Apache Kafka 確保您資料的高可用性

了解如何設定 Kafka 主題的磁碟分割複本，以利用基礎硬體機架組態。 此組態可確保在 HDInsight 上 Apache Kafka 中儲存之資料的可用性。

## <a name="fault-and-update-domains-with-kafka"></a>容錯和更新網域與 Kafka

容錯網域是 Azure 資料中心內基礎硬體的邏輯群組。 每個容錯網域會共用通用電源和網路交換器。 實作 HDInsight 叢集內節點的虛擬機器和受控磁碟會分散於這些容錯網域。 此架構會限制實體硬體故障的潛在影響。

每個 Azure 區域有特定數目的容錯網域。 如需網域清單及其包含的容錯網域數目，請參閱[可用性設定組](../../virtual-machines/windows/regions-and-availability.md#availability-sets)文件。

> [!IMPORTANT]
> Kafka 不知道容錯網域。 當您在 Kafka 中建立主題時，它可將所有磁碟分割複本儲存在相同的容錯網域中。 為了解決這個問題，HDInsight 提供 [Kafka 磁碟分割重新平衡工具](https://github.com/hdinsight/hdinsight-kafka-tools)。

## <a name="when-to-rebalance-partition-replicas"></a>何時重新平衡磁碟分割複本

若要確保 Kafka 資料的最高的可用性，您應該在下列時間重新平衡您主題的磁碟分割複本：

* 建立新主題或磁碟分割時

* 當您相應增加叢集時

## <a name="replication-factor"></a>複寫因子

> [!IMPORTANT]
> 我們建議使用包含三個容錯網域的 Azure 地區，以及使用複寫因子 3。

如果您必須使用只包含兩個容錯網域的區域，請使用複寫因子 4 將複本平均分散於兩個容錯網域。

如需建立主題及設定複寫因子的範例，請參閱[開始使用 HDInsight 上的 Kafka](apache-kafka-get-started.md)文件。

## <a name="how-to-rebalance-partition-replicas"></a>如何重新平衡磁碟分割複本

使用 [Kafka 磁碟割區重新平衡工具](https://github.com/hdinsight/hdinsight-kafka-tools)來重新平衡所選的主題。 必須從 Kafka 叢集前端節點的 SSH 工作階段執行此工具。

如需使用 SSH 連線至 HDInsight 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

## <a name="next-steps"></a>後續步驟

* [HDInsight 上的 Kafka 延展性](apache-kafka-scalability.md)
* [使用 HDInsight 上的 Kafka 進行鏡像處理](apache-kafka-mirroring.md)
