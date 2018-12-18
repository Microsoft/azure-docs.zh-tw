---
title: 適用於 SUSE Linux Enterprise Server 之 Azure VM 上 NFS 的高可用性 | Microsoft Docs
description: 適用於 SUSE Linux Enterprise Server 之 Azure VM 上 NFS 的高可用性
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: mssedusch
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/16/2018
ms.author: sedusch
ms.openlocfilehash: 6ad943d2ae98093f7e5b94df59757ff603083cfe
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43668950"
---
# <a name="high-availability-for-nfs-on-azure-vms-on-suse-linux-enterprise-server"></a>適用於 SUSE Linux Enterprise Server 之 Azure VM 上 NFS 的高可用性

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
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[sles-hae-guides]:https://www.suse.com/documentation/sle-ha-12/
[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md

本文說明如何部署虛擬機器、設定虛擬機器、安裝叢集架構，以及安裝可用來儲存高可用性 SAP 系統之共用資料的高可用性 NFS 伺服器。
本指南說明如何設定供兩個 SAP 系統 (NW1 和 NW2) 使用的高可用性 NFS 伺服器。 範例中資源 (例如虛擬機器、虛擬網路) 的名稱是假設您已使用資源前置詞為 **prod**的 [SAP 檔案伺服器範本][template-file-server]。

請先閱讀下列 SAP Note 和文件

* SAP Note [1928533]，其中包含：
  * SAP 軟體部署支援的 Azure VM 大小清單
  * Azure VM 大小的重要容量資訊
  * 支援的 SAP 軟體，以及作業系統 (OS) 與資料庫組合
  * Microsoft Azure 上 Windows 和 Linux 所需的 SAP 核心版本

* SAP Note [2015553] 列出 Azure 中 SAP 支援的 SAP 軟體部署先決條件。
* SAP Note [2205917] 包含適用於 SUSE Linux Enterprise Server for SAP Applications 的建議 OS 設定
* SAP Note [1944799] 包含適用於 SUSE Linux Enterprise Server for SAP Applications 的 SAP HANA 指導方針
* SAP Note [2178632] 包含在 Azure 中針對 SAP 回報的所有監視計量詳細資訊。
* SAP Note [2191498] 包含 Azure 中 Linux 所需的 SAP Host Agent 版本。
* SAP Note [2243692] 包含 Azure 中 Linux 上的 SAP 授權相關資訊。
* SAP Note [1984787] 包含 SUSE LINUX Enterprise Server 12 的一般資訊。
* SAP Note [1999351] 包含 Azure Enhanced Monitoring Extension for SAP 的其他疑難排解資訊。
* [SAP Community WIKI](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) 包含 Linux 所需的所有 SAP Note。
* [適用於 SAP on Linux 的 Azure 虛擬機器規劃和實作][planning-guide]
* [適用於 SAP on Linux 的 Azure 虛擬機器部署 (本文)][deployment-guide]
* [適用於 SAP on Linux 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* [SUSE Linux Enterprise 高可用性擴充 12 SP3 最佳做法指南][sles-hae-guides]
  * 搭配 DRBD 與 Pacemaker 的高可用性 NFS 儲存體
* [適用於 SAP Applications 12 SP3 的 SUSE Linux Enterprise Server 最佳做法指南][sles-for-sap-bp]

## <a name="overview"></a>概觀

為了實現高可用性，SAP NetWeaver 需要使用 NFS 伺服器。 NFS 伺服器會設定於不同叢集中，並可供多個 SAP 系統使用。

![SAP NetWeaver 高可用性概觀](./media/high-availability-guide-nfs/ha-suse-nfs.png)

NFS 伺服器會針對每個使用此 NFS 伺服器的 SAP 系統，使用專用的虛擬主機名稱和虛擬 IP 位址。 在 Azure 上必須有負載平衡器才能使用虛擬 IP 位址。 下列清單顯示負載平衡器的組態。        

* 前端組態
  * NW1 的 IP 位址為 10.0.0.4
  * NW2 的 IP 位址為 10.0.0.5
* 後端組態
  * 連線到應該屬於 NFS 叢集一部分之所有虛擬機器的主要網路介面
* 探查連接埠
  * NW1 的連接埠為 61000
  * NW2 的連接埠為 61001
* 負載平衡規則
  * NW1 的 TCP 為 2049
  * NW1 的 UDP 為 2049
  * NW2 的 TCP 為 2049
  * NW2 的 UDP 為 2049

## <a name="set-up-a-highly-available-nfs-server"></a>設定高可用性 NFS 伺服器

您可以使用來自 GitHub 的 Azure 範本來部署所有必要的 Azure 資源，包括虛擬機器、可用性設定組及負載平衡器，也可以手動部署資源。

### <a name="deploy-linux-via-azure-template"></a>透過 Azure 範本部署 Linux

Azure Marketplace 包含 SUSE Linux Enterprise Server for SAP Applications 12 的映像，讓您可用來部署新的虛擬機器。
您可以使用 GitHub 上的其中一個快速入門範本來部署所有必要資源。 範本會部署虛擬機器、負載平衡器、可用性設定組等。請遵循下列步驟來部署範本：

1. 在 Azure 入口網站中開啟 [SAP 檔案伺服器範本][template-file-server]   
1. 輸入下列參數
   1. 資源前置詞  
      輸入您想要使用的前置詞。 該值會作為所部署之資源的前置詞。
   2. SAP 系統計數  
      輸入將使用此檔案伺服器的 SAP 系統數目。 這會部署所需數量的前端設定、負載平衡規則、探查連接埠、磁碟等。
   3. OS 類型  
      選取一個 Linux 發行版本。 在此範例中，請選取 SLES 12
   4. 管理員使用者名稱和管理員密碼  
      建立可用來登入電腦的新使用者。
   5. 子網路識別碼  
      虛擬機器應該連接的子網路識別碼。 如果您想要建立新的虛擬網路，請讓此參數保持空白，或者，您也可以選取將虛擬機器連線到內部部署網路之 VPN 或快速路由虛擬網路的子網路。 識別碼通常如下所示：/subscriptions/**&lt;訂用帳戶識別碼&gt;**/resourceGroups/**&lt;資源群組名稱&gt;**/providers/Microsoft.Network/virtualNetworks/**&lt;虛擬網路名稱&gt;**/subnets/**&lt;子網路名稱&gt;**

### <a name="deploy-linux-manually-via-azure-portal"></a>透過 Azure 入口網站手動部署 Linux

您必須先為此 NFS 叢集建立虛擬機器。 之後，您需建立負載平衡器，然後使用後端集區中的虛擬機器。

1. 建立資源群組
1. 建立虛擬網路
1. 建立可用性設定組  
   設定更新網域上限
1. 建立虛擬機器 1。至少使用 SLES4SAP 12 SP3，此範例中使用 SLES4SAP 12 SP3 BYOS 映像 SLES For SAP Applications 12 SP3 (BYOS)  
   選取稍早建立的「可用性設定組」  
1. 建立虛擬機器 2。至少使用 SLES4SAP 12 SP3，此範例中使用 SLES4SAP 12 SP3 BYOS 映像  
   SLES For SAP Applications 12 SP3 (BYOS)  
   選取稍早建立的「可用性設定組」  
1. 在兩部虛擬機器上為每個 SAP 系統各新增一個資料磁碟。
1. 建立負載平衡器 (內部)  
   1. 建立前端 IP 位址
      1. NW1 的 IP 位址為 10.0.0.4
         1. 開啟負載平衡器，選取前端 IP 集區，然後按一下 [新增]
         1. 輸入新前端 IP 集區 的名稱 (例如 **nw1-frontend**)
         1. 將 [指派] 設定為 [靜態]，然後輸入 IP 位址 (例如 **10.0.0.4**)
         1. Click OK
      1. NW2 的 IP 位址為 10.0.0.5
         * 針對 NW2 重複上述步驟
   1. 建立後端集區
      1. 連線到應屬於 NW1 之 NFS 叢集的所有虛擬機器的主要網路介面
         1. 開啟負載平衡器，選取後端集區，然後按一下 [新增]
         1. 輸入新後端集區的名稱 (例如 **nw1-backend**)
         1. 按一下 [新增虛擬機器]
         1. 選取您稍早建立的可用性設定組
         1. 選取 NFS 叢集的虛擬機器
         1. Click OK
      1. 連線到應屬於 NW2 之 NFS 叢集的所有虛擬機器的主要網路介面
         * 重複上述步驟來為 NW2 建立後端集區
   1. 建立健康狀態探查
      1. NW1 的連接埠為 61000
         1. 開啟負載平衡器，選取健康情況探查，然後按一下 [新增]
         1. 輸入新健康狀態探查的名稱 (例如 **nw1-hp**)
         1. 選取 [TCP] 作為通訊協定、連接埠 610**00**，保留 [間隔] 5 和 [狀況不良閾值] 2
         1. Click OK
      1. NW2 的連接埠為 61001
         * 重複上述步驟來為 NW2 建立健康狀態探查
   1. 負載平衡規則
      1. NW1 的 TCP 為 2049
         1. 開啟負載平衡器、選取負載平衡規則，然後按一下 [新增]
         1. 輸入新負載平衡器規則的名稱 (例如 **nw1-lb-2049**)
         1. 選取您稍早建立的前端 IP 位址、後端集區及健康狀態探查 (例如 **nw1-frontend**)
         1. 保留通訊協定 [TCP]，輸入連接埠 **2049**
         1. 將閒置逾時增加為 30 分鐘
         1. **務必啟用浮動 IP**
         1. Click OK
      1. NW1 的 UDP 為 2049
         * 重複上述步驟來為 NW1 設定連接埠 2049 和 UDP
      1. NW2 的 TCP 為 2049
         * 重複上述步驟來為 NW2 設定連接埠 2049 和 TCP
      1. NW2 的 UDP 為 2049
         * 重複上述步驟來為 NW2 設定連接埠 2049 和 UDP

### <a name="create-pacemaker-cluster"></a>建立 Pacemaker 叢集

依照[在 Azure 中於 SUSE Linux Enterprise Server 上設定 Pacemaker](high-availability-guide-suse-pacemaker.md) 中的步驟，建立此 NFS 伺服器的基本 Pacemaker 叢集。

### <a name="configure-nfs-server"></a>設定 NFS 伺服器

下列項目會加上下列其中一個前置詞：**[A]** - 適用於所有節點、**[1]** - 僅適用於節點 1 或 **[2]** - 僅適用於節點 2。

1. **[A]** 設定主機名稱解析

   您可以使用 DNS 伺服器，或修改所有節點上的 /etc/hosts。 這個範例示範如何使用 /etc/hosts 檔案。
   取代下列命令中的 IP 位址和主機名稱

   <pre><code>sudo vi /etc/hosts
   </code></pre>
   
   將下列幾行插入至 /etc/hosts。 變更 IP 位址和主機名稱以符合您的環境
   
   <pre><code># IP address of the load balancer frontend configuration for NFS
   <b>10.0.0.4 nw1-nfs</b>
   <b>10.0.0.5 nw2-nfs</b>
   </code></pre>

1. **[A]** 啟用 NFS 伺服器

   建立根 NFS 匯出項目

   <pre><code>sudo sh -c 'echo /srv/nfs/ *\(rw,no_root_squash,fsid=0\)>/etc/exports'
   
   sudo mkdir /srv/nfs/
   </code></pre>

1. **[A]** 安裝 drbd 元件

   <pre><code>sudo zypper install drbd drbd-kmp-default drbd-utils
   </code></pre>

1. **[A]** 為 drbd 裝置建立分割區

   列出所有可用的資料磁碟

   <pre><code>sudo ls /dev/disk/azure/scsi1/
   </code></pre>

   範例輸出
   
   ```
   lun0  lun1
   ```

   為每個資料磁碟建立分割區

   <pre><code>sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun0'
   sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun1'
   </code></pre>

1. **[A]** 建立 LVM 組態

   列出所有可用的分割區

   <pre><code>ls /dev/disk/azure/scsi1/lun*-part*
   </code></pre>

   範例輸出
   
   ```
   /dev/disk/azure/scsi1/lun0-part1  /dev/disk/azure/scsi1/lun1-part1
   ```

   為每個分割區建立 LVM 磁碟區

   <pre><code>sudo pvcreate /dev/disk/azure/scsi1/lun0-part1  
   sudo vgcreate vg-<b>NW1</b>-NFS /dev/disk/azure/scsi1/lun0-part1
   sudo lvcreate -l 100%FREE -n <b>NW1</b> vg-<b>NW1</b>-NFS

   sudo pvcreate /dev/disk/azure/scsi1/lun1-part1
   sudo vgcreate vg-<b>NW2</b>-NFS /dev/disk/azure/scsi1/lun1-part1
   sudo lvcreate -l 100%FREE -n <b>NW2</b> vg-<b>NW2</b>-NFS
   </code></pre>

1. **[A]** 設定 drbd

   <pre><code>sudo vi /etc/drbd.conf
   </code></pre>

   確定 drbd.conf 檔案包含以下兩行

   <pre><code>include "drbd.d/global_common.conf";
   include "drbd.d/*.res";
   </code></pre>

   變更全域 drbd 設定

   <pre><code>sudo vi /etc/drbd.d/global_common.conf
   </code></pre>

   將下列項目新增至處理常式和網路區段。

   <pre><code>global {
        usage-count no;
   }
   common {
        handlers {
             fence-peer "/usr/lib/drbd/crm-fence-peer.sh";
             after-resync-target "/usr/lib/drbd/crm-unfence-peer.sh";
             split-brain "/usr/lib/drbd/notify-split-brain.sh root";
             pri-lost-after-sb "/usr/lib/drbd/notify-pri-lost-after-sb.sh; /usr/lib/drbd/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
        }
        startup {
             wfc-timeout 0;
        }
        options {
        }
        disk {
             md-flushes yes;
             disk-flushes yes;
             c-plan-ahead 1;
             c-min-rate 100M;
             c-fill-target 20M;
             c-max-rate 4G;
        }
        net {
             after-sb-0pri discard-younger-primary;
             after-sb-1pri discard-secondary;
             after-sb-2pri call-pri-lost-after-sb;
             protocol     C;
             tcp-cork yes;
             max-buffers 20000;
             max-epoch-size 20000;
             sndbuf-size 0;
             rcvbuf-size 0;
        }
   }
   </code></pre>

1. **[A]** 建立 NFS drbd 裝置

   <pre><code>sudo vi /etc/drbd.d/<b>NW1</b>-nfs.res
   </code></pre>

   插入新 drbd 裝置的組態並結束

   <pre><code>resource <b>NW1</b>-nfs {
        protocol     C;
        disk {
             on-io-error       detach;
        }
        on <b>prod-nfs-0</b> {
             address   <b>10.0.0.6:7790</b>;
             device    /dev/drbd<b>0</b>;
             disk      /dev/<b>vg-NW1-NFS</b>/<b>NW1</b>;
             meta-disk internal;
        }
        on <b>prod-nfs-1</b> {
             address   <b>10.0.0.7:7790</b>;
             device    /dev/drbd<b>0</b>;
             disk      /dev/<b>vg-NW1-NFS</b>/<b>NW1</b>;
             meta-disk internal;
        }
   }
   </code></pre>

   <pre><code>sudo vi /etc/drbd.d/<b>NW2</b>-nfs.res
   </code></pre>

   插入新 drbd 裝置的組態並結束

   <pre><code>resource <b>NW2</b>-nfs {
        protocol     C;
        disk {
             on-io-error       detach;
        }
        on <b>prod-nfs-0</b> {
             address   <b>10.0.0.6:7791</b>;
             device    /dev/drbd<b>1</b>;
             disk      /dev/<b>vg-NW2-NFS</b>/<b>NW2</b>;
             meta-disk internal;
        }
        on <b>prod-nfs-1</b> {
             address   <b>10.0.0.7:7791</b>;
             device    /dev/drbd<b>1</b>;
             disk      /dev/<b>vg-NW2-NFS</b>/<b>NW2</b>;
             meta-disk internal;
        }
   }
   </code></pre>

   建立 drbd 裝置並加以啟動

   <pre><code>sudo drbdadm create-md <b>NW1</b>-nfs
   sudo drbdadm create-md <b>NW2</b>-nfs
   sudo drbdadm up <b>NW1</b>-nfs
   sudo drbdadm up <b>NW2</b>-nfs
   </code></pre>

1. **[1]** 略過首次同步處理

   <pre><code>sudo drbdadm new-current-uuid --clear-bitmap <b>NW1</b>-nfs
   sudo drbdadm new-current-uuid --clear-bitmap <b>NW2</b>-nfs
   </code></pre>

1. **[1]** 設定主要節點

   <pre><code>sudo drbdadm primary --force <b>NW1</b>-nfs
   sudo drbdadm primary --force <b>NW2</b>-nfs
   </code></pre>

1. **[1]** 等候新的 drbd 裝置完成同步處理

   <pre><code>sudo drbdsetup wait-sync-resource NW1-nfs
   sudo drbdsetup wait-sync-resource NW2-nfs
   </code></pre>

1. **[1]** 在 drbd 裝置上建立檔案系統

   <pre><code>sudo mkfs.xfs /dev/drbd0
   sudo mkdir /srv/nfs/NW1
   sudo chattr +i /srv/nfs/NW1
   sudo mount -t xfs /dev/drbd0 /srv/nfs/NW1
   sudo mkdir /srv/nfs/NW1/sidsys
   sudo mkdir /srv/nfs/NW1/sapmntsid
   sudo mkdir /srv/nfs/NW1/trans
   sudo mkdir /srv/nfs/NW1/ASCS
   sudo mkdir /srv/nfs/NW1/ASCSERS
   sudo mkdir /srv/nfs/NW1/SCS
   sudo mkdir /srv/nfs/NW1/SCSERS
   sudo umount /srv/nfs/NW1

   sudo mkfs.xfs /dev/drbd1
   sudo mkdir /srv/nfs/NW2
   sudo chattr +i /srv/nfs/NW2
   sudo mount -t xfs /dev/drbd1 /srv/nfs/NW2
   sudo mkdir /srv/nfs/NW2/sidsys
   sudo mkdir /srv/nfs/NW2/sapmntsid
   sudo mkdir /srv/nfs/NW2/trans
   sudo mkdir /srv/nfs/NW2/ASCS
   sudo mkdir /srv/nfs/NW2/ASCSERS
   sudo mkdir /srv/nfs/NW2/SCS
   sudo mkdir /srv/nfs/NW2/SCSERS
   sudo umount /srv/nfs/NW2
   </code></pre>

1. **[A]** 設定 drbd 核心分裂偵測

   使用 drbd 將資料從一部主機同步處理至另一部主機時，可能發生所謂的核心分裂。 核心分裂係指兩個叢集節點都將 drbd 裝置升階成主要裝置，而造成不同步的情況。這種情況可能相當罕見，但您仍然會想要儘快處理和解決核心分裂問題。 因此，在發生核心分裂時收到通知便相當重要。

   如需了解如何設定核心分裂通知，請參閱[官方 drbd 文件](http://docs.linbit.com/doc/users-guide-83/s-configure-split-brain-behavior/#s-split-brain-notification) \(英文\)。

   也有可能從核心分裂情況自動復原。 如需詳細資訊，請參閱[核心分裂自動復原原則](http://docs.linbit.com/doc/users-guide-83/s-configure-split-brain-behavior/#s-automatic-split-brain-recovery-configuration) \(英文\)
   
### <a name="configure-cluster-framework"></a>設定叢集架構

1. **[1]** 將 SAP 系統 NW1 的 NFS drbd 裝置新增至叢集設定

   <pre><code>sudo crm configure rsc_defaults resource-stickiness="200"

   # Enable maintenance mode
   sudo crm configure property maintenance-mode=true
   
   sudo crm configure primitive drbd_<b>NW1</b>_nfs \
     ocf:linbit:drbd \
     params drbd_resource="<b>NW1</b>-nfs" \
     op monitor interval="15" role="Master" \
     op monitor interval="30" role="Slave"
   
   sudo crm configure ms ms-drbd_<b>NW1</b>_nfs drbd_<b>NW1</b>_nfs \
     meta master-max="1" master-node-max="1" clone-max="2" \
     clone-node-max="1" notify="true" interleave="true"
   
   sudo crm configure primitive fs_<b>NW1</b>_sapmnt \
     ocf:heartbeat:Filesystem \
     params device=/dev/drbd0 \
     directory=/srv/nfs/<b>NW1</b>  \
     fstype=xfs \
     op monitor interval="10s"
   
   sudo crm configure primitive nfsserver systemd:nfs-server \
     op monitor interval="30s"
   sudo crm configure clone cl-nfsserver nfsserver

   sudo crm configure primitive exportfs_<b>NW1</b> \
     ocf:heartbeat:exportfs \
     params directory="/srv/nfs/<b>NW1</b>" \
     options="rw,no_root_squash,crossmnt" clientspec="*" fsid=1 wait_for_leasetime_on_stop=true op monitor interval="30s"
   
   sudo crm configure primitive vip_<b>NW1</b>_nfs \
     IPaddr2 \
     params ip=<b>10.0.0.4</b> cidr_netmask=<b>24</b> op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW1</b>_nfs \
     anything \
     params binfile="/usr/bin/nc" cmdline_options="-l -k <b>61000</b>" op monitor timeout=20s interval=10 depth=0
   
   sudo crm configure group g-<b>NW1</b>_nfs \
     fs_<b>NW1</b>_sapmnt exportfs_<b>NW1</b> nc_<b>NW1</b>_nfs vip_<b>NW1</b>_nfs
   
   sudo crm configure order o-<b>NW1</b>_drbd_before_nfs inf: \
     ms-drbd_<b>NW1</b>_nfs:promote g-<b>NW1</b>_nfs:start
   
   sudo crm configure colocation col-<b>NW1</b>_nfs_on_drbd inf: \
     g-<b>NW1</b>_nfs ms-drbd_<b>NW1</b>_nfs:Master
   </code></pre>

1. **[1]** 將 SAP 系統 NW2 的 NFS drbd 裝置新增至叢集設定

   <pre><code># Enable maintenance mode
   sudo crm configure property maintenance-mode=true
   
   sudo crm configure primitive drbd_<b>NW2</b>_nfs \
     ocf:linbit:drbd \
     params drbd_resource="<b>NW2</b>-nfs" \
     op monitor interval="15" role="Master" \
     op monitor interval="30" role="Slave"
   
   sudo crm configure ms ms-drbd_<b>NW2</b>_nfs drbd_<b>NW2</b>_nfs \
     meta master-max="1" master-node-max="1" clone-max="2" \
     clone-node-max="1" notify="true" interleave="true"
   
   sudo crm configure primitive fs_<b>NW2</b>_sapmnt \
     ocf:heartbeat:Filesystem \
     params device=/dev/drbd1 \
     directory=/srv/nfs/<b>NW2</b>  \
     fstype=xfs \
     op monitor interval="10s"
   
   sudo crm configure primitive exportfs_<b>NW2</b> \
     ocf:heartbeat:exportfs \
     params directory="/srv/nfs/<b>NW2</b>" \
     options="rw,no_root_squash" clientspec="*" fsid=2 wait_for_leasetime_on_stop=true op monitor interval="30s"
   
   sudo crm configure primitive vip_<b>NW2</b>_nfs \
     IPaddr2 \
     params ip=<b>10.0.0.5</b> cidr_netmask=<b>24</b> op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW2</b>_nfs \
     anything \
     params binfile="/usr/bin/nc" cmdline_options="-l -k <b>61001</b>" op monitor timeout=20s interval=10 depth=0
   
   sudo crm configure group g-<b>NW2</b>_nfs \
     fs_<b>NW2</b>_sapmnt exportfs_<b>NW2</b> nc_<b>NW2</b>_nfs vip_<b>NW2</b>_nfs
   
   sudo crm configure order o-<b>NW2</b>_drbd_before_nfs inf: \
     ms-drbd_<b>NW2</b>_nfs:promote g-<b>NW2</b>_nfs:start
   
   sudo crm configure colocation col-<b>NW2</b>_nfs_on_drbd inf: \
     g-<b>NW2</b>_nfs ms-drbd_<b>NW2</b>_nfs:Master
   </code></pre>

1. **[1]** 停用維護模式
   
   <pre><code>sudo crm configure property maintenance-mode=false
   </code></pre>

## <a name="next-steps"></a>後續步驟

* [安裝 SAP ASCS 和資料庫](high-availability-guide-suse.md)
* [適用於 SAP 的 Azure 虛擬機器規劃和實作][planning-guide]
* [適用於 SAP 的 Azure 虛擬機器部署][deployment-guide]
* [適用於 SAP 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* 若要了解如何建立高可用性並為 Azure 上的 SAP HANA 規劃災害復原，請參閱 [Azure 上的 SAP HANA (大型執行個體) 高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md)。
* 若要了解如何建立高可用性並為 Azure VM 上的 SAP HANA 規劃災害復原，請參閱 [Azure 虛擬機器 (VM) 上 SAP HANA 的高可用性][sap-hana-ha]
