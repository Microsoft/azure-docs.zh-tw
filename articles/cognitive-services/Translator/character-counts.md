---
title: Microsoft Translator Text API 字元計數 | Microsoft Docs
description: Microsoft Translator Text API 如何計算字元。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.component: translator-text
ms.topic: article
ms.date: 12/20/2017
ms.author: v-jansko
ms.openlocfilehash: 1b4987509c17e4064d7c54608395e272efa8de3b
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42093855"
---
# <a name="how-the-microsoft-translator-text-api-counts-characters"></a>Microsoft Translator Text API 如何計算字元

Microsoft Translator 會計算每個輸入字元。 這是指 Unicode (而非位元組) 意義上的字元。 Unicode Surrogate 會算成兩個字元。 空白字元和標記也算是字元。 回應的長度無關緊要。

對 Detect 和 BreakSentence 方法的呼叫不會計入字元使用量中。 不過，我們預期在這些對 Detect 和 BreakSentence 方法的呼叫，與其他被計入使用量之函式的使用之間，會有合理的比例。 Microsoft 保留開始計算 Detect 和 BreakSentence 的權利。 

如需有關字元計數的詳細資訊，請參閱 [Microsoft Translator 常見問題集](https://www.microsoft.com/en-us/translator/faq.aspx) \(英文\)。
