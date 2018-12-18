---
title: 教學課程：建立公用基本負載平衡器 - Azure 入口網站 | Microsoft Docs
description: 本教學課程會示範如何使用 Azure 入口網站建立內部的基本負載平衡器。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: As an IT administrator, I want to create a load balancer that load balances incoming internet traffic to virtual machines within a specific zone in a region.
ms.assetid: aa9d26ca-3d8a-4a99-83b7-c410dd20b9d0
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/28/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: c0d19c53a0bd217935a494dfb4affbaa85062247
ms.sourcegitcommit: d7725f1f20c534c102021aa4feaea7fc0d257609
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37097473"
---
# <a name="tutorial-load-balance-internal-traffic-with-basic-load-balancer-to-vms-using-the-azure-portal"></a>教學課程：在 Azure 入口網站中使用基本負載平衡器，將內部流量負載平衡到 VM

負載平衡會將傳入要求分散於多部虛擬機器，藉此提供高可用性和範圍。 您可以在 Azure 入口網站中使用基本負載平衡器，將內部流量負載平衡到虛擬機器。 本教學課程示範如何建立網路資源、後端伺服器和內部基本負載平衡器。

您可以依偏好使用 [Azure CLI](load-balancer-get-started-ilb-arm-cli.md) 或 [Azure PowerShell](load-balancer-get-started-ilb-arm-ps.md) 完成本教學課程。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。

## <a name="create-a-virtual-network"></a>建立虛擬網路
1. 在螢幕的左上方，按一下 [新增] > [網路] > [虛擬網路]，然後輸入虛擬網路的下列值：
    - myVnet - 作為虛擬網路的名稱。
    - myResourceGroupILB - 作為現有資源群組的名稱
    - myBackendSubnet - 作為子網路名稱。
2. 按一下 [建立] 以建立虛擬網路。

![建立負載平衡器](./media/tutorial-load-balancer-basic-internal-portal/1-load-balancer.png)

## <a name="create-a-basic-load-balancer"></a>建立基本負載平衡器
使用入口網站建立內部基本負載平衡器。

1. 在畫面的左上方，按一下 [建立資源] > [網路] > [負載平衡器]。
2. 在 [建立負載平衡器] 頁面中輸入負載平衡器的下列值：
    - myLoadBalancer - 作為負載平衡器的名稱。
    - **內部** - 作為負載平衡器的類型。
    - **基本** - 作為 SKU 版本。
    - **10.1.0.7** - 作為靜態私人 IP 位址。
    - myVNet - 您從現有網路清單中選擇的虛擬網路。
    - mySubnet - 您從現有子網路清單中選擇的子網路。
    - myResourceGroupILB - 作為您新建立的資源群組名稱。
3. 按一下 [建立] 以建立負載平衡器。
   
    ## <a name="create-backend-servers"></a>建立後端伺服器

本節中，您會針對基本負載平衡器的後端集區建立兩部虛擬機器，然後在虛擬機器上安裝 IIS，協助測試負載平衡器。

### <a name="create-virtual-machines"></a>建立虛擬機器

1. 在畫面左上方按一下 [建立資源] > [計算] > [Windows Server 2016 Datacenter]，然後輸入虛擬網路的下列值：
    - myVM1 - 作為虛擬機器的名稱。        
    - azureuser - 作為系統管理員使用者名稱。   
    - myResourceGroupILB - 作為 [資源群組]，選取 [使用現有的]，然後選取 myResourceGroupILB。
2. 按一下 [確定]。
3. 選取 [DS1_V2] 作為虛擬機器的大小，然後按一下 [選取]。
4. 輸入 VM 設定的下列值：
    - myAvailabilitySet - 作為您新建立的可用性設定組名稱。
    -  myVNet - 確保已選取作為虛擬網路。
    - myBackendSubnet - 確保已選取作為子網路。
5. 在 [網路安全性群組] 下方，選取 [進階]。 接下來，針對 [網路安全性群組 (防火牆)]，選取 [無]。
5. 按一下 [停用] 來停用開機診斷。
6. 按一下 [確定]，檢閱 [摘要] 頁面上的設定，然後按一下 [建立]。
7. 使用步驟 1-6，建立名為 VM2 的第二部 VM，其可用性設定組為 myAvailabilityset、虛擬網路為 myVnet、子網路為 myBackendSubnet，然後針對 [網路安全性群組(防火牆)] 選取 [無]。 

### <a name="install-iis-and-customize-the-default-web-page"></a>安裝 IIS 和自訂預設網頁

1. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下 **myVM1**，其位於 myResourceGroupILB 資源群組中。
2. 在 [概觀] 頁面上，按一下 [連線] 以透過 RDP 連入 VM。
3. 登入 VM。
4. 在伺服器桌面上，瀏覽至 [Windows 系統管理工具]>[伺服器管理員]。
5. 啟動 VM1 上的 Windows PowerShell 上，並使用下列命令來安裝 IIS 伺服器及更新預設 htm 檔。
    ```powershell-interactive
    # Install IIS
      Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    ```
5. 關閉與 myVM1 的 RDP 連線。
6. 對 myVM2 重複步驟 1-5，以安裝 IIS 和自訂預設網頁。

## <a name="create-basic-load-balancer-resources"></a>建立基本負載平衡器資源

在本節中，您會設定後端位址集區和健康狀態探查的負載平衡器設定，並指定負載平衡器和 NAT 規則。


### <a name="create-a-backend-address-pool"></a>建立後端位址集區

若要將流量分散至 VM，後端位址集區包含已連線至負載平衡器之虛擬 (NIC) 的 IP 位址。 建立後端位址集區 myBackendPool 以納入 VM1 和 VM2。

1. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下 [myLoadBalancer]。
2. 在 [設定] 之下，依序按一下 [後端集區] 和 [新增]。
3. 在 [新增後端集區] 頁面上，執行下列操作：
    - 針對名稱，輸入 myBackEndPool，作為您後端集區的名稱。
    - 針對 [關聯對象]，從下拉式功能表按一下 [可用性設定組]
    - 針對 [可用性設定組]，按一下 [myAvailabilitySet]。
    - 按一下 [新增目標網路 IP 組態]，將您建立的每部虛擬機器 (myVM1 & myVM2) 新增至後端集區。
    - 按一下 [確定]。

        ![新增至後端位址集區 ](./media/tutorial-load-balancer-basic-internal-portal/3-load-balancer-backend-02.png)

3. 請檢查以確定負載平衡器後端集區設定顯示  **VM1** 和 **VM2** 這兩個 VM。

### <a name="create-a-health-probe"></a>建立健康狀態探查

若要讓基本負載平衡器監視您應用程式的狀態，請使用健康狀態探查。 健康狀態探查會根據 VM 對健康狀態檢查的回應，以動態方式從負載平衡器輪替中新增或移除 VM。 建立健康狀態探查 myHealthProbe 以監視 VM 的健康狀態。

1. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下 [myLoadBalancer]。
2. 在 [設定] 之下，依序按一下 [健康狀態探查] 和 [新增]。
3. 使用下列值來建立健康狀態探查：
    - myHealthProbe - 作為健康狀態探查的名稱。
    - HTTP - 作為通訊協定類型。
    - 80 - 作為連接埠號碼。
    - 15 - 探查嘗試的 [間隔] 秒數。
    - 2 - 作為 [狀況不良閾值]，或將 VM 視為狀況不良之前，必須達到的連續探查失敗次數。
4. 按一下 [確定]。

   ![新增探查](./media/tutorial-load-balancer-basic-internal-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>建立負載平衡器規則

負載平衡器規則用來定義如何將流量分散至 VM。 您可定義連入流量的前端 IP 組態及後端 IP 集區來接收流量，以及所需的來源和目的地連接埠。 建立負載平衡器規則 myLoadBalancerRuleWeb，用來接聽前端 LoadBalancerFrontEnd 中的連接埠 80，以及用來將負載平衡的網路流量傳送到後端位址集區 myBackEndPool (也是使用連接埠 80)。 

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
    
    ![新增負載平衡規則](./media/tutorial-load-balancer-basic-internal-portal/5-load-balancing-rules.png)

## <a name="create-a-virtual-machine-to-test-the-load-balancer"></a>建立虛擬機器以測試負載平衡器
為測試內部負載平衡器，您必須將虛擬機器建立在後端伺服器 VM 所在的虛擬網路中。
1. 在畫面左上方按一下 [建立資源] > [計算] > [Windows Server 2016 Datacenter]，然後輸入虛擬網路的下列值：
    - myVMTest - 作為虛擬機器的名稱。        
    - myResourceGroupILB - 作為 [資源群組]，選取 [使用現有的]，然後選取 myResourceGroupILB。
2. 按一下 [確定]。
3. 選取 [DS1_V2] 作為虛擬機器的大小，然後按一下 [選取]。
4. 輸入 VM 設定的下列值：
    -  myVNet - 確保已選取作為虛擬網路。
    - myBackendSubnet - 確保已選取作為子網路。
5. 按一下 [停用] 來停用開機診斷。
6. 按一下 [確定]，檢閱 [摘要] 頁面上的設定，然後按一下 [建立]。

## <a name="test-the-load-balancer"></a>測試負載平衡器
1. 在 Azure 入口網站中，從 [概觀] 畫面上取得負載平衡器的私人 IP 位址。 若要這樣做：a. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下 [myLoadBalancer]。
    b. 在 [概觀] 詳細資料頁面上，複製私人 IP 位址 (在此範例中是 10.1.0.7)。

2. 與 myVMTest 建立遠端連線，如下所示：a. 按一下左側功能表中的 [所有資源]，然後從資源清單按一下 ，其位於 myResourceGroupILB 資源群組中。
2. 在 [概觀] 頁面上，按一下 [連線] 以對 VM 啟動遠端工作階段。
3. 登入 myVMTest。
3. 在 myVMTest 的瀏覽器網址列中貼上私人 IP 位址。 IIS Web 伺服器的預設頁面會顯示在瀏覽器上。

      ![IIS Web 伺服器](./media/tutorial-load-balancer-basic-internal-portal/9-load-balancer-test.png)

若要查看負載平衡器如何將流量分散於執行您應用程式的這兩個 VM，您可以強制重新整理您的網頁瀏覽器。

## <a name="clean-up-resources"></a>清除資源

若不再需要，可刪除資源群組、負載平衡器和所有相關資源。 若要這樣做，請選取包含負載平衡器的資源群組，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立資源群組、網路資源和後端伺服器。 然後您使用這些資源來建立內部負載平衡器，將內部流量負載平衡到 VM。 接下來，您可以了解如何[跨越多個可用性區域為 VM 進行負載平衡](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
