---
title: 使用 Azure 入口網站模擬 API 回應 | Microsoft Docs
description: 此教學課程示範如何使用 API 管理 (APIM) 來設定 API 上的原則，讓它能夠傳回模擬的回應。 萬一無法使用後端傳送實際回應，這個方法讓開發人員能夠繼續實作和測試 API 管理執行個體。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom: mvc
ms.topic: tutorial
ms.date: 06/15/2018
ms.author: apimpm
ms.openlocfilehash: 916d0cf37ab3588091d4ca2d45f43a5669afe4f1
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094890"
---
# <a name="mock-api-responses"></a>模擬 API 回應

後端 API 可以匯入至 APIM API，或者以手動方式建立及管理。 此教學課程中的步驟示範如何使用 APIM 來建立空白的 API 並手動管理。 此教學課程示範如何設定 API 上的原則，讓它能夠傳回模擬的回應。 即使無法使用後端傳送實際回應，這個方法還是能讓開發人員繼續實作和測試 APIM 執行個體。 模擬回應的能力適用於許多案例：

+ 先設計 API 外觀，再於稍後進行後端實作時。 或者，以平行方式開發後端時。
+ 當後端暫時無法運作或無法調整規模時。

在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立測試 API 
> * 將作業新增到測試 API
> * 啟用回應模擬
> * 測試模擬的 API

![模擬的作業回應](./media/mock-api-responses/mock-api-responses01.png)

## <a name="prerequisites"></a>先決條件

完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。

## <a name="create-a-test-api"></a>建立測試 API 

此節中的步驟示範如何建立不具後端的空白 API。 它也會示範如何將作業新增到 API。 完成此節中的步驟之後呼叫作業會產生錯誤。 當您完成「啟用回應模擬」一節中的步驟之後，將不會收到任何錯誤。

1. 從 [API 管理] 服務選取 [API]。
2. 從左側功能表中選取 [+ 新增 API]。
3. 從清單選取 [空白 API]。
4. 針對 [顯示名稱] 輸入*測試 API*。
5. 針對 [產品] 輸入*無限制*。
6. 選取 [建立]。

## <a name="add-an-operation-to-the-test-api"></a>將作業新增到測試 API

1. 選取您在上一個步驟中建立的 API。
2. 按一下 [+ 新增作業]。
    ![模擬的作業回應](./media/mock-api-responses/mock-api-responses-add-operation.png)

    |設定|值|說明|
    |---|---|---|
    |**顯示名稱**|測試呼叫|顯示於**開發人員入口網站**中的名稱。|
    |**URL** (HTTP 指令動詞)|GET|您可以選擇其中一個預先定義的 HTTP 指令動詞。|
    |**URL** |*/test*|API 的 URL 路徑。 |
    |**說明**||提供作業的描述，它可用來提供文件給在**開發人員入口網站**中使用此 API 的開發人員。|
    |[查詢] 索引標籤||您可以新增查詢參數。 除了提供名稱和描述，您還能提供可指派給這個參數的值。 其中一個值可標示為預設值 (選擇性)。|
    |[要求] 索引標籤||您可以定義要求內容類型、範例及結構描述。 |
    |[回應] 索引標籤|請參閱此表格之後的步驟。|定義回應狀態碼、內容類型、範例及結構描述。|

3. 選取 [回應] 索引標籤 (位於 [URL]、[顯示名稱] 和 [描述] 欄位下方)。
4. 按一下 [+ 新增回應]。
5. 從清單中選取 [200 確定]。
6. 在右側的 [表示法] 標題下方，選取 [+ 新增表示法]。
7. 在搜尋方塊中輸入 "*application/json*"，然後選取 [application/json] 內容類型。
8. 在 [範例] 文字方塊中，輸入  `{ 'sampleField' : 'test' }`。
9. 選取 [建立]。

## <a name="enable-response-mocking"></a>啟用回應模擬

1. 選取您在「建立測試 API」步驟中建立的 API。
2. 選取您新增的測試作業。
3. 在右側視窗中，按一下 [設計] 索引標籤。
4. 在 [輸入處理] 視窗中，按一下鉛筆圖示。
5. 在 [模擬] 索引標籤上，針對 [模擬行為] 選取 [靜態回應]。
6. 在 [API 管理會傳回下列回應:] 文字方塊中，輸入 **200 確定，application/json**。 此選項表示您的 API 應該會傳回您在上一節中定義的回應範例。
    ![啟用回應模擬](./media/mock-api-responses/mock-api-responses-set-mocking.png)
7. 按一下 [檔案]。

## <a name="test-the-mocked-api"></a>測試模擬的 API

1. 選取您在「建立測試 API」步驟中建立的 API。
2. 開啟 [測試] 索引標籤。
3. 確定已選取 [測試呼叫] API。

    > [!TIP]
    > 具有**已啟用模擬**字樣的黃色列，表示從 API 管理傳回的回應會傳送模擬原則且不是實際的後端回應。

4. 選取 [傳送] 進行測試呼叫。
5. **HTTP 回應**會顯示此教學課程第一節中提供來作為範例的 JSON。
    ![啟用回應模擬](./media/mock-api-responses/mock-api-responses-test-response.png)

## <a name="video"></a>影片

> [!VIDEO https://www.youtube.com/embed/i9PjUAvw7DQ]
> 
> 

## <a name="next-steps"></a>後續步驟
在此教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立測試 API
> * 將作業新增到測試 API
> * 啟用回應模擬
> * 測試模擬的 API

前進到下一個教學課程：

> [!div class="nextstepaction"]
> [轉換及保護已發佈的 API](transform-api.md)
