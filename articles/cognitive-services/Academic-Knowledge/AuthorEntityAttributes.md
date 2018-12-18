---
title: 學術知識 API 中的作者實體屬性 | Microsoft Docs
description: 了解您可以在認知服務的學術知識 API 中搭配作者實體使用的屬性。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/23/2017
ms.author: alch
ms.openlocfilehash: afcc3257c49522a4631c4aae0e0c2b88373f9af1
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367823"
---
# <a name="author-entity"></a>作者實體
<sub> *下列屬性專屬於作者實體。(Ty = '1') </sub>

Name    |說明                            |類型       | 作業
------- | ------------------------------------- | --------- | ----------------------------
id      |實體識別碼                              |Int64      |Equals
AuN     |作者標準化名稱                 |字串     |Equals
DAuN    |作者顯示名稱                    |字串     |None
CC      |作者引用總數            |Int32      |None  
ECC     |作者引用預估總數  |Int32      |None
E       |擴充中繼資料 (請參閱「擴充中繼屬性」表格)  |字串     |None  


## <a name="extended-metadata-attributes"></a>擴充中繼資料屬性 ##

Name    | 說明               
--------|---------------------------    
LKA.Afn     | 與作者相關聯的服務機關顯示名稱  
LKA.AfId        | 與作者相關聯的服務機關實體識別碼