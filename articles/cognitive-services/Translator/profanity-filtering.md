---
title: 使用 Microsoft Translator Text API 來篩選粗話 | Microsoft Docs
description: 使用 Microsoft Translator Text API 中的粗話篩選功能。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.component: translator-text
ms.topic: article
ms.date: 12/14/2017
ms.author: v-jansko
ms.openlocfilehash: a7172e1e8aa336c011fb06e93fc5c4b54d26a3cd
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368647"
---
# <a name="how-to-add-profanity-filtering-with-the-microsoft-translator-text-api"></a>如何使用 Microsoft Translator Text API 來新增粗話篩選功能

Translator 服務通常會在翻譯中保留存在於來源中的粗話。 粗話程度和導致單字不雅的內容在文化特性之間各異。 因此，粗話程度在目標語言中可能增強，也可能減弱。

如果您想要避免在翻譯中看到粗話 (不論來源文字中是否有粗話)，便可在 Translate() 方法中使用粗話篩選選項。 此選項可讓您選擇是要看到粗話被刪除、以適當的標籤標記起來，還是不採取任何動作。

Translate() 方法會接受 “options” 參數，其中包含了新的元素 “ProfanityAction”。 可接受的 ProfanityAction 值包括 “NoAction”、“Marked” 及 “Deleted”。

## <a name="accepted-values-of-profanityaction-and-examples"></a>可接受的 ProfanityAction 值和範例
|ProfanityAction 值 | 動作 | 範例：來源 - 日文 | 範例：目標 - 英文|
| :---|:---|:---|:---|
| NoAction | 預設值。 與未設定此選項時相同。 粗話會從來源傳遞到目標。 | 彼は変態です。 | He is a jerk. |
| Marked | 不雅的單字會括上 XML 標籤 \<profanity> … \</profanity>。 | 彼は変態です。 | He is a \<profanity>jerk\</profanity>. |
| Deleted | 將會從輸出中移除不雅單字，而不予以取代。 | 彼は。 | He is a. |

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用 Translator API 呼叫來套用粗話篩選功能](reference/v3-0-translate.md)

