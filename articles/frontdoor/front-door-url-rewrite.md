---
title: Azure Front Door Service - URL 重寫 | Microsoft Docs
description: 此文章可協助您了解 Azure Front Door Service 針對您的路由進行 URL 重寫的方式 (若已設定)。
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
ms.openlocfilehash: 00fe3aa7a641b9d07aad90a9d008a99efc6e9d97
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993460"
---
# <a name="url-rewrite-custom-forwarding-path"></a>URL 重寫 (自訂轉送路徑)
Azure Front Door Service 能透過允許您設定選擇性的**自訂轉送路徑**，以用於建構要轉送至後端之要求的情況，來支援 URL 重寫功能。 根據預設，若沒有提供自訂轉送路徑，Front Door 會將傳入的 URL 路徑複製到用於轉送要求的 URL。 用於轉送要求中的主機標頭，與針對所選後端設定的標頭相同。 請參閱[後端主機標頭](front-door-backend-pool.md#hostheader)以了解其功用與設定方式。

URL 重寫使用自訂轉送路徑的強大之處，在於它會複製傳入路徑中符合針對轉送路徑之萬用字元路徑 (這些路徑區段為下列範例中的**綠色**區段) 的所有部分：
</br>
![Azure Front Door URL 重寫][1]

## <a name="url-rewrite-example"></a>URL 重寫範例
以已設定下列前端主機與路徑的路由規則為例：

| 主機      | 路徑       |
|------------|-------------|
| www.contoso.com | /\*         |
|            | /foo        |
|            | /foo/\*     |
|            | /foo/bar/\* |

下表的第一欄顯示傳入要求的範例，而第二欄則顯示「最明確」的相符路由「路徑」。  表格第一列的第三欄與後續的欄為已設定的**自訂轉送路徑**範例，而那些欄中其餘的列，則代表轉送路徑在符合該列之要求時的模樣。

例如，如果我們閱讀第二列的內容，便能知道針對傳入要求 `www.contoso.com/sub`，如果自訂轉送路徑為 `/`，則轉送路徑將會是 `/sub`。 如果自訂轉送路徑為 `/fwd/`，則轉送路徑將會是 `/fwd/sub`。 針對其餘的欄也依此類推。 下列路徑中的**粗體**部分，則代表符合萬用字元的部分。


| 傳入要求       | 最明確的相符路徑 | /          | /fwd/          | /foo/          | /foo/bar/          |
|------------------------|--------------------------|------------|----------------|----------------|--------------------|
| www.contoso.com/            | /\*                      | /          | /fwd/          | /foo/          | /foo/bar/          |
| www.contoso.com/**sub**     | /\*                      | /**sub**   | /fwd/**sub**   | /foo/**sub**   | /foo/bar/**sub**   |
| www.contoso.com/**a/b/c**   | /\*                      | /**a/b/c** | /fwd/**a/b/c** | /foo/**a/b/c** | /foo/bar/**a/b/c** |
| www.contoso.com/foo         | /foo                     | /          | /fwd/          | /foo/          | /foo/bar/          |
| www.contoso.com/foo/        | /foo/\*                  | /          | /fwd/          | /foo/          | /foo/bar/          |
| www.contoso.com/foo/**bar** | /foo/\*                  | /**bar**   | /fwd/**bar**   | /foo/**bar**   | /foo/bar/**bar**   |


## <a name="optional-settings"></a>選擇性設定
還有其他選擇性設定可供您針對任何指定的路由規則設定進行指定：

* **快取設定**：若停用或未指定，則符合此路由規則的要求將不會嘗試使用快取內容，而一律會從後端擷取。 深入閱讀[搭配 Front Door 進行快取](front-door-caching.md)。



## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。

<!--Image references-->
[1]: ./media/front-door-url-rewrite/front-door-url-rewrite-example.jpg