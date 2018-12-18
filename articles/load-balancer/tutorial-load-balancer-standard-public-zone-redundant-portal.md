---
title: 教學課程：可用性區域間的 Load Balancer VM - Azure 入口網站 | Microsoft Docs
description: 此教學課程示範如何使用 Azure 入口網站以區域備援前端建立標準負載平衡器，來平衡多可用性區域間的 VM 負載。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: As an IT administrator, I want to create a load balancer that load balances incoming internet traffic to virtual machines across availability zones in a region, so that the customers can still access the web service if a datacenter is unavailable.
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/17/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: 5ec1cc42a0c932e47c08493fa632495426abc4c7
ms.sourcegitcommit: 688a394c4901590bbcf5351f9afdf9e8f0c89505
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/18/2018
ms.locfileid: "34304455"
---
# <a name="tutorial-load-balance-vms-across-availability-zones-with-a-standard-load-balancer-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站透過標準負載平衡器來平衡可用性區域間的 VM 負載

負載平衡會將傳入要求分散到多部虛擬機器，藉此提供高可用性。 此教學課程逐步說明如何建立公用負載平衡器標準，以便平衡可用性區域間的 VM 負載。 萬一整個資料中心失敗或遺失，這有助於保護您的應用程式和資料免於受害。 透過區域備援，即便有一或多個可用性區域可能失敗，但是地區中只要有一個區域維持良好狀況，資料路徑就得以存留。 您會了解如何：

> [!div class="checklist"]
> * 建立標準負載平衡器
> * 建立網路安全性群組以定義連入流量規則
> * 建立跨多個區域的區域備援 VM 並連結至負載平衡器
> * 建立負載平衡器健康狀態探查
> * 建立負載平衡器流量規則
> * 建立基本 IIS 網站
> * 檢視作用中的負載平衡器

如需關於搭配使用可用性區域和標準 Load Balancer 的詳細資訊，請參閱[標準 Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。

如果您想要，您可以使用 [Azure CLI](load-balancer-standard-public-zone-redundant-cli.md) 完成本教學課程。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 

## <a name="sign-in-to-azure"></a>登入 Azure

在 [http://portal.azure.com](http://portal.azure.com) 登入 Azure 入口網站。

## <a name="create-a-standard-load-balancer"></a>建立標準負載平衡器

標準負載平衡器只支援標準公用 IP 位址。 當您在建立負載平衡器期間建立新的公用 IP 時，它會自動設定為標準 SKU 版本，而且也會自動具備區域備援能力。

1. 在畫面的左上方，按一下 [建立資源] > [網路] > [負載平衡器]。
2. 在 [建立負載平衡器] 頁面中輸入負載平衡器的下列值：
    - myLoadBalancer - 作為負載平衡器的名稱。
    - Public - 作為負載平衡器的類型。
     - myPublicIP - 作為您建立的新公用 IP 位址。 若要執行此動作，請依序按一下 [選擇公用 IP 位址] 和 [新建]。 名稱類型 myPublicIP 的 SKU 預設為 [標準]，而針對 [可用性區域] 請選取 [區域備援]。
    - myResourceGroupLBAZ - 作為您新建立的資源群組的名稱。
    - westeurope - 作為位置。
3. 按一下 [建立] 以建立負載平衡器。
   
    ![建立負載平衡器](./media/load-balancer-standard-public-availability-zones-portal/1a-load-balancer.png)


## <a name="create-backend-servers"></a>建立後端伺服器

本節中，您會建立一個虛擬網路、在地區的不同區域中建立虛擬機器，然後在虛擬機器上安裝 IIS，協助測試區域備援的負載平衡器。 因此，如果某個區域失敗，相同區域中的 VM 健康情況探查也失敗，其他區域中的 VM 將繼續為流量提供服務。

### <a name="create-a-virtual-network"></a>建立虛擬網路
建立虛擬網路以便部署您的後端伺服器。

1. 在畫面左上方按一下 [建立資源] > [網路] > [虛擬網路]，然後輸入虛擬網路的下列值：
    - myVnet - 作為虛擬網路的名稱。
    - myResourceGroupLBAZ - 作為現有資源群組的名稱
    - myBackendSubnet - 作為子網路名稱。
2. 按一下 [建立] 以建立虛擬網路。

    ![建立虛擬網路](./media/load-balancer-standard-public-availability-zones-portal/2-load-balancer-virtual-network.png)

## <a name="create-a-network-security-group"></a>建立網路安全性群組

建立網路安全性群組，以定義虛擬網路的輸入連線。

1. 在畫面左上方按一下 [建立資源]，在搜尋方塊中輸入 *Network Security Group*，然後在網路安全性群組頁面上按一下 [建立]。
2. 在 [建立網路安全性群組] 頁面中，輸入下列值：
    - myNetworkSecurityGroup - 作為網路安全性群組的名稱。
    - myResourceGroupLBAZ - 作為現有資源群組的名稱。
   
![建立虛擬網路](./media/load-balancer-standard-public-availability-zones-portal/create-nsg.png)

### <a name="create-network-security-group-rules"></a>建立網路安全性群組規則

在本節中，您會建立網路安全性群組規則，以允許在 Azure 入口網站中使用 HTTP 與 RDP 進行輸入連線。

1. 在 Azure 入口網站中按一下左側功能表中的 [所有資源]，然後搜尋並按一下位於 **myResourceGroupLBAZ** 資源群組中的 **myNetworkSecurityGroup**。
2. 在 [設定] 底下，按一下 [輸入安全性規則]，然後按一下 [新增]。
3. 輸入輸入安全性規則 (名為 myHTTPRule) 的下列值，以允許使用連接埠 80 的輸入 HTTP 連線：
    - 服務標記 - 作為 [來源]。
    - 網際網路 - 作為 [來源服務標記]
    - 80 - 作為 [目的地連接埠範圍]
    - TCP - 作為 [通訊協定]
    - 允許 - 作為 [動作]
    - 100 作為 [優先順序]
    - myHTTPRule - 負載平衡器規則的名稱。
    - 允許 HTTP - 負載平衡器規則的描述。
4. 按一下 [確定]。
 
 ![建立虛擬網路](./media/load-balancer-standard-public-availability-zones-portal/8-load-balancer-nsg-rules.png)
5. 重複步驟 2 到 4 來建立另一個名為 myRDPRule 的規則，以允許使用連接埠 3389 的輸入 RDP 連線，其具有下列值：
    - 服務標記 - 作為 [來源]。
    - 網際網路 - 作為 [來源服務標記]
    - 3389 - 作為 [目的地連接埠範圍]
    - TCP - 作為 [通訊協定]
    - 允許 - 作為 [動作]
    - 200 作為 [優先順序]
    - myRDPRule 作為名稱
    - 允許 RDP - 作為描述

### <a name="create-virtual-machines"></a>建立虛擬機器

在地區的不同區域 (區域 1、區域 2 和區域 3) 中建立虛擬機器，以作為負載平衡器的後端伺服器。

1. 在畫面左上方按一下 [建立資源] > [計算] > [Windows Server 2016 Datacenter]，然後輸入虛擬網路的下列值：
    - myVM1 - 作為虛擬機器的名稱。        
    - azureuser - 作為系統管理員使用者名稱。    
    - myResourceGroupLBAZ - 針對 [資源群組]，選取 [使用現有的]，然後選取 myResourceGroupLBAZ。
2. 按一下 [確定]。
3. 選取 [DS1_V2] 作為虛擬機器的大小，然後按一下 [選取]。
4. 輸入 VM 設定的下列值：
    - 區域 1 - VM 放置所在的區域。
    -  myVNet - 確保已選取作為虛擬網路。
    - myBackendSubnet - 確保已選取作為子網路。
    - myNetworkSecurityGroup - 作為網路安全性群組 (防火牆) 的名稱。
5. 按一下 [停用] 來停用開機診斷。
6. 按一下 [確定]，檢閱 [摘要] 頁面上的設定，然後按一下 [建立]。
  
 ![建立虛擬機器](./media/load-balancer-standard-public-availability-zones-portal/create-vm-standard-ip.png)

7. 使用步驟 1-6，在區域 2 中建立名為 VM2 的第二個 VM，並在區域 3 中建立第三個 VM，且其虛擬網路為 myVnet、子網路為 myBackendSubnet，而網路安全性群組為 *myNetworkSecurityGroup。

### <a name="install-iis-on-vms"></a>在 VM 上安裝 IIS

1. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下位於 myResourceGroupLB 資源群組中的 **myVM1**。
2. 在 [概觀] 頁面上，按一下 [連線] 以透過 RDP 連入 VM。
3. 以使用者名稱 azureuser 登入 VM。
4. 在伺服器桌面上，瀏覽至 [Windows 系統管理工具]>[Windows PowerShell]。
5. 在 PowerShell 視窗中，執行下列命令以安裝 IIS 伺服器、移除預設 iisstart.htm 檔案，然後新增會顯示 VM 名稱的 iisstart.htm 檔案：
   ```azurepowershell-interactive
    
    # install IIS server role
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    # Add a new htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from" + $env:computername)
   ```
6. 使用 myVM1 關閉 RDP 工作階段。
7. 重複步驟 1 到 6，在 myVM2 和 myVM3 上安裝 IIS 和更新的 iisstart.htm 檔案。

## <a name="create-load-balancer-resources"></a>建立負載平衡器資源

在本節中，您會設定後端位址集區和健康狀態探查的負載平衡器設定，並指定負載平衡器和 NAT 規則。


### <a name="create-a-backend-address-pool"></a>建立後端位址集區

若要將流量分散至 VM，後端位址集區包含已連線至負載平衡器之虛擬 (NIC) 的 IP 位址。 建立後端位址集區 myBackendPool 以納入 VM1、VM2 和 VM3。

1. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下 [myLoadBalancer]。
2. 在 [設定] 之下，依序按一下 [後端集區] 和 [新增]。
3. 在 [新增後端集區] 頁面上，執行下列操作：
    - 針對名稱，輸入 myBackEndPool，作為您後端集區的名稱。
    - 針對 [虛擬網路]，在下拉式功能表中按一下[myVNet]
    - 針對 [虛擬機器]，在下拉式功能表中按一下[myVM1]。
    - 針對 [IP 位址]，在下拉式功能表中按一下 myVM1 的 IP 位址。
4. 按一下 [新增後端資源]，將每個虛擬機器 (myVM2 和 myVM3) 新增至負載平衡器的後端集區。
5. 按一下 [新增] 。

    ![新增至後端位址集區 ](./media/load-balancer-standard-public-availability-zones-portal/add-backend-pool.png)

3. 確認您的負載平衡器後端集區設定將三個 VM 全都顯示出來 - **myVM1**、**myVM2** 和 **myVM3**。

### <a name="create-a-health-probe"></a>建立健康狀態探查

若要讓負載平衡器監視您應用程式的狀態，請使用健康狀態探查。 健康狀態探查會根據 VM 對健康狀態檢查的回應，以動態方式從負載平衡器輪替中新增或移除 VM。 建立健康狀態探查 myHealthProbe 以監視 VM 的健康狀態。

1. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下 [myLoadBalancer]。
2. 在 [設定] 之下，依序按一下 [健康狀態探查] 和 [新增]。
3. 使用下列值來建立健康狀態探查：
    - myHealthProbe - 作為健康狀態探查的名稱。
    - HTTP - 作為通訊協定類型。
    - 80 - 作為連接埠號碼。
    - 15 - 探查嘗試的 [間隔] 秒數。
    - 2 - 作為 [狀況不良閾值]，或將 VM 視為狀況不良之前，必須達到的連續探查失敗次數。
4. 按一下 [確定]。

   ![新增探查](./media/load-balancer-standard-public-availability-zones-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>建立負載平衡器規則

負載平衡器規則用來定義如何將流量分散至 VM。 您可定義連入流量的前端 IP 組態及後端 IP 集區來接收流量，以及所需的來源和目的地連接埠。 建立負載平衡器規則 myLoadBalancerRuleWeb，用來接聽前端 FrontendLoadBalancer 中的連接埠 80，以及用來將負載平衡的網路流量傳送到後端位址集區 myBackEndPool (也是使用連接埠 80)。 

1. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下 [myLoadBalancer]。
2. 在 [設定] 之下，按一下 [負載平衡規則]，然後按一下 [新增]。
3. 使用下列值來設定負載平衡規則：
    - myHTTPRule - 作為負載平衡器規則的名稱。
    - TCP - 作為通訊協定類型。
    - 80 - 作為連接埠號碼。
    - 80 - 作為後端連接埠。
    - myBackendPool - 作為後端集區的名稱。
    - myHealthProbe - 作為健康狀態探查的名稱。
4. 按一下 [確定]。
    
    ![新增負載平衡規則](./media/load-balancer-standard-public-availability-zones-portal/load-balancing-rule.png)

## <a name="test-the-load-balancer"></a>測試負載平衡器
1. 在 [概觀] 畫面上尋找負載平衡器的公用 IP 位址。 按一下 [所有資源]，然後按一下 [myPublicIP]。

2. 將公用 IP 位址複製並貼到您瀏覽器的網址列。 IIS Web 伺服器的預設頁面會顯示在瀏覽器上。

      ![IIS Web 伺服器](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)

若要查看負載平衡器如何將流量分散到整個區域中的 VM，您可以強制重新整理您的網頁瀏覽器。

## <a name="clean-up-resources"></a>清除資源

若不再需要，可刪除資源群組、負載平衡器和所有相關資源。 若要這樣做，請選取包含負載平衡器的資源群組，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

深入了解[標準負載平衡器](load-balancer-standard-overview.md)。
