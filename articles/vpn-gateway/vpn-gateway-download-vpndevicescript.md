---
title: 下載 S2S VPN 連線的 VPN 裝置設定指令碼：Azure Resource Manager | Microsoft Docs
description: 本文將逐步引導您使用 Azure Resource Manager 來下載 VPN 裝置設定指令碼，以便利用 Azure VPN 閘道進行 S2S VPN 連線。
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
ms.date: 03/29/2018
ms.author: yushwang
ms.openlocfilehash: 254f5012bfbf827aebc20d90405636dcb204193c
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30317767"
---
# <a name="download-vpn-device-configuration-scripts-for-s2s-vpn-connections"></a>下載可供進行 S2S VPN 連線的 VPN 裝置設定指令碼

本文將逐步引導您使用 Azure Resource Manager 來下載 VPN 裝置設定指令碼，以便利用 Azure VPN 閘道進行 S2S VPN 連線。 下圖顯示高階工作流程。

![下載指令碼](./media/vpn-gateway-download-vpndevicescript/downloaddevicescript.png)

下列裝置有可用的指令碼：

[!INCLUDE [scripts](../../includes/vpn-gateway-device-configuration-scripts.md)]

## <a name="about"></a>關於 VPN 裝置設定指令碼

跨單位 VPN 連線是由 Azure VPN 閘道、內部部署 VPN 裝置以及讓兩者相連的 IPsec S2S VPN 通道所組成。 典型的工作流程包含下列步驟：

1. 建立及設定 Azure VPN 閘道 (虛擬網路閘道)
2. 建立及設定能表示您內部部署網路和 VPN 裝置的 Azure 區域網路閘道
3. 建立及設定 Azure VPN 閘道與區域網路閘道之間的 Azure VPN 連線
4. 設定以區域網路閘道表示的內部部署 VPN 裝置，以使用 Azure VPN 閘道建立實際的 S2S VPN 通道

您可以使用 Azure [入口網站](vpn-gateway-howto-site-to-site-resource-manager-portal.md)、[PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md) 或 [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md) 來完成步驟 1 到 3。 最後一個步驟牽涉到設定 Azure 外部的內部部署 VPN 裝置。 這項功能可讓您下載 VPN 裝置的設定指令碼，且對應的 Azure VPN 閘道、虛擬網路、內部部署網路位址首碼和 VPN 連線屬性等值均已填入。 您可以使用指令碼作為起點，或透過設定主控台將指令碼直接套用至您的內部部署 VPN 裝置。

> [!IMPORTANT]
> * 每個 VPN 裝置設定指令碼的語法不同，高度相依於機型和韌體版本。 請針對可用範本特別注意您的裝置機型和版本資訊。
> * 某些參數值在裝置上必須是唯一的，而且不存取裝置就無法確定。 Azure 產生的設定指令碼會預先填入這些值，但是您必須確定所提供的值在您的裝置上是有效的。 例如：
>    * 介面號碼
>    * 存取控制清單號碼
>    * 原則名稱或號碼等等。
> * 尋找內嵌在指令碼中的關鍵字 "**REPLACE**"，以在套用指令碼之前找到進行確認所需的參數。
> * 某些範本包含 "**CLEANUP**" 區段，您可套用以移除設定。 清除區段預設會註解化。

## <a name="download-the-configuration-script-from-azure-portal"></a>從 Azure 入口網站下載設定指令碼

建立 Azure VPN 閘道、區域網路閘道以及讓兩者相連的連線資源。 下列網頁會指導您逐步執行步驟：

* [在 Azure 入口網站中建立站對站連線](vpn-gateway-howto-site-to-site-resource-manager-portal.md)

在建立連線資源後，請遵循下列指示以下載 VPN 裝置設定指令碼：

1. 透過瀏覽器瀏覽至 [Azure 入口網站](http://portal.azure.com)，並視需要使用您的 Azure 帳戶登入
2. 移至您建立的連線資源。 您可以藉由按一下 [所有服務]，然後按一下 [網路] 和 [連線]，以尋找所有連線資源的清單。

    ![連線清單](./media/vpn-gateway-download-vpndevicescript/connectionlist.png)

3. 按一下要設定的連線。

    ![連線概觀](./media/vpn-gateway-download-vpndevicescript/connectionoverview.png)

4. 按一下 [連線] 概觀頁面中以紅色醒目提示的 [下載設定] 連結；這會開啟 [下載設定] 頁面。

    ![download-script-1](./media/vpn-gateway-download-vpndevicescript/downloadscript-1.png)

5. 選取您 VPN 裝置的機型系列和韌體版本，然後按一下 [下載設定] 按鈕。

    ![download66-script-2](./media/vpn-gateway-download-vpndevicescript/downloadscript-2.PNG)

6. 系統會提示您從瀏覽器儲存所下載的指令碼 (文字檔)。
7. 在下載設定指令碼後，使用文字編輯器加以開啟，並搜尋關鍵字 "REPLACE" 以找出並檢查可能需要取代的參數。

    ![edit-script](./media/vpn-gateway-download-vpndevicescript/editscript.png)

## <a name="download-the-configuration-script-using-azure-powershell"></a>使用 Azure PowerShell 下載設定指令碼

您也可以使用 Azure PowerShell 下載設定指令碼，如下列範例所示：

```azurepowershell-interactive
$RG          = "TestRG1"
$GWName      = "VNet1GW"
$Connection  = "VNet1toSite1"

# List the available VPN device models and versions
Get-AzureRmVirtualNetworkGatewaySupportedVpnDevice -Name $GWName -ResourceGroupName $RG

# Download the configuration script for the connection
Get-AzureRmVirtualNetworkGatewayConnectionVpnDeviceConfigScript -Name $Connection -ResourceGroupName $RG -DeviceVendor Juniper -DeviceFamily Juniper_SRX_GA -FirmwareVersion Juniper_SRX_12.x_GA
```

## <a name="apply-the-configuration-script-to-your-vpn-device"></a>將設定指令碼套用至您的 VPN 裝置

在下載並驗證設定指令碼之後，下一步是將指令碼套用至您的 VPN 裝置。 實際程序會根據您的 VPN 裝置廠牌與機型而異。 請參閱 VPN 裝置的操作手冊或指示頁面。

## <a name="next-steps"></a>後續步驟

繼續設定您的[站對站連線](vpn-gateway-howto-site-to-site-resource-manager-portal.md)。