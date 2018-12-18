---
title: Azure 搜尋服務中的 moreLikeThis (預覽) | Microsoft Docs
description: Azure Search REST API 中公開的 moreLikeThis (預覽) 功能預備文件。
authors: mhko
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 10/27/2016
ms.author: nateko
ms.openlocfilehash: 29d9a478ca2e91e658d7d0f52e7a193ba694bc16
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31790724"
---
# <a name="morelikethis-in-azure-search-preview"></a>Azure 搜尋服務中的 moreLikeThis (預覽)

`moreLikeThis=[key]` 是[搜尋 API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 中的查詢參數。 在搜尋查詢中指定 `moreLikeThis` 參數時，所尋找的文件類似文件索引鍵所指定的文件。 使用 `moreLikeThis` 提出搜尋要求時，系統會利用最能描述該文件的指定文件中擷取的搜尋詞彙，產生查詢。 然後將產生的查詢用於提出搜尋要求。 除非使用 `searchFields` 參數限制欄位，否則所有 `searchable` 欄位的內容預設都會納入考慮。 `moreLikeThis` 參數無法搭配搜尋參數 `search=[string]` 使用。

## <a name="examples"></a>範例 

以下是 moreLikeThis 查詢的範例。 查詢會尋找其描述欄位最類似 `moreLikeThis` 參數所指定來源文件之欄位的文件。

```  
Get /indexes/hotels/docs?moreLikeThis=1002&searchFields=description&api-version=2016-09-01-Preview
```  

```  
POST /indexes/hotels/docs/search?api-version=2016-09-01-Preview
    {  
      "moreLikeThis": "1002",  
      "searchFields": "description"  
    }  
```  

## <a name="feature-availability"></a>功能可用性

moreLikeThis 功能目前僅提供預覽，因此只在預覽 API 版本 `2015-02-28-Preview` 和 `2016-09-01-Preview` 中提供支援。 因為需在要求中指定才能取得 API 版本，因此可以在同一個應用程式中結合就可以結合正式推出 (GA) 與預覽 API 功能。 然而，預覽 API 不受 SLA 約束，且功能可能會變更，因此不建議在生產應用程式中使用。