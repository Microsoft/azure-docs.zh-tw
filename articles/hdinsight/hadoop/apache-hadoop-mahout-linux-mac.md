---
title: 在 Azure HDInsight 中使用 Apache Mahout 產生推薦
description: 了解如何搭配 HDInsight 使用 Apache Mahout 機器學習程式庫來產生電影推薦。
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 05/14/2020
ms.openlocfilehash: c31ffaf094801bdd49e5800bd338a15d8b8315f6
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98946492"
---
# <a name="generate-recommendations-using-apache-mahout-in-azure-hdinsight"></a>在 Azure HDInsight 中使用 Apache Mahout 產生推薦

了解如何使用搭配 Azure HDInsight 的 [Apache Mahout](https://mahout.apache.org) 機器學習庫產生電影推薦。

Mahout 是 Apache Hadoop 的[機器學習](https://en.wikipedia.org/wiki/Machine_learning) \(英文\) 程式庫。 Mahout 包含可處理資料的演算法，例如篩選、分類和叢集化。 在本文中，您會使用推薦引擎，以根據朋友看過的電影來產生電影推薦。

如需 HDInsight 中 Mahout 版本的詳細資訊，請參閱 [HDInsight 版本和 Apache Hadoop 元件](../hdinsight-component-versioning.md)。

## <a name="prerequisites"></a>Prerequisites

HDInsight 上的 Apache Hadoop 叢集。 請參閱[開始在 Linux 上使用 HDInsight](./apache-hadoop-linux-tutorial-get-started.md)。

## <a name="understanding-recommendations"></a>了解推薦

Mahout 提供的其中一項功能是推薦引擎。 這個引擎接受 `userID`、`itemId` 和 `prefValue` (項目的喜好設定) 格式的資料。 Mahout 接著可以執行共生分析來判斷：「偏好某項目的使用者同時也偏好其他這些項目」。 接著 Mahout 會以偏好的類似項目判斷使用者，並以此做出推薦。

以下工作流程是一個使用電影資料的簡化範例：

* **共生**：Joe、Alice 和 Bob 都喜歡 *《星際大戰》* 、 *《帝國大反擊》* 和 *《絕地大反攻》* 。 Mahout 將判斷喜歡上述任何一部電影的使用者，也會喜歡另外兩部電影。

* **共生**：Bob 和 Alice 同時也喜歡 *《威脅潛伏》* 、 *《複製人全面進攻》* 和 *《西斯大帝的復仇》* 。 Mahout 將判斷喜歡前三部電影的使用者，也會喜歡這三部電影。

* **相似性推薦**：因為 Joe 喜歡前三部電影，Mahout 會查看具有相似偏好的其他使用者所喜歡但 Joe 還沒看過 (喜歡/評價) 的電影。 在此情況下，Mahout 將會推薦 *《威脅潛伏》* 、 *《複製人全面進攻》* 和 *《西斯大帝的復仇》* 。

### <a name="understanding-the-data"></a>了解資料

[GroupLens 研究](https://grouplens.org/datasets/movielens/) \(英文\) 提供 Mahout 相容格式的電影評價資料，相當方便。 您可在位於 `/HdiSamples/HdiSamples/MahoutMovieData`的叢集預設儲存體取得這份資料。

有兩個檔案 `moviedb.txt` 和 `user-ratings.txt`。 `user-ratings.txt` 檔案是用於分析期間。 檢視結果時，`moviedb.txt` 用來提供使用者易記的文字資訊。

`user-ratings.txt` 內包含的資料具有 `userID`、`movieID`、`userRating` 和 `timestamp` 結構，可指出每位使用者對於影片的評價為何。 以下是資料範例：

```output
    196    242    3    881250949
    186    302    3    891717742
    22     377    1    878887116
    244    51     2    880606923
    166    346    1    886397596
```

## <a name="run-the-analysis"></a>執行分析

1. 使用 [ssh 命令](../hdinsight-hadoop-linux-use-ssh-unix.md)來連線到您的叢集。 編輯以下命令並將 CLUSTERNAME 取代為您叢集的名稱，然後輸入命令：

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. 使用下列命令來執行推薦工作：

    ```bash
    mahout recommenditembased -s SIMILARITY_COOCCURRENCE -i /HdiSamples/HdiSamples/MahoutMovieData/user-ratings.txt -o /example/data/mahoutout --tempDir /temp/mahouttemp
    ```

> [!NOTE]  
> 此工作可能需要幾分鐘的時間才能完成，並可能執行多項 MapReduce 工作。

## <a name="view-the-output"></a>檢視輸出

1. 工作完成後，使用以下命令來檢視所產生的輸出：

    ```bash
    hdfs dfs -text /example/data/mahoutout/part-r-00000
    ```

    輸出看起來會像下面這樣：

    ```output
    1    [234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
    2    [282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
    3    [284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
    4    [690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]
    ```

    第一欄是 `userID`。 '[' 和 ']' 中包含的值是 `movieId`:`recommendationScore`。

2. 您可以使用輸出和 moviedb.txt，來顯示更多建議的相關資訊。 首先，使用下列命令將檔案複製到本機上︰

    ```bash
    hdfs dfs -get /example/data/mahoutout/part-r-00000 recommendations.txt
    hdfs dfs -get /HdiSamples/HdiSamples/MahoutMovieData/* .
    ```

    這個命令會將輸出資料和影片資料檔一起複製到目前目錄中名為 **recommendations.txt** 的檔案中。

3. 使用下列命令來建立 Python 指令碼，該指令碼會在建議輸出資料中查閱影片名稱：

    ```bash
    nano show_recommendations.py
    ```

    開啟編輯器時，請使用下列文字做為檔案的內容：

   ```python
   #!/usr/bin/env python

   import sys

   if len(sys.argv) != 5:
        print "Arguments: userId userDataFilename movieFilename recommendationFilename"
        sys.exit(1)

   userId, userDataFilename, movieFilename, recommendationFilename = sys.argv[1:]

   print "Reading Movies Descriptions"
   movieFile = open(movieFilename)
   movieById = {}
   for line in movieFile:
       tokens = line.split("|")
       movieById[tokens[0]] = tokens[1:]
   movieFile.close()

   print "Reading Rated Movies"
   userDataFile = open(userDataFilename)
   ratedMovieIds = []
   for line in userDataFile:
       tokens = line.split("\t")
       if tokens[0] == userId:
           ratedMovieIds.append((tokens[1],tokens[2]))
   userDataFile.close()

   print "Reading Recommendations"
   recommendationFile = open(recommendationFilename)
   recommendations = []
   for line in recommendationFile:
       tokens = line.split("\t")
       if tokens[0] == userId:
           movieIdAndScores = tokens[1].strip("[]\n").split(",")
           recommendations = [ movieIdAndScore.split(":") for movieIdAndScore in movieIdAndScores ]
           break
   recommendationFile.close()

   print "Rated Movies"
   print "------------------------"
   for movieId, rating in ratedMovieIds:
       print "%s, rating=%s" % (movieById[movieId][0], rating)
   print "------------------------"

   print "Recommended Movies"
   print "------------------------"
   for movieId, score in recommendations:
       print "%s, score=%s" % (movieById[movieId][0], score)
   print "------------------------"
   ```

    按下 **CTRL-X**、**Y**，最後再按 **Enter** 鍵儲存資料。

4. 執行 Python 指令碼。 以下命令假設您已在所有檔案的下載目錄中︰

    ```bash
    python show_recommendations.py 4 user-ratings.txt moviedb.txt recommendations.txt
    ```

    此命令會查看為使用者 ID 4 所產生的建議。

   * **user-ratings.txt** 檔案可用來擷取已評分的影片。

   * **moviedb.txt** 檔案用來擷取影片名稱。

   * **recommendations.txt** 用來擷取這位使用者的電影建議。

     此命令的輸出類似下列文字︰

        ```output
        Seven Years in Tibet (1997), score=5.0
        Indiana Jones and the Last Crusade (1989), score=5.0
        Jaws (1975), score=5.0
        Sense and Sensibility (1995), score=5.0
        Independence Day (ID4) (1996), score=5.0
        My Best Friend's Wedding (1997), score=5.0
        Jerry Maguire (1996), score=5.0
        Scream 2 (1997), score=5.0
        Time to Kill, A (1996), score=5.0
        ```

## <a name="delete-temporary-data"></a>刪除暫存資料

Mahout 工作不會移除處理工作時所建立的暫存資料。 範例工作中指定 `--tempDir` 參數將暫存檔隔離到特定路徑中以方便刪除。 若要移除暫存檔案，請使用下列命令：

```bash
hdfs dfs -rm -f -r /temp/mahouttemp
```

> [!WARNING]  
> 如果您要再次執行該命令，您必須也刪除輸出目錄。 使用以下命令刪除此目錄：
>
> `hdfs dfs -rm -f -r /example/data/mahoutout`

## <a name="next-steps"></a>後續步驟

您現在已了解如何使用 Mahout，請繼續探索在 HDInsight 上使用資料的其他方法：

* [搭配 HDInsight 使用 Apache Hive](hdinsight-use-hive.md)
* [搭配 HDInsight 使用 MapReduce](hdinsight-use-mapreduce.md)
