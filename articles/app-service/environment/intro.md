---
title: Azure App Service Environment 簡介
description: Azure App Service Environment 簡要概觀
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: 3c7eaefa-1850-4643-8540-428e8982b7cb
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 04/19/2018
ms.author: ccompy
ms.custom: mvc
ms.openlocfilehash: 34c041bf96ecc8ce7f4440d7afaa64de775baef7
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576931"
---
# <a name="introduction-to-the-app-service-environments"></a>App Service 環境簡介 #
 
## <a name="overview"></a>概觀 ##

Azure App Service Environment 是 Azure App Service 的功能，可提供完全隔離和專用的環境，以便安全地大規模執行 App Service 應用程式。 此功能可以裝載您的：

* Windows Web 應用程式
* Linux Web 應用程式 
* Docker 容器
* 行動應用程式
* Functions

App Service Environment (ASE) 適合需要下列項目的應用程式工作負載：

* 非常高的延展性。
* 隔離和安全的網路存取。
* 高記憶體使用率。

客戶可以在單一 Azure 區域中或跨多個 Azure 區域建立多個 ASE。 這種彈性讓 ASE 很適合用於水平調整無狀態應用程式層的規模，以支援高 RPS 的工作負載。

ASE 已經過隔離，可執行只有單一客戶的應用程式，且一律會部署到虛擬網路。 客戶可以精確控制輸入和輸出的應用程式網路流量。 應用程式可以透過 VPN 建立內部部署公司資源的高速安全連線。

* ASE 隨附在自己的定價層中，了解[隔離式供應項目](https://channel9.msdn.com/Shows/Azure-Friday/Security-and-Horsepower-with-App-Service-The-New-Isolated-Offering?term=app%20service%20environment)如何有助於驅動超大規模作業及安全性。
* [App Service Environment v2](https://channel9.msdn.com/Blogs/Azure/Azure-Application-Service-Environments-v2-Private-PaaS-Environments-in-the-Cloud?term=app%20service%20environment) 提供一個環境來保護您網路子網路中的應用程式，並可讓您擁有私人的 Azure App Service 部署。
* 您可以使用多個 ASE 進行水平調整。 如需詳細資訊，請參閱[如何設定異地分散應用程式使用量](app-service-app-service-environment-geo-distributed-scale.md)。
* 您可以將 ASE 用於設定安全性架構，如 AzureCon 深入探討所示。 若要了解 AzureCon 深入探討中示範之安全性架構的設定方式，請參閱有關如何透過 App Service Environment [實行分層安全性架構的文章](app-service-app-service-environment-layered-security.md)。
* 在 ASE 中執行之應用程式的存取權可能會受到 Web 應用程式防火牆 (WAF) 等上游裝置的管制。 如需詳細資訊，請參閱 [Web 應用程式防火牆 (WAF)][AppGW]。

## <a name="dedicated-environment"></a>專用的環境 ##

ASE 以獨佔方式專屬於單一訂用帳戶，並可以裝載 100 個 App Service 方案執行個體。 不管是單一 App Service 方案中的 100 個執行個體或 100 個單一執行個體的 App Service 方案，只要加總之執行個體數在 100 以下皆可。

ASE 是由前端和背景工作角色所組成。 前端負責處理 HTTP/HTTPS 終止和 ASE 中應用程式要求的自動負載平衡。 前端會隨 ASE 中的 App Service 方案相應放大而自動新增。

背景工作角色是裝載客戶應用程式的角色。 背景工作角色可以三個固定的大小提供：

* 一個 vCPU/3.5 GB RAM
* 兩個 vCPU/7 GB RAM
* 四個 vCPU/14 GB RAM

客戶不需要管理前端和背景工作角色。 所有的基礎結構會隨客戶的 App Service 方案相應放大而自動新增。 隨著 App Service 方案建立或相應縮小 ASE，會視需要將基礎結構新增或移除。

ASE 會有一般每月費率來支付基礎結構，且不會依 ASE 的大小而變更。 此外，每個 App Service 方案 vCPU 核心都會有其成本。 ASE 中裝載的所有應用程式都會位於隔離價格 SKU 中。 如需有關 ASE 價格的資訊，請參閱 [App Service 價格][Pricing]頁面並檢閱 ASE 的可用選項。

## <a name="virtual-network-support"></a>虛擬網路支援 ##

ASE 功能是將 Azure App Service 直接部署到客戶 Azure Resource Manager 虛擬網路的部署。 若要深入了解 Azure 虛擬網路，請參閱 [Azure 虛擬網路常見問題集](https://azure.microsoft.com/documentation/articles/virtual-networks-faq/)。 ASE 一律存在於虛擬網路；更精確地說，是虛擬網路的子網路內。 您可以使用虛擬網路的安全性功能控制應用程式的輸入和輸出網路通訊。

ASE 可以是具有公用 IP 位址的網際網路對應，或只具有 Azure 內部負載平衡器 (ILB) 位址的內部對應。

[網路安全性群組][NSGs]會將輸入網路通訊限制於 ASE 所在的子網路。 您可以使用 NSG 在上游裝置和服務 (例如 WAF 和網路 SaaS 提供者) 背後執行應用程式。

應用程式也經常需要存取公司資源，例如內部資料庫和 Web 服務。 如果您在具有內部部署網路 VPN 連線的虛擬網路中部署 ASE，ASE 中的應用程式便可以存取內部部署資源。 無論 VPN 是[站對站](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-multi-site)或 [Azure ExpressRoute](http://azure.microsoft.com/services/expressroute/) VPN，此功能都可適用。

如需有關 ASE 與虛擬網路和內部部署網路搭配運作方式的詳細資訊，請參閱 [App Service Environment 的網路考量][ASENetwork]。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Application-Service-Environments-v2-Private-PaaS-Environments-in-the-Cloud/player]

## <a name="app-service-environment-v1"></a>App Service 環境 v1 ##

App Service Environment 有兩個版本：ASEv1 和 ASEv2。 前述資訊架構在 ASEv2 上。 本節說明 ASEv1 與 ASEv2 之間的差異。 

在 ASEv1 中，您必須手動管理所有資源。 其中包括前端、背景工作角色和用於 IP 型 SSL 的 IP 位址。 首先，您必須將想要在其中裝載的背景工作角色集區相應放大，才能相應放大 App Service 方案。

ASEv1 使用與 ASEv2 不同的定價模式。 在 ASEv1 中，您需要支付每個配置的 vCPU。 其中包括用於前端或未裝載任何工作負載之背景工作角色的 vCPU。 在 ASEv1 中，ASE 的預設最大調整大小總計是 55 個主機， 包括背景工作角色與前端。 ASEv1 的其中一個優點，是可以部署在傳統虛擬網路和 Resource Manager虛擬網路中。 若要深入了解 ASEv1，請參閱 [App Service Environment v1 簡介][ASEv1Intro]。

<!--Links-->
[App Service Environments v2]: https://channel9.msdn.com/Blogs/Azure/Azure-Application-Service-Environments-v2-Private-PaaS-Environments-in-the-Cloud?term=app%20service%20environment
[Isolated offering]: https://channel9.msdn.com/Shows/Azure-Friday/Security-and-Horsepower-with-App-Service-The-New-Isolated-Offering?term=app%20service%20environment
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/security-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[webapps]: ../app-service-web-overview.md
[mobileapps]: ../../app-service-mobile/app-service-mobile-value-prop.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: http://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/resource-group-overview.md
[ConfigureSSL]: ../web-sites-purchase-ssl-web-site.md
[Kudu]: http://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../application-gateway/waf-overview.md
