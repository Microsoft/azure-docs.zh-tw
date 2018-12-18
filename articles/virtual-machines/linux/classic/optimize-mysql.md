---
title: 在 Azure Linux 上最佳化 MySQL 效能 | Microsoft Docs
description: 了解如何最佳化在執行 Linux 之 Azure 虛擬機器 (VM) 上執行的 MySQL。
services: virtual-machines-linux
documentationcenter: ''
author: NingKuang
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: 0c1c7fc5-a528-4d84-b65d-2df225f2233f
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/31/2017
ms.author: ningk
ms.openlocfilehash: 447532452a848c88fd927f42e4263cef4742dd89
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "30841498"
---
# <a name="optimize-mysql-performance-on-azure-linux-vms"></a>在 Azure Linux 上最佳化 MySQL 效能
有許多因素會影響 Azure 上的 MySQL 效能，均與虛擬硬體選取和軟體設定有關。 本文著重於透過儲存體、系統和資料庫設定最佳化效能。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有兩種：[Azure Resource Manager](../../../resource-manager-deployment-model.md) 和傳統。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如需使用 Resource Manager 模型進行 Linux VM 最佳化的詳細資訊，請參閱[在 Azure 上最佳化您的 Linux VM](../optimization.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

## <a name="utilize-raid-on-an-azure-virtual-machine"></a>在 Azure 虛擬機器上利用 RAID
儲存體是影響雲端環境中的資料庫效能的關鍵因素。 相較於單一磁碟，RAID 可透過並行提供更快速的存取。 如需詳細資訊，請參閱[標準 RAID 層級](http://en.wikipedia.org/wiki/Standard_RAID_levels)。   

可透過 RAID 改進 Azure 中的磁碟 I/O 輸送量和 I/O 回應時間。 我們的實驗室測試顯示︰當 RAID 磁碟數目增加一倍 (從 2 到 4、4 到 8 等) 時，磁碟 I/O 輸送量可以增加一倍，而 I/O 回應時間則平均減少一半。 如需詳細資訊，請參閱 [附錄 A](#AppendixA) 。  

除了磁碟 I/O 以外，MySQL 效能會在您提高 RAID 層級時改善。  如需詳細資訊，請參閱 [附錄 B](#AppendixB) 。  

您也可能要考慮區塊大小。 通常當您有較大的區塊大小時，您的額外負荷會比較低，尤其是大量寫入時。 不過，當區塊大小太大時，可能會增加額外的負荷，進而使您無法利用 RAID。 目前的預設大小為 512 KB，經證明是大多數一般實際執行環境的最佳大小。 如需詳細資訊，請參閱 [附錄 C](#AppendixC) 。   

您可針對不同虛擬機器類型新增的磁碟數目會有所限制。 [Azure 的虛擬機器和雲端服務大小](http://msdn.microsoft.com/library/azure/dn197896.aspx)中有這些限制的詳細說明。 雖然您可以選擇設定具有較少磁碟的 RAID，但您需要連接四個資料磁碟，才能遵循本文中的 RAID 4 範例。  

本文假設您已經建立 Linux 虛擬機器並已安裝及設定 MYSQL 。 如需開始使用的詳細資訊，請參閱「如何在 Azure 上安裝 MySQL」。  

### <a name="set-up-raid-on-azure"></a>在 Azure 上設定 RAID
下列步驟說明如何使用 Azure 入口網站，在 Azure 上建立 RAID。 您也可以使用 Windows PowerShell 指令碼設定 RAID。
在此範例中，我們將設定具有四個磁碟的 RAID 0。  

#### <a name="add-a-data-disk-to-your-virtual-machine"></a>將資料磁碟新增至您的虛擬機器
在 Azure 入口網站中移至儀表板，並選取要新增資料磁碟的虛擬機器。 在此範例中，虛擬機器是 mysqlnode1。  

<!--![Virtual machines][1]-->

按一下 [磁碟]，然後按一下 [連結新項目]。

![虛擬機器新增磁碟](media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-Disks-option.png)

建立一個新的 500 GB 磁碟。 確定 [主機快取偏好設定] 是設為 [無]。  完成時，請按一下 [確定] 。

![連接空的磁碟](media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-attach-empty-disk.png)


這會將一個空的磁碟新增到虛擬機器中。 再重複執行此步驟三次，您的 RAID 就有四個資料磁碟。  

查看核心訊息記錄檔，您可以看到虛擬機器中新增的磁碟機。 例如，若要在 Ubuntu 上查看此資料，請使用下列命令：  

    sudo grep SCSI /var/log/dmesg

#### <a name="create-raid-with-the-additional-disks"></a>建立具有額外磁碟的 RAID
下列步驟說明如何[在 Linux 上設定軟體 RAID](../configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

> [!NOTE]
> 如果您使用 XFS 檔案系統，請在建立 RAID 後執行下列步驟。
>
>

若要在 Debian、Ubuntu 或 Linux Mint 上安裝 XFS，請使用下列命令：  

    apt-get -y install xfsprogs  

若要在 Fedora 、CentOS 或 RHEL 上安裝 XFS，請使用下列命令：  

    yum -y install xfsprogs  xfsdump


#### <a name="set-up-a-new-storage-path"></a>設定新的儲存體路徑
使用下列命令來設定新的儲存體路徑：  

    root@mysqlnode1:~# mkdir -p /RAID0/mysql

#### <a name="copy-the-original-data-to-the-new-storage-path"></a>將原始資料複製到新的儲存體路徑
使用下列命令將資料複製到新的儲存體路徑：  

    root@mysqlnode1:~# cp -rp /var/lib/mysql/* /RAID0/mysql/

#### <a name="modify-permissions-so-mysql-can-access-read-and-write-the-data-disk"></a>修改權限，因此 MySQL 可以存取 (讀取和寫入) 資料磁碟
使用下列命令來修改權限：  

    root@mysqlnode1:~# chown -R mysql.mysql /RAID0/mysql && chmod -R 755 /RAID0/mysql


## <a name="adjust-the-disk-io-scheduling-algorithm"></a>調整磁碟 I/O 排程演算法
Linux 會實作四種類型的 I/O 排程演算法：  

* NOOP 演算法 (沒有作業)
* 期限演算法 (期限)
* 完全公平佇列演算法 (CFQ)
* 預算週期演算法 (預期)  

您可以在不同的狀況下選取不同的 I/O 排程器，讓效能達到最佳化。 在完全隨機存取環境中，CFQ 與期限演算法之間的效能沒有明顯差別。 我們建議將 MySQL 資料庫環境設為 [期限]，以求穩定性。 如果有大量循序 I/O，CFQ 可能會降低磁碟 I/O 效能。   

對於 SSD 和其他設備，NOOP 或 [期限] 可達到比預設排程器更佳的效能。   

在核心 2.5 之前，預設 I/O 排程演算法為 [期限]。 從核心 2.6.18 開始，CFQ 成為預設 I/O 排程演算法。  您可以在核心開機時指定此設定，或在系統執行時動態修改此設定。  

下列範例示範如何檢查預設排程器並將其設定為 Debian 散發版本系列中的 NOOP 演算法。  

### <a name="view-the-current-io-scheduler"></a>檢視目前的 I/O 排程器
若要檢視排程器，請執行下列命令：  

    root@mysqlnode1:~# cat /sys/block/sda/queue/scheduler

您會看到下列輸出，其表示目前的排程器：  

    noop [deadline] cfq


### <a name="change-the-current-device-devsda-of-the-io-scheduling-algorithm"></a>變更 I/O 排程演算法的目前裝置 (/dev/sda)
執行下列命令以變更目前的裝置：  

    azureuser@mysqlnode1:~$ sudo su -
    root@mysqlnode1:~# echo "noop" >/sys/block/sda/queue/scheduler
    root@mysqlnode1:~# sed -i 's/GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=noop"/g' /etc/default/grub
    root@mysqlnode1:~# update-grub

> [!NOTE]
> 單獨針對 /dev/sda 設定此項並不是很有用。 必須在資料庫所在的所有資料磁碟上進行設定。  
>
>

您應會看到下列輸出，表示 grub.cfg 已成功重建且預設排程器已更新為 NOOP：  

    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-3.13.0-34-generic
    Found initrd image: /boot/initrd.img-3.13.0-34-generic
    Found linux image: /boot/vmlinuz-3.13.0-32-generic
    Found initrd image: /boot/initrd.img-3.13.0-32-generic
    Found memtest86+ image: /memtest86+.elf
    Found memtest86+ image: /memtest86+.bin
    done

若為 Red Hat 散發版本系列，您只需要使用下列命令：

    echo 'echo noop >/sys/block/sda/queue/scheduler' >> /etc/rc.local

## <a name="configure-system-file-operations-settings"></a>設定系統檔案作業設定
最佳作法是停用檔案系統上的 atime 記錄功能。 Atime 是上次檔案存取時間。 每當存取檔案時，檔案系統就會在記錄檔中記錄時間戳記。 不過，很少使用這項資訊。 如果您不需要它，可予以停用，將會減少整體的磁碟存取時間。  

若要停用 atime 記錄，您必須修改檔案系統組態檔案 /etc/ fstab，並新增 **noatime** 選項。  

例如，編輯 vim /etc/fstab 檔案，新增如下範例所示的 noatime：  

    # CLOUD_IMG: This file was created/modified by the Cloud Image build process
    UUID=3cc98c06-d649-432d-81df-6dcd2a584d41       /        ext4   defaults,discard        0 0
    #Add the “noatime” option below to disable atime logging
    UUID="431b1e78-8226-43ec-9460-514a9adf060e"     /RAID0   xfs   defaults,nobootwait, noatime 0 0
    /dev/sdb1       /mnt    auto    defaults,nobootwait,comment=cloudconfig 0       2

然後，使用下列命令重新掛接檔案系統：  

    mount -o remount /RAID0

測試修改過的結果。 當您修改測試檔案時，存取時間不會更新。 下列範例顯示修改之前和之後的程式碼外觀。

之前：        

![存取修改前的程式碼][5]

之後︰

![存取修改後的程式碼][6]

## <a name="increase-the-maximum-number-of-system-handles-for-high-concurrency"></a>增加高並行存取的系統控制代碼數目上限
MySQL 是高並行存取資料庫。 Linux 的並行控制代碼預設數目為 1024，這並不足夠。 使用下列步驟來增加系統的並行控制代碼上限，以支援 MySQL 的高並行存取。

### <a name="modify-the-limitsconf-file"></a>修改 limits.conf 檔案
若要增加允許的並行控制代碼上限，在 /etc/security/limits.conf 檔案中新增下列四行。 請注意 65536 是系統可支援的數目上限。   

    * soft nofile 65536
    * hard nofile 65536
    * soft nproc 65536
    * hard nproc 65536

### <a name="update-the-system-for-the-new-limits"></a>更新系統，以取得新的限制
若要更新系統，請執行下列命令：  

    ulimit -SHn 65536
    ulimit -SHu 65536

### <a name="ensure-that-the-limits-are-updated-at-boot-time"></a>確定會在開機時更新限制
將下列啟動命令放入 /etc/rc.local 檔案中，以便在開機時生效。  

    echo “ulimit -SHn 65536” >>/etc/rc.local
    echo “ulimit -SHu 65536” >>/etc/rc.local

## <a name="mysql-database-optimization"></a>MySQL 資料庫最佳化
若要在 Azure 上設定 MySQL，您可以使用與內部部署電腦上使用的相同效能微調策略。  

主要的 I/O 最佳化規則如下：   

* 增加快取的大小。
* 減少 I/O 回應時間。  

若要最佳化 MySQL 伺服器設定，您可以更新 my.cnf 檔案，該檔案是伺服器和用戶端電腦的預設組態檔。  

下列組態項目是影響 MySQL 效能的主要因素：  

* **innodb_buffer_pool_size**：緩衝集區包含經過緩衝處理的資料和索引。 這通常設定為 70% 的實體記憶體。
* **innodb_log_file_size**：這是重做記錄檔的大小。 您可以使用重做記錄檔來確保寫入作業快速、可靠並可在當機後復原。 這會設定為 512 MB，將提供大量空間給您記錄寫入作業。
* **max_connections**：應用程式有時不會正確關閉連線。 較大的值讓伺服器有更多的時間來回收閒置的連線。 連線數目上限為 10000，但建議的上限為 5000。
* **Innodb_file_per_table**：此設定可啟用或停用 InnoDB 在個別檔案中儲存資料表的功能。 開啟此選項可確保有效地套用數個進階管理作業。 從效能觀點來看，它可以加速資料表空間傳輸，並將 debris 管理效能最佳化。 這個選項的建議設定為 ON。</br></br>
從 MySQL 5.6 開始，預設設定為 ON，所以不需要採取任何動作。 舊版的預設值為 OFF。 應在載入資料之前變更此設定，因為只有新建的資料表會受影響。
* **innodb_flush_log_at_trx_commit**：預設值為 1，其範圍設為 0~2。 對獨立 MySQL DB 而言，預設值是最適合的選項。 設定為 2 可達到最大資料完整性，適合於 MySQL 叢集中的主機。 設定為 0 會讓資料遺失，這可能會影響可靠性，在某些情況下，效能會更佳，適合於 MySQL 叢集中的從屬。
* **Innodb_log_buffer_size**：記錄緩衝區允許交易執行，而不需在交易認可前將記錄檔排清到磁碟。 不過，如果有大型二進位物件或文字欄位，將會快速地耗用快取，並將觸發頻繁的磁碟 I/O。 如果 Innodb_log_waits 狀態變數不是 0，最好能增加緩衝區大小。
* **query_cache_size**：最佳選項是一開始就將它停用。 將 query_cache_size 設為 0 (這是 MySQL 5.6 中的預設值)，並使用其他方法來加速查詢。  

請參閱[附錄 D](#AppendixD) 來比較最佳化前後的效能。

## <a name="turn-on-the-mysql-slow-query-log-for-analyzing-the-performance-bottleneck"></a>開啟 MySQL 緩慢查詢記錄，以便分析效能瓶頸
MySQL 緩慢查詢記錄檔可協助您識別 MySQL 的較慢查詢。 啟用 MySQL 緩慢查詢記錄檔之後，您可以使用 **mysqldumpslow** 之類的 MySQL 工具來識別效能瓶頸。  

預設不會啟用此項目。 開啟緩慢查詢記錄檔可能會耗用一些 CPU 資源。 我們建議您暫時啟用這個選項，以便排解效能瓶頸。 若要開啟緩慢查詢記錄檔︰

1. 將下列幾行加至 my.cnf 檔案結尾，以修改該檔案：

        long_query_time = 2
        slow_query_log = 1
        slow_query_log_file = /RAID0/mysql/mysql-slow.log

2. 重新啟動 MySQL 伺服器。

        service  mysql  restart

3. 使用 **show** 命令檢查設定是否生效。

![Slow-query-log ON][7]   

![Slow-query-log 結果][8]

在此範例中，您可以看到緩慢查詢功能已開啟。 您可以接著使用 **mysqldumpslow** 工具，判斷效能瓶頸並將效能最佳化，例如，新增索引。

## <a name="appendices"></a>附錄
以下是在目標實驗室環境中產生的範例效能測試資料。 其提供採用不同效能調整方法之效能資料趨勢的一般背景。 在不同的環境或產品版本之下，結果可能會有所不同。

### <a name="AppendixA"></a>附錄 A  
**不同 RAID 層級的磁碟效能 (IOPS)**

![不同 RAID 層級的磁碟 IOPS][9]

**測試命令**  

    fio -filename=/path/test -iodepth=64 -ioengine=libaio -direct=1 -rw=randwrite -bs=4k -size=5G -numjobs=64 -runtime=30 -group_reporting -name=test-randwrite

> [!NOTE]
> 這項測試的工作負載會使用 64 個執行緒，並嘗試達到 RAID 的上限。
>
>

### <a name="AppendixB"></a>附錄 B  
**不同 RAID 層級的 MySQL 效能 (輸送量) 比較**   
(XFS檔案系統)

![不同 RAID 層級的 MySQL 效能比較][10]  
![不同 RAID 層級的 MySQL 效能比較][11]

**測試命令**

    mysqlslap -p0ps.123 --concurrency=2 --iterations=1 --number-int-cols=10 --number-char-cols=10 -a --auto-generate-sql-guid-primary --number-of-queries=10000 --auto-generate-sql-load-type=write –engine=innodb

**不同 RAID 層級的 MySQL 效能 (OLTP) 比較**  
![不同 RAID 層級的 MySQL 效能 (OLTP) 比較][12]

**測試命令**

    time sysbench --test=oltp --db-driver=mysql --mysql-user=root --mysql-password=0ps.123  --mysql-table-engine=innodb --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-socket=/var/run/mysqld/mysqld.sock --mysql-db=test --oltp-table-size=1000000 prepare

### <a name="AppendixC"></a>附錄 C   
**不同區塊大小的磁碟效能 (IOPS) 比較**  
(XFS檔案系統)

![][13]

**測試命令**  

    fio -filename=/path/test -iodepth=64 -ioengine=libaio -direct=1 -rw=randwrite -bs=4k -size=30G -numjobs=64 -runtime=30 -group_reporting -name=test-randwrite
    fio -filename=/path/test -iodepth=64 -ioengine=libaio -direct=1 -rw=randwrite -bs=4k -size=1G -numjobs=64 -runtime=30 -group_reporting -name=test-randwrite  

這項測試所使用的檔案大小分別為 30 GB 和 1 GB，搭配 RAID 0 (4 磁碟) XFS fie 系統。

### <a name="AppendixD"></a>附錄 D  
**最佳化之前和之後的 MySQL 效能 (輸送量) 比較**  
(XFS檔案系統)

![最佳化之前和之後的 MySQL 效能 (輸送量) 比較][14]

**測試命令**

    mysqlslap -p0ps.123 --concurrency=2 --iterations=1 --number-int-cols=10 --number-char-cols=10 -a --auto-generate-sql-guid-primary --number-of-queries=10000 --auto-generate-sql-load-type=write –engine=innodb,misam

**預設值與最佳化的組態設定如下所示：**

| 參數 | 預設值 | 最佳化 |
| --- | --- | --- |
| **innodb_buffer_pool_size** |None |7 GB |
| **innodb_log_file_size** |5 MB |512 MB |
| **max_connections** |100 |5000 |
| **innodb_file_per_table** |0 |1 |
| **innodb_flush_log_at_trx_commit** |1 |2 |
| **innodb_log_buffer_size** |8 MB |128 MB |
| **query_cache_size** |16 MB |0 |

如需更詳細的[最佳化設定參數](http://dev.mysql.com/doc/refman/5.6/en/innodb-configuration.html)，請參閱 [MySQL 官方指示](http://dev.mysql.com/doc/refman/5.6/en/innodb-parameters.html#sysvar_innodb_flush_method)。  

  **測試環境**  

| 硬體 | 詳細資料 |
| --- | --- |
| CPU |AMD Opteron(tm) 處理器 4171 HE/4 核心 |
| 記憶體 |14 GB |
| 磁碟 |10 GB/磁碟 |
| 作業系統 |Ubuntu 14.04.1 LTS |

[1]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-01.png
[2]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-02.png
[3]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-03.png
[4]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-04.png
[5]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-05.png
[6]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-06.png
[7]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-07.png
[8]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-08.png
[9]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-09.png
[10]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-10.png
[11]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-11.png
[12]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-12.png
[13]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-13.png
[14]:media/optimize-mysql/virtual-machines-linux-optimize-mysql-perf-14.png

