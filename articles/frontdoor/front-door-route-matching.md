---
title: Azure Front Door Service - 路由規則比對監視 | Microsoft Docs
description: 此文章可協助您了解 Azure Front Door Service 如何比對要為連入要求使用的路由規則
services: front-door
documentationcenter: ''
author: sharad4u
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: sharadag
ms.openlocfilehash: 23582215654ff2d5003fe611c7149ad760d72bc5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957035"
---
# <a name="how-front-door-matches-requests-to-a-routing-rule"></a>Front Door 比對要求與路由規則的方式

建立連線並進行 SSL 交握之後，當要求進入 Front Door 環境時，Front Door 首先會從所有設定中判斷符合該要求的特定路由規則，然後再執行定義的動作。 下列文件說明 Front Door 在處理 HTTP 要求時，如何判斷要使用的路由設定。

## <a name="structure-of-a-front-door-route-configuration"></a>Front Door 路由設定的結構
Front Door 路由規則設定主要由兩個部分組成：「左邊」與「右邊」。 我們會比對連入要求與左邊的路由，而右邊則定義處理要求的方式。

### <a name="incoming-match-left-hand-side"></a>連入比對 (左邊)
下列屬性可判斷連入要求與路由規則 (或左邊) 是否相符：

* **HTTP 通訊協定** (HTTP/HTTPS)
* **主機** (例如 www.foo.com、\*.bar.com)
* **路徑** (例如 /\*、/users/\*、/file.gif)

這些屬性是從內部展開，因此通訊協定/主機/路徑的每個組合都是可能的相符項目集合。

### <a name="route-data-right-hand-side"></a>路由資料 (右邊)
如何處理要求的決策取決於是否為特定路由啟用快取。 如果沒有快取的要求回應，我們會將要求轉送至所設定後端集區中的適當後端。

## <a name="route-matching"></a>路由比對
此節著重在如何我們比對指定的 Front Door 路由規則。 基本概念是我們只查看「左邊」，一律**先比對最明確的相符項目**。  我們首先會根據 HTTP 通訊協定來比對，然後比對前端主機、路徑。

### <a name="frontend-host-matching"></a>前端主機比對
比對前端主機時，我們會使用如下所示的邏輯：

1. 尋找主機上完全相符的任何路由。
2. 如果沒有完全相符的前端主機項目，則拒絕要求，並傳送「400 不正確的要求」錯誤。

為了進一步說明此程序，讓我們看看 Front Door 路由 (僅左邊) 的範例設定：

| 路由規則 | 前端主機 | Path |
|-------|--------------------|-------|
| A | foo.contoso.com | /\* |
| B | foo.contoso.com | /users/\* |
| C | www.fabrikam.com、foo.adventure-works.com  | /\*、/images/\* |

如果已將下列連入要求傳送到 Front Door，則會由上而下來比對下列路由規則：

| 連入前端主機 | 相符的路由規則 |
|---------------------|---------------|
| foo.contoso.com | A、B |
| www.fabrikam.com | C |
| images.fabrikam.com | 錯誤 400：不正確的要求 |
| foo.adventure-works.com | C |
| contoso.com | 錯誤 400：不正確的要求 |
| www.adventure-works.com | 錯誤 400：不正確的要求 |
| www.northwindtraders.com | 錯誤 400：不正確的要求 |

### <a name="path-matching"></a>路徑比對
在判斷特定前端主機並將可能的路由規則篩選到只剩該前端主機的路由之後，Front Door 接著會根據要求路徑來篩選路由規則。 我們會使用和前端主機類似的邏輯：

1. 尋找路徑上完全相符的任何路由規則
2. 如果沒有完全相符的路徑，則利用萬用字元比對路徑，尋找路由規則
3. 如果找不到與路徑相符的任何路由規則，則拒絕要求並傳回「400：不正確的要求」錯誤的 HTTP 回應。

>[!NOTE]
> 任何不含萬用字元的路徑都會被視為完全相符的路徑。 即使路徑以斜線為結尾，仍會被視為完全相符。

為了進一步說明，讓我們看看另一組範例：

| 路由規則 | 前端主機    | Path     |
|-------|---------|----------|
| A     | www.contoso.com | /        |
| B     | www.contoso.com | /\*      |
| C     | www.contoso.com | /ab      |
| D     | www.contoso.com | /abc     |
| E     | www.contoso.com | /abc/    |
| F     | www.contoso.com | /abc/\*  |
| G     | www.contoso.com | /abc/def |
| H     | www.contoso.com | /path/   |

在給定該設定的情況下，會產生下列範例比對表格：

| 連入要求    | 相符的路由 |
|---------------------|---------------|
| www.contoso.com/            | A             |
| www.contoso.com/a           | B             |
| www.contoso.com/ab          | C             |
| www.contoso.com/abc         | D             |
| www.contoso.com/abzzz       | B             |
| www.contoso.com/abc/        | E             |
| www.contoso.com/abc/d       | F             |
| www.contoso.com/abc/def     | G             |
| www.contoso.com/abc/defzzz  | F             |
| www.contoso.com/abc/def/ghi | F             |
| www.contoso.com/path        | B             |
| www.contoso.com/path/       | H             |
| www.contoso.com/path/zzz    | B             |

>[!WARNING]
> </br> 如果使用全面涵蓋路由路徑 (`/*`) 的完全相符前端主機找不到任何路由規則，則任何路由規則都不會有相符項目。
>
> 範例設定：
>
> | 路由 | 主機             | 路徑    |
> |-------|------------------|---------|
> | A     | profile.contoso.com | /api/\* |
>
> 比對表格：
>
> | 連入要求       | 相符的路由 |
> |------------------------|---------------|
> | profile.domain.com/other | 無。 錯誤 400：不正確的要求 |

### <a name="routing-decision"></a>路由決策
在比對到單一 Front Door 路由規則之後，接著我們需要選擇如何處理該要求。 如果 Front Door 有快取的回應可供相符的路由規則使用，則會以相同的回應傳回給用戶端。 否則，下一步就是評估是否您已為相符的路由規則設定 [URL 重寫 (自訂轉送路徑)](front-door-url-rewrite.md)。 如果未定義自訂轉送路徑，則會將要求依現況轉送到所設定後端集區中的適當後端。 否則會根據定義的[自訂轉送路徑](front-door-url-rewrite.md)更新要求路徑，然後轉送至後端。

## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。
