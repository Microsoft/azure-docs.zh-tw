---
title: 在 Azure 中建立及上傳 CentOS 型 Linux VHD
description: 了解如何建立及上傳包含 CentOS 型 Linux 作業系統的 Azure 虛擬硬碟 (VHD)。
services: virtual-machines-linux
documentationcenter: ''
author: szarkos
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager,azure-service-management
ms.assetid: 0e518e92-e981-43f4-b12c-9cba1064c4bb
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/04/2018
ms.author: szark
ms.openlocfilehash: d7c35b79dcdf75dbb3f891dc4c66cbf893b61c03
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33777683"
---
# <a name="prepare-a-centos-based-virtual-machine-for-azure"></a>準備適用於 Azure 的 CentOS 型虛擬機器
* [準備適用於 Azure 的 CentOS 6.x 虛擬機器](#centos-6x)
* [準備適用於 Azure 的 CentOS 7.0+ 虛擬機器](#centos-70)

[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

## <a name="prerequisites"></a>先決條件
本文假設您已將 CentOS (或類似的衍生物件) Linux 作業系統安裝到虛擬硬碟。 有多個工具可用來建立 .vhd 檔案，例如，像是 Hyper-V 的虛擬化解決方案。 如需指示，請參閱 [安裝 Hyper-V 角色及設定虛擬機器](http://technet.microsoft.com/library/hh846766.aspx)。

**CentOS 安裝注意事項**

* 另請參閱[一般 Linux 安裝注意事項](create-upload-generic.md#general-linux-installation-notes)，了解為 Azure 準備 Linux 的更多祕訣。
* Azure 不支援 VHDX 格式，只支援 **固定 VHD**。  您可以使用 Hyper-V 管理員或 convert-vhd Cmdlet，將磁碟轉換為 VHD 格式。 如果您是使用 VirtualBox，即會在建立磁碟時選取 [固定大小]  而不是預設的動態配置。
* 安裝 Linux 系統時，*建議*您使用標準磁碟分割而不是 LVM (常是許多安裝的預設設定)。 這可避免 LVM 與複製之 VM 的名稱衝突，特別是為了疑難排解而需要將作業系統磁碟連接至另一個相同的 VM 時。 如果願意，您可以在資料磁碟上使用 [LVM](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 或 [RAID](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
* 需要掛接 UDF 檔案系統的核心支援。 在 Azure 上第一次開機時，佈建組態會透過連接客體的 UDF 格式媒體傳遞至 Linux VM。 Azure Linux 代理程式必須能夠掛接 UDF 檔案系統讀取其組態並佈建 VM。
* Linux Kernel 2.6.37 以下的版本不支援較大 VM 大小 Hyper-V 上的NUMA。 這個問題主要會影響使用上游 Red Hat 2.6.32 kernel 的較舊散發套件，RHEL 6.6 (kernel-2.6.32-504) 已加以修正。 執行的自訂核心是 2.6.37 以前版本的系統，或 2.6.32-504 以前以 RHEL 為基礎的核心必須在 grub.conf 的核心命令列上設定開機參數 `numa=off`。 如需詳細資訊，請參閱 Red Hat [KB 436883](https://access.redhat.com/solutions/436883)。
* 請勿在作業系統磁碟上設定交換磁碟分割。 您可以設定 Linux 代理程式在暫存資源磁碟上建立交換檔。  您可以在以下步驟中找到與此有關的詳細資訊。
* Azure 上的所有 VHD 必須具有與 1 MB 對應的虛擬大小。 從未經處理的磁碟轉換成 VHD 時，您必須確定未經處理的磁碟大小在轉換前是 1 MB 的倍數。 如需詳細資訊，請參閱 [Linux 安裝注意事項](create-upload-generic.md#general-linux-installation-notes)。

## <a name="centos-6x"></a>CentOS 6.x

1. 在 Hyper-V 管理員中，選取虛擬機器。

2. 按一下 [連接] ，以開啟虛擬機器的主控台視窗。

3. 在 CentOS 6 中，NetworkManager 可能會對 Azure Linux 代理程式造成干擾。 執行下列命令以將此套件解除安裝：
   
        # sudo rpm -e --nodeps NetworkManager

4. 建立或編輯檔案 `/etc/sysconfig/network` 並新增下列文字：
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. 建立或編輯檔案 `/etc/sysconfig/network-scripts/ifcfg-eth0` 並新增下列文字：
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

6. 修改 udev 角色可防止產生乙太網路介面的靜態規則。 在 Microsoft Azure 或 Hyper-V 中複製虛擬機器時，這些規則可能會造成問題：
   
        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
        # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

7. 要確保開機時會啟動網路服務，可執行以下命令：
   
        # sudo chkconfig network on

8. 如果您想要使用在 Azure 資料中心內託管的 OpenLogic 鏡像，請使用下列存放庫來取代 `/etc/yum.repos.d/CentOS-Base.repo` 檔案。  這也會新增包括額外套件 (例如 Azure Linux 代理程式) 的 **[openlogic]** 存放庫：

        [openlogic]
        name=CentOS-$releasever - openlogic packages for $basearch
        baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
        enabled=1
        gpgcheck=0
        
        [base]
        name=CentOS-$releasever - Base
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
        
        #released updates
        [updates]
        name=CentOS-$releasever - Updates
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
        
        #additional packages that may be useful
        [extras]
        name=CentOS-$releasever - Extras
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

        #additional packages that extend functionality of existing packages
        [centosplus]
        name=CentOS-$releasever - Plus
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
        
        #contrib - packages by Centos Users
        [contrib]
        name=CentOS-$releasever - Contrib
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

    >[!Note]
    本指南的其他部分將假設您至少會使用 `[openlogic]` 存放庫，該存放庫稍後將可用來安裝下面的 Azure Linux 代理程式。


9. 在 /etc/yum.conf 中加入這一行：
    
        http_caching=packages

10. 執行下列命令，以清除目前的 yum 中繼資料並使用最新的套件來更新系統：
    
        # yum clean all

    除非您要為舊版 CentOS 建立映像，否則建議您將所有套件更新到最新版本：

        # sudo yum -y update

    執行此命令之後可能需要重新開機。

11. (選擇性) 安裝 Linux 整合服務 (LIS) 的驅動程式。
   
    >[!IMPORTANT]
    此步驟對 CentOS 6.3 與更舊版本而言為**必要步驟**，但對於較新的版本而言則為選擇性步驟。

        # sudo rpm -e hypervkvpd  ## (may return error if not installed, that's OK)
        # sudo yum install microsoft-hyper-v

    或者，您可以依照 [LIS 下載頁面](https://go.microsoft.com/fwlink/?linkid=403033)上的手動安裝指示執行，以在您的 VM 上安裝該 RPM。
 
12. 安裝 Azure Linux 代理程式與相依性：
    
        # sudo yum install python-pyasn1 WALinuxAgent
    
    如果未如步驟 3 所述移除 NetworkManager 和 NetworkManager-gnome 套件，則 WALinuxAgent 套件會將這兩個套件移除。


13. 修改 grub 組態中的核心開機那一行，使其額外包含用於 Azure 的核心參數。 若要這樣做，請在文字編輯器中開啟 `/boot/grub/menu.lst`，並確定預設核心包含以下參數：
    
        console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    
    這也將確保所有主控台訊息都會傳送給第一個序列埠，有助於 Azure 支援團隊進行問題偵錯程序。
    
    除了上述以外，我們還建議您 *移除* 下列參數：
    
        rhgb quiet crashkernel=auto
    
    在雲端環境中，我們會將所有記錄傳送到序列埠，因此不適合使用圖形化和無訊息啟動。  如有需要，您可以保留 `crashkernel` 選項的設定，但請注意，此參數將會減少 VM 中約 128MB 或以上的可用記憶體數量，這在較小的 VM 中可能會是個問題。

    >[!Important]
    CentOS 6.5 與較舊版本也必須設定核心參數 `numa=off`。 請參閱 Red Hat [KB 436883](https://access.redhat.com/solutions/436883)。

14. 確定您已安裝 SSH 伺服器，並已設定為在開機時啟動。  這通常是預設值。

15. 請勿在作業系統磁碟上建立交換空間。
    
    Azure Linux 代理程式可在 VM 佈建於 Azure 後，使用附加至 VM 的本機資源磁碟自動設定交換空間。 請注意，資源磁碟是 *暫存* 磁碟，可能會在 VM 取消佈建時清空。 安裝 Azure Linux 代理程式 (請參閱上一個步驟) 後，請在 `/etc/waagent.conf` 中適當修改下列參數：
    
        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16. 執行下列命令，以取消佈建虛擬機器，並準備將它佈建於 Azure 上：
    
        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

17. 在 Hyper-V 管理員中，依序按一下 [動作] -> [關閉]。 您現在可以將 Linux VHD 上傳至 Azure。


- - -
## <a name="centos-70"></a>CentOS 7.0+
**CentOS 7 (和類似的衍生項目) 中的變更**

準備適用於 Azure 的 CentOS 7 虛擬機器會與 CentOS 6 極為類似，不過，其中有幾個重要差異值得注意：

* NetworkManager 封裝不會再與 Azure Linux 代理程式發生衝突。 依預設會安裝此封裝，建議您不要將它移除。
* GRUB2 現已作為預設的開機載入器使用，因此我們已變更編輯核心參數的程序 (如下所示)。
* XFS 現為預設的檔案系統。 如有需要，您仍可使用 ext4 檔案系統。

**組態步驟**

1. 在 Hyper-V 管理員中，選取虛擬機器。

2. 按一下 [連接] ，以開啟虛擬機器的主控台視窗。

3. 建立或編輯檔案 `/etc/sysconfig/network` 並新增下列文字：
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

4. 建立或編輯檔案 `/etc/sysconfig/network-scripts/ifcfg-eth0` 並新增下列文字：
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

5. 修改 udev 角色可防止產生乙太網路介面的靜態規則。 在 Microsoft Azure 或 Hyper-V 中複製虛擬機器時，這些規則可能會造成問題：
   
        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

6. 如果您想要使用在 Azure 資料中心內託管的 OpenLogic 鏡像，請使用下列存放庫來取代 `/etc/yum.repos.d/CentOS-Base.repo` 檔案。  這也會新增包含 Azure Linux 代理程式封裝的 **[openlogic]** 儲存機制：
   
        [openlogic]
        name=CentOS-$releasever - openlogic packages for $basearch
        baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
        enabled=1
        gpgcheck=0
        
        [base]
        name=CentOS-$releasever - Base
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
        
        #released updates
        [updates]
        name=CentOS-$releasever - Updates
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
        
        #additional packages that may be useful
        [extras]
        name=CentOS-$releasever - Extras
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
        
        #additional packages that extend functionality of existing packages
        [centosplus]
        name=CentOS-$releasever - Plus
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

    >[!Note]
    本指南的其他部分將假設您至少會使用 `[openlogic]` 存放庫，該存放庫稍後將可用來安裝下面的 Azure Linux 代理程式。

7. 執行下列命令，以清除目前的 yum 中繼資料並安裝任何更新：
   
        # sudo yum clean all

    除非您要為舊版 CentOS 建立映像，否則建議您將所有套件更新到最新版本：

        # sudo yum -y update

    執行此命令之後可能需要重新開機。

8. 修改 grub 組態中的核心開機那一行，使其額外包含用於 Azure 的核心參數。 若要這樣做，請在文字編輯器中開啟 `/etc/default/grub` 並編輯 `GRUB_CMDLINE_LINUX` 參數，例如：
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
   這也將確保所有主控台訊息都會傳送給第一個序列埠，有助於 Azure 支援團隊進行問題偵錯程序。 也會關閉新的 CentOS 7 對 NIC 的命名慣例。 除了上述以外，我們還建議您 *移除* 下列參數：
   
        rhgb quiet crashkernel=auto
   
    在雲端環境中，我們會將所有記錄傳送到序列埠，因此不適合使用圖形化和無訊息啟動。 如有需要，您可以保留 `crashkernel` 選項的設定，但請注意，此參數將會減少 VM 中約 128MB 或以上的可用記憶體數量，這在較小的 VM 中可能會是個問題。

9. 在您參照上述完成編輯 `/etc/default/grub` 之後，請執行下列命令以重建 grub 組態：
   
        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

10. 若要從 **VMWare、VirtualBox 或 KVM 建置映像：** 請確定 initramfs 中已包括 Hyper-V 驅動程式：
   
   編輯 `/etc/dracut.conf`，新增內容：
   
        add_drivers+=”hv_vmbus hv_netvsc hv_storvsc”
   
   重建 initramfs：
   
        # sudo dracut -f -v

11. 安裝 Azure Linux 代理程式與相依性：

        # sudo yum install python-pyasn1 WALinuxAgent
        # sudo systemctl enable waagent

12. 請勿在作業系統磁碟上建立交換空間。
   
   Azure Linux 代理程式可在 VM 佈建於 Azure 後，使用附加至 VM 的本機資源磁碟自動設定交換空間。 請注意，資源磁碟是 *暫存* 磁碟，可能會在 VM 取消佈建時清空。 安裝 Azure Linux 代理程式 (請參閱上一個步驟) 後，請在 `/etc/waagent.conf` 中適當修改下列參數：
   
        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13. 執行下列命令，以取消佈建虛擬機器，並準備將它佈建於 Azure 上：
   
        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

14. 在 Hyper-V 管理員中，依序按一下 [動作] -> [關閉]。 您現在可以將 Linux VHD 上傳至 Azure。

## <a name="next-steps"></a>後續步驟
您現在可以開始使用您的 CentOS Linux 虛擬硬碟在 Azure 建立新的虛擬機器。 如果您是第一次將 .vhd 檔案上傳至 Azure，請參閱[從自訂磁碟建立 Linux VM](upload-vhd.md#option-1-upload-a-vhd)。

