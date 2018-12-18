---
title: 使用 Microsoft Translator Text API 傳回前 N 名最佳翻譯 | Microsoft Docs
description: 使用 Microsoft Translator Text API 傳回前 N 名最佳翻譯。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.component: translator-text
ms.topic: article
ms.date: 12/14/2017
ms.author: v-jansko
ms.openlocfilehash: 3eafe50f69ae1a6748342e64a414ecee4467d0d1
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368654"
---
# <a name="how-to-return-n-best-translations"></a>如何傳回前 N 名最佳翻譯

> [!NOTE]
> 此方法已被取代。 Translator Text API V3.0 中不提供此功能。

Microsoft Translator API 的 GetTranslations() 和 GetTranslationsArray() 方法包含選擇性的布林值旗標 "IncludeMultipleMTAlternatives"。
方法最多會傳回 maxTranslations 個替代項目，其中會從翻譯工具引擎的前 N 名最佳清單提供差異。

簽章為：

**語法**

| C# |
|:---|
| GetTranslationsResponse Microsoft.Translator.GetTranslations(appId, text, from, to, maxTranslations, options); |

**參數**

| 參數 | 說明 |
|:---|:---|
| appId | **必要** 如果使用 Authorization 標頭，請將 appid 欄位保留空白，否則請指定一個包含 "Bearer" + " " + 存取權杖的字串。|
| text | **必要** 一個代表所要翻譯文字的字串。 文字大小不得超過 10000 個字元。|
| from | **必要** 一個代表所要翻要文字之語言代碼的字串。 |
| to | **必要** 一個代表文字之目標翻譯語言代碼的字串。 |
| maxTranslations | **必要** 一個代表要傳回之翻譯數目上限的整數。 |
| options | **選擇性** 一個包含下列值的 TranslateOptions 物件。 它們都是選擇性項目，且預設為最常見設定。

* Category：唯一支援且為預設值的選項是 "general"。
* ContentType：唯一支援且為預設值的選項是 "text/plain"。
* State：協助將要求與回應相互關聯的使用者狀態。 將在回應中傳回相同的內容。
* IncludeMultipleMTAlternatives：用以決定是否要從 MT 引擎傳回多個替代項目的旗標。 預設值為 false，因此只會包含 1 個替代項目。

## <a name="ratings"></a>評等
評分套用如下：最佳自動翻譯的評分為 5。
自動產生之 (前 N 名最佳) 翻譯替代項目的評分為 0，而相符程度則為 100。

## <a name="number-of-alternatives"></a>替代項目數目
傳回的替代項目數目最多為 maxTranslations 個，但可能更少。

## <a name="language-pairs"></a>語言組
此功能不適用於簡體中文與繁體中文之間的翻譯 (雙向)。 這適用於所有其他 Microsoft Translator 支援的語言組。
