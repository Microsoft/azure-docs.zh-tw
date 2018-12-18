---
title: 教學課程：區域內的 Load Balancer VM--Azure 入口網站 | Microsoft Docs
description: 本教學課程示範如何使用 Azure 入口網站以區域前端建立 Standard Load Balancer，以平衡可用性區域內的 VM 負載
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: As an IT administrator, I want to create a load balancer that load balances incoming internet traffic to virtual machines within a specific zone in a region.
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/17/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: 580015b7f8b1f894c69ddec0f26daeb524932e4b
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34637288"
---
# <a name="tutorial-load-balance-vms-within-an-availability-zone-with-standard-load-balancer-by-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站透過 Standard Load Balancer 來平衡可用性區域內的 VM 負載

本教學課程會透過 Azure 入口網站，使用標準公用 IP 位址來建立具有區域前端的公用 [Azure Standard Load Balancer 執行個體](https://aka.ms/azureloadbalancerstandard)。 在此案例中，您需為前端和後端執行個體指定特定區域，以讓您的資料路徑和資源與特定區域搭配運作。 您會了解如何執行下列功能：

> [!div class="checklist"]
> * 建立具有區域前端的 Standard Load Balancer 執行個體。
> * 建立網路安全性群組以定義連入流量規則。
> * 建立區域虛擬機器 (VM) 並連結至負載平衡器。
> * 建立負載平衡器健全狀況探查。
> * 建立負載平衡器流量規則。
> * 建立基本網際網路資訊服務 (IIS) 網站。
> * 檢視作用中的負載平衡器。

如需關於搭配使用可用性區域和 Standard Load Balancer 的詳細資訊，請參閱 [Standard Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。

如果您想要，請使用 [Azure CLI](load-balancer-standard-public-zonal-cli.md) 完成本教學課程。

## <a name="sign-in-to-azure"></a>登入 Azure

在 [http://portal.azure.com](http://portal.azure.com) 登入 Azure 入口網站。

## <a name="create-a-public-standard-load-balancer-instance"></a>建立公用 Standard Load Balancer 執行個體

Standard Load Balancer 只支援標準公用 IP 位址。 當您在建立負載平衡器期間建立新的公用 IP 時，它會自動設定為標準 SKU 版本。 而且也會自動具備區域備援能力。

1. 在畫面的左上方，選取 [建立資源] > [網路] > [負載平衡器]。
2. 在 [建立負載平衡器] 頁面中，輸入負載平衡器的下列值：
    - [myLoadBalancer] 作為負載平衡器的名稱。
    - [公用] 作為負載平衡器的類型。
     - [myPublicIPZonal] 作為您建立的新公用 IP 位址。 選取 [選擇公用 IP 位址] 。 然後選取 [建立新的] 。 針對名稱，輸入 **myPublicIP**。 SKU 預設為 [標準]。 針對 [可用性區域]，選取 [區域 1]。
    - [myResourceGroupZLB] 作為您新建立的資源群組的名稱。
    - [westeurope] 作為位置。
3. 選取 [建立] 以建立負載平衡器。
   
    ![使用 Azure 入口網站建立區域 Standard Load Balancer 執行個體](./media/tutorial-load-balancer-standard-zonal-portal/create-load-balancer-zonal-frontend.png)


## <a name="create-backend-servers"></a>建立後端伺服器

在本節中，您會建立虛擬網路。 您也會針對要新增至負載平衡器後端集區的地區，在相同區域 (也就是區域 1) 中建立兩部虛擬機器。 然後在虛擬機器上安裝 IIS，協助測試區域備援負載平衡器。 如果有一個 VM 失敗，則相同區域中 VM 的健康狀態探查會失敗。 相同區域內的其他 VM 會繼續提供流量。

### <a name="create-a-virtual-network"></a>建立虛擬網路
1. 在畫面的左上方，選取 [建立資源] > [網路] > [虛擬網路]。  針對虛擬網路輸入下列值：
    - [myVnet] 作為虛擬網路的名稱。
    - [myResourceGroupZLB] 作為現有資源群組的名稱。
    - [myBackendSubnet] 作為子網路名稱。
2. 選取 [建立] 以建立虛擬網路。

    ![建立虛擬網路](./media/tutorial-load-balancer-standard-zonal-portal/create-virtual-network.png)

## <a name="create-a-network-security-group"></a>建立網路安全性群組

1. 在畫面的左上方，選取 [建立資源]。 在搜尋方塊中，輸入「網路安全性群組」。 在 [網路安全性群組] 頁面中，選取 [建立]。
2. 在 [建立網路安全性群組] 頁面中，輸入下列值：
    - [myNetworkSecurityGroup] 作為網路安全性群組的名稱。
    - [myResourceGroupLBAZ] 作為現有資源群組的名稱。
   
    ![建立網路安全性群組](./media/tutorial-load-balancer-standard-zonal-portal/create-network-security-group.png)

### <a name="create-nsg-rules"></a>建立 NSG 規則

在本節中，您會建立 NSG 規則，允許在 Azure 入口網站中使用 HTTP 與 Microsoft 遠端桌面通訊協定 (RDP) 進行輸入連線。

1. 在 Azure 入口網站中，選取最左邊功能表中的 [所有資源]。 然後搜尋並選取 [myNetworkSecurityGroup]。 它位於 [myResourceGroupZLB] 資源群組中。
2. 在 [設定] 下，選取 [輸入安全性規則]。 然後選取 [新增]。
3. 針對名為 **myHTTPRule** 的輸入安全性規則輸入下列值，以允許使用連接埠 80 的輸入 HTTP 連線：
    - [服務標記] 作為 [來源]。
    - [網際網路] 作為 [來源服務標記]。
    - [80] 作為 [目的地連接埠範圍]。
    - [vTCP] 作為 [通訊協定]。
    - [允許] 作為 [動作]。
    - [100] 作為 [優先順序]。
    - [myHTTPRule] 作為 [名稱]。
    - [允許 HTTP] 作為 [描述]。
4. 選取 [確定] 。
 
 ![建立 NSG 規則](./media/load-balancer-standard-public-availability-zones-portal/8-load-balancer-nsg-rules.png)

5. 重複步驟 2 到 4，以建立名為 [myRDPRule] 的另一個規則。 此規則允許使用連接埠 3389 的輸入 RDP 連線，採用下列值：
    - [服務標記] 作為 [來源]。
    - [網際網路] 作為 [來源服務標記]。
    - [3389] 作為 [目的地連接埠範圍]。
    - [TCP] 作為 [通訊協定]。
    - [允許] 作為 [動作]。
    - [200] 作為 [優先順序]。
    - [myRDPRule] 作為 [名稱]。
    - [允許 RDP] 作為 [描述]。

    ![建立 RDP 規則](./media/tutorial-load-balancer-standard-zonal-portal/create-rdp-rule.png)

### <a name="create-virtual-machines"></a>建立虛擬機器

1. 在畫面的左上方，選取 [建立資源] > **[計算]** > [Windows Server 2016 Datacenter]。 針對虛擬機器，請輸入這些值：
    - [myVM1] 作為虛擬機器的名稱。        
    - [azureuser] 作為系統管理員使用者名稱。    
    - [myResourceGroupZLB] 作為 [資源群組]。 選取 [使用現有的]，然後選取 [myResourceGroupZLB]。
2. 選取 [確定] 。
3. 選取 [DS1_V2] 作為虛擬機器的大小。 選擇 [選取]。
4. 輸入 VM 設定的下列值：
    - [區域 1] 作為您放置 VM 的可用性區域。
    -  **myVNet**。 確保已選取它作為虛擬網路。
    - [myVM1PIP] 作為您建立的標準公用 IP 位址。 選取 [建立新的]。 然後針對名稱類型，選取 [myVM1PIP]。 針對 [區域]，選取 **1**。 IP 位址 SKU 預設為 [標準]。
    - **myBackendSubnet**。 確保已選取它作為子網路。
    - [myNetworkSecurityGroup] 作為現存網路安全性群組防火牆的名稱。
5. 選取 [停用] 來停用開機診斷。
6. 選取 [確定] 。 在摘要頁面上檢閱設定。 然後選取 [建立]。
7. 重複步驟 1 到 6，在區域 1 中建立名為 **myVM2** 的第二個 VM。 [myVnet] 作為虛擬網路。 [myVM2PIP] 作為標準公用 IP 位址。 [myBackendSubnet] 作為子網路。 [myNetworkSecurityGroup] 作為網路安全性群組。

    ![建立虛擬機器](./media/tutorial-load-balancer-standard-zonal-portal/create-virtual-machine.png) 

### <a name="install-iis-on-vms"></a>在 VM 上安裝 IIS

1. 選取最左邊功能表中的 [所有資源]。 然後從 [資源] 清單中，選取 [myVM1]。 它位於 [myResourceGroupZLB] 資源群組中。
2. 在 [概觀] 頁面上，選取 [連線] 以使用 RDP 移至 VM。
3. 使用您建立 VM 時指定的使用者名稱和密碼來登入 VM。 若要指定您在建立 VM 時輸入的認證，您可能需要選取 [其他選擇]。 然後選取 [使用不同的帳戶]。 然後選取 [確定]。 您可能會在登入過程中收到憑證警告。 選取 [是] 以繼續進行連線。
4. 在伺服器桌面上，瀏覽至 [Windows 系統管理工具] > [Windows PowerShell]。
6. 在 [PowerShell] 視窗中，執行下列命令來執行 IIS 伺服器。 這些命令也會移除預設 iisstart.htm 檔案，然後新增可顯示 VM 名稱的新 iisstart.htm 檔案：

   ```azurepowershell-interactive
    # install IIS server role
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    # remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    # Add a new htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from" + $env:computername)
   ```
7. 使用 myVM1 關閉 RDP 工作階段。
8. 重複步驟 1 到 7，在 myVM2 上安裝 IIS。

## <a name="create-load-balancer-resources"></a>建立負載平衡器資源

在本節中，您會設定後端位址集區和健康狀態探查的負載平衡器設定。 您也會指定負載平衡器和網路位址轉譯規則。


### <a name="create-a-backend-address-pool"></a>建立後端位址集區

若要將流量分散至 VM，後端位址集區包含已連線至負載平衡器的虛擬網路介面卡 IP 位址。 建立後端位址集區 myBackendPool 以納入 VM1 和 VM2。

1. 選取最左邊功能表中的 [所有資源]。 然後從 [資源] 清單中選取 [myLoadBalancer]。
2. 在 [設定] 底下選取 [後端集區]。 然後選取 [新增]。
3. 在 [新增後端集區] 頁面上，採取下列動作：
    - 針對名稱，輸入 myBackEndPool，作為您後端集區的名稱。
    - 針對 [虛擬網路]，在下拉式功能表中選取 [myVNet]。 
    - 針對 [虛擬機器] 和 [IP 位址]，新增 myVM1 和 myVM2 以及其對應的公用 IP 位址。
4. 選取 [新增] 。
5. 請檢查以確定負載平衡器後端集區設定顯示 **myVM1** 和 **myVM2** 這兩個 VM。
 
    ![建立後端集區](./media/tutorial-load-balancer-standard-zonal-portal/create-backend-pool.png) 

### <a name="create-a-health-probe"></a>建立健康狀態探查

使用健康狀態探查，所以負載平衡器可以監視應用程式的狀態。 健康狀態探查會根據 VM 對健康狀態檢查的回應，以動態方式從負載平衡器輪替中新增或移除 VM。 建立健康狀態探查 myHealthProbe 以監視 VM 的健康狀態。

1. 選取最左邊功能表中的 [所有資源]。 然後從 [資源] 清單中選取 [myLoadBalancer]。
2. 在 [設定] 下，選取 [健康狀態探查]。 然後選取 [新增]。
3. 使用下列值來建立健康狀態探查：
    - [myHealthProbe] 作為健康狀態探查的名稱。
    - [HTTP] 作為通訊協定類型。
    - [80] 作為連接埠號碼。
    - [15] 作為探查嘗試之間的 [間隔] 秒數。
    - [2] 作為 [狀況不良閾值]，或將 VM 視為狀況不良之前，必須達到的連續探查失敗次數。
4. 選取 [確定] 。

   ![新增健康狀態探查](./media/load-balancer-standard-public-availability-zones-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>建立負載平衡器規則

負載平衡器規則可定義如何將流量分散至 VM。 您可定義連入流量的前端 IP 組態及後端 IP 集區來接收流量，以及所需的來源和目的地連接埠。 建立 **myLoadBalancerRuleWeb** 負載平衡器規則，以便接聽前端 **FrontendLoadBalancer** 中的連接埠 80。 此規則會將負載平衡的網路流量傳送到後端位址集區 **myBackEndPool**，也可藉由使用連接埠 80。 

1. 選取最左邊功能表中的 [所有資源]。 然後從 [資源] 清單中選取 [myLoadBalancer]。
2. 在 [設定] 下，選取 [負載平衡規則]。 然後選取 [新增]。
3. 使用下列值來設定負載平衡規則：
    - [myHTTPRule] 作為負載平衡規則的名稱。
    - [TCP] 作為通訊協定類型。
    - [80] 作為連接埠號碼。
    - [80] 作為後端連接埠。
    - [myBackendPool] 作為後端集區的名稱。
    - [myHealthProbe] 作為健康狀態探查的名稱。
4. 選取 [確定] 。
    
    ![新增負載平衡規則](./media/tutorial-load-balancer-standard-zonal-portal/load-balancing-rule.png)

## <a name="test-the-load-balancer"></a>測試負載平衡器
1. 在 [概觀] 畫面上尋找負載平衡器的公用 IP 位址。 選取 [所有資源]。 然後選取 [myPublicIP]。 

2. 複製公用 IP 位址。 然後將它貼在瀏覽器的網址列中。 瀏覽器會顯示包含 Web 伺服器頁面名稱的預設頁面。

      ![IIS 網頁伺服器](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)
3. 若要查看作用中的負載平衡器，請強制停止所顯示的 VM。 重新整理瀏覽器，以查看瀏覽器上顯示的其他伺服器名稱。

## <a name="clean-up-resources"></a>清除資源

若不再需要，可刪除資源群組、負載平衡器和所有相關資源。 選取包含負載平衡器的資源群組。 然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

- 深入了解[標準負載平衡器](load-balancer-standard-overview.md)。
- [跨越多個可用性區域為虛擬機器進行負載平衡](tutorial-load-balancer-standard-public-zone-redundant-portal.md)。
