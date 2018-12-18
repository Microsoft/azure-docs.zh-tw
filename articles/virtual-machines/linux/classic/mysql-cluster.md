---
title: 使用負載平衡集將 MySQL 叢集化 | Microsoft Docs
description: 在 Azure 上安裝以傳統部署模型建立的負載平衡、高可用性 Linux MySQL 叢集
services: virtual-machines-linux
documentationcenter: ''
author: bureado
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: 6c413a16-e9b5-4ffe-a8a3-ae67046bbdf3
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 04/14/2015
ms.author: jparrel
ms.openlocfilehash: e2671def47879e3d4eae000c9084cd458e29b933
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "30841259"
---
# <a name="use-load-balanced-sets-to-clusterize-mysql-on-linux"></a>在 Linux 上使用負載平衡集合將 MySQL 叢集化
> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有兩種：[Azure Resource Manager](../../../resource-manager-deployment-model.md) 和傳統。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如果您需要部署 MySQL 叢集，則可取得 [Resource Manager 範本](https://azure.microsoft.com/documentation/templates/mysql-replication/)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

本文探索與說明要在 Microsoft Azure 上部署高度可用 Linux 架構服務的其他可用方法，首先從 MySQL Server 高可用性開始。 您可在 [第 9 頻道](http://channel9.msdn.com/Blogs/Open/Load-balancing-highly-available-Linux-services-on-Windows-Azure-OpenLDAP-and-MySQL)(英文) 上找到說明此方法的影片。

我們會依據 DRBD、Corosync 和 Pacemaker，說明無共用、兩個節點、單一主要的 MySQL 高可用性解決方案。 一次只會有一個節點執行 MySQL。 也限制一次只會有一個節點從 DRBD 資源讀取和寫入。

因為您將使用 Microsoft Azure 中的負載平衡集合，來提供循環配置資源功能和 VIP 的端點偵測、移除及正常復原，所以不需要 VIP 解決方案 (如 LVS)。 VIP 是一個可全域路由的 IPv4 位址，會在您第一次建立雲端服務時由 Microsoft Azure 指定。

還有其他可能架構可供 MySQL 使用，包括 NBD 叢集、Percona、Galera 以及數種中繼軟體解決方案 (包括至少有一個可作為 [VM Depot](http://vmdepot.msopentech.com) 上的 VM)。 只要這些解決方案可以在單點傳送與多點傳送上複寫或廣播，且不仰賴共用儲存體或多個網路介面，那麼應不難在 Microsoft Azure 上部署這些案例。

您可以將這些叢集架構以類似的方式延伸到其他產品，如 PostgreSQL 和 OpenLDAP。 例如，已透過多個主要 OpenLDAP 成功測試使用無共用的負載平衡程序，您可以在我們的第 9 頻道 (Channel 9) 部落格上觀看此測試。

## <a name="get-ready"></a>準備就緒
您需要下列資源和功能：

  - 具備有效訂用帳戶的 Microsoft Azure 帳戶，能夠建立至少兩個 VM (在此範例中使用 XS)
  - 網路和子網路
  - 同質群組
  - 可用性設定組
  - 能夠在與雲端服務相同的區域中建立 VHD，並將它們連接至 Linux VM

### <a name="tested-environment"></a>測試環境
* Ubuntu 13.10
  * DRBD
  * MySQL Server
  * Corosync 和 Pacemaker

### <a name="affinity-group"></a>同質群組
建立解決方案的同質群組，方法是登入 Azure 入口網站，選取 [設定]，然後建立同質群組。 並將稍後建立的配置資源指派給此同質群組。

### <a name="networks"></a>網路
建立新網路，並在該網路內建立一個子網路。 此範例使用內部只有一個 /24 子網路的 10.10.10.0/24 網路。

### <a name="virtual-machines"></a>虛擬機器
使用經過背書的 Ubuntu 映像庫映像來建立第一個 Ubuntu 13.10 VM，並將它命名為 `hadb01`。 在此程序中會建立一個名為 hadb 的新雲端服務。 此名稱可說明新增更多資源時，服務將具備的共用、負載平衡性質。 已使用入口網站順利完成建立 `hadb01`。 系統會自動建立 SSH 的端點，並選取新的網路。 您可以為 VM 建立可用性設定組。

建立第一個 VM 後 (從技術角度來看，也就是建立雲端服務時)，我們會建立第二個 VM `hadb02`。 在第二個 VM 中，透過入口網站使用資源庫中的 Ubuntu 13.10 VM，但使用現有的雲端服務 `hadb.cloudapp.net`，而非建立新的雲端服務。 系統應會自動選取網路和可用性設定組。 並建立 SSH 端點。

在建立這兩部 VM 之後，請記下 `hadb01` 的 SSH 連接埠 (TCP 22) 及 `hadb02` (由 Azure 自動指派)。

### <a name="attached-storage"></a>連接的儲存體
將新磁碟連接至這兩部 VM，並在此程序中建立 5 GB 的磁碟。 這些磁碟將在 VHD 容器中託管，並供我們的主要作業系統磁碟使用。 在建立並連接磁碟後，我們不需要重新啟動 Linux，因為核心將會看到新裝置。 此裝置通常是 `/dev/sdc`。 檢查輸出的 `dmesg`。

在每個 VM 上，使用 `cfdisk` 建立磁碟分割 (主要是 Linux 磁碟分割)，並寫入新的磁碟分割資料表。 請勿在此磁碟分割上建立檔案系統。

## <a name="set-up-the-cluster"></a>設定叢集
在這兩部 Ubuntu VM 上使用 APT 來安裝 Corosync、Pacemaker 和 DRBD。 若要利用 `apt-get` 這麼做，請執行下列程式碼：

    sudo apt-get install corosync pacemaker drbd8-utils.

此時請勿安裝 MySQL。 Debian 和 Ubuntu 安裝指令碼將會初始化 `/var/lib/mysql` 上的 MySQL 資料目錄，但因為 DRBD 檔案系統會取代此目錄，所以您必須稍後安裝 MySQL。

驗證 (使用 `/sbin/ifconfig`) 這兩部 VM 是否使用 10.10.10.0/24 子網路中的位址，以及它們是否可以根據名稱互相 Ping 到對方。 您也可以使用 `ssh-keygen` 和 `ssh-copy-id`，來確定這兩部 VM 無需密碼即可透過 SSH 彼此通訊。

### <a name="set-up-drbd"></a>設定 DRBD
建立使用基礎 `/dev/sdc1` 磁碟分割的 DRBD 資源，以產生能夠使用 ext3 進行格式化的 `/dev/drbd1` 資源，以供主要和次要節點使用。

1. 開啟 `/etc/drbd.d/r0.res` 並複製兩部 VM 上的下列資源定義：

        resource r0 {
          on `hadb01` {
            device  /dev/drbd1;
            disk   /dev/sdc1;
            address  10.10.10.4:7789;
            meta-disk internal;
          }
          on `hadb02` {
            device  /dev/drbd1;
            disk   /dev/sdc1;
            address  10.10.10.5:7789;
            meta-disk internal;
          }
        }

2. 在這兩部 VM 上使用 `drbdadm` 來初始化資源：

        sudo drbdadm -c /etc/drbd.conf role r0
        sudo drbdadm up r0

3. 在主要 VM (`hadb01`) 上，強制執行 DRBD 資源的擁有權 (主要)：

        sudo drbdadm primary --force r0

如果您在這兩個 VM 上檢查 /proc/drbd (`sudo cat /proc/drbd`) 的內容，應可在 `hadb01` 上看到 `Primary/Secondary` 且在 `hadb02` 上看到 `Secondary/Primary` (此時應與解決方案一致)。 客戶可免費透過 10.10.10.0/24 網路同步處理這個 5 GB 磁碟。

同步處理此磁碟後，您可以在 `hadb01` 上建立檔案系統。 基於測試目的，我們會使用 ext2，但下列程式碼將建立 ext3 檔案系統：

    mkfs.ext3 /dev/drbd1

### <a name="mount-the-drbd-resource"></a>掛接 DRBD 資源
您現在即可在 `hadb01` 上掛接 DRBD 資源。 Debian 及其衍生版本會使用 `/var/lib/mysql` 做為 MySQL 的資料目錄。 因為您尚未安裝 MySQL，請建立此目錄並掛接 DRBD 資源。 若要執行此選項，請在 `hadb01` 上執行下列程式碼：

    sudo mkdir /var/lib/mysql
    sudo mount /dev/drbd1 /var/lib/mysql

## <a name="set-up-mysql"></a>設定 MySQL
現在您可以開始在 `hadb01`上安裝 MySQL：

    sudo apt-get install mysql-server

在 `hadb02`中，您有兩個選擇。 您可以安裝 mysql-server，這將會建立 /var/lib/mysql，並在其中填入新的資料目錄，然後移除內容。 若要執行此選項，請在 `hadb02` 上執行下列程式碼：

    sudo apt-get install mysql-server
    sudo service mysql stop
    sudo rm –rf /var/lib/mysql/*

第二個選擇是容錯移轉至 `hadb02`，然後在那裡安裝 mysql-server。 安裝指令碼將會注意到現有安裝，且不會對它執行任何動作。

在 `hadb01` 上執行下列程式碼：

    sudo drbdadm secondary –force r0

在 `hadb02` 上執行下列程式碼：

    sudo drbdadm primary –force r0
    sudo apt-get install mysql-server

如果您不打算現在容錯移轉 DRBD，雖然第一個選擇可以說是較為繁瑣，但相對是比較容易的。 安裝完成後，您可以開始使用 MySQL 資料庫。 在 `hadb02` (或對 DRBD 而言為作用中伺服器的任一部伺服器) 上執行下列程式碼：

    mysql –u root –p
    CREATE DATABASE azureha;
    CREATE TABLE things ( id SERIAL, name VARCHAR(255) );
    INSERT INTO things VALUES (1, "Yet another entity");
    GRANT ALL ON things.\* TO root;

> [!WARNING]
> 最後一個陳述式會有效地停用此資料表中根使用者的驗證。 這應由您的生產等級 GRANT 陳述式取代，這裡僅是為了說明的目的才包括此陳述式。

如果您想要在 VM 外進行查詢 (這是本指南的目的)，您還必須啟用 MySQL 的網路功能。 在兩部 VM 上，開啟 `/etc/mysql/my.cnf` 並移至 `bind-address`。 將位址從 127.0.0.1 變更為 0.0.0.0。 儲存檔案後，在您目前的主要 VM 上發佈 `sudo service mysql restart` 。

### <a name="create-the-mysql-load-balanced-set"></a>建立 MySQL 負載平衡集
回到入口網站，移至 `hadb01`，然後選擇 [端點]。 若要建立端點，從下拉式清單選擇 [MySQL (TCP 3306)]，然後選取 [建立新的負載平衡集]。 替負載平衡端點 `lb-mysql` 命名。 將 [時間] 設定為 5 秒 (最小值)。

建立端點後，請移至 `hadb02`，選擇 [端點]，然後建立端點。 選擇 `lb-mysql`，然後從下拉式清單選取 MySQL。 在此步驟中，您也可以使用 Azure CLI。

您現在已具備手動叢集作業所需的一切。

### <a name="test-the-load-balanced-set"></a>設定負載平衡集
您可以使用任何 MySQL 用戶端，或使用特定應用程式 (例如以 Azure 網站身分執行的 phpMyAdmin)，從機器外部執行測試。 在此案例中，您在另一個 Linux 方塊上使用 MySQL 的命令列工具：

    mysql azureha –u root –h hadb.cloudapp.net –e "select * from things;"

### <a name="manually-failing-over"></a>手動容錯移轉
您可以透過關閉 MySQL、切換 DRBD 的主要 VM，然後重新啟動 MySQL 來模擬容錯移轉。

若要執行此工作，請在 hadb01 上執行下列程式碼：

    service mysql stop && umount /var/lib/mysql ; drbdadm secondary r0

接著，在 hadb02 上：

    drbdadm primary r0 ; mount /dev/drbd1 /var/lib/mysql && service mysql start

手動容錯移轉後，您可以重複遠端查詢，它應可正常運作。

## <a name="set-up-corosync"></a>設定 Corosync
Corosync 是需要 Pacemaker 才能運作的基礎叢集基礎結構。 若是 Heartbeat (及其他方法，如 Ultramonkey)，Corosync 是 CRM 功能的分割，而 Pacemaker 會維持類似 Hearbeat 的功能。

Corosync 在 Azure 上的主要限制是，Corosync 偏好的通訊順序為多點傳送大於廣播大於單點傳送，但 Microsoft Azure 網路僅支援單點傳送。

還好，Corosync 具備有效的單點傳送模式。 唯一的真正限制是，因為所有節點彼此之間不會神奇地自動 進行通訊，您必須在組態檔中定義節點，包括其 IP 位址。 我們可以使用 Corosync 範例檔案進行單點傳送和變更繫結位址、節點清單和記錄目錄 (Ubuntu 會使用 `/var/log/corosync`，而範例檔案會使用 `/var/log/cluster`)，以及啟用仲裁工具。

> [!NOTE]
> 對兩個節點使用下列 `transport: udpu` 指示詞和手動定義的 IP 位址。

在 `/etc/corosync/corosync.conf` 上對兩個節點執行下列程式碼：

    totem {
      version: 2
      crypto_cipher: none
      crypto_hash: none
      interface {
        ringnumber: 0
        bindnetaddr: 10.10.10.0
        mcastport: 5405
        ttl: 1
      }
      transport: udpu
    }

    logging {
      fileline: off
      to_logfile: yes
      to_syslog: yes
      logfile: /var/log/corosync/corosync.log
      debug: off
      timestamp: on
      logger_subsys {
        subsys: QUORUM
        debug: off
        }
      }

    nodelist {
      node {
        ring0_addr: 10.10.10.4
        nodeid: 1
      }

      node {
        ring0_addr: 10.10.10.5
        nodeid: 2
      }
    }

    quorum {
      provider: corosync_votequorum
    }

複製兩部 VM 上的這個組態檔，並在這兩個節點中啟動 Corosync：

    sudo service start corosync

在啟動服務不久後，系統應會在目前通道中建立叢集，並組成仲裁。 我們可以透過檢閱記錄檔或執行下列程式碼來檢查此功能：

    sudo corosync-quorumtool –l

您將看到類似下圖的輸出：

![corosync-quorumtool -l sample output](./media/mysql-cluster/image001.png)

## <a name="set-up-pacemaker"></a>設定 Pacemaker
Pacemaker 會使用叢集來監視資源，定義在主要 VM 故障時將這些資源切換到次要 VM。 在所有其他選擇中，您可從一組可用指令碼或從 LSB (如 init) 指令碼來定義資源。

我們要 Pacemaker「擁有」DRBD 資源、掛接點及 MySQL 服務。 如果 Pacemaker 可以在主要 VM 發生問題時，依正確順序來開啟與關閉 DRBD、掛接/取消掛接 DRBD，然後啟動和停止 MySQL，則安裝便已完成。

首次安裝 Pacemaker 時，您的組態應十分簡單，如下所示：

    node $id="1" hadb01
      attributes standby="off"
    node $id="2" hadb02
      attributes standby="off"

1. 執行 `sudo crm configure show` 以檢查組態。
2. 然後使用下列資源來建立檔案 (如 `/tmp/cluster.conf`)：

        primitive drbd_mysql ocf:linbit:drbd \
              params drbd_resource="r0" \
              op monitor interval="29s" role="Master" \
              op monitor interval="31s" role="Slave"

        ms ms_drbd_mysql drbd_mysql \
              meta master-max="1" master-node-max="1" \
                clone-max="2" clone-node-max="1" \
                notify="true"

        primitive fs_mysql ocf:heartbeat:Filesystem \
              params device="/dev/drbd/by-res/r0" \
              directory="/var/lib/mysql" fstype="ext3"

        primitive mysqld lsb:mysql

        group mysql fs_mysql mysqld

        colocation mysql_on_drbd \
               inf: mysql ms_drbd_mysql:Master

        order mysql_after_drbd \
               inf: ms_drbd_mysql:promote mysql:start

        property stonith-enabled=false

        property no-quorum-policy=ignore

3. 將檔案載入組態中。 您只需要在一個節點做一次這個動作。

        sudo crm configure
          load update /tmp/cluster.conf
          commit
          exit

4. 請確定開機時會啟動兩個節點的 Pacemaker：

        sudo update-rc.d pacemaker defaults

5. 使用 `sudo crm_mon –L`，驗證其中一個節點已成為叢集的主要節點，並且正在執行所有資源。 您可以使用掛接和 PS 來檢查資源是否正在執行。

下列螢幕擷取畫面顯示 `crm_mon`，且其中一個節點已停止 (選取 Ctrl+C 以結束)：

![crm_mon node stopped](./media/mysql-cluster/image002.png)

此快照說明兩個節點 (一個主要和一個從屬)：

![crm_mon operational master/slave](./media/mysql-cluster/image003.png)

## <a name="testing"></a>測試
您已準備好開始自動容錯移轉模擬。 作法有二：軟式和硬式。

軟式方法使用叢集的關機功能：``crm_standby -U `uname -n` -v on``。 如果您在主要 VM 上使用此選項，從屬 VM 便會取代主要 VM。 記得將此選項設回 off。 如果不這麼做，crm_mon 會顯示一個待命中的節點。

硬式方法是透過入口網站將主要 VM (hadb01) 關機，或變更 VM 上的執行層級 (例如，終止、關機)。 這會藉由發出主要 VM 出現故障的訊號來協助 Corosync 和 Pacemaker。 您可以測試這個方法 (在維護期間非常有用)，但您也可以透過凍結 VM 來強制執行此案例。

## <a name="stonith"></a>STONITH
它應該能透過 Azure CLI 發出 VM 關機命令，來代替可控制實體裝置的 STONITH 指令碼。 您可以使用 `/usr/lib/stonith/plugins/external/ssh` 做為基礎，並在叢集的組態中啟用 STONITH。 系統應會全域安裝 Azure CLI，並載入叢集使用者的發佈設定和設定檔。

您可在 [GitHub](https://github.com/bureado/aztonith) 上找到資源的範例程式碼。 變更叢集的組態，方法是將下列程式碼新增至 `sudo crm configure`：

    primitive st-azure stonith:external/azure \
      params hostlist="hadb01 hadb02" \
      clone fencing st-azure \
      property stonith-enabled=true \
      commit

> [!NOTE]
> 此指令碼不會執行使用/停用檢查。 原始的 SSH 資源會有 15 個 Ping 檢查，但 Azure VM 的復原時間可能會有較多的變數。

## <a name="limitations"></a>限制
套用下列限制：

* 在 Pacemaker 中，將 DRBD 當成資源進行管理的 Linbit DRBD 資源指令碼會在關閉節點時使用 `drbdadm down`，即使該節點只是進入待命中狀態也一樣。 因為主要 VM 在開始遭到寫入時，從屬 VM 並不會同步處理 DRBD 資源，所以這不是理想的作法。 如果主要 VM 不是順利關機，從屬 VM 會接收較舊的檔案系統狀態。 可能解決此問題的方法有兩種：
  * 在所有叢集節點中透過本機 (未叢集化) 看門狗強制執行 `drbdadm up r0`
  * 在 `/usr/lib/ocf/resource.d/linbit/drbd` 中編輯 linbit DRBD 指令碼，以確保不會呼叫 `down`
* 負載平衡器至少需要 5 秒的時間進行回應，因此應用程式應為叢集感知且應可容許逾時。 其他架構也可提供協助，例如應用程式內部佇列、查詢中繼軟體等。
* 若要確保寫入作業會以可管理的步調結束，且會儘可能頻繁地將快取清除到磁碟以減少記憶體損失，MySQL 調整是有必要的。
* VM 互連中的寫入效能將會取決於虛擬開關，因為虛擬開關是 DRBD 用來複寫裝置的機制。
