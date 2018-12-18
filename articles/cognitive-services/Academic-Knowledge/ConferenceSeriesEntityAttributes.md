---
title: 學術知識 API 中的會議系列實體屬性 | Microsoft Docs
description: 了解您可以在認知服務中搭配會議系列實體使用的屬性。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/23/2017
ms.author: alch
ms.openlocfilehash: 332736c927bdaa00334546f626a6eabb8e11d3b5
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367830"
---
# <a name="conference-series-entity"></a>會議系列實體

<sub> *下列屬性專屬於會議系列實體。(Ty = '3') </sub>

Name    |說明                            |類型       | 作業
------- | ------------------------------------- | --------- | ----------------------------
id      |實體識別碼                              |Int64      |Equals
CN      |會議系列標準化名稱      |字串     |Equals
DCN     |會議系列顯示名稱         |字串     |None
CC      |會議系列引用總數         |Int32      |None  
ECC     |會議系列引用預估總數   |Int32      |None
F.FId   |與會議系列相關聯的研究領域實體識別碼 |Int64  | Equals
F.FN    |與會議系列相關聯的研究領域名稱  | Equals,<br/>StartsWith