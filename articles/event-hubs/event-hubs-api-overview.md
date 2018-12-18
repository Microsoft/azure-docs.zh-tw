---
title: Azure 事件中樞 API 概觀 | Microsoft Docs
description: 可用的 Azure 事件中樞 API 概觀
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: 3f221a0c-182d-4e39-9f3d-3a3c16c5c6ed
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/02/2018
ms.author: shvija
ms.openlocfilehash: 2523db3d817b4281b6435374ef0cca4e1d2da695
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005852"
---
# <a name="available-event-hubs-apis"></a>可用的事件中樞 API

本文說明一組可用來管理事件中樞資源的可用 API 用戶端。

## <a name="runtime-apis"></a>執行階段 API

下一節說明所有目前可用的 Azure 事件中樞執行階段用戶端。 雖然這些程式庫其中也包括有限的管理功能，但也有專用於管理作業的[特定程式庫](#management-apis)。 這些程式庫的核心重點是從事件中樞傳送和接收訊息。

如需與每個執行階段程式庫的目前狀態有關的詳細資訊，請參閱[其他資訊](#additional-information)。

| 語言/平台 | 用戶端封裝 | EventProcessorHost 封裝 | 存放庫 |
| --- | --- | --- | --- |
| .NET Standard | [NuGet](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) | [NuGet](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor/) | [GitHub](https://github.com/azure/azure-event-hubs-dotnet) |
| .NET Framework | [NuGet](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) | [NuGet](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/) | N/A |
| Java | [Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22) | [Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22) | [GitHub](https://github.com/Azure/azure-event-hubs-java) |
| 節點 | [NPM](https://www.npmjs.com/package/azure-event-hubs) | N/A | [GitHub](https://github.com/Azure/azure-event-hubs-node) |
| C | N/A | N/A | [GitHub](https://github.com/Azure/azure-event-hubs-c) |

### <a name="additional-information"></a>其他資訊

#### <a name="net"></a>.NET

.NET 生態系統有多個執行階段，因此有多個適用於事件中樞的 .NET 程式庫。 .NET Standard 程式庫可使用 .NET Core 或 .NET Framework 執行，然而 .NET Framework 程式庫只能在 .NET Framework 環境中執行。 如需 .NET Framework 版本的詳細資訊，請參閱[架構版本](https://docs.microsoft.com/dotnet/articles/standard/frameworks#framework-versions)。

#### <a name="node"></a>節點

[Node.js 程式庫](https://github.com/Azure/azure-event-hubs-node)目前為預覽版，並且由 Microsoft 員工和外部參與者以業餘專案的形式維護。 我們非常歡迎所有的參與，包括原始程式碼，而且都會加以檢閱。

## <a name="management-apis"></a>管理 API

下表列出目前所有可用的管理特定程式庫。 這些程式庫都不包含執行階段作業，而且唯一的用途是管理事件中樞項目。

| 語言/平台 | 管理封裝 | 存放庫 |
| --- | --- | --- | --- |
| .NET Standard | [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Management.EventHub) | [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/ResourceManagement/EventHub) |

## <a name="next-steps"></a>後續步驟
您可以造訪下列連結以深入了解事件中樞︰

* [事件中心概觀](event-hubs-what-is-event-hubs.md)
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)