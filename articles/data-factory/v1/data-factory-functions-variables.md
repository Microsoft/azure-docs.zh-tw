---
title: Data Factory 函式與系統變數 | Microsoft Docs
description: 提供 Azure Data Factory 函式與系統變數清單
documentationcenter: ''
author: sharonlo101
manager: craigg
services: data-factory
ms.assetid: b6b3c2ae-b0e8-4e28-90d8-daf20421660d
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: b4d9a684c2c21ed9ec00b04963432f9ebcff7493
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048449"
---
# <a name="azure-data-factory---functions-and-system-variables"></a>Azure Data Factory - 函式與系統變數
> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱 [Data Factory 中的系統變數](../control-flow-system-variables.md)。

本文提供 Azure Data Factory 支援之函式和變數的相關資訊。

## <a name="data-factory-system-variables"></a>Data Factory 系統變數
| 變數名稱 | 說明 | 物件範圍 | JSON 範圍和使用案例 |
| --- | --- | --- | --- |
| WindowStart |目前活動執行時段的時間間隔開始 |activity |<ol><li>指定資料選取範圍查詢。 請參閱[資料移動活動](data-factory-data-movement-activities.md)文章中參考的連接器文章。</li> |
| WindowEnd |目前活動執行時段的時間間隔結束 |活動 |與 WindowStart 相同。 |
| SliceStart |所產生之資料配量的時間間隔開始 |activity<br/>資料集 |<ol><li>指定使用 [Azure Blob](data-factory-azure-blob-connector.md) 和[檔案系統資料集](data-factory-onprem-file-system-connector.md)時的動態資料夾路徑與檔案名稱。</li><li>使用 Data Factory 函式在活動輸入集合中指定輸入相依性。</li></ol> |
| SliceEnd |目前資料配量的時間間隔結束。 |activity<br/>資料集 |與 SliceStart 相同。 |

> [!NOTE]
> 目前 Data Factory 需要活動中指定的排程與輸出資料集之可用性中指定的排程完全相符。 因此，WindowStart、WindowEnd、SliceStart 與 SliceEnd 一律對應到相同的時間期間和單一輸出配量。
> 

### <a name="example-for-using-a-system-variable"></a>使用系統變數的範例
在下列範例中，**SliceStart** 的年、月、日和時間會擷取到 **folderPath** 和 **fileName** 屬性所使用的個別變數。

```json
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
 [
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```

## <a name="data-factory-functions"></a>Data Factory 函式
您可以針對下列目的搭配使用 Data Factory 中的函式與系統變數：

1. 指定資料選取範圍查詢 (請參閱 [資料移動活動](data-factory-data-movement-activities.md) 文章中參考的連接器文章)。
   
   針對資料選取範圍查詢和活動與資料集中的其他屬性，叫用 Data Factory 函式的語法是： **$$<function>** 。  
2. 使用 Data Factory 函式在活動輸入集合中指定輸入相依性。
   
    指定輸入相依性運算式不需要 $$。     

在下列範例中，JSON 檔案中的 **sqlReaderQuery** 屬性指派給 `Text.Format` 函式所傳回的值。 此範例也會使用名為 **WindowStart**的系統變數，它代表活動執行時段的開始時間。

```json
{
    "Type": "SqlSource",
    "sqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
}
```

請參閱說明可使用的各種格式化選項 (例如：ay 與 yyyy) 的[自訂日期和時間格式字串](https://msdn.microsoft.com/library/8kb3ddd4.aspx)主題。 

### <a name="functions"></a>Functions
下表列出 Azure Data Factory 中的所有函式：

| 類別 | 函式 | 參數 | 說明 |
| --- | --- | --- | --- |
| 時間 |AddHours(X,Y) |X：DateTime  <br/><br/>Y：int |將 Y 小時新增至指定時間 X。 <br/><br/>範例：`9/5/2013 12:00:00 PM + 2 hours = 9/5/2013 2:00:00 PM` |
| 時間 |AddMinutes(X,Y) |X：DateTime  <br/><br/>Y：int |將 Y 分鐘新增至 X。<br/><br/>範例： `9/15/2013 12: 00:00 PM + 15 minutes = 9/15/2013 12: 15:00 PM` |
| 時間 |StartOfHour(X) |X：DateTime  |取得 X 之小時元件代表的小時開始時間。 <br/><br/>範例： `StartOfHour of 9/15/2013 05: 10:23 PM is 9/15/2013 05: 00:00 PM` |
| 日期 |AddDays(X,Y) |X：DateTime <br/><br/>Y：int |將 Y 天數新增至 X。 <br/><br/>範例：9/15/2013 12:00:00 PM + 2 天 = 9/17/2013 12:00:00 PM。<br/><br/>您也可以藉由將 Y 指定為負數來減去天。<br/><br/>範例： `9/15/2013 12:00:00 PM - 2 days = 9/13/2013 12:00:00 PM`. |
| 日期 |AddMonths(X,Y) |X：DateTime <br/><br/>Y：int |將 Y 月數新增至 X。<br/><br/>`Example: 9/15/2013 12:00:00 PM + 1 month = 10/15/2013 12:00:00 PM` 。<br/><br/>您也可以藉由將 Y 指定為負數來減去月份。<br/><br/>範例：`9/15/2013 12:00:00 PM - 1 month = 8/15/2013 12:00:00 PM`.|
| 日期 |AddQuarters(X,Y) |X：DateTime  <br/><br/>Y：int |將 Y * 3 個月新增至 X。<br/><br/>範例： `9/15/2013 12:00:00 PM + 1 quarter = 12/15/2013 12:00:00 PM` |
| 日期 |AddWeeks(X,Y) |X：DateTime <br/><br/>Y：int |將 Y * 7 天新增至 X<br/><br/>範例：9/15/2013 12:00:00 PM + 1 週 = 9/22/2013 12:00:00 PM<br/><br/>您也可以藉由將 Y 指定為負數來減去週。<br/><br/>範例： `9/15/2013 12:00:00 PM - 1 week = 9/7/2013 12:00:00 PM`. |
| 日期 |AddYears(X,Y) |X：DateTime <br/><br/>Y：int |將 Y 年新增至 X。<br/><br/>`Example: 9/15/2013 12:00:00 PM + 1 year = 9/15/2014 12:00:00 PM`<br/><br/>您也可以藉由將 Y 指定為負數來減去年。<br/><br/>範例： `9/15/2013 12:00:00 PM - 1 year = 9/15/2012 12:00:00 PM`. |
| 日期 |Day(X) |X：DateTime  |取得 X 的日元件。<br/><br/>範例： `Day of 9/15/2013 12:00:00 PM is 9`. |
| 日期 |DayOfWeek(X) |X：DateTime  |取得 X 的週中日元件。<br/><br/>範例： `DayOfWeek of 9/15/2013 12:00:00 PM is Sunday`. |
| 日期 |DayOfYear(X) |X：DateTime  |取得 X 之年元件代表的一年當中日期。<br/><br/>範例：<br/>`12/1/2015: day 335 of 2015`<br/>`12/31/2015: day 365 of 2015`<br/>`12/31/2016: day 366 of 2016 (Leap Year)` |
| 日期 |DaysInMonth(X) |X：DateTime  |取得參數 X 之月元件代表的月份中日期。<br/><br/>範例： `DaysInMonth of 9/15/2013 are 30 since there are 30 days in the September month`. |
| 日期 |EndOfDay(X) |X：DateTime  |取得 X 之結尾天數 (日元件) 代表的日期時間。<br/><br/>範例： `EndOfDay of 9/15/2013 05:10:23 PM is 9/15/2013 11:59:59 PM`. |
| 日期 |EndOfMonth(X) |X：DateTime  |取得參數 X 之月元件代表的月底。 <br/><br/>範例：`EndOfMonth of 9/15/2013 05:10:23 PM is 9/30/2013 11:59:59 PM` (代表 9 月份月底的日期時間) |
| 日期 |StartOfDay(X) |X：DateTime  |取得參數 X 之日元件代表的日期開始。<br/><br/>範例： `StartOfDay of 9/15/2013 05:10:23 PM is 9/15/2013 12:00:00 AM`. |
| Datetime |From(X) |X：字串 |將字串 X 剖析為日期時間。 |
| Datetime |Ticks(X) |X：DateTime  |取得參數 X 的刻度屬性。一個刻度等於 100 奈秒。 這個屬性的值代表從 0001 年 1 月 1 日午夜 12:00:00 經過的刻度數。 |
| 文字 |Format(X) |X：字串變數 |將文字格式化 (使用 `\\'` 組合來逸出 `'` 字元)。|

> [!IMPORTANT]
> 在另一個函式中使用函式時，您不需要針對內部函式使用 **$$** 前置詞。 例如：$$Text.Format('PartitionKey eq \\'my_pkey_filter_value\\' and RowKey ge \\'{0: yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(SliceStart, -6)). 在此範例中，請注意 **$$** 前置詞不能用於 **Time.AddHours** 函式。 

#### <a name="example"></a>範例
在下列範例中，Hive 活動的輸入和輸出參數是經由使用 `Text.Format` 函式和 SliceStart 系統變數決定。 

```json  
{
    "name": "HiveActivitySamplePipeline",
        "properties": {
    "activities": [
            {
            "name": "HiveActivitySample",
            "type": "HDInsightHive",
            "inputs": [
                    {
                    "name": "HiveSampleIn"
                    }
            ],
            "outputs": [
                    {
                    "name": "HiveSampleOut"
                }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "typeproperties": {
                    "scriptPath": "adfwalkthrough\\scripts\\samplehive.hql",
                    "scriptLinkedService": "StorageLinkedService",
                    "defines": {
                        "Input": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/samplein/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)",
                        "Output": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/sampleout/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)"
                    },
                    "scheduler": {
                        "frequency": "Hour",
                        "interval": 1
                }
            }
            }
    ]
    }
}
```

### <a name="example-2"></a>範例 2

在下列範例中，預存程序活動的日期時間參數是經由使用 Text.Format 函式 和 SliceStart 變數決定。 

```json
{
    "name": "SprocActivitySamplePipeline",
    "properties": {
        "activities": [
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "sp_sample",
                    "storedProcedureParameters": {
                        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)"
                    }
                },
                "outputs": [
                    {
                        "name": "sprocsampleout"
                    }
                ],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "SprocActivitySample"
            }
        ],
            "start": "2016-08-02T00:00:00Z",
            "end": "2016-08-02T05:00:00Z",
        "isPaused": false
    }
}
```

### <a name="example-3"></a>範例 3
若要讀取前一天的資料，而不是由 SliceStart 呈現的資料，請使用 AddDays 函式，如下列範例所示： 

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-01-01T08:00:00",
        "end": "2017-01-01T11:00:00",
        "description": "hive activity",
        "activities": [
            {
                "name": "SampleHiveActivity",
                "inputs": [
                    {
                        "name": "MyAzureBlobInput",
                        "startTime": "Date.AddDays(SliceStart, -1)",
                        "endTime": "Date.AddDays(SliceEnd, -1)"
                    }
                ],
                "outputs": [
                    {
                        "name": "MyAzureBlobOutput"
                    }
                ],
                "linkedServiceName": "HDInsightLinkedService",
                "type": "HDInsightHive",
                "typeProperties": {
                    "scriptPath": "adftutorial\\hivequery.hql",
                    "scriptLinkedService": "StorageLinkedService",
                    "defines": {
                        "Year": "$$Text.Format('{0:yyyy}',WindowsStart)",
                        "Month": "$$Text.Format('{0:MM}',WindowStart)",
                        "Day": "$$Text.Format('{0:dd}',WindowStart)"
                    }
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 2,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

請參閱說明可使用的各種格式化選項 (例如：yy 與 yyyy) 的 [自訂日期和時間格式字串](https://msdn.microsoft.com/library/8kb3ddd4.aspx) 主題。 

