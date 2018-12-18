---
title: 什麼是學術知識 API？
titlesuffix: Azure Cognitive Services
description: 使用學術知識 API 來解譯使用者查詢，以及從 Academic Graph 中擷取豐富資訊。
services: cognitive-services
author: mvorvoreanu
manager: cgronlun
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: overview
ms.date: 03/27/2017
ms.author: mivorvor
ms.openlocfilehash: d08cd7124b232e50365e72753eba97c6309f401c
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48901205"
---
# <a name="academic-knowledge-api"></a>Academic Knowledge API

歡迎使用學術知識 API。 有了這項服務，您便可針對學術目的來解譯使用者查詢，並從 Microsoft Academic Graph (MAG) 擷取豐富資訊。 MAG 知識庫是 Web 規模的異質實體圖表，並由為學術活動建模的實體所組成：研究領域、作者、機構、論文、地點和事件。 

MAG 資料都採礦於 Bing Web 索引和 Bing 之中的內部知識庫。 由於 Bing 的索引編制會持續進行，所以此 API 會因為 Bing 的探索和索引編製而包含最新 Web 資訊。 根據此資料集，學術知識 API 可啟用知識驅動的互動式對話，其透過主動式建議體驗、多種研究論文的圖表搜尋結果，以及一組論文和相關實體屬性值的長條圖分佈，與回應式搜尋完美結合。

如需有關 Microsoft Academic Graph 的詳細資訊，請參閱 [http://aka.ms/academicgraph](http://aka.ms/academicgraph)。

學術知識 API 已經從認知服務預覽移至認知服務實驗室。 此專案的新首頁是：[https://labs.cognitive.microsoft.com/en-us/project-academic-knowledge](https://labs.cognitive.microsoft.com/en-us/project-academic-knowledge)。 您現有 API 金鑰的有效期限是 2018 年 5 月 24 日。 在此日期之後，請產生新的 API 金鑰。 請注意，您現有金鑰到期之後，將無法再使用付費預覽版。 如果 API 的免費層不夠您使用，連絡我們的團隊。 

## <a name="features"></a>特性
學術知識 API 包含四個相關的 REST 端點：  
  1. **interpret** – 解譯自然語言的使用者查詢字串。 傳回已標註的解譯來獲得豐富的搜尋方塊自動完成體驗，進而預期使用者所鍵入的項目。  
  2. **evaluate** – 評估查詢運算式並傳回學術知識實體結果。  
  3. **calchistogram** – 計算查詢運算式所傳回的學術實體屬性值發佈長條圖，例如指定的作者依年份的引文發佈。  
  4. **graph search** – 搜尋指定的圖表模式並傳回相符的實體結果。

一起使用這些 API 方法，即可打造豐富的語意搜尋體驗。 指定使用者查詢字串後，**interpret** 方法會提供註釋版的查詢和結構化查詢運算式，同時根據基礎學術資料的語意選擇性地完成使用者查詢。 例如，如果使用者輸入字串 latent s，**interpret** 方法會提供一組有排名的解譯，並暗示使用者可能會搜尋 latent semantic analysis 研究領域、latent structure analysis 論文或其他開頭為 latent s 的實體運算式。 這項資訊可用來快速將使用者指引至所需的搜尋結果。

**evaluate**方法可用於從學術知識庫中擷取一組相符的論文實體，而 **calchistogram** 方法可用於計算一組論文實體的屬性值分佈，其可用來進一步篩選搜尋結果。        

**graph search**方法有兩種模式：json 和 lambda。 Json 模式可以依據 JSON 物件所指定的圖表模式來執行圖表模式比對。 lambda 模式可以根據使用者指定的 lambda 運算式，在圖表周遊期間執行伺服器端計算。

## <a name="getting-started"></a>開始使用 
請如需詳細文件，請參閱左側的子主題。  請注意，為了改善範例的可讀性，REST API 呼叫包含未經過 URL 編碼的字元 (例如空格)。  您的程式碼必須套用適當的 URL 編碼。
