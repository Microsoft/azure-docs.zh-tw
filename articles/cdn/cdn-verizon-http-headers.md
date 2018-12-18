---
title: Azure CDN 規則引擎的 Verizon 專屬 HTTP 標頭 | Microsoft Docs
description: 本文說明如何使用 Verizon 專屬 HTTP 標頭搭配 Azure CDN 規則引擎。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/16/2018
ms.author: magattus
ms.openlocfilehash: 7fa76a2c5b01e623e490edd0091f7fb372b7085f
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093233"
---
# <a name="verizon-specific-http-headers-for-azure-cdn-rules-engine"></a>Azure CDN 規則引擎的 Verizon 專屬 HTTP 標頭

對於**來自 Verizon 的進階 Azure CDN** 產品，當 HTTP 要求傳送至原始伺服器時，網路節點 (POP) 伺服器可以將用戶端要求中的一個或多個保留標頭 (或 Proxy 特殊標頭) 新增至 POP。 這些是除了接收標準轉送標頭以外會接收的標頭。 如需標準要求標頭相關資訊，請參閱[要求欄位](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Request_fields)。

如果您想要避免其中一個保留標頭新增於原始伺服器的 Azure CDN (內容傳遞網路) POP 要求中，您必須使用規則引擎中的 [Proxy 特殊標頭功能](cdn-rules-engine-reference-features.md#proxy-special-headers)建立規則。 在此規則中，排除您想要從標頭欄位中預設標頭清單移除的標頭。 如果您已啟用[偵錯快取回應標頭功能](cdn-rules-engine-reference-features.md#debug-cache-response-headers)，務必新增必要的 `X-EC-Debug` 標頭。 

例如，若要移除 `Via` 標頭，規則的標頭欄位應該包含下列標頭清單：X-Forwarded-For、X-Forwarded-Proto、X-Host、X-Midgress、X-Gateway-List、X-EC-Name、Host。 

![Proxy 特殊標頭規則](./media/cdn-http-headers/cdn-proxy-special-header-rule.png)

下表說明可由 Verizon CDN POP 在要求中新增的標頭：

要求標頭 | 說明 | 範例
---------------|-------------|--------
[Via](#via-request-header) | 識別已將要求透過 Proxy 處理至原始伺服器的 POP 伺服器。 | HTTP/1.1 ECS (dca/1A2B)
X-Forwarded-For | 表示要求者的 IP 位址。| 10.10.10.10
X-Forwarded-Proto | 表示要求的通訊協定。 | http
X-Host | 表示要求的主機名稱。 | cdn.mydomain.com
X-Midgress | 表示要求是否已透過其他 CDN 伺服器進行 Proxy 處理。 例如，POP server-to-origin 防護伺服器或 POP server-to-ADN 閘道伺服器。 <br />只有發生 Midgress 流量時，才會將此標頭新增至要求。 在此情況下，標頭設定為 1，表示要求已透過其他 CDN 伺服器進行 Proxy 處理。| 1
[Host](#host-request-header) | 識別可以找到所要求內容的主機和連接埠。 | marketing.mydomain.com:80
[X-Gateway-List](#x-gateway-list-request-header) | ADN：識別指派給客戶原始伺服器的 ADN 閘道伺服器容錯移轉清單。 <br />原始防護：表示指派給客戶原始伺服器的一組原始防護伺服器。 | `icn1,hhp1,hnd1`
X-EC-_&lt;name&gt;_ | 開頭為 X-EC 的要求標頭 (例如，X-EC-Tag、[X-EC-Debug](cdn-http-debug-headers.md)) 已保留供 CDN 使用。| waf-production

## <a name="via-request-header"></a>Via 要求標頭
`Via` 要求標頭可透過此格式識別下列語法所指定的 POP 伺服器：

`Via: Protocol from Platform (POP/ID)` 

語法中使用的詞彙定義如下：
- 通訊協定：表示用來以 Proxy 處理要求的通訊協定版本 (例如，HTTP/1.1)。 

- 平台：表示在其上要求內容的平台。 下列程式碼適用於此欄位： 
    代碼 | 平台
    -----|---------
    ECAcc | HTTP Large
    ECS   | HTTP Small
    ECD   | 應用程式傳遞網路 (ADN)

- POP：表示處理要求的 [POP](cdn-pop-abbreviations.md)。 

- ID：僅供內部使用。

### <a name="example-via-request-header"></a>範例 Via 要求標頭

`Via: HTTP/1.1 ECD (dca/1A2B)`

## <a name="host-request-header"></a>主機要求標頭
當下列兩個條件都成立時，POP 伺服器會覆寫 `Host` 標頭：
- 所要求內容的來源是客戶原始伺服器。
- 對應客戶原始伺服器的 [HTTP 主機標頭] 選項不是空白的。

`Host` 要求標頭會遭到覆寫，以反映 [HTTP 主機標頭] 選項中定義的值。
如果客戶原始伺服器的 [HTTP 主機標頭] 選項設為空白，則要求者所提交的 `Host` 要求標頭會轉送至客戶的原始伺服器。

## <a name="x-gateway-list-request-header"></a>X-Gateway-List 要求標頭
符合下列任一條件時，POP 伺服器就會新增/覆寫 X-Gateway-List 要求標頭：
- 指向 ADN 平台的要求。
- 要求已轉送至「原始防護」功能所保護的客戶原始伺服器。

