---
title: 學術知識 API 中的期刊實體屬性 | Microsoft Docs
description: 了解您可以在認知服務的學術知識 API 中搭配期刊實體使用的屬性。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/23/2017
ms.author: alch
ms.openlocfilehash: e782c57b8ac57028e070c16382c53ec76666e94d
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367818"
---
# <a name="journal-entity"></a>期刊實體

<sub> *下列屬性專屬於期刊實體。(Ty = '2') </sub>

Name    |說明                            |類型       | 作業
------- | ------------------------------------- | --------- | ----------------------------
id      |實體識別碼                              |Int64      |Equals
DJN     |期刊標準化名稱                |字串     |None
JN      |期刊顯示名稱                   |字串     |Equals
CC      |期刊引用總數           |Int32      |None  
ECC     |期刊引用預估總數 |Int32      |None