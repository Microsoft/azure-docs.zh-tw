---
title: MapReduce 與 HDInsight 上的 Hadoop
description: 了解如何在 HDInsight 叢集中的 Hadoop 上執行 MapReduce 工作。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.openlocfilehash: 3b9cc70a1adc55850923f2313f17be435257117d
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43044864"
---
# <a name="use-mapreduce-in-hadoop-on-hdinsight"></a>在 HDInsight 上的 Hadoop 中使用 MapReduce

了解如何在 HDInsight 叢集上執行 MapReduce 工作。 使用下表了解可搭配 HDInsight 使用 MapReduce 的各種方式︰

| **使用此方法**... | **...執行此工作** | ...搭配此 **叢集作業系統** | ...從此 **用戶端作業系統** |
|:--- |:--- |:--- |:--- |
| [SSH](apache-hadoop-use-mapreduce-ssh.md) |透過 **SSH** |Linux |Linux、Unix、Mac OS X 或 Windows |
| [REST](apache-hadoop-use-mapreduce-curl.md) |使用 **REST** (例如使用 cURL) 從遠端提交工作 |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [Windows PowerShell](apache-hadoop-use-mapreduce-powershell.md) |使用 **Windows PowerShell** |Linux 或 Windows |Windows |

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。
>

## <a id="whatis"></a>什麼是 MapReduce

Hadoop MapReduce 是一種可撰寫工作來處理大量資料的軟體架構。 輸入的資料會分割成獨立的區塊。 每個區塊會在叢集的節點之間平行處理。 MapReduce 工作由兩項功能組成：

* **對應程式**：取用輸入資料、分析 (通常使用篩選及排序作業)，以及發出 Tuple (機碼值組)

* **減壓器**：取用對應程式發出的 Tuple 並執行摘要作業，從對應程式資料建立較小的組合結果

下圖說明了基本字數統計 MapReduce 工作範例：

![HDI.WordCountDiagram][image-hdi-wordcountdiagram]

此工作的輸出是文字中每個單字出現的次數統計。

* 對應程式從輸入的文字中取得每一行當作一個輸入，然後將其打散成單字。 對應程式會在單字出現時發出機碼/值組，並在該字之後加上 1。 輸出會先經過排序再傳送至減壓器。
* 接著，減壓器會加總每個單字的個別計數並發出單一機碼/值組，其中包含該單字和尾隨在後的出現次數總和。

MapReduce 可在各種語言中實作。 Java 是最常見的實作，基於示範目的用於此文件中。

## <a name="development-languages"></a>開發語言

以 Java 及 Java Virtual Machine 為基礎的語言或架構可以 MapReduce 工作的形式直接執行。 本文件中使用的範例是 Java MapReduce 應用程式。 非 Java 語言 (例如 C#、Python 或獨立可執行檔) 必須使用 **Hadoop 串流**。

Hadoop 串流會透過 STDIN 與 STDOUT 與對應工具和歸納工具進行通訊。 對應工具和歸納工具會一次從 STDIN 讀取一行資料，並將輸出寫入至 STDOUT。 對應工具和歸納工具所讀取或發出的每行資料，必須為機碼/值組格式，並以索引標籤字元分隔：

    [key]/t[value]

如需詳細資訊，請參閱 [Hadoop 資料流](http://hadoop.apache.org/docs/r1.2.1/streaming.html)(英文)。

如需搭配 HDInsight 使用 Hadoop 資料流的範例，請參閱下列文件：

* [開發 C# MapReduce 工作](apache-hadoop-dotnet-csharp-mapreduce-streaming.md)

* [開發 Python MapReduce 工作](apache-hadoop-streaming-python.md)

## <a id="data"></a>範例資料

HDInsight 提供各種範例資料集，這些範例資料及儲存在 `/example/data` 和 `/HdiSamples` 目錄。 這些目錄位於您叢集的預設儲存體中。 在本文件中，我們使用 `/example/data/gutenberg/davinci.txt` 檔案。 這個檔案包含達文西手稿筆記。

## <a id="job"></a>範例 MapReduce

範例 MapReduce 字數統計應用程式會包含您的 HDInsight 叢集。 此範例位於您叢集的預設儲存體上的 `/example/jars/hadoop-mapreduce-examples.jar`。

下列 Java 程式碼是 `hadoop-mapreduce-examples.jar` 檔案中包含的 MapReduce 應用程式原始碼︰

```java
package org.apache.hadoop.examples;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class WordCount {

    public static class TokenizerMapper
        extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
        StringTokenizer itr = new StringTokenizer(value.toString());
        while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
        }
    }
    }

    public static class IntSumReducer
        extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                        Context context
                        ) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
        sum += val.get();
        }
        result.set(sum);
        context.write(key, result);
    }
    }

    public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
    if (otherArgs.length != 2) {
        System.err.println("Usage: wordcount <in> <out>");
        System.exit(2);
    }
    Job job = new Job(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
    FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

如須撰寫您自己的 MapReduce 應用程式相關的指示，請參閱下列文件︰

* [開發 HDInsight 的 Java MapReduce 應用程式](apache-hadoop-develop-deploy-java-mapreduce-linux.md)

* [開發 HDInsight 的 Python MapReduce 應用程式](apache-hadoop-streaming-python.md)

## <a id="run"></a>執行 MapReduce

HDInsight 可以使用各種方法執行 HiveQL 工作。 請使用下表決定適合您的方法，然後跟著連結逐項閱讀介紹。

| **使用此方法**... | **...執行此工作** | ...搭配此 **叢集作業系統** | ...從此 **用戶端作業系統** |
|:--- |:--- |:--- |:--- |
| [SSH](apache-hadoop-use-mapreduce-ssh.md) |透過 **SSH** |Linux |Linux、Unix、Mac OS X 或 Windows |
| [Curl](apache-hadoop-use-mapreduce-curl.md) |使用 **REST** |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [Windows PowerShell](apache-hadoop-use-mapreduce-powershell.md) |使用 **Windows PowerShell** |Linux 或 Windows |Windows |

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a id="nextsteps"></a>後續步驟

若要深入了解如何處理 HDInsight 的資料，請參閱下列文件：

* [開發 HDInsight 的 Java MapReduce 程式](apache-hadoop-develop-deploy-java-mapreduce-linux.md)

* [開發 HDInsight 的 Python 資料流 MapReduce 程式](apache-hadoop-streaming-python.md)

* [搭配 HDInsight 使用 Hivet][hdinsight-use-hive]

* [搭配 HDInsight 使用 Pig][hdinsight-use-pig]


[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-get-started]:apache-hadoop-linux-tutorial-get-started.md
[hdinsight-develop-mapreduce-jobs]: apache-hadoop-develop-deploy-java-mapreduce-linux.md
[hdinsight-use-hive]:../hdinsight-use-hive.md
[hdinsight-use-pig]:hdinsight-use-pig.md


[powershell-install-configure]: /powershell/azureps-cmdlets-docs

[image-hdi-wordcountdiagram]: ./media/hdinsight-use-mapreduce/HDI.WordCountDiagram.gif
