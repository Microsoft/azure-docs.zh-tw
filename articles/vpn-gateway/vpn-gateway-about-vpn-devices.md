---
title: 關於跨單位 Azure 連接的 VPN 裝置 | Microsoft Docs
description: 本文討論 S2S VPN 閘道跨單位連接的 VPN 裝置和 IPsec 參數。 其中提供設定指示和範例的連結。
services: vpn-gateway
author: yushwang
manager: rossort
ms.service: vpn-gateway
ms.topic: article
ms.workload: infrastructure-services
ms.date: 10/05/2018
ms.author: yushwang
ms.openlocfilehash: 7153b4054c1efc6e8e320cee61bc36674061b7d3
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48815803"
---
# <a name="about-vpn-devices-and-ipsecike-parameters-for-site-to-site-vpn-gateway-connections"></a>關於 VPN 裝置和站對站 VPN 閘道連線的 IPsec/IKE 參數

使用 VPN 閘道設定站對站 (S2S) 跨單位 VPN 連接需要有 VPN 裝置。 站對站連線可以用來建立混合式解決方案，或者用於您想要在內部部署網路與虛擬網路之間建立安全連線之時。 本文提供已驗證的 VPN 裝置清單和 VPN 閘道的 IPsec/IKE 參數清單。

> [!IMPORTANT]
> 如果您的內部部署 VPN 裝置與 VPN 閘道之間發生連線問題，請參考[已知的裝置相容性問題](#known)。
>

### <a name="items-to-note-when-viewing-the-tables"></a>檢視表格時應注意的項目：

* Azure VPN 閘道的名稱已經變更。 只有名稱會變更。 功能未變更。
  * 靜態路由 = 原則式
  * 動態路由 = 路由式
* 除非另有說明，否則高效能 VPN 閘道和路由式 VPN 閘道的規格相同。 例如，已經驗證與路由式 VPN 閘道相容的 VPN 裝置，也能與高效能 VPN 閘道相容。

## <a name="devicetable"></a>已經驗證的 VPN 裝置及裝置設定指南

> [!NOTE]
> 設定站對站連線時，您的 VPN 裝置需要公開的 IPv4 IP 位址。
>

我們已與裝置廠商合作驗證一組標準 VPN 裝置。 在以下清單的裝置系列中，所有裝置應該都能與 VPN 閘道搭配運作。 請參閱[關於 VPN 閘道設定](vpn-gateway-about-vpn-gateway-settings.md#vpntype)，了解您要設定的 VPN 閘道解決方案使用的 VPN 類型 (原則式或路由式)。

為了協助設定您的 VPN 裝置，請參閱對應到適當裝置系列的連結。 會以最佳方式來提供組態指示的連結。 如需 VPN 裝置的支援，請連絡裝置製造商。

|**廠商**          |**裝置系列**     |**作業系統最低版本** |**原則式設定指示** |**路由式設定指示** |
| ---                | ---                  | ---                   | ---            | ---           |
| A10 Networks, Inc. |Thunder CFW           |ACOS 4.1.1             |不相容  |[設定指南](https://www.a10networks.com/resources/deployment-guides/a10-thunder-cfw-ipsec-vpn-interoperability-azure-vpn-gateways)|
| Allied Telesis     |AR 系列 VPN 路由器 |2.9.2                  |敬請期待     |不相容  |
| Barracuda Networks, Inc. |Barracuda NextGen Firewall F-series |原則式︰5.4.3<br>路由式︰6.2.0 |[設定指南](https://techlib.barracuda.com/NGF/AzurePolicyBasedVPNGW) |[設定指南](https://techlib.barracuda.com/NGF/AzureRouteBasedVPNGW) |
| Barracuda Networks, Inc. |Barracuda NextGen Firewall X-series |Barracuda Firewall 6.5 |[設定指南](https://techlib.barracuda.com/BFW/ConfigAzureVPNGateway) |不相容 |
| Brocade            |Vyatta 5400 vRouter   |Virtual Router 6.6R3 GA|[設定指南](http://www1.brocade.com/downloads/documents/html_product_manuals/vyatta/vyatta_5400_manual/wwhelp/wwhimpl/js/html/wwhelp.htm#href=VPN_Site-to-Site%20IPsec%20VPN/Preface.1.1.html) |不相容 |
| Check Point |Security Gateway |R77.30 |[設定指南](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) |[設定指南](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) |
| Cisco              |ASA       |8.3<br>8.4+ (IKEv2\*) |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASA) |[設定指南\*](vpn-gateway-3rdparty-device-config-cisco-asa.md) |
| Cisco |ASR |原則式：IOS 15.1<br>路由式：IOS 15.2 |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASR) |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASR) |
| Cisco |ISR |原則式：IOS 15.0<br>路由式*：IOS 15.1 |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ISR) |[設定範例\*\*](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ISR) |
| Cisco |Meraki |N/A |不相容 |不相容 |
| Citrix |NetScaler MPX、SDX、VPX |10.1 和更新版本 |[設定指南](https://docs.citrix.com/en-us/netscaler/11-1/system/cloudbridge-connector-introduction/cloudbridge-connector-azure.html) |不相容 |
| F5 |BIG-IP 系列 |12.0 |[設定指南](https://devcentral.f5.com/articles/connecting-to-windows-azure-with-the-big-ip) |[設定指南](https://devcentral.f5.com/articles/big-ip-to-azure-dynamic-ipsec-tunneling) |
| Fortinet |FortiGate |FortiOS 5.6 |  |[設定指南](http://cookbook.fortinet.com/ipsec-vpn-microsoft-azure-56/) |
| Internet Initiative Japan (IIJ) |SEIL 系列 |SEIL/X 4.60<br>SEIL/B1 4.60<br>SEIL/x86 3.20 |[設定指南](http://www.iij.ad.jp/biz/seil/ConfigAzureSEILVPN.pdf) |不相容 |
| Juniper |SRX |原則式：JunOS 10.2<br>路由式：JunOS 11.4 |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SRX) |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SRX) |
| Juniper |J 系列 |原則式：JunOS 10.4r9<br>路由式：JunOS 11.4 |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/JSeries) |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/JSeries) |
| Juniper |ISG |ScreenOS 6.3 |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/ISG) |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/ISG) |
| Juniper |SSG |ScreenOS 6.2 |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SSG) |[設定範例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SSG) |
| Microsoft |路由及遠端存取服務 |Windows Server 2012 |不相容 |[設定範例](http://go.microsoft.com/fwlink/p/?LinkId=717761) |
| 開啟系統 AG |任務控制安全性閘道 |N/A |[設定指南](https://www.open.ch/_pdf/Azure/AzureVPNSetup_Installation_Guide.pdf) |不相容 |
| Palo Alto Networks |所有執行 PAN-OS 的裝置 |PAN-OS<br>原則式：6.1.5 或更新版本<br>路由式：7.1.4 |[設定指南](https://live.paloaltonetworks.com/t5/Configuration-Articles/How-to-Configure-VPN-Tunnel-Between-a-Palo-Alto-Networks/ta-p/59065) |[設定指南](https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA10g000000Cm6WCAS) |
| ShareTech | 新一代 UTM (NU 系列) | 9.0.1.3 | 不相容 | [設定指南](http://www.sharetech.com.tw/images/file/Solution/NU_UTM/S2S_VPN_with_Azure_Route_Based_en.pdf) |
| SonicWall |TZ 系列、NSA 系列<br>SuperMassive 系列<br>E-Class NSA 系列 |SonicOS 5.8.x<br>SonicOS 5.9.x<br>SonicOS 6.x |不相容 |[設定指南](https://www.sonicwall.com/support/knowledge-base/170505320011694) |
| Sophos | XG 新一代防火牆 | XG v17 | | [設定指南](https://community.sophos.com/kb/127546) |
| Ubiquiti | EdgeRouter | EdgeOS v1.10 |  | [透過 IKEv2/IPsec 的 BGP](https://help.ubnt.com/hc/en-us/articles/115012374708) \(英文\)<br><br>[透過 IKEv2/IPsec 的 VTI](https://help.ubnt.com/hc/en-us/articles/115012305347) \(英文\)
| WatchGuard |全部 |Fireware XTM<br> 原則式：v11.11.x<br>路由式：v11.12.x |[設定指南](http://watchguardsupport.force.com/publicKB?type=KBArticle&SFDCID=kA2F00000000LI7KAM&lang=en_US) |[設定指南](http://watchguardsupport.force.com/publicKB?type=KBArticle&SFDCID=kA22A000000XZogSAG&lang=en_US)|

> [!NOTE]
>
> (\*) Cisco ASA 8.4 版以上新增了 IKEv2 支援，可使用自訂 IPsec/IKE 原則並搭配 "UsePolicyBasedTrafficSelectors" 選項來連線到 Azure VPN 閘道。 請參閱這篇[操作說明文章](vpn-gateway-connect-multiple-policybased-rm-ps.md)。
>
> (\*\*) ISR 7200 系列路由器僅支援原則式 VPN。

## <a name="configscripts"></a>從 Azure 下載 VPN 裝置設定指令碼

針對特定裝置，您可以直接從 Azure 下載設定指令碼。 如需詳細資訊和下載指示，請參閱[下載 VPN 裝置設定指令碼](vpn-gateway-download-vpndevicescript.md)。

### <a name="devices-with-available-configuration-scripts"></a>具有可用設定指令碼的裝置

[!INCLUDE [scripts](../../includes/vpn-gateway-device-configuration-scripts.md)]

## <a name="additionaldevices"></a>未經驗證的 VPN 裝置

如果沒有看到您的裝置列在「已經驗證的 VPN 裝置」表格中，您的裝置仍然可能可以與站對站連線搭配使用。 如需額外支援和設定指示，請連絡裝置製造商。

## <a name="editing"></a>編輯裝置組態範本

下載提供的 VPN 裝置組態範本之後，您將需要取代其中一些值，以反映您環境的設定。

### <a name="to-edit-a-sample"></a>編輯範本：

1. 使用 [記事本] 開啟範本。
2. 搜尋所有 <文字> 字串並使用適合您環境的值加以取代。 請務必加上 < 和 >。 當有指定名稱時，您選取的名稱應該是唯一名稱。 如果命令無法運作，請參閱裝置製造商文件。

| **範本中的文字** | **變更為** |
| --- | --- |
| &lt;RP_OnPremisesNetwork&gt; |您為此物件選擇的名稱。 例如：myOnPremisesNetwork |
| &lt;RP_AzureNetwork&gt; |您為此物件選擇的名稱。 例如：myAzureNetwork |
| &lt;RP_AccessList&gt; |您為此物件選擇的名稱。 例如：myAzureAccessList |
| &lt;RP_IPSecTransformSet&gt; |您為此物件選擇的名稱。 例如：myIPSecTransformSet |
| &lt;RP_IPSecCryptoMap&gt; |您為此物件選擇的名稱。 例如：myIPSecCryptoMap |
| &lt;SP_AzureNetworkIpRange&gt; |指定範圍。 例如：192.168.0.0 |
| &lt;SP_AzureNetworkSubnetMask&gt; |指定子網路遮罩。 例如：255.255.0.0 |
| &lt;SP_OnPremisesNetworkIpRange&gt; |指定內部部署範圍。 例如：10.2.1.0 |
| &lt;SP_OnPremisesNetworkSubnetMask&gt; |指定內部部署子網路遮罩。 例如：255.255.255.0 |
| &lt;SP_AzureGatewayIpAddress&gt; |此為您虛擬網路的特定資訊，位於管理入口網站中的 [閘道器 IP 位址] 。 |
| &lt;SP_PresharedKey&gt; |此資訊專屬於您的虛擬網路，是 [管理入口網站] 中的管理金鑰。 |

## <a name="ipsec"></a>IPsec/IKE 參數

> [!IMPORTANT]
> 1. 下面的資料表包含了 Azure VPN 閘道在預設組態中使用的演算法和參數的組合。 對於使用 Azure Resource Management 部署模型所建立的路由式 VPN 閘道，您可以對每個個別的連線指定自訂原則。 如需詳細指示，請參閱[設定 IPsec/IKE 原則](vpn-gateway-ipsecikepolicy-rm-powershell.md)。
>
> 2. 此外，您必須將 TCP **MSS** 固定在 **1350**。 或者，如果您的 VPN 裝置不支援 MSS 固定，您也可以將通道介面上的 **MTU** 改設為 **1400** 位元組。
>

在下列資料表中：

* SA = 安全性關聯
* IKE 階段 1 也稱為「主要模式」
* IKE 階段 2 也稱為「快速模式」

### <a name="ike-phase-1-main-mode-parameters"></a>IKE 階段 1 (主要模式) 參數

| **屬性**          |**原則式**    | **路由式**    |
| ---                   | ---               | ---               |
| IKE 版本           |IKEv1              |IKEv2              |
| Diffie-Hellman 群組  |群組 2 (1024 位元) |群組 2 (1024 位元) |
| 驗證方法 |預先共用金鑰     |預先共用金鑰     |
| 加密與雜湊演算法 |1.AES256、SHA256<br>2.AES256、SHA1<br>3.AES128、SHA1<br>4. 3DES、SHA1 |1.AES256、SHA1<br>2.AES256、SHA256<br>3.AES128、SHA1<br>4.AES128、SHA256<br>5. 3DES、SHA1<br>6. 3DES、SHA256 |
| SA 存留期           |28,800 秒     |28,800 秒     |

### <a name="ike-phase-2-quick-mode-parameters"></a>IKE 階段 2 (快速模式) 參數

| **屬性**                  |**原則式**| **路由式**                              |
| ---                           | ---           | ---                                         |
| IKE 版本                   |IKEv1          |IKEv2                                        |
| 加密與雜湊演算法 |1.AES256、SHA256<br>2.AES256、SHA1<br>3.AES128、SHA1<br>4. 3DES、SHA1 |[RouteBased QM SA 供應項目](#RouteBasedOffers) |
| SA 存留期 (時間)            |3,600 秒  |27,000 秒                                |
| SA 存留期 (位元組)           |102,400,000 KB | -                                           |
| 完整轉寄密碼 (PFS) |否             |[RouteBased QM SA 供應項目](#RouteBasedOffers) |
| 停用的對等偵測 (DPD)     |不支援  |支援                                    |


### <a name ="RouteBasedOffers"></a>RouteBased VPN IPsec 安全性關聯 (IKE 快速模式 SA) 供應項目

下表列出 IPsec SA (IKE 快速模式) 供應項目。 供應項目是依其呈現或被接受的喜好設定順序而列出。

#### <a name="azure-gateway-as-initiator"></a>Azure 閘道器為啟動者

|-  |**加密**|**驗證**|**PFS 群組**|
|---| ---          |---               |---          |
| 1 |GCM AES256    |GCM (AES256)      |None         |
| 2 |AES256        |SHA1              |None         |
| 3 |3DES          |SHA1              |None         |
| 4 |AES256        |SHA256            |None         |
| 5 |AES128        |SHA1              |None         |
| 6 |3DES          |SHA256            |None         |

#### <a name="azure-gateway-as-responder"></a>Azure 閘道器為回應者

|-  |**加密**|**驗證**|**PFS 群組**|
|---| ---          | ---              |---          |
| 1 |GCM AES256    |GCM (AES256)      |None         |
| 2 |AES256        |SHA1              |None         |
| 3 |3DES          |SHA1              |None         |
| 4 |AES256        |SHA256            |None         |
| 5 |AES128        |SHA1              |None         |
| 6 |3DES          |SHA256            |None         |
| 7 |DES           |SHA1              |None         |
| 8 |AES256        |SHA1              |1            |
| 9 |AES256        |SHA1              |2            |
| 10|AES256        |SHA1              |14           |
| 11|AES128        |SHA1              |1            |
| 12|AES128        |SHA1              |2            |
| 13|AES128        |SHA1              |14           |
| 14|3DES          |SHA1              |1            |
| 15|3DES          |SHA1              |2            |
| 16|3DES          |SHA256            |2            |
| 17|AES256        |SHA256            |1            |
| 18|AES256        |SHA256            |2            |
| 19|AES256        |SHA256            |14           |
| 20|AES256        |SHA1              |24           |
| 21|AES256        |SHA256            |24           |
| 22|AES128        |SHA256            |None         |
| 23|AES128        |SHA256            |1            |
| 24|AES128        |SHA256            |2            |
| 25|AES128        |SHA256            |14           |
| 26|3DES          |SHA1              |14           |

* 您可以使用路由式和高效能 VPN 閘道指定 IPsec ESP NULL 加密。 以 Null 為基礎的加密不提供傳輸中資料的保護，應該只用於時需要最大輸送量和最小延遲時。 用戶端可能會選擇在 VNet 對 VNet 通訊案例中，或當加密套用至解決方案中的其他地方時，使用此功能。
* 透過網際網路的跨單位連線，請使用含有加密和雜湊演算法的預設 Azure VPN 閘道設定 (如上表所列)，以確保重要通訊的安全性。

## <a name="known"></a>已知的裝置相容性問題

> [!IMPORTANT]
> 協力廠商 VPN 裝置與 Azure VPN 閘道之間有已知的相容性問題。 Azure 小組正積極與廠商合作來解決這裡所列出的問題。 解決這些問題之後，就會更新此頁面來提供最新資訊。 請定期回來查看。
>
>

### <a name="feb-16-2017"></a>2017 年 2 月 16 日

適用於 Azure 路由式 VPN 但**版本比 7.1.4 舊的 Palo Alto Networks 裝置**：如果您使用來自 Palo Alto Networks、PAN-OS 版本比 7.1.4 舊的 VPN 裝置，而在連線到 Azure 路由式 VPN 閘道時發生問題，請執行下列步驟：

1. 檢查您 Palo Alto Networks 裝置的韌體版本。 如果您的 PAN-OS 版本比 7.1.4 舊，請升級至 7.1.4。
2. 在 Palo Alto Networks 裝置上，於連線到 Azure VPN 閘道時，將 [Phase 2 SA (第 2 階段 SA)] \(或 [Quick Mode SA (快速模式 SA)]) 存留期變更為 28,800 秒 (8 小時)。
3. 如果您仍然遇到連線問題，請從 Azure 入口網站開啟支援要求。
