---
title: 利用 Azure HDInsight 中的 Apache Hive 分析和處理 JSON 文件
description: 了解如何使用 JSON 文件，以集使用 Azure HDInsight 中的 Hive 分析它們
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: 5388d0d6c05cdfd60f3f840761a2cf87d4b6a41e
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046196"
---
# <a name="process-and-analyze-json-documents-by-using-apache-hive-in-azure-hdinsight"></a>使用 Azure HDInsight 中的 Apache Hive 處理並分析 JSON 文件

了解如何使用 Azure HDInsight 中的 Apache Hive 中的處理和分析 JavaScript Object Notation (JSON) 檔案。 本教學課程會使用下列 JSON 文件：

```json
{
  "StudentId": "trgfg-5454-fdfdg-4346",
  "Grade": 7,
  "StudentDetails": [
    {
      "FirstName": "Peggy",
      "LastName": "Williams",
      "YearJoined": 2012
    }
  ],
  "StudentClassCollection": [
    {
      "ClassId": "89084343",
      "ClassParticipation": "Satisfied",
      "ClassParticipationRank": "High",
      "Score": 93,
      "PerformedActivity": false
    },
    {
      "ClassId": "78547522",
      "ClassParticipation": "NotSatisfied",
      "ClassParticipationRank": "None",
      "Score": 74,
      "PerformedActivity": false
    },
    {
      "ClassId": "78675563",
      "ClassParticipation": "Satisfied",
      "ClassParticipationRank": "Low",
      "Score": 83,
      "PerformedActivity": true
    }
  ]
}
```

檔案位於 **wasb://processjson@hditutorialdata.blob.core.windows.net/**。 如需關於搭配 HDInsight 使用 Azure Blob 儲存體的詳細資訊，請參閱[在 HDInsight 上搭配 Hadoop 使用 HDFS 相容的 Azure Blob 儲存體](../hdinsight-hadoop-use-blob-storage.md)。 您可以將檔案複製到叢集的預設容器。

在本教學課程中，您會使用 Hive 主控台。 如需開啟 Hive 主控台的指示，請參閱 [利用遠端桌面搭配使用 Hive 與 HDInsight 上的 Hadoop](apache-hadoop-use-hive-remote-desktop.md)。

## <a name="flatten-json-documents"></a>簡維 JSON 文件
下一節所列的方法需要於單一資料列中撰寫 JSON 文件。 因此，您必須將 JSON 文件壓平合併成一個字串。 如果已壓平合併 JSON 文件，您就可以略過此步驟，直接進入與分析 JSON 資料相關的下一節。 若要壓平合併 JSON 文件，執行下列指令碼：

```sql
DROP TABLE IF EXISTS StudentsRaw;
CREATE EXTERNAL TABLE StudentsRaw (textcol string) STORED AS TEXTFILE LOCATION "wasb://processjson@hditutorialdata.blob.core.windows.net/";

DROP TABLE IF EXISTS StudentsOneLine;
CREATE EXTERNAL TABLE StudentsOneLine
(
  json_body string
)
STORED AS TEXTFILE LOCATION '/json/students';

INSERT OVERWRITE TABLE StudentsOneLine
SELECT CONCAT_WS(' ',COLLECT_LIST(textcol)) AS singlelineJSON
      FROM (SELECT INPUT__FILE__NAME,BLOCK__OFFSET__INSIDE__FILE, textcol FROM StudentsRaw DISTRIBUTE BY INPUT__FILE__NAME SORT BY BLOCK__OFFSET__INSIDE__FILE) x
      GROUP BY INPUT__FILE__NAME;

SELECT * FROM StudentsOneLine
```

原始 JSON 檔案位於 **wasb://processjson@hditutorialdata.blob.core.windows.net/**。 **StudentsRaw** Hive 資料表指向原始未簡維的 JSON 文件。

StudentsOneLine Hive 資料表會將資料儲存在 HDInsight 預設檔案系統的 /json/students/ 路徑下。

**INSERT**陳述式會將壓平合併的 JSON 資料填入 **StudentOneLine** 資料表。

**SELECT** 陳述式應該只會傳回一個資料列。

以下是 **SELECT** 陳述式的輸出：

![壓平合併 JSON 文件][image-hdi-hivejson-flatten]

## <a name="analyze-json-documents-in-hive"></a>在 Hive 中分析 JSON 文件
Hive 提供三種不同的機制，可在 JSON 文件上執行查詢。您也可以自行撰寫：

* 使用 get_json_object 使用者定義函式 (UDF)。
* 使用 json_tuple UDF。
* 使用自訂序列化/還原序列化程式 (SerDe)。
* 使用 Python 或其他語言撰寫您自己的 UDF。 如需有關搭配 Hive 執行您自己的 Python 程式碼的詳細資訊，請參閱[Apache Hive 搭配 Pig Python UDF][hdinsight-python]。

### <a name="use-the-getjsonobject-udf"></a>使用 get_json_object UDF
Hive 提供的內建 UDF 稱為 [get_json_objec](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object)，可在執行階段期間執行 JSON 查詢。 此方法採用兩個引數--資料表名稱和方法名稱，後者具有扁平化的 JSON 文件和必須剖析的 JSON 欄位。 讓我們看看此 UDF 如何運作的範例。

下列查詢會傳回每個學生的姓氏與名字：

```sql
SELECT
  GET_JSON_OBJECT(StudentsOneLine.json_body,'$.StudentDetails.FirstName'),
  GET_JSON_OBJECT(StudentsOneLine.json_body,'$.StudentDetails.LastName')
FROM StudentsOneLine;
```

以下是在主控台視窗中執行此查詢時的輸出：

![get_json_object UDF][image-hdi-hivejson-getjsonobject]

get-json_object UDF 有幾項限制：

* 因為查詢中的每個欄位都需要重新剖析查詢，所以它會影響效能。
* **GET\_JSON_OBJECT()** 會傳回陣列的字串表示法。 若要將此陣列轉換成 Hive 陣列，您必須使用規則運算式來取代方括號「[」和「]」，此外您也要呼叫分割以取得陣列。

這就是 Hive wiki 建議使用 json_tuple 的原因。  

### <a name="use-the-jsontuple-udf"></a>使用 json_tuple UDF
Hive 所提供的另一個 UDF 稱為 [json_tuple](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-json_tuple)，其效能比 [get_ json _object](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object) 更高。 這個方法會採用一組索引鍵和 JSON 字串，並使用一個函式傳回值的 tuple。 下列查詢會從 JSON 文件傳回學生識別碼以及年級：

```sql
SELECT q1.StudentId, q1.Grade
FROM StudentsOneLine jt
LATERAL VIEW JSON_TUPLE(jt.json_body, 'StudentId', 'Grade') q1
  AS StudentId, Grade;
```

這個指令碼在 Hive 主控台的輸出：

![json_tuple UDF][image-hdi-hivejson-jsontuple]

json_tuple UDF 會使用 Hive 中的[橫向檢視](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView)語法，讓 json\_ttuple 將 UDT 函式套用到原始資料表的每個資料列，以建立一個虛擬資料表。 複雜 JSON 會重複使用**橫向檢視**，因此變得難以使用。 此外，**JSON_TUPLE** 無法處理巢狀 JSON。

### <a name="use-a-custom-serde"></a>使用自訂 SerDe
SerDe 是剖析巢狀 JSON 文件的最佳選擇。 它可讓您定義的 JSON 結構描述，然後您可以使用結構來剖析文件。 如需指示，請參閱[如何搭配 Microsoft Azure HDInsight 來使用自訂 JSON SerDe](https://blogs.msdn.microsoft.com/bigdatasupport/2014/06/18/how-to-use-a-custom-json-serde-with-microsoft-azure-hdinsight/)。

## <a name="summary"></a>總結
總而言之，您在 Hive 中選擇的 JSON 運算子類型取決於您的案例。 如果您有一個簡單的 JSON 文件，且您只需要查閱一個欄位 – 您就可以選擇使用 Hive UDF get_json_object。 如果您有多個金鑰需要查閱，可以使用 json_tuple。 如果您有巢狀文件，您應該使用 JSON SerDe。

## <a name="next-steps"></a>後續步驟

如需其他相關文章，請參閱：

* [搭配 HDInsight 中的 Hadoop 使用 Hive 和 HiveQL 來分析範例 Apache Log4j 檔案](../hdinsight-use-hive.md)
* [在 HDInsight 中使用 Hive 分析航班延誤資料](../hdinsight-analyze-flight-delay-data.md)
* [在 HDInsight 中使用 Hive 分析 Twitter 資料](../hdinsight-analyze-twitter-data.md)

[hdinsight-python]:python-udf-hdinsight.md

[image-hdi-hivejson-flatten]: ./media/using-json-in-hive/flatten.png
[image-hdi-hivejson-getjsonobject]: ./media/using-json-in-hive/getjsonobject.png
[image-hdi-hivejson-jsontuple]: ./media/using-json-in-hive/jsontuple.png
[image-hdi-hivejson-jdk]: ./media/hdinsight-using-json-in-hive/jdk.png
[image-hdi-hivejson-maven]: ./media/hdinsight-using-json-in-hive/maven.png
[image-hdi-hivejson-serde]: ./media/hdinsight-using-json-in-hive/serde.png
[image-hdi-hivejson-addjar]: ./media/hdinsight-using-json-in-hive/addjar.png
[image-hdi-hivejson-serde_query1]: ./media/hdinsight-using-json-in-hive/serde_query1.png
[image-hdi-hivejson-serde_query2]: ./media/hdinsight-using-json-in-hive/serde_query2.png
[image-hdi-hivejson-serde_query3]: ./media/hdinsight-using-json-in-hive/serde_query3.png
[image-hdi-hivejson-serde_result]: ./media/hdinsight-using-json-in-hive/serde_result.png

