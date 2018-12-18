---
title: 學術知識 API 中的服務機關實體屬性 | Microsoft Docs
description: 了解您可以在認知服務的學術知識 API 中搭配服務機關實體使用的屬性。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/23/2017
ms.author: alch
ms.openlocfilehash: a0ec67cb811ca207b3d038028491da2516028f0b
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367822"
---
# <a name="affiliation-entity"></a>服務機關實體

<sub> *下列屬性專屬於服務機關實體。(Ty = '5') </sub>

Name    |說明                            |類型       | 作業
------- | ------------------------------------- | --------- | ----------------------------
id      |實體識別碼                              |Int64      |Equals
AfN     |服務機關標準化名稱        |字串     |Equals
DAfN    |服務機關顯示名稱       |字串     |None
CC      |服務機關引用總數           |Int32      |None  
ECC     |服務機關引用預估總數 |Int32      |None

## <a name="extended-metadata-attributes"></a>擴充中繼資料屬性 ##

Name    | 說明               
--------|---------------------------    
PC      |服務機關的論文計數