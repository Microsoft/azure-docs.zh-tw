---
title: 監視 Azure 容器概觀 | Microsoft Docs
description: 本文提供在 Azure 中可用來監視 Azure 容器的不同方法概觀，以快速了解叢集健康情況與可用性。
services: log-analytics
documentationcenter: ''
author: MGoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: magoedte
ms.openlocfilehash: db85f85011154dcc7adfa9d569e9015a9c5c33ca
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47055040"
---
# <a name="overview-of-monitoring-containers-in-azure"></a>在 Azure 中監視容器的概觀
使用 Azure，您可以有效地監視和管理執行 Kubernetes 或 Docker 的 Azure 容器上所部署的工作負載。 務必要了解具有多個微服務應用程式的容器如何執行，以大規模提供可靠服務並支援監視計劃。 本文提供 Azure 中管理和監視功能的簡短概觀，協助您了解這些功能，以及適合您需求的功能。

使用[適用於容器的 Azure 監視器](monitoring-container-insights-overview.md)，您可以快速檢視 Linux 容器基礎結構的效能與健康情況，並迅速調查問題。 遙測資料儲存在 Log Analytics 工作區並整合於 Azure 入口網站中，您可以在此處透過儀表板探索、篩選及分割彙總資料，留意負載、效能及健康情況。  

對於在裝載的 Azure Kubernetes 服務外部執行的容器，Log Analytics [Windows 與 Docker 容器解決方案](../log-analytics/log-analytics-containers.md)可協助您檢視和管理 Windows 與 Docker 容器主機。 您可以從 Log Analytics 工作區檢視清查環境中節點與容器的詳細資料、效能及事件。 您可以檢視詳細的稽核資訊，顯示和容器搭配使用的命令，且可以藉由檢視和搜尋集中式記錄檔，對容器進行疑難排解，完全不必從遠端存取 Docker 或 Windows 主機。

若要達成監視整體或端對端應用程式，應該使用 Azure 監視器或 Log Analytics 監視任何相依性，不論是 Azure 或內部部署資源。  應將應用程式層包含在內，才能使用 Application Insights 在平台與應用程式層級，另外添加一層健康情況感知。 在平台層級，有 Application Insights SDK 可用於 [Kubernetes]( https://github.com/Microsoft/ApplicationInsights-Kubernetes)、[Docker](https://hub.docker.com/r/microsoft/applicationinsights/) 及 [Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-analysis-appinsights)。 對於微服務應用程式，支援 [Java](../application-insights/app-insights-java-get-started.md)、[Node.js](../application-insights/app-insights-nodejs-quick-start.md)、[.Net](../application-insights/app-insights-asp-net.md)、[.Net Core](../application-insights/app-insights-asp-net-core.md)，以及一些其他[語言/架構](../application-insights/app-insights-platforms.md)。 

否則會無法辨識問題，而影響應用程式的可用性，而且無法達到服務等級目標。  
