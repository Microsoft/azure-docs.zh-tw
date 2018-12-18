---
title: 在 Azure 中建立及上傳 Linux VHD
description: 了解如何建立及上傳包含 Linux 作業系統的 Azure 虛擬硬碟 (VHD)。
services: virtual-machines-linux
documentationcenter: ''
author: szarkos
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager,azure-service-management
ms.assetid: d351396c-95a0-4092-b7bf-c6aae0bbd112
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 03/12/2018
ms.author: szark
ms.openlocfilehash: 17b4df83b141d5365a8d6244c4ab73b0eba5ed73
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38972344"
---
# <a name="information-for-non-endorsed-distributions"></a>非背書散發套件的資訊
[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

只有使用其中一個[背書散發套件](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)時，Azure 平台 SLA 才適用於執行 Linux OS 的虛擬機器。 對於這些背書散發套件，Linux 映像會以所需的組態在 Azure Marketplace 中供應。

* [Azure 背書散發套件上的 Linux](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [支援 Microsoft Azure 中的 Linux 映像](https://support.microsoft.com/kb/2941892)

在 Azure 上執行的所有散發套件都必須符合一些先決條件，才能在平台上正常運作。  本文並未列出所有先決條件，因為每個散發套件都不同；而且即使您符合下列所有條件，還是可能需要詳加調整您的 Linux 系統，以確保它可以在平台上正常運作。

基於這個原因，我們建議您盡可能從 [Azure 背書散發套件上的 Linux](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 開始著手。 下列文章會逐步引導您如何準備 Azure 支援的各種 Linux 背書散發套件：

* **[CentOS 型散發套件](create-upload-centos.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Debian Linux](debian-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Oracle Linux](oracle-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Red Hat Enterprise Linux](redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[SLES 和 openSUSE](suse-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Ubuntu](create-upload-ubuntu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**

本文接下來會將重點放在於 Azure 上執行 Linux 散發套件時的一般指導方針。

## <a name="general-linux-installation-notes"></a>一般 Linux 安裝注意事項
* Azure 不支援 VHDX 格式，只支援 **固定 VHD**。  您可以使用 Hyper-V 管理員或 convert-vhd Cmdlet，將磁碟轉換為 VHD 格式。 如果您是使用 VirtualBox，即會在建立磁碟時選取 [固定大小]  而不是預設的動態配置。
* Azure 僅支援第 1 代虛擬機器。 您可以將第 1 代虛擬機器從 VHDX 轉換為 VHD 檔案格式，並從動態擴充轉換為固定大小的磁碟。 但您無法變更虛擬機器的世代。 如需詳細資訊，請參閱[應該在 Hyper-V 中建立第 1 代還是第 2 代的虛擬機器？](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)
* 允許的 VHD 大小上限為 1023 GB。
* 安裝 Linux 系統時，*建議*您使用標準磁碟分割而不是 LVM (常是許多安裝的預設設定)。 這可避免 LVM 與複製之 VM 的名稱衝突，特別是為了疑難排解而需要將作業系統磁碟連接至另一個相同的 VM 時。 如果願意，您可以在資料磁碟上使用 [LVM](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 或 [RAID](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
* 需要掛接 UDF 檔案系統的核心支援。 在 Azure 上第一次開機時，佈建組態會透過連接客體的 UDF 格式媒體傳遞至 Linux VM。 Azure Linux 代理程式必須能夠掛接 UDF 檔案系統讀取其組態並佈建 VM。
* Linux Kernel 2.6.37 以下的版本不支援較大 VM 大小 Hyper-V 上的NUMA。 這個問題主要會影響使用上游 Red Hat 2.6.32 kernel 的較舊散發套件，RHEL 6.6 (kernel-2.6.32-504) 已加以修正。 執行的自訂核心是 2.6.37 以前版本的系統，或 2.6.32-504 以前以 RHEL 為基礎的核心必須在 grub.conf 的核心命令列上設定開機參數 `numa=off`。 如需詳細資訊，請參閱 Red Hat [KB 436883](https://access.redhat.com/solutions/436883)。
* 請勿在作業系統磁碟上設定交換磁碟分割。 您可以設定 Linux 代理程式在暫存資源磁碟上建立交換檔。  您可以在以下步驟中找到與此有關的詳細資訊。
* Azure 上的所有 VHD 必須具有與 1 MB 對應的虛擬大小。 從未經處理的磁碟轉換成 VHD 時，您必須確定未經處理的磁碟大小在轉換前是 1 MB 的倍數。 如需更多詳細資料，請參閱以下步驟。

### <a name="installing-kernel-modules-without-hyper-v"></a>安裝不含 Hyper-V 的核心模組
Azure 在 Hyper-V Hypervisor 上執行，因此 Linux 會要求必須安裝某些核心模組，才能在 Azure 中執行。 如果您的 VM 是在 Hyper-V 外部建立的，除非 Linux 安裝程式偵測到執行所在環境是 Hyper-V 環境，否則 Linux 安裝程式在初始 Ramdisk (initrd 或 initramfs) 中可能不包含 Hyper-V 的驅動程式。 使用不同的虛擬化系統 (亦即 Virtualbox、KVM 等) 來準備您的 Linux 映像時，可能需要重新建置 initrd，以確保在初始 ramdisk 上至少有 `hv_vmbus` 和 `hv_storvsc` 核心模組可以使用。  目前至少已知在以上游 Red Hat 散發套件為基礎的系統上有此問題。

重新建置 initrd 或 initramfs 映像的機制會根據散發套件而有所不同。 請參閱散發套件的文件或洽支援人員，以了解適當程序。  以下是如何使用 `mkinitrd` 公用程式重新建置 initrd 的範例：

首先，備份現有的 initrd 映像：

    # cd /boot
    # sudo cp initrd-`uname -r`.img  initrd-`uname -r`.img.bak

接下來，使用 `hv_vmbus` 和 `hv_storvsc` 核心模組重新建置 initrd：

    # sudo mkinitrd --preload=hv_storvsc --preload=hv_vmbus -v -f initrd-`uname -r`.img `uname -r`


### <a name="resizing-vhds"></a>調整 VHD 的大小
Azure 上的 VHD 映像必須具有與 1 MB 對齊的虛擬大小。  一般而言，使用 Hyper-V 建立的 VHD 應已正確對應。  如果未正確對齊 VHD，當您嘗試從 VHD 建立「映像」時，可能會收到類似下面的錯誤訊息：

    "The VHD http://<mystorageaccount>.blob.core.windows.net/vhds/MyLinuxVM.vhd has an unsupported virtual size of 21475270656 bytes. The size must be a whole number (in MBs).”

若要補救這種行為，您可以使用 Hyper-V 管理員主控台或 [Resize-VHD](http://technet.microsoft.com/library/hh848535.aspx) Powershell Cmdlet 調整 VM 的大小。  如果您不是在 Windows 環境中執行，建議使用 qemu-img 來轉換 VHD (如果需要) 並調整其大小。

> [!NOTE]
> qemu-img >=2.2.1 的版本中已知有 Bug 會導致 VHD 的格式不正確。 此問題已在 QEMU 2.6 中修正。 建議使用 qemu-img 2.2.0 或更舊版本，或更新至 2.6 或更新版本。 參考：https://bugs.launchpad.net/qemu/+bug/1490611。
> 
> 

1. 直接使用工具 (例如 `qemu-img` 或 `vbox-manage`) 調整 VHD 的大小，可能會導致 VHD 無法開機。  因此，建議先將 VHD 轉換為 RAW 磁碟映像。  如果 VM 映像已建立為 RAW 磁碟映像 (有些 Hypervisor 的預設值，例如 KVM)，您可以省略此步驟：
   
       # qemu-img convert -f vpc -O raw MyLinuxVM.vhd MyLinuxVM.raw

2. 計算所需的磁碟映像大小，以確保虛擬大小對齊 1 MB。  下列 bash 殼層指令碼有助於此作業。  此指令碼會使用 "`qemu-img info`" 來判斷磁碟映像的虛擬大小，然後計算到下一個 1 MB 的大小：
   
       rawdisk="MyLinuxVM.raw"
       vhddisk="MyLinuxVM.vhd"
   
       MB=$((1024*1024))
       size=$(qemu-img info -f raw --output json "$rawdisk" | \
              gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
   
       rounded_size=$((($size/$MB + 1)*$MB))
       echo "Rounded Size = $rounded_size"

3. 依照上述指令碼中的設定，使用 $rounded_size 調整原始磁碟的大小：
   
       # qemu-img resize MyLinuxVM.raw $rounded_size

4. 現在，將 RAW 磁碟轉換回固定大小的 VHD：
   
       # qemu-img convert -f raw -o subformat=fixed -O vpc MyLinuxVM.raw MyLinuxVM.vhd

   或者，qemu 版本 **2.6 +** 包含 `force_size` 選項︰

       # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc MyLinuxVM.raw MyLinuxVM.vhd

## <a name="linux-kernel-requirements"></a>Linux Kernel 需求
適用於 Hyper-V 和 Azure 的 Linux Integration Services (LIS) 驅動程式會直接提供給上游 Linux Kernel。 許多包括最新 Linux 核心版本 (亦即 3.x) 的散發套件已經有這些驅動程式，或以其核心 提供這些驅動程式的向後移植 (Backport) 版本。  上游核心會透過新的修正和功能來不斷更新這些驅動程式，因此如果可能的話，建議您執行包含這些修正與更新的[背書散發套件](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

如果您要執行 Red Hat Enterprise Linux 版本 **6.0-6.3** 的變體，便必須安裝 Hyper-V 的最新 LIS 驅動程式。 您可以在 [這裡](http://go.microsoft.com/fwlink/p/?LinkID=254263&clcid=0x409)找到驅動程式。 從 RHEL **6.4+** (及衍生物件) 開始，核心已隨附 LIS 驅動程式，因此無需額外的安裝套件，即可在 Azure 上執行這些系統。

如果需要自訂核心，建議您使用最新的核心版本 (例如 **3.8+**)。 針對那些散發套件或自行維護核心的廠商，需要花費一點心力，定期將 LIS 驅動程式從上游核心向後移植 (Backport) 到您的自訂核心。  即使您已經在執行相對較新的核心版本，還是強烈建議您持續追蹤任何 LIS 驅動程式的上游修正，並視需要 Backport 這些修正。 您可以在 Linux Kernel 來源樹狀目錄中的 [MAINTAINERS](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/MAINTAINERS) 檔案找到 LIS 驅動程式原始程式檔的位置。

    F:    arch/x86/include/asm/mshyperv.h
    F:    arch/x86/include/uapi/asm/hyperv.h
    F:    arch/x86/kernel/cpu/mshyperv.c
    F:    drivers/hid/hid-hyperv.c
    F:    drivers/hv/
    F:    drivers/input/serio/hyperv-keyboard.c
    F:    drivers/net/hyperv/
    F:    drivers/scsi/storvsc_drv.c
    F:    drivers/video/fbdev/hyperv_fb.c
    F:    include/linux/hyperv.h
    F:    tools/hv/

至少，我們已知缺少下列修補程式有可能會在 Azure 上造成問題，因此這些修補程式必須包含在核心內。 這絕對不是詳盡或完整的所有散發套件清單：

* [ata_piix：依預設將磁碟委託給 Hyper-V 驅動程式](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/ata/ata_piix.c?id=cd006086fa5d91414d8ff9ff2b78fbb593878e3c)
* [storvsc：負責 RESET 路徑中的在途封包](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/scsi/storvsc_drv.c?id=5c1b10ab7f93d24f29b5630286e323d1c5802d5c)
* [storvsc：避免使用 WRITE_SAME](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=3e8f4f4065901c8dfc51407e1984495e1748c090)
* [storvsc：針對 RAID 和虛擬主機介面卡驅動程式停用 WRITE SAME](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=54b2b50c20a61b51199bedb6e5d2f8ec2568fb43)
* [storvsc：NULL 指標取值修正](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=b12bb60d6c350b348a4e1460cd68f97ccae9822e)
* [storvsc：信號緩衝區失敗可能導致 I/O 凍結](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=e86fb5e8ab95f10ec5f2e9430119d5d35020c951)
* [scsi_sysfs︰防範 __scsi_remove_device 雙重執行](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/scsi_sysfs.c?id=be821fd8e62765de43cc4f0e2db363d0e30a7e9b)

## <a name="the-azure-linux-agent"></a>Azure Linux 代理程式
若要在 Azure 中正確佈建 Linux 虛擬機器， [Azure Linux 代理程式](../extensions/agent-linux.md) (waagent) 是必要選項。 您可以在 [Linux 代理程式 GitHub 儲存機制](https://github.com/Azure/WALinuxAgent)中取得最新的版本、檔案問題或提交提取要求。

* Linux 代理程式已在 Apache 2.0 授權下發行。 許多散發套件已提供代理程式的 RPM 或 Deb 套件，因此在某些情況下，您不費吹灰之力就可以安裝及更新此代理程式。
* Azure Linux 代理程式需要 Python v2.6+。
* 代理程式還需要 python-pyasn1 模組。 大多數的散發套件會以可個別安裝的套件形式提供此項目。
* 在某些情況下，Azure Linux 代理程式可能與 NetworkManager 不相容。 散發套件提供的許多 RPM/Deb 套件會將 NetworkManager 設定為 waagent 套件的衝突，因此會在您安裝 Linux 代理程式套件時解除安裝 NetworkManager。
* Azure Linux 代理程式必須高於最小支援版本，請參閱本文中的[詳細資料](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support)。

## <a name="general-linux-system-requirements"></a>一般的 Linux 系統需求

* 在 GRUB 或 GRUB2 中修改核心開機行，以加入下列參數。 這也會確保所有主控台訊息都會傳送到第一個序列埠，以協助 Azure 支援團隊進行問題偵錯程序：
  
        console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300
  
    這也會確保所有主控台訊息都會傳送到第一個序列埠，以協助 Azure 支援團隊進行問題偵錯程序。
  
    除了上述以外，我們還建議您 *移除* 下列參數 (如果存在的話)：
  
        rhgb quiet crashkernel=auto
  
    在雲端環境中，我們會將所有記錄傳送到序列埠，因此不適合使用圖形化和無訊息啟動。 如有需要，您可以保留 `crashkernel` 選項的設定，但請注意，此參數會減少 VM 中約 128MB 或以上的可用記憶體數量，這在較小的 VM 中可能會是個問題。

* 安裝 Azure Linux 代理程式
  
    如需在 Azure 上佈建 Linux 映像，您需要 Azure Linux 代理程式。  許多散發套件以 RPM 或 Deb 套件 (此套件通常稱為 'WALinuxAgent' 或 'walinuxagent') 的形式提供代理程式。  您也可以遵循 [Linux 代理程式指南](../extensions/agent-linux.md)中的步驟來手動安裝代理程式。

* 確定您已安裝 SSH 伺服器，並已設定為在開機時啟動。  這通常是預設值。

* 請不要在 OS 磁碟上建立交換空間
  
    Azure Linux 代理程式可在 VM 佈建於 Azure 後，使用附加至 VM 的本機資源磁碟自動設定交換空間。 本機資源磁碟是「暫存」磁碟，可能會在 VM 取消佈建時清空。 安裝 Azure Linux 代理程式 (請參閱上一個步驟) 後，請在 /etc/waagent.conf 中適當修改下列參數：
  
        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

* 最後一個步驟是，執行下列命令以取消佈建虛擬機器：
  
        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout
  
  > [!NOTE]
  > 在 Virtualbox 上，執行 'waagent -force -deprovision' 之後，您可能會看到以下錯誤：`[Errno 5] Input/output error`。 此錯誤訊息並不重要，您可以忽略。
  > 
  > 

* 關閉虛擬機器，並將 VHD 上傳至 Azure。

