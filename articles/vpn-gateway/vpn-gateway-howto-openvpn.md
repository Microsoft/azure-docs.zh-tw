---
title: 如何設定 Azure VPN 閘道上的 OpenVPN：PowerShell |Microsoft Docs
description: 為 Azure VPN 閘道設定 OpenVPN 的步驟
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/26/2018
ms.author: cherylmc
ms.openlocfilehash: 958f4f46ec6ba407df7c739b7c62aa1489458485
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47408271"
---
# <a name="configure-openvpn-for-azure-point-to-site-vpn-gateway-preview"></a>為 Azure 點對站 VPN 閘道設定 OpenVPN (預覽)

本文協助您在 Azure VPN 閘道上設定 OpenVPN。 本文假設您已具有可用的點對站環境。 如果還沒有，請使用步驟 1 中的指示來建立點對站 VPN。

> [!IMPORTANT]
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="register"></a>註冊此功能

按一下這些步驟中的 [TryIt] 即可輕鬆地使用 Azure Cloud Shell 註冊此功能。

>[!NOTE]
>如果您未註冊此功能，將無法使用它。
>

按一下 [TryIt] 可開啟 Azure Cloud Shell，請複製並貼上下列命令：

```azurepowershell-interactive
Register-AzureRmProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowVnetGatewayOpenVpnProtocol
```
 
```azurepowershell-interactive
Get-AzureRmProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowVnetGatewayOpenVpnProtocol
```

一旦該功能顯示為已註冊，請將訂用帳戶註冊到 Microsoft.Network 命名空間。

```azurepowershell-interactive
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

## <a name="vnet"></a>1.建立點對站 VPN

如果您還沒有正常運作的點對站台環境，請依照指示來建立一個。 請參閱[建立點對站 VPN](vpn-gateway-howto-point-to-site-resource-manager-portal.md)，來建立及設及使用原生 Azure 憑證驗證的點對站 VPN 閘道。

## <a name="cmdlets"></a>2.安裝 PowerShell Cmdlet

安裝最新版的 Resource Manager PowerShell Cmdlet。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 這一點很重要，因為較早版本的 Cmdlet 未包含您在此練習中所需的現行值。

## <a name="enable"></a>3.在閘道上啟用 OpenVPN

在閘道上啟用 OpenVPN。 執行下列命令之前，務必確認閘道已針對點對站 (IKEv2 或 SSTP) 進行設定：

```powershell
$gw = Get-AzureRmVirtualNetworkGateway -ResourceGroupName $rgname -name $name
Set-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -VpnClientProtocol OpenVPN
```

## <a name="next-steps"></a>後續步驟

若要設定 OpenVPN 的用戶端，請參閱[設定 OpenVPN 用戶端](vpn-gateway-howto-openvpn-clients.md)。