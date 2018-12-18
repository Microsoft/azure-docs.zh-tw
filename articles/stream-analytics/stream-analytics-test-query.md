---
title: 使用樣本資料測試 Azure 串流分析作業
description: 如何在串流分析作業中測試查詢。
keywords: 本文說明如何使用 Azure 入口網站測試 Azure 串流分析作業、輸入取樣，以及上傳樣本資料。
services: stream-analytics
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
manager: kfile
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/27/2018
ms.openlocfilehash: 63f9b98bcd70378ae02769298f0ffcd796378612
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44027260"
---
# <a name="test-a-stream-analytics-query-with-sample-data"></a>使用樣本資料測試串流分析查詢

您可以使用 Azure 串流分析，在 Azure 入口網站中上傳樣本資料並測試查詢，而無須啟動或停止作業。

## <a name="upload-sample-data-and-test-the-query"></a>上傳範例資料和測試查詢

1. 登入 Azure 入口網站。 

2. 找到現有的串流分析作業並加以選取。

3. 在 [串流分析作業] 頁面上，[工作拓撲] 標題下方，選取 [查詢] 開啟 [查詢] 編輯器視窗。 

4. 若要使用樣本輸入資料測試查詢，請以滑鼠右鍵按一下任何輸入資料。  然後選取 [從檔案上傳樣本資料]。 必須以 JSON、CSV 或 AVRO 將資料序列化。 範例輸入必須以 UTF-8 編碼而且不壓縮。 在入口網站上測試 CSV 輸入時僅支援逗號 (,) 分隔符號。

    ![串流分析查詢編輯器的測試查詢](media/stream-analytics-test-query/stream-analytics-test-query-editor-upload.png)

5. 上傳完成後，選取 [測試] 以根據您提供的樣本資料測試此查詢。

    ![串流分析查詢編輯器的樣本資料](media/stream-analytics-test-query/stream-analytics-test-query-editor-test.png)

6. 如果您需要該測試輸出供稍後使用，您的查詢輸出會顯示在瀏覽器中，並提供連結，以便下載結果。 

7. 反覆修改查詢，然後再次測試，以查看輸出的變更狀況。

   ![串流分析查詢編輯器樣本輸出](media/stream-analytics-test-query/stream-analytics-test-query-editor-samples-output.png)

   當您在查詢中使用多個輸出時，結果會分別顯示在不同的索引標籤中，您可以輕鬆地切換索引標籤。

8. 驗證顯示在瀏覽器中的結果後，請按一下 [儲存] 以儲存查詢。 然後按一下 [啟動] 以啟動作業，並讓其處理連入事件。

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
