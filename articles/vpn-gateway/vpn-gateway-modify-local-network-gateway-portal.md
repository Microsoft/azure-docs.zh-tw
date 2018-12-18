---
title: 修改區域網路閘道 IP 位址首碼和 VPN 閘道 IP 位址| Azure| 入口網站| Microsoft Docs
description: 本文逐步解說如何使用 Azure 入口網站來變更區域網路閘道的 IP 位址首碼。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/19/2017
ms.author: cherylmc
ms.openlocfilehash: bdd6f90fe97408bd45a72adf58bfdc190207de46
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38651931"
---
# <a name="modify-local-network-gateway-settings-using-the-azure-portal"></a>使用 Azure 入口網站修改區域網路閘道設定

有時候區域網路閘道 AddressPrefix 或 GatewayIPAddress 的設定會變更。 本文將說明如何修改區域網路閘道設定。 您也可以從下列清單選取不同的選項來使用不同的方法修改這些設定：

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-modify-local-network-gateway-portal.md)
> * [PowerShell](vpn-gateway-modify-local-network-gateway.md)
> * [Azure CLI](vpn-gateway-modify-local-network-gateway-cli.md)
>
>


## <a name="ipaddprefix"></a>修改 IP 位址首碼

當您修改 IP 位址首碼時，使用的步驟會依您的區域網路閘道是否有連線而不同。

[!INCLUDE [modify prefix](../../includes/vpn-gateway-modify-ip-prefix-portal-include.md)]

## <a name="gwip"></a>修改閘道 IP 位址

如果您想要連線的 VPN 裝置已變更其公用 IP 位址，您需要修改區域網路閘道，以反映該變更。 當您變更公用 IP 位址時，使用的步驟會依您的區域網路閘道是否有連線而不同。

[!INCLUDE [modify gateway IP](../../includes/vpn-gateway-modify-lng-gateway-ip-portal-include.md)]

## <a name="next-steps"></a>後續步驟

您可以驗證閘道連線。 請參閱 [驗證閘道連線](vpn-gateway-verify-connection-resource-manager.md)。