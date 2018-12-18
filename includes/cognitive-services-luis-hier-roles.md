---
title: 包含檔案
description: 包含檔案
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: luis
ms.topic: include
ms.custom: include file
ms.date: 08/02/2018
ms.author: diberry
ms.openlocfilehash: dd99218765c873120c4117a3ec1712fe0a605e66
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44164582"
---
**題**：您應該使用階層式實體，或包含角色的簡單實體模式？ 

**答**：視情況而定。 模式和範例語句相似，因為這兩者均代表使用者在特定意圖下使用的語句。  

若語句沒有清楚的模式，請使用階層式實體。 

|階層式實體|包含角色的簡單實體|
|--|--|
|必須有在意圖中已標記為包含子實體的範例語句|必須有範例語句，**無法在意圖中標記角色**|
|可在模式中使用|**必須**在模式中使用|
|可能需要**更多**該意圖的範例語句|可能需要更少該意圖的範例語句|