---
title: 範例：呼叫電腦視覺 API
titlesuffix: Azure Cognitive Services
description: 了解如何使用「Azure 認知服務」中的 REST 呼叫電腦視覺 API。
services: cognitive-services
author: KellyDF
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: sample
ms.date: 01/20/2017
ms.author: kefre
ms.openlocfilehash: e8297fbe59ebe2dea9caf112ebea4517447cf9e0
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45981740"
---
# <a name="example-how-to-call-the-computer-vision-api"></a>範例：如何呼叫電腦視覺 API

此指南示範如何使用 REST 呼叫電腦視覺 API。 這些範例均使用電腦視覺 API 用戶端程式庫以 C# 撰寫，並作為 HTTP POST/GET 呼叫。 我們將著重在：

-   如何取得「標籤」、「描述」與「類別」。
-   如何取得「特定領域」資訊 (名人)。

### <a name="Prerequisites">先決條件</a> 
在本機儲存影像的影像 URL 或路徑。
  * 支援的輸入方法：應用程式/八位元資料流或影像 URL 形式的原始影像二進位檔
  * 支援的影像格式：JPEG、PNG、GIF、BMP
  * 影像檔案大小：小於 4 MB
  * 影像尺寸：大於 50 x 50 像素
  
下面的範例會示範下列功能：

1. 分析影像並取得傳回的標籤與描述陣列。
2. 以特定領域模型 (具體而言，「名人」模型) 分析影像，然後取得所傳回 JSON 中的對應結果。

功能細分如下：

  * **選項一：** 特定範圍分析 - 只分析指定的模型
  * **選項二︰** 強化分析 - 使用 [86 個類別的分類法](../Category-Taxonomy.md)分析，提供其他詳細資料
  
### <a name="Step1">步驟 1：授權 API 呼叫</a> 
每次呼叫電腦視覺 API 時，都需要訂用帳戶金鑰。 這個金鑰必須透過查詢字串參數傳遞，或是在要求標頭中指定。 

若要取得訂用帳戶金鑰，請參閱[如何取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md
)。

**1.** 透過查詢字串傳遞訂用帳戶金鑰，請參閱下面的電腦視覺 API 範例：

```https://westus.api.cognitive.microsoft.com/vision/v2.0/analyze?visualFeatures=Description,Tags&subscription-key=<Your subscription key>```

**2.** 傳遞訂用帳戶金鑰的作業也可以在 HTTP 要求標頭中指定：

```ocp-apim-subscription-key: <Your subscription key>```

**3.** 使用用戶端程式庫時，訂用帳戶金鑰會透過 VisionServiceClient 的建構函式傳入：

```var visionClient = new VisionServiceClient(“Your subscriptionKey”);```

### <a name="Step2">步驟 2：將影像上傳到電腦視覺 API 服務，並取回標籤、描述與名人</a>
執行電腦視覺 API 呼叫的基本方式，就是直接上傳影像。 這是透過傳送 "POST" 要求並附上應用程式/八位元資料流內容類型，以及從影像讀取的資料來完成。 針對「標籤」與「描述」，所有電腦視覺 API 呼叫都使用這一種上傳方法。 唯一的差異是使用者指定的查詢參數。 

以下說明如何取得指定影像的「標籤」與「描述」:

**選項一：** 取得「標籤」清單與一個「描述」
```
POST https://westus.api.cognitive.microsoft.com/vision/v2.0/analyze?visualFeatures=Description,Tags&subscription-key=<Your subscription key>
```
```
using Microsoft.ProjectOxford.Vision;
using Microsoft.ProjectOxford.Vision.Contract;
using System.IO;

AnalysisResult analysisResult;
var features = new VisualFeature[] { VisualFeature.Tags, VisualFeature.Description };

using (var fs = new FileStream(@"C:\Vision\Sample.jpg", FileMode.Open))
{
  analysisResult = await visionClient.AnalyzeImageAsync(fs, features);
}
```
**選項二：** 只取得「標籤」清單，或只有「描述」清單

###### <a name="tags-only"></a>僅標籤：
```
POST https://westus.api.cognitive.microsoft.com/vision/v2.0/tag&subscription-key=<Your subscription key>
var analysisResult = await visionClient.GetTagsAsync("http://contoso.com/example.jpg");
```

###### <a name="description-only"></a>僅描述：
```
POST https://westus.api.cognitive.microsoft.com/vision/v2.0/describe&subscription-key=<Your subscription key>
using (var fs = new FileStream(@"C:\Vision\Sample.jpg", FileMode.Open))
{
  analysisResult = await visionClient.DescribeAsync(fs);
}
```
### <a name="here-is-how-to-get-domain-specific-analysis-in-our-case-for-celebrities"></a>以下說明如何取得特定領域分析 (在此案例中為名人)。

**選項一：** 特定範圍分析 - 只分析指定的模型
```
POST https://westus.api.cognitive.microsoft.com/vision/v2.0/models/celebrities/analyze
var celebritiesResult = await visionClient.AnalyzeImageInDomainAsync(url, "celebrities");
```
針對此選項，所有其他查詢參數 {visualFeatures, details} 皆無效。 如果您想要查看所有支援的模型，請使用： 
```
GET https://westus.api.cognitive.microsoft.com/vision/v2.0/models 
var models = await visionClient.ListModelsAsync();
```
**選項二︰** 強化分析 - 使用 [86 個類別的分類法](../Category-Taxonomy.md)分析，提供其他詳細資料

除了一或多個特定領域模型中的詳細資料之外，針對您還想要取得一般影像分析的應用程式，我們使用模型查詢參數來擴充 v1 API。
```
POST https://westus.api.cognitive.microsoft.com/vision/v2.0/analyze?details=celebrities
```
叫用此方法時，將先呼叫 86 類別分類器。 如果有任何類別符合已知/比對模型的類別，則會發生第二輪的分類器叫用。 例如，如果 "details=all" 或 "details" 包含 ‘celebrities’，將在呼叫 86 類別分類器之後，呼叫名人模型，而結果會包含該類別的人員。 相較於選項一，這對於對名人感興趣的使用者，會增加延遲。

在此案例中，所有 v1 查詢參數的運作方式都會相同。  如果未指定 visualFeatures=categories，則會隱含地啟用。

### <a name="Step3">步驟 3：擷取和了解 analyze&visualFeatures = 標籤、描述的 JSON 輸出</a>

以下是範例：
```
  {
    “tags”: [
    {
    "name": "outdoor",
        "score": 0.976
    },
    {
    "name": "bird",
        "score": 0.95
    }
            ],
    “description”: 
    {
    "tags": [
    "outdoor",
    "bird"
            ],
    "captions": [
    {
    "text”: “partridge in a pear tree”,
            “confidence”: 0.96
    }
                ]
    }
  }
```
欄位   | 類型  | 內容
------|------|------|
標記    | 物件    | 標籤陣列的最上層物件
tags[].Name | 字串    | 標籤分類器的關鍵字
tags[].Score    | number    | 信賴分數，介於 0 到 1 之間。
說明  | 物件   | 描述的最上層物件。
description.tags[] |    字串  | 標籤清單。  如果能否產生標題的信賴度不足，呼叫者可能僅有標籤的資訊可用。
description.captions[].text | 字串    | 描述影像的片語。
description.captions[].confidence   | number    | 片語的信賴度。

### <a name="Step4">步驟 4：擷取和了解 JSON 輸出的特定領域模型</a>

**選項一：** 特定範圍分析 - 只分析指定的模型

輸出就是標籤陣列，範例如下：
```
  { 
    "result": [ 
      { 
    "name": "golden retriever", 
    "score": 0.98
    },
    { 
    "name": "Labrador retriever", 
    "score": 0.78
    }
              ]
  }
```

**選項二︰** 強化分析 - 使用 86 個類別的分類法分析，提供其他詳細資料

針對使用「選項二 (強化分析)」的特定領域模型，會擴充類別傳回類型。 範例如下：
```
  {
    "requestId": "87e44580-925a-49c8-b661-d1c54d1b83b5",
    "metadata":     {
      "width": 640,
      "height": 430,
      "format": "Jpeg"
                    },
    "result": {
      "celebrities": 
      [
        {
        "name": "Richard Nixon",
        "faceRectangle": {
          "left": 107,
          "top": 98,
          "width": 165,
          "height": 165
                         },
        "confidence": 0.9999827
        }
      ]
  }
```

類別欄位是原始分類法中 [86 個類別](../Category-Taxonomy.md)的一或多個類別。 也請注意，以底線結尾的類別會符合該類別與其子項 (例如，名人模型的 people_ 與 people_group)。

欄位   | 類型  | 內容
------|------|------|
類別 | 物件 | 最上層物件
categories[].name    | 字串   | 86 個類別分類法的名稱
categories[].score  | number    | 信賴分數，介於 0 到 1 之間
categories[].detail  | 物件？      | 選擇性的詳細資料物件

請注意，如果多個類別皆符合 (例如，當 model=celebrities 時，86 個類型分類器傳回 people_ 與 people_young 兩者的分數)，以附加在最一般層級的詳細資料為符合者 (在該範例中為 people_ in)。

### <a name="Errors">錯誤回應</a>
這些和 vision.analyze 完全相同，還有可能會在「選項一」與「選項二」情節中傳回 NotSupportedModel error (HTTP 400) 的其他錯誤。 針對「選項二 (強化分析)」，如果無法辨識詳細資料中指定的任何模型，即使其中一或多個有效，API 也會傳回 NotSupportedModel。  使用者可以呼叫 listModels，以了解支援哪些模型。

### <a name="Summary">總結</a>

以下都是電腦視覺 API 的基本功能：如何上傳影像以及擷取傳回的寶貴中繼資料。

若要使用 REST API，請前往[電腦視覺 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44) \(英文\)。
 
