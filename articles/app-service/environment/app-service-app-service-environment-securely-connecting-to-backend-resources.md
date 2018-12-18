---
title: 安全地從 App Service 環境連接到後端資源
description: 了解如何安全地從 App Service 環境連接到後端資源。
services: app-service
documentationcenter: ''
author: stefsch
manager: erikre
editor: ''
ms.assetid: f82eb283-a6e7-4923-a00b-4b4ccf7c4b5b
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/04/2016
ms.author: stefsch
ms.openlocfilehash: 1732e6778febac60a25da74c330cb3d3da94154d
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45580037"
---
# <a name="securely-connecting-to-backend-resources-from-an-app-service-environment"></a>安全地從 App Service 環境連接到後端資源
## <a name="overview"></a>概觀
因為 App Service 環境一律會在 Azure Resource Manager 虛擬網路或者傳統式部署模型[虛擬網路][virtualnetwork]兩者之一中建立，從 App Service 環境傳出至其他後端資源的連線可以獨佔方式透過虛擬網路傳送。  在 2016 年 6 月所進行的最新變更之後，ASE 也可以部署到使用公用位址範圍或 RFC1918 位址空間 (也就是私人位址) 的虛擬網路。  

例如，SQL Server 可能會在已鎖定連接埠 1433 的虛擬機器叢集上執行。  此端點可能已納入 ACL，只允許從相同虛擬網路上的其他資源進行存取。  

另一個例子則是，敏感性端點可能會執行內部部署並透過[站台對站台][SiteToSite]或 [Azure ExpressRoute][ExpressRoute] 連線至 Azure。  因此，只有虛擬網路中連接到站台對站台或 ExpressRoute 通道的資源能夠存取內部部署端點。

在上述這些案例中，在 App Service 環境上執行的應用程式將能夠安全地連接到各種伺服器和資源。  從 App Service 環境中執行之應用程式送至相同虛擬網路中私密端點 (或連接到相同的虛擬網路) 的輸出流量，只會透過虛擬網路傳送。  送至私密端點的輸出流量不會透過公用網際網路傳送。

從 App Service 環境輸出至虛擬網路內端點的流量有一點值得注意。  App Service 環境無法連線到與 App Service 環境位於「相同」  子網路的虛擬機器端點。  只要 App Service 環境是部署到保留給 App Service 環境專用的子網路中，這通常應該不致於構成問題。

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="outbound-connectivity-and-dns-requirements"></a>輸出連線和 DNS 需求
為了讓 App Service 環境正確運作，它需要不同端點的輸出存取權。 [ExpressRoute 的網路組態](app-service-app-service-environment-network-configuration-expressroute.md#required-network-connectivity) 文章的＜需要的網路連線＞一節中有提供 ASE 所使用的外部端點完整清單。

App Service 環境需要針對虛擬網路設定的有效 DNS 基礎結構。  如果 DNS 設定在建立 App Service 環境之後因為任何原因而變更，開發人員可以強制 App Service 環境挑選新的 DNS 組態。  使用位於入口網站中 [App Service 環境管理] 刀鋒視窗頂端的 [重新啟動] 圖示觸發輪流環境重新開機，會導致環境挑選新的 DNS 組態。

也建議事先在虛擬網路上設定任何自訂 DNS 伺服器，再建立 App Service 環境。  如果在建立 App Service 環境時變更虛擬網路的 DNS 組態，則會導致 App Service 環境建立程序失敗。  同樣地，若自訂 DNS 伺服器存在於 VPN 閘道的另一端，且 DNS 伺服器無法連線或使用，則 App Service 環境建立程序也會失敗。

## <a name="connecting-to-a-sql-server"></a>連接至 SQL Server
常見的 SQL Server 組態會有在連接埠 1433 上接聽的端點：

![SQL Server 端點][SqlServerEndpoint]

有兩種方法可限制送至此端點的流量：

* [網路存取控制清單][NetworkAccessControlLists] (網路 ACL)
* [網路安全性群組][NetworkSecurityGroups]

## <a name="restricting-access-with-a-network-acl"></a>利用網路 ACL 限制存取
使用網路存取控制清單可以保護連接埠 1433。  下列範例將源自虛擬網路內部的用戶端位址列入白名單，並封鎖對所有其他用戶端的存取。

![網路存取控制清單範例][NetworkAccessControlListExample]

在與 SQL Server 相同虛擬網路的 App Service 環境中執行的所有應用程式，都將能夠使用 SQL Server 虛擬機器的 **VNet 內部** IP 位址連接到 SQL Server 執行個體。  

下列連接字串範例會使用其私密 IP 位址參考 SQL Server。

    Server=tcp:10.0.1.6;Database=MyDatabase;User ID=MyUser;Password=PasswordHere;provider=System.Data.SqlClient

雖然虛擬機器也有公用端點，但使用公用 IP 位址的連接嘗試將會因為網路 ACL 而遭到拒絕。 

## <a name="restricting-access-with-a-network-security-group"></a>利用網路安全性群組限制存取
另一種保護存取安全的方法是利用網路安全性群組。  網路安全性群組可以套用到個別的虛擬機器，或含有虛擬機器的子網路。

首先需要建立網路安全性群組：

    New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"

利用網路安全性群組來限制僅只存取 VNet 內部流量極為容易。  網路安全性群組中的預設規則只允許從相同虛擬網路中的其他網路用戶端存取。

因此鎖定 SQL Server 的存取權，就如同將網路安全性群組及其預設規則套用到執行 SQL Server 的虛擬機器或含有虛擬機器的子網路一樣簡單。

下列範例將網路安全性群組套用至包含的子網路：

    Get-AzureNetworkSecurityGroup -Name "testNSGExample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-1'

最終結果是一組可封鎖外部存取，同時允許 VNet 內部存取的安全性規則：

![預設網路安全性規則][DefaultNetworkSecurityRules]

## <a name="getting-started"></a>開始使用
若要開始使用 App Service Environment，請參閱 [App Service Environment 簡介][IntroToAppServiceEnvironment]

如需控制 App Service 環境輸入流量的詳細資訊，請參閱[控制 App Service 環境的輸入流量][ControlInboundASE]

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[virtualnetwork]: https://azure.microsoft.com/documentation/articles/virtual-networks-faq/
[ControlInboundTraffic]:  app-service-app-service-environment-control-inbound-traffic.md
[SiteToSite]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-multi-site
[ExpressRoute]: http://azure.microsoft.com/services/expressroute/
[NetworkAccessControlLists]: https://azure.microsoft.com/documentation/articles/virtual-networks-acl/
[NetworkSecurityGroups]: https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[IntroToAppServiceEnvironment]:  app-service-app-service-environment-intro.md
[ControlInboundASE]:  app-service-app-service-environment-control-inbound-traffic.md

<!-- IMAGES -->
[SqlServerEndpoint]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/SqlServerEndpoint01.png
[NetworkAccessControlListExample]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/NetworkAcl01.png
[DefaultNetworkSecurityRules]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/DefaultNetworkSecurityRules01.png 
