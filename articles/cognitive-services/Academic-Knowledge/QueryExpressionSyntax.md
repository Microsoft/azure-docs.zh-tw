---
title: 學術知識 API 中的查詢運算式語法 | Microsoft Docs
description: 了解如何在 Microsoft 認知服務的學術知識 API 中使用查詢運算式語法。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/27/2017
ms.author: alch
ms.openlocfilehash: 6ec338fff09954e2f14066ce2b83bc1228794af8
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367878"
---
# <a name="query-expression-syntax"></a>查詢運算式語法

我們發現**解譯**要求的回應包含查詢運算式。 解譯使用者查詢的文法為每個解譯建立了一個查詢運算式。 查詢運算式可接著用來發出**評估**要求，以擷取實體搜尋結果。

您也可以建構自己的查詢運算式，並且在**評估**要求中使用它們。 如果您要建置自己的使用者介面，以便建立查詢運算式來回應使用者的動作，這就很實用。 若要這樣做，您需要知道查詢運算式的語法。  

可包含在查詢運算式中的每個實體屬性都具有特定資料類型以及一組可能的查詢運算子。 實體屬性集和每個屬性支援的運算子均在[實體屬性](EntityAttributes.md)中指定。 單一值查詢需要支援 *Equals* 作業的屬性。 首碼查詢需要支援 *StartsWith* 作業的屬性。 數值範圍查詢需要支援 *IsBetween* 作業的屬性。

有些實體資料會儲存為複合屬性，如屬性名稱中的點 '.' 所表示。 例如，作者/服務機關資訊是以複合屬性表示。 它包含 4 個元件：AuN、AuId、AfN、AfId。 這些元件是形成單一實體屬性值的不同資料部分。


**字串屬性：單一值** (包括同義字的相符項目)  
Ti='indexing by latent semantic analysis'  
Composite(AA.AuN='sue dumais')

**字串屬性：確切單一值** (僅符合標準值)  
Ti=='indexing by latent semantic analysis'  
Composite(AA.AuN=='susan t dumais')
     
**字串屬性：首碼值**   
Ti='indexing by latent seman'...  
Composite(AA.AuN='sue du'...)

**數值屬性：單一值**  
Y=2010
 
**數值屬性：範圍值**  
Y>2005  
Y>=2005  
Y<2010  
Y<=2010  
Y=\[2010, 2012\) (includes only left boundary value: 2010, 2011)  
Y=\[2010, 2012\] (includes both boundary values: 2010, 2011, 2012)
 
**數值屬性：首碼值**  
Y='19'... (any numeric value that starts with 19) 
 
**日期屬性：單一值**  
D='2010-02-04'

**日期屬性：範圍值**  
D>'2010-02-03'  
D=['2010-02-03','2010-02-05']

**And/Or 查詢：**  
And(Y=1985, Ti='disordered electronic systems')  
Or(Ti='disordered electronic systems', Ti='fault tolerance principles and practice')  
And(Or(Y=1985,Y=2008), Ti='disordered electronic systems')
 
**複合查詢：**  
若要查詢複合屬性的元件，您必須在 Composite() 函式中放入查詢運算式的一部分 (參照複合屬性)。 

例如，若要依照作者名稱查詢論文，請使用下列查詢：
```
Composite(AA.AuN='mike smith')
```
<br>若要依照在特定機構時的特定作者查詢論文，請使用下列查詢：
```
Composite(And(AA.AuN='mike smith',AA.AfN='harvard university'))
```
<br>Composite() 函式會將複合屬性的兩個部分繫結在一起。 這表示我們只取得其中一位作者 "Mike Smith" 在 Harvard 時期的論文。 

若要依照服務機關中的特定作者與來自特定機構的 (其他) 作者查詢論文，請使用下列查詢：
```
And(Composite(AA.AuN='mike smith'),Composite(AA.AfN='harvard university'))
```
<br>在此版本中，Composite() 會在 And() 之前個別地套用至作者和服務機關，所以我們會取得其中一位作者是 “Mike Smith” 且其中一位作者服務機關是 “Harvard” 的所有論文。 這聽起來類似於先前的查詢範例，但並不相同。

一般而言，請考慮下列範例：我們具有複合屬性 C，而該屬性具有兩個元件 A 和 b。實體對於屬性 C 可以有多個值。以下是我們的實體：
```
E1: C={A=1, B=1}  C={A=1,B=2}  C={A=2,B=3}
E2: C={A=1, B=3}  C={A=3,B=2}
```

<br>此查詢 
```
Composite(And(C.A=1, C.B=2))
```

<br>只比對具有屬性 C 值 (其中元件 C.A 為 1 且元件 C.B 為 2) 的實體。 只有 E1 符合此查詢。

此查詢 
```
And(Composite(C.A=1), Composite(C.B=2))
```
<br>比對具有屬性 C 值 (其中元件 C.A 為 1)，也具有屬性 C 值 (其中元件 C.B 為 2) 的實體。 E1 和 E2 都符合此查詢。

請注意：  
- 您無法參考 Composite() 函式外複合屬性的組件。
- 您無法參考相同 Composite() 函式內兩個不同複合屬性的組件。
- 您無法參考不屬於 Composite() 函式內複合屬性的屬性。
