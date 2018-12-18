---
title: 關於 Azure 點對站 VPN 連線 | Microsoft 文件
description: 本文可協助您了解點對站連線，並協助您決定所要使用的 P2S VPN 閘道驗證類型。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/06/2018
ms.author: cherylmc
ms.openlocfilehash: 8cdc80e8e4f8d3feb36ca82740d5610e60716ec6
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39003354"
---
# <a name="about-point-to-site-vpn"></a>關於點對站 VPN

點對站 (P2S) VPN 閘道連線可讓您建立從個別用戶端電腦到您的虛擬網路的安全連線。 P2S 連線的建立方式是從用戶端電腦開始。 此解決方案適合於想要從遠端位置 (例如從住家或會議) 連線到 Azure VNet 的遠距工作者。 當您只有少數用戶端必須連線至 VNet 時，P2S VPN 也是很實用的解決方案 (而不是 S2S VPN)。 本文適用於資源管理員部署模型。

## <a name="protocol"></a>P2S 使用哪種通訊協定？

店對站 VPN 可以使用下列其中一個通訊協定：

* 安全通訊端通道通訊協定 (SSTP)，這是以 SSL 為基礎的專屬 VPN 通訊協定。 SSL VPN 解決方案可以穿透防火牆，因為大部分防火牆都會開啟 SSL 使用的 TCP 連接埠 443。 SSTP 僅在 Microsoft 裝置上提供支援。 Azure 支援所有具有 SSTP (Windows 7 及更新版本) 的 Windows 版本。

* IKEv2 VPN，標準型 IPsec VPN 解決方案。 IKEv2 VPN 可用於從 Mac 裝置連線 (OSX 版本 10.11 和更新版本)。

如果您有包含 Windows 和 Mac 裝置的混合用戶端環境，請設定 SSTP 和 IKEv2。

>[!NOTE]
>適用於 P2S 的 IKEv2 僅供 Resource Manager 部署模型使用。 它不適用於傳統部署模型。
>

## <a name="authentication"></a>P2S VPN 用戶端的驗證方式

在 Azure 接受 P2S VPN 連線之前，使用者必須先進行驗證。 Azure 提供兩個機制來驗證連線使用者。

### <a name="authenticate-using-native-azure-certificate-authentication"></a>使用原生 Azure 憑證驗證進行驗證

使用原生 Azure 憑證驗證時，裝置上存在的用戶端憑證會用來驗證連線使用者。 用戶端憑證是從根憑證產生，然後安裝在每部用戶端電腦上。 您可以使用透過企業解決方案產生的根憑證，也可以產生自我簽署憑證。

用戶端憑證的驗證是由 VPN 閘道執行，並發生於 P2S VPN 連線建立期間。 根憑證需要驗證，且必須上傳至 Azure。

### <a name="authenticate-using-active-directory-ad-domain-server"></a>使用 Azure Active Directory (AD) 網域伺服器進行驗證

AD 網域驗證可讓使用者使用其組織網域認證來連線至 Azure。 它需要可與 AD 伺服器整合的 RADIUS 伺服器。 組織也可利用其現有的 RADIUS 部署。   
 RADIUS 伺服器可以部署在內部部署環境或 Azure VNET 中。 在驗證期間，Azure VPN 閘道可作為 RADIUS 伺服器與連線裝置之間的通道，雙向轉送驗證訊息。 所以閘道觸達 RADIUS 伺服器的能力很重要。 如果 RADIUS 伺服器位於內部部署環境，則需要從 Azure 到內部部署網站的 VPN S2S 連線才能觸達。  
 RADIUS 伺服器也可以與 AD 憑證服務整合。 這可讓您對 P2S 憑證驗證使用 RADIUS 伺服器和企業憑證部署，來替代 Azure 憑證驗證。 優點是，您不需要將根憑證及撤銷的憑證上傳至 Azure。

RADIUS 伺服器也可以與其他外部身分識別系統整合。 這會開啟 P2S VPN 的許多驗證選項，包括多重因素選項。

![點對站](./media/point-to-site-about/p2s.png "點對站")

## <a name="what-are-the-client-configuration-requirements"></a>設定用戶端有哪些需求？

>[!NOTE]
>對於 Windows 用戶端，您在用戶端裝置上必須具備系統管理員權限，才能將用戶端裝置到 Azure 的 VPN 連線初始化。
>

使用者會在 P2S 的 Windows 和 Mac 裝置上使用原生 VPN 用戶端。 Azure 會提供 VPN 用戶端組態 zip 檔案，其中包含這些原生用戶端連線到 Azure 所需的設定。

* 對於 Windows 裝置，VPN 用戶端組態包含使用者在其裝置上安裝的安裝程式套件。
* 對於 Mac 裝置，其中包含使用者在其裝置上安裝的 mobileconfig 檔案。

Zip 檔案也會提供 Azure 端的某些重要設定值，以便用於為這些裝置建立自己的設定檔。 這些值包括 VPN 閘道位址，已設定的通道類型、路由，以及用於閘道驗證的根憑證。

>[!NOTE]
>[!INCLUDE [TLS version changes](../../includes/vpn-gateway-tls-change.md)]
>

## <a name="gwsku"></a>哪些閘道 SKU 支援 P2S VPN？

[!INCLUDE [p2s-skus](../../includes/vpn-gateway-table-point-to-site-skus-include.md)]

* 「彙總輸送量基準測試」是以透過單一閘道彙總之多個通道的量值為基礎。 由於網際網路流量條件和您的應用程式行為，這不是保證的輸送量。
* 可以在 [價格] 頁面上找到價格資訊 
* 可以在 SLA 頁面上找到 SLA (服務等級協定) 資訊。

>[!NOTE]
>基本 SKU 不支援 IKEv2 或 RADIUS 驗證。
>

## <a name="configure"></a>如何設定 P2S 連線？

P2S 設定需要相當多的特定步驟。 下列文章包含的步驟可引導您進行 P2S 設定，然後連結以設定 VPN 用戶端裝置：

* [設定 P2S 連線 - RADIUS 驗證](point-to-site-how-to-radius-ps.md)

* [設定 P2S 連線 - Azure 原生憑證驗證](vpn-gateway-howto-point-to-site-rm-ps.md)

## <a name="faqcert"></a>原生 Azure 憑證驗證的常見問題集

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="faqradius"></a>RADIUS 驗證的常見問題集

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="next-steps"></a>後續步驟

* [設定 P2S 連線 - RADIUS 驗證](point-to-site-how-to-radius-ps.md)

* [設定 P2S 連線 - Azure 原生憑證驗證](vpn-gateway-howto-point-to-site-rm-ps.md)
