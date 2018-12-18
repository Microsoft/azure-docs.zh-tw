---
title: 如何設定 Azure VPN 閘道的 OpenVPN 用戶端 |Microsoft Docs
description: 為 Azure VPN 閘道設定 OpenVPN 用戶端的步驟
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: cherylmc
ms.openlocfilehash: 141f29ea1c7ebdd8208dda6bc39a60f7873dab7d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977837"
---
# <a name="configure-openvpn-clients-for-azure-vpn-gateway-preview"></a>設定 Azure VPN 閘道的 OpenVPN 用戶端 (預覽)

這篇文章可協助您設定 OpenVPN 用戶端。

> [!IMPORTANT]
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。
>

## <a name="before-you-begin"></a>開始之前

請確認您已完成為 VPN 閘道設定 OpenVPN 的步驟。 如需詳細資訊，請參閱[設定 Azure VPN 閘道的 OpenVPN 用戶端](vpn-gateway-howto-openvpn.md)。

## <a name="windows"></a>Windows 用戶端

1. 從官方 [OpenVPN 網站](https://openvpn.net/index.php/open-source/downloads.html)下載並安裝 OpenVPN 用戶端。
2. 下載閘道的 VPN 設定檔。 這可從 Azure 入口網站中的 [點對站設定] 索引標籤，或是 PowerShell 中的 'New-AzureRmVpnClientConfiguration' 來完成。
3. 將設定檔解壓縮。 然後，在記事本中開啟 OpenVPN 資料夾中的 vpnconfig.ovpn 設定檔。
4. 以 Base64 的 P2S 用戶端憑證公開金鑰填入 P2S 用戶端憑證區段。 在 PEM 格式的憑證中，您只需開啟 .cer 檔案並在憑證標題之間複製 Base64 金鑰。 在這裡查看如何匯出憑證來取得編碼的公開金鑰。
5. 以 Base64 的 P2S 用戶端憑證私密金鑰填入私密金鑰區段。 如需如何擷取私用金鑰的資訊，請參閱[匯出金鑰](vpn-gateway-certificates-point-to-site.md#clientexport)。
6. 請勿變更任何其他欄位。 使用用戶端輸入中填入的設定來連線至 VPN。
7. 將 vpnconfig.ovpn 檔案複製到 C:\Program Files\OpenVPN\config 資料夾。
8. 以滑鼠右鍵按一下系統匣中的 OpenVPN 圖示，然後按一下 [連線]。

## <a name="mac"></a>Mac 用戶端

1. 下載並安裝 OpenVPN 用戶端，例如 [TunnelBlik](https://tunnelblick.net/downloads.html) \(英文\)。 
2. 下載閘道的 VPN 設定檔。 這可從 Azure 入口網站中的 [點對站設定] 索引標籤，或使用 PowerShell 中的 'New-AzureRmVpnClientConfiguration' 來完成。
3. 將設定檔解壓縮。 在記事本中開啟 OpenVPN 資料夾中的 vpnconfig.ovpn 設定檔。
4. 以 Base64 的 P2S 用戶端憑證公開金鑰填入 P2S 用戶端憑證區段。 在 PEM 格式的憑證中，您只需開啟 .cer 檔案並在憑證標題之間複製 Base64 金鑰。 如需如何匯出憑證以取得編碼的公開金鑰的詳細資訊，請參閱[匯出公開金鑰](vpn-gateway-certificates-point-to-site.md#cer)。
5. 以 Base64 的 P2S 用戶端憑證私密金鑰填入私密金鑰區段。 如需如何擷取私用金鑰的資訊，請參閱[匯出私用金鑰](https://www.geotrust.eu/en/support/manuals/microsoft/all+windows+servers/export+private+key+or+certificate/)。
6. 請勿變更任何其他欄位。 使用用戶端輸入中填入的設定來連線至 VPN。
7. 按兩下設定檔檔案以在 Tunnelblik 中建立設定檔。
8. 從應用程式資料夾啟動 Tunnelblik。
9. 按一下系統匣中的 Tunneblik 圖示並挑選連線。

## <a name="linux"></a>Linux 用戶端

1. 開啟新的終端機工作階段。 同時按下 'Ctrl + Alt + t' 即可開啟新的工作階段
2. 輸入下列命令以安裝所需的元件：

  ```
  sudo apt-get install openvpn
  sudo apt-get -y install network-manager-openvpn
  sudo service network-manager restart
  ```
3. 下載閘道的 VPN 設定檔。 這可從 Azure 入口網站中的 [點對站設定] 索引標籤，或使用 PowerShell 中的 'New-AzureRmVpnClientConfiguration' 來完成。
4. 以 Base64 的 P2S 用戶端憑證私密金鑰填入私密金鑰區段。 如需如何擷取私用金鑰的資訊，請參閱[匯出私用金鑰](https://www.geotrust.eu/en/support/manuals/microsoft/all+windows+servers/export+private+key+or+certificate/)。
5. 若要使用命令列建立連線，請輸入下列命令：
  
  ```
  Sudo openvpn –config <name and path of your VPN profile file>
  ```
5. 若要使用 GUI 建立連線，請移至系統設定。
6. 按一下 **+** 以新增新的 VPN 連線。
7. 在 [新增 VPN]，點選 [匯入檔案]
8. 瀏覽至設定檔並按兩下或點選 [開啟]
9. 按一下 [新增 VPN] 視窗上的 [新增]。
  
  ![匯入檔案](./media/vpn-gateway-howto-openvpn-clients/importfromfile.png)
10. 在 [網路設定] 頁面上或在系統匣中網路圖示下將 VPN 切換到 [開] 即可建立連線。

## <a name="next-steps"></a>後續步驟

如果您想要 VPN 用戶端能夠存取另一個 vnet (生產) 中的資源，那麼請按照 [Vnet 對 VNet](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) 文章的指示操作，以設定 vnet 對 vnet 連線。 請務必啟用閘道上的 BGP 和連線，否則流量無法流動。
