---
title: Azure 串流分析中 JavaScript 使用者定義的彙總
description: 本文說明如何使用 Azure 串流分析中 JavaScript 使用者定義的彙總來執行進階的查詢機制。
services: stream-analytics
author: rodrigoamicrosoft
ms.author: rodrigoa
manager: kfile
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 10/28/2017
ms.openlocfilehash: bdf5b5188dd584c5eb20f72ff4a98ba6904bc53e
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702369"
---
# <a name="azure-stream-analytics-javascript-user-defined-aggregates-preview"></a>Azure 串流分析 JavaScript 使用者定義彙總 (預覽)
 
「Azure 串流分析」支援以 JavaScript 撰寫的使用者定義彙總 (UDA)，可讓您實作複雜的具狀態商務邏輯。 在 UDA 內，您可以完全控制狀態資料結構、狀態累積、狀態取消累積，以及彙總結果計算。 本文將介紹兩種不同的 JavaScript UDA 介面、建立 UDA 的步驟，以及如何在「串流分析」中搭配 Windows 型作業使用 UDA。

## <a name="javascript-user-defined-aggregates"></a>JavaScript 使用者定義彙總

使用者定義彙總是在指定的時間範圍上用來彙總該範圍中的事件，然後產生單一的結果值。 目前「串流分析」支援兩種類型的 UDA 介面：AccumulateOnly 和 AccumulateDeaccumulate。 這兩種類型的 UDA 都可供「輪轉時間範圍」、「跳動時間範圍」及「滑動時間範圍」使用。 與「跳動時間範圍」及「滑動時間範圍」搭配使用時，AccumulateDeaccumulate UDA 的執行效能會比 AccumulateOnly UDA 好。 您需根據所使用的演算法來選擇這兩種類型其中之一。

### <a name="accumulateonly-aggregates"></a>AccumulateOnly 彙總

AccumulateOnly 彙總只會將新事件累積至其狀態，此演算法不會允許將值取消累積。 無法進行將事件資訊從狀態值取消累積的實作時，請選擇此彙總類型。 以下是 AccumulatOnly 彙總的 JavaScript 範本：

````JavaScript
// Sample UDA which state can only be accumulated.
function main() {
    this.init = function () {
        this.state = 0;
    }

    this.accumulate = function (value, timestamp) {
        this.state += value;
    }

    this.computeResult = function () {
        return this.state;
    }
}
````

### <a name="accumulatedeaccumulate-aggregates"></a>AccumulateDeaccumulate 彙總

AccumulateDeaccumulate 彙總可允許將先前累積的值從狀態中取消累積，例如從事件值清單中移除機碼值組，或從總和彙總的狀態中減去某個值。 以下是 AccumulateDeaccumulate 彙總的 JavaScript 範本：

````JavaScript
// Sample UDA which state can be accumulated and deaccumulated.
function main() {
    this.init = function () {
        this.state = 0;
    }

    this.accumulate = function (value, timestamp) {
        this.state += value;
    }

    this.deaccumulate = function (value, timestamp) {
        this.state -= value;
    }

    this.deaccumulateState = function (otherState){
        this.state -= otherState.state;
    }

    this.computeResult = function () {
        return this.state;
    }
}
````

## <a name="uda---javascript-function-declaration"></a>UDA - JavaScript 函式宣告

每個 JavaScript UDA 都由一個「函式」物件宣告來定義。 以下是 UDA 定義中的主要元素。

### <a name="function-alias"></a>函式別名

函式別名是 UDA 識別碼。 在「串流分析」查詢中呼叫時，請一律將 UDA 別名與 "uda" 搭配使用。 。

### <a name="function-type"></a>函式類型

就 UDA 而言，函式類型應該是 **Javascript UDA**。

### <a name="output-type"></a>輸出類型

「串流分析」作業支援的特定類型，或如果您想要在查詢中處理類型，則為 "Any"。

### <a name="function-name"></a>函式名稱

此「函式」物件的名稱。 函式名稱應該在字面上與 UDA 別名相符 (這是預覽版行為，我們考慮在正式運作版時支援匿名函式)。

### <a name="method---init"></a>方法 - init()

init() 方法會將彙總狀態初始化。 呼叫此方法的時機是在時間範圍開始時。

### <a name="method--accumulate"></a>方法 – accumulate()

accumulate() 方法會根據先前的狀態和目前的事件值來計算 UDA 狀態。 呼叫此方法的時機是在事件進入某個時間範圍 (TUMBLINGWINDOW、HOPPINGWINDOW 或 SLIDINGWINDOW) 時。

### <a name="method--deaccumulate"></a>方法 – deaccumulate()

deaccumulate() 方法會根據先前的狀態和目前的事件值來重新計算狀態。 呼叫此方法的時機是在事件離開 SLIDINGWINDOW 時。

### <a name="method--deaccumulatestate"></a>方法 – deaccumulateState()

deaccumulateState() 方法會根據先前的狀態和躍點的狀態來重新計算狀態。 呼叫此方法的時機是在一組事件離開 HOPPINGWINDOW 時。

### <a name="method--computeresult"></a>方法 – computeResult()

computeResult() 方法會根據目前的狀態傳回彙總結果。 呼叫此方法的時機是在時間範圍 (TUMBLINGWINDOW、HOPPINGWINDOW 及 SLIDINGWINDOW) 結束時。

## <a name="javascript-uda-supported-input-and-output-data-types"></a>JavaScript UDA 支援的輸入和輸出資料類型
針對 JavaScript UDA 資料類型，請參閱[整合 JavaScript UDF](stream-analytics-javascript-user-defined-functions.md) 的**串流分析與 JavaScript 類型轉換**一節。

## <a name="adding-a-javascript-uda-from-the-azure-portal"></a>從 Azure 入口網站新增 JavaScript UDA

以下我們將會逐步完成從「入口網站」建立 UDA 的程序。 我們在這裡使用的範例是計算時間加權平均值。

現在，我們將依循步驟，在現有的 ASA 作業底下建立一個 JavaScript UDA。

1. 登入 Azure 入口網站並找出您現有的「串流分析」作業。
1. 然後按一下 [作業拓撲] 底下的函式連結。
1. 按一下 [新增] 圖示來新增函式。
1. 在 [新增函式] 檢視上，選取 [JavaScript UDA] 作為 [函式類型]，然後您就會在編輯器中看見預設的 UDA 範本。
1. 填入 "TWA" 作為 UDA 別名，然後依下列方式變更函式實作：

    ````JavaScript
    // Sample UDA which calculate Time-Weighted Average of incoming values.
    function main() {
        this.init = function () {
            this.totalValue = 0.0;
            this.totalWeight = 0.0;
        }

        this.accumulate = function (value, timestamp) {
            this.totalValue += value.level * value.weight;
            this.totalWeight += value.weight;

        }

        // Uncomment below for AccumulateDeaccumulate implementation
        /*
        this.deaccumulate = function (value, timestamp) {
            this.totalValue -= value.level * value.weight;
            this.totalWeight -= value.weight;
        }

        this.deaccumulateState = function (otherState){
            this.state -= otherState.state;
            this.totalValue -= otherState.totalValue;
            this.totalWeight -= otherState.totalWeight;
        }
        */

        this.computeResult = function () {
            if(this.totalValue == 0) {
                result = 0;
            }
            else {
                result = this.totalValue/this.totalWeight;
            }
            return result;
        }
    }
    ````

1. 按一下 [儲存] 按鈕之後，您的 UDA 就會顯示在函式清單上。

1. 按一下新函式 "TWA" 即可查看函式定義。

## <a name="calling-javascript-uda-in-asa-query"></a>在 ASA 查詢中呼叫 JavaScript UDA

在 Azure 入口網站中，開啟您的作業、編輯查詢，然後使用授權前置詞 "uda" 來呼叫 TWA() 函式。 例如︰

````SQL
WITH value AS
(
    SELECT
    NoiseLevelDB as level,
    DurationSecond as weight
FROM
    [YourInputAlias] TIMESTAMP BY EntryTime
)
SELECT
    System.Timestamp as ts,
    uda.TWA(value) as NoseDoseTWA
FROM value
GROUP BY TumblingWindow(minute, 5)
````

## <a name="testing-query-with-uda"></a>使用 UDA 來測試查詢

使用下列內容來建立一個本機 JSON 檔案，將此檔案上傳到「串流分析」作業中，然後測試上述查詢。

````JSON
[
  {"EntryTime": "2017-06-10T05:01:00-07:00", "NoiseLevelDB": 80, "DurationSecond": 22.0},
  {"EntryTime": "2017-06-10T05:02:00-07:00", "NoiseLevelDB": 81, "DurationSecond": 37.8},
  {"EntryTime": "2017-06-10T05:02:00-07:00", "NoiseLevelDB": 85, "DurationSecond": 26.3},
  {"EntryTime": "2017-06-10T05:03:00-07:00", "NoiseLevelDB": 95, "DurationSecond": 13.7},
  {"EntryTime": "2017-06-10T05:03:00-07:00", "NoiseLevelDB": 88, "DurationSecond": 10.3},
  {"EntryTime": "2017-06-10T05:05:00-07:00", "NoiseLevelDB": 103, "DurationSecond": 5.5},
  {"EntryTime": "2017-06-10T05:06:00-07:00", "NoiseLevelDB": 99, "DurationSecond": 23.0},
  {"EntryTime": "2017-06-10T05:07:00-07:00", "NoiseLevelDB": 108, "DurationSecond": 1.76},
  {"EntryTime": "2017-06-10T05:07:00-07:00", "NoiseLevelDB": 79, "DurationSecond": 17.9},
  {"EntryTime": "2017-06-10T05:08:00-07:00", "NoiseLevelDB": 83, "DurationSecond": 27.1},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 91, "DurationSecond": 17.1},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 115, "DurationSecond": 7.9},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 80, "DurationSecond": 28.3},
  {"EntryTime": "2017-06-10T05:10:00-07:00", "NoiseLevelDB": 55, "DurationSecond": 18.2},
  {"EntryTime": "2017-06-10T05:10:00-07:00", "NoiseLevelDB": 93, "DurationSecond": 25.8},
  {"EntryTime": "2017-06-10T05:11:00-07:00", "NoiseLevelDB": 83, "DurationSecond": 11.4},
  {"EntryTime": "2017-06-10T05:12:00-07:00", "NoiseLevelDB": 89, "DurationSecond": 7.9},
  {"EntryTime": "2017-06-10T05:15:00-07:00", "NoiseLevelDB": 112, "DurationSecond": 3.7},
  {"EntryTime": "2017-06-10T05:15:00-07:00", "NoiseLevelDB": 93, "DurationSecond": 9.7},
  {"EntryTime": "2017-06-10T05:18:00-07:00", "NoiseLevelDB": 96, "DurationSecond": 3.7},
  {"EntryTime": "2017-06-10T05:20:00-07:00", "NoiseLevelDB": 108, "DurationSecond": 0.99},
  {"EntryTime": "2017-06-10T05:20:00-07:00", "NoiseLevelDB": 113, "DurationSecond": 25.1},
  {"EntryTime": "2017-06-10T05:22:00-07:00", "NoiseLevelDB": 110, "DurationSecond": 5.3}
]
````

## <a name="get-help"></a>取得說明

如需其他協助，請參閱我們的 [Azure 串流分析論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。

## <a name="next-steps"></a>後續步驟

* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure 串流分析查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
