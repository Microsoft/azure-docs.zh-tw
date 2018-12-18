---
title: 使用 Python 元件的 Apache Storm - Azure HDInsight
description: 了解如何建立使用 Python 元件的 Apache Storm 拓撲。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
keywords: apache storm python
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 04/30/2018
ms.author: jasonh
ms.openlocfilehash: 753c870f5d99d7bf887944b0d180645dc019b2b1
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045762"
---
# <a name="develop-apache-storm-topologies-using-python-on-hdinsight"></a>在 HDInsight 上使用 Python 開發 Apache Storm 拓撲

了解如何建立使用 Python 元件的 Apache Storm 拓撲。 Apache Storm 支援多種語言，甚至可讓您將數種語言的元件結合成一個拓撲。 Flux 架構 (隨 Storm 0.10.0 一起引進) 可讓您輕鬆建立使用 Python 元件的解決方案。

> [!IMPORTANT]
> 本文件中的資訊已使用 Storm on HDInsight 3.6 進行測試。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

此專案的程式碼位於 [https://github.com/Azure-Samples/hdinsight-python-storm-wordcount](https://github.com/Azure-Samples/hdinsight-python-storm-wordcount)。

## <a name="prerequisites"></a>必要條件

* Python 2.7 或更新版本

* Java JDK 1.8 或更新版本

* Maven 3

* (選擇性) 本機 Storm 開發環境。 只有當您想要在本機執行拓撲時，才需要本機 Storm 環境。 如需詳細資訊，請參閱[設定開發環境](http://storm.apache.org/releases/1.1.2/Setting-up-development-environment.html)。

## <a name="storm-multi-language-support"></a>Storm 多語言支援

Apache Storm 專門用來搭配以任何程式設計語言撰寫的元件。 這些元件必須了解如何使用 [Storm 的 Thrift 定義](https://github.com/apache/storm/blob/master/storm-core/src/storm.thrift)。 在 Python 中，Apache Storm 專案隨附一個模組，可讓您輕鬆地與 Strom 互動。 您可以在 [https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py](https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py) 找到此模組。

Storm 是在 Java 虛擬機器 (JVM) 上執行的 Java 程序。 以其他語言撰寫的元件會以子流程執行。 Storm 會使用透過 stdin/stdout 傳送的 JSON 訊息，與這些子流程進行通訊。 如需各元件之間通訊的詳細資訊，請參閱 [多語言通訊協定](https://storm.apache.org/documentation/Multilang-protocol.html) 文件。

## <a name="python-with-the-flux-framework"></a>採用 Flux 架構的 Python

Flux 架構可讓您獨立於元件之外定義 Storm 拓撲。 Flux 架構會使用 YAML 來定義 Storm 拓撲。 下列文字是如何在 YAML 文件中參考 Python 元件的範例：

```yaml
# Spout definitions
spouts:
  - id: "sentence-spout"
    className: "org.apache.storm.flux.wrappers.spouts.FluxShellSpout"
    constructorArgs:
      # Command line
      - ["python", "sentencespout.py"]
      # Output field(s)
      - ["sentence"]
    # parallelism hint
    parallelism: 1
```

類別 `FluxShellSpout` 用來啟動實作 Spout 的 `sentencespout.py` 指令碼。

Flux 要求 Python 指令碼位於拓撲所在之 jar 檔案內的 `/resources` 目錄。 因此，這個範例會將 Python 指令碼儲存在 `/multilang/resources` 目錄。 `pom.xml` 使用下列 XML 來包含此檔案：

```xml
<!-- include the Python components -->
<resource>
    <directory>${basedir}/multilang</directory>
    <filtering>false</filtering>
</resource>
```

如先前所述，有一個 `storm.py` 檔案實作 Storm 的 Thrift 定義。 建置專案時，Flux 架構會自動包含 `storm.py`，因此您不必擔心要包含它。

## <a name="build-the-project"></a>建置專案

從專案根目錄中，使用下列命令︰

```bash
mvn clean compile package
```

此命令會建立 `target/WordCount-1.0-SNAPSHOT.jar` 檔案，其中包含已編譯的拓撲。

## <a name="run-the-topology-locally"></a>在本機測試拓撲

若要在本機執行拓撲，請使用下列命令：

```bash
storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -l -R /topology.yaml
```

> [!NOTE]
> 此命令需要本機 Storm 開發環境。 如需詳細資訊，請參閱[設定開發環境](http://storm.apache.org/releases/current/Setting-up-development-environment.html)。

拓撲啟動之後，就會將類似下列文字的資訊發出至本機主控台︰


    24302 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
    24302 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting the
    24302 [Thread-28] INFO  o.a.s.t.ShellBolt - ShellLog pid:2437, name:counter-bolt Emitting years:160
    24302 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=the, count=599}
    24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=seven, count=302}
    24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=dwarfs, count=143}
    24303 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
    24303 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting cow
    24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=four, count=160}


若要停止拓撲，請使用 __Ctrl+C__。

## <a name="run-the-storm-topology-on-hdinsight"></a>在 HDInsight 上執行 Storm 拓撲

1. 使用下列命令將 `WordCount-1.0-SNAPSHOT.jar`檔案複製到 Storm on HDInsight 叢集：

    ```bash
    scp target\WordCount-1.0-SNAPSHOT.jar sshuser@mycluster-ssh.azurehdinsight.net
    ```

    將 `sshuser` 取代為叢集的 SSH 使用者。 將 `mycluster` 取代為叢集名稱。 系統可能會提示您輸入 SSH 使用者的密碼。

    如需有關使用 SSH 和 SCP 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 上傳檔案後，使用 SSH 連線至叢集：

    ```bash
    ssh sshuser@mycluster-ssh.azurehdinsight.net
    ```

3. 從 SSH 工作階段中，使用下列命令啟動叢集上的拓撲：

    ```bash
    storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -r -R /topology.yaml
    ```

3. 您可以使用 Storm UI 來檢視叢集上的拓撲。 Storm UI 位於 https://mycluster.azurehdinsight.net/stormui。 將 `mycluster` 取代為您的叢集名稱。

> [!NOTE]
> Storm 拓撲啟動之後會一直執行到停止為止。 若要停止拓撲，請使用下列其中一種方法：
>
> * 從命令列執行 `storm kill TOPOLOGYNAME` 命令
> * Storm UI 中的 [終止] 按鈕。


## <a name="next-steps"></a>後續步驟

請參閱下列文件，了解搭配使用 Python 與 HDInsight 的其他方式。

* [如何使用 Python 串流處理 MapReduce 工作](../hadoop/apache-hadoop-streaming-python.md)
* [如何在 Pig 和 Hive 中使用 Python 使用者定義函數 (UDF)](../hadoop/python-udf-hdinsight.md)
