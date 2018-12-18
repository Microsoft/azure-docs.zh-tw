---
title: Red Hat Enterprise Linux for SAP NetWeaver 上適用於 Azure VM 的 GlusterFS | Microsoft Docs
description: Red Hat Enterprise Linux for SAP NetWeaver 上適用於 Azure VM 的 GlusterFS
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: mssedusch
manager: timlt
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
ms.openlocfilehash: bbb0bbc6ddbbabb37932edc6946394b020631231
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45636424"
---
# <a name="glusterfs-on-azure-vms-on-red-hat-enterprise-linux-for-sap-netweaver"></a>Red Hat Enterprise Linux for SAP NetWeaver 上適用於 Azure VM 的 GlusterFS

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1999351]:https://launchpad.support.sap.com/#/notes/1999351

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability-rhel.md

本文說明如何部署虛擬機器、設定虛擬機器，以及安裝可用來儲存高可用性 SAP 系統之共用資料的 GlusterFS 叢集。
本指南說明如何設定供兩個 SAP 系統 (NW1 和 NW2) 使用的 GlusterFS。 範例中資源 (例如虛擬機器、虛擬網路) 的名稱是假設您已使用資源前置詞為 **glust** 的 [SAP 檔案伺服器範本][template-file-server]。

請先閱讀下列 SAP Note 和文件

* SAP Note [1928533]，其中包含：
  * SAP 軟體部署支援的 Azure VM 大小清單
  * Azure VM 大小的重要容量資訊
  * 支援的 SAP 軟體，以及作業系統 (OS) 與資料庫組合
  * Microsoft Azure 上 Windows 和 Linux 所需的 SAP 核心版本

* SAP Note [2015553] 列出 Azure 中 SAP 支援的 SAP 軟體部署先決條件。
* SAP Note [2002167] 建議適用於 Red Hat Enterprise Linux 的作業系統設定
* SAP Note [2009879] 提供適用於 Red Hat Enterprise Linux 的 SAP Hana 指導方針
* SAP Note [2178632] 包含在 Azure 中針對 SAP 回報的所有監視計量詳細資訊。
* SAP Note [2191498] 包含 Azure 中 Linux 所需的 SAP Host Agent 版本。
* SAP Note [2243692] 包含 Azure 中 Linux 上的 SAP 授權相關資訊。
* SAP Note [1999351] 包含 Azure Enhanced Monitoring Extension for SAP 的其他疑難排解資訊。
* [SAP Community WIKI](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) 包含 Linux 所需的所有 SAP Note。
* [適用於 SAP on Linux 的 Azure 虛擬機器規劃和實作][planning-guide]
* [適用於 SAP on Linux 的 Azure 虛擬機器部署 (本文)][deployment-guide]
* [適用於 SAP on Linux 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* [Red Hat Gluster Storage 產品文件](https://access.redhat.com/documentation/red_hat_gluster_storage/)
* 一般 RHEL 文件
  * [高可用性附加元件概觀](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [高可用性附加元件管理](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [高可用性附加元件參考](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
* Azure 專用 RHEL 文件：
  * [RHEL 高可用性叢集的支援原則 - Microsoft Azure 虛擬機器做為叢集成員](https://access.redhat.com/articles/3131341)
  * [在 Microsoft Azure 上安裝和設定 Red Hat Enterprise Linux 7.4 (和更新版本) 高可用性叢集](https://access.redhat.com/articles/3252491)

## <a name="overview"></a>概觀

為了實現高可用性，SAP NetWeaver 需要使用共用儲存體。 GlusterFS 會於獨立的叢集中設定，且可供多個 SAP 系統使用。

![SAP NetWeaver 高可用性概觀](./media/high-availability-guide-rhel-glusterfs/rhel-glusterfs.png)

## <a name="set-up-glusterfs"></a>設定 GlusterFS

您可以使用來自 GitHub 的 Azure 範本來部署所有必要的 Azure 資源，包括虛擬機器、可用性設定組及網路介面，也可以手動部署資源。

### <a name="deploy-linux-via-azure-template"></a>透過 Azure 範本部署 Linux

Azure Marketplace 包含 Red Hat Enterprise Linux 的映像，您可用來部署新的虛擬機器。
您可以使用 Github 上的其中一個快速入門範本來部署所有必要資源。 範本會部署虛擬機器、可用性設定組等項目。請遵循下列步驟來部署範本：

1. 在 Azure 入口網站中開啟 [SAP 檔案伺服器範本][template-file-server]
1. 輸入下列參數
   1. 資源前置詞  
      輸入您想要使用的前置詞。 該值會作為所部署之資源的前置詞。
   2. SAP 系統計數 - 輸入將使用此檔案伺服器的 SAP 系統數目。 這麼做將會部署所需數量的磁碟等項目。
   3. OS 類型  
      選取一個 Linux 發行版本。 在此範例中，請選取 RHEL 7
   4. 管理員使用者名稱、管理員密碼或 SSH 金鑰  
      建立可用來登入電腦的新使用者。
   5. 子網路識別碼  
      如果您想要將 VM 部署至現有的 VNet 中，而 VNet 中已定義應指派 VM 的子網路，請提供該特定子網路的識別碼。 識別碼通常如下所示：/subscriptions/**&lt;訂用帳戶識別碼&gt;**/resourceGroups/**&lt;資源群組名稱&gt;**/providers/Microsoft.Network/virtualNetworks/**&lt;虛擬網路名稱&gt;**/subnets/**&lt;子網路名稱&gt;**

### <a name="deploy-linux-manually-via-azure-portal"></a>透過 Azure 入口網站手動部署 Linux

您必須先為此叢集建立虛擬機器。 之後，您需建立負載平衡器，然後使用後端集區中的虛擬機器。

1. 建立資源群組
1. 建立虛擬網路
1. 建立可用性設定組  
   設定更新網域上限
1. 建立虛擬機器 1  
   至少使用 RHEL 7，本範例中使用 Red Hat Enterprise Linux 7.4 映像 <https://ms.portal.azure.com/#create/RedHat.RedHatEnterpriseLinux74-ARM>  
   選取稍早建立的「可用性設定組」  
1. 建立虛擬機器 2  
   至少使用 RHEL 7，本範例中使用 Red Hat Enterprise Linux 7.4 映像 <https://ms.portal.azure.com/#create/RedHat.RedHatEnterpriseLinux74-ARM>  
   選取稍早建立的「可用性設定組」  
1. 在兩部虛擬機器上為每個 SAP 系統各新增一個資料磁碟。

### <a name="configure-glusterfs"></a>設定 GlusterFS

下列項目會加上以下其中一個前置詞：**[A]** - 適用於所有節點、**[1]** - 僅適用於節點 1、**[2]** - 僅適用於節點 2、**[3]** - 僅適用於節點 3。

1. **[A]** 設定主機名稱解析

   您可以使用 DNS 伺服器，或修改所有節點上的 /etc/hosts。 這個範例示範如何使用 /etc/hosts 檔案。
   取代下列命令中的 IP 位址和主機名稱

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   將下列幾行插入至 /etc/hosts。 變更 IP 位址和主機名稱以符合您的環境

   <pre><code># IP addresses of the Gluster nodes
   <b>10.0.0.40 glust-0</b>
   <b>10.0.0.41 glust-1</b>
   <b>10.0.0.42 glust-2</b>
   </code></pre>

1. **[A]** 註冊

   註冊您的虛擬機器，並將它附加至集區，集區中需包含適用於 RHEL 7 和 GlusterFS 的存放庫

   <pre><code>sudo subscription-manager register
   sudo subscription-manager attach --pool=&lt;pool id&gt;
   </code></pre>

1. **[A]** 啟用 GlusterFS 存放庫

   若要安裝必要的套件，請啟用下列存放庫。

   <pre><code>sudo subscription-manager repos --disable "*"
   sudo subscription-manager repos --enable=rhel-7-server-rpms
   sudo subscription-manager repos --enable=rh-gluster-3-for-rhel-7-server-rpms
   </code></pre>
  
1. **[A]** 安裝 GlusterFS 封裝

   在所有 GlusterFS 節點上安裝這些封裝

   <pre><code>sudo yum -y install redhat-storage-server
   </code></pre>

   安裝後重新啟動節點。

1. **[A]** 修改防火牆

   新增防火牆規則以允許用戶端流量傳送至 GlusterFS 節點。

   <pre><code># list the available zones
   firewall-cmd --get-active-zones
   
   sudo firewall-cmd --zone=public --add-service=glusterfs --permanent
   sudo firewall-cmd --zone=public --add-service=glusterfs
   </code></pre>

1. **[A]** 啟用並啟動 GlusterFS 服務

   在所有節點上啟動 GlusterFS 服務。

   <pre><code>sudo systemctl start glusterd
   sudo systemctl enable glusterd
   </code></pre>

1. **[1]** 建立 GluserFS

   執行下列命令以建立 GlusterFS 叢集

   <pre><code>sudo gluster peer probe glust-1
   sudo gluster peer probe glust-2
   
   # Check gluster peer status
   sudo gluster peer status
   
   # Number of Peers: 2
   # 
   # Hostname: glust-1
   # Uuid: 10d43840-fee4-4120-bf5a-de9c393964cd
   # State: Accepted peer request (Connected)
   # 
   # Hostname: glust-2
   # Uuid: 9e340385-12fe-495e-ab0f-4f851b588cba
   # State: Accepted peer request (Connected)
   </code></pre>

1. **[2]** 測試對等狀態

   測試第二個節點上的對等狀態

   <pre><code>sudo gluster peer status
   # Number of Peers: 2
   #
   # Hostname: glust-0
   # Uuid: 6bc6927b-7ee2-461b-ad04-da123124d6bd
   # State: Peer in Cluster (Connected)
   #
   # Hostname: glust-2
   # Uuid: 9e340385-12fe-495e-ab0f-4f851b588cba
   # State: Peer in Cluster (Connected)
   </code></pre>

1. **[3]** 測試對等狀態

   測試第三個節點上的對等狀態

   <pre><code>sudo gluster peer status
   # Number of Peers: 2
   #
   # Hostname: glust-0
   # Uuid: 6bc6927b-7ee2-461b-ad04-da123124d6bd
   # State: Peer in Cluster (Connected)
   #
   # Hostname: glust-1
   # Uuid: 10d43840-fee4-4120-bf5a-de9c393964cd
   # State: Peer in Cluster (Connected)
   </code></pre>

1. **[A]** 建立 LVM

   在此範例中，GlusterFS 用於兩個 SAP 系統 (NW1 和 NW2)。 使用下列命令來為這兩個 SAP 系統建立 LVM 設定。

   以下命令用於 NW1

   <pre><code>sudo pvcreate --dataalignment 1024K /dev/disk/azure/scsi1/lun0
   sudo pvscan
   sudo vgcreate --physicalextentsize 256K rhgs-<b>NW1</b> /dev/disk/azure/scsi1/lun0
   sudo vgscan
   sudo lvcreate -l 50%FREE -n rhgs-<b>NW1</b>/sapmnt
   sudo lvcreate -l 20%FREE -n rhgs-<b>NW1</b>/trans
   sudo lvcreate -l 10%FREE -n rhgs-<b>NW1</b>/sys
   sudo lvcreate -l 50%FREE -n rhgs-<b>NW1</b>/ascs
   sudo lvcreate -l 100%FREE -n rhgs-<b>NW1</b>/aers
   sudo lvscan
   
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/sapmnt
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/trans
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/sys
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/ascs
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW1</b>/aers
   
   sudo mkdir -p /rhs/<b>NW1</b>/sapmnt
   sudo mkdir -p /rhs/<b>NW1</b>/trans
   sudo mkdir -p /rhs/<b>NW1</b>/sys
   sudo mkdir -p /rhs/<b>NW1</b>/ascs
   sudo mkdir -p /rhs/<b>NW1</b>/aers
   
   sudo chattr +i /rhs/<b>NW1</b>/sapmnt
   sudo chattr +i /rhs/<b>NW1</b>/trans
   sudo chattr +i /rhs/<b>NW1</b>/sys
   sudo chattr +i /rhs/<b>NW1</b>/ascs
   sudo chattr +i /rhs/<b>NW1</b>/aers

   echo -e "/dev/rhgs-<b>NW1</b>/sapmnt\t/rhs/<b>NW1</b>/sapmnt\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW1</b>/trans\t/rhs/<b>NW1</b>/trans\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW1</b>/sys\t/rhs/<b>NW1</b>/sys\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW1</b>/ascs\t/rhs/<b>NW1</b>/ascs\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW1</b>/aers\t/rhs/<b>NW1</b>/aers\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   sudo mount -a
   </code></pre>

   以下命令用於 NW2

   <pre><code>sudo pvcreate --dataalignment 1024K /dev/disk/azure/scsi1/lun1
   sudo pvscan
   sudo vgcreate --physicalextentsize 256K rhgs-<b>NW2</b> /dev/disk/azure/scsi1/lun1
   sudo vgscan
   sudo lvcreate -l 50%FREE -n rhgs-<b>NW2</b>/sapmnt
   sudo lvcreate -l 20%FREE -n rhgs-<b>NW2</b>/trans
   sudo lvcreate -l 10%FREE -n rhgs-<b>NW2</b>/sys
   sudo lvcreate -l 50%FREE -n rhgs-<b>NW2</b>/ascs
   sudo lvcreate -l 100%FREE -n rhgs-<b>NW2</b>/aers
   
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/sapmnt
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/trans
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/sys
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/ascs
   sudo mkfs.xfs -f -K -i size=512 -n size=8192 /dev/rhgs-<b>NW2</b>/aers
   
   sudo mkdir -p /rhs/<b>NW2</b>/sapmnt
   sudo mkdir -p /rhs/<b>NW2</b>/trans
   sudo mkdir -p /rhs/<b>NW2</b>/sys
   sudo mkdir -p /rhs/<b>NW2</b>/ascs
   sudo mkdir -p /rhs/<b>NW2</b>/aers
   
   sudo chattr +i /rhs/<b>NW2</b>/sapmnt
   sudo chattr +i /rhs/<b>NW2</b>/trans
   sudo chattr +i /rhs/<b>NW2</b>/sys
   sudo chattr +i /rhs/<b>NW2</b>/ascs
   sudo chattr +i /rhs/<b>NW2</b>/aers
   sudo lvscan
   
   echo -e "/dev/rhgs-<b>NW2</b>/sapmnt\t/rhs/<b>NW2</b>/sapmnt\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW2</b>/trans\t/rhs/<b>NW2</b>/trans\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW2</b>/sys\t/rhs/<b>NW2</b>/sys\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW2</b>/ascs\t/rhs/<b>NW2</b>/ascs\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   echo -e "/dev/rhgs-<b>NW2</b>/aers\t/rhs/<b>NW2</b>/aers\txfs\tdefaults,inode64,nobarrier,noatime,nouuid 0 2" | sudo tee -a /etc/fstab
   sudo mount -a
   </code></pre>

1. **[1]** 建立分散式磁碟區

   使用下列命令來為 NW1 建立 GlusterFS 磁碟區並啟動它。

   <pre><code>sudo gluster vol create <b>NW1</b>-sapmnt replica 3 glust-0:/rhs/<b>NW1</b>/sapmnt glust-1:/rhs/<b>NW1</b>/sapmnt glust-2:/rhs/<b>NW1</b>/sapmnt force
   sudo gluster vol create <b>NW1</b>-trans replica 3 glust-0:/rhs/<b>NW1</b>/trans glust-1:/rhs/<b>NW1</b>/trans glust-2:/rhs/<b>NW1</b>/trans force
   sudo gluster vol create <b>NW1</b>-sys replica 3 glust-0:/rhs/<b>NW1</b>/sys glust-1:/rhs/<b>NW1</b>/sys glust-2:/rhs/<b>NW1</b>/sys force
   sudo gluster vol create <b>NW1</b>-ascs replica 3 glust-0:/rhs/<b>NW1</b>/ascs glust-1:/rhs/<b>NW1</b>/ascs glust-2:/rhs/<b>NW1</b>/ascs force
   sudo gluster vol create <b>NW1</b>-aers replica 3 glust-0:/rhs/<b>NW1</b>/aers glust-1:/rhs/<b>NW1</b>/aers glust-2:/rhs/<b>NW1</b>/aers force
   
   sudo gluster volume start <b>NW1</b>-sapmnt
   sudo gluster volume start <b>NW1</b>-trans
   sudo gluster volume start <b>NW1</b>-sys
   sudo gluster volume start <b>NW1</b>-ascs
   sudo gluster volume start <b>NW1</b>-aers
   </code></pre>

   使用下列命令來為 NW2 建立 GlusterFS 磁碟區並啟動它。

   <pre><code>sudo gluster vol create <b>NW2</b>-sapmnt replica 3 glust-0:/rhs/<b>NW2</b>/sapmnt glust-1:/rhs/<b>NW2</b>/sapmnt glust-2:/rhs/<b>NW2</b>/sapmnt force
   sudo gluster vol create <b>NW2</b>-trans replica 3 glust-0:/rhs/<b>NW2</b>/trans glust-1:/rhs/<b>NW2</b>/trans glust-2:/rhs/<b>NW2</b>/trans force
   sudo gluster vol create <b>NW2</b>-sys replica 3 glust-0:/rhs/<b>NW2</b>/sys glust-1:/rhs/<b>NW2</b>/sys glust-2:/rhs/<b>NW2</b>/sys force
   sudo gluster vol create <b>NW2</b>-ascs replica 3 glust-0:/rhs/<b>NW2</b>/ascs glust-1:/rhs/<b>NW2</b>/ascs glust-2:/rhs/<b>NW2</b>/ascs force
   sudo gluster vol create <b>NW2</b>-aers replica 3 glust-0:/rhs/<b>NW2</b>/aers glust-1:/rhs/<b>NW2</b>/aers glust-2:/rhs/<b>NW2</b>/aers force
   
   sudo gluster volume start <b>NW2</b>-sapmnt
   sudo gluster volume start <b>NW2</b>-trans
   sudo gluster volume start <b>NW2</b>-sys
   sudo gluster volume start <b>NW2</b>-ascs
   sudo gluster volume start <b>NW2</b>-aers
   </code></pre>

## <a name="next-steps"></a>後續步驟

* [安裝 SAP ASCS 和資料庫](high-availability-guide-rhel.md)
* [適用於 SAP 的 Azure 虛擬機器規劃和實作][planning-guide]
* [適用於 SAP 的 Azure 虛擬機器部署][deployment-guide]
* [適用於 SAP 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* 若要了解如何建立高可用性並為 Azure 上的 SAP HANA 規劃災害復原，請參閱 [Azure 上的 SAP HANA (大型執行個體) 高可用性和災害復原](hana-overview-high-availability-disaster-recovery.md)。
* 若要了解如何建立高可用性並為 Azure VM 上的 SAP HANA 規劃災害復原，請參閱 [Azure 虛擬機器 (VM) 上 SAP HANA 的高可用性][sap-hana-ha]
