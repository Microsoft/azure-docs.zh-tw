---
title: 路由網路流量 - 教學課程 - Azure 入口網站 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure 入口網站以路由表路由網路流量。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: 81478ace72a538f4970e114cd704fd64ceb94aa6
ms.sourcegitcommit: 756f866be058a8223332d91c86139eb7edea80cc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37344883"
---
# <a name="tutorial-route-network-traffic-with-a-route-table-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站以路由表路由網路流量

根據預設，Azure 會自動路由傳送虛擬網路內所有子網路之間的流量。 您可以建立您自己的路由，以覆寫 Azure 的預設路由。 舉例來說，如果您想要通過網路虛擬設備 (NVA) 路由傳送子網路之間的流量，則建立自訂路由的能力很有幫助。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立路由表
> * 建立路由
> * 建立有多個子網路的虛擬網路
> * 建立路由表與子網路的關聯
> * 建立會路由傳送流量的 NVA
> * 將虛擬機器 (VM) 部署到不同子網路
> * 透過 NVA 從一個子網路將流量路由傳送到另一個子網路

您可以依偏好使用 [Azure CLI](tutorial-create-route-table-cli.md) 或 [Azure PowerShell](tutorial-create-route-table-powershell.md) 完成本教學課程。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="log-in-to-azure"></a>登入 Azure 

在 http://portal.azure.com 上登入 Azure 入口網站。

## <a name="create-a-route-table"></a>建立路由表

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [網路]，然後選取 [路由表]。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    |設定|值|
    |---|---|
    |Name|myRouteTablePublic|
    |訂用帳戶| 選取您的訂用帳戶。|
    |資源群組 | 選取 [新建]，然後輸入 *myResourceGroup*。|
    |位置|美國東部|
 
    ![建立路由表](./media/tutorial-create-route-table-portal/create-route-table.png) 

## <a name="create-a-route"></a>建立路由

1. 在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，開始輸入 myRouteTablePublic。 當 **myRouteTablePublic** 出現在搜尋結果時，選取它。
2. 選取 [設定] 底下的 [路由]，然後選取 [+新增]，如下圖所示：

    ![新增路由](./media/tutorial-create-route-table-portal/add-route.png) 
 
3. 在 [新增路由] 底下，輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    |設定|值|
    |---|---|
    |路由名稱|ToPrivateSubnet|
    |位址首碼| 10.0.1.0/24|
    |下一個躍點類型 | 選取 [虛擬設備]。|
    |下一個躍點位址| 10.0.2.4|

## <a name="associate-a-route-table-to-a-subnet"></a>建立路由表與子網路的關聯

在您可以讓路由表與子網路產生關聯之前，您必須先建立虛擬網路和子網路，然後才可以讓路由表與子網路產生關聯：

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [網絡]，然後選取 [虛擬網路]。
3. 在 [建立虛擬網路] 底下，輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    |設定|值|
    |---|---|
    |Name|myVirtualNetwork|
    |位址空間| 10.0.0.0/16|
    |訂用帳戶 | 選取您的訂用帳戶。|
    |資源群組|選取 [使用現有的]，然後選取 [myResourceGroup]。|
    |位置|選取 [美國東部]|
    |子網路名稱|公開|
    |位址範圍|10.0.0.0/24|
    
4. 在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，開始輸入 myVirtualNetwork。 當搜尋結果中出現 **myVirtualNetwork** 時加以選取。
5. 在 [設定] 底下選取 [子網路]，然後選取 [+ 子網路]，如下圖所示：

    ![新增子網路](./media/tutorial-create-route-table-portal/add-subnet.png) 

6. 選取或輸入下列資訊，然後選取 [確定]︰

    |設定|值|
    |---|---|
    |Name|私人|
    |位址空間| 10.0.1.0/24|

7. 再次完成步驟 5 和 6，以提供下列資訊：

    |設定|值|
    |---|---|
    |Name|DMZ|
    |位址空間| 10.0.2.0/24|

8. [myVirtualNetwork - 子網路] 方塊會在完成上一個步驟之後顯示。 在 [設定] 底下選取 [子網路]，然後選取 [公用]。
9. 如下圖所示，依序選取 [路由表]、[MyRouteTablePublic]，然後選取 [儲存]：

    ![關聯路由表](./media/tutorial-create-route-table-portal/associate-route-table.png) 

## <a name="create-an-nva"></a>建立 NVA

NVA 是會執行網路功能的虛擬機器，例如路由、防火牆或 WAN 最佳化。

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [計算]，然後選取 [Windows Server 2016 Datacenter]。 您可以選取不同的作業系統，但是其餘步驟假設您選取的是 **Windows Server 2016 Datacenter**。 
3. 針對 [基本] 選取或輸入下列資訊，然後選取 [確定]︰

    |設定|值|
    |---|---|
    |Name|myVmNva|
    |使用者名稱|輸入您選擇的使用者名稱。|
    |密碼|輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |資源群組| 選取 [使用現有的]，然後選取 [myResourceGroup]。|
    |位置|選取 [美國東部]。|
4. 在 [選擇大小] 底下選取虛擬機器大小。
5. 針對 [設定] 選取或輸入下列資訊，然後選取 [確定]︰

    |設定|值|
    |---|---|
    |虛擬網路|myVirtualNetwork - 若未選取，請選取 [虛擬網路]，然後在 [選擇虛擬網路] 底下選取 [myVirtualNetwork]。|
    |子網路|選取 [子網路]，然後在 [選擇子網路] 底下選取 [DMZ]。 |
    |公用 IP 位址| 選取 [公用 IP 位址]，然後在 [選擇公用 IP 位址] 底下選取 [無]。 由於不會從網際網路連線至此 VM，因此不會為其指派公用 IP 位址。
6. 在 [摘要] 的 [建立] 底下，選取 [建立] 來開始部署虛擬機器。

    建立 VM 需要幾分鐘的時間。 在 Azure 完成虛擬機器建立並開啟具有虛擬機器相關資訊的方塊之前，請勿繼續進行下一個步驟。

7. 在建立虛擬機器之後開啟的方塊中，選取 [設定] 底下的 [網路]，然後選取 [myvmnva158] (Azure 為您的虛擬機器建立的網路介面在 **myvmnva** 後面有不同的數字)，如下圖所示：

    ![VM 網路功能](./media/tutorial-create-route-table-portal/virtual-machine-networking.png) 

8. 若要讓網路介面能夠將傳送給它的網路流量轉送 (目的地不是它自己的 IP 位址)，則必須對網路介面啟用 IP 轉送。 在 [設定] 底下選取 [IP 組態]，針對 [IP 轉送] 選取 [已啟用]，然後選取 [儲存]，如下圖所示：

    ![啟用 IP 轉送](./media/tutorial-create-route-table-portal/enable-ip-forwarding.png) 

## <a name="create-virtual-machines"></a>建立虛擬機器

在虛擬網路中建立兩部虛擬機器，您就可以在後續步驟中驗證來自「公用」子網路的流量是否透過 NVA 路由傳送至「私人」子網路。 完成[建立 NVA](#create-a-network-virtual-appliance) 的步驟 1-6。 在步驟 3 和 5 中使用相同的設定，下列變更除外：

|虛擬機器名稱      |子網路      | 公用 IP 位址     |
|--------- | -----------|---------              |
| myVmPublic  | 公開     | 接受入口網站預設值 |
| myVmPrivate | 私人    | 接受入口網站預設值 |

您可以在 Azure 建立 myVmPublic 虛擬機器的時候建立 myVmPrivate 虛擬機器。 在 Azure 完成兩部虛擬機器建立之前，請勿繼續進行後續步驟。

## <a name="route-traffic-through-an-nva"></a>透過 NVA 路由傳送流量

1. 在入口網站頂端的 [搜尋] 方塊中，開始輸入 myVmPrivate。 當 **myVmPrivate** 虛擬機器出現在搜尋結果中時，請加以選取。
2. 藉由選取 [連線] 來建立與 myVmPrivate 虛擬機器的遠端桌面連線，如下圖所示：

    ![連接到 VM ](./media/tutorial-create-route-table-portal/connect-to-virtual-machine.png)  

3. 若要連線至虛擬機器，請開啟所下載的 RDP 檔案。 如果出現提示，請選取 [連接]。
4. 輸入您在建立虛擬機器時指定的使用者名稱和密碼 (您可能需要選取 [更多選擇]，然後選取 [使用不同的帳戶] 以指定您在建立虛擬機器時輸入的認證)，然後選取 [確定]。
5. 您可能會在登入過程中收到憑證警告。 選取 [是] 以繼續進行連線。
6. 在後續步驟中，我們將使用追蹤路由工具來測試路由。 追蹤路由會使用網際網路控制訊息通訊協定 (ICMP)，它在通過 Windows 防火牆時會遭到拒絕。 從 myVmPrivate VM 上的 PowerShell 中輸入下列命令，讓 ICMP 通過 Windows 防火牆：

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```

    雖然本教學課程使用追蹤路由來測試路由，但不建議在生產環境部署中允許 ICMP 通過 Windows 防火牆。
7. 您已在[啟用 IP 轉送](#enable-ip-forwarding)中針對虛擬機器的網路介面啟用在 Azure 內 IP 轉送。 在虛擬機器內，作業系統或在虛擬機器內執行的應用程式也必須能夠轉送網路流量。 在 myVmNva VM 的作業系統內啟用 IP 轉送：

    從 myVmPrivate VM 上的命令提示字元，使用遠端桌面連線至 myVmNva VM：

    ``` 
    mstsc /v:myvmnva
    ```
    
    若要在作業系統內啟用 IP 轉送，請從 myVmNva VM 上的 PowerShell 輸入下列命令：

    ```powershell
    Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -Name IpEnableRouter -Value 1
    ```
    
    重新啟動 myVmNva VM，也會中斷與遠端桌面工作階段的連線。
8. 在連線至 myVmPrivate VM 的狀態下，在 myVmNva VM 重新啟動後建立 myVmPublic VM 的遠端桌面工作階段：

    ``` 
    mstsc /v:myVmPublic
    ```
    
    從 myVmPublic VM 上的 PowerShell 中輸入下列命令，讓 ICMP 通過 Windows 防火牆：

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```

9. 若要測試從 myVmPublic VM 到 myVmPrivate VM 的網路流量路由，請在 myVmPublic VM 上的 PowerShell 中輸入下列命令：

    ```
    tracert myVmPrivate
    ```

    回應如下列範例所示：
    
    ```
    Tracing route to myVmPrivate.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.1.4]
    over a maximum of 30 hops:
        
    1    <1 ms     *        1 ms  10.0.2.4
    2     1 ms     1 ms     1 ms  10.0.1.4
        
    Trace complete.
    ```
      
    您可以看到第一個躍點是 10.0.2.4，也就是 NVA 的私人 IP 位址。 第二躍點是 10.0.1.4，也就是 myVmPrivate 虛擬機器的私人 IP 位址。 新增至 myRouteTablePublic 路由表且與「公用」子網路產生關聯的路由，會導致 Azure 透過 NVA 路由傳送流量，而不是直接路由傳送到「私人」子網路。
10.  關閉 myVmPublic 虛擬機器的遠端桌面工作階段，但您仍然與 myVmPrivate 虛擬機器連線。
11. 若要測試從 myVmPrivate VM 到 myVmPublic VM 的網路流量路由，請在 myVmPrivate VM 上的命令提示字元中輸入下列命令：

    ```
    tracert myVmPublic
    ```

    回應如下列範例所示：

    ```
    Tracing route to myVmPublic.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.0.4]
    over a maximum of 30 hops:
    
    1     1 ms     1 ms     1 ms  10.0.0.4
    
    Trace complete.
    ```

    您可以看到流量是直接從 myVmPrivate 虛擬機器直接路由傳送到 myVmPublic 虛擬機器。 根據預設，Azure 會直接路由傳送子網路之間的流量。
12. 關閉 myVmPrivate 虛擬機器的遠端桌面工作階段。

## <a name="clean-up-resources"></a>清除資源

當不再需要資源群組時，請將資源群組及其包含的所有資源刪除： 

1. 在入口網站頂端的 [搜尋] 方塊中，輸入 myResourceGroup。 當您在搜尋結果中看到 myResourceGroup 時，請加以選取。
2. 選取 [刪除資源群組]。
3. 針對 [輸入資源群組名稱:] 輸入 myResourceGroup，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立路由表，並將其與子網路產生關聯。 您已建立簡單的 NVA，它會將來自公用子網路的流量路由傳送至私人子網路。 從 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking) 部署各種預先設定的 NVA，這些設備會執行例如防火牆和 WAN 最佳化的網路功能。 若要深入了解路由，請參閱[路由概觀](virtual-networks-udr-overview.md)和[管理路由表](manage-route-table.md)。


雖然您可以在虛擬網路內部署許多 Azure 資源，但是某些 Azure PaaS 服務的資源無法部署到虛擬網路中。 您仍可將某些 Azure PaaS 服務的資源存取，限制為僅來自虛擬網路子網路的流量。 若要了解如何限制對 Azure PaaS 資源的網路存取，請繼續進行下一個教學課程。

> [!div class="nextstepaction"]
> [限制對 PaaS 資源的網路存取](tutorial-restrict-network-access-to-resources.md)
