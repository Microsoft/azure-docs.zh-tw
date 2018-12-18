---
title: Microsoft 翻譯工具文字 API 動態字典 | Microsoft Docs
description: 如何使用 Microsoft 翻譯工具文字 API 的動態字典功能。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.component: translator-text
ms.topic: article
ms.date: 12/14/2017
ms.author: v-jansko
ms.openlocfilehash: dbc754093827730b8709d67f314e5b327518ef50
ms.sourcegitcommit: 17fe5fe119bdd82e011f8235283e599931fa671a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/11/2018
ms.locfileid: "42093821"
---
# <a name="how-to-use-the-dynamic-dictionary-feature-of-the-microsoft-translator-text-api"></a>如何使用 Microsoft 翻譯工具文字 API 的動態字典功能

如果已經知道想要套用至單字或片語的翻譯，則可以在要求內以標記來提供。 動態字典僅適用於專有名稱和產品名稱這類複合名詞。 

**語法：** 

<mstrans:dictionary translation=”translation of phrase”>phrase</mstrans:dictionary>

**範例：en-de:**

輸入來源：The word <mstrans:dictionary translation=\"wordomatic\">word or phrase</mstrans:dictionary> is a dictionary entry.

目標輸出：Das Wort "wordomatic" ist ein Wörterbucheintrag.

不論有無 HTML 模式，此功能都以相同的方式運作。 

應該謹慎使用此功能。 自訂翻譯的適當且最適合方式是使用 Custom Translator。 Custom Translator 會完全利用內容和統計機率。 如果您已有或能夠建立定型資料，以便在上下文中顯示您的單字或片語，則會得到更好的結果。 您可以在 [http://aka.ms/CustomTranslator](http://aka.ms/CustomTranslator) \(英文\) 找到更多 Custom Translator 的詳細資訊。

