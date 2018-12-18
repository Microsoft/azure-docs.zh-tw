---
title: 搭配 Azure 上的自訂語音服務使用自訂發音 | Microsoft Docs
description: 了解如何在認知服務中使用自訂語音服務建立語言模型。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 11/23/2017
ms.author: panosper
ms.openlocfilehash: 1e9de9755ef4baee7e2a0e6c093e2cf27cb4281f
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339809"
---
# <a name="enable-custom-pronunciation"></a>啟用自訂發音

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

自訂發音可讓使用者定義語音形式和顯示字組或字詞。 它可用於處理自訂的字詞，如產品名稱或縮略字。 您只需要有發音檔 (簡單的 .txt 檔)。

其運作方式如下。 您可以在單一 .txt 檔內輸入數個自訂的發音項目。 其結構如下所示：

```
Display form <Tab> Spoken form <Newline>
```

*範例*：

| 顯示形式 | 口語形式 |
|----------|-------|
| C3PO | see three pea o |
| L8R | late are |
| CNTK | see n tea k|

## <a name="requirements-for-the-spoken-form"></a>口語形式的需求
口語形式必須是小寫，您可以在匯入期間強制執行此規定。 此外，您必須在資料匯入工具內提供檢查。 口語形式和顯示形式中都不允許定位點。 不過，顯示形式中可以有更多受到禁止的字元 (例如，~ 和 ^)。

每個 .txt 檔都可以有數個項目。 例如，請看下列螢幕擷取畫面：

![有數個縮略字發音項目的記事本螢幕擷取畫面](../../../media/cognitive-services/custom-speech-service/custom-speech-pronunciation-file.png)

口語形式是顯示形式的語音順序。 它是由字母、字組或音節所組成的。 目前沒有任何進一步的指導方針或一組標準可幫助您規劃口語形式。 

## <a name="supported-pronunciation-characters"></a>支援的發音字元
美式英文 (en-US) 和德文 (de-DE) 目前可支援自訂發音。 下表顯示可用來表達字詞口語形式 (在自訂發音檔中) 的字元集： 

| 語言 | 字元 |
|---------- |----------|
| 美式英文 (en-US) | a, b, c, d, e, f, g, h, i, j, k, l, o, p, q, r, s, t, u, v, w, x, y, z |
| 德文 (de-de) | ä, ö, ü, ẞ, a, b, c, d, e, f, g, h, i, j, k, l, o, p, q, r, s, t, u, v, w, x, y, z |

>[注意] 字詞的顯示形式 (在發音檔中) 應該以相同方式撰寫於語言適應資料集。

## <a name="requirements-for-the-display-form"></a>顯示形式的需求
顯示形式只能是自訂字組、字詞、縮略字或結合現有字組的複合字組。 您也可以為常見字組輸入其他發音。 

>[!NOTE]
不建議使用這項功能來重新規劃常見字組或修改口語形式。 最好是執行解碼器來看看某些不常見的字組 (例如縮寫、技術字組和外文字組) 是否未正確解碼。 如果是，請將這些字組新增至自訂發音檔。 在語言模型中，請一律使用且只使用字組的顯示形式。 

## <a name="requirements-for-the-file-size"></a>檔案大小的需求
包含發音項目的 .txt 檔案大小上限為 1 MB。 一般而言，您並不需要透過此檔案上傳大量資料。 大部分的自訂發音檔可能只有幾 KB 的大小。 所有地區設定的 .txt 檔編碼應該是 UTF-8 BOM。 美式英文地區設定也可以接受 ANSI。

## <a name="next-steps"></a>後續步驟
* 藉由建立[自訂原音模型](cognitive-services-custom-speech-create-acoustic-model.md)來提升辨識準確度。
* [建立自訂的語音轉文字端點](cognitive-services-custom-speech-create-endpoint.md)，以供從應用程式使用。
