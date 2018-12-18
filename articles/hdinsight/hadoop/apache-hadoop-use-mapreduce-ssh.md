---
title: MapReduce 和 SSH 連線與 HDInsight 中的 Hadoop - Azure
description: 了解如何使用 SSH，以利用 HDInsight 上的 Hadoop 來執行 MapReduce 工作。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/10/2018
ms.author: jasonh
ms.openlocfilehash: 71472edf74fba433f24b83362b2880575b73ce85
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047318"
---
# <a name="use-mapreduce-with-hadoop-on-hdinsight-with-ssh"></a>搭配使用 MapReduce 與 HDInsight 上的 Hadoop 和 SSH

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

了解如何從 HDInsight 的安全殼層 (SSH) 連線提交 MapReduce 作業。

> [!NOTE]
> 如果您已熟悉使用以 Linux 為基礎的 Hadoop 伺服器，但剛接觸 HDInsight，請參閱 [以 Linux 為基礎的 HDInsight 秘訣](../hdinsight-hadoop-linux-information.md)。

## <a id="prereq"></a>必要條件

* Linux 型 HDInsight (HDInsight 上的 Hadoop) 叢集

  > [!IMPORTANT]
  > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* SSH 用戶端。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)

## <a id="ssh"></a>使用 SSH 連線

使用 SSH 連線到叢集。 例如，下列命令會以 **sshuser** 帳戶的身分連線至名為 **myhdinsight** 的叢集：

```bash
ssh sshuser@myhdinsight-ssh.azurehdinsight.net
```

**如果您使用憑證金鑰進行 SSH 驗證**，您可能需要指定用戶端系統上私密金鑰的位置，例如：

```bash
ssh -i ~/mykey.key sshuser@myhdinsight-ssh.azurehdinsight.net
```

**如果您使用密碼進行 SSH 驗證**，您需要在接獲提示時提供密碼。

如需搭配 HDInsight 使用 SSH 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a id="hadoop"></a>使用 Hadoop 命令

1. 連線到 HDInsight 叢集之後，請使用下列命令來啟動 MapReduce 作業：

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/WordCountOutput
    ```

    此命令會啟動 `hadoop-mapreduce-examples.jar` 檔案中包含的 `wordcount` 類別。 它會使用 `/example/data/gutenberg/davinci.txt` 文件作為輸入，而輸出會儲存在 `/example/data/WordCountOutput`。

    > [!NOTE]
    > 如需關於此 MapReduce 工作和範例資料的詳細資訊，請參閱 [在 HDInsight 上的 Hadoop 中使用 MapReduce](hdinsight-use-mapreduce.md)。

2. 作業會在處理時發出詳細資料，並於作業完成時傳回類似下列文字的資訊：

        File Input Format Counters
        Bytes Read=1395666
        File Output Format Counters
        Bytes Written=337623

3. 作業完成時，請使用下列命令來列出輸出檔案︰

    ```bash
    hdfs dfs -ls /example/data/WordCountOutput
    ```

    此命令會顯示兩個檔案︰`_SUCCESS` 和 `part-r-00000`。 `part-r-00000` 檔案包含此作業的輸出。

    > [!NOTE]
    > 某些 MapReduce 工作可能會將結果分成多個 **part-r-#####** 檔案。 若是如此，請使用 ##### 尾碼指出檔案的順序。

4. 若要檢視輸出，請使用下列命令：

    ```bash
    hdfs dfs -cat /example/data/WordCountOutput/part-r-00000
    ```

    此命令會顯示 **wasb://example/data/gutenberg/davinci.txt** 檔案中所含的單字清單和每個單字的出現次數。 下列文字此檔案所包含之資料的範例：

        wreathed        3
        wreathing       1
        wreaths         1
        wrecked         3
        wrenching       1
        wretched        6
        wriggling       1

## <a id="summary"></a>摘要

如您所見，Hadoop 命令提供簡單的方法，在 HDInsight 叢集中執行 MapReduce 工作，然後檢視工作輸出。

## <a id="nextsteps"></a>接續步驟

如需 HDInsight 中 MapReduce 工作的一般資訊：

* [在 HDInsight Hadoop 上使用 MapReduce](hdinsight-use-mapreduce.md)

如需您可以在 HDInsight 上使用 Hadoop 之其他方式的詳細資訊：

* [搭配使用 Hive 與 HDInsight 上的 Hadoop](hdinsight-use-hive.md)
* [搭配使用 Pig 與 HDInsight 上的 Hadoop](hdinsight-use-pig.md)
