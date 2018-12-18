---
title: 關於 Log Analytics 中的網路監視 | Microsoft Docs
description: 可管理跨雲端、內部部署和混合式環境之網路的網路監視解決方案 (包括 NPM) 概觀。
services: monitoring-and-diagnostics
documentationcenter: na
author: agummadi
manager: ''
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: monitoring-and-diagnostics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2018
ms.author: ajaycode
ms.openlocfilehash: 0656cfcc2dcded284be1a337f797681117f3b313
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42140494"
---
# <a name="network-monitoring-solutions"></a>網路監視解決方案 

Azure 提供許多可監視網路資產的解決方案。 Azure 擁有的解決方案和公用程式可監視網路連線、ExpressRoute 線路的健康情況以及分析雲端中的網路流量。

## <a name="network-performance-monitor-npm"></a>網路效能監視器 (NPM)

網路效能監控 (NPM) 是一套功能，其中的每項功能都可供監視網路的健康情況、應用程式的網路連線，以及提供網路效能的深入解析。 NPM 是以雲端作為基礎，所提供的網路監視解決方案可監視以下兩者之間的連線：
 
* 雲端部署和內部部署位置
* 多個資料中心和分公司
* 任務關鍵性多層應用程式/微服務
* 使用者位置和 Web 架構應用程式 (HTTP/HTTPS) 

效能監視器、ExpressRoute 監視器及服務連線能力監視器都是 NPM 內的監視功能，如下所述。

## <a name="performance-monitor"></a>效能監視器

效能監控是 NPM 的一部分，可供監視雲端、混合式和內部部署環境的網路。 您可以監視遠端分公司和辦事處、商店位置、資料中心和雲端之間的網路連線。 您可以在使用者提出抱怨之前偵測到網路問題。 主要優點如下：

* 監視各種子網路的遺失和延遲並設定警示
* 監視網路上的所有路徑 (包括備援路徑)
* 針對難以複寫的暫時性和時間點網路問題進行疑難排解
* 判斷導致效能衰退的特定網路區段
* 監視網路健康情況，而不需 SNMP

![NPM 拓撲地圖](./media/network-monitoring-overview/npm-topology-map.png) 

如需詳細資訊，請觀看下列文章：

* [在 Log Analytics 中設定網路效能監控解決方案](../log-analytics/log-analytics-network-performance-monitor.md) 
* [使用案例](https://blogs.technet.microsoft.com/msoms/2016/08/30/monitor-on-premises-cloud-iaas-and-hybrid-networks-using-oms-network-performance-monitor/)
*  產品更新：[2017 年 2 月](https://blogs.technet.microsoft.com/msoms/2017/02/27/oms-network-performance-monitor-is-now-generally-available/)、[2017 年 8 月](https://blogs.technet.microsoft.com/msoms/2017/08/14/improvements-to-oms-network-performance-monitor/)

## <a name="expressroute-monitor"></a>ExpressRoute 監視

Expressroute 的 NPM 提供對於 Azure 私用對等連線和 Microsoft 對等連線的全面性 ExpressRoute 監視。 您可以透過 ExpressRoute 監視分公司與 Azure 之間的 E2E 連線和效能。 主要功能有：

* 自動偵測與您的訂用帳戶相關聯的 ER 線路
* 偵測從內部部署至雲端應用程式的網路拓撲
* 容量規劃、頻寬使用率分析
* 監視主要和次要路徑並且提出警示
* 透過 ExpressRoute 監視與 Azure 服務 (例如 Office 365、Dynamics 365 等) 的連線
* 偵測 VNet 連線能力的降低

![顯示跨區域流量的地理資訊地圖](./media/network-monitoring-overview/expressroute-topology-map.png) 

如需詳細資訊，請參閱下列文章：

* [設定 ExpressRoute 線路的網路效能監視器](../expressroute/how-to-npm.md)
* [部落格文章](https://aka.ms/NPMExRmonitorGA)

## <a name="service-connectivity-monitor"></a>服務連線能力監視

您現在可以使用服務連線能力監視，測試應用程式的連線能力，並跨內部部署、電訊廠商網路和雲端/私用資料中心偵測效能瓶頸。

* 監視對應用程式的端對端網路連線
* 將應用程式傳遞與網路效能相互關聯，沿著使用者和應用程式之間的路徑偵測效能降低的確切位置
* 從全球多個使用者位置測試應用程式連線能力
* 確定企業營運與 SaaS 應用程式的網路延遲和封包遺失
* 確定網路上可能會造成應用程式效能不佳的作用點
* 使用針對 Microsoft Office 365、Dynamics 365、商務用 Skype 和其他 Microsoft 服務所內建的測試，監視對 Office 365 應用程式的連線能力

如需詳細資訊，請參閱下列文章：

* [設定網路效能監控來監視服務端點](https://aka.ms/applicationconnectivitymonitorguide)
* [部落格文章](https://aka.ms/svcendptmonitor)

## <a name="traffic-analytics"></a>流量分析
流量分析是一項雲端式解決方案，可顯示雲端網路上的使用者和應用程式活動。 系統會分析 NSG 流量記錄，讓您深入了解：

* 您網路中 Azure 與網際網路、公用雲端區域、VNET 及子網路之間的流量
* 您網路上的應用程式和通訊協定，而不需要 Sniffer 或專用的流量收集器設備
* 發言最多者、通訊流量高的應用程式、雲端中的 VM 對話、流量作用區
* 跨 VNET 的流量來源和目的地，關鍵業務服務和應用程式之間的相互關係
* 安全性 – 惡意流量、對網際網路開放的連接埠、嘗試存取網際網路的應用程式或 VM...
* 容量使用率 - 透過監視 VPN 閘道和其他服務的使用率趨勢，協助您避免過度佈建或使用率過低的問題

流量分析會為您提供可採取動作的資訊，從而協助您稽核貴組織的網路活動、保護應用程式和資料、將工作負載效能最佳化並保持符合規範。

![顯示跨區域流量的地理資訊地圖](../network-watcher/media/traffic-analytics/geo-map-view-showcasing-traffic-distribution-to-countries-and-continents.png) 

相關連結：
* [部落格文章](https://aka.ms/trafficanalytics)、[文件](https://aka.ms/trafficanalyticsdocs)、[常見問題集](https://docs.microsoft.com/azure/network-watcher/traffic-analytics-faq)

## <a name="dns-analytics"></a>DNS 分析
此解決方案針對 DNS 系統管理員所建置，可收集、分析 DNS 記錄並且相互關聯，以提供安全性、作業和效能相關的深入解析。  其部分功能如下：

* 識別嘗試解析惡意網域的用戶端
* 識別過時的資源記錄
* 顯示經常查詢的網域名稱和 Talkative DNS 用戶端
* 顯示 DNS 伺服器上的要求負載
* 監視動態 DNS 註冊失敗

![DNS 分析儀表板](./media/network-monitoring-overview/dns-analytics-overview.png) 

相關連結：
* [部落格文章](https://blogs.technet.microsoft.com/msoms/2017/04/19/introducing-oms-dns-analytics/)、[文件](https://docs.microsoft.com/azure/log-analytics/log-analytics-dns)

## <a name="miscellaneous"></a>其他資訊

* [新定價](https://docs.microsoft.com/azure/log-analytics/log-analytics-network-performance-monitor-pricing-faq)
