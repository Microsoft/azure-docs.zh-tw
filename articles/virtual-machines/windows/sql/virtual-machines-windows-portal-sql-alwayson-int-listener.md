---
title: 在 Azure 虛擬機器中建立 SQL Server 可用性群組接聽程式 | Microsoft Docs
description: 在 Azure 虛擬機器中建立 SQL Server 的 AlwaysOn 可用性群組接聽程式的逐步指示
services: virtual-machines
documentationcenter: na
author: MikeRayMSFT
manager: craigg
editor: monicar
ms.assetid: d1f291e9-9af2-41ba-9d29-9541e3adcfcf
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 02/16/2017
ms.author: mikeray
ms.openlocfilehash: e87b58ecd72291365f9eba70c807e3018c02ae07
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43382734"
---
# <a name="configure-a-load-balancer-for-an-always-on-availability-group-in-azure"></a>在 Azure 中設定 Always On 可用性群組的負載平衡器
本文說明如何在使用 Azure Resource Manager 執行的 Azure 虛擬機器中建立 SQL Server AlwaysOn 可用性群組的負載平衡器。 當 SQL Server 執行個體位於 Azure 虛擬機器時，可用性群組需要負載平衡器。 負載平衡器會儲存可用性群組接聽程式的 IP 位址。 如果可用性群組跨越多個區域，則每個區域都需要負載平衡器。

若要完成這項工作，您必須在使用 Resource Manager 執行的 Azure 虛擬機器上部署 SQL Server 可用性群組。 這兩部 SQL Server 虛擬機器必須屬於相同的可用性設定組。 您可以使用 [Microsoft 範本](virtual-machines-windows-portal-sql-alwayson-availability-groups.md)在 Resource Manager 中自動建立可用性群組。 此範本會自動為您建立內部負載平衡器。 

如果您想要的話，也可以 [手動設定可用性群組](virtual-machines-windows-portal-sql-availability-group-tutorial.md)。

本文會要求您已經設定可用性群組。  

相關主題包括：

* [在 Azure VM (GUI) 中設定 Always On 可用性群組](virtual-machines-windows-portal-sql-availability-group-tutorial.md)   
* [使用 Azure Resource Manager 和 PowerShell 來設定 VNet 對 VNet 連線](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

依照本文逐步執行，在 Azure 入口網站中建立和設定負載平衡器。 此程序完成之後，您會設定叢集，以將來自負載平衡器的 IP 位址使用於可用性群組接聽程式。

## <a name="create-and-configure-the-load-balancer-in-the-azure-portal"></a>在 Azure 入口網站中建立及設定負載平衡器
在這部分的工作中，執行下列作業︰

1. 在 Azure 入口網站中，建立負載平衡器和設定 IP 位址。
2. 設定後端集區。
3. 建立探查。 
4. 設定負載平衡規則。

> [!NOTE]
> 如果 SQL Server 執行個體位於多個資源群組和區域中，請執行每個步驟兩次，在每個資源群組中各一次。
> 
> 

### <a name="step-1-create-the-load-balancer-and-configure-the-ip-address"></a>步驟 1：建立負載平衡器和設定 IP 位址
首先，建立負載平衡器。 

1. 在 Azure 入口網站中，開啟包含 SQL Server 虛擬機器的資源群組。 

2. 在資源群組中，按一下 [新增] 。

3. 搜尋**負載平衡器**，然後在搜尋結果中，選取由 **Microsoft** 發佈的 [負載平衡器]。

4. 在 [負載平衡器] 刀鋒視窗上，按一下 [建立]。

5. 在 [建立負載平衡器] 對話方塊中，依下列方式設定負載平衡器︰

   | 設定 | 值 |
   | --- | --- |
   | **名稱** |代表負載平衡器的文字名稱。 例如 **sqlLB**。 |
   | **類型** |**內部**︰大部分的實作都會使用內部負載平衡器，可讓相同虛擬網路內的應用程式連線到可用性群組。  </br> **外部**︰可讓應用程式透過公用網際網路連線來連線到可用性群組。 |
   | **虛擬網路** |選取 SQL Server 執行個體所在的虛擬網路。 |
   | **子網路** |選取 SQL Server 執行個體所在的子網路。 |
   | **IP 位址指派** |**靜態** |
   | **私人 IP 位址** |從子網路指定可用的 IP 位址。 當您在叢集上建立接聽程式時，使用此 IP 位址。 在本文稍後的 PowerShell 指令碼中，將此位址用於 `$ILBIP` 變數。 |
   | **訂用帳戶** |如果您有多個訂用帳戶，此欄位才會出現。 選取您想要與此資源相關聯的訂用帳戶。 通常是與可用性群組的所有資源相同的訂用帳戶。 |
   | **資源群組** |選取 SQL Server 執行個體所在的資源群組。 |
   | **位置** |選取 SQL Server 執行個體所在的 Azure 位置。 |

6. 按一下頁面底部的 [新增] 。 

Azure 會建立負載平衡器。 此負載平衡器屬於特定的網路、子網路、資源群組和位置。 Azure 完成工作之後，請確認 Azure 中的負載平衡器設定。 

### <a name="step-2-configure-the-back-end-pool"></a>步驟 2：設定後端集區
Azure 會呼叫後端位址集區 *backend pool*。 在此情況下，後端集區是您的可用性群組中兩部 SQL Server 執行個體的位址。 

1. 在資源群組中，按一下您建立的負載平衡器。 

2. 在 [設定] 上，按一下 [後端集區]。

3. 在 [後端集區] 上，按一下 [新增] 以建立後端位址集區。 

4. 在 [新增後端集區] 的 [名稱] 之下，輸入後端集區的名稱。

5. 在 [虛擬機器] 之下，按一下 [新增虛擬機器]。 

6. 在 [選擇虛擬機器] 之下，按一下 [選擇可用性設定組]，然後指定 SQL Server 虛擬機器所屬的可用性設定組。

7. 在您選擇可用性設定組之後，請按一下 [選擇虛擬機器]，選取可用性群組中主控 SQL Server 執行個體的兩部虛擬機器，然後按一下 [選取]。 

8. 按一下 [確定] 以關閉 [選擇虛擬機器] 和 [新增後端集區] 的刀鋒視窗。 

Azure 更新後端位址集區的設定。 您的可用性設定組現在有包含兩個 SQL Server 執行個體的集區。

### <a name="step-3-create-a-probe"></a>步驟 3：建立探查
探查會定義 Azure 如何確認哪一個 SQL Server 執行個體目前擁有可用性群組接聽程式。 Azure 會根據在建立探查時定義的連接埠上的 IP 位址來探查服務。

1. 在負載平衡器的 [設定] 刀鋒視窗上，按一下 [健全狀況探查]。 

2. 在 [健全狀況探查] 刀鋒視窗上，按一下 [新增]。

3. 在 [新增探查]  刀鋒視窗上設定探查。 使用下列值來設定探查：

   | 設定 | 值 |
   | --- | --- |
   | **名稱** |代表探查的文字名稱。 例如 **SQLAlwaysOnEndPointProbe**。 |
   | **通訊協定** |**TCP** |
   | **連接埠** |您可以使用任何可用的連接埠。 例如 *59999*。 |
   | **間隔** |*5* |
   | **狀況不良臨界值** |*2* |

4.  按一下 [確定]。 

> [!NOTE]
> 確定您指定的連接埠會在兩個 SQL Server 執行個體的防火牆上開啟。 這兩個執行個體需要您所用 TCP 通訊埠的輸入規則。 如需詳細資訊，請參閱[新增或編輯防火牆規則](http://technet.microsoft.com/library/cc753558.aspx)。 
> 
> 

Azure 會建立探查，然後使用它來測試那一個 SQL Server 執行個體具有可用性群組的接聽程式。

### <a name="step-4-set-the-load-balancing-rules"></a>步驟 4：設定負載平衡規則
負載平衡規則會設定負載平衡器將流量路由傳送至 SQL Server 執行個體的方式。 對此負載平衡器，您會啟用伺服器直接回傳，因為兩個 SQL Server 執行個體中一次只有一個會擁有可用性群組接聽程式資源。

1. 在負載平衡器的 [設定] 刀鋒視窗上，按一下 [負載平衡規則]。 

2. 在 [負載平衡規則] 刀鋒視窗上，按一下 [新增]。

3. 在 [新增負載平衡規則] 刀鋒視窗上，設定負載平衡規則。 套用下列設定： 

   | 設定 | 值 |
   | --- | --- |
   | **名稱** |代表負載平衡規則的文字名稱。 例如 **SQLAlwaysOnEndPointListener**。 |
   | **通訊協定** |**TCP** |
   | **連接埠** |*1433* |
   | **後端連接埠** |*1433*.系統會忽略此值，因為此規則使用 [浮動 IP (伺服器直接回傳)]。 |
   | **探查** |使用您為此負載平衡器建立之探查的名稱。 |
   | **工作階段持續性** |**None** |
   | **閒置逾時 (分鐘)** |*4* |
   | **浮動 IP (伺服器直接回傳)** |**已啟用** |

   > [!NOTE]
   > 您可能必須向下捲動刀鋒視窗，以檢視所有的設定。
   > 

4. 按一下 [確定]。 
5. Azure 會設定負載平衡規則。 負載平衡器現已設定成將流量路由傳送到裝載可用性群組接聽程式的 SQL Server 執行個體。 

此時，資源群組有一個連接到這兩部 SQL Server 電腦的負載平衡器。 負載平衡器也包含 SQL Server AlwaysOn 可用性群組接聽程式的 IP 位址，以便電腦回應對可用性群組的要求。

> [!NOTE]
> 如果您的 SQL Server 執行個體位於兩個不同的區域，請在另一個區域中重複執行步驟。 每個區域都需要負載平衡器。 
> 
> 

## <a name="configure-the-cluster-to-use-the-load-balancer-ip-address"></a>設定叢集以使用負載平衡器 IP 位址
下一個步驟是在叢集上設定接聽程式，並且讓接聽程式上線。 執行下列動作： 

1. 在容錯移轉叢集上建立可用性群組接聽程式。 

2. 使接聽程式上線。

### <a name="step-5-create-the-availability-group-listener-on-the-failover-cluster"></a>步驟 5：在容錯移轉叢集上建立可用性群組接聽程式
在此步驟中，您會在容錯移轉叢集管理員和 SQL Server Management Studio 中手動建立可用性群組接聽程式。

[!INCLUDE [ag-listener-configure](../../../../includes/virtual-machines-ag-listener-configure.md)]

### <a name="verify-the-configuration-of-the-listener"></a>驗證接聽程式的組態

如果已正確設定叢集資源和相依性，您應該能夠檢視 SQL Server Management Studio 中的接聽程式。 若要設定接聽程式連接埠，請執行下列步驟︰

1. 啟動 SQL Server Management Studio，然後連線到主要複本。

2. 移至 [AlwaysOn 高可用性] > [可用性群組] > [可用性群組接聽程式]。  
    您現在應該會看到在容錯移轉叢集管理員中建立的接聽程式名稱。 

3. 以滑鼠右鍵按一下接聽程式名稱，然後按一下 [屬性]。

4. 在 [連接埠] 方塊中，使用您稍早所用的 $EndpointPort (預設值是 1433) 來指定可用性群組接聽程式的連接埠號碼，然後按一下 [確定]。

現在，您在以 Resource Manager 模式執行的 Azure 虛擬機器中，已有一個可用性群組。 

## <a name="test-the-connection-to-the-listener"></a>測試接聽程式的連線
透過下列方式測試連線︰

1. 透過 RDP 連接到相同虛擬網路中不擁有複本的 SQL Server 執行個體。 此伺服器可以是叢集中的其他 SQL Server 執行個體。

2. 使用 **sqlcmd** 公用程式來測試連線。 例如，下列指令碼會透過接聽程式搭配 Windows 驗證來建立主要複本的 **sqlcmd** 連線︰
   
        sqlcmd -S <listenerName> -E

SQLCMD 連線會自動連線到裝載主要複本的 SQL Server 執行個體。 

## <a name="create-an-ip-address-for-an-additional-availability-group"></a>建立其他可用性群組的 IP 位址

每個可用性群組都會使用個別的接聽程式。 每個接聽程式有其自己的 IP 位址。 使用相同的負載平衡器為其他接聽程式保存 IP 位址。 建立可用性群組之後，將 IP 位址新增至負載平衡器，然後設定接聽程式。

若要使用 Azure 入口網站來將 IP 位址新增到負載平衡器，請執行下列步驟：

1. 在 Azure 入口網站中，開啟包含負載平衡器的資源群組，然後按一下負載平衡器。 

2. 在 [設定] 下按一下 [前端 IP 集區]，然後按一下 [新增]。 

3. 在 [新增前端 IP 位址] 下指派前端的名稱。 

4. 請確定**虛擬網路**和**子網路**與 SQL Server 執行個體相同。

5. 設定接聽程式的 IP 位址。 
   
   >[!TIP]
   >您可以設定為靜態 IP 位址，並輸入目前未在子網路中使用的位址。 或者，您可以設定動態 IP 位址，並儲存新的前端 IP 集區。 當您這樣做時，Azure 入口網站會自動將可用的 IP 位址指派給集區。 然後您可以重新開啟前端 IP 集區，並將指派變更為靜態。 

6. 儲存接聽程式的 IP 位址。 

7. 使用下列設定來新增健康狀態探查︰

   |設定 |值
   |:-----|:----
   |**名稱** |用於識別探查的名稱。
   |**通訊協定** |TCP
   |**連接埠** |未使用的 TCP 通訊埠，其必須可在所有虛擬機器上使用。 不能用於其他用途。 兩個接聽程式不可使用相同的探查連接埠。 
   |**間隔** |探查嘗試間隔的時間長度。 使用預設值 (5)。
   |**狀況不良臨界值** |連續發生錯誤的臨界值數目，超過此數目後虛擬機器會被視為狀況不良。

8. 按一下 [確定] 儲存探查。 

9. 建立負載平衡規則。 按一下 [負載平衡規則]，然後按一下 [新增]。

10. 使用下列設定來設定新的負載平衡規則：

   |設定 |值
   |:-----|:----
   |**名稱** |用於識別負載平衡規則的名稱。 
   |**前端 IP 位址** |選取您所建立的 IP 位址。 
   |**通訊協定** |TCP
   |**連接埠** |使用 SQL Server 執行個體正在使用的連接埠。 預設的執行個體會使用連接埠 1433，除非您有進行變更。 
   |**後端連接埠** |使用相同的值作為**連接埠**。
   |**後端集區** |包含虛擬機器和 SQL Server 執行個體的集區。 
   |**健康狀態探查** |選擇您所建立的探查。
   |**工作階段持續性** |None
   |**閒置逾時 (分鐘)** |預設值 (4)
   |**浮動 IP (伺服器直接回傳)** | 已啟用

### <a name="configure-the-availability-group-to-use-the-new-ip-address"></a>設定可用性群組以使用新的 IP 位址

若要完成設定叢集，請重複您建立第一個可用性群組時遵循的步驟。 也就是，設定[叢集以使用新的 IP 位址](#configure-the-cluster-to-use-the-load-balancer-ip-address)。 

新增接聽程式的 IP 位址之後，請執行下列步驟來設定其他可用性群組： 

1. 確認新 IP 位址的探查連接埠都已經在兩部 SQL Server 虛擬機器上開啟。 

2. [在叢集管理員中，新增用戶端存取點](#addcap)。

3. [為可用性群組設定 IP 資源](#congroup)。

   >[!IMPORTANT]
   >當您建立 IP 位址時，請使用您在負載平衡器中新增的 IP 位址。  

4. [讓 SQL Server 可用性群組資源依存於用戶端存取點](#dependencyGroup)。

5. [讓用戶端存取點資源依存於 IP 位址](#listname)。
 
6. [在 PowerShell 中設定叢集參數](#setparam)。

當您設定可用性群組以使用新的 IP 位址之後，請設定與接聽程式間的連線。 

## <a name="add-load-balancing-rule-for-distributed-availability-group"></a>為分散式可用性群組新增負載平衡規則

如果可用性群組加入分散式可用性群組中，負載平衡器會需要額外的規則。 此規則會儲存分散式可用性群組接聽程式所使用的連接埠。

>[!IMPORTANT]
>只有在可用性群組加入[分散式可用性群組](http://docs.microsoft.com/sql/database-engine/availability-groups/windows/configure-distributed-availability-groups)時，才適用此步驟。 

1. 在加入分散式可用性群組的每個伺服器上，對分散式可用性群組接聽程式的 TCP 連接埠建立輸入規則。 在許多範例中，文件會使用 5022。 

1. 在 Azure 入口網站中，按一下負載平衡器，按一下 [負載平衡器規則]，然後按一下 [+新增]。 

1. 使用下列設定來建立負載平衡規則：

   |設定 |值
   |:-----|:----
   |**名稱** |用來識別分散式可用性群組之負載平衡規則的名稱。 
   |**前端 IP 位址** |使用和可用性群組相同的前端 IP 位址。
   |**通訊協定** |TCP
   |**連接埠** |5022 - [分散式可用性群組端點接聽程式](http://docs.microsoft.com/sql/database-engine/availability-groups/windows/configure-distributed-availability-groups)的連接埠。</br> 可以是任何可用連接埠。  
   |**後端連接埠** | 5022 - 使用和 [連接埠] 相同的值。
   |**後端集區** |包含虛擬機器和 SQL Server 執行個體的集區。 
   |**健康狀態探查** |選擇您所建立的探查。
   |**工作階段持續性** |None
   |**閒置逾時 (分鐘)** |預設值 (4)
   |**浮動 IP (伺服器直接回傳)** | 已啟用

針對其他加入分散式可用性群組之可用性群組上的負載平衡器重複這些步驟。

如果您使用 Azure 網路安全性群組來限制存取，請確定允許規則包含後端 SQL Server VM IP 位址和 AG 接聽程式的負載平衡器浮動 IP 位址，以及叢集核心 IP 位址 (如果適用的話)。

## <a name="next-steps"></a>後續步驟

- [在不同區域中的 Azure 虛擬機器上設定 SQL Server Always On 可用性群組](virtual-machines-windows-portal-sql-availability-group-dr.md)
