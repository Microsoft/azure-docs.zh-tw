---
title: 什麼是 Azure 轉送？ | Microsoft Docs
description: 本文概述 Azure 轉送服務，此服務可讓您開發雲端應用程式來取用公司網路中所執行的內部部署服務，而不需要開啟防火牆連線或對網路基礎結構進行侵入式變更。
services: service-bus-relay
author: spelluru
manager: ''
editor: ''
ms.assetid: 1e3e971d-2a24-4f96-a88a-ce3ea2b1a1cd
ms.service: service-bus-relay
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: get-started-article
ms.date: 10/08/2018
ms.author: spelluru
ms.openlocfilehash: 3cc87c0acbed317cccaccec687f27c23a1d32cf0
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319331"
---
# <a name="what-is-azure-relay"></a>什麼是 Azure 轉送？
Azure 轉送服務可讓您在公用雲端中安全地公開公司網路中所執行的服務。 您不需要開啟防火牆連線，也不需要對公司網路基礎結構進行侵入式變更，就能做到這一點。 

轉送服務可在內部部署服務以及雲端或另一個內部部署環境中所執行的應用程式之間，支援下列案例。 

- 傳統的單向、要求/回應和點對點通訊 
- 在網際網路範圍散發事件，以實現發佈/訂閱案例 
- 跨網路界限的雙向、未緩衝通訊端通訊。

Azure 轉送不同於網路層級的整合技術，例如 VPN。 Azure 轉送可以侷限在單一機器上的單一應用程式端點內。 VPN 技術的侵入性則高得多，原因是其依賴於改變網路環境。 

## <a name="basic-flow"></a>基本流程
在轉送的資料傳輸模式中，所需的基本步驟如下：

1. 內部部署服務透過輸出連接埠連線到轉送服務。 
2. 它會建立雙向通訊端以供進行繫結至特定位址的通訊。 
3. 用戶端接著可將流量傳送至以該位址為目標的轉送服務，藉此與內部部署服務通訊。 
4. 轉送服務接著會透過用戶端專用的雙向通訊端，將資料「轉送」至內部部署服務。 用戶端不需要直接連線到內部部署服務。 它不需要知道服務的所在位置。 而且，內部部署服務也不需要在防火牆上開啟任何輸入連接埠。


## <a name="features"></a>特性 
Azure 轉送有兩項功能︰

- [混合式連線](#hybrid-connections) - 使用開放式標準 Web 通訊端來啟用多平台案例。
- [WCF 轉送](#wcf-relays) - 使用 Windows Communication Foundation (WCF) 來啟用遠端程序呼叫。 WCF 轉送是舊版的轉送供應項目，許多客戶已將該服務用於其 WCF 程式設計模型。

## <a name="hybrid-connections"></a>混合式連線

Azure 轉送中的混合式連線功能，是從先前存在的轉送功能演化而來的安全、開放式通訊協定。 您可以在任何平台以任何語言使用它。 Azure 轉送中的混合式連線功能會以 HTTP 和 Websocket 通訊協定作為基礎。 它可讓您透過 Web 通訊端或 HTTP (S) 傳送要求並接收回應。 此功能可與常用網頁瀏覽器中的 WebSocket API 相容。 

如需混合式連線通訊協定的詳細資訊，請參閱[混合式連線通訊協定指南](relay-hybrid-connections-protocol.md)。 您可以搭配適用於任何執行階段/語言的任何 Web 通訊端程式庫來使用混合式連線。

> [!NOTE]
> Azure 轉送的混合式連線取代了 BizTalk 服務舊有的混合式連線功能。 BizTalk 服務中的混合式連線功能是建置在 Azure 服務匯流排的 WCF 轉送之上。 Azure 轉送中的混合式連線功能可補充既有 WCF 轉送功能的不足之處。 這兩個服務功能 (WCF 轉送和混合式連線) 並存於 Azure 轉送服務中。 它們共用通用的閘道，但有不同的實作方式。

## <a name="wcf-relay"></a>WCF 轉送
WCF 轉送適用於完整的 .NET Framework 和 WCF。 您可以在內部部署服務與使用一組 WCF「轉送」繫結的轉送服務之間建立連線。 在幕後，轉送繫結會對應至新的傳輸繫結元素，其設計來建立與雲端中服務匯流排整合的 WCF 通道元件。 如需詳細資訊，請參閱[開始使用 WCF 轉送](relay-wcf-dotnet-get-started.md)。

## <a name="hybrid-connections-vs-wcf-relay"></a>混合式連線與WCF 轉送
混合式連線和 WCF 轉送都能夠對存在於網路內的資產進行安全的連線。 視您的特定需求使用其中一項功能，詳述於下表︰

|  | WCF 轉送 | 混合式連線 |
| --- |:---:|:---:|
| **WCF** |x | |
| **.NET Core** | |x |
| **.NET Framework** |x |x |
| **Java 指令碼/Node.JS** | |x |
| **標準型開放式通訊協定** | |x |
| **多個 RPC 程式設計模型** | |x |

## <a name="architecture-processing-of-incoming-relay-requests"></a>架構：處理內送轉送要求
下圖說明 Azure 轉送服務會如何處理內送的轉送要求：

![處理內送 WCF 轉送要求](./media/relay-what-is-it/ic690645.png)

1. 接聽方用戶端會對 Azure 轉送服務傳送接聽要求。 Azure 負載平衡器會將要求路由傳送到其中一個閘道節點。 
2. Azure 轉送服務會在閘道存放區建立轉送。 
3. 傳送方用戶端會傳送要連線到接聽服務的要求。 
4. 收到要求的閘道會查詢閘道存放區中的轉送。 
5. 閘道會將連線要求轉送給閘道存放區中提及的正確閘道。 
6. 閘道會將要求傳送給接聽方用戶端，讓它對最接近傳送方用戶端的閘道節點建立暫時通道。 
7. 接聽方用戶端會建立暫時通道，並傳送回應給最接近傳送方用戶端的閘道。 現在，用戶端之間已透過閘道建立連線，用戶端可以開始互相交換訊息。 
8. 閘道會將來自接聽方用戶端的任何訊息轉送至傳送方用戶端。 
9. 閘道會將來自傳送方用戶端的任何訊息轉送至接聽方用戶端。  

## <a name="next-steps"></a>後續步驟
* [開始使用 .NET Websocket](relay-hybrid-connections-dotnet-get-started.md)
* [開始使用 .NET HTTP 要求](relay-hybrid-connections-http-requests-dotnet-get-started.md)
* [開始使用 Node Websocket](relay-hybrid-connections-node-get-started.md)
* [開始使用 Node HTTP 要求](relay-hybrid-connections-http-requests-node-get-started.md)
* [轉送常見問題集](relay-faq.md)

