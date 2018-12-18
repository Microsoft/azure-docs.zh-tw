---
title: 開始在 Azure Data Lake Analytics 中使用 U-SQL 語言
description: 了解 Azure Data Lake Analytics 中 U-SQL 語言的基本概念。
services: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 57143396-ab86-47dd-b6f8-613ba28c28d2
ms.service: data-lake-analytics
ms.topic: conceptual
ms.date: 06/23/2017
ms.openlocfilehash: daef00bc7b39326b79e5f56bf803dc6aad1ac0ff
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221929"
---
# <a name="get-started-with-u-sql-in-azure-data-lake-analytics"></a>開始在 Azure Data Lake Analytics 中使用 U-SQL
U-SQL 是一種語言，結合了宣告式 SQL 與命令式 C#，可讓您處理任何規模的資料。 透過 U-SQL 的可調整分散式查詢功能，您可以有效率地分析各關聯式存放區 (Azure SQL Database) 中的資料。 使用 U-SQL，您可以透過在讀取時套用結構描述並插入自訂邏輯和 UDF，來處理非結構化資料。 此外，U-SQL 所含有的擴充性可讓您細微控制如何大規模執行。 

## <a name="learning-resources"></a>學習資源

* [U-SQL 教學課程](http://aka.ms/usqltutorial)提供大多數 U-SQL 語言的引導式逐步解說。 此文件的建議閱讀對象是所有想要學習 U-SQL 的開發人員。
* 如需 **U-SQL 語言語法**的詳細資訊，請參閱 [U-SQL 語言參考 (英文)](http://go.microsoft.com/fwlink/p/?LinkId=691348)。
* 若要了解 U-SQL 的設計原理，請參閱 Visual Studio 部落格文章[簡介 U-SQL – 讓巨量資料的處理變簡單的語言 (英文)](https://blogs.msdn.microsoft.com/visualstudio/2015/09/28/introducing-u-sql-a-language-that-makes-big-data-processing-easy/)。

## <a name="prerequisites"></a>必要條件

在進行此文章中的 U-SQL 範例前，請閱讀並完成[教學課程：使用適用於 Visual Studio 的 Data Lake 工具開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)。 此教學課程說明將 U-SQL 搭配 Azure Data Lake Tools for Visual Studio 使用的機制。

## <a name="your-first-u-sql-script"></a>您的第一個 U-SQL 指令碼

以下是一個簡單的 U-SQL 指令碼，可讓我們探索 U-SQL 語言的許多層面。

```
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

OUTPUT @searchlog   
    TO "/output/SearchLog-first-u-sql.csv"
    USING Outputters.Csv();
```

此指令碼沒有任何轉換步驟。 它會從名為 `SearchLog.tsv` 的原始程式檔讀取資料，為其建立結構描述，並將資料列集回寫到名為 SearchLog-first-u-sql.csv 檔案。

請注意 `Duration` 欄位中資料類型旁的問號， 它表示 `Duration` 欄位可能是 null。

### <a name="key-concepts"></a>重要概念
* **資料列集變數**：每個會產生資料列集的查詢運算式都可以指派給變數。 在指令碼中，U-SQL 會遵循 T-SQL 變數命名模式 (例如 `@searchlog`)。
* EXTRACT 關鍵字會從檔案讀取資料，並在讀取時定義結構描述。 `Extractors.Tsv` 是內建的 U-SQL 擷取器，適用於以定位點分隔值的檔案。 您可以開發自訂擷取器。
* OUTPUT 會將資料列集的資料寫入檔案。 `Outputters.Csv()` 是內建的 U-SQL 輸出器，用於建立以逗號分隔值的檔案。 您可以開發自訂輸出器。

### <a name="file-paths"></a>檔案路徑

EXTRACT 與 OUTPUT 陳述式使用檔案路徑。 檔案路徑可以是絕對或相對路徑：

下面這個絕對檔案路徑會參考名為 `mystore` 的 Data Lake Store 中的檔案：

    adl://mystore.azuredatalakestore.net/Samples/Data/SearchLog.tsv

下面這個檔案路徑的開頭為 `"/"`。 它會參考預設 Data Lake Store 帳戶中的檔案：

    /output/SearchLog-first-u-sql.csv

## <a name="use-scalar-variables"></a>使用純量變數

您也可以使用純量變數以方便維護指令碼。 前述 U-SQL 指令碼也可以撰寫成：

    DECLARE @in  string = "/Samples/Data/SearchLog.tsv";
    DECLARE @out string = "/output/SearchLog-scalar-variables.csv";

    @searchlog =
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM @in
        USING Extractors.Tsv();

    OUTPUT @searchlog   
        TO @out
        USING Outputters.Csv();

## <a name="transform-rowsets"></a>轉換資料列集

使用 **SELECT** 來轉換資料列集：

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

    @rs1 =
        SELECT Start, Region, Duration
        FROM @searchlog
    WHERE Region == "en-gb";

    OUTPUT @rs1   
        TO "/output/SearchLog-transform-rowsets.csv"
        USING Outputters.Csv();

WHERE 子句使用 [C# 布林運算式](https://msdn.microsoft.com/library/6a71f45d.aspx)。 您可以使用 C# 運算式語言來建置自己的運算式和函式。 您甚至可以將它們與邏輯結合 (AND) 和邏輯分離 (OR) 做結合，以執行更複雜的篩選。

下列指令碼使用 DateTime.Parse() 方法和邏輯結合。

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

    @rs1 =
        SELECT Start, Region, Duration
        FROM @searchlog
    WHERE Region == "en-gb";

    @rs1 =
        SELECT Start, Region, Duration
        FROM @rs1
        WHERE Start >= DateTime.Parse("2012/02/16") AND Start <= DateTime.Parse("2012/02/17");

    OUTPUT @rs1   
        TO "/output/SearchLog-transform-datetime.csv"
        USING Outputters.Csv();

 >[!NOTE]
 >第二個查詢會對第一個資料列集的結果起作用，因而建立兩個篩選條件的組合。 您也可以重複使用變數名稱，因為它們是語彙範圍型名稱。

## <a name="aggregate-rowsets"></a>彙總資料列集
U-SQL 提供您已熟悉使用的 ORDER BY、GROUP BY 和各種彙總語法。

下列查詢會尋找每個區域的總持續時間，然後按順序顯示前五大持續時間。

U-SQL 資料列集不會保留它們的順序以供下一次查詢使用。 因此，若要對輸出排序，您需要將 ORDER BY 新增至 OUTPUT 陳述式：

    DECLARE @outpref string = "/output/Searchlog-aggregation";
    DECLARE @out1    string = @outpref+"_agg.csv";
    DECLARE @out2    string = @outpref+"_top5agg.csv";

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

    @rs1 =
        SELECT
            Region,
            SUM(Duration) AS TotalDuration
        FROM @searchlog
    GROUP BY Region;

    @res =
        SELECT *
        FROM @rs1
        ORDER BY TotalDuration DESC
        FETCH 5 ROWS;

    OUTPUT @rs1
        TO @out1
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();

    OUTPUT @res
        TO @out2
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();

U-SQL ORDER BY 子句必須在 SELECT 運算式中使用 FETCH 子句。

U-SQL 的 HAVING 子句可以用來將輸出限制為符合 HAVING 條件的群組：

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

    @res =
        SELECT
            Region,
            SUM(Duration) AS TotalDuration
        FROM @searchlog
        GROUP BY Region
        HAVING SUM(Duration) > 200;

    OUTPUT @res
        TO "/output/Searchlog-having.csv"
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();

如需進階的彙總案例，請參閱 U-SQL 的[彙總、分析及參考函式](https://msdn.microsoft.com/library/azure/mt621335.aspx)參考文件

## <a name="next-steps"></a>後續步驟
* [Microsoft Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)
* [使用 Data Lake Tools for Visual Studio 來開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)
