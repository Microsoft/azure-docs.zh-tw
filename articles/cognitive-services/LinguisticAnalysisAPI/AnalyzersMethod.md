---
title: 語言分析 API 中的分析器方法 | Microsoft Docs
description: 分析器 REST API 提供 Microsoft 認知服務中的服務目前支援的分析器清單。
services: cognitive-services
author: RichardSunMS
manager: wkwok
ms.service: cognitive-services
ms.component: linguistic-analysis
ms.topic: article
ms.date: 06/30/2016
ms.author: lesun
ms.openlocfilehash: 3fc243a0da77c5bae9009929f2b82e1353347752
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367895"
---
# <a name="analyzers-method"></a>分析器方法

**分析器** REST API 提供服務目前支援的分析器清單。
回應包含其[名稱](Analyzer-Names.md)和每個分析器所支援的語言 (例如"en"代表英文)。

## <a name="request-parameters"></a>要求參數
None

<br>

## <a name="response-parameters"></a>回應參數
Name | 類型 | 說明
-----|------|--------------
語言 | 字串的清單 | 可以使用此分析器的兩個字母 ISO 語言代碼清單。
id   | 字串 | 此分析器的唯一識別碼
kind | 字串 | 此處分析器的廣泛類型
規格 | 字串 | 用於此分析器的規格名稱
實作 | 字串 | 此分析器後方的模型和/或演算法描述

<br>
## <a name="example"></a>範例
GET / 分析器

回應：JSON
```json
[
    {
        "id": "22A6B758-420F-4745-8A3C-46835A67C0D2",
        "languages": ["en"],
        "kind": "Constituency_Tree",  
        "specification": "PennTreebank3", 
        "implementation": "SplitMerge"
    }, 
    {
        "id" : "4FA79AF1-F22C-408D-98BB-B7D7AEEF7F04",
        "languages": ["en"],
        "kind": "POS_Tags", 
        "specification": "PennTreebank3", 
        "implementation": "cmm"
    },
    {
        "id" : "08EA174B-BFDB-4E64-987E-602F85DA7F72",
        "languages": ["en"],
        "kind": "Tokens", 
        "specification":"PennTreebank3", 
        "implementation": "regexes"
    } 
]
```
