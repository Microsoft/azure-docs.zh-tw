---
title: Windows 停止錯誤 - 狀態無記憶體
description: 本文提供的步驟可解決 Windows 無法啟動的問題，並顯示「狀態沒有記憶體」的狀態。
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: d29c7e49-4578-43c8-b829-831da4e48932
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 06/26/2020
ms.author: v-mibufo
ms.openlocfilehash: 5d6e738152e542617046834980d3e7c58e497093
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98664676"
---
# <a name="windows-stop-error---status-no-memory"></a>Windows 停止錯誤 - 狀態無記憶體

本文提供的步驟可解決 Windows 無法啟動的問題，並顯示狀態或錯誤碼 #0xC0000017，也稱為「無狀態記憶體」。

## <a name="symptom"></a>徵狀

當您使用 [開機診斷](./boot-diagnostics.md) 來查看虛擬機器 (VM) 的螢幕擷取畫面時，您會看到螢幕擷取畫面顯示錯誤碼： `0xC0000017` 。 視您正在執行的 Windows 版本而定，您可能會看到此程式碼顯示在 **Windows 開機管理** 程式或 **修復畫面** 中。

   **Windows 開機管理程式**

   ![Windows 開機管理程式，指出「Windows 無法啟動。 最近的硬體或軟體變更可能是「原因」。 您會看到狀態碼 "0xC0000017"，以及說明「因為必要的檔案遺失或包含錯誤，所以無法載入應用程式或作業系統」的資訊。](./media/troubleshoot-windows-stop-error/1.png)

   **修復畫面**
 
   ![復原畫面會指出「您的電腦/裝置必須修復。 沒有足夠的記憶體可用來建立 ramdisk 裝置」。 您也應該會看到錯誤碼 "0xC0000017"。](./media/troubleshoot-windows-stop-error/2.png)

## <a name="cause"></a>原因

作業系統的磁片已滿、太過分散，或作業系統 (作業系統) 無法存取記憶體或分頁檔案，或兩者都無法存取。

## <a name="solution"></a>解決方法

### <a name="process-overview"></a>程序概觀：

> [!TIP]
> 如果您有最新的 VM 備份，您可以嘗試 [從備份還原 vm](../../backup/backup-azure-arm-restore-vms.md) 以修正開機問題。

1. 建立和存取修復 VM
1. 釋放磁片上的空間
1. 清除 BCD 存放區中的錯誤記憶體
1. 將頁面檔案還原至其預設位置
1. 啟用序列主控台和記憶體傾印收集
1. 重建 VM

> [!NOTE]
> 發生此錯誤時，客體作業系統將無法運作。 您將在離線模式下進行疑難排解，以解決此問題。

### <a name="create-and-access-a-repair-vm"></a>建立及存取修復 VM

1. 使用 [VM 修復命令的步驟 1-3](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) 準備修復 VM。
1. 使用遠端桌面連線連線至修復 VM。

### <a name="for-generation-2-vms-assign-a-letter-to-the-extensible-firmware-interface-efi-partition"></a>針對第2代 Vm，請將信件指派給可延伸的固件介面 (EFI) 磁碟分割：

如果您使用第2代 VM，連接磁片的 EFI 磁碟分割可能沒有指派給它的字母。 繼續進行此疑難排解指南之前，您必須先遵循下列步驟，將信件指派給磁碟分割。

1. 在 Windows 搜尋中，輸入 `diskmgmt` 並開啟 **磁片管理主控台**。
1. 識別連接至修復 VM 的已中斷磁片。 一般而言，此磁片會在主控台中最後列出，而且具有最大的數值。
1. 請注意，如果在該磁片中有一個保存 **EFI 系統磁碟分割** 的磁碟分割，也就是沒有指派給它的字母值 (例如磁片磁碟機 *F：*) 。 如果已指派所有磁碟分割，您可以直接跳過以釋出磁片上的空間。 否則，請繼續將字母指派至此磁片。

   ![磁片管理主控台的連接磁片「磁片2」，以及 100 MB 的未指派磁碟分割，也是「EFI 系統磁碟分割」。](./media/troubleshoot-windows-stop-error/3.png)

1. 以系統管理員身分開啟提升許可權的命令提示字元，然後輸入 `diskpart` 以啟動 **DISKPART** 工具。
1. 輸入下列命令：

   ```
   list disk
   sel disk <NUMBER OF THE ATTACHED DISK>
   list partition
   sel partition <NUMBER OF THE SYSTEM (EFI) PARTION>
   assign
   ```
   
   - 在命令中，以 `<NUMBER OF THE ATTACHED DISK>` 您在步驟2中識別的磁片編號取代。
   - 在命令中， `<NUMBER OF THE SYSTEM PARTION>` 將取代為 EFI 系統磁碟分割的磁碟分割編號。 尚未指派字母給此分割區，但它的類型應該是 [ **系統** ]，大小必須是 100 mb。

   > [!NOTE]
   > 將磁片管理主控台中的磁碟分割與 DISKPART 工具中列出的磁碟分割進行比較，有助於判斷哪個磁碟分割編號對應到連接磁片中的 EFI 系統磁碟分割。

1. 關閉 [命令提示字元] 視窗。

### <a name="free-up-space-on-the-disk"></a>釋放磁片上的空間

既然中斷的磁片已連接至修復 VM，您應該確認該磁片上的 OS 有足夠的空間可正常運作。 

1. 以滑鼠右鍵按一下連接的磁片磁碟機，然後選取 [內容]，檢查磁片是否已 **滿。**
1. 如果磁片的 **可用空間少於 300 Mb**，請 [使用 PowerShell 將它擴充到最大 1 Tb](../windows/expand-os-disk.md)。
1. 一旦磁片大小為 **1 Tb**，您就必須執行磁片清理。 您可以使用 [ [磁片清理] 工具](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) 來釋放空間。
1. 開啟提升許可權的命令提示字元 (以系統管理員身分) 實例執行，並在磁片磁碟機上執行還原片段：

   ``
   defrag <LETTER ASSIGNED TO THE OS DISK>: /u /x /g
   ``
   
   - 視片段層級而定，可能需要數小時的時間才能完成磁碟重組。
   - 在命令中，將取代為 `<LETTER ASSIGNED TO THE OS DISK>` 作業系統磁片的字母 (例如磁片磁碟機 *F：*) 。

### <a name="clean-out-bad-memory-from-the-boot-configuration-data-bcd-store"></a>清除開機設定資料 (BCD) 存放區中的錯誤記憶體

1. 開啟提升許可權的命令提示字元 (以系統管理員身分執行) 。
1. 使用下列命令，查詢開機設定檔中是否有錯誤的記憶體旗標：

   ``
   bcdedit /store <LETTER ASSIGNED TO THE OS DISK>:\boot\bcd /enum {badmemory}
   ``
   
   - 在命令中，將取代為 `<LETTER ASSIGNED TO THE OS DISK>` 作業系統磁片的字母 (例如磁片磁碟機 *F：*) 。

1. 如果查詢未顯示任何錯誤的記憶體區塊，請跳到下一個工作。 否則，請繼續進行步驟4。
1. 如果識別出不正確的記憶體區塊，這些區塊會導致無法建立 ramdisk，而且應該使用下列命令加以刪除：

   ``
   bcdedit /store <LETTER ASSIGNED TO THE OS DISK>:\boot\bcd /deletevalue {badmemory} badmemorylist
   ``
   
   - 在命令中，將取代為 `<LETTER ASSIGNED TO THE OS DISK>` 作業系統磁片的字母 (例如磁片磁碟機 *F：*) 。

### <a name="restore-the-page-file-to-its-default-location"></a>將頁面檔案還原至其預設位置

分頁檔案儲存的資料無法由電腦的隨機存取記憶體 (RAM) 為溢位/備份的形式。 檔案可能會裝載于 VHD 而非暫存磁片磁碟機，這是預設的 Azure 位置。 若為 true，則可能無法存取檔案，且應還原至預設位置。

在採取任何步驟之前，您應該在狀況良好的磁片上建立 **\windows\system32\config** 資料夾的複本。 此步驟可確保能夠復原任何不必要的變更。 您將會處理重要的系統檔案，因此強烈建議您這樣做。

1. 在 Windows 搜尋中，輸入 **regedit** ，然後開啟 [登錄編輯程式] 應用程式。
1. 在 [登錄編輯程式] 中，反白顯示 **HKEY_LOCAL_MACHINE** 的機碼，然後從功能表選取 [檔案] **> [載入 Hive ...** ]。

   ![登錄編輯程式的 [載入 Hive] 功能表。](./media/troubleshoot-windows-stop-error/4.png)

1. 在 [載入 Hive] 對話方塊中，選取 [ **\windows\system32\config\SYSTEM** ]，然後按一下 [開啟]。
   1. 系統會提示您輸入名稱，您應該輸入 **BROKENSYSTEM**。 當您進行疑難排解時，此名稱將有助於區分受影響的 hive。
   1. 展開 **HKEY_LOCAL_MACHINE** 以查看您已新增的新 BROKENSYSTEM 金鑰。
1. 使用登錄編輯程式，判斷電腦的開機 ControlSet。
   1. 流覽至 **HKEY_LOCAL_MACHINE >> BROKENSYSTEM >> 選取**。
   1. 在所列的索引鍵中，記下目前的資料值。 例如，如果這個值是 **1** 或 **0x00000001 (1)**，則會 ControlSet001 控制項集。
1. 檢查已設定分頁檔建立的位置。
   1. 在 HKEY_LOCAL_MACHINE\BROKENSYSTEM 中，展開符合您在步驟4中所識別之 ControlSet 號碼的目錄，例如 **ControlSet001**。
   1. 流覽至 **控制項 >> 會話管理員 >> 記憶體管理** ，並記下 **ExistingPageFiles** 金鑰的位置。
   1. 此金鑰應該位於暫存磁片磁碟機的預設 Azure 位置。 如果它不存在且位於另一個位置（例如資料磁片磁碟機或 OS 磁片磁碟機）中的 VHD，則必須將其刪除。
   1. 在檔案總管中流覽至該位置，然後刪除 **pagefile.sys** 檔案。

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>啟用序列主控台和記憶體傾印集合

**建議**：重建 VM 之前，請先藉由執行下列指令碼來啟用序列主控台和記憶體傾印集合：

若要啟用記憶體傾印收集和序列主控台，請執行下列腳本：

1. 以系統管理員身分開啟提升權限的命令提示字元工作階段。
1. 列出 BCD 存放區資料，並判斷您將在下一個步驟中使用的開機載入器識別碼。

   1. 針對第1代 VM，請輸入下列命令，並記下列出的識別碼：
   
      ``
      bcdedit /store <BOOT PARTITON>:\boot\bcd /enum
      ``
   
   - 在命令中， `<BOOT PARTITON>` 將取代為連接磁片中包含開機資料夾之磁碟分割的字母。

      ![在第1代 VM 中列出 BCD 存放區的輸出，其列于 Windows 開機載入器的識別碼號碼。](./media/troubleshoot-windows-stop-error/5.png)

   1. 針對第2代 VM，請輸入下列命令，並記下列出的識別碼：
   
      ``
      bcdedit /store <LETTER OF THE EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum
      ``
   
   - 在命令中，以 `<LETTER OF THE EFI SYSTEM PARTITION>` EFI 系統磁碟分割的字母取代。
   - 啟動磁片管理主控台來識別標示為 **EFI 系統磁碟分割** 的適當系統磁碟分割可能很有説明。
   - 識別碼可以是唯一的 GUID，也可以是預設的 **bootmgr**。

1. 執行下列命令以啟用序列主控台：

   ```
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON 
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```
   
   - 在命令中， `<VOLUME LETTER WHERE THE BCD FOLDER IS>` 將取代為 BCD 資料夾的字母。
   - 在命令中，以 `<BOOT LOADER IDENTIFIER>` 您在上一個步驟中找到的識別碼取代。

1. 確認 OS 磁碟上的可用空間大於 VM 上的記憶體大小 (RAM)。

   如果 OS 磁碟上沒有足夠的空間，請變更記憶體傾印檔案的建立位置，並將該位置指向任何已與 VM 連結且具有足夠可用空間的資料磁碟位置。 若要變更位置，請使用下列命令中的資料磁片磁碟機號（例如磁片磁碟機 **F：**）來取代 **% SystemRoot%** 。

   啟用 OS 傾印的建議設定：

   **從中斷的 OS 磁碟載入登錄區：**

   ```
   REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM
   ```

   **在 ControlSet001 上啟用：**

   ```
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f 
   ```

   **在 ControlSet002 上啟用：**

   ```
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f 
   ```

   **卸載中斷的 OS 磁碟：**

   ```
   REG UNLOAD HKLM\BROKENSYSTEM
   ```
   
### <a name="rebuild-the-vm"></a>重建 VM

使用 [VM 修復命令的步驟 5](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example)\(部分機器翻譯\)重建 VM。
