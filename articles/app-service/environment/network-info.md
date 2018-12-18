---
title: Azure App Service Environment 的網路考量
description: 說明 ASE 網路流量與如何使用 ASE 設定 NSG 和 UDR
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: 955a4d84-94ca-418d-aa79-b57a5eb8cb85
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/29/2018
ms.author: ccompy
ms.openlocfilehash: 6d4f7fab0c36095d96cec0038a39744102e8972b
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433747"
---
# <a name="networking-considerations-for-an-app-service-environment"></a>App Service Environment 的網路考量 #

## <a name="overview"></a>概觀 ##

 Azure [App Service Environment][Intro] 是將 Azure App Service 部署到您 Azure 虛擬網路 (VNet) 中子網路的一種部署。 App Service Environment (ASE) 有二種部署類型：

- **外部 ASE**：會在可存取網際網路的 IP 位址上公開 ASE 裝載的應用程式。 如需詳細資訊，請參閱[建立外部 ASE][MakeExternalASE]。
- **ILB ASE**：會在您 VNet 內部的 IP 位址上公開 ASE 裝載的應用程式。 內部端點是一個內部負載平衡器 (ILB)，這就是它稱為 ILB ASE 的原因。 如需詳細資訊，請參閱[建立和使用 ILB ASE][MakeILBASE]。

App Service Environment 有兩個版本：ASEv1 和 ASEv2。 如需 ASEv1 的資訊，請參閱 [App Service Environment v1 簡介][ASEv1Intro]。 ASEv1 可以部署至傳統或 Resource Manager VNet 中。 ASEv2 只能部署至 Resource Manager VNet 中。

來自 ASE 並傳送至網際網路的所有呼叫，都會透過為 ASE 指派的 VIP 離開 VNet。 此 VIP 的公用 IP 是來自 ASE 並傳送至網際網路之所有呼叫的來源 IP。 如果 ASE 中的應用程式會呼叫位於 VNet 中或跨 VPN 的資源，則來源 IP 就會是 ASE 所用子網路中的其中一個 IP。 因為 ASE 是位於 VNet 之內，所以它也可以存取 VNet 內的資源，而不需要任何額外設定。 如果 VNet 連線至您的內部部署網路，您 ASE 中的應用程式也擁有該處資源的存取權，而不需其他設定。

![外部 ASE][1] 

如果您有外部 ASE，公用 VIP 也會是您 ASE 應用程式針對以下項目進行解析的端點：

* HTTP/S。 
* FTP/S。 
* Web 部署。
* 遠端偵錯。

![ILB ASE][2]

如果您有 ILB ASE，則 ILB 的位址是 HTTP/S、FTP/S、Web 部署和遠端偵錯的端點。

一般的應用程式存取連接埠為：

| 使用 | 從 | 至 |
|----------|---------|-------------|
|  HTTP/HTTPS  | 可由使用者設定 |  80、443 |
|  FTP/FTPS    | 可由使用者設定 |  21、990、10001-10020 |
|  Visual Studio 遠端偵錯  |  可由使用者設定 |  4016、4018、4020、4022 |

這適用於您位於外部 ASE 或 ILB ASE 上的情況。 如果您是在外部 ASE 中，就會叫用公用 VIP 上的那些連接埠。 如果您是在 ILB ASE 中，就會叫用 ILB 上的那些連接埠。 如果您鎖定連接埠 443，可能會影響在入口網站中公開的某些功能。 如需詳細資訊，請參閱[入口網站相依性](#portaldep)。

## <a name="ase-subnet-size"></a>ASE 子網路大小 ##

ASE 部署之後，就無法變更用來裝載 ASE 的子網路大小。  每個基礎結構角色以及每個隔離的 App Service 方案執行個體，ASE 都會使用一個位址。  此外，每建立一個子網路，Azure 網路就會使用 5 個位址。  在您建立應用程式之前，完全沒有 App Service 方案的 ASE 會使用 12 個位址。  如果是 ILB ASE，在您於其中建立應用程式之前，該 ASE 會使用 13 個位址。 當您相應放大 ASE 時，您的 App Service 方案執行個體每 15 和 20 的倍數便會新增基礎結構角色。

   > [!NOTE]
   > 除了 ASE 以外，子網路中可能沒有其他項目。 請務必選擇預留未來成長空間的位址空間。 您之後無法變更此設定。 我們建議使用包含 256 個位址的 `/24` 大小。

當您相應增加或減少時，即會新增適當大小的新角色，然後將您的工作負載從目前大小移轉為目標大小。 只有在移轉應用程式之後，才會移除原始的 VM。 這表示如果您擁有的 ASE 有 100 個 ASP 執行個體，則當您需要將 VM 數量加倍時會需要一段時間。  基於這個原因，我們建議使用 '/24' 來配合您可能需要的任何變更。  

## <a name="ase-dependencies"></a>ASE 相依性 ##

### <a name="ase-inbound-dependencies"></a>ASE 連入相依性 ###

ASE 連入存取相依性如下：

| 使用 | 從 | 至 |
|-----|------|----|
| 管理 | App Service 管理位址 | ASE 子網路：454、455 |
|  ASE 內部通訊 | ASE 子網路：所有連接埠 | ASE 子網路：所有連接埠
|  允許 Azure Load Balancer 輸入 | Azure Load Balancer | ASE 子網路：所有連接埠
|  應用程式指派的 IP 位址 | 應用程式指派的位址 | ASE 子網路：所有連接埠

除了系統監視之外，連入管理流量也提供 ASE 的命令與控制。 此流量的來源位址列於 [ASE 管理位址][ASEManagement]文件中。 網路安全性設定需在連接埠 454 和 455 上允許來自所有 IP 的存取。 如果您封鎖來自那些位址的存取，則您的 ASE 將變成狀況不良，接著會變成暫時停權。

ASE 子網路中有許多用於內部元件通訊的連接埠，您可以變更這些連接埠。  ASE 子網路的所有連接埠都必須能夠從 ASE 子網路存取。 

您必須開啟最小連接埠 454、455 和 16001，才能在 Azure Load Balancer 與 ASE 子網路之間進行通訊。 16001 連接埠可用來保持負載平衡器與 ASE 之間的流量運作。 如果您使用 ILB ASE，則可以將流量僅鎖定於 454、455、16001 連接埠。  如果您使用外部 ASE，則需要考慮一般應用程式存取連接埠。  如果您使用應用程式指派的位址，則需要在所有連接埠將它開啟。  將位址指派給特定應用程式時，負載平衡器會使用事先不知道的連接埠將 HTTP 和 HTTPS 流量傳送至 ASE。

如果您使用應用程式指派的 IP 位址，則需要允許將來自應用程式指派之 IP 的流量傳送至 ASE 子網路。

在連接埠 454 與 455 傳入的 TCP 流量必須回到相同的 VIP，否則您將會有非對稱式路由問題。 

### <a name="ase-outbound-dependencies"></a>ASE 連出相依性 ###

針對連出存取，ASE 取決於多個外部系統。 那些系統相依性中有許多都會以 DNS 名稱定義，且不會對應至一組固定的 IP 位址。 因此，ASE 需要透過不同的連接埠進行從 ASE 子網路至所有外部 IP 的連出存取。 

連出相依性的完整清單列於說明[鎖定 App Service Environment 連出流量](./firewall-integration.md)的文件中。 如果 ASE 失去對其相依性的存取，就會停止運作。 當停止運作時間達到一定的長度之後，ASE 就會暫停。 

### <a name="customer-dns"></a>客戶 DNS ###

如果已經使用客戶定義的 DNS 伺服器設定 VNet，租用戶工作負載就會使用該伺服器。 基於管理目的，ASE 仍需要和 Azure DNS 通訊。 

如果 VNet 在 VPN 的另一端使用客戶 DNS 進行設定，DNS 伺服器必須能夠從包含 ASE 的子網路中進行連線。

若要從 Web 應用程式測試解決方法，您可以使用主控台命令 *nameresolver*。 移至應用程式 scm 網站中的偵錯視窗，或移至入口網站中的應用程式，然後選取主控台。 從殼層提示字元中，您可以發出命令 *nameresolver* 以及您想要查閱的位址。 您取得的結果會與您應用程式在進行相同查閱時所取得的結果一樣。 如果您使用 nslookup，就要改為使用 Azure DNS 進行查閱。

如果您變更 ASE 所在之 VNet 的 DNS 設定，就必須重新啟動 ASE。 若要避免重新啟動 ASE，強烈建議您在建立 ASE 之前設定 VNet 的 DNS 設定。  

<a name="portaldep"></a>

## <a name="portal-dependencies"></a>入口網站相依性 ##

除了 ASE 功能性相依性之外，還有幾個額外項目和入口網站體驗有關。 Azure 入口網站的部分功能需要能夠直接存取「SCM 網站」。 Azure App Service 中的每個應用程式都有兩個 URL。 第一個 URL 是用來存取您的應用程式。 第二個 URL 則是用來存取 SCM 網站，也稱為「Kudu 主控台」。 使用 SCM 網站的功能包括：

-   Web Job
-   Functions
-   記錄串流
-   Kudu
-   擴充功能
-   處理序總管
-   主控台

當您使用 ILB ASE 時，無法從 VNet 外部透過網際網路存取 SCM 網站。 當您的應用程式裝載於 ILB ASE 時，將無法從入口網站使用某些功能。  

那些需要依賴 SCM 網站的功能中，有許多功能也會在 Kudo 主控台中直接提供。 您可以直接與它連線，不需要使用入口網站。 如果應用程式是裝載在 ILB ASE 中，請使用發行認證登入。 存取 ILB ASE 所裝載應用程式之 SCM 網站的 URL 具有以下格式： 

```
<appname>.scm.<domain name the ILB ASE was created with> 
```

如果 ILB ASE 的網域名稱為 *contoso.net* 中，且應用程式名稱為 *testapp*，則應用程式已連線至 *testapp.contoso.net*。 與它搭配的 SCM 網站已經連線至 *testapp.scm.contoso.net*。

## <a name="functions-and-web-jobs"></a>函式和 Web 工作 ##

函式和 Web 工作都相依於 SCM 網站，但受到支援可用於入口網站，即使您的應用程式在 ILB ASE 中，只要您的瀏覽器可以到達 SCM 網站就沒問題。  如果您搭配 ILB ASE 使用自我簽署憑證，您必須啟用瀏覽器以信任該憑證。  若是 IE 和 Edge，這表示憑證必須在電腦信任存放區中。  如果您使用 Chrome，則表示您假設瀏覽器會直接到達 SCM 網站，而過早接受瀏覽器中的憑證。  最佳解決方法是使用瀏覽器信任鏈結中的商業憑證。  

## <a name="ase-ip-addresses"></a>ASE IP 位址 ##

ASE 有一些 IP 位址需要注意。 如下：

- **公用輸入 IP 位址**：用於外部 ASE 中的應用程式流量，以及外部 ASE 和 ILB ASE 中的管理流量。
- **輸出公用 IP**：用來作為 ASE 輸出連線離開 VNet 時的「來源」IP (不會透過 VPN 進行路由)。
- **ILB IP 位址**：如果您是使用 ILB ASE。
- **應用程式指派之以 IP 為主的 SSL 位址**：只有在使用外部 ASE 並已設定以 IP 為主的 SSL 時才能使用。

在 Azure 入口網站中，所有這些 IP 位址都可以很容易地在 ASEv2 的 ASE UI 中看出來。 如果您有 ILB ASE，系統便會列出 ILB 的 IP。

   > [!NOTE]
   > 只要您的 ASE 保持啟動且執行中，這些 IP 位址就不會變更。  如果您的 ASE 變成暫時停權且稍後還原，則 ASE 所使用的位址就會變更。 使 ASE 變成暫時停權的一般原因是您封鎖連入管理存取，或封鎖對 ASE 相依性的存取。 

![IP 位址][3]

### <a name="app-assigned-ip-addresses"></a>應用程式指派的 IP 位址 ###

在外部 ASE 的情況下，您可以將 IP 位址指派給個別的應用程式。 您無法在 ILB ASE 的情況下那樣做。 如需如何讓應用程式有自己 IP 位址的設定方式詳細資訊，請參閱[將現有的自訂 SSL 憑證繫結至 Azure Web Apps](../app-service-web-tutorial-custom-ssl.md)。

當應用程式有自己以 IP 為主的 SSL 位址時，ASE 會保留兩個連接埠以對應至該 IP 位址。 一個連接埠供 HTTP 流量使用，另一個連接埠則供 HTTPS 使用。 那些連接埠會列在 ASE UI 的 IP 位址區段中。 流量必須能夠從 VIP 觸達那些連接埠，否則會無法存取應用程式。 在您設定網路安全性群組 (NSG) 時，請務必記住這項重要需求。

## <a name="network-security-groups"></a>網路安全性群組 ##

[網路安全性群組][NSGs]提供控制 VNet 內網路存取的能力。 使用入口網站時，會有一個會拒絕一切內容的最低優先順序隱含拒絕規則。 您所建立的是您的允許規則。

在 ASE 中，您不會有存取用來裝載 ASE 本身之 VM 的存取權。 它們處於由 Microsoft 管理的訂用帳戶之中。 如果想要針對 ASE 上的應用程式限制存取，請在 ASE 子網路上設定 NSG。 這樣做時，請特別注意 ASE 相依性。 如果您封鎖任何相依性，ASE 會停止運作。

NSG 可以透過 Azure 入口網站或 PowerShell 來設定。 這裡的資訊僅針對 Azure 入口網站說明。 您會在入口網站中的 [網路] 底下，以最上層資源的形式建立及管理 NSG。

將輸入和輸出需求納入考量時，NSG 看起來應類似此範例中顯示的 NSG。 VNet 位址範圍為 _192.168.250.0/23_，且 ASE 所在的子網路為 _192.168.251.128/25_。

讓 ASE 能夠運作的前兩個輸入需求顯示在此範例中清單的最上方。 它們能啟用 ASE 管理，並允許 ASE 和自己通訊。 其他項目都是租用戶設定項目，而且可以管理對 ASE 裝載應用程式的網路存取。 

![輸入安全性規則][4]

預設規則可讓 VNet 中的 IP 與 ASE 子網路通訊。 另一個預設規則可讓負載平衡器 (也稱為公用 VIP) 和 ASE 通訊。 您可以選取 [新增] 圖示旁邊的 [預設規則] 來查看預設規則。 如果您在顯示的 NSG 規則後面加入拒絕其他任何內容的規則，便可以防止 VIP 和 ASE 之間產生流量。 若要防止來自 VNet 內部的流量，請新增您自己的規則來允許輸入。 使用來源等於 AzureLoadBalancer，目的地為 **Any**，以及 **\*** 的連接埠範圍。 由於 NSG 規則是套用至 ASE 子網路，因此不需要特別指定目的地。

如果指派 IP 位址給應用程式，請確定維持開啟連接埠。 若要查看連接埠，請選取 [App Service Environment] > [IP 位址]。  

下列輸出規則中顯示的所有項目都是需要的項目，但不包含最後一個項目。 它們可啟用針對本文章之前所提到之 ASE 相依性的網路存取。 如果封鎖它們任何一項，ASE 會停止運作。 清單中的最後一個項目可讓 ASE 和 VNet 中的其他資源通訊。

![輸出安全性規則][5]

定義 NSG 之後，請將它們指派給 ASE 所在的子網路。 如果您不記得 ASE VNet 或子網路，可以從 ASE 入口網站頁面查看。 若要將 NSG 指派給子網路，請移至子網路 UI 並選取 NSG。

## <a name="routes"></a>路由 ##

強制通道就是您在 VNet 中設定路由，讓輸出流量不會直接流向網際網路，而是流向其他地方，像是 ExpressRoute 閘道、虛擬應用裝置。  如果需要以此方式設定 ASE，請閱讀[為 App Service Environment 設定強制通道][forcedtunnel]文件。  這份文件會告訴您可與 ExpressRoute 和強制通道搭配使用的選項。

當您在入口網站中建立 ASE 時，我們也會在隨著 ASE 建立的子網路上建立一組路由資料表。  這些路由只單純指示直接將輸出流量傳送至網際網路。  
若要手動建立路由，請依照下列步驟執行︰

1. 移至 Azure 入口網站。 選取 [網路] > [路由表]。

2. 在和您 VNet 相同的區域內建立一個新的路由表。

3. 從您的路由表 UI 內，選取 [路由] > [新增]。

4. 將 [下一個躍點類型] 設為 [網際網路]，將 [位址首碼] 設為 **0.0.0.0/0**。 選取 [ **儲存**]。

    您就會看到類似以下的畫面：

    ![功能性路由][6]

5. 建立新的路由表之後，請移至包含您 ASE 的子網路。 從在入口網站取得的清單中選取您的路由表。 儲存變更之後，應該就會看見 NSG 和路由已註明了您的子網路。

    ![NSG 和路由][7]

## <a name="service-endpoints"></a>服務端點 ##

服務端點可讓您將多租用戶服務的存取權限制於一組 Azure 虛擬網路和子網路。 您可以在[虛擬網路服務端點][serviceendpoints]文件中深入了解服務端點。 

當您在資源上啟用服務端點時，所建立的路由具有高於所有其他路由的優先順序。 如果您使用服務端點搭配強制通道 ASE，Azure SQL 和 Azure 儲存體管理流量不會使用強制通道。 

透過 Azure SQL 執行個體在子網路上啟用服務端點時，從該子網路連線的所有 Azure SQL 執行個體都必須啟用服務端點。 如果您想要從相同的子網路存取多個 Azure SQL 執行個體，您就無法在一個 Azure SQL 執行個體啟用服務端點，而不要在另一個執行個體上啟用。 Azure 儲存體與 Azure SQL 的運作方式不同。 當您使用 Azure 儲存體啟用服務端點時，您會封鎖您的子網路存取該資源，，但仍可存取其他 Azure 儲存體帳戶 (即使它們未啟用服務端點)。  

![服務端點][8]

<!--Image references-->
[1]: ./media/network_considerations_with_an_app_service_environment/networkase-overflow.png
[2]: ./media/network_considerations_with_an_app_service_environment/networkase-overflow2.png
[3]: ./media/network_considerations_with_an_app_service_environment/networkase-ipaddresses.png
[4]: ./media/network_considerations_with_an_app_service_environment/networkase-inboundnsg.png
[5]: ./media/network_considerations_with_an_app_service_environment/networkase-outboundnsg.png
[6]: ./media/network_considerations_with_an_app_service_environment/networkase-udr.png
[7]: ./media/network_considerations_with_an_app_service_environment/networkase-subnet.png
[8]: ./media/network_considerations_with_an_app_service_environment/serviceendpoint.png

<!--Links-->
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
[mobileapps]: ../../app-service-mobile/app-service-mobile-value-prop.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: http://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/resource-group-overview.md
[ConfigureSSL]: ../web-sites-purchase-ssl-web-site.md
[Kudu]: http://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../application-gateway/application-gateway-web-application-firewall-overview.md
[ASEManagement]: ./management-addresses.md
[serviceendpoints]: ../../virtual-network/virtual-network-service-endpoints-overview.md
[forcedtunnel]: ./forced-tunnel-support.md
[serviceendpoints]: ../../virtual-network/virtual-network-service-endpoints-overview.md
