---
title: 了解如何在以模式為基礎的實體中使用角色
titleSuffix: Azure Cognitive Services
description: 角色是實體的具名內容子類型，只能使用於模式中。 例如，在 buy a ticket from New York to London 語句中，New York 和 London 都是城市，但是在句子中各有不同的意義。 紐約是出發城市，而輪多是目的地城市。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 033e5e5e054b0a29961ad60e72b1466b51d1df0c
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47035193"
---
# <a name="entity-roles-in-patterns-are-contextual-subtypes"></a>模式中的實體角色為內容子類型
角色是實體的具名內容子類型，只能使用於[模式](luis-concept-patterns.md)中。

例如，在 `buy a ticket from New York to London` 語句中，紐約和倫敦都是城市，但是在句子中各有不同的意義。 紐約是出發城市，而輪多是目的地城市。 

角色會賦予這些差異一個名稱：

|實體|角色|目的|
|--|--|--|
|位置|來源|飛機起飛的位置|
|位置|目的地|飛機降落的位置|

## <a name="how-are-roles-used-in-patterns"></a>如何在模式中使用角色？
在模式的範本語句中，角色會使用於語句內： 

|模式與實體角色|
|--|
|`buy a ticket from {Location:origin} to {Location:destination}`|


## <a name="role-syntax-in-patterns"></a>模式中的角色語法
實體和角色都會以大括號 `{}` 括住。 實體與角色是以冒號隔開。 

## <a name="roles-versus-hierarchical-entities"></a>角色與階層式實體
階層式實體會提供與角色相同的內容資訊，但僅限於**意圖**中的語句。 同樣地，角色會提供與階層式實體相同的內容資訊，但僅限於**模式**。

|內容學習|使用於|
|--|--|
|階層式實體|意圖|
|角色|模式|

## <a name="next-steps"></a>後續步驟

* 了解如何新增[角色](luis-how-to-add-entities.md#add-role-to-pattern-based-entity)
