---
title: App Service 環境 v1 簡介
description: 了解可提供安全、VNet 聯結、專用延展單位的 App Service 環境 v1 功能，以便執行您所有的應用程式。
services: app-service
documentationcenter: ''
author: stefsch
manager: erikre
editor: ''
ms.assetid: 78e6d4f5-da46-4eb5-a632-b5fdc17d2394
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.openlocfilehash: a2efb3177a97aa362a269ab3a00239a0d391f026
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45574686"
---
# <a name="introduction-to-app-service-environment-v1"></a>App Service 環境 v1 簡介

> [!NOTE]
> 這篇文章是關於 App Service 環境 v1。  有較新版本的 App Service 環境，更易於使用，並且可以在功能更強大的基礎結構上執行。 若要深入了解新版本，請從 [App Service 環境簡介](intro.md)開始。
> 

## <a name="overview"></a>概觀
App Service 環境是 [Azure App Service](../app-service-web-overview.md) 的[進階][PremiumTier]服務方案選項，提供完全隔離的專用環境，能夠極為安全地執行 Azure App Service 應用程式，包括 Web Apps、Mobile Apps 和 API Apps。  

適合應用程式工作負載的 App Service 環境需要：

* 非常高的延展性
* 隔離和安全的網路存取

客戶可以在單一 Azure 區域，以及跨多個 Azure 區域中建立多個 App Service 環境。  這使得 App Service 環境很適合用來水平調整無狀態應用程式層的規模，以支援高 RPS 工作負載。

App Service 環境已經過隔離，可執行只有單一客戶的應用程式，且一律會部署到虛擬網路。  客戶對於輸入和輸出的應用程式網路流量都有更細微的控制，且應用程式可以透過虛擬網路建立與內部部署公司資源的高速安全連線。

如需 App Service Environment 如何提供高延展性和安全的網路存取的概觀，請參閱關於 App Service Environment 的 [AzureCon 深入探討][AzureConDeepDive]！

如需使用多個 App Service Environment 水平延展的深入探討，請參閱關於如何設定[地理位置分散的應用程式使用量][GeodistributedAppFootprint]一文。

若要查看 AzureCon Deep Dive 中顯示之安全性架構的設定方式，請參閱有關使用 App Service Environment 實作 [分層安全性架構](app-service-app-service-environment-layered-security.md) 的文章。

在 App Service 環境中執行之應用程式的存取權可能會受到 Web 應用程式防火牆 (WAF) 等上游裝置的管制。  [設定 App Service Environment 的 WAF](app-service-app-service-environment-web-application-firewall.md) 上的文章將說明這種情況。 

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="dedicated-compute-resources"></a>專用計算資源
App Service Environment 中的所有計算資源皆專屬於單一訂用帳戶，且 App Service Environment可以設定最多五十 (50) 個計算資源，讓單一應用程式獨佔使用。

App Service Environment 是由前端計算資源集區，以及一到三個背景工作計算資源集區所組成。 

前端集區包含負責處理 SSL 終止以及 App Service Environment 中應用程式要求的自動負載平衡的計算資源。 

每個背景工作集區都含有配置給 [App Service 方案][AppServicePlan]的計算資源，其中又包含一或多個 Azure App Service 應用程式。  因為 App Service Environment 中可有多達三個不同的背景工作集區，所以您有彈性可為每個背景工作集區選擇不同的計算資源。  

比方說，您可以針對主要用於開發或測試應用程式的 App Service 方案，建立一個計算資源較不強大的背景工作集區。  第二個 (或甚至第三個) 背景工作集區可以使用比較強大的運算資源，以供 App Service 方案執行生產應用程式。

如需前端和背景工作集區可用計算資源數量的詳細資訊，請參閱[如何設定 App Service Environment][HowToConfigureanAppServiceEnvironment]。  

如需 App Service Environment 中支援的可用計算資源大小的詳細資訊，請參閱 [App Service 定價][AppServicePricing]頁面，並檢閱 Premium 定價層中 App Service Environment可用的選項。

## <a name="virtual-network-support"></a>虛擬網路支援
App Service Environment 可以在 Azure Resource Manager 虛擬網路或者傳統式部署模型虛擬網路其中之一中建立 ([更多有關虛擬網路的資訊][MoreInfoOnVirtualNetworks])。  因為 App Service Environment 一律存在於虛擬網路中，而且更精確來說是在虛擬網路的子網路內，所以您可以運用虛擬網路的安全性功能來控制傳入和傳出網路通訊。  

App Service Environment 可以是具有公用 IP 位址的網際網路對向，或只具有 Azure 內部負載平衡器 (ILB) 位址的內部對向。

您可以使用[網路安全性群組][NetworkSecurityGroups]將傳入網路通訊限制為 App Service Environment 所在的子網路。  這可讓您在上游裝置和服務 (例如 Ｗeb 應用程式防火牆和網路 SaaS 提供者) 背後執行應用程式。

應用程式也經常需要存取公司資源，例如內部資料庫和 Web 服務。  常見的方法是讓這些端點僅可用於在 Azure 虛擬網路中傳送的內部網路流量。  一旦 App Service Environment 加入與內部服務相同的虛擬網路，在此環境中執行的應用程式即可存取這些內部服務，包括可透過[站台對站台][SiteToSite]和 [Azure ExpressRoute][ExpressRoute] 連線聯繫的端點。

如需 App Service Environment 如何搭配虛擬網路和內部部署網路使用的詳細資訊，請參閱下列文章：[網路架構][NetworkArchitectureOverview]、[控制輸入流量][ControllingInboundTraffic]和[安全地連接到後端][SecurelyConnectingToBackends]。 

## <a name="getting-started"></a>開始使用
若要開始使用 App Service 環境，請參閱[如何建立 App Service 環境][HowToCreateAnAppServiceEnvironment]

如需 App Service Environment 網路架構的概觀，請參閱[網路架構概觀][NetworkArchitectureOverview]一文。

如需搭配 ExpressRoute 使用 App Service Environment 的詳細資訊，請參閱 [Express Route 與 App Service Environment][NetworkConfigDetailsForExpressRoute]一文。

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[PremiumTier]: http://azure.microsoft.com/pricing/details/app-service/
[MoreInfoOnVirtualNetworks]: https://azure.microsoft.com/documentation/articles/virtual-networks-faq/
[AppServicePlan]: ../azure-web-sites-web-hosting-plans-in-depth-overview.md
[HowToCreateAnAppServiceEnvironment]: app-service-web-how-to-create-an-app-service-environment.md
[LogicApps]: http://azure.microsoft.com/documentation/articles/app-service-logic-what-are-logic-apps/
[AzureConDeepDive]:  https://azure.microsoft.com/documentation/videos/azurecon-2015-deploying-highly-scalable-and-secure-web-and-mobile-apps/
[GeodistributedAppFootprint]:  app-service-app-service-environment-geo-distributed-scale.md
[NetworkSecurityGroups]: https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[SiteToSite]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-multi-site
[ExpressRoute]: http://azure.microsoft.com/services/expressroute/
[HowToConfigureanAppServiceEnvironment]:  app-service-web-configure-an-app-service-environment.md
[ControllingInboundTraffic]:  app-service-app-service-environment-control-inbound-traffic.md
[SecurelyConnectingToBackends]:  app-service-app-service-environment-securely-connecting-to-backend-resources.md
[NetworkArchitectureOverview]:  app-service-app-service-environment-network-architecture-overview.md
[NetworkConfigDetailsForExpressRoute]:  app-service-app-service-environment-network-configuration-expressroute.md
[AppServicePricing]: http://azure.microsoft.com/pricing/details/app-service/ 

<!-- IMAGES -->


