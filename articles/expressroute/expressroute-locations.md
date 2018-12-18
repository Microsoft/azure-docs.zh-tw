---
title: 連線提供者和位置︰Azure ExpressRoute |Microsoft Docs
description: 本文提供提供服務所在位置以及如何連線到 Azure 區域的詳細概觀。 依連線提供者排序。
services: expressroute
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
ms.assetid: c878513a-d594-42ad-8b0e-403efd0c4b25
ms.service: expressroute
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/16/2018
ms.author: pareshmu
ms.openlocfilehash: 0a4ef0a627544de91b5f6d33d3e62eedf2cb7895
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49378885"
---
# <a name="expressroute-partners-and-peering-locations"></a>ExpressRoute 合作夥伴和對等互連位置

> [!div class="op_single_selector"]
> * [依提供者的位置](expressroute-locations.md)
> * [依位置的提供者](expressroute-locations-providers.md)


本文中的資料表會提供有關 ExpressRoute 連線提供者、ExpressRoute 地理涵蓋範圍、透過 ExpressRoute 支援的 Microsoft 雲端服務，以及 ExpressRoute 系統整合者 (SI) 的資訊。

## <a name="partners"></a>ExpressRoute 連線提供者
所有的 Azure 區域和位置都支援 ExpressRoute。 以下地圖提供了 Azure 區域和 ExpressRoute 位置的清單。 ExpressRoute 位置是指 Microsoft 與數個服務提供者對等互連的位置。

![位置圖][0]

如果您至少與地緣政治區域內的一個 ExpressRoute 位置連線，您將有權存取地緣政治區域內所有區域中的 Azure 服務。

### <a name="azure-regions-to-expressroute-locations-within-a-geopolitical-region"></a>地緣政治區域內 ExpressRoute 位置的 Azure 區域。
下表提供地緣政治區域內 ExpressRoute 位置的 Azure 區域對應。

| **地緣政治區域** | **Azure 區域** | **ExpressRoute 位置** |
| --- | --- | --- |
| **北美洲** |美國東部、美國西部、美國東部 2、美國西部 2、美國中部、美國中南部、美國中北部、美國中西部、加拿大中部、加拿大東部 |亞特蘭大、芝加哥、達拉斯、丹佛、拉斯維加斯、洛杉磯、邁阿密、紐約、聖安東尼奧、西雅圖、矽谷、華盛頓特區、蒙特婁、魁北克市、多倫多 |
| **南美洲** |巴西南部 |聖保羅 |
| **歐洲** |法國中部、法國南部、北歐、西歐、英國西部、英國南部 |阿姆斯特丹、阿姆斯特丹2、都柏林、倫敦、馬賽、紐波特 (威爾斯)、巴黎 |
| **亞洲** |東亞、東南亞 |香港特別行政區、吉隆坡、新加坡、新加坡2 |
| **日本** |日本西部、日本東部 |大阪、東京 |
| **澳大利亞** |澳洲東南部、澳洲東部 |墨爾本、雪梨 |
| **澳洲政府** | 澳大利亞中部、澳大利亞中部 2 |坎培拉、坎培拉 2 | 
| **印度** |印度西部、印度中部、印度南部 |辰內，孟買 |
| **南韓** |南韓中部、南韓南部 |釜山、首爾 |
| **南非** |[南非西部+、南非北部+](https://blogs.microsoft.com/blog/2017/05/18/microsoft-deliver-microsoft-cloud-datacenters-africa/) |開普敦、約翰尼斯堡 |

 **+** 表示即將推出

### <a name="regions-and-geopolitical-boundaries-for-national-clouds"></a>國家雲端的區域和地理政治界限
下表提供國家雲端的區域和地理政治界限等資訊。

| **地緣政治區域** | **Azure 區域** | **ExpressRoute 位置** |
| --- | --- | --- |
| **美國政府雲端** |US Gov 亞利桑那州、US Gov 愛荷華州、US Gov 德克薩斯州、US Gov 維吉尼亞州、US DoD 中部、US DoD 東部  |芝加哥、達拉斯、紐約、鳳凰城、聖安東尼奧、西雅圖、矽谷、華盛頓特區 |
| **中國東部** |中國東部、中國東部 2 |上海 |
| **中國北部** |中國北部、中國北部 2 |北京 |
| **德國** |德國中部、德國東部 |柏林、法蘭克福 |

標準 ExpressRoute SKU 不支援跨地緣政治區域的連線。 您必須啟用 ExpressRoute 進階附加元件，以支援全球連線。 不支援連線至國家雲端環境。 如果有需要的話，您可以聯絡您的連線提供者。

## <a name="locations"></a>連線提供者位置

下表顯示服務提供者的位置。 如果您想依位置檢視可用的提供者，請參閱[位置服務提供者](expressroute-locations-providers.md#locations)。


### <a name="production-azure"></a>生產 Azure
| **服務提供者** | **Microsoft Azure** | **Office 365 和 Dynamics 365** | **位置** |
| --- | --- | --- | --- |
| **[AARNet](https://www.aarnet.edu.au/network-and-services/cloud-services-applications/azure-expressroute/)** |支援 |支援 |墨爾本、雪梨 |
| **[Airtel](http://www.airtel.in/creatingsmiles/)** | 支援 | 支援 | 辰內，孟買 |
| **[Aryaka Networks](http://www.aryaka.com/)** |支援 |支援 |阿姆斯特丹、達拉斯、香港、矽谷、新加坡、東京、華盛頓特區 |
| **[Ascenty Data Centers](https://ascenty.com/servicos/cloud-connect/microsoft-expressroute/)** |支援 |支援 |聖保羅 |
| **[AT&T NetBond](https://www.synaptic.att.com/clouduser/html/productdetail/ATT_NetBond.htm)** |支援 |支援 |阿姆斯特丹、芝加哥、達拉斯、倫敦、矽谷、新加坡、雪梨、東京、多倫多、華盛頓特區 |
| **[Bell Canada](https://business.bell.ca/shop/enterprise/cloud-connect-access-to-cloud-partner-services)** |支援 |支援 |蒙特婁、多倫多、魁北克市 |
| **[British Telecom](https://globalservices.bt.com/en/solutions/products/bt-compute-for-microsoft-azure)** |支援 |支援 |阿姆斯特丹、香港、倫敦、矽谷、新加坡、雪梨、東京、華盛頓特區 |
| **[C3ntro](https://c3ntro.com/data/express-route/)** |敬請期待 |敬請期待 |邁阿密 |
| **CDC** | 支援 | 支援 | 坎培拉、坎培拉 2 |
| **[CenturyLink Cloud Connect](http://www.centurylink.com/cloudconnect)** |支援 |支援 |拉斯維加斯、紐約、聖安東尼奧、矽谷、東京、多倫多 |
| **China Telecom Global** |支援 |不支援 |香港 |
| **[Cologix](http://www.cologix.com/solutions/cloud-connect/public-clouds/microsoft-cloud/)** |支援 |支援 |達拉斯、蒙特婁、多倫多 |
| **[Colt](http://www.colt.net/uk/en/news/colt-announces-dedicated-cloud-access-for-microsoft-azure-services-en.htm)** |支援 |支援 |阿姆斯特丹、都柏林、倫敦、巴黎、東京 |
| **[Comcast](https://business.comcast.com/landingpage/microsoft-azure)** |支援 |支援 |芝加哥、矽谷、華盛頓特區 |
| **[CoreSite](http://www.coresite.com/solutions/cloud-services/public-cloud-providers/microsoft-azure-expressroute)** |支援 |支援 |芝加哥、丹佛、洛杉磯、紐約、矽谷、華盛頓特區 |
| **eir** |支援 |支援 |都柏林|
| **Epsilon Global Communications** |支援 |支援 |新加坡 |
| **[Equinix](http://www.equinix.com/partners/microsoft-azure/)** |支援 |支援 |阿姆斯特丹、亞特蘭大、芝加哥、達拉斯、都柏林、香港、倫敦、洛杉磯、墨爾本、邁阿密、紐約、大阪、巴黎+、聖保羅、西雅圖、矽谷、新加坡、雪梨、東京、多倫多、華盛頓特區 |
| **euNetworks** |支援 |支援 |阿姆斯特丹、都柏林、倫敦 |
| **GÉANT** |支援 |支援 |阿姆斯特丹 |
| **[Global Cloud Xchange (GCX)] (http://globalcloudxchange.com/cloud-platform/cloud-x-fusion/cloud-x-fusion-for-azure/)** | 支援| 支援 | 辰內，孟買 |
| **[InterCloud](https://www.intercloud.com/)** |支援 |支援 |阿姆斯特丹、倫敦、巴黎、矽谷、新加坡、華盛頓特區 |
| **Internet2** |支援 |支援 |華盛頓 |
| **[Internet Initiative Japan Inc. - IIJ](http://www.iij.ad.jp/en/news/pressrelease/2015/1216-2.html)** |支援 |支援 |大阪、東京 |
| **[Internet Solutions - Cloud Connect](https://www.is.co.za/solution/cloud-connect/)** |支援 |支援 |開普敦、約翰尼斯堡、倫敦 |
| **[Interxion](http://www.interxion.com/why-interxion/colocate-with-the-clouds/Microsoft-Azure/)** |支援 |支援 |阿姆斯特丹、阿姆斯特丹2、都柏林、馬賽、倫敦、巴黎 |
| **[IX Reach](https://www.ixreach.com/services/cloud-connectivity/microsoft-azure/)**|支援 |支援 | 阿姆斯特丹、矽谷、多倫多 |
| **Jisc** |支援 |支援 |倫敦 |
| **[KINX](https://www.kinx.net/service/network/cloudhub/ms-expressroute/?lang=en)** |支援 |支援 |首爾 |
| **[KPN](http://www.kpn.com/cloudconnect)** | 支援 | 支援 | 阿姆斯特丹 | 
| **[Level 3 Communications](http://your.level3.com/LP=882?WT.tsrc=02192014LP882AzureVanityAzureText)** |支援 |支援 |阿姆斯特丹、芝加哥、達拉斯、倫敦、新港、聖保羅、西雅圖、矽谷、新加坡、華盛頓特區 |
| **LG CNS** |支援 |支援 |釜山、首爾 |
| **[Liquid Telecom](https://www.liquidtelecom.com/products-and-services/cloud.html)** |支援 |支援 |開普敦、約翰尼斯堡 |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |支援 |支援 |阿姆斯特丹、亞特蘭大、芝加哥、達拉斯、丹佛、都柏林、香港、拉斯維加斯、倫敦、洛杉磯、墨爾本、邁阿密、紐約、魁北克市、聖安東尼奧、西雅圖、矽谷、新加坡、新加坡 2、雪梨、多倫多、華盛頓特區 |
| **MTN** |支援 |支援 |倫敦 |
| **[Neutrona Networks](http://www.neutrona.com/index.php/azure-expressroute/)** |支援 |支援 |達拉斯、邁阿密、聖保羅 |
| **[新一代資料](http://www.nextgenerationdata.co.uk/ngd-cloud-gateway/)** |支援 |支援 |Newport(Wales) |
| **NEXTDC** |支援 |支援 |墨爾本、雪梨 |
| **[NTT Communications](http://www.ntt.com/en/services/network/virtual-private-network.html)** |支援 |支援 |阿姆斯特丹、香港、倫敦、洛杉磯、大阪、新加坡、雪梨、東京、華盛頓特區 |
| **[NTT EAST](https://flets.com/cloudgateway/crossconnect/)** |支援 |支援 |東京 |
| **[NTT SmartConnect](http://cloud.nttsmc.com/cxc/azure.html)** |支援 |支援 |大阪 |
| **[Optus](https://www.optus.com.au/enterprise/networking/network-connectivity/express-link)** |支援 |支援 |墨爾本+、雪梨 |
| **[Orange](http://www.orange-business.com/en/products/business-vpn-galerie)** |支援 |支援 |阿姆斯特丹、香港、倫敦、巴黎、矽谷、新加坡、雪梨、華盛頓特區 |
| **[PacketFabric](https://www.packetfabric.com/packetcor/microsoft-azure/)** |支援 |支援 |芝加哥、矽谷、華盛頓特區 |
| **[PCCW Global Limited](https://consoleconnect.com/clouds/#azureRegions)** |支援 |支援 |芝加哥、香港、倫敦 |
| **[Sejong Telecom](https://www.sejongtelecom.net/en/pages/service/cloud_ms)** |支援 |支援 |首爾 |
| **[SIFY](http://telecom.sify.com/azure-expressroute.html)** |支援 |支援 |辰內，孟買 |
| **[SingTel](http://info.singtel.com/about-us/news-releases/singtel-provide-secure-private-access-microsoft-azure-public-cloud)** |支援 |支援 |新加坡、新加坡 2 |
| **[Softbank](http://www.softbank.jp/biz/cloud/cloud_access/direct_access_for_az/)** |支援 |支援 |大阪、東京 |
| **[Sprint](https://business.sprint.com/solutions/cloud-networking/)** |支援 |支援 |芝加哥、矽谷、華盛頓特區 |
| **[Tata Communications](http://www.tatacommunications.com/lp/izo/azure/azure_index.html)** |支援 |支援 |阿姆斯特丹、辰內、香港、倫敦、孟買、矽谷、新加坡、華盛頓特區 |
| **[Telefonica](https://www.business-solutions.telefonica.com/es/enterprise/solutions/efficient-infrastructure/managed-voice-data-connectivity/)** |支援 |支援 |阿姆斯特丹、聖保羅 |
| **[Telehouse - KDDI](http://www.telehouse.net/solutions/cloud-services/cloud-link)** |支援 |支援 |倫敦 |
| **Telenor** |支援 |支援 |阿姆斯特丹、倫敦 |
| **[Telia Carrier](https://teliacarrier.com/our-services/connectivity/cloud-connect.html?title=Cloud%20Connect)** | 支援 | 支援 |倫敦、華盛頓特區 |
| **Telmex Uninet**| 敬請期待 | 敬請期待 | 達拉斯+ |
| **[Telstra Corporation](http://www.telstra.com.au/business-enterprise/network-services/networks/cloud-direct-connect/)** |支援 |支援 |墨爾本、雪梨 |
| **[Telus](http://www.telus.com)** |支援 |支援 |蒙特婁、多倫多 |
| **[Teraco](https://www.teraco.co.za/services/africa-cloud-exchange/)** |支援 |支援 |開普敦、約翰尼斯堡 |
| **TIME** | 支援 | 支援 | 吉隆坡 |
| **[UOLDIVEO](http://www.uoldiveo.com.br/solucoes/cloud.html#rmcl)** |支援 |支援 |聖保羅 |
| **[Verizon](http://www.verizonenterprise.com/products/networking/secure-cloud-interconnect/)** |支援 |支援 |阿姆斯特丹、芝加哥、達拉斯、香港、倫敦、矽谷、新加坡、雪梨、東京、華盛頓特區 |
| **[Vodafone](http://www.vodafone.com/business/global-enterprise/global-connectivity/vodafone-ip-vpn-cloud-connect)** |支援 |不支援 |倫敦 |
| **Vodafone Idea** | 支援 | 支援 | 孟買 |
| **[Zayo](http://www.zayo.com/solutions/industries/cloud-connectivity/microsoft-expressroute)** |支援 |支援 |阿姆斯特丹、芝加哥、達拉斯、倫敦、洛杉磯、蒙特婁、紐約、矽谷、多倫多、華盛頓特區 |

 **+** 表示即將推出

### <a name="national-cloud-environment"></a>國家雲端環境

### <a name="us-government-cloud"></a>美國政府雲端
| **服務提供者** | **Microsoft Azure** | **Office 365** | **位置** |
| --- | --- | --- | --- |
| **[AT&T NetBond](https://www.synaptic.att.com/clouduser/html/productdetail/ATT_NetBond.htm)** |支援 |支援 |芝加哥、華盛頓特區 |
| **[CenturyLink Cloud Connect](http://www.centurylink.com/cloudconnect)** |支援 |支援 |鳳凰城、紐約 |
| **[Equinix](http://www.equinix.com/partners/microsoft-azure/)** |支援 |支援 |芝加哥、達拉斯、紐約、西雅圖、矽谷、華盛頓特區 |
| **[Level 3 Communications](http://your.level3.com/LP=882?WT.tsrc=02192014LP882AzureVanityAzureText)** |支援 |支援 |芝加哥、矽谷、華盛頓特區 |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |支援 | 支援 | 芝加哥、達拉斯、聖安東尼奧 |
| **[Verizon](http://news.verizonenterprise.com/2014/04/secure-cloud-interconnect-solutions-enterprise/)** |支援 |支援 |芝加哥、達拉斯、紐約、華盛頓特區 |

### <a name="china"></a>中國
| **服務提供者** | **Microsoft Azure** | **Office 365** | **位置** |
| --- | --- | --- | --- |
| **China Telecom** |支援 |不支援 |北京、上海 |

若要深入了解，請參閱 [ExpressRoute (中國)](http://www.windowsazure.cn/home/features/expressroute/)。

### <a name="germany"></a>德國
| **服務提供者** | **Microsoft Azure** | **Office 365** | **位置** |
| --- | --- | --- | --- |
| **[Colt](http://www.colt.net/uk/en/news/colt-announces-dedicated-cloud-access-for-microsoft-azure-services-en.htm)** |支援 |不支援 |法蘭克福 |
| **[Equinix](http://www.equinix.com/partners/microsoft-azure/)** |支援 |不支援 |法蘭克福 |
| **[e-shelter](https://www.e-shelter.de/en/microsoft-expressroutetm)** |支援 |不支援 |柏林 |
| **Interxion** |支援 |不支援 |法蘭克福 |
| **[Megaport](https://www.megaport.com/services/microsoft-expressroute/)** |支援  | 不支援 | 柏林 |
| **T-Systems** |支援 |不支援 |柏林 |

## <a name="connectivity-through-exchange-providers"></a>透過 Exchange 提供者連線

如果上一節中未列出您的連線提供者，您仍然可以建立連線。

* 請洽詢您的連線提供者，以了解他們是否連線到上方表格中列出的任何 Exchange 提供者。 您可以檢查下列連結，以收集 Exchange 提供者所提供之服務的相關詳細資訊。 已有數個連線提供者連線到乙太網路 Exchange。
  * [Cologix](http://www.cologix.com/)
  * [CoreSite](http://www.coresite.com/)
  * [Equinix Cloud Exchange](http://www.equinix.com/services/interconnection-connectivity/cloud-exchange/)
  * [Interxion](http://www.interxion.com/products/interconnection/cloud-connect/)
  * [IX Reach](https://www.ixreach.com/services/cloud-connectivity/microsoft-azure/)
  * [Megaport](https://www.megaport.com/services/microsoft-expressroute/)
  * [NextDC](http://www.nextdc.com/)
  * [PacketFabric](https://www.packetfabric.com/packetcor/microsoft-azure/) 
* 請您的連線提供者將您的網路延伸到選擇的對等互連位置。
  * 請確保您的連線提供者以高可用性的方式延伸您的連線，因此不會有單一失敗點。
* 排序一個 ExpressRoute 循環，將 Exchange 視為連線至 Microsoft 的連線提供者。
  * 依照 [建立 ExpressRoute 循環](expressroute-howto-circuit-classic.md) 中的步驟來設定連線。

## <a name="connectivity-through-additional-service-providers"></a>透過額外服務提供者連線

| **連線提供者** | **Exchange** | **位置** |
| --- | --- | --- |
| **[1CLOUDSTAR](http://www.1cloudstar.com/service/cloudconnect-azure-expressroute/)** |Equinix |新加坡 |
| **[Airgate Technologies, Inc.](http://airgate.ca/cloud-express/)** | Equinix、Cologix | 多倫多、蒙特婁 |
| **[Alaska Communications](http://www.alaskacommunications.com/For-Your-Business/Direct-Cloud-Service)** |Equinix |Seattle |
| **[Altice Business](https://golightpath.com/transport)** |Equinix |紐約、華盛頓特區 |
| **[Arteria Networks Corporation](https://www.arteria-net.com/business/service/cloud/sca/)** |Equinix |東京 |
| **[Axtel](http://alestra.mx/landing/expressrouteazure/)** |Equinix |達拉斯|
| **[Bezeq International Ltd.](https://www.bezeqint.net/english)** | euNetworks | 倫敦 |
| **[BICS](https://bics.com/bics-solutions-suite/cloud-connect/)** | Equinix | 阿姆斯特丹、法蘭克福、倫敦、新加坡、華盛頓特區 |
| **[BroadBand Tower, Inc.](http://www.bbtower.co.jp/product-service/data-center/network/dcconnect-for-azure/)** | Equinix | 東京 |
| **[C3ntro Telecom](http://www.c3ntro.com/data/cloud-conectivity/)** | Equinix、Megaport | 達拉斯 |
| **[Chief](https://www.chief.com.tw/dispPageBox/ct.aspx?ddsPageID=ENCLOUDSERVICE&dbid=4852937342)** | Equinix | 香港 |
| **[Cinia](https://www.cinia.fi/en/services/connectivity-services/direct-public-cloud-connection.html)** | Equinix、Megaport | 法蘭克福、漢堡 |
| **[CloudXpress](https://www2.telenet.be/fr/business/produits-services/internet/cloudxpress/)** | Equinix | 阿姆斯特丹 | 
| **[Cogeco Peer 1](https://www.cogecopeer1.com/en/)**| Equinix | 蒙特婁、多倫多 |
| **[Cox Business](https://www.cox.com/business/networking/cloud-connectivity.html)** | Equinix | 達拉斯、矽谷、華盛頓特區 | 
| **[Data Foundry](https://www.datafoundry.com/services/cloud-connect)** | Megaport | 達拉斯 |
| **[Epsilon Telecommunications Limited](http://www.epsilontel.com/data-connectivity/cloud-access/)** | Equinix | 倫敦、新加坡、華盛頓特區 |
| **[Eurofiber](https://eurofiber.nl/microsoft-azure/)** | Equinix | 阿姆斯特丹 |
| **[指數 E](http://www.exponential-e.com/services/connectivity-services/cloud-connect-exchange)** | Equinix | 倫敦 |
| **[Fastweb S.p.A](http://www.fastweb.it/grandi-aziende/connessione-voce-e-wifi/scheda-prodotto/rete-privata-virtuale/)** | Equinix | 阿姆斯特丹 |
| **[Fibrenoire](https://www.fibrenoire.ca/en/cloudextn)** | Megaport | 魁北克市 |
| **[Gtt Communications Inc](https://www.gtt.net/wp-content/uploads/2017/04/EtherCloud-Data-Sheet.pdf)** |Equinix | 華盛頓 |
| **[HSO](http://www.hso.co.uk/products/cloud-direct)** |Equinix | 倫敦、斯勞 |
| **[IVedha Inc](http://www.ivedha.com/cloud/manage-azure-cloud/express-route-4/)**| Equinix | 多倫多 |
| **LGA Telecom** |Equinix |新加坡|
| **[Lightower](http://www.lightower.com/network-solutions/cloud-connect/#microsoft-azure)** |Equinix | 芝加哥、紐約、華盛頓特區 |
| **[Macroview Telecom](http://www.macroview.com/en/scripts/catitem.php?catid=solution&sectionid=expressroute)** |Equinix |香港 |
| **[Macquarie Telecom Group](https://macquariegovernment.com/secure-cloud/secure-cloud-exchange/)** | Megaport | 雪梨 |
| **[MainOne](https://www.mainone.net/services/connectivity/cloud-connect/)** |Equinix | 阿姆斯特丹 |
| **[Masergy](https://www.masergy.com/solutions/hybrid-networking/cloud-marketplace/microsoft-azure)** | Equinix | 華盛頓 |
| **[NexGen Networks](http://www.nexgen-net.com/nexgen-networks-direct-connect-microsoft-azure-expressroute.html)** | Interxion | 倫敦 |
| **[Nianet](https://nianet.dk/produkter/internet/microsoft-expressroute)** |Telecity | 阿姆斯特丹、法蘭克福 |
| **[Post](https://www.teralinksolutions.com/cloud-connectivity/cloudbridge-to-azure-expressroute/)**|Equinix | 阿姆斯特丹 |
| **[Proximus](https://www.proximus.be/en/id_b_cl_proximus_external_cloud_connect/companies-and-public-sector/discover/magazines/expert-blog/proximus-external-cloud-connect.html)**|Equinix | 阿姆斯特丹、都柏林、倫敦、巴黎 |
| **[QSC AG](https://www.qsc.de/de/produkte-loesungen/cloud-services-und-it-outsourcing/pure-enterprise-cloud/multi-cloud-management/azure-expressroute/)** |Interxion | 法蘭克福 |  
| **Rogers** | Cologix、Equinix | 蒙特婁、多倫多 |
| **[Tamares Telecom](http://www.tamarestelecom.com/our-services/#Connectivity)** | Telecity | 倫敦 | 
| **[Telecom Italia Sparkle](https://www.tisparkle.com/our-platform/corporate-platform/sparkle-cloud-connect#catalogue)**| Equinix | 阿姆斯特丹 |
| **[Telia](https://www.telia.se/foretag/losningar/produkter-tjanster/datanet)** | Equinix | 阿姆斯特丹 |
| **[ThinkTel](http://www.thinktel.ca/services/agile-ix-data/expressroute/)** | Equinix | 多倫多 | 
| **[Transtelco](http://www.transtelco.net/tcloud/microsoft)** |Equinix | 達拉斯、洛杉磯 |  
| **[United Information Highway (UIH)](https://www.uih.co.th/en/internet-solution/cloud-direct/uih-cloud-direct-for-microsoft-azure-expressroute)**| Equinix | 新加坡 |
| **[Venha Pra Nuvem](http://venhapranuvem.com.br/infraestrutura/oferta-especial-digital-edge-expressroute/)** | Equinix | 聖保羅 |
| **[Webair](https://www.webair.com/microsoft-express-route-partnership/)**| Megaport | 紐約 |
| **[Windstream](http://www.windstreambusiness.com/solutions/cloud-services/cloud-and-managed-hosting-services)**| Equinix | 芝加哥、矽谷、華盛頓特區 |
| **Zain** |Equinix |倫敦|
| **[Zertia](http://www.zertia.es/index.php/novedades)**| Level 3 | 馬德里 |
| **[Zirro](https://zirro.com/services/)**| Equinix | 蒙特婁、多倫多 |

## <a name="connectivity-through-datacenter-providers"></a>透過資料中心提供者連線
| **提供者** | **Exchange** |
| --- | --- |
| **[Cyrus One](https://cyrusone.com/enterprise-data-center-services/connectivity-and-interconnection/cloud-connectivity-reaching-amazon-microsoft-google-and-more/microsoft-azure-expressroute/?doing_wp_cron=1498512235.6733090877532958984375)** | Megaport |
| **[Cyxtera](https://www.cyxtera.com/data-center-services/interconnection)** | Megaport |
| **[Digital Realty](https://www.digitalrealty.com/services/interconnection/service-exchange/)** | Megaport |
| **[EdgeConnex](http://www.edgeconnex.com/services/edge-data-centers-proximity-matters/)** | Megaport |
| **[RagingWire Data Centers](http://www.ragingwire.com/wholesale/wholesale-data-centers-worldwide-nexcenters)** | IX Reach |
| **[T5 Datacenters](http://t5datacenters.com/network-cloud-connect/)** | IX Reach |

## <a name="connectivity-through-national-research-and-education-networks-nren"></a>透過 National Research and Education Networks (NREN) 連線

| **提供者**|
| --- |
| **AARNET**| 
| **DeIC，透過 GÉANT**|
| **GARR，透過 GÉANT**|
| **GÉANT**|
| **HEAnet，透過 GÉANT**|
| **Internet2**|
| **JISC**|
| **RedIRIS，透過 GÉANT**|
| **SINET**|
| **Surfnet，透過 GÉANT**|

* 如果連線提供者未列於此處，請查看他們是否已連線到上列的任何 ExpressRoute Exchange 提供者。

## <a name="expressroute-system-integrators"></a>ExpressRoute 系統整合者
根據您的網路規模，為符合您的需求而啟用私人連線可能有一定的難度。 您可以使用下表所列出的任何系統整合者來協助您開始使用 ExpressRoute。

| **系統整合者** | **Continent** |
| --- | --- |
| **[Altogee](https://altogee.be/diensten/express-route/)** | 歐洲 |
| **[Avanade Inc.](http://www.avanade.com/)** | 亞洲、歐洲、北美洲、南美洲 |
| **[Bright Skies GmbH](http://bskies.io/expressroute)** | 歐洲
| **[Ensyst](http://www.ensyst.com.au)** | 亞洲
| **[Equinix Professional Services](http://www.equinix.com/services/consulting/)** | 北美洲 |
| **[FlexManage](http://www.flexmanage.com/cloud)** | 北美洲 |
| **[Inframon](http://www.inframon.com/partner/microsoft/)** | 歐洲 |
| **[Lightstream](https://www.ltstream.com/expressroute)** | 北美洲 |
| **[The IT Consultancy Group](http://itconsult.com.au/microsoft-expressroute)** | 澳大利亞 |
| **[MOQdigital](http://www.moqdigital.com.au/insights/technical/network-connectivity-options-for-azure)** | 澳大利亞 |
| **[MSG Services](https://www.msg-services.de/it-services/managed-services/cloud-outsourcing/)** | 歐洲 (德國) |
| **[Nelite](https://www.nelite.com/offres-services/)** | 歐洲 |
| **[New Signature](https://newsignature.com/technologies/express-route/)** | 歐洲 |
| **[OneAs1a](http://www.oneas1a.com/connectivity.html)** | 亞洲 |
| **[Orange Networks](https://orange-networks.com/blog/88-azureexpressroute)** | 歐洲 |
| **[Perficient](http://www.perficient.com/Partners/Microsoft/Cloud/Azure-ExpressRoute)** | 北美洲 |
| **[Presidio](http://info.presidio.com/microsoft-azure-expressroute)** | 北美洲 |
| **[sol-tec](https://www.sol-tec.com/services)** | 歐洲 |
| **[Venha Pra Nuvem](http://venhapranuvem.com.br/infraestrutura/oferta-especial-digital-edge-expressroute/)** | 南美洲 |
| **[Vigilant.IT](https://vigilant.it/expressroute)** | 澳大利亞 |

## <a name="next-steps"></a>後續步驟
* 如需有關 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。
* 請確定符合所有必要條件。 請參閱 [ExpressRoute 必要條件](expressroute-prerequisites.md)。

<!--Image References-->
[0]: ./media/expressroute-locations/expressroute-locations-map.png "位置圖"
