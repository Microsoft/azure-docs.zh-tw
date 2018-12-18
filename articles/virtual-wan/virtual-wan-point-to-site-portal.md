---
title: 使用 Azure 虛擬 WAN 來與 Azure 建立點對站連線 | Microsoft Docs
description: 在本教學課程中，了解如何使用 Azure 虛擬 WAN 來與 Azure 建立點對站 VPN 連線。
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 09/26/2018
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to connect remote users to my VNets using Virtual WAN and I don't want to go through a Virtual WAN partner.
ms.openlocfilehash: 8a4c0c1426200e6c2d5041131fd0dd9cde4761cf
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409281"
---
# <a name="tutorial-create-a-point-to-site-connection-using-azure-virtual-wan-preview"></a>教學課程：使用 Azure 虛擬 WAN 來建立點對站連線 (預覽)

本教學課程會示範如何使用虛擬 WAN，透過 IPsec/IKE (IKEv2) 或 OpenVPN VPN 連線來與 Azure 中的資源連線。 此類型的連線需要在用戶端電腦上設定用戶端。 如需有關虛擬 WAN 的詳細資訊，請參閱[虛擬 WAN 概觀](virtual-wan-about.md)。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 WAN
> * 建立 P2S 設定
> * 建立中樞
> * 將 P2S 設定套用到中樞
> * 將 VNet 連線至中樞
> * 下載並套用 VPN 用戶端設定
> * 檢視您的虛擬 WAN
> * 檢視資源健康情況
> * 監視連線

> [!IMPORTANT]
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。
>

## <a name="before-you-begin"></a>開始之前

[!INCLUDE [Before you begin](../../includes/virtual-wan-tutorial-vwan-before-include.md)]

## <a name="register"></a>註冊此功能

按一下 [TryIt] 即可輕鬆地使用 Azure Cloud Shell 註冊此功能。

>[!NOTE]
>如果您未註冊此功能，將無法使用它，或在入口網站中看到它。
>
>

按一下 [TryIt] 可開啟 Azure Cloud Shell，請複製並貼上下列命令：

```azurepowershell-interactive
Register-AzureRmProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowP2SCortexAccess
```
 
```azurepowershell-interactive
Register-AzureRmProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowVnetGatewayOpenVpnProtocol
```

```azurepowershell-interactive
Get-AzureRmProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowP2SCortexAccess
```

```azurepowershell-interactive
Get-AzureRmProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowVnetGatewayOpenVpnProtocol
```

一旦該功能顯示為已註冊，請將訂用帳戶註冊到 Microsoft.Network 命名空間。

```azurepowershell-interactive
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

## <a name="vnet"></a>1.建立虛擬網路

[!INCLUDE [Create a virtual network](../../includes/virtual-wan-tutorial-vnet-include.md)]

## <a name="openvwan"></a>2.建立虛擬 WAN

透過瀏覽器瀏覽至 [Azure 入口網站 (預覽)](http://aka.ms/azurevirtualwanpreviewfeatures)，並使用您的 Azure 帳戶登入。

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-vwan-include.md)]

## <a name="hub"></a>3.建立中樞

> [!NOTE]
> 請勿在此步驟中選取「包含點對站閘道」設定。
>

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-hub-include.md)]

## <a name="site"></a>4.建立 P2S 設定

P2S 設定會定義用於連線遠端用戶端的參數。

1. 瀏覽至 [所有資源]。
2. 按一下您建立的虛擬 WAN。
3. 在 [虛擬 WAN 架構] 底下，按一下 [點對站設定]。
4. 按一下頁面頂端的 [+ 新增點對站設定] 以開啟 [建立新的點對站設定] 頁面。
5. 在 [建立新的點對站設定] 頁面上，填寫下列欄位：

  *  **設定名稱** - 這是您要用來指稱設定的名稱。
  *  **通道類型** - 用於通道的通訊協定。
  *  **位址集區** - 這是會指派給用戶端的 IP 位址所來自的集區。
  *  **根憑證名稱** - 憑證的描述性名稱。
  *  **根憑證資料** - Base-64 編碼的 X.509 憑證資料。

5. 按一下 [建立] 來建立設定。

## <a name="hub"></a>5.編輯中樞指派

1. 在虛擬 WAN 的頁面上，按一下 [點對站設定]。
2. 在 [中樞] 下，您會看到尚未連線至中樞的設定清單。
3. 選取您想要與之關聯的設定，然後按一下 [編輯中樞指派]
4. 從下拉式清單中，選取您想要與設定關聯的中樞。
5. 按一下 [指派]。 
6. 此作業最多可能需要 30 分鐘的時間才能完成。

## <a name="vnet"></a>6.將 VNet 連線至中樞

在此步驟中，您會在中樞和 VNet 之間建立等互連的連線。 為您想要連線的每個 VNet 重複這些步驟。

1. 在您的虛擬 WAN 頁面上，按一下 [虛擬網路連線]。
2. 在 [虛擬網路連線] 頁面上，按一下 [+ 新增連線]。
3. 在 [新增連線] 頁面上，填寫下列欄位︰

    * **名稱** - 為您的連線命名。
    * **中樞** - 選取要與此連線產生關聯的中樞。
    * **訂用帳戶** - 請確認訂用帳戶。
    * **虛擬網路** - 選取要與此中樞連線的虛擬網路。 虛擬網路不能有現有的虛擬網路閘道。

## <a name="device"></a>7.下載 VPN 設定檔

使用 VPN 設定檔來設定用戶端。

1. 在虛擬 WAN 的頁面上，按一下 [點對站設定]。
2. 在頁面頂端，按一下 [下載點對站設定檔]。 
3. 檔案建立完成之後，您可以按一下連結來下載。
4. 使用設定檔檔案來設定點對站用戶端。

## <a name="device"></a>8.設定點對站用戶端
使用下載的設定檔來設定遠端存取用戶端。 每個作業系統的程序不同，請遵循下面的正確指示：

### <a name="microsoft-windows"></a>Microsoft Windows
#### <a name="openvpn"></a>OpenVPN

1.  從官方網站下載並安裝 OpenVPN 用戶端。
2.  下載閘道的 VPN 設定檔。 這可從 Azure 入口網站中的 [點對站設定] 索引標籤，或是 PowerShell 中的 New-AzureRmVpnClientConfiguration 來完成。
3.  將設定檔解壓縮。 在記事本中開啟 OpenVPN 資料夾中的 vpnconfig.ovpn 設定檔。
4.  以 Base64 的 P2S 用戶端憑證公開金鑰填入 P2S 用戶端憑證區段。 在 PEM 格式的憑證中，您只需開啟 .cer 檔案並在憑證標題之間複製 Base64 金鑰。 在這裡查看如何匯出憑證來取得編碼的公開金鑰。
5.  以 base64 的 P2S 用戶端憑證私密金鑰填入私密金鑰區段。 在這裡查看如何擷取私密金鑰。
6.  請勿變更任何其他欄位。 使用用戶端輸入中填入的設定來連線至 VPN。
7.  將 vpnconfig.ovpn 檔案複製到 C:\Program Files\OpenVPN\config 資料夾。
8.  以滑鼠右鍵按一下系統匣中的 OpenVPN 圖示，然後按一下 [連線]。

#### <a name="ikev2"></a>IKEv2

1. 選取 Windows 電腦架構所對應的 VPN 用戶端組態檔。 若是 64 位元的處理器架構，請選擇 'VpnClientSetupAmd64' 安裝程式套件。 若是 32 位元的處理器架構，請選擇 'VpnClientSetupX86' 安裝程式套件。
2. 對套件按兩下來加以安裝。 如果您看到 SmartScreen 快顯視窗，請按一下 [更多資訊]，然後按一下 [仍要執行]。
3. 在用戶端電腦上，瀏覽至 [網路設定]，然後按一下 [VPN]。 VPN 連線會顯示其連線的虛擬網路名稱。
4. 嘗試連線之前，請確認您已在用戶端電腦上安裝用戶端憑證。 使用原生 Azure 憑證驗證類型時，必須提供用戶端憑證才能通過驗證。 如需有關產生憑證的詳細資訊，請參閱產生憑證。 如需有關如何安裝用戶端憑證的資訊，請參閱安裝用戶端憑證。

### <a name="mac-os-x"></a>Mac (OS X)
#### <a name="openvpn"></a>OpenVPN

1.  下載並安裝 OpenVPN 用戶端，例如 https://tunnelblick.net/downloads.html 中的 TunnelBlik 
2.  下載閘道的 VPN 設定檔。 這可從 Azure 入口網站中的 [點對站設定] 索引標籤，或是 PowerShell 中的 New-AzureRmVpnClientConfiguration 來完成。
3.  將設定檔解壓縮。 在記事本中開啟 OpenVPN 資料夾中的 vpnconfig.ovpn 設定檔。
4.  以 Base64 的 P2S 用戶端憑證公開金鑰填入 P2S 用戶端憑證區段。 在 PEM 格式的憑證中，您只需開啟 .cer 檔案並在憑證標題之間複製 Base64 金鑰。 在這裡查看如何匯出憑證來取得編碼的公開金鑰。
5.  以 base64 的 P2S 用戶端憑證私密金鑰填入私密金鑰區段。 在這裡查看如何擷取私密金鑰。
6.  請勿變更任何其他欄位。 使用用戶端輸入中填入的設定來連線至 VPN。
7.  按兩下設定檔檔案以在 tunnelblik 中建立設定檔
8.  從應用程式資料夾啟動 Tunnelblik
9.  按一下系統匣中的 Tunneblik 圖示並挑選連線

#### <a name="ikev2"></a>IKEv2

Azure 不提供用於原生 Azure 憑證驗證的 mobileconfig 檔案。 您必須在將會連線到 Azure 的每部 Mac 上，手動設定原生 IKEv2 VPN 用戶端。 Generic 資料夾含有設定所需的所有資訊。 如果未出現下載的 Generic 資料夾，則可能是未將 IKEv2 選為通道型別。 一旦選取 IKEv2 後，重新產生 zip 檔案以擷取 Generic 資料夾。 Generic 資料夾包含下列檔案：

- VpnSettings.xml，此檔案包含重要的設定，例如伺服器位址和通道類型。
- VpnServerRoot.cer，此檔案包含所需的根憑證，以供您在 P2S 連線設定期間驗證 Azure VPN 閘道。

## <a name="viewwan"></a>9.檢視您的虛擬 WAN

1. 瀏覽至虛擬 WAN。
2. 在 [概觀] 頁面中，地圖上的每個點都代表著中樞。 暫留在任一點上，即可檢視中樞健康情況摘要。
3. 在 [中樞與連線] 區段中，您可以檢視中樞狀態、網站、區域、VPN 連線狀態和輸入與輸出位元組。

## <a name="viewhealth"></a>10.檢視資源健康情況

1. 瀏覽至您的 WAN。
2. 在 WAN 頁面上的 [支援 + 疑難排解] 區段中，按一下 [健康情況] 並檢視您的資源。

## <a name="connectmon"></a>11.監視連線

建立連線以監視 Azure VM 和遠端站台之間的通訊。 如需有關如何設定連線監視的資訊，請參閱[監視網路通訊](~/articles/network-watcher/connection-monitor.md)。 來源欄位是 Azure 中的 VM IP，目的地 IP 是位址是網站 IP。

## <a name="cleanup"></a>12.清除資源

您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組，以及其所包含的所有資源。 將 "myResourceGroup" 取代為您的資源群組名稱，然後執行下列 PowerShell 命令：

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立 WAN
> * 建立網站
> * 建立中樞
> * 將中樞連線至網站
> * 將 VNet 連線至中樞
> * 下載並套用 VPN 裝置組態
> * 檢視您的虛擬 WAN
> * 檢視資源健康情況
> * 監視連線

若要深入了解虛擬 WAN，請參閱[虛擬 WAN 概觀](virtual-wan-about.md)頁面。