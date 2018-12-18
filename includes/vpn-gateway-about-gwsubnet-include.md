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
ms.openlocfilehash: 3d76f955f8d8d3d2b269c09387717a6571adf8c3
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30197868"
---
虛擬網路閘道會使用稱為閘道子網路的特定子網路。 閘道子網路是您設定虛擬網路時，所指定虛擬網路 IP 位址範圍的一部分。 其包含虛擬網路閘道資源和服務所使用的 IP 位址。 子網路必須命名為 'GatewaySubnet' 才可供 Azure 部署閘道資源。 您無法指定不同的子網路來作為閘道資源的部署目的地。 如果您沒有名為 'GatewaySubnet' 的子網路，當您建立 VPN 閘道時，它將會失敗。

當您建立閘道子網路時，您可指定子網路包含的 IP 位址數目。 所需的 IP 位址數目取決於您想要建立的 VPN 閘道組態。 有些組態需要的 IP 位址比其他組態多。 我們建議您建立使用 /27 或/28 的閘道子網路。

如果您看到錯誤指出位址空間與子網路重疊，或子網路未包含在虛擬網路的位址空間內，請檢查您的 VNet 位址範圍。 您為虛擬網路所建立的位址範圍中可能沒有足夠的可用 IP 位址。 例如，如果您的預設子網路包含整個位址範圍，則沒有剩餘任何 IP 位址可供建立其他子網路。 您可以調整現有位址空間內的子網路以釋出 IP 位址，也可以指定其他位址範圍並於該處建立閘道子網路。