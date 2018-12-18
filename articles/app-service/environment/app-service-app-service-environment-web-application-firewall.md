---
title: 設定 App Service 環境的 Web 應用程式防火牆 (WAF)
description: 了解如何設定 Web 應用程式防火牆來保護 App Service 環境。
services: app-service\web
documentationcenter: ''
author: naziml
manager: erikre
editor: jimbe
ms.assetid: a2101291-83ba-4169-98a2-2c0ed9a65e8d
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/03/2018
ms.author: naziml
ms.custom: mvc
ms.openlocfilehash: 3fc9677d72dacd06bde2fcfa4812cf4613efef01
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394259"
---
# <a name="configuring-a-web-application-firewall-waf-for-app-service-environment"></a>設定 App Service 環境的 Web 應用程式防火牆 (WAF)
## <a name="overview"></a>概觀

Web 應用程式防火牆 (WAF) 會檢查輸入的 Web 流量以封鎖 SQL 插入、跨網站指令碼、惡意程式碼上傳和應用程式 DDoS 以及其他攻擊，以保護您的 Web 應用程式。 它們也會檢查來自後端 Web 伺服器的回應，以進行資料外洩防護 (DLP)。 這與隔離以及 App Service 環境所提供的額外調整合併使用，以提供裝載商務關鍵 Web 應用程式的理想環境，而這些 Web 應用程式需要防禦惡意要求和大量流量。 Azure 提供隨附於[應用程式閘道](http://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)的 WAF 功能。  若想了解如何整合您的 App Service 環境與應用程式閘道，請參閱[整合您的 ILB ASE 與應用程式閘道](http://docs.microsoft.com/azure/app-service/environment/integrate-with-application-gateway)一文。

除了 Azure 應用程式閘道以外，[Azure Marketplace](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf-byol/) 還提供多個市集選項，例如 [Barracuda WAF for Azure](https://www.barracuda.com/programs/azure)。 本文的其餘內容將著重於如何整合您的 App Service 環境與 Barracuda WAF 裝置。

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)] 

## <a name="setup"></a>設定
在本文中，我們會設定受多個 Barracuda WAF 負載平衡執行個體保護的 App Service 環境，只讓來自 WAF 的流量到達 App Service 環境，而且無法從 DMZ 存取。 在 Barracuda WAF 執行個體之前，我們也有「Azure 流量管理員」可在 Azure 資料中心和區域中進行負載平衡。 設定的概覽圖表看起來會如下圖所示：

![架構][Architecture] 

> [!NOTE]
> 透過引進 [App Service 環境的 ILB 支援](app-service-environment-with-internal-load-balancer.md)，您可以設定讓 ASE 無法從 DMZ 存取，且僅可供私人網路使用。 
> 
> 

## <a name="configuring-your-app-service-environment"></a>設定 App Service 環境
若要設定 App Service 環境，請參閱有關本主題的[文件](app-service-web-how-to-create-an-app-service-environment.md)。 建立 App Service 環境之後，即可在此環境中建立 Web App、API App 和 [Mobile Apps](../../app-service-mobile/app-service-mobile-value-prop.md)，而這些都是由下一節中設定的 WAF 所保護。

## <a name="configuring-your-barracuda-waf-cloud-service"></a>設定 Barracuda WAF 雲端服務
Barracuda 具有在 Azure 中於虛擬機器上部署其 WAF 的 [詳細文章](https://campus.barracuda.com/product/webapplicationfirewall/article/WAF/DeployWAFInAzure) 。 但是，因為我們想要具有備援性，且不想要引進單一失敗點，您在遵循這些指示時要將至少 2 個 WAF 執行個體 VM 部署至相同的雲端服務。

### <a name="adding-endpoints-to-cloud-service"></a>將端點加入雲端服務
雲端服務中有 2 個以上的 WAF VM 執行個體之後，即可使用 [Azure 入口網站](https://portal.azure.com/)加入應用程式所使用的 HTTP 和 HTTPS 端點，如下圖所示：

![設定端點][ConfigureEndpoint]

如果您的應用程式使用其他端點，請務必也將它們加入到此清單中。 

### <a name="configuring-barracuda-waf-through-its-management-portal"></a>透過管理入口網站設定 Barracuda WAF
Barracuda WAF 使用 TCP 連接埠 8000，以透過其管理入口網站進行設定。 如果有多個 WAF VM 執行個體，您需要針對每個 VM 執行個體重複這些步驟。 

> [!NOTE]
> 完成 WAF 設定之後，請從所有 WAF VM 移除 TCP/8000 端點，以保護 WAF 的安全。
> 
> 

如下圖所示加入管理端點，以設定您的 Barracuda WAF。

![加入管理端點][AddManagementEndpoint]

使用瀏覽器瀏覽至雲端服務上的管理端點。 如果您的雲端服務名為 test.cloudapp.net，則瀏覽至 http://test.cloudapp.net:8000 即可存取此端點。 您應該會看到如下圖的登入頁面，可以使用您在 WAF VM 設定階段中指定的認證登入。

![管理登入頁面][ManagementLoginPage]

登入之後，應該會看到如下圖中所顯示的儀表板，此儀表板顯示 WAF 保護的基本統計資料。

![管理儀表板][ManagementDashboard]

按一下 [服務] 索引標籤可讓您設定所保護服務的 WAF。 如需設定 Barracuda WAF 的詳細資訊，請參閱[其文件](https://techlib.barracuda.com/waf/getstarted1) \(英文\)。 在下列範例中，已設定處理 HTTP 和 HTTPS 流量的 Azure Web 應用程式。

![管理加入服務][ManagementAddServices]

> [!NOTE]
> 根據應用程式的設定方式與 App Service 環境中正在使用的功能而定，您需要轉送非 80 和 443 之 TCP 連接埠的流量 (例如，如果您為 Web 應用程式設定 IP SSL)。 如需 App Service 環境中所使用網路連接埠的清單，請參閱[控制輸入流量文件](app-service-app-service-environment-control-inbound-traffic.md)的＜網路連接埠＞一節。
> 
> 

## <a name="configuring-microsoft-azure-traffic-manager-optional"></a>設定 Microsoft Azure 流量管理員 (選擇性)
如果多個區域中都有您的應用程式，則您會想要使用 [Azure 流量管理員](../../traffic-manager/traffic-manager-overview.md)對它們進行負載平衡。 若要這樣做，您可以使用「流量管理員」設定檔中 WAF 的雲端服務名稱在 [Azure 入口網站](https://portal.azure.com)中加入端點，如下圖中所示。 

![流量管理員端點][TrafficManagerEndpoint]

如果您的應用程式需要驗證，請確定您有某個資源不需要任何驗證，以讓流量管理員進行 Ping 處理確認應用程式是否可用。 您可以在 [Azure 入口網站](https://portal.azure.com)的 [設定] 頁面上設定 URL，如下圖所示：

![設定流量管理員][ConfigureTrafficManager]

若要將「流量管理員」Ping 從 WAF 轉送給應用程式，則需要在 Barracuda WAF 上設定網站轉譯以將流量轉送給應用程式，如下列範例所示：

![網站轉譯][WebsiteTranslations]

## <a name="securing-traffic-to-app-service-environment-using-network-security-groups-nsg"></a>使用網路安全性群組 (NSG) 保護 App Service 環境流量的安全
如需使用雲端服務的 VIP 位址只限制從 WAF 流入 App Service 環境之流量的詳細資訊，請遵循 [控制輸入流量文件](app-service-app-service-environment-control-inbound-traffic.md) 。 以下是針對 TCP 通訊埠 80 執行這項工作的範例 Powershell 命令。

    Get-AzureNetworkSecurityGroup -Name "RestrictWestUSAppAccess" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP Barracuda" -Type Inbound -Priority 201 -Action Allow -SourceAddressPrefix '191.0.0.1'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP

將 SourceAddressPrefix 取代為 WAF 雲端服務的虛擬 IP 位址 (VIP)。

> [!NOTE]
> 刪除並重新建立雲端服務之後，雲端服務的 VIP 會變更。 在這樣做之後，請一定要更新網路資源群組中的 IP 位址。 
> 
> 

<!-- IMAGES -->
[Architecture]: ./media/app-service-app-service-environment-web-application-firewall/Architecture.png
[ConfigureEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/ConfigureEndpoint.png
[AddManagementEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/AddManagementEndpoint.png
[ManagementAddServices]: ./media/app-service-app-service-environment-web-application-firewall/ManagementAddServices.png
[ManagementDashboard]: ./media/app-service-app-service-environment-web-application-firewall/ManagementDashboard.png
[ManagementLoginPage]: ./media/app-service-app-service-environment-web-application-firewall/ManagementLoginPage.png
[TrafficManagerEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/TrafficManagerEndpoint.png
[ConfigureTrafficManager]: ./media/app-service-app-service-environment-web-application-firewall/ConfigureTrafficManager.png
[WebsiteTranslations]: ./media/app-service-app-service-environment-web-application-firewall/WebsiteTranslations.png
