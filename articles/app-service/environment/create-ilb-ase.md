---
title: 建立及使用內部負載平衡器與 Azure App Service Environment
description: 如何建立及使用與網際網路隔離的 Azure App Service Environment 的詳細資料
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: 0f4c1fa4-e344-46e7-8d24-a25e247ae138
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 06/12/2018
ms.author: ccompy
ms.custom: mvc
ms.openlocfilehash: e9d1f77a85d4b5cfb5bb7d3cb80380be3c79315d
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44378273"
---
# <a name="create-and-use-an-internal-load-balancer-with-an-app-service-environment"></a>建立及使用內部負載平衡器與 App Service Environment #

 Azure App Service Environment (ASE) 是將 Azure App Service 部署到客戶 Azure 虛擬網路 (VNet) 中子網路的一種部署。 有二種方法可以部署 App Service Environment (ASE)： 

- 使用外部 IP 位址上的 VIP，通常稱為「外部 ASE」。
- 使用內部 IP 位址上的 VIP，通常稱為 「ILB ASE」，因為內部端點是內部負載平衡器 (ILB)。 

本文說明如何建立 ILB ASE。 如需 ASE 的概觀，請參閱 [App Service Environment 簡介][Intro]。 若要深了解如何建立外部 ASE，請參閱[建立外部 ASE][MakeExternalASE]。

## <a name="overview"></a>概觀 ##

您可以使用網際網路可存取端點或是您的 VNet 中的 IP 位址來部署 ASE。 為了將 IP 位址設定為 VNet 位址，ASE 必須與 ILB 一起部署。 在部署 ASE 與 ILB 時，必須提供：

-   當您建立您的應用程式時，所使用的您自己的網域。
-   用於 HTTPS 的憑證。
-   您的網域的 DNS 管理。

相對的，您可以執行以下動作：

-   在您可以透過「端對端」或 Azure ExpressRoute VPN 存取的雲端，安全地裝載內部網路應用程式。
-   在雲端裝載未在公用 DNS 伺服器中列出的應用程式。
-   建立與網際網路隔離，且您的前端 app 可以安全地與之整合的後端應用程式。

### <a name="disabled-functionality"></a>已停用的功能 ###

使用 ILB ASE 時，有一些動作您無法執行：

-   使用以 IP 為主的 SSL。
-   將 IP 位址指派給特定應用程式。
-   透過 Azure 入口網站購買憑證並搭配應用程式使用。 您可以直接透過「憑證授權單位」取得憑證並搭配您的應用程式使用。 您無法透過 Azure 入口網站取得它們。

## <a name="create-an-ilb-ase"></a>建立 ILB ASE ##

若要建立 ILB ASE：

1. 在 Azure 入口網站中，選取 [建立資源] > [Web] > [App Service 環境]。

1. 選取您的訂用帳戶。

1. 選取或建立資源群組。

1. 選取或建立 VNet。

1. 如果您選取現有的 VNet，則需要建立子網路來存放 ASE。 確定將子網路的大小設為足以容納 ASE 的任何未來成長。 建議的大小是 `/24`，具有 256 個位址，而且可以處理最大大小的 ASE 和任何縮放需求。 

1. 選取 [虛擬網路/位置]  >  [虛擬網路設定]， 並將 [VIP 類型] 設定為 [內部]。

1. 輸入網域名稱。 這個網域會成為在此 ASE 中建立之應用程式所使用的網域。 有某些限制。 不能是：

    * net   

    * azurewebsites.net

    * p.azurewebsites.net

    * &lt;asename&gt;.p.azurewebsites.net

   有一項名為自訂網域名稱的功能，可讓您將現有的 DNS 名稱對應至 Web 應用程式。 您可以閱讀[將現有的 DNS 名稱對應至 Web 應用程式][customdomain]一文，以深入了解該功能。 應用程式使用的自訂網域名稱，與您的 ASE 使用的網域名稱不可重疊。 若 ILB ASE 的網域名稱為 contoso.com，則您的應用程式不能使用像這樣的自訂網域名稱：

    * www.contoso.com

    * abcd.def.contoso.com

    * abcd.contoso.com

   如果您知道您的應用程式自訂網域名稱，請為 ILB ASE 選擇不會與這些自訂網域名稱相衝突的網域。 在此範例中，您可以為您的 ASE 使用類似 contoso internal.com 的網域，因為不會與 contoso.com結尾的自訂網域名稱衝突。

1. 選取 [確定]，然後選取 [建立]。

    ![ASE 建立][1]

在 [虛擬網路] 刀鋒視窗中，會有一個 [虛擬網路設定] 選項。 您可以用來選擇 [外部 VIP] 或 [內部 VIP]。 預設為「外部」。 如果您選取 [外部]，您的 ASE 會使用一個網際網路可存取的 VIP。 如果您選取 [內部]，您的 ASE 會使用您的 VNet 內 IP 位址搭配 ILB 進行設定。

選取 [內部] 之後，系統會移除把更多 IP 位址新增至您 ASE 的功能。 取而代之的是您必須提供 ASE 的網域。 在使用外部 VIP 的 ASE 中，ASE 的名稱會在網域中用於在該 ASE 中建立的 app。

如果您將 [VIP 類型] 設定為 [內部]，您的 ASE 名稱不會在 ASE 的網域中使用。 您可以明確地指定網域。 如果您的網域是 contoso.corp.net 而您在該 ASE 中建立一個名為 timereporting 的應用程式，該應用程式的 URL 會是 timereporting.contoso.corp.net。


## <a name="create-an-app-in-an-ilb-ase"></a>在 ILB ASE 中建立應用程式： ##

在 ILB ASE 中建立應用程式的做法，與在 ASE 中建立應用程式相同。

1. 在 Azure 入口網站中，選取 [建立資源] >  [Web + 行動] > [Web 應用程式]。

1. 輸入應用程式的名稱。

1. 選取訂用帳戶。

1. 選取或建立資源群組。

1. 選取您的作業系統。 

    * 如果您想要使用自訂的 Docker 容器建立 Linux 應用程式，則可以使用[這裡][linuxapp]的指示以使用自己的容器。 

1. 選取或建立 App Service 方案。 如果您想建立新的 App Service 方案，請選取您的 ASE 作為位置。 選取您想要建立 App Service 方案的背景工作集區。 當您建立 App Service 方案時，選取您的 ASE 作為位置與背景工作角色集區。 指定應用程式的名稱時，會看見您的應用程式名稱底下的網域已被您的 ASE 網域取代。

1. 選取 [建立] 。 如果希望應用程式顯示在儀表板上，選取 [釘選到儀表板] 核取方塊。

    ![App Service 方案建立][2]

    在 [應用程式名稱] 底下，網域名稱會更新，以反映您的 ASE 網域。

## <a name="post-ilb-ase-creation-validation"></a>ILB ASE 建立後驗證 ##

ILB ASE 與非 ILB ASE 稍微有些不同。 如先前所述，您需要管理自己的 DNS。 您也必須提供您自己的 HTTPS 連線憑證。

建立 ASE 之後，網域名稱會顯示您所指定的網域。 在 [設定] 功能表中會出現新的項目 [ILB 憑證]。 ASE 使用憑證建立，該憑證未指定 ILB ASE 網域。 如果您使用 ASE 與該憑證，您的瀏覽器會告訴您它是無效的。 此憑證可讓您更輕鬆地測試 HTTPS，但您必須上傳您自己的繫結至 ILB ASE 網域的憑證。 無論您的憑證是自我簽署或是從憑證授權單位取得，這個步驟都是必要的。

![ILB ASE 網域名稱][3]

您的 ILB ASE 需要有效的 SSL 憑證。 使用內部憑證授權單位、向外部簽發者購買憑證、或使用自我簽署的憑證。 無論 SSL 憑證的來源，都需要正確設定下列憑證屬性︰

* **Subject**︰此屬性必須設為 *.your-root-domain-here。
* **Subject Alternative Name**︰此屬性必須同時包含 *.your-root-domain-here 和 *.scm.your-root-domain-here。 系統將使用 your-app-name.scm.your-root-domain-here 形式的位址，進行與每個應用程式相關聯的 SCM/Kudu 網站的 SSL 連線。

將 SSL 憑證轉換/儲存為 .pfx 檔案。 .pfx 檔案必須包含所有中繼和根憑證。 使用密碼保護其安全。

如果您想要建立自我簽署憑證，可以使用這裡的 PowerShell命令。 務必使用您的 ILB ASE 網域名稱，而不是 internal.contoso.com： 

    $certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"
    
    $certThumbprint = "cert:\localMachine\my\" +$certificate.Thumbprint
    $password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText
    
    $fileName = "exportedcert.pfx" 
    Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password

這些 PowerShell 命令產生的憑證會被瀏覽器標示，因為憑證不是由瀏覽器信任鏈結中的憑證授權單位建立。 若要取得瀏覽器信任的憑證，可向瀏覽器信任鏈結中的商業憑證授權單位購買。 

![設定 ILB 憑證][4]

若要上傳您自己的憑證並測試存取：

1. 建立 ASE 之後，移至 ASE UI。 選取 [ASE]  >  [設定]  >  [ILB 憑證]。

1. 若要設定 ILB 憑證，選取憑證的 .pfx 檔案並輸入密碼。 這個步驟需要一些處理時間。 會出現訊息指出上傳作業正在進行中。

1. 取得 ASE 的 ILB 位址。 選取 [ASE]  >  [屬性]  >  [虛擬 IP 位址]。

1. 建立您的 ASE 後，在 ASE 中建立 Web 應用程式。

1. 如果您在該 VNet 中還沒有 VM，則建立 VM。

    > [!NOTE] 
    > 請勿嘗試在與 ASE 相同的子網路中建立此 VM，因為會失敗或造成問題。
    >

1. 設定 ASE 網域的 DNS。 您可以在您的 DNS 中使用萬用字元搭配您的網域。 若要執行一些簡單測試，請編輯 VM 上的主機檔案來將 Web 應用程式名稱設定為 VIP IP 位址：

    a. 如果您的 ASE 網域名稱為 .ilbase.com，且您建立名為 mytestapp 的 Web 應用程式，則它將定址為 mytestapp.ilbase.com。 然後，您設定 mytestapp.ilbase.com 以解析 ILB 位址。 (在 Windows 上，主機檔案位於 _C:\Windows\System32\drivers\etc\_。)

    b. 若要測試 Web 部署發佈或存取進階主控台，建立 mytestapp.scm.ilbase.com 的記錄。

1. 在該 VM 上使用瀏覽器並移至 http://mytestapp.ilbase.com。 (或移至任何名稱含您的網域的 Web 應用程式。)

1. 在該 VM 上使用瀏覽器並移至 https://mytestapp.ilbase.com。 如果您使用自我簽署憑證，就必須接受安全性不足。

    您的 ILB IP 位址列在 [IP 位址] 底下。 此清單中也有外部 VIP 使用的 IP 位址以及用於輸入管理流量的 IP 位址。

    ![ILB IP 位址][5]

## <a name="web-jobs-functions-and-the-ilb-ase"></a>Web 工作、函式和 ILB ASE ##

ILB ASE 支援函式和 Web 工作，但若要讓入口網站可以使用，您必須具有 SCM 網站的網路存取。  這表示您的瀏覽器必須在主機上，或是在虛擬網路中或已連線到虛擬網路。  

當您在 ILB ASE 中使用 Azure Functions 時，可能會遇到錯誤，指出「我們無法立即擷取您的函式。 請稍後再試。」 由於 Functions UI 透過 HTTPS 利用 SCM 網站，而且根憑證不在瀏覽器信任鏈結中，因此會發生此錯誤。 Web 工作具有類似的問題。 若要避免此問題，您可以執行下列其中一個動作：

- 將憑證新增至您的信任憑證存放區。 這會解除封鎖 Edge 及 Internet Explorer。
- 先使用 Chrome 並前往 SCM 網站，接受不受信任的憑證，然後前往入口網站。
- 使用瀏覽器信任鏈結中的商業憑證。  這是最佳選擇。  

## <a name="dns-configuration"></a>DNS 組態 ##

當您使用外部 VIP，DNS 是由 Azure 管理。 在您 ASE 中建立的任何 app 都會自動新增至 Azure DNS，這是一個公用 DNS。 在 ILB ASE 中，您必須管理您自己的 DNS。 針對指定的網域 (例如 _contoso.net_)，您必須為下列項目，在 DNS 中建立指向您 ILB 位址的 DNS A 記錄︰

- *.contoso.net
- *.scm.contoso.net

如果 ILB ASE 網域用於此 ASE 之外的多項作業，您可能需要針對每個應用程式名稱進行 DNS 管理。 這個做法更有挑戰性，因為每當您建立應用程式時，必須將新的應用程式名稱新增至您的 DNS。 基於這個理由，建議使用專用網域。

## <a name="publish-with-an-ilb-ase"></a>使用 ILB ASE 發佈 ##

每一個建立的應用程式，都有兩個端點。 在 ILB ASE 中，您有 &lt;app name>.&lt;ILB ASE Domain> 和 &lt;app name>.scm.&lt;ILB ASE Domain>。 

SCM 網站名稱會帶您前往 Azure 入口網站中的 Kudu 主控台，稱為 [進階入口網站]。 Kudu 主控台可讓您檢視環境變數、探索磁碟、使用主控台等等。 如需詳細資訊，請參閱 [Azure App Service 的 Kudu 主控台][Kudu]。 

在多租用戶 App Service 和外部 ASE 中，Azure 入口網站與 Kudu 主控台之間有單一登入。 不過，對於 ILB ASE，您必須使用發佈認證來登入 Kudu 主控台。

網際網路型的 CI 系統 (如 GitHub 和 Azure DevOps)，依然可以搭配 ILB ASE 使用，前提是組建代理程式可以存取網際網路，而且位在與 ILB ASE 相同的網路中。 對於 Azure DevOps，如果建立組建代理程式的 VNET 與 ILB ASE 相同 (子網路不同沒關係)，就能從 Azure DevOps git 提取程式碼，並部署到 ILB ASE。 如果您不想要建立自己的組建代理程式，便需要使用採用提取模型的 CI 系統，如 Dropbox。

ILB ASE 中應用程式的發佈端點會使用用來建立 ILB ASE 的網域。 在應用程式的發行設定檔中，以及應用程式的入口網站刀鋒視窗中可以看到這個網域 (在 [概觀]  >  [基本資訊] 以及 [屬性] 中)。 如果您的 ILB ASE 具有子網域 contoso.net，且應用程式名為 mytest，則要在 FTP 中使用 mytest.contoso.net，在 Web 部署中使用 mytest.scm.contoso.net。

## <a name="couple-an-ilb-ase-with-a-waf-device"></a>將 ILB ASE 與 WAF 裝置耦合 ##

Azure App Service 提供許多安全性措施來保護您的系統。 它們也可協助您判斷應用程式是否遭到駭客入侵。 Web 應用程式的最佳保護是耦合裝載平台，例如 Azure App Service 與 Web 應用程式防火牆 (WAF)。 由於 ILB ASE 具有與網路隔離的應用程式端點，因此適合這樣的使用方式。

若要深入了解如何設定 ILB ASE 與 WAF 裝置，請參閱[使用 App Service Environment 設定 Web 應用程式防火牆][ASEWAF]。 此文章說明如何搭配使用您的 ASE 與 Barracuda 虛擬應用裝置。 另一個選項是使用 Azure 應用程式閘道。 應用程式閘道會使用 OWASP 核心規則來保護後面的任何應用程式。 如需有關應用程式閘道的詳細資訊，請參閱 [Azure Web 應用程式防火牆簡介][AppGW]。

## <a name="get-started"></a>開始使用 ##

* 若要開始使用ASE，請參閱 [App Service Environment 簡介][Intro]。
 
<!--Image references-->
[1]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-network.png
[2]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-webapp.png
[3]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-certificate.png
[4]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-certificate2.png
[5]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-ipaddresses.png

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
[webapps]: ../app-service-web-overview.md
[mobileapps]: ../../app-service-mobile/app-service-mobile-value-prop.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: http://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/resource-group-overview.md
[ConfigureSSL]: ../web-sites-purchase-ssl-web-site.md
[Kudu]: http://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../application-gateway/application-gateway-web-application-firewall-overview.md
[customdomain]: ../app-service-web-tutorial-custom-domain.md
[linuxapp]: ../containers/app-service-linux-intro.md
