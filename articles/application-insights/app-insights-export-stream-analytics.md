---
title: 使用 Azure Application Insights 中的串流分析匯出 | Microsoft Docs
description: 串流分析可以持續轉換、篩選和路由傳送您從 Application Insights 匯出的資料。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 31594221-17bd-4e5e-9534-950f3b022209
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 01/04/2018
ms.author: mbullwin
ms.openlocfilehash: 86e37a186536a2037d3cc46cb1cc69c1a7a651f6
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719421"
---
# <a name="use-stream-analytics-to-process-exported-data-from-application-insights"></a>使用串流分析來處理從 Application Insights 匯出的資料
[Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/)是處理[從 Application Insights 匯出](app-insights-export-telemetry.md)之資料的理想工具。 串流分析可以從各種來源提取資料。 它可以轉換和篩選資料，然後將它路由傳送至各種接收。

在此範例中，將建立配接器，以從 Application Insights 取得資料、重新命名與處理某些欄位，以及透過管線將它傳送到 Power BI。

> [!WARNING]
> [在 Power BI 中顯示 Application Insights 資料的建議方式](app-insights-export-power-bi.md)更好也更容易。 這裡所述的路徑只是說明如何處理所匯出資料的範例。
> 
> 

![透過 SA 匯出至 PBI 的區塊圖](./media/app-insights-export-stream-analytics/020.png)

## <a name="create-storage-in-azure"></a>在 Azure 中建立儲存體
連續匯出一律會將資料輸出至 Azure 儲存體帳戶，因此您必須先建立儲存體。

1. 在 [Azure 入口網站](https://portal.azure.com)的訂用帳戶中建立「傳統」儲存體帳戶。
   
   ![在 Azure 入口網站中，依序選擇 [新增]、[資料]、[儲存體]](./media/app-insights-export-stream-analytics/030.png)
2. 建立容器
   
    ![在新的儲存體中，選取 [容器]，按一下容器磚，然後按一下 [新增]](./media/app-insights-export-stream-analytics/040.png)
3. 複製儲存體存取金鑰
   
    稍後您會需要使用它來設定串流分析服務的輸入。
   
    ![在儲存體中，依序開啟 [設定]、[金鑰]，然後複製主要存取金鑰](./media/app-insights-export-stream-analytics/045.png)

## <a name="start-continuous-export-to-azure-storage"></a>啟動對 Azure 儲存體的連續匯出
[連續匯出](app-insights-export-telemetry.md) 會將資料從 Application Insights 移入 Azure 儲存體。

1. 在 Azure 入口網站中，瀏覽至您為應用程式建立的 Application Insights 資源。
   
    ![依序選擇 [瀏覽]、[Application Insights]、您的應用程式](./media/app-insights-export-stream-analytics/050.png)
2. 建立連續匯出。
   
    ![依序選擇 [設定]、[連續匯出]、[新增]](./media/app-insights-export-stream-analytics/060.png)

    選取您稍早建立的儲存體帳戶：

    ![設定匯出目的地](./media/app-insights-export-stream-analytics/070.png)

    設定您想要查看的事件類型：

    ![選擇事件類型](./media/app-insights-export-stream-analytics/080.png)

1. 可讓一些資料累積。 請休息一下，讓其他人使用您的應用程式一段時間。 遙測資料會送過來，而您會在[計量瀏覽器](app-insights-metrics-explorer.md)中看到統計圖表，並在[診斷搜尋](app-insights-diagnostic-search.md)中看到個別事件。 
   
    此外，資料會匯出至您的儲存體。 
2. 檢查匯出的資料。 在 Visual Studio 中，依序選擇 [檢視] 和 [Cloud Explorer]，然後依序開啟 [Azure] 和 [儲存體]。 (如果您沒有此功能表選項，您需要安裝 Azure SDK：開啟 [新增專案] 對話方塊，然後開啟 [Visual C#] / [Cloud] / [取得 Microsoft Azure SDK for .NET]。)
   
    ![](./media/app-insights-export-stream-analytics/04-data.png)
   
    記下衍生自應用程式名稱和檢測金鑰之路徑名稱的共同部分。 

事件會以 JSON 格式寫入至 Blob 檔案。 每個檔案可能會包含一或多個事件。 因此我們想要讀取事件資料，並篩選出需要的欄位。 該處有用這些資料所能做到的所有事情種類，但我們現在計劃要使用串流分析將資料傳送至 Power BI。

## <a name="create-an-azure-stream-analytics-instance"></a>建立 Azure 串流分析執行個體
在 [Azure 入口網站](https://portal.azure.com/)中，選取 Azure 串流分析服務，然後建立新的串流分析工作：

![](./media/app-insights-export-stream-analytics/SA001.png)

![](./media/app-insights-export-stream-analytics/SA002.png)

建立新的工作之後，選取 [前往資源]。

![](./media/app-insights-export-stream-analytics/SA003.png)

### <a name="add-a-new-input"></a>加入新的輸入

![](./media/app-insights-export-stream-analytics/SA004.png)

將此設定為從您的連續匯出 Blob 接收輸入：

![](./media/app-insights-export-stream-analytics/SA0005.png)

現在您需要儲存體帳戶的主要存取金鑰 (您已在稍早記下此金鑰)。 請將此金鑰設為儲存體帳戶金鑰。

### <a name="set-path-prefix-pattern"></a>設定路徑前置詞模式

**請務必將 [日期格式] 設為 [YYYY-MM-DD] \(含連接號)。**

路徑前置詞模式會指定串流分析在存放區中尋找輸入檔案的位置。 您需要將它設定為與連續匯出儲存資料的方式相對應。 請設定如下：

    webapplication27_12345678123412341234123456789abcdef0/PageViews/{date}/{time}

在此範例中：

* `webapplication27` 是 Application Insights 資源名稱，**全部小寫**。
* `1234...` 是 Application Insights 資源的檢測金鑰， **省略破折號**。 
* `PageViews` 是您想要分析的資料類型。 可用的類型取決於您在「連續匯出」中設定的篩選。 檢查匯出的資料以查看其他可用的類型，並查看 [匯出資料模型](app-insights-export-data-model.md)。
* `/{date}/{time}` 是要依字面意思寫入資訊的格式。

> [!NOTE]
> 檢查儲存區以確定您取得正確的路徑。
> 

## <a name="add-new-output"></a>新增輸出
現在請依序選取您的工作 > [輸出] > [新增]。

![](./media/app-insights-export-stream-analytics/SA006.png)


![選取新的通道，依序按一下 [輸出]、[新增]、[Power BI]](./media/app-insights-export-stream-analytics/SA010.png)

提供您的 **工作或學校帳戶** 來授權串流分析存取您的 Power BI 資源。 接著自創一個輸出的名稱，以及目標 Power BI 資料集和資料表的名稱。

## <a name="set-the-query"></a>設定查詢
查詢會控管從輸入到輸出的轉譯。

使用測試函式來確認您取得正確的輸出。 填入您從輸入頁面所採用的範例資料。 

### <a name="query-to-display-counts-of-events"></a>顯示事件計數的查詢
貼上此查詢：

```SQL

    SELECT
      flat.ArrayValue.name,
      count(*)
    INTO
      [pbi-output]
    FROM
      [export-input] A
    OUTER APPLY GetElements(A.[event]) as flat
    GROUP BY TumblingWindow(minute, 1), flat.ArrayValue.name
```

* export-input 是我們提供給串流輸入的別名
* pbi-output 是我們所定義的輸出別名
* 我們會使用 [OUTER APPLY GetElements](https://msdn.microsoft.com/library/azure/dn706229.aspx) \(英文\)，因為事件名稱是在巢狀 JSON 陣列中。 然後 Select 會取用事件名稱，以及時間週期內具有該名稱之執行個體數目的計數。 [Group By](https://msdn.microsoft.com/library/azure/dn835023.aspx) 子句會依照一分鐘的時間間隔來將元素分組。

### <a name="query-to-display-metric-values"></a>顯示度量值的查詢
```SQL

    SELECT
      A.context.data.eventtime,
      avg(CASE WHEN flat.arrayvalue.myMetric.value IS NULL THEN 0 ELSE  flat.arrayvalue.myMetric.value END) as myValue
    INTO
      [pbi-output]
    FROM
      [export-input] A
    OUTER APPLY GetElements(A.context.custom.metrics) as flat
    GROUP BY TumblingWindow(minute, 1), A.context.data.eventtime

``` 

* 此查詢會切入度量遙測，來取得事件時間和度量值。 度量值位於陣列中，因此我們使用 OUTER APPLY GetElements 模式來擷取資料列。 在此情況下，"myMetric" 是度量的名稱。 

### <a name="query-to-include-values-of-dimension-properties"></a>包含維度屬性值的查詢
```SQL

    WITH flat AS (
    SELECT
      MySource.context.data.eventTime as eventTime,
      InstanceId = MyDimension.ArrayValue.InstanceId.value,
      BusinessUnitId = MyDimension.ArrayValue.BusinessUnitId.value
    FROM MySource
    OUTER APPLY GetArrayElements(MySource.context.custom.dimensions) MyDimension
    )
    SELECT
     eventTime,
     InstanceId,
     BusinessUnitId
    INTO AIOutput
    FROM flat

```

* 此查詢包括維度屬性值，而不需根據陣列索引中固定索引的特定維度。

## <a name="run-the-job"></a>執行工作
您可以選取一個啟動工作的過去日期。 

![選取工作並按一下 [查詢]。 貼上下面的範例。](./media/app-insights-export-stream-analytics/SA008.png)

請等候直到作業執行。

## <a name="see-results-in-power-bi"></a>在 Power BI 中查看結果
> [!WARNING]
> [在 Power BI 中顯示 Application Insights 資料的建議方式](app-insights-export-power-bi.md)更好也更容易。 這裡所述的路徑只是說明如何處理所匯出資料的範例。
> 
> 

以您的工作或學校帳戶開啟 Power BI，並選取您定義為串流分析工作輸出的資料集與資料表。

![在 Power BI 中，選取您的資料集和欄位。](./media/app-insights-export-stream-analytics/200.png)

現在您可以使用報告中的此資料集和 [Power BI](https://powerbi.microsoft.com)中的儀表板。

![在 Power BI 中，選取您的資料集和欄位。](./media/app-insights-export-stream-analytics/210.png)

## <a name="no-data"></a>沒有資料？
* 請檢查是否正確 [設定日期格式](#set-path-prefix-pattern) ：YYYY-MM-DD (含連接號)。

## <a name="video"></a>影片
Noam Ben Zeev 示範如何使用串流分析來處理匯出的資料。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Export-to-Power-BI-from-Application-Insights/player]
> 
> 

## <a name="next-steps"></a>後續步驟
* [連續匯出](app-insights-export-telemetry.md)
* [屬性類型和值的詳細資料模型參考。](app-insights-export-data-model.md)
* [Application Insights](app-insights-overview.md)

