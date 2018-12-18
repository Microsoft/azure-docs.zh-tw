---
title: 在 Azure Logic Apps 中處理大型訊息 | Microsoft Docs
description: 了解如何在 Logic Apps 中利用區塊化處理大型訊息大小
services: logic-apps
documentationcenter: ''
author: shae-hurst
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: logic-apps
ms.workload: logic-apps
ms.devlang: ''
ms.tgt_pltfrm: ''
ms.topic: article
ms.date: 4/27/2018
ms.author: shhurst
ms.openlocfilehash: 6064db5455d92d15dca0e2a4a78285f0aeade904
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35299040"
---
# <a name="handle-large-messages-with-chunking-in-logic-apps"></a>在 Logic Apps 中利用區塊化處理大型訊息大小

在處理訊息時，Logic Apps 會限制訊息內容的大小上限。 此限制有助於減少儲存和處理大型訊息所造成的額外負荷。 若要處理大於此限制的訊息，Logic Apps 可將大型訊息分割成多個較小的訊息。 透過這種方式，您仍可在特定情況下使用 Logic Apps 傳輸大型檔案。 在透過連接器或 HTTP 與其他服務通訊時，Logic Apps 可以消化大型訊息，但前提是訊息必須先分割為區塊。 這項條件代表連接器也必須支援區塊化，或者 Logic Apps 與這些服務間的基礎 HTTP 訊息交換必須使用區塊化。

本文會為您示範如何為大過限制的訊息設定區塊化支援。

## <a name="what-makes-messages-large"></a>要如何判定訊息「過大」？

訊息是否「過大」須以處理這些訊息的服務為基準。 因此 Logic Apps 和連接器對大型訊息的實際大小限制會有所不同。 當 Logic Apps 和連接器都無法直接消化大型訊息時，就必須加以分塊。 針對 Logic Apps 的訊息大小限制，請參閱 [Logic Apps limits and configuration](../logic-apps/logic-apps-limits-and-config.md) (Logic Apps 限制與設定)。
針對各連接器的訊息大小限制，則請參閱 [connector's specific technical details](../connectors/apis-list.md) (連接器的特定技術詳細資料)。

### <a name="chunked-message-handling-for-logic-apps"></a>Logic Apps 的分塊訊息處理

Logic Apps 無法直接使用超過訊息大小上限的分塊訊息輸出。 只有支援區塊化的動作能夠存取這些輸出中的訊息內容。 因此，處理大型訊息的動作須符合以下任一準則：

* 當該動作屬於連接器時，原本就支援區塊化。 
* 已在該動作的執行階段設定中啟用了區塊化支援。 

若未符合任一準則，您就會在嘗試存取大型內容輸出時得到執行階段錯誤。 若要啟用區塊化，請參閱 [Set up chunking support](#set-up-chunking) (設定區塊化支援)。

### <a name="chunked-message-handling-for-connectors"></a>連接器的分塊訊息處理

與 Logic Apps 通訊的服務可能有其本身的訊息大小限制。 這些限制通常小於 Logic Apps 的限制。 舉例來說，假設連接器支援區塊化，那麼連接器可能會認為 30 MB 的訊息過大，而 Logic Apps 則否。 為了與此連接器的限制達成一致性，Logic Apps 必須將所有大於 30 MB 的訊息分割為較小的區塊。

這些支援區塊化的連接器終端使用者，並無法查看基礎區塊化通訊協定。 而因為並非所有連接器均支援區塊化，所以這些連接器會在傳入的訊息超過連接器的大小限制時，產生執行階段錯誤。

<a name="set-up-chunking"></a>

## <a name="set-up-chunking-over-http"></a>在 HTTP 上設定區塊化

在一般 HTTP 案例中，您可以分割大型內容下載並透過 HTTP 上傳，使您的邏輯應用程式與端點得以交換大型訊息。 不過，您分割訊息的方式必須符合 Logic Apps 的標準。 

若端點已為下載或上傳啟用了區塊化，則您邏輯應用程式中的 HTTP 動作就會自動分割大型訊息。 反之，您就得在端點上設定區塊化支援。 如果您不具有端點或連接器的擁有權或控制權，就可能沒有設定區塊化的選項。

此外，如果 HTTP 動作尚未啟用區塊化，您也必須在動作的 `runTimeConfiguration` 屬性中設定區塊化。 您可以直接在之後提及的程式碼檢視編輯器，或是此處所述的 Logic Apps 設計工具中，在動作內設定此屬性。

1. 在 HTTP 動作的右上角選擇省略符號按鈕 (**...**)，然後選擇 [設定]。

   ![在動作上開啟 [設定] 功能表](./media/logic-apps-handle-large-messages/http-settings.png)

2. 在 [Content Transfer] \(內容傳輸\) 下將 [Allow chunking] \(允許區塊化\) 設定為 [On] \(開啟\)。

   ![開啟區塊化](./media/logic-apps-handle-large-messages/set-up-chunking.png)

3. 若要繼續為下載或上傳設定區塊化，請遵循以下各節繼續。

<a name="download-chunks"></a>

## <a name="download-content-in-chunks"></a>以區塊下載內容

許多端點在接收透過 HTTP GET 要求的下載時，會自動以區塊傳送大型訊息。 為了從端點透過 HTTP 下載區塊化訊息，端點必須支援 partial content (部分內容) 要求，或是 *chunked downloads* (分塊下載)。 當您的邏輯應用程式向端點傳送 HTTP GET 要求以下載內容，而端點回應 "206" 狀態碼時，回應會包含分塊的內容。 Logic Apps 無法控制端點是否支援部分要求。 不過，當您的邏輯應用程式得到第一個 "206" 回應時，會自動傳送多個要求以下載所有內容。

若要確認端點是否支援部分內容，請傳送 HEAD 要求。 此要求可協助您判斷回應是否包含 `Accept-Ranges` 標頭。 如此一來，就能在端點支援分塊下載但未傳送分塊內容時，在 HTTP GET 要求中設定 `Range`標頭來建議此選項。 

以下步驟詳述了 Logic Apps 從端點將分塊內容下載至您邏輯應用程式所用的流程：

1. 您的邏輯應用程式傳送 HTTP GET 要求至端點。

   要求標頭可包含 `Range` 欄位，描述要求內容區塊的位元組範圍。

2. 端點回應 "206" 狀態碼和 HTTP 訊息本文。

    此區塊內容的詳細資料會出現在回應的 `Content-Range` 標頭中，包含可協助 Logic Apps 在進行區塊化前，判斷區塊頭尾與整體內容大小總計。

3. 您的邏輯應用程式自動傳送後續 HTTP GET 要求。

    您的邏輯應用程式繼續傳送後續 GET 要求，直到擷取所有內容為止。

舉例來說，這項動作定義呈現了設定 `Range` 標頭的 HTTP GET 要求。 該標頭會建議端點應以分塊內容回應：

```json
"getAction": {
    "inputs": {
        "headers": {
            "Range": "bytes=0-1023"
        },
       "method": "GET",
       "uri": "http://myAPIendpoint/api/downloadContent"
    },
    "runAfter": {},
    "type": "Http"
}
```

GET 要求將 "Range" 標頭設定為 "bytes=0-1023"，這是位元組範圍。 若端點支援部分內容的要求，則端點會根據要求的範圍以內容區塊回應。 根據端點而定，"Range" 標頭欄位的實際格式可能會有所不同。

<a name="upload-chunks"></a>

## <a name="upload-content-in-chunks"></a>以區塊上傳內容

若要從 HTTP 動作上傳分塊內容，動作本身的 `runtimeConfiguration` 屬性中必須已啟用區塊化支援。 此設定可讓動作啟動區塊化通訊協定。 您的邏輯應用程式隨後可傳送起始 POST 或 PUT 訊息至目標端點。 在端點以建議的區塊大小回應後，您的邏輯應用程式隨後會傳送包含內容區塊的 HTTP PATCH 要求。

以下步驟詳述了 Logic Apps 從您邏輯應用程式將分塊內容上傳至端點所用的流程：

1. 您的邏輯應用程式送包含空白訊息本文的起始 HTTP POST 或 PUT 要求。 要求標頭會包含此資訊，提其您邏輯應用程式想要以區塊上傳的內容：

   | Logic Apps 要求標頭欄位 | 值 | 類型 | 說明 |
   |---------------------------------|-------|------|-------------|
   | **x-ms-transfer-mode** | chunked | 字串 | 指出內容以區塊上傳 |
   | **x-ms-content-length** | <*content-length*> | 整數  | 進行區塊化前的所有內容大小 (位元組) |
   ||||

2. 端點回應 "200" 成功狀態碼和此選擇性資訊：

   | 端點回應標頭欄位 | 類型 | 必要 | 說明 |
   |--------------------------------|------|----------|-------------|
   | **x-ms-chunk-size** | 整數  | 否 | 建議的區塊大小 (位元組) |
   | **位置** | 字串 | 否 | 傳送 HTTP PATCH 訊息的 URL 位置 |
   ||||

3. 您的邏輯應用程式建立並傳送後續 HTTP PATCH 訊息，每項訊息都帶有此資訊：

   * 內容區塊會以 **x-ms-chunk-size**或其他內部計算的大小為基準，直到所有總計 **x-ms-content-length** 的內容均依序上傳完畢為止。

   * 這些與內容區塊相關的標題詳細資料會在各個 PATCH 訊息中傳出：

     | Logic Apps 要求標頭欄位 | 值 | 類型 | 說明 |
     |---------------------------------|-------|------|-------------|
     | **Content-Range** | <*range*> | 字串 | 目前內容區塊的位元組範圍，包含開始值、結束值和內容大小總計，例如："bytes=0-1023/10100" |
     | **Content-Type** | <*content-type*> | 字串 | 分塊內容的類型 |
     | **Content-Length** | <*content-length*> | 字串 | 目前區塊的大小長度 (位元組) |
     |||||

4. 端點會在收到每項 PATCH 要求之後回應 "200" 狀態碼，來確認各區塊的接收狀況。

舉例來說，這項動作定義呈現了將分塊內容傳送至端點的 HTTP POST 要求。 在動作的 `runTimeConfiguration` 屬性中，`contentTransfer` 屬性將 `transferMode` 設定為 `chunked`：

```json
"postAction": {
    "runtimeConfiguration": {
        "contentTransfer": {
            "transferMode": "chunked"
        }
    },
    "inputs": {
        "method": "POST",
        "uri": "http://myAPIendpoint/api/action",
        "body": "@body('getAction')"
    },
    "runAfter": {
    "getAction": ["Succeeded"]
    },
    "type": "Http"
}
```