---
title: 在 Azure HDInsight 中搭配 HBase 使用 Apache Phoenix 和 SQLLine
description: 了解如何在 HDInsight 中使用 Apache Phoenix。 也了解如何在電腦上安裝並設定 SQLLine，以連線到在 HDInsight 中的 HBase 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/03/2018
ms.author: jasonh
ms.openlocfilehash: 92e6dcf7bf467bac31798df994e7efa8da40f035
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045653"
---
# <a name="use-apache-phoenix-with-linux-based-hbase-clusters-in-hdinsight"></a>在 HDInsight 中搭配 Linux 型 HBase 叢集使用 Apache Phoenix
了解如何在 Azure HDInsight 中使用 [Apache Phoenix](http://phoenix.apache.org/)，以及如何使用 SQLLine。 如需有關 Phoenix 的詳細資訊，請參閱 [15 分鐘內了解 Phoenix](http://phoenix.apache.org/Phoenix-in-15-minutes-or-less.html)。 如需 Phoenix 文法，請參閱 [Phoenix 文法](http://phoenix.apache.org/language/index.html)。

> [!NOTE]
> 如需關於 HDInsight 的 Phoenix 版本資訊，請參閱 [HDInsight 在 Hadoop 叢集版本中提供的新功能](../hdinsight-component-versioning.md)。
>
>

## <a name="use-sqlline"></a>使用 SQLLine
[SQLLine](http://sqlline.sourceforge.net/) \(英文\) 是執行 SQL 的命令列公用程式。

### <a name="prerequisites"></a>必要條件
開始使用 SQLLine 之前，您必須具備下列項目：

* **HDInsight 中的 HBase 叢集**。 若要建立，請參閱[開始使用 HDInsight 中的 Apache HBase](./apache-hbase-tutorial-get-started-linux.md)。

連線到 HBase 叢集時，您必須連線到其中一個 ZooKeeper VM。 每個 HDInsight 叢集有三個 ZooKeeper VM。

**取得 ZooKeeper 主機名稱**

1. 瀏覽至 **https://\<叢集名稱\>.azurehdinsight.net** 來開啟 Ambari。
2. 輸入 HTTP (叢集) 使用者名稱和密碼來登入。
3. 在左側功能表中，選取 **ZooKeeper**。 系統會列出三個 **ZooKeeper 伺服器** 執行個體。
4. 選取其中一個 **ZooKeeper 伺服器**執行個體。 在 [摘要] 窗格中，找到 [主機名稱]。 主機名稱會類似 *zk1-jdolehb.3lnng4rcvp5uzokyktxs4a5dhd.bx.internal.cloudapp.net*。

**使用 SQLLine**

1. 使用 SSH 連線到叢集。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 在 SSH 中使用下列命令以執行 SQLLine：

        cd /usr/hdp/2.2.9.1-7/phoenix/bin
        ./sqlline.py <ZOOKEEPER SERVER FQDN>:2181:/hbase-unsecure
3. 若要建立 HBase 資料表並插入一些資料，請執行下列命令︰

        CREATE TABLE Company (COMPANY_ID INTEGER PRIMARY KEY, NAME VARCHAR(225));

        !tables

        UPSERT INTO Company VALUES(1, 'Microsoft');

        SELECT * FROM Company;

        !quit

如需詳細資訊，請參閱 [SQLLine 手冊](http://sqlline.sourceforge.net/#manual)和 [Phoenix 文法](http://phoenix.apache.org/language/index.html)。

## <a name="next-steps"></a>後續步驟
在本文中，您已經學會如何在 HDInsight 中使用 Apache Phoenix。 若要深入了解，請參閱下列文章：

* [HDInsight HBase 概觀][hdinsight-hbase-overview]。
  HBase 是建置於 Hadoop 上的 Apache 開放原始碼 NoSQL 資料庫，可針對大量非結構化及半結構化資料，提供隨機存取功能和強大一致性。
* [在 Azure 虛擬網路上佈建 HBase 叢集][hdinsight-hbase-provision-vnet]。
  由於 HBase 叢集已與虛擬網路整合，因此能夠部署到與您應用程式相同的虛擬網路，讓應用程式得以和 HBase 直接通訊。
* [在 HDInsight 中設定 HBase 複寫](apache-hbase-replication.md)。 了解如何跨兩個 Azure 資料中心設定 HBase 複寫。


[azure-portal]: https://portal.azure.com
[vnet-point-to-site-connectivity]: https://msdn.microsoft.com/library/azure/09926218-92ab-4f43-aa99-83ab4d355555#BKMK_VNETPT

[hdinsight-manage-portal]: hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp
[hdinsight-hbase-provision-vnet]:apache-hbase-provision-vnet.md
[hdinsight-hbase-overview]:apache-hbase-overview.md


