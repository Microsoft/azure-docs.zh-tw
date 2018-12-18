---
title: 針對 Azure 備份失敗：無法使用客體代理程式狀態進行疑難排解
description: 與代理程式、延伸模組及磁碟相關之 Azure 備份失敗的徵狀、原因和解決方案。
services: backup
author: genlin
manager: cshepard
keywords: Azure 備份; VM 代理程式; 網路連線;
ms.service: backup
ms.topic: troubleshooting
ms.date: 06/25/2018
ms.author: genli
ms.openlocfilehash: ce4a889cae852d333ea9862138f4d44471677c26
ms.sourcegitcommit: f983187566d165bc8540fdec5650edcc51a6350a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2018
ms.locfileid: "45544008"
---
# <a name="troubleshoot-azure-backup-failure-issues-with-the-agent-or-extension"></a>針對 Azure 備份失敗進行疑難排解：與代理程式或延伸模組相關的問題

本文提供疑難排解步驟，協助您解決與 VM 代理程式和延伸模組通訊相關的 Azure 備份錯誤。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="vm-agent-unable-to-communicate-with-azure-backup"></a>VM 代理程式無法與 Azure 備份通訊

錯誤訊息：「VM 代理程式無法與 Azure 備份通訊」<br>
錯誤碼："UserErrorGuestAgentStatusUnavailable"

在註冊及排程備份服務的 VM 之後，備份就會藉由與 VM 代理程式通訊以取得時間點快照集，來起始作業。 下列任一種狀況都可能會阻止觸發快照集。 若未觸發快照集，備份可能會失敗。 請依照列出的順序完成下列疑難排解步驟，然後重試作業：

**原因 1：[VM 沒有網際網路存取](#the-vm-has-no-internet-access)**  
**原因 2：[代理程式已安裝到 VM 中，但沒有回應 (適用於 Windows VM)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**    
**原因 3︰[VM 中安裝的代理程式已過時 (適用於 Linux VM)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**  
**原因 4︰[無法擷取快照集狀態或無法取得快照集](#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken)**    
**原因 5︰[備份延伸模組無法更新或載入](#the-backup-extension-fails-to-update-or-load)**  

## <a name="snapshot-operation-failed-due-to-no-network-connectivity-on-the-virtual-machine"></a>快照集作業因為虛擬機器沒有網路連線而失敗

錯誤訊息：「因為虛擬機器沒有網路連線，所以快照集作業失敗」<br>
錯誤碼："ExtensionSnapshotFailedNoNetwork"

在註冊及排程 Azure 備份服務的 VM 之後，備份就會藉由與 VM 備份擴充功能通訊以取得時間點快照，來起始作業。 下列任一種狀況都可能會阻止觸發快照集。 如果未觸發快照集，可能會發生備份失敗。 請依照列出的順序完成下列疑難排解步驟，然後重試作業：    
**原因 1：[VM 沒有網際網路存取](#the-vm-has-no-internet-access)**  
**原因 2︰[無法擷取快照集狀態或無法取得快照集](#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken)**  
**原因 3︰[備份延伸模組無法更新或載入](#the-backup-extension-fails-to-update-or-load)**  

## <a name="vmsnapshot-extension-operation-failed"></a>VMSnapshot 延伸模組作業失敗

錯誤訊息：「VMSnapshot 延伸模組作業失敗」<br>
錯誤碼："ExtentionOperationFailed"

在註冊及排程 Azure 備份服務的 VM 之後，備份就會藉由與 VM 備份擴充功能通訊以取得時間點快照，來起始作業。 下列任一種狀況都可能會阻止觸發快照集。 如果未觸發快照集，可能會發生備份失敗。 請依照列出的順序完成下列疑難排解步驟，然後重試作業：  
**原因 1︰[無法擷取快照集狀態或無法取得快照集](#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken)**  
**原因 2︰[備份延伸模組無法更新或載入](#the-backup-extension-fails-to-update-or-load)**  
**原因 3：[代理程式已安裝到 VM 中，但沒有回應 (適用於 Windows VM)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**  
**原因 4︰[VM 中安裝的代理程式已過時 (適用於 Linux VM)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**

## <a name="backup-fails-because-the-vm-agent-is-unresponsive"></a>備份因為 VM 代理程式沒有回應而失敗

錯誤訊息：「無法與 VM 代理程式通訊來取得快照集狀態」 <br>
錯誤碼："GuestAgentSnapshotTaskStatusError"

在註冊及排程 Azure 備份服務的 VM 之後，備份就會藉由與 VM 備份擴充功能通訊以取得時間點快照，來起始作業。 下列任一種狀況都可能會阻止觸發快照集。 如果未觸發快照集，可能會發生備份失敗。 請依照列出的順序完成下列疑難排解步驟，然後重試作業：  
**原因 1：[代理程式已安裝到 VM 中，但沒有回應 (適用於 Windows VM)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**  
**原因 2︰[VM 中安裝的代理程式已過時 (適用於 Linux VM)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**  
**原因 3：[VM 沒有網際網路存取](#the-vm-has-no-internet-access)**  

## <a name="backup-fails-with-an-internal-error"></a>備份失敗，發生內部錯誤

錯誤訊息：「備份因為內部錯誤而失敗 - 請在幾分鐘內重試此作業」 <br>
錯誤碼："BackUpOperationFailed"/ "BackUpOperationFailedV2"

在註冊及排程 Azure 備份服務的 VM 之後，備份就會藉由與 VM 備份擴充功能通訊以取得時間點快照，來起始作業。 下列任一種狀況都可能會阻止觸發快照集。 如果未觸發快照集，可能會發生備份失敗。 請依照列出的順序完成下列疑難排解步驟，然後重試作業：  
**原因 1：[VM 沒有網際網路存取](#the-vm-has-no-internet-access)**  
**原因 2：[代理程式已安裝到 VM 中，但沒有回應 (適用於 Windows VM)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)**  
**原因 3︰[VM 中安裝的代理程式已過時 (適用於 Linux VM)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)**  
**原因 4︰[無法擷取快照集狀態或無法取得快照集](#the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken)**  
**原因 5︰[備份延伸模組無法更新或載入](#the-backup-extension-fails-to-update-or-load)**  
**原因 6：[備份服務因資源群組鎖定而沒有刪除舊還原點的權限](#backup-service-does-not-have-permission-to-delete-the-old-restore-points-due-to-resource-group-lock)**

## <a name="causes-and-solutions"></a>原因和解決方案

### <a name="the-vm-has-no-internet-access"></a>VM 沒有網際網路存取
根據部署需求，VM 無法存取網際網路。 或者，它可能會有防止存取 Azure 基礎結構的限制。

備份延伸模組需要連線到 Azure 公用 IP 位址，才能正確運作。 延伸模組會將命令傳送至 Azure 儲存體端點 (HTTPS URL) 來管理 VM 的快照集。 如果延伸模組無法存取公用網際網路，則備份最終會失敗。

您可以部署 Proxy 伺服器來路由傳送 VM 流量。
##### <a name="create-a-path-for-https-traffic"></a>建立 HTTPS 流量的路徑

1. 如果您已有網路限制 (例如，網路安全性群組)，請部署 HTTPS Proxy 伺服器來路由傳送流量。
2. 若要允許從 HTTPS Proxy 伺服器存取網際網路，可將規則新增到網路安全性群組 (如果您有一個)。

若要了解如何設定 VM 備份的 HTTPS Proxy，請參閱[準備環境以備份 Azure 虛擬機器](backup-azure-arm-vms-prepare.md#establish-network-connectivity)。

已備份的 VM，抑或用於路由傳送流量的 Proxy 伺服器需要存取 Azure 公用 IP 位址

####  <a name="solution"></a>解決方法
若要解決此問題，請嘗試下列其中一個方法：

##### <a name="allow-access-to-azure-storage-that-corresponds-to-the-region"></a>允許存取對應該區域的 Azure 儲存體

您可以使用[服務標籤](../virtual-network/security-overview.md#service-tags)，允許連線至特定區域的儲存體。 請確定允許存取儲存體帳戶的規則優先順序，高於封鎖網際網路存取的規則。 

![網路安全性群組與區域的儲存體標籤](./media/backup-azure-arm-vms-prepare/storage-tags-with-nsg.png)

若要了解設定服務標記的逐步程序，請觀看[這個影片](https://youtu.be/1EjLQtbKm1M)。

> [!WARNING]
> 儲存體服務標籤處於預覽狀態。 僅在特定區域中提供使用。 如需區域清單，請參閱[儲存體的服務標籤](../virtual-network/security-overview.md#service-tags)。

如果您使用 Azure 受控磁碟，您可能需要在防火牆上開啟其他連接埠 (連接埠 8443)。

此外，如果您的子網路沒有互聯網輸出流量的路由，則您需要將含有服務標籤「Microsoft.Storage」的服務端點新增至您的子網路。 

### <a name="the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms"></a>代理程式已安裝在 VM 中，但沒有回應 (適用於 Windows VM)

#### <a name="solution"></a>解決方法
VM 代理程式可能已損毀，或服務可能已停止。 重新安裝 VM 代理程式有助於取得最新版本。 也有助於重新開始與服務通訊。

1. 判斷 Windows 客體代理程式服務是否在 VM 服務 (services.msc) 中執行。 請嘗試重新啟動 Windows 客體代理程式服務並啟動備份。    
2. 如果 Windows 客體代理程式服務未顯示在控制台的服務中，請移至 [程式和功能] 來判斷是否已安裝 Windows 客體代理程式服務。
4. 如果此 Windows 客體代理程式顯示在 [程式和功能] 中，請將 Windows 客體代理程式解除安裝。
5. 下載並安裝[最新版的代理程式 MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。 您必須擁有系統管理員權限，才能完成安裝。
6. 確認 Windows 客體代理程式服務顯示在服務中。
7. 執行隨選備份： 
    * 在入口網站中，選取 [立即備份]。

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
4. 執行新的測試備份。 如果失敗持續發生，請從 VM 收集下列記錄：

   * /var/lib/waagent/*.xml
   * /var/log/waagent.log
   * /var/log/azure/*

如果我們需要 waagent 的詳細資訊記錄，請遵循下列步驟：

1. 在 /etc/waagent.conf 檔案中，找出下一行︰**Enable verbose logging (y|n)**
2. 將 **Logs.Verbose** 值從 *n* 變更為 *y*。
3. 儲存變更，然後完成本節前面所述的步驟來重新啟動 waagent。

###  <a name="the-snapshot-status-cannot-be-retrieved-or-a-snapshot-cannot-be-taken"></a>無法擷取快照集狀態或無法取得快照集
VM 備份仰賴發給底層儲存體帳戶的快照命令。 備份可能會失敗，因為它無權存取儲存體帳戶，或是因為快照集工作延遲執行。

#### <a name="solution"></a>解決方法
下列狀況可能導致快照集工作失敗：

| 原因 | 解決方法 |
| --- | --- |
| 因為遠端桌面通訊協定 (RDP) 中的 VM 關機，而導致報告的 VM 狀態不正確。 | 如果您關閉 RDP 中的 VM，請檢查入口網站，以判斷 VM 狀態是否正確。 如果不正確，可使用 VM 儀表板上的 [關閉] 選項來關閉入口網站中的 VM。 |
| VM 無法從 DHCP 取得主機或網狀架構位址。 | 必須在來賓內啟用 DHCP，IaaS VM 備份才能運作。 如果 VM 無法從 DHCP 回應 245 取得主機或網狀架構位址，則無法下載或執行任何延伸模組。 如果您需要靜態私人 IP 位址，請透過平台進行設定。 VM 內的 DHCP 選項應保持啟用。 如需詳細資訊，請參閱[設定靜態內部私人 IP 位址](../virtual-network/virtual-networks-reserved-private-ip.md)。 |

### <a name="the-backup-extension-fails-to-update-or-load"></a>備份擴充功能無法更新或載入
如果無法載入延伸模組，備份就會因為無法取得快照集而失敗。

#### <a name="solution"></a>解決方法

將擴充功能解除安裝，以強制重新載入 VMSnapshot 擴充功能。 下一次備份嘗試會重新載入解除安裝。

若要將解除安裝解除安裝：

1. 在 [Azure 入口網站](https://portal.azure.com/)中，移至發生備份失敗的 VM。
2. 選取 [Settings] \(設定) 。
3. 選取 [擴充功能]。
4. 選取 [Vmsnapshot 解除安裝]。
5. 選取 [解除安裝]。

針對 Linux VM，如果 VMSnapshot 延伸模組未顯示在 Azure 入口網站中，[更新 Azure Linux 代理程式](../virtual-machines/linux/update-agent.md)，然後再執行備份。 

完成這些步驟之後，下一次備份期間會重新安裝延伸模組。

### <a name="backup-service-does-not-have-permission-to-delete-the-old-restore-points-due-to-resource-group-lock"></a>備份服務因資源群組鎖定而沒有刪除舊還原點的權限
這個問題是使用者在其中鎖定資源群組之受控虛擬機器特有的問題。 在此情況下，備份服務無法刪除較舊的還原點。 因為有 18 個還原點的限制，所以新的備份會開始失敗。

#### <a name="solution"></a>解決方法

若要解決此問題，請移除資源群組的鎖定，並完成下列步驟來移除還原點集合： 
 
1. 移除 VM 所在資源群組中的鎖定。 
2. 使用 Chocolatey 安裝 ARMClient： <br>
   https://github.com/projectkudu/ARMClient
3. 登入 ARMClient： <br>
    `.\armclient.exe login`
4. 取得 VM 對應的還原點集合： <br>
    `.\armclient.exe get https://management.azure.com/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Compute/restorepointcollections/AzureBackup_<VM-Name>?api-version=2017-03-30`

    範例： `.\armclient.exe get https://management.azure.com/subscriptions/f2edfd5d-5496-4683-b94f-b3588c579006/resourceGroups/winvaultrg/providers/Microsoft.Compute/restorepointcollections/AzureBackup_winmanagedvm?api-version=2017-03-30`
5. 刪除還原點集合： <br>
    `.\armclient.exe delete https://management.azure.com/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Compute/restorepointcollections/AzureBackup_<VM-Name>?api-version=2017-03-30` 
6. 下一個排定的備份會自動建立還原點集合和新的還原點。

完成後，您可以再次於 VM 資源群組放回鎖定。 
