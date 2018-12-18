---
title: 針對 Azure Site Recovery 代理程式失敗：無法使用客體代理程式狀態進行疑難排解 | Microsoft Docs
description: 與代理程式和延伸模組相關的 Azure Site Recovery 失敗徵狀、原因和解決方案
services: site-recovery
author: asgang
manager: rochakm
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.date: 07/06/2018
ms.author: asgang
ms.openlocfilehash: c0429e87f6c58ef2b9c7a268bee596d769e95910
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37919799"
---
# <a name="troubleshoot-azure-site-recovery-extension-failures-issues-with-the-agent-or-extension"></a>針對 Azure Site Recovery 延伸模組失敗進行疑難排解：代理程式或延伸模組的問題

本文提供疑難排解步驟，可協助您解決與 VM 代理程式和延伸模組相關的 Azure Site Recovery 錯誤。


## <a name="azure-site-recovery-extension-time-out"></a>Azure Site Recovery 延伸模組逾時  

錯誤訊息：「追蹤要啟動的延伸模組作業時，工作執行逾時」<br>
錯誤碼："151076"

 Azure Site Recovery 會在虛擬機器上安裝延伸模組，以形成保護作業中的一環。 下列任一種狀況都可能會阻止觸發保護，進而使得作業失敗。 請完成下列疑難排解步驟，然後重試作業：

**原因 1：[代理程式已安裝到 VM 中，但沒有回應 (適用於 Windows VM)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**    
**原因 2︰[VM 中安裝的代理程式已過時 (適用於 Linux VM)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**  
**原因 3︰[Site Recovery 延伸模組無法更新或載入](#the-site-recovery-extension-fails-to-update-or-load)**  

錯誤訊息：「先前的站台復原延伸模組作業所花時間超出預期」。<br>
錯誤碼："150066"<br>

**原因 1：[代理程式已安裝到 VM 中，但沒有回應 (適用於 Windows VM)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**    
**原因 2︰[VM 中安裝的代理程式已過時 (適用於 Linux VM)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**  
**Cause 3：[Site Recovery 延伸模組狀態不正確](#the-site-recovery-extension-fails-to-update-or-load)**  

## <a name="protection-fails-because-the-vm-agent-is-unresponsive"></a>因為 VM 代理程式沒有回應，而使得保護失敗

錯誤訊息：「追蹤要啟動的延伸模組作業時，工作執行逾時。」<br>
錯誤碼："151099"<br>

如果虛擬機器中的 Azure 客體代理程式不是處於就緒狀態，就會發生此錯誤。
您可以在 [Azure 入口網站](https://portal.azure.com/)中檢查 Azure 客體代理程式的狀態。 移至您想要保護的虛擬機器，並在 [VM] > [設定] > [內容] > [代理程式狀態] 中檢查狀態。 重新啟動虛擬機器之後，大部分情況下代理程式狀態便會成為就緒。 不過，如果無法選擇重新開機，或您仍然遇到此問題，則請完成下列疑難排解步驟。

**原因 1：[代理程式已安裝到 VM 中，但沒有回應 (適用於 Windows VM)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**    
**原因 2︰[VM 中安裝的代理程式已過時 (適用於 Linux VM)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**  


錯誤訊息：「追蹤要啟動的延伸模組作業時，工作執行逾時。」<br>
錯誤碼："151095"<br>

當 Linux 機器上的代理程式為舊版時，便會發生此狀況。 請完成下列疑難排解步驟。<br>
  **原因 1︰[VM 中安裝的代理程式已過時 (適用於 Linux VM)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**  
## <a name="causes-and-solutions"></a>原因和解決方案

### <a name="the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms"></a>代理程式已安裝在 VM 中，但沒有回應 (適用於 Windows VM)

#### <a name="solution"></a>解決方法
VM 代理程式可能已損毀，或服務可能已停止。 重新安裝 VM 代理程式將有助於取得最新版本。 也有助於重新開始與服務通訊。

1. 判斷「Windows Azure 客體代理程式服務」是否在 VM 服務 (services.msc) 中執行。 嘗試重新啟動「Windows Azure 客體代理程式服務」。    
2. 如果「Windows Azure 客體代理程式服務」未顯示在控制台的服務中，請移至 [程式和功能] 來判斷是否已安裝「Windows 客體代理程式服務」。
4. 如果「Windows Azure 客體代理程式」顯示在 [程式和功能] 中，請將「Windows 客體代理程式」解除安裝。
5. 下載並安裝[最新版的代理程式 MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。 您必須擁有系統管理員權限，才能完成安裝。
6. 確認「Windows Azure 客體代理程式服務」顯示在服務中。
7. 重新啟動保護作業。

此外，確認 VM 中[已安裝 Microsoft .NET 4.5](https://docs.microsoft.com/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed)。 VM 代理程式需有 .NET 4.5 才能與服務通訊。

### <a name="the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms"></a>VM 中安裝的代理程式已過時 (適用於 Linux VM)

#### <a name="solution"></a>解決方法
針對 Linux VM，與代理程式或擴充功能相關的多數失敗是由於會影響過時 VM 代理程式的問題所造成。 若要對此問題進行疑難排解，請遵循下列一般方針：

1. 請遵循[更新 Linux VM 代理程式](../virtual-machines/linux/update-agent.md)的指示。

 > [!NOTE]
 > 我們強烈建議您只透過散發套件存放庫更新代理程式。 我們不建議直接從 GitHub 下載代理程式程式碼，並加以更新。 如果最新的代理程式不適用於您的散發套件，請連絡散發套件支援以取得如何進行安裝的指示。 若要檢查最新的代理程式，請移至 GitHub 儲存機制中的 [Microsoft Azure Linux 代理程式 (英文)](https://github.com/Azure/WALinuxAgent/releases) 頁面。

2. 執行下列命令，確定 Azure 代理程式正在 VM 上執行：`ps -e`

 如果此程序不在執行中，請使用下列命令來重新啟動它：

 * 針對 Ubuntu：`service walinuxagent start`
 * 針對其他散發套件︰`service waagent start`

3. [設定自動重新啟動代理程式](https://github.com/Azure/WALinuxAgent/wiki/Known-Issues#mitigate_agent_crash)。
4. 啟用虛擬機器的保護。



### <a name="the-site-recovery-extension-fails-to-update-or-load"></a>Site Recovery 延伸模組無法更新或載入
如果延伸模組狀態為 'Empty' (空白)、'NotReady' (未就緒) 或 'Transitioning' (正在轉換)。

#### <a name="solution"></a>解決方法

將延伸模組解除安裝，並再次重新啟動作業。

若要將解除安裝解除安裝：

1. 在 [Azure 入口網站](https://portal.azure.com/)中，移至發生備份失敗的 VM。
2. 選取 [Settings] \(設定) 。
3. 選取 [擴充功能]。
4. 選取 [Site Recovery 延伸模組]。
5. 選取 [解除安裝]。

針對 Linux VM，如果 VMSnapshot 延伸模組未顯示在 Azure 入口網站中，請[更新 Azure Linux 代理程式](../virtual-machines/linux/update-agent.md)，然後再執行保護。 

完成這些步驟會使得在保護期間重新安裝延伸模組。


