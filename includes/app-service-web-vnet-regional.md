---
author: ccompy
ms.service: app-service-web
ms.topic: include
ms.date: 10/21/2020
ms.author: ccompy
ms.openlocfilehash: 3f9dd35959980eef4e1bec550bf7e9f583cf30d2
ms.sourcegitcommit: f5b8410738bee1381407786fcb9d3d3ab838d813
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98225558"
---
使用區域 VNet 整合可讓您的應用程式存取：

* VNet 中的資源，與您的應用程式位於相同區域中。
* Vnet 中的資源會對等互連至您的應用程式所整合的 VNet。
* 服務端點保護的服務。
* 跨 Azure ExpressRoute 連線的資源。
* 您要整合的 VNet 中的資源。
* 跨對等互連連接的資源，包括 Azure ExpressRoute 連線。
* 私人端點 

當您在相同區域中使用 VNet 與 Vnet 整合時，您可以使用下列 Azure 網路功能：

* **網路安全性群組 (nsg)**：您可以使用位於整合子網的 NSG 來封鎖輸出流量。 因為您無法使用 VNet 整合來提供應用程式的輸入存取權，所以不適用輸入規則。
* **路由表 (udr)**：您可以在整合子網上放置路由表，以傳送您想要的輸出流量。

根據預設，您的應用程式只會將 RFC1918 流量路由傳送至您的 VNet。 如果您想要將所有輸出流量路由傳送至您的 VNet，請將應用程式設定 WEBSITE_VNET_ROUTE_ALL 套用至您的應用程式。 若要設定應用程式設定：

1. 移至應用程式入口網站 **中的設定** UI。 選取 [新增應用程式設定]。
1. 在 [**名稱**] 方塊中輸入 **WEBSITE_VNET_ROUTE_ALL** ，然後在 [**值**] 方塊中輸入 **1** 。

   ![提供應用程式設定][4]

1. 選取 [確定]。
1. 選取 [儲存]。

> [!NOTE]
> 如果您將所有輸出流量路由傳送至您的 VNet，則會受限於套用至您的整合子網的 Nsg 和 Udr。 當您將所有輸出流量路由傳送至您的 VNet 時，您的輸出位址仍是您應用程式屬性中所列的輸出位址，除非您提供路由以將流量傳送到其他地方。

在相同區域中使用 VNet 與 Vnet 的整合有一些限制：

* 您無法跨全球對等互連連線來連線到資源。
* 您可以從 Premium V2 和 Premium V3 中的所有 App Service 縮放單位取得此功能。 它也適用于標準版，但只適用于較新的 App Service 縮放單位。 如果您是在較舊的縮放單位上，則只能使用 Premium V2 App Service 方案中的功能。 如果您想要能夠在標準 App Service 方案中使用該功能，請在 Premium V3 App Service 方案中建立您的應用程式。 這些方案僅支援最新的縮放單位。 如果您想要的話，可以縮小。  
* 整合子網只能由一個 App Service 方案使用。
* App Service 環境中的隔離式方案應用程式無法使用此功能。
* 此功能需要 Azure Resource Manager VNet 中的/28 或更大的未使用子網。
* 應用程式和 VNet 必須位於相同的區域。
* 您無法使用整合式應用程式刪除 VNet。 刪除 VNet 之前，請先移除整合。
* 每 App Service 方案只能有一個區域 VNet 整合。 相同 App Service 方案中的多個應用程式可以使用相同的 VNet。
* 當有使用區域 VNet 整合的應用程式時，您無法變更應用程式或方案的訂用帳戶。
* 您的應用程式無法在沒有設定變更的 Azure DNS 私人區域中解析位址

VNet 整合取決於使用專用子網。  當您布建子網時，Azure 子網會從一開始就失去5個 Ip。 其中一個位址是從每個方案實例的整合子網使用。 如果您將應用程式調整為四個實例，則會使用四個位址。 來自子網大小的5個位址付款表示每個 CIDR 區塊的可用位址上限為：

- /28 有11個位址
- /27 有27個位址
- /26 有59位址

如果您的大小擴大或縮小，您需要一小段時間才需要兩次的位址。 大小的限制表示，如果您的子網為，每個子網大小的真實可用支援實例為：

- /28，最大水準規模為5個實例
- /27，最大水準規模為13個實例
- /26，最大水準規模為29個實例

最大水準延展上所述的限制會假設您在某個時間點需要擴大或縮小大小或 SKU。 

因為子網大小在指派後無法變更，所以請使用夠大的子網來容納您的應用程式可能達到的任何規模。 為了避免任何子網容量發生問題，建議的大小為/26 （含64位址）。  

如果您想要讓另一個方案中的應用程式連接到已由另一個方案中的應用程式連線到的 VNet，請選取與預先存在的 VNet 整合所使用的子網不同的子網。

這項功能完全支援 Windows 和 Linux 應用程式，包括 [自訂容器](../articles/app-service/quickstart-custom-container.md)。 所有的行為在 Windows 應用程式和 Linux 應用程式之間的行為都相同。

### <a name="service-endpoints"></a>服務端點

區域 VNet 整合可讓您使用服務端點。 若要將服務端點與您的應用程式搭配使用，請使用區域 VNet 整合來連線至選取的 VNet，然後在您用於整合的子網上，使用目的地服務來設定服務端點。 如果您之後想要透過服務端點存取服務：

1. 設定與 web 應用程式的區域 VNet 整合
1. 移至目的地服務，並針對用於整合的子網設定服務端點

### <a name="network-security-groups"></a>網路安全性群組

您可以使用網路安全性群組來封鎖 VNet 中資源的輸入和輸出流量。 使用區域 VNet 整合的應用程式可以使用 [網路安全性群組][VNETnsg] 來封鎖對您 VNet 或網際網路中資源的輸出流量。 若要封鎖公用位址的流量，您必須將應用程式設定 WEBSITE_VNET_ROUTE_ALL 設定為1。 NSG 中的輸入規則不適用於您的應用程式，因為 VNet 整合只會影響來自您應用程式的輸出流量。

若要控制應用程式的輸入流量，請使用存取限制功能。 無論套用至您的整合子網的任何路由，套用至您的整合子網的 NSG 都會有效。 如果 WEBSITE_VNET_ROUTE_ALL 設定為1，而且您沒有任何會影響您整合子網上之公用位址流量的路由，則所有的輸出流量仍受限於 Nsg 指派給您的整合子網。 如果未設定 WEBSITE_VNET_ROUTE_ALL，Nsg 只會套用至 RFC1918 流量。

### <a name="routes"></a>路由

您可以使用路由表，將來自您應用程式的輸出流量路由傳送到您想要的任何位置。 根據預設，路由表只會影響您的 RFC1918 目的地流量。 如果您將 WEBSITE_VNET_ROUTE_ALL 設定為1，則所有的輸出呼叫都會受到影響。 在您的整合子網上設定的路由不會影響對輸入應用程式要求的回復。 常見的目的地可以包含防火牆裝置或閘道。

如果您想要將所有輸出流量路由傳送至內部部署，您可以使用路由表將所有輸出流量傳送至您的 ExpressRoute 閘道。 如果您將流量路由傳送至閘道，請務必在外部網路中設定路由，以傳送任何回復。

邊界閘道協定 (BGP) 路由也會影響您的應用程式流量。 如果您有來自類似 ExpressRoute 閘道的 BGP 路由，您的應用程式輸出流量將會受到影響。 根據預設，BGP 路由只會影響您的 RFC1918 目的地流量。 如果 WEBSITE_VNET_ROUTE_ALL 設定為1，則所有輸出流量都會受到您 BGP 路由的影響。

### <a name="azure-dns-private-zones"></a>Azure DNS 私人區域 

當您的應用程式與您的 VNet 整合之後，它會使用您的 VNet 設定所在的相同 DNS 伺服器。 根據預設，您的應用程式不會使用 Azure DNS 私人區域。 若要使用 Azure DNS 私人區域，您需要新增下列應用程式設定：


1. 具有值168.63.129.16 的 WEBSITE_DNS_SERVER
1. 值為1的 WEBSITE_VNET_ROUTE_ALL


除了讓您的應用程式使用 Azure DNS 私人區域以外，這些設定還會將您的應用程式的所有輸出呼叫傳送至 VNet。   這些設定會將來自您應用程式的所有輸出呼叫傳送至您的 VNet。 此外，它也會藉由在背景工作角色層級查詢私人 DNS 區域，讓應用程式可以使用 Azure DNS。 當執行中的應用程式正在存取私人 DNS 區域時，就會使用這項功能。

> [!NOTE]
>使用私人 DNS 區域嘗試將自訂網域新增至 Web 應用程式，並不可能使用 VNET 整合。 自訂網域驗證是在控制器層級執行，而不是背景工作角色層級，這樣會防止出現 DNS 記錄。 若要從私人 DNS 區域使用自訂網域，必須使用應用程式閘道或 ILB App Service 環境來略過驗證。

### <a name="private-endpoints"></a>私人端點

如果您想要對 [私人端點][privateendpoints]進行呼叫，您必須確定您的 DNS 查閱會解析為私人端點。 為了確保來自您應用程式的 DNS 查閱會指向您的私人端點，您可以：

* 與 Azure DNS 私人區域整合。 如果您的 VNet 沒有自訂 DNS 伺服器，這將會自動
* 在您的應用程式所使用的 DNS 伺服器中管理私人端點。 若要這樣做，您必須知道私人端點位址，然後使用 A 記錄，將您嘗試抵達的端點指向該位址。
* 設定您自己的 DNS 伺服器以轉寄至 Azure DNS 私人區域

<!--Image references-->
[4]: ../includes/media/web-sites-integrate-with-vnet/vnetint-appsetting.png

<!--Links-->
[VNETnsg]: /azure/virtual-network/security-overview/
[privateendpoints]: ../articles/app-service/networking/private-endpoint.md
