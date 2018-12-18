---
title: 什麼是 Azure SignalR | Microsoft Docs
description: Azure SignalR 服務的概觀。
services: signalr
documentationcenter: ''
author: sffamily
manager: cfowler
editor: ''
ms.service: signalr
ms.devlang: na
ms.topic: overview
ms.workload: tbd
ms.date: 09/13/2018
ms.author: zhshang
ms.openlocfilehash: a159833936ec4762213f063e235fa4f9237af95b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46951095"
---
# <a name="what-is-azure-signalr-service"></a>什麼是 Azure SignalR 服務

Azure SignalR 服務簡化了透過 HTTP 將即時 Web 功能新增到應用程式的程序。 此即時功能可讓服務將內容更新推播至連線的用戶端，例如單一頁面的 Web 或行動應用程式。 因此，用戶端可以更新而不需要輪詢伺服器或送出新的 HTTP 更新要求。

此文章提供 Azure SignalR 服務的概觀。

## <a name="what-is-azure-signalr-service-used-for"></a>Azure SignalR 服務的用途為何？ 

許多應用程式類型都需要即時內容更新。 下列範例很適合使用 Azure SignalR 服務：

* 需要經常從伺服器取得更新的應用程式。 範例為遊戲、投票、拍賣、地圖和 GPS 應用程式。
* 儀表板和監視應用程式。 範例包括公司儀表板與即時銷售更新。
* 共同作業應用程式。 共同作業應用程式的範例包括白板應用程式和小組會議軟體。
* 需要通知的應用程式。 社交網路、電子郵件、交談、遊戲、旅行警示和其他使用通知的應用程式。

SignalR 為建置即時 Web 應用程式所用的數種技術提供抽象表示。 [WebSockets](https://wikipedia.org/wiki/WebSocket) 是最佳的傳輸，但是當其他選項無法使用時會使用 [Server-Sent Events (SSE)](https://wikipedia.org/wiki/Server-sent_events) 和「長輪詢」等技術。 SignalR 會根據伺服器和用戶端支援的功能，自動偵測和初始化適當的傳輸。

此外，SignalR 提供適用於即時應用程式的程式設計模型，可讓伺服器傳送訊息到所有連線、屬於特定使用者的連線子集，或置於任意群組的連線子集。

## <a name="how-to-use-azure-signalr-service"></a>如何使用 Azure SignalR 服務

目前有三種方式可使用 Azure SignalR 服務：

- **[調整 ASP.NET Core SignalR 應用程式](signalr-overview-scale-aspnet-core.md)** - 將 Azure SignalR 服務與 ASP.NET Core SignalR 應用程式整合，以相應放大為成千上萬個連線。
- **[建置無伺服器的即時應用程式](signalr-overview-azure-functions.md)** - 使用 Azure Functions 與 SignalR 服務的整合來以 JavaScript、C# 和 Java 等語言建置無伺服器的即時應用程式。
- **[透過 REST API 將訊息從伺服器傳送至用戶端](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md)** - Azure SignalR 服務提供 REST API，讓應用程式能以任何支援 REST 的程式設計語言，將訊息張貼到使用 SignalR 服務來連線的用戶端。

