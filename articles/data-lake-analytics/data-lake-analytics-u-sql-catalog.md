---
title: 開始在 Azure Data Lake Analytics 中使用 U-SQL 目錄
description: 了解如何使用 U-SQL 目錄來共用程式碼和資料。
services: data-lake-analytics
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 57143396-ab86-47dd-b6f8-613ba28c28d2
ms.topic: conceptual
ms.date: 05/09/2017
ms.openlocfilehash: 62f43fc082969bf04b7177725478585ce41aa347
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045933"
---
# <a name="get-started-with-the-u-sql-catalog-in-azure-data-lake-analytics"></a>開始在 Azure Data Lake Analytics 中使用 U-SQL 目錄

## <a name="create-a-tvf"></a>建立 TVF

在先前的 U-SQL 指令碼中，您重複使用 EXTRACT 來從相同原始程式檔進行讀取。 使用 U-SQL 資料表值函式 (TVF)，您就可以封裝資料以供日後重複使用。  

下列指令碼會在預設的資料庫和結構描述中建立名為 `Searchlog()` 的 TVF：

```
DROP FUNCTION IF EXISTS Searchlog;

CREATE FUNCTION Searchlog()
RETURNS @searchlog TABLE
(
            UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
)
AS BEGIN
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
USING Extractors.Tsv();
RETURN;
END;
```

下列指令碼會示範如何使用先前的指令碼中定義的 TVF：

```
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM Searchlog() AS S
GROUP BY Region
HAVING SUM(Duration) > 200;

OUTPUT @res
    TO "/output/SerachLog-use-tvf.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

## <a name="create-views"></a>建立檢視

如果您有單一查詢運算式，則您可以不使用 TVF，而是使用「U-SQL 檢視」來封裝該運算式。

下列指令碼會在預設的資料庫和結構描述中建立名為 `SearchlogView` 的 視：

```
DROP VIEW IF EXISTS SearchlogView;

CREATE VIEW SearchlogView AS  
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
USING Extractors.Tsv();
```

下列指令碼示範如何使用定義的檢視：

```
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM SearchlogView
GROUP BY Region
HAVING SUM(Duration) > 200;

OUTPUT @res
    TO "/output/Searchlog-use-view.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

## <a name="create-tables"></a>建立資料表
和關聯式資料庫資料表一樣，使用 U-SQL 可讓您使用預先定義的結構描述建立資料表，或建立資料表以從填入資料表的查詢推斷結構描述 (也就是 CREATE TABLE AS SELECT 或 CTAS)。

使用下列指令碼建立一個資料庫和兩個資料表：

```
DROP DATABASE IF EXISTS SearchLogDb;
CREATE DATABASE SearchLogDb;
USE DATABASE SearchLogDb;

DROP TABLE IF EXISTS SearchLog1;
DROP TABLE IF EXISTS SearchLog2;

CREATE TABLE SearchLog1 (
            UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string,

            INDEX sl_idx CLUSTERED (UserId ASC)
                DISTRIBUTED BY HASH (UserId)
);

INSERT INTO SearchLog1 SELECT * FROM master.dbo.Searchlog() AS s;

CREATE TABLE SearchLog2(
    INDEX sl_idx CLUSTERED (UserId ASC)
            DISTRIBUTED BY HASH (UserId)
) AS SELECT * FROM master.dbo.Searchlog() AS S; // You can use EXTRACT or SELECT here
```

## <a name="query-tables"></a>查詢資料表
您可以運用和查詢資料檔案一樣的方式來查詢資料表 (例如，上一個指令碼所建立的資料表)。 您現在可以直接參考資料表名稱，而不必使用 EXTRACT 建立資料列集。

若要從資料表進行讀取，請修改您先前使用的轉換指令碼：

```
@rs1 =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM SearchLogDb.dbo.SearchLog2
GROUP BY Region;

@res =
    SELECT *
    FROM @rs1
    ORDER BY TotalDuration DESC
    FETCH 5 ROWS;

OUTPUT @res
    TO "/output/Searchlog-query-table.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

 >[!NOTE]
 >您目前無法在用來建立資料表的相同指令碼中，對該資料表執行 SELECT。

## <a name="next-steps"></a>後續步驟
* [Microsoft Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)
* [使用 Data Lake Tools for Visual Studio 開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)
* [使用 Azure 入口網站監視和疑難排解 Azure Data Lake Analytics 作業](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
