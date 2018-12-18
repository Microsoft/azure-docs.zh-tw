---
title: 執行自訂 MapReduce 程式 - Azure HDInsight
description: 何時及如何在 HDInsight 中執行自訂 MapReduce 程式。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 12/04/2017
ms.author: ashishth
ms.openlocfilehash: 80f58157e69ff5a6e707408d795889b5bcd677b7
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045566"
---
# <a name="run-custom-mapreduce-programs"></a>執行自訂 MapReduce 程式

Hadoop 型巨量資料系統 (例如 HDInsight) 可讓您使用各種工具和技術來處理資料。 下表說明各項工具和技術的主要優點和注意事項。

| 查詢機制 | 優點 | 考量 |
| --- | --- | --- |
| **使用 HiveQL 的 Hive** | <ul><li>這是一個絕佳的解決方案，除了適用於大量不可變資料的批次處理與分析之外，也適用於進行資料摘要及依需求進行查詢。 它使用類似 SQL 的熟悉語法。</li><li>可用來產生可輕鬆進行分割和編製索引的永久資料表。</li><li>可在相同的資料上建立多個外部資料表和檢視表。</li><li>支援簡單的資料倉儲實作，可提供大規模向外延展和容錯功能來進行資料儲存和處理。</li></ul> | <ul><li>要求來源資料必須至少有某種可識別的結構。</li><li>不適用於即時查詢和資料列層級的更新。 最適合用於在大型資料集上進行的批次工作。</li><li>可能無法執行某些類型的複雜處理工作。</li></ul> |
| **使用 Pig Latin 的 Pig** | <ul><li>這是一個絕佳的解決方案，除了適用於以集合方式操作資料、合併及篩選資料集、將函式套用至記錄或記錄群組之外，也適用於透過定義資料行、將值分組或將資料行轉換成資料列來重組資料結構。</li><li>可使用以工作流程為基礎的方法作為資料上的一系列作業。</li></ul> | <ul><li>SQL 使用者可能會發現與 HiveQL 相比，Pig Latin 的語法較為陌生而較難使用。</li><li>預設輸出通常是文字檔，因此較難與視覺化工具 (例如 Excel) 搭配使用。 通常您會在輸出上加上一層 Hive 資料表。</li></ul> |
| **自訂 Map/Reduce** | <ul><li>提供對於對應和縮減階段及執行的完整控制。</li><li>允許將查詢最佳化以從叢集發揮最大效能，或是將伺服器和網路上的負載降到最低。</li><li>可以用一系列知名的語言來撰寫元件。</li></ul> | <ul><li>與使用 Pig 或 Hive 相比較難使用，因為您必須建立自己的 map 和 reduce 元件。</li><li>較難實作需要聯結幾組資料的程序。</li><li>雖然有測試架構可供使用，但偵錯程式碼比一般應用程式更為複雜，因為此程式碼會以批次作業的形式在 Hadoop 作業排程器的控制下執行。</li></ul> |
| **HCatalog** | <ul><li>會擷取儲存體的路徑詳細資料，既可讓系統管理更容易進行，又可讓使用者無須知道資料的儲存位置。</li><li>可啟用事件 (例如當資料可供使用時) 通知功能，讓其他工具 (例如 Oozie) 能夠偵測作業何時發生。</li><li>會公開資料的關聯式檢視 (包括依索引鍵進行分割)，讓資料變得容易存取。</li></ul> | <ul><li>預設支援 RCFile、CSV 文字、JSON 文字、SequenceFile 及 ORC 檔案格式，但您可能需要撰寫自訂 SerDe，才能支援其他格式。</li><li>HCatalog 不是安全執行緒。</li><li>在 Pig 指令碼中使用 HCatalog 載入器時，對資料行的資料類型有一些限制。 如需詳細資訊，請參閱 Apache HCatalog 文件中的 [HCatLoader 資料類型](http://cwiki.apache.org/confluence/display/Hive/HCatalog%20LoadStore#HCatalogLoadStore-HCatLoaderDataTypes)(英文\)。</li></ul> |

通常您會使用這些方法中能夠提供所需結果的最簡單方法。 例如，您可能只需使用 Hive 即可獲得這樣的結果，但針對較複雜的案例，則可能需要使用 Pig，或甚至撰寫自己的 map 和 reduce 元件。 您也可以在進行 Hive 或 Pig 試驗之後，再判定自訂 map 和 reduce 元件是否能夠透過讓您對處理進行微調和最佳化，來提供更佳的效能。

## <a name="custom-mapreduce-components"></a>自訂 Map/Reduce 元件

Map/Reduce 程式碼是由以 **map** 和 **reduce** 元件來實作的兩個個別函式所組成。 **map** 元件會在多個叢集節點上平行執行，其中每個節點都會將對應套用至節點自己的資料子集。 **reduce** 元件會對來自所有 map 函式的結果進行整理和摘要。 如需有關這兩個元件的詳細資訊，請參閱[在 HDInsight 上的 Hadoop 中使用 MapReduce](hdinsight-use-mapreduce.md)。

在大多數 HDInsight 處理案例中，使用較高階的抽象 (例如 Pig 或 Hive) 會較簡單且更有效率。 您也可以建立要在 Hive 指令碼內使用的自訂 map 和 reduce 元件，以執行更精細的處理。

自訂 Map/Reduce 元件通常會以 Java 撰寫。 Hadoop 有提供一個串流介面，可一併允許使用以其他語言 (例如 C#、F#、Visual Basic、Python 及 JavaScript) 開發的元件。

* 如需有關開發自訂 Java MapReduce 程式的逐步解說，請參閱[開發適用於 HDInsight 上 Hadoop 的 Java MapReduce 程式](apache-hadoop-develop-deploy-java-mapreduce-linux.md)。
* 若要查看使用 Python 的範例，請參閱[開發 HDInsight 的 Python 串流處理 MapReduce 程式](apache-hadoop-streaming-python.md)。

針對下列情況，請考慮建立您自己的 map 和 reduce 元件：

* 您需要剖析資料並使用自訂邏輯從該資料取得結構化資訊，來處理完全非結構化的資料。
* 您想要執行很難 (或無法) 在不訴諸建立 UDF 的情況下以 Pig 或 Hive 表達的複雜工作。 例如，您可能需要使用外部地理編碼服務，將來源資料中的緯度和經度座標或 IP 位址，轉換成地理位置名稱。
* 您想要使用 Hadoop 串流介面，在 Map/Reduce 元件中重複使用現有的 .NET、Python 或 JavaScript 程式碼。

## <a name="upload-and-run-your-custom-mapreduce-program"></a>上傳並執行自訂 MapReduce 程式

最常見的 MapReduce 程式是以 Java 撰寫並編譯成 Jar 檔案的程式。

1. 在您開發、編譯及測試 MapReduce 程式之後，請使用 `scp` 命令將 Jar 檔案上傳至前端節點。

    ```bash
    scp mycustomprogram.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
    ```

    將 **USERNAME** 取代為您叢集的 SSH 使用者帳戶。 將 **CLUSTERNAME** 取代為叢集名稱。 如果您使用密碼來保護 SSH 帳戶，系統會提示您輸入密碼。 如果您使用憑證，可能需要使用 `-i` 參數來指定私密金鑰檔案。

2. 使用 [SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)來連線至叢集。

    ```bash
    ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
    ```

3. 從 SSH 工作階段中，透過 YARN 執行您的 MapReduce 程式。

    ```bash
    yarn jar mycustomprogram.jar mynamespace.myclass /example/data/sample.log /example/data/logoutput
    ```

    此命令會向 YARN 提交 MapReduce 作業。 輸入檔案為 `/example/data/sample.log`，輸出目錄則為 `/example/data/logoutput`。 輸入檔及所有輸出檔都會儲存至叢集的預設儲存體。

## <a name="next-steps"></a>後續步驟

* [搭配 HDInsight 的 Hadoop 上的 MapReduce 串流使用 C#](apache-hadoop-dotnet-csharp-mapreduce-streaming.md)
* [開發適用於 HDInsight 上 Hadoop 的 Java MapReduce 程式](apache-hadoop-develop-deploy-java-mapreduce-linux.md)
* [開發 HDInsight 的 Python 資料流 MapReduce 程式](apache-hadoop-streaming-python.md)
* [使用適用於 Eclipse 的 Azure 工具組建立適用於 HDInsight 叢集的 Spark 應用程式](../spark/apache-spark-eclipse-tool-plugin.md)
* [在 HDInsight 上搭配 Hive 和 Pig 使用 Python 使用者定義函數 (UDF)](python-udf-hdinsight.md)
