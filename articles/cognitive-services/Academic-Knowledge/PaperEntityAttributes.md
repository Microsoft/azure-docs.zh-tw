---
title: 學術知識 API 中的論文實體屬性 | Microsoft Docs
description: 了解您可以在認知服務的學術知識 API 中搭配論文實體使用的屬性。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/31/2017
ms.author: alch
ms.openlocfilehash: 75efba2c2ce8842f233f766876ca00844338fb25
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094706"
---
# <a name="paper-entity"></a>論文實體

<sub> *下列屬性專屬於論文實體。(Ty = '0') </sub>


名稱    |說明                                        |類型       | 作業
------- | ------------------------------------------------- | --------- | ----------------------------
id      |實體識別碼                                          |Int64      |Equals
Ti      |論文標題                                        |字串     |Equals,<br/>StartsWith
L       |以 "\@@@" 分隔的論文語言程式碼            |字串     |Equals
Y       |論文年度                                         |Int32      |Equals,<br/>IsBetween
D       |論文日期                                         |日期       |Equals,<br/>IsBetween
CC      |引用計數                                     |Int32      |None  
ECC     |預估引用計數                           |Int32      |None
AA.AuN  |作者姓名                                        |字串     |Equals,<br/>StartsWith
AA.AuId |作者識別碼                                          |Int64      |Equals
AA.AfN  |作者服務機關名稱                            |字串     |Equals,<br/>StartsWith
AA.AfId |作者服務機關識別碼                              |Int64      |Equals
AA.S    |論文的作者順序                         |Int32      |Equals
F.FN    |研究領域名稱                                |字串     |Equals,<br/>StartsWith
F.FId   |研究領域識別碼                                  |Int64      |Equals
J.JN    |期刊名稱                                       |字串     |Equals,<br/>StartsWith
J.JId   |期刊識別碼                                         |Int64      |Equals
C.CN    |會議系列名稱                             |字串     |Equals,<br/>StartsWith
C.CId   |會議系列識別碼                               |Int64      |Equals
RId     |所參考論文的識別碼                              |Int64[]    |Equals
W       |論文標題和摘要的文字                |String[]   |Equals
E       |擴充中繼資料 (請參閱下表)                |字串     |None  
        


## <a name="extended-metadata-attributes"></a>擴充中繼資料屬性 ##

名稱    | 說明               
--------|---------------------------    
DN      | 論文的顯示名稱 
S       | 來源 - 論文的 Web 來源清單，依靜態順位排序
S.Ty    | 來源類型 (1：HTML、2：Text、3：PDF、4：DOC、5：PPT、6：XLS、7：PS)
S.U     | 來源 URL
VFN     | 場地全名 - 期刊或會議的全名
VSN     | 場地簡短名稱 - 期刊或會議的簡短名稱
V       | 合訂本 - 期刊合訂本
BV      | 期刊名稱
BT      | 
PB      | 期刊縮寫
I       | 期號 - 期刊期號
FP      | FirstPage - 論文第一頁
LP      | LastPage - 論文最後一頁
DOI     | 數位對象識別碼
CC      | 引用內容 – 所參考論文識別碼和論文中對應內容的清單 (例如 [{123:[“brown foxes are known for jumping as referenced in paper 123”, “the lazy dogs are a historical misnomer as shown in paper 123”]})
IA      | 反向摘要
IA.IndexLength| 索引中的項目數目 (摘要的字數統計)
IA.InvertedIndex| 摘要文字和其在原始摘要中對應位置的清單 (例如 [{“the”:[0, 15, 30]}, {“brown”:[1]}, {“fox”:[2]}])
