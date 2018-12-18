---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 8be1144bd01634dce0db7f3129406c57e1f7bdac
ms.sourcegitcommit: baed5a8884cb998138787a6ecfff46de07b8473d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "39359386"
---
[!INCLUDE [P2S FAQ All](vpn-gateway-faq-p2s-all-include.md)]

### <a name="is-radius-authentication-supported-on-all-azure-vpn-gateway-skus"></a>是否所有的 Azure VPN 閘道 SKU 都可支援 RADIUS 驗證？

VpnGw1、VpnGw2 和 VpnGw3 SKU 可支援 RADIUS 驗證。 如果您使用舊版 SKU，則標準和高效能 SKU 可支援 RADIUS 驗證。 基本閘道 SKU 則不支援 RADIUS 驗證。 
 
### <a name="is-radius-authentication-supported-for-the-classic-deployment-model"></a>傳統部署模型是否支援 RADIUS 驗證？
 
否。 傳統部署模型不支援 RADIUS 驗證。
 
### <a name="are-3rd-party-radius-servers-supported"></a>是否支援第 3 方的 RADIUS 伺服器？

是，可支援第 3 方的 RADIUS 伺服器。
 
### <a name="what-are-the-connectivity-requirements-to-ensure-that-the-azure-gateway-is-able-to-reach-an-on-premises-radius-server"></a>若要確保 Azure 閘道能夠連線到內部部署 RADIUS 伺服器，需符合哪些連線需求？

您必須有內部部署站台的 VPN 站對站連線，並已正確設定路由。  
 
### <a name="can-traffic-to-an-on-premises-radius-server-from-the-azure-vpn-gateway-be-routed-over-an-expressroute-connection"></a>是否可以透過 ExpressRoute 連線來路由傳送從 Azure VPN 閘道流往內部部署 RADIUS 伺服器的流量？

否。 該流量只能透過站對站連線來路由傳送。
 
### <a name="is-there-a-change-in-the-number-of-sstp-connections-supported-with-radius-authentication-what-is-the-maximum-number-of-sstp-and-ikev2-connections-supported"></a>RADIUS 驗證所支援的 SSTP 連線數目是否有變更？ 所支援的 SSTP 和 IKEv2 連線數目上限為何？

RADIUS 驗證在閘道上所支援的 SSTP 連線數目上限並未變更。 上限仍是 128。 不論閘道是設定了 SSTP 和 (或) IKEv2，所支援的連線數目上限都是 128。
 
### <a name="what-is-the-difference-between-doing-certificate-authentication-using-a-radius-server-vs-using-azure-native-certificate-authentication-by-uploading-a-trusted-certificate-to-azure"></a>使用 RADIUS 伺服器來執行憑證驗證和使用 Azure 原生憑證驗證 (藉由將受信任的憑證上傳至 Azure) 來執行憑證驗證有何差異。

在 RADIUS 驗證憑證中，驗證要求會轉送至 RADIUS 伺服器以處理實際的憑證驗證。 如果您想要與您已透過 RADIUS 而擁有的憑證驗證基礎結構整合，這個選項非常有用。
  
在使用 Azure 進行憑證驗證時，Azure VPN 閘道會執行憑證驗證。 您需要將憑證公開金鑰上傳到閘道。 您也可以指定不該允許連線的已撤銷憑證清單。

### <a name="does-radius-authentication-work-with-both-ikev2-and-sstp-vpn"></a>RADIUS 驗證可以同時與 IKEv2 和 SSTP VPN 搭配運作嗎？

是，IKEv2 和 SSTP VPN 都支援 RADIUS 驗證。 
