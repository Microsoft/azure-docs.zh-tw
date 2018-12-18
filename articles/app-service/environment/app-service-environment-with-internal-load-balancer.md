---
title: 搭配 App Service 環境建立及使用內部負載平衡器 | Microsoft Docs
description: 搭配 ILB 建立及使用 ASE
services: app-service
documentationcenter: ''
author: ccompy
manager: stefsch
editor: ''
ms.assetid: ad9a1e00-d5e5-413e-be47-e21e5b285dbf
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.openlocfilehash: f7c94b790c6aa7c75c62fd05671f016b7185b2a2
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29388810"
---
# <a name="using-an-internal-load-balancer-with-an-app-service-environment"></a>搭配 App Service 環境使用內部負載平衡器

> [!NOTE] 
> 這篇文章是關於 App Service 環境 v1。 有較新版本的 App Service 環境，更易於使用，並且可以在功能更強大的基礎結構上執行。 若要深入了解新版本，請從 [App Service 環境簡介](intro.md)開始。
>

App Service 環境 (ASE) 功能是 Azure App Service 的進階服務選項，可提供多租用戶戳記中不提供的增強式設定功能。 ASE功能基本上會在您的 Azure 虛擬網路 (VNet) 中部署 Azure App Service。 若要更深入了解 App Service Environment 所提供的功能，請閱讀[什麼是 App Service Environment][WhatisASE] 文件。 如果您不了解在 VNet 中操作的優點，請閱讀 [Azure 虛擬網路常見問題集][virtualnetwork]。 

## <a name="overview"></a>概觀
ASE 可以使用網際網路可存取的端點或您 Vnet 中的 IP 位址加以部署。 為了將 IP 位址設定為 VNet 位址，您必須搭配內部負載平衡器 (ILB) 來部署您的 ASE。 當您的 ASE 是使用 ILB 設定時，您要提供：

* 您自己的網域或子網域。 為了能順利進行，本文件假設是子網域，但是您還是可以設定。 
* 針對 HTTPS 使用的憑證
* 您子網域的 DNS 管理。 

相對的，您可以執行以下動作：

* 在您可以透過「端對端」或 ExpressRoute VPN 存取的雲端，安全地裝載內部網路應用程式 (例如企業營運應用程式)
* 在雲端裝載未在公用 DNS 伺服器中列出的 app
* 建立與網際網路隔離，且您的前端 app 可以安全地與之整合的後端應用程式

#### <a name="disabled-functionality"></a>已停用的功能
當您使用 ILB ASE 時，有一些動作您無法執行。 這些動作包括︰

* 使用 IPSSL
* 將 IP 位址指派給特定 app
* 透過入口網站購買憑證並搭配 app 使用。 您當然也可以直接透過「憑證授權單位」取得憑證並搭配您的 app 使用，但是無法透過 Azure 入口網站這樣做。

## <a name="creating-an-ilb-ase"></a>建立 ILB ASE
建立 ILB ASE 通常與建立 ASE 沒有太大差異。 如需有關建立 ASE 的深入討論，請參閱[如何建立 App Service 環境][HowtoCreateASE]。 在 ASE 建立期間建立 VNet 或選取既存的 VNet 之間，建立 ILB ASE 的程序是相同的。 若要建立 ILB ASE： 

1. 在 Azure 入口網站中，選取 [建立資源] -> [Web + 行動] -> [App Service 環境]。
2. 選取您的訂用帳戶。
3. 選取或建立資源群組。
4. 選取或建立 VNet。
5. 建立子網路 (如果選取 VNet)。
6. 選取 [虛擬網路/位置] -> [VNet 設定]，並將 [VIP 類型] 設定為 [內部]。
7. 提供子網域名稱 (此名稱是在此 ASE 中建立的 app 所使用的子網域)。
8. 選取 [確定]，然後選取 [建立]。

![][1]

在 [虛擬網路] 窗格中有 [VNet 設定] 選項，可讓您選取外部 VIP 或 VIP 內部。 預設為「外部」。 如果您將其設為 [外部]，您的 ASE 會使用一個網際網路可存取的 VIP。 如果您選取 [內部]，您的 ASE 會使用您的 VNet 內 IP 位址搭配 ILB 進行設定。 

選取內部之後，將會移除把更多 IP 位址新增至您 ASE 的功能，取而代之的是您必須提供 ASE 的子網域。 在使用外部 VIP 的 ASE 中，ASE 的名稱會在子網域中用於在該 ASE 中建立的 app。 如果您的 ASE 名為 contosotest，而您在該 ASE 中的應用程式名為 mytest，子網域的格式就會是 contosotest.p.azurewebsites.net，該應用程式的 URL 則會是 mytest.contosotest.p.azurewebsites.net。 如果您將 VIP 類型設定為 [內部]，您的 ASE 名稱不會在 ASE 的子網域中使用。 您可以明確地指定子網域。 如果您的子網域是 contoso.corp.net 而您在該 ASE 中建立一個名為 timereporting 的應用程式，該應用程式的 URL 會是 timereporting.contoso.corp.net。

## <a name="apps-in-an-ilb-ase"></a>ILB ASE 中的 App
在 ILB ASE 中建立 app，通常與在 ASE 中建立 app 相同。 

1. 在 Azure 入口網站中，選取 [建立資源] -> [Web + 行動] -> [Web] 或 [行動] 或 [API 應用程式]。
2. 輸入應用程式的名稱。
3. 選取您的訂用帳戶。
4. 選取或建立資源群組。
5. 選取或建立 App Service 方案 (ASP)。 如果是建立新的 ASP，請選取您的 ASE 作為位置，並選取您希望在其中建立 ASP 的背景工作角色集區。 當您建立 ASP 時，可以選取您的 ASE 作為位置與背景工作角色集區。 當您指定 app 的名稱時，您會看見您 app 名稱底下的子網域會由您 ASE 的子網域取代。 
6. 選取 [建立] 。 如果您希望 app 顯示在儀表板上，請務必選取 [釘選到儀表板] 核取方塊。 

![][2]

在 app 名稱底下，子網域名稱會更新，以反映您的 ASE 子網域。 

## <a name="post-ilb-ase-creation-validation"></a>ILB ASE 建立後驗證
ILB ASE 與非 ILB ASE 稍微有些不同。 如先前所述，您必須管理您自己的 DNS，而且您也必須提供您自己的 HTTPS 連線憑證。 

建立您的 ASE 之後，您會注意到子網域顯示您所指定的子網域，且 [設定] 功能表中會有一個稱為 [ILB 憑證] 的新項目。 ASE 是使用可易於測試 HTTPS 的自我簽署憑證建立。 入口網站會讓您知道您需要針對 HTTPS 提供自己的憑證，但這是要鼓勵您擁有子網域附帶的憑證。 

![][3]

如果您只是要測試而且不知道如何建立憑證，您可以使用 IIS MMC 主控台應用程式來建立自我簽署憑證。 建立之後，您可以將它匯出為 .pfx 檔案，然後在 ILB 憑證 UI 中上傳。 當您存取使用自我簽署憑證保護的網站時，您的瀏覽器會發出警告指出您正在存取的網站不安全，因為無法驗證憑證。 如果您想要避免這個警告產生，您將需要一個符合您的子網域、具有您的瀏覽器已識別的信任鏈結，並且已經正確簽署的憑證。

![][6]

如果您想要使用您自己的憑證嘗試流程，並測試對 ASE 的 HTTP 和 HTTPS 存取：

1. 在建立 ASE 之後，移至 ASE UI ([ASE] -> [設定] -> [ILB 憑證])。
2. 選取憑證 .pfx 檔案並提供密碼，來設定 ILB 憑證。 這個步驟需要一些時間來處理，且會出現調整作業正在進行中的訊息。
3. 取得您 ASE 的 ILB 位址 ([ASE] -> [屬性] -> [虛擬 IP 位址])。
4. 建立後，在 ASE 中建立 Web 應用程式。 
5. 如果您在該 VNET 中沒有 VM 的話，請建立一個 (不是在與 ASE 相同的子網路中，否則會無法運作)。
6. 設定您子網域的 DNS。 您可以在您 DNS 中使用萬用字元搭配您的子網域，或者如果您想要執行一些簡單測試，請編輯您 VM 上的主機檔案來將 Web 應用程式名稱設定為 VIP IP 位址。 如果您的 ASE 具有子網域名稱 .ilbase.com 且 Web 應用程式名稱為 mytestapp，它將會定址為 mytestapp.ilbase.com，在您的主機檔案中設定此位址。 (在 Windows 上，主機檔案位於 C:\Windows\System32\drivers\etc\)
7. 在該 VM 上使用瀏覽器並移至 http://mytestapp.ilbase.com (或者您的 Web 應用程式名稱與您的子網域)。
8. 在該 VM 上使用瀏覽器並移至 https://mytestapp.ilbase.com   。如果使用自我簽署憑證，就必須接受安全性不足這件事。 

您的 ILB 的 IP 位址會在您的 [屬性] 中列出為 [虛擬 IP 位址]。

![][4]

## <a name="using-an-ilb-ase"></a>使用 ILB ASE
#### <a name="network-security-groups"></a>網路安全性群組
ILB ASE 可讓您的應用程式與網路隔離。 app 無法透過網際網路存取或讓 app 在網際網路中完全找不到。 這個做法非常適合用來裝載內部網路網站，例如企業營運應用程式。 當您需要更進一步地限制存取時，您仍然可以使用「網路安全性群組 (NSG)」來控制網路層級的存取。 

如果您想要使用 NSG 來進一步限制存取，您必須確定您不會中斷 ASE 運作所需的通訊。 即使 HTTP/HTTPS 存取只會透過 ASE 所使用的 ILB 進行，ASE 仍需依賴 VNet 外部資源。 若要查看仍需要何種網路存取權，請參閱[控制 App Service 環境的輸入流量][ControlInbound]和[使用 ExpressRoute 的 App Service 環境的網路組態詳細資料][ExpressRoute]。 

若要設定您的 NSG，您必須知道 Azure 所使用的 IP 位址，以管理您的 ASE。 如果該 IP 位址提出網際網路要求，它也會成為您 ASE 的輸出 IP 位址。 在 ASE 的存留期內，ASE 的輸出 IP 位址仍維持不變。 如果您刪除並重建 ASE，您會收到新的 IP 位址。 若要尋找此 IP 位址，請移至 [設定] -> [屬性]，尋找 [輸出 IP 位址]。 

![][5]

#### <a name="general-ilb-ase-management"></a>一般 ILB ASE 管理
管理 ILB ASE 通常大部分與管理 ASE 相同。 您必須相應增加您的背景工作集區來裝載更多 ASP 執行個體，並相應增加您的前端伺服器，以處理增加的 HTTP/HTTPS 流量。 如需管理 ASE 組態的一般資訊，請參閱[設定 App Service 環境][ASEConfig]。 

其他管理項目是憑證管理和 DNS 管理。 在建立 ILB ASE 之後，您必須取得並上傳針對 HTTPS 使用的憑證，並在它到期之前將它取代。 因為 Azure 擁有基底網域，所以可以使用外部 VIP 提供 ASE 的憑證。 因為 ILB ASE 所使用的子網域可以是任何項目，所以您必須提供您自己的 HTTPS 憑證。 

#### <a name="dns-configuration"></a>DNS 組態
使用外部 VIP 時，DNS 是由 Azure 管理。 在您 ASE 中建立的任何 app 都會自動新增至 Azure DNS，這是一個公用 DNS。 在 ILB ASE 中，您必須管理您自己的 DNS。 針對指定的子網域 (例如 contoso.corp.net)，您必須為以下項目建立指向您的 ILB 位址的 DNS A 記錄︰

    * 
    *.scm ftp 發佈 


## <a name="getting-started"></a>開始使用
若要開始使用 App Service Environment，請參閱 [App Service Environment 簡介][WhatisASE]

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!--Image references-->
[1]: ./media/app-service-environment-with-internal-load-balancer/ilbase-createilbase.png
[2]: ./media/app-service-environment-with-internal-load-balancer/ilbase-createapp.png
[3]: ./media/app-service-environment-with-internal-load-balancer/ilbase-newase.png
[4]: ./media/app-service-environment-with-internal-load-balancer/ilbase-vip.png
[5]: ./media/app-service-environment-with-internal-load-balancer/ilbase-externalvip.png
[6]: ./media/app-service-environment-with-internal-load-balancer/ilbase-ilbcertificate.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[ControlInbound]: app-service-app-service-environment-control-inbound-traffic.md
[virtualnetwork]: https://azure.microsoft.com/documentation/articles/virtual-networks-faq/
[AppServicePricing]: http://azure.microsoft.com/pricing/details/app-service/
[ASEAutoscale]: app-service-environment-auto-scale.md
[ExpressRoute]: app-service-app-service-environment-network-configuration-expressroute.md
[vnetnsgs]: http://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[ASEConfig]: app-service-web-configure-an-app-service-environment.md
