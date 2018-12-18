---
title: Azure Linux VM 中的 DPDK | Microsoft Docs
description: 了解如何在 Linux 虛擬機器中設定 DPDK。
services: virtual-network
documentationcenter: na
author: laxmanrb
manager: gedegrac
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: ''
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/27/2018
ms.author: labattul
ms.openlocfilehash: 205a1e399eadd268ffaa390a7ebb4397fda9feff
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42444648"
---
# <a name="setup-dpdk-in-a-linux-virtual-machine"></a>在 Linux 虛擬機器中設定 DPDK

Azure 上的資料平面開發套件 (DPDK) 針對可略過虛擬機器核心網路堆疊的效能密集應用程式，提供更快速的使用者空間封包處理架構。

使用核心網路堆疊的典型封包處理為中斷驅動式。 每次網路介面接收連入封包時，會有一個核心中斷可處理從核心空間到使用者空間的封包和環境切換。 DPDK 可免除環境切換和中斷驅動式方法，以便使用輪詢模式驅動程式進行使用者空間實作，而達到快速的封包處理。

DPDK 是由一組使用者空間庫所組成，可供存取較低層級的資源，例如硬體、邏輯核心、記憶體管理，以及網路介面卡的輪詢模式驅動程式。

DPDK 可以在 Azure 虛擬機器中執行，並支援多個作業系統散發套件。 DPDK 以網路虛擬設備 (NVA) (例如虛擬路由器、防火牆、VPN、負載平衡器、進化版封包核心和拒絕服務 (DDoS) 應用程式) 的形式，在推動網路功能虛擬化實作方面的提供關鍵效能差異。

## <a name="benefit"></a>優點

**更高的每秒封包數 (PPS)**：略過核心並掌控使用者空間中的封包，可藉由消除環境切換來減少週期計數，並改善 Azure Linux 虛擬機器中每秒處理的封包率。


## <a name="supported-operating-systems"></a>受支援的作業系統

以下是 Azure 資源庫中支援的散發套件：

| Linux 作業系統     | 核心版本        |
|--------------|----------------       |
| Ubuntu 16.04 | 4.15.0-1015-azure     |
| Ubuntu 18.04 | 4.15.0-1015-azure     |
| SLES 15      | 4.12.14-5.5-azure     |
| RHEL 7.5     | 3.10.0-862.9.1.el7    |
| CentOS 7.5   | 3.10.0-862.3.3.el7    |

**自訂核心支援**

請參閱[適用於建置 Azure 調整 Linux 核心的修補程式](https://github.com/microsoft/azure-linux-kernel)以取得任何未列出的 Linux 核心版本，或如需進一步資訊，請連絡 [azuredpdk@microsoft.com](mailto:azuredpdk@microsoft.com)。 

## <a name="region-support"></a>區域支援

所有 Azure 區域都支援 DPDK。

## <a name="prerequisites"></a>必要條件

必須在 Linux 虛擬機器上啟用加速網路。 虛擬機器應該有至少兩個網路介面，其中一個介面用於管理。 了解如何[建立已啟用加速網路的 Linux 虛擬機器](create-vm-accelerated-networking-cli.md)。

## <a name="install-dpdk-dependencies"></a>安裝 DPDK 相依項目

### <a name="ubuntu-1604"></a>Ubuntu 16.04

```bash
sudo add-apt-repository ppa:canonical-server/dpdk-azure -y
sudo apt-get update
sudo apt-get install -y librdmacm-dev librdmacm1 build-essential libnuma-dev libmnl-dev
```

### <a name="ubuntu-1804"></a>Ubuntu 18.04

```bash
sudo apt-get update
sudo apt-get install -y librdmacm-dev librdmacm1 build-essential libnuma-dev libmnl-dev
```

### <a name="rhel75centos-75"></a>RHEL7.5/CentOS 7.5

```bash
yum -y groupinstall "Infiniband Support"
sudo dracut --add-drivers "mlx4_en mlx4_ib mlx5_ib" -f
yum install -y gcc kernel-devel-`uname -r` numactl-devel.x86_64 librdmacm-devel libmnl-devel
```

### <a name="sles-15"></a>SLES 15

**Azure 核心**

```bash
zypper  \
  --no-gpg-checks \
  --non-interactive \
  --gpg-auto-import-keys install kernel-azure kernel-devel-azure gcc make libnuma-devel numactl librdmacm1 rdma-core-devel
```

**預設核心**

```bash
zypper \
  --no-gpg-checks \
  --non-interactive \
  --gpg-auto-import-keys install kernel-default-devel gcc make libnuma-devel numactl librdmacm1 rdma-core-devel
```

## <a name="setup-virtual-machine-environment-once"></a>設定虛擬機器環境 (一次)

1. [下載最新的 DPDK](https://core.dpdk.org/download)。 Azure 需要 18.02 版或更高版本。
2. 首先使用 `make config T=x86_64-native-linuxapp-gcc` 建置預設組態。
3. 使用 `sed -ri 's,(MLX._PMD=)n,\1y,' build/.config` 在產生的組態中啟用 Mellanox PMD。
4. 使用 `make` 編譯。
5. 使用 `make install DESTDIR=<output folder>` 安裝。

# <a name="configure-runtime-environment"></a>設定執行階段環境

在重新開機之後，執行下列命令一次：

1. Hugepage

   * 對所有 numanodes 執行一次下列命令來設定 Hugepage：

     ```bash
     echo 1024 | sudo tee
     /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages
     ```

   *  使用 `mkdir /mnt/huge` 建立可供掛接的目錄。
   *  使用 `mount -t hugetlbfs nodev /mnt/huge` 掛接 Hugepage。
   *  使用 `grep Huge /proc/meminfo` 檢查 Hugepage 是否已保留。

     > [!NOTE]
     > 依照 DPDK 的[指示](http://dpdk.org/doc/guides/linux_gsg/sys_reqs.html#use-of-hugepages-in-the-linux-environment)可以修改 grub 檔案，以便在開機時保留 Hugepage。 指示位於頁面底部。 在 Azure Linux 虛擬機器中執行時，請改為修改 /etc/config/grub.d 之下的檔案，以在重新開機期間保留 Hugepage。

2. MAC 和 IP 位址： 使用 `ifconfig –a` 來檢視網路介面的 MAC 和 IP 位址。 *VF* 網路介面和 *NETVSC* 網路介面都有相同的 MAC 位址，但僅限於具有 IP 位址的 *NETVSC* 網路介面。 VF 介面會當作 NETVSC 介面的從屬介面執行。

3. PCI 位址

   * 使用 `ethtool -i <vf interface name>` 來了解哪個 PCI 位址要用於 *VF*。
   * 如果 *eth0* 已啟用加速網路，請確定 testpmd 不會意外接管 *eth0* 的 VF pci 裝置。 如果 DPDK 應用程式不小心接管了管理網路介面並導致 SSH 連線中斷，請使用序列主控台來終止 DPDK 應用程式，或是停止或啟動虛擬機器。

4. 在每次開機時使用 `modprobe -a ib_uverbs` 載入 *ibuverbs*。 僅限於 SLES 15，也可使用 `modprobe -a mlx4_ib` 載入 *mlx4_ib*。

## <a name="failsafe-pmd"></a>保全 PMD

DPDK 應用程式必須透過在 Azure 中公開的保全 PMD 執行。 如果應用程式直接透過 VF PMD 執行，它不會收到**所有**傳送到 VM 的封包，因為某些封包將會透過綜合介面顯示。 透過保全 PMD 執行可保證應用程式收到所有傳送給它的封包，也可確保應用程式會繼續在 DPDK 模式中執行，即使 VF 在主機運作時遭到撤銷亦然。 如需有關保全 PMD 的詳細資訊，請參閱[保全輪詢模式驅動程式庫](http://doc.dpdk.org/guides/nics/fail_safe.html)。

## <a name="run-testpmd"></a>執行 testpmd

在 *testpmd* 命令之前使用 `sudo`，以在根模式中執行。

### <a name="basic-sanity-check-failsafe-adapter-initialization"></a>基本：例行性檢查、保全配接器初始化

1. 執行下列命令來啟動單一連接埠 testpmd 應用程式：

   ```bash
   testpmd -w <pci address from previous step> \
     --vdev="net_vdev_netvsc0,iface=eth1" \
     -- -i \
     --port-topology=chained
    ```

2. 執行下列命令來啟動雙重連接埠 testpmd 應用程式：

   ```bash
   testpmd -w <pci address nic1> \
   -w <pci address nic2> \
   --vdev="net_vdev_netvsc0,iface=eth1" \
   --vdev="net_vdev_netvsc1,iface=eth2" \
   -- -i
   ```

   如果使用 2 個以上的 NIC 執行，則 `--vdev` 引數會遵循此模式：`net_vdev_netvsc<id>,iface=<vf’s pairing eth>`。

3.  啟動後，執行 `show port info all` 來檢查連接埠資訊。 您應該會看到一或兩個 net_failsafe (不是 *net_mlx4*) 的 DPDK 連接埠。
4.  使用 `start <port> /stop <port>` 來啟動流量。

前述命令會在互動模式中啟動 *testpmd* (建議做法)，以試用一些 testpmd 命令。

### <a name="basic-single-sendersingle-receiver"></a>基本：單一傳送者/單一接收者

下列命令會定期列印每秒封包數的統計資料：

1. 在 TX 端上，執行下列命令：

   ```bash
   testpmd \
     -l <core-list> \
     -n <num of mem channels> \
     -w <pci address of the device intended to use> \
     --vdev="net_vdev_netvsc<id>,iface=<the iface to attach to>" \
     -- --port-topology=chained \
     --nb-cores <number of cores to use for test pmd> \
     --forward-mode=txonly \
     --eth-peer=<port id>,<receiver peer MAC address> \
     --stats-period <display interval in seconds>
   ```

2. 在 RX 端上，執行下列命令：

   ```bash
   testpmd \
     -l <core-list> \
     -n <num of mem channels> \
     -w <pci address of the device intended to use> \
     --vdev="net_vdev_netvsc<id>,iface=<the iface to attach to>" \
     -- --port-topology=chained \
     --nb-cores <number of cores to use for test pmd> \
     --forward-mode=rxonly \
     --eth-peer=<port id>,<sender peer MAC address> \
     --stats-period <display interval in seconds>
   ```

在虛擬機器上執行前述命令時，變更 `app/test-pmd/txonly.c` 中的 *IP_SRC_ADDR* 和 *IP_DST_ADDR*，以在編譯前符合虛擬機器的實際 IP 位址。 否則，封包會在到達接收者之前捨棄。

### <a name="advanced-single-sendersingle-forwarder"></a>進階：單一傳送者/單一轉寄者
下列命令會定期列印每秒封包數的統計資料：

1. 在 TX 端上，執行下列命令：

   ```bash
   testpmd \
     -l <core-list> \
     -n <num of mem channels> \
     -w <pci address of the device intended to use> \
     --vdev="net_vdev_netvsc<id>,iface=<the iface to attach to>" \
     -- --port-topology=chained \
     --nb-cores <number of cores to use for test pmd> \
     --forward-mode=txonly \
     --eth-peer=<port id>,<receiver peer MAC address> \
     --stats-period <display interval in seconds>
    ```

2. 在 FWD 端上，執行下列命令：

   ```bash
   testpmd \
     -l <core-list> \
     -n <num of mem channels> \
     -w <pci address NIC1> \
     -w <pci address NIC2> \
     --vdev="net_vdev_netvsc<id>,iface=<the iface to attach to>" \
     --vdev="net_vdev_netvsc<2nd id>,iface=<2nd iface to attach to>" (you need as many --vdev arguments as the number of devices used by testpmd, in this case) \
     -- --nb-cores <number of cores to use for test pmd> \
     --forward-mode=io \
     --eth-peer=<recv port id>,<sender peer MAC address> \
     --stats-period <display interval in seconds>
    ```

在虛擬機器上執行前述命令時，變更 `app/test-pmd/txonly.c` 中的 *IP_SRC_ADDR* 和 *IP_DST_ADDR*，以在編譯前符合虛擬機器的實際 IP 位址。 否則，封包會在到達轉寄者之前捨棄。 您將無法讓第三部電腦接收轉寄的流量，因為 *testpmd* 轉寄者不會修改第 3 層位址 (除非您變更一些程式碼)。

## <a name="references"></a>參考

* [EAL 選項](https://dpdk.org/doc/guides/testpmd_app_ug/run_app.html#eal-command-line-options)
* [Testpmd 命令](https://dpdk.org/doc/guides/testpmd_app_ug/run_app.html#testpmd-command-line-options)
