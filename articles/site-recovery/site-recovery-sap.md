---
title: 使用 Azure Site Recovery 保護多層式 SAP NetWeaver 應用程式部署 | Microsoft Docs
description: 本文說明如何使用 Azure Site Recovery 保護 SAP NetWeaver 應用程式部署。
services: site-recovery
documentationcenter: ''
author: asgang
manager: rochakm
editor: ''
ms.assetid: ''
ms.service: site-recovery
ms.workload: backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2018
ms.author: asgang
ms.openlocfilehash: a498ac9f973bbcf87bec104f18b542cc7e8b5800
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318685"
---
# <a name="protect-a-multi-tier-sap-netweaver-application-deployment-by-using-site-recovery"></a>使用 Site Recovery 保護多層式 SAP NetWeaver 應用程式部署

大部分的大型與中型 SAP 部署都有某種形式的災害復原解決方案。 隨著移至應用程式 (如 SAP) 的核心商務程序愈來愈多，強固且可測試的災害復原解決方案愈形重要。 Azure Site Recovery 已經過測試並與 SAP 應用程式整合。 Site Recovery 超越大部分內部部署災害復原解決方案的功能，且具有比競爭解決方案低的總體擁有成本 (TCO)，。

使用 Site Recovery，您可以：
* 藉由將元件複寫到 Azure，**啟用在內部部署環境執行之 SAP NetWeaver 和非 NetWeaver 生產應用程式的保護功能**。
* 藉由將元件複寫到另一個 Azure 資料中心，**啟用執行 Azure 之 SAP NetWeaver 和非 NetWeaver 生產應用程式的保護功能**。
* 使用 Site Recovery 將您的 SAP 部署移轉至 Azure，來**簡化雲端移轉**。
* 藉由建立隨選生產複本來測試 SAP 應用程式，**簡化 SAP 專案升級、測試和原型設計**。

本文說明如何使用 [Azure Site Recovery](site-recovery-overview.md) 保護 SAP NetWeaver 應用程式部署。 本文涵蓋在 Azure 上保護三層式 SAP NetWeaver 部署的最佳做法，也就是使用 Site Recovery 來複寫至另一個 Azure 資料中心。 其說明支援的案例和組態，以及如何執行測試容錯移轉 (災害復原演練) 和實際的容錯移轉。

## <a name="prerequisites"></a>必要條件
在開始之前，請確定您了解如何執行下列工作：

* [將虛擬機器複寫至 Azure](azure-to-azure-walkthrough-enable-replication.md)
* [設計復原網路](site-recovery-azure-to-azure-networking-guidance.md)
* [執行測試容錯移轉至 Azure](azure-to-azure-walkthrough-test-failover.md)
* [執行容錯移轉到 Azure](site-recovery-failover.md)
* [複寫網域控制站](site-recovery-active-directory.md)
* [複寫 SQL Server](site-recovery-sql.md)

## <a name="supported-scenarios"></a>支援的案例
您可以使用 Site Recovery，在下列案例中實作災害復原解決方案：
* 在一個 Azure 資料中心執行並複寫至另一個 Azure 資料中心 (Azure 對 Azure 災害復原) 的 SAP 系統。 如需詳細資訊，請參閱 [Azure 對 Azure 複寫架構](https://aka.ms/asr-a2a-architecture)。
* 在內部部署 VMware (或實體) 伺服器上執行並複寫到 Azure 資料中心內災害復原站台 (VMware 對 Azure 災害復原) 的 SAP 系統。 此案例需要一些額外的元件。 如需詳細資訊，請參閱 [VMware 對 Azure 複寫架構](https://aka.ms/asr-v2a-architecture)。
* 在內部部署 Hyper-V 上執行並複寫到 Azure 資料中心內災害復原站台 (Hyper-V 對 Azure 災害復原) 的 SAP 系統。 此案例需要一些額外的元件。 如需詳細資訊，請參閱 [Hyper-V 對 Azure 複寫架構](https://aka.ms/asr-h2a-architecture)。

在本文中，我們會使用 **Azure 對 Azure** 災害復原案例來說明 Site Recovery 的 SAP 災害復原功能。 由於 Site Recovery 複寫不限於特定應用程式，因此所述程序應也可以套用至其他案例。

### <a name="required-foundation-services"></a>必要的基礎服務
我們將在本文討論的案例中，部署下列基礎服務：
* Azure ExpressRoute 或 Azure VPN 閘道
* 至少一個在 Azure 中執行的 Active Directory 網域控制站和 DNS 伺服器

我們建議您先建立此基礎結構，再部署 Site Recovery。

## <a name="reference-sap-application-deployment"></a>參考 SAP 應用程式部署

此參考架構會顯示在 Windows 環境中執行於高可用性 Azure 上的 SAP NetWeaver。  此架構以特定虛擬機器 (VM) 大小進行部署，大小可以變更以符合您的組織需求。

![典型 SAP 部署模式的圖表](./media/site-recovery-sap/reference_sap.png)

## <a name="disaster-recovery-considerations"></a>災害復原考量

針對災害復原 (DR)，您必須能夠容錯移轉到次要區域。 每一層會使用不同的策略來提供災害復原 (DR) 保護。

#### <a name="vms-running-sap-web-dispatcher-pool"></a>執行 SAP Web Dispatcher 集區的 VM 
Web Dispatcher 元件是用來作為 SAP 應用程式伺服器之間 SAP 流量的負載平衡器。 若要達到 Web Dispatcher 元件的高可用性，系統會使用 Azure Load Balancer，針對平衡器集區中可用 Web Dispatcher 之間的 HTTP(S) 流量分配，在循環配置資源組態中實作平行 Web Dispatcher 設定。 系統會使用 Azure Site Recovery(ASR) 進行其複寫，並使用自動化指令碼在災害復原區域上設定負載平衡器。 

#### <a name="vms-running-application-servers-pool"></a>執行應用程式伺服器集區的 VM
若要管理 ABAP 應用程式伺服器的登入群組，請使用 SMLG 交易。 它會在中央服務的訊息伺服器內使用負載平衡函式，以針對 SAPGUI 和 RFC 流量，分配 SAP 應用程式伺服器集區之間的工作負載。 系統會使用 Azure Site Recovery 進行其複寫 

#### <a name="vms-running-sap-central-services-cluster"></a>執行 SAP 中央服務叢集的 VM
此參考架構會在應用程式層中的虛擬機器上執行中央服務。 部署到單一虛擬機器 (不需要高可用性時的典型部署) 時，中央服務是潛在的單一失敗點 (SPOF)。<br>

若要實作高可用性解決方案，可以使用共用磁碟叢集或檔案共用叢集。若要設定共用磁碟叢集的 VM，請使用 Windows Server 容錯移轉叢集。 建議將雲端見證作為仲裁見證。 
 > [!NOTE]
 > Azure Site Recovery 並不會複寫雲端見證，因此建議您在災害復原區域中部署雲端見證。

若要支援容錯移轉叢集環境，[SIOS DataKeeper 叢集版本](https://azuremarketplace.microsoft.com/marketplace/apps/sios_datakeeper.sios-datakeeper-8)會執行叢集共用磁碟區函式，方法為複寫叢集節點所擁有的獨立磁碟。 Azure 本身不支援共用磁碟，因此需要 SIOS 所提供的解決方案。 

處理叢集的另一種方式，是實作檔案共用叢集。 [SAP](https://blogs.sap.com/2018/03/19/migration-from-a-shared-disk-cluster-to-a-file-share-cluster) 最近修改了中央服務部署模式，來透過 UNC 路徑存取 /sapmnt 全域目錄。 不過，仍然建議確保 /sapmnt UNC 共用具有高可用性。 您可以藉由使用 Windows Server 容錯移轉叢集與相應放大檔案伺服器 (SOFS) 和 Windows Server 2016 中的儲存空間直接存取 (S2D) 功能，在中央服務執行個體上完成這項操作。 
 > [!NOTE]
 > 目前，Azure Site Recovery 僅支援使用「儲存空間直接存取」對虛擬機器進行「當機時保持一致復原點」複寫 


## <a name="disaster-recovery-considerations"></a>災害復原考量

您可以使用 Azure Site Recovery 在各個 Azure 區域間協調完整 SAP 部署的容錯移轉。
以下是設定災害復原的步驟 

1. 複寫虛擬機器 
2. 設計復原網路
3.  複寫網域控制站
4.  將資料複寫到基本層 
5.  執行測試容錯移轉 
6.  執行容錯移轉 

以下我們針對此範例中使用的各層提供的災害復原建議。 

 **SAP 層** | **建議**
 --- | ---
**SAP Web Dispatcher 集區** |  使用 Site Recovery 進行複寫 
**SAP 應用程式伺服器集區** |  使用 Site Recovery 進行複寫 
**SAP 中央服務叢集** |  使用 Site Recovery 進行複寫 
**Active Directory 虛擬機器** |  Active Directory 複寫 
**SQL 資料庫伺服器** |  SQL Always On 複寫

## <a name="replicate-virtual-machines"></a>複寫虛擬機器

若要開始將所有 SAP 應用程式虛擬機器都複寫至 Azure 災害復原資料中心，請遵循[將虛擬機器複寫至 Azure](azure-to-azure-walkthrough-enable-replication.md) 中的指引。


* 如需保護 Active Directory 和 DNS 的指引，請參閱[保護 Active Directory 和 DNS](site-recovery-active-directory.md) 文件。

* 如需保護在 SQL Server 上執行之資料庫層的指引，請參閱[保護 SQL Server](site-recovery-active-directory.md) 文件。

## <a name="networking-configuration"></a>網路設定

如果您使用靜態 IP 位址，您可以指定您想要虛擬機器使用的 IP 位址。 若要設定 IP 位址，請移至 [計算和網路設定] > [網路介面卡]。

![示範如何在 Site Recovery 網路介面卡窗格設定私人 IP 位址的螢幕擷取畫面](./media/site-recovery-sap/sap-static-ip.png)


## <a name="creating-a-recovery-plan"></a>建立復原計劃
復原方案支援在容錯移轉期間對多層式應用程式中的各層進行排序。 排序有助於維持應用程式的一致性。 當您為多層式 Web 應用程式建立復原方案時，請完成[使用 Site Recovery 建立復原方案](site-recovery-create-recovery-plans.md)中所述的步驟。

### <a name="adding-virtual-machines-to-failover-groups"></a>將虛擬機器新增至容錯移轉群組

1.  新增應用程式伺服器、Web Dispatcher 和 SAP 中央服務 VM，以建立復原方案。
2.  按一下 [自訂] 將 VM 分組。 根據預設，所有 VM 都是「群組 1」的一部分。



### <a name="add-scripts-to-the-recovery-plan"></a>將指令碼新增至復原計畫
為了讓應用程式可正常運作，您可能需要在容錯移轉後或測試容錯移轉期間，於 Azure 虛擬機器上執行某些作業。 您可以自動化某些容錯移轉後的作業。 例如，您可以透過將對應的指令碼新增至復原計畫，以更新 DNS 項目和變更繫結與連線。


您可以按一下底下的 [部署至 Azure] 按鈕，將最常用的 Azure Site Recovery 指令碼部署至您的自動化帳戶。 當您使用任何已發佈的指令碼時，請務必遵循指令碼中的指引。

[![部署至 Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

1. 將動作前指令碼新增至「群組 1」，以容錯移轉 SQL 可用性群組。 使用在範例指令碼中發佈的 'ASR-SQL-FailoverAG' 指令碼。 請務必遵循指令碼中的指引，並在指令碼中適當地進行必要的變更。
2. 新增動作後指令碼，以連結 Web 層 (群組 1) 已容錯移轉虛擬機器上的負載平衡器。 使用在範例指令碼中發佈的 'ASR-AddSingleLoadBalancer' 指令碼。 請務必遵循指令碼中的指引，並在指令碼中適當地進行必要的變更。

![SAP 復原方案](./media/site-recovery-sap/sap_recovery_plan.png)


## <a name="run-a-test-failover"></a>執行測試容錯移轉

1.  在 Azure 入口網站中，選取您的復原服務保存庫。
2.  選取為 SAP 應用程式建立的復原計畫。
3.  選取 [測試容錯移轉]。
4.  若要啟動測試容錯移轉程序，請選取復原點和 Azure 虛擬網路。
5.  當次要環境啟動時，即可執行您驗證。
6.  完成驗證後，若要清除此容錯移轉環境，請選取 [清除測試容錯移轉]。

如需詳細資訊，請參閱[在 Site Recovery 中測試容錯移轉至 Azure](site-recovery-test-failover-to-azure.md)。

## <a name="run-a-failover"></a>執行容錯移轉

1.  在 Azure 入口網站中，選取您的復原服務保存庫。
2.  選取為 SAP 應用程式建立的復原計畫。
3.  選取 [容錯移轉]。
4.  若要啟動容錯移轉程序，請選取復原點。

如需詳細資訊，請參閱[在 Site Recovery 中容錯移轉](site-recovery-failover.md)。

## <a name="next-steps"></a>後續步驟
* 若要深入了解如何使用 Site Recovery 為 SAP NetWeaver 部署建置災害復原解決方案，請參閱可下載的白皮書＜[SAP NetWeaver：使用 Azure Site Recovery 建置災害復原方案](http://aka.ms/asr-sap)＞。 本白皮書會討論各種 SAP 架構的建議，列出支援 Azure 上的 SAP 的應用程式和 VM 類型，並說明災害復原解決方案的測試計劃選項。
* 深入了解如何使用 Site Recovery [複寫其他工作負載](site-recovery-workload.md)。
