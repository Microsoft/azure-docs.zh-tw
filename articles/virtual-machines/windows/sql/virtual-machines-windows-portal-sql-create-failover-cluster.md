---
title: SQL Server FCI - Azure 虛擬機器 | Microsoft Docs
description: 本文說明如何在 Azure 虛擬機器上建立 SQL Server 容錯移轉叢集執行個體。
services: virtual-machines
documentationCenter: na
authors: MikeRayMSFT
manager: craigg
editor: monicar
tags: azure-service-management
ms.assetid: 9fc761b1-21ad-4d79-bebc-a2f094ec214d
ms.service: virtual-machines-sql
ms.devlang: na
ms.custom: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/11/2018
ms.author: mikeray
ms.openlocfilehash: 8e107c1721d5623239a694eba39b32e8a2a6089d
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42145362"
---
# <a name="configure-sql-server-failover-cluster-instance-on-azure-virtual-machines"></a>在 Azure 虛擬機器上設定 SQL Server 容錯移轉叢集執行個體

本文說明如何在 Resource Manager 模型內的 Azure 虛擬機器中，建立 SQL Server 容錯移轉叢集執行個體 (FCI)。 此解決方案會使用 [Windows Server 2016 Datacenter 版本儲存空間直接存取 \(S2D\) ](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/storage-spaces-direct-overview) 做為軟體型虛擬 SAN，以同步 Windows 叢集中各節點 (Azure VM) 間的儲存體 (資料磁碟)。 S2D 是 Windows Server 2016 中的新功能。

下圖顯示 Azure 虛擬機器中的完整解決方案：

![可用性群組](./media/virtual-machines-windows-portal-sql-create-failover-cluster/00-sql-fci-s2d-complete-solution.png)

上圖顯示︰

- 兩部位於 Windows 容錯移轉叢集中的 Azure 虛擬機器。 位於容錯移轉叢集中的虛擬機器也稱為「叢集節點」或「節點」。
- 每部虛擬機器有兩個以上的資料磁碟。
- S2D 會同步處理資料磁碟上的資料，並將已同步處理的儲存體當做儲存體集區使用。
- 儲存體集區會向容錯移轉提供叢集共用磁碟區 (CSV)。
- SQL Server FCI 叢集角色會針對資料硬碟使用 CSV。
- 保留 SQL Server FCI IP 位址的 Azure Load Balancer。
- 保留所有資源的 Azure 可用性設定組。

   >[!NOTE]
   >圖表內的所有 Azure 資源都位於相同的資源群組中。

如需有關 S2D 的詳細資料，請參閱 [Windows Server 2016 Datacenter 版本儲存空間直接存取 \(S2D\)](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/storage-spaces-direct-overview)。

S2D 支援兩種類型的架構 - 交集和超交集。 本文件中的架構為超交集。 超交集基礎結構會將儲存體放置於與裝載叢集應用程式相同的伺服器上。 在此架構中，儲存體會放置在每個 SQL Server FCI 節點上。

## <a name="licensing-and-pricing"></a>授權和價格

在 Azure 虛擬機器上，您可以使用隨收隨付制 (PAYG) 進行 SQL Server 的授權，而 VM 映像也可自備授權 (BYOL)。 您選擇的映像類型會影響向您收費的方式。

使用 PAYG 授權時，在 Azure 虛擬機器上，SQL Server 的容錯移轉叢集執行個體 (FCI) 會產生所有 FCI 節點的費用，包括被動節點。 如需詳細資訊，請參閱 [SQL Server Enterprise 虛擬機器定價](http://azure.microsoft.com/pricing/details/virtual-machines/sql-server-enterprise/)。 

至於有簽訂軟體保證 Enterprise 合約的客戶，每個作用中的節點有權使用一個免費的被動 FCI 節點。 若要利用 Azure 的這項優點，可使用 BYOL VM 映像，然後在 FCI 的主動和被動節點上使用相同的授權。 如需詳細資訊，請參閱 [Enterprise 合約](http://www.microsoft.com/en-us/Licensing/licensing-programs/enterprise.aspx)。

若要比較 Azure 虛擬機器上 SQL Server 的 PAYG 和 BYOL 授權，請參閱[開始使用 SQL VM](virtual-machines-windows-sql-server-iaas-overview.md#get-started-with-sql-vms)。

如需授權 SQL Server 的完整資訊，請參閱[定價](http://www.microsoft.com/sql-server/sql-server-2017-pricing)。

### <a name="example-azure-template"></a>Azure 範本範例

您可以在 Azure 中，從範本開始建立整個解決方案。 GitHub [Azure 快速入門範本](https://github.com/MSBrett/azure-quickstart-templates/tree/master/sql-server-2016-fci-existing-vnet-and-ad) 內有提供範本範例。 此範例並非為任何特定工作負載而設計或測試。 您可以執行範本來建立 SQL Server FCI，並將 S2D 儲存區連接至您的網域。 您可以評估此範本，並依需要進行修改。

## <a name="before-you-begin"></a>開始之前

開始之前，您必須先了解一些需要注意的事項，並先備妥幾個項目。

### <a name="what-to-know"></a>注意事項
您應了解下列技術的操作方式：

- [Windows 叢集技術](http://technet.microsoft.com/library/hh831579.aspx)
- [SQL Server 容錯移轉叢集執行個體](http://msdn.microsoft.com/library/ms189134.aspx)。

此外，您也應對下列技術有大概了解：

- [在 Windows Server 2016 中使用儲存空間直接存取的超交集解決方案](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct)
- [Azure 資源群組](../../../azure-resource-manager/resource-group-portal.md)

> [!IMPORTANT]
> 目前不支援將 [SQL Server IaaS 代理程式擴充功能](virtual-machines-windows-sql-server-agent-extension.md)用於 Azure 上的 SQL Server FCI。 建議您在參與 FCI 的 VM 上解除安裝此擴充功能。 針對 SQL，此擴充功能支援自動備份和修補之類的功能，以及部分入口網站功能。 在代理程式解除安裝後，這些功能即不適用於 SQL VM。

### <a name="what-to-have"></a>應備項目

在依照本文中的指示進行之前，您應先備妥下列項目：

- Microsoft Azure 訂用帳戶。
- Azure 虛擬機器上的 Windows 網域。
- 擁有在 Azure 虛擬機器中建立物件之權限的帳戶。
- Azure 虛擬網路和子網路，且具有足夠 IP 位址空間以容納下列元件：
   - 兩部虛擬機器。
   - 容錯移轉叢集的 IP 位址。
   - 每個 FCI 上一個 IP 位址。
- 設定 Azure 網路上的 DNS，並指向網域控制站。

備妥這些先決條件後，便可繼續建置您的容錯移轉叢集。 第一步是建立虛擬機器。

## <a name="step-1-create-virtual-machines"></a>步驟 1：建立虛擬機器

1. 使用您的訂用帳戶登入 [Azure 入口網站](http://portal.azure.com)。

1. [建立 Azure 可用性設定組](../tutorial-availability-sets.md)。

   可用性設定組群組虛擬機器遍佈在容錯網域與更新網域中， 可用性設定組可確保應用程式不受單一失敗點 (如網路交換器或伺服器機架的電源裝置) 所影響。

   若您尚未建立虛擬機器的資源群組，請在建立 Azure 可用性設定組時建立。 若您正在使用 Azure 入口網站建立可用性設定組，請執行下列步驟︰

   - 在 Azure 入口網站中，按一下 **+** 以開啟 Azure Marketplace。 搜尋 [可用性設定組]。
   - 按一下 [可用性設定組]。
   - 按一下頁面底部的 [新增] 。
   - 在 [建立可用性設定組] 刀鋒視窗中，設定下列值︰
      - **名稱**：可用性設定組的名稱。
      - **訂用帳戶**：您的 Azure 訂用帳戶。
      - **資源群組**︰若您想要使用現有的群組，請按一下 [使用現有的群組] 並從下拉式清單中選取群組。 否則，選擇 [建立新的] 並輸入群組名稱。
      - **位置**︰設定您計劃建立虛擬機器的位置。
      - **容錯網域**︰使用預設值 (3)。
      - **更新網域**︰使用預設值 (5)。
   - 按一下 [建立] 來建立可用性設定組。

1. 在可用性設定組中建立虛擬機器。

   在 Azure 可用性設定組中佈建兩部 SQL Server 虛擬機器。 如需指示，請參閱[在 Azure 入口網站中佈建 SQL Server 虛擬機器](virtual-machines-windows-portal-sql-server-provision.md)。

   將兩部虛擬機器放置於：

   - 與可用性設定組相同的 Azure 資源群組中。
   - 您網域控制站的相同網路上。
   - 有足夠 IP 位址空間存放兩部虛擬機器，與最終會用於此叢集的所有 FCI 的子網路上。
   - Azure 可用性設定組中。   

      >[!IMPORTANT]
      >建立虛擬機器後，您無法設定或變更可用性設定組。

   從 Azure Marketplace 中選擇映像。 您可以使用包含 Windows Server 與 SQL Server，或只包含 Windows Server 的 Marketplace 映像。 如需詳細資料，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](virtual-machines-windows-sql-server-iaas-overview.md)

   Azure 資源庫中的官方 SQL Server 映像包含已安裝的 SQL Server 執行個體、SQL Server 安裝軟體與所需金鑰。

   按照您喜好的付款方式購買 SDL Server 授權，並選擇合適的映像：

   - **每次使用授權時付款**：費用會以秒鐘計算，且會包含下列 SQL Server 授權：
      - **Windows Server Datacenter 2016 上的 SQL Server 2016 Enterprise 版本**
      - **Windows Server Datacenter 2016 上的 SQL Server 2016 Standard 版本**
      - **Windows Server Datacenter 2016 上的 SQL Server 2016 Developer 版本**

   - **自備授權 (BYOL)**

      - **{BYOL} Windows Server Datacenter 2016 上的 SQL Server 2016 Enterprise 版本**
      - **{BYOL} Windows Server Datacenter 2016 上的 SQL Server 2016 Standard 版本**

   >[!IMPORTANT]
   >建立虛擬機器後，請移除預先安裝的獨立 SQL Server 執行個體。 設定容錯移轉叢集與 S2D 後，您將使用預先安裝的 SQL Server 媒體來建立 SQL Server FCI。

   或者，您也可以只將 Azure Marketplace 映像與作業系統搭配使用。 設定容錯移轉叢集與 S2D 後，請選擇 **Windows Server 2016 Datacenter** 映像並安裝 SQL Server FCI。 此映像不包含 SQL Server 安裝媒體。 將安裝媒體放置於可在每個伺服器上執行 SQL Server 安裝的位置。

1. Azure 建立虛擬機器後，請透過 RDP 連接至每部虛擬機器。

   首次透過 RDP 連接至虛擬機器時，電腦會詢問您是否允許此電腦在網路上可供搜尋。 按一下 [是] 。

1. 若您正在使用其中一個 SQL Server 型虛擬機器映像，請移除 SQL Server 執行個體。

   - 在 [程式和功能] 中，以滑鼠右鍵按一下 [Microsoft SQL Server 2016 (64 位元)] 然後按一下 [解除安裝/變更]。
   - 按一下 [移除] 。
   - 選取預設執行個體。
   - 移除所有在 [Database Engine 服務] 下的功能。 請勿移除 [共用功能]。 請參閱下圖︰

      ![移除功能](./media/virtual-machines-windows-portal-sql-create-failover-cluster/03-remove-features.png)

   - 按一下 [下一步]，然後按一下 [移除]。

1. <a name="ports"></a>開啟防火牆連接埠。

   在每部虛擬機器上，開啟 Windows 防火牆上的下列連接埠。

   | 目的 | TCP 連接埠 | 注意
   | ------ | ------ | ------
   | SQL Server | 1433 | 適用於 SDL Server 預設執行個體的一般連接埠。 若您曾使用來自資源庫的映像，此連接埠會自動開啟。
   | 健全狀況探查 | 59999 | 任何開啟的 TCP 連接埠。 在接下來的步驟中，設定負載平衝器[健全狀況探查](#probe)和要使用此連接埠的叢集。  

1. 將儲存體新增至虛擬機器中。 如需詳細資訊，請參閱[新增儲存區](../premium-storage.md)。

   每部虛擬機器需要至少兩個資料磁碟。

   連接原始磁碟 - 非 NTFS 格式的磁碟。
      >[!NOTE]
      >若您連接 NTFS 格式的磁碟，您只能啟用 S2D 而無法進行磁碟適用性檢查。  

   每部 VM 至少可連接兩個進階儲存體 (SSD 磁碟)。 建議使用 P30 (1 TB) 或以上的磁碟。

   將主機快取設為**唯讀**。

   在生產環境中使用的儲存體容量是依您的工作負載而定。 本文所描述的值僅供示範與測試之用。

1. [將虛擬機器新增至既存的網域](virtual-machines-windows-portal-sql-availability-group-prereq.md#joinDomain)。

建立並設定虛擬機器後，您便可設定容錯移轉叢集。

## <a name="step-2-configure-the-windows-failover-cluster-with-s2d"></a>步驟 2：透過 S2D 設定 Windows 容錯移轉叢集

下一步是透過 S2D 設定容錯移轉叢集。 在此步驟中，您將執行下列子步驟：

1. 新增 Windows 容錯移轉叢集功能
1. 驗證叢集
1. 建立容錯移轉叢集
1. 建立雲端見證
1. 新增儲存體

### <a name="add-windows-failover-clustering-feature"></a>新增 Windows 容錯移轉叢集功能

1. 若要開始，請使用本機系統管理員成員，且具有在 Active Directory 中建立物件之權限的網域帳戶，透過 RDP 連接至第一部虛擬機器。 在之後的設定程序中繼續使用此帳戶。

1. [將容錯移轉叢集功能新增至每部虛擬機器](virtual-machines-windows-portal-sql-availability-group-prereq.md#add-failover-clustering-features-to-both-sql-server-vms)。

   若要從 UI 安裝容錯移轉叢集功能，請於兩部虛擬機器上執行下列步驟。
   - 在 [伺服器管理員] 中，按一下 [管理]，然後按一下 [新增角色及功能]。
   - 在 [新增角色及功能精靈] 中，連續按 [下一步] 直到到達 [選取功能]。
   - 在 [選取功能] 中，按一下 [容錯移轉叢集]。 選取所有所需功能與管理工具。 按一下 [新增功能]。
   - 按一下 [下一步]，然後按一下 [完成] 以安裝功能。

   若要透過 PowerShell 安裝容錯移轉叢集功能，請在其中一部虛擬機器上執行下列來自系統管理員 PowerShell 工作階段的指令碼。

   ```PowerShell
   $nodes = ("<node1>","<node2>")
   Invoke-Command  $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools}
   ```

如需參考，請在接下來的步驟依照[在 Windows Server 2016 中使用儲存空間直接存取的超交集解決方案](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct#step-3-configure-storage-spaces-direct) 步驟 3 的指示操作。

### <a name="validate-the-cluster"></a>驗證叢集

本指南參考[驗證叢集](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct#step-31-run-cluster-validation)下的指示。

在 UI 中或透過 PowerShell 驗證叢集。

若要透過 UI 驗證叢集，請在其中一部虛擬機器上執行下列步驟。

1. 在 [伺服器管理員] 中，按一下 [工具]，然後按一下 [容錯移轉叢集管理員]。
1. 在 [容錯移轉叢集管理員] 中，按一下 [動作]，然後按一下 [驗證設定...]。
1. 按 [下一步] 。
1. 在 [選取伺服器或叢集] 中，輸入這兩部虛擬機器的名稱。
1. 在 [測試選項] 中，選擇 [僅執行我選取的測試]。 按 [下一步] 。
1. 在 [測試選取範圍] 中，選取**儲存體**以外的所有測試。 請參閱下圖︰

   ![驗證測試](./media/virtual-machines-windows-portal-sql-create-failover-cluster/10-validate-cluster-test.png)

1. 按 [下一步] 。
1. 在 [確認] 中，，按一下 [下一步]。

[驗證設定精靈] 會執行驗證測試。

若要透過 PowerShell 驗證叢集，請在其中一部虛擬機器上，執行下列來自系統管理員 PowerShell 工作階段的指令碼。

   ```PowerShell
   Test-Cluster –Node ("<node1>","<node2>") –Include "Storage Spaces Direct", "Inventory", "Network", "System Configuration"
   ```

驗證叢集後，請建立容錯移轉叢集。

### <a name="create-the-failover-cluster"></a>建立容錯移轉叢集

本指南會參考[建立容錯移轉叢集](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct#step-32-create-a-cluster)。

若要建立容錯移轉叢集，您需要：
- 成為叢集節點的虛擬機器名稱。
- 容錯移轉叢集的名稱
- 容錯移轉叢集的 IP 位址。 您可以使用與叢集節點一樣，沒有在 Azure 虛擬網路和子網路上用過的 IP 位址。

下列 PowerShell 會建立容錯移轉叢集。 使用節點名稱 (虛擬機器名稱) 和 Azure VNET 中可用的 IP 位址來更新指令碼：

```PowerShell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage
```   

### <a name="create-a-cloud-witness"></a>建立雲端見證

雲端見證儲存在 Azure 儲存體 Blob 中，是一種新型的叢集仲裁見證。 如有雲端見證，便不再需要使用獨立的 VM 來裝載見證共用。

1. [建立容錯移轉叢集的雲端見證](http://technet.microsoft.com/windows-server-docs/failover-clustering/deploy-cloud-witness)。

1. 建立 Blob 容器。

1. 儲存存取金鑰和容器 URL。

1. 設定容錯移轉叢集的叢集仲裁見證。 請參閱使用者介面中的[在使用者介面中設定仲裁見證](http://technet.microsoft.com/windows-server-docs/failover-clustering/deploy-cloud-witness#to-configure-cloud-witness-as-a-quorum-witness)。

### <a name="add-storage"></a>新增儲存體

S2D 的磁碟需為空白且不含分割區或其他資料。 若要清理磁碟，請遵循[本指南內的步驟](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct#step-34-clean-disks)。

1. [啟用儲存空間直接存取 \(S2D\)](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct#step-35-enable-storage-spaces-direct)。

   下列 PowerShell 會啟用儲存空間直接存取。  

   ```PowerShell
   Enable-ClusterS2D
   ```

   現在，您可以在**容錯移轉管理員**中看到儲存集區。

1. [建立磁碟區](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct#step-36-create-volumes)。

   S2D 的其中一項功能，是會在您啟用時自動建立儲存集區。 您現在可以開始建立磁碟區。 PowerShell commandlet `New-Volume` 會自動執行磁碟區建立程序，包括格式化、新增至叢集、和建立叢集共用磁碟區 (CSV)。 下列範列會建立 800 GB 的 CSV。

   ```PowerShell
   New-Volume -StoragePoolFriendlyName S2D* -FriendlyName VDisk01 -FileSystem CSVFS_REFS -Size 800GB
   ```   

   完成此命令後，系統會將 800 GB 的磁碟區掛接為叢集資源。 磁碟區會位於 `C:\ClusterStorage\Volume1\`。

   下圖顯示具有 S2D 的叢集共用磁碟區︰

   ![ClusterSharedVolume](./media/virtual-machines-windows-portal-sql-create-failover-cluster/15-cluster-shared-volume.png)

## <a name="step-3-test-failover-cluster-failover"></a>步驟 3︰測試容錯移轉叢集的容錯移轉

在容錯移轉叢集管理員中，請確認您可以將儲存體資源移至其他叢集節點。 如果您可以透過**容錯移轉叢集管理員**連接至容錯移轉叢集，並且能將儲存體從一個節點移到另一個，則您可以設定 FCI。

## <a name="step-4-create-sql-server-fci"></a>步驟 4：建立 SDL Server FCI

設定容錯移轉叢集與所有叢集元件 (包括儲存體) 後，您可以建立 SQL Server FCI。

1. 透過 RDP 連接至第一部虛擬機器。

1. 在**容錯移轉叢集管理員**中，請確認所有叢集核心資源皆位於第一部虛擬機器中。 如有必要，請將所有資源移至此虛擬機器。

1. 找出安裝媒體。 若虛擬機器是使用其中一個 Azure Marketplace 映像，則媒體會位於 `C:\SQLServer_<version number>_Full`。 按一下 [設定] 。

1. 在 [SQL Server 安裝中心] 中，按一下 [安裝]。

1. 按一下 [安裝新的 SQL Server 容錯移轉叢集]。 請遵循精靈內的指示安裝 SQL Server FCI。

   FCI 資料目錄必須位於叢集儲存體中。 與 S2D 搭配使用時，FCI 會做為每個伺服器上的磁碟區掛接點，而非共用磁碟。 S2D 會在節點之間同步處理磁碟區。 叢集會將磁碟區當做叢集共用磁碟區使用。 在資料目錄上使用 CSV 掛接點。

   ![DataDirectories](./media/virtual-machines-windows-portal-sql-create-failover-cluster/20-data-dicrectories.png)

1. 完成精靈中的指示後，安裝程式會在第一個節點上安裝 SQL Server FCI。

1. 安裝程式在第一個節點上成功安裝 FCI 後，請透過 RDP 連接至第二個節點。

1. 開啟 [SQL Server 安裝中心]。 按一下 [安裝]。

1. 按一下 [將節點新增到 SQL Server 容錯移轉叢集]。 請遵循精靈中的指示安裝 SQL Server FCI，並將此伺服器新增至 FCI。

   >[!NOTE]
   >若您曾透過 SQL Server 使用 Azure Marketplace 資源庫映像，映像會包含 SQL Server 工具。 若您沒有用過此映像，請另外安裝 SQL Server 工具。 請參閱[下載 Server Management Studio (SSMS)](http://msdn.microsoft.com/library/mt238290.aspx)。

## <a name="step-5-create-azure-load-balancer"></a>步驟 5：建立 Azure Load Balancer

在 Azure 虛擬機器上，叢集會使用負載平衡器來保留每次均需在一個節點上的 IP 位址。 在此解決方案中，負載平衡器會保留 SQL Server FCI 的 IP 位址。

[建立並設定 Azure Load Balancer](virtual-machines-windows-portal-sql-availability-group-tutorial.md#configure-internal-load-balancer)。

### <a name="create-the-load-balancer-in-the-azure-portal"></a>在 Azure 入口網站中建立負載平衡器

若要建立負載平衡器：

1. 在 Azure 入口網站中，請透過虛擬機器移至資源群組。

1. 按一下 [+ 新增]。 在 Marketplace 內搜尋 [負載平衡器]。 按一下 [負載平衡器]。

1. 按一下頁面底部的 [新增] 。

1. 使用下列項目設定負載平衡器：

   - **名稱**︰用以識別負載平衡器的名稱。
   - **類型**︰負載平衡器分為公開或私人兩種類型。 私人負載平衡器可從相同的 VNET 內存取。 大部分的 Azure 應用程式都能使用私人負載平衡器。 若您的應用程式需要直接透過網際網路存取 SQL Server，請使用公開負載平衡器。
   - **虛擬網路**︰與虛擬機器相同的網路。
   - **子網路**︰與虛擬機器相同的子網路。
   - **私人 IP 位址**︰與您指派至 SQL Server FCI 叢集網路資源相同的 IP 位址。
   - **訂用帳戶**：您的 Azure 訂用帳戶。
   - **資源群組**︰使用與虛擬機器相同的資源群組。
   - **位置**︰使用與虛擬機器相同的 Azure 位置。
   請參閱下圖︰

   ![CreateLoadBalancer](./media/virtual-machines-windows-portal-sql-create-failover-cluster/30-load-balancer-create.png)

### <a name="configure-the-load-balancer-backend-pool"></a>設定負載平衡器後端集區

1. 透過虛擬機器返回 Azure 資源群組，並尋找新的負載平衡器。 您可能需要重新整理資源群組的畫面。 按一下 [負載平衡器]。

1. 按一下 [後端集區]，然後按一下 [+ 新增] 以新增後端集區。

1. 將後端集區關聯至包含 VM 的可用性設定組。

1. 在 [目標網路 IP 設定] 下方，檢查**虛擬機器**並選擇將加入為叢集節點的虛擬機器。 請務必包含將裝載 FCI 的所有虛擬機器。 

1. 按一下 [確定]  以建立後端集區。

### <a name="configure-a-load-balancer-health-probe"></a>設定負載平衡器健全狀況探查

1. 在 [負載平衡器] 刀鋒視窗中，按一下 [健全狀況探查]。

1. 按一下 [+ 新增]。

1. 在 [新增健全狀況探查] 刀鋒視窗中，<a name="probe"></a>設定健全狀況探查參數︰

   - **名稱**：健全狀況探查的名稱。
   - **通訊協定**：TCP。
   - **連接埠**︰設為可用的 TCP 連接埠。 此連接埠需要開啟防火牆的連接埠。 使用與您在防火牆設定健全狀況探查[相同的連接埠](#ports)。
   - **間隔**：5 秒。
   - **狀況不良臨界值**：2 次連續失敗。

1. 按一下 [確定]。

### <a name="set-load-balancing-rules"></a>設定負載平衡規則

1. 在 [負載平衡器] 刀鋒視窗中，按一下 [負載平衡規則]。

1. 按一下 [+ 新增]。

1. 設定負載平衡規則參數：

   - **名稱**︰負載平衡規則的名稱。
   - **前端 IP 位址**︰使用 SQL Server FCI 叢集網路資源的 IP 位址。
   - **連接埠**：設定為 SQL Server FCI TCP 連接埠。 預設執行個體連接埠為 1433。
   - **後端連接埠**：此值使用的連接埠與您啟用**浮動 IP (伺服器直接回傳)** 時的**連接埠**值相同。
   - **後端集區**︰使用您稍早設定的後端集區名稱。
   - **健全狀況探查**：使用您稍早設定的健全狀況探查。
   - **工作階段持續性**：無。
   - **閒置逾時 (分鐘)**：4。
   - **浮動 IP (伺服器直接回傳)**：已啟用。

1. 按一下 [確定]。

## <a name="step-6-configure-cluster-for-probe"></a>步驟 6：設定探查叢集

設定 PowerShell 中的叢集探查連接埠參數。

若要設定叢集探查連接埠參數，請將下列指令碼中的變數更新為您環境中的值。 從指令碼移除角括弧 `<>`。 

   ```PowerShell
   $ClusterNetworkName = "<Cluster Network Name>"
   $IPResourceName = "<SQL Server FCI IP Address Resource Name>" 
   $ILBIP = "<n.n.n.n>" 
   [int]$ProbePort = <nnnnn>

   Import-Module FailoverClusters

   Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   ```

在上述指令碼中，設定您環境的值。 下列清單說明這些值：

   - `<Cluster Network Name>`：Windows Server 容錯移轉叢集網路名稱。 在 **[容錯移轉叢集管理員]** > **[網路]** 中，以滑鼠右鍵按一下網路，然後按一下 [內容]。 正確的值在 [一般] 索引標籤的 [名稱] 底下。 

   - `<SQL Server FCI IP Address Resource Name>`：SQL Server FCI IP 位址資源名稱。 在 **[容錯移轉叢集管理員]** > **[角色]** 中，SQL Server FCI 角色的 [伺服器名稱] 底下，以滑鼠右鍵按一下 IP 位址資源，然後按一下 [內容]。 正確的值在 [一般] 索引標籤的 [名稱] 底下。 

   - `<ILBIP>`：ILB IP 位址。 此位址在 Azure 入口網站中會設定為 ILB 前端位址。 這也是 SQL Server FCI IP 位址。 您可以在 [容錯移轉叢集管理員] 中，您找到 `<SQL Server FCI IP Address Resource Name>` 所在位置的相同內容頁面上找到該位址。  

   - `<nnnnn>`：您在負載平衡器健全狀況探查中設定的探查連接埠。 任何未使用的 TCP 連接埠都有效。 

>[!IMPORTANT]
>叢集參數的子網路遮罩必須是 TCP IP 廣播位址：`255.255.255.255`。

設定叢集探查之後，您可以在 PowerShell 中查看所有叢集參數。 執行下列指令碼：

   ```PowerShell
   Get-ClusterResource $IPResourceName | Get-ClusterParameter 
  ```

## <a name="step-7-test-fci-failover"></a>步驟 7：測試 FCI 容錯移轉

測試 FCI 的容錯移轉來驗證叢集功能。 請執行下列步驟：

1. 透過 RDP 連接至其中一個 SQL Server FCI 叢集節點。

1. 開啟 [容錯移轉叢集管理員]。 按一下 [角色]。 請注意哪個節點擁有 SQL Server FCI 角色。

1. 以滑鼠右鍵按一下 SQL Server FCI 角色。

1. 按一下 [移動]，然後按一下 [最佳可行節點]。

**容錯移轉叢集管理員**會顯示角色，其資源則會變成離線狀態。 接著資源會移動，並在另一個節點上線。

### <a name="test-connectivity"></a>測試連線能力

若要測試連線能力，請在相同的虛擬網路內登入至另一部虛擬機器。 開啟 [SQL Server Management Studio]，然後連接到 SQL Server FCI 名稱。

>[!NOTE]
>如有需要，您可以[下載 SQL Server Management Studio](http://msdn.microsoft.com/library/mt238290.aspx)。

## <a name="limitations"></a>限制

Azure 虛擬機器支援 Windows Server 2019 上的 Microsoft 分散式交易協調器 (MSDTC)，並且可使用叢集共用磁碟區 (CSV) 上的儲存體和[標準負載平衡器](../../../load-balancer/load-balancer-standard-overview.md)。

在 Azure 虛擬機器上，Windows Server 2016 或更早版本無法支援 MSDTC，因為：

- 叢集 MSDTC 資源無法設定為使用共用儲存體。 若在 Windows Server 2016 上建立 MSDTC 資源，即使有共用儲存體存在，系統也不會顯示任何可用的共用儲存體。 Windows Server 2019 中已修正此問題。
- 基本負載平衡器不會處理 RPC 連接埠。

## <a name="see-also"></a>另請參閱

[透過遠端桌面 (Azure) 設定 S2D](http://technet.microsoft.com/windows-server-docs/compute/remote-desktop-services/rds-storage-spaces-direct-deployment)

[具有儲存空間直接存取的超交集解決方案](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/hyper-converged-solution-using-storage-spaces-direct)。

[儲存空間直接存取概觀](http://technet.microsoft.com/windows-server-docs/storage/storage-spaces/storage-spaces-direct-overview)

[適用於 S2D 的 SQL Server 支援](https://blogs.technet.microsoft.com/dataplatforminsider/2016/09/27/sql-server-2016-now-supports-windows-server-2016-storage-spaces-direct/)
