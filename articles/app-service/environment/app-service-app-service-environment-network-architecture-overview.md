---
title: App Service 環境的網路架構概觀
description: App Service 環境網路拓撲的架構概觀。
services: app-service
documentationcenter: ''
author: stefsch
manager: erikre
editor: ''
ms.assetid: 13d03a37-1fe2-4e3e-9d57-46dfb330ba52
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/04/2016
ms.author: stefsch
ms.openlocfilehash: 3362a55524da42914681db06b8d2c0da8df773d8
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22986773"
---
# <a name="network-architecture-overview-of-app-service-environments"></a>App Service 環境的網路架構概觀
## <a name="introduction"></a>簡介
App Service 環境一律建立於[虛擬網路][virtualnetwork] - 的子網路內，而在 App Service 環境中執行的應用程式可以與相同虛擬網路拓撲內的私用端點通訊。  因為客戶可能會鎖定其虛擬網路基礎結構的組件，所以請務必了解與 App Service 環境發生的網路通訊流程類型。

## <a name="general-network-flow"></a>一般網路流程
當 App Service 環境 (ASE) 針對應用程式使用公開虛擬 IP 位址 (VIP) 時，所有傳入的流量都會到達該公用 VIP。  這包括應用程式的 HTTP 和 HTTPS 流量，以及 FTP 的其他流量、遠端偵錯功能和 Azure 管理作業。  如需公用 VIP 上可用特定連接埠 (必要和選擇性) 的完整清單，請參閱有關[控制輸入流量][controllinginboundtraffic]至 App Service 環境的文章。 

App Service 環境也支援執行僅繫結至虛擬網路內部位址，也稱為 ILB (內部負載平衡器) 位址的應用程式。  在已啟用 ILB 的 ASE 上，應用程式的 HTTP 和 HTTPS 流量以及遠端偵錯呼叫會送達 ILB 位址。  針對大部分的 ILB-ASE 組態，FTP/FTPS 流量也會送達 ILB 位址。  不過，Azure 管理作業仍會流向公用 VIP (屬於已啟用 ILB 的 ASE) 上的連接埠 454/455。

下圖顯示 App Service 環境的各種傳入和傳出網路流量的概觀，其中應用程式是繫結至公用虛擬 IP 位址：

![一般網路流程][GeneralNetworkFlows]

App Service 環境可以與各種私用客戶端點進行通訊。  例如，在 App Service 環境中執行的應用程式可以連線至在相同虛擬網路拓撲之 IaaS 虛擬機器上執行的資料庫伺服器。

> [!IMPORTANT]
> 請查看網狀圖，「其他計算資源」會部署在與 App Service 環境不同的子網路中。 將資源部署於和 ASE 相同的子網路中，會封鎖從 ASE 連線到這些資源的連線 (除了特定的內部 ASE 路由之外)。 請改為部署到 (相同 VNET 中) 不同的子網路。 App Service 環境就能連線。 不需要任何其他設定。
> 
> 

App Service 環境也會與管理和操作 App Service 環境所需的 SQL 資料庫和 Azure 儲存體資源進行通訊。  App Service 環境與之通訊的一些 SQL 和儲存體資源位於與 App Service 環境相同的區域中，有些則位於遠端 Azure 區域中。  因此，一律需要到網際網路的輸出連線，App Service 環境才能正常運作。 

因為在子網路中部署 App Service 環境，所以可以使用網路安全性群組來控制對子網路的輸入流量。  如需如何控制對 App Service 環境之輸入流量的詳細資料，請參閱下列[文章][controllinginboundtraffic]。

如需如何允許來自 App Service 環境之輸出網際網路連線的詳細資料，請參閱下列有關使用 [Express Route][ExpressRoute] 的文章。  本文所述的相同方法適用於使用站對連線以及使用強制通道時。

## <a name="outbound-network-addresses"></a>輸出網路位址
App Service 環境進行輸出呼叫時，IP 位址一律會與輸出呼叫相關聯。  使用的特定 IP 位址取決於所呼叫的端點位於虛擬網路拓撲內部還是外部。

如果所呼叫的端點是在虛擬網路拓撲 **外部** ，則使用的輸出位址 (也稱為輸出 NAT 位址) 是 App Service 環境的公用 VIP。  您可以在 App Service 環境的入口網站的使用者介面中找到此位址 ([屬性] 刀鋒視窗)。

![輸出 IP 位址][OutboundIPAddress]

透過在 App Service 環境中建立應用程式，然後在應用程式的位址上執行 *nslookup* ，也可以針對僅具有公用 VIP 的 ASE 來決定此位址。 產生的 IP 位址是公用 VIP，也是 App Service 環境的輸出 NAT 位址。

如果所呼叫的端點是在虛擬網路拓撲 **內部** ，則呼叫端應用程式的輸出位址會是執行應用程式之個別計算資源的內部 IP 位址。  不過，虛擬網路內部 IP 位址與應用程式不會有持續性的對應。  應用程式可以在不同的計算資源之間移動，而且可以基於調整作業而變更 App Service 環境中的可用計算資源集區。

不過，因為 App Service 環境一律位在子網路內，所以執行應用程式之計算資源的內部 IP 位址保證一律落在子網路的 CIDR 範圍內。  因此，使用更細緻的 ACL 或網路安全性群組來保護虛擬網路內其他端點的存取時，需要將存取權授與包含 App Service 環境的子網路範圍。

下圖更詳細說明這些概念：

![輸出網路位址][OutboundNetworkAddresses]

在上圖中：

* 因為 App Service 環境的公用 VIP 是 192.23.1.2，所以這是呼叫「網域網路」端點時所使用的輸出 IP 位址。
* App Service 環境所含子網路的 CIDR 範圍是 10.0.1.0/26。  相同虛擬網路基礎結構內的其他端點，將會看到源自此位址範圍內某處之應用程式的呼叫。

## <a name="calls-between-app-service-environments"></a>App Service 環境之間的呼叫
如果您在相同的虛擬網路中部署多個 App Service 環境，並從一個 App Service 環境傳出呼叫至另一個 App Service 環境，則可能產生更複雜的案例。  這些跨 App Service 環境的呼叫也會被視為「網際網路」呼叫。

下圖顯示分層架構範例，其中應用程式在一個 App Service 環境上 (例如「前端」Web 應用程式) 呼叫第二個 App Service 環境上的應用程式 (例如：內部後端 API 應用程式不需要可從網際網路存取)。 

![App Service 環境之間的呼叫][CallsBetweenAppServiceEnvironments] 

在上述範例中，App Service 環境 "ASE One" 有一個輸出 IP 位址 192.23.1.2。  如果 App Service 環境上執行的應用程式，對位於相同虛擬網路中第二個 App Service 環境 ("ASE Two") 上執行的應用程式進行輸出呼叫，會將輸出呼叫視為「網際網路」呼叫。  因此，到達第二個 App Service 環境的網路流量會將來源顯示為來自 192.23.1.2 (亦即，不是第一個 App Service 環境的子網路位址範圍)。

即使不同 App Service 環境之間的呼叫會視為「網際網路」呼叫，當兩個 App Service 環境同時位於相同的 Azure 區域時，網路流量會維持在 Azure 區域網路上，而不會實際在公用網際網路流動。  因此，您可以使用第二個 App Service 環境的子網路上的網路安全性群組，僅允許來自第一個 App Service (其傳入 IP 位址為 192.23.1.2) 的傳入呼叫，因而確保 App Service 環境之間的通訊安全。

## <a name="additional-links-and-information"></a>其他連結和資訊
如需 App Service 環境所使用輸入連接埠以及使用網路安全性群組來控制輸入流量的詳細資料，請參閱[這裡][controllinginboundtraffic]。

如需利用使用者定義路徑來授與到 App Service 環境之輸出網際網路存取的詳細資料，請參閱本[文章][ExpressRoute]。 

<!-- LINKS -->
[virtualnetwork]: http://azure.microsoft.com/services/virtual-network/
[controllinginboundtraffic]:  app-service-app-service-environment-control-inbound-traffic.md
[ExpressRoute]:  app-service-app-service-environment-network-configuration-expressroute.md

<!-- IMAGES -->
[GeneralNetworkFlows]: ./media/app-service-app-service-environment-network-architecture-overview/NetworkOverview-1.png
[OutboundIPAddress]: ./media/app-service-app-service-environment-network-architecture-overview/OutboundIPAddress-1.png
[OutboundNetworkAddresses]: ./media/app-service-app-service-environment-network-architecture-overview/OutboundNetworkAddresses-1.png
[CallsBetweenAppServiceEnvironments]: ./media/app-service-app-service-environment-network-architecture-overview/CallsBetweenEnvironments-1.png

