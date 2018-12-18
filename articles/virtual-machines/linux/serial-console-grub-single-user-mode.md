---
title: 適用於 GRUB 與單一使用者模式的 Azure 序列主控台 | Microsoft Docs
description: 針對 Azure 虛擬機器中的 grub 使用序列主控台。
services: virtual-machines-linux
documentationcenter: ''
author: alsin
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/14/2018
ms.author: alsin
ms.openlocfilehash: 059cb0cbc7e62af16dbf95693be421feebcc1ee0
ms.sourcegitcommit: 76797c962fa04d8af9a7b9153eaa042cf74b2699
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42141726"
---
# <a name="use-serial-console-to-access-grub-and-single-user-mode"></a>使用序列主控台來存取 GRUB 與單一使用者模式
單一使用者模式是具有最基本功能的最基本環境。 它非常適用於調查開機問題或網路問題，因為背景只有基本的服務在執行，而且視執行層級而定，甚至可能不會自動掛載檔案系統。 它非常適用於調查檔案系統損毀、fstab 損毀或網路連線無法正確運作 (不正確的 iptable 設定) 等情況。

某些發行版本在 VM 無法開機時，會自動讓您進入單一使用者模式或緊急模式。 不過，其他發行版本要求您進行額外的設定，才會自動讓您進入單一使用者模式或緊急模式。

建議您確定已在您的 VM 上啟用 GRUB，以便存取單一使用者模式。 視您的發行版本而定，您可能必須完成一些設定工作，才能啟用 GRUB。 


## <a name="access-for-red-hat-enterprise-linux-rhel"></a>Red Hat Enterprise Linux (RHEL) 存取
RHEL 會在無法正常開機時讓您進入單一使用者模式。 不過，若您尚未設定單一使用者模式的開機存取，您將不會有 root 密碼，而且將無法登入。 有一個因應措施 (請參閱下面的＜手動進入單一使用者模式＞)，但還是建議您一開始就設定 root 存取。

### <a name="grub-access-in-rhel"></a>RHEL 中的 GRUB 存取
RHEL 預設會啟用 GRUB。 若要進入 GRUB，請使用 `sudo reboot` 將您的 VM 重新開機並按任意鍵。 您將會看到 GRUB 畫面。

> 注意：Red Hat 也提供開機到「救援模式」、「緊急模式」、「偵錯模式」以及重設 root 密碼的文件。 [按一下這裡以存取該文件](https://aka.ms/rhel7grubterminal)。

### <a name="set-up-root-access-for-single-user-mode-in-rhel"></a>在 RHEL 中針對單一使用者模式設定 root 存取
RHEL 中的單一使用者模式要求啟用 root 使用者 (預設為停用)。 如果您需要啟用單一使用者模式，請依下列指示進行操作：

1. 透過 SSH 登入 Red Hat 系統
1. 切換至 root
1. 啟用 root 使用者的密碼 
    * `passwd root` (設定強式 root 密碼)
1. 確定 root 使用者只能透過 ttyS0 進行登入
    * `edit /etc/ssh/sshd_config` 並確定將 PermitRootLogIn 設定為 no
    * `edit /etc/securetty file` 以只允許透過 ttyS0 進行登入 

現在，如果系統開機進入單一使用者模式，您將能夠透過 root 密碼進行登入。

或者，針對 RHEL 7.4 以上或 6.9 以上的版本，您也可以在 GRUB 提示字元中啟用單一使用者模式，請參閱[這裡](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/installation_guide/s1-rescuemode-booting-single)的指示

### <a name="manually-enter-single-user-mode-in-rhel"></a>在 RHEL 中手動進入單一使用者模式
若要根據上述指示設定 GRUB 與 root 存取，您可以使用下列指示來進入單一使用者模式：

1. 在重新啟動 VM 之後按 'Esc' 以進入 GRUB
1. 在 GRUB 中，按 'e' 以編輯您要開機進入其中的特定 OS (通常是第一行)
1. 尋找核心行，在 Azure 中，它的開頭是 `linux16`
1. 按 Ctrl + E 以移至行尾
1. 在行尾加入下列內容：`systemd.unit=rescue.target`
    * 這會讓您開機進入單一使用者模式。 若要使用緊急模式，請將 `systemd.unit=emergency.target` 加到行尾，而不是 `systemd.unit=rescue.target`
1. 按 Ctrl + X 以結束並以套用的設定重新開機
1. 系統會要求您輸入系統管理員密碼，才能進入單一使用者模式，此密碼與您在上述指示中建立的密碼相同    

    ![](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-rhel-enter-emergency-shell.gif)

### <a name="enter-single-user-mode-without-root-account-enabled-in-rhel"></a>在 RHEL 中於未啟用 root 帳戶的情況下進入單一使用者模式
若未依照上述步驟啟用 root 使用者，您仍然可以重設您的 root 密碼。 使用下列指示：

> 注意：若您使用 SELinux，當您重設 root 密碼之前，請確定您已執行[這裡](https://aka.ms/rhel7grubterminal)的 Red Hat 文件中所述的額外步驟。

1. 在重新啟動 VM 之後按 'Esc' 以進入 GRUB
1. 在 GRUB 中，按 'e' 以編輯您要開機進入其中的特定 OS (通常是第一行)
1. 尋找核心行，在 Azure 中，它的開頭是 `linux16`
1. 將 `rd.break` 加到行尾，並確定 `rd.break` 前面有一個空格 (請參閱下面的範例)
    - 這會在控制權從 `initramfs` 傳遞到 `systemd` 之前中斷開機程序，如[這裡](https://aka.ms/rhel7rootpassword) Red Hat 文件所述。
1. 按 Ctrl + X 以結束並以套用的設定重新開機
1. 開機之後，您會進入緊急模式，而且檔案系統是唯讀的。 在殼層中輸入 `mount -o remount,rw /sysroot` 以使用讀取/寫入權限重新掛載 root 檔案系統
1. 開機進入單一使用者模式之後，輸入 `chroot /sysroot` 以切換到 `sysroot` jail
1. 您現在是 root 了。 您可以使用 `passwd` 重設您的 root 密碼，然後使用上述指示來進入單一使用者模式。 完成之後，輸入 `reboot -f` 以重新開機。

![](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-rhel-emergency-mount-no-root.gif)

> 注意：執行上述指示將會進入緊急殼層，因此您也可以執行諸如編輯 `fstab` 的工作。 不過，一般公認建議是重設您的 root 密碼，並使用它來進入單一使用者模式。 


## <a name="access-for-centos"></a>CentOS 的存取
與 Red Hat Enterprise Linux 非常像，CentOS 中的單一使用者模式也要求必須啟用 GRUB 與 root 使用者。 

### <a name="grub-access-in-centos"></a>CentOS 中的 GRUB 存取
CentOS 預設會啟用 GRUB。 若要進入 GRUB，請使用 `sudo reboot` 將您的 VM 重新開機並按任意鍵。 您將會看到 GRUB 畫面。

### <a name="single-user-mode-in-centos"></a>CentOS 中的單一使用者模式
依照上述適用於 RHEL 的指示來在 CentOS 中啟用單一使用者模式。

## <a name="access-for-ubuntu"></a>Ubuntu 的存取 
Ubuntu 映像不需要 root 密碼。 如果系統開機進入單一使用者模式，您無須額外的認證即可存取。 

### <a name="grub-access-in-ubuntu"></a>Ubuntu 中的 GRUB 存取
若要存取 GRUB，請在 VM 開機時長按 'Esc'。

### <a name="single-user-mode-in-ubuntu"></a>Ubuntu 中的單一使用者模式
Ubuntu 會在無法正常開機時讓您進入單一使用者模式。 若要手動進入單一使用者模式，請依下列指示執行：

1. 從 GRUB，按 'e' 以編輯您的開機項目 (Ubuntu 項目)
1. 尋找開頭為 `linux` 的行，然後尋找 `ro`
1. 將 `single` 加到 `ro` 後方，並確定 `single` 前後都有一個空格
1. 按 Ctrl + X 以使用者些設定重新開機並進入單一使用者模式

## <a name="access-for-coreos"></a>CoreOS 的存取
CoreOS 中的單一使用者模式要求您必須啟用 GRUB。 

### <a name="grub-access-in-coreos"></a>CoreOS 中的 GRUB 存取
若要存取 GRUB，請在您的 VM 開機時按任意鍵。

### <a name="single-user-mode-in-coreos"></a>CoreOS 中的單一使用者模式
CoreOS 會在無法正常開機時讓您進入單一使用者模式。 若要手動進入單一使用者模式，請依下列指示執行：
1. 從 GRUB，按 'e' 以編輯您的開機項目
1. 尋找開頭為 `linux$` 的行。 應該有 2 個項目，包裝在在不同的 if/else 子句中
1. 將 `coreos.autologin=ttyS0` 附加到 2 個 `linux$` 行的結尾
1. 按 Ctrl + X 以使用者些設定重新開機並進入單一使用者模式

## <a name="access-for-suse-sles"></a>SUSE SLES 的存取
SLES 12 SP3 以上版本的新版映像允許在系統開機進入緊急模式時，透過序列主控台存取。 

### <a name="grub-access-in-suse-sles"></a>SUSE SLES 中的 GRUB 存取
SLES 中的 GRUB 存取要求必須透過 YaST 設定開機載入程式。 若要這樣做，請依照下列指示執行：

1. ssh 到您的 SLES VM 並執行 `sudo yast bootloader`。 使用 `tab` 鍵、`enter` 鍵與方向鍵來瀏覽功能表。 
1. 瀏覽到 `Kernel Parameters` `Use serial console`。 
1. 將 `serial --unit=0 --speed=9600 --parity=no` 加到 Console 引數中

1. 按 F10 以儲存您的設定並結束
1. 若要進入 GRUBM，請將您的 VM 重新開機並在開機順序期間按任意鍵以讓 GRUB 維持顯示在畫面上
    - GRUB 的預設逾時是 1 秒。 您可以透過變更 `/etc/default/grub` 中的 `GRUB_TIMEOUT` 變數以變更此時間長度

![](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-sles-yast-grub-config.gif)

### <a name="single-user-mode-in-suse-sles"></a>SUSE SLES 中的單一使用者模式
若 SLES 無法正常開機，將會自動讓您進入緊急殼層。 若要手動進入緊急殼層，請依下列指示執行：

1. 從 GRUB，按 'e' 以編輯您的開機項目 (SLES 項目)
1. 尋找核心行，它的開頭是 `linux`
1. 將 `systemd.unit=emergency.target` 附加到行尾
1. 按 Ctrl + X 以使用者些設定重新開機並進入緊急殼層
> 請注意，您會進入緊急殼層，而且檔案系統是_唯讀_的。 若要對任何檔案進行任何變更，您將必須使用讀取/寫入權限重新掛載檔案系統。 若要這樣做，請輸入 `mount -o remount,rw /` 到殼層

## <a name="access-for-oracle-linux"></a>Oracle Linux 的存取
與 Red Hat Enterprise Linux 非常像，Oracle Linux 中的單一使用者模式也要求必須啟用 GRUB 與 root 使用者。 

### <a name="grub-access-in-oracle-linux"></a>Oracle Linux 中的 GRUB 存取
Oracle Linux 預設會啟用 GRUB。 若要進入 GRUB，請使用 `sudo reboot` 將您的 VM 重新開機並按 'Esc'。 您將會看到 GRUB 畫面。

### <a name="single-user-mode-in-oracle-linux"></a>Oracle Linux 中的單一使用者模式
依照上述適用於 RHEL 的指示來在 Oracle Linux 中啟用單一使用者模式。

## <a name="next-steps"></a>後續步驟
* 主要序列主控台 Linux 文件頁面在[這裡](serial-console.md)。
* 適用於 [NMI 與 SysRq 呼叫](serial-console-nmi-sysrq.md)的序列主控台
* 序列主控台也適用於 [Windows](../windows/serial-console.md) VM
* 深入了解[開機診斷](boot-diagnostics.md)