---
title: Azure CDN 規則引擎條件運算式 | Microsoft Docs
description: Azure CDN 規則引擎比對條件和功能的參考文件。
services: cdn
documentationcenter: ''
author: Lichard
manager: akucer
editor: ''
ms.assetid: 669ef140-a6dd-4b62-9b9d-3f375a14215e
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: rli
ms.openlocfilehash: 57e56c38e003cb83dcf44f455c4451d159db8a59
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22997973"
---
# <a name="azure-cdn-rules-engine-conditional-expressions"></a>Azure CDN 規則引擎條件運算式
本主題會針對 Azure 內容傳遞網路 (CDN) [規則引擎](cdn-rules-engine.md)的條件運算式列出詳細說明。

規則的第一個部分是條件運算式。

條件運算式 | 說明
-----------------------|-------------
IF | IF 運算式永遠是規則中第一個陳述式的一部分。 像所有其他條件運算式一樣，此 IF 陳述式必須與符合項目相關聯。 如果未定義其他條件運算式，則此相符項目會決定在將一組功能套用至要求之前必須符合的準則。
AND IF | AND IF 運算式只能在下列類型的條件運算式之後新增︰IF、AND IF。 它表示針對初始 IF 陳述式有必須符合的另一個條件。
ELSE IF| ELSE IF 運算式會指定其他條件，必須在此 ELSE IF 陳述式特定的一組功能發生之前符合。 有 ELSE IF 陳述式表示前一個陳述式的結尾。 可以放在 ELSE IF 陳述式之後的條件運算式是另一個 ELSE IF 陳述式。 這表示 ELSE IF 陳述式只能用來指定必須符合的單一其他條件。

**範例**：![CDN 相符條件](./media/cdn-rules-engine-reference/cdn-rules-engine-conditional-expression.png)

 > [!TIP]
   > 後一項規則可能會覆寫前一項規則所指定的動作。 範例︰全面涵蓋規則會保護所有透過以權杖為基礎的驗證要求。 可以直接在它底下建立另一個規則，對特定類型的要求產生例外狀況。

### <a name="next-steps"></a>後續步驟
* [Azure CDN 概觀](cdn-overview.md)
* [規則引擎參考 (英文)](cdn-rules-engine-reference.md)
* [規則引擎比對條件](cdn-rules-engine-reference-match-conditions.md)
* [規則引擎功能](cdn-rules-engine-reference-features.md)
* [使用規則引擎覆寫預設的 HTTP 行為](cdn-rules-engine.md)
