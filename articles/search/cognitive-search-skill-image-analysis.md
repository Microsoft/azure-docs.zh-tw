---
title: 映像分析認知搜尋技能 (Azure 搜尋服務) | Microsoft Docs
description: 在 Azure 搜尋服務擴充管線中，使用 ImageAnalysis 認知技能，透過映像分析擷取語意文字。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: luisca
ms.openlocfilehash: 8ec3b6c5dfdd63de45e287cf0b68e90c7b0cbbd8
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48829541"
---
#   <a name="image-analysis-cognitive-skill"></a>映像分析認知技能

**映像分析**技能可根據映像內容擷取一組豐富的視覺功能。 例如，您可以從映像產生標題、產生標記，或識別名人和地標。

> [!NOTE]
> 認知搜尋目前為公開預覽狀態。 技能集執行、映像擷取及正規化目前為免費提供。 我們將在不久後宣告這些功能的定價。 

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Vision.ImageAnalysisSkill 

## <a name="skill-parameters"></a>技能參數

這些參數會區分大小寫。

| 參數名稱     | 說明 |
|--------------------|-------------|
| defaultLanguageCode   |  此字串表示要傳回的語言。 服務會以指定語言傳回辨識結果。 如果未指定此屬性，則預設值為「en」。 <br/><br/>支援的語言為： <br/>*en* - 英文 (預設) <br/> *zh* - 簡體中文|
|visualFeatures |   此字串陣列表示要傳回的視覺功能類型。 有效的視覺功能類型包括：  <ul><li> *類別* - 根據認知服務[說明文件](https://docs.microsoft.com/azure/cognitive-services/computer-vision/category-taxonomy)中定義的分類法，來分類映像內容。</li><li> *標記* - 使用與映像內容相關之字組的詳細清單標記映像。</li><li>*說明* - 用完整的英文句子說明映像內容。</li><li>*臉部* - 偵測臉部是否存在。 如果有的話，會產生座標、性別和年齡。</li><li> *ImageType* - 偵測映像是美工圖案或線條繪圖。</li><li>   *色彩* - 決定輔色、主要色彩，以及映像是否為黑白。</li><li>*成人* - 偵測映像是否涉及色情 (描繪裸露或性行為)。 也會偵測出性暗示內容。</li></ul> 視覺功能的名稱會區分大小寫。|
| 詳細資料   | 字串陣列表示要傳回的特定領域詳細資料。 有效的視覺功能類型包括： <ul><li>*名人* - 識別映像中偵測到的名人。</li><li>*地標* - 識別映像中偵測到的地標。</li></ul>
 |

## <a name="skill-inputs"></a>技能輸入

| 輸入名稱      | 說明                                          |
|---------------|------------------------------------------------------|
| 映像         | 複雜類型。 目前僅適用於當 ```imageAction``` 設為 ```generateNormalizedImages``` 時，由 Azure Blob 索引子產生的「/document/normalized_images」欄位。 如需詳細資訊，請參閱[範例](#sample-output)。|



##  <a name="sample-definition"></a>範例定義
```json
{
    "@odata.type": "#Microsoft.Skills.Vision.ImageAnalysisSkill",
    "context": "/document/normalized_images/*",
    "visualFeatures": [
        "Tags",
        "Faces",
        "Categories",
        "Adult",
        "Description",
        "ImageType",
        "Color"
    ],
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "image",
            "source": "/document/normalized_images/*"
        }
    ],
    "outputs": [
        {
            "name": "categories",
            "targetName": "myCategories"
        },
        {
            "name": "tags",
            "targetName": "myTags"
        },
        {
            "name": "description",
            "targetName": "myDescription"
        },
        {
            "name": "faces",
            "targetName": "myFaces"
        },
        {
            "name": "imageType",
            "targetName": "myImageType"
        },
        {
            "name": "color",
            "targetName": "myColor"
        },
        {
            "name": "adult",
            "targetName": "myAdultCategory"
        }
    ]
}
```

##  <a name="sample-input"></a>範例輸入

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {                
                "image":  {
                               "data": "BASE64 ENCODED STRING OF A JPEG IMAGE",
                               "width": 500,
                               "height": 300,
                               "originalWidth": 5000,  
                               "originalHeight": 3000,
                               "rotationFromOriginal": 90,
                               "contentOffset": 500  
                           }
            }
        }
    ]
}
```


##  <a name="sample-output"></a>範例輸出

```json
{
    "values": [
      {
        "recordId": "1",
            "data": {
                "categories": [
           {
                        "name": "abstract_",
                        "score": 0.00390625
                    },
                    {
                "name": "people_",
                        "score": 0.83984375,
                "detail": {
                            "celebrities": [
                                {
                                    "name": "Satya Nadella",
                                    "faceBoundingBox": {
                                        "left": 597,
                                        "top": 162,
                                        "width": 248,
                                        "height": 248
                                    },
                                    "confidence": 0.999028444
                                }
                            ],
                            "landmarks": [
                                {
                                    "name": "Forbidden City",
                                    "confidence": 0.9978346
                                }
                            ]
                        }
                    }
                ],
                "adult": {
                    "isAdultContent": false,
                    "isRacyContent": false,
                    "adultScore": 0.0934349000453949,
                    "racyScore": 0.068613491952419281
                },
                "tags": [
                    {
                        "name": "person",
                        "confidence": 0.98979085683822632
                    },
                    {
                        "name": "man",
                        "confidence": 0.94493889808654785
                    },
                    {
                        "name": "outdoor",
                        "confidence": 0.938492476940155
                    },
                    {
                        "name": "window",
                        "confidence": 0.89513939619064331
                    }
                ],
                "description": {
                    "tags": [
                        "person",
                        "man",
                        "outdoor",
                        "window",
                        "glasses"
                    ],
                    "captions": [
                        {
                            "text": "Satya Nadella sitting on a bench",
                            "confidence": 0.48293603002174407
                        }
                    ]
                },
                "requestId": "0dbec5ad-a3d3-4f7e-96b4-dfd57efe967d",
                "metadata": {
                    "width": 1500,
                    "height": 1000,
                    "format": "Jpeg"
                },
                "faces": [
                    {
                        "age": 44,
                        "gender": "Male",
                    "faceBoundingBox": {
                            "left": 593,
                            "top": 160,
                            "width": 250,
                            "height": 250
                        }
                    }
                ],
                "color": {
                    "dominantColorForeground": "Brown",
                    "dominantColorBackground": "Brown",
                    "dominantColors": [
                        "Brown",
                        "Black"
                    ],
                    "accentColor": "873B59",
                    "isBwImg": false
                    },
                "imageType": {
                    "clipArtType": 0,
                    "lineDrawingType": 0
                }
           }
      }
    ]
}
```


## <a name="error-cases"></a>錯誤案例
在下列錯誤案例中，不會擷取任何元素。

| 錯誤碼 | 說明 |
|------------|-------------|
| NotSupportedLanguage | 不支援提供的語言。 |
| InvalidImageUrl | 映像 URL 格式不正確或無法存取。|
| InvalidImageFormat | 輸入資料不是有效映像。 |
| InvalidImageSize | 輸入映像太大。 |
| NotSupportedVisualFeature  | 指定的功能類型無效。 |
| NotSupportedImage | 不支援的映像，例如兒童色情內容。 |
| InvalidDetails | 不支援的特定領域模型。 |

## <a name="see-also"></a>另請參閱

+ [預先定義的技能](cognitive-search-predefined-skills.md)
+ [如何定義技能集](cognitive-search-defining-skillset.md) (英文)
+ [建立索引子 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)
