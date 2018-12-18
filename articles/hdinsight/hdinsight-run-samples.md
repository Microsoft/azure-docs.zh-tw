---
title: 在 HDInsight 上執行 Hadoop 範例 - Azure
description: 利用提供的範例開始使用 Azure HDInsight 服務。 使用 PowerShell 指令碼在資料叢集上執行 MapReduce 程式。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/25/2017
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: 59e6fd0a05374329efb20201c8d904cbbdc72b1a
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43110994"
---
# <a name="run-hadoop-mapreduce-samples-in-windows-based-hdinsight"></a>在以 Windows 為基礎的 HDInsight 中執行 Hadoop MapReduce 範例
[!INCLUDE [samples-selector](../../includes/hdinsight-run-samples-selector.md)]

我們提供了一組範例，協助您使用 Azure HDInsight 並開始在 Hadoop 叢集上執行 MapReduce 工作。 這些範例可套用在您所建立的每個 HDInsight 受控叢集上。 執行這些範例可協助您熟悉使用 Azure PowerShell Cmdlet 在 Hadoop 叢集上執行作業。

* [字數統計][hdinsight-sample-wordcount]：計算文字檔中的文字出現次數。
* [C# 串流字數統計][hdinsight-sample-csharp-streaming]：使用 Hadoop 串流介面計算文字檔中的文字出現次數。
* [Pi 估算器][hdinsight-sample-pi-estimator]：使用統計 (擬蒙特卡羅法) 方法來估計 Pi 的值。
* [10-GB Graysort][hdinsight-sample-10gb-graysort]：使用 HDInsight 在 10 GB 檔案上執行一般用途的 GraySort。 有三個工作可執行：Teragen、Terasort 和 Teravalidate，分別用來產生資料、排序資料，以及確認資料已適當排序。

> [!NOTE]
> 原始程式碼可以在附錄中找到。

網路上有許多 Hadoop 相關技術 (例如 Java 型 MapReduce 程式設計和串流) 的文件可供參考，此外也有適用於 Windows PowerShell 指令碼之 Cmdlet 的相關文件。 如需有關這些資源的詳細資訊，請參閱：

* [在 HDInsight 上開發 Hadoop 的 Java MapReduce 程式](hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md)
* [在 HDInsight 上提交 Hadoop 工作](hadoop/submit-apache-hadoop-jobs-programmatically.md)
* [Azure HDInsight 簡介][hdinsight-introduction]

時至今日，很多人選擇 Hive 和 Pig 而非 MapReduce。  如需詳細資訊，請參閱

* [在 HDInsight 中使用 Hive](hadoop/hdinsight-use-hive.md)
* [在 HDInsight 中使用 Pig](hadoop/hdinsight-use-pig.md)

**必要條件**：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **HDInsight 叢集**。 如需可建立此類叢集之各種方式的指示，請參閱 [在 HDInsight 中使用 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md)。
* **具有 Azure PowerShell 的工作站**。

    > [!IMPORTANT]
    > 使用 Azure Service Manager 管理 HDInsight 資源的 Azure PowerShell 支援已**被取代**，將會在 2017 年 1 月 1 日前移除。 本文件中的步驟會使用可與 Azure Resource Manager 搭配使用的新 HDInsight Cmdlet。
    >
    > 請遵循[安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs) 中的步驟來安裝最新版的 Azure PowerShell。 如果您需要修改指令碼才能使用適用於 Azure Resource Manager 的新 Cmdlet，請參閱[移轉至以 Azure Resource Manager 為基礎的開發工具 (適用於 HDInsight 叢集)](hdinsight-hadoop-development-using-azure-resource-manager.md)。

## <a name="hdinsight-sample-wordcount"></a>字數統計 - Java
如果要提交 MapReduce 專案，您可以先建立 MapReduce 工作定義。 在工作定義中，您指定 MapReduce 程式 jar 檔案和該 jar 檔案的位置，這會是 **wasb:///example/jars/hadoop-mapreduce-examples.jar**、類別名稱和引數。  字數統計 MapReduce 程式會採用兩個引數：用來統計字數的原始程式檔與輸出的位置。

原始程式碼可以在 [附錄 A](#apendix-a---the-word-count-MapReduce-program-in-java)中找到。

如需開發 Java MapReduce 程式之程序，請參閱 - [在 HDInsight 中針對 Hadoop 開發 Java MapReduce 程式](hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md)

**提交字數統計 MapReduce 工作**

1. 開啟 **Windows PowerShell ISE**。 如需指示，請參閱[安裝和設定 Azure PowerShell][powershell-install-configure]。
2. 貼上下列 PowerShell 指令碼：

    ```powershell
    $subscriptionName = "<Azure Subscription Name>"
    $resourceGroupName = "<Resource Group Name>"
    $clusterName = "<HDInsight cluster name>"             # HDInsight cluster name

    Select-AzureRmSubscription -SubscriptionName $subscriptionName

    # Define the MapReduce job
    $mrJobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
                                -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" `
                                -ClassName "wordcount" `
                                -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput"

    # Submit the job and wait for job completion
    $cred = Get-Credential -Message "Enter the HDInsight cluster HTTP user credential:"
    $mrJob = Start-AzureRmHDInsightJob `
                        -ResourceGroupName $resourceGroupName `
                        -ClusterName $clusterName `
                        -HttpCredential $cred `
                        -JobDefinition $mrJobDefinition

    Wait-AzureRmHDInsightJob `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $clusterName `
        -HttpCredential $cred `
        -JobId $mrJob.JobId

    # Get the job output
    $cluster = Get-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $clusterName
    $defaultStorageAccount = $cluster.DefaultStorageAccount -replace '.blob.core.windows.net'
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccount)[0].Value
    $defaultStorageContainer = $cluster.DefaultStorageContainer

    Get-AzureRmHDInsightJobOutput `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $clusterName `
        -HttpCredential $cred `
        -DefaultStorageAccountName $defaultStorageAccount `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultContainer $defaultStorageContainer  `
        -JobId $mrJob.JobId `
        -DisplayOutputType StandardError

    # Download the job output to the workstation
    $storageContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccount -StorageAccountKey $defaultStorageAccountKey
    Get-AzureStorageBlobContent -Container $defaultStorageContainer -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

    # Display the output file
    cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"
    ```

    MapReduce 工作會產生一個名為 *part-r-00000*的檔案，內有文字和字數。 指令碼使用 **findstr** 命令列出包含 "there" 的所有文字。
3. 設定前三個變數，然後執行指令碼。

## <a name="hdinsight-sample-csharp-streaming"></a>字數統計 - C# 串流
Hadoop 提供 MapReduce 一個串流 API，可讓您以 Java 以外的語言撰寫 map 和 reduce 函數。

> [!NOTE]
> 本教學課程的步驟只適用於 Windows HDInsight 叢集。 如需 Linux HDInsight 叢集的串流範例，請參閱 [開發適用於 HDInsight 的 Python 串流程式](hadoop/apache-hadoop-streaming-python.md)。

在範例中，mapper 和 reducer 是從 [stdin][stdin-stdout-stderr] 讀取輸入 (循行) 並將輸出發出到 [stdout][stdin-stdout-stderr] 的可執行檔。 程式會計算內容中的所有文字。

在已為 **mappers**指定可執行檔的情況下，當 mapper 初始化時，每個 mapper 工作都會將可執行檔啟動成為個別的處理程序。 當 mapper 工作執行時，它會將輸入傳換成行，並將這些行饋送至處理程序的 [stdin][stdin-stdout-stderr]。

同時，mapper 會收集來自處理程序 stdout 的行導向輸出。 它會將每一行轉換成索引鍵/值組，其會做為 mapper 的輸出來收集。 根據預設，從一行的前置詞一直到第一個定位字元即是索引鍵，行的其餘部分 (不包含定位字元) 則為值。 如果行中沒有定位字元，則整行都會被視為索引鍵，而值則為 null。

在已為 **reducers**指定可執行檔的情況下，當 reducer 初始化時，每個 reducer 工作都會將可執行檔啟動成為個別的處理程序。 當 reducer 工作執行時，它會將輸入索引鍵/值組傳換成行，並將這些行饋送至處理程序的 [stdin][stdin-stdout-stderr]。

同時，reducer 會收集來自處理程序 [stdout][stdin-stdout-stderr] 的行導向輸出。 它會將每一行轉換成索引鍵/值組，其會做為 reducer 的輸出來收集。 根據預設，從一行的前置詞一直到第一個定位字元即是索引鍵，行的其餘部分 (不包含定位字元) 則為值。

**提交 C# 串流字數統計工作**

* 遵循[字數統計 - Java](#word-count-java) 中的程序，並使用下行內容取代作業定義：

    ```powershell
    $mrJobDefinition = New-AzureRmHDInsightStreamingMapReduceJobDefinition `
                            -Files "/example/apps/cat.exe","/example/apps/wc.exe" `
                            -Mapper "cat.exe" `
                            -Reducer "wc.exe" `
                            -InputPath "/example/data/gutenberg/davinci.txt" `
                            -OutputPath "/example/data/StreamingOutput/wc.txt"
    ```

    輸出檔案應該為：

        example/data/StreamingOutput/wc.txt/part-00000

## <a name="hdinsight-sample-pi-estimator"></a>PI 估算器
Pi 估算器會使用統計 (擬蒙特卡羅法) 方法來估計 pi 的值。 單位正方形內隨機散佈的點，也會落在該正方形的內切圓之內，且機率等於圓面積 Pi/4。 Pi 的值可從 4R 的值來估計，其中 R 是圓內點數佔正方形內總點數的比例。 使用的樣本點越多，估計越準確。

此範例的提供指令碼會提交 Hadoop jar 工作，且設定為以 16 個對應的值來執行，每個對應都必須依參數值來計算 1 千萬個樣本點。 這些參數可變更來改善 Pi 的估計值。 Pi 的前 10 位小數是 3.1415926535，供您參考。

**提交 Pi 估算器工作**

* 遵循[字數統計 - Java](#word-count-java) 中的程序，並使用下行內容取代作業定義：

    ```powershell
    $mrJobJobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
                                -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" `
                                -ClassName "pi" `
                                -Arguments "16", "10000000"
    ```

## <a name="hdinsight-sample-10gb-graysort"></a>10-GB Graysort
本範例使用不太大的 10GB 資料，所以執行起來相對較快。 本範例使用 Owen O'Malley 和 Arun Murthy 所開發的 MapReduce 應用程式，此應用程式於 2009 年的年度一般目的 (「耐力賽」) TB 排序效能評定中，以 0.578TB/分鐘 (173 分鐘內達到 100TB) 的速率獲勝。 如需此效能評比和其他排序效能評比的詳細資訊，請參閱 [Sortbenchmark](http://sortbenchmark.org/) 網站。

本範例使用三組 MapReduce 程式：

1. **TeraGen** 是可用來產生排序資料列的 MapReduce 程式。
2. **TeraSort** 可取樣輸入資料並利用 MapReduce 將資料依全序排列。 TeraSort 是 MapReduce 函數的標準排序，但自訂分割器除外，它使用 N-1 個樣本索引鍵的排序清單來定義每次歸納的索引鍵範圍。 尤其是，會傳送使得 sample[i-1] <= key < sample[i] 的所有索引鍵給歸納 i。 這保證歸納 i 的輸出全都小於歸納 i+1 的輸出。
3. **TeraValidate** 是一個驗證全域排序輸出的 MapReduce 程式。 它會在輸出目錄中為每一個檔案建立一個對應，而每個對應可確保每一個索引鍵一定小於或等於前一個對應。 對應函數也會產生每個檔案的第一個和最後一個索引鍵的記錄，而歸納函數可確保檔案 i 的第一個索引鍵大於檔案 i-1 的最後一個索引鍵。 任何問題皆會回報為具錯誤索引鍵的歸納輸出。

這三個應用程式所使用的輸入和輸出格式會以正確格式讀取和寫入文字檔。 歸納的輸出將複寫設為 1，而不是預設值 3，因為效能評定競賽不需要將輸出資料複寫至多個節點。

範例需要三項工作，各對應至簡介中描述的其中一個 MapReduce 程式：

1. 執行 **TeraGen** MapReduce 工作來產生要排序的資料。
2. 執行 **TeraSort** MapReduce 工作來排序資料。
3. 執行 **TeraValidate** MapReduce 工作來確認資料排序正確。

**提交工作**

* 遵循[字數統計 - Java](#word-count-java) 中的程序，並使用下列工作定義：

    ```powershell
    $teragen = New-AzureRmHDInsightMapReduceJobDefinition `
                                -JarFile "/example/jars/hadoop-mapreduce-examples.jar" `
                                -ClassName "teragen" `
                                -Arguments "-Dmapred.map.tasks=50", "100000000", "/example/data/10GB-sort-input"

    $terasort = New-AzureRmHDInsightMapReduceJobDefinition `
                                -JarFile "/example/jars/hadoop-mapreduce-examples.jar" `
                                -ClassName "terasort" `
                                -Arguments "-Dmapred.map.tasks=50", "-Dmapred.reduce.tasks=25", "/example/data/10GB-sort-input", "/example/data/10GB-sort-output"

    $teravalidate = New-AzureRmHDInsightMapReduceJobDefinition `
                                -JarFile "/example/jars/hadoop-mapreduce-examples.jar" `
                                -ClassName "teravalidate" `
                                -Arguments "-Dmapred.map.tasks=50", "-Dmapred.reduce.tasks=25", "/example/data/10GB-sort-output", "/example/data/10GB-sort-validate"
    ```

## <a name="next-steps"></a>後續步驟
透過本文和關於各範例的文章，您已了解如何使用 Azure PowerShell 執行 HDInsight 叢集隨附的範例。 如需透過 HDInsight 使用 Pig、Hive 和 MapReduce 的教學課程，請參閱下列主題：

* [開始在 HDInsight 中搭配 Hive 使用 Hadoop 以分析行動電話使用][hdinsight-get-started]
* [搭配使用 Pig 與 HDInsight 上的 Hadoop][hdinsight-use-pig]
* [搭配使用 Hive 與 HDInsight 上的 Hadoop][hdinsight-use-hive]
* [在 HDInsight 中提交 Hadoop 作業][hdinsight-submit-jobs]
* [Azure HDInsight SDK 文件][hdinsight-sdk-documentation]

## <a name="appendix-a---the-word-count-source-code"></a>附錄 A - 字數統計原始程式碼

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

## <a name="appendix-b---the-word-count-streaming-source-code"></a>附錄 B - 字數統計串流原始程式碼
MapReduce 程式使用 cat.exe 應用程式做為對應介面以將文字串流至主控台，並使用 wc.exe 應用程式做為縮減介面以計算從文件串流的字數。 mapper 和 reducer 都會從標準輸入資料流 (stdin) 逐行讀取字元並寫入至標準輸出資料流 (stdout)。

```csharp
// The source code for the cat.exe (Mapper).

using System;
using System.IO;

namespace cat
{
    class cat
    {
        static void Main(string[] args)
        {
            if (args.Length > 0)
            {
                Console.SetIn(new StreamReader(args[0]));
            }

            string line;
            char[] separators = { ' ', '\n'};
            while ((line = Console.ReadLine()) != null)
            {
                string[] words = line.Split(separators);
                foreach (var word in words)
                {
                    Console.WriteLine("{0}\t1", word);
                }
            }
        }
    }
}
```

cat.cs 檔案中的 mapper 程式碼會使用 [StreamReader][streamreader] 物件，將連入串流的字元讀取至主控台，再由主控台以靜態 [Console.Writeline][console-writeline] 方法將串流寫入標準輸出串流。

```csharp
// The source code for wc.exe (Reducer) is:

using System;
using System.IO;
using System.Linq;
using System.Collections;

namespace wc
{
    class wc
    {
        static void Main(string[] args)
        {
            string line;

            if (args.Length > 0)
            {
                Console.SetIn(new StreamReader(args[0]));
            }

            Hashtable wordCount = new Hashtable();
            while ((line = Console.ReadLine()) != null)
            {
                string[] words = line.Split('\t');

                string key = words[0];

                if (wordCount.ContainsKey(key) == true)
                {
                    int n = Convert.ToInt32(wordCount[key]);
                    wordCount[key] = Convert.ToString(n + 1);
                }
                else
                {
                    wordCount[key] = words[1];
                }
            }
            foreach (var key in wordCount.Keys)
            {
                Console.WriteLine("{0} {1}", key, wordCount[key]);
            }
        }
    }
}
```

wc.cs 檔案中的 reducer 程式碼會使用 [StreamReader][streamreader] 物件，從已經由 cat.exe mapper 輸出的標準輸入串流讀取字元。 當它使用 [Console.Writeline][console-writeline] 方法讀取字元時，它會藉由計算空格和每個字尾端的行尾字元來計算字數。 然後使用 [Console.Writeline][console-writeline] 方法將總計寫入標準輸出串流。

## <a name="appendix-c---the-pi-estimator-source-code"></a>附錄 C - Pi 估算器原始程式碼
以下提供包含 mapper 和 reducer 函數的 Pi 估算器 Java 程式碼以進行檢查。 對應器程式會產生單位正方形內隨機散佈的一定點數，然後計算落在圓內的點數。 Reducer 程式會累計 mapper 所計算的點數，然後從公式 4R 估計 Pi 的值，其中 R 是圓內計算的點數佔正方形內總點數的比例。

```java
/**
* Licensed to the Apache Software Foundation (ASF) under one
* or more contributor license agreements. See the NOTICE file
* distributed with this work for additional information
* regarding copyright ownership. The ASF licenses this file
* to you under the Apache License, Version 2.0 (the
* "License"); you may not use this file except in compliance
* with the License. You may obtain a copy of the License at
*
* http://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or     implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/

package org.apache.hadoop.examples;

import java.io.IOException;
import java.math.BigDecimal;
import java.util.Iterator;

import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BooleanWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.SequenceFile;
import org.apache.hadoop.io.Writable;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.SequenceFile.CompressionType;
import org.apache.hadoop.mapred.FileInputFormat;
import org.apache.hadoop.mapred.FileOutputFormat;
import org.apache.hadoop.mapred.JobClient;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapred.MapReduceBase;
import org.apache.hadoop.mapred.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reducer;
import org.apache.hadoop.mapred.Reporter;
import org.apache.hadoop.mapred.SequenceFileInputFormat;
import org.apache.hadoop.mapred.SequenceFileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

//A Map-reduce program to estimate the value of Pi
//using quasi-Monte Carlo method.
//
//Mapper:
//Generate points in a unit square
//and then count points inside/outside of the inscribed circle of the square.
//
//Reducer:
//Accumulate points inside/outside results from the mappers.
//Let numTotal = numInside + numOutside.
//The fraction numInside/numTotal is a rational approximation of
//the value (Area of the circle)/(Area of the square),
//where the area of the inscribed circle is Pi/4
//and the area of unit square is 1.
//Then, Pi is estimated value to be 4(numInside/numTotal).
//

public class PiEstimator extends Configured implements Tool {
//tmp directory for input/output
static private final Path TMP_DIR = new Path(
PiEstimator.class.getSimpleName() + "_TMP_3_141592654");

//2-dimensional Halton sequence {H(i)},
//where H(i) is a 2-dimensional point and i >= 1 is the index.
//Halton sequence is used to generate sample points for Pi estimation.
private static class HaltonSequence {
// Bases
static final int[] P = {2, 3};
//Maximum number of digits allowed
static final int[] K = {63, 40};

private long index;
private double[] x;
private double[][] q;
private int[][] d;

//Initialize to H(startindex),
//so the sequence begins with H(startindex+1).
HaltonSequence(long startindex) {
index = startindex;
x = new double[K.length];
q = new double[K.length][];
d = new int[K.length][];
for(int i = 0; i < K.length; i++) {
q[i] = new double[K[i]];
d[i] = new int[K[i]];
}

for(int i = 0; i < K.length; i++) {
long k = index;
x[i] = 0;

for(int j = 0; j < K[i]; j++) {
q[i][j] = (j == 0? 1.0: q[i][j-1])/P[i];
d[i][j] = (int)(k % P[i]);
k = (k - d[i][j])/P[i];
x[i] += d[i][j] * q[i][j];
}
}
}

//Compute next point.
//Assume the current point is H(index).
//Compute H(index+1).
//@return a 2-dimensional point with coordinates in [0,1)^2
double[] nextPoint() {
index++;
for(int i = 0; i < K.length; i++) {
for(int j = 0; j < K[i]; j++) {
d[i][j]++;
x[i] += q[i][j];
if (d[i][j] < P[i]) {
break;
}
d[i][j] = 0;
x[i] -= (j == 0? 1.0: q[i][j-1]);
}
}
return x;
}
}

//Mapper class for Pi estimation.
//Generate points in a unit square and then
//count points inside/outside of the inscribed circle of the square.
public static class PiMapper extends MapReduceBase
implements Mapper<LongWritable, LongWritable, BooleanWritable, LongWritable> {

//Map method.
//@param offset samples starting from the (offset+1)th sample.
//@param size the number of samples for this map
//@param out output {true->numInside, false->numOutside}
//@param reporter
public void map(LongWritable offset,
LongWritable size,
OutputCollector<BooleanWritable, LongWritable> out,
Reporter reporter) throws IOException {

final HaltonSequence haltonsequence = new HaltonSequence(offset.get());
long numInside = 0L;
long numOutside = 0L;

for(long i = 0; i < size.get(); ) {
//generate points in a unit square
final double[] point = haltonsequence.nextPoint();

//count points inside/outside of the inscribed circle of the square
final double x = point[0] - 0.5;
final double y = point[1] - 0.5;
if (x*x + y*y > 0.25) {
numOutside++;
} else {
numInside++;
}

//report status
i++;
if (i % 1000 == 0) {
reporter.setStatus("Generated " + i + " samples.");
}
}

//output map results
out.collect(new BooleanWritable(true), new LongWritable(numInside));
out.collect(new BooleanWritable(false), new LongWritable(numOutside));
}
}

//Reducer class for Pi estimation.
//Accumulate points inside/outside results from the mappers.
public static class PiReducer extends MapReduceBase
implements Reducer<BooleanWritable, LongWritable, WritableComparable<?>, Writable> {

private long numInside = 0;
private long numOutside = 0;
private JobConf conf; //configuration for accessing the file system

//Store job configuration.
@Override
public void configure(JobConf job) {
conf = job;
}

// Accumulate number of points inside/outside results from the mappers.
// @param isInside Is the points inside?
// @param values An iterator to a list of point counts
// @param output dummy, not used here.
// @param reporter

public void reduce(BooleanWritable isInside,
Iterator<LongWritable> values,
OutputCollector<WritableComparable<?>, Writable> output,
Reporter reporter) throws IOException {
if (isInside.get()) {
for(; values.hasNext(); numInside += values.next().get());
} else {
for(; values.hasNext(); numOutside += values.next().get());
}
}

//Reduce task done, write output to a file.
@Override
public void close() throws IOException {
//write output to a file
Path outDir = new Path(TMP_DIR, "out");
Path outFile = new Path(outDir, "reduce-out");
FileSystem fileSys = FileSystem.get(conf);
SequenceFile.Writer writer = SequenceFile.createWriter(fileSys, conf,
outFile, LongWritable.class, LongWritable.class,
CompressionType.NONE);
writer.append(new LongWritable(numInside), new LongWritable(numOutside));
writer.close();
}
}

//Run a map/reduce job for estimating Pi.
//@return the estimated value of Pi.
public static BigDecimal estimate(int numMaps, long numPoints, JobConf jobConf
)
throws IOException {
//setup job conf
jobConf.setJobName(PiEstimator.class.getSimpleName());

jobConf.setInputFormat(SequenceFileInputFormat.class);

jobConf.setOutputKeyClass(BooleanWritable.class);
jobConf.setOutputValueClass(LongWritable.class);
jobConf.setOutputFormat(SequenceFileOutputFormat.class);

jobConf.setMapperClass(PiMapper.class);
jobConf.setNumMapTasks(numMaps);

jobConf.setReducerClass(PiReducer.class);
jobConf.setNumReduceTasks(1);

// turn off speculative execution, because DFS doesn't handle
// multiple writers to the same file.
jobConf.setSpeculativeExecution(false);

//setup input/output directories
final Path inDir = new Path(TMP_DIR, "in");
final Path outDir = new Path(TMP_DIR, "out");
FileInputFormat.setInputPaths(jobConf, inDir);
FileOutputFormat.setOutputPath(jobConf, outDir);

final FileSystem fs = FileSystem.get(jobConf);
if (fs.exists(TMP_DIR)) {
throw new IOException("Tmp directory " + fs.makeQualified(TMP_DIR)
+ " already exists. Remove it first.");
}
if (!fs.mkdirs(inDir)) {
throw new IOException("Cannot create input directory " + inDir);
}

//generate an input file for each map task
try {
for(int i=0; i < numMaps; ++i) {
final Path file = new Path(inDir, "part"+i);
final LongWritable offset = new LongWritable(i * numPoints);
final LongWritable size = new LongWritable(numPoints);
final SequenceFile.Writer writer = SequenceFile.createWriter(
fs, jobConf, file,
LongWritable.class, LongWritable.class, CompressionType.NONE);
try {
writer.append(offset, size);
} finally {
writer.close();
}
System.out.println("Wrote input for Map #"+i);
}

//start a map/reduce job
System.out.println("Starting Job");
final long startTime = System.currentTimeMillis();
JobClient.runJob(jobConf);
final double duration = (System.currentTimeMillis() - startTime)/1000.0;
System.out.println("Job Finished in " + duration + " seconds");

//read outputs
Path inFile = new Path(outDir, "reduce-out");
LongWritable numInside = new LongWritable();
LongWritable numOutside = new LongWritable();
SequenceFile.Reader reader = new SequenceFile.Reader(fs, inFile, jobConf);
try {
reader.next(numInside, numOutside);
} finally {
reader.close();
}

//compute estimated value
return BigDecimal.valueOf(4).setScale(20)
.multiply(BigDecimal.valueOf(numInside.get()))
.divide(BigDecimal.valueOf(numMaps))
.divide(BigDecimal.valueOf(numPoints));
} finally {
fs.delete(TMP_DIR, true);
}
}

//Parse arguments and then runs a map/reduce job.
//Print output in standard out.
//@return a non-zero if there is an error. Otherwise, return 0.
public int run(String[] args) throws Exception {
if (args.length != 2) {
System.err.println("Usage: "+getClass().getName()+" <nMaps> <nSamples>");
ToolRunner.printGenericCommandUsage(System.err);
return -1;
}

final int nMaps = Integer.parseInt(args[0]);
final long nSamples = Long.parseLong(args[1]);

System.out.println("Number of Maps = " + nMaps);
System.out.println("Samples per Map = " + nSamples);

final JobConf jobConf = new JobConf(getConf(), getClass());
System.out.println("Estimated value of Pi is "
+ estimate(nMaps, nSamples, jobConf));
return 0;
}

//main method for running it as a stand alone command.
public static void main(String[] argv) throws Exception {
System.exit(ToolRunner.run(null, new PiEstimator(), argv));
}
}
```

## <a name="appendix-d---the-10gb-graysort-source-code"></a>附錄 D - 10gb Graysort 原始程式碼
本節顯示 TeraSort MapReduce 程式的程式碼以進行檢查。

```java
/**
    * Licensed to the Apache Software Foundation (ASF) under one
    * or more contributor license agreements.  See the NOTICE file
    * distributed with this work for additional information
    * regarding copyright ownership.  The ASF licenses this file
    * to you under the Apache License, Version 2.0 (the
    * "License"); you may not use this file except in compliance
    * with the License.  You may obtain a copy of the License at
    *
    *     http://www.apache.org/licenses/LICENSE-2.0
    *
    * Unless required by applicable law or agreed to in writing, software
    * distributed under the License is distributed on an "AS IS" BASIS,
    * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    * See the License for the specific language governing permissions and
    * limitations under the License.
    */

package org.apache.hadoop.examples.terasort;

import java.io.IOException;
import java.io.PrintStream;
import java.net.URI;
import java.util.ArrayList;
import java.util.List;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.filecache.DistributedCache;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.SequenceFile;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.FileOutputFormat;
import org.apache.hadoop.mapred.JobClient;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapred.Partitioner;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

/**
    * Generates the sampled split points, launches the job,
    * and waits for it to finish.
    * <p>
    * To run the program:
    * <b>bin/hadoop jar hadoop-examples-*.jar terasort in-dir out-dir</b>
    */

public class TeraSort extends Configured implements Tool {
    private static final Log LOG = LogFactory.getLog(TeraSort.class);

    /**
    * A partitioner that splits text keys into roughly equal
    * partitions in a global sorted order.
    */

    static class TotalOrderPartitioner implements Partitioner<Text,Text>{
    private TrieNode trie;
    private Text[] splitPoints;

    /**
        * A generic trie node
        */
    static abstract class TrieNode {
        private int level;
        TrieNode(int level) {
        this.level = level;
        }
        abstract int findPartition(Text key);
        abstract void print(PrintStream strm) throws IOException;
        int getLevel() {
        return level;
        }
    }

    /**
        * An inner trie node that contains 256 children based on the next
        * character.
        */
    static class InnerTrieNode extends TrieNode {
        private TrieNode[] child = new TrieNode[256];

        InnerTrieNode(int level) {
        super(level);
        }
        int findPartition(Text key) {
        int level = getLevel();
        if (key.getLength() <= level) {
            return child[0].findPartition(key);
        }
        return child[key.getBytes()[level]].findPartition(key);
        }
        void setChild(int idx, TrieNode child) {
        this.child[idx] = child;
        }
        void print(PrintStream strm) throws IOException {
        for(int ch=0; ch < 255; ++ch) {
            for(int i = 0; i < 2*getLevel(); ++i) {
            strm.print(' ');
            }
            strm.print(ch);
            strm.println(" ->");
            if (child[ch] != null) {
            child[ch].print(strm);
            }
        }
        }
    }

    /**
        * A leaf trie node that does string compares to figure out where the given
        * key belongs between lower..upper.
        */
    static class LeafTrieNode extends TrieNode {
        int lower;
        int upper;
        Text[] splitPoints;
        LeafTrieNode(int level, Text[] splitPoints, int lower, int upper) {
        super(level);
        this.splitPoints = splitPoints;
        this.lower = lower;
        this.upper = upper;
        }
        int findPartition(Text key) {
        for(int i=lower; i<upper; ++i) {
            if (splitPoints[i].compareTo(key) >= 0) {
            return i;
            }
        }
        return upper;
        }
        void print(PrintStream strm) throws IOException {
        for(int i = 0; i < 2*getLevel(); ++i) {
            strm.print(' ');
        }
        strm.print(lower);
        strm.print(", ");
        strm.println(upper);
        }
    }

    /**
        * Read the cut points from the given sequence file.
        * @param fs the file system
        * @param p the path to read
        * @param job the job config
        * @return the strings to split the partitions on
        * @throws IOException
        */
    private static Text[] readPartitions(FileSystem fs, Path p,
                                            JobConf job) throws IOException {
        SequenceFile.Reader reader = new SequenceFile.Reader(fs, p, job);
        List<Text> parts = new ArrayList<Text>();
        Text key = new Text();
        NullWritable value = NullWritable.get();
        while (reader.next(key, value)) {
        parts.add(key);
        key = new Text();
        }
        reader.close();
        return parts.toArray(new Text[parts.size()]);
    }

    /**
        * Given a sorted set of cut points, build a trie that will find the correct
        * partition quickly.
        * @param splits the list of cut points
        * @param lower the lower bound of partitions 0..numPartitions-1
        * @param upper the upper bound of partitions 0..numPartitions-1
        * @param prefix the prefix that we have already checked against
        * @param maxDepth the maximum depth we will build a trie for
        * @return the trie node that will divide the splits correctly
        */
    private static TrieNode buildTrie(Text[] splits, int lower, int upper,
                                        Text prefix, int maxDepth) {
        int depth = prefix.getLength();
        if (depth >= maxDepth || lower == upper) {
        return new LeafTrieNode(depth, splits, lower, upper);
        }
        InnerTrieNode result = new InnerTrieNode(depth);
        Text trial = new Text(prefix);
        // append an extra byte on to the prefix
        trial.append(new byte[1], 0, 1);
        int currentBound = lower;
        for(int ch = 0; ch < 255; ++ch) {
        trial.getBytes()[depth] = (byte) (ch + 1);
        lower = currentBound;
        while (currentBound < upper) {
            if (splits[currentBound].compareTo(trial) >= 0) {
            break;
            }
            currentBound += 1;
        }
        trial.getBytes()[depth] = (byte) ch;
        result.child[ch] = buildTrie(splits, lower, currentBound, trial,
                                        maxDepth);
        }
        // pick up the rest
        trial.getBytes()[depth] = 127;
        result.child[255] = buildTrie(splits, currentBound, upper, trial,
                                    maxDepth);
        return result;
    }

    public void configure(JobConf job) {
        try {
        FileSystem fs = FileSystem.getLocal(job);
        Path partFile = new Path(TeraInputFormat.PARTITION_FILENAME);
        splitPoints = readPartitions(fs, partFile, job);
        trie = buildTrie(splitPoints, 0, splitPoints.length, new Text(), 2);
        } catch (IOException ie) {
        throw new IllegalArgumentException("can't read paritions file", ie);
        }
    }

    public TotalOrderPartitioner() {
    }

    public int getPartition(Text key, Text value, int numPartitions) {
        return trie.findPartition(key);
    }

    }

    public int run(String[] args) throws Exception {
    LOG.info("starting");
    JobConf job = (JobConf) getConf();
    Path inputDir = new Path(args[0]);
    inputDir = inputDir.makeQualified(inputDir.getFileSystem(job));
    Path partitionFile = new Path(inputDir, TeraInputFormat.PARTITION_FILENAME);
    URI partitionUri = new URI(partitionFile.toString() +
                                "#" + TeraInputFormat.PARTITION_FILENAME);
    TeraInputFormat.setInputPaths(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    job.setJobName("TeraSort");
    job.setJarByClass(TeraSort.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(Text.class);
    job.setInputFormat(TeraInputFormat.class);
    job.setOutputFormat(TeraOutputFormat.class);
    job.setPartitionerClass(TotalOrderPartitioner.class);
    TeraInputFormat.writePartitionFile(job, partitionFile);
    DistributedCache.addCacheFile(partitionUri, job);
    DistributedCache.createSymlink(job);
    job.setInt("dfs.replication", 1);
    TeraOutputFormat.setFinalSync(job, true);
    JobClient.runJob(job);
    LOG.info("done");
    return 0;
    }

    /**
    * @param args
    */

    public static void main(String[] args) throws Exception {
    int res = ToolRunner.run(new JobConf(), new TeraSort(), args);
    System.exit(res);
    }
}
```

[hdinsight-sdk-documentation]: https://msdn.microsoft.com/library/azure/dn479185.aspx

[hdinsight-submit-jobs]: hadoop/submit-apache-hadoop-jobs-programmatically.md
[hdinsight-introduction]:hadoop/apache-hadoop-introduction.md

[powershell-install-configure]: /powershell/azureps-cmdlets-docs

[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md

[hdinsight-samples]: hdinsight-run-samples.md
[hdinsight-sample-10gb-graysort]: #hdinsight-sample-10gb-graysort
[hdinsight-sample-csharp-streaming]: #hdinsight-sample-csharp-streaming
[hdinsight-sample-pi-estimator]: #hdinsight-sample-pi-estimator
[hdinsight-sample-wordcount]: #hdinsight-sample-wordcount

[hdinsight-use-hive]: hadoop/hdinsight-use-hive.md
[hdinsight-use-pig]: hadoop/hdinsight-use-pig.md

[streamreader]: http://msdn.microsoft.com/library/system.io.streamreader.aspx
[console-writeline]: http://msdn.microsoft.com/library/system.console.writeline
[stdin-stdout-stderr]: https://msdn.microsoft.com/library/3x292kth.aspx
