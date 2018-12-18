---
title: 在 Azure 媒體服務中管理 Azure CDN 快取原則 | Microsoft Docs
description: 了解如何在 Azure 媒體服務中管理 Azure CDN 快取原則。
services: media-services,cdn
documentationcenter: .NET
author: juliako
manager: erikre
editor: ''
ms.assetid: be33aecc-6dbe-43d7-a056-10ba911e0e94
ms.service: media-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/04/2017
ms.author: juliako
ms.openlocfilehash: dce8d588a78b028223776e3ade737d4abd26094b
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33765280"
---
# <a name="manage-azure-cdn-caching-policy-in-azure-media-services"></a>在 Azure 媒體服務中管理 Azure CDN 快取原則
Azure 媒體服務提供 HTTP 式「彈性資料流」和漸進式下載功能。 HTTP 式串流具有快取 Proxy 和 CDN 層，以及快取用戶端的優點，所以延展性極佳。 資料流端點提供一般串流功能，以及 HTTP 快取標頭的組態。 串流端點會設定 HTTP Cache-Control: max-age 和 Expires 標頭。 您可以從 [W3.org](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html)(英文) 取得 HTTP 快取標頭的詳細資訊。

## <a name="default-caching-headers"></a>預設快取標頭
根據預設，資料流端點會針對隨選資料流處理的資料 (實際的媒體片段/區塊) 和資訊清單 (播放清單) 套用 3 天快取標頭。 如果是即時資料流，資料流端點會針對資料 (實際的媒體片段/區塊) 套用 3 天快取標頭，針對資訊清單 (播放清單) 要求套用 2 秒快取標頭。 當即時節目變成隨選 (即時封存) 時，便會套用隨選資料流處理快取標頭。

## <a name="azure-cdn-integration"></a>Azure CDN 整合
Azure 媒體服務為資料流端點提供 [整合式 CDN](https://azure.microsoft.com/updates/azure-media-services-now-fully-integrated-with-azure-cdn/) 。 Cache-control 標頭的套用方式與將資料流端點套用支援 CDN 的資料流端點的方式相同。 Azure CDN 使用資料流端點設定的快取值，定義內部快取物件的存留期，也用此值設定傳遞快取標頭。 使用支援 CDN 的資料流端點時，不建議將快取值設得太小。 將值設得太小會降低效能，並減少 CDN 帶來的好處。 支援 CDN 的資料流端點的快取標頭值不得設為 600 秒以下。

> [!IMPORTANT]
>Azure 媒體服務可與 Azure CDN 完整整合。 只要按一下滑鼠，就能將所有可用的 Azure CDN 提供者整合到您的串流端點，包括標準和進階產品。 如需詳細資訊，請參閱[此公告](https://azure.microsoft.com/blog/standardstreamingendpoint/) 。
> 
> 只有當 CDN 是透過串流端點 API 或是 Azure 入口網站的串流端點區段啟用時，才會停用串流端點對 CDN 的資料傳輸費用。 手動整合或使用 CDN API (或入口網站區段) 直接建立 CDN 端點並不會停用資料傳輸費用。

## <a name="configuring-cache-headers-with-azure-media-services"></a>使用 Azure 媒體服務設定快取標頭
您可以使用 Azure 入口網站或 Azure 媒體服務 API，設定快取標頭的值。

1. 若要使用設定 Azure 入口網站設定快取標頭，請參閱[如何管理串流端點](../media-services/previous/media-services-portal-manage-streaming-endpoints.md)一節中的「設定串流端點」中的。
2. Azure 媒體服務 REST API， [StreamingEndpoint](https://msdn.microsoft.com/library/azure/dn783468.aspx#StreamingEndpointCacheControl)。
3. Azure 媒體服務 .NET SDK， [StreamingEndpointCacheControl 屬性](http://go.microsoft.com/fwlink/?LinkId=615302)。

## <a name="cache-configuration-precedence-order"></a>快取組態的優先順序
1. Azure 媒體服務的已設定快取值會覆寫預設值。
2. 如果沒有任何手動組態，系統會套用預設值。
3. 根據預設，2 秒快取標頭會套用至即時資料流資訊清單 (播放清單)，無論 Azure 媒體或 Azure 儲存體組態為何，且此值無法被覆寫。

