---
title: 使用 Azure SignalR 調整 ASP.NET Core SignalR | Microsoft Docs
description: 使用 Azure SignalR 服務調整 ASP.NET Core SignalR 應用程式的概觀。
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
ms.openlocfilehash: af100c015845d095f39ee4245f71689a4d2a43c0
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49384985"
---
# <a name="scale-aspnet-core-signalr-applications-with-azure-signalr-service"></a>使用 Azure SignalR 服務調整 ASP.NET Core SignalR 應用程式

## <a name="developing-signalr-apps"></a>開發 SignalR 應用程式

目前，有[兩個版本](https://docs.microsoft.com/aspnet/core/signalr/version-differences)的 SignalR 可供您搭配 Web 應用程式使用：「適用於 ASP.NET 的 SignalR」和「ASP.NET Core SignalR」，後者為最新版本。 Azure SignalR 服務是建置在 ASP.NET Core SignalR 上的 Azure 受控服務。 

ASP.NET Core SignalR 是前一個版本重新撰寫的版本。 因此，ASP.NET Core SignalR 與先前版本的 SignalR 不回溯相容。 API 和行為不同。 ASP.NET Core SignalR SDK 是以 .NET Standard 為目標，所以您仍然可以搭配 .NET Framework 使用它。 不過，您必須使用新的 API，而不是舊的。 如果您是使用 SignalR，且想要移至 ASP.NET Core SignalR 或 Azure SignalR 服務，則必須變更程式碼來處理 API 中的差異。

使用 Azure SignalR 服務時，伺服器端的 ASP.NET Core SignalR 元件是裝載在 Azure 中。 不過，因為該技術是建置在 ASP.NET Core 之上，所以您能夠在多個平台上 (Windows、Linux 和 MacOS) 執行實際的 Web 應用程式，同時使用 [Azure App Service](../app-service/app-service-web-overview.md)、[IIS](https://docs.microsoft.com/aspnet/core/host-and-deploy/iis/index)、[Nginx](https://docs.microsoft.com/aspnet/core/host-and-deploy/linux-nginx)、[Apache](https://docs.microsoft.com/aspnet/core/host-and-deploy/linux-apache)、[Docker](https://docs.microsoft.com/aspnet/core/host-and-deploy/docker/index) 裝載。 您也可以在自己的處理序中使用自我裝載。

如果您應用程式的目標包括：支援使用即時內容更新以更新 Web 用戶端的最新功能、跨多個平台 (Azure、Windows、Linux 和 macOS) 執行，以及裝載於不同的環境中，則 Azure SignalR 服務就是您可以利用的最佳選擇。


## <a name="why-not-deploy-signalr-myself"></a>為何不自行部署 SignalR？

自行部署支援 ASP.NET Core SignalR 的 Azure 應用程式，作為您整體 Web 應用程式的後端元件仍是有效的方法。

使用 Azure SignalR 服務的其中一個主要原因是簡單。 使用 Azure SignalR 服務，您不需要處理效能、延展性、可用性等問題。 該服務以 99.9% 的服務等級協定為您處理這些問題。

此外，WebSocket 通常是支援即時內容更新所偏好使用的技術。 不過，隨著您調整，對大量持續的 WebSocket 連線進行負載平衡變成要解決的複雜問題。 常見的解決方案利用：DNS 負載平衡、硬體負載平衡器和軟體負載平衡。 Azure SignalR 服務會為您處理此問題。

另一個原因可能是您完全不需要實際上裝載一個 Web 應用程式。 您 Web 應用程式的邏輯可能會利用[無伺服器運算](https://azure.microsoft.com/overview/serverless-computing/)。 舉例來說，或許您的程式碼只在需要時才使用 [Azure 函式](https://docs.microsoft.com/azure/azure-functions/)觸發程序裝載和執行。 這種情況可能變得棘手，因為您的程式碼在需要時才執行，而且不需要維持長時間用戶端連線。 Azure SignalR 服務可以處理這種情況，因為該服務已經為您管理連線。 如需詳細資訊，請參閱[如何使用 SignalR 服務與 Azure Functions 概觀](signalr-overview-azure-functions.md)。 

## <a name="how-does-it-scale"></a>它如何調整？

SignalR 通常隨 SQL Server、Azure 服務匯流排或 Redis 快取調整。 Azure SignalR 服務會為您處理調整方式。 效能和成本與這些方法相當，但沒有處理這些其他服務的複雜性。 您只需要更新服務的單位計數即可。 每個單位最高支援 1000 個用戶端連線。

## <a name="next-steps"></a>後續步驟
* [快速入門：使用 Azure SignalR 建立聊天室](signalr-quickstart-dotnet-core.md)  
  

