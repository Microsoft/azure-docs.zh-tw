---
title: SUSE Linux Enterprise Server 上 Azure VM 的 SAP HANA 高可用性 | Microsoft Docs
description: SUSE Linux Enterprise Server 上 Azure VM 的 SAP HANA 高可用性
services: virtual-machines-linux
documentationcenter: ''
author: MSSedusch
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/16/2018
ms.author: sedusch
ms.openlocfilehash: e2e76e3cd058e5798b0159923118b050f38d077e
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47034632"
---
# <a name="high-availability-of-sap-hana-on-azure-vms-on-suse-linux-enterprise-server"></a>SUSE Linux Enterprise Server 上 Azure VM 的 SAP HANA 高可用性

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2388694]:https://launchpad.support.sap.com/#/notes/2388694

[hana-ha-guide-replication]:sap-hana-high-availability.md#14c19f65-b5aa-4856-9594-b81c7e4df73d
[hana-ha-guide-shared-storage]:sap-hana-high-availability.md#498de331-fa04-490b-997c-b078de457c9d
[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/

[suse-hana-ha-guide]:https://www.suse.com/docrep/documents/ir8w88iwu7/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf
[sap-swcenter]:https://launchpad.support.sap.com/#/softwarecenter
[template-multisid-db]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged%2Fazuredeploy.json

進行內部部署開發時，您可以使用 HANA 系統複寫或使用共用儲存體來建立 SAP HANA 的高可用性。
在 Azure 虛擬機器 (VM) 上，Azure 上的 HANA 系統複寫是目前為止唯一支援的高可用性功能。 SAP HANA 複寫包含一個主要節點以及至少一個次要節點。 對主要節點上資料的變更，會以同步或非同步方式複寫到次要節點。

本文說明如何部署和設定虛擬機器、安裝叢集架構，以及安裝和設定 SAP HANA 系統複寫。
在組態範例中，會使用安裝命令、執行個體號碼 **03** 和 HANA 系統識別碼 **HN1**。

請先閱讀下列 SAP Note 和文件：

* SAP Note [1928533]，其中包含：
  * SAP 軟體部署支援的 Azure VM 大小清單。
  * Azure VM 大小的重要容量資訊。
  * 支援的 SAP 軟體，以及作業系統 (OS) 與資料庫組合。
  * Microsoft Azure 上 Windows 和 Linux 所需的 SAP 核心版本。
* SAP Note [2015553] 列出 Azure 中 SAP 支援的 SAP 軟體部署先決條件。
* SAP Note [2205917] 包含適用於 SUSE Linux Enterprise Server for SAP Applications 的建議 OS 設定。
* SAP Note [1944799] 包含適用於 SUSE Linux Enterprise Server for SAP Applications 的 SAP HANA 指導方針。
* SAP Note [2178632] 包含在 Azure 中針對 SAP 回報的所有監視計量詳細資訊。
* SAP Note [2191498] 包含 Azure 中 Linux 所需的 SAP Host Agent 版本。
* SAP Note [2243692] 包含 Azure 中 Linux 上的 SAP 授權相關資訊。
* SAP Note [1984787] 包含 SUSE LINUX Enterprise Server 12 的一般資訊。
* SAP Note [1999351] 包含 Azure Enhanced Monitoring Extension for SAP 的其他疑難排解資訊。
* [SAP Community WIKI](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) 包含 Linux 所需的所有 SAP Note。
* [SAP Hana 認證 IaaS 平台](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure)
* [適用於 SAP on Linux 的 Azure 虛擬機器規劃和實作][planning-guide]指南。
* [適用於 SAP on Linux 的 Azure 虛擬機器部署 (本文)][deployment-guide]。
* [適用於 SAP on Linux 的 Azure 虛擬機器 DBMS 部署][dbms-guide]指南。
* [適用於 SAP Applications 12 SP3 的 SUSE Linux Enterprise Server 最佳做法指南][sles-for-sap-bp]
  * 設定 SAP HANA SR 效能最佳化基礎結構 (SLES for SAP Applications 12 SP1)。 此指南包含所有必要資訊，可供您設定 SAP HANA 系統複寫以供內部部署開發之用。 請使用此指南做為基礎。
  * 設定 SAP HANA SR 成本最佳化基礎結構 (SLES for SAP Applications 12 SP1)

## <a name="overview"></a>概觀

為了達到高可用性，SAP HANA 會安裝在兩個虛擬機器上。 資料會使用「HANA 系統複寫」進行複寫。

![SAP HANA 高可用性概觀](./media/sap-hana-high-availability/ha-suse-hana.png)

SAP HANA 系統複寫設定會使用專用的虛擬主機名稱和虛擬 IP 位址。 在 Azure 上必須有負載平衡器才能使用虛擬 IP 位址。 下列清單顯示負載平衡器的組態：

* 前端組態：IP 位址 10.0.0.13 (針對 hn1-db)
* 後端組態：連線到應屬於 HANA 系統複寫一部分的所有虛擬機器的主要網路介面
* 探查連接埠：連接埠 62503
* 負載平衡規則：30313 TCP、30315 TCP、30317 TCP

## <a name="deploy-for-linux"></a>針對 Linux 進行部署

SAP HANA 的資源代理程式隨附於 SUSE Linux Enterprise Server for SAP Applications 中。
Azure Marketplace 包含 SUSE Linux Enterprise Server for SAP Applications 12 的映像，讓您可用來部署新的虛擬機器。

### <a name="deploy-with-a-template"></a>透過範本進行部署

您可以使用 GitHub 上的其中一個快速入門範本來部署所有必要資源。 範本會部署虛擬機器、負載平衡器、可用性設定組等。
若要部署範本，請遵循下列步驟：

1. 在 Azure 入口網站上開啟[資料庫範本][template-multisid-db]或[交集範本][template-converged]。 
    資料庫範本只會針對資料庫建立負載平衡規則。 交集範本還會針對 ASCS/SCS 和 ERS (僅限 Linux) 執行個體建立負載平衡規則。 如果您打算安裝 SAP NetWeaver 架構的系統，而且想要在同一部電腦上安裝 ASCS/SCS 執行個體，請使用[交集範本][template-converged]。

1. 輸入下列參數：
    - **SAP 系統識別碼**：輸入您想要安裝的 SAP 系統識別碼。 該識別碼會作為所部署之資源的前置詞。
    - **堆疊類型**：(此參數只有在您使用交集範本時才適用)。選取 SAP NetWeaver 堆疊類型。
    - **OS 類型**：選取其中一個 Linux 發行版本。 在此範例中，請選取 **SLES 12**。
    - **DB 類型**：選取 **HANA**。
    - **SAP 系統大小**：輸入新系統要提供的 SAP 數量。 如果您不確定系統需要多少 SAP，請詢問您的 SAP 技術合作夥伴或系統整合者。
    - **系統可用性**選取 [HA]。
    - **管理員使用者名稱和管理員密碼**：會建立可用來登入機器的新使用者。
    - **新的或現有的子網路**︰決定應該建立新的虛擬網路和子網路，還是使用現有的子網路。 如果您已經有連線到內部部署網路的虛擬網路，請選取 [現有]。
    - **子網路識別碼**：如果您想要將 VM 部署至您已定義應將 VM 指派到之目標子網路的現有 VNet，請提供該特定子網路的識別碼。 識別碼通常如下所示：**/subscriptions/\<訂用帳戶識別碼>/resourceGroups/\<資源群組名稱>/providers/Microsoft.Network/virtualNetworks/\<虛擬網路名稱>/subnets/\<子網路名稱>**。

### <a name="manual-deployment"></a>手動部署

> [!IMPORTANT]
> 請確定您選取的作業系統在您所使用的特定 VM 類型上已獲得 SAP HANA 認證。 在 [SAP HANA 認證 IaaS 平台](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) 中可查閱 SAP HANA 認證的 VM 類型和作業系統版本清單。 請務必按一下所列 VM 類型的詳細資料，以取得 SAP HANA 針對特定 VM 類型支援的作業系統版本完整清單。
>  

1. 建立資源群組。
1. 建立虛擬網路。
1. 建立可用性設定組。
   - 設定更新網域上限。
1. 建立負載平衡器 (內部)。
   - 選取步驟 2 所建立的虛擬網路。
1. 建立虛擬機器 1。
   - 您所選取的 VM 類型上使用 Azure 資源庫中 SAP HANA 支援的 SLES4SAP 映像。
   - 選取步驟 3 所建立的可用性設定組。
1. 建立虛擬機器 2。
   - 您所選取的 VM 類型上使用 Azure 資源庫中 SAP HANA 支援的 SLES4SAP 映像。
   - 選取步驟 3 所建立的可用性設定組。 
1. 新增資料磁碟。
1. 設定負載平衡器。 首先，建立前端 IP 集區：

   1. 開啟負載平衡器，選取 [前端 IP 集區]，然後選取 [新增]。
   1. 輸入新前端 IP 集區的名稱 (例如 **hana-frontend**)。
   1. 將 [指派] 設定為 [靜態]，然後輸入 IP 位址 (例如 **10.0.0.13**)。
   1. 選取 [確定] 。
   1. 建立新的前端 IP 集區之後，請記下集區 IP 位址。

1. 接下來，建立後端集區：

   1. 開啟負載平衡器，選取 [後端集區]，然後選取 [新增]。
   1. 輸入新後端集區的名稱 (例如 **hana-backend**)。
   1. 選取 [新增虛擬機器]。
   1. 選取步驟 3 所建立的可用性設定組。
   1. 選取 SAP HANA 叢集的虛擬機器。
   1. 選取 [確定] 。

1. 接下來，建立健康情況探查：

   1. 開啟負載平衡器，選取 [健康情況探查]，然後選取 [新增]。
   1. 輸入新健康情況探查的名稱 (例如 **hana-hp**)。
   1. 選取 [TCP] 通訊協定和連接埠 **62503**。 讓 [間隔] 值保持設定為 5，而讓 [狀況不良閾值] 值保持設定為 2。
   1. 選取 [確定] 。

1. 針對 SAP HANA 1.0，建立負載平衡規則：

   1. 開啟負載平衡器，選取 [負載平衡規則]，然後選取 [新增]。
   1. 輸入新負載平衡器規則的名稱 (例如 hana-lb-3**03**15)。
   1. 選取您稍早建立的前端 IP 位址、後端集區及健康情況探查 (例如 **hana-frontend**)。
   1. 讓 [通訊協定] 保持設定為 [TCP]，然後輸入連接埠 3**03**15。
   1. 將 [閒置逾時] 增加為 30 分鐘。
   1. 務必**啟用浮動 IP**。
   1. 選取 [確定] 。
   1. 對連接埠 3**03**17 重複執行這些步驟。

1. 若為 SAP Hana 2.0，請針對系統資料庫建立負載平衡規則：

   1. 開啟負載平衡器，選取 [負載平衡規則]，然後選取 [新增]。
   1. 輸入新負載平衡器規則的名稱 (例如 hana-lb-3**03**13)。
   1. 選取您稍早建立的前端 IP 位址、後端集區及健康情況探查 (例如 **hana-frontend**)。
   1. 讓 [通訊協定] 保持設定為 [TCP]，然後輸入連接埠 3**03**13。
   1. 將 [閒置逾時] 增加為 30 分鐘。
   1. 務必**啟用浮動 IP**。
   1. 選取 [確定] 。
   1. 針對連接埠 3**03**14 重複這些步驟。

1. 若為 SAP Hana 2.0，請先針對租用戶資料庫建立負載平衡規則：

   1. 開啟負載平衡器，選取 [負載平衡規則]，然後選取 [新增]。
   1. 輸入新負載平衡器規則的名稱 (例如 hana-lb-3**03**40)。
   1. 選取您稍早建立的前端 IP 位址、後端集區及健康情況探查 (例如 **hana-frontend**)。
   1. 讓 [通訊協定] 保持設定為 [TCP]，然後輸入連接埠 3**03**40。
   1. 將 [閒置逾時] 增加為 30 分鐘。
   1. 務必**啟用浮動 IP**。
   1. 選取 [確定] 。
   1. 針對連接埠 3**03**41 和 3**03**42 重複這些步驟。

如需 SAP Hana 必要連接埠的詳細資訊，請參閱 [SAP Hana 租用戶資料庫](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6)指南的[連線到租用戶資料庫](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6/latest/en-US/7a9343c9f2a2436faa3cfdb5ca00c052.html)章節或 [SAP Note 2388694][2388694]。


## <a name="create-a-pacemaker-cluster"></a>建立 Pacemaker 叢集

依照[在 Azure 中設定 SUSE Linux Enterprise Server 的 Pacemaker](high-availability-guide-suse-pacemaker.md) 中的步驟，建立此 HANA 伺服器的基本 Pacemaker 叢集。 您可以將相同的 Pacemaker 叢集用於 SAP HANA 和 SAP NetWeaver (A)SCS。

## <a name="install-sap-hana"></a>安裝 SAP HANA

本節中的步驟會使用下列首碼：
- **[A]**：此步驟適用於所有節點。
- **[1]**：此步驟僅適用於節點 1。
- **[2]**：此步驟僅適用於 Pacemaker 叢集的節點 2。

1. **[A]** 設定磁碟配置：**邏輯磁碟區管理員 (LVM)**。

   建議您針對儲存資料和記錄檔的磁碟區使用 LVM。 下列範例假設虛擬機器已連接四個資料磁碟，這些資料磁碟會用來建立兩個磁碟區。

   列出所有可用的磁碟：

   <pre><code>ls /dev/disk/azure/scsi1/lun*
   </code></pre>

   範例輸出︰

   <pre><code>
   /dev/disk/azure/scsi1/lun0  /dev/disk/azure/scsi1/lun1  /dev/disk/azure/scsi1/lun2  /dev/disk/azure/scsi1/lun3
   </code></pre>

   針對您想要使用的所有磁碟建立實體磁碟區：

   <pre><code>sudo pvcreate /dev/disk/azure/scsi1/lun0
   sudo pvcreate /dev/disk/azure/scsi1/lun1
   sudo pvcreate /dev/disk/azure/scsi1/lun2
   sudo pvcreate /dev/disk/azure/scsi1/lun3
   </code></pre>

   建立資料檔案的磁碟區群組。 一個磁碟區群組用於記錄檔，一個用於 SAP HANA 的共用目錄：

   <pre><code>sudo vgcreate vg_hana_data_<b>HN1</b> /dev/disk/azure/scsi1/lun0 /dev/disk/azure/scsi1/lun1
   sudo vgcreate vg_hana_log_<b>HN1</b> /dev/disk/azure/scsi1/lun2
   sudo vgcreate vg_hana_shared_<b>HN1</b> /dev/disk/azure/scsi1/lun3
   </code></pre>

   建立邏輯磁碟區。 當您使用 `lvcreate` 卻未搭配 `-i` 參數時，會建立線性磁碟區。 建議您建立等量磁碟區以獲得更好的 I/O 效能，其中，`-i` 引數應該為基礎實體磁碟區的數目。 本文件會使用 2 個實體磁碟區來作為資料磁碟區，因此 `-i` 參數引數會設定為 **2**。 1 個實體磁碟區會作為記錄磁碟區，因此不會明確使用 `-i` 參數。 當您要對每個資料、記錄或共用磁碟區使用多個實體磁碟區時，請使用 `-i` 參數，並將其值設定為基礎實體磁碟區的數目。

   <pre><code>sudo lvcreate <b>-i 2</b> -l 100%FREE -n hana_data vg_hana_data_<b>HN1</b>
   sudo lvcreate -l 100%FREE -n hana_log vg_hana_log_<b>HN1</b>
   sudo lvcreate -l 100%FREE -n hana_shared vg_hana_shared_<b>HN1</b>
   sudo mkfs.xfs /dev/vg_hana_data_<b>HN1</b>/hana_data
   sudo mkfs.xfs /dev/vg_hana_log_<b>HN1</b>/hana_log
   sudo mkfs.xfs /dev/vg_hana_shared_<b>HN1</b>/hana_shared
   </code></pre>

   建立掛接目錄，並複製所有邏輯磁碟區的 UUID：

   <pre><code>sudo mkdir -p /hana/data/<b>HN1</b>
   sudo mkdir -p /hana/log/<b>HN1</b>
   sudo mkdir -p /hana/shared/<b>HN1</b>
   # Write down the ID of /dev/vg_hana_data_<b>HN1</b>/hana_data, /dev/vg_hana_log_<b>HN1</b>/hana_log, and /dev/vg_hana_shared_<b>HN1</b>/hana_shared
   sudo blkid
   </code></pre>

   針對這三個邏輯磁碟區建立 `fstab` 項目：       

   <pre><code>sudo vi /etc/fstab
   </code></pre>

   在 `/etc/fstab` 檔案中插入下列行：      

   <pre><code>/dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_data_<b>HN1</b>-hana_data&gt;</b> /hana/data/<b>HN1</b> xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_log_<b>HN1</b>-hana_log&gt;</b> /hana/log/<b>HN1</b> xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_shared_<b>HN1</b>-hana_shared&gt;</b> /hana/shared/<b>HN1</b> xfs  defaults,nofail  0  2
   </code></pre>

   掛接新的磁碟區：

   <pre><code>sudo mount -a
   </code></pre>

1. **[A]** 設定磁碟配置：**一般磁碟**。

   針對示範系統，您可以將 HANA 資料和記錄檔放在一個磁碟上。 在 /dev/disk/azure/scsi1/lun0 上建立磁碟分割，並使用 xfs 將其格式化：

   <pre><code>sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun0'
   sudo mkfs.xfs /dev/disk/azure/scsi1/lun0-part1
   
   # Write down the ID of /dev/disk/azure/scsi1/lun0-part1
   sudo /sbin/blkid
   sudo vi /etc/fstab
   </code></pre>

   將這行插入至 /etc/fstab 檔案：

   <pre><code>/dev/disk/by-uuid/<b>&lt;UUID&gt;</b> /hana xfs  defaults,nofail  0  2
   </code></pre>

   建立目標目錄並掛接磁碟：

   <pre><code>sudo mkdir /hana
   sudo mount -a
   </code></pre>

1. **[A]** 設定所有主機的主機名稱解析。

   您可以使用 DNS 伺服器，或修改所有節點上的 /etc/hosts 檔案。 這個範例示範如何使用 /etc/hosts 檔案。
   請取代下列命令中的 IP 位址和主機名稱：

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   將下列幾行插入至 /etc/hosts 檔案。 變更 IP 位址和主機名稱以符合您的環境：

   <pre><code><b>10.0.0.5 hn1-db-0</b>
   <b>10.0.0.6 hn1-db-1</b>
   </code></pre>

1. **[A]** 安裝 SAP HANA 高可用性套件：

   <pre><code>sudo zypper install SAPHanaSR
   </code></pre>

若要安裝 SAP HANA 系統複寫，請遵循 [SAP HANA SR 效能最佳化案例指南](https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/)的第 4 章。

1. **[A]** 從 HANA DVD 執行 **hdblcm** 程式。 在提示上輸入下列值：
   * 選擇安裝：輸入 **1**。
   * 選取要安裝的其他元件：輸入 **1**。
   * 輸入安裝路徑 [/hana/shared]：選取 Enter 鍵。
   * 輸入本機主機名稱 [..]：選取 Enter 鍵。
   * 您是否要將其他主機新增至系統？ (y/n) [n]：選取 Enter 鍵。
   * 輸入 SAP HANA 系統識別碼：輸入 HANA 的 SID，例如：**HN1**。
   * 輸入執行個體號碼 [00]：輸入 HANA 執行個體號碼。 如果您使用了 Azure 範本或遵循了本文的手動部署章節，請輸入 **03**。
   * 選取資料庫模式 / 輸入索引 [1]：選取 Enter 鍵。
   * 選取系統使用量 / 輸入索引 [4]：選取系統使用量的值。
   * 輸入資料磁碟區的位置 [/hana/data/HN1]：選取 Enter 鍵。
   * 輸入記錄磁碟區的位置 [/hana/log/HN1]：選取 Enter 鍵。
   * 是否限制記憶體配置上限？ [n]：選取 Enter 鍵。
   * 輸入主機的憑證主機名稱 '...' [...]：選取 Enter 鍵。
   * 輸入 SAP 主機代理程式使用者 (sapadm) 密碼：輸入主機代理程式使用者密碼。
   * 確認 SAP 主機代理程式使用者 (sapadm) 密碼：再次輸入主機代理程式使用者密碼以做確認。
   * 輸入系統管理員 (hdbadm) 密碼：輸入系統管理員密碼。
   * 確認系統管理員 (hdbadm) 密碼：再次輸入系統管理員密碼以做確認。
   * 輸入系統管理員主目錄 [/usr/sap/HN1/home]：選取 Enter 鍵。
   * 輸入系統管理員登入殼層 [/bin/sh]：選取 Enter 鍵。
   * 輸入系統管理員使用者識別碼 [1001]：選取 Enter 鍵。
   * 輸入使用者群組 (sapsys) 的識別碼 [79]：選取 Enter 鍵。
   * 輸入資料庫使用者 (SYSTEM) 密碼：輸入資料庫使用者密碼。
   * 確認資料庫使用者 (SYSTEM) 密碼：再次輸入資料庫使用者密碼以做確認。
   * 是否在電腦重新開機後重新啟動系統？ [n]：選取 Enter 鍵。
   * 是否要繼續？ (y/n)：驗證摘要。 輸入 **y** 繼續。

1. **[A]** 升級 SAP 主機代理程式。

   從 [SAP 軟體中心 (英文)][sap-swcenter] 下載最新的 SAP 主機代理程式封存檔，然後執行下列命令來升級代理程式。 取代封存檔的路徑以指向您所下載的檔案：

   <pre><code>sudo /usr/sap/hostctrl/exe/saphostexec -upgrade -archive &lt;path to SAP Host Agent SAR&gt;
   </code></pre>

## <a name="configure-sap-hana-20-system-replication"></a>設定 SAP HANA 2.0 系統複寫

本節中的步驟會使用下列首碼：

* **[A]**：此步驟適用於所有節點。
* **[1]**：此步驟僅適用於節點 1。
* **[2]**：此步驟僅適用於 Pacemaker 叢集的節點 2。

1. **[1]** 建立租用戶資料庫。

   如果您使用 SAP HANA 2.0 或 MDC，請為您的 SAP NetWeaver 系統建立租用戶資料庫。 請將 **NW1** 取代為您 SAP 系統的 SID。

   以 \<hanasid>adm 身分登入，然後執行下列命令：

   <pre><code>hdbsql -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> -d SYSTEMDB 'CREATE DATABASE <b>NW1</b> SYSTEM USER PASSWORD "<b>passwd</b>"'
   </code></pre>

1. **[1]** 在第一個節點上設定系統複寫：

   以 \<hanasid>adm 身分登入，然後備份資料庫：

   <pre><code>hdbsql -d SYSTEMDB -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupSYS</b>')"
   hdbsql -d <b>HN1</b> -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupHN1</b>')"
   hdbsql -d <b>NW1</b> -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupNW1</b>')"
   </code></pre>

   將系統 PKI 檔案複製到次要網站：

   <pre><code>scp /usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/data/SSFS_<b>HN1</b>.DAT   <b>hn1-db-1</b>:/usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/data/
   scp /usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/key/SSFS_<b>HN1</b>.KEY  <b>hn1-db-1</b>:/usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/key/
   </code></pre>

   建立主要網站：

   <pre><code>hdbnsutil -sr_enable --name=<b>SITE1</b>
   </code></pre>

1. **[2]** 在第二個節點上設定系統複寫：
    
   註冊第二個節點，以啟動系統複寫。 以 \<hanasid>adm 身分登入，然後執行下列命令：

   <pre><code>sapcontrol -nr <b>03</b> -function StopWait 600 10
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b> 
   </code></pre>

## <a name="configure-sap-hana-10-system-replication"></a>設定 SAP HANA 1.0 系統複寫

本節中的步驟會使用下列首碼：

* **[A]**：此步驟適用於所有節點。
* **[1]**：此步驟僅適用於節點 1。
* **[2]**：此步驟僅適用於 Pacemaker 叢集的節點 2。

1. **[1]** 建立必要的使用者。

   以 root 身分登入，然後執行下列命令。 請務必以 SAP HANA 安裝的值取代粗體字串 (HANA 系統識別碼 **HN1** 和執行個體號碼 **03**)：

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbsql -u system -i <b>03</b> 'CREATE USER <b>hdb</b>hasync PASSWORD "<b>passwd</b>"'
   hdbsql -u system -i <b>03</b> 'GRANT DATA ADMIN TO <b>hdb</b>hasync'
   hdbsql -u system -i <b>03</b> 'ALTER USER <b>hdb</b>hasync DISABLE PASSWORD LIFETIME'
   </code></pre>

1. **[A]** 建立金鑰儲存區項目。

   以 root 身分登入並執行下列命令，以建立新的金鑰儲存區項目：

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbuserstore SET <b>hdb</b>haloc localhost:3<b>03</b>15 <b>hdb</b>hasync <b>passwd</b>
   </code></pre>

1. **[1]** 備份資料庫。

   以 root 身分登入，然後備份資料庫：

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbsql -d SYSTEMDB -u system -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackup</b>')"
   </code></pre>

   如果您使用多租用戶安裝，請同時備份租用戶資料庫：

   <pre><code>hdbsql -d <b>HN1</b> -u system -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackup</b>')"
   </code></pre>

1. **[1]** 在第一個節點上設定系統複寫。

   以 \<hanasid>adm 身分登入，然後建立主要網站：

   <pre><code>su - <b>hdb</b>adm
   hdbnsutil -sr_enable –-name=<b>SITE1</b>
   </code></pre>

1. **[2]** 在次要節點上設定系統複寫。

   以 \<hanasid>adm 身分登入，然後註冊次要網站：

   <pre><code>sapcontrol -nr <b>03</b> -function StopWait 600 10
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b> 
   </code></pre>

## <a name="create-sap-hana-cluster-resources"></a>建立 SAP HANA 叢集資源

首先，建立 HANA 拓撲。 在其中一個 Pacemaker 叢集節點上執行下列命令：

<pre><code>sudo crm configure property maintenance-mode=true

# Replace the bold string with your instance number and HANA system ID

sudo crm configure primitive rsc_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> ocf:suse:SAPHanaTopology \
  operations \$id="rsc_sap2_<b>HN1</b>_HDB<b>03</b>-operations" \
  op monitor interval="10" timeout="600" \
  op start interval="0" timeout="600" \
  op stop interval="0" timeout="300" \
  params SID="<b>HN1</b>" InstanceNumber="<b>03</b>"

sudo crm configure clone cln_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> rsc_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> \
  meta is-managed="true" clone-node-max="1" target-role="Started" interleave="true"
</code></pre>

接下來，建立 HANA 資源：

<pre><code># Replace the bold string with your instance number, HANA system ID, and the front-end IP address of the Azure load balancer. 

sudo crm configure primitive rsc_SAPHana_<b>HN1</b>_HDB<b>03</b> ocf:suse:SAPHana \
  operations \$id="rsc_sap_<b>HN1</b>_HDB<b>03</b>-operations" \
  op start interval="0" timeout="3600" \
  op stop interval="0" timeout="3600" \
  op promote interval="0" timeout="3600" \
  op monitor interval="60" role="Master" timeout="700" \
  op monitor interval="61" role="Slave" timeout="700" \
  params SID="<b>HN1</b>" InstanceNumber="<b>03</b>" PREFER_SITE_TAKEOVER="true" \
  DUPLICATE_PRIMARY_TIMEOUT="7200" AUTOMATED_REGISTER="false"

sudo crm configure ms msl_SAPHana_<b>HN1</b>_HDB<b>03</b> rsc_SAPHana_<b>HN1</b>_HDB<b>03</b> \
  meta is-managed="true" notify="true" clone-max="2" clone-node-max="1" \
  target-role="Started" interleave="true"

sudo crm configure primitive rsc_ip_<b>HN1</b>_HDB<b>03</b> ocf:heartbeat:IPaddr2 \
  meta target-role="Started" is-managed="true" \
  operations \$id="rsc_ip_<b>HN1</b>_HDB<b>03</b>-operations" \
  op monitor interval="10s" timeout="20s" \
  params ip="<b>10.0.0.13</b>"

sudo crm configure primitive rsc_nc_<b>HN1</b>_HDB<b>03</b> anything \
  params binfile="/usr/bin/nc" cmdline_options="-l -k 625<b>03</b>" \
  op monitor timeout=20s interval=10 depth=0

sudo crm configure group g_ip_<b>HN1</b>_HDB<b>03</b> rsc_ip_<b>HN1</b>_HDB<b>03</b> rsc_nc_<b>HN1</b>_HDB<b>03</b>

sudo crm configure colocation col_saphana_ip_<b>HN1</b>_HDB<b>03</b> 4000: g_ip_<b>HN1</b>_HDB<b>03</b>:Started \
  msl_SAPHana_<b>HN1</b>_HDB<b>03</b>:Master  

sudo crm configure order ord_SAPHana_<b>HN1</b>_HDB<b>03</b> Optional: cln_SAPHanaTopology_<b>HN1</b>_HDB<b>03</b> \
  msl_SAPHana_<b>HN1</b>_HDB<b>03</b>

# Clean up the HANA resources. The HANA resources might have failed because of a known issue.
sudo crm resource cleanup rsc_SAPHana_<b>HN1</b>_HDB<b>03</b>

sudo crm configure property maintenance-mode=false
sudo crm configure rsc_defaults resource-stickiness=1000
sudo crm configure rsc_defaults migration-threshold=5000
</code></pre>

請確定叢集狀態正常，且所有資源皆已啟動。 資源在哪一個節點上執行並不重要。

<pre><code>sudo crm_mon -r

# Online: [ hn1-db-0 hn1-db-1 ]
#
# Full list of resources:
#
# stonith-sbd     (stonith:external/sbd): Started hn1-db-0
# rsc_st_azure    (stonith:fence_azure_arm):      Started hn1-db-1
# Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
#     Started: [ hn1-db-0 hn1-db-1 ]
# Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
#     Masters: [ hn1-db-0 ]
#     Slaves: [ hn1-db-1 ]
# Resource Group: g_ip_HN1_HDB03
#     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
#     rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
</code></pre>

## <a name="test-the-cluster-setup"></a>測試叢集設定

本節將說明如何測試您的設定。 每一個測試都假設您是 root 身分，而 SAP HANA 主要節點會在 **hn1-db-0** 虛擬機器上執行。

### <a name="test-the-migration"></a>測試移轉

開始測試之前，請先確定 Pacemaker 沒有任何失敗的動作 (透過 crm_mon -r)、沒有未預期的位置條件約束 (例如，移轉測試的殘餘項目)，且 HANA 是同步處理狀態，例如使用 SAPHanaSR-showAttr：

<pre><code>hn1-db-0:~ # SAPHanaSR-showAttr

Global cib-time
--------------------------------
global Mon Aug 13 11:26:04 2018

Hosts    clone_state lpa_hn1_lpt node_state op_mode   remoteHost    roles                            score site  srmode sync_state version                vhost
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
hn1-db-0 PROMOTED    1534159564  online     logreplay nws-hana-vm-1 4:P:master1:master:worker:master 150   SITE1 sync   PRIM       2.00.030.00.1522209842 nws-hana-vm-0
hn1-db-1 DEMOTED     30          online     logreplay nws-hana-vm-0 4:S:master1:master:worker:master 100   SITE2 sync   SOK        2.00.030.00.1522209842 nws-hana-vm-1
</code></pre>

您可以執行下列命令來移轉 SAP HANA 主要節點：

<pre><code>crm resource migrate msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-1</b>
</code></pre>

如果您設定 `AUTOMATED_REGISTER="false"`，此命令序列應會將 SAP HANA 主要節點以及包含虛擬 IP 位址的群組移轉到 hn1-db-1。

完成移轉之後，crm_mon -r 輸出看起來像這樣

<pre><code>Online: [ hn1-db-0 hn1-db-1 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started hn1-db-1
 Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     Started: [ hn1-db-0 hn1-db-1 ]
 Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     Masters: [ hn1-db-1 ]
     Stopped: [ hn1-db-0 ]
 Resource Group: g_ip_HN1_HDB03
     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
     rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-1

Failed Actions:
* rsc_SAPHana_HN1_HDB03_start_0 on hn1-db-0 'not running' (7): call=84, status=complete, exitreason='none',
    last-rc-change='Mon Aug 13 11:31:37 2018', queued=0ms, exec=2095ms
</code></pre>

hn1-db-0 上的 SAP HANA 資源會無法啟動為次要資源。 在此情況下，執行下列命令來將 HANA 執行個體設為次要：

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr <b>03</b> -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>
</code></pre>

移轉會建立需要再次刪除的位置條件約束：

<pre><code># Switch back to root and clean up the failed state
exit
hn1-db-0:~ # crm resource unmigrate msl_SAPHana_<b>HN1</b>_HDB<b>03</b>
</code></pre>

您也必須清除次要節點資源的狀態：

<pre><code>hn1-db-0:~ # crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

使用 crm_mon -r 監視 HANA 資源的狀態。 hn1-db-0 上啟動 HANA 之後，輸出應該看起來像這樣

<pre><code>Online: [ hn1-db-0 hn1-db-1 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started hn1-db-1
 Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
     Started: [ hn1-db-0 hn1-db-1 ]
 Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
     Masters: [ hn1-db-1 ]
     Slaves: [ hn1-db-0 ]
 Resource Group: g_ip_HN1_HDB03
     rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
     rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-1
</code></pre>

### <a name="test-the-azure-fencing-agent-not-sbd"></a>測試 Azure 隔離代理程式 (非 SBD)

您可以藉由停用 hn1-db-0 節點上的網路介面，來測試 Azure 隔離代理程式的設定：

<pre><code>sudo ifdown eth0
</code></pre>

根據您的叢集組態，虛擬機器現在應該會重新啟動或停止。
如果您將 `stonith-action` 設定設為 off，虛擬機器會停止，並將資源移轉到執行中的虛擬機器。

當您重新啟動虛擬機器之後，如果您設定 `AUTOMATED_REGISTER="false"`，SAP HANA 資源就無法啟動為次要。 在此情況下，執行下列命令來將 HANA 執行個體設為次要：

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
sapcontrol -nr <b>03</b> -function StopWait 600 10
hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>

# Switch back to root and clean up the failed state
exit
crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

### <a name="test-sbd-fencing"></a>測試 SBD 隔離

您可以終止 inquisitor 處理序，以測試 SBD 的設定。

<pre><code>hn1-db-0:~ # ps aux | grep sbd
root       1912  0.0  0.0  85420 11740 ?        SL   12:25   0:00 sbd: inquisitor
root       1929  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-360014056f268462316e4681b704a9f73 - slot: 0 - uuid: 7b862dba-e7f7-4800-92ed-f76a4e3978c8
root       1930  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-360014059bc9ea4e4bac4b18808299aaf - slot: 0 - uuid: 5813ee04-b75c-482e-805e-3b1e22ba16cd
root       1931  0.0  0.0  85456 11776 ?        SL   12:25   0:00 sbd: watcher: /dev/disk/by-id/scsi-36001405b8dddd44eb3647908def6621c - slot: 0 - uuid: 986ed8f8-947d-4396-8aec-b933b75e904c
root       1932  0.0  0.0  90524 16656 ?        SL   12:25   0:00 sbd: watcher: Pacemaker
root       1933  0.0  0.0 102708 28260 ?        SL   12:25   0:00 sbd: watcher: Cluster
root      13877  0.0  0.0   9292  1572 pts/0    S+   12:27   0:00 grep sbd

hn1-db-0:~ # kill -9 1912
</code></pre>

叢集節點 hn1-db-0 應該重新啟動。 Pacemaker 服務之後可能不會啟動。 請務必重新啟動它。

### <a name="test-a-manual-failover"></a>測試手動容錯移轉

您可以藉由停止 hn1-db-0 節點上的 `pacemaker` 服務，來測試手動容錯移轉：

<pre><code>service pacemaker stop
</code></pre>

容錯移轉之後，您可以重新啟動服務。 如果您設定 `AUTOMATED_REGISTER="false"`，hn1-db-0 節點上的 SAP HANA 資源會無法以次要身分啟動。 在此情況下，執行下列命令來將 HANA 執行個體設為次要：

<pre><code>service pacemaker start
su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
sapcontrol -nr <b>03</b> -function StopWait 600 10
hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b> 

# Switch back to root and clean up the failed state
exit
crm resource cleanup msl_SAPHana_<b>HN1</b>_HDB<b>03</b> <b>hn1-db-0</b>
</code></pre>

### <a name="suse-tests"></a>SUSE 測試

> [!IMPORTANT]
> 請確定您選取的作業系統在您所使用的特定 VM 類型上已獲得 SAP HANA 認證。 在 [SAP HANA 認證 IaaS 平台](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) 中可查閱 SAP HANA 認證的 VM 類型和作業系統版本清單。 請務必按一下所列 VM 類型的詳細資料，以取得 SAP HANA 針對特定 VM 類型支援的作業系統版本完整清單。

請根據您的使用案例，執行 SAP HANA SR 效能最佳化案例或 SAP HANA SR 成本最佳化案例指南中所列出的所有測試案例。 您可以在 [SLES for SAP 最佳做法頁面][sles-for-sap-bp]上找到這些指南。

下列測試是 SAP HANA SR 效能最佳化案例 SUSE Linux Enterprise Server for SAP Applications 12 SP1 指南的測試描述複本。 如需最新版本，請一定還要閱讀指南本身。 請一定要確定 HANA 處於同步狀態再開始測試，也請確定 Pacemaker 的組態正確無誤。

在下列測試描述中，我們假設 PREFER_SITE_TAKEOVER="true" 且 AUTOMATED_REGISTER="false"。
注意：下列測試依設計必須循序執行，且必須等前面的測試結束後才會開始執行。

1. 測試 1：在節點 1 上停止主要資料庫

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

   以節點 hn1-db-0 上的 \<hanasid>adm 身分執行下列命令：

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker 應該會偵測到已停止的 HANA 執行個體，並容錯移轉至其他節點。 容錯移轉完成後，因為 Pacemaker 不會自動將節點 hn1-db-0 註冊為 HANA 次要節點，所以該節點上的 HANA 執行個體會停止。

   執行下列命令以將節點 hn1-db-0 註冊為次要節點，並清除失敗的資源。

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-1
   </code></pre>

1. 測試 2：在節點 2 上停止主要資料庫

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-1
   </code></pre>

   以節點 hn1-db-1 上的 \<hanasid>adm 身分執行下列命令：

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker 應該會偵測到已停止的 HANA 執行個體，並容錯移轉至其他節點。 容錯移轉完成後，因為 Pacemaker 不會自動將節點 hn1-db-1 註冊為 HANA 次要節點，所以該節點上的 HANA 執行個體會停止。

   執行下列命令以將節點 hn1-db-1 註冊為次要節點，並清除失敗的資源。

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

1. 測試 3：在節點上損毀主要資料庫

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

   以節點 hn1-db-0 上的 \<hanasid>adm 身分執行下列命令：

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>
   
   Pacemaker 應該會偵測到已終止的 HANA 執行個體，並容錯移轉至其他節點。 容錯移轉完成後，因為 Pacemaker 不會自動將節點 hn1-db-0 註冊為 HANA 次要節點，所以該節點上的 HANA 執行個體會停止。

   執行下列命令以將節點 hn1-db-0 註冊為次要節點，並清除失敗的資源。

   <pre><code>hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-1
   </code></pre>

1. 測試 4：在節點 2 上損毀主要資料庫

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-1
   </code></pre>

   以節點 hn1-db-1 上的 \<hanasid>adm 身分執行下列命令：

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>

   Pacemaker 應該會偵測到已終止的 HANA 執行個體，並容錯移轉至其他節點。 容錯移轉完成後，因為 Pacemaker 不會自動將節點 hn1-db-1 註冊為 HANA 次要節點，所以該節點上的 HANA 執行個體會停止。

   執行下列命令以將節點 hn1-db-1 註冊為次要節點，並清除失敗的資源。

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

1. 測試 5：損毀主要網站節點 (節點 1)

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

   以節點 hn1-db-0 上的 root 身分執行下列命令：

   <pre><code>hn1-db-0:~ #  echo 'b' > /proc/sysrq-trigger
   </code></pre>

   Pacemaker 應該會偵測到已終止的叢集節點，並將節點隔離。 節點隔離後，Pacemaker 會觸發 HANA 執行個體的接管作業。 當隔離的節點重新啟動時，Pacemaker 不會自動啟動。

   執行下列命令來啟動 Pacemaker、清除節點 hn1-db-0 的 SBD 訊息、將節點 hn1-db-0 註冊為次要節點，然後清除失敗的資源。

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-0:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-0:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-0 clear
   
   hn1-db-0:~ # systemctl start pacemaker
   
   # run as &lt;hanasid&gt;adm
   hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMode=sync --name=SITE1
   
   # run as root
   hn1-db-0:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-0
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-1
   </code></pre>

1. 測試 6：損毀次要網站節點 (節點 2)

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-1 ]
      Slaves: [ hn1-db-0 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-1
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-1
   </code></pre>

   以節點 hn1-db-1 上的 root 身分執行下列命令：

   <pre><code>hn1-db-1:~ #  echo 'b' > /proc/sysrq-trigger
   </code></pre>

   Pacemaker 應該會偵測到已終止的叢集節點，並將節點隔離。 節點隔離後，Pacemaker 會觸發 HANA 執行個體的接管作業。 當隔離的節點重新啟動時，Pacemaker 不會自動啟動。

   執行下列命令來啟動 Pacemaker、清除節點 hn1-db-1 的 SBD 訊息、將節點 hn1-db-1 註冊為次要節點，然後清除失敗的資源。

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-1:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-1:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-1 clear
   
   hn1-db-1:~ # systemctl start pacemaker
   
   # run as &lt;hanasid&gt;adm
   hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2
   
   # run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

1. 測試 7：在節點 2 上停止次要資料庫

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

   以節點 hn1-db-1 上的 \<hanasid>adm 身分執行下列命令：

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB stop
   </code></pre>

   Pacemaker 會偵測到已停止的 HANA 執行個體，並在節點 hn1-db-1 上將資源標記為已失敗。 Pacemaker 應該會自動重新啟動 HANA 執行個體。 執行下列命令以清除失敗的狀態。

   <pre><code># run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

1. 測試 8：在節點 2 上損毀次要資料庫

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

   以節點 hn1-db-1 上的 \<hanasid>adm 身分執行下列命令：

   <pre><code>hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> HDB kill-9
   </code></pre>

   Pacemaker 會偵測到已終止的 HANA 執行個體，並在節點 hn1-db-1 上將資源標記為已失敗。 執行下列命令以清除失敗的狀態。 接著，Pacemaker 應該會自動重新啟動 HANA 執行個體。

   <pre><code># run as root
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

1. 測試 9：損毀執行次要 HANA 資料庫的次要網站節點 (節點 2)

   開始測試之前的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

   以節點 hn1-db-1 上的 root 身分執行下列命令：

   <pre><code>hn1-db-1:~ # echo b > /proc/sysrq-trigger
   </code></pre>

   Pacemaker 應該會偵測到已終止的叢集節點，並將節點隔離。 當隔離的節點重新啟動時，Pacemaker 不會自動啟動。

   執行下列命令來啟動 Pacemaker、清除節點 hn1-db-1 的 SBD 訊息，然後清除失敗的資源。

   <pre><code># run as root
   # list the SBD device(s)
   hn1-db-1:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   hn1-db-1:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message hn1-db-1 clear
   
   hn1-db-1:~ # systemctl start pacemaker  
   
   hn1-db-1:~ # crm resource cleanup msl_SAPHana_HN1_HDB03 hn1-db-1
   </code></pre>

   測試完成之後的資源狀態：

   <pre><code>Clone Set: cln_SAPHanaTopology_HN1_HDB03 [rsc_SAPHanaTopology_HN1_HDB03]
      Started: [ hn1-db-0 hn1-db-1 ]
   Master/Slave Set: msl_SAPHana_HN1_HDB03 [rsc_SAPHana_HN1_HDB03]
      Masters: [ hn1-db-0 ]
      Slaves: [ hn1-db-1 ]
   Resource Group: g_ip_HN1_HDB03
      rsc_ip_HN1_HDB03   (ocf::heartbeat:IPaddr2):       Started hn1-db-0
      rsc_nc_HN1_HDB03   (ocf::heartbeat:anything):      Started hn1-db-0
   </code></pre>

## <a name="next-steps"></a>後續步驟

* [適用於 SAP 的 Azure 虛擬機器規劃和實作][planning-guide]
* [適用於 SAP 的 Azure 虛擬機器部署][deployment-guide]
* [適用於 SAP 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* 若要了解如何建立高可用性並為 Azure 上的 SAP HANA 規劃災害復原，請參閱 [Azure 上的 SAP HANA (大型執行個體) 高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md)。
