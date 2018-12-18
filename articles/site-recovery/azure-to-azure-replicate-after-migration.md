---
title: 使用 Azure Site Recovery 移轉至 Azure 之後，設定 Azure VM 的災害復原 | Microsoft Docs
description: 本文說明如何準備機器，以在使用 Azure Site Recovery 移轉至 Azure 之後，設定 Azure 區域之間的災害復原。
services: site-recovery
author: ponatara
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: ponatara
ms.openlocfilehash: 3e26e40c26a27fdab1ec565dd4112b40acdd17d2
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39213362"
---
# <a name="set-up-disaster-recovery-for-azure-vms-after-migration-to-azure"></a>移轉至 Azure 之後，設定 Azure VM 的災害復原 


在您使用 [Site Recovery](site-recovery-overview.md) 服務[將內部部署機器移轉到 Azure VM](tutorial-migrate-on-premises-to-azure.md) 之後，請使用本文章。 本文可協助您使用 Site Recovery，準備將 Azure VM 的災害復原設定至 Azure 次要地區。



## <a name="before-you-start"></a>開始之前

設定災害復原之前，請確定移轉已如預期般完成。 若要成功完成移轉，您就要在容錯移轉之後，針對您需要移轉的每部機器，選取 [完成移轉] 選項。 



## <a name="install-the-azure-vm-agent"></a>安裝 Azure VM 代理程式

VM 上必須安裝 Azure [VM 代理程式](../virtual-machines/extensions/agent-windows.md)，Site Recovery 才能加以複寫。


1. 若要在執行 Windows 的 VM 上安裝 VM 代理程式，請下載並執行[代理程式安裝程式](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。 您需要有 VM 的系統管理員權限，才能完成安裝。
2. 若要在執行 Linux 的 VM 上安裝 VM 代理程式，請安裝最新的 [Linux 代理程式](../virtual-machines/extensions/agent-linux.md)。 您需要有系統管理員權限，才能完成安裝。 建議您從散發存放庫進行安裝。 不建議您直接從 GitHub 安裝 Linux VM 代理程式。 


## <a name="validate-the-installation-on-windows-vms"></a>驗證 Windows VM 上的安裝

1. 在 Azure VM 的 C:\WindowsAzure\Packages 資料夾中，您應該會看到 WaAppAgent.exe 檔案。
2. 在該檔案上按一下滑鼠右鍵，並在 [屬性] 中選取 [詳細資料] 索引標籤。
3. 確認 [產品版本] 欄位顯示 2.6.1198.718 或更高版本。



## <a name="migration-from-vmware-vms-or-physical-servers"></a>從 VMware VM 或實體伺服器進行移轉

如果您將內部部署 VMware VM (或實體伺服器) 移轉至 Azure，請注意：

- 僅在移轉後的機器上安裝之行動服務是 9.6 版或更早版本時，才需要安裝 Azure VM 代理程式。
- 在執行 9.7.0.0 版以上行動服務的 Windows VM 上，服務安裝程式也會安裝最新可用的 Azure VM 代理程式。 在移轉時，這些 VM 已符合任何 VM 擴充功能的代理程式安裝必要條件，包括 Site Recovery 擴充功能。
- 您必須使用下列方法之一，以手動方式從 Azure VM 將行動服務解除安裝。 設定複寫之前，請將 VM 重新啟動。
    - 對於 Windows，請在 [控制台] > [新增/移除程式] 中，將 **Microsoft Azure Site Recovery 行動服務/主要目標伺服器**解除安裝。 在提升權限的命令提示字元上，執行：
        ```
        MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
        ```
    - 若為 Linux，請以根使用者身分登入。 在終端機中，移至 **/user/local/ASR**，然後執行下列命令：
        ```
        uninstall.sh -Y
        ```


## <a name="next-steps"></a>後續步驟

將 Azure VM [快速複寫](azure-to-azure-quickstart.md)到次要地區。
