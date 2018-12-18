---
title: 在不同 Azure Stack 開發套件環境中的兩個虛擬網路之間建立站對站 VPN 連線 |Microsoft Docs
description: 雲端系統管理員用來在兩個單節點「Azure Stack 開發套件」環境之間建立站對站 VPN 連線的逐步程序。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 3f1b4e02-dbab-46a3-8e11-a777722120ec
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/12/2018
ms.author: sethm
ms.reviewer: scottnap
ROBOTS: NOINDEX
ms.openlocfilehash: b17b1b077a1acac5ddb0e9eea8c4a2cf44697979
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078897"
---
# <a name="create-a-site-to-site-vpn-connection-between-two-virtual-networks-in-different-azure-stack-development-kit-environments"></a>在不同 Azure Stack 開發套件環境中的兩個虛擬網路之間建立站對站 VPN 連線
## <a name="overview"></a>概觀
此文章說明如何在兩個個別「Azure Stack 開發套件」環境中的兩個虛擬網路之間建立站對站 VPN 連線。 在您設定連線時，會了解 VPN 閘道在 Azure Stack 中的運作方式。

### <a name="connection-diagram"></a>連接圖表
下圖顯示當您完成時應有的連線組態。

![站對站 VPN 連線組態](media/azure-stack-create-vpn-connection-one-node-tp2/OneNodeS2SVPN.png)

### <a name="before-you-begin"></a>開始之前
若要完成連線組態，請務必在開始前備妥下列項目：

* 兩部符合如[快速入門：評估 Azure Stack 開發套件](azure-stack-deploy-overview.md)所述之 Azure Stack 開發套件硬體需求的伺服器和其他先決條件。 
* [Azure Stack 開發套件](https://azure.microsoft.com/overview/azure-stack/try/)部署套件。

## <a name="deploy-the-azure-stack-development-kit-environments"></a>部署 Azure Stack 開發套件環境
若要完成連線組態，您必須部署兩個「Azure Stack 開發套件」環境。
> [!NOTE] 
> 針對您部署的每個「Azure Stack 開發套件」，依照[部署指示](azure-stack-run-powershell-script.md)操作。 在此文章中，「Azure Stack 開發套件」環境的名稱是 *POC1* 和 *POC2*。


## <a name="prepare-an-offer-on-poc1-and-poc2"></a>在 POC1 和 POC2 上準備供應項目
在 POC1 和 POC2 上都準備一個供應項目，以便讓使用者能夠訂閱該供應項目並部署虛擬機器。 如需有關如何建立供應項目的資訊，請參閱[將虛擬機器提供給您的 Azure Stack 使用者](azure-stack-tutorial-tenant-vm.md)。

## <a name="review-and-complete-the-network-configuration-table"></a>檢閱並完成網路組態表
下表摘要說明這兩個「Azure Stack 開發套件」環境的網路組態。 請使用此表格後面所列的程序來新增您網路特定的外部 BGPNAT 位址。

**網路組態表**
|   |POC1|POC2|
|---------|---------|---------|
|虛擬網路名稱     |VNET-01|VNET-02 |
|虛擬網路位址空間 |10.0.10.0/23|10.0.20.0/23|
|子網路名稱     |Subnet-01|Subnet-02|
|子網路位址範圍|10.0.10.0/24 |10.0.20.0/24 |
|閘道器子網路     |10.0.11.0/24|10.0.21.0/24|
|外部 BGPNAT 位址     |         |         |

> [!NOTE]
> 範例環境中的外部 BGPNAT IP 位址是 10.16.167.195 (用於 POC1) 和 10.16.169.131 (用於 POC2)。 使用下列程序來決定您「Azure Stack 開發套件」主機的外部 BGPNAT IP 位址，然後將這些位址新增到先前的網路組態表。


### <a name="get-the-ip-address-of-the-external-adapter-of-the-nat-vm"></a>取得 NAT VM 的外部介面卡 IP 位址
1. 登入 POC1 的 Azure Stack 實體機器。
2. 編輯下列 Powershell 程式碼來取代您的系統管理員密碼，然後在 POC 主機上執行該程式碼：

   ```powershell
   cd \AzureStack-Tools-master\connect
   Import-Module .\AzureStack.Connect.psm1
   $Password = ConvertTo-SecureString "<your administrator password>" `
    -AsPlainText `
    -Force
   Get-AzureStackNatServerAddress `
    -HostComputer "AzS-bgpnat01" `
    -Password $Password
   ```
3. 將 IP 位址新增到上一節中出現的網路組態表。

4. 在 POC2 上重複執行此程序。

## <a name="create-the-network-resources-in-poc1"></a>在 POC 1 中建立網路資源
現在，您將建立設定閘道所需的 POC1 網路資源。 下列指示說明如何使用使用者入口網站來建立資源。 您也可以使用 PowerShell 程式碼來建立資源。

![用來建立資源的工作流程](media/azure-stack-create-vpn-connection-one-node-tp2/image2.png)

### <a name="sign-in-as-a-tenant"></a>以租用戶身分登入
服務系統管理員可以用租用戶身分登入，以測試其租用戶可能使用的方案、供應項目及訂用帳戶。 如果您還沒有租用戶帳戶，請先[建立租用戶帳戶](azure-stack-add-new-user-aad.md)再登入。

### <a name="create-the-virtual-network-and-vm-subnet"></a>建立虛擬網路和 VM 子網路
1. 使用租用戶帳戶來登入使用者入口網站。
2. 在使用者入口網站中，選取 [+ 建立資源]。
3. 移至 [Marketplace]，然後選取 [網路]。
4. 選取 [虛擬網路]。
5. 針對 [名稱]、[位址空間]、[子網路名稱] 及 [子網路位址範圍]，使用先前網路組態表中顯示的值。
6. 在 [訂用帳戶] 中，會顯示您先前建立的訂用帳戶。
7. 針對 [資源群組]，您可以建立資源群組，或選取 [使用現有的] \(如果您已經有資源群組)。
8. 驗證預設位置。
9. 選取 [釘選到儀表板]。
10. 選取 [建立] 。

### <a name="create-the-gateway-subnet"></a>建立閘道子網路
1. 在儀表板上，開啟您先前建立的 VNET-01 虛擬網路資源。
2. 在 [設定] 刀鋒視窗上選取 [子網路]。
3. 若要將閘道子網路新增到虛擬網路，請選取 [閘道子網路]。
   
    ![新增閘道子網路](media/azure-stack-create-vpn-connection-one-node-tp2/image4.png)

4. 子網路名稱預設為 **GatewaySubnet**。
   閘道子網路相當特殊。 為了正常運作，它們必須使用 *GatewaySubnet* 名稱。
5. 在 [位址範圍] 中，確認位址是 **10.0.11.0/24**。
6. 選取 [確定] 以建立閘道子網路。

### <a name="create-the-virtual-network-gateway"></a>建立虛擬網路閘道
1. 在 Azure 入口網站中，選取 [+ 建立資源]。 
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從網路資源清單中，選取 [虛擬網路閘道]。
4. 在 [名稱] 中，輸入 **GW1**。
5. 選取 [虛擬網路] 項目以選擇虛擬網路。
   從清單中選取 [VNET-01]。
6. 選取 [公用 IP 位址] 功能表項目。 當 [選擇公用 IP 位址] 刀鋒視窗開啟時，請選取 [新建]。
7. 在 [名稱] 中輸入 **GW1-PiP**，然後選取 [確定]。
8.  針對 [VPN 類型]，預設會選取 [依路由]。
    保留 [依路由] VPN 類型。
9. 確認 [訂用帳戶] 和 [位置] 均正確無誤。 您可以將資源釘選到儀表板。 選取 [建立] 。

### <a name="create-the-local-network-gateway"></a>建立區域網路閘道
在此 Azure Stack 評估部署與實際 Azure 部署中實作*區域網路閘道*的方式有點不同。

在 Azure 部署中，區域網路閘道代表您用來連線到 Azure 中虛擬網路閘道的內部部署 (位於租用戶) 實體裝置。 在這個 Azure Stack 評估部署中，連線的兩端都是虛擬網路閘道！

更廣泛來說，區域網路閘道資源一律是指連線另一端的遠端閘道。 由於「Azure Stack 開發套件」的設計方式緣故，因此您必須提供另一個「Azure Stack 開發套件」之網路位址轉譯 (NAT) VM 上的外部網路介面卡 IP 位址作為區域網路閘道的「公用 IP 位址」。 您接著需在 NAT VM 上建立 NAT 對應，以確保正確連接兩端。


### <a name="create-the-local-network-gateway-resource"></a>建立區域網路閘道資源
1. 登入 POC1 的 Azure Stack 實體機器。
2. 在使用者入口網站中，選取 [+ 建立資源]。
3. 移至 [Marketplace]，然後選取 [網路]。
4. 從資源清單中，選取 [區域網路閘道]。
5. 在 [名稱] 中，輸入 **POC2-GW**。
6. 在 [IP 位址] 中，輸入 POC2 的外部 BGPNAT 位址。 此位址先前已顯示在網路組態表中。
7. 在 [位址空間] 中，針對您稍後建立的 POC2 VNET 的位址空間，輸入 **10.0.20.0/23**。
8. 確認您的 [訂用帳戶]、[資源群組] 和 [位置] 正確無誤，然後選取 [建立]。

### <a name="create-the-connection"></a>建立連線
1. 在使用者入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從資源清單中，選取 [連線]。
4. 在 [基本] 設定刀鋒視窗上，針對 [連線類型]，選取 [站對站 (IPSec)]。
5. 選取 [訂用帳戶]、[資源群組] 和 [位置]，然後選取 [確定]。
6. 在 [設定] 刀鋒視窗上，選取 [虛擬網路閘道]，然後選取 [GW1]。
7. 選取 [區域網路閘道]，然後選取 [POC2-GW]。
8. 在 [連線名稱] 中，輸入 **POC1-POC2**。
9. 在 [共用金鑰 (PSK)] 中輸入 **12345**，然後選取 [確定]。
10. 在 [摘要] 刀鋒視窗上，選取 [確定]。

### <a name="create-a-vm"></a>建立 VM
若要驗證透過 VPN 連線傳輸的資料，您將需要虛擬機器，以在每個「Azure Stack 開發套件」中傳送和接收資料。 立即在 POC1 中建立虛擬機器，然後將它放在您虛擬網路的 VM 子網路上。

1. 在 Azure 入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [計算]。
3. 在虛擬機器映像清單中，選取 [Windows Server 2016 Datacenter 評估版] 映像。
4. 在 [基本] 刀鋒視窗的 [名稱] 中，輸入 **VM01**。
5. 輸入有效的使用者名稱和密碼。 建立 VM 之後，您將使用此帳戶來登入 VM。
6. 提供 [訂用帳戶]、[資源群組] 和 [位置]，然後選取 [確定]。
7. 在 [大小] 刀鋒視窗上，為此執行個體選取虛擬機器大小，然後選取 [選取]。
8. 在 [設定] 刀鋒視窗上，接受預設值。 確定選取的是 [VNET-01] 虛擬網路。 確認子網路已設定為 **10.0.10.0/24**。 然後選取 [確定]。
9. 在 [摘要] 刀鋒視窗上檢閱設定，然後選取 [確定]。



## <a name="create-the-network-resources-in-poc2"></a>在 POC 2 中建立網路資源

下一個步驟是建立 POC2 的網路資源。 下列指示說明如何使用使用者入口網站來建立資源。

### <a name="sign-in-as-a-tenant"></a>以租用戶身分登入
服務系統管理員可以用租用戶身分登入，以測試其租用戶可能使用的方案、供應項目及訂用帳戶。 如果您還沒有租用戶帳戶，請先[建立租用戶帳戶](azure-stack-add-new-user-aad.md)再登入。

### <a name="create-the-virtual-network-and-vm-subnet"></a>建立虛擬網路和 VM 子網路

1. 使用租用戶帳戶來登入。
2. 在使用者入口網站中，選取 [+ 建立資源]。
3. 移至 [Marketplace]，然後選取 [網路]。
4. 選取 [虛擬網路]。
5. 使用先前網路組態表中顯示的資訊，來指定 POC2 [名稱]、[位址空間]、[子網路名稱] 及 [子網路位址範圍] 的值。
6. 在 [訂用帳戶] 中，會顯示您先前建立的訂用帳戶。
7. 針對 [資源群組]，建立資源群組，或選取 [使用現有的] \(如果您已經有資源群組)。
8. 確認預設 [位置]。
9. 選取 [釘選到儀表板]。
10. 選取 [建立] 。

### <a name="create-the-gateway-subnet"></a>建立閘道子網路
1. 從儀表板開啟您建立的虛擬網路資源 (**VNET-02**)。
2. 在 [設定] 刀鋒視窗上選取 [子網路]。
3. 選取 [閘道子網路] 以將閘道子網路新增到虛擬網路。
4. 子網路名稱預設為 **GatewaySubnet**。
   閘道子網路很特別﹐必須具有此特定名稱，才能正常運作。
5. 在 [位址範圍] 欄位中，確認位址是 **10.0.21.0/24**。
6. 選取 [確定] 以建立閘道子網路。

### <a name="create-the-virtual-network-gateway"></a>建立虛擬網路閘道
1. 在 Azure 入口網站中，選取 [+ 建立資源]。  
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從網路資源清單中，選取 [虛擬網路閘道]。
4. 在 [名稱] 中，輸入 **GW2**。
5. 若要選擇虛擬網路，請選取 [虛擬網路]。 然後從清單中選取 [VNET-02]。
6. 選取 [公用 IP 位址]。 當 [選擇公用 IP 位址] 刀鋒視窗開啟時，請選取 [新建]。
7. 在 [名稱] 中輸入 **GW2-PiP**，然後選取 [確定]。
8. 針對 [VPN 類型]，預設會選取 [依路由]。
    保留 [依路由] VPN 類型。
9. 確認 [訂用帳戶] 和 [位置] 均正確無誤。 您可以將資源釘選到儀表板。 選取 [建立] 。

### <a name="create-the-local-network-gateway-resource"></a>建立區域網路閘道資源

1. 在 POC2 使用者入口網站中，選取 [+ 建立資源]。 
4. 移至 [Marketplace]，然後選取 [網路]。
5. 從資源清單中，選取 [區域網路閘道]。
6. 在 [名稱] 中，輸入 **POC1-GW**。
7. 在 [IP 位址] 中，輸入先前網路組態表中所列的 POC1 外部 BGPNAT 位址。
8. 在 [位址空間] 欄位中，從 POC1，輸入 **VNET-01** 的位址空間 **10.0.10.0/23**。
9. 確認您的 [訂用帳戶]、[資源群組] 和 [位置] 正確無誤，然後選取 [建立]。

## <a name="create-the-connection"></a>建立連線
1. 在使用者入口網站中，選取 [+ 建立資源]。 
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從資源清單中，選取 [連線]。
4. 在 [基本] 設定刀鋒視窗上，針對 [連線類型]，選擇 [站對站 (IPSec)]。
5. 選取 [訂用帳戶]、[資源群組] 和 [位置]，然後選取 [確定]。
6. 在 [設定] 刀鋒視窗上，選取 [虛擬網路閘道]，然後選取 [GW2]。
7. 選取 [區域網路閘道]，然後選取 [POC1-GW]。
8. 在 [連線名稱] 中，輸入 **POC2-POC1**。
9. 在 [共用金鑰 (PSK)] 中，輸入 **12345**。 如果您選擇不同的值，請記住該值「必須」與您在 POC1 上所建立共用金鑰的值相符。 選取 [確定] 。
10. 檢閱 [摘要] 刀鋒視窗，然後選取 [確定]。

## <a name="create-a-virtual-machine"></a>建立虛擬機器
請立即在 POC2 中建立虛擬機器，然後將它放在您虛擬網路的 VM 子網路上。

1. 在 Azure 入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [計算]。
3. 在虛擬機器映像清單中，選取 [Windows Server 2016 Datacenter 評估版] 映像。
4. 在 [基本] 刀鋒視窗上，針對 [名稱]，輸入 **VM02**。
5. 輸入有效的使用者名稱和密碼。 建立虛擬機器之後，您將使用此帳戶來登入虛擬機器。
6. 提供 [訂用帳戶]、[資源群組] 和 [位置]，然後選取 [確定]。
7. 在 [大小] 刀鋒視窗上，為此執行個體選取虛擬機器大小，然後選取 [選取]。
8. 在 [設定] 刀鋒視窗上，您可以接受預設值。 確定選取的是 [VNET-02] 虛擬網路，並確認子網路已設定為 **10.0.20.0/24**。 選取 [確定] 。
9. 檢閱 [摘要] 刀鋒視窗上的設定，然後選取 [確定]。

## <a name="configure-the-nat-virtual-machine-on-each-azure-stack-development-kit-for-gateway-traversal"></a>在每個 Azure Stack 開發套件上設定 NAT 虛擬機器以進行閘道周遊
由於「Azure Stack 開發套件」是獨立的並與實體主機部署所在的網路隔離，因此閘道所連接的「外部」VIP 網路實際上並不在外部。 取而代之的是，VIP 網路隱藏在執行網路位址轉譯的路由器之後。 

路由器是一部名為 *AzS-bgpnat01* 的 Windows Server 虛擬機器，在「Azure Stack 開發套件」基礎結構中執行「路由及遠端存取服務」(RRAS) 角色。 您必須在 AzS-bgpnat01 虛擬機器上設定 NAT，才能允許站對站 VPN 連線連接兩端。 

若要設定 VPN 連線，您必須建立靜態 NAT 對應路由，以將 BGPNAT 虛擬機器上的外部介面對應至「邊緣閘道集區」的 VIP。 VPN 連線中的每個連接埠都必須要有一個靜態 NAT 對應路由。

> [!NOTE]
> 只有「Azure Stack 開發套件」環境才需要此組態。
> 
> 

### <a name="configure-the-nat"></a>設定 NAT
> [!IMPORTANT]
> 您必須為兩個「Azure Stack 開發套件」環境都完成此程序。

1. 決定要在下列 PowerShell 指令碼中使用的「內部 IP 位址」。 開啟虛擬網路閘道 (GW1 和 GW2)，然後在 [概觀] 刀鋒視窗上，儲存 [公用 IP 位址] 的值以供稍後使用。
![內部 IP 位址](media/azure-stack-create-vpn-connection-one-node-tp2/InternalIP.PNG)
2. 登入 POC1 的 Azure Stack 實體機器。
3. 複製並編輯下列 PowerShell 指令碼。 若要在每個「Azure Stack 開發套件」上設定 NAT，請在已提高權限的 Windows PowerShell ISE 中執行該指令碼。 在指令碼中，將值新增到 [外部 BGPNAT 位址] 和 [內部 IP 位址] 預留位置：

   ```powershell
   # Designate the external NAT address for the ports that use the IKE authentication.
   Invoke-Command `
    -ComputerName AzS-bgpnat01 `
     {Add-NetNatExternalAddress `
      -NatName BGPNAT `
      -IPAddress <External BGPNAT address> `
      -PortStart 499 `
      -PortEnd 501}
   Invoke-Command `
    -ComputerName AzS-bgpnat01 `
     {Add-NetNatExternalAddress `
      -NatName BGPNAT `
      -IPAddress <External BGPNAT address> `
      -PortStart 4499 `
      -PortEnd 4501}
   # create a static NAT mapping to map the external address to the Gateway
   # Public IP Address to map the ISAKMP port 500 for PHASE 1 of the IPSEC tunnel
   Invoke-Command `
    -ComputerName AzS-bgpnat01 `
     {Add-NetNatStaticMapping `
      -NatName BGPNAT `
      -Protocol UDP `
      -ExternalIPAddress <External BGPNAT address> `
      -InternalIPAddress <Internal IP address> `
      -ExternalPort 500 `
      -InternalPort 500}
   # Finally, configure NAT traversal which uses port 4500 to
   # successfully establish the complete IPSEC tunnel over NAT devices
   Invoke-Command `
    -ComputerName AzS-bgpnat01 `
     {Add-NetNatStaticMapping `
      -NatName BGPNAT `
      -Protocol UDP `
      -ExternalIPAddress <External BGPNAT address> `
      -InternalIPAddress <Internal IP address> `
      -ExternalPort 4500 `
      -InternalPort 4500}
   ```

4. 在 POC2 上重複執行此程序。

## <a name="test-the-connection"></a>測試連線
既然已建立站對站連線，您現在應驗證是否可以透過它傳送流量。 若要驗證，請登入您在其中一個「Azure Stack 開發套件」環境中建立的其中一部虛擬機器。 接著，Ping 您在另一個環境中建立的虛擬機器。 

為了確保您是透過站對站連線傳送流量，請確定您 Ping 的是遠端子網路上虛擬機器的直接 IP (DIP) 位址，而非 VIP。 若要這麼做，請找出連線另一端的 DIP 位址。 儲存位址以供稍後使用。

### <a name="sign-in-to-the-tenant-vm-in-poc1"></a>登入 POC1 中的租用戶 VM
1. 登入 POC1 的 Azure Stack 實體機器，然後使用租用戶帳戶來登入使用者入口網站。
2. 在左導覽列中，選取 [計算]。
3. 在 VM 清單中，尋找您先前建立的 **VM01**，然後選取它。
4. 在虛擬機器的刀鋒視窗上，按一下 [連線]，然後開啟 VM01.rdp 檔案。
   
     ![[連線] 按鈕](media/azure-stack-create-vpn-connection-one-node-tp2/image17.png)
5. 使用您建立虛擬網路時所設定的帳戶來登入。
6. 開啟已提高權限的 [Windows PowerShell] 視窗。
7. 輸入 **ipconfig /all**。
8. 在輸出中，尋找「IPv4 位址」，然後儲存該位址以供稍後使用。 這是您將從 POC2 Ping 的位址。 在範例環境中，位址是 **10.0.10.4**，但在您的環境中可能會不同。 它應該落在您先前建立的 **10.0.10.0/24** 子網路內。
9. 若要建立允許虛擬機器回應 Ping 的防火牆規則，請執行下列 PowerShell 命令：

   ```powershell
   New-NetFirewallRule `
    –DisplayName “Allow ICMPv4-In” `
    –Protocol ICMPv4
   ```

### <a name="sign-in-to-the-tenant-vm-in-poc2"></a>登入 POC2 中的租用戶 VM
1. 登入 POC2 的 Azure Stack 實體機器，然後使用租用戶帳戶來登入使用者入口網站。
2. 在左導覽列中，按一下 [計算]。
3. 從虛擬機器清單中，尋找您先前建立的 **VM02**，然後選取它。
4. 在虛擬機器的刀鋒視窗中，按一下 [ **連線**]。
5. 使用您建立虛擬網路時所設定的帳戶來登入。
6. 開啟已提高權限的 [Windows PowerShell] 視窗。
7. 輸入 **ipconfig /all**。
8. 您應該會看到落在 **10.0.20.0/24** 內的 IPv4 位址。 在範例環境中，該位址是 **10.0.20.4**，但您的位址可能會不同。
9. 若要建立允許虛擬機器回應 Ping 的防火牆規則，請執行下列 PowerShell 命令：

   ```powershell
   New-NetFirewallRule `
    –DisplayName “Allow ICMPv4-In” `
    –Protocol ICMPv4
   ```

10. 從 POC2 上的虛擬機器，透過通道 Ping POC1 上的虛擬機器。 若要這麼做，您需 Ping 從 VM01 記錄的 DIP。
   在範例環境中，這是 **10.0.10.4**，但請務必 Ping 您在實驗室中記下的位址。 您應該會看到如下所示的結果：
   
    ![成功的 Ping](media/azure-stack-create-vpn-connection-one-node-tp2/image19b.png)
11. 來自遠端虛擬機器的回覆表示測試成功！ 您可以關閉虛擬機器視窗。 若要測試您的連線，您可以嘗試其他類型的資料傳輸，例如檔案複製。

### <a name="viewing-data-transfer-statistics-through-the-gateway-connection"></a>檢視透過閘道連線的資料傳輸統計資料
如果您想要知道有多少資料通過您的站對站連線，可在 [連線] 刀鋒視窗中取得此資訊。 此測試也是確認您剛傳送的 Ping 是否真的通過 VPN 連線的另一種方法。

1. 在您已登入 POC2 中租用戶虛擬機器的情況下，使用您的租用戶帳戶來登入使用者入口網站。
2. 移至 [所有資源]，然後選取 [POC2-POC1] 連線。 [連線] 隨即顯示。
4. 在 [連線] 刀鋒視窗上，會顯示 [資料輸入] 和 [資料輸出] 的統計資料。 在以下的螢幕擷取畫面中，那些大的數字歸因於額外的檔案傳輸。 您應該會在該處看到一些非零的值。
   
    ![資料輸入和輸出](media/azure-stack-create-vpn-connection-one-node-tp2/image20.png)
