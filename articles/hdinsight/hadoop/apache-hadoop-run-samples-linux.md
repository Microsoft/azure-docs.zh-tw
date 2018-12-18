---
title: 在 HDInsight 上執行 Hadoop MapReduce 範例 - Azure
description: 開始使用 HDInsight 中隨附的 jar 檔案中的 MapReduce 範例。 使用 SSH 連接到叢集，然後使用 Hadoop 命令執行範例工作。
keywords: hadoop 範例 jar、Hadoop 範例 jar、hadoop mapreduce 範例、mapreduce 範例
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: b0a4088a4473a731f9dec2d5f1e495b2eb9e937c
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047775"
---
# <a name="run-the-mapreduce-examples-included-in-hdinsight"></a>執行包含在 HDInsight 中的 MapReduce 範例

[!INCLUDE [samples-selector](../../../includes/hdinsight-run-samples-selector.md)]

了解如何執行隨附於 HDInsight 上之 Hadoop 的 MapReduce 範例。

## <a name="prerequisites"></a>必要條件

* **HDInsight 叢集**：請參閱 [在 Linux 上開始在 HDInsight 中搭配使用 Hadoop 與 Hive](apache-hadoop-linux-tutorial-get-started.md)

    > [!IMPORTANT]
    > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* **SSH 用戶端**：如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="the-mapreduce-examples"></a>MapReduce 範例

**位置**：範例位於 HDInsight 叢集上的 `/usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar`。

**內容**：下列範例都包含在此封存中：

* `aggregatewordcount`：以彙總為基礎的 mapreduce 程式，可計算輸入檔中的字數。
* `aggregatewordhist`：以彙總為基礎的 mapreduce 程式，可計算輸入檔中字數的長條圖。
* `bbp`：mapreduce 程式，使用貝利-波爾溫-普勞夫公式 (Bailey-Borwein-Plouffe) 計算 Pi 的確切位數。
* `dbcount`：範例作業，計數儲存在資料庫中的 pageview 記錄。
* `distbbp`：mapreduce 程式，使用 BBP 類型的公式來計算 Pi 的確切位數。
* `grep`：mapreduce 程式，可計算輸入中 regex 的相符項目。
* `join`：作業，可執行已排序且平均分割之資料集的聯結。
* `multifilewc`：可從數個檔案計算字數的作業。
* `pentomino`：mapreduce 圖格配置程式，可尋找五格骨牌 (pentomino) 問題的解答。
* `pi`：mapreduce 程式，使用擬蒙特卡羅 (quasi-Monte Carlo) 方法估計 Pi。
* `randomtextwriter`：mapreduce 程式，可為每個節點寫入 10 GB 的隨機文字資料。
* `randomwriter`：mapreduce 程式，可為每個節點寫入 10 GB 的隨機資料。
* `secondarysort`：範例，定義要簡化階段的二次排序。
* `sort`：mapreduce 程式，可排序隨機寫入器所寫入的資料。
* `sudoku`：數獨解答程式。
* `teragen`：產生用於 TeraSort 的資料。
* `terasort`：執行 TeraSort。
* `teravalidate`：檢查 TeraSort 的結果。
* `wordcount`：mapreduce 程式，可計算輸入檔中的字數。
* `wordmean`：mapreduce 程式，可計算輸入檔中字詞的平均長度。
* `wordmedian`：mapreduce 程式，可計算輸入檔中字詞的中位數長度。
* `wordstandarddeviation`：mapreduce 程式，可計算輸入檔中字詞長度的標準差。

**原始程式碼**：這些範例的原始程式碼包含在 HDInsight 叢集上，位於 `/usr/hdp/current/hadoop-client/src/hadoop-mapreduce-project/hadoop-mapreduce-examples`。

## <a name="run-the-wordcount-example"></a>執行 wordcount 範例

1. 使用 SSH 連線到 HDInsight。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 在 `username@#######:~$` 提示中，使用下列命令來列出範例：

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar
    ```

    此命令會產生本文件上一節中的範例清單。

3. 使用下列命令可取得特定範例的說明。 在此情況下為 **wordcount** 範例：

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount
    ```

    您會看見下列訊息：

        Usage: wordcount <in> [<in>...] <out>

    此訊息表示您可以為來源文件提供數個輸入路徑。 最後一個路徑是輸出 (來源文件中的字數計數) 的儲存處。

4. 使用以下命令計算達文西手稿筆記中的所有字數，該文件已隨附於您的叢集做為範例資料：

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/davinciwordcount
    ```

    此作業的輸入讀取自 `/example/data/gutenberg/davinci.txt`。 此範例的輸出會儲存在 `/example/data/davinciwordcount`。 這兩個路徑都位於叢集的預設儲存體上，而不是本機檔案系統上。

   > [!NOTE]
   > 如 wordcount 範例的說明所述，您也可以指定多個輸入檔。 例如， `hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/gutenberg/ulysses.txt /example/data/twowordcount` 會計算 davinci.txt 和 ulysses.txt 中的字數。

5. 工作完成後，使用以下命令來檢視輸出：

    ```bash
    hdfs dfs -cat /example/data/davinciwordcount/*
    ```

    此命令會串連作業所產生的所有輸出檔。 它會在主控台顯示輸出。 輸出大致如下：

        zum     1
        zur     1
        zwanzig 1
        zweite  1

    每一行代表一個字詞，以及該字詞在輸入資料中出現的次數。

## <a name="the-sudoku-example"></a>數獨範例

[數獨](https://en.wikipedia.org/wiki/Sudoku) 是由九個 3x3 的方格所組成的邏輯謎題。 方格中的一些格子含有數字，而有些格子則為空白，因此目標就是解出空白格子的數字。 先前的連結提供謎題的詳細資訊，而此範例的目的是要解出空白格子的數字。 所以我們的輸入應該是具有下列格式的檔案：

* 具有九個資料列的九個資料行
* 每個資料行可以包含一個數字或 `?` (表示空白的格子)
* 格子之間以空格分隔

建立數獨謎題必須採用特定的方法，那就是您不能在某個資料行或資料列中使用重複的數字。 HDInsight 叢集上已經有一個正確建立的範例。 它位於 `/usr/hdp/*/hadoop/src/hadoop-mapreduce-project/hadoop-mapreduce-examples/src/main/java/org/apache/hadoop/examples/dancing/puzzle1.dta` 且包含下列文字：

    8 5 ? 3 9 ? ? ? ?
    ? ? 2 ? ? ? ? ? ?
    ? ? 6 ? 1 ? ? ? 2
    ? ? 4 ? ? 3 ? 5 9
    ? ? 8 9 ? 1 4 ? ?
    3 2 ? 4 ? ? 8 ? ?
    9 ? ? ? 8 ? 5 ? ?
    ? ? ? ? ? ? 2 ? ?
    ? ? ? ? 4 5 ? 7 8

若要在 Sudoku 範例中執行此範例問題，請使用以下命令：

```bash
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar sudoku /usr/hdp/*/hadoop/src/hadoop-mapreduce-project/hadoop-mapreduce-examples/src/main/java/org/apache/hadoop/examples/dancing/puzzle1.dta
```

出現的結果會類似如下文字：

    8 5 1 3 9 2 6 4 7
    4 3 2 6 7 8 1 9 5
    7 9 6 5 1 4 3 8 2
    6 1 4 8 2 3 7 5 9
    5 7 8 9 6 1 4 2 3
    3 2 9 4 5 7 8 1 6
    9 4 7 2 8 6 5 3 1
    1 8 5 7 3 9 2 6 4
    2 6 3 1 4 5 9 7 8

## <a name="pi--example"></a>Pi (π) 範例

Pi 範例會使用統計 (擬蒙特卡羅法) 方法來估計 pi 的值。 點隨機放置在單位正方形中。 正方形也包含一個圓形。 點落在圓形中的機率等於圓面積，Pi/4。 Pi 的值可從 4R 的值來估計。 其中 R 是圓內點數佔正方形內總點數的比例。 使用的樣本點越多，估計越準確。

使用以下命令來執行此範例。 此命令會每次使用 16 個對應搭配 10,000,000 個取樣來估計 Pi 的值：

```bash
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar pi 16 10000000
```

此命令傳回的值類似於 **3.14159155000000000000**。 Pi 的前 10 個小數位數是 3.1415926535，供您參考。

## <a name="10-gb-greysort-example"></a>10 GB Greysort 範例

GraySort 是一種效能評定排序。 其計量為排序大量資料時 (通常至少為 100 TB) 所達成的排序速率 (TB/分鐘)。

本範例使用不太大的 10 GB 資料，所以執行起來相對較快。 本範例使用由 Owen O'Malley 和 Arun Murthy 共同開發的 MapReduce 應用程式。 這些應用程式於 2009 年的年度一般目的 (「耐力賽」) TB 排序效能評定中，以 0.578 TB/分鐘 (173 分鐘內達到 100 TB) 的速率獲勝。 如需此效能評比和其他排序效能評比的詳細資訊，請參閱 [Sortbenchmark](http://sortbenchmark.org/) 網站。

本範例使用三組 MapReduce 程式：

* **TeraGen**：MapReduce 程式，產生要排序的資料列

* **TeraSort**可取樣輸入資料並利用 MapReduce 將資料依全序排列

    TeraSort 是標準的 MapReduce 排序，但有一個自訂 Partitioner。 此 Partitioner 使用已排序的 N-1 取樣索引鍵清單，其定義每個歸納的索引鍵範圍。 尤其是，會傳送使得 sample[i-1] <= key < sample[i] 的所有索引鍵給歸納 i。 此 Partitioner 會保證歸納 i 的輸出全都小於歸納 i+1 的輸出。

* **TeraValidate**：可驗證全域排序輸出的 MapReduce 程式

    它會在輸出目錄中為每一個檔案建立一個對應，而每個對應可確保每一個索引鍵一定小於或等於前一個對應。 對應函式會產生每個檔案的第一個和最後一個金鑰的記錄。 歸納函式可確保檔案 i 的第一個金鑰大於檔案 i-1 的最後一個金鑰。 任何問題皆會回報為具備錯誤金鑰歸納階段的輸出。

使用下列步驟來產生資料、排序，並驗證輸出：

1. 產生 10 GB 的資料，這些資料稍後會儲存在 HDInsight 叢集預設儲存體中，位於 `/example/data/10GB-sort-input`：

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapred.map.tasks=50 100000000 /example/data/10GB-sort-input
    ```

    `-Dmapred.map.tasks` 會告訴 Hadoop 在這項工作中要使用多少 map 工作。 最後兩個參數會指示作業建立 10 GB 的資料，並將資料儲存在 `/example/data/10GB-sort-input`。

2. 使用以下命令來排序資料：

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapred.map.tasks=50 -Dmapred.reduce.tasks=25 /example/data/10GB-sort-input /example/data/10GB-sort-output
    ```

    `-Dmapred.reduce.tasks` 會告訴 Hadoop 在這項工作中要使用多少 reduce 工作。 最後兩個參數只是資料的輸入和輸出位置。

3. 使用以下命令驗證依排序產生的資料：

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapred.map.tasks=50 -Dmapred.reduce.tasks=25 /example/data/10GB-sort-output /example/data/10GB-sort-validate
    ```

## <a name="next-steps"></a>後續步驟

您已在本文中學到如何執行以 Linux 為基礎的 HDInsight 叢集所隨附的範例。 如需透過 HDInsight 使用 Pig、Hive 和 MapReduce 的教學課程，請參閱下列主題：

* [搭配使用 Pig 與 HDInsight 上的 Hadoop](hdinsight-use-pig.md)
* [搭配使用 Hive 與 HDInsight 上的 Hadoop](hdinsight-use-hive.md)
* [搭配使用 MapReduce 與 HDInsight 上的 Hadoop](hdinsight-use-mapreduce.md)

[hdinsight-sdk-documentation]: https://msdn.microsoft.com/library/azure/dn479185.aspx

[hdinsight-submit-jobs]:submit-apache-hadoop-jobs-programmatically.md
[hdinsight-introduction]:apache-hadoop-introduction.md



[hdinsight-samples]: hdinsight-run-samples.md

