---
title: 適用於 SysRq 和 NMI 呼叫的 Azure 序列主控台 | Microsoft Docs
description: 在 Azure 虛擬機器中使用適用於 SysRq 和 NMI 呼叫的序列主控台。
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
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
ms.openlocfilehash: ac3e01756ad66dcb44869556bb103eb20bc2658c
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221480"
---
# <a name="use-serial-console-for-sysrq-and-nmi-calls"></a>使用適用於 SysRq 和 NMI 呼叫的序列主控台

## <a name="system-request-sysrq"></a>系統要求 (SysRq)
SysRq 是可讓 Linux 作業系統核心理解的按鍵序列，可觸發一組預先定義的動作。 這些命令通常會用在虛擬機器的疑難排解上，或用在無法透過傳統系統管理執行的復原上 (例如，VM 終止時)。 使用 Azure 序列主控台的 SysRq 功能，就像按下 SysRq 鍵及在實體鍵盤上輸入的字元。

SysRq 序列傳遞出去後，核心組態將會控制系統的回應方式。 如需啟用和停用 SysRq 的資訊，請參閱＜SysRq 系統管理指南＞ [文字](https://aka.ms/kernelorgsysreqdoc) | [markdown](https://aka.ms/linuxsysrq)。  

使用命令列中的鍵盤圖示，即可使用 Azure 序列主控台將 SysRq 傳送至 Azure 虛擬機器，如下所示。

![](../media/virtual-machines-serial-console/virtual-machine-serial-console-command-menu.jpg)

選擇 [傳送 SysRq 命令] 後會開啟對話方塊，其中會提供一般 SysRq 選項，或接受輸入至對話方塊中的一串 SysRq 命令。  這可讓一系列 SysRq 命令執行高階作業，例如，使用 `REISUB` 進行安全的重新開機。

![](../media/virtual-machines-serial-console/virtual-machine-serial-console-sysreq_UI.png)

SysRq 命令不能在已停止或其核心處於無回應狀態 (例如內核錯誤) 的虛擬機器上使用 。

### <a name="enable-sysrq"></a>啟用 SysRq 
如上方＜SysRq 系統管理指南＞中所述，您可以將 SysRq 設定為可使用全部命令、不使用命令或只使用特定命令。 您可以使用下列步驟來啟用所有 SysRq 命令，但重新開機後並不會保留此設定：
```
echo "1" >/proc/sys/kernel/sysrq
```
若要保留 SysReq 設定，您可以執行下列命令來啟用所有 SysRq 命令
1. 將這一行新增至 /etc/sysctl.conf <br>
    `kernel.sysrq = 1`
1. 執行下列命令來重新開機或更新 sysctl <br>
    `sysctl -p`

### <a name="command-keys"></a>命令鍵 
來自上述的 SysRq 系統管理指南：

|命令| 函式
| ------| ----------- |
|``b``  |   將立即重新啟動系統，並且不會同步或取消掛接磁碟。
|``c``  |   將執行 NULL 指標取值 (Dereference) 引起的系統損壞。 如有設定，將會採用損毀傾印 (crashdump)。
|``d``  |   顯示所有保留的鎖定項目。
|``e``  |   將 SIGTERM 傳送給所有處理程序，但不包括 init。
|``f``  |   將呼叫 OOM Killer 來終止消耗大量記憶體的處理程序，但不會在沒有項目可終止時發生錯誤。
|``g``  |   由 kgdb (核心偵錯) 使用
|``h``  |   將顯示說明 (除了此處列出的按鍵，任何其他按鍵也可顯示說明，但 ``h`` 比較好記 :-)
|``i``  |    將 SIGKILL 傳送給所有處理程序，但不包括 init。
|``j``  |    對由 FIFREEZE ioctl 凍結的檔案系統強制執行「直接解除凍結」。
|``k``  |    安全存取金鑰 (SAK) 會終止目前虛擬主控台上的所有程式。 注意：請參閱下方 SAK 一節中的重要註解。
|``l``  |    顯示所有作用中 CPU 的堆疊回溯追蹤 (stack backtrace)。
|``m``  |    將目前記憶體資訊傾印到主控台。
|``n``  |    用以讓 RT 工作的 nice 值可調整
|``o``  |    將關閉您的系統 (如果已設定而且支援的話)。
|``p``  |    將旗標和暫存器傾印到您的主控台。
|``q``  |    傾印每個 CPU 上所有已裝備的 hrtimer 清單 (而非一般 timer_list 計時器)，以及所有有關 clockevent 裝置的詳細資訊。
|``r``  |    關閉鍵盤的 raw 模式，並將它設定為 XLATE。
|``s``  |    將嘗試同步所有已掛接的檔案系統。
|``t``  |    將目前的工作清單和其資訊傾印到主控台。
|``u``  |    嘗試將所有已掛接的檔案系統重新掛接為唯讀狀態。
|``v``  |    強制還原 framebuffer 主控台
|``v``  |    造成 ETM 緩衝區傾印 [ARM 特有]
|``w``  |    傾印處於不可中斷 (已封鎖) 狀態的工作。
|``x``  |    在 ppc/powerpc 平台上供 xmon 介面使用。 顯示 sparc64 上的全域 PMU 暫存器。 傾印 MIPS 上的所有 TLB 項目。
|``y``  |    顯示全域 CPU 暫存器 [SPARC 64 特有]
|``z``  |    傾印 ftrace 緩衝區
|``0``-``9`` | 設定主控台記錄層級，控制要列印至主控台的核心訊息。 (例如，``0`` 只會讓 PANIC 或 OOPS 這類緊急訊息列印至主控台。)

### <a name="distribution-specific-documentation"></a>散發版本特有的文件 ###
如需散發版本特有的 SysRq 相關文件，以及在 Linux 收到 SysRq 的「損毀」命令時，可建立損毀傾印的設定步驟，請參閱下列連結：

#### <a name="ubuntu"></a>Ubuntu ####
 - [核心損毀傾印](https://help.ubuntu.com/lts/serverguide/kernel-crash-dump.html)

#### <a name="red-hat"></a>Red Hat ####
- [什麼是 SysRq 設施？該如何使用？](https://access.redhat.com/articles/231663)
- [如何使用 SysRq 設施從 RHEL 伺服器收集資訊](https://access.redhat.com/solutions/2023)

#### <a name="suse"></a>SUSE ####
- [設定核心的核心傾印擷取](https://www.suse.com/support/kb/doc/?id=3374462)

#### <a name="coreos"></a>CoreOS ####
- [收集損毀記錄](https://coreos.com/os/docs/latest/collecting-crash-logs.html)

## <a name="non-maskable-interrupt-nmi"></a>非遮罩式插斷 (NMI) 
非遮罩式插斷 (NMI) 旨在建立虛擬機器上軟體不會忽略的訊號。 在過去，NMI 已用來監視系統上需要特定回應時間的硬體問題。  現在，程式設計人員和系統管理員通常會使用 NMI 作為對無回應系統進行偵錯和疑難排解的機制。

使用命令列中的鍵盤圖示，即可使用序列主控台將 NMI 傳送至 Azure 虛擬機器，如下所示。 NMI 序列傳遞出去後，虛擬機器組態將會控制系統的回應方式。  Linux 作業系統可以設定為損毀，並在作業系統收到 NMI 時，建立記憶體傾印。

![](../media/virtual-machines-serial-console/virtual-machine-serial-console-command-menu.jpg) <br>

針對支援 sysctl 來設定核心參數的 Linux 系統，您可以在收到此 NMI 時，使用下列命令來引發系統異常：
1. 將這一行新增至 /etc/sysctl.conf <br>
    `kernel.panic_on_unrecovered_nmi=1`
1. 執行下列命令來重新開機或更新 sysctl <br>
    `sysctl -p`

如需有關 Linux 核心組態的資訊 (包括 `unknown_nmi_panic`、`panic_on_io_nmi` 和 `panic_on_unrecovered_nmi`)，請參閱：[/proc/sys/kernel/* 的文件](https://www.kernel.org/doc/Documentation/sysctl/kernel.txt)。 如需散發版本特有的 NMI 相關文件，以及在 Linux 收到 NMI 時，可建立損毀傾印的設定步驟，請參閱下列連結：
 
### <a name="ubuntu"></a>Ubuntu 
 - [核心損毀傾印](https://help.ubuntu.com/lts/serverguide/kernel-crash-dump.html)

### <a name="red-hat"></a>Red Hat 
 - [什麼是 NMI？該如何使用？](https://access.redhat.com/solutions/4127)
 - [如何在推送 NMI 參數時，將我的系統設定為損毀？](https://access.redhat.com/solutions/125103)
 - [損毀傾印的系統管理指南](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/pdf/kernel_crash_dump_guide/kernel-crash-dump-guide.pdf)

### <a name="suse"></a>SUSE 
- [設定核心的核心傾印擷取](https://www.suse.com/support/kb/doc/?id=3374462)

### <a name="coreos"></a>CoreOS 
- [收集損毀記錄](https://coreos.com/os/docs/latest/collecting-crash-logs.html)

## <a name="next-steps"></a>後續步驟
* 主要序列主控台 Linux 文件頁面在[這裡](serial-console.md)。
* 使用序列主控台在開機時進入 [GRUB 並進入單一使用者模式](serial-console-grub-single-user-mode.md)
* 序列主控台也適用於 [Windows](../windows/serial-console.md) VM
* 深入了解[開機診斷](boot-diagnostics.md)