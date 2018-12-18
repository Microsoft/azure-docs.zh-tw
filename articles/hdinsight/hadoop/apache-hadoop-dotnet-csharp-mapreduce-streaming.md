---
title: 搭配 HDInsight 的 Hadoop 上的 MapReduce 使用 C# - Azure
description: 了解如何搭配 Azure HDInsight 的 Hadoop 使用 C# 以建立 MapReduce 方案。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: jasonh
ms.openlocfilehash: 91558ae4fd592c7490aef0a0f2fd930e970f498c
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049058"
---
# <a name="use-c-with-mapreduce-streaming-on-hadoop-in-hdinsight"></a>搭配 HDInsight 的 Hadoop 上的 MapReduce 串流使用 C#

了解如何使用 C# 在 HDInsight 上建立 MapReduce 方案。

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [HDInsight 元件版本設定](../hdinsight-component-versioning.md)。

Hadoop 串流是一個公用程式，可讓您使用指令碼或可執行檔執行 MapReduce 作業。 在此範例中，.NET 用來實作字數統計方案的對應工具和歸納器。

## <a name="net-on-hdinsight"></a>HDInsight 上的 .NET

__以 Linux 為基礎的 HDInsight__ 叢集會使用 [Mono (https://mono-project.com)](https://mono-project.com) 來執行 .NET 應用程式。 4.2.1 版的 Mono 隨附於 3.6 版的 HDInsight。 如需 HDInsight 包含之 Mono 版本的詳細資訊，請參閱 [HDInsight 元件版本](../hdinsight-component-versioning.md)。 若要使用特定版本的 Mono，請參閱[安裝或更新 Mono](../hdinsight-hadoop-install-mono.md) 文件。

如需 Mono 與 .NET Framework 版本之相容性的詳細資訊，請參閱 [Mono 相容性](http://www.mono-project.com/docs/about-mono/compatibility/) \(英文\)。

## <a name="how-hadoop-streaming-works"></a>Hadoop 串流的運作方式

本文件中使用於串流的基本程序如下︰

1. Hadoop 在 STDIN 上將資料傳遞至對應工具 (在此範例中是 mapper.exe)。
2. 對應工具會處理資料，然後發出以 Tab 分隔的機碼值組到 STDOUT。
3. Hadoop 會讀取輸出，接著輸出會在 STDIN 上傳遞至歸納器 (在此範例中是 reducer.exe)。
4. 歸納器會讀取以 Tab 分隔的機碼值組、處理資料，然後在 STDOUT 發出以 Tab 分隔的機碼值組格式結果。
5. Hadoop 會讀取輸出，接著輸出會寫入至輸出目錄。

如需有關串流的詳細資訊，請參閱 [Hadoop 串流](https://hadoop.apache.org/docs/r2.7.1/hadoop-streaming/HadoopStreaming.html) \(英文\)。

## <a name="prerequisites"></a>必要條件

* 熟悉如何撰寫和建置以 .NET Framework 4.5 為目標的 C# 程式碼。 本文件中的步驟使用 Visual Studio 2017。

* 將 .exe 檔案上傳至叢集的方式。 本文件中的步驟使用 Data Lake Tools for Visual Studio 將檔案上傳至叢集的主要儲存體。

* Azure PowerShell 或 SSH 用戶端。

* HDInsight 叢集上的 Hadoop。 如需有關建立叢集的詳細資訊，請參閱[建立 HDInsight 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。

## <a name="create-the-mapper"></a>建立對應工具

在 Visual Studio 中，建立名為 __mapper__ 的新__主控台應用程式__。 針對此應用程式使用下列程式碼：

```csharp
using System;
using System.Text.RegularExpressions;

namespace mapper
{
    class Program
    {
        static void Main(string[] args)
        {
            string line;
            //Hadoop passes data to the mapper on STDIN
            while((line = Console.ReadLine()) != null)
            {
                // We only want words, so strip out punctuation, numbers, etc.
                var onlyText = Regex.Replace(line, @"\.|;|:|,|[0-9]|'", "");
                // Split at whitespace.
                var words = Regex.Matches(onlyText, @"[\w]+");
                // Loop over the words
                foreach(var word in words)
                {
                    //Emit tab-delimited key/value pairs.
                    //In this case, a word and a count of 1.
                    Console.WriteLine("{0}\t1",word);
                }
            }
        }
    }
}
```

建立應用程式之後，建置它以在專案目錄中產生 `/bin/Debug/mapper.exe` 檔案。

## <a name="create-the-reducer"></a>建立歸納器

在 Visual Studio 中，建立名為 __reducer__ 的新__主控台應用程式__。 針對此應用程式使用下列程式碼：

```csharp
using System;
using System.Collections.Generic;

namespace reducer
{
    class Program
    {
        static void Main(string[] args)
        {
            //Dictionary for holding a count of words
            Dictionary<string, int> words = new Dictionary<string, int>();

            string line;
            //Read from STDIN
            while ((line = Console.ReadLine()) != null)
            {
                // Data from Hadoop is tab-delimited key/value pairs
                var sArr = line.Split('\t');
                // Get the word
                string word = sArr[0];
                // Get the count
                int count = Convert.ToInt32(sArr[1]);

                //Do we already have a count for the word?
                if(words.ContainsKey(word))
                {
                    //If so, increment the count
                    words[word] += count;
                } else
                {
                    //Add the key to the collection
                    words.Add(word, count);
                }
            }
            //Finally, emit each word and count
            foreach (var word in words)
            {
                //Emit tab-delimited key/value pairs.
                //In this case, a word and a count of 1.
                Console.WriteLine("{0}\t{1}", word.Key, word.Value);
            }
        }
    }
}
```

建立應用程式之後，建置它以在專案目錄中產生 `/bin/Debug/reducer.exe` 檔案。

## <a name="upload-to-storage"></a>上傳至儲存體

1. 在 Visual Studio 中，開啟 [伺服器總管]。

2. 展開 [Azure]，然後展開 [HDInsight]。

3. 如果出現提示，請輸入您的 Azure 訂用帳戶認證，然後按一下 [登入]。

4. 展開您要部署此應用程式的 HDInsight 叢集。 就會列出含有文字 __(預設儲存體帳戶)__ 的項目。

    ![顯示叢集之儲存體帳戶的 [伺服器總管]](./media/apache-hadoop-dotnet-csharp-mapreduce-streaming/storage.png)

    * 如果此項目可以展開，表示您是使用 __Azure 儲存體帳戶__作為叢集的預設儲存體。 若要檢視叢集之預設儲存體上的檔案，請展開項目，然後按兩下 [(預設容器)]。

    * 如果此項目無法展開，表示您是使用 __Azure Data Lake Store__ 作為叢集的預設儲存體。 若要檢視叢集之預設儲存體上的檔案，請按兩下 [(預設儲存體帳戶)] 項目。

5. 若要上傳 .exe 檔案，請使用下列其中一種方法：

    * 如果使用 __Azure 儲存體帳戶__，請按一下上傳圖示，然後瀏覽至 **mapper** 專案的 **bin\debug** 資料夾。 最後，選取 **mapper.exe** 檔案，然後按一下 [確定]。

        ![上傳圖示](./media/apache-hadoop-dotnet-csharp-mapreduce-streaming/upload.png)
    
    * 如果使用 __Azure Data Lake Store__，請以滑鼠右鍵按一下檔案清單中的空白區域，然後選取 [上傳]。 最後，選取 **mapper.exe** 檔案，然後按一下 [開啟]。

    __mapper.exe__ 上傳完成後，請針對 __reducer.exe__ 檔案重複上傳程序。

## <a name="run-a-job-using-an-ssh-session"></a>執行工作︰使用 SSH 工作階段

1. 使用 SSH 連接到 HDInsight 叢集。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 使用下列其中一個命令來啟動 MapReduce 作業：

    * 如果使用 __Data Lake Store__ 作為預設儲存體：

        ```bash
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar -files adl:///mapper.exe,adl:///reducer.exe -mapper mapper.exe -reducer reducer.exe -input /example/data/gutenberg/davinci.txt -output /example/wordcountout
        ```
    
    * 如果使用 __Azure 儲存體__作為預設儲存體：

        ```bash
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar -files wasb:///mapper.exe,wasb:///reducer.exe -mapper mapper.exe -reducer reducer.exe -input /example/data/gutenberg/davinci.txt -output /example/wordcountout
        ```

    下列清單描述每個參數的用途︰

    * `hadoop-streaming.jar`︰包含串流處理 MapReduce 功能的 jar 檔案。
    * `-files`︰將 `mapper.exe` 和 `reducer.exe` 檔案加入至此作業。 每個檔案前面的 `adl:///` 或 `wasb:///` 是叢集的預設儲存體根目錄路徑。
    * `-mapper`︰指定哪個檔案實作對應工具。
    * `-reducer`︰指定哪個檔案實作歸納器。
    * `-input`：輸入資料。
    * `-output`︰輸出目錄。

3. 在 MapReduce 作業完成後，使用下列命令來檢視結果：

    ```bash
    hdfs dfs -text /example/wordcountout/part-00000
    ```

    下列文字是此命令所傳回資料的範例︰

        you     1128
        young   38
        younger 1
        youngest        1
        your    338
        yours   4
        yourself        34
        yourselves      3
        youth   17

## <a name="run-a-job-using-powershell"></a>執行作業：使用 PowerShell

使用下列 PowerShell 指令碼來執行 MapReduce 作業並下載結果。

[!code-powershell[main](../../../powershell_scripts/hdinsight/use-csharp-mapreduce/use-csharp-mapreduce.ps1?range=5-87)]

此指令碼會提示您輸入叢集登入帳戶名稱和密碼，以及 HDInsight 叢集名稱。 作業完成後，會將輸出下載到名為 `output.txt` 的檔案。 以下文字是 `output.txt` 檔案中的資料範例：

    you     1128
    young   38
    younger 1
    youngest        1
    your    338
    yours   4
    yourself        34
    yourselves      3
    youth   17

## <a name="next-steps"></a>後續步驟

如需搭配 HDInsight 使用 MapReduce 的詳細資訊，請參閱[搭配 HDInsight 使用 MapReduce](hdinsight-use-mapreduce.md)。

如需有關使用 C# 搭配 Hive 和 Pig 的詳細資訊，請參閱[使用 C# 使用者定義函式搭配 Hive 和 Pig](apache-hadoop-hive-pig-udf-dotnet-csharp.md)。

如需搭配 HDInsight 上的 Storm 使用 C# 的詳細資訊，請參閱[開發適用於 Storm on HDInsight 的 C# 拓撲](../storm/apache-storm-develop-csharp-visual-studio-topology.md)。
