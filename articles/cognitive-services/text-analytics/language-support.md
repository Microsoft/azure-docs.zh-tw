---
title: 語言支援 - 文字分析 API
titleSuffix: Azure Cognitive Services
description: 文字分析 API 支援的自然語言清單。 本文說明每項作業支援的語言：情感分析、關鍵片語擷取和語言偵測。
services: cognitive-services
author: ashmaka
manager: cgronlun
ms.service: cognitive-services
ms.technology: text-analytics
ms.topic: article
ms.date: 09/25/2018
ms.author: ashmaka
ms.openlocfilehash: e9f466ac6bce98a6a9f2d79a443c9602ca40bb26
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47435629"
---
# <a name="language-and-region-support-for-the-text-analytics-api"></a>文字分析 API 支援的語言和區域

本文說明每項作業支援的語言：情感分析、關鍵片語擷取和語言偵測。

## <a name="language-detection"></a>語言偵測

文字分析 API 最多可偵測 120 種不同的語言。 語言偵測會傳回語言的「指令碼」。 例如，針對片語 "I have a dog"，它會傳回 `en` 而不是 `en-US`。 唯一的特殊案例是中文，若它可根據提供的文字決定指令碼，語言偵測功能會傳回 `zh_CHS` 或 `zh_CHT`。 若無法識別中文文件的特定指令碼，則只會傳回 `zh`。

## <a name="sentiment-analysis-key-phrase-extraction-and-entity-linking"></a>情感分析、關鍵片語擷取和實體連結

情感分析、關鍵片語擷取和實體連結的支援語言清單更具選擇性，因為分析器會進一步調整以配合其他語言的語言規則。

## <a name="language-list-and-status"></a>語言清單和狀態

語言支援一開始推出時處於預覽階段，之後會進入正式運作 (GA) 狀態，彼此且與整體文字分析服務互不影響。 即使文字分析 API 轉換成正式運作，也可以保留預覽階段中的語言。

| 語言    | 語言代碼 | 情感 | 主要片語 | 實體連結 |   注意  |
|:----------- |:-------------:|:---------:|:-----------:|:-----------:|:-----------:
| 丹麥文      | `da`          | ✔ \*     | ✔           |             |     |
| 荷蘭文       | `nl`          | ✔ \*     | ✔          |             |     |
| English     | `en`          | ✔        | ✔           |  ✔ \*   |      |
| 芬蘭文     | `fi`          | ✔ \*     | ✔           |             |     |
| 法文      | `fr`          | ✔        | ✔           |             |     |
| 德文      | `de`          | ✔ \*     | ✔           |            |     |
| 希臘文       | `el`          | ✔ \*     |             |            |     |
| 義大利文     | `it`          | ✔ \*     | ✔           |             |     |
| 日文    | `ja`          |          | ✔           |            |     |
| 韓文      | `ko`          |          | ✔           |            |     |
| 挪威文 (巴克摩) | `no`          | ✔ \*     |  ✔          |             |     |
| 波蘭文      | `pl`          | ✔ \*     |  ✔          |             |     |
| 葡萄牙文 (葡萄牙) | `pt-PT`| ✔        |  ✔          |       |也接受 `pt`|
| 葡萄牙文 (巴西)   | `pt-BR`|          |  ✔   |         |     |
| 俄文     | `ru`          | ✔ \*     | ✔           |             |     |
| 西班牙文     | `es`          | ✔        | ✔           |     |     |
| 瑞典文     | `sv`          | ✔ \*     | ✔           |             |     |
| 土耳其文     | `tr`          | ✔ \*     |             |             |     |

\* 表示預覽階段的語言支援

## <a name="see-also"></a>另請參閱

[認知服務文件頁面](https://docs.microsoft.com/azure/cognitive-services/)   
[認知服務產品頁面](https://azure.microsoft.com/services/cognitive-services/)
