---
title: 使用 Azure 串流分析進行高頻率交易模擬
description: 如何在相同的 Azure 串流分析作業中執行線性迴歸模型訓練和評分。
services: stream-analytics
author: zhongc
ms.author: zhongc
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 11/05/2017
ms.openlocfilehash: 85f80ef1ea776d48d9c2f8091568d40dbf46db46
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39716465"
---
# <a name="high-frequency-trading-simulation-with-stream-analytics"></a>使用串流分析進行高頻率交易模擬
Azure 串流分析中的 SQL 語言、JavaScript 使用者定義函式 (UDF) 和使用者定義彙總 (UDA) 組合，讓使用者能夠執行進階分析。 進階分析可能包含線上機器學習訓練和評分，以及可設定狀態的程序模擬。 本文說明如何在 Azure 串流分析作業中執行線性迴歸，以在高頻率的交易案例中進行連續訓練和評分。

## <a name="high-frequency-trading"></a>高頻率交易
高頻率交易的邏輯流程是關於：
1. 從證券交易所取得即時報價。
2. 建立以報價為主的預測模型，所以我們可以預期價格變動。
3. 提出買單或賣單，從成功的價格變動預測獲利。 

因此，我們需要：
* 即時報價摘要。
* 可對即時報價操作的預測模型。
* 可示範交易演算法損益的交易模擬。

### <a name="real-time-quote-feed"></a>即時報價摘要
IEX 使用 socket.io 提供免費的[即時買賣方報價](https://iextrading.com/developer/docs/#websockets)。 您可以撰寫簡單的主控台程式來接收即時報價並推送到 Azure 事件中樞作為資料來源。 下列程式碼是此程式的基本架構。 為求簡單明瞭，此程式碼會忽略錯誤處理。 您還必須在您的專案中包含 SocketIoClientDotNet 和 WindowsAzure.ServiceBus NuGet 套件。


    using Quobject.SocketIoClientDotNet.Client;
    using Microsoft.ServiceBus.Messaging;

    var symbols = "msft,fb,amzn,goog";
    var eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, eventHubName);
    var socket = IO.Socket("https://ws-api.iextrading.com/1.0/tops");

    socket.On(Socket.EVENT_MESSAGE, (message) =>
    {
        eventHubClient.Send(new EventData(Encoding.UTF8.GetBytes((string)message)));
    });

    socket.On(Socket.EVENT_CONNECT, () =>
    {
        socket.Emit("subscribe", symbols);
    });

以下是所產生的一些範例事件：

    {"symbol":"MSFT","marketPercent":0.03246,"bidSize":100,"bidPrice":74.8,"askSize":300,"askPrice":74.83,"volume":70572,"lastSalePrice":74.825,"lastSaleSize":100,"lastSaleTime":1506953355123,"lastUpdated":1506953357170,"sector":"softwareservices","securityType":"commonstock"}
    {"symbol":"GOOG","marketPercent":0.04825,"bidSize":114,"bidPrice":870,"askSize":0,"askPrice":0,"volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,"lastUpdated":1506953357633,"sector":"softwareservices","securityType":"commonstock"}
    {"symbol":"MSFT","marketPercent":0.03244,"bidSize":100,"bidPrice":74.8,"askSize":100,"askPrice":74.83,"volume":70572,"lastSalePrice":74.825,"lastSaleSize":100,"lastSaleTime":1506953355123,"lastUpdated":1506953359118,"sector":"softwareservices","securityType":"commonstock"}
    {"symbol":"FB","marketPercent":0.01211,"bidSize":100,"bidPrice":169.9,"askSize":100,"askPrice":170.67,"volume":39042,"lastSalePrice":170.67,"lastSaleSize":100,"lastSaleTime":1506953351912,"lastUpdated":1506953359641,"sector":"softwareservices","securityType":"commonstock"}
    {"symbol":"GOOG","marketPercent":0.04795,"bidSize":100,"bidPrice":959.19,"askSize":0,"askPrice":0,"volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,"lastUpdated":1506953360949,"sector":"softwareservices","securityType":"commonstock"}
    {"symbol":"FB","marketPercent":0.0121,"bidSize":100,"bidPrice":169.9,"askSize":100,"askPrice":170.7,"volume":39042,"lastSalePrice":170.67,"lastSaleSize":100,"lastSaleTime":1506953351912,"lastUpdated":1506953362205,"sector":"softwareservices","securityType":"commonstock"}
    {"symbol":"GOOG","marketPercent":0.04795,"bidSize":114,"bidPrice":870,"askSize":0,"askPrice":0,"volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,"lastUpdated":1506953362629,"sector":"softwareservices","securityType":"commonstock"}

>[!NOTE]
>事件的時間戳記為 **lastUpdated** (epoch 時間)。

### <a name="predictive-model-for-high-frequency-trading"></a>高頻率交易的預測模型
為了進行示範，我們使用 Darryl Shen 在[他的論文](http://eprints.maths.ox.ac.uk/1895/1/Darryl%20Shen%20%28for%20archive%29.pdf)中說明的線性模型。

數量委託單不平衡 (VOI) 是目前買入/賣出價格和數量的函式，以及最後一個檔位中買入/賣出價格和數量的函式。 本文件可識別 VOI 與未來價格變動之間的相互關聯。 它建立過去 5 個 VOI 值與接下來 10 個檔位中價格變動之間的線性模型。 使用前一天的資料搭配線性迴歸來訓練模型。 

定型的模型接著用於對目前交易日中的報價，進行即時的價格變動預測。 預測到夠大的價格變動時，就會執行交易。 根據臨界值設定，一檔股票在一個交易日中可能會有數千筆交易。

![VOI 定義](./media/stream-analytics-high-frequency-trading/voi-formula.png)

現在，讓我們加快 Azure 串流分析作業中的訓練和預測作業。

首先會清除輸入。 Epoch 時間會透過 **DATEADD** 轉換成日期時間。 **TRY_CAST** 用來強制資料類型，而不會查詢失敗。 將輸入欄位轉換為預期的資料類型，一直是理想的做法，如此欄位的操作或比較就不會出現非預期的行為。

    WITH
    typeconvertedquotes AS (
        /* convert all input fields to proper types */
        SELECT
            System.Timestamp AS lastUpdated,
            symbol,
            DATEADD(millisecond, CAST(lastSaleTime as bigint), '1970-01-01T00:00:00Z') AS lastSaleTime,
            TRY_CAST(bidSize as bigint) AS bidSize,
            TRY_CAST(bidPrice as float) AS bidPrice,
            TRY_CAST(askSize as bigint) AS askSize,
            TRY_CAST(askPrice as float) AS askPrice,
            TRY_CAST(volume as bigint) AS volume,
            TRY_CAST(lastSaleSize as bigint) AS lastSaleSize,
            TRY_CAST(lastSalePrice as float) AS lastSalePrice
        FROM quotes TIMESTAMP BY DATEADD(millisecond, CAST(lastUpdated as bigint), '1970-01-01T00:00:00Z')
    ),
    timefilteredquotes AS (
        /* filter between 7am and 1pm PST, 14:00 to 20:00 UTC */
        /* clean up invalid data points */
        SELECT * FROM typeconvertedquotes
        WHERE DATEPART(hour, lastUpdated) >= 14 AND DATEPART(hour, lastUpdated) < 20 AND bidSize > 0 AND askSize > 0 AND bidPrice > 0 AND askPrice > 0
    ),

接下來，我們會使用 **LAG** 函式來取得最後一個檔位的值。 任意選擇一小時的 **LIMIT DURATION** 值。 基於報價頻率，假定您可以找到前一小時的前一個檔位很安全。  

    shiftedquotes AS (
        /* get previous bid/ask price and size in order to calculate VOI */
        SELECT
            symbol,
            (bidPrice + askPrice)/2 AS midPrice,
            bidPrice,
            bidSize,
            askPrice,
            askSize,
            LAG(bidPrice) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS bidPricePrev,
            LAG(bidSize) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS bidSizePrev,
            LAG(askPrice) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS askPricePrev,
            LAG(askSize) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS askSizePrev
        FROM timefilteredquotes
    ),

然後，我們可以計算 VOI 值。 如果先前的檔位不存在，我們會篩選出 null 值，以防萬一。

    currentPriceAndVOI AS (
        /* calculate VOI */
        SELECT
            symbol,
            midPrice,
            (CASE WHEN (bidPrice < bidPricePrev) THEN 0
                ELSE (CASE WHEN (bidPrice = bidPricePrev) THEN (bidSize - bidSizePrev) ELSE bidSize END)
             END) -
            (CASE WHEN (askPrice < askPricePrev) THEN askSize
                ELSE (CASE WHEN (askPrice = askPricePrev) THEN (askSize - askSizePrev) ELSE 0 END)
             END) AS VOI
        FROM shiftedquotes
        WHERE
            bidPrice IS NOT NULL AND
            bidSize IS NOT NULL AND
            askPrice IS NOT NULL AND
            askSize IS NOT NULL AND
            bidPricePrev IS NOT NULL AND
            bidSizePrev IS NOT NULL AND
            askPricePrev IS NOT NULL AND
            askSizePrev IS NOT NULL
    ),

現在，我們再次使用 **LAG** 來建立包含 2 個連續 VOI 值的序列，後面接著 10 個連續的中間價格值。

    shiftedPriceAndShiftedVOI AS (
        /* get 10 future prices and 2 previous VOIs */
        SELECT
            symbol,
            midPrice AS midPrice10,
            LAG(midPrice, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice9,
            LAG(midPrice, 2) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice8,
            LAG(midPrice, 3) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice7,
            LAG(midPrice, 4) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice6,
            LAG(midPrice, 5) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice5,
            LAG(midPrice, 6) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice4,
            LAG(midPrice, 7) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice3,
            LAG(midPrice, 8) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice2,
            LAG(midPrice, 9) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice1,
            LAG(midPrice, 10) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice,
            LAG(VOI, 10) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI1,
            LAG(VOI, 11) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI2
        FROM currentPriceAndVOI
    ),

然後，我們會將資料調整成兩個變數線性模型的輸入。 再次篩選出我們沒有所有資料的事件。

    modelInput AS (
        /* create feature vector, x being VOI, y being delta price */
        SELECT
            symbol,
            (midPrice1 + midPrice2 + midPrice3 + midPrice4 + midPrice5 + midPrice6 + midPrice7 + midPrice8 + midPrice9 + midPrice10)/10.0 - midPrice AS y,
            VOI1 AS x1,
            VOI2 AS x2
        FROM shiftedPriceAndShiftedVOI
        WHERE
            midPrice1 IS NOT NULL AND
            midPrice2 IS NOT NULL AND
            midPrice3 IS NOT NULL AND
            midPrice4 IS NOT NULL AND
            midPrice5 IS NOT NULL AND
            midPrice6 IS NOT NULL AND
            midPrice7 IS NOT NULL AND
            midPrice8 IS NOT NULL AND
            midPrice9 IS NOT NULL AND
            midPrice10 IS NOT NULL AND
            midPrice IS NOT NULL AND
            VOI1 IS NOT NULL AND
            VOI2 IS NOT NULL
    ),

因為 Azure 串流分析沒有內建的線性迴歸函式，所以我們使用 **SUM** 和 **AVG** 彙總值來計算線性模型的係數。

![線性迴歸公式](./media/stream-analytics-high-frequency-trading/linear-regression-formula.png)

    modelagg AS (
        /* get aggregates for linear regression calculation,
         http://faculty.cas.usf.edu/mbrannick/regression/Reg2IV.html */
        SELECT
            symbol,
            SUM(x1 * x1) AS x1x1,
            SUM(x2 * x2) AS x2x2,
            SUM(x1 * y) AS x1y,
            SUM(x2 * y) AS x2y,
            SUM(x1 * x2) AS x1x2,
            AVG(y) AS avgy,
            AVG(x1) AS avgx1,
            AVG(x2) AS avgx2
        FROM modelInput
        GROUP BY symbol, TumblingWindow(hour, 24, -4)
    ),
    modelparambs AS (
        /* calculate b1 and b2 for the linear model */
        SELECT
            symbol,
            (x2x2 * x1y - x1x2 * x2y)/(x1x1 * x2x2 - x1x2 * x1x2) AS b1,
            (x1x1 * x2y - x1x2 * x1y)/(x1x1 * x2x2 - x1x2 * x1x2) AS b2,
            avgy,
            avgx1,
            avgx2
        FROM modelagg
    ),
    model AS (
        /* calculate a for the linear model */
        SELECT
            symbol,
            avgy - b1 * avgx1 - b2 * avgx2 AS a,
            b1,
            b2
        FROM modelparambs
    ),

為了將前一天的模型用於目前事件的評分，我們想要透過此模型加入報價。 不過，我們不會使用 **JOIN**，而會 **UNION** 模型事件和報價事件。 然後使用 **LAG** 來配對事件與前一天的模型，因此可以取得正好一個相符項目。 因為是週末，我們必須回頭查看三天。 如果直接使用 **JOIN**，則每個報價事件會得到三個模型。

    shiftedVOI AS (
        /* get two consecutive VOIs */
        SELECT
            symbol,
            midPrice,
            VOI AS VOI1,        
            LAG(VOI, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI2
        FROM currentPriceAndVOI
    ),
    VOIAndModel AS (
        /* combine VOIs and models */
        SELECT
            'voi' AS type,
            symbol,
            midPrice,
            VOI1,
            VOI2,
            0.0 AS a,
            0.0 AS b1,
            0.0 AS b2
        FROM shiftedVOI
        UNION
        SELECT
            'model' AS type,
            symbol,
            0.0 AS midPrice,
            0 AS VOI1,
            0 AS VOI2,
            a,
            b1,
            b2
        FROM model
    ),
    VOIANDModelJoined AS (
        /* match VOIs with the latest model within 3 days (72 hours, to take the weekend into account) */
        SELECT
            symbol,
            midPrice,
            VOI1 as x1,
            VOI2 as x2,
            LAG(a, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS a,
            LAG(b1, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS b1,
            LAG(b2, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS b2
        FROM VOIAndModel
        WHERE type = 'voi'
    ),

現在，我們可以進行預測並產生以模型為基礎的買賣訊號 (臨界值為 0.02)。 交易值為 10 表示買進。 交易值為 -10 表示賣出。

    prediction AS (
        /* make prediction if there is a model */
        SELECT
            symbol,
            midPrice,
            a + b1 * x1 + b2 * x2 AS efpc
        FROM VOIANDModelJoined
        WHERE
            a IS NOT NULL AND
            b1 IS NOT NULL AND
            b2 IS NOT NULL AND
            x1 IS NOT NULL AND
            x2 IS NOT NULL
    ),
    tradeSignal AS (
        /* generate buy/sell signals */
        SELECT
            DateAdd(hour, -7, System.Timestamp) AS time,
            symbol,     
            midPrice,
            efpc,
            CASE WHEN (efpc > 0.02) THEN 10 ELSE (CASE WHEN (efpc < -0.02) THEN -10 ELSE 0 END) END AS trade,
            DATETIMEFROMPARTS(DATEPART(year, System.Timestamp), DATEPART(month, System.Timestamp), DATEPART(day, System.Timestamp), 0, 0, 0, 0) as date
        FROM prediction
    ),

### <a name="trading-simulation"></a>交易模擬
擁有交易訊號之後，我們想測試交易策略的有效性，而不需實際進行交易。 

我們透過跳動視窗 (每一分鐘跳動)，使用 UDA 來達成這項測試。 其他日期群組和 having 子句允許此視窗只考量屬於同一天的事件。 對於橫跨兩天的跳動視窗，**GROUP BY** 日期會將群組分為前一天和當天。 **HAVING** 子句會篩選出在當天結束、但在前一天分組的視窗。

    simulation AS
    (
        /* perform trade simulation for the past 7 hours to cover an entire trading day, and generate output every minute */
        SELECT
            DateAdd(hour, -7, System.Timestamp) AS time,
            symbol,
            date,
            uda.TradeSimulation(tradeSignal) AS s
        FROM tradeSignal
        GROUP BY HoppingWindow(minute, 420, 1), symbol, date
        Having DateDiff(day, date, time) < 1 AND DATEPART(hour, time) < 13
    )

JavaScript UDA 會初始化 `init` 函式中的所有累加器、以新增至視窗的每個事件計算狀態轉換，並在視窗結尾傳回模擬結果。 一般交易程序是：

- 在收到購買訊號而且沒有庫存持股時，買入股票。
- 在收到賣出訊號而且有庫存持股時，賣出股票。
- 如果沒有任何持股，則買空。 

如果出現空頭，而且收到買入訊號，我們會空單補回。 我們從未在此模擬中持有或放空 10 股的股票。 交易成本一律為 $8。


    function main() {
        var TRADE_COST = 8.0;
        var SHARES = 10;

        this.init = function () {
            this.own = false;
            this.pos = 0;
            this.pnl = 0.0;
            this.tradeCosts = 0.0;
            this.buyPrice = 0.0;
            this.sellPrice = 0.0;
            this.buySize = 0;
            this.sellSize = 0;
            this.buyTotal = 0.0;
            this.sellTotal = 0.0;
        }

        this.accumulate = function (tradeSignal, timestamp) {

            if(!this.own && tradeSignal.trade == 10) {
              // Buy to open
              this.own = true;
              this.pos = 1;
              this.buyPrice = tradeSignal.midprice;
              this.tradeCosts += TRADE_COST;
              this.buySize += SHARES;
              this.buyTotal += SHARES * tradeSignal.midprice;
            } else if(!this.own && tradeSignal.trade == -10) {
              // Sell to open
              this.own = true;
              this.pos = -1
              this.sellPrice = tradeSignal.midprice;
              this.tradeCosts += TRADE_COST;
              this.sellSize += SHARES;
              this.sellTotal += SHARES * tradeSignal.midprice;
            } else if(this.own && this.pos == 1 && tradeSignal.trade == -10) {
              // Sell to close
              this.own = false;
              this.pos = 0;
              this.sellPrice = tradeSignal.midprice;
              this.tradeCosts += TRADE_COST;
              this.pnl += (this.sellPrice - this.buyPrice)*SHARES - 2*TRADE_COST;
              this.sellSize += SHARES;
              this.sellTotal += SHARES * tradeSignal.midprice;

              // Sell to open
              this.own = true;
              this.pos = -1;
              this.sellPrice = tradeSignal.midprice;
              this.tradeCosts += TRADE_COST;
              this.sellSize += SHARES;        
              this.sellTotal += SHARES * tradeSignal.midprice;
            } else if(this.own && this.pos == -1 && tradeSignal.trade == 10) {
              // Buy to close
              this.own = false;
              this.pos = 0;
              this.buyPrice = tradeSignal.midprice;
              this.tradeCosts += TRADE_COST;
              this.pnl += (this.sellPrice - this.buyPrice)*SHARES - 2*TRADE_COST;
              this.buySize += SHARES;
              this.buyTotal += SHARES * tradeSignal.midprice;

              // Buy to open
              this.own = true;
              this.pos = 1;
              this.buyPrice = tradeSignal.midprice;
              this.tradeCosts += TRADE_COST;
              this.buySize += SHARES;         
              this.buyTotal += SHARES * tradeSignal.midprice;
            }
        }

        this.computeResult = function () {
            var result = {
                "pnl": this.pnl,
                "buySize": this.buySize,
                "sellSize": this.sellSize,
                "buyTotal": this.buyTotal,
                "sellTotal": this.sellTotal,
                "tradeCost": this.tradeCost
                };
            return result;
        }
    }

我們最後會輸出至 Power BI 以視覺效果呈現。

    SELECT * INTO tradeSignalDashboard FROM tradeSignal /* output tradeSignal to PBI */
    SELECT
        symbol,
        time,
        date,
        TRY_CAST(s.pnl as float) AS pnl,
        TRY_CAST(s.buySize as bigint) AS buySize,
        TRY_CAST(s.sellSize as bigint) AS sellSize,
        TRY_CAST(s.buyTotal as float) AS buyTotal,
        TRY_CAST(s.sellTotal as float) AS sellTotal
        INTO pnlDashboard
    FROM simulation /* output trade simulation to PBI */

![交易](./media/stream-analytics-high-frequency-trading/trades.png)

![PNL](./media/stream-analytics-high-frequency-trading/pnl.png)


## <a name="summary"></a>總結
我們可以在 Azure 串流分析中，使用適度複雜的查詢來實作實際高頻率的交易模型。 因為缺少內建線性迴歸變數，所以我們必須將模型從五個輸入變數簡化為兩個。 但是對於下定決心的使用者而言，具有更高維度和複雜度的演算法也可能實作為 JavaScript UDA。 

值得注意的是，JavaScript UDA 以外的大部分查詢，都可以透過[適用於 Visual Studio 的 Azure 串流分析工具](stream-analytics-tools-for-visual-studio-install.md)在 Visual Studio 中進行測試和偵錯。 寫入初始查詢之後，作者花不到 30 分鐘的時間在 Visual Studio 中進行查詢測試和偵錯。 

目前，UDA 無法在 Visual Studio 中偵錯。 我們正在研究如何透過逐步執行 JavaScript 程式碼的能力來啟用該功能。 此外，請注意達到 UDA 的欄位具有小寫名稱。 這不是查詢測試期間的明顯行為。 但使用 Azure 串流分析相容性層級 1.1，我們會保留欄位名稱大小寫，所以此行為更加自然。

我希望這篇文章可以鼓舞所有的 Azure 串流分析使用者，他們可以使用我們的服務持續執行幾近即時的進階分析。 請讓我們知道您的任何意見反應，進而讓實作進階分析案例的查詢變得更輕鬆。
