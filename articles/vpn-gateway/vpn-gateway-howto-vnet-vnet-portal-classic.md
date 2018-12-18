---
title: 建立 VNet 之間的連線：傳統：Azure 入口網站 | Microsoft Docs
description: 使用 PowerShell 和 Azure 入口網站將 Azure 虛擬網路連接在一起。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/14/2018
ms.author: cherylmc
ms.openlocfilehash: d9766afefa793baf66ea5218843f06031b1b364c
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31601051"
---
# <a name="configure-a-vnet-to-vnet-connection-classic"></a>設定 VNet 對 VNet 連線 (傳統)

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

本文可協助您建立虛擬網路之間的 VPN 閘道連線。 虛擬網路可位於相同或不同的區域，以及來自相同或不同的訂用帳戶。 本文中的步驟適用於傳統部署模型和 Azure 入口網站。 您也可從下列清單中選取不同的選項，以使用不同的部署工具或部署模型來建立此組態：

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Azure 入口網站 (傳統)](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [連線不同的部署模型 - Azure 入口網站](vpn-gateway-connect-different-deployment-models-portal.md)
> * [連線不同的部署模型 - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

![VNet 對 VNet 連線能力圖表](./media/vpn-gateway-howto-vnet-vnet-portal-classic/v2vclassic.png)

## <a name="about-vnet-to-vnet-connections"></a>關於 VNet 對 VNet 連線

在傳統部署模型中使用 VPN 閘道將一個虛擬網路連線到另一個虛擬網路 (VNet 對 VNet)，類似於將虛擬網路連線到內部部署網站位置。 這兩種連線類型都使用 VPN 閘道提供使用 IPsec/IKE 的安全通道。

您所連線的 Vnet 可位於不同的訂用帳戶和不同的區域。 您可以將 VNet 對 VNet 通訊與多站台組態結合。 這可讓您建立結合了跨單位連線與內部虛擬網路連線的網路拓撲。

![VNet 對 VNet 連線](./media/vpn-gateway-howto-vnet-vnet-portal-classic/aboutconnections.png)

### <a name="why"></a>為什麼要連線虛擬網路？

針對下列原因，您可能希望連接虛擬網路：

* **跨區域的異地備援和異地目前狀態**

  * 您可以使用安全連線設定自己的異地複寫或同步處理，而不用查看網際網路對向端點。
  * 有了 Azure Load Balancer 和 Microsoft 或第三方叢集技術，您便可以利用跨多個 Azure 區域的異地備援，設定具有高可用性的工作負載。 其中一個重要的範例就是使用分散在多個 Azure 區域的可用性群組來設定 SQL Always On。
* **具有嚴密隔離界限的區域性多層式應用程式**

  * 在相同的區域中，您可以設定既有多個 VNet 相連又有嚴密的隔離及安全的層次間通訊的多層式應用程式。
* **Azure 中的跨訂用帳戶、組織間通訊**

  * 如果您有多個 Azure 訂用帳戶，您可以將虛擬網路之間不同訂用帳戶的工作負載安全地連接在一起。
  * 針對企業或服務提供者，您可以在 Azure 中使用安全 VPN 技術啟用跨組織通訊。

如需 VNet 對 VNet 連接的詳細資訊，請參閱本文結尾處的 [VNet 對 VNet 的考量](#faq)。

### <a name="before-you-begin"></a>開始之前

開始本練習之前，請下載並安裝最新版的 Azure 服務管理 (SM) PowerShell Cmdlet。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。 我們使用入口網站來執行大部分的步驟，但是您必須使用 PowerShell 來建立 Vnet 之間的連線。 您無法使用 Azure 入口網站建立連線。

## <a name="plan"></a>步驟 1 - 規劃 IP 位址範圍

決定將用來設定虛擬網路的範圍相當重要。 就此組態而言，您必須確定沒有任何 VNet 範圍彼此重疊，或是與其連接的區域網路重疊。

下表顯示一個範例，說明如何定義 VNet。 範圍僅供作為指導方針。 請記下您虛擬網路的範圍。 稍後的步驟將會需要這項資訊。

**範例**

| 虛擬網路 | 位址空間 | 區域 | 連接至區域網路站台 |
|:--- |:--- |:--- |:--- |
| TestVNet1 |TestVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |美國東部 |VNet4Local<br>(10.41.0.0/16)<br>(10.42.0.0/16) |
| TestVNet4 |TestVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |美國西部 |VNet1Local<br>(10.11.0.0/16)<br>(10.12.0.0/16) |

## <a name="vnetvalues"></a>步驟 2 - 建立虛擬網路

在 [Azure 入口網站](https://portal.azure.com)中建立兩個虛擬網路。 如需建立傳統虛擬網路的步驟，請參閱[建立傳統虛擬網路](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)。 

使用入口網站建立傳統虛擬網路時，您必須使用下列步驟瀏覽至 [虛擬網路] 頁面，否則建立傳統虛擬網路的選項就不會出現：

1. 按一下 [+] 以開啟 [新增] 頁面。
2. 在 [搜尋 Marketplace] 欄位中，輸入「虛擬網路」。 如果您改為選取 [網路功能]-> [虛擬網路]，將無法取得建立傳統 VNet 的選項。
3. 在傳回的清單中找出 [虛擬網路]，然後按一下以開啟 [虛擬網路] 頁面。 
4. 在 [虛擬網路] 頁面上，選取 [傳統] 以建立傳統的 VNet。 

如果您使用這篇文章作為練習，您可以使用下列範例值：

**TestVNet1 的值**

名稱︰TestVNet1<br>
位址空間︰10.11.0.0/16、10.12.0.0/16 (選擇性)<br>
子網路名稱：default<br>
子網路位址範圍：10.11.0.1/24<br>
資源群組：ClassicRG<br>
位置：美國東部<br>
GatewaySubnet：10.11.1.0/27

**TestVNet4 的值**

名稱︰TestVNet4<br>
位址空間︰10.41.0.0/16、10.42.0.0/16 (選擇性)<br>
子網路名稱：default<br>
子網路位址範圍：10.41.0.1/24<br>
資源群組：ClassicRG<br>
位置：美國西部<br>
GatewaySubnet：10.41.1.0/27

**在建立 VNet 時，請牢記下列設定︰**

* **虛擬網路位址空間** – 在 [虛擬網路位址空間] 頁面上，指定您想要用於虛擬網路的位址範圍。 這些是將指派給 VM 的動態 IP 位址，以及指派給您部署至此虛擬網路之其他角色執行個體的動態 IP 位址。<br>您選取的位址空間不能與任何其他 VNet 的位址空間，或此 VNet 連線之內部部署位置的位址空間重疊。

* **位置** – 當您建立虛擬網路時，您會將該位置與 Azure 位置 (區域) 產生關聯。 例如，如果您希望部署到虛擬網路的 VM 實際位於美國西部，請選取該位置。 建立關聯之後，您就無法變更與您的虛擬網路相關聯的位置。

**建立 VNet 之後，您可以新增下列設定︰**

* **位址空間** – 此組態不需要額外的位址空間，但是您可以在建立 VNet 之後新增額外的位址空間。

* **子網路** – 此組態不需要額外的子網路，但是您可能想要讓您的 VM 位於與其他角色執行個體不同的子網路中。

* **DNS 伺服器** – 輸入 DNS 伺服器名稱和 IP 位址。 此設定不會建立 DNS 伺服器。 它可讓您指定要用於此虛擬網路之名稱解析的 DNS 服務。

在本節中，您會設定連線類型、本機網站，並建立閘道。

## <a name="localsite"></a>步驟 3 - 設定本機網站

Azure 會使用每個區域網路站台中指定的設定，來決定如何路由傳送 VNet 之間的流量。 每個 VNet 都必須指向要做為流量路由傳送目的地的個別區域網路。 您需決定要用來參考每個區域網路站台的名稱。 最好使用描述性的項目。

例如，TestVNet1 會連線到您所建立名為 "VNet4Local" 的區域網路網站。 VNet4Local 的設定包含 TestVNet4 位址首碼。

每個 VNet 的本機網站是其他 VNet。 在我們的組態中使用下列範例值︰

| 虛擬網路 | 位址空間 | 區域 | 連接至區域網路站台 |
|:--- |:--- |:--- |:--- |
| TestVNet1 |TestVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |美國東部 |VNet4Local<br>(10.41.0.0/16)<br>(10.42.0.0/16) |
| TestVNet4 |TestVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |美國西部 |VNet1Local<br>(10.11.0.0/16)<br>(10.12.0.0/16) |

1. 在 Azure 入口網站中找出 TestVNet1。 在頁面的 [VPN 連線] 區段中，按一下 [閘道]。

    ![沒有閘道](./media/vpn-gateway-howto-vnet-vnet-portal-classic/nogateway.png)
2. 在 [新增 VPN 連線] 頁面上，選取 [站對站]。
3. 按一下 [本機網站] 以開啟 [本機網站] 頁面，並且進行設定。
4. 在 [本機網站] 頁面上，為您的本機網站命名。 在我們的範例中，我們將本機網站命名為 'VNet4Local'。
5. 針對 [VPN 閘道 IP 位址]，只要是有效的格式，您可以使用您想要的任何 IP 位址。 一般而言，您會將實際的外部 IP 位址用於 VPN 裝置。 但是針對傳統 VNet 對 VNet 組態，您需使用指派給您 VNet 閘道的公用 IP 位址。 由於您尚未建立虛擬網路閘道，因此您可以指定任何有效的公用 IP 位址作為預留位置。<br>請勿將此欄位留白 - 這不是此組態的選擇性欄位。 在稍後的步驟中，您將在 Azure 產生閘道之後，回到這些設定，並使用對應的虛擬網路閘道 IP 位址來進行設定。
6. 針對 [用戶端位址空間]，使用其他 VNet 的位址空間。 請參閱您的計劃範例。 按一下 [確定] 來儲存設定，並且返回 [新增 VPN 連線] 頁面。

    ![本機網站](./media/vpn-gateway-howto-vnet-vnet-portal-classic/localsite.png)

## <a name="gw"></a>步驟 4 - 建立虛擬網路閘道

每個虛擬網路都必須有虛擬網路閘道。 虛擬網路閘道會路由傳送流量並且加密。

1. 在 [新增 VPN 連線] 頁面上，選取 [立即建立閘道] 核取方塊。
2. 按一下 [子網路、大小和路由類型]。 在 [閘道組態] 頁面上，按一下 [子網路]。
3. 閘道子網路名稱會自動填入必要名稱 'GatewaySubnet'。 [位址範圍] 包含配置給 VPN 閘道服務的 IP 位址。 某些組態允許閘道子網路 /29，但是最好是使用 /28 或 /27 以容納未來可能需要更多閘道服務 IP 位址的組態。 在我們的範例設定中，我們會使用 10.11.1.0/27。 調整位址空間，然後按一下 [確定]。
4. 設定**閘道大小**。 此設定表示[閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。
5. 設定**路由類型**。 此組態的路由類型必須是**動態**。 除非您卸除閘道並且建立一個新閘道，否則您無法在稍後變更路由類型。
6. 按一下 [SERVICEPRINCIPAL] 。
7. 在 [新增 VPN 連線] 頁面上，按一下 [確定] 以開始建立虛擬網路閘道。 建立閘道通常可能需要 45 分鐘或更久，視選取的閘道 SKU 而定。

## <a name="vnet4settings"></a>步驟 5 - 進行 TestVNet4 設定

重複 [建立本機網站](#localsite) 和 [建立虛擬網路閘道](#gw) 的步驟以設定 TestVNet4，並且視需要取代值。 如果您執行這個操作作為練習，請使用[範例值](#vnetvalues)。

## <a name="updatelocal"></a>步驟 6 - 更新本機網站

為兩個 VNet 建立虛擬網路閘道之後，您必須調整本機網站 **VPN 閘道 IP 位址**值。

|VNet 名稱|已連線網站|閘道 IP 位址|
|:--- |:--- |:--- |
|TestVNet1|VNet4Local|TestVNet4 的 VPN 閘道 IP 位址|
|TestVNet4|VNet1Local|TestVNet1 的 VPN 閘道 IP 位址|

### <a name="part-1---get-the-virtual-network-gateway-public-ip-address"></a>第 1 部分 - 取得虛擬網路閘道公用 IP 位址

1. 在 Azure 入口網站中找到虛擬網路。
2. 按一下以開啟 VNet 的 [概觀] 頁面。 在頁面的 [VPN 連線] 中，您可以檢視虛擬網路閘道的 IP 位址。

  ![公用 IP](./media/vpn-gateway-howto-vnet-vnet-portal-classic/publicIP.png)
3. 複製 IP 位址。 您在下一節將會用到此位址。
4. 對 TestVNet4 重複執行這些步驟

### <a name="part-2---modify-the-local-sites"></a>第 2 部分 - 修改本機網站

1. 在 Azure 入口網站中找到虛擬網路。
2. 在 VNet 的 [概觀] 頁面上，按一下本機網站。

  ![建立本機網站](./media/vpn-gateway-howto-vnet-vnet-portal-classic/local.png)
3. 在 [站對站 VPN 連線] 頁面上，按一下您想要修改之本機網站的名稱。

  ![開啟本機網站](./media/vpn-gateway-howto-vnet-vnet-portal-classic/openlocal.png)
4. 按一下您想要修改的 [本機網站]。

  ![修改網站](./media/vpn-gateway-howto-vnet-vnet-portal-classic/connections.png)
5. 更新 [VPN 閘道 IP 位址]，然後按一下 [確定] 以儲存設定。

  ![閘道 IP](./media/vpn-gateway-howto-vnet-vnet-portal-classic/gwupdate.png)
6. 關閉其他頁面。
7. 對 TestVNet4 重複執行這些步驟。

## <a name="getvalues"></a>步驟 7 - 從網路組態檔擷取值

當您在 Azure 入口網站中建立傳統 VNet 時，您所檢視的名稱不是在 PowerShell 中使用的完整名稱。 例如，在入口網站中名稱顯示為 **TestVNet1** 的 VNet，在網路組態檔中的名稱可能更長。 名稱可能如下︰**Group ClassicRG TestVNet1**。 當您建立連線時，務必使用您在網路組態檔中看到的值。

在下列步驟中，您將會連線到您的 Azure 帳戶，並且下載及檢視網路組態檔，以取得連線的必要值。

1. 下載並安裝最新版的 Azure 服務管理 (SM) PowerShell Cmdlet。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。

2. 以提高的權限開啟 PowerShell 主控台並連接到您的帳戶。 使用下列範例來協助您連接：

  ```powershell
  Connect-AzureRmAccount
  ```

  檢查帳戶的訂用帳戶。

  ```powershell
  Get-AzureRmSubscription
  ```

  如果您有多個訂用帳戶，請選取您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
  ```

  接下來，使用下列 Cmdlet，將您的 Azure 訂用帳戶新增到 PowerShell，以供傳統部署模型使用。

  ```powershell
  Add-AzureAccount
  ```
3. 匯出並檢視網路組態檔。 在您的電腦上建立目錄，然後將網路組態檔匯出到該目錄。 在此範例中，會將網路組態檔匯出到 **C:\AzureNet**。

  ```powershell
  Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
  ```
4. 使用文字編輯器開啟檔案，並且檢視 VNet 和網站的名稱。 這些名稱是您建立連線時使用的名稱。<br>VNet 名稱會列為 **VirtualNetworkSite name =**<br>網站名稱會列為 **LocalNetworkSiteRef name =**

## <a name="createconnections"></a>步驟 8 - 建立 VPN 閘道連線

當所有先前的步驟都已完成時，您可以設定 IPsec/IKE 預先共用金鑰並建立連線。 這組步驟會使用 PowerShell。 無法在 Azure 入口網站中設定傳統部署模型的 VNet 對 VNet 連線。

在這些範例中，請注意，共用金鑰是完全相同的。 共用金鑰必須一律相符。 請務必使用 VNet 和區域網路網站的確切名稱，取代這些範例中的值。

1. 建立 TestVNet1 至 TestVNet4 的連線。

  ```powershell
  Set-AzureVNetGatewayKey -VNetName 'Group ClassicRG TestVNet1' `
  -LocalNetworkSiteName '17BE5E2C_VNet4Local' -SharedKey A1b2C3D4
  ```
2. 建立 TestVNet4 至 TestVNet1 的連線。

  ```powershell
  Set-AzureVNetGatewayKey -VNetName 'Group ClassicRG TestVNet4' `
  -LocalNetworkSiteName 'F7F7BFC7_VNet1Local' -SharedKey A1b2C3D4
  ```
3. 等待連線初始化。 一旦閘道初始化，其狀態為「成功」。

  ```
  Error          :
  HttpStatusCode : OK
  Id             :
  Status         : Successful
  RequestId      :
  StatusCode     : OK
  ```

## <a name="faq"></a>傳統 VNet 的 VNet 對 VNet 考量
* 虛擬網路可位於相同或不同的訂用帳戶。
* 虛擬網路可位於相同或不同的 Azure 區域 (位置)。
* 即使虛擬網路連接在一起，雲端服務或負載平衡端點也無法跨虛擬網路。
* 將多個虛擬網路連接在一起並不需要任何 VPN 裝置。
* VNet 對 VNet 支援連接 Azure 虛擬網路。 它不支援連接未部署到虛擬網路中的虛擬機器或雲端服務。
* VNet 對 VNet 需要動態路由閘道。 不支援 Azure 靜態路由閘道。
* 虛擬網路連線能力可以與多站台 VPN 同時使用。 一個虛擬網路 VPN 閘道最多可以有 10 條 VPN 通道連接至其他虛擬網路或內部部署站台。
* 虛擬網路與內部部署區域網路網站的位址空間不得重疊。 位址空間重疊會導致建立虛擬網路或上傳 netcfg 組態檔失敗。
* 不支援成對虛擬網路之間的備援通道。
* VNet 的所有 VPN 通道 (包括 P2S VPN) 在 Azure 中皆共用 VPN 閘道的可用頻寬及相同的 VPN 閘道執行時間 SLA。
* VNet 對 VNet 流量會經過 Azure 的骨幹。

## <a name="next-steps"></a>後續步驟
確認您的連線。 [確認 VPN 閘道連線](vpn-gateway-verify-connection-resource-manager.md)。
