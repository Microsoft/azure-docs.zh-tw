---
title: 針對 Azure 站對站 VPN 間歇性中斷問題進行疑難排解| Microsoft Docs
description: 了解如何針對 Azure 站對站 VPN 連線間歇性中斷問題進行疑難排解。
services: vpn-gateway
documentationcenter: na
author: chadmath
manager: cshepard
editor: ''
tags: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: 9c827469080195054d4ff70ab72fc123365a73df
ms.sourcegitcommit: e14229bb94d61172046335972cfb1a708c8a97a5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/14/2018
ms.locfileid: "34160563"
---
# <a name="troubleshooting-azure-site-to-site-vpn-disconnects-intermittently"></a>疑難排解：Azure 站對站 VPN 間歇性中斷

您可能會遇到新的或現有的 Microsoft Azure 網站間 VPN 連線能力不穩定或間歇性中斷的問題。 本文提供疑難排解步驟，可協助您找出原因並解決問題。 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-steps"></a>疑難排解步驟

### <a name="prerequisite-step"></a>必要步驟

檢查 Azure 虛擬網路閘道的類型：

1. 移至 [Azure 入口網站 ](https://portal.azure.com)。
2. 檢查虛擬網路閘道的 [概觀] 頁面來取得類型資訊。
    
    ![閘道概觀](media\vpn-gateway-troubleshoot-site-to-site-disconnected-intermittently\gatewayoverview.png)

### <a name="step-1-check-whether-the-on-premises-vpn-device-is-validated"></a>步驟 1：檢查內部部署 VPN 裝置是否經過驗證

1. 檢查您是否使用[經過驗證的 VPN 裝置和作業系統版本](vpn-gateway-about-vpn-devices.md#devicetable)。 如果 VPN 裝置未經驗證，您可能需要連絡裝置製造商，以了解是否有任何相容性問題。
2. 確定已正確設定 VPN 裝置。 如需詳細資訊，請參閱[編輯裝置組態範例](vpn-gateway-about-vpn-devices.md#editing)。

### <a name="step-2-check-the-security-association-settingsfor-policy-based-azure-virtual-network-gateways"></a>步驟 2：檢查安全性關聯設定 (適用於原則式 Azure 虛擬網路閘道)

1. 確定 Microsoft Azure 中的「區域網路閘道」定義中的虛擬網路、子網路和範圍皆與內部部署 VPN 裝置上的設定相同。
2. 確認安全性關聯設定相符合。

### <a name="step-3-check-for-user-defined-routes-or-network-security-groups-on-gateway-subnet"></a>步驟 3：檢查閘道子網路上使用者定義的路由或網路安全性群組

閘道子網路上的使用者定義路由可能會限制某些流量並允許其他流量。 這可能會讓某些流量認為 VPN 連線不穩定，而其他流量覺得它穩定。 

### <a name="step-4-check-the-one-vpn-tunnel-per-subnet-pair-setting-for-policy-based-virtual-network-gateways"></a>步驟 4：檢查「每個子網路配一個 VPN 通道」設定 (適用於原則式虛擬網路閘道)

確定已針對原則式虛擬網路閘道將內部部署 VPN 裝置設定為「每個子網路配一個 VPN 通道」。

### <a name="step-5-check-for-security-association-limitation-for-policy-based-virtual-network-gateways"></a>步驟 5：檢查安全性關聯限制 (適用於原則式虛擬網路閘道)

原則式虛擬網路閘道有 200 個子網路安全性關聯組的限制。 如果 Azure 虛擬網路子網路數目乘以區域子網路數目的結果大於 200，則您會遇到子網路偶爾連線中斷的情況。

### <a name="step-6-check-on-premises-vpn-device-external-interface-address"></a>步驟 6：檢查內部部署 VPN 裝置外部介面位址

- 如果 Azure 的「區域網路閘道」定義中包含 VPN 裝置對網際網路的 IP 位址，則您可能會遇到偶爾連線中斷的情況。
- 裝置的外部介面必須直接位在網際網路上。 網際網路與裝置之間不應該有網路位址轉譯 (NAT) 或防火牆。
-  如果您將防火牆叢集設定為具有虛擬 IP，則您必須解散叢集，並直接將 VPN 設備公開給閘道可介接的公用介面。

### <a name="step-7-check-whether-the-on-premises-vpn-device-has-perfect-forward-secrecy-enabled"></a>步驟 7：檢查內部部署 VPN 裝置是否已啟用「完整轉寄密碼」

「完整轉寄密碼」功能可能會造成連線中斷的問題。 如果 VPN 裝置已啟用「完整轉寄密碼」，請停用該功能。 然後[更新虛擬網路閘道 IPsec 原則](vpn-gateway-ipsecikepolicy-rm-powershell.md#managepolicy)。

## <a name="next-steps"></a>後續步驟

- [設定對虛擬網路的站對站連線](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
- [設定站對站 VPN 連線的 IPsec/IKE 原則](vpn-gateway-ipsecikepolicy-rm-powershell.md)

