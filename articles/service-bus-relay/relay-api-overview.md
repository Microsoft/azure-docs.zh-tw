---
title: Azure 轉送 API 概觀 | Microsoft Docs
description: 可用的 Azure 轉送 API 概觀
services: event-hubs
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: fdaa1d2b-bd80-4e75-abb9-0c3d0773af2d
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/02/2018
ms.author: spelluru
ms.openlocfilehash: 2c7aec700a1077bc1a1b56afb5d8d07f47f4c6e8
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43700367"
---
# <a name="available-relay-apis"></a>可用的轉送 API

## <a name="runtime-apis"></a>執行階段 API

下表列出所有目前可用的轉送執行階段用戶端。

[更多資訊](#additional-information)章節包含每個執行階段程式庫之狀態的詳細資訊。

| 語言/平台 | 可用的功能 | 用戶端封裝 | 存放庫 |
| --- | --- | --- | --- |
| .NET Standard | 混合式連線 | [Microsoft.Azure.Relay](https://www.nuget.org/packages/Microsoft.Azure.Relay/) | [GitHub](https://github.com/azure/azure-relay-dotnet) |
| .NET Framework | WCF 轉送 | [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) | N/A |
| 節點 | 混合式連線 | [Websocket：`hyco-ws`](https://www.npmjs.com/package/hyco-ws)<br/>[Websocket：`hyco-websocket`](https://www.npmjs.com/package/hyco-websocket)<br/>[HTTP 要求：`hyco-https`](https://www.npmjs.com/package/hyco-https) | [GitHub](https://github.com/Azure/azure-relay-node) |

### <a name="additional-information"></a>其他資訊

#### <a name="net"></a>.NET

.NET 生態系統有多個執行階段，因此有多個適用於轉送的 .NET 程式庫。 .NET Standard 程式庫可使用 .NET Core 或 .NET Framework 執行，然而 .NET Framework 程式庫只能在 .NET Framework 環境中執行。 如需 .NET Framework 的詳細資訊，請參閱[架構版本](/dotnet/articles/standard/frameworks#framework-versions)。

.NET Framework 程式庫只支援 WCF 程式設計模型，並且會依賴 WCF `net.tcp` 傳輸型的專屬二進位通訊協定。 此通訊協定和程式庫會保有適用於現有應用程式的回溯相容性。

.NET Standard 程式庫是以開放式通訊協定定義為基礎，該定義為根據 HTTP 和 WebSocket 所建置的混合式連線轉送所有。 此程式庫支援透過 Websocket 的資料流抽取，以及用於回應 HTTP 要求的簡單要求-回應 API 手勢。 [Web API](https://github.com/Azure/azure-relay-dotnet) 範例會說明如何整合混合式連線與適用於 Web 服務的 ASP.NET Core。

#### <a name="nodejs"></a>Node.js

上表所列出的混合式連線模組，會以接聽 Azure 轉送服務而非本機網路堆疊的替代實作，來取代或修改現有的 Node.js 模組。

`hyco-https` 模組會修改並局部覆寫核心的 Node.js 模組 `http` 和 `https`，提供可與許多依賴這些核心模組的現有 Node.js 模組和應用程式相容的 HTTPS 接聽程式實作。

`hyco-ws` 和 `hyco-websocket` 模組會修改熱門的 Node.js `ws` 和 `websocket` 模組，提供可讓模組和依賴任一模組的應用程式在混合式連線轉送背後運作的替代接聽程式實作。

這些模組的相關詳細資料可於 [azure-relay-node](https://github.com/Azure/azure-relay-node) GitHub 存放庫中找到。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 轉送，請造訪下列連結：
* [什麼是 Azure 轉送？](relay-what-is-it.md)
* [轉送常見問題集](relay-faq.md)