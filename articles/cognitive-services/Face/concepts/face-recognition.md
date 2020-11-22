---
title: 臉部辨識概念
titleSuffix: Azure Cognitive Services
description: 本文說明驗證、尋找類似專案、群組，以及識別臉部辨識作業和基礎資料結構的概念。
services: cognitive-services
author: PatrickFarley
manager: nitime
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: pafarley
ms.openlocfilehash: 00dadf8a91b7ed01ab9f91933d296744305a95af
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518801"
---
# <a name="face-recognition-concepts"></a>臉部辨識概念

本文說明驗證、尋找類似專案、群組，以及識別臉部辨識作業和基礎資料結構的概念。 大致上，辨識會描述比較兩個不同臉部的工作，以判斷它們是否類似或屬於同一個人。

## <a name="recognition-related-data-structures"></a>辨識相關資料結構

辨識作業主要使用下列資料結構。 這些物件會儲存在雲端中，並可由其識別碼字串參考。 識別碼字串在訂用帳戶中一律是唯一的。 名稱欄位可以重複。

|名稱|說明|
|:--|:--|
|DetectedFace| [臉部偵測](../Face-API-How-to-Topics/HowtoDetectFacesinImage.md)作業會取出這個單一臉部標記法。 它的識別碼會在建立後24小時到期。|
|PersistedFace| 將 >detectedface 物件新增至群組（例如 FaceList 或 Person）時，它們會變成 >persistedface 物件。 您可以隨時抓取它們，而 [不會過期](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524c) 。|
|[FaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b) 或 [LargeFaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc)| 此資料結構是 >persistedface 物件的各種清單。 FaceList 具有唯一識別碼、名稱字串，以及選擇性的使用者資料字串。|
|[人員](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c)| 此資料結構是屬於相同人員的 >persistedface 物件清單。 它有唯一的識別碼、名稱字串，以及選擇性的使用者資料字串。|
|[PersonGroup](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) 或 [LargePersonGroup](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d)| 此資料結構是人員物件的各種清單。 它有唯一的識別碼、名稱字串，以及選擇性的使用者資料字串。 PersonGroup 必須先 [經過定型](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) ，才能在辨識作業中使用。|

## <a name="recognition-operations"></a>辨識作業

本節將詳細說明四項辨識作業如何使用先前所述的資料結構。 如需每個辨識作業的廣泛描述，請參閱 [總覽](../Overview.md)。

### <a name="verify"></a>確認

[驗證](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a)作業會接受來自 >detectedface 或 >persistedface 的臉部識別碼，以及另一個臉部識別碼或 Person 物件，並判斷它們是否屬於同一個人。 如果您傳入 Person 物件，您可以選擇性地傳入該人員所屬的 PersonGroup 來改善效能。

### <a name="find-similar"></a>尋找類似項目

「 [尋找相似](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) 」作業會接受來自 >detectedface 或 >persistedface 的臉部識別碼，以及 FaceList 或其他臉部識別碼的陣列。 使用 FaceList 時，它會傳回較小的臉部 FaceList，類似于指定的臉部。 使用臉部識別碼陣列時，類似的會傳回較小的陣列。

### <a name="group"></a>Group

[群組](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238)作業會接受來自 >detectedface 或 >persistedface 的各種臉部識別碼陣列，並傳回群組為數個較小陣列的相同識別碼。 每個「群組」陣列都包含看似類似的臉部識別碼。 單一 "messyGroup" 陣列包含找不到相似性的臉部識別碼。

### <a name="identify"></a>識別

[識別](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239)作業會接受來自 >detectedface 或 >persistedface 的一或多個臉部識別碼和 PersonGroup，並傳回每個臉部可能屬於的人員物件清單。 傳回的 Person 物件會包裝為具有預測信賴值的候選物件。

## <a name="input-data"></a>輸入資料

使用下列秘訣，以確保您的輸入影像提供最精確的辨識結果：

* 支援的輸入影像格式包括 JPEG、PNG、GIF (第一個畫面格) 、BMP。
* 影像檔案大小不應大於 6 MB。
* 當您建立 Person 物件時，請使用具有不同角度和照明特性的相片。
* 某些臉部可能因技術挑戰而無法辨識，例如：
  * 具有極端照明的影像，例如嚴重的背光。
  * 封鎖一或兩個眼睛的障礙物。
  * 頭髮型別或臉部線的差異。
  * 臉部外觀因為年齡的變化。
  * 極端臉部表情。

## <a name="next-steps"></a>後續步驟

既然您已熟悉臉部辨識概念，請撰寫腳本，以根據定型的 PersonGroup 來識別臉部。

* [臉部用戶端程式庫快速入門](../Quickstarts/client-libraries.md)
