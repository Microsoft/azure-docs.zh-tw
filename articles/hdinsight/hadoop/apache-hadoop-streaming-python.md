---
title: 使用 HDInsight 開發 Python 串流處理 MapReduce 工作 - Azure
description: 了解如何在串流處理 MapReduce 工作中使用 Python。 Hadoop 提供適用於 MapReduce 的串流處理 API，可以 Java 以外的語言撰寫。
services: hdinsight
keyword: mapreduce python,python map reduce,python mapreduce
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 04/10/2018
ms.author: jasonh
ms.openlocfilehash: 8e2d13e0d9e51855bc8945db61c78a2fec736c33
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041705"
---
# <a name="develop-python-streaming-mapreduce-programs-for-hdinsight"></a>開發 HDInsight 的 Python 串流處理 MapReduce 程式

了解如何在串流處理 MapReduce 作業中使用 Python。 Hadoop 為 MapReduce 提供一個串流 API，可讓您以 Java 以外的語言撰寫 map 和 reduce 函數。 本文件中的步驟會以 Python 實作對應和歸納元件。

## <a name="prerequisites"></a>必要條件

* HDInsight 叢集上的 Linux 型 Hadoop

  > [!IMPORTANT]
  > 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* 文字編輯器

  > [!IMPORTANT]
  > 文字編輯器必須使用 LF 做為行尾結束符號。 在以 Linux 為基礎的 HDInsight 叢集上執行 MapReduce 作業時，使用 CRLF 行尾結束符號會導致錯誤。

* `ssh` 和 `scp` 命令，或 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-3.8.0)

## <a name="word-count"></a>字數統計

此範例是使用 python 對應器和歸納器實作的基本字數統計。 對應器會將句子拆解成個別文字，歸納器則會彙總文字和計數來產生輸出。

下列流程圖說明在對應和歸納階段期間所執行的程序。

![mapreduce 程序圖](./media/apache-hadoop-streaming-python/HDI.WordCountDiagram.png)

## <a name="streaming-mapreduce"></a>串流 MapReduce

Hadoop 可讓您指定包含工作所使用的對應和歸納邏輯的檔案。 對應和歸納邏輯的特殊需求如下：

* **輸入**：對應和歸納元件必須從 STDIN 讀取輸入資料。
* **輸出**：對應和歸納元件必須將輸出資料寫入至 STDOUT。
* **資料格式**：所取用和產生的資料必須是機碼/值組，並以 Tab 字元隔開。

Python 可以使用 `sys` 模組來讀取 STDIN 並使用 `print` 來列印到 STDOUT，輕鬆應付這些需求。 剩下的工作便只需要在機碼和值之間以 Tab (`\t`) 字元格式化資料。

## <a name="create-the-mapper-and-reducer"></a>建立對應器和歸納器

1. 建立名為 `mapper.py` 的檔案並使用下列程式碼作為內容：

   ```python
   #!/usr/bin/env python

   # Use the sys module
   import sys

   # 'file' in this case is STDIN
   def read_input(file):
       # Split each line into words
       for line in file:
           yield line.split()

   def main(separator='\t'):
       # Read the data using read_input
       data = read_input(sys.stdin)
       # Process each word returned from read_input
       for words in data:
           # Process each word
           for word in words:
               # Write to STDOUT
               print '%s%s%d' % (word, separator, 1)

   if __name__ == "__main__":
       main()
   ```

2. 建立名為 **reducer.py** 的檔案並使用下列程式碼作為內容：

   ```python
   #!/usr/bin/env python

   # import modules
   from itertools import groupby
   from operator import itemgetter
   import sys

   # 'file' in this case is STDIN
   def read_mapper_output(file, separator='\t'):
       # Go through each line
       for line in file:
           # Strip out the separator character
           yield line.rstrip().split(separator, 1)

   def main(separator='\t'):
       # Read the data using read_mapper_output
       data = read_mapper_output(sys.stdin, separator=separator)
       # Group words and counts into 'group'
       #   Since MapReduce is a distributed process, each word
       #   may have multiple counts. 'group' will have all counts
       #   which can be retrieved using the word as the key.
       for current_word, group in groupby(data, itemgetter(0)):
           try:
               # For each word, pull the count(s) for the word
               #   from 'group' and create a total count
               total_count = sum(int(count) for current_word, count in group)
               # Write to stdout
               print "%s%s%d" % (current_word, separator, total_count)
           except ValueError:
               # Count was not a number, so do nothing
               pass

   if __name__ == "__main__":
       main()
   ```

## <a name="run-using-powershell"></a>使用 PowerShell 執行

若要確保您的檔案有正確的行尾結束符號，請使用下列 PowerShell 指令碼︰

[!code-powershell[main](../../../powershell_scripts/hdinsight/streaming-python/streaming-python.ps1?range=138-140)]

使用下列 PowerShell 指令碼來上傳檔案、執行作業，以及檢視輸出︰

[!code-powershell[main](../../../powershell_scripts/hdinsight/streaming-python/streaming-python.ps1?range=5-134)]

## <a name="run-from-an-ssh-session"></a>從 SSH 工作階段執行

1. 從開發環境上，在 `mapper.py` 和 `reducer.py` 檔案所在的目錄中，使用下列命令：

    ```bash
    scp mapper.py reducer.py username@clustername-ssh.azurehdinsight.net:
    ```

    以叢集的 SSH 使用者取代 `username`，並以叢集的名稱取代 `clustername`。

    此命令會將檔案從本機系統複製到前端節點。

    > [!NOTE]
    > 如果您使用密碼來保護 SSH 帳戶，系統就會提示您輸入密碼。 如果您使用 SSH 金鑰，您可能必須使用 `-i` 參數和私密金鑰的路徑。 例如： `scp -i /path/to/private/key mapper.py reducer.py username@clustername-ssh.azurehdinsight.net:`。

2. 使用 SSH 連線到叢集：

    ```bash
    ssh username@clustername-ssh.azurehdinsight.net`
    ```

    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

3. 若要確保 mapper.py 和 reducer.py 有正確的行尾結束符號，請使用下列命令︰

    ```bash
    perl -pi -e 's/\r\n/\n/g' mapper.py
    perl -pi -e 's/\r\n/\n/g' reducer.py
    ```

4. 使用下列命令啟動 MapReduce 工作。

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar -files mapper.py,reducer.py -mapper mapper.py -reducer reducer.py -input /example/data/gutenberg/davinci.txt -output /example/wordcountout
    ```

    此命令有下列幾個部分：

   * **hadoop-streaming.jar**：執行串流 MapReduce 作業時使用。 它能連結 Hadoop 和您提供的外部 MapReduce 程式碼。

   * **-file**：將指定的檔案新增至 MapReduce 作業。

   * **-mapper**：告訴 Hadoop 要做為對應器的檔案。

   * **-reducer**：告訴 Hadoop 要做為歸納器的檔案。

   * **-input**：要從中計算字數的輸入檔案。

   * **-output**：輸出時寫入的目錄。

    MapReduce 作業執行時會以百分比顯示過程。

        15/02/05 19:01:04 INFO mapreduce.Job:  map 0% reduce 0%    15/02/05 19:01:16 INFO mapreduce.Job:  map 100% reduce 0%    15/02/05 19:01:27 INFO mapreduce.Job:  map 100% reduce 100%


5. 若要檢視輸出，請使用下列命令：

    ```bash
    hdfs dfs -text /example/wordcountout/part-00000
    ```

    此命令會顯示單字清單及單字出現次數。

## <a name="next-steps"></a>後續步驟

現在您已學會如何搭配 HDInsight 使用串流 MapRedcue 工作，接著請使用下列連結來探索 Azure HDInsight 的其他使用方式。

* [〈搭配 HDInsight 使用 Hivet〉](hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hdinsight-use-pig.md)
* [搭配 HDInsight 使用 MapReduce 工作](hdinsight-use-mapreduce.md)
