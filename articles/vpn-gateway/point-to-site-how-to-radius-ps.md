---
title: 使用點對站和 RADIUS 驗證將電腦連線至虛擬網路︰PowerShell | Azure
description: 使用 P2S 和 RADIUS 驗證，將 Windows 和 Mac OS X 用戶端安全地連線到虛擬網路。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/12/2018
ms.author: anzaman
ms.openlocfilehash: df7afe9324831ffb8e79d7320f2c716ed18a7b4f
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38719680"
---
# <a name="configure-a-point-to-site-connection-to-a-vnet-using-radius-authentication-powershell"></a>使用 RADIUS 驗證設定 VNet 的點對站連線：PowerShell

本文顯示如何建立具有點對站連線 (使用 RADIUS 驗證) 的 VNet。 此組態只適用於 Resource Manager 部署模型。

點對站 (P2S) VPN 閘道可讓您建立從個別用戶端電腦到您的虛擬網路的安全連線。 當您想要從遠端位置連線到您的 VNet 時 (例如，當您從住家或會議進行遠距工作時)，點對站 VPN 連線很實用。 當您只有少數用戶端必須連線至 VNet 時，P2S VPN 也是很實用的方案 (而不是使用站對站 VPN)。

P2S VPN 連線會從 Windows 和 Mac 裝置啟動。 您可使用下列驗證方法來與用戶端連線：

* RADIUS 伺服器
* VPN 閘道原生憑證驗證

本文協助您設定具有使用 RADIUS 伺服器進行驗證的 P2S 組態。 如果您想要改為使用所產生的憑證和 VPN 閘道原生憑證驗證進行驗證，請參閱[使用 VPN 閘道原生憑證驗證來設定 VNet 的點對站連線](vpn-gateway-howto-point-to-site-rm-ps.md)。

![連線圖表 - RADIUS](./media/point-to-site-how-to-radius-ps/p2sradius.png)

點對站連線不需要 VPN 裝置或公眾對應 IP 位址。 P2S 會建立透過 SSTP (安全通訊端通道通訊協定) 或 IKEv2 的 VPN 連線。

* SSTP 是僅在 Windows 用戶端平台上支援的 SSL 型 VPN 通道。 它可以穿透防火牆，這是從任何地方連線到 Azure 的理想選項。 我們在伺服器端上支援 SSTP 1.0、1.1 和 1.2 版。 用戶端會決定要使用的版本。 若為 Windows 8.1 和更新版本，SSTP 預設使用 1.2。

* IKEv2 VPN，標準型 IPsec VPN 解決方案。 IKEv2 VPN 可用於從 Mac 裝置連線 (OSX 版本 10.11 和更新版本)。

P2S 連線需要下列各個條件：

* RouteBased VPN 閘道。 
* 用於處理使用者驗證的 RADIUS 伺服器。 RADIUS 伺服器可以部署在內部部署環境或 Azure VNet 中。
* 將連線至 VNet 之 Windows 裝置的 VPN 用戶端組態套件。 VPN 用戶端組態套件提供 VPN 用戶端透過 P2S 連線所需的設定。

## <a name="aboutad"></a>關於適用於 P2S VPN 的 Active Directory (AD) 網域驗證

AD 網域驗證可讓使用者使用其組織網域認證來登入 Azure。 它需要可與 AD 伺服器整合的 RADIUS 伺服器。 組織也可利用其現有的 RADIUS 部署。
 
RADIUS 伺服器可位於內部部署環境或 Azure VNet 中。 在驗證期間，VPN 閘道可作為 RADIUS 伺服器與連線裝置之間的通道，在兩者間來回轉送驗證訊息。 務必讓 VPN 閘道能夠連線到 RADIUS 伺服器。 如果 RADIUS 伺服器位於內部部署環境，則需要從 Azure 到內部部署網站的 VPN 站對站連線。

除了 Active Directory，RADIUS 伺服器也可以與其他外部身分識別系統整合。 這會開啟許多點對站 VPN 的驗證選項，包括 MFA 選項。 查閱 RADIUS 伺服器廠商文件，以取得與其整合的身分識別系統清單。

![連線圖表 - RADIUS](./media/point-to-site-how-to-radius-ps/radiusimage.png)

> [!IMPORTANT]
>VPN 站對站連線可用來連線到內部部署環境中的 RADIUS 伺服器。 無法使用 ExpressRoute 連線。
>
>

## <a name="before"></a>開始之前

* 請確認您有 Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details)或註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial)。

* 安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

### <a name="log-in"></a>登入

[!INCLUDE [Log in](../../includes/vpn-gateway-ps-login-include.md)]

### <a name="example"></a>範例值

您可以使用範例值來建立測試環境，或參考這些值來進一步了解本文中的範例。 您可以使用這些步驟做為逐步解說並使用未經變更的值，或變更這些值以反映您的環境。

* **名稱：VNet1**
* **位址空間：192.168.0.0/16** 和 **10.254.0.0/16**<br>在此範例中，我們使用一個以上的位址空間來說明此組態可以與多個位址空間搭配使用。 不過，此組態不需要多個位址空間。
* **子網路名稱：FrontEnd**
  * **子網路位址範圍：192.168.1.0/24**
* **子網路名稱：BackEnd**
  * **子網路位址範圍：10.254.1.0/24**
* **子網路名稱：GatewaySubnet**<br>子網路名稱 *GatewaySubnet* 是 VPN 閘道能夠運作的必要項目。
  * **閘道子網路位址範圍：192.168.200.0/24** 
* **VPN 用戶端位址集區：172.16.201.0/24**<br>使用這個點對站連線來連線到 VNet 的 VPN 用戶端，會收到來自 VPN 用戶端位址集區的 IP 位址。
* **訂用帳戶：** 如果您有一個以上的訂用帳戶，請確認您使用正確的訂用帳戶。
* **資源群組：TestRG**
* **位置：美國東部**
* **DNS 伺服器：** 您想要用於 VNet 名稱解析的 DNS 伺服器的 IP 位址。 (選用)
* **GW 名稱：Vnet1GW**
* **公用 IP 名稱：VNet1GWPIP**
* **VpnType：RouteBased** 

## 1.<a name="vnet"></a>建立資源群組、VNet 和公用 IP 位址

下列步驟會建立一個資源群組，並在此資源群組中建立具有三個子網路的虛擬網路。 替代值時，務必一律將您的閘道子網路特定命名為 GatewaySubnet。 如果您將其命名為其他名稱，閘道建立會失敗。

1. 建立資源群組。

  ```powershell
  New-AzureRmResourceGroup -Name "TestRG" -Location "East US"
  ```
2. 為虛擬網路建立子網路組態，將其命名為 FrontEndBackEnd 和 GatewaySubnet。 這些前置詞必須是您宣告的 VNet 位址空間的一部分。

  ```powershell
  $fesub = New-AzureRmVirtualNetworkSubnetConfig -Name "FrontEnd" -AddressPrefix "192.168.1.0/24"  
  $besub = New-AzureRmVirtualNetworkSubnetConfig -Name "Backend" -AddressPrefix "10.254.1.0/24"  
  $gwsub = New-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "192.168.200.0/24"
  ```
3. 建立虛擬網路

  在此範例中，-DnsServer 伺服器是選擇性的。 指定一個值並不會建立新的 DNS 伺服器。 您指定的 DNS 伺服器 IP 位址應該是可以解析您從 VNet 連線之資源名稱的 DNS 伺服器。 在此範例中，我們使用了私人 IP 位址，但這可能不是您 DNS 伺服器的 IP 位址。 請務必使用您自己的值。 您指定的值是由您部署至 VNet 的資源使用，不是由 P2S 連線使用。

  ```powershell
  New-AzureRmVirtualNetwork -Name "VNet1" -ResourceGroupName "TestRG" -Location "East US" -AddressPrefix "192.168.0.0/16","10.254.0.0/16" -Subnet $fesub, $besub, $gwsub -DnsServer 10.2.1.3
  ```
4. VPN 閘道必須具有公用 IP 位址。 您會先要求 IP 位址資源，然後在建立虛擬網路閘道時參考它。 建立 VPN 閘道時，系統會將 IP 位址動態指派給此資源。 VPN 閘道目前僅支援*動態*公用 IP 位址配置。 您無法要求靜態公用 IP 位址指派。 不過，這不表示 IP 位址變更之後已被指派至您的 VPN 閘道。 公用 IP 位址只會在刪除或重新建立閘道時變更。 它不會因為重新調整、重設或 VPN 閘道的其他內部維護/升級而變更。

  指定可要求動態指派之公用 IP 位址的變數。

  ```powershell
  $vnet = Get-AzureRmVirtualNetwork -Name "VNet1" -ResourceGroupName "TestRG"  
  $subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet 
  $pip = New-AzureRmPublicIpAddress -Name "VNet1GWPIP" -ResourceGroupName "TestRG" -Location "East US" -AllocationMethod Dynamic 
  $ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name "gwipconf" -Subnet $subnet -PublicIpAddress $pip
  ```

## 2.<a name="radius"></a>設定 RADIUS 伺服器

在建立和設定虛擬網路閘道之前，應正確設定 RADIUS 伺服器以供驗證。

1. 如果您沒有已部署的 RADIUS 伺服器，請部署一部。 如需部署步驟，請參閱 RADIUS 廠商所提供的設定指南。  
2. 將 VPN 閘道設定為 RADIUS 上的 RADIUS 用戶端。 新增此 RADIUS 用戶端時，指定您所建立的虛擬網路 GatewaySubnet。 
3. 設定 RADIUS 伺服器後，請取得 RADIUS 伺服器的 IP 位址和 RADIUS 用戶端應用來與 RADIUS 伺服器通訊的共用祕密。 如果 RADIUS 伺服器位於 Azure VNet 中，請使用 RADIUS 伺服器 VM 的 CA IP。

[網路原則伺服器 (NPS)](https://docs.microsoft.com/windows-server/networking/technologies/nps/nps-top) 一文提供有關設定 Windows RADIUS 伺服器 (NPS) 以便進行 AD 網域驗證的指引。

## 3.<a name="creategw"></a>建立 VPN 閘道

設定和建立 VNet 的 VPN 閘道。

* -GatewayType 必須是 'Vpn'，而 -VpnType 必須是 'RouteBased'。
* 視您選取的 [閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)  而定，VPN 閘道可能需要 45 分鐘的時間才能完成。

```powershell
New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
-Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
-VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1
```

## 4.<a name="addradius"></a>新增 RADIUS 伺服器和用戶端位址集區
 
* 可以依名稱或依 IP 位址指定 -RadiusServer。 如果您指定名稱且伺服器位於內部部署環境，則 VPN 閘道可能無法解析此名稱。 如果是這樣，最好是指定伺服器的 IP 位址。 
* -RadiusSecret 應符合 RADIUS 伺服器的設定。
* -VpnCientAddressPool 是連線 VPN 用戶端時從中接收 IP 位址的範圍。 使用不會重疊的私人 IP 位址範圍搭配您將從其連線的內部部署位置，或搭配您要連線至的 VNet。 確定您已設定夠大的位址集區。  

1. 為 Radius 祕密建立安全字串。

  ```powershell
  $Secure_Secret=Read-Host -AsSecureString -Prompt "RadiusSecret"
  ```

2. 系統會提示您輸入 RADIUS 祕密。 您輸入的字元並不會顯示，將由 "*" 字元所取代。

  ```powershell
  RadiusSecret:***
  ```
3. 新增 VPN 用戶端位址集區和 RADIUS 伺服器資訊。

  對於 SSTP 組態：

    ```powershell
    $Gateway = Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "SSTP" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

  對於 IKEv2 組態：

    ```powershell
    $Gateway = Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "IKEv2" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

  對於 SSTP + IKEv2

    ```powershell
    $Gateway = Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol @( "SSTP", "IkeV2" ) `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

## 5.<a name="vpnclient"></a>下載 VPN 用戶端組態套件並設定 VPN 用戶端

VPN 用戶端組態可讓裝置透過 P2S 連線來連線至 VNet。 若要產生 VPN 用戶端組態套件及設定 VPN 用戶端，請參閱[建立 VPN 用戶端組態以便進行 RADIUS 驗證](point-to-site-vpn-client-configuration-radius.md)。

## <a name="connect"></a>6.連接到 Azure

### <a name="to-connect-from-a-windows-vpn-client"></a>從 Windows VPN 用戶端連線

1. 若要連接至您的 VNet，在用戶端電腦上瀏覽到 VPN 連線，然後找出所建立的 VPN 連線。 其名稱會與虛擬網路相同。 輸入網域認證並按一下 [連線]。 隨即出現要求提高權限的快顯訊息。 接受它並然後輸入認證。

  ![VPN 用戶端連線至 Azure](./media/point-to-site-how-to-radius-ps/client.png)
2. 已建立您的連線。

  ![連線已建立](./media/point-to-site-how-to-radius-ps/connected.png)

### <a name="connect-from-a-mac-vpn-client"></a>從 Mac VPN 用戶端連線

從 [網路] 對話方塊，找出您要使用的用戶端設定檔，然後按一下 [連線]。

  ![Mac 連線](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png)

## <a name="verify"></a>驗證您的連線

1. 若要驗證您的 VPN 連線為作用中狀態，請開啟提升權限的命令提示字元，並執行 *ipconfig/all*。
2. 檢視結果。 請注意，您接收到的 IP 位址是您在組態中指定的點對站 VPN 用戶端位址集區中的其中一個位址。 結果類似於此範例：

  ```
  PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.3(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
  ```

如要針對 P2S 進行疑難排解，請參閱[針對 Azure 點對站連線進行疑難排解](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md)。

## <a name="connectVM"></a>連線至虛擬機器

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-p2s-include.md)]

## <a name="faq"></a>常見問題集

此常見問題集適用於使用 RADIUS 驗證的 P2S

[!INCLUDE [Point-to-Site RADIUS FAQ](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="next-steps"></a>後續步驟

一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 如需詳細資訊，請參閱[虛擬機器](https://docs.microsoft.com/azure/#pivot=services&panel=Compute)。 若要了解網路與虛擬機器的詳細資訊，請參閱 [Azure 與 Linux VM 網路概觀](../virtual-machines/linux/azure-vm-network-overview.md)。
