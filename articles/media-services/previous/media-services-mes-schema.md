---
title: 媒體編碼器標準結構描述 | Microsoft Docs
description: 本文概述媒體編碼器標準結構描述。
author: Juliako
manager: cfowler
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 4c060062-8ef2-41d9-834e-e81e8eafcf2e
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2017
ms.author: juliako
ms.openlocfilehash: 346d7aecb6a4295f8ceb64bc1b5c6494b7b41bfd
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783887"
---
# <a name="media-encoder-standard-schema"></a>媒體編碼器標準結構描述
本文描述[媒體編碼器標準預設值](media-services-mes-presets-overview.md)所根據之 XML 結構描述的一些元素和類型。 本文提供元素和其有效值的說明。  

## <a name="Preset"></a> 預設值 (根元素)
定義編碼預設值。  

### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **編碼** |[編碼](media-services-mes-schema.md#Encoding) |根元素，表示即將編碼的輸入來源。 |
| **輸出** |[輸出](media-services-mes-schema.md#Output) |想要的輸出檔案集合。 |

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **版本**<br/><br/> 必要 |**xs: decimal** |預設版本。 套用下列限制︰xs:fractionDigits value="1" 和 xs:minInclusive value="1"。例如，**version="1.0"**。 |

## <a name="Encoding"></a> 編碼
包含下列一連串的元素：  

### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **H264Video** |[H264Video](media-services-mes-schema.md#H264Video) |視訊的 H.264 編碼設定。 |
| **AACAudio** |[AACAudio](media-services-mes-schema.md#AACAudio) |音訊的 AAC 編碼設定。 |
| **BmpImage** |[BmpImage](media-services-mes-schema.md#BmpImage) |Bmp 影像的設定。 |
| **PngImage** |[PngImage](media-services-mes-schema.md#PngImage) |Png 影像的設定。 |
| **JpgImage** |[JpgImage](media-services-mes-schema.md#JpgImage) |Jpg 影像的設定。 |

## <a name="H264Video"></a> H264Video
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **TwoPass**<br/><br/> minOccurs="0" |**xs:boolean** |目前，只支援 One-pass 編碼。 |
| **KeyFrameInterval**<br/><br/> minOccurs="0"<br/><br/> **default="00:00:02"** |**xs:time** |決定 IDR 框架之間的固定間距 (以秒為單位)。 也稱為 GOP 持續時間。 若要控制編碼器是否可以脫離此值，請參閱 **SceneChangeDetection**。 |
| **SceneChangeDetection**<br/><br/> minOccurs="0"<br/><br/> 預設值=”false” |**xs: boolean** |如果設為 true，編碼器會嘗試偵測視訊中的場景變更並插入 IDR 畫面格。 |
| **Complexity**<br/><br/> minOccurs="0"<br/><br/> 預設值="Balanced" |**xs:string** |控制編碼速度和視訊品質之間的取捨。 可能是下列值之一︰**Speed**、**Balanced** 或 **Quality**<br/><br/> 預設值：**Balanced** |
| **SyncMode**<br/><br/> minOccurs="0" | |此功能將在未來的版本中公開。 |
| **H264Layers**<br/><br/> minOccurs="0" |[H264Layers](media-services-mes-schema.md#H264Layers) |輸出視訊圖層的集合。 |

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Condition** |**xs:string** | 當輸入沒有視訊時，您可能需要強制編碼器插入單色視訊播放軌。若要這樣做，請使用 Condition="InsertBlackIfNoVideoBottomLayerOnly" (只以最低位元速率插入視訊) 或 Condition="InsertBlackIfNoVideo" (以所有輸出位元速率插入視訊)。 如需詳細資訊，請參閱[本篇文章](media-services-advanced-encoding-with-mes.md#no_video)。|

## <a name="H264Layers"></a> H264Layers

根據預設，如果您傳送僅包含音訊但不含視訊的編碼器輸入，則輸出資產會包含只有音訊資料的檔案。 某些播放器可能無法處理此類型輸出資料流。 您可以在該案例中使用 H264Video 的 **InsertBlackIfNoVideo** 屬性設定來強制編碼器將視訊播放軌新增至輸出。 如需詳細資訊，請參閱[本篇文章](media-services-advanced-encoding-with-mes.md#no_video)。
              
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **H264Layer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[H264Layer](media-services-mes-schema.md#H264Layer) |H264 圖層的集合。 |

## <a name="H264Layer"></a> H264Layer
> [!NOTE]
> 視訊限制是以 [H264 層級](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC#Levels)表格中描述的值為基礎。  
> 
> 

### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **設定檔**<br/><br/> minOccurs="0"<br/><br/> 預設值=”Auto” |**xs:string** |可能是下列其中一個 **xs:string** 值：**Auto**、**Baseline**、**Main**、**High**。 |
| **Level**<br/><br/> minOccurs="0"<br/><br/> 預設值=”Auto” |**xs:string** | |
| **Bitrate**<br/><br/> minOccurs="0" |**xs:int** |用於此視訊圖層的位元速率 (以 kbps 指定)。 |
| **MaxBitrate**<br/><br/> minOccurs="0" |**xs:int** |用於此視訊圖層的最大位元速率 (以 kbps 指定)。 |
| **BufferWindow**<br/><br/> minOccurs="0"<br/><br/> 預設值="00:00:05" |**xs:time** |視訊緩衝區的長度。 |
| **Width**<br/><br/> minOccurs="0" |**xs:int** |輸出視訊畫面格的寬度 (以像素為單位)。<br/><br/> 您目前必須指定 Width 和 Height。 Width 和 Height 都必須是偶數。 |
| **Height**<br/><br/> minOccurs="0" |**xs:int** |輸出視訊畫面格的高度 (以像素為單位)。<br/><br/> 您目前必須指定 Width 和 Height。 Width 和 Height 都必須是偶數。|
| **BFrames**<br/><br/> minOccurs="0" |**xs:int** |參考畫面格之間的 B 畫面格數目。 |
| **ReferenceFrames**<br/><br/> minOccurs="0"<br/><br/> 預設值=”3” |**xs:int** |GOP 中的參考畫面格數目。 |
| **EntropyMode**<br/><br/> minOccurs="0"<br/><br/> 預設值=”Cabac” |**xs:string** |可能是下列其中一個值：**Cabac** 和 **Cavlc**。 |
| **FrameRate**<br/><br/> minOccurs="0" |有理數 |決定輸出視訊的畫面播放速率。 使用預設值 "0/1"，讓編碼器使用與輸入視訊相同的畫面播放速率。 允許的值應該是常見的視訊畫面播放速率。 不過，允許任何有效的有理數。 例如，1/1 會是 1 fps 並且有效。<br/><br/> - 12/1 (12 fps)<br/><br/> - 15/1 (15 fps)<br/><br/> - 24/1 (24 fps)<br/><br/> - 24000/1001 (23.976 fps)<br/><br/> - 25/1 (25 fps)<br/><br/>  - 30/1 (30 fps)<br/><br/> - 30000/1001 (29.97 fps) <br/> <br/>**注意** 如果您要針對多重位元速率編碼建立自訂預設集，則此預設集的所有圖層都**必須**使用相同的 FrameRate 值。|
| **AdaptiveBFrame**<br/><br/> minOccurs="0" |**xs: boolean** |從 Azure 媒體編碼器複製 |
| **Slices**<br/><br/> minOccurs="0"<br/><br/> 預設值="0" |**xs:int** |決定將畫面格分成幾個片段。 建議使用預設值。 |

## <a name="AACAudio"></a> AACAudio
 包含下列元素和群組。  

 如需有關 AAC 的詳細資訊，請參閱 [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding)。  

### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **設定檔**<br/><br/> minOccurs="0 "<br/><br/> 預設值="AACLC" |**xs:string** |可能是下列其中一個值：**AACLC**、**HEAACV1** 或 **HEAACV2**。 |

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Condition** |**xs:string** |若要強制編碼器在輸入不含音訊時產生包含靜音曲目的資產，請指定 "InsertSilenceIfNoAudio" 值。<br/><br/> 依照預設，如果您傳送僅包含視訊不含音訊的輸入到編碼器，輸出資產將包含僅含視訊資料的檔案。 某些播放器可能無法處理此類型輸出資料流。 您可以在該案例中使用此設定來強制編碼器將靜音曲目新增至輸出。 |

### <a name="groups"></a>群組
| 參考 | 說明 |
| --- | --- |
| [AudioGroup](media-services-mes-schema.md#AudioGroup)<br/><br/> minOccurs="0" |請參閱 [AudioGroup](media-services-mes-schema.md#AudioGroup) 的說明，以得知適當的聲道數目、取樣率以及可為每個設定檔設定的位元速率。 |

## <a name="AudioGroup"></a> AudioGroup
如需每個設定檔的有效值詳細資訊，請參閱後面的「音訊轉碼器詳細資料」表格。  

### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Channels**<br/><br/> minOccurs="0" |**xs:int** |編碼的音訊聲道數目。 以下是有效的選項︰1、2、5、6、8。<br/><br/> 預設值︰2。 |
| **SamplingRate**<br/><br/> minOccurs="0" |**xs:int** |音訊取樣率 (以 Hz 指定)。 |
| **Bitrate**<br/><br/> minOccurs="0" |**xs:int** |音訊編碼時使用的位元速率 (以 kbps 指定)。 |

### <a name="audio-codec-details"></a>音訊轉碼器詳細資訊
音訊轉碼器|詳細資料  
-----------------|---  
**AACLC**|1:<br/><br/> - 11025 : 8 &lt;= 位元速率 &lt; 16<br/><br/> - 12000 : 8 &lt;= 位元速率 &lt; 16<br/><br/> - 16000 : 8 &lt;= 位元速率 &lt;32<br/><br/>- 22050 : 24 &lt;= 位元速率 &lt; 32<br/><br/> - 24000 : 24 &lt;= 位元速率 &lt; 32<br/><br/> - 32000 : 32 &lt;= 位元速率 &lt;= 192<br/><br/> - 44100 : 56 &lt;= 位元速率 &lt;= 288<br/><br/> - 48000 : 56 &lt;= 位元速率 &lt;= 288<br/><br/> - 88200 : 128 &lt;= 位元速率 &lt;= 288<br/><br/> - 96000 : 128 &lt;= 位元速率 &lt;= 288<br/><br/> 2：<br/><br/> - 11025 : 16 &lt;= 位元速率 &lt; 24<br/><br/> - 12000 : 16 &lt;= 位元速率 &lt; 24<br/><br/> - 16000 : 16 &lt;= 位元速率 &lt; 40<br/><br/> - 22050 : 32 &lt;= 位元速率 &lt; 40<br/><br/> - 24000 : 32 &lt;= 位元速率 &lt; 40<br/><br/> - 32000 :  40 &lt;= 位元速率 &lt;= 384<br/><br/> - 44100 : 96 &lt;= 位元速率 &lt;= 576<br/><br/> - 48000 : 96 &lt;= 位元速率 &lt;= 576<br/><br/> - 88200 : 256 &lt;= 位元速率 &lt;= 576<br/><br/> - 96000 : 256 &lt;= 位元速率 &lt;= 576<br/><br/> 5/6:<br/><br/> - 32000 : 160 &lt;= 位元速率 &lt;= 896<br/><br/> - 44100 : 240 &lt;= 位元速率 &lt;= 1024<br/><br/> - 48000 : 240 &lt;= 位元速率 &lt;= 1024<br/><br/> - 88200 : 640 &lt;= 位元速率 &lt;= 1024<br/><br/> - 96000 : 640 &lt;= 位元速率 &lt;= 1024<br/><br/> 8:<br/><br/> - 32000 : 224 &lt;= 位元速率 &lt;= 1024<br/><br/> - 44100 : 384 &lt;= 位元速率 &lt;= 1024<br/><br/> - 48000 : 384 &lt;= 位元速率 &lt;= 1024<br/><br/> - 88200 : 896 &lt;= 位元速率 &lt;= 1024<br/><br/> - 96000 : 896 &lt;= 位元速率 &lt;= 1024  
**HEAACV1**|1:<br/><br/> - 22050 : 位元速率 = 8<br/><br/> - 24000 : 8 &lt;= 位元速率 &lt;= 10<br/><br/> - 32000 : 12 &lt;= 位元速率 &lt;= 64<br/><br/> - 44100 : 20 &lt;= 位元速率 &lt;= 64<br/><br/> - 48000 : 20 &lt;= 位元速率 &lt;= 64<br/><br/> - 88200 : 位元速率 = 64<br/><br/> 2：<br/><br/> - 32000 : 16 &lt;= 位元速率 &lt;= 128<br/><br/> - 44100 : 16 &lt;= 位元速率 &lt;= 128<br/><br/> - 48000 : 16 &lt;= 位元速率 &lt;= 128<br/><br/> - 88200 : 96 &lt;= 位元速率 &lt;= 128<br/><br/> - 96000 : 96 &lt;= 位元速率 &lt;= 128<br/><br/> 5/6:<br/><br/> - 32000 : 64 &lt;= 位元速率 &lt;= 320<br/><br/> - 44100 : 64 &lt;= 位元速率 &lt;= 320<br/><br/> - 48000 : 64 &lt;= 位元速率 &lt;= 320<br/><br/> - 88200 : 256 &lt;= 位元速率 &lt;= 320<br/><br/> - 96000 : 256 &lt;= 位元速率 &lt;= 320<br/><br/> 8:<br/><br/> - 32000 : 96 &lt;= 位元速率 &lt;= 448<br/><br/> - 44100 : 96 &lt;= 位元速率 &lt;= 448<br/><br/> - 48000 : 96 &lt;= 位元速率 &lt;= 448<br/><br/> - 88200 : 384 &lt;= 位元速率 &lt;= 448<br/><br/> - 96000 : 384 &lt;= 位元速率 &lt;= 448  
**HEAACV2**|2：<br/><br/> - 22050 : 8 &lt;= 位元速率 &lt;= 10<br/><br/> - 24000 : 8 &lt;= 位元速率 &lt;= 10<br/><br/> - 32000 : 12 &lt;= 位元速率 &lt;= 64<br/><br/> - 44100 : 20 &lt;= 位元速率 &lt;= 64<br/><br/> - 48000 : 20 &lt;= 位元速率 &lt;= 64<br/><br/> - 88200 : 64 &lt;= 位元速率 &lt;= 64  
  

## <a name="Clip"></a> 剪輯
### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **StartTime** |**xs:duration** |指定簡報的開始時間。 StartTime 值必須符合輸入視訊的絕對時間戳記。 例如，如果輸入視訊的第一個畫面有 12:00:10.000 的時間戳記，則 StartTime 至少應該為 12:00:10.000 或以上。 |
| **Duration** |**xs:duration** |指定簡報的持續時間 (例如，視訊中的覆疊外觀)。 |

## <a name="Output"></a> 輸出
### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **FileName** |**xs:string** |輸出檔案的名稱。<br/><br/> 您可以使用下表中所述的巨集來建置輸出檔案名稱。 例如︰<br/><br/> **"Outputs": [      {       "FileName": "{Basename}*{Resolution}*{Bitrate}.mp4",       "Format": {         "Type": "MP4Format"       }     }   ]** |

### <a name="macros"></a>巨集
| 巨集 | 說明 |
| --- | --- |
| **{Basename}** |如果您正在進行 VoD 編碼，{Basename} 是輸入資產中主要檔案之 AssetFile.Name 屬性的前 32 個字元。<br/><br/> 如果輸入資產是即時封存，則 {Basename} 衍生自伺服器資訊清單中的 trackName 屬性。 如果您使用 TopBitrate 提交子片段作業 (例如：“<VideoStream\>TopBitrate</VideoStream\>”)，而且輸出檔案包含視訊，則 {Basename} 是最高位元速率之視訊層的 trackName 的前 32 個字元。<br/><br/> 如果您使用所有輸入位元速度提交子片段作業 (例如：“<VideoStream\>*</VideoStream\>”)，而且輸出檔案包含視訊，則 {Basename} 是對應視訊層之 trackName 的前 32 個字元。 |
| **{Codec}** |對應至視訊的 “H264” 和音訊的 “AAC”。 |
| **{Bitrate}** |目標視訊位元速率 (如果輸出檔包含視訊和音訊)，或目標音訊位元速率 (如果輸出檔只包含音訊)。 使用的值是以 kbps 為單位的位元速率。 |
| **{Channel}** |如果檔案包含音訊，則為音訊聲道計數。 |
| **{Width}** |如果輸出檔包含視訊，則為檔案中視訊的寬度 (以像素為單位)。 |
| **{Height}** |如果輸出檔包含視訊，則為檔案中視訊的高度 (以像素為單位)。 |
| **{Extension}** |繼承自輸出檔的 “Type” 屬性。 輸出檔案名稱的副檔名會是其中一個：“mp4”、“ts”、“jpg”、“png” 或 “bmp”。 |
| **{Index}** |縮圖的必要項。 應該只會出現一次。 |

## <a name="Video"></a> 視訊 (複雜類型繼承自轉碼器)
### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **啟動** |**xs:string** | |
| **Step** |**xs:string** | |
| **Range** |**xs:string** | |
| **PreserveResolutionAfterRotation** |**xs:boolean** |如需詳細說明，請參閱下一節︰[PreserveResolutionAfterRotation](media-services-mes-schema.md#PreserveResolutionAfterRotation) |

### <a name="PreserveResolutionAfterRotation"></a> PreserveResolutionAfterRotation
建議使用 **PreserveResolutionAfterRotation** 旗標搭配以百分比表示的解析度值 (Width="100%"、Height="100%")。  

根據預設，媒體編碼器標準 (MES) 預設值的編碼解析度設定 (Width、Height) 是以旋轉 0 度的視訊為目標。 例如，如果輸入視訊是 1280x720 且旋轉零度，則預設值可確保輸出具有相同的解析度。  

![MESRoation1](./media/media-services-shemas/media-services-mes-roation1.png) 

如果已使用非零旋轉 (例如，垂直握住的智慧型手機或平板電腦) 方式擷取輸入視訊，則 MES 預設會將編碼解析度設定 (Width、Height) 套用到輸入視訊，然後彌補旋轉。 如需範例，請參閱下圖。 預設使用 Width = “100%”、Height = “100%”，MES 將其解譯為要求輸出為 1280 像素寬、720 像素高。 旋轉視訊之後，接著會縮小圖片以放入該視窗中，並導向左右兩邊的黑邊 (pillar-box) 區域。  

![MESRoation2](./media/media-services-shemas/media-services-mes-roation2.png) 

或者，您可以利用 **PreserveResolutionAfterRotation** 旗標，並將它設定為 "true" (預設值為 "false")。 因此，如果預設值為 Width = "100%"、Height = "100%"，且 PreserveResolutionAfterRotation 設定為 "true"，則 1280 像素寬、720 像素高且旋轉 90 度的輸入視訊會產生旋轉零度的輸出，但為 720 像素寬和 1280 像素高。 請參閱下圖︰  

![MESRoation3](./media/media-services-shemas/media-services-mes-roation3.png) 

## <a name="FormatGroup"></a> FormatGroup (群組)
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **BmpFormat** |**BmpFormat** | |
| **PngFormat** |**PngFormat** | |
| **JpgFormat** |**JpgFormat** | |

## <a name="BmpLayer"></a> BmpLayer
### <a name="element"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="PngLayer"></a> PngLayer
### <a name="element"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="JpgLayer"></a> JpgLayer
### <a name="element"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |
| **Quality**<br/><br/> minOccurs="0" |**xs:int** |有效值：1 (最差) - 100 (最佳) |

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="PngLayers"></a> PngLayers
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **PngLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[PngLayer](media-services-mes-schema.md#PngLayer) | |

## <a name="BmpLayers"></a> BmpLayers
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **BmpLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[BmpLayer](media-services-mes-schema.md#BmpLayer) | |

## <a name="JpgLayers"></a> JpgLayers
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **JpgLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[JpgLayer](media-services-mes-schema.md#JpgLayer) | |

## <a name="BmpImage"></a> BmpImage (複雜類型繼承自視訊)
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Png 圖層 |

## <a name="JpgImage"></a> JpgImage (複雜類型繼承自視訊)
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Png 圖層 |

## <a name="PngImage"></a> PngImage (複雜類型繼承自視訊)
### <a name="elements"></a>元素
| Name | 類型 | 說明 |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Png 圖層 |

## <a name="examples"></a>範例
如需根據此結構描述建置的 XML 預設值範例，請參閱 [MES (媒體編碼器標準) 的工作預設值](media-services-mes-presets-overview.md)。

## <a name="next-steps"></a>後續步驟
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

