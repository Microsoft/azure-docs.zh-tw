---
title: 將傳統虛擬網路連接到 Azure Resource Manager VNet：入口網站 | Microsoft Docs
description: 使用 VPN 閘道和入口網站將傳統 VNet 連線到 Resource Manager VNet 的步驟
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: 5a90498c-4520-4bd3-a833-ad85924ecaf9
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/13/2018
ms.author: cherylmc
ms.openlocfilehash: 6a585155e1d1050498754f5b9a7ec3dfc82d7a2c
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319926"
---
# <a name="connect-virtual-networks-from-different-deployment-models-using-the-portal"></a>使用入口網站從不同的部署模型連接虛擬網路

本文說明如何將傳統 VNet 連線到 Resource Manager VNet，以允許位於不同部署模型中的資源彼此通訊。 本文中的步驟主要使用 Azure 入口網站，但您也可以選取這份清單中的文章，進而使用 PowerShell 建立此組態。

> [!div class="op_single_selector"]
> * [入口網站](vpn-gateway-connect-different-deployment-models-portal.md)
> * [PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
> 
> 

將傳統 VNet 連線至 Resource Manager VNet，類似於將 VNet 連線至內部部署網站位置。 這兩種連線類型都使用 VPN 閘道提供使用 IPsec/IKE 的安全通道。 您可以在不同訂用帳戶和不同區域中的 VNet 之間建立連線。 只要設定的閘道是動態或路由式，您也可以連接已連線到內部部署網路的 Vnet。 如需 VNet 對 VNet 連線的詳細資訊，請參閱本文結尾處的 [VNet 對 VNet 常見問題集](#faq) 。 

如果您還沒有虛擬網路閘道，而且不想建立閘道，可能要考慮改為使用 VNet 對等互連連線您的 VNet。 VNet 對等互連不會使用 VPN 閘道。 如需詳細資訊，請參閱 [VNet 對等互連](../virtual-network/virtual-network-peering-overview.md)。

### <a name="before"></a>開始之前

* 這些步驟假設已建立兩個 VNet。 如果您使用這篇文章作為練習且沒有 VNet，步驟中有連結可以協助您建立它們。
* 確認 VNet 的位址範圍不會彼此重疊，或與閘道可能連線的任何其他連線範圍重疊。
* 為 Resource Manager 和服務管理 (傳統) 安裝 PowerShell Cmdlet。 在本文中，我們會使用 Azure 入口網站和 PowerShell。 需要 PowerShell 才能建立從傳統 VNet 到 Resource Manager VNet 的連線。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。 

### <a name="values"></a>設定範例

您可以使用這些值來建立測試環境，或參考這些值，進一步了解本文中的範例。

**傳統 VNet**

VNet 名稱 = ClassicVNet  <br>
位址空間 = 10.0.0.0/24 <br>
子網路名稱 = Subnet-1 <br>
子網路位址範圍 = 10.0.0.0/27 <br>
訂用帳戶 = 您需要使用的訂用帳戶 <br>
資源群組 = ClassicRG <br>
位置 = West US <br>
GatewaySubnet = 10.0.0.32/28 <br>
本機站台 = RMVNetLocal <br>

**Resource Manager VNet**

VNet 名稱 = RMVNet  <br>
位址空間 = 192.168.0.0/16 <br>
資源群組 = RG1 <br>
位置 = 美國東部 <br>
子網路名稱 = Subnet-1 <br>
位址範圍 = 192.168.1.0/24 <br>
GatewaySubnet = 192.168.0.0/26 <br>
虛擬網路閘道名稱 = RMGateway <br>
閘道類型 = VPN <br>
VPN 類型 = 路由式 <br>
SKU = VpnGw1 <br>
位置 = 美國東部 <br>
虛擬網路 = RMVNet <br> (將 VPN 閘道關聯到此 VNet) 第一個 IP 組態 = rmgwpip <br> (閘道公用 IP 位址) 區域網路閘道 = ClassicVNetLocal <br>
連線名稱 = RMtoClassic

### <a name="connectoverview"></a>連線概觀

針對此組態，您透過虛擬網路之間的 IPsec/IKE VPN 通道，建立 VPN 閘道連線。 確定沒有任何 VNet 範圍彼此重疊，或是與其連線的區域網路重疊。

下表顯示範例，示範如何定義範例 VNet 和本機網站︰

| 虛擬網路 | 位址空間 | 區域 | 連接至區域網路站台 |
|:--- |:--- |:--- |:--- |
| ClassicVNet |(10.0.0.0/24) |美國西部 | RMVNetLocal (192.168.0.0/16) |
| RMVNet | (192.168.0.0/16) |美國東部 |ClassicVNetLocal (10.0.0.0/24) |

## <a name="classicvnet"></a>第 1 節 - 進行傳統 VNet 設定

在本節中，您會建立傳統 VNet、區域網路 (本機網站) 及虛擬網路閘道。 已提供螢幕擷取畫面做為範例。 請務必將值取代為您自己的值，或使用[範例](#values)值。

### 1.<a name="classicvnet"></a>建立傳統 VNet

如果您沒有傳統 VNet，而且正在執行這些步驟作為練習，您可以使用[這篇文章](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)和上述的[範例](#values)設定值來建立 VNet。

如果您已經有具備 VPN 閘道的 VNet，請確認閘道是動態的。 如果是靜態，您必須先刪除 VPN 閘道後，才能繼續進行[設定本機站台](#local)。

1. 開啟 [Azure 入口網站](https://ms.portal.azure.com)並使用您的 Azure 帳戶登入。
2. 按一下 [+ 建立資源] 可開啟 [新增] 頁面。
3. 在 [搜尋 Marketplace] 欄位中，輸入「虛擬網路」。 如果您改為選取 [網路功能]-> [虛擬網路]，將無法取得建立傳統 VNet 的選項。
4. 在傳回的清單中找出 [虛擬網路]，然後按一下以開啟 [虛擬網路] 頁面。 
5. 在 [虛擬網路] 頁面上，選取 [傳統] 以建立傳統的 VNet。 如果您在這裡採用預設值，就會改為得到 Resource Manager VNet。

### 2.<a name="local"></a>設定本機網站

1. 瀏覽至 [所有資源]，並且在清單中找出 **ClassicVNet**。
2. 在 [概觀] 頁面上的 [VPN 連線] 區段中，按一下 [閘道] 以建立閘道。
  ![設定 VPN 閘道](./media/vpn-gateway-connect-different-deployment-models-portal/gatewaygraphic.png "設定 VPN 閘道")
3. 在 [新增 VPN 連線] 頁面上，針對 [連線類型] 選取 [站對站]。
4. 針對 [本機站台]，按一下 [設定必要設定]。 這會開啟 [本機網站] 頁面。
5. 在 [本機網站] 頁面上，建立用來參照 Resource Manager VNet 的名稱。 例如，'RMVNetLocal'。
6. 如果 Resource Manager VNet 的 VPN 閘道已經有公用 IP 位址，請使用 [VPN 閘道 IP 位址] 欄位的值。 如果您要執行這些步驟作為練習，或者您的 Resource Manager VNet 還沒有虛擬網路閘道，您可以自行設定預留位置 IP 位址。 請確定預留位置 IP 位址使用有效的格式。 稍後，將預留位置 IP 位址以 Resource Manager 虛擬網路閘道的公用 IP 位址取代。
7. 針對 [用戶端位址空間]，使用 Resource Manager VNet 的虛擬網路 IP 位址空間的[值](#connectoverview)。 此設定是用來指定要路由傳送至 Resource Manager 虛擬網路的位址空間。 在範例中，我們使用 192.168.0.0/16，RMVNet 的位址範圍。
8. 按一下 [確定] 來儲存值，並返回 [新增 VPN 連線] 頁面。

### <a name="classicgw"></a>3.建立虛擬網路閘道

1. 在 [新增 VPN 連線] 頁面上，選取 [立即建立閘道] 核取方塊。
2. 按一下 [選擇性閘道組態] 可開啟 [閘道組態] 頁面。

  ![開啟 [閘道組態] 頁面](./media/vpn-gateway-connect-different-deployment-models-portal/optionalgatewayconfiguration.png "開啟 [閘道組態] 頁面")
3. 按一下 [子網路 - 設定必要設定] 以開啟 [新增子網路] 頁面。 [名稱] 已使用必要值 **GatewaySubnet** 進行設定。
4. [位址範圍] 指的是閘道子網路的範圍。 雖然您可以使用 /29 位址範圍 (3 個位址) 建立閘道子網路，但是我們建議您建立包含更多 IP 位址的閘道子網路。 這可以容納未來可能需要更多可用 IP 位址的組態。 可能的話，請使用 /27 或 /28。 如果您使用這些步驟作為練習，您可以參考[範例值](#values)。 在此範例中，我們使用 '10.0.0.32/28'。 按一下 [確定] 以建立閘道子網路。
5. 在 [閘道設定] 頁面上，[大小] 指的是閘道 SKU。 選取您 VPN 閘道的閘道 SKU。
6. 確認 [路由類型] 是 [動態]，然後按一下 [確定] 以返回 [新增 VPN 連線] 頁面。
7. 在 [新增 VPN 連線] 頁面上，按一下 [確定] 以開始建立您的 VPN 閘道。 建立 VPN 閘道可能需要 45 分鐘的時間才能完成。

### <a name="ip"></a>4.複製虛擬網路閘道公用 IP 位址

建立虛擬網路閘道之後，您可以檢視閘道 IP 位址。 

1. 導覽至您的傳統 VNet，然後按一下 [概觀]。
2. 按一下 [VPN 連線] 以開啟 [VPN 連線] 頁面。 在 [VPN 連線] 頁面上，您可以檢視公用 IP 位址。 這是指派給虛擬網路閘道的公用 IP 位址。 記下 IP 位址。 當您在稍後的步驟中處理 Resource Manager 區域網路閘道組態設定時，便會用到該 IP 位址。 
3. 您可以檢視閘道連線的狀態。 請注意，您所建立的區域網路網站會列為「連線中」。 在您建立連線之後，狀態會變更。 當您完成檢視狀態時，可以關閉此頁面。

## <a name="rmvnet"></a>第 2 節 - 進行 Resource Manager VNet 設定

在本節中，您會建立 Resource Manager VNet 的虛擬網路閘道和區域網路閘道。 已提供螢幕擷取畫面做為範例。 請務必將值取代為您自己的值，或使用[範例](#values)值。

### <a name="1-create-a-virtual-network"></a>1.建立虛擬網路

**範例值：**

* VNet 名稱 = RMVNet  <br>
* 位址空間 = 192.168.0.0/16 <br>
* 資源群組 = RG1 <br>
* 位置 = 美國東部 <br>
* 子網路名稱 = Subnet-1 <br>
* 位址範圍 = 192.168.1.0/24 <br>


如果您沒有 Resource Manager VNet，而且正在執行這些步驟作為練習，請使用範例值，搭配[建立虛擬網路](../virtual-network/quick-create-portal.md)中的步驟來建立虛擬網路。

### <a name="2-create-a-gateway-subnet"></a>2.建立閘道子網路

**範例值：** GatewaySubnet = 192.168.0.0/26

建立虛擬網路閘道之前，您必須先建立閘道子網路。 以 /28 或更高的 CIDR 計數建立閘道子網路 (/27、/26 等)。 如果您要建立此項作為練習的一部分，可以使用範例值。

[!INCLUDE [vpn-gateway-add-gwsubnet-rm-portal](../../includes/vpn-gateway-add-gwsubnet-rm-portal-include.md)]

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="creategw"></a>3.建立虛擬網路閘道

**範例值：**

* 虛擬網路閘道名稱 = RMGateway <br>
* 閘道類型 = VPN <br>
* VPN 類型 = 路由式 <br>
* SKU = VpnGw1 <br>
* 位置 = 美國東部 <br>
* 虛擬網路 = RMVNet <br>
* 第一個 IP 組態 = rmgwpip <br>

[!INCLUDE [vpn-gateway-add-gw-rm-portal](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

### <a name="createlng"></a>4.建立區域網路閘道

**範例值：** 區域網路閘道 = ClassicVNetLocal

| 虛擬網路 | 位址空間 | 區域 | 連接至區域網路站台 |閘道公用 IP 位址|
|:--- |:--- |:--- |:--- |:--- |
| ClassicVNet |(10.0.0.0/24) |美國西部 | RMVNetLocal (192.168.0.0/16) |指派給 ClassicVNet 閘道的公用 IP 位址|
| RMVNet | (192.168.0.0/16) |美國東部 |ClassicVNetLocal (10.0.0.0/24) |指派給 RMVNet 閘道的公用 IP 位址。|

區域網路閘道會指定與您的傳統 VNet 及其虛擬網路閘道相關聯之位址範圍和公用 IP 位址。 如果您執行這些步驟作為練習，請參考範例值。

[!INCLUDE [vpn-gateway-add-lng-rm-portal](../../includes/vpn-gateway-add-lng-rm-portal-include.md)]

## <a name="modifylng"></a>第 3 節 - 修改傳統 VNet 本機站台設定

在本節中，您會使用 Resource Manager VPN 閘道 IP 位址，取代在指定本機網站設定時所使用的預留位置 IP 位址。 本節會使用傳統 (SM) PowerShell Cmdlet。

1. 在 Azure 入口網站中，瀏覽至傳統虛擬網路。
2. 在您虛擬網路的頁面上，按一下 [概觀]。
3. 在 [VPN 連線] 區段中，按一下圖形中您本機站台的名稱。

  ![VPN-connections](./media/vpn-gateway-connect-different-deployment-models-portal/vpnconnections.png "VPN 連線")
4. 在 [站對站 VPN 連線] 頁面上，按一下站台的名稱。

  ![Site-name](./media/vpn-gateway-connect-different-deployment-models-portal/sitetosite3.png "本機站台名稱")
5. 在您本機站台的 [連線] 頁面上，按一下本機站台的名稱以開啟 [本機站台] 頁面。

  ![Open-local-site](./media/vpn-gateway-connect-different-deployment-models-portal/openlocal.png "開啟本機站台")
6. 在 [本機網站] 頁面上，使用 Resource Manager 閘道的 IP 位址取代 [VPN 閘道 IP 位址]。

  ![Gateway-ip-address](./media/vpn-gateway-connect-different-deployment-models-portal/gwipaddress.png "閘道 IP 位址")
7. 按一下 [確定] 以更新 IP 位址。

## <a name="RMtoclassic"></a>第 4 節 - 建立 Resource Manager 與傳統的連線

在這些步驟中，您會使用 Azure 入口網站設定從 Resource Manager VNet 到傳統 VNet 的連線。

1. 在 [所有資源] 中，找出區域網路閘道。 在我們的範例中，區域網路閘道是 **ClassicVNetLocal**。
2. 按一下 [組態]，並確認 IP 位址值是傳統 VNet 的 VPN 閘道。 視需要更新，然後按一下 [儲存]。 關閉頁面。
3. 在 [所有資源] 中，按一下區域網路閘道。
4. 按一下 [連線] 以開啟 [連線] 頁面。
5. 在 [連線] 頁面上，按一下 **+** 以新增連線。
6. 在 [新增連線] 頁面上，為連線命名。 例如，'RMtoClassic'。
7. 已在此頁面上選取 [站對站]。
8. 選取您想要與此網站產生關聯的虛擬網路閘道。
9. 建立**共用金鑰**。 這個金鑰也會用於您建立之傳統 VNet 到 Resource Manager VNet 的連線。 您可以產生金鑰或自行建立金鑰。 在我們的範例中，我們使用的是 'abc123'，但是您可以(且應該) 使用更為複雜的值。
10. 按一下 [確定] 來建立連線。

## <a name="classictoRM"></a>第 5 節 - 建立傳統到 Resource Manager 的連線

在這些步驟中，您會設定從傳統 VNet 到 Resource Manager VNet 的連線。 這些步驟需要 PowerShell。 您無法在入口網站中建立此連線。 確定您已下載並安裝傳統 (SM) 和 Resource Manager (RM) PowerShell Cmdlet。

### <a name="1-connect-to-your-azure-account"></a>1.連線至您的 Azure 帳戶

以提高的權限開啟 PowerShell 主控台並登入您的 Azure 帳戶。 登入之後，便會下載您的帳戶設定，以供 Azure PowerShell 使用。 下列 Cmdlet 會提示您輸入 Azure 帳戶的登入認證，以使用 Resource Manager 部署模型：

```powershell
Connect-AzureRmAccount
```

取得您的 Azure 訂用帳戶清單。

```powershell
Get-AzureRmSubscription
```

如果您有多個訂用帳戶，請指定您要使用的訂用帳戶。

```powershell
Select-AzureRmSubscription -SubscriptionName "Name of subscription"
```

接下來，請登入以使用傳統 PowerShell Cmdlet (服務管理)。 請使用以下命令新增您的 Azure 帳戶，以使用傳統部署模型：

```powershell
Add-AzureAccount
```

取得您的訂用帳戶清單。 新增服務管理 Cmdlet 時，此步驟可能是必要的，需視您安裝的 Azure 模組而定。

```powershell
Get-AzureSubscription
```

如果您有多個訂用帳戶，請指定您要使用的訂用帳戶。

```powershell
Select-AzureSubscription -SubscriptionName "Name of subscription"
```

### <a name="2-view-the-network-configuration-file-values"></a>2.檢視網路組態檔值

當您在 Azure 入口網站中建立 VNet 時，不會在 Azure 入口網站中顯示 Azure 所使用的完整名稱。 例如，在 Azure 入口網站中名稱顯示為 'ClassicVNet' 的 VNet，在網路組態檔中的名稱可能更長。 名稱可能如下︰'Group ClassicRG ClassicVNet'。 在這些步驟中，您下載網路組態檔並且檢視值。

在您的電腦上建立目錄，然後將網路組態檔匯出到該目錄。 在此範例中，會將網路組態檔匯出到 C:\AzureNet。

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

使用文字編輯器開啟檔案，並檢視傳統 VNet 的名稱。 執行 PowerShell Cmdlet 時，請使用網路組態檔中的名稱。

- VNet 名稱會列為 **VirtualNetworkSite name =**
- 網站名稱會列為 **LocalNetworkSite name=**

### <a name="3-create-the-connection"></a>3.建立連線

設定共用金鑰，並且建立從傳統 VNet 到 Resource Manager VNet 的連線。 您無法使用入口網站來設定共用金鑰。 若已使用傳統 PowerShell Cmdlet 版本登入，請務必執行這些步驟。 若要這麼做，請使用 **Add-azureaccount**。 否則，您將無法設定 '-AzureVNetGatewayKey'。

- 在此範例中，**-VNetName** 是在網路組態檔中找到的傳統 VNet 名稱。 
- **-LocalNetworkSiteName** 是您為網路組態檔中找到之本機網站指定的名稱。
- **-SharedKey** 是您產生和指定的值。 針對此範例，我們使用的是 *abc123*，但是您可以使用更為複雜的值。 重要的是，您在此指定的值必須與您在建立 Resource Manager 到傳統連線時指定的值相同。

```powershell
Set-AzureVNetGatewayKey -VNetName "Group ClassicRG ClassicVNet" `
-LocalNetworkSiteName "172B9E16_RMVNetLocal" -SharedKey abc123
```

## <a name="verify"></a>第 6 節 - 驗證您的連線

您可以使用 Azure 入口網站或 PowerShell 來驗證您的連線。 進行驗證時，您可能需要等候一或兩分鐘來建立連線。 連線成功時，連線狀態就會從「連線中」變更為「已連線」。

### <a name="to-verify-the-connection-from-your-classic-vnet-to-your-resource-manager-vnet"></a>驗證從傳統 VNet 到 Resource Manager VNet 的連線

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

### <a name="to-verify-the-connection-from-your-resource-manager-vnet-to-your-classic-vnet"></a>驗證從 Resource Manager VNet 到傳統 VNet 的連線

[!INCLUDE [vpn-gateway-verify-connection-portal-rm](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="faq"></a>VNet 對 VNet 常見問題集

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]
