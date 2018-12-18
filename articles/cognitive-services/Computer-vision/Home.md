---
title: 什麼是電腦視覺 API？
titlesuffix: Azure Cognitive Services
description: 電腦視覺服務可供開發人員存取進階演算法，以處理影像及傳回資訊。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: overview
ms.date: 08/22/2018
ms.author: pafarley
ms.openlocfilehash: 03cf2fab1200cd617f456a6fcfb9067673d01a19
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340897"
---
# <a name="what-is-computer-vision"></a>什麼是電腦視覺？

雲端式電腦視覺服務可供開發人員存取進階演算法，以處理影像及傳回資訊。 電腦視覺可處理廣受使用的影像格式，例如 JPEG 和 PNG。 若要分析影像，您可以上傳影像或指定影像 URL。 電腦視覺演算法可根據您感興趣的視覺化功能，以不同的方式分析影像的內容。 例如，電腦視覺可判斷影像中是否包含成人或猥褻內容，或尋找影像中的所有臉部。

您可以使用我們的[用戶端程式庫](quickstarts-sdk/csharp-analyze-sdk.md)叫用服務，或直接叫用 [REST API](vision-api-how-to-topics/howtocallvisionapi.md)，在應用程式中使用電腦視覺執行下列作業：

- [深入分析影像](#analyzing-images-for-insight)
- [擷取影像中的文字](#extracting-text-from-images)
- [調節影像中的內容](#moderating-content-in-images)

## <a name="analyzing-images-for-insight"></a>深入分析影像

您可以使用電腦視覺功能分析影像，以偵測並提供與影像的視覺特徵和特性有關的深入解析。 您可以上傳影像的內容以分析本機影像，或是指定影像的 URL 以分析遠端影像。

電腦視覺在分析影像時可執行下列動作：

| 動作 | 說明 |
| ------ | ----------- |
|**[標記視覺特徵](concept-tagging-images.md)**|依據 2,000 多個可辨識的物件、生物、景象和動作，識別及標記影像中的視覺特徵。 若標記不明確或不屬於常識，回應會提供「提示」來釐清標記在已知設定內容中的意涵。 標記並未限定於主體 (例如前景中的人物)，而是包含周遭環境 (室內或室外)、家具、工具、植物、動物、配件和小工具等。|
|**[將影像分類](concept-categorizing-images.md)**|使用具有父/子承襲階層的[類別分類法](Category-Taxonomy.md)來識別及分類整個影像。 類別可單獨使用，或與我們新的標記模型搭配使用。<br/>目前，英文是唯一支援影像標記和分類的語言。|
|**[說明影像](concept-describing-images.md)**|以一般人看得懂的語言，使用完整的句子產生整個影像的描述。 電腦視覺的演算法會根據在影像中識別出來的物件產生各種描述。 這些描述會個別受到評估，並產生信賴分數。 接著會傳回一份清單，並依照信賴分數由高至低排序。<br/>您可以在 [GitHub](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) 取得使用這項技術產生影像標題的 Bot 範例。|
|**[偵測臉部](concept-detecting-faces.md)** |偵測影像中的臉部，並提供與每個偵測到的臉部有關的資訊。 電腦視覺會針對每個偵測到的臉部傳回座標、矩形、性別和年齡。<br/>電腦視覺提供位於[臉部](/azure/cognitive-services/face/)的功能子集，且您可以使用「臉部」服務執行更詳細的分析，例如臉部識別和姿勢偵測。|
|**[偵測影像類型](concept-detecting-image-types.md)**|偵測影像的關於特性，例如影像是否為線條繪圖，或影像為美工圖案的可能性。|
|**[偵測特定領域內容](concept-detecting-domain-content.md)**|使用領域模型可偵測及識別影像中的特定領域內容，例如名人和地標。 例如，如果影像中包含人物，電腦視覺即可使用服務隨附的名人領域模型，判斷在影像中偵測到的人物是否和已知的名人相符。|
|**[偵測色彩配置](concept-detecting-color-schemes.md)**|分析影像中的用色方式。 電腦視覺可判斷影像是黑白還是彩色的，如果是彩色影像，則會找出主色和輔色。|
|**[產生縮圖](concept-generating-thumbnails.md)**|分析影像的內容，為其產生適當的縮圖。 電腦視覺會先產生高品質的縮圖，然後分析該影像內的物件，以判斷*相關區域* (ROI)。 接著，電腦視覺會裁剪影像以符合「相關區域」的需求。 產生的縮圖可以使用與原始影像的外觀比例不同的外觀比例來呈現，視您的需求而定。|

## <a name="extracting-text-from-images"></a>擷取影像中的文字

您可以使用電腦視覺功能，將影像中的[文字擷取 (使用 OCR)](concept-extracting-text-ocr.md) 為機器可讀取的字元資料流。 如有需要，OCR 會沿著水平影像軸以度為單位校正已辨識文字的旋轉角度，並提供每個字的框架座標。 OCR 支援 25 種語言，且會根據擷取的文字自動偵測其語言。

您也可以從影像中[辨識印刷和手寫文字](concept-recognizing-text.md)。 電腦視覺可從表層和背景不同的各種物件影像中偵測並擷取出印刷和手寫的文字，例如收據、海報、名片、信件和白板等。 目前，辨識印刷和手寫文字的功能處於預覽階段，英文是唯一支援的語言。  

## <a name="moderating-content-in-images"></a>調節影像中的內容

您可以使用電腦視覺在影像中[偵測成人和猥褻內容](concept-detecting-adult-content.md)、評估影像中包含成人或猥褻內容的可能性，及產生兩者的信賴分數。 成人和猥褻內容偵測的篩選條件可用滑動標尺來設定，以配合您的喜好設定。

## <a name="image-requirements"></a>影像需求

電腦視覺可分析符合下列需求的影像：

- 必須以 JPEG、PNG、GIF 或 BMP 格式呈現的影像
- 影像的檔案大小必須小於 4 MB
- 影像的維度必須大於 50 x 50 像素  
  針對 OCR，影像的大小必須介於 50 x 50 與 4200 x 4200 像素之間

## <a name="next-steps"></a>後續步驟

透過下列其中一個快速入門開始使用電腦視覺：

- [分析影像](quickstarts-sdk/csharp-analyze-sdk.md)
- [擷取手寫文字](quickstarts-sdk/csharp-hand-text-sdk.md)
- [產生縮圖](quickstarts-sdk/csharp-thumb-sdk.md)
