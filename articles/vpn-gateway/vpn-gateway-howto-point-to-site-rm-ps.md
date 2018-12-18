---
title: 使用點對站和原生 Azure 憑證驗證將電腦連線至 Azure 虛擬網路︰PowerShell | Microsoft Docs
description: 使用 P2S 和自我簽署或 CA 核發的憑證，將 Windows 和 Mac OS X 用戶端安全地連線至 Azure 虛擬網路。 本文使用 PowerShell。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.assetid: 3eddadf6-2e96-48c4-87c6-52a146faeec6
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/12/2018
ms.author: cherylmc
ms.openlocfilehash: 42afdee5ac58db005a7ecfb6388c88a974704a03
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38295725"
---
# <a name="configure-a-point-to-site-connection-to-a-vnet-using-native-azure-certificate-authentication-powershell"></a>使用原生 Azure 憑證驗證設定 VNet 的點對站連線：PowerShell

本文可協助您將執行 Windows 或 Mac OS X 的個別用戶端安全地連線至 Azure VNet。 當您想要從遠端位置 (例如當您從住家或會議進行遠距工作) 連線到您的 VNet 時，點對站 VPN 連線很實用。 如果您只有少數用戶端必須連線至 VNet，您也可以使用 P2S，而不使用站對站 VPN。 點對站連線不需要 VPN 裝置或公眾對應 IP 位址。 P2S 會建立透過 SSTP (安全通訊端通道通訊協定) 或 IKEv2 的 VPN 連線。 如需點對站 VPN 的詳細資訊，請參閱[關於點對站 VPN](point-to-site-about.md)。

![將電腦連接至 Azure VNet - 點對站連線圖表](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/p2snativeportal.png)


## <a name="architecture"></a>架構

點對站原生 Azure 憑證驗證連線須使用下列項目 (您可以在此練習中設定)：

* RouteBased VPN 閘道。
* 已上傳至 Azure 之根憑證的公開金鑰 (.cer 檔案)。 一旦上傳憑證，憑證就會被視為受信任的憑證並且用於驗證。
* 從根憑證產生的用戶端憑證。 此用戶端憑證須安裝在每部將連線至 VNet 的用戶端電腦上。 此憑證使用於用戶端憑證。
* VPN 用戶端組態。 VPN 用戶端組態檔包含要讓用戶端連線到 VNet 所需的資訊。 此檔案會設定作業系統原生的現有 VPN 用戶端。 您必須使用組態檔中的設定來設定每個進行連線的用戶端。

## <a name="before-you-begin"></a>開始之前

* 請確認您有 Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details)或註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial)。
* 安裝最新版的 Resource Manager PowerShell Cmdlet。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 這一點很重要，因為較早版本的 Cmdlet 未包含您在此練習中所需的現行值。

### <a name="example"></a>範例值

您可以使用範例值來建立測試環境，或參考這些值來進一步了解本文中的範例。 本文的第 [1](#declare) 節會設定變數。 您可以使用這些步驟做為逐步解說並使用未經變更的值，或變更這些值以反映您的環境。

* **名稱：VNet1**
* **位址空間：192.168.0.0/16** 和 **10.254.0.0/16**<br>此範例使用一個以上的位址空間來說明此組態可以與多個位址空間搭配使用。 不過，此組態不需要多個位址空間。
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
* **DNS 伺服器：** 您想要用於名稱解析的 DNS 伺服器的 IP 位址。 (選用)
* **GW 名稱：Vnet1GW**
* **公用 IP 名稱：VNet1GWPIP**
* **VpnType：RouteBased** 

## <a name="declare"></a>1.登入和設定變數

在此區段中，您可以登入並宣告用於此組態的值。 在範例指令碼中，會使用宣告的值。 請變更相關值以反映自己的環境。 或者，可以使用宣告的值並執行各步驟做為練習。

1. 以提高的權限開啟 PowerShell 主控台並登入您的 Azure 帳戶。 此 Cmdlet 會提示您提供登入認證。 登入之後，它會下載您的帳戶設定以供 Azure PowerShell 使用。

  ```powershell
  Connect-AzureRmAccount
  ```
2. 取得您的 Azure 訂用帳戶清單。

  ```powershell
  Get-AzureRmSubscription
  ```
3. 指定您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Name of subscription"
  ```
4. 宣告您想要使用的變數。 使用以下範例，在需要時將該值替換為您自己的值。

  ```powershell
  $VNetName  = "VNet1"
  $FESubName = "FrontEnd"
  $BESubName = "Backend"
  $GWSubName = "GatewaySubnet"
  $VNetPrefix1 = "192.168.0.0/16"
  $VNetPrefix2 = "10.254.0.0/16"
  $FESubPrefix = "192.168.1.0/24"
  $BESubPrefix = "10.254.1.0/24"
  $GWSubPrefix = "192.168.200.0/26"
  $VPNClientAddressPool = "172.16.201.0/24"
  $RG = "TestRG"
  $Location = "East US"
  $GWName = "VNet1GW"
  $GWIPName = "VNet1GWPIP"
  $GWIPconfName = "gwipconf"
  ```

## <a name="ConfigureVNet"></a>2.設定 VNet

1. 建立資源群組。

  ```powershell
  New-AzureRmResourceGroup -Name $RG -Location $Location
  ```
2. 為虛擬網路建立子網路組態，將其命名為 FrontEndBackEnd 和 GatewaySubnet。 這些前置詞必須是您宣告的 VNet 位址空間的一部分。

  ```powershell
  $fesub = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
  $besub = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName -AddressPrefix $BESubPrefix
  $gwsub = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix
  ```
3. 建立虛擬網路

  在此範例中，-DnsServer 伺服器是選擇性的。 指定一個值並不會建立新的 DNS 伺服器。 您指定的 DNS 伺服器 IP 位址應該是可以解析您從 VNet 連線之資源名稱的 DNS 伺服器。 此範例使用私人 IP 位址，但這可能不是您 DNS 伺服器的 IP 位址。 請務必使用您自己的值。 您指定的值會供部署至 VNet 的資源使用，而非供 P2S 連線或 VPN 用戶端使用。

  ```powershell
  New-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG -Location $Location -AddressPrefix $VNetPrefix1,$VNetPrefix2 -Subnet $fesub, $besub, $gwsub -DnsServer 10.2.1.3
  ```
4. 指定您所建立的虛擬網路的變數。

  ```powershell
  $vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG
  $subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
  ```
5. VPN 閘道必須具有公用 IP 位址。 您會先要求 IP 位址資源，然後在建立虛擬網路閘道時參考它。 建立 VPN 閘道時，系統會將 IP 位址動態指派給此資源。 VPN 閘道目前僅支援*動態*公用 IP 位址配置。 您無法要求靜態公用 IP 位址指派。 不過，這不表示之後的 IP 位址變更已指派至您的 VPN 閘道。 公用 IP 位址只會在刪除或重新建立閘道時變更。 它不會因為重新調整、重設或 VPN 閘道的其他內部維護/升級而變更。

  要求動態指派的公用 IP 位址。

  ```powershell
  $pip = New-AzureRmPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
  $ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
  ```

## <a name="creategateway"></a>3.建立 VPN 閘道

設定和建立 VNet 的虛擬網路閘道。

* -GatewayType 必須是 **Vpn**，而 -VpnType 必須是 **RouteBased**。
* 使用 -VpnClientProtocol 可指定您想要啟用的通道類型。 通道有 **SSTP** 和 **IKEv2** 這兩個選項。 您可以選擇啟用其中一個或兩個都啟用。 如果您想要將兩個選項都啟用，請同時指定這兩個名稱並以逗號隔開。 Android 和 Linux 上的 strongSwan 用戶端以及 iOS 和 OSX 上的原生 IKEv2 VPN 用戶端只會使用 IKEv2 通道來進行連線。 Windows 用戶端會先嘗試 IKEv2，如果無法連線，就會切換回使用 SSTP。
* 視您選取的[閘道 sku](vpn-gateway-about-vpn-gateway-settings.md) 而定，VPN 閘道可能需要 45 分鐘的時間才能完成。 此範例使用 IKEv2。

```powershell
New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
-Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
-VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1 -VpnClientProtocol "IKEv2"
```

## <a name="addresspool"></a>4.新增 VPN 用戶端位址集區

VPN 閘道完成建立之後，您可以新增 VPN 用戶端位址集區。 VPN 用戶端位址集區是 VPN 用戶端在連線時將從其接收 IP 位址的範圍。 使用不會重疊的私人 IP 位址範圍搭配您從其連線的內部部署位置，或搭配您要連線至的 VNet。 在此範例中，VPN 用戶端位址集區會宣告為步驟 1 中的[變數](#declare)。

```powershell
$Gateway = Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
Set-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientAddressPool $VPNClientAddressPool
```

## <a name="Certificates"></a>5.產生憑證

憑證是 Azure 用於點對站 VPN 的 VPN 用戶端驗證。 您會將根憑證的公開金鑰資訊上傳至 Azure。 公開金鑰就會被視為「受信任」。 用戶端憑證必須從信任的根憑證產生，然後安裝在 [憑證-目前使用者/個人憑證] 存放區中的每部用戶端電腦上。 在用戶端初始 VNet 連線時，此憑證用來驗證用戶端。 

如果您使用自我簽署憑證，則必須使用特定參數來建立這些憑證。 您可以依循 [PowerShell 和 Windows 10](vpn-gateway-certificates-point-to-site.md) 的指示建立自我簽署憑證；或者，如果您沒有 Windows 10，可以使用 [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md)。 產生自我簽署的根憑證和用戶端憑證時，請務必遵循指示中的步驟。 否則，您所產生的憑證將無法與 P2S 連線相容，而且會收到連線錯誤的訊息。

### <a name="cer"></a>1.取得根憑證的 .cer 檔案

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-rootcert-include.md)]


### <a name="generate"></a>2.產生用戶端憑證 

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-clientcert-include.md)]

## <a name="upload"></a>6.上傳根憑證公開金鑰資訊

確認 VPN 閘道已完成建立。 完成之後，您可以將受信任根憑證的 .cer 檔案 (其中包含公開金鑰資訊) 上傳至 Azure。 一旦上傳 .cer 檔案，Azure 就可以使用它來驗證已安裝從受信任根憑證產生之用戶端憑證的用戶端。 如有需要，您稍後可以上傳其他受信任的根憑證檔案 (最多總計 20 個檔案)。

1. 宣告您的憑證名稱的變數，以自己的值取代。

  ```powershell
  $P2SRootCertName = "P2SRootCert.cer"
  ```
2. 將檔案路徑以您的路徑取代，然後執行 Cmdlet。

  ```powershell
  $filePathForCert = "C:\cert\P2SRootCert.cer"
  $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
  $CertBase64 = [system.convert]::ToBase64String($cert.RawData)
  $p2srootcert = New-AzureRmVpnClientRootCertificate -Name $P2SRootCertName -PublicCertData $CertBase64
  ```
3. 將公開金鑰資訊上傳至 Azure。 一旦上傳憑證資訊，Azure 會認為這是受信任的根憑證。

  ```powershell
  Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG" -PublicCertData $CertBase64
  ```

## <a name="clientcertificate"></a>7.安裝匯出的用戶端憑證

如果您想要從不同於用來產生用戶端憑證的用戶端電腦建立 P2S 連線，您需要安裝用戶端憑證。 安裝用戶端憑證時，您需要匯出用戶端憑證時所建立的密碼。

請確定用戶端憑證已隨著整個憑證鏈結匯出為 .pfx (這是預設值)。 否則，根憑證資訊不存在於用戶端電腦上，而且用戶端將無法正確驗證。 

如需安裝步驟，請參閱[安裝用戶端憑證](point-to-site-how-to-vpn-client-install-azure-cert.md)。

## <a name="clientconfig"></a>8.設定原生 VPN 用戶端

VPN 用戶端組態檔所包含的設定，可用來將裝置設定為透過 P2S 連線來連線至 VNet。 如需產生和安裝 VPN 用戶端組態檔的指示，請參閱[建立和安裝適用於原生 Azure 憑證驗證 P2S 組態的 VPN 用戶端組態檔](point-to-site-vpn-client-configuration-azure-cert.md)。

## <a name="connect"></a>9.連接到 Azure

### <a name="to-connect-from-a-windows-vpn-client"></a>從 Windows VPN 用戶端連線

>[!NOTE]
>您必須在您所連線的 Windows 用戶端電腦上具有系統管理員權限。
>
>

1. 若要連接至您的 VNet，在用戶端電腦上瀏覽到 VPN 連線，然後找出所建立的 VPN 連線。 其名稱會與虛擬網路相同。 按一下 [ **連接**]。 可能會出現與使用憑證有關的快顯訊息。 按一下 [繼續] 以使用較高的權限。 
2. 在 [連線] 狀態頁面上，按一下 [連線] 以便開始連線。 如果出現 [選取憑證]  畫面，請確認顯示的用戶端憑證是要用來連接的憑證。 如果沒有，請使用下拉箭頭來選取正確的憑證，然後按一下 [確定] 。

  ![VPN 用戶端連線至 Azure](./media/vpn-gateway-howto-point-to-site-rm-ps/clientconnect.png)
3. 已建立您的連線。

  ![連線已建立](./media/vpn-gateway-howto-point-to-site-rm-ps/connected.png)

#### <a name="troubleshooting-windows-client-p2s-connections"></a>針對 Windows 用戶端 P2S 連線進行疑難排解

[!INCLUDE [client certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

### <a name="to-connect-from-a-mac-vpn-client"></a>從 Mac VPN 用戶端連線

從 [網路] 對話方塊，找出您要使用的用戶端設定檔，然後按一下 [連線]。

  ![Mac 連線](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png)

## <a name="verify"></a>驗證您的連線

這些指示適用於 Windows 用戶端。

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

## <a name="connectVM"></a>連線至虛擬機器

這些指示適用於 Windows 用戶端。

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-p2s-include.md)]

## <a name="addremovecert"></a>新增或移除根憑證

您可以從 Azure 新增和移除受信任的根憑證。 當您移除根憑證時，從根憑證產生憑證的用戶端無法進行驗證，而且無法進行連線。 若希望用戶端進行驗證和連線，您需要安裝從 Azure 信任 (已上傳至 Azure) 的根憑證產生的新用戶端憑證。

### <a name="addtrustedroot"></a>新增受信任的根憑證

您最多可將 20 個根憑證 .cer 檔案新增至 Azure。 下列步驟可協助您新增根憑證︰

#### <a name="certmethod1"></a>方法 1

這是上傳根憑證的最有效方法。

1. 準備要上傳的 .cer 檔案：

  ```powershell
  $filePathForCert = "C:\cert\P2SRootCert3.cer"
  $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
  $CertBase64_3 = [system.convert]::ToBase64String($cert.RawData)
  $p2srootcert = New-AzureRmVpnClientRootCertificate -Name $P2SRootCertName -PublicCertData $CertBase64_3
  ```
2. 上傳檔案。 您一次只能上傳一個檔案。

  ```powershell
  Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG" -PublicCertData $CertBase64_3
  ```

3. 若要確認憑證檔案已上傳：

  ```powershell
  Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
  -VirtualNetworkGatewayName "VNet1GW"
  ```

#### <a name="certmethod2"></a>方法 2

這個方法的步驟比方法 1 多，但結果相同。 納入此方法，以免您需要檢視憑證資料。

1. 建立並準備要新增至 Azure 的新根憑證。 將公開金鑰匯出成 Base-64 編碼 X.509 (.CER) 並使用文字編輯器開啟。 如下列範例所示，複製相關值：

  ![憑證](./media/vpn-gateway-howto-point-to-site-rm-ps/copycert.png)

  > [!NOTE]
  > 複製憑證資料時，請確定您是以連續一行的形式複製文字，而不含歸位字元或換行字元。 您可能必須將文字編輯器中的檢視修改成 [顯示符號] 或 [顯示所有字元]，才能看到歸位字元和換行字元。
  >
  >

2. 指定憑證名稱和金鑰資訊做為變數。 如下列範例所示，以您自己的資訊加以取代：

  ```powershell
  $P2SRootCertName2 = "ARMP2SRootCert2.cer"
  $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
  ```
3. 加入新的根憑證。 您一次只能新增一個憑證。

  ```powershell
  Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG" -PublicCertData $MyP2SCertPubKeyBase64_2
  ```
4. 您可使用下列範例確認新憑證已正確新增：

  ```powershell
  Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
  -VirtualNetworkGatewayName "VNet1GW"
  ```

### <a name="removerootcert"></a>移除根憑證

1. 宣告變數。

  ```powershell
  $GWName = "Name_of_virtual_network_gateway"
  $RG = "Name_of_resource_group"
  $P2SRootCertName2 = "ARMP2SRootCert2.cer"
  $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
  ```
2. 移除憑證。

  ```powershell
  Remove-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -PublicCertData $MyP2SCertPubKeyBase64_2
  ```
3. 使用下列範例確認已成功移除憑證。

  ```powershell
  Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
  -VirtualNetworkGatewayName "VNet1GW"
  ```

## <a name="revoke"></a>撤銷用戶端憑證

您可以撤銷用戶端憑證。 憑證撤銷清單可讓您選擇性地拒絕以個別的用戶端憑證為基礎的點對站連線。 這與移除受信任的根憑證不同。 若您從 Azure 移除受信任的根憑證 .cer，就會撤銷所有由撤銷的根憑證所產生/簽署的用戶端憑證之存取權。 撤銷用戶端憑證，而不是根憑證，可以繼續使用從根憑證產生的憑證進行驗證。

常見的做法是使用根憑證管理小組或組織層級的存取權，然後使用撤銷的用戶端憑證針對個別使用者進行細部的存取控制。

### <a name="revokeclientcert"></a>撤銷用戶端憑證

1. 擷取用戶端憑證指紋。 如需詳細資訊，請參閱[做法：擷取憑證的指紋](https://msdn.microsoft.com/library/ms734695.aspx)。
2. 將資訊複製到文字編輯器，並移除所有的空格，讓它是連續字串。 這個字串在下一個步驟中會宣告為變數。
3. 宣告變數。 請確定宣告您在上一個步驟中擷取的指紋。

  ```powershell
  $RevokedClientCert1 = "NameofCertificate"
  $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
  $GWName = "Name_of_virtual_network_gateway"
  $RG = "Name_of_resource_group"
  ```
4. 將指紋新增到撤銷的憑證清單。 指紋新增時，您會看到「成功」。

  ```powershell
  Add-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
  -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG `
  -Thumbprint $RevokedThumbprint1
  ```
5. 確認指紋已新增到憑證撤銷清單。

  ```powershell
  Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
  ```
6. 新增指紋之後，憑證無法再用於連線接。 嘗試使用此憑證進行連線的用戶端會收到訊息，指出憑證不再有效。

### <a name="reinstateclientcert"></a>恢復用戶端憑證

您可以從撤銷的用戶端憑證清單中移除指紋來恢復用戶端憑證。

1. 宣告變數。 請確定針對您要恢復的憑證宣告正確指紋。

  ```powershell
  $RevokedClientCert1 = "NameofCertificate"
  $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
  $GWName = "Name_of_virtual_network_gateway"
  $RG = "Name_of_resource_group"
  ```
2. 從憑證撤銷清單中移除憑證指紋。

  ```powershell
  Remove-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
  -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1
  ```
3. 檢查指紋是否已從撤銷的清單中移除。

  ```powershell
  Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
  ```

## <a name="faq"></a>點對站常見問題集

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="next-steps"></a>後續步驟
一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 如需詳細資訊，請參閱[虛擬機器](https://docs.microsoft.com/azure/#pivot=services&panel=Compute)。 若要了解網路與虛擬機器的詳細資訊，請參閱 [Azure 與 Linux VM 網路概觀](../virtual-machines/linux/azure-vm-network-overview.md)。

如需 P2S 疑難排解詳細資訊，請參閱[疑難排解：Azure 點對站連線問題](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md)。
