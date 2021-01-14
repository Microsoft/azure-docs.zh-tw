---
title: Azure VM 在套用 Windows Update 時，未回應 C01A001D 錯誤
description: 本文提供的步驟可解決 Windows update 會產生錯誤，並在 Azure VM 中變成沒有回應的問題。
services: virtual-machines-windows
documentationcenter: ''
author: TobyTu
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: a97393c3-351d-4324-867d-9329e31b3528
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 03/31/2020
ms.author: v-mibufo
ms.openlocfilehash: 55dcd65e901a5ce8665ef8d3a2a29d7943581df9
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98200714"
---
# <a name="vm-is-unresponsive-with-c01a001d-error-when-applying-windows-update"></a>在套用 Windows Update 時，VM 未回應 "C01A001D" 錯誤

本文提供的步驟可解決 Windows Update (KB) 產生錯誤，並在 Azure VM 中變成沒有回應的問題。

## <a name="symptoms"></a>徵兆

使用 [開機診斷](./boot-diagnostics.md) 來查看 VM 的螢幕擷取畫面時，會顯示正在進行的 WINDOWS UPDATE (KB) ，但會失敗並出現錯誤碼： ' C01A001D '。

![沒有回應 Windows Update](./media/unresponsive-vm-apply-windows-update/unresponsive-windows-update.png)

## <a name="cause"></a>原因

無法在檔案系統中建立核心檔案。 作業系統無法將檔案寫入磁片。

## <a name="resolution"></a>解決方案

### <a name="process-overview"></a>程序概觀

1. [建立並存取修復 VM](#create-and-access-a-repair-vm)。
2. [釋放硬碟上的空間](#free-up-space-on-the-hard-disk)。
3. [建議：在重建 VM 之前，請啟用序列主控台和記憶體傾印收集](#recommended-before-rebuilding-the-vm-enable-serial-console-and-memory-dump-collection)。
4. [重建 VM](#rebuild-the-vm)。

> [!NOTE]
> 發生此錯誤時，虛擬作業系統無法運作。 您必須在離線模式中進行疑難排解，才能解決此問題。

### <a name="create-and-access-a-repair-vm"></a>建立及存取修復 VM

1. 請依照 [VM 修復命令的步驟 1-3](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) 來準備修復 VM。
2. 使用遠端桌面連線連接到修復 VM。

### <a name="free-up-space-on-the-hard-disk"></a>釋放硬碟空間

如果磁片還不是 1 Tb，您必須調整它的大小。 磁片為 1 TB 之後，請執行磁片清理和磁片磁碟機的磁碟重組。

1. 檢查磁碟是否已滿。 如果磁片低於 1 Tb，請 [使用 PowerShell 將它擴充到最大 1 tb](../windows/expand-os-disk.md)。
2. 磁片為 1 Tb 之後，請執行磁片清理。
    - [從中斷的 VM 卸離資料磁片](../windows/detach-disk.md)。
    - [將資料磁片連接至正常運作的 VM](../windows/attach-disk-ps.md#attach-an-existing-data-disk-to-a-vm)。
    - 使用[磁碟清理工具](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup)釋放空間。
3. 調整大小和清除之後，重組磁片磁碟機：

    ```
    defrag <LETTER ASSIGN TO THE OS DISK>: /u /x /g
    ```
    視分散程度而定，這可能需要數小時的時間。

### <a name="recommended-before-rebuilding-the-vm-enable-serial-console-and-memory-dump-collection"></a>建議：重建 VM 之前，請先啟用序列主控台和記憶體傾印收集

1. 開啟提升權限的命令提示字元工作階段 (以系統管理員身分執行)。
2. 執行下列命令：

    啟用序列主控台：

    ```
    bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
    ```
3. 請確定 OS 磁片上的可用空間至少等於 VM 記憶體 (RAM) 大小。

    如果 OS 磁片上沒有足夠的空間，請變更將建立記憶體傾印檔案的位置，並將它參考到連接至 VM 的資料磁片，並使用足夠的可用空間。 若要變更位置，請以 `%SystemRoot%` 磁碟機號取代 (例如下列命令中資料磁片的 "F：" ) ：

    **啟用 OS 傾印建議的設定：**

    載入中斷的 OS 磁碟：

    ```
    REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM
    ```

    在 ControlSet001 上啟用：

    ```
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
    ```

    在 ControlSet002 上啟用：

    ```
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
    ```

    解除載入中斷的 OS 磁碟：

    ```
    REG UNLOAD HKLM\BROKENSYSTEM
    ```

### <a name="rebuild-the-vm"></a>重建 VM

使用 [vm repair 命令的步驟 5](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) 來重組 vm。
