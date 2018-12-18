---
title: 在 HDInsight 叢集上搭配使用 Hadoop Pig 與 SSH - Azure
description: 學習如何使用 SSH 連線到以 Linux 為基礎的 Hadoop 叢集，然後使用 Pig 命令以互動方式或批次工作形式執行 Pig Latin 陳述式。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: jasonh
ms.openlocfilehash: 4d380c44511ad330542402499829c19de62bd39a
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43051521"
---
# <a name="run-pig-jobs-on-a-linux-based-cluster-with-the-pig-command-ssh"></a>使用 Pig 命令 (SSH) 在以 Linux 為基礎的叢集上執行 Pig 工作

[!INCLUDE [pig-selector](../../../includes/hdinsight-selector-use-pig.md)]

了解如何透過與 HDInsight 叢集的 SSH 連線以互動方式執行 Pig 作業。 Pig Latin 程式設計語言可讓您描述套用至輸入資料來產生想要輸出的轉換。

> [!IMPORTANT]
> 此文件中的步驟需要以 Linux 為基礎的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a id="ssh"></a>使用 SSH 連線

使用 SSH 連線到 HDInsight 叢集。 下列範例會以名為 **sshuser**的帳戶連線至名為 **myhdinsight** 的叢集：

```bash
ssh sshuser@myhdinsight-ssh.azurehdinsight.net
```

如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a id="pig"></a>使用 Pig 命令

1. 連線之後，使用下列命令來啟動 Pig 命令列介面 (CLI)：

    ```bash
    pig
    ```

    隨後，提示字元會變更為 `grunt>`。

2. 輸入下列陳述式：

    ```piglatin
    LOGS = LOAD '/example/data/sample.log';
    ```

    此命令會將 sample.log 檔案的內容載入至 LOGS。 您可以使用下列陳述式檢視檔案的內容：

    ```piglatin
    DUMP LOGS;
    ```

3. 接下來是轉換資料，方法是套用規則運算式，並使用下列陳述式僅擷取每筆記錄的記錄層級：

    ```piglatin
    LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
    ```

    轉換之後，您可以使用 **DUMP** 來檢視資料。 在此案例中，請使用 `DUMP LEVELS;`。

4. 使用下列表格中的陳述式繼續套用轉換：

    | Pig Latin 陳述式 | 陳述式的作用 |
    | ---- | ---- |
    | `FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;` | 移除含有記錄層級之 Null 值的資料列，並將結果儲存到 `FILTEREDLEVELS`。 |
    | `GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;` | 依記錄層級對資料列進行分組，並將結果儲存到 `GROUPEDLEVELS`。 |
    | `FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;` | 建立一組資料，其中包含每個唯一記錄層級值和其發生次數。 資料集會儲存到 `FREQUENCIES`。 |
    | `RESULT = order FREQUENCIES by COUNT desc;` | 依計數排序記錄層級 (遞減)，並且儲存到 `RESULT`。 |

    > [!TIP]
    > 使用 `DUMP` 檢視每個步驟後的轉換結果。

5. 您也可以使用 `STORE` 陳述式儲存轉換結果。 例如，下列陳述式會將 `RESULT` 儲存到叢集之預設儲存體上的 `/example/data/pigout` 目錄：

    ```piglatin
    STORE RESULT into '/example/data/pigout';
    ```

   > [!NOTE]
   > 資料會儲存到所指定目錄中名為 `part-nnnnn` 的檔案。 如果目錄已經存在，則會收到錯誤。

6. 若要結束 grunt 提示字元，請輸入下列陳述式：

    ```piglatin
    QUIT;
    ```

### <a name="pig-latin-batch-files"></a>Pig Latin 批次檔

您也可以使用 Pig 命令執行檔案中所含的 Pig Latin。

1. 結束 Grunt 提示字元之後，請使用下列命令來建立名為 `pigbatch.pig` 的檔案：

    ```bash
    nano ~/pigbatch.pig
    ```

2. 輸入或貼上下列行：

    ```piglatin
    LOGS = LOAD '/example/data/sample.log';
    LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
    FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
    GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
    FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
    RESULT = order FREQUENCIES by COUNT desc;
    DUMP RESULT;
    ```

    完成後，請使用 __Ctrl__ + __X__、__Y__，然後按 __Enter__ 來儲存檔案。

3. 使用下列命令，以使用 Pig 命令來執行 `pigbatch.pig` 檔案。

    ```bash
    pig ~/pigbatch.pig
    ```

    當批次作業完成時，您會看到下列輸出︰

        (TRACE,816)
        (DEBUG,434)
        (INFO,96)
        (WARN,11)
        (ERROR,6)
        (FATAL,2)


## <a id="nextsteps"></a>後續步驟

如需有關 HDInsight 中 Pig 的一般資訊，請參閱下列文件：

* [搭配使用 Pig 與 HDInsight 上的 Hadoop](hdinsight-use-pig.md)

如需搭配 HDInsight 上的 Hadoop 使用之其他方式的詳細資訊，請參閱下列文件：

* [搭配 HDInsight 上的 Hadoop 使用 Hive](hdinsight-use-hive.md)
* [搭配使用 MapReduce 與 HDInsight 上的 Hadoop](hdinsight-use-mapreduce.md)
