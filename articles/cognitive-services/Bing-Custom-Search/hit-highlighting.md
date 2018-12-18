---
title: Bing 自訂搜尋：使用裝飾標記醒目提示文字 | Microsoft Docs
description: 示範如何啟用搜尋回應中的文字裝飾。
services: cognitive-services
author: brapel
manager: ehansen
ms.assetid: 5365B568-EA55-4D97-8FBE-0AF60158D4D5
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: article
ms.date: 09/28/2017
ms.author: v-brapel
ms.openlocfilehash: d2d0070865aa29257ac827bbb4fc313d87ea7282
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367991"
---
# <a name="using-decoration-markers-to-highlight-text"></a>使用裝飾標記醒目提示文字

Bing 支援搜尋結果醒目提示，可在某些回答的顯示字串中標示查詢詞彙 (或 Bing 找到的其他相關詞彙)。 例如，網頁的 `name`、`displayUrl` 和 `snippet` 欄位可標示查詢詞彙。

根據預設，Bing 不會在顯示字串中包含醒目提示標記。 若要加入標記，請在要求中包含 `textDecorations` 查詢參數，並將它設定為 **true**。 Bing 會使用 E000 和 E001 Unicode 字元來標示查詢詞彙，以便標示詞彙的開頭和結尾。 比方說，如果查詢詞彙為 Sailing Dinghy 且欄位中存在任一詞彙，則該詞彙會含括在搜尋結果醒目提示字元內，如下列範例所示︰  
  
![搜尋結果醒目提示](./media/bing-hit-highlighting.PNG) 

在使用者介面中顯示字串之前，您會以您的顯示格式適用的字元來取代 Unicode 字元。 例如，如果您以 HTML 格式顯示文字，您可以用 <b\> 取代 E000 以及用 </b\> 取代 E001。 如果您不想要套用格式設定，請從字串中移除標記。 

Bing 提供以 Unicode 字元或 HTML 標籤作為標記的選項。 若要指定所要使用的標記，請包含 `textFormat` 查詢參數。 若要以 Unicode 字元標示內容，請將 `textFormat` 設定為 Raw (預設值)；若要以 HTML 標籤標示內容，請將 `textFormat` 設定為 HTML。 
  
如果 `textDecorations` 為 **true**，Bing 可能會在回答的顯示字串中包含下列標記。 如果沒有 HTML 對等項目，HTML 資料表儲存格會是空的。

|Unicode|HTML|說明
|-|-|-
|U+E000|\<b>|標示查詢詞彙的開頭 (搜尋結果醒目提示)
|U+E001|\</b>|標示查詢詞彙的結尾
|U+E002|\<i>|標示斜體內容的開頭 
|U+E003|\</i>|標示斜體內容的結尾
|U+E004|\<br/>|標示分行符號
|U+E005||標示電話號碼的開頭
|U+E006||標示電話號碼的結尾
|U+E007||標示地址的開頭
|U+E008||標示地址的結尾
|U+E009|\&nbsp;|標示不分行空格
|U+E00C|\<strong>|標示粗體內容的開頭
|U+E00D|\</strong>|標示粗體內容的結尾
|U+E00E||標示背景應該比其周圍背景淡之內容的開頭
|U+E00F||標示背景應該比其周圍背景淡之內容的結尾
|U+E010||標示背景應該比其周圍背景深之內容的開頭
|U+E011||標示背景應該比其周圍背景深之內容的結尾
|U+E012|\<del>|標示應刪除之內容的開頭
|U+E013|\</del>|標示應刪除之內容的結尾
|U+E016|\<sub>|標示下標內容的開頭
|U+E017|\</sub>|標示下標內容的結尾
|U+E018|\<sup>|標示上標內容的結尾
|U+E019|\</sup>|標示上標內容的開頭

以下範例顯示的 `Computation` 回答包含 log(2) 查詢詞彙的下標標記。 只有在 `textDecoration` 為 **true** 時，`expression` 欄位才會包含標記。

![computation 標記](./media/bing-markers-computation.PNG) 

如果要求並未要求裝飾，運算式會是 log10(2)。 
  
