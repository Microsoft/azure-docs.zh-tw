---
title: 在 Azure 上設計和實作 Oracle 資料庫 | Microsoft Docs
description: 在 Azure 環境中設計和實作 Oracle 資料庫。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: romitgirdhar
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/02/2018
ms.author: rogirdh
ms.openlocfilehash: d4c0bbdfb1afcef33727ba4b5b432c5de79168d4
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39495215"
---
# <a name="design-and-implement-an-oracle-database-in-azure"></a>在 Azure 中設計和實作 Oracle 資料庫

## <a name="assumptions"></a>假設

- 您打算將 Oracle 資料庫從內部部署環境移轉至 Azure。
- 您已了解 Oracle AWR 報表中的各種計量。
- 您已基本了解應用程式效能和平台使用量。

## <a name="goals"></a>目標

- 了解如何將 Azure 中的 Oracle 部署最佳化。
- 探索 Azure 環境中 Oracle 資料庫的效能調整選項。

## <a name="the-differences-between-an-on-premises-and-azure-implementation"></a>內部部署和 Azure 實作之間的差異 

以下是一些您在將內部部署應用程式移轉至 Azure 時應該謹記的重要事項。 

其中的一項重要差異是，Azure 實作中的資源 (例如 VM、磁碟和虛擬網路) 會與其他用戶端共用。 此外，您可以根據需求來節流資源。 Azure 更聚焦在讓失敗存活 (MTTR)，而不是聚焦於避免失敗 (MTBF)。

下表列出 Oracle 資料庫在內部部署實作和 Azure 實作之間的一些差異。

> 
> |  | **內部部署實作** | **Azure 實作** |
> | --- | --- | --- |
> | **網路功能** |LAN/WAN  |SDN (軟體定義網路)|
> | **安全性群組** |IP/連接埠限制工具 |[網路安全性群組 (NSG)](https://azure.microsoft.com/blog/network-security-groups) |
> | **恢復功能** |MTBF (平均失敗時間) |MTTR (平均復原時間)|
> | **預定的維修** |修補/升級|[可用性設定組](https://docs.microsoft.com/azure/virtual-machines/windows/infrastructure-availability-sets-guidelines) (Azure 所管理的修補/升級) |
> | **Resource** |專用  |與其他用戶端共用|
> | **區域** |資料中心 |[區域配對](https://docs.microsoft.com/azure/virtual-machines/windows/regions-and-availability)|
> | **儲存體** |SAN/實體磁碟 |[Azure 受控儲存體](https://azure.microsoft.com/pricing/details/managed-disks/?v=17.23h)|
> | **調整** |垂直調整 |水平調整|


### <a name="requirements"></a>需求

- 決定資料庫大小和成長率。
- 決定 IOPS 需求，您可以根據 Oracle AWR 報表或其他網路監視工具進行評估。

## <a name="configuration-options"></a>組態選項

有四個可能的區域可供您進行調整，以改善 Azure 環境中的效能：

- 虛擬機器大小
- 網路輸送量
- 磁碟類型和設定
- 磁碟快取設定

### <a name="generate-an-awr-report"></a>產生 AWR 報表

如果您目前已有 Oracle 資料庫，且打算移轉至 Azure，您會有數個選項。 您可以執行 Oracle AWR 報表來取得計量 (IOPS、Mbps、GiB 等)。 然後根據收集到的計量選擇 VM。 或者，連絡基礎結構小組，取得類似的資訊。

您可以考慮在一般和尖峰工作負載期間執行 AWR 報表，以進行比較。 根據這些報表，您可以根據平均工作負載或最大工作負載來調整 VM 大小。

以下是如何產生 AWR 報表的範例：

```bash
$ sqlplus / as sysdba
SQL> EXEC DBMS_WORKLOAD_REPOSITORY.CREATE_SNAPSHOT;
SQL> @?/rdbms/admin/awrrpt.sql
```

### <a name="key-metrics"></a>重要計量

以下是您可以從 AWR 報表取得的計量：

- 核心總數
- CPU 時脈速度
- 記憶體總計 (GB)
- CPU 使用率
- 尖峰資料傳送速率
- I/O 變更率 (讀/寫)
- 重做記錄速率 (MBPs)
- 網路輸送量
- 網路延遲速率 (低/高)
- 資料庫大小 (GB)
- 透過 SQL*Net 從/至用戶端收到的位元組

### <a name="virtual-machine-size"></a>虛擬機器大小

#### <a name="1-estimate-vm-size-based-on-cpu-memory-and-io-usage-from-the-awr-report"></a>1.根據 AWR 報表中的 CPU、記憶體和 I/O 使用量來預估 VM 大小

您可以查看的一個事項是前五個定時前景事件，其指出系統瓶頸位置。

例如在下圖中，記錄檔案同步在頂端。 這代表等候 LGWR 將記錄緩衝區寫入重做記錄檔的次數。 這些結果代表您需要效能更好的儲存體或磁碟。 此外，此圖也會顯示 CPU (核心) 數目和記憶體數量。

![AWR 報表頁面的螢幕擷取畫面](./media/oracle-design/cpu_memory_info.png)

下圖顯示讀取和寫入的 I/O 總計。 報表統計期間共有 59 GB 的讀取和 247.3 GB 的寫入。

![AWR 報表頁面的螢幕擷取畫面](./media/oracle-design/io_info.png)

#### <a name="2-choose-a-vm"></a>2.選擇 VM

根據您從 AWR 報表收集到的資訊，下一個步驟是選擇符合您需求且大小類似的 VM。 您可以在[記憶體最佳化](../../linux/sizes-memory.md)一文中找到可用 VM 的清單。

#### <a name="3-fine-tune-the-vm-sizing-with-a-similar-vm-series-based-on-the-acu"></a>3.根據 ACU 微調類似 VM 系列的 VM 大小

在您選擇 VM 之後，請注意 VM 的 ACU。 您可以根據較適合您需求的 ACU 值，選擇不同的 VM。 如需詳細資訊，請參閱 [Azure 計算單位](https://docs.microsoft.com/azure/virtual-machines/windows/acu)。

![ACU 單位頁面的螢幕擷取畫面](./media/oracle-design/acu_units.png)

### <a name="network-throughput"></a>網路輸送量

下圖顯示輸送量與 IOPS 之間的關聯性：

![輸送量的螢幕擷取畫面](./media/oracle-design/throughput.png)

總網路輸送量是根據下列資訊進行預估：
- SQL*Net 流量
- MBps x 伺服器數目 (輸出串流，例如 Oracle Data Guard)
- 其他因素，例如應用程式複寫

![SQL*Net 輸送量的螢幕擷取畫面](./media/oracle-design/sqlnet_info.png)

根據您的網路頻寬需求，有各種閘道類型可供您選擇。 這些類型包括基本、VpnGw 和 Azure ExpressRoute。 如需詳細資訊，請參閱 [VPN 閘道定價頁面](https://azure.microsoft.com/pricing/details/vpn-gateway/?v=17.23h)。

**建議**

- 與內部部署相較之下，網路延遲較高。 減少網路來回行程可以大幅改善效能。
- 若要減少來回行程，請合併相同虛擬機器上具有高交易或 “Chatty” 應用程式的應用程式。

### <a name="disk-types-and-configurations"></a>磁碟類型和設定

- 預設的 OS 磁碟：這些磁碟類型可提供持續性資料和快取。 它們最適合用在啟動時的 OS 存取，但其設計目的並非用於交易式或資料倉儲 (分析) 工作負載。

- 非受控磁碟：您可以使用這些磁碟類型來管理用來儲存虛擬硬碟 (VHD) 檔案 (對應至您的 VM 磁碟) 的儲存體帳戶。 VHD 檔案會以分頁 Blob 的形式儲存在 Azure 儲存體帳戶中。

- 受控磁碟：Azure 會管理您用於 VM 磁碟的儲存體帳戶。 您可以指定需要的磁碟類型 (進階或標準) 和磁碟大小。 Azure 會為您建立並管理該磁碟。

- 進階儲存體磁碟：這些磁碟類型最適合生產工作負載使用。 進階儲存體支援可連結至特定大小系列 VM (例如 DS、DSv2、GS 和 F 系列 VM) 的 VM 磁碟。 進階磁碟會隨附不同的大小，而且您可以選擇範圍從 32 GB 到 4096 GB 的磁碟。 每個磁碟大小都有自己的效能規格。 端視您的應用程式需求而定，您可以將一或多個磁碟連結至您的 VM。

當您從入口網站建立新的受控磁碟時，可以選擇您想要使用之磁碟類型的 [帳戶類型]。 請記住，並非所有可用的磁碟都會顯示在下拉式功能表中。 在選擇特定的 VM 大小之後，功能表只會根據該 VM 大小顯示可用的進階儲存體 SKU。

![受控磁碟頁面的螢幕擷取畫面](./media/oracle-design/premium_disk01.png)

如需詳細資訊，請參閱 [VM 高效能進階儲存體與受控磁碟](https://docs.microsoft.com/azure/storage/storage-premium-storage)。

在 VM 上設定儲存體之後，您可能會想要在建立資料庫之前對磁碟進行負載測試。 知道延遲和輸送量方面的 I/O 速率可以協助您判斷 VM 是否支援具有延遲目標的預期輸送量。

有數種工具可以進行應用程式負載測試，例如 Oracle Orion、Sysbench 和 Fio。

在部署好 Oracle 資料庫之後，請再次執行負載測試。 啟動您的一般和尖峰工作負載，其結果會顯示您環境的基準數據。

根據 IOPS 速率而非儲存體大小來調整儲存體大小可能更為重要。 例如，如果所需的 IOPS 是 5000，但您只需要 200 GB，則您可能仍會得到 P30 等級的進階磁碟，即使它隨附 200 GB 以上的儲存體。

IOPS 速率可以從 AWR 報表取得。 此速率是由重做記錄、實體讀取和寫入速率所決定。

![AWR 報表頁面的螢幕擷取畫面](./media/oracle-design/awr_report.png)

例如：重做大小是每秒 12,200,000 個位元組，相當於 11.63 MBPs。
IOPS 是 12,200,000 / 2,358 = 5,174。

在清楚了解 I/O 需求之後，即可選擇最符合這些需求的磁碟機組合。

**建議**

- 針對資料的資料表空間，使用受控儲存體或 Oracle ASM，將 I/O 工作負載分散到一些磁碟。
- 隨著 I/O 區塊大小的增加，針對讀取密集和寫入密集作業，新增更多資料磁碟。
- 增加大型循序處理序的區塊大小。
- 使用資料壓縮來減少 I/O (適用於資料和索引)。
- 區隔不同資料磁碟上的重做記錄、系統、暫時和重做 TS。
- 不要將任何應用程式檔案放在預設 OS 磁碟 (/dev/sda)。 這些磁碟不適合用於快速 VM 啟動階段，因此可能不會為您的應用程式提供良好的效能。

### <a name="disk-cache-settings"></a>磁碟快取設定

有三個主機快取選項：

- 唯讀：快取所有要求，以供未來讀取。 所有寫入會直接保存到 Azure Blob 儲存體。

- 讀寫：這是「預先讀取」演算法。 快取讀取和寫入，以供未來讀取。 非直接寫入式寫入會先保存到本機快取。 針對 SQL Server，因為它使用直接寫入式，所以寫入會保存到 Azure 儲存體。 它也會為輕量工作負載提供最低的磁碟延遲。

- 無 (停用)：使用此選項即可略過快取。 所有資料都會傳輸至磁碟，並保存到 Azure 儲存體。 這種方法可提供您最高 I/O 速率來進行 I/O 密集式工作負載。 您也需要考量「交易成本」。

**建議**

若要將輸送量最大化，建議您一開始先對主機快取使用 [無]。 針對進階儲存體，請記住您必須在使用 [唯讀] 或 [無] 選項掛接檔案系統時停用「屏障」。 將具有 UUID 的 /etc/fstab 檔案更新到磁碟。

如需詳細資訊，請參閱[適用於 Linux VM 的進階儲存體](https://docs.microsoft.com/azure/storage/storage-premium-storage#premium-storage-for-linux-vms)。

![受控磁碟頁面的螢幕擷取畫面](./media/oracle-design/premium_disk02.png)

- 針對 OS 磁碟，使用預設的 [讀取/寫入] 快取。
- 針對 SYSTEM、TEMP 和 UNDO，對快取功能使用 [無]。
- 針對 DATA，對快取功能使用 [無]。 但是，如果您的資料庫是唯讀或讀取密集，請使用 [唯讀] 快取。

除非您卸載 OS 層級的磁碟機，然後在進行變更後重新予以掛接，否則在儲存資料磁碟設定之後，就無法變更主機快取設定。


## <a name="security"></a>安全性

在安裝並設定 Azure 環境之後，下一個步驟是保護您的網路。 以下是一些建議：

- NSG 原則：可以透過子網路或 NIC 來定義 NSG。 它更容易控制應用程式防火牆這類事項之安全性和強制路由的子網路層級存取。

- Jumpbox：基於更安全的存取，系統管理員不應該直接連線至應用程式服務或資料庫。 Jumpbox 作為系統管理員機器與 Azure 資源之間的媒體。
![Jumpbox 拓撲頁面的螢幕擷取畫面](./media/oracle-design/jumpbox.png)

    系統管理員機器只應該對 Jumpbox 提供 IP 受限的存取權。 Jumpbox 應該要能夠存取應用程式和資料庫。

- 私人網路 (子網路)：我們建議您將應用程式服務和資料庫放在不同的子網路上，讓 NSG 原則可以設定更好的控制。


## <a name="additional-reading"></a>其他閱讀資料

- [設定 Oracle ASM](configure-oracle-asm.md)
- [設定 Oracle Data Guard](configure-oracle-dataguard.md)
- [設定 Oracle Golden Gate](configure-oracle-golden-gate.md)
- [Oracle 備份和復原](oracle-backup-recovery.md)

## <a name="next-steps"></a>後續步驟

- [教學課程︰建立高可用性 VM](../../linux/create-cli-complete.md)
- [瀏覽 VM 部署 Azure CLI 範例](../../linux/cli-samples.md)
