---
title: 使用 Azure 認知服務中的語音服務自訂發音
description: 了解如何使用 Azure 認知服務中的語音服務自訂發音。
services: cognitive-services
author: PanosPeriorellis
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 07/02/2018
ms.author: panosper
ms.openlocfilehash: a608d1e48112fbb2adb56191eeb7f168de507e77
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423199"
---
# <a name="enable-custom-pronunciation"></a>啟用自訂發音
藉由使用自訂發音，您可以定義語音形式和顯示字組或字詞。 它可用於處理自訂的字詞，如產品名稱或縮略字。 您只需要有發音檔 (簡單的 .txt 檔)。

其運作方式如下。 您可以在單一 .txt 檔內輸入數個自訂的發音項目。 其結構如下所示：

```
Display form <Tab> Spoken form <Newline>
```

下表顯示數個範例：

| 顯示形式 | 口語形式 |
|----------|-------|
| C3PO | see three pea o |
| L8R | late are |
| CNTK | see n tea k|

## <a name="requirements-for-the-spoken-form"></a>口語形式的需求
口語形式必須是小寫，您可以在匯入期間強制執行此規定。 您也必須在資料匯入工具中提供檢查。 口語形式和顯示形式中都不允許定位點。 不過，顯示形式中可以有更多受到禁止的字元 (例如，~ 和 ^)。

每個 .txt 檔案都可以有數個項目，如下圖所示：

![縮寫發音的範例](media/stt/custom-speech-pronunciation-file.png)

口語形式是顯示形式的語音順序。 它是由字母、字組或音節所組成的。 目前沒有任何進一步的指導方針或一組標準可幫助您規劃口語形式。 

## <a name="supported-pronunciation-characters"></a>支援的發音字元
美式英文 (en-US) 和德文 (de-DE) 目前可支援自訂發音。 下表顯示可用來表達字詞口語形式 (在自訂發音檔中) 的字元集： 

| 語言 | 字元 |
|---------- |----------|
| 美式英文 (en-US) | a, b, c, d, e, f, g, h, i, j, k, l, o, p, q, r, s, t, u, v, w, x, y, z |
| 德文 (de-de) | ä, ö, ü, ?, a, b, c, d, e, f, g, h, i, j, k, l, o, p, q, r, s, t, u, v, w, x, y, z |

> [!NOTE]
> 字詞的顯示形式 (在發音檔中) 應該以相同方式撰寫於語言適應資料集。

## <a name="requirements-for-the-display-form"></a>顯示形式的需求
顯示形式只能是自訂字組、字詞、縮略字或結合現有字組的複合字組。 您也可以為常見字組輸入其他發音。 

>[!NOTE]
>不建議使用這項功能來重新規劃常見字組或修改口語形式。 最好是執行解碼器來看看某些不常見的字組 (例如縮寫、技術字組或外文字組) 是否未正確解碼。 如果是，請將這些字組新增至自訂發音檔。 在語言模型中，請一律使用且只使用字組的顯示形式。 

## <a name="requirements-for-the-file-size"></a>檔案大小的需求
包含發音項目的 .txt 檔案大小上限為 1 MB。 通常，您並不需要透過此檔案上傳大量資料。 大部分的自訂發音檔可能只有幾 KB 的大小。 所有地區設定的 .txt 檔編碼應該是 UTF-8 BOM。 美式英文地區設定也可以接受 ANSI。

## <a name="next-steps"></a>後續步驟
* 建立[自訂原音模型](how-to-customize-acoustic-models.md)來提升辨識準確度。
* 建立[自訂語言模型](how-to-customize-language-model.md)來提升辨識準確度。
 