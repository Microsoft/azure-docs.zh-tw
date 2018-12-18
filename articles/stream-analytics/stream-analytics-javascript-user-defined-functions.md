---
title: '教學課程：Azure 串流分析 JavaScript 使用者定義函式 | Microsoft Docs '
description: 在此教學課程中，您將使用 JavaScript 使用者定義函式來執行進階查詢技術
keywords: javascript, 使用者定義函式, udf
services: stream-analytics
author: rodrigoamicrosoft
manager: kfile
ms.assetid: ''
ms.service: stream-analytics
ms.topic: tutorial
ms.reviewer: mamccrea
ms.custom: mvc
ms.date: 04/01/2018
ms.workload: data-services
ms.author: rodrigoa
ms.openlocfilehash: 56ac08593d29a9003f5509f2be52f50fa250bd75
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47431590"
---
# <a name="tutorial-azure-stream-analytics-javascript-user-defined-functions"></a>教學課程：Azure 串流分析 JavaScript 使用者定義函式
 
Azure 串流分析支援以 JavaScript 撰寫的使用者定義函式。 JavaScript 提供豐富的 **String**、**RegExp**、**Math**、**Array** 和 **Date** 方法，可讓使用串流分析作業建立複雜的資料轉換變得更容易。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 定義 JavaScript 使用者定義函式
> * 將函式新增至入口網站
> * 定義執行函式的查詢

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="javascript-user-defined-functions"></a>JavaScript 使用者定義函式
JavaScript 使用者定義的函式支援無狀態且只做為計算用途的純量函式，而且不需要外部連線能力。 函數的傳回值只能是純量 (單一) 值。 將 JavaScript 使用者定義函式新增至作業之後，您可以在查詢中的任何位置使用函式，就像是內建的純量函式。

從以下的一些案例可以看出 JavaScript 使用者定義函式很實用：
* 剖析及操作具有規則運算式函式的字串，例如：**Regexp_Replace()** 和 **Regexp_Extract()**
* 解碼和編碼資料，例如：從二進位轉換成十六進位
* 使用 JavaScript **Math** 函式做數學運算
* 執行陣列作業，例如，排序、連結、尋找及填入

以下是一些在串流分析中無法使用 JavaScript 使用者定義函式達成的事項：
* 呼叫外部 REST 端點，例如，執行反向 IP 查詢或從外部來源提取參考資料
* 對輸入/輸出執行自訂事件格式序列化或還原序列化
* 建立自訂彙總

雖然沒有在函式定義中封鎖 **Date.GetDate()** 或 **Math.random()** 等函式，您仍應避免使用它們。 這些函式**不會**在每次呼叫時都傳回同樣的結果，且「串流分析」服務不會記錄函式叫用和傳回值的日誌。 若函式針對同樣事件傳回不同的值，當您或串流分析重新啟動某作業之後，將不保證它具有可重覆性。

## <a name="add-a-javascript-user-defined-function-in-the-azure-portal"></a>在 Azure 入口網站中新增 JavaScript 使用者定義函式
若要在現有串流分析作業下建立簡單的 JavaScript 使用者定義函式，請執行下列步驟：

1.  在 Azure 入口網站中，找出您的串流分析作業。
2.  在 [作業拓撲] 下，選取您的函式。 空白的函式清單隨即出現。
3.  若要建立新的使用者定義函式，請選取 [新增]。
4.  在 [新增函式] 刀鋒視窗中，針對 [函式類型]，選取 [JavaScript]。 預設的函式範本會出現在編輯器中。
5.  針對 **UDF 別名**，輸入 **hex2Int**，並變更函式實作，如下所示：

    ```
    // Convert Hex value to integer.
    function hex2Int(hexValue) {
        return parseInt(hexValue, 16);
    }
    ```

6.  選取 [ **儲存**]。 您的函式會出現在函式的清單中。
7.  選取新的 **hex2Int** 函式，並檢查函式定義。 所有函式必須在函式別名前端新增 **UDF** 前置詞。 在串流分析查詢中呼叫函式時，您需要*包含前置詞*。 在此情況下，您會呼叫 **UDF.hex2Int**。

## <a name="call-a-javascript-user-defined-function-in-a-query"></a>在查詢中呼叫 JavaScript 使用者定義函式

1. 在查詢編輯器中，於 [作業拓撲] 下，選取 [查詢]。
2.  編輯您的查詢，然後呼叫使用者定義函式，就像這樣：

    ```
    SELECT
        time,
        UDF.hex2Int(offset) AS IntOffset
    INTO
        output
    FROM
        InputStream
    ```

3.  若要上傳範例資料檔案，請以滑鼠右鍵按一下作業輸入。
4.  若要測試您的查詢，請選取 [測試]。


## <a name="supported-javascript-objects"></a>支援的 JavaScript 物件
Azure 串流分析 JavaScript 使用者定義函式支援標準的內建 JavaScript 物件。 如需這些物件的清單，請參閱[全域物件](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects)。

### <a name="stream-analytics-and-javascript-type-conversion"></a>串流分析與 JavaScript 類型轉換

串流分析查詢語言與 JavaScript 支援的類型有一些差異。 此表列出兩者之間的轉換對應：

串流分析 | JavaScript
--- | ---
bigint | Number (JavaScript 只能準確地表示最高到 2^53 的整數)
Datetime | Date (JavaScript 只支援毫秒)
double | 數字
nvarchar(MAX) | 字串
Record | Object
陣列 | 陣列
NULL | Null


以下是 JavaScript 至串流分析的轉換：


JavaScript | 串流分析
--- | ---
數字 | Bigint (若數字為整數且介於 long.MinValue 和 long.MaxValue 之間，否則為 double)
日期 | Datetime
字串 | nvarchar(MAX)
Object | Record
陣列 | 陣列
Null、Undefined | NULL
任何其他類型 (例如，函式或錯誤) | 不支援 (產生執行階段錯誤)

## <a name="troubleshooting"></a>疑難排解
JavaScript 執行階段錯誤會被視為嚴重問題，並顯示在活動記錄。 若要擷取記錄檔，在 Azure 入口網站中，請移至您的作業並選取 [活動記錄]。


## <a name="other-javascript-user-defined-function-patterns"></a>其他 JavaScript 使用者定義函式模式

### <a name="write-nested-json-to-output"></a>將巢狀 JSON 寫入至輸出
如果您的後續處理步驟使用串流分析作業輸出做為輸入，且其需要 JSON 格式，您可以將 JSON 字串寫入至輸出。 以下範例會呼叫 **JSON.stringify()** 函式以包裝所有輸入的名稱/值對，並將它們以單一字串值於輸出中寫入。

**JavaScript 使用者定義函式定義：**

```
function main(x) {
return JSON.stringify(x);
}
```

**範例查詢︰**
```
SELECT
    DataString,
    DataValue,
    HexValue,
    UDF.json_stringify(input) As InputEvent
INTO
    output
FROM
    input PARTITION BY PARTITIONID
```

## <a name="clean-up-resources"></a>清除資源

若不再需要，可刪除資源群組、串流作業和所有相關資源。 刪除作業可避免因為作業使用串流單位而產生費用。 如果您計劃在未來使用該作業，您可以將其停止並在之後需要時重新啟動。 如果您將不繼續使用此作業，請使用下列步驟，刪除本快速入門所建立的所有資源：

1. 從 Azure 入口網站的左側功能表中，按一下 [資源群組]，然後按一下您所建立資源的名稱。  
2. 在資源群組頁面上，按一下 [刪除]，在文字方塊中輸入要刪除之資源的名稱，然後按一下 [刪除]。

## <a name="get-help"></a>取得說明
如需其他協助，請參閱我們的 [Azure 串流分析論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立可執行簡單 JavaScript 使用者定義函式的串流分析作業。 若要深入了解串流分析，請繼續前往即時案例的文章：

> [!div class="nextstepaction"]
> [Azure 串流分析中的即時 Twitter 情感分析](stream-analytics-twitter-sentiment-analysis-trends.md)
