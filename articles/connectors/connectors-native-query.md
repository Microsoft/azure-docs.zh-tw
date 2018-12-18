---
title: 在邏輯應用程式中新增查詢動作 | Microsoft Docs
description: 執行如篩選陣列等動作的查詢動作概觀。
services: ''
documentationcenter: ''
author: jeffhollan
manager: erikre
editor: ''
tags: connectors
ms.assetid: 34e702c7-f9e5-4885-9266-fc7404adecfe
ms.service: logic-apps
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/20/2016
ms.author: jehollan
ms.openlocfilehash: 05dd4ae3c4ee439d66401a3f5595f9104051f8ee
ms.sourcegitcommit: be9a42d7b321304d9a33786ed8e2b9b972a5977e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2018
ms.locfileid: "27962640"
---
# <a name="get-started-with-the-query-action"></a>開始使用查詢動作
您可以使用查詢動作來處理完成以下工作流程的批次和陣列︰

* 從資料庫建立所有高優先順序記錄的工作。
* 將電子郵件的所有 PDF 附件儲存到 Azure Blob。

若要開始在邏輯應用程式中使用查詢動作，請參閱 [建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="use-the-query-action"></a>使用查詢動作
動作是由邏輯應用程式中定義的工作流程所執行的作業。 [深入了解動作](connectors-overview.md)。  

查詢動作目前在設計工具中公開一項作業 (名為篩選陣列)。 這可讓您查詢陣列並傳回一組篩選結果。

以下說明如何將它新增至邏輯應用程式︰

1. 選取 [新增步驟]  按鈕。
2. 選擇 [新增動作] 。
3. 在動作搜尋方塊中，輸入**篩選**以列出**篩選陣列**動作。
   
    ![使用查詢動作](./media/connectors-native-query/using-action-1.png)
4. 選取要篩選的陣列 (以下螢幕擷取畫面顯示來自 Twitter 搜尋的結果陣列)。
5. 建立要針對每個項目評估的條件 (以下螢幕擷取畫面會篩選來自有 100 個以上追隨者之使用者的推文)。
   
    ![使用查詢動作](./media/connectors-native-query/using-action-2.png)
   
    此動作會輸出新的陣列，其中僅包含符合篩選條件需求的結果。
6. 按一下工具列左上角以便儲存，然後邏輯應用程式便會儲存並發佈 (啟動)。

\* 如果您要呼叫 HTTP 端點並接收 JSON 回應，請使用_剖析 JSON_ 動作來剖析 JSON 回應。 若不採取此步驟，_篩選陣列_就只會看到本文，但並不了解 JSON 裝載的結構。

## <a name="query-action"></a>查詢動作
以下是此連接器所支援動作的詳細資料。 連接器有一個可能的動作。

| 動作 | 說明 |
| --- | --- |
| 篩選陣列 |評估陣列中的每個項目是否符合條件並傳回結果 |

## <a name="action-details"></a>動作詳細資料
查詢動作隨附 1 個可能的動作。 下表說明動作的必要與選擇性輸入欄位，以及與使用動作相關聯的對應輸出詳細資料。

### <a name="filter-array"></a>篩選陣列
以下是動作的輸入欄位，可進行 HTTP 輸出要求。
標示 * 代表必要欄位。

| 顯示名稱 | 屬性名稱 | 說明 |
| --- | --- | --- |
| 從* |from |要篩選的陣列 |
| 條件* |其中 |要針對每個項目評估的條件 |

<br>

### <a name="output-details"></a>輸出詳細資料
以下是 HTTP 回應的輸出詳細資料。

| 屬性名稱 | 資料類型 | 說明 |
| --- | --- | --- |
| 篩選的陣列 |array |陣列，其中包含每筆篩選結果的物件 |

## <a name="next-steps"></a>後續步驟
立即試用平台和 [建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 您可以查看我們的 [API 清單](apis-list.md)，以探索邏輯應用程式中其他可用的連接器。

