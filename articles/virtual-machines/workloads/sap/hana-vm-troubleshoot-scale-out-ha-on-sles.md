---
title: 針對在 Azure 虛擬機器上使用 SLES 12 SP3 設定 SAP HANA 2.0 scale-out HSR-Pacemaker 進行疑難排解 | Microsoft Docs
description: 在 Azure 虛擬機器執行的 SLES 12 SP3 上，針對根據 SAP HANA System Replication (HSR) 和 Pacemaker 的複雜 SAP HANA scale-out 高可用性設定進行檢查和疑難排解的指南
services: virtual-machines-linux
documentationcenter: ''
author: hermannd
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/24/2018
ms.author: hermannd
ms.openlocfilehash: 6c0d6397246e8b8db1d59c26229e37a722d49f48
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47184532"
---
# <a name="verify-and-troubleshoot-sap-hana-scale-out-high-availability-setup-on-sles-12-sp3"></a>在 SLES 12 SP3 上驗證 SAP HANA scale-out 高可用性設定並為其進行疑難排解 

[sles-pacemaker-ha-guide]:high-availability-guide-suse-pacemaker.md
[sles-hana-scale-out-ha-paper]:https://www.suse.com/documentation/suse-best-practices/singlehtml/SLES4SAP-hana-scaleOut-PerfOpt-12/SLES4SAP-hana-scaleOut-PerfOpt-12.html
[sap-hana-iaas-list]:https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html
[suse-pacemaker-support-log-files]:https://www.suse.com/support/kb/doc/?id=7022702
[azure-linux-multiple-nics]:https://docs.microsoft.com/azure/virtual-machines/linux/multiple-nics
[suse-cloud-netconfig]:https://www.suse.com/c/multi-nic-cloud-netconfig-ec2-azure/
[sap-list-port-numbers]:https://help.sap.com/viewer/ports
[sles-12-ha-paper]:https://www.suse.com/documentation/sle-ha-12/pdfdoc/book_sleha/book_sleha.pdf
[sles-zero-downtime-paper]:https://www.suse.com/media/presentation/TUT90846_towards_zero_downtime%20_how_to_maintain_sap_hana_system_replication_clusters.pdf
[sap-nw-ha-guide-sles]:high-availability-guide-suse.md
[sles-12-for-sap]:https://www.suse.com/media/white-paper/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf


本文撰寫目的是要協助檢查 Azure 虛擬機器上執行的 SAP HANA scale-out 的 Pacemaker 叢集設定。 叢集設定是與 SAP HANA System Replication (HSR) 和 SUSE RPM 套件 SAPHanaSR-ScaleOut 一起完成。 所有測試都只是在 SUSE SLES 12 SP3 上完成。 有數個章節涵蓋不同區域且包含來自設定檔的範例命令和摘要。 這些範例建議作為驗證和檢查整個叢集設定的方法。



## <a name="important-notes"></a>重要事項

SAP HANA scale-out 與 SAP HANA System Replication 和 Pacemaker 的所有測試都僅使用 SAP HANA 2.0 完成。 作業系統版本是 SUSE Linux Enterprise Server 12 SP3 for SAP Applications。 此外，還會使用 SUSE 中的最新 RPM 套件 SAPHanaSR-ScaleOut 來設定 pacemaker 叢集。
SUSE 已發佈此效能最佳化設定的詳細描述，而詳細描述位在[這裡][sles-hana-scale-out-ha-paper]

針對 SAP HANA scale-out 所支援的虛擬機器類型，檢查 [SAP HANA 認證 IaaS 目錄][sap-hana-iaas-list]

搭配使用 SAP HANA scale-out 與多個子網路和 vNIC 並設定 HSR 時發生技術問題。 必須使用已修正此問題的最新 SAP HANA 2.0 修補程式。 支援的 SAP HANA 版本如下： 

**rev2.00.024.04 或更高版本與 rev2.00.032 或更高版本。**

如果是需要 SUSE 支援的情況，請遵循本[指南][suse-pacemaker-support-log-files]。 收集所有 SAP HANA HA 叢集資訊，如本文中所述。 SUSE 支援需要這項資訊，以進一步分析。

在內部測試期間，叢集設定會與透過 Azure 入口網站的一般正常 VM 關機混淆。 因此，建議您透過其他方法來測試叢集容錯移轉。 請使用強制核心異常或關閉網路這類方法，或移轉 **msl** 資源 (請參閱下列各節中的詳細資料)。 假設蓄意發生標準關機。 蓄意關機的最佳範例是維護 (如需詳細資料，請參閱有關計劃性維護一節)。

在內部測試期間，叢集設定於手動 SAP HANA 接管之後會混淆，而叢集處於維護模式。 建議先手動重新將其切回，再結束叢集維護模式。 另一個選項是先觸發容錯移轉，再將叢集置入維護模式 (如需詳細資料，請參閱有關計劃性維護一節)。 SUSE 文件描述在這部分如何使用 crm 命令重設叢集。 但是，在內部測試期間，之前所述的方法似乎更為穩固，而且絕不會顯示任何未預期的副作用。

使用 crm migrate 命令時，請記得清除叢集設定。 它會新增您可能不知道的位置條件約束。 這些條件約束會影響叢集行為 (請參閱計劃性維護一節中的詳細資料)。



## <a name="test-system-description"></a>測試系統描述

針對 SAP HANA scale-out HA 驗證和認證，會使用設定，其中包含各有三個 SAP HANA 節點的兩個系統，即一個主要節點及兩個背景工作節點。 以下是 VM 名稱和內部 IP 位址清單。 所有更深入的驗證範例都是在這些 VM 上完成。 在命令範例中使用這些 VM 名稱和 IP 位址，應該有助於深入了解命令和其輸出。


| 節點類型 | VM 名稱 | IP 位址 |
| --- | --- | --- |
| 站台 1 上的主要節點 | hso-hana-vm-s1-0 | 10.0.0.30 |
| 站台 1 上的背景工作節點 1 | hso-hana-vm-s1-1 | 10.0.0.31 |
| 站台 1 上的背景工作節點 2 | hso-hana-vm-s1-2 | 10.0.0.32 |
| | | |
| 站台 2 上的主要節點 | hso-hana-vm-s2-0 | 10.0.0.40 |
| 站台 2 上的背景工作節點 1 | hso-hana-vm-s2-1 | 10.0.0.41 |
| 站台 2 上的背景工作節點 2 | hso-hana-vm-s2-2  | 10.0.0.42 |
| | | |
| 多數製作者節點 | hso-hana-dm | 10.0.0.13 |
| SBD 裝置伺服器 | hso-hana-sbd | 10.0.0.19 |
| | | |
| NFS 伺服器 1 | hso-nfs-vm-0 | 10.0.0.15 |
| NFS 伺服器 2 | hso-nfs-vm-1 | 10.0.0.14 |



## <a name="multiple-subnets-and-vnics"></a>多個子網路和 vNIC

在遵循 SAP HANA 網路建議之後，會在一個 Azure 虛擬網路內建立三個子網路。 必須以非共用模式安裝 Azure 上的 SAP HANA scale-out，表示每個節點都會使用本機磁碟區 **/hana/data** 和 **/hana/log**。 因為僅使用本機磁碟區，所以不需要定義個別子網路予以儲存：

- 10.0.2.0/24   適用於 SAP HANA 節點間通訊
- 10.0.1.0/24   適用於 SAP HANA System Replication HSR
- 10.0.0.0/24   適用於其餘作業

如需使用多個網路的 SAP HANA 設定資訊，請深入參閱 **global.ini** 一節。

與子網路數目對應，叢集中的每個 VM 都會有三個 vNIC。 [本][azure-linux-multiple-nics]文描述部署 Linux VM 時的 Azure 潛在路由問題。 本特定路由主題只適用於使用多個 vNIC。 根據預設，在 SLES 12 SP3 中，是由 SUSE 來解決問題。 在[這裡][suse-cloud-netconfig]可以找到有關本主題的 SUSE 文章。


驗證是否正確設定 SAP HANA 以使用多個網路的基本檢查，僅需要執行下面的命令。 第一個步驟只需要仔細檢查所有三個子網路的所有三個內部 IP 位址都為使用中的 OS 層級。 如果您定義具有不同 IP 位址範圍的子網路，則必須調整命令：

<pre><code>
ifconfig | grep "inet addr:10\."
</code></pre>

以下是來自站台 2 第二個背景工作節點的範例輸出。 您可以看到來自 eth0、eth1 和 eth2 的三個不同內部 IP 位址：

<pre><code>
inet addr:10.0.0.42  Bcast:10.0.0.255  Mask:255.255.255.0
inet addr:10.0.1.42  Bcast:10.0.1.255  Mask:255.255.255.0
inet addr:10.0.2.42  Bcast:10.0.2.255  Mask:255.255.255.0
</code></pre>


第二個步驟是驗證 nameserver 和 HSR 的 SAP HANA 連接埠。 SAP HANA 應該接聽對應的子網路。 根據 SAP HANA 執行個體數目，您必須調整命令。 在測試系統中，執行個體數目是 **00**。 有不同的方法可以找出所使用的連接埠。 

您在下面會看到 SQL 陳述式，其傳回其他資訊，例如執行個體識別碼和執行個體數目：

<pre><code>
select * from "SYS"."M_SYSTEM_OVERVIEW"
</code></pre>

例如，若要尋找正確的連接埠號碼，您可以查看 HANA Studio 的 [Configuration] \(設定\) 或透過 SQL 陳述式：

<pre><code>
select * from M_INIFILE_CONTENTS WHERE KEY LIKE 'listen%'
</code></pre>

若要尋找 SAP 軟體堆疊中所使用的每個連接埠 (包含 SAP HANA)，請搜尋[這裡][sap-list-port-numbers]。

假設 SAP HANA 2.0 測試系統中的執行個體數目為 **00**，nameserver 的連接埠號碼為 **30001**。 HSR 中繼資料通訊的連接埠號碼為 **40002**。 其中一個選項是登入背景工作節點，然後檢查主要節點服務。 這裡檢查站台 2 上的背景工作節點 2，請嘗試連線至站台 2 上的主要節點。

檢查 nameserver 連接埠：

<pre><code>
nc -vz 10.0.0.40 30001
nc -vz 10.0.1.40 30001
nc -vz 10.0.2.40 30001
</code></pre>

結果應該類似下面的範例輸出，證明節點間通訊使用子網路 **10.0.2.0/24**。
應該只有透過子網路 **10.0.2.0/24** 的連線會成功：

<pre><code>
nc: connect to 10.0.0.40 port 30001 (tcp) failed: Connection refused
nc: connect to 10.0.1.40 port 30001 (tcp) failed: Connection refused
Connection to 10.0.2.40 30001 port [tcp/pago-services1] succeeded!
</code></pre>

現在，檢查 HSR 連接埠 **40002**：

<pre><code>
nc -vz 10.0.0.40 40002
nc -vz 10.0.1.40 40002
nc -vz 10.0.2.40 40002
</code></pre>

結果應該類似下面的範例輸出，證明 HSR 通訊使用子網路 **10.0.1.0/24**。
應該只有透過子網路 **10.0.1.0/24** 的連線會成功：

<pre><code>
nc: connect to 10.0.0.40 port 40002 (tcp) failed: Connection refused
Connection to 10.0.1.40 40002 port [tcp/*] succeeded!
nc: connect to 10.0.2.40 port 40002 (tcp) failed: Connection refused
</code></pre>



## <a name="corosync"></a>Corosync


在叢集的每個節點上，corosync 設定檔必須正確 (包含多數製作者節點)。 如果節點的叢集聯結未如預期運作，請手動在所有節點上建立和/或複製 **/etc/corosync/corosync.conf**，並重新啟動服務。

例如，以下是測試系統的 **corosync.conf** 內容。

第一個區段是 **totem**，如本[文件][sles-pacemaker-ha-guide] (區段叢集安裝步驟 11) 中所述。 您可以忽略 **mcastaddr** 的值。 只需要保留現有項目。 必須根據 Microsoft Azure SAP HANA 文件設定 **token** 和 **consensus** 的項目 (可以在[這裡][sles-pacemaker-ha-guide]找到)

<pre><code>
totem {
    version:    2
    secauth:    on
    crypto_hash:    sha1
    crypto_cipher:  aes256
    cluster_name:   hacluster
    clear_node_high_bit: yes

    token:      30000
    token_retransmits_before_loss_const: 10
    join:       60
    consensus:  36000
    max_messages:   20

    interface {
        ringnumber:     0
        bindnetaddr: 10.0.0.0
        mcastaddr:  239.170.19.232
        mcastport:  5405

        ttl:        1
    }
    transport:      udpu

}
</code></pre>

第二個區段 **logging** 未變更指定的預設值：

<pre><code>
logging {
    fileline:   off
    to_stderr:  no
    to_logfile:     no
    logfile:    /var/log/cluster/corosync.log
    to_syslog:  yes
    debug:      off
    timestamp:  on
    logger_subsys {
        subsys:     QUORUM
        debug:  off
    }
}
</code></pre>

第三個區段顯示 **nodelist**。 必須顯示叢集的所有節點與其節點識別碼：

<pre><code>
nodelist {
  node {
   ring0_addr:hso-hana-vm-s1-0
   nodeid: 1
   }
  node {
   ring0_addr:hso-hana-vm-s1-1
   nodeid: 2
   }
  node {
   ring0_addr:hso-hana-vm-s1-2
   nodeid: 3
   }
  node {
   ring0_addr:hso-hana-vm-s2-0
   nodeid: 4
   }
  node {
   ring0_addr:hso-hana-vm-s2-1
   nodeid: 5
   }
  node {
   ring0_addr:hso-hana-vm-s2-2
   nodeid: 6
   }
  node {
   ring0_addr:hso-hana-dm
   nodeid: 7
   }
}
</code></pre>

在最後一個區段 **quorum** 中，務必正確地設定 **expected_votes** 的值。 它必須是節點數目 (包含多數製作者節點)。 而且 **two_node** 的值必須是 **0**。 請不要整個移除項目。 只需要將值設定為 **0**。

<pre><code>
quorum {
    # Enable and configure quorum subsystem (default: off)
    # see also corosync.conf.5 and votequorum.5
    provider: corosync_votequorum
    expected_votes: 7
    two_node: 0
}
</code></pre>


透過 **systemctl** 重新啟動服務：

<pre><code>
systemctl restart corosync
</code></pre>




## <a name="sbd-device"></a>SBD 裝置

[這裡][sles-pacemaker-ha-guide] (區段 sbd-fencing) 描述如何在 Azure VM 上設定 SBD 裝置的文件。

如果叢集中的每個節點都有 ACL 項目，則再次檢查的第一件事是查看 SBD 伺服器 VM。 在 SBD 伺服器 VM 上執行下列命令：


<pre><code>
targetcli ls
</code></pre>


在測試系統上，命令的輸出與下面範例類似。 必須輸入 **iqn.2006-04.hso-db-0.local:hso-db-0** 這類 ACL 名稱，作為 VM 上的對應啟動器名稱。 每個 VM 都需要不同的項目。

<pre><code>
 | | o- sbddbhso ................................................................... [/sbd/sbddbhso (50.0MiB) write-thru activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2006-04.dbhso.local:dbhso ..................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 7]
  |     | o- iqn.2006-04.hso-db-0.local:hso-db-0 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-1.local:hso-db-1 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-2.local:hso-db-2 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-3.local:hso-db-3 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-4.local:hso-db-4 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-5.local:hso-db-5 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-6.local:hso-db-6 .................................................................. [Mapped LUNs: 1]
</code></pre>

之後，請檢查所有 VM 上的啟動器名稱都不同，並且對應至上面顯示的項目。 以下是站台 1 上的背景工作節點 1 的其中一個範例：

<pre><code>
cat /etc/iscsi/initiatorname.iscsi
</code></pre>

輸出與下面範例類似：

<pre><code>
##
## /etc/iscsi/iscsi.initiatorname
##
## Default iSCSI Initiatorname.
##
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
## If you change the InitiatorName, existing access control lists
## may reject this initiator.  The InitiatorName must be unique
## for each iSCSI initiator.  Do NOT duplicate iSCSI InitiatorNames.
InitiatorName=iqn.2006-04.hso-db-1.local:hso-db-1
</code></pre>

接下來，請驗證 **discovery** 是否正確運作，並在使用 SBD 伺服器 VM IP 位址的每個叢集節點上執行下列命令：

<pre><code>
iscsiadm -m discovery --type=st --portal=10.0.0.19:3260
</code></pre>

輸出應該與下面範例類似：

<pre><code>
10.0.0.19:3260,1 iqn.2006-04.dbhso.local:dbhso
</code></pre>

下一個證明點是驗證節點看到 SDB 裝置。 在每個節點 (包括多數製作者節點) 上檢查它：

<pre><code>
lsscsi | grep dbhso
</code></pre>

輸出應該與下面範例類似。 請注意，名稱可能會不同 (在 VM 重新開機之後，裝置名稱也可能會變更)：

<pre><code>
[6:0:0:0]    disk    LIO-ORG  sbddbhso         4.0   /dev/sdm
</code></pre>

根據系統的狀態，重新啟動 iSCSI 服務有時可能有助於解決問題。 然後，執行下列命令：

<pre><code>
systemctl restart iscsi
systemctl restart iscsid
</code></pre>


從任何節點中，您可以檢查所有節點是否 **clear**。 只需要留意在特定節點上使用正確的裝置名稱：

<pre><code>
sbd -d /dev/sdm list
</code></pre>

針對叢集中的每個節點，輸出應該顯示 **clear**：

<pre><code>
0       hso-hana-vm-s1-0        clear
1       hso-hana-vm-s2-2        clear
2       hso-hana-vm-s2-1        clear
3       hso-hana-dm     clear
4       hso-hana-vm-s1-1        clear
5       hso-hana-vm-s2-0        clear
6       hso-hana-vm-s1-2        clear
</code></pre>


另一項 SBD 檢查是 sbd 命令的 **dump** 選項。 下列範例命令和輸出來自裝置名稱不是 **sdm** 而是 **sdd** 的多數製作者節點：

<pre><code>
sbd -d /dev/sdd dump
</code></pre>

所有節點上的輸出 (除了裝置名稱之外) 應該都相同：

<pre><code>
==Dumping header on disk /dev/sdd
Header version     : 2.1
UUID               : 9fa6cc49-c294-4f1e-9527-c973f4d5a5b0
Number of slots    : 255
Sector size        : 512
Timeout (watchdog) : 60
Timeout (allocate) : 2
Timeout (loop)     : 1
Timeout (msgwait)  : 120
==Header on disk /dev/sdd is dumped
</code></pre>

另一項 SBD 檢查是將訊息傳送至另一個節點的可能性。 您在站台 2 上的背景工作節點 1 執行下列命令，才能將訊息傳送至站台 2 上的背景工作節點 2：

<pre><code>
sbd -d /dev/sdm message hso-hana-vm-s2-2 test
</code></pre>

在目標 VM 端 (在此範例中為 **hso-hana-vm-s2-2**)，您可以於 **/var/log/messages** 中找到下列項目：

<pre><code>
/dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68:   notice: servant: Received command test from hso-hana-vm-s2-1 on disk /dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68
</code></pre>

請仔細檢查 **/etc/sysconfig/sbd** 中的項目是否對應至[文件][sles-pacemaker-ha-guide] 中的描述 (區段 sbd-fencing)。 驗證 **/etc/iscsi/iscsid.conf** 中的啟動設定設定為 automatic (自動)。

**/etc/sysconfig/sbd** 中的重要項目 (必要時，請調整識別碼值)：

<pre><code>
SBD_DEVICE="/dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68;"
SBD_PACEMAKER=yes
SBD_STARTMODE=always
SBD_WATCHDOG=yes
</code></pre>


另一項要檢查的項目是 **/etc/iscsi/iscsid.conf** 中的啟動設定。 下面顯示的 **iscsiadm** 命令應該已經執行必要設定，如文件中所述。 需要驗證並使用 **vi** 手動採用它 (如果不同)。

設定啟動行為的命令：

<pre><code>
iscsiadm -m node --op=update --name=node.startup --value=automatic
</code></pre>

**/etc/iscsi/iscsid.conf** 中的項目：

<pre><code>
node.startup = automatic
</code></pre>

在重新啟動 VM 後的測試和驗證期間，不會再看到 SBD 裝置。 啟動設定與 yast2 所顯示的內容有所差異。 若要再次檢查設定，請執行下列步驟：

1. 啟動 yast2
2. 選取左側的 [網路服務]
3. 在右側，向下捲動至 [iSCSI 啟動器] 並予以選取
4. 在下一個畫面的 [服務] 索引標籤下方，您應該會看到節點的唯一啟動器名稱
5. 在啟動器名稱上方，確定 [啟動服務] 值設定為 [When Booting] \(開機時\)
6. 如果不是這種情況，則請將它設定為 [When Booting] \(開機時\)，而不是 [手動]
7. 接下來，將最上層索引標籤切換至 [Connected Targets] \(連線的目標\)
8. 在 [Connected Targets] \(連線的目標\) 畫面上，您應該會看到 SBD 裝置的項目，例如此範例：**10.0.0.19:3260 iqn.2006-04.dbhso.local:dbhso**
9. 檢查 [啟動] 值是否設定為 "**onboot**"
10. 否則，請選擇 [編輯]，並予以變更
11. 儲存變更並結束 yast2



## <a name="pacemaker"></a>Pacemaker

正確設定所有項目之後，即可在每個節點上執行下列命令，以檢查 Pacemaker 服務的狀態：

<pre><code>
systemctl status pacemaker
</code></pre>

輸出頂端應該與下面範例類似。 [使用中] 後的狀態必須顯示為 [loaded] \(已載入\) 和 [active (running)] \(使用中 (執行)\)。 [Loaded] \(已載入\) 後的狀態必須顯示為 [enabled] \(已啟用\)。

<pre><code>
  pacemaker.service - Pacemaker High Availability Cluster Manager
   Loaded: loaded (/usr/lib/systemd/system/pacemaker.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-09-07 05:56:27 UTC; 4 days ago
     Docs: man:pacemakerd
           http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/index.html
 Main PID: 4496 (pacemakerd)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/pacemaker.service
           ├─4496 /usr/sbin/pacemakerd -f
           ├─4499 /usr/lib/pacemaker/cib
           ├─4500 /usr/lib/pacemaker/stonithd
           ├─4501 /usr/lib/pacemaker/lrmd
           ├─4502 /usr/lib/pacemaker/attrd
           ├─4503 /usr/lib/pacemaker/pengine
           └─4504 /usr/lib/pacemaker/crmd
</code></pre>

如果設定仍然為 [disabled] \(已停用\)，則請執行下列命令：

<pre><code>
systemctl enable pacemaker
</code></pre>

若要在 Pacemaker 中查看所有已設定的資源，請執行下列命令：

<pre><code>
crm status
</code></pre>

輸出應該與下面範例類似。 多數製作者 VM (**hso-hana-dm**) 上的 cln 和 msl 資源就會顯示為 [stopped] \(已停止\)。 多數製作者節點上未安裝 SAP HANA。 因此，**cln** 和 **msl** 資源會顯示為 [stopped] \(已停止\)。 它必須顯示正確的 VM 總數 (**7**)。 所有屬於叢集一部分的 VM 必須列為狀態 [連線]。 必須正確地辨識目前主要的主要節點 (在此範例中是 **hso-hana-vm-s1-0**)。

<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Tue Sep 11 15:56:40 2018
Last change: Tue Sep 11 15:56:23 2018 by root via crm_attribute on hso-hana-vm-s1-0

7 nodes configured
17 resources configured

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00]
     Started: [ hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00]
     Masters: [ hso-hana-vm-s1-0 ]
     Slaves: [ hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s1-0
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s1-0
</code></pre>

Pacemaker 的重要功能是讓它進入維護模式。 此模式允許修改 (例如 VM 重新開機)，而不需要導致立即叢集動作。 典型使用案例會是計劃性 OS 或 Azure 基礎結構維護 (另請參閱有關計劃性維護的單獨一節)。 使用下列命令，讓 Pacemaker 進入維護模式：

<pre><code>
crm configure property maintenance-mode=true
</code></pre>

檢查 **crm status** 時，您會在輸出中注意到所有資源都標示為 [非受控]。 在此狀態中，叢集不會反應任何變更，例如啟動/停止 SAP HANA。
以下是叢集處於維護模式時的 **crm status** 命令範例輸出：

<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Wed Sep 12 07:48:10 2018
Last change: Wed Sep 12 07:46:54 2018 by root via cibadmin on hso-hana-vm-s2-1

7 nodes configured
17 resources configured

              *** Resource management is DISABLED ***
  The cluster will not attempt to start, stop or recover services

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm (unmanaged)
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00] (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-1 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-0 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-2 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-1 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-2 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-0 (unmanaged)
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00] (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s1-1 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s1-2 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s2-1 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s2-2 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Master hso-hana-vm-s2-0 (unmanaged)
     Stopped: [ hso-hana-dm hso-hana-vm-s1-0 ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s2-0 (unmanaged)
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s2-0 (unmanaged)
</code></pre>


在下面的命令範例中，您會看到如何結束叢集維護模式：

<pre><code>
crm configure property maintenance-mode=false
</code></pre>


另一個 crm 命令可讓您將完整的叢集設定放入可能用來編輯的編輯器。 儲存變更之後，叢集會啟動適當的動作：

<pre><code>
crm configure edit
</code></pre>

若只要查看完整的叢集設定，請使用 crm **show** 選項：

<pre><code>
crm configure show
</code></pre>



叢集資源失敗之後，**crm status** 命令會顯示 [失敗的動作] 清單。 請參閱下面這個輸出的範例：


<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Thu Sep 13 07:30:44 2018
Last change: Thu Sep 13 07:30:20 2018 by root via crm_attribute on hso-hana-vm-s1-0

7 nodes configured
17 resources configured

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00]
     Started: [ hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00]
     Masters: [ hso-hana-vm-s1-0 ]
     Slaves: [ hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm hso-hana-vm-s2-0 ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s1-0
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s1-0

Failed Actions:
* rsc_SAPHanaCon_HSO_HDB00_monitor_60000 on hso-hana-vm-s2-0 'unknown error' (1): call=86, status=complete, exitreason='none',
    last-rc-change='Wed Sep 12 17:01:28 2018', queued=0ms, exec=277663ms
</code></pre>

必須在失敗之後執行叢集清理。 只需要再次使用 crm 命令，並使用 **cleanup** 命令選項去除這些可命名對應叢集資源的失敗動作項目，如下所示：

<pre><code>
crm resource cleanup rsc_SAPHanaCon_HSO_HDB00
</code></pre>

此命令應該傳回輸出，而輸出與下面範例類似：

<pre><code>
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-dm
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-0
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-1
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-2
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-0
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-1
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-2
Waiting for 7 replies from the CRMd....... OK
</code></pre>



## <a name="failover--takeover"></a>容錯移轉/接管

如具有重要事項的第一節中所述，您不應該使用標準正常關機來測試叢集容錯移轉或 SAP HANA HSR 接管。 相反地，例如，建議您觸發核心異常或強制資源移轉，或可能關閉 VM 之 OS 層級上的所有網路。 另一種方法會是 **crm \<節點\> standby** 命令。 另請參閱[這裡][sles-12-ha-paper]的 SUSE 文件。 您會在下面看到三個強制叢集容錯移轉的範例命令：

<pre><code>
echo c &gt /proc/sysrq-trigger

crm resource migrate msl_SAPHanaCon_HSO_HDB00 hso-hana-vm-s2-0 force

wicked ifdown eth0
wicked ifdown eth1
wicked ifdown eth2
......
wicked ifdown eth&ltn&gt
</code></pre>

而且，如計劃性維護一節中所述，監視叢集活動的一項好方式是搭配執行 **SAPHanaSR-showAttr** 與 **watch** 命令：

<pre><code>
watch SAPHanaSR-showAttr
</code></pre>

此外，最好查看來自 SAP Python 指令碼的 SAP HANA 橫向狀態。 此狀態值就是叢集設定所尋找的值。 考慮到背景工作節點失敗時，就十分清楚。 如果背景工作節點關閉，則 SAP HANA 不會立即傳回整個相應放大系統的健康狀態錯誤。 需要重試數次，避免不必要的容錯移轉。 只有在狀態從正常 (return value 4) 變更為錯誤 (return value 1) 時，叢集才會做出反應。 因此，它是正確的，如果 **SAPHanaSR-showAttr** 的輸出顯示狀態為 [離線] 的 VM，但還沒有活動可以切換主要與次要。 只要 SAP HANA 未傳回錯誤，就不會觸發任何叢集活動。

您可以透過下列方式呼叫 SAP Python 指令碼 (您可能需要調整路徑)，以使用者 \<HANA SID\>adm 身分監視 SAP HANA 橫向健康狀態：

<pre><code>
watch python /hana/shared/HSO/exe/linuxx86_64/HDB_2.00.032.00.1533114046_eeaf4723ec52ed3935ae0dc9769c9411ed73fec5/python_support/landscapeHostConfiguration.py
</code></pre>

此命令的輸出應該與下面範例類似。 重要的是 [主機狀態] 資料行，以及 [整體主機狀態]。 實際輸出實際上會比其他資料行還要寬。
為了更容易閱讀本文件內的輸出資料表，已移除右側的大部分資料行：

<pre><code>
| Host             | Host   | Host   | Failover | Remove | 
|                  | Active | Status | Status   | Status | 
|                  |        |        |          |        | 
| ---------------- | ------ | ------ | -------- | ------ |    .......
| hso-hana-vm-s2-0 | yes    | ok     |          |        |        
| hso-hana-vm-s2-1 | yes    | ok     |          |        |         
| hso-hana-vm-s2-2 | yes    | ok     |          |        |        

overall host status: ok
</code></pre>


還有另一個命令可檢查目前叢集活動。 終止主要站台的主要節點之後，請參閱下面命令和輸出結尾。 您可以看到轉換動作清單，例如將先前的次要主要節點 (**hso-hana-vm-s2-0**) **提升**為新主要的主要節點。 如果一切正常，而且所有活動都已完成，則這份 [轉換摘要] 清單必須為空白。

<pre><code>
 crm_simulate -Ls

...........

Transition Summary:
 * Fence hso-hana-vm-s1-0
 * Stop    rsc_SAPHanaTop_HSO_HDB00:1   (hso-hana-vm-s1-0)
 * Demote  rsc_SAPHanaCon_HSO_HDB00:1   (Master -> Stopped hso-hana-vm-s1-0)
 * Promote rsc_SAPHanaCon_HSO_HDB00:5   (Slave -> Master hso-hana-vm-s2-0)
 * Move    rsc_ip_HSO_HDB00     (Started hso-hana-vm-s1-0 -> hso-hana-vm-s2-0)
 * Move    rsc_nc_HSO_HDB00     (Started hso-hana-vm-s1-0 -> hso-hana-vm-s2-0)
</code></pre>



## <a name="planned-maintenance"></a>預定的維修 

進入計劃性維護時，有不同的使用案例。 例如，其中一個問題是它只是基礎結構維護 (例如 OS 層級和磁碟設定的變更) 還是 HANA 升級。
您可以在 SUSE 文件中找到其他資訊，例如[這裡][sles-zero-downtime-paper]或[這裡的另一份文件][sles-12-for-sap]。 這些文件也包含如何手動移轉主要站台的範例。

密集內部測試是要驗證基礎結構維護使用案例。 若要避免任何一種移轉主要站台的相關問題，請決定一律先移轉主要站台，再讓叢集進入維護模式。 因此，不需要讓叢集忘記先前的情況 (哪一端是主要站台，哪一端是次要站台)。

在這部分有兩種不同的情況：

1. 目前次要站台上的計劃性維護。 
   在此情況下，您只能讓叢集進入維護模式，以及在次要上執行工作，而不影響叢集

2. 目前主要站台上的計劃性維護。 
   若要允許使用者在維護期間繼續工作，就必須強制容錯移轉。 使用此方式，您必須由 Pacemaker 觸發叢集容錯移轉，而不只是位於 SAP HANA HSR 層級。 Pacemaker 設定會自動觸發 SAP HANA 接管。 此外，必須先完成容錯移轉，再讓叢集進入維護模式。

目前次要站台維護程序的步驟如下：

1. 讓叢集進入維護模式
2. 完成次要站台上的工作 
3. 結束叢集維護模式

目前主要站台維護程序更為複雜：

1. 透過 Pacemaker 資源移轉，手動觸發容錯移轉/SAP HANA 接管 (請參閱下面的詳細資料)
2. 叢集設定會關閉先前主要站台上的 SAP HANA
3. 讓叢集進入維護模式
4. 維護工作完成之後，請將先前的主要站台註冊為新的次要站台
5. 清除叢集設定 (請參閱下面的詳細資料)
6. 結束叢集維護模式


移轉資源 (例如強制容錯移轉) 會在叢集設定中新增項目。 您必須先清除這些項目，再結束維護模式。 以下是範例：

第一個步驟是將 MSL 資源移轉至目前次要主要節點，來強制叢集容錯移轉。 下面的命令會提供已建立「移動條件約束」警告。

<pre><code>
crm resource migrate msl_SAPHanaCon_HSO_HDB00 force

INFO: Move constraint created for msl_SAPHanaCon_HSO_HDB00
</code></pre>


透過 **SAPHanaSR-showAttr** 命令，檢查容錯移轉程序。 對監視叢集狀態有幫助的方式是開啟專用殼層視窗，並啟動具有 **watch** 的命令：

<pre><code>
watch SAPHanaSR-showAttr
</code></pre>

輸出應該會反映手動容錯移轉。 **提升**先前次要主要節點 (在此範例中為 **hso-hana-vm-s2-0**) 並停止先前主要站台 (**lss** 值 **1** 表示先前主要的主要節點 **hso-hana-vm-s1-0**)： 

<pre><code>
Global cib-time                 prim  sec srHook sync_state
------------------------------------------------------------
global Wed Sep 12 07:40:02 2018 HSOS2 -   SFAIL  SFAIL


Sites lpt        lss mns              srr
------------------------------------------
HSOS1 10         1   hso-hana-vm-s1-0 P
HSOS2 1536738002 4   hso-hana-vm-s2-0 P


Hosts            clone_state node_state roles                        score  site
----------------------------------------------------------------------------------
hso-hana-dm                  online
hso-hana-vm-s1-0 UNDEFINED   online     master1::worker:             150    HSOS1
hso-hana-vm-s1-1 DEMOTED     online     slave::worker:               -10000 HSOS1
hso-hana-vm-s1-2 DEMOTED     online     slave::worker:               -10000 HSOS1
hso-hana-vm-s2-0 PROMOTED    online     master1:master:worker:master 150    HSOS2
hso-hana-vm-s2-1 DEMOTED     online     slave:slave:worker:slave     -10000 HSOS2
hso-hana-vm-s2-2 DEMOTED     online     slave:slave:worker:slave     -10000 HSOS2
</code></pre>

叢集容錯移轉和 SAP HANA 接管之後，請讓叢集進入維護模式，如 pacemaker 區段中所述。

**SAPHanaSR-showAttr** 或 **crm status** 命令不會指出有關資源移轉所建立條件約束的任何資訊。 顯示這些條件約束的其中一個選項是使用下列命令來顯示完整叢集資源設定：

<pre><code>
crm configure show
</code></pre>

在叢集設定內，您會找到先前手動資源移轉所導致的新位置條件約束。 以下是範例 (開頭為 **location cli-**) 的項目：

<pre><code>
location cli-ban-msl_SAPHanaCon_HSO_HDB00-on-hso-hana-vm-s1-0 msl_SAPHanaCon_HSO_HDB00 role=Started -inf: hso-hana-vm-s1-0
</code></pre>

不幸的是，這類條件約束可能會影響整體叢集行為。 因此，必須先再次移除它們，再備份整個系統。 使用 **unmigrate** 命令，可以清除先前建立的位置條件約束。 命名方式可能會讓人混淆。 這並不表示它會嘗試將資源移轉回其從中移轉的原始 VM。 它只會移除位置條件約束，並在執行命令時一併傳回對應的資訊：


<pre><code>
crm resource unmigrate msl_SAPHanaCon_HSO_HDB00

INFO: Removed migration constraints for msl_SAPHanaCon_HSO_HDB00
</code></pre>

在維護工作結束時，您會停止叢集維護模式，如 Pacemaker 區段中所述。



## <a name="hbreport-to-collect-log-files"></a>hb_report 以收集記錄檔

若要分析 Pacemaker 叢集問題，這十分有用，而且一併由 SUSE 支援人員要求執行 **hb_report** 公用程式。 它會收集所有重要記錄檔，以允許分析所發生的問題。 以下範例呼叫使用發生特定事件的開始和結束時間 (另請參閱有關重要事項的第一節)：

<pre><code>
hb_report -f "2018/09/13 07:36" -t "2018/09/13 08:00" /tmp/hb_report_log
</code></pre>

此命令會溢出放置壓縮記錄檔的位置：

<pre><code>
The report is saved in /tmp/hb_report_log.tar.bz2
Report timespan: 09/13/18 07:36:00 - 09/13/18 08:00:00
</code></pre>

您接著可以透過標準 **tar** 命令來解壓縮個別檔案：

<pre><code>
tar -xvf hb_report_log.tar.bz2
</code></pre>

查看所有記錄檔中您找到的解壓縮檔案。 其中大部分都會放入叢集中每個節點的個別目錄：

<pre><code>
-rw-r--r-- 1 root root  13655 Sep 13 09:01 analysis.txt
-rw-r--r-- 1 root root  14422 Sep 13 09:01 description.txt
-rw-r--r-- 1 root root      0 Sep 13 09:01 events.txt
-rw-r--r-- 1 root root 275560 Sep 13 09:00 ha-log.txt
-rw-r--r-- 1 root root     26 Sep 13 09:00 ha-log.txt.info
drwxr-xr-x 4 root root   4096 Sep 13 09:01 hso-hana-dm
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-0
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-1
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-2
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-0
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-1
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-2
-rw-r--r-- 1 root root 264726 Sep 13 09:00 journal.log
</code></pre>


在時間範圍內，其指定終止目前主要節點 **hso-hana-vm-s1-0**。 在 **journal.log** 中，您可以找到與此事件相關的項目：

<pre><code>
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 su[93494]: (to hsoadm) root on none
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 su[93494]: pam_unix(su-l:session): session opened for user hsoadm by (uid=0)
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 systemd[1]: Started Session c44290 of user hsoadm.
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [TOTEM ] A new membership (10.0.0.13:120996) was formed. Members left: 1
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [TOTEM ] Failed to receive the leave message. failed: 1
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Removing all hso-hana-vm-s1-0 attributes for peer loss
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 stonith-ng[28311]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 stonith-ng[28311]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 cib[28310]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [QUORUM] Members[6]: 7 2 3 4 5 6
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [MAIN  ] Completed service synchronization, ready to provide service.
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 crmd[28315]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 pacemakerd[28308]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 cib[28310]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:03+0000 hso-hana-vm-s2-1 su[93494]: pam_unix(su-l:session): session closed for user hsoadm
</code></pre>

另一個範例是次要主要節點上的 Pacemaker 記錄檔，其已成為新主要的主要節點。 以下是摘要，其顯示已終止主要的主要節點狀態設定為**離線**。

<pre><code>
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 3 still member of group stonith-ng (peer=hso-hana-vm-s1-2, counter=5.1)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 4 still member of group stonith-ng (peer=hso-hana-vm-s2-0, counter=5.2)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 5 still member of group stonith-ng (peer=hso-hana-vm-s2-1, counter=5.3)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 6 still member of group stonith-ng (peer=hso-hana-vm-s2-2, counter=5.4)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 7 still member of group stonith-ng (peer=hso-hana-dm, counter=5.5)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: pcmk_cpg_membership:      Node 1 left group crmd (peer=hso-hana-vm-s1-0, counter=5.0)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: crm_update_peer_proc:     pcmk_cpg_membership: Node hso-hana-vm-s1-0[1] - corosync-cpg is now offline
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: peer_update_callback:     Client hso-hana-vm-s1-0/peer now has status [offline] (DC=hso-hana-dm, changed=4000000)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: pcmk_cpg_membership:      Node 2 still member of group crmd (peer=hso-hana-vm-s1-1, counter=5.0)
</code></pre>





## <a name="sap-hana-globalini"></a>SAP HANA global.ini


例如，在下方，您會看到叢集站台 2 上 SAP HANA global.ini 檔案的摘要，以顯示將不同的網路用於 SAP HANA 節點間通訊和 HSR 的 hostname 解析項目：

<pre><code>
[communication]
tcp_keepalive_interval = 20
internal_network = 10.0.2/24
listeninterface = .internal
</code></pre>


<pre><code>
[internal_hostname_resolution]
10.0.2.40 = hso-hana-vm-s2-0
10.0.2.42 = hso-hana-vm-s2-2
10.0.2.41 = hso-hana-vm-s2-1
</code></pre>

<pre><code>
[ha_dr_provider_SAPHanaSR]
provider = SAPHanaSR
path = /hana/shared/myHooks
execution_order = 1
</code></pre>

<pre><code>
[system_replication_communication]
listeninterface = .internal

[system_replication_hostname_resolution]
10.0.1.30 = hso-hana-vm-s1-0
10.0.1.31 = hso-hana-vm-s1-1
10.0.1.32 = hso-hana-vm-s1-2
10.0.1.40 = hso-hana-vm-s2-0
10.0.1.41 = hso-hana-vm-s2-1
10.0.1.42 = hso-hana-vm-s2-2
</code></pre>



## <a name="hawk"></a>HAWK

與殼層層級上的所有命令相較之下，叢集解決方案也會提供瀏覽器介面，以針對偏好功能表和圖形的人員提供不錯的 GUI。
若要使用瀏覽器介面，請採用下面顯示的 URL，並將 **\<node\>** 取代為實際 SAP HANA 節點，然後輸入叢集的認證 (使用者 **hacluster**)：

<pre><code>
https://&ltnode&gt:7630
</code></pre>

下列螢幕擷取畫面顯示叢集儀表板：


![HAWK 叢集儀表板](media/hana-vm-scale-out-HA-troubleshooting/hawk-1.png)


在第二個螢幕擷取畫面上，您可以看到叢集資源移轉所導致的位置條件約束範例，如計劃性維護一節中所述：


![HAWK 列出條件約束](media/hana-vm-scale-out-HA-troubleshooting/hawk-2.png)


另一個不錯功能是讓您上傳 [歷程記錄] 下 **HAWK** 中的 **hb_report** 輸出 (請參閱 **hb_report** 一節)，如下一個螢幕擷取畫面上所示：

![HAWK 會上傳 hb_report 輸出](media/hana-vm-scale-out-HA-troubleshooting/hawk-3.png)

**History Explorer** 接著會允許執行 **hb_report** 輸出中所含的所有叢集轉換：

![HAWK 會查看 hb_report 輸出內的轉換](media/hana-vm-scale-out-HA-troubleshooting/hawk-4.png)

在最後一個螢幕擷取畫面上，您可以看到單一轉換的詳細資料區段，以顯示叢集顯示主要的主要節點損毀 (節點 **hso-hana-vm-s1-0**) 而且現在會將次要節點提升為新的主要節點 (**hso-hana-vm-s2-0**)：

![HAWK 會查看單一轉換](media/hana-vm-scale-out-HA-troubleshooting/hawk-5.png)


## <a name="next-steps"></a>後續步驟

本疑難排解指南是有關相應放大設定中的 SAP HANA 高可用性。 除了資料庫之外，SAP 橫向內的另一個重要元件是 SAP NetWeaver 堆疊。 接下來，您應該閱讀[本文][sap-nw-ha-guide-sles]中在 Azure 虛擬機器上使用 SUSE Enterprise Linux Server 的 SAP NetWeaver 高可用性。

