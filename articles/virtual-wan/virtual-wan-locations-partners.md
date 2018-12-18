---
title: Azure 虛擬 WAN 夥伴位置 | Microsoft Docs
description: 本文包含一份 Azure 虛擬 WAN 的夥伴和中樞位置的清單
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to connect find a Virtual WAN partner
ms.openlocfilehash: 76542392522ff14642ed7ccc6de8ed543d66513f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984988"
---
# <a name="virtual-wan-partners-and-virtual-hub-locations"></a>虛擬 WAN 夥伴與虛擬中樞位置

本文章提供有關虛擬 WAN 支援區域的資訊，以及連線到虛擬中樞的偏好的夥伴。

Azure 虛擬 WAN 是一種網路服務，透過 Azure 提供最佳且自動化的分支對分支連線。 虛擬 WAN 可讓您連線，並設定與 Azure 通訊的分支裝置。 您可以手動完成，或是透過虛擬 WAN 偏好的夥伴使用偏好的提供者裝置來完成。 使用偏好的夥伴裝置可讓您易於使用、簡化連線和組態管理。 從內部部署裝置連線到虛擬中樞是透過自動化方式建立。 虛擬中樞是受 Microsoft 管理的虛擬網路。 中樞包含不同的服務端點，可啟用您內部部署網路 (vpnsite) 中的連線。 每個地區都只能有一個中樞。

## <a name="regions"></a>區域

支援及可用的區域清單如下所示：

|地緣政治區域 | Azure 區域|
|---|---|
|北美洲 | 美國東部、美國西部、美國東部 2、美國西部 2、美國中部、美國中南部、美國中北部、美國中西部、加拿大中部、加拿大東部 |
|南美洲 |巴西南部 |
| 歐洲 | 法國中部、法國南部、北歐、西歐、英國西部、英國南部 |
| 亞洲 | 東亞、東南亞 |
| 日本  | 日本西部、日本東部 |
| 澳大利亞 | 澳洲東南部、澳洲東部 | 
| 澳洲政府 | 澳大利亞中部、澳大利亞中部 2 |
| 印度 | 印度西部、印度中部、印度南部 |
| 南韓 | 南韓中部、南韓南部 | 

## <a name="automation-from-connectivity-partners"></a>自動化連線的夥伴

本節描述連線提供者的自動化之概略詳細資訊。

連接到 Azure 虛擬 WAN 的裝置已內建自動化連接能力。 這通常是在裝置管理 UI (或相同功能) 中設定，它可在 VPN 分支裝置與 Azure 虛擬中樞 VPN 端點 (VPN 閘道) 之間設定連線和組態管理。

下列高階自動化是在裝置主控台/管理中心中設定：

* 裝置擁有存取 Azure 虛擬 WAN 資源群組的適當權限
* 將分支裝置上傳到 Azure 虛擬 WAN
* 自動下載 Azure 的連線資訊 
* 內部部署分支裝置的設定 

有些連線夥伴可能會擴充自動化功能以包括建立 Azure 虛擬中樞 VNet 與 VPN 閘道的能力。 如果您想要深入了解自動化，請參閱[設定自動化 – WAN 夥伴](virtual-wan-configure-automation-providers.md)。

## <a name="connectivity-through-preferred-partners"></a>透過偏好的夥伴連線

如果您的分支裝置夥伴未列在下一節中，請連絡您的分支裝置提供者，請他們傳送電子郵件到 azurevirtualwan@microsoft.com 與我們連絡。

您可以查看下列連結，以收集偏好的夥伴所提供之服務的詳細資訊。 

|偏好的夥伴|
|---|
|[Barracuda Networks](https://www.barracuda.com/AzurevWAN)|
| [Check Point](https://www.checkpoint.com/solutions/microsoft-azure-virtual-wan/) |
| [Citrix](https://www.citrix.com/global-partners/microsoft/sd-wan-for-azure-virtual-wan.html)|
|[Netfoundry](https://netfoundry.io/solutions/netfoundry-for-microsoft-azure-virtual-wan/)|
|[Palo Alto Networks](https://researchcenter.paloaltonetworks.com/2018/09/azure-vwan-integration/) |
|[Riverbed Technology](https://www.riverbed.com/go/steelconnect-azurewan.html)|
|[128 Technology](https://www.128technology.com/partners/azure) |

## <a name="next-steps"></a>後續步驟

如需有關虛擬 WAN 的詳細資訊，請參閱[虛擬 WAN 常見問題集](virtual-wan-faq.md)。

如需如何自動化連線到 Azure 虛擬 WAN 的詳細資訊，請參閱[虛擬 WAN 夥伴 - 如何自動化](virtual-wan-configure-automation-providers.md)。
