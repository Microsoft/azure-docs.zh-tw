---
title: 什麼是電腦視覺 API？
titlesuffix: Azure Cognitive Services
description: 電腦視覺 API 可供開發人員存取進階演算法，以處理影像及傳回資訊。
services: cognitive-services
author: KellyDF
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: overview
ms.date: 08/10/2017
ms.author: kefre
ms.openlocfilehash: e2f3a5655b2fbedf3ad80d555421599e26225196
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45982074"
---
# <a name="what-is-computer-vision-api-version-10"></a>什麼是電腦視覺 API 1.0 版？

> [!IMPORTANT]
> 現在已有新版的電腦視覺 API 可供使用，請參閱：
>- [概觀](https://docs.microsoft.com/azure/cognitive-services/computer-vision/home)
>- [電腦視覺 API 2.0 版](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44)

雲端式電腦視覺 API 可供開發人員存取進階演算法，以處理影像及傳回資訊。 藉由上傳影像或指定影像 URL，Microsoft 電腦視覺演算法可根據輸入和使用者選擇，透過不同方式分析視覺內容。 透過電腦視覺 API，使用者將可分析影像以執行下列動作：
* [根據內容為影像加上標記。](#Tagging)
* [將影像分類。](#Categorizing)
* [識別影像的類型和品質。](#Identifying)
* [偵測人臉並傳回其座標。](#Faces)
* [辨識特定領域內容。](#Domain-Specific)
* [產生內容的描述。](#Descriptions)
* [使用光學字元辨識來識別影像中的列印文字。](#OCR)
* [辨識手寫文字。](#RecognizeText)
* [區分色彩配置。](#Color)
* [標示成人內容。](#Adult)
* [裁剪要作為縮圖的相片。](#Thumbnails)

## <a name="requirements"></a>需求
* 支援的輸入法：應用程式/八位元資料流或影像 URL 形式的原始影像二進位檔。
* 支援的影像格式：JPEG、PNG、GIF、BMP。
* 影像檔案大小：小於 4 MB。
* 影像尺寸：大於 50 x 50 像素。

## <a name="tagging-images"></a>標記影像
電腦視覺 API 會依據超過 2,000 個的可辨識物件、生物、景象及動作傳回標記。 若標記不明確或不屬於常識，API 回應會提供「提示」來釐清標記在已知設定內容中的意涵。 標籤不會組織成分類，且不會有繼承階層存在。 內容標記的集合會形成影像「描述」的基礎，讓此描述顯示為完整句子中已格式化而可供閱讀的語言。 請注意，英文是影像描述目前唯一支援的語言。

在上傳影像或指定影像 URL 之後，電腦視覺 API 的演算法會根據在影像中識別出的物件、生物和動作來輸出標記。 標記並未限定於主體 (例如前景中的人物)，而是包含周遭環境 (室內或室外)、家具、工具、植物、動物、配件和小工具等。

### <a name="example"></a>範例
![House_Yard](./Images/house_yard.png) '

```json
Returned Json
{
   'tags':[
      {
         "name":"grass",
         "confidence":0.999999761581421
      },
      {
         "name":"outdoor",
         "confidence":0.999970674514771
      },
      {
         "name":"sky",
         "confidence":999289751052856
      },
      {
         "name":"building",
         "confidence":0.996463239192963
      },
      {
         "name":"house",
         "confidence":0.992798030376434
      },
      {
         "name":"lawn",
         "confidence":0.822680294513702
      },
      {
         "name":"green",
         "confidence":0.641222536563873
      },
      {
         "name":"residential",
         "confidence":0.314032256603241
      },
   ],
}
```
## <a name="categorizing-images"></a>將影像分類
除了標記和描述以外，電腦視覺 API 也會傳回舊版本中定義的分類型類別。 這些類別會以父/子承襲階層組織為分類。 所有類別皆採用英文。 這些類別可單獨使用，或與我們的新模型搭配使用。

### <a name="the-86-category-concept"></a>86 類別概念
根據下圖中列出的 86 種概念，影像中的視覺功能可進行明確或特定範圍的分類。 如需文字格式的完整分類，請參閱[類別分類](https://docs.microsoft.com/azure/cognitive-services/computer-vision/category-taxonomy)。

![分析類別](./Images/analyze_categories.png)

影像                                                  | 回應
------------------------------------------------------ | ----------------
![屋頂上的女人](./Images/woman_roof.png)                 | people
![家人照片](./Images/family_photo.png)             | people_crowd
![可愛的小狗](./Images/cute_dog.png)                     | animal_dog
![戶外山景](./Images/mountain_vista.png)       | outdoor_mountain
![視覺分析食物 (麵包)](./Images/bread.png)       | food_bread

## <a name="identifying-image-types"></a>識別影像類型
有數個方法可以將影像分類。 電腦視覺 API 可以設定布林值旗標，以指出影像是黑白還是彩色的。 它也可以設定用來指出影像是否為線圖的旗標。 它也可以指出影像是否為美工圖案，並以 0-3 的等級指出其品質。

### <a name="clip-art-type"></a>美工圖案類型
偵測影像是否為美工圖案。  

值 | 意義
----- | --------------
0     | 非美工圖案
1     | 不明確
2     | 一般美工圖案
3     | good-clip-art

映像|Response
----|----
![視覺分析起司美工圖案](./Images/cheese_clipart.png)|3 良好美工圖案
![視覺分析住家庭院](./Images/house_yard.png)|0 非美工圖案

### <a name="line-drawing-type"></a>線圖類型
偵測影像是否為線圖。

影像|回應
----|----
![視覺分析獅子繪圖](./Images/lion_drawing.png)|True
![視覺分析花朵](./Images/flower.png)|False

### <a name="faces"></a>臉部
偵測圖片中的人臉，並產生臉部座標、臉部的矩形、性別和年齡。 這些視覺功能是為臉部產生的中繼資料所含的子集。 如需更多為臉部產生的中繼資料 (臉部識別、姿勢偵測等等)，請使用臉部 API。  

影像|回應
----|----
![視覺分析屋頂女人的臉部](./Images/woman_roof_face.png) | [ { "age": 23, "gender": "Female", "faceRectangle": { "left": 1379, "top": 320, "width": 310, "height": 310 } } ]
![視覺分析母女的臉部](./Images/mom_daughter_face.png) | [ { "age": 28, "gender": "Female", "faceRectangle": { "left": 447, "top": 195, "width": 162, "height": 162 } }, { "age": 10, "gender": "Male", "faceRectangle": { "left": 355, "top": 87, "width": 143, "height": 143 } } ]
![視覺分析家人合照的臉部](./Images/family_photo_face.png) | [ { "age": 11, "gender": "Male", "faceRectangle": { "left": 113, "top": 314, "width": 222, "height": 222 } }, { "age": 11, "gender": "Female", "faceRectangle": { "left": 1200, "top": 632, "width": 215, "height": 215 } }, { "age": 41, "gender": "Male", "faceRectangle": { "left": 514, "top": 223, "width": 205, "height": 205 } }, { "age": 37, "gender": "Female", "faceRectangle": { "left": 1008, "top": 277, "width": 201, "height": 201 } } ]


## <a name="domain-specific-content"></a>特定領域內容

除了標記與最上層分類以外，電腦視覺 API 也支援特殊 (或特定領域) 資訊。 特殊資訊可作為獨立的方法來實作，或透過高階分類實作。 它可作為透過特定領域模型的新增而進一步細分 86 類別分類的機制。

目前支援的特殊資訊只有名人辨識和地標辨識。 這些地標是人員和人員群組類別以及世界各地地標的特定領域細分。

使用特定領域模型時可採用兩種選項：

### <a name="option-one---scoped-analysis"></a>選項一 - 特定範圍分析
藉由叫用 HTTP POST 呼叫，僅分析選擇的模型。 使用此選項時，如果您知道您想要使用哪一個模型，您可以指定模型的名稱，而您將只會取得該模型的相關資訊。 例如，您可以使用此選項，針對名人辨識進行搜尋。 其回應會包含可能符合的名人清單，及其信賴分數。

### <a name="option-two---enhanced-analysis"></a>選項二 - 強化分析
進行分析，並提供與 86 類別分類中的類別有關的其他詳細資料。 此選項可運用在使用者除了一或多個特定領域模型中的詳細資料以外，還想要取得一般影像分析的應用程式中。 叫用此方法時，必須先呼叫 86 類別分類器。 如果有任何類別符合已知/比對模型的類別，則會繼續進行第二輪的分類器叫用。 例如，如果「詳細資料=全部」或「詳細資料」包含「名人」，則此方法會呼叫 86 類別分類器之後呼叫名人分類器。 其結果會包含開頭為 'people_' 的標記。

## <a name="generating-descriptions"></a>產生描述 
電腦視覺 API 的演算法會分析影像中的內容。 此分析會形成「描述」的基礎，讓此描述顯示為完整句子中可供閱讀的語言。 描述會概略說明影像中的項目。 電腦視覺 API 的演算法會根據在影像中識別出來的物件產生各種描述。 這些描述會個別受到評估，並產生信賴分數。 接著會傳回一份清單，並依照信賴分數由高至低排序。 您可以在[這裡](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption)取得使用此技術產生影像標題的 Bot 範例。  

### <a name="example-description-generation"></a>範例描述產生
![B&W 建築](./Images/bw_buildings.png) '
```json
 Returned Json

'description':{
   "captions":[
      {
         "type":"phrase",
         'text':'a black and white photo of a large city',
         'confidence':0.607638706850331
      }
   ]   
   "captions":[
      {
         "type":"phrase",
         'text':'a photo of a large city',
         'confidence':0.577256764264197
      }
   ]   
   "captions":[
      {
         "type":"phrase",
         'text':'a black and white photo of a city',
         'confidence':0.538493271791207
      }
   ]   
   'description':[
      "tags":{
         "outdoor",
         "city",
         "building",
         "photo",
         "large",
      }
   ]
}
```

## <a name="perceiving-color-schemes"></a>察覺色彩配置
電腦視覺演算法會從影像中擷取色彩。 色彩會以三種不同的內容進行分析：前景、背景及整體。 色彩會分組為 12 種主要輔色。 這些輔色是黑色、藍色、棕色、灰色、綠色、橙色、粉紅色、紫色、紅色、藍綠色、白色和黃色。 根據影像中的色彩，簡單的黑白或輔色可用十六進位色彩代碼傳回。

影像                                                       | 前景 |背景| 色彩
----------------------------------------------------------- | --------- | ------- | ------
![戶外山景](./Images/mountain_vista.png)            | 黑色     | 黑色   | 白色
![視覺分析花朵](./Images/flower.png)               | 黑色     | 白色   | 白色、黑色、綠色
![視覺分析火車站](./Images/train_station.png) | 黑色     | 黑色   | 黑色

### <a name="accent-color"></a>輔色
從影像中擷取的色彩，用以透過主色和飽和度的搭配向使用者呈現最鮮明的色彩。

影像                                                       | 回應
----------------------------------------------------------- | ----
![戶外山景](./Images/mountain_vista.png)            | #BC6F0F
![視覺分析花朵](./Images/flower.png)               | #CAA501
![視覺分析火車站](./Images/train_station.png) | #484B83


### <a name="black--white"></a>黑白
指出影像是否為黑白的布林值旗標。

影像                                                      | 回應
---------------------------------------------------------- | ----
![視覺分析建築](./Images/bw_buildings.png)      | True
![視覺分析住家庭院](./Images/house_yard.png)      | False

## <a name="flagging-adult-content"></a>標示成人內容
各種不同的視覺類別中包含成人和猥褻群組，可用來偵測成人內容及限制包含色情內容的影像顯示。 成人和猥褻內容偵測的篩選條件可用滑動標尺來設定，以配合使用者的喜好設定。

## <a name="optical-character-recognition-ocr"></a>光學字元辨識 (OCR)
OCR 技術可偵測影像中的文字內容，並將識別出來的文字詞擷取到電腦可讀取的字元資料流中。 您可以將其結果用於搜尋和其他多種用途上，像是醫療記錄、保全和銀行業務等。 它會自動偵測語言。 OCR 讓使用者只需拍攝文字相片而不必謄寫文字，省時又方便。

OCR 支援 25 種語言。 這些語言是：阿拉伯文、簡體中文、繁體中文、捷克文、丹麥文、荷蘭文、英文、芬蘭文、法文、德文、希臘文、匈牙利文、義大利文、日文、韓文、挪威文、波蘭文、葡萄牙文、羅馬尼亞文、俄文、塞爾維亞文 (斯拉夫文和英文)、斯洛伐克文、西班牙文、瑞典文和土耳其文。

如有需要，OCR 會沿著水平影像軸以度為單位校正已辨識文字的旋轉角度。 OCR 會提供每個字的框架座標，如下圖所示。

![OCR 概觀](./Images/vision-overview-ocr.png)OCR 的需求：
- 輸入影像的大小必須介於 40 x 40 與 3200 x 3200 像素之間。
- 影像不可大於 1000 萬像素。

輸入影像可以旋轉 90 度的任何倍數，並可輔以不超過 40 度的小角度旋轉。

文字辨識的準確性取決於影像品質。 在下列情況下可能會導致不正確的讀取：
- 影像模糊。
- 手寫或草寫的文字。
- 藝術字型樣式。
- 文字太小。
- 複雜的背景、陰影、文字反光或透視失真。
- 文字開頭處的大寫字母過大或遺漏
- 文字加上了下標、上標或刪除線。

限制：在主要由文字構成的相片上，誤判可能會來自於部分辨識的文字。 在某些相片 (特別是沒有任何文字的相片) 上，精確度可能會隨著影像的類型而有極大差異。

## <a name="recognize-handwritten-text"></a>辨識手寫文字
此技術可讓您偵測筆記、信件、文章、白板、表格等的手寫文字並加以擷取，並且可在不同的介面及背景中運作，例如技術白皮書、黃色自黏便箋及白板。

您可透過手寫文字辨識擷取文字的影像而無須謄寫，進而節省時間及心力並更具生產力。 記事因此而得以數位化。 此一數位化可讓您實作快速且輕鬆的搜尋。 同時也可減少紙張用量。

輸入需求：
- 支援的影像格式：JPEG、PNG 和 BMP。
- 影像檔案大小必須小於 4 MB。
- 影像尺寸必須介於 40 x 40 到 3200 x 3200 之間。

附註：此技術目前為預覽狀態，且只適用於英文文字。

## <a name="generating-thumbnails"></a>產生縮圖
縮圖是完整大小影像的小型表示法。 手機、平板電腦和 PC 等不同裝置需要不同的使用者體驗 (UX) 配置和縮圖大小。 此電腦視覺 API 功能使用智慧裁剪，因此有助於解決此問題。

上傳影像之後，即可產生高品質的縮圖，且電腦視覺 API 演算法會分析影像內的物件。 接著，它會裁剪影像以符合「相關區域」(ROI) 的需求。 輸出會顯示在特殊架構中，如下圖所示。 產生的縮圖可以使用與原始影像的外觀比例不同的外觀比例來呈現，以因應使用者的需求。

縮圖演算法的運作方式如下：

1. 從影像中移除雜亂的元素，並辨識主要物件「相關區域」(ROI)。
2. 根據已識別的相關區域裁剪影像。
3. 變更外觀比例以符合目標縮圖尺寸。

![縮圖](./Images/thumbnail-demo.png)
