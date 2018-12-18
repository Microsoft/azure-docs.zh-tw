---
title: 關於密碼編譯需求和 Azure VPN 閘道 | Microsoft Docs
description: 本文說明密碼編譯需求與 Azure VPN 閘道。
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: 238cd9b3-f1ce-4341-b18e-7390935604fa
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/22/2017
ms.author: yushwang
ms.openlocfilehash: d2f3da47f1d4eebe1b81964790ff6612dd78155d
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "23125176"
---
# <a name="about-cryptographic-requirements-and-azure-vpn-gateways"></a>關於密碼編譯需求和 Azure VPN 閘道

此文章說明如何針對 Azure 內的跨單位 S2S VPN 通道和 VNet 對 VNet 連線，設定 Azure VPN 閘道，以滿足您的密碼編譯需求。 

## <a name="about-ipsec-and-ike-policy-parameters-for-azure-vpn-gateways"></a>關於 Azure VPN 閘道的 IPsec 和 IKE 原則參數
IPsec 和 IKE 通訊協定標準支援各種不同的密碼編譯演算法的各種組合。 如果客戶未要求特定的密碼編譯演算法和參數組合，則 Azure VPN 閘道會使用一組預設的提案。 選擇預設原則集的用意，是要在預設設定中將與各種協力廠商 VPN 裝置的互通性最大化。 如此一來，原則和提案數目就無法涵蓋所有可用的密碼編譯演算法和金鑰長度組合。

下列文件列出 Azure VPN 閘道的預設原則集：[關於 VPN 裝置和站對站 VPN 閘道連線的 IPsec/IKE 參數](vpn-gateway-about-vpn-devices.md)。

## <a name="cryptographic-requirements"></a>密碼編譯需求
針對需要特定密碼編譯演算法或參數的通訊 (通常是基於相容性或安全性需求的原因)，客戶現在可以自行設定 Azure VPN 閘道，以搭配使用自訂的 IPsec/IKE 原則和特定的密碼編譯演算法和金鑰長度，而不使用 Azure 的預設原則集。

例如，Azure VPN 閘道的 IKEv2 主要模式原則僅使用 Diffie-Hellman 群組 2 (1024 位元)，但客戶可能需要指定更強的群組以用於 IKE，例如群組 14 (2048 位元)、群組 24 (2048 位元 MODP 群組) 或 ECP (橢圓曲線群組) 256 或 384 位元 (分別為群組 19 和群組 20)。 類似的需求亦適用於 IPsec 快速模式原則。

## <a name="custom-ipsecike-policy-with-azure-vpn-gateways"></a>自訂的 IPsec/IKE 原則與 Azure VPN 閘道
Azure VPN 閘道現在支援個別連線的自訂 IPsec/IKE 原則。 針對站對站或 VNet 對 VNet 連線，您可以針對 IPsec 和 IKE 選擇特定的密碼編譯演算法組合，搭配所需的金鑰強度，如下列範例所示：

![ipsec-ike-policy](./media/vpn-gateway-about-compliance-crypto/ipsecikepolicy.png)

您可以建立 IPsec/IKE 原則，並將其套用至新連線或現有連線。 

### <a name="workflow"></a>工作流程

1. 按照其他使用說明文件所述，依據連線的拓撲來建立虛擬網路、VPN 閘道或區域網路閘道。
2. 建立 IPsec/IKE 原則
3. 您可以在建立 S2S 或 VNet 對 VNet 連線時套用原則。
4. 如果已經建立連線，您可以將原則套用至現有連線或將其更新。


## <a name="ipsecike-policy-faq"></a>IPsec/IKE 原則常見問題集

[!INCLUDE [vpn-gateway-ipsecikepolicy-faq-include](../../includes/vpn-gateway-faq-ipsecikepolicy-include.md)]


## <a name="next-steps"></a>後續步驟
如需為連線設定自訂 IPsec/IKE 原則的逐步指示，請參閱[設定 IPsec/IKE 原則](vpn-gateway-ipsecikepolicy-rm-powershell.md)。

另請參閱[連線多個原則型 VPN 裝置](vpn-gateway-connect-multiple-policybased-rm-ps.md)，以深入了解 UsePolicyBasedTrafficSelectors 選項。