---
title: Azure VPN 閘道常見問題集 | Microsoft Docs
description: VPN 閘道常見問題集。 Microsoft Azure 虛擬網路跨單位連線、混合式組態連線和 VPN 閘道的常見問題集。
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 07/27/2018
ms.author: cherylmc
ms.openlocfilehash: 245e5e1048f27fc76141312cf6f9021dc0060ae6
ms.sourcegitcommit: baed5a8884cb998138787a6ecfff46de07b8473d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "39359055"
---
# <a name="vpn-gateway-faq"></a>VPN 閘道常見問題集

## <a name="connecting"></a>連線到虛擬網路

### <a name="can-i-connect-virtual-networks-in-different-azure-regions"></a>是否可以連接不同 Azure 區域中的虛擬網路？

是。 事實上，沒有區域限制。 一個虛擬網路可以連接到相同區域或不同 Azure 區域中的另一個虛擬網路。 

### <a name="can-i-connect-virtual-networks-in-different-subscriptions"></a>是否可以使用不同訂用帳戶連接虛擬網路？

是。

### <a name="can-i-connect-to-multiple-sites-from-a-single-virtual-network"></a>是否可以從單一虛擬網路連接到多個網站？

您可以使用 Windows PowerShell 和 Azure REST API 連接到多個網站。 請參閱＜ [多網站和 VNet 對 VNet 連線能力](#V2VMulti) ＞常見問題集一節。

### <a name="what-are-my-cross-premises-connection-options"></a>有哪些跨單位連線選項？

支援下列跨單位連線：

* 站對站 – 透過 IPsec (IKE v1 和 IKE v2) 的 VPN 連線。 此類型的連線需要 VPN 裝置或 RRAS。 如需詳細資訊，請參閱[站對站](vpn-gateway-howto-site-to-site-resource-manager-portal.md)。
* 點對站 – 透過 SSTP (安全通訊端通道通訊協定) 的 VPN 連線或 IKE v2。 此連線不需要 VPN 裝置。 如需詳細資訊，請參閱[點對站](vpn-gateway-howto-point-to-site-resource-manager-portal.md)。
* VNet 對 VNet – 此類型的連線與站對站組態相同。 VNet 對 VNet 是透過 IPsec (IKE v1 和 IKE v2) 的 VPN 連線， 並不需要 VPN 裝置。 如需詳細資訊，請參閱 [VNet 對 VNet](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)。
* 多網站 - 這是站對站組態的變體，可讓您將多個內部部署網站連線至虛擬網路。 如需詳細資訊，請參閱[多網站](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)。
* ExpressRoute - ExpressRoute 是從 WAN 到 Azure 的直接連線，而不是透過公用網際網路的 VPN 連線。 如需詳細資訊，請參閱 [ExpressRoute 技術概觀](../expressroute/expressroute-introduction.md)和 [ExpressRoute 常見問題集](../expressroute/expressroute-faqs.md)。

如需 VPN 閘道連線的詳細資訊，請參閱[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)。

### <a name="what-is-the-difference-between-a-site-to-site-connection-and-point-to-site"></a>網站間連線和點對站台之間的差異為何？

**站對站**(IPsec/IKE VPN 通道) 介於內部部署位置與 Azure 之間。 這表示視您選擇如何設定路由和權限而定﹐您可以從任何位於您內部部署的電腦連線至虛擬網路內的任何虛擬機器或角色執行個體。 對於永遠可用的跨單位連線而言，這是不錯的選項，而且非常適合於混合式組態。 這種類型的連線依賴 IPsec VPN 應用裝置 (硬體裝置或軟體應用裝置)，而此應用裝置必須部署在網路的邊緣。 若要建立此類型的連線，您必須擁有不在 NAT 之後的對外公開 IPv4 位址。

**點對站** (VPN over SSTP) 組態可讓您從任何地方的單一電腦連線到位於虛擬網路的任何項目。 並會使用 Windows 內建的 VPN 用戶端。 做為點對站台組態的一部分，您會安裝憑證和 VPN 用戶端組態封裝，其中包含可讓您的電腦連接到虛擬網路內任何虛擬機器或角色執行個體的設定。 當您想要連接到不在內部部署的虛擬網路時，這樣做很有用。 當您無權存取 VPN 硬體或對外公開的 IPv4 位址時也是不錯的選項，因為這兩者都是網站間連線的必要項目。

您可以將虛擬網路設定為同時使用網站間和點對站台，只要您使用路由式 VPN 類型來為閘道建立網站間連線即可。 路由式 VPN 類型在傳統部署模型中稱為「動態閘道」。

## <a name="gateways"></a>虛擬網路閘道

### <a name="is-a-vpn-gateway-a-virtual-network-gateway"></a>VPN 閘道是虛擬網路閘道嗎？

VPN 閘道是一種虛擬網路閘道。 VPN 閘道可透過公用連線在您的虛擬網路和內部部署位置之間傳送加密流量。 您也可以使用 VPN 閘道在虛擬網路之間傳送流量。 當您建立 VPN 閘道時，您可以使用 -GatewayType 值 'Vpn'。 如需詳細資訊，請參閱[關於 VPN 閘道組態設定](vpn-gateway-about-vpn-gateway-settings.md)。

### <a name="what-is-a-policy-based-static-routing-gateway"></a>什麼是以原則為基礎的 (靜態路由) 閘道？

以原則為基礎的閘道會實作原則式 VPN。 原則式 VPN 會根據內部部署網路與 Azure VNet 之間的位址首碼組合，透過 IPsec 通道加密和直接封包。 原則 (或流量選取器) 通常定義為 VPN 組態中的存取清單。

### <a name="what-is-a-route-based-dynamic-routing-gateway"></a>什麼是以路由為基礎的 (動態路由) 閘道？

路由式閘道實作路由式 VPN。 路由式 VPN 會使用 IP 轉送或路由表中的「路由」，直接封包至其對應的通道介面。 然後，通道介面會加密或解密輸入和輸出通道的封包。 路由式 VPN 的原則或流量選取器會設定為任意對任意 (或萬用字元)。

### <a name="can-i-update-my-policy-based-vpn-gateway-to-route-based"></a>可以將我的原則型 VPN 閘道更新為路由型嗎？
否。 Azure Vnet 閘道類型無法從原則型變更為路由型或其他方式。 閘道必須刪除並重新建立，程序大約要 60 分鐘的時間。 閘道的 IP 位址不會保留，預先共用金鑰 (PSK) 也不會保留。
1. 刪除與要刪除之閘道相關聯的任何連線。
2. 刪除閘道：
* [Azure 入口網站](vpn-gateway-delete-vnet-gateway-portal.md)
* [Azure PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
* [Azure Powershell - 傳統](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
3. [建立所需類型的新閘道，並且完成 VPN 設定](vpn-gateway-howto-site-to-site-resource-manager-portal.md#VNetGateway)

### <a name="do-i-need-a-gatewaysubnet"></a>是否需要 'GatewaySubnet'？

是。 閘道子網路包含虛擬網路閘道服務使用的 IP 位址。 若要設定虛擬網路閘道，您必須為 VNet 建立閘道子網路。 所有閘道子網路都必須命名為 'GatewaySubnet' 才能正常運作。 請勿將閘道子網路命名為其他名稱。 請勿對閘道子網路部署 VM 或任何其他項目。

當您建立閘道子網路時，您可指定子網路包含的 IP 位址數目。 閘道子網路中的 IP 位址會配置給閘道服務。 某些組態要求將較多 IP 位址配置給閘道服務 (相較於其他服務)。 您想要確定您的閘道子網路包含足夠的 IP 位址，以因應未來成長及可能的其他新連線組態。 所以﹐您可以建立像 /29 這麼小的閘道子網路，但建議您建立 /27 或更大 (/27、/26、/25 等) 的閘道子網路。 查看您想要建立的組態需求﹐並確認您擁有的閘道子網路將符合這些需求。

### <a name="can-i-deploy-virtual-machines-or-role-instances-to-my-gateway-subnet"></a>是否可以將虛擬機器或角色執行個體部署到閘道子網路？

否。

### <a name="can-i-get-my-vpn-gateway-ip-address-before-i-create-it"></a>在建立之前是否可以取得我的 VPN 閘道 IP 位址？

否。 您必須先建立閘道才能取得 IP 位址。 如果您刪除並重新建立 VPN 閘道，IP 位址就會變更。

### <a name="can-i-request-a-static-public-ip-address-for-my-vpn-gateway"></a>是否可以要求我的 VPN 閘道的靜態公用 IP 位址？

否。 僅支援動態 IP 位址指派。 不過，這不表示 IP 位址變更之後已被指派至您的 VPN 閘道。 VPN 閘道 IP 位址只會在刪除或重新建立閘道時變更。 VPN 閘道公用 IP 位址不會因為重新調整、重設或 VPN 閘道的其他內部維護/升級而變更。 

### <a name="how-does-my-vpn-tunnel-get-authenticated"></a>我的 VPN 通道如何獲得驗證？

Azure VPN 使用 PSK (預先共用金鑰) 驗證。 當建立 VPN 通道時，就會產生預先共用金鑰 (PSK)。 您可以使用設定預先共用金鑰 PowerShell Cmdlet 或 REST API，將自動產生的 PSK 變更為您自己的。

### <a name="can-i-use-the-set-pre-shared-key-api-to-configure-my-policy-based-static-routing-gateway-vpn"></a>是否可以使用「設定預先共用金鑰 API」來設定我的原則式 (靜態路由) 閘道 VPN？

是，設定預先共用金鑰 API 和 PowerShell Cmdlet 可用來同時設定 Azure 原則式 (靜態) VPN 和路由式 (動態) 路由 VPN。

### <a name="can-i-use-other-authentication-options"></a>是否可以使用其他驗證選項？

我們受限於只能使用預先共用的金鑰 (PSK) 進行驗證。

### <a name="how-do-i-specify-which-traffic-goes-through-the-vpn-gateway"></a>如何指定通過 VPN 閘道的流量？

#### <a name="resource-manager-deployment-model"></a>資源管理員部署模型。

* PowerShell：使用 "AddressPrefix" 以指定區域網路閘道的流量。
* Azure 入口網站：瀏覽至 [區域網路閘道] > [組態] > [位址空間]。

#### <a name="classic-deployment-model"></a>傳統部署模型

* Azure 入口網站︰瀏覽至傳統虛擬網路 > VPN 連線 > 站對站 VPN 連線 > 本機網站名稱 > 本機網站 > 用戶端位址空間。 

### <a name="can-i-configure-force-tunneling"></a>是否可以設定強制通道？

是。 請參閱[設定強制通道](vpn-gateway-about-forced-tunneling.md)。

### <a name="can-i-set-up-my-own-vpn-server-in-azure-and-use-it-to-connect-to-my-on-premises-network"></a>是否可以在 Azure 中設定自己的 VPN 伺服器，並用來連接到內部部署網路？

是，您可以從 Azure Marketplace 或建立自己的 VPN 路由器，在 Azure 中部署自己的 VPN 閘道或伺服器。 您需要在虛擬網路中設定使用者定義的路由，以確保在您的內部部署網路與虛擬網路子網路之間適當地路由傳送流量。

### <a name="why-are-certain-ports-opened-on-my-vpn-gateway"></a>為什麼 VPN 閘道上的某些連接埠已開啟？

Azure 基礎結構通訊需要這些連接埠。 它們受到 Azure 憑證的保護 (鎖定)。 若沒有適當的憑證，外部實體 (包括這些閘道的客戶) 將無法對這些端點造成任何影響。

VPN 閘道基本上是一個多重主目錄的裝置，擁有一個使用客戶私人網路的 NIC，以及一個面向公用網路的 NIC。 Azure 基礎結構實體因為相容性，無法使用客戶私人網路，因此他們需要利用公用端點進行基礎結構通訊。 Azure 安全性稽核會定期掃描公用端點。

### <a name="more-information-about-gateway-types-requirements-and-throughput"></a>閘道類型、需求和輸送量的詳細資訊

如需詳細資訊，請參閱[關於 VPN 閘道組態設定](vpn-gateway-about-vpn-gateway-settings.md)。

## <a name="s2s"></a>站對站連線和 VPN 裝置

### <a name="what-should-i-consider-when-selecting-a-vpn-device"></a>選取 VPN 裝置時應該考慮什麼？

我們已與裝置廠商合作驗證一組標準網站間 VPN 裝置。 在[關於 VPN 裝置](vpn-gateway-about-vpn-devices.md)一文中可找到已知的相容 VPN 裝置、其對應組態指示或範例和裝置規格的清單。 裝置系列中列為已知相容裝置的所有裝置應該使用虛擬網路。 為了協助設定 VPN 裝置，請參閱裝置組態範例或對應到適當裝置系列的連結。

### <a name="where-can-i-find-vpn-device-configuration-settings"></a>哪裡可以找到 VPN 裝置組態設定？

[!INCLUDE [vpn devices](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

### <a name="how-do-i-edit-vpn-device-configuration-samples"></a>如何編輯 VPN 裝置組態範例？

如需編輯裝置組態範例的相關資訊，請參閱[編輯範例](vpn-gateway-about-vpn-devices.md#editing)。

### <a name="where-do-i-find-ipsec-and-ike-parameters"></a>哪裡可以找到 IPsec 和 IKE 參數？

如需 IPsec/IKE 參數，請參閱[參數](vpn-gateway-about-vpn-devices.md#ipsec)。

### <a name="why-does-my-policy-based-vpn-tunnel-go-down-when-traffic-is-idle"></a>為什麼我的原則型 VPN 通道會在流量閒置時終止？

這是原則型 (也稱為靜態路由) VPN 閘道的預期行為。 當通道上的流量閒置超過 5 分鐘時，通道就會終止。 當流量開始流向任何一個方向時，便會立即重新建立通道。

### <a name="can-i-use-software-vpns-to-connect-to-azure"></a>可以使用軟體 VPN 連接到 Azure 嗎？

我們支援 Windows Server 2012 路由及遠端存取 (RRAS) 伺服器進行網站間跨單位設定。

其他軟體 VPN 解決方案只要符合業界標準 IPsec 實作，應該就能使用我們的閘道。 如需設定和支援指示，請連絡軟體供應商。

## <a name="P2S"></a>使用原生 Azure 憑證驗證的點對站

本節適用於資源管理員部署模型。

[!INCLUDE [P2S Azure cert](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="P2SRADIUS"></a>使用 RADIUS 驗證的點對站

本節適用於資源管理員部署模型。

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="V2VMulti"></a>VNet 對 VNet 和多站台連線

[!INCLUDE [vpn-gateway-vnet-vnet-faq-include](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

### <a name="can-i-use-azure-vpn-gateway-to-transit-traffic-between-my-on-premises-sites-or-to-another-virtual-network"></a>是否可以使用 Azure VPN 閘道，在我的內部部署網站之間傳輸流量，或將流量傳輸到另一個虛擬網路？

**Resource Manager 部署模型**<br>
是。 如需詳細資訊，請參閱 [BGP](#bgp) 一節。

**傳統部署模型**<br>
使用傳統部署模型即可透過 Azure VPN 閘道傳輸流量，但其依賴網路組態檔中靜態定義的位址空間。 使用傳統部署模型的 Azure 虛擬網路和 VPN 閘道尚未支援 BGP。 若沒有 BGP，手動定義傳輸位址空間很容易出錯，因此並不建議。

### <a name="does-azure-generate-the-same-ipsecike-pre-shared-key-for-all-my-vpn-connections-for-the-same-virtual-network"></a>Azure 是否針對相同虛擬網路的所有我的 VPN 連線產生相同的 IPsec/IKE 預先共用金鑰？

否，Azure 預設會針對不同 VPN 連線產生不同的預先共用金鑰。 不過，您可以使用設定 VPN 閘道金鑰 REST API 或 PowerShell Cmdlet 來設定您偏好的金鑰值。 金鑰必須是長度介於 1 到 128 個字元的英數字串。

### <a name="do-i-get-more-bandwidth-with-more-site-to-site-vpns-than-for-a-single-virtual-network"></a>比起單一虛擬網路，是否可以使用更多的網站間 VPN，取得更多的頻寬？

否，所有 VPN 通道 (包括點對站 VPN) 都共用相同的 Azure VPN 閘道與可用的頻寬。

### <a name="can-i-configure-multiple-tunnels-between-my-virtual-network-and-my-on-premises-site-using-multi-site-vpn"></a>是否可以使用多網站 VPN，在我的虛擬網路與內部部署網站之間設定多個通道？

是，但您必須將這兩個通道上的 BGP 設定為相同的位置。

### <a name="can-i-use-point-to-site-vpns-with-my-virtual-network-with-multiple-vpn-tunnels"></a>是否可以使用點對站台 VPN 搭配具有多個 VPN 通道的虛擬網路？

是，點對站台 (P2S) VPN 可以與連接到多個內部部署網站和其他虛擬網路的 VPN 閘道搭配使用。

### <a name="can-i-connect-a-virtual-network-with-ipsec-vpns-to-my-expressroute-circuit"></a>是否可以將使用 IPsec VPN 的虛擬網路連接到我的 ExpressRoute 線路？

是，支援此做法。 如需詳細資訊，請參閱 [設定並存的 ExpressRoute 和網站間 VPN 連線](../expressroute/expressroute-howto-coexist-classic.md)。

## <a name="ipsecike"></a>IPsec/IKE 原則

[!INCLUDE [vpn-gateway-ipsecikepolicy-faq-include](../../includes/vpn-gateway-faq-ipsecikepolicy-include.md)]


## <a name="bgp"></a>BGP

[!INCLUDE [vpn-gateway-faq-bgp-include](../../includes/vpn-gateway-faq-bgp-include.md)]

## <a name="vms"></a>跨單位連線與 VM

### <a name="if-my-virtual-machine-is-in-a-virtual-network-and-i-have-a-cross-premises-connection-how-should-i-connect-to-the-vm"></a>如果我的虛擬機器在虛擬網路中，而且我有跨單位連線，應該如何連接至 VM？

您有幾個選項。 如果您的 VM 已啟用 RDP，您可以使用私人 IP 位址連線到您的虛擬機器。 在此情況下，您會指定私人 IP 位址以及您想要連線的連接埠 (通常為 3389)。 您將需要在虛擬機器上設定流量的連接埠。

您也可以從位於相同虛擬網路的另一部虛擬機器經由私人 IP 位址連線到您的虛擬機器。 如果您是從虛擬網路外的位置連線，則無法使用私人 IP 位址透過 RDP 連線到虛擬機器。 例如，如果您已設定點對站台虛擬網路，而且未從您的電腦建立連線，則無法經由私人 IP 位址連線到虛擬機器。

### <a name="if-my-virtual-machine-is-in-a-virtual-network-with-cross-premises-connectivity-does-all-the-traffic-from-my-vm-go-through-that-connection"></a>如果我的虛擬機器位於具有跨單位連線能力的虛擬網路，所有來自我的 VM 的流量是否都會通過該連線？

否。 只有目地的 IP 包含在您指定之虛擬網路區域網路 IP 位址範圍的流量，才會通過虛擬網路閘道。 目的地 IP 位於虛擬網路內的流量仍會留在虛擬網路內。 其他流量是透過負載平衡器傳送至公用網路，或者如果使用強制通道，則透過 Azure VPN 閘道傳送。

### <a name="how-do-i-troubleshoot-an-rdp-connection-to-a-vm"></a>如何針對 VM 的 RDP 連線進行疑難排解

[!INCLUDE [Troubleshoot VM connection](../../includes/vpn-gateway-connect-vm-troubleshoot-include.md)]


## <a name="faq"></a>虛擬網路常見問題集

您可以在 [虛擬網路常見問題集](../virtual-network/virtual-networks-faq.md)中檢視其他虛擬網路資訊。

## <a name="next-steps"></a>後續步驟

* 如需 VPN 閘道的詳細資訊，請參閱[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)。
* 如需 VPN 閘道組態設定的詳細資訊，請參閱[關於 VPN 閘道組態設定](vpn-gateway-about-vpn-gateway-settings.md)。
