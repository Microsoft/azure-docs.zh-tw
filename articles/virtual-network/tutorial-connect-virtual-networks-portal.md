---
title: 使用虛擬網路對等互連來連線虛擬網路 -教學課程 - Azure 入口網站 | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Azure 入口網站，透過虛擬網路對等互連來連線虛擬網路。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to connect two virtual networks so that virtual machines in one virtual network can communicate with virtual machines in the other virtual network.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: d702253c7b58b0a29c03e6563238b56ae75fa0d1
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "30841783"
---
# <a name="tutorial-connect-virtual-networks-with-virtual-network-peering-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站透過虛擬網路對等互連來連線虛擬網路

您可以使用虛擬網路對等互連，讓虛擬網路彼此連線。 一旦虛擬網路對等互連，兩個虛擬網路中的資源就可以彼此通訊，且通訊時會有相同的延遲和頻寬，彷彿這些資源是位於相同的虛擬網路中。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立兩個虛擬網路
> * 使用虛擬網路對等互連連線兩個虛擬網路
> * 將虛擬機器 (VM) 部署到每個虛擬網路
> * 虛擬機器之間的通訊

您可以依偏好使用 [Azure CLI](tutorial-connect-virtual-networks-cli.md) 或 [Azure PowerShell](tutorial-connect-virtual-networks-powershell.md) 完成本教學課程。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="log-in-to-azure"></a>登入 Azure 

在 https://portal.azure.com 上登入 Azure 入口網站。

## <a name="create-virtual-networks"></a>建立虛擬網路

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [網絡]，然後選取 [虛擬網路]。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    |設定|值|
    |---|---|
    |Name|myVirtualNetwork1|
    |位址空間|10.0.0.0/16|
    |訂用帳戶| 選取您的訂用帳戶。|
    |資源群組| 選取 [新建]，然後輸入 *myResourceGroup*。|
    |位置| 選取 [美國東部]。|
    |子網路名稱|Subnet1|
    |子網路位址範圍|10.0.0.0/24|

      ![建立虛擬網路](./media/tutorial-connect-virtual-networks-portal/create-virtual-network.png)

4. 以下列變更再次完成步驟 1-3：

    |設定|值|
    |---|---|
    |Name|myVirtualNetwork2|
    |位址空間|10.1.0.0/16|
    |資源群組| 選取 [使用現有的]，然後選取 [myResourceGroup]。|
    |子網路位址範圍|10.1.0.0/24|

## <a name="peer-virtual-networks"></a>對等互連虛擬網路

1. 在 Azure 入口網站頂端的 [搜尋] 方塊中，開始輸入 MyVirtualNetwork1。 當搜尋結果中出現 myVirtualNetwork1 時加以選取。
2. 選取 [設定] 底下的 [對等互連]，然後選取 [+新增]，如下圖所示：

    ![建立對等互連](./media/tutorial-connect-virtual-networks-portal/create-peering.png)

3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [確定]。

    |設定|值|
    |---|---|
    |Name|myVirtualNetwork1-myVirtualNetwork2|
    |訂用帳戶| 選取您的訂用帳戶。|
    |虛擬網路|myVirtualNetwork2 - 若要選取 myVirtualNetwork2 虛擬網路，請選取 [虛擬網路]，然後選取 [myVirtualNetwork2]。|

    ![對等互連設定](./media/tutorial-connect-virtual-networks-portal/peering-settings.png)

    「對等互連狀態」是「已啟動」，如下圖所示：

    ![對等互連狀態](./media/tutorial-connect-virtual-networks-portal/peering-status.png)

    如果您沒有看到狀態，請重新整理瀏覽器。

4. 在 Azure 入口網站頂端的 [搜尋] 方塊中，開始輸入 MyVirtualNetwork2。 當搜尋結果中出現 **myVirtualNetwork2** 時加以選取。
5. 以下列變更再次完成步驟 2-3，然後選取 [確定]：

    |設定|值|
    |---|---|
    |Name|myVirtualNetwork2-myVirtualNetwork1|
    |虛擬網路|myVirtualNetwork1|

    「對等互連狀態」是「已連線」。 Azure 也會將 myVirtualNetwork2-myVirtualNetwork1 對等互連的對等互連狀態從「已啟動」變更為「已連線」。 在兩個虛擬網路的對等互連狀態變為「已連線」之後，您才能成功建立虛擬網路對等互連。 

## <a name="create-virtual-machines"></a>建立虛擬機器

在每個虛擬網路中建立虛擬機器，以便您可以在稍後的步驟中於彼此之間通訊。

### <a name="create-the-first-vm"></a>建立第一個 VM

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [計算]，然後選取 [Windows Server 2016 Datacenter]。 您可以選取不同的作業系統，但是其餘步驟假設您選取的是 **Windows Server 2016 Datacenter**。 
3. 針對 [基本資料] 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    |設定|值|
    |---|---|
    |Name|myVm1|
    |使用者名稱| 輸入您選擇的使用者名稱。|
    |密碼| 輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |資源群組| 選取 [使用現有的]，然後選取 [myResourceGroup]。|
    |位置| 選取 [美國東部]。|
4. 在 [選擇大小] 底下選取虛擬機器大小。
5. 針對 [設定] 選取下列值，然後選取 [確定]：
    |設定|值|
    |---|---|
    |虛擬網路| myVirtualNetwork1 - 若未選取，請選取 [虛擬網路]，然後在 [選擇虛擬網路] 底下選取 [myVirtualNetwork1]。|
    |子網路| Subnet1 - 若未選取，請選取 [子網路]，然後在 [選擇子網路] 底下選取 [Subnet1]。|
    
    ![虛擬機器設定](./media/tutorial-connect-virtual-networks-portal/virtual-machine-settings.png)
 
6. 在 [摘要] 的 [建立] 底下，選取 [建立] 來開始部署虛擬機器。

### <a name="create-the-second-vm"></a>建立第二個 VM

以下列變更再次完成步驟 1-6：

|設定|值|
|---|---|
|Name | myVm2|
|虛擬網路 | myVirtualNetwork2|

可能需要數分鐘才會建立虛擬機器。 請等到這兩個虛擬機器都已建立，再繼續進行其餘步驟。

## <a name="communicate-between-vms"></a>虛擬機器之間的通訊

1. 在入口網站頂端的 [搜尋] 方塊中，開始輸入 myVm1。 當 **myVm1** 出現在搜尋結果中時，選取它。
2. 藉由選取 [連線] 來建立與 myVm1 虛擬機器的遠端桌面連線，如下圖所示：

    ![連線至虛擬機器](./media/tutorial-connect-virtual-networks-portal/connect-to-virtual-machine.png)  

3. 若要連線至虛擬機器，請開啟所下載的 RDP 檔案。 如果出現提示，請選取 [連接]。
4. 輸入您在建立虛擬機器時指定的使用者名稱和密碼 (您可能需要選取 [更多選擇]，然後選取 [使用不同的帳戶] 以指定您在建立虛擬機器時輸入的認證)，然後選取 [確定]。
5. 您可能會在登入過程中收到憑證警告。 選取 [是] 以繼續進行連線。
6. 稍後的步驟會使用 ping 從 myVm1 虛擬機器與 myVm2 虛擬機器通訊。 Ping 會使用網際網路控制訊息通訊協定 (ICMP)，它在通過 Windows 防火牆時預設會遭到拒絕。 在 *myVm1* VM 上，讓 ICMP 能夠通過 Windows 防火牆，您就能在稍後的步驟中使用 PowerShell，從 *myVm2* 針對此 VM 進行 Ping 操作：

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```
    
    雖然本教學課程使用 Ping 在 VM 之間進行通訊，但不建議在生產環境部署中允許 ICMP 通過 Windows 防火牆。

7. 若要連線至 myVm2 虛擬機器，請在 myVm1 虛擬機器上從命令提示字元輸入下列命令：

    ```
    mstsc /v:10.1.0.4
    ```
    
8. 由於您在 myVm1 上啟用 ping，您現在可以依據 IP 位址來 ping 它：

    ```
    ping 10.0.0.4
    ```
    
9. 同時中斷與 myVm1 和 myVm2 的 RDP 工作階段。

## <a name="clean-up-resources"></a>清除資源

當不再需要資源群組時，請將資源群組及其包含的所有資源刪除： 

1. 在入口網站頂端的 [搜尋] 方塊中，輸入 myResourceGroup。 當您在搜尋結果中看到 myResourceGroup 時，請加以選取。
2. 選取 [刪除資源群組]。
3. 針對 [輸入資源群組名稱:] 輸入 myResourceGroup，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用虛擬網路對等互連來連線相同 Azure 區域中的兩個網路。 您也可以針對不同[支援區域](virtual-network-manage-peering.md#cross-region)和[不同 Azure 訂用帳戶](create-peering-different-subscriptions.md#portal)中的虛擬網路進行對等互連，以及使用對等互連來建立[中樞和輪輻網路設計](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json#vnet-peering)。 若要深入了解虛擬網路對等互連，請參閱[虛擬網路對等互連概觀](virtual-network-peering-overview.md)和[管理虛擬網路對等互連](virtual-network-manage-peering.md)。

若要透過 VPN 將自己的電腦連線到虛擬網路，並且與虛擬網路中或已對等互連之虛擬網路中的資源進行互動，請參閱[將電腦連線至虛擬網路](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
