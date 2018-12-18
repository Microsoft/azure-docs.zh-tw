---
title: 從 Azure Application Insights 匯出至 SQL | Microsoft Docs
description: 使用 Stream Analytics 持續將 Application Insights 資料匯出至 SQL。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 48903032-2c99-4987-9948-d6e4559b4a63
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 03/06/2015
ms.author: mbullwin
ms.openlocfilehash: 7d4bf0c5beeba22569e0000b28b007fb5b0ff68f
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35294171"
---
# <a name="walkthrough-export-to-sql-from-application-insights-using-stream-analytics"></a>逐步解說：使用串流分析從 Application Insights 匯出至 SQL
本文將說明如何使用[連續匯出][export]和 [Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/)，將您的遙測資料從 [Azure Application Insights][start] 移入 Azure SQL Database。 

連續匯出會以 JSON 格式將遙測資料移入 Azure 儲存體。 我們將使用 Azure 串流分析來剖析 JSON 物件，並在資料庫資料表中建立資料列。

(一般來說，「連續匯出」是對應用程式傳送至 Application Insights 的遙測資料自行進行分析的方式。 您可以調整這個程式碼範例，以使用匯出的遙測資料執行其他作業，例如彙總資料)。

我們先假設您已經有想要監視的應用程式。

在此範例中，我們使用頁面檢視資料，但相同的模式可以很輕易地延伸到其他資料類型，例如自訂事件和例外狀況。 

## <a name="add-application-insights-to-your-application"></a>將 Application Insights 加入應用程式中
開始進行之前：

1. [為您的網頁設定 Application Insights](app-insights-javascript.md)。 
   
    (在此範例中，我們將著重於處理來自用戶端瀏覽器的頁面檢視資料，但您也可以針對 [Java](app-insights-java-get-started.md) 或 [ASP.NET](app-insights-asp-net.md) 應用程式的伺服器端設定 Application Insights，並處理要求、相依性及其他伺服器遙測。)
2. 發行應用程式，並觀察出現在 Application Insights 資源中的遙測資料。

## <a name="create-storage-in-azure"></a>在 Azure 中建立儲存體
連續匯出一律會將資料輸出至 Azure 儲存體帳戶，因此您必須先建立儲存體。

1. 在 [Azure 入口網站][portal]的訂用帳戶中建立儲存體帳戶。
   
    ![在 Azure 入口網站中，依序選擇 [新增]、[資料]、[儲存體]。 選取 [傳統]，然後選擇 [建立]。 提供儲存體的名稱。](./media/app-insights-code-sample-export-sql-stream-analytics/040-store.png)
2. 建立容器
   
    ![在新的儲存體中，選取 [容器]，按一下容器磚，然後按一下 [新增]](./media/app-insights-code-sample-export-sql-stream-analytics/050-container.png)
3. 複製儲存體存取金鑰
   
    稍後您會需要使用它來設定串流分析服務的輸入。
   
    ![在儲存體中，依序開啟 [設定]、[金鑰]，然後複製主要存取金鑰](./media/app-insights-code-sample-export-sql-stream-analytics/21-storage-key.png)

## <a name="start-continuous-export-to-azure-storage"></a>啟動對 Azure 儲存體的連續匯出
1. 在 Azure 入口網站中，瀏覽至您為應用程式建立的 Application Insights 資源。
   
    ![依序選擇 [瀏覽]、[Application Insights]、您的應用程式](./media/app-insights-code-sample-export-sql-stream-analytics/060-browse.png)
2. 建立連續匯出。
   
    ![依序選擇 [設定]、[連續匯出]、[新增]](./media/app-insights-code-sample-export-sql-stream-analytics/070-export.png)

    選取您稍早建立的儲存體帳戶：

    ![設定匯出目的地](./media/app-insights-code-sample-export-sql-stream-analytics/080-add.png)

    設定您想要查看的事件類型：

    ![選擇事件類型](./media/app-insights-code-sample-export-sql-stream-analytics/085-types.png)


1. 可讓一些資料累積。 請休息一下，讓其他人使用您的應用程式一段時間。 遙測資料會送過來，而您會在[計量瀏覽器](app-insights-metrics-explorer.md)中看到統計圖表，並在[診斷搜尋](app-insights-diagnostic-search.md)中看到個別事件。 
   
    此外，資料會匯出至您的儲存體。 
2. 在入口網站 (選擇 [瀏覽]、選取您的儲存體帳戶，然後選取 [容器]) 或 Visual Studio 中，檢查匯出的資料。 在 Visual Studio 中，依序選擇 [檢視] 和 [Cloud Explorer]，然後依序開啟 [Azure] 和 [儲存體]。 (如果您沒有此功能表選項，您需要安裝 Azure SDK：開啟 [新增專案] 對話方塊，然後開啟 [Visual C#] / [Cloud] / [取得 Microsoft Azure SDK for .NET]。)
   
    ![在 Visual Studio 中，依序開啟 [Server Browser]、[Azure]、[儲存體]](./media/app-insights-code-sample-export-sql-stream-analytics/087-explorer.png)
   
    記下衍生自應用程式名稱和檢測金鑰之路徑名稱的共同部分。 

事件會以 JSON 格式寫入至 Blob 檔案。 每個檔案可能會包含一或多個事件。 因此我們想要讀取事件資料，並篩選出需要的欄位。 我們可以利用資料執行各種作業，但我們現在打算使用串流分析，將資料移至 SQL Database。 這麼做可讓您輕鬆執行許多有趣的查詢工作。

## <a name="create-an-azure-sql-database"></a>建立 Azure SQL Database
同樣地，請從您在 [Azure 入口網站][portal]中的訂用帳戶開始，建立您將寫入資料的資料庫 (和一部新伺服器，除非您已經有新伺服器)。

![[新增]、[資料]、[SQL]](./media/app-insights-code-sample-export-sql-stream-analytics/090-sql.png)

確定資料庫伺服器允許存取 Azure 服務：

![[瀏覽]、[伺服器]、您的伺服器、[設定]、[防火牆]、[允許存取 Azure]](./media/app-insights-code-sample-export-sql-stream-analytics/100-sqlaccess.png)

## <a name="create-a-table-in-azure-sql-db"></a>在 Azure SQL Database 中建立資料表
使用您慣用的管理工具，連接到上一節所建立的資料庫。 在本逐步解說中，我們將使用 [SQL Server 管理工具](https://msdn.microsoft.com/ms174173.aspx) (SSMS)。

![](./media/app-insights-code-sample-export-sql-stream-analytics/31-sql-table.png)

建立新的查詢，然後執行下列 T-SQL：

```SQL

CREATE TABLE [dbo].[PageViewsTable](
    [pageName] [nvarchar](max) NOT NULL,
    [viewCount] [int] NOT NULL,
    [url] [nvarchar](max) NULL,
    [urlDataPort] [int] NULL,
    [urlDataprotocol] [nvarchar](50) NULL,
    [urlDataHost] [nvarchar](50) NULL,
    [urlDataBase] [nvarchar](50) NULL,
    [urlDataHashTag] [nvarchar](max) NULL,
    [eventTime] [datetime] NOT NULL,
    [isSynthetic] [nvarchar](50) NULL,
    [deviceId] [nvarchar](50) NULL,
    [deviceType] [nvarchar](50) NULL,
    [os] [nvarchar](50) NULL,
    [osVersion] [nvarchar](50) NULL,
    [locale] [nvarchar](50) NULL,
    [userAgent] [nvarchar](max) NULL,
    [browser] [nvarchar](50) NULL,
    [browserVersion] [nvarchar](50) NULL,
    [screenResolution] [nvarchar](50) NULL,
    [sessionId] [nvarchar](max) NULL,
    [sessionIsFirst] [nvarchar](50) NULL,
    [clientIp] [nvarchar](50) NULL,
    [continent] [nvarchar](50) NULL,
    [country] [nvarchar](50) NULL,
    [province] [nvarchar](50) NULL,
    [city] [nvarchar](50) NULL
)

CREATE CLUSTERED INDEX [pvTblIdx] ON [dbo].[PageViewsTable]
(
    [eventTime] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)

```

![](./media/app-insights-code-sample-export-sql-stream-analytics/34-create-table.png)

在此範例中，我們會使用頁面檢視的資料。 若要查看其他可用的資料，請檢查您的 JSON 輸出，並查看 [匯出資料模型](app-insights-export-data-model.md)。

## <a name="create-an-azure-stream-analytics-instance"></a>建立 Azure 串流分析執行個體
在 [Azure 入口網站](https://portal.azure.com/)中，選取 Azure 串流分析服務，然後建立新的串流分析工作：

![](./media/app-insights-code-sample-export-sql-stream-analytics/SA001.png)

![](./media/app-insights-code-sample-export-sql-stream-analytics/SA002.png)

建立新的工作之後，選取 [前往資源]。

![](./media/app-insights-code-sample-export-sql-stream-analytics/SA003.png)

#### <a name="add-a-new-input"></a>加入新的輸入

![](./media/app-insights-code-sample-export-sql-stream-analytics/SA004.png)

將此設定為從您的連續匯出 Blob 接收輸入：

![](./media/app-insights-code-sample-export-sql-stream-analytics/SA005.png)

現在您需要儲存體帳戶的主要存取金鑰 (您已在稍早記下此金鑰)。 請將此金鑰設為儲存體帳戶金鑰。

#### <a name="set-path-prefix-pattern"></a>設定路徑前置詞模式

**請務必將 [日期格式] 設為 [YYYY-MM-DD] \(含連接號)。**

[路徑前置詞模式] 會指定串流分析在儲存體中尋找輸入檔案的方式。 您需要將它設定為與連續匯出儲存資料的方式相對應。 請設定如下：

    webapplication27_12345678123412341234123456789abcdef0/PageViews/{date}/{time}

在此範例中：

* `webapplication27` 是 Application Insights 資源名稱， **全部小寫**。 
* `1234...` 是 Application Insights 資源的檢測金鑰， **但移除了連字號**。 
* `PageViews` 是我們想要分析的資料類型。 可用的類型取決於您在「連續匯出」中設定的篩選。 檢查匯出的資料以查看其他可用的類型，並查看 [匯出資料模型](app-insights-export-data-model.md)。
* `/{date}/{time}` 是要依字面意思寫入資訊的格式。

若要取得 Application Insights 資源的名稱和 iKey，請在資源的概觀頁面中開啟 Essentials，或開啟 [設定]。

> [!TIP]
> 使用範例函數來檢查您是否已正確設定輸入路徑。 如果此方法失敗：請檢查儲存體中您所選擇的範例時間範圍的資料。 編輯輸入定義，並檢查您是否已正確設定儲存體帳戶、路徑前置詞和日期格式。
> 
> 
## <a name="set-query"></a>設定查詢
開啟 [查詢] 區段：

將預設查詢替換為以下內容：

```SQL

    SELECT flat.ArrayValue.name as pageName
    , flat.ArrayValue.count as viewCount
    , flat.ArrayValue.url as url
    , flat.ArrayValue.urlData.port as urlDataPort
    , flat.ArrayValue.urlData.protocol as urlDataprotocol
    , flat.ArrayValue.urlData.host as urlDataHost
    , flat.ArrayValue.urlData.base as urlDataBase
    , flat.ArrayValue.urlData.hashTag as urlDataHashTag
      ,A.context.data.eventTime as eventTime
      ,A.context.data.isSynthetic as isSynthetic
      ,A.context.device.id as deviceId
      ,A.context.device.type as deviceType
      ,A.context.device.os as os
      ,A.context.device.osVersion as osVersion
      ,A.context.device.locale as locale
      ,A.context.device.userAgent as userAgent
      ,A.context.device.browser as browser
      ,A.context.device.browserVersion as browserVersion
      ,A.context.device.screenResolution.value as screenResolution
      ,A.context.session.id as sessionId
      ,A.context.session.isFirst as sessionIsFirst
      ,A.context.location.clientip as clientIp
      ,A.context.location.continent as continent
      ,A.context.location.country as country
      ,A.context.location.province as province
      ,A.context.location.city as city
    INTO
      AIOutput
    FROM AIinput A
    CROSS APPLY GetElements(A.[view]) as flat


```

請注意，前幾個屬性是頁面檢視資料的特定屬性。 其他遙測資料類型的匯出會有不同的屬性。 請參閱 [屬性類型和值的詳細資料模型參考。](app-insights-export-data-model.md)

## <a name="set-up-output-to-database"></a>設定資料庫的輸出
選取 SQL 做為輸出。

![在串流分析中選取 [輸出]](./media/app-insights-code-sample-export-sql-stream-analytics/SA006.png)

指定 SQL Database。

![填入資料庫的詳細資料](./media/app-insights-code-sample-export-sql-stream-analytics/SA007.png)

關閉精靈，然後等待輸出設定完成的通知。

## <a name="start-processing"></a>開始處理
從動作列啟動工作：

![在串流分析中，按一下 [開始]](./media/app-insights-code-sample-export-sql-stream-analytics/SA008.png)

您可以選擇是否要處理從現在開始的資料，或是從較早的資料開始處理。 如果您的「連續匯出」已經執行了一段時間，則後者會是很好用的選項。

經過數分鐘後，即可返回 SQL Server 管理工具，觀看資料的流入。 例如，使用如下查詢：

    SELECT TOP 100 *
    FROM [dbo].[PageViewsTable]


## <a name="related-articles"></a>相關文章
* [使用串流分析匯出至 PowerBI](app-insights-export-power-bi.md)
* [屬性類型和值的詳細資料模型參考。](app-insights-export-data-model.md)
* [Application Insights 中的連續匯出](app-insights-export-telemetry.md)
* [Application Insights](https://azure.microsoft.com/services/application-insights/)

<!--Link references-->

[diagnostic]: app-insights-diagnostic-search.md
[export]: app-insights-export-telemetry.md
[metrics]: app-insights-metrics-explorer.md
[portal]: http://portal.azure.com/
[start]: app-insights-overview.md

