---
title: 搭配使用 C# 與 HDInsight 的 Hadoop 上的 Hive 和 Pig - Azure
description: 了解如何搭配使用 C# 使用者定義函數 (UDF) 與 Azure HDInsight 中的 Hive 和 Pig 串流處理。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: jasonh
ms.openlocfilehash: ebc4c918745e273700a0efb54db4408bb702c75e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46991363"
---
# <a name="use-c-user-defined-functions-with-hive-and-pig-streaming-on-hadoop-in-hdinsight"></a>搭配使用 C# 使用者定義函數與 HDInsight 的 Hadoop 上的 Hive 和 Pig 串流處理。

了解如何搭配使用 C# 使用者定義函式 (UDF) 與 HDInsight 上的 Apache Hive 和 Pig。

> [!IMPORTANT]
> 本文件中的步驟適用於以 Linux 為基礎和以 Windows 為基礎的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [HDInsight 元件版本設定](../hdinsight-component-versioning.md)。

Hive 和 Pig 都可以將資料傳遞至外部應用程式進行處理。 這個程序稱為_串流處理_。 使用 .NET 應用程式時，資料會在 STDIN 上傳遞至應用程式，而應用程式會在 STDOUT 上傳回結果。 為了在 STDIN 和 STDOUT 讀取和寫入，您可以從主控台應用程式使用 `Console.ReadLine()` 和 `Console.WriteLine()`。

## <a name="prerequisites"></a>必要條件

* 熟悉如何撰寫和建置以 .NET Framework 4.5 為目標的 C# 程式碼。

    * 使用您想要的任何 IDE。 建議您使用 [Visual Studio](https://www.visualstudio.com/vs) 2015、2017 或 [Visual Studio Code](https://code.visualstudio.com/)。 本文件中的步驟使用 Visual Studio 2017。

* 將 .exe 檔案上傳至叢集並執行 Pig 和 Hive 作業所採取的方式。 建議使用 Data Lake Tools for Visual Studio、Azure PowerShell 與 Azure 傳統 CLI。 此文件中的步驟使用 Data Lake Tools for Visual Studio 上傳檔案並執行範例 Hive 查詢。

    如需執行 Hive 查詢和 Pig 作業之其他方式的詳細資訊，請參閱下列文件︰

    * [搭配 HDInsight 使用 Apache Hive](hdinsight-use-hive.md)

    * [搭配 HDInsight 使用 Apache Pig](hdinsight-use-pig.md)

* HDInsight 叢集上的 Hadoop。 如需有關建立叢集的詳細資訊，請參閱[建立 HDInsight 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。

## <a name="net-on-hdinsight"></a>HDInsight 上的 .NET

* __以 Linux 為基礎的 HDInsight__ 叢集會使用 [Mono (https://mono-project.com)](https://mono-project.com) 來執行 .NET 應用程式。 4.2.1 版的 Mono 隨附於 3.6 版的 HDInsight。

    如需 Mono 與 .NET Framework 版本之相容性的詳細資訊，請參閱 [Mono 相容性](http://www.mono-project.com/docs/about-mono/compatibility/) \(英文\)。

    若要使用特定版本的 Mono，請參閱[安裝或更新 Mono](../hdinsight-hadoop-install-mono.md) 文件。

* __以 Windows 為基礎的 HDInsight__ 叢集使用 Microsoft.NET CLR 來執行.NET 應用程式。

如需 HDInsight 版本包含之 .NET Framework 和 Mono 版本的詳細資訊，請參閱 [HDInsight 元件版本](../hdinsight-component-versioning.md)。

## <a name="create-the-c-projects"></a>建立 C\# 專案

### <a name="hive-udf"></a>Hive UDF

1. 開啟 Visual Studio 並建立方案。 在專案類型中選取 [主控台應用程式 (.NET Framework)]，然後將新專案命名為 **HiveCSharp**。

    > [!IMPORTANT]
    > 如果您使用以 Linux 為基礎的 HDInsight 叢集，請選取 __.NET Framework 4.5__。 如需 Mono 與 .NET Framework 版本之相容性的詳細資訊，請參閱 [Mono 相容性](http://www.mono-project.com/docs/about-mono/compatibility/) \(英文\)。

2. 以下列程式碼取代 **Program.cs** 的內容：

    ```csharp
    using System;
    using System.Security.Cryptography;
    using System.Text;
    using System.Threading.Tasks;

    namespace HiveCSharp
    {
        class Program
        {
            static void Main(string[] args)
            {
                string line;
                // Read stdin in a loop
                while ((line = Console.ReadLine()) != null)
                {
                    // Parse the string, trimming line feeds
                    // and splitting fields at tabs
                    line = line.TrimEnd('\n');
                    string[] field = line.Split('\t');
                    string phoneLabel = field[1] + ' ' + field[2];
                    // Emit new data to stdout, delimited by tabs
                    Console.WriteLine("{0}\t{1}\t{2}", field[0], phoneLabel, GetMD5Hash(phoneLabel));
                }
            }
            /// <summary>
            /// Returns an MD5 hash for the given string
            /// </summary>
            /// <param name="input">string value</param>
            /// <returns>an MD5 hash</returns>
            static string GetMD5Hash(string input)
            {
                // Step 1, calculate MD5 hash from input
                MD5 md5 = System.Security.Cryptography.MD5.Create();
                byte[] inputBytes = System.Text.Encoding.ASCII.GetBytes(input);
                byte[] hash = md5.ComputeHash(inputBytes);

                // Step 2, convert byte array to hex string
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < hash.Length; i++)
                {
                    sb.Append(hash[i].ToString("x2"));
                }
                return sb.ToString();
            }
        }
    }
    ```

3. 建置專案。

### <a name="pig-udf"></a>Pig UDF

1. 開啟 Visual Studio 並建立方案。 在專案類型中選取 [主控台應用程式]，然後將新專案命名為 **PigUDF**。

2. 使用下列程式碼取代 **Program.cs** 檔案的內容：

    ```csharp
    using System;

    namespace PigUDF
    {
        class Program
        {
            static void Main(string[] args)
            {
                string line;
                // Read stdin in a loop
                while ((line = Console.ReadLine()) != null)
                {
                    // Fix formatting on lines that begin with an exception
                    if(line.StartsWith("java.lang.Exception"))
                    {
                        // Trim the error info off the beginning and add a note to the end of the line
                        line = line.Remove(0, 21) + " - java.lang.Exception";
                    }
                    // Split the fields apart at tab characters
                    string[] field = line.Split('\t');
                    // Put fields back together for writing
                    Console.WriteLine(String.Join("\t",field));
                }
            }
        }
    }
    ```

    此程式碼將剖析 Pig 送出的程式碼行，並將其重新格式化為以 `java.lang.Exception` 開頭的程式碼行。

3. 儲存 **Program.cs**，然後建置專案。

## <a name="upload-to-storage"></a>上傳至儲存體

1. 在 Visual Studio 中，開啟 [伺服器總管]。

2. 展開 [Azure]，然後展開 [HDInsight]。

3. 如果出現提示，請輸入您的 Azure 訂用帳戶認證，然後按一下 [登入]。

4. 展開您要部署此應用程式的 HDInsight 叢集。 就會列出含有文字 __(預設儲存體帳戶)__ 的項目。

    ![顯示叢集之儲存體帳戶的 [伺服器總管]](./media/apache-hadoop-hive-pig-udf-dotnet-csharp/storage.png)

    * 如果此項目可以展開，表示您是使用 __Azure 儲存體帳戶__作為叢集的預設儲存體。 若要檢視叢集之預設儲存體上的檔案，請展開項目，然後按兩下 [(預設容器)]。

    * 如果此項目無法展開，表示您是使用 __Azure Data Lake Store__ 作為叢集的預設儲存體。 若要檢視叢集之預設儲存體上的檔案，請按兩下 [(預設儲存體帳戶)] 項目。

6. 若要上傳 .exe 檔案，請使用下列其中一種方法：

    * 如果使用 __Azure 儲存體帳戶__，請按一下上傳圖示，然後瀏覽至 **HiveCSharp** 專案的 **bin\debug** 資料夾。 最後，選取 **HiveCSharp.exe** 檔案並按一下 [確定]。

        ![上傳圖示](./media/apache-hadoop-hive-pig-udf-dotnet-csharp/upload.png)
    
    * 如果使用 __Azure Data Lake Store__，請以滑鼠右鍵按一下檔案清單中的空白區域，然後選取 [上傳]。 最後，選取 **HiveCSharp.exe** 檔案並按一下 [開啟]。

    __HiveCSharp.exe__ 上傳完成後，請針對 __PigUDF.exe__ 檔案重複上傳程序。

## <a name="run-a-hive-query"></a>執行 HIVE 查詢

1. 在 Visual Studio 中，開啟 [伺服器總管]。

2. 展開 [Azure]，然後展開 [HDInsight]。

3. 以滑鼠右鍵按一下部署 **HiveCSharp** 應用程式的叢集，然後選取 [撰寫 Hive 查詢]。

4. 在 Hive 查詢中使用下列文字：

    ```hiveql
    -- Uncomment the following if you are using Azure Storage
    -- add file wasb:///HiveCSharp.exe;
    -- Uncomment the following if you are using Azure Data Lake Store
    -- add file adl:///HiveCSharp.exe;

    SELECT TRANSFORM (clientid, devicemake, devicemodel)
    USING 'HiveCSharp.exe' AS
    (clientid string, phoneLabel string, phoneHash string)
    FROM hivesampletable
    ORDER BY clientid LIMIT 50;
    ```

    > [!IMPORTANT]
    > 將和使用於您叢集之預設儲存體類型相符的 `add file` 陳述式取消註解。

    此查詢會從 `hivesampletable` 中選取 `clientid`、`devicemake` 和 `devicemodel` 欄位，然後將這些欄位傳遞給 HiveCSharp.exe 應用程式。 此查詢預期應用程式會傳回儲存為 `clientid`、`phoneLabel` 和 `phoneHash` 的三個欄位。 此查詢也預期會在預設儲存體容器的根目錄中找到 HiveCSharp.exe。

5. 按一下 [提交]，將工作提交至 HDInsight 叢集。 [Hive 作業摘要] 視窗隨即開啟。

6. 按一下 [重新整理] 以重新整理摘要，直到 [作業狀態] 變更為 [已完成] 為止。 若要檢視作業輸出，請按一下 [作業輸出]。

## <a name="run-a-pig-job"></a>執行 Pig 作業

1. 使用下列其中一種方法來連線到您的 HDInsight 叢集︰

    * 如果您使用__以 Linux 為基礎的__ HDInsight 叢集，請使用 SSH。 例如，`ssh sshuser@mycluster-ssh.azurehdinsight.net`。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)
    
    * 如果您使用__以 Windows 為基礎的__ HDInsight 叢集，請[使用遠端桌面連線到叢集](../hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp)

2. 使用下列其中一個命令來啟動 Pig 命令列：

        pig

    > [!IMPORTANT]
    > 如果您使用以 Windows 為基礎的叢集，請改為使用下列命令︰
    > ```
    > cd %PIG_HOME%
    > bin\pig
    > ```

    `grunt>` 提示隨即顯示。

3. 輸入下列命令，以執行使用 .NET Framework 應用程式的 Pig 作業：

        DEFINE streamer `PigUDF.exe` CACHE('/PigUDF.exe');
        LOGS = LOAD '/example/data/sample.log' as (LINE:chararray);
        LOG = FILTER LOGS by LINE is not null;
        DETAILS = STREAM LOG through streamer as (col1, col2, col3, col4, col5);
        DUMP DETAILS;

    `DEFINE` 陳述式會為 pigudf.exe 應用程式建立別名 `streamer`，然後 `CACHE` 會從叢集的預設儲存體載入它。 稍後，`streamer` 會與 `STREAM` 運算子搭配使用，進而處理 LOG 中所包含的單行，並以一連串資料行的方式傳回資料。

    > [!NOTE]
    > 建立別名時，用於串流處理的應用程式名稱的兩邊必須加上 \` (倒引號) 字元，與 `SHIP` 搭配使用時，必須加上 ' (單引號) 字元。

4. 輸入最後一行後，作業應該就會開始。 它會傳回類似下列文字的輸出：

        (2012-02-03 20:11:56 SampleClass5 [WARN] problem finding id 1358451042 - java.lang.Exception)
        (2012-02-03 20:11:56 SampleClass5 [DEBUG] detail for id 1976092771)
        (2012-02-03 20:11:56 SampleClass5 [TRACE] verbose detail for id 1317358561)
        (2012-02-03 20:11:56 SampleClass5 [TRACE] verbose detail for id 1737534798)
        (2012-02-03 20:11:56 SampleClass7 [DEBUG] detail for id 1475865947)

## <a name="next-steps"></a>後續步驟

在本文中，您已學會如何從 HDInsight 上的 Hive 和 Pig 使用 .NET Framework 應用程式。 如果您想要了解如何搭配使用 Python 與 Hive 和 Pig，請參閱[搭配使用 Python 與 HDInsight 中的 Hive 和 Pig](python-udf-hdinsight.md)。

若要了解Pig 和 Hive 的其他使用方式，以及了解使用 MapReduce 的方式，請參閱下列文件：

* [〈搭配 HDInsight 使用 Hivet〉](hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hdinsight-use-pig.md)
* [〈搭配 HDInsight 使用 MapReduce〉](hdinsight-use-mapreduce.md)
