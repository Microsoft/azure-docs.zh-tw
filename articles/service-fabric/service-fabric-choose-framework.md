---
title: Service Fabric 程式撰寫模型概觀 | Microsoft Docs
description: Service Fabric 提供兩種架構來建置服務：動作項目架構和服務架構。 它們在簡化與控制中提供不同的取捨。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: vturecek
ms.assetid: 974b2614-014e-4587-a947-28fcef28b382
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/02/2017
ms.author: vturecek
ms.openlocfilehash: f37e1ed2c7dd720e4a77076c8587fdf540b29ca5
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44713947"
---
# <a name="service-fabric-programming-model-overview"></a>Service Fabric 程式設計模型概觀
Service Fabric 提供多種撰寫和管理服務的方式。 服務可選擇使用 Service Fabric API 以善加運用平台的功能和應用程式架構。 服務也可以是以任何語言撰寫的任何已編譯可執行程式，或是在 Service Fabric 叢集所裝載之容器中執行的程式碼。

## <a name="guest-executables"></a>客體可執行檔
[客體可執行檔](service-fabric-guest-executables-introduction.md)是可在應用程式中作為服務執行的現有任意可執行檔 (以任何語言撰寫)。 客體可執行檔不直接呼叫 Service Fabric SDK API。 不過，它們仍然受惠於功能和平台供應項目，例如透過呼叫 Service Fabric 公開的 REST API 使用探索服務、自訂健康情況和負載報告。 它們也具備完整的應用程式生命週期支援。

部署您的第一個 [來賓可執行的應用程式](service-fabric-deploy-existing-app.md)，開始使用來賓可執行檔。

## <a name="containers"></a>容器
根據預設，Service Fabric 會以處理程序形式部署和啟用這些服務。 Service Fabric 也可以在[容器](service-fabric-containers-overview.md)中部署服務。 Service Fabric 支援在 Windows Server 2016 上部署 Linux 容器和 Windows 容器。 可以從任何容器存放庫提取容器映像，並部署至機器。 您可以在容器中部署現有的應用程式成為客體可執行檔、Service Fabric 無狀態或具狀態 Reliable Services，或 Reliable Actors，並且可以在同一個應用程式中混合使用處理序中的服務和容器中的服務。

[深入了解如何在 Windows 或 Linux 中將服務容器化](service-fabric-deploy-container.md)

## <a name="reliable-services"></a>Reliable Services
Reliable Services 是輕量級的服務撰寫架構，這些服務與 Service Fabric 平台整合，並得益於完整的平台功能。 Reliable Services 提供最基本的 API 集合，允許 Service Fabric 執行階段管理服務的生命週期，也允許服務與執行階段互動。 應用程式架構最為精簡，讓您完整掌控設計和實作選擇，而且可用來裝載任何其他應用程式架構，例如 ASP.NET Core。

Reliable Services 與大多數服務平台 (例如 Web 伺服器) 一樣，可以是無狀態的，其中每個服務執行個體都是以平等方式建立，並且狀態保存在外部解決方案中，例如 Azure DB 或「Azure 資料表儲存體」。

Reliable Services 也可以是具狀態且為 Service Fabric 專有，其狀態使用 Reliable Collections 直接保存在服務本身中。 狀態透過複寫變得高度可用，並透過資料分割散發，全由 Service Fabric 自動管理。

[深入了解 Reliable Services](service-fabric-reliable-services-introduction.md)或從[撰寫第一個 Reliable Services](service-fabric-reliable-services-quick-start.md) 開始。

## <a name="aspnet-core"></a>ASP.NET Core
ASP.NET Core 是新的開放原始碼和跨平台架構，可建置現代化雲端網際網路連線的應用程式，例如 web 應用程式、IoT 應用程式，以及行動後端。 Service Fabric 可與 ASP.NET Core 整合，因此您可以同時撰寫能夠利用可靠集合和 Service Fabric 的進階協調流程功能的無狀態與具狀態的 ASP.NET Core 應用程式。

[深入了解 Service Fabric 中的 ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md)，或從[撰寫第一個 ASP.NET Core Service Fabric 應用程式](service-fabric-tutorial-create-dotnet-app.md)開始使用。

## <a name="reliable-actors"></a>Reliable Actors
Reliable Actor 架構是建置在 Reliable Services 最上層的應用程式架構，其根據動作項目設計模式來實作 Virtual Actor 模式。 Reliable Actor 架構使用獨立的計算單位，和以單一執行緒方式執行稱為動作項目的狀態。 Reliable Actor 架構提供動作項目的內建通訊和預先設定狀態持續性和相應放大組態。

由於 Reliable Actors 本身是建置在 Reliable Services 上的應用程式架構，因此與 Service Fabric 平台完全整合，並可得益於平台提供的完整功能集。

[深入了解 Reliable Actors](service-fabric-reliable-actors-introduction.md) 或從[撰寫第一項 Reliable Actor 服務](service-fabric-reliable-actors-get-started.md)開始


[使用 ASP.NET Core 組建前端服務](service-fabric-reliable-services-communication-aspnetcore.md)

## <a name="next-steps"></a>後續步驟
[Service Fabric 和容器概觀](service-fabric-containers-overview.md)

[Reliable Services 概觀](service-fabric-reliable-services-introduction.md)

[Reliable Actors 概觀](service-fabric-reliable-actors-introduction.md)

[Service Fabric 和 ASP.NET Core ](service-fabric-reliable-services-communication-aspnetcore.md)




