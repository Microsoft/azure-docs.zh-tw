---
title: 針對 Azure 站對站 VPN 連線無法連線進行疑難排解| Microsoft Docs
description: 了解如何針對突然停止運作且無法重新連線的站對站 VPN 連線進行疑難排解。
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
ms.openlocfilehash: dfd29e0956793cf776b9c0ea5ddbd4689ebcb015
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34072295"
---
# <a name="troubleshooting-an-azure-site-to-site-vpn-connection-cannot-connect-and-stops-working"></a>疑難排解：Azure 站對站 VPN 連線無法連線並停止運作

當您在內部部署網路與 Azure 虛擬網路之間設定站對站 VPN 連線之後，該 VPN 連線突然停止運作且無法重新連線。 本文提供可協助您解決此問題的疑難排解步驟。 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-steps"></a>疑難排解步驟

若要解決此問題，請先嘗試[重設 Azure VPN 閘道](vpn-gateway-resetgw-classic.md)，並從內部部署 VPN 裝置重設通道。 如果問題持續發生，請依照下列步驟執行以找出問題的原因。

### <a name="prerequisite-step"></a>必要步驟

檢查 Azure VPN 閘道的類型。

1. 移至 [Azure 入口網站](https://portal.azure.com)。

2. 檢查 VPN 閘道的 [概觀] 頁面來取得類型資訊。
    
    ![閘道概觀](media\vpn-gateway-troubleshoot-site-to-site-cannot-connect\gatewayoverview.png)

### <a name="step-1-check-whether-the-on-premises-vpn-device-is-validated"></a>步驟 1. 檢查內部部署 VPN 裝置是否經過驗證

1. 檢查您是否使用[經過驗證的 VPN 裝置和作業系統版本](vpn-gateway-about-vpn-devices.md#devicetable)。 如果裝置不是經過驗證的 VPN 裝置，您可能需要連絡裝置製造商，以了解是否有任何相容性問題。

2. 確定已正確設定 VPN 裝置。 如需詳細資訊，請參閱[編輯裝置組態範例](vpn-gateway-about-vpn-devices.md#editing)。

### <a name="step-2-verify-the-shared-key"></a>步驟 2. 確認共用金鑰

比較內部部署 VPN 裝置和 Azure 虛擬網路 VPN 的共用金鑰，以確認金鑰相符合。 

若要檢視 Azure VPN 連線的共用金鑰，請使用下列其中一個方法：

**Azure 入口網站**

1. 移至您建立的 VPN 閘道站對站連線。

2. 在 [設定] 區段中，按一下 [共用金鑰]。
    
    ![共用金鑰](media/vpn-gateway-troubleshoot-site-to-site-cannot-connect/sharedkey.png)

**Azure PowerShell**

針對 Azure Resource Manager 部署模型：

    Get-AzureRmVirtualNetworkGatewayConnectionSharedKey -Name <Connection name> -ResourceGroupName <Resource group name>

針對傳統部署模型：

    Get-AzureVNetGatewayKey -VNetName -LocalNetworkSiteName

### <a name="step-3-verify-the-vpn-peer-ips"></a>步驟 3. 確認 VPN 對等互連 IP

-   Azure 中「區域網路閘道」物件內的 IP 定義應與內部部署裝置 IP 相符合。
-   內部部署裝置上設定的 Azure 閘道 IP 定義應與 Azure 閘道 IP 相符合。

### <a name="step-4-check-udr-and-nsgs-on-the-gateway-subnet"></a>步驟 4. 檢查閘道子網路上的 UDR 和 NSG

檢查並移除閘道子網路上的使用者定義路由 (UDR) 或網路安全性群組 (NSG)，然後測試結果。 如果問題已解決，請驗證 UDR 或 NSG 套用的設定。

### <a name="step-5-check-the-on-premises-vpn-device-external-interface-address"></a>步驟 5。 檢查內部部署 VPN 裝置外部介面位址

- 如果 Azure 的**區域網路**定義中包含 VPN 裝置的網際網路對應 IP 位址，則您可能偶爾會遇到連線中斷的情況。
- 裝置的外部介面必須直接位在網際網路上。 網際網路與裝置之間不應該有網路位址轉譯或防火牆。
- 若要設定防火牆叢集以具有虛擬 IP，您必須解散叢集，並直接將 VPN 設備公開給閘道可介接的公用介面。

### <a name="step-6-verify-that-the-subnets-match-exactly-azure-policy-based-gateways"></a>步驟 6. 確認子網路完全相符合 (Azure 原則式閘道)

-   確認 Azure 虛擬網路和內部部署定義的子網路位址空間完全相符。
-   確認子網路在**區域網路閘道**和內部部署網路的內部部署定義之間完全相符合。

### <a name="step-7-verify-the-azure-gateway-health-probe"></a>步驟 7. 確認 Azure 閘道健康狀態探查

1. 瀏覽至下列 URL 來開啟健康狀態探查：

    `https://<YourVirtualNetworkGatewayIP>:8081/healthprobe`

2. 按一下以略過憑證警告。
3. 如果您收到回應，表示 VPN 閘道的健康狀態良好。 如果未收到回應，閘道的健康狀態可能有問題，或可能是閘道子網路上的 NSG 造成問題。 下列文字是回應的範例：

    &lt;?xml version="1.0"?> <string xmlns="http://schemas.microsoft.com/2003/10/Serialization/">主要執行個體: GatewayTenantWorker_IN_1 GatewayTenantVersion: 14.7.24.6</string&gt;

### <a name="step-8-check-whether-the-on-premises-vpn-device-has-the-perfect-forward-secrecy-feature-enabled"></a>步驟 8。 檢查內部部署 VPN 裝置是否已啟用完整轉寄密碼功能

完整轉寄密碼功能可能會造成連線中斷的問題。 如果 VPN 裝置已啟用完整轉寄密碼，請停用該功能。 然後更新 VPN 閘道 IPsec 原則。

## <a name="next-steps"></a>後續步驟

-   [設定對虛擬網路的站對站連線](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
-   [設定站對站 VPN 連線的 IPsec/IKE 原則](vpn-gateway-ipsecikepolicy-rm-powershell.md)
