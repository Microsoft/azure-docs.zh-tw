---
title: 使用以 Linux 為基礎的 HDInsight 開發指令碼動作 - Azure
description: 深入了解如何使用 Bash 指令碼自訂以 Linux 為基礎的 HDInsight 叢集。 HDInsight 的指令碼動作功能可讓您在建立叢集期間或之後執行指令碼。 指令碼可用於變更叢集組態設定或安裝其他軟體。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/10/2018
ms.author: jasonh
ms.openlocfilehash: 9a10620e3a388fa47076afd54e1789607e5ca47e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986246"
---
# <a name="script-action-development-with-hdinsight"></a>使用 HDInsight 開發指令碼動作

了解如何使用 Bash 指令碼來自訂 HDInsight 叢集。 指令碼動作是在建立叢集期間或之後自訂 HDInsight 的一種方法。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="what-are-script-actions"></a>什麼是指令碼動作

指令碼動作是 Azure 在叢集節點上執行以進行組態變更或安裝軟體的 Bash 指令碼。 指令碼動作是依據根權限來執行，並具有叢集節點的完整存取權限。

您可以透過下列方法套用指令碼動作︰

| 使用此方法來套用指令碼... | 在叢集建立期間... | 在執行中的叢集上... |
| --- |:---:|:---:|
| Azure 入口網站 |✓ |✓ |
| Azure PowerShell |✓ |✓ |
| Azure 傳統 CLI |&nbsp; |✓ |
| HDInsight .NET SDK |✓ |✓ |
| Azure Resource Manager 範本 |✓ |&nbsp; |

如需使用這些方法來套用指令碼動作的詳細資訊，請參閱 [使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。

## <a name="bestPracticeScripting"></a>指令碼開發的最佳做法

為 HDInsight 叢集開發自訂的指令碼時，有數個最佳做法需要牢記在心：

* [目標 Hadoop 版本](#bPS1)
* [鎖定 OS 版本](#bps10)
* [提供穩定的指令碼資源連結](#bPS2)
* [使用預先編譯的資源](#bPS4)
* [確保叢集自訂指令碼具有等冪性](#bPS3)
* [確保叢集架構具有高可用性](#bPS5)
* [設定自訂元件來使用 Azure Blob 儲存體](#bPS6)
* [將資訊寫入至 STDOUT 和 STDERR](#bPS7)
* [將檔案儲存為具有 LF 行尾結束符號的 ASCII](#bps8)
* [使用重試邏輯從暫時性錯誤復原](#bps9)

> [!IMPORTANT]
> 指令碼動作必須在 60 分鐘內完成，否則程序就會失敗。 在節點佈建期間，指令碼會與其他安裝和設定程序同時執行。 與在您開發環境中的執行時間相較，爭用 CPU 時間和網路頻寬等資源可能會導致指令碼需要較長的時間才能完成。

### <a name="bPS1"></a>目標 Hadoop 版本

不同版本的 HDInsight 有不同版本的 Hadoop 服務和已安裝的元件。 如果您的指令碼預期特定版本的服務或元件，您應該只在包含必要元件的 HDInsight 版本中使用指令碼。 您可以使用 [HDInsight 元件版本設定](hdinsight-component-versioning.md) 文件，找到有關 HDInsight 隨附的元件版本資訊。

### <a name="bps10"></a>鎖定 OS 版本

以 Linux 為基礎的 HDInsight 所根據的是 Ubuntu Linux 散發套件。 不同 HDInsight 版本仰賴不同的 Ubuntu 版本，這可能會改變指令碼的運作方式。 例如，HDInsight 3.4 及更早版本所根據的是使用 Upstart 的 Ubuntu 版本。 3.5 版和更高版本根據的是 Ubuntu 16.04，其使用的是 Systemd。 Systemd 和 Upstart 仰賴不同的命令，因此應將指令碼撰寫為適用兩者。

HDInsight 3.4 和 3.5 之間的另一個重要差異在於，`JAVA_HOME` 現在指向 Java 8。

您可以使用 `lsb_release` 檢查 OS 版本。 下列程式碼示範如何判斷指令碼是在 Ubuntu 14 或 16 上執行：

```bash
OS_VERSION=$(lsb_release -sr)
if [[ $OS_VERSION == 14* ]]; then
    echo "OS verion is $OS_VERSION. Using hue-binaries-14-04."
    HUE_TARFILE=hue-binaries-14-04.tgz
elif [[ $OS_VERSION == 16* ]]; then
    echo "OS verion is $OS_VERSION. Using hue-binaries-16-04."
    HUE_TARFILE=hue-binaries-16-04.tgz
fi
...
if [[ $OS_VERSION == 16* ]]; then
    echo "Using systemd configuration"
    systemctl daemon-reload
    systemctl stop webwasb.service    
    systemctl start webwasb.service
else
    echo "Using upstart configuration"
    initctl reload-configuration
    stop webwasb
    start webwasb
fi
...
if [[ $OS_VERSION == 14* ]]; then
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
elif [[ $OS_VERSION == 16* ]]; then
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
fi
```

您可以在 https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh 找到含有這些程式碼片段的完整指令碼。

如需 HDInsight 所使用的 Ubuntu 版本，請參閱 [HDInsight 元件版本](hdinsight-component-versioning.md)文件。

若要了解 Systemd 和 Upstart 之間的差異，請參閱 [Upstart 使用者的 Systemd](https://wiki.ubuntu.com/SystemdForUpstartUsers)。

### <a name="bPS2"></a>提供穩定的指令碼資源連結

在叢集的整個存留期，指令碼和相關聯的資源必須維持可用。 如果已在調整作業期間將新節點加入至叢集，則需要這些資源。

最佳做法是下載並封存您的訂用帳戶上 Azure 儲存體帳戶中的所有項目。

> [!IMPORTANT]
> 使用的儲存體帳戶必須是叢集的預設儲存體帳戶，或是位於其他任何儲存體帳戶上的公用唯讀容器。

例如，Microsoft 所提供的範例會儲存於 [https://hdiconfigactions.blob.core.windows.net/](https://hdiconfigactions.blob.core.windows.net/) 儲存體帳戶。 這個位置是 HDInsight 小組維護的公用、唯讀容器。

### <a name="bPS4"></a>使用預先編譯的資源

若要減少執行指令碼所花費的時間，請避免這類從原始程式碼直接編譯的作業。 例如，先行編譯資源並以 HDInsight 格式將其儲存於相同資料中心中的 Azure 儲存體帳戶 Blob。

### <a name="bPS3"></a>確保叢集自訂指令碼具有等冪性

指令碼必須具有等冪性。 如果指令碼執行多次，則每次都應該讓叢集回到相同狀態。

例如，如果一個會修改組態檔的指令碼執行多次，則不應該新增重複的項目。

### <a name="bPS5"></a>確保叢集架構具有高可用性

以 Linux 為基礎的 HDInsight 叢集提供在叢集裡作用中的兩個前端節點，且指令碼動作會針對兩個節點執行。 如果您安裝的元件只需要一個前端節點，請勿在這兩個前端節點上都安裝元件。

> [!IMPORTANT]
> HDInsight 隨附的服務已設計為在兩個前端節點之間視需要容錯移轉。 這項功能不會延伸至透過指令碼動作所安裝的自訂元件。 如果自訂元件需要有高可用性時，您必須實作自己的容錯移轉機制。

### <a name="bPS6"></a>設定自訂元件來使用 Azure Blob 儲存體

您安裝在叢集上的元件可能有使用 Hadoop 分散式檔案系統 (HDFS) 儲存體的預設組態。 HDInsight 使用 Azure 儲存體或 Data Lake Store 作為預設儲存體。 兩者都提供 HDFS 相容的檔案系統，即使刪除叢集，也能保存資料。 您可能需要將您所安裝的元件設定為使用 WASB 或 ADL，而非 HDFS。

對於大部分作業，您不需要指定檔案系統。 例如，以下會將 giraph-examples.jar 檔案從本機檔案系統複製到叢集儲存體：

```bash
hdfs dfs -put /usr/hdp/current/giraph/giraph-examples.jar /example/jars/
```

在此範例中，`hdfs` 命令會自動使用預設叢集儲存體。 對於某些作業，您可能需要指定 URI。 例如，Data Lake Store 的 `adl:///example/jars`，或 Azure 儲存體的 `wasb:///example/jars`。

### <a name="bPS7"></a>將資訊寫入至 STDOUT 和 STDERR

HDInsight 會記錄指令碼輸出，並將輸出寫入 STDOUT 和 STDERR。 您可以使用 Ambari Web UI 檢視這項資訊。

> [!NOTE]
> 只有在成功建立叢集之後，才能使用 Ambari。 如果您在叢集建立期間使用指令碼動作，但建立失敗，請參閱 [使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md#troubleshooting) 的疑難排解一節，以取得存取記錄資訊的其他方式。

大部分的公用程式和安裝套件已經將資訊寫入 STDOUT 和 STDERR，不過您可以新增其他記錄。 若要將文字傳送到 STDOUT，請使用 `echo`。 例如︰

```bash
echo "Getting ready to install Foo"
```

根據預設，`echo` 會將字串傳送至 STDOUT。 若要將其導向至 STDERR，請在 `echo` 之前加入 `>&2`。 例如︰

```bash
>&2 echo "An error occurred installing Foo"
```

這會將寫入 STDOUT 的資訊改為重新導向至 STDERR (2)。 如需 IO 重新導向的詳細資訊，請參閱 [http://www.tldp.org/LDP/abs/html/io-redirection.html](http://www.tldp.org/LDP/abs/html/io-redirection.html)。

如需檢視指令碼動作記錄之資訊的詳細資訊，請參閱 [使用指令碼動作來自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md#troubleshooting)

### <a name="bps8"></a> 將檔案儲存為具有 LF 行尾結束符號的 ASCII

Bash 指令碼應該儲存為 ASCII 格式，該格式以 LF 做為行尾結束符號。 儲存為 UTF-8 或使用 CRLF 作為行尾結束符號的檔案，可能會發生下列錯誤而失敗︰

```
$'\r': command not found
line 1: #!/usr/bin/env: No such file or directory
```

### <a name="bps9"></a> 使用重試邏輯從暫時性錯誤復原

下載檔案時，使用 apt-get 安裝封裝的動作，或其他透過網際網路傳輸資料的動作，可能會因為暫時性網路錯誤而失敗。 例如，您正進行通訊的遠端資源可能正處於容錯移轉至備份節點的程序中。

若要讓您的指令碼從暫時性錯誤中復原，可以實作重試邏輯。 下列函式示範如何實作重試邏輯。 在失敗之前，它會重試作業三次。

```bash
#retry
MAXATTEMPTS=3

retry() {
    local -r CMD="$@"
    local -i ATTMEPTNUM=1
    local -i RETRYINTERVAL=2

    until $CMD
    do
        if (( ATTMEPTNUM == MAXATTEMPTS ))
        then
                echo "Attempt $ATTMEPTNUM failed. no more attempts left."
                return 1
        else
                echo "Attempt $ATTMEPTNUM failed! Retrying in $RETRYINTERVAL seconds..."
                sleep $(( RETRYINTERVAL ))
                ATTMEPTNUM=$ATTMEPTNUM+1
        fi
    done
}
```

下列範例示範如何使用此函式。

```bash
retry ls -ltr foo

retry wget -O ./tmpfile.sh https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh
```

## <a name="helpermethods"></a>自訂指令碼的協助程式方法

指令碼動作協助程式方法是您在撰寫字訂指令碼時可以使用的公用程式。 [https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh) 指令碼中含有這些方法。 請使用下列命令下載這些方法，然後在您的指令碼中使用︰

```bash
# Import the helper method module.
wget -O /tmp/HDInsightUtilities-v01.sh -q https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh && source /tmp/HDInsightUtilities-v01.sh && rm -f /tmp/HDInsightUtilities-v01.sh
```

下列協助程式可用於您的指令碼：

| 協助程式使用方式 | 說明 |
| --- | --- |
| `download_file SOURCEURL DESTFILEPATH [OVERWRITE]` |從來源 URI 將檔案下載到指定的檔案路徑中。 根據預設，不會覆寫現有的檔案。 |
| `untar_file TARFILE DESTDIR` |將 tar 檔案解壓縮 (使用 `-xf`) 至目的地目錄。 |
| `test_is_headnode` |如果在叢集前端節點上執行，則會傳回 1，否則傳回 0。 |
| `test_is_datanode` |如果目前節點是資料 (背景工作角色) 節點，則會傳回 1，否則傳回 0。 |
| `test_is_first_datanode` |如果目前節點是第一個資料 (背景工作角色) 節點 (名為 workernode0)，則會傳回 1，否則傳回 0。 |
| `get_headnodes` |傳回叢集中前端節點的完整網域名稱。 名稱會以逗號分隔。 發生錯誤時會傳回空字串。 |
| `get_primary_headnode` |取得主要前端節點的完整網域名稱。 發生錯誤時會傳回空字串。 |
| `get_secondary_headnode` |取得次要前端節點的完整網域名稱。 發生錯誤時會傳回空字串。 |
| `get_primary_headnode_number` |取得主要前端節點的數字尾碼。 發生錯誤時會傳回空字串。 |
| `get_secondary_headnode_number` |取得次要前端節點的數字尾碼。 發生錯誤時會傳回空字串。 |

## <a name="commonusage"></a>常見使用模式

本節指引您如何實作某些撰寫自訂指令碼時可能遇到的常見使用模式。

### <a name="passing-parameters-to-a-script"></a>將參數傳遞給指令碼

在某些情況下，您的指令碼可能需要參數。 例如，使用 Ambari REST API 時，您可能需要叢集的系統管理員密碼。

傳遞至指令碼的參數稱為「位置參數」，並且指派至 `$1` 作為第一個參數，指派至 `$2` 作為第二個參數，依此類推。 `$0` 包含指令碼本身的名稱。

當作參數傳遞給指令碼的值必須以單引號 (') 括住。 這麼做可確保傳遞的值會被視為常值。

### <a name="setting-environment-variables"></a>設定環境變數

下列陳述式可以設定環境變數：

    VARIABLENAME=value

其中 VARIABLENAME 是變數的名稱。 若要存取變數，請使用 `$VARIABLENAME`。 例如，若要將位置參數提供的值指派為名為 PASSWORD 的環境變數，請使用下列陳述式：

    PASSWORD=$1

後續的資訊存取則可以使用 `$PASSWORD`。

在指令碼中設定的環境變數只會存在於指令碼的範圍內。 在某些情況下，您可能需要新增全系統環境變數，這些變數在指令碼完成之後持續存在。 若要新增全系統環境變數，請將變數新增至 `/etc/environment`。 例如，下列陳述式會新增 `HADOOP_CONF_DIR`：

```bash
echo "HADOOP_CONF_DIR=/etc/hadoop/conf" | sudo tee -a /etc/environment
```

### <a name="access-to-locations-where-the-custom-scripts-are-stored"></a>存取自訂指令碼儲存所在位置

用來自訂叢集的指令碼必須儲存在下列其中一個位置︰

* 與叢集相關聯的 __Azure 儲存體帳戶__。

* 與叢集相關聯的__其他儲存體帳戶__。

* __可公開讀取的 URI__。 例如，在 OneDrive、Dropbox 或其他檔案裝載服務上儲存之資料的 URL。

* 與 HDInsight 叢集相關聯的 __Azure Data Lake Store 帳戶__。 如需有關使用 Azure Data Lake Store 與 HDInsight 的詳細資訊，請參閱[快速入門：在 HDInsight 中設定叢集](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)。

    > [!NOTE]
    > 用來存取 Data Lake Store 的服務主體 HDInsight 必須具有指令碼的讀取權限。

指令碼所使用的資源也必須可公開使用。

將檔案儲存在 Azure 儲存體帳戶或 Azure Data Lake Store 中可快速存取，因為兩者都在 Azure 網路內。

> [!NOTE]
> 用來參考指令碼的 URI 格式會隨所使用的服務而有所不同。 若為與 HDInsight 叢集相關聯的儲存體帳戶，請使用 `wasb://` 或 `wasbs://`。 若為可公開讀取的 URI，請使用 `http://` 或 `https://`。 若為 Data Lake Store，請使用 `adl://`。

### <a name="checking-the-operating-system-version"></a>檢查作業系統版本

不同 HDInsight 版本各仰賴特定的 Ubuntu 版本。 您在指令碼中必須檢查的 OS 版本之間可能會有差異。 例如，您可能必須安裝繫結至 Ubuntu 版本的二進位檔。

若要檢查 OS 版本，請使用 `lsb_release`。 例如，下列指令碼示範如何根據 OS 版本來參考特定的 tar 檔案︰

```bash
OS_VERSION=$(lsb_release -sr)
if [[ $OS_VERSION == 14* ]]; then
    echo "OS verion is $OS_VERSION. Using hue-binaries-14-04."
    HUE_TARFILE=hue-binaries-14-04.tgz
elif [[ $OS_VERSION == 16* ]]; then
    echo "OS verion is $OS_VERSION. Using hue-binaries-16-04."
    HUE_TARFILE=hue-binaries-16-04.tgz
fi
```

## <a name="deployScript"></a>指令碼動作的部署檢查清單

以下是準備部署指令碼時所採取的步驟：

* 將包含自訂指令碼的檔案放在叢集節點可於部署期間存取的位置。 例如，叢集的預設儲存體。 檔案也可以儲存在可公開讀取的主機服務中。
* 確定指令碼具有等冪性。 這麼做可讓指令碼在相同節點上執行多次。
* 使用暫存檔案目錄 /tmp 來存放指令碼所使用的下載檔案，然後在執行完指令碼之後將這些檔案清除。
* 如果作業系統層級設定或 Hadoop 服務組態檔變更，建議您重新啟動 HDInsight 服務。

## <a name="runScriptAction"></a>如何執行指令碼動作

您可以透過下列方法，使用指令碼動作來自訂 HDInsight 叢集︰

* Azure 入口網站
* Azure PowerShell
* Azure 資源管理員範本
* HDInsight .NET SDK。

如需有關如何使用每種方法的詳細資訊，請參閱[如何使用指令碼動作](hdinsight-hadoop-customize-cluster-linux.md)。

## <a name="sampleScripts"></a>自訂指令碼範例

Microsoft 提供了在 HDInsight 叢集上安裝元件的範例指令碼。 請參閱下列連結以取得更多的範例指令碼動作。

* [在 HDInsight 叢集上安裝及使用色調](hdinsight-hadoop-hue-linux.md)
* [在 HDInsight 叢集上安裝及使用 Solr](hdinsight-hadoop-solr-install-linux.md)
* [在 HDInsight 叢集上安裝及使用 Giraph](hdinsight-hadoop-giraph-install-linux.md)
* [在 HDInsight 叢集上安裝或升級 Mono](hdinsight-hadoop-install-mono.md)

## <a name="troubleshooting"></a>疑難排解

使用您所開發的指令碼時可能會遇到下列錯誤：

**錯誤**：`$'\r': command not found`。 有時候後面接續 `syntax error: unexpected end of file`。

原因：這個錯誤的原因是指令碼中以 CRLF 作為行尾結束符號。 Unix 系統預期只有 LF 當做行尾結束符號。

此問題最常發生在於 Windows 環境中撰寫指令碼時，因為 CRLF 是 Windows 上許多文字編輯器中常見的行尾結束符號。

解決方式：如果您的文字編輯器提供選項，請選取 Unix 格式或 LF 作為行尾結束符號。 您也可以在 Unix 系統上使用下列命令，將 CRLF 變更為 LF：

> [!NOTE]
> 下列命令大致相當於將 CRLF 行尾結束符號變更為 LF。 根據您的系統上可用的公用程式，選取其中一個。

| 命令 | 注意 |
| --- | --- |
| `unix2dos -b INFILE` |原始檔案會以 .BAK 副檔名進行備份 |
| `tr -d '\r' < INFILE > OUTFILE` |OUTFILE 會包含只有 LF 行尾結束符號的版本 |
| `perl -pi -e 's/\r\n/\n/g' INFILE` | 直接修改檔案 |
| ```sed 's/$'"/`echo \\\r`/" INFILE > OUTFILE``` |OUTFILE 會包含只有 LF 行尾結束符號的版本。 |

**錯誤**：`line 1: #!/usr/bin/env: No such file or directory`。

*原因*：指令碼儲存為具有位元組順序標記 (BOM) 的 UTF-8 時，就會發生這個錯誤。

*解決方式*：將檔案儲存為 ASCII，或不具有 BOM 的 UTF-8。 您也可以在 Linux 或 Unix 系統上使用下列命令，以建立不具有 BOM 的檔案：

    awk 'NR==1{sub(/^\xef\xbb\xbf/,"")}{print}' INFILE > OUTFILE

以包含 BOM 的檔案取代 `INFILE`。 `OUTFILE` 應該是新檔案的名稱，且包含不具有 BOM 的指令碼。

## <a name="seeAlso"></a>後續步驟

* 深入了解 [使用指令碼動作來自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)
* 使用 [HDInsight.NET SDK 參考](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight) ，深入了解如何建立 .NET 應用程式來管理 HDInsight
* 使用 [HDInsight REST API](https://msdn.microsoft.com/library/azure/mt622197.aspx) ，以了解如何使用 REST 在 HDInsight 叢集上執行管理動作。
