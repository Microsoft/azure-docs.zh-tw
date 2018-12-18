---
title: 適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量 | Microsoft Docs
description: 適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/06/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e46503f8dc97f58db1cd5acfd2122e2895fb15b0
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44162303"
---
# <a name="considerations-for-azure-virtual-machines-dbms-deployment-for-sap-workload"></a>適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量
[1114181]:https://launchpad.support.sap.com/#/notes/1114181
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2069760]:https://launchpad.support.sap.com/#/notes/2069760
[2171857]:https://launchpad.support.sap.com/#/notes/2171857
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[deployment-guide]:deployment-guide.md 
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e 
[planning-guide]:planning-guide.md 

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png


[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

本指南是在 Microsoft Azure 上實作和部署 SAP 軟體之文件的一部分。 閱讀本指南之前，請先閱讀[規劃和實作指南][planning-guide]。 本文件說明如何使用 Azure 基礎結構即服務 (IaaS) 功能，在 Microsoft Azure 虛擬機器 (VM) 上 SAP 相關 DBMS 系統的一般部署層面。

本白皮書會對「SAP 安裝文件」和「SAP 附註」進行補充說明，指出用來在指定平台上安裝和部署 SAP 軟體的主要資源。

本文件說明在 Azure VM 中執行 SAP 相關 DBMS 系統的考量。 有數個對於本章中特定 DBMS 系統的參考資料。 在本白皮書內，改為在本文件之後討論特定的 DBMS 系統。

## <a name="definitions-upfront"></a>預先定義
本文件中使用下列詞彙︰

* IaaS：基礎結構即服務。
* PaaS：平台即服務。
* SaaS：軟體即服務。
* SAP 元件︰個別的 SAP 應用程式，例如 ECC、BW、Solution Manager 或 EP。  SAP 元件可以傳統 ABAP 或 Java 技術為基礎，或以非 NetWeaver 應用程式 (例如商務物件) 為基礎。
* SAP 環境 (SAP Environment)︰一或多個以邏輯方式分組的 SAP 元件，可執行像是開發、QAS、訓練、DR 或生產等商務功能。
* SAP 架構 (SAP Landscape)︰此詞彙是指客戶 IP 環境中的整個 SAP 資產。 SAP 環境包含所有生產和非生產環境。
* SAP 系統 (SAP System)︰DBMS 層與應用程式層的組合，例如 SAP ERP 開發系統、SAP BW 測試系統、SAP CRM 生產系統等。在 Azure 部署中，不支援在內部部署與 Azure 之間分割這兩個層級。 這表示 SAP 系統可以在內部部署或在 Azure 部署。 不過，您可以將具 SAP 結構的不同系統部署於 Azure 或內部部署。 例如，您可以在 Azure 部署 SAP CRM 開發和測試系統，但在內部部署 SAP CRM 生產系統。
* 跨單位：描述將 VM 部署到 Azure 訂用帳戶的案例，該訂用帳戶在內部部署資料中心與 Azure 之間具有站對站、多站台或 ExpressRoute 連線能力。 在一般 Azure 文件中，這類部署也會描述為跨單位案例。 連線的原因是為了將內部部署網域、內部部署 Active Directory 和內部部署 DNS 擴充到 Azure。 內部部署的架構會擴充到訂用帳戶的 Azure 資產。 在此擴充下，VM 可以是內部部署網域的一部分。 內部部署網域的網域使用者可以存取伺服器，並且可在這些 VM 上執行服務 (例如 DBMS 服務)。 您可以在內部部署的 VM 與 Azure 中部署的 VM 之間進行通訊與名稱解析。 此案例是在 Azure 上部署 SAP 資產的最常見案例。 如需詳細資訊，請參閱[規劃與設計 VPN 閘道](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-plan-design)。

> [!NOTE]
> SAP 生產系統支援跨單位部署 SAP 系統，其中執行 SAP 系統的 Azure 虛擬機器是內部部署網域的成員。 支援跨單位組態，以便將部分或完整的 SAP 架構部署到 Azure。 即使在 Azure 中執行完整的 SAP 架構，也會要求這些 VM 隸屬於內部部署網域和 AD/LDAP。 在本文件的先前版本中，曾經提到混合式 IT 案例，其中「混合式」一詞基本上是指內部部署與 Azure 之間有跨單位連線能力。 在此案例中，*混合式*也表示 Azure 中的 VM 是內部部署 Active Directory 的一部分。
> 
> 

有些 Microsoft 文件在描述跨單位案例時稍有不同，特別是針對 DBMS HA 組態。 在 SAP 相關的文件中，跨單位案例會縮減為站對站或私人 [ExpressRoute](https://azure.microsoft.com/services/expressroute/) 連線能力，以及實際上 SAP 架構會分散至內部部署與 Azure 之間。

## <a name="resources"></a>資源
目前有各種公佈的文章提到 Azure 上的 SAP 工作負載。  建議您先從[Azure 上的 SAP 工作負載 - 入門](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/get-started)開始，然後挑選感興趣的領域

下列的 SAP 附註，在本文所討論的領域上與 Azure 上的 SAP 有關：

| 附註編號 | 標題 |
| --- | --- |
| [1928533] |Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型 |
| [2015553] |Microsoft Azure 上的 SAP：支援的必要條件 |
| [1999351] |疑難排解適用於 SAP 且已強化的 Azure 監視功能 |
| [2178632] |Microsoft Azure 上的 SAP 主要監視度量 |
| [1409604] |Windows 上的虛擬化︰增強型監視功能 |
| [2191498] |Linux 和 Azure 上的 SAP：增強型監視功能 |
| [2039619] |Microsoft Azure 上使用 Oracle 資料庫的 SAP 應用程式︰支援的產品和版本 |
| [2233094] |DB6︰Azure 上使用 IBM DB2 for Linux, UNIX, and Windows 的應用程式 - 其他資訊 |
| [2243692] |Microsoft Azure (IaaS) VM 上的 Linux：SAP 授權問題 |
| [1984787] |SUSE LINUX Enterprise Server 12：安裝注意事項 |
| [2002167] |Red Hat Enterprise Linux 7.x：安裝和升級 |
| [2069760] |Oracle Linux 7.x SAP 安裝和升級 |
| [1597355] |適用於 Linux 的交換空間建議 |
| [2171857] |Oracle Database 12c - Linux 上的檔案系統支援 |
| [1114181] |Oracle Database 11g - Linux 上的檔案系統支援 |


另請參閱 [SCN Wiki](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes)，其中包含適用於 Linux 的所有 SAP 附註。

您應該具備有關 Microsoft Azure 架構以及如何部署和操作 Microsoft Azure 虛擬機器的有效知識。 您可以在 [Azure 文件](https://docs.microsoft.com/azure/)找到詳細資訊。

雖然所討論的是 IaaS，但一般而言，Windows、Linux 和 DBMS 的安裝和組態基本上與您要安裝內部部署的任何虛擬機器或裸機機器相同。 不過，有一些架構和系統管理實作決策會與使用 Azure IaaS 時的不同。 本文件的目的是說明您在使用 Azure IaaS 時必須備妥的特定架構和系統管理差異。


## <a name="65fa79d6-a85f-47ee-890b-22e794f51a64"></a>RDBMS 部署的 VM 儲存體結構
為了依循本章內容，必須了解[部署指南][deployment-guide]的[這個][deployment-guide-3]章節所提供的內容。 閱讀本章之前，必須先了解並熟悉有關不同的 VM 系列及其差異，以及 Azure 標準和[進階儲存體](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage)差異的知識。

就 Azure VM 的 Azure 儲存體方面來說，您應該熟悉下列文章：

- [關於 Azure Windows VM 的磁碟儲存體](https://docs.microsoft.com/azure/virtual-machines/windows/about-disks-and-vhds)
- [有關 Azure Linux VM 的磁碟儲存體](https://docs.microsoft.com/azure/virtual-machines/linux/about-disks-and-vhds) 

在基本組態中，我們通常建議以下部署結構：作業系統、DBMS 和最終的 SAP 二進位檔會與資料庫檔案分隔開來。 因此，我們建議在 Azure 虛擬機器中執行的 SAP 系統使用作業系統、資料庫管理系統可執行檔和 SAP 可執行檔來安裝基底 VHD (或磁碟)。 DBMS 資料和記錄檔會儲存於個別磁碟中的 Azure 儲存體 (標準或進階儲存體)，並以邏輯磁碟形式連接到原始的 Azure 作業系統映像 VM。 尤其在 Linux 部署中，可能會記載不同的建議。 特別以 SAP HANA 為主。  

規劃磁碟配置時，您必須找到下列各項的最佳平衡︰

* 資料檔案數量。
* 包含檔案的磁碟數目。
* 單一磁碟的 IOPS 配額。
* 每個磁碟的資料輸送量。
* 每個 VM 大小可能的額外資料磁碟數目。
* VM 可提供的整體儲存體輸送量。
* 不同的 Azure 儲存體類型可以提供的延遲。
* VM SLA

Azure 會強制執行每個資料磁碟的 IOPS 配額。 這些配額與 Azure 標準儲存體和進階儲存體上裝載的磁碟不同。 I/O 延遲在兩個儲存體類型之間也有所不同。  使用進階儲存體，提供更佳的 I/O 延遲。 不同 VM 類型的每一個可提供來讓您連接的資料磁碟數目有限。 另一個限制是只有特定的 VM 類型可以利用 Azure 進階儲存體。 因此，適用於特定 VM 類型的決策可能不只受到 CPU 和記憶體需求影響，而且也會受到 IOPS、延遲及磁碟輸送量需求影響，這些需求通常是利用磁碟數目或進階儲存體磁碟的類型來調整。 特別是進階儲存體，磁碟的大小可能也會受到每個磁碟需要到達的 IOPS 數目和輸送量所支配。

> [!NOTE]
> 針對 DBMS 部署，強烈建議將進階儲存體使用於任何資料、交易記錄或重做檔案。 因此，不論您想要部署生產或非生產系統，都沒有差別。

> [!NOTE]
> 若要受益於 Azure 的獨特[單一 VM SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/)，所有連結的磁碟類型都必須為 Azure 進階儲存體，包括基底 VHD。
>

資料庫檔案和記錄/重做檔案的位置以及所使用的 Azure 儲存體類型應該依據 IOPS、延遲和輸送量需求來定義。 為了有足夠的 IOPS，您可能被迫利用多個磁碟或使用更大的進階儲存體磁碟。 如果使用多個磁碟，您會建置橫跨多個磁碟的軟體等量磁碟區，其中包含資料檔案或記錄/重做檔案。 在這類情況下，基礎進階儲存體磁碟的 IOPS 和磁碟輸送量，或Azure 標準儲存體磁碟的最大可達成 IOPS 都會針對所產生的等量磁碟區組累加。 

如先前所述，如果 IOPS 需求超過單一 VHD 可提供的，您需要平衡橫跨多個 VHD 的資料庫檔案所需的 IOPS 數目。 將 IOPS 負載分散於各磁碟的最簡單方式，就是在不同的磁碟上建置軟體等量磁碟區。 然後在劃分出軟體等量磁碟區的 LUN 上放置多個 SAP DBMS 的資料檔。 等量磁碟區中的磁碟數目取決於 IOP 需求、磁碟輸送量需求，以及磁碟區需求。 


- - -
> ![Windows][Logo_Windows] Windows
> 
> 我們建議使用 Windows 儲存空間來建立橫跨多個 Azure VHD 的這類等量磁碟區組。 建議至少使用 Windows Server 2012 R2 或 Windows Server 2016。
> 
> ![Linux][Logo_Linux] Linux
> 
> 只支援使用 MDADM 和 LVM (邏輯磁碟區管理員) 在 Linux 上建立軟體 RAID。 如需詳細資訊，請參閱下列文章：
> 
> - 使用 MDADM [在 Linux 上設定軟體 RAID](https://docs.microsoft.com/azure/virtual-machines/linux/configure-raid)
> - 使用 LVM [在 Azure 中的 Linux VM 上設定 LVM](https://docs.microsoft.com/azure/virtual-machines/linux/configure-lvm)
> 
> 

- - -
 
> [!NOTE]
> 由於 Azure 儲存體會保留三個 VHD 映像，所以在等量配置時設定備援並不合理。 您只需要設定等量配置，以便讓 I/O 分散於不同的 VHD。
>

### <a name="managed-or-non-managed-disks"></a>受控或非受控磁碟
Azure 儲存體帳戶不只是一個系統管理的建構，而且還是限制的緣由。 Azure 標準儲存體帳戶與 Azure 進階儲存體帳戶的限制有所不同。 如需了解確切的功能和限制，請參閱 [Azure 儲存體延展性和效能目標](https://docs.microsoft.com/azure/storage/common/storage-scalability-targets)一文

就 Azure 標準儲存體而言，請特別記得每個儲存體帳戶都有 IOPS 限制 ([Azure 儲存體延展性和效能目標](https://docs.microsoft.com/azure/storage/common/storage-scalability-targets)一文中包含「總要求率」的資料列)。 此外，每個 Azure 訂用帳戶都有儲存體帳戶數目初始限制。 因此，您需要將較大 SAP 架構的 VHD 平衡分散於不同的儲存體帳戶，以避免達到這些儲存體帳戶的限制。 討論有超過上千個 VHD 的數百個虛擬機器，是一項令人乏味的工作。 

因為不建議使用 DBMS 部署的 SAP 工作負載搭配 Azure 標準儲存體，Azure 標準儲存體的參考和建議僅限於此簡短的[文章](https://blogs.msdn.com/b/mast/archive/2014/10/14/configuring-azure-virtual-machines-for-optimal-storage-performance.aspx)

為了避免規劃並將 VHD 部署於不同 Azure 儲存體帳戶的系統管理工作，Microsoft 在 2017 年引進了所謂的[受控磁碟](https://azure.microsoft.com/services/managed-disks/)。 受控磁碟適用於 Azure 標準儲存體，以及 Azure 進階儲存體。 相較於非受控磁碟，受控磁碟的主要優點如下：

- 對於受控磁碟，Azure 會在部署階段自動將不同的 VHD 分散於不同的儲存體帳戶，因而避免達到 Azure 儲存體帳戶在資料磁碟區、I/O 輸送量和 IOPS 等方面的限制。
- 使用受控磁碟，Azure 儲存體會接受 Azure 可用性設定組的概念，而如果 VM 是 Azure 可用性設定組的一部分，則會將 VM 的基底 VHD 和 連結的磁碟部署到不同的容錯和更新網域。


> [!IMPORTANT]
> 考慮到 Azure 受控磁碟的優點，通常強烈建議將 Azure 受控磁碟使用於 DBMS 部署和 SAP 部署。
>

若要從非受控磁碟轉換為受控磁碟，請參閱下列文章：

- [將 Windows 虛擬機器從非受控磁碟轉換成受控磁碟](https://docs.microsoft.com/azure/virtual-machines/windows/convert-unmanaged-to-managed-disks)
- [將 Linux 虛擬機器從非受控磁碟轉換成受控磁碟](https://docs.microsoft.com/azure/virtual-machines/linux/convert-unmanaged-to-managed-disks)


### <a name="c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f"></a>VM 和資料磁碟的快取
當您將磁碟掛接到 VM 時，可以選擇是否快取在 VM 與位於 Azure 儲存體之磁碟間的 I/O 流量。 Azure 標準和進階儲存體會針對這種快取類型使用兩種不同技術。 

下列建議假設標準 DBMS 的 I/O 特性如下：

- 主要是對資料庫資料檔案的讀取工作負載。 這些讀取對於 DBMS 系統的效能有重大影響
- 根據檢查點或持續串流，對資料檔案的寫入會發生高載。 不過，平均一天的寫入為小於讀取。 相對於來自資料檔案的讀取，這些寫入是非同步的，且不會造成任何使用者交易延遲。
- 幾乎不會從交易記錄或重做檔案進行任何讀取。 例外狀況是在執行交易記錄備份時的大型 I/O。 
- 交易或重做記錄檔的主要負載是寫入。 根據工作負載的性質，您的 I/O 可以小到 4 KB，而在其他情況下，I/O 大小也可以是 1 MB 或更大。
- 所有寫入都必須以可靠的方式保存在磁碟上

針對 Azure 標準儲存體，可能的快取類型如下︰

* None
* 讀取
* 讀取/寫入

為了取得一致且具決定性的效能，您應該在「Azure 標準儲存體」上，針對包含 **DBMS 相關資料檔、記錄/重做檔和資料表空間的所有磁碟，將快取設定為「無」**。 基底 VHD 的快取可以保留預設值。

針對 Azure 進階儲存體，提供下列快取選項︰

* None
* 讀取 
* 讀取/寫入 
* 無 + 寫入加速器 (僅適用於 Azure M 系列 VM)
* 讀取 + 寫入加速器 (僅適用於 Azure M 系列 VM)

對於「Azure 進階儲存體」的建議是利用 SAP 資料庫的「資料檔案讀取快取」，並選擇「不對記錄檔的磁碟進行快取」。

對於 M 系列部署，強烈建議將 Azure 寫入加速器使用於 DBMS 部署。 如需 Azure 寫入加速器的限制和部署詳細資訊，請參閱[寫入加速器](https://docs.microsoft.com/azure/virtual-machines/windows/how-to-enable-write-accelerator)文件。 


### <a name="azure-non-persistent-disks"></a>Azure 非永續性磁碟
Azure VM 會在部署 VM 之後提供非永續性磁碟。 如果 VM 重新開機，將會抹除這些磁碟機上的所有內容。因此，假設在任何情況下，資料庫的資料檔案和記錄/重做檔案都不得位於這些非持續性磁碟機上。 有些資料庫可能會有一些例外狀況，在這類狀況下這些非持續性磁碟機可能適合用於 tempdb 和暫存資料表空間。 不過，請避免將這些磁碟機使用於 A 系列 VM，因為這些非持續性磁碟機對於該 VM 系列的輸送量受限。 如需詳細資訊，請閱讀[了解 Windows Azure 虛擬機器上的暫存磁碟機](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)

- - -
> ![Windows][Logo_Windows] Windows
> 
> Azure VM 中的磁碟機 D:\ 不是永續性磁碟機，而是由 Azure 計算節點上的一些本機磁碟所組成。 由於它不是永續性的，這表示當 VM 重新開機時，即會遺失對 D:\ 磁碟機的內容所做的任何變更。 「任何變更」就如同已儲存的檔案、已建立的目錄、已安裝的應用程式等。
> 
> ![Linux][Logo_Linux] Linux
> 
> Linux Azure VM 會在 /mnt/resource 上自動掛接磁碟機，這個非永續性磁碟機會利用 Azure 計算節點上的本機磁碟來備份。 由於它不是永續性的，這表示當 VM 重新開機時，會遺失對 /mnt/resource 的內容所做的任何變更。 任何變更就如同已儲存的檔案、已建立的目錄、已安裝的應用程式等。
> 
> 

- - -



### <a name="10b041ef-c177-498a-93ed-44b3441ab152"></a>Microsoft Azure 儲存體復原
Microsoft Azure 儲存體會將基底 VHD (含 OS) 以及連結的磁碟或 BLOB 儲存於至少 3 個不同的儲存體節點。 這項事實稱為「本機備援儲存體 (LRS)」。 LRS 是 Azure 中所有儲存體類型的預設值。 

[Azure 儲存體複寫](https://docs.microsoft.com/azure/storage/common/storage-redundancy?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)一文中有其他數個備援方法的說明。

> [!NOTE]
>至於 Azure 進階儲存體 (這是建議的儲存體類型，適用於 DBMS VM 和儲存資料庫和記錄/重做檔案的磁碟)，唯一可用的方法為 LRS。 因此，您必須設定資料庫方法 (例如 SQL Server Always On、Oracle Data Guard 或 HANA System Replication)，才能將資料庫資料複寫到另一個 Azure 區域或另一個 Azure 可用性區域。


> [!NOTE]
> 對於 DBMS 部署，不建議使用 Azure 標準儲存體可用的異地備援儲存體，因為它有嚴重的效能影響，而且不接受跨越 VM 所連結的不同 VHD 的寫入順序。 如果資料庫和記錄/重做檔案散布於來源 VM 端上的多個 VHD (大部分的情況)，不接受跨越不同 VHD 的寫入順序的這項事實，極有可能在複寫目標端造成不一致的資料庫。

 

## <a name="vm-node-resiliency"></a>VM 節點復原
Azure 平台會提供適用於 VM 的數個不同 SLA。 如需確切的詳細資訊，請參閱最新版的[虛擬機器的 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/)。 由於 DBMS 層通常是 SAP 系統的重要可用性部分，所以您必須熟悉可用性設定組、可用性區域和維護事件的概念。 說明上述所有概念的文章包含[管理 Azure 中 Windows 虛擬機器的可用性](https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability)和[管理 Linux 虛擬機器的可用性](https://docs.microsoft.com/azure/virtual-machines/linux/manage-availability)。  

對於具有 SAP 工作負載的生產 DBMS 案例，最低建議如下：

- 在相同 Azure 區域中的個別可用性設定組中部署兩個 VM。
- 這兩個 VM 會在相同的 Azure VNet 中執行，而且會從相同的子網路連結 NIC。
- 使用資料庫方法讓第二部 VM 保持熱待命。 方法可以是 SQL Server Always On、Oracle Data Guard 或 HANA System Replication。

此外，您可以在另一個 Azure 區域中部署第三個 VM，並使用相同的資料庫方法在另一個 Azure 區域中提供非同步複本。

這個[教學課程](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets)會示範設定 Azure 可用性設定組的方式。



## <a name="azure-network-considerations"></a>Azure 網路考量事項 
在大規模的 SAP 部署中，建議您將 [Azure 虛擬資料中心](https://docs.microsoft.com/azure/architecture/vdc/networking-virtual-datacenter)的藍圖使用於其 VNet 組態，以及其不同組織部分的權限和角色指派。

最佳做法有數種，其出自於數百個客戶部署：

- 部署 SAP 應用程式的 VNet 沒有網際網路存取權。
- 資料庫 VM 會在與應用程式層相同的 VNet 中執行。
- VNet 內的 VM 具有私人 IP 位址的靜態配置。 請參考 [Azure 中的 IP 位址類型及配置方法](https://docs.microsoft.com/azure/virtual-network/virtual-network-ip-addresses-overview-arm)一文。
- DBMS VM 的往返路由限制**不會**與安裝在本機 DBMS VM 上的防火牆一起設定。 流量路由反而會與 [Azure 網路安全性群組 (NSG)](https://docs.microsoft.com/azure/virtual-network/security-overview)一起定義
- 為了區隔和隔離 DBMS VM 的流量，您可以將不同的 NIC 指派給此 VM。 其中的每個 NIC 有不同的 IP 位址，而每個 NIC 會指派給不同的 VNet 子網路，於是有不同的 NSG 規則。 請記住，隔離或區隔網路流量只是一種路由措施，不得設定網路輸送量的配額。

> [!NOTE]
> 您應該透過 Azure 方式將靜態 IP 位址指派給個別 vNIC。 您不應該將客體 OS 內的靜態 IP 位址指派給 vNIC。 某些 Azure 服務 (例如 Azure 備份服務) 依賴至少將主要 vNIC 設為 DHCP 的事實，而不是設為靜態 IP 位址。 另請參閱[針對 Azure 虛擬機器備份進行疑難排解](https://docs.microsoft.com/azure/backup/backup-azure-vms-troubleshoot#networking)文件。 如果您需要將多個靜態 IP 位址指派給 VM，必須將多個 vNIC 指派給 VM。
>
>

將兩部 VM 使用於 Azure 可用性設定組內的生產 DBMS 部署，加上 SAP 應用程式層的個別路由，以及兩部 DBMS VM 的管理和操作流量，概略圖表如下所示：

![兩個子網路中兩部 VM 的圖表](./media/virtual-machines-shared-sap-deployment-guide/general_two_dbms_two_subnets.PNG)


### <a name="azure-load-balancer-for-redirecting-traffic"></a>用於重新導向流量的 Azure Load Balancer
需要設定 Azure Load Balancer，才能使用 SQL Server Always On 或 HANA System Replication 等功能中使用的私人虛擬 IP 位址。 Azure Load Balancer 可透過探查連接埠來判斷作用中 DBMS 節點，並以獨佔方式將流量路由傳送至該作用中資料庫節點。 在資料庫節點的容錯移轉狀況下，不需要重新設定 SAP 應用程式。 然而，最常見的 SAP 應用程式架構會重新連線私人虛擬 IP 位址。 Azure Load Balancer 同時會將私人虛擬 IP 位址的流量重新導向至第二個節點，藉此回應節點容錯移轉。

Azure 提供兩個不同的[負載平衡器 SKU](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview)。 基本 SKU 與標準 SKU。 除非您想要部署於不同的 Azure 可用性區域，否則基本負載平衡器 SKU 的效果良好。 

DBMS VM 與 SAP 應用程式層之間的流量是否一律透過 Azure Load Balancer 路由傳送？ 答案取決於您設定負載平衡器的方式。 在此時間點，DBMS VM 的連入流量一律會透過 Azure Load Balancer 路由傳送。 從 DBMS VM 路由傳送至應用程式層 VM 的連出流量，取決於 Azure Load Balancer 的組態。 負載平衡器會提供 DirectServerReturn 選項。 如果已設定該選項，則從 DBMS VM 導向 SAP 應用程式層的流量**不會**透過 Azure Load Balancer 路由傳送。 它反而會直接前往應用程式層。 如果未設定 DirectServerReturn，則傳送至 SAP 應用程式層的傳回流量會透過 Azure Load Balancer 路由傳送。

建議您設定 DirectServerReturn 搭配位於 SAP 應用程式層與 DBMS 層之間的 Azure Load Balancer，以減少兩層之間的網路延遲。

如需設定 SQL Server Always On 這類組態的範例，請參閱[這篇文章](https://docs.microsoft.com/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener)。

如果您選擇使用已發佈的 github JSON 範本作為 Azure 中 SAP 基礎結構部署的參考，您應該研究 [SAP 3 層系統的範本](https://github.com/Azure/azure-quickstart-templates/tree/4099ad9bee183ed39b88c62cd33f517ae4e25669/sap-3-tier-marketplace-image-converged-md)。 此範本中，您也可以了解 Azure Load Balancer 的正確設定。

### <a name="azure-accelerated-networking"></a>Azure 加速網路
為了進一步減少 Azure VM 之間的網路延遲，強烈建議在針對 SAP 工作負載部署 Azure VM 時選擇 [Azure 加速網路](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/)選項。 特別針對 SAP 應用程式層與 SAP DBMS 層。 

> [!NOTE]
> 並非所有的 VM 類型都支援加速網路。 參考文章會列出支援加速網路的 VM 類型。 
>  

- - -
> ![Windows][Logo_Windows] Windows
> 
> 針對 Windows，請參閱[使用加速網路建立 Windows 虛擬機器](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-powershell)一文，以了解概念以及使用加速網路部署 VM 的方式
> 
> ![Linux][Logo_Linux] Linux
> 
> 針對 Linux 閱讀[使用加速網路建立 Linux 虛擬機器](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli)一文，以取得 Linux 散發套件的詳細資料。 
> 
> 

- - -

> [!NOTE]
> 如果是 SUSE、Red Hat 和 Oracle Linux，最近版本都支援加速網路。 SLES 12 SP2 或 RHEL 7.2 等較舊的版本不支援 Azure 加速網路 
>  


## <a name="deployment-of-host-monitoring"></a>部署主機監視功能
若要使 Azure 虛擬機器中的 SAP 應用程式以具生產力的方式運作，SAP 需要能夠從執行 Azure 虛擬機器的實際主機取得主機監視資料。 需要有特定的 SAP HostAgent 修補程式等級，才能在 SAPOSCOL 和 SAP HostAgent 中啟用此功能。 確切的修補程式等級記載於 SAP 附註 [1409604]。

如需了解如何部署將主機資料傳遞給 SAPOSCOL 和 SAPHostAgent 的元件及這些元件的生命週期管理，請參閱[部署指南][deployment-guide]


## <a name="next-steps"></a>後續步驟
如需特定 DBMS 的文件，請參閱下列文章：

- [適用於 SAP 工作負載的 SQL Server Azure 虛擬機器 DBMS 部署](dbms_guide_sqlserver.md)
- [適用於 SAP 工作負載的 Oracle Azure 虛擬機器 DBMS 部署](dbms_guide_oracle.md)
- [適用於 SAP 工作負載的 IBM DB2 Azure 虛擬機器 DBMS 部署](dbms_guide_ibm.md)
- [適用於 SAP 工作負載的 SAP ASE Azure 虛擬機器 DBMS 部署](dbms_guide_sapase.md)
- [Azure 上的 SAP maxDB、Live Cache 和 Content Server 部署](dbms_guide_maxdb.md)
- [Azure 上的 SAP HANA 作業指南](hana-vm-operations.md)
- [Azure 虛擬機器的 SAP Hana 高可用性](sap-hana-availability-overview.md)
- [Azure 虛擬機器上的 SAP HANA 備份指南](sap-hana-backup-guide.md)

