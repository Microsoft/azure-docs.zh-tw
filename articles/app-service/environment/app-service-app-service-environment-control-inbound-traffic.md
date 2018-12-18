---
title: 如何控制 App Service 環境的輸入流量
description: 了解如何設定網路安全性規則，以控制 App Service 環境的輸入流量。
services: app-service
documentationcenter: ''
author: ccompy
manager: erikre
editor: ''
ms.assetid: 4cc82439-8791-48a4-9485-de6d8e1d1a08
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/11/2017
ms.author: stefsch
ms.openlocfilehash: ed72bf3202d6cb2d2161bc0df693d3e6a1fc58ef
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22987003"
---
# <a name="how-to-control-inbound-traffic-to-an-app-service-environment"></a>如何控制 App Service 環境的輸入流量
## <a name="overview"></a>概觀
您可以在 Azure Resource Manager 虛擬網路**或**傳統部署模型[虛擬網路][virtualnetwork]中建立 App Service Environment。  建立 APP Service 環境時，可以定義新的虛擬網路和新的子網路。  或者亦可在先前既存的虛擬網路和既存的子網路中建立 APP Service 環境。  在 2016 年 6 月所進行的變更之後，ASE 也可以部署到使用公用位址範圍或 RFC1918 位址空間 (也就是私人位址) 的虛擬網路。  如需有關建立 App Service Environment 的詳細資訊，請參閱[如何建立 App Service Environment][HowToCreateAnAppServiceEnvironment]。

App Service 環境必須一律建立於子網路中，因為子網路可提供網路界限以便用來鎖定上游裝置和服務背後的輸入流量，因此只接受來自特定上游 IP 位址的 HTTP 和 HTTPS 流量。

您可以使用[網路安全性群組][NetworkSecurityGroups]來控制子網路上的輸入和輸出網路流量。 控制輸入流量時，需要在網路安全性群組中建立網路安全性規則，然後將網路安全性群組指派給包含 App Service 環境的子網路。

將網路安全性群組指派給子網路後，會根據網路安全性群組中定義的允許和拒絕規則，允許/封鎖 App Service 環境中應用程式的輸入流量。

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="inbound-network-ports-used-in-an-app-service-environment"></a>App Service Environment 中使用的輸入網路連接埠
利用網路安全性群組鎖定輸入網路流量之前，請務必知道 App Service 環境所用的必要和選用網路連接埠集合。  意外關閉送至某些連接埠的流量，可能會導致在 App Service 環境中喪失功能。

以下是 App Service 環境所使用的連接埠清單。 除非明確註明，否則所有連接埠為 **TCP**：

* 454：Azure 基礎結構透過 SSL 用來管理和維護 App Service 環境的**必要連接埠**。  不會封鎖對此連接埠的流量。  此連接埠一律繫結至 ASE 的公用 VIP。
* 455：Azure 基礎結構透過 SSL 用來管理和維護 App Service 環境的**必要連接埠**。  不會封鎖對此連接埠的流量。  此連接埠一律繫結至 ASE 的公用 VIP。
* 80：對於在 App Service 環境的 App Service 方案中執行的應用程式，其輸入 HTTP 流量的預設連接埠。  在啟用 ILB 的 ASE 上，此連接埠繫結至 ASE 的 ILB 位址。
* 443：對於在 App Service 環境的 App Service 方案中執行的應用程式，其輸入 SSL 流量的預設連接埠。  在啟用 ILB 的 ASE 上，此連接埠繫結至 ASE 的 ILB 位址。
* 21：FTP 的控制通道。  如果未使用 FTP，就可以安全地封鎖此連接埠。  在啟用 ILB 的 ASE 上，此連接埠可以繫結至 ASE 的 ILB 位址。
* 990：FTPS 的控制通道。  如果未使用 FTPS，就可以安全地封鎖此連接埠。  在啟用 ILB 的 ASE 上，此連接埠可以繫結至 ASE 的 ILB 位址。
* 10001-10020：FTP 的資料通道。  如同控制通道，若未使用 FTP，即可安全地封鎖這些連接埠。  在啟用 ILB 的 ASE 上，此連接埠可以繫結至 ASE 的 ILB 位址。
* 4016：用於 Visual Studio 2012 的遠端偵錯。  如果未使用此功能，就可以安全地封鎖此連接埠。  在啟用 ILB 的 ASE 上，此連接埠繫結至 ASE 的 ILB 位址。
* 4018：用於 Visual Studio 2013 的遠端偵錯。  如果未使用此功能，就可以安全地封鎖此連接埠。  在啟用 ILB 的 ASE 上，此連接埠繫結至 ASE 的 ILB 位址。
* 4020：用於 Visual Studio 2015 的遠端偵錯。  如果未使用此功能，就可以安全地封鎖此連接埠。  在啟用 ILB 的 ASE 上，此連接埠繫結至 ASE 的 ILB 位址。

## <a name="outbound-connectivity-and-dns-requirements"></a>輸出連線和 DNS 需求
若要讓 App Service Environment 正確運作，它還需要不同端點的輸出存取權。 [ExpressRoute 的網路組態](app-service-app-service-environment-network-configuration-expressroute.md#required-network-connectivity) 文章的＜需要的網路連線＞一節中有提供 ASE 所使用的外部端點完整清單。

App Service 環境需要針對虛擬網路設定的有效 DNS 基礎結構。  如果 DNS 設定在建立 App Service 環境之後因為任何原因而變更，開發人員可以強制 App Service 環境挑選新的 DNS 組態。  若是使用位於 [Azure 入口網站][NewPortal]中 [App Service Environment 管理] 刀鋒視窗頂端的 [重新啟動] 圖示來觸發輪流環境重新開機，就會導致環境挑選新的 DNS 組態。

也建議事先在虛擬網路上設定任何自訂 DNS 伺服器，再建立 App Service 環境。  如果在建立 App Service 環境時變更虛擬網路的 DNS 組態，則會導致 App Service 環境建立程序失敗。  同樣地，若自訂 DNS 伺服器存在於 VPN 閘道的另一端，且 DNS 伺服器無法連線或使用，則 App Service 環境建立程序也會失敗。

## <a name="creating-a-network-security-group"></a>建立網路安全性群組
如需有關網路安全性群組如何運作的完整詳細資料，請參閱下列[資訊][NetworkSecurityGroups]。  下面的「Azure 服務管理」範例說明網路安全性群組的重點，其中是將焦點放在設定網路安全群組並套用到包含 App Service Environment 的子網路。

**注意︰** 您可以使用 [Azure 入口網站](https://portal.azure.com) 或透過 Azure PowerShell，利用圖形方式設定網路安全性群組。

網路安全性群組首次會建立為與訂用帳戶相關聯的獨立實體。 由於網路安全性群組建立於 Azure 區域，所以請確保網路安全性群組建立於與 App Service 環境相同的區域中。

以下示範如何建立網路安全性群組：

    New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"

建立網路安全性群組後，會加入一或多個網路安全性規則。  由於這組規則會隨著時間改變，所以建議區隔用於規則優先順序的編號配置，以便陸續插入其他規則。

下列範例顯示的規則可明確授與以下連接埠的存取權：Azure 基礎結構管理和維護 App Service 環境所需的管理連接埠。  請注意，所有管理流量都會透過 SSL 傳送並受用戶端憑證保護，因此即使連接埠已開啟，Azure 管理基礎結構以外的任何實體都無法予以存取。

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "ALLOW AzureMngmt" -Type Inbound -Priority 100 -Action Allow -SourceAddressPrefix 'INTERNET'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '454-455' -Protocol TCP


鎖定對連接埠 80 和 443 的存取，以「隱藏」上游裝置或服務背後的 App Service 環境時，您必須知道上游 IP 位址。  例如，如果您使用 Web 應用程式防火牆 (WAF)，則 WAF 將會有自己的 IP 位址，以便在將流量 Proxy 處理至下游 App Service 環境時使用。  您必須在網路安全性規則的 *SourceAddressPrefix* 參數中使用此 IP 位址。

在下面範例中，明確允許來自特定上游 IP 位址的輸入流量。  位址 *1.2.3.4* 會做為上游 WAF 的 IP 位址預留位置。  變更此值，以符合您的上游裝置或服務所使用的位址。

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT HTTP" -Type Inbound -Priority 200 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT HTTPS" -Type Inbound -Priority 300 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP

如果需要 FTP 支援，可以使用下列規則做為範本，以授與對 FTP 控制連接埠和資料通道連接埠的存取權。  由於 FTP 是可設定狀態的通訊協定，所以您無法透過傳統 HTTP/HTTPS 防火牆或 Proxy 裝置路由傳送 FTP 流量。  在此情況下，您將需要將 *SourceAddressPrefix* 設定為不同的值 -例如，FTP 用戶端執行所在開發人員或部署電腦的 IP 位址範圍。 

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT FTPCtrl" -Type Inbound -Priority 400 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '21' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT FTPDataRange" -Type Inbound -Priority 500 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '10001-10020' -Protocol TCP

(**注意**：資料通道連接埠範圍可能會在預覽期間變更。)

如果使用 Visual Studio 遠端偵錯，則下列規則會示範如何授與存取權。  因為每個支援的 Visual Studio 版本使用不同的連接埠進行遠端偵錯，所以每個版本會有個別的規則。  如同 FTP 存取，遠端偵錯流量可能不會透過傳統 WAF 或 Proxy 裝置正確傳送。  可將 *SourceAddressPrefix* 改為設定成執行 Visual Studio 之開發人員電腦的 IP 位址範圍。

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2012" -Type Inbound -Priority 600 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4016' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2013" -Type Inbound -Priority 700 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4018' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2015" -Type Inbound -Priority 800 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4020' -Protocol TCP

## <a name="assigning-a-network-security-group-to-a-subnet"></a>將網路安全性群組指派給子網路
網路安全性群組有拒絕存取所有外部流量的預設安全性規則。  結合上述的網路安全性規則與用來封鎖輸入流量的預設安全性規則，結果就是只有來自與 *允許* 動作相關聯之來源位址範圍的流量，能夠傳送到在 App Service 環境中執行的應用程式。

在網路安全性群組填入安全性規則之後，必須將該群組指派給包含 App Service 環境的子網路。  指派命令會同時參考 App Service 環境所在的虛擬網路名稱，以及 App Service 環境建立所在的子網路名稱。  

下列範例顯示要指派給子網路和虛擬網路的網路安全性群組：

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-test'

網路安全性群組指派成功後 (指派是一項長時間執行的作業並可能需要幾分鐘才能完成)，只有符合 *允許* 規則的輸入流量才能成功抵達 App Service 環境中的應用程式。

基於完整性，下列範例示範如何移除進而取消網路安全性群組與子網路的關聯：

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Remove-AzureNetworkSecurityGroupFromSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-test'

## <a name="special-considerations-for-explicit-ip-ssl"></a>明確 IP-SSL 的特殊考量
如果為應用程式設定了明確的 IP-SSL 位址 (「僅」適用於具有公用 VIP 的 ASE)，而不是使用 App Service Environment 的預設 IP 位址，則 HTTP 和 HTTPS 流量都會透過一組與連接埠 80 和 443 不同的連接埠流向子網路。

每個 IP-SSL 位址使用的個別連接埠組合，可以在 App Service 環境的詳細資料 UX 刀鋒視窗的入口網站使用者介面中找到。  選取 [所有設定] --> [IP 位址]。  「IP 位址」刀鋒視窗會顯示 App Service 環境所有明確設定的 IP-SSL 位址，以及特殊連接埠配對，這些特殊連接埠配對則是用於路由與每個 IP-SSL 位址關聯的 HTTP 和 HTTPS 流量。  在網路安全性群組中設定規則時，必須針對 DestinationPortRange 參數使用此連接埠配對。

當 ASE 上的 app 設定為使用 IP-SSL 時，外部客戶將不會看到且不需要擔心特殊連接埠配對對應。  應用程式流量會正常流向設定的 IP-SSL 位址。  將流量路由到包含 ASE 的子網路時，在路由流量最後階段期間，特殊連接埠組合的轉譯會於內部自動發生。 

## <a name="getting-started"></a>開始使用
若要開始使用 App Service Environment，請參閱 [App Service Environment 簡介][IntroToAppServiceEnvironment]

如需有關將應用程式安全地從 App Service Environment 連接到後端資源的詳細資料，請參閱[安全地從 App Service Environment 連接到後端資源][SecurelyConnecttoBackend]

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[virtualnetwork]: https://azure.microsoft.com/documentation/articles/virtual-networks-faq/
[HowToCreateAnAppServiceEnvironment]: app-service-web-how-to-create-an-app-service-environment.md
[NetworkSecurityGroups]: https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[IntroToAppServiceEnvironment]:  app-service-app-service-environment-intro.md
[SecurelyConnecttoBackend]:  app-service-app-service-environment-securely-connecting-to-backend-resources.md
[NewPortal]:  https://portal.azure.com  

<!-- IMAGES -->

