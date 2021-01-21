---
title: 開機錯誤–「這不是可開機的磁片」
description: 本文提供的步驟可解決在 Azure 虛擬機器中無法開機的問題
services: virtual-machines-windows
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 5d6db4e3-c2f5-40c7-afea-54edf745f5eb
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 03/25/2020
ms.author: v-mibufo
ms.openlocfilehash: 14da41815e177ece64c72ac27a7cb126e69fdc62
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98633183"
---
# <a name="boot-error--this-is-not-a-bootable-disk"></a>開機錯誤-這不是可開機的磁片

本文提供的步驟可解決在 Azure 虛擬機器 (VM) 中無法啟動的問題。

## <a name="symptoms"></a>徵兆

當您使用 [開機診斷](./boot-diagnostics.md) 來查看 VM 的螢幕擷取畫面時，您會看到螢幕擷取畫面顯示提示訊息「這不是可開機的磁片。 請插入可開機的磁片，然後按任意鍵再試一次 ...」。

   圖 1

   ![[圖 1] 顯示訊息 * 「這不是可開機的磁片。 請插入可開機的磁片，然後按任意鍵再試一次 ...」 *](media/troubleshoot-guide-not-bootable-disk/1.jpg)

## <a name="cause"></a>原因

此錯誤訊息表示 OS 開機程式找不到作用中的系統磁碟分割。 這項錯誤也可能表示開機設定資料 (BCD) 存放區中有遺失的參考，因此無法找到 Windows 磁碟分割。

## <a name="solution"></a>解決方法

### <a name="process-overview"></a>程序概觀

> [!TIP]
> 如果您有最新的 VM 備份，您可以嘗試 [從備份還原 vm](../../backup/backup-azure-arm-restore-vms.md) 以修正開機問題。

1. 建立及存取修復 VM。
2. 將資料分割狀態設定為作用中。
3. 修正磁碟分割。
4. **建議**：重建 VM 之前，請先啟用序列主控台和記憶體傾印集合。
5. 重建原始 VM。

   > [!NOTE]
   > 當遇到此開機錯誤時，虛擬作業系統無法運作。 您將在離線模式下進行疑難排解，以解決此問題。

### <a name="create-and-access-a-repair-vm"></a>建立和存取修復 VM

1. 使用 [VM 修復命令](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md)的步驟 1-3 準備修復 VM。
2. 使用遠端桌面連線連線至修復 VM。

### <a name="set-partition-status-to-active"></a>將資料分割狀態設定為作用中

第1代 Vm 應先確認保存 BCD 存放區的作業系統分割區已標示為使用 *中。* 如果您有第2代 VM，請直接跳到 [修正磁碟分割](#fix-the-disk-partition)，因為 *狀態* 旗標在之後的版本中已淘汰。

1. 開啟提升許可權的命令提示字元 *( # A0)*。
2. 輸入 *diskpart* 以啟動 DISKPART 工具。
3. 輸入 *清單磁片* 以列出系統上的磁片，並識別連接的 OS VHD。
4. 找到連結的 OS VHD 之後，請輸入 *sel disk #* 以選取磁碟。  請參閱圖2，其中 Disk 1 是連接的 OS VHD。

   圖 2

   ![[圖 2] 顯示的 * DISKPART * 視窗顯示清單磁片命令的輸出，磁片0和磁片1顯示在表格中。  也會顯示 [sel disk 1] 命令的輸出，磁片1是選取的磁片](media/troubleshoot-guide-not-bootable-disk/2.jpg)

5. 選取磁碟之後，請輸入 *list partition* 以列出所選擇磁碟的磁碟分割。
6. 識別出開機磁碟分割之後，請輸入 *sel partition #* 以選取該磁碟分割。  開機磁碟分割通常會有大約 350 MB 的大小。  請參閱 [圖 3]，其中的磁碟分割1是開機磁碟分割。

   圖 3

   ![[圖 3] 顯示 * DISKPART * 視窗，內含 * list partition * 命令的輸出。 資料分割1和資料分割2會顯示在資料表中。 當資料分割1是選取的磁片時，它也會顯示 * sel partition 1 * 命令的輸出。](media/troubleshoot-guide-not-bootable-disk/3.jpg)

7. 輸入「詳細資料分割」以檢查資料分割的狀態。 請參閱 [圖 4]，其中的磁碟分割為作用中 *： [否*] 或 [圖 5]，其中的分割區為「使用中：是」。

   圖 4

   ![[圖 4] 顯示了 * DISKPART * 視窗，其中當分割區1設定為 * Active： No * 時，會有 * detail partition * 命令的輸出。](media/troubleshoot-guide-not-bootable-disk/4.jpg)

   圖 5

   ![[圖 5] 顯示當分割區1設定為 * Active： Yes * 時，具有 * detail partition * 命令輸出的 * DISKPART * 視窗。](media/troubleshoot-guide-not-bootable-disk/5.jpg)

8. 如果分割區 **不** 在使用中，請輸入 *active* 以變更使用 *中的旗* 標。
9. 輸入 *詳細資料分割*，檢查狀態變更是否已正確完成。

   圖 6

   ![[圖 6] 顯示當分割區1設定為 * Active： Yes * 時，具有 * detail partition * 命令輸出的 diskpart 視窗](media/troubleshoot-guide-not-bootable-disk/6.jpg)

10. 輸入 *exit* 以關閉 DISKPART 工具，並儲存您的設定變更。

### <a name="fix-the-disk-partition"></a>修正磁碟分割

1. 開啟提升權限的命令提示字元 (cmd.exe)。
2. 使用下列命令在磁片上執行 *CHKDSK* (s) 並修正錯誤：

   `chkdsk <DRIVE LETTER>: /f`

   新增 '/f ' 命令選項可修正磁片上的所有錯誤。 請務必以 <DRIVE LETTER> 連接的作業系統 VHD 的字母取代。

### <a name="recommended-before-you-rebuild-the-vm-enable-serial-console-and-memory-dump-collection"></a>建議：在重建 VM 之前，請先啟用序列主控台和記憶體傾印收集

若要啟用記憶體傾印收集和序列主控台，請執行下列腳本：

1. 開啟提升權限的命令提示字元工作階段 (以系統管理員身分執行)。
2. 執行下列命令：

   啟用序列主控台

   `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON`

   `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200`

3. 確認 OS 磁片上的可用空間與 VM 上 (RAM) 的記憶體大小一樣多。

   如果 OS 磁片上沒有足夠的空間，您應該變更將建立記憶體傾印檔案的位置，並將其參考到連接至 VM 且具有足夠可用空間的任何資料磁片。 若要變更位置，請將 "% SystemRoot%" 取代為磁碟機號 (例如，以下命令中的資料磁片 "F：" ) 。

#### <a name="suggested-configuration-to-enable-os-dump"></a>啟用 OS 傾印的建議設定

**載入中斷的作業系統磁碟**：

`REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM`

**在 ControlSet001 上啟用：**

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f`

**在 ControlSet002 上啟用：**

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f`

**卸載中斷的 OS 磁碟：**

`REG UNLOAD HKLM\BROKENSYSTEM`

### <a name="rebuild-the-original-vm"></a>重建原始 VM

使用 [VM 修復命令的步驟 5](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) 重新組裝 VM。
