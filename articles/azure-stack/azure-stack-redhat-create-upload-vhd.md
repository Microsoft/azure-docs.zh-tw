---
title: 建立及上傳 Red Hat Enterprise Linux VHD 以在 Azure Stack 中使用 | Microsoft Docs
description: 了解如何建立及上傳包含 Red Hat Linux 作業系統的 Azure 虛擬硬碟 (VHD)。
services: azure-stack
documentationcenter: ''
author: JeffGoldner
manager: BradleyB
editor: ''
tags: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: jeffgo
ms.openlocfilehash: d24902b894a632e9fe8c57f2fb2b652b44ab128c
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "41946497"
---
# <a name="prepare-a-red-hat-based-virtual-machine-for-azure-stack"></a>準備適用於 Azure Stack 的 Red Hat 型虛擬機器

在本文中，您將了解如何準備 Red Hat Enterprise Linux (RHEL) 虛擬機器以在 Azure Stack 中使用。 本文涵蓋的 RHEL 版本為 7.1+。 本文章所述之準備作業使用 Hyper-V、核心為基礎之虛擬機器 (KVM) 及 VMware 等 Hypervisor。

如需 Red Hat Enterprise Linux 支援資訊，請參閱 [Red Hat and Azure Stack: Frequently Asked Questions](https://access.redhat.com/articles/3413531) (Red Hat 和 Azure Stack：常見問題集)。

## <a name="prepare-a-red-hat-based-virtual-machine-from-hyper-v-manager"></a>從 Hyper-V 管理員準備 Red Hat 型虛擬機器

本節假設您已經從 Red Hat 網站取得 ISO 檔案並將 RHEL 映像安裝至虛擬硬碟 (VHD)。 如需有關如何使用 Hyper-V 管理員來安裝作業系統映像的詳細資訊，請參閱[安裝 Hyper-V 角色和設定虛擬機器](http://technet.microsoft.com/library/hh846766.aspx)。

### <a name="rhel-installation-notes"></a>RHEL 安裝注意事項

* Azure Stack 不支援 VHDX 格式。 Azure 只支援固定 VHD。 您可以使用 Hyper-V 管理員將磁碟轉換為 VHD 格式，或者使用 convert-vhd Cmdlet。 如果您使用 VirtualBox，請選取 [固定大小] ，而不是預設在建立磁碟時動態配置的選項。
* Azure Stack 僅支援第 1 代虛擬機器。 您可以將第 1 代虛擬機器從 VHDX 轉換為 VHD 檔案格式，並從動態擴充轉換為固定大小的磁碟。 您無法變更虛擬機器的世代。 如需詳細資訊，請參閱[應該在 Hyper-V 中建立第 1 代還是第 2 代的虛擬機器？](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)。
* 允許的 VHD 大小上限為 1,023 GB。
* 安裝 Linux 作業系統時，我們建議您使用標準磁碟分割而不是邏輯磁碟區管理員 (LVM)，其經常是許多安裝的預設設定。 此練習可避免 LVM 名稱與複製的虛擬機器發生衝突，特別是為了疑難排解而需要將作業系統磁碟連結至另一部相同虛擬機器時。
* 需要掛接通用磁碟格式 (UDF) 檔案系統的核心支援。 第一次開機時，連結至客體的 UDF 格式媒體會將佈建組態傳遞至 Linux 虛擬機器。 Azure Linux 代理程式必須掛接 UDF 檔案系統，才能讀取其組態並佈建虛擬機器。
* 請勿在作業系統磁碟上設定交換磁碟分割。 您可以設定 Linux 代理程式在暫存資源磁碟上建立交換檔。 在下列步驟中可以找到與此有關的詳細資訊。
* Azure 上的所有 VHD 必須具有與 1 MB 對應的虛擬大小。 從原始磁碟轉換成 VHD 時，您必須在轉換前先確定原始磁碟大小是 1 MB 的倍數。 如需更多詳細資料，請參閱以下步驟。
* Azure Stack 不支援 cloud-init。 您的 VM 必須設定為使用支援的 Windows Azure Linux 代理程式 (WALA) 版本。

### <a name="prepare-a-rhel-7-virtual-machine-from-hyper-v-manager"></a>從 Hyper-V 管理員準備 RHEL 7 虛擬機器

1. 在 Hyper-V 管理員中，選取虛擬機器。

1. 按一下 [連接] ，以開啟虛擬機器的主控台視窗。

1. 建立或編輯 `/etc/sysconfig/network` 檔案，然後新增下列文字：

    ```sh
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. 建立或編輯 `/etc/sysconfig/network-scripts/ifcfg-eth0` 檔案，然後視需要新增下列文字：

    ```sh
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. 若要確保開機時會啟動網路服務，可執行下列命令：

    ```bash
    sudo systemctl enable network
    ```

1. 透過執行以下命令來註冊 Red Hat 訂用帳戶，以便從 RHEL 儲存機制安裝封裝：

    ```bash
    sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. 修改 grub 組態中的核心開機那一行，使其額外包含用於 Azure 的核心參數。 若要執行此修改，請在文字編輯器中開啟 `/etc/default/grub`，然後修改 `GRUB_CMDLINE_LINUX` 參數。 例如︰

    ```sh
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   這會確保所有主控台訊息都會傳送到第一個序列埠，以協助 Azure 支援團隊進行問題偵錯程序。 此組太也會關閉新的 RHEL 7 對 NIC 的命名慣例。

   在雲端環境中，我們會將所有記錄傳送到序列埠，因此不適合使用圖形化和無訊息啟動。 您可以視需要保留 `crashkernel` 選項的設定。 請注意，此參數會減少虛擬機器中約 128 MB 或以上的可用記憶體數量，這可能會對小型虛擬機器造成問題。 我們建議移除下列參數：

    ```sh
    rhgb quiet crashkernel=auto
    ```

1. 完成 `/etc/default/grub`的編輯之後，請執行下列命令以重建 grub 組態：

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. 確定已安裝 SSH 伺服器，並已設定為在開機時啟動 (這通常為預設值)。 修改 `/etc/ssh/sshd_config` 以包含下面一行：

    ```sh
    ClientAliveInterval 180
    ```

1. WALinuxAgent 套件 `WALinuxAgent-<version>` 已推送至 Red Hat extras 儲存機制。 執行下列命令以啟用 extras 儲存機制：

    ```bash
    subscription-manager repos --enable=rhel-7-server-extras-rpms
    ```

1. 執行以下命令來安裝 Azure Linux 代理程式：

    ```bash
    sudo yum install WALinuxAgent
    sudo systemctl enable waagent.service
    ```

1. 請勿在作業系統磁碟上建立交換空間。

    Azure Linux 代理程式可在虛擬機器佈建於 Azure 後，使用連結至虛擬機器的本機資源磁碟自動設定交換空間。 本機資源磁碟是暫存磁碟，可能會在虛擬機器取消佈建時清空。 在上一個步驟安裝 Azure Linux 代理程式後，請在 `/etc/waagent.conf` 中適當修改下列參數：

    ```sh
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    #NOTE: set this to whatever you need it to be.
    ```

1. 如果您要取消註冊訂用帳戶，請執行下列命令：

    ```bash
    sudo subscription-manager unregister
    ```

1. 如果您使用的系統是使用企業憑證授權單位來部署，RHEL 虛擬機器將不會信任 Azure Stack 根憑證。 您需要將它放入信任的根存放區。 請參閱[將信任的根憑證新增到伺服器](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html)。

1. 執行下列命令，以取消佈建虛擬機器，並準備將它佈建於 Azure 上：

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. 在 Hyper-V 管理員中，按一下 [動作] > [關閉]。

1. 使用 Hyper-V 管理員的 [編輯磁碟] 功能或 Convert-VHD PowerShell 命令，將 VHD 轉換為固定大小 VHD。 您現在可以將 Linux VHD 上傳至 Azure。

## <a name="prepare-a-red-hat-based-virtual-machine-from-kvm"></a>從 KVM 準備 Red Hat 型虛擬機器

1. 從 Red Hat 網站下載 RHEL 7 的 KVM 映像。 此程序會使用 RHEL 7 做為範例。

1. 設定根密碼。

    產生加密的密碼，並複製命令的輸出：

    ```bash
    openssl passwd -1 changeme
    ```

   使用 guestfish 設定根密碼：

    ```sh
    guestfish --rw -a <image-name>
    > <fs> run
    > <fs> list-filesystems
    > <fs> mount /dev/sda1 /
    > <fs> vi /etc/shadow
    > <fs> exit
    ```

   將根使用者的第二個欄位從 "!!" 變更為加密的密碼。

1. 在 KVM 中從 qcow2 映像建立虛擬機器。 將磁碟類型設定為 **qcow2**，再將虛擬網路介面裝置模型設定為 **virtio**。 然後啟動虛擬機器並以 root 身分登入。

1. 建立或編輯 `/etc/sysconfig/network` 檔案，然後新增下列文字：

    ```sh
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. 建立或編輯 `/etc/sysconfig/network-scripts/ifcfg-eth0` 檔案，然後新增下列文字：

    ```sh
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. 若要確保開機時會啟動網路服務，可執行下列命令：

    ```bash
    sudo systemctl enable network
    ```

1. 透過執行以下命令來註冊 Red Hat 訂用帳戶，以便從 RHEL 儲存機制安裝封裝：

    ```bash
    subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. 修改 grub 組態中的核心開機那一行，使其額外包含用於 Azure 的核心參數。 若要進行此設定，請在文字編輯器中開啟 `/etc/default/grub`，然後修改 `GRUB_CMDLINE_LINUX` 參數。 例如︰

    ```sh
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   此命令也會確保所有主控台訊息都會傳送給第一個序列埠，以協助 Azure 支援團隊進行問題偵錯程序。 而且也會關閉新的 RHEL 7 對 NIC 的命名慣例。

   在雲端環境中，所有記錄都會傳送到序列埠，因此不適合使用圖形化和無訊息啟動。 您可以視需要保留 `crashkernel` 選項的設定。 此參數會減少虛擬機器中約 128 MB 或以上的可用記憶體數量，這可能會對小型虛擬機器造成問題。 我們建議移除下列參數：

    ```sh
    rhgb quiet crashkernel=auto
    ```

1. 完成 `/etc/default/grub`的編輯之後，請執行下列命令以重建 grub 組態：

    ```bash
    grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. 將 Hyper-V 模組新增至 initramfs。

    編輯 `/etc/dracut.conf` 並加入內容：

    ```sh
    add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
    ```

    重建 initramfs：

    ```bash
    dracut -f -v
    ```

1. 解除安裝 cloud-init：

    ```bash
    yum remove cloud-init
    ```

1. 確定您已安裝 SSH 伺服器，並已設定為在開機時啟動：

    ```bash
    systemctl enable sshd
    ```

    修改 /etc/ssh/sshd_config 以包含下面一行：

    ```sh
    PasswordAuthentication yes
    ClientAliveInterval 180
    ```

1. WALinuxAgent 套件 `WALinuxAgent-<version>` 已推送至 Red Hat extras 儲存機制。 執行下列命令以啟用 extras 儲存機制：

    ```bash
    subscription-manager repos --enable=rhel-7-server-extras-rpms
    ```

1. 執行以下命令來安裝 Azure Linux 代理程式：

    ```bash
    yum install WALinuxAgent
    ```

    啟用 waagent 服務：

    ```bash
    systemctl enable waagent.service
    ```

1. 請勿在作業系統磁碟上建立交換空間。

    Azure Linux 代理程式可在虛擬機器佈建於 Azure 後，使用連結至虛擬機器的本機資源磁碟自動設定交換空間。 本機資源磁碟是暫存磁碟，可能會在虛擬機器取消佈建時清空。 在上一個步驟安裝 Azure Linux 代理程式後，請在 `/etc/waagent.conf` 中適當修改下列參數：

    ```sh
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    #NOTE: set this to whatever you need it to be.
    ```

1. 執行下列命令以取消註冊訂用帳戶 (如有必要)：

    ```bash
    subscription-manager unregister
    ```

1. 如果您使用的系統是使用企業憑證授權單位來部署，RHEL 虛擬機器將不會信任 Azure Stack 根憑證。 您需要將它放入信任的根存放區。 請參閱[將信任的根憑證新增到伺服器](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html)。

1. 執行下列命令，以取消佈建虛擬機器，並準備將它佈建於 Azure 上：

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. 在 KVM 中關閉虛擬機器。

1. 將 qcow2 映像轉換成 VHD 格式。

    > [!NOTE]
    > qemu-img >=2.2.1 的版本中已知有 Bug 會導致 VHD 的格式不正確。 此問題已在 QEMU 2.6 中修正。 建議使用 qemu-img 2.2.0 或更舊版本，或更新至 2.6 或更新版本。 參考： https://bugs.launchpad.net/qemu/+bug/1490611。

    先將映像轉換成原始格式：

    ```bash
    qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw
    ```

    確認原始映像的大小符合 1 MB， 否則將大小四捨五入為 1 MB︰

    ```bash
    MB=$((1024*1024))
    size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
    gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
    rounded_size=$((($size/$MB + 1)*$MB))
    qemu-img resize rhel-7.4.raw $rounded_size
    ```

    將原始磁碟轉換成固定大小的 VHD：

    ```bash
    qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

    或者，qemu 版本 **2.6 +** 包含 `force_size` 選項︰

    ```bash
    qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

## <a name="prepare-a-red-hat-based-virtual-machine-from-vmware"></a>從 VMware 準備 Red Hat 型虛擬機器

本節假設您已在 VMware 中安裝 RHEL 虛擬機器。 如需有關如何在 VMware 中安裝作業系統的詳細資訊，請參閱 [VMware 客體作業系統安裝指南](http://partnerweb.vmware.com/GOSIG/home.html)。

* 安裝 Linux 作業系統時，我們建議您使用標準磁碟分割而不是 LVM (這通常是許多安裝的預設設定)。 這可避免 LVM 名稱與複製的虛擬機器發生衝突，特別是為了疑難排解而需要將作業系統磁碟連結至另一部虛擬機器時。 如果願意，您可以在資料磁碟上使用 LVM 或 RAID。
* 請勿在作業系統磁碟上設定交換磁碟分割。 您可以設定 Linux 代理程式以在暫存資源磁碟上建立交換檔。 您可以在以下步驟中找到與此相關的詳細資訊。
* 建立虛擬硬碟時，請選取 [將虛擬磁碟儲存為單一檔案] 。

### <a name="prepare-a-rhel-7-virtual-machine-from-vmware"></a>從 VMWare 準備 RHEL 7 虛擬機器

1. 建立或編輯 `/etc/sysconfig/network` 檔案，然後新增下列文字：

    ```sh
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. 建立或編輯 `/etc/sysconfig/network-scripts/ifcfg-eth0` 檔案，然後新增下列文字：

    ```sh
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

1. 若要確保開機時會啟動網路服務，可執行下列命令：

    ```bash
    sudo chkconfig network on
    ```

1. 透過執行以下命令來註冊 Red Hat 訂用帳戶，以便從 RHEL 儲存機制安裝封裝：

    ```bash
    sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. 修改 grub 組態中的核心開機那一行，使其額外包含用於 Azure 的核心參數。 若要執行此修改，請在文字編輯器中開啟 `/etc/default/grub`，然後修改 `GRUB_CMDLINE_LINUX` 參數。 例如︰

    ```sh
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

    此組態也會確保所有主控台訊息都會傳送給第一個序列埠，以協助 Azure 支援團隊進行問題偵錯程序。 也會關閉新的 RHEL 7 對 NIC 的命名慣例。 此外，我們還建議您移除下列參數：

    ```sh
    rhgb quiet crashkernel=auto
    ```

    在雲端環境中，我們會將所有記錄傳送到序列埠，因此不適合使用圖形化和無訊息啟動。 您可以視需要保留 `crashkernel` 選項的設定。 請注意，此參數會減少虛擬機器中約 128 MB 或以上的可用記憶體數量，這可能會對小型虛擬機器造成問題。

1. 完成 `/etc/default/grub`的編輯之後，請執行下列命令以重建 grub 組態：

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. 將 Hyper-V 模組新增至 initramfs。

    編輯 `/etc/dracut.conf`，新增內容：

    ```sh
    add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
    ```

    重建 initramfs：

    ```bash
    dracut -f -v
    ```

1. 確定您已安裝 SSH 伺服器，並已設定為在開機時啟動。 此設定通常是預設值。 修改 `/etc/ssh/sshd_config` 以包含下面一行：

    ```sh
    ClientAliveInterval 180
    ```

1. WALinuxAgent 套件 `WALinuxAgent-<version>` 已推送至 Red Hat extras 儲存機制。 執行下列命令以啟用 extras 儲存機制：

    ```bash
    subscription-manager repos --enable=rhel-7-server-extras-rpms
    ```

1. 執行以下命令來安裝 Azure Linux 代理程式：

    ```bash
    sudo yum install WALinuxAgent
    sudo systemctl enable waagent.service
    ```

1. 請勿在作業系統磁碟上建立交換空間。

    Azure Linux 代理程式可在虛擬機器佈建於 Azure 後，使用連結至虛擬機器的本機資源磁碟自動設定交換空間。 請注意，本機資源磁碟是暫存磁碟，可能會在虛擬機器取消佈建時清空。 在上一個步驟安裝 Azure Linux 代理程式後，請在 `/etc/waagent.conf` 中適當修改下列參數：

    ```sh
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    NOTE: set this to whatever you need it to be.
    ```

1. 如果您要取消註冊訂用帳戶，請執行下列命令：

    ```bash
    sudo subscription-manager unregister
    ```

1. 如果您使用的系統是使用企業憑證授權單位來部署，RHEL 虛擬機器將不會信任 Azure Stack 根憑證。 您需要將它放入信任的根存放區。 請參閱[將信任的根憑證新增到伺服器](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html)。

1. 執行下列命令，以取消佈建虛擬機器，並準備將它佈建於 Azure 上：

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

1. 關閉虛擬機器，然後將 VMDK 檔案轉換成 VHD 格式。

    > [!NOTE]
    > qemu-img >=2.2.1 的版本中已知有 Bug 會導致 VHD 的格式不正確。 此問題已在 QEMU 2.6 中修正。 建議使用 qemu-img 2.2.0 或更舊版本，或更新至 2.6 或更新版本。 參考： <https://bugs.launchpad.net/qemu/+bug/1490611>。

    先將映像轉換成原始格式：

    ```bash
    qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw
    ```

    確認原始映像的大小符合 1 MB， 否則將大小四捨五入為 1 MB︰

    ```bash
    MB=$((1024*1024))
    size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
    gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
    rounded_size=$((($size/$MB + 1)*$MB))
    qemu-img resize rhel-7.4.raw $rounded_size
    ```

    將原始磁碟轉換成固定大小的 VHD：

    ```bash
    qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

    或者，qemu 版本 **2.6 +** 包含 `force_size` 選項︰

    ```bash
    qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

## <a name="prepare-a-red-hat-based-virtual-machine-from-an-iso-by-using-a-kickstart-file-automatically"></a>使用 kickstart 檔案自動從 ISO 準備 Red Hat 型虛擬機器

1. 建立包含以下內容的 kickstart 檔案，然後儲存此檔案。 如需有關 kickstart 安裝的詳細資訊，請參閱 [Kickstart 安裝指南](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html)。

    ```sh
    Kickstart for provisioning a RHEL 7 Azure VM

    System authorization information
    auth --enableshadow --passalgo=sha512

    Use graphical install
    text

    Do not run the Setup Agent on first boot
    firstboot --disable

    Keyboard layouts
    keyboard --vckeymap=us --xlayouts='us'

    System language
    lang en_US.UTF-8

    Network information
    network  --bootproto=dhcp

    Root password
    rootpw --plaintext "to_be_disabled"

    System services
    services --enabled="sshd,waagent,NetworkManager"

    System timezone
    timezone Etc/UTC --isUtc --ntpservers 0.rhel.pool.ntp.org,1.rhel.pool.ntp.org,2.rhel.pool.ntp.org,3.rhel.pool.ntp.org

    Partition clearing information
    clearpart --all --initlabel

    Clear the MBR
    zerombr

    Disk partitioning information
    part /boot --fstype="xfs" --size=500
    part / --fstyp="xfs" --size=1 --grow --asprimary

    System bootloader configuration
    bootloader --location=mbr

    Firewall configuration
    firewall --disabled

    Enable SELinux
    selinux --enforcing

    Don't configure X
    skipx

    Power down the machine after install
    poweroff

    %packages
    @base
    @console-internet
    chrony
    sudo
    parted
    -dracut-config-rescue

    %end

    %post --log=/var/log/anaconda/post-install.log

    #!/bin/bash

    Register Red Hat Subscription
    subscription-manager register --username=XXX --password=XXX --auto-attach --force

    Install latest repo update
    yum update -y

    Enable extras repo
    subscription-manager repos --enable=rhel-7-server-extras-rpms

    Install WALinuxAgent
    yum install -y WALinuxAgent

    Unregister Red Hat subscription
    subscription-manager unregister

    Enable waaagent at boot-up
    systemctl enable waagent

    Disable the root account
    usermod root -p '!!'

    Configure swap in WALinuxAgent
    sed -i 's/^\(ResourceDisk\.EnableSwap\)=[Nn]$/\1=y/g' /etc/waagent.conf
    sed -i 's/^\(ResourceDisk\.SwapSizeMB\)=[0-9]*$/\1=2048/g' /etc/waagent.conf

    Set the cmdline
    sed -i 's/^\(GRUB_CMDLINE_LINUX\)=".*"$/\1="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"/g' /etc/default/grub

    Enable SSH keepalive
    sed -i 's/^#\(ClientAliveInterval\).*$/\1 180/g' /etc/ssh/sshd_config

    Build the grub cfg
    grub2-mkconfig -o /boot/grub2/grub.cfg

    Configure network
    cat << EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    EOF

    Deprovision and prepare for Azure
    waagent -force -deprovision

    %end
    ```

1. 將 kickstart 檔案放在可存取它的安裝系統。

1. 在 Hyper-V 管理員中建立新的虛擬機器。 在 [連線虛擬硬碟] 頁面上，選取 [稍後連接虛擬硬碟]，並完成 [新增虛擬機器精靈]。

1. 開啟虛擬機器設定：

    a. 將新的虛擬硬碟連結至虛擬機器。 請務必選取 [VHD 格式] 和 [固定大小]。

    b. 將安裝 ISO 連接到 DVD 光碟機。

    c. 將 BIOS 設定成從 CD 開機。

1. 啟動虛擬機器。 當安裝指南出現時，請按 **Tab** 鍵以設定開機選項。

1. 在開機選項結尾輸入 `inst.ks=<the location of the kickstart file>` ，然後按 **Enter**鍵。

1. 等待安裝完成。 完成後，虛擬機器會自動關閉。 您現在可以將 Linux VHD 上傳至 Azure。

## <a name="known-issues"></a>已知問題

### <a name="the-hyper-v-driver-could-not-be-included-in-the-initial-ram-disk-when-using-a-non-hyper-v-hypervisor"></a>使用非 Hyper-V Hypervisor 時，初始 RAM 磁碟未包含 Hyper-V 驅動程式

在某些情況下，Linux 安裝程式可能不會在初始 RAM 磁碟 (initrd 或 initramfs) 中包含 Hyper-V 的驅動程式，除非 Linux 偵測到自己在 Hyper-V 環境中執行。

使用不同的虛擬化系統 (即 Virtualbox、Xen 等) 來準備 Linux 映像時，您可能需要重新建置 initrd，以確保在初始 RAM 磁碟上至少有 hv_vmbus 和 hv_storvsc 核心模組可以使用。 目前至少已知在以上游 Red Hat 散發套件為基礎的系統上有此問題。

若要解決這個問題，請將 Hyper-V 模組新增至 initramfs 並加以重建︰

編輯 `/etc/dracut.conf` 檔案並新增下列內容：

    ```sh
    add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
    ```

重建 initramfs：

    ```bash
    dracut -f -v
    ```

如需詳細資訊，請參閱[重建 initramfs](https://access.redhat.com/solutions/1958)。

## <a name="next-steps"></a>後續步驟

您現在可以開始使用您的 Red Hat Enterprise Linux 虛擬硬碟在 Azure Stack 建立新的虛擬機器。 如果您是第一次將 VHD 檔案上傳至 Azure Stack，請參閱[使用 Marketplace 工具組建立及發行 Marketplace 項目](azure-stack-marketplace-publisher.md)。

如需已通過認證可執行 Red Hat Enterprise Linux 的 Hypervisor 詳細資訊，請參閱 [Red Hat 網站](https://access.redhat.com/certified-hypervisors)。
