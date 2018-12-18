---
title: 搭配 Apache Hive 和 Pig 的 Python UDF - Azure HDInsight
description: 了解如何在 HDInsight 中從 Hive 和 Pig (Azure 上的 Hadoop 技術堆疊) 使用 Python 使用者定義函數 (UDF)。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 02/27/2018
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.openlocfilehash: ef9292e7e36f5accabf532ef4a26d334fb880859
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43052106"
---
# <a name="use-python-user-defined-functions-udf-with-hive-and-pig-in-hdinsight"></a>在 HDInsight 上使用 Python 使用者定義函數 (UDF) 與 Hive 和 Pig

了解如何在 Azure HDInsight 的 Hadoop 中搭配使用 Python 使用者定義函式 (UDF) 與 Apache Hive 和 Pig。

## <a name="python"></a>HDInsight 上的 Python

HDInsight 3.0 和更新版本上預設已安裝 Python2.7。 Apache Hive 可以與此版本的 Python 搭配使用，以進行資料流處理。 資料流處理會使用 STDOUT 和 STDIN，以在 Hive 和 UDF 之間傳遞資料。

HDInsight 也包含 Jython (以 Java 撰寫的 Python 實作)。 Jython 直接在 Java 虛擬機器上執行，並不使用資料流。 搭配使用 Python 與 Pig 時，建議使用的 Python 解譯器為 Jython。

> [!WARNING]
> 本文件中的這些步驟進行下列假設： 
>
> * 您在本機開發環境中建立 Python 指令碼。
> * 您從本機 Bash 工作階段使用 `scp` 命令，或者使用提供的 PowerShell 指令碼，將指令碼上傳至 HDInsight 。
>
> 如果您想要使用 [Azure Cloud Shell (bash)](https://docs.microsoft.com/azure/cloud-shell/overview) 預覽來利用 HDInsight，則必須：
>
> * 建立 Cloud Shell 環境內的指令碼。
> * 使用 `scp` 將檔案從 Cloud Shell 上傳至 HDInsight。
> * 使用 `ssh` 從 Cloud Shell 命令連線至 HDInsight，並執行範例。

## <a name="hivepython"></a>Hive UDF

從 Hive 中，透過 HiveQL `TRANSFORM` 陳述式，可將 Python 當作 UDF 使用。 例如，下列 HiveQL 會叫用叢集的預設 Azure 儲存體帳戶所儲存的 `hiveudf.py` 檔案。

**以 Linux 為基礎的 HDInsight**

```hiveql
add file wasb:///hiveudf.py;

SELECT TRANSFORM (clientid, devicemake, devicemodel)
    USING 'python hiveudf.py' AS
    (clientid string, phoneLable string, phoneHash string)
FROM hivesampletable
ORDER BY clientid LIMIT 50;
```

**以 Windows 為基礎的 HDInsight**

```hiveql
add file wasb:///hiveudf.py;

SELECT TRANSFORM (clientid, devicemake, devicemodel)
    USING 'D:\Python27\python.exe hiveudf.py' AS
    (clientid string, phoneLable string, phoneHash string)
FROM hivesampletable
ORDER BY clientid LIMIT 50;
```

> [!NOTE]
> 在以 Windows 為基礎的 HDInsight 叢集上，`USING` 子句必須指定 python.exe 的完整路徑。

以下是此範例所執行的動作：

1. 檔案開頭的 `add file` 陳述式將 `hiveudf.py` 檔案加入至分散式快取，供叢集中的所有節點存取。
2. 使用 `SELECT TRANSFORM ... USING` 陳述式選取來自 `hivesampletable` 的資料。 它也會將 clientid、devicemake 和 devicemodel 值傳遞至 `hiveudf.py` 指令碼。
3. `AS` 子句描述從 `hiveudf.py` 中傳回的欄位。

<a name="streamingpy"></a>

### <a name="create-the-hiveudfpy-file"></a>建立 hiveudf.py 檔案


在開發環境中，建立名為 `hiveudf.py` 的文字檔。 使用下列程式碼作為此檔案的內容：

```python
#!/usr/bin/env python
import sys
import string
import hashlib

while True:
    line = sys.stdin.readline()
    if not line:
        break

    line = string.strip(line, "\n ")
    clientid, devicemake, devicemodel = string.split(line, "\t")
    phone_label = devicemake + ' ' + devicemodel
    print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])
```

此指令碼會執行下列動作：

1. 從 STDIN 中讀取資料行。
2. 使用 `string.strip(line, "\n ")` 移除結尾新行字元。
3. 執行串流處理時，有一行包含所有的值，而每個值之間是一個定位字元。 因此， `string.split(line, "\t")` 可在每個索引標籤進行分割輸入，並只傳回欄位。
4. 處理完成時，輸出必須以一行寫入 STDOUT，而每一個欄位之間是一個定位字元。 例如： `print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])`。
5. `while` 迴圈會一直重複直到沒有 `line` 讀取。

指令碼輸出是 `devicemake` 和 `devicemodel` 的輸入值串連，並且是串連值的雜湊。

請參閱 [執行範例](#running) ，以了解如何在 HDInsight 叢集上執行此範例。

## <a name="pigpython"></a>Pig UDF

從 Pig 中，透過 `GENERATE` 陳述式，可將 Python 指令碼當作 UDF 使用。 您可以使用 Jython 或 C Python 執行程式碼。

* Jython 是在 JVM 上執行，而且可以從 Pig 原生呼叫。
* C Python 是外部處理序，因此 JVM 上的 Pig 所提供的資料會外送到 Python 處理序中執行的指令碼。 Python 指令碼的輸出會傳送回 Pig。

若要指定 Python 解譯器，請在參考 Python 指令碼時使用 `register`。 下列範例使用 Pig 將指令碼註冊為 `myfuncs`：

* **若要使用 Jython**：`register '/path/to/pigudf.py' using jython as myfuncs;`
* **若要使用 C Python**：`register '/path/to/pigudf.py' using streaming_python as myfuncs;`

> [!IMPORTANT]
> 在使用 Jython 時，pig_jython 檔案的路徑可以是本機路徑或 WASB:// 路徑。 不過，在使用 C Python 時，您必須參考您用來提交 Pig 作業之節點的本機檔案系統上的檔案。

通過註冊之後，此範例針對兩者的 Pig Latin 是相同的︰

```pig
LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);
LOG = FILTER LOGS by LINE is not null;
DETAILS = FOREACH LOG GENERATE myfuncs.create_structure(LINE);
DUMP DETAILS;
```

以下是此範例所執行的動作：

1. 第一行將範例資料檔 `sample.log` 載入 `LOGS` 中。 它也會將每一筆記錄定義為 `chararray`。
2. 下一行會濾除任何 null 值，然後將操作的結果儲存至 `LOG`。
3. 接下來，逐一查看 `LOG` 中的記錄，並使用 `GENERATE` 叫用以 `myfuncs` 載入的 Python/Jython 指令碼所包含的 `create_structure` 方法。 `LINE` 用來將目前的記錄傳給函式。
4. 最後，使用 `DUMP` 指令將輸出傾印至 STDOUT。 這個命令會在作業完成之後顯示結果。

### <a name="create-the-pigudfpy-file"></a>建立 pigudf.py 檔案

在開發環境中，建立名為 `pigudf.py` 的文字檔。 使用下列程式碼作為此檔案的內容：

<a name="streamingpy"></a>

```python
# Uncomment the following if using C Python
#from pig_util import outputSchema

@outputSchema("log: {(date:chararray, time:chararray, classname:chararray, level:chararray, detail:chararray)}")
def create_structure(input):
    if (input.startswith('java.lang.Exception')):
        input = input[21:len(input)] + ' - java.lang.Exception'
    date, time, classname, level, detail = input.split(' ', 4)
    return date, time, classname, level, detail
```

在 Pig Latin 範例中，因為該輸入沒有一致的結構描述，因而將 `LINE` 輸入定義為 chararray。 Python 指令碼會將資料轉換成一致的結構描述，以便輸出。

1. `@outputSchema` 陳述式定義將傳回給 Pig 的資料格式。 在此案例中，這是一個 **data bag**(一種 Pig 資料類型)。 Bag 包含下列欄位，全部都是 chararray (字串)：

   * date - 記錄項目的建立日期
   * time - 記錄項目的建立時間
   * classname - 建立項目所針對的類別名稱
   * level - 記錄層級
   * detail - 記錄項目的詳細資料

2. 接下來，`def create_structure(input)` 定義可供 Pig 傳遞行項目的函數。

3. 範例資料 `sample.log` 大致上符合 date、time、classname、level 和 detail 結構描述。 不過，它包含開頭為 `*java.lang.Exception*` 的數行。 這幾行必須經過修改以符合結構描述。 `if` 陳述式會檢查它們，然後調整輸入資料以將 `*java.lang.Exception*` 字串移至尾端，使資料符合預期的輸出結構描述。

4. 接下來，使用 `split` 命令，在前四個空白字元處分割資料。 輸出即獲指派 `date`、`time`、 `classname`、 `level` 和 `detail` 。

5. 最後，將值傳回給 Pig。

當資料傳回至 Pig 時，其將具有如同 `@outputSchema` 陳述式中定義的一致性結構描述。

## <a name="running"></a>上傳及執行範例

> [!IMPORTANT]
> **SSH** 步驟只適用於以 Linux 為基礎的 HDInsight 叢集。 **PowerShell** 步驟適用於以 Linux 或 Windows 為基礎的 HDInsight 叢集，但需要 Windows 用戶端。

### <a name="ssh"></a>SSH

如需使用 SSH 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

1. 使用 `scp` 將檔案複製到您的 HDInsight 叢集。 例如，下列命令會將檔案複製到名為 **mycluster**的叢集。

    ```bash
    scp hiveudf.py pigudf.py myuser@mycluster-ssh.azurehdinsight.net:
    ```

2. 使用 SSH 連接到叢集。

    ```bash
    ssh myuser@mycluster-ssh.azurehdinsight.net
    ```

    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) 文件。

3. 從 SSH 工作階段，將先前上傳的 python 檔案加入叢集的 WASB 儲存體。

    ```bash
    hdfs dfs -put hiveudf.py /hiveudf.py
    hdfs dfs -put pigudf.py /pigudf.py
    ```

上傳檔案之後，請使用下列步驟執行 Hive 和 Pig 工作。

#### <a name="use-the-hive-udf"></a>使用 Hive UDF

1. 如需連線到 Hive，請使用下列命令：

    ```bash
    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
    ```

    此命令會啟動 Beeline 用戶端。

2. 在 `0: jdbc:hive2://headnodehost:10001/>` 提示上輸入下列查詢：

   ```hive
   add file wasb:///hiveudf.py;
   SELECT TRANSFORM (clientid, devicemake, devicemodel)
       USING 'python hiveudf.py' AS
       (clientid string, phoneLabel string, phoneHash string)
   FROM hivesampletable
   ORDER BY clientid LIMIT 50;
   ```

3. 輸入最後一行後，作業應該就會開始。 作業完成之後，它會傳回與下列範例類似的輸出：

        100041    RIM 9650    d476f3687700442549a83fac4560c51c
        100041    RIM 9650    d476f3687700442549a83fac4560c51c
        100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
        100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
        100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9

4. 如需結束 Beeline，請使用下列命令：

    ```hive
    !q
    ```

#### <a name="use-the-pig-udf"></a>使用 Pig UDF

1. 如需連線到 pig，請使用下列命令：

    ```bash
    pig
    ```

2. 在出現 `grunt>` 提示時輸入下列陳述式：

   ```pig
   Register wasb:///pigudf.py using jython as myfuncs;
   LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);
   LOG = FILTER LOGS by LINE is not null;
   DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
   DUMP DETAILS;
   ```

3. 輸入下列行之後，應該就會開始作業。 作業完成之後，它會傳回與下列資料類似的輸出：

        ((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
        ((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
        ((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
        ((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
        ((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

4. 使用 `quit` 結束 Grunt shell，然後使用下列命令編輯本機檔案系統上的 pigudf.py 檔案：

    ```bash
    nano pigudf.py
    ```

5. 進入編輯器後，移除行首的 `#` 字元將以下這行取消註解：

    ```bash
    #from pig_util import outputSchema
    ```

    這行會修改 Python 指令碼以搭配 C Python 使用，而非 Jython。 完成變更後，使用 **Ctrl+X** 結束編輯器。 選取 [Y]，然後按 **Enter** 儲存變更。

6. 使用 `pig` 命令再次啟動 Shell。 進入 `grunt>` 提示字元後，使用下列命令以使用 C Python 解譯器執行 Python 指令碼。

   ```pig
   Register 'pigudf.py' using streaming_python as myfuncs;
   LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);
   LOG = FILTER LOGS by LINE is not null;
   DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
   DUMP DETAILS;
   ```

    此作業完成後，您應該會看到和先前使用 Jython 執行指令碼時所得到的相同輸出。

### <a name="powershell-upload-the-files"></a>PowerShell：上傳檔案

您可以使用 PowerShell 將檔案上傳至 HDInsight 伺服器。 使用下列指令碼來上傳 Python 檔案：

> [!IMPORTANT] 
> 本節中的步驟是使用 Azure PowerShell。 如需使用 Azure PowerShell 的詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

[!code-powershell[main](../../../powershell_scripts/hdinsight/run-python-udf/run-python-udf.ps1?range=5-41)]

> [!IMPORTANT]
> 將 `C:\path\to` 值變更為您開發環境上檔案的路徑。

此指令碼會擷取 HDInsight 叢集的資訊，然後擷取預設儲存體帳戶的帳戶和金鑰，再將檔案上傳至容器的根目錄。

> [!NOTE]
> 如需上傳檔案的詳細資訊，請參閱[在 HDInsight 中上傳 Hadoop 作業的資料](../hdinsight-upload-data.md)文件。

#### <a name="powershell-use-the-hive-udf"></a>PowerShell：使用 Hive UDF

PowerShell 也可用來從遠端執行 Hive 查詢。 使用下列 PowerShell 指令碼，執行使用 **hiveudf.py** 指令碼的 Hive 查詢：

> [!IMPORTANT]
> 執行前，指令碼會提示您輸入您 HDInsight 叢集的 HTTPs/系統管理帳戶資訊。

[!code-powershell[main](../../../powershell_scripts/hdinsight/run-python-udf/run-python-udf.ps1?range=45-94)]

**Hive** 作業的輸出應該如下範例所示：

    100041    RIM 9650    d476f3687700442549a83fac4560c51c
    100041    RIM 9650    d476f3687700442549a83fac4560c51c
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9

#### <a name="pig-jython"></a>Pig (Jython)

PowerShell 也可用來執行 Pig Latin 作業。 若要執行使用 **pigudf.py** 指令碼的 Pig Latin 作業，請使用下列 PowerShell 指令碼：

> [!NOTE]
> 使用 PowerShell 遠端提交作業時，無法使用 C Python 做為解譯器。

[!code-powershell[main](../../../powershell_scripts/hdinsight/run-python-udf/run-python-udf.ps1?range=98-144)]

**Pig** 作業的輸出應該如下列資料所示：

    ((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
    ((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
    ((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
    ((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
    ((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

## <a name="troubleshooting"></a>疑難排解

### <a name="errors-when-running-jobs"></a>執行工作時發生錯誤

執行 Hive 作業時，您可能會遇到類似以下文字的錯誤：

    Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: [Error 20001]: An error occurred while reading or writing to your custom script. It may have crashed with an error.

這個問題可能是由 Python 檔案中的行尾結束符號所引起。 許多 Windows 編輯器預設都是使用 CRLF 做為行尾結束符號，但是 Linux 應用程式通常預期使用 LF。

若要在檔案上傳至 HDInsight 之前移除 CR 字元，您可以使用下列 PowerShell 陳述式︰

[!code-powershell[main](../../../powershell_scripts/hdinsight/run-python-udf/run-python-udf.ps1?range=148-150)]

### <a name="powershell-scripts"></a>PowerShell 指令碼

用來執行範例的兩個範例 PowerShell 指令碼都包含一行註解，此行會顯示作業的錯誤輸出。 如果您沒有看到預期的工作輸出，請將下列這一行取消註解，再查看錯誤資訊是否指出問題。

[!code-powershell[main](../../../powershell_scripts/hdinsight/run-python-udf/run-python-udf.ps1?range=135-139)]

錯誤資訊 (STDERR) 和作業 (STDOUT) 的結果也會記錄至您的 HDInsight 儲存體中。

| 關於此工作... | 請在 Blob 容器中查看這些檔案 |
| --- | --- |
| Hive |/HivePython/stderr<p>/HivePython/stdout |
| Pig |/PigPython/stderr<p>/PigPython/stdout |

## <a name="next"></a>後續步驟

如果您需要載入非預設提供的 Python 模組，請參閱 [如何將模組部署至 Azure HDInsight](http://blogs.msdn.com/b/benjguin/archive/2014/03/03/how-to-deploy-a-python-module-to-windows-azure-hdinsight.aspx) (英文)。

若要了解使用 MapReduce，及Pig、Hive 的其他使用方式，請參閱下列文件：

* [〈搭配 HDInsight 使用 Hivet〉](hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hdinsight-use-pig.md)
* [〈搭配 HDInsight 使用 MapReduce〉](hdinsight-use-mapreduce.md)
