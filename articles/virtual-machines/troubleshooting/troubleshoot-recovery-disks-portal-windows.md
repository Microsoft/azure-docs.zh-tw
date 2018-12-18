---
title: 在 Azure 入口網站中使用 Windows 疑難排解 VM | Microsoft Docs
description: 了解如何在 Azure 中使用 Azure 入口網站將 OS 磁碟連接至復原 VM，以針對 Windows 虛擬機器問題進行疑難排解
services: virtual-machines-windows
documentationCenter: ''
authors: genlin
manager: jeconnoc
editor: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 08/013/2018
ms.author: genli
ms.openlocfilehash: 42494ef538fa9840afe5f489074934da3965d56f
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411523"
---
# <a name="troubleshoot-a-windows-vm-by-attaching-the-os-disk-to-a-recovery-vm-using-the-azure-portal"></a>使用 Azure 入口網站將 OS 磁碟連結至復原 VM，以針對 Windows VM 進行疑難排解
如果 Azure 中的 Windows 虛擬機器 (VM) 發生開機或磁碟錯誤，您可能需要對虛擬硬碟本身執行疑難排解步驟。 常見的例子是應用程式更新無效，導致 VM 無法成功開機。 本文詳細說明如何使用 Azure 入口網站將虛擬硬碟連接至另一個 Windows VM，以修正任何錯誤，然後重新建立原始 VM。

## <a name="recovery-process-overview"></a>復原程序概觀
疑難排解程序如下所示︰

1. 刪除遇到問題的 VM，保住虛擬硬碟。
2. 將虛擬硬碟連結和掛接至另一個 Windows VM，以進行疑難排解。
3. 連接至疑難排解 VM。 編輯檔案或執行任何工具來修正原始虛擬硬碟的問題。
4. 從疑難排解 VM 卸載並中斷連結虛擬硬碟。
5. 使用原始虛擬硬碟建立 VM。

針對使用受控磁碟的 VM，我們現在可以使用 Azure PowerShell 來變更 VM 的 OS 磁碟。 我們不再需要刪除及重新建立虛擬機器。 如需詳細資訊，請參閱[使用 Azure PowerShell 將 OS 磁碟連結至復原 VM，以針對 Windows VM 進行疑難排解](troubleshoot-recovery-disks-windows.md)。

## <a name="determine-boot-issues"></a>判斷開機問題
若要判斷 VM 為何無法正常開機，檢查開機診斷 VM 螢幕擷取畫面。 常見的例子是應用程式更新無效，或因為刪除或移動基礎虛擬硬碟。

在入口網站中選取您的 VM，然後向下捲動至 [支援 + 疑難排解] 區段。 按一下 [開機診斷] 檢視螢幕擷取畫面。 請注意任何特定的錯誤訊息或錯誤碼，以協助判斷 VM 為何發生問題。 下列範例會顯示正在等候停止服務的 VM:

![檢視 VM 開機診斷主控台記錄](./media/troubleshoot-recovery-disks-portal-windows/screenshot-error.png)

您也可以按一下 [螢幕擷取畫面] 下載 VM 螢幕擷取畫面的擷取。


## <a name="view-existing-virtual-hard-disk-details"></a>檢視現有的虛擬硬碟詳細資料
您需要先識別虛擬硬碟 (VHD) 的名稱，才能將虛擬硬碟連結至另一個 VM。 

從入口網站選取資源群組，然後選取儲存體帳戶。 按一下 [Blob]，如下列範例所示︰

![選取儲存體 Blob](./media/troubleshoot-recovery-disks-portal-windows/storage-account-overview.png)

一般來說，您會有一個名為 **vhds** 的容器，以儲存虛擬硬碟。 選取該容器以檢視虛擬硬碟清單。 記下 VHD 的名稱 (其前置詞通常是 VM 的名稱)︰

![識別儲存體容器中的 VHD](./media/troubleshoot-recovery-disks-portal-windows/storage-container.png)

從清單中選取您現有的虛擬硬碟，並複製 URL 以供下列步驟使用︰

![複製現有的虛擬硬碟 URL](./media/troubleshoot-recovery-disks-portal-windows/copy-vhd-url.png)


## <a name="delete-existing-vm"></a>刪除現有的 VM
虛擬硬碟和 VM 在 Azure 中是兩個不同的資源。 虛擬硬碟中儲存作業系統本身、應用程式和設定。 VM 本身只是定義大小或位置的中繼資料，還會參考資源，例如虛擬硬碟或虛擬網路介面卡 (NIC)。 每個虛擬硬碟連結至 VM 時會獲派租用。 雖然即使 VM 正在執行時也可以連結和中斷連結資料磁碟，但除非刪除 VM 資源，否則無法中斷連結 OS 磁碟。 即使 VM 處於已停止和解除配置的狀態，租用仍會持續讓 OS 磁碟與 VM 產生關聯。

復原 VM 的第一個步驟是刪除 VM 資源本身。 刪除 VM 時，虛擬硬碟會留在儲存體帳戶中。 刪除 VM 之後，您需要將虛擬硬碟連結至另一個 VM，以進行疑難排解並解決錯誤。

在入口網站中選取 VM，然後按一下 [刪除]：

![顯示開機錯誤的 VM 開機診斷螢幕擷取畫面](./media/troubleshoot-recovery-disks-portal-windows/stop-delete-vm.png)

請等到 VM 完成刪除之後，再將虛擬硬碟連結至另一個 VM。 在虛擬硬碟上，將它與 VM 產生關聯的租用必須先釋放，您才能將虛擬硬碟連結至另一個 VM。


## <a name="attach-existing-virtual-hard-disk-to-another-vm"></a>將現有的虛擬硬碟連結至另一個 VM
在接下來幾個步驟中，您將使用另一個 VM 進行疑難排解。 您要將現有的虛擬硬碟連結至這個疑難排解 VM，以便能夠瀏覽並編輯磁碟的內容。 舉例來說，此程序可讓您更正任何設定錯誤，或檢閱其他應用程式記錄檔或系統記錄檔。 選擇或建立另一個 VM 以進行疑難排解。

1. 從入口網站選取資源群組，然後選取疑難排解 VM。 選取 [磁碟]，然後按一下 [連結現有項目]：

    ![在入口網站中連結現有磁碟](./media/troubleshoot-recovery-disks-portal-windows/attach-existing-disk.png)

2. 若要選取您現有的虛擬硬碟，請按一下 [VHD 檔案]：

    ![瀏覽現有 VHD](./media/troubleshoot-recovery-disks-portal-windows/select-vhd-location.png)

3. 選取儲存體帳戶和容器，然後按一下您現有的 VHD。 按一下 [選取] 按鈕，以確認您的選擇︰

    ![選取現有 VHD](./media/troubleshoot-recovery-disks-portal-windows/select-vhd.png)

4. 在已選取 VHD 之後，按一下 [確定] 以連結現有虛擬硬碟︰

    ![確認連結現有虛擬硬碟](./media/troubleshoot-recovery-disks-portal-windows/attach-disk-confirm.png)

5. 幾秒鐘後，VM 的 [磁碟] 窗格便會將您現有的已連接虛擬硬碟列示為資料磁碟︰

    ![現有虛擬硬碟已連結為資料磁碟](./media/troubleshoot-recovery-disks-portal-windows/attached-disk.png)


## <a name="mount-the-attached-data-disk"></a>掛接已連結的資料磁碟

1. 開啟對 VM 的遠端桌面連線。 在入口網站中選取 VM，然後按一下 [連接]。 下載並開啟 RDP 連接檔案。 輸入您的認證來登入您的 VM，如下所示︰

    ![使用遠端桌面登入 VM](./media/troubleshoot-recovery-disks-portal-windows/open-remote-desktop.png)

2. 開啟 [伺服器管理員]，然後選取 [檔案和存放服務]。 

    ![選取伺服器管理員內的檔案和存放服務](./media/troubleshoot-recovery-disks-portal-windows/server-manager-select-storage.png)

3. 會自動偵測並連接資料磁碟。 若要查看已連接磁碟的清單，請選取 [磁碟]。 您可以選取您的資料磁碟以檢視磁碟區資訊，包括磁碟機代號。 下列範例會顯示已連接的資料磁碟並使用 **F:**：

    ![伺服器管理員中的已連接磁碟和磁碟區資訊](./media/troubleshoot-recovery-disks-portal-windows/server-manager-disk-attached.png)


## <a name="fix-issues-on-original-virtual-hard-disk"></a>修正原始虛擬硬碟的問題
已掛接現有的虛擬硬碟掛，您現在可以視需要執行任何維護和疑難排解步驟。 解決問題之後，請繼續進行下列步驟。


## <a name="unmount-and-detach-original-virtual-hard-disk"></a>卸載並中斷連結原始虛擬硬碟
錯誤解決之後，請將現有虛擬硬碟與疑難排解 VM 的連結中斷。 直到將虛擬硬碟連結至疑難排解 VM 的租用釋放，您才能將虛擬硬碟用於其他任何 VM。

1. 從 RDP 工作階段到您的 VM，開啟 [伺服器管理員]，然後選取 [檔案和存放服務]：

    ![在伺服器管理員中選取檔案和存放服務](./media/troubleshoot-recovery-disks-portal-windows/server-manager-select-storage.png)

2. 依序選取 [磁碟] 及您的資料磁碟。 在資料磁碟上按一下滑鼠右鍵，然後選取 [離線]：

    ![在伺服器管理員中將資料磁碟設為離線](./media/troubleshoot-recovery-disks-portal-windows/server-manager-set-disk-offline.png)

3. 現在從 VM 中斷連結虛擬硬碟。 在 Azure 入口網站中選取 VM，然後按一下 [磁碟]。 選取您現有的虛擬硬碟，然後按一下 [中斷連結]：

    ![將現有虛擬硬碟中斷連結](./media/troubleshoot-recovery-disks-portal-windows/detach-disk.png)

    請稍候，等待 VM 成功將資料磁碟中斷連結再繼續。

## <a name="create-vm-from-original-hard-disk"></a>從原始硬碟建立 VM
若要從原始虛擬硬碟建立 VM，請使用[這個 Azure Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-specialized-vhd-existing-vnet)。 此範本會使用來自先前命令的 VHD URL，將 VM 部署至現有的虛擬網路。 按一下 [部署至 Azure] 按鈕，如下所示︰

![從 Github 的範本部署 VM](./media/troubleshoot-recovery-disks-portal-windows/deploy-template-from-github.png)

範本會載入到 Azure 入口網站以供部署。 輸入新 VM 和現有 Azure 資源的名稱，並貼上您現有虛擬硬碟的 URL。 若要開始部署，請按一下 [購買]：

![從範本部署 VM](./media/troubleshoot-recovery-disks-portal-windows/deploy-from-image.png)


## <a name="re-enable-boot-diagnostics"></a>重新啟用開機診斷
當您從現有的虛擬硬碟建立 VM 時，可能不會自動啟用開機診斷。 若要檢查開機診斷狀態並在需要時開啟，請在入口網站中選取 VM。 在 [監視] 底下，按一下 [診斷設定]。 請確定狀態是 [開啟]，而且已選取 [開機診斷] 旁邊的核取記號。 如果有進行任何變更，請按一下 [儲存]：

![更新開機診斷設定](./media/troubleshoot-recovery-disks-portal-windows/reenable-boot-diagnostics.png)

## <a name="next-steps"></a>後續步驟
如果連接至 VM 時發生問題，請參閱[針對 Azure VM 的 RDP 連接進行疑難排解](troubleshoot-rdp-connection.md)。 如果存取 VM 上執行的應用程式時發生問題，請參閱[針對 Windows VM 上的應用程式連線問題進行疑難排解](troubleshoot-app-connection.md)。

如需使用 Resource Manager 的詳細資訊，請參閱 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)。

