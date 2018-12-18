---
title: Azure 備份代理程式常見問題集
description: 有關以下常見問題的解答︰Azure 備份代理程式的運作方式、備份和保留限制。
services: backup
author: trinadhk
manager: shreeshd
keywords: 備份和災害復原; 備份服務
ms.service: backup
ms.topic: conceptual
ms.date: 8/6/2018
ms.author: saurse;trinadhk
ms.openlocfilehash: 177e44bce7d8f159892d78c7003945ba55ef4b84
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39577876"
---
# <a name="questions-about-the-azure-backup-agent"></a>關於 Azure 備份代理程式的問題
本文包含常見問題的解答，可協助您快速了解 Azure 備份代理程式元件。 在某些答案中，有具有完整資訊的文章連結。 您也可以在 [論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazureonlinebackup)中張貼有關 Azure 備份服務的問題。

## <a name="configure-backup"></a>設定備份
### <a name="where-can-i-download-the-latest-azure-backup-agent-br"></a>哪裡可以下載最新的 Azure 備份代理程式？ <br/>
您可以下載最新的代理程式，以便從 [這裡](http://aka.ms/azurebackup_agent)備份 Windows Server、System Center DPM 或 Windows 用戶端。 如果您想要備份虛擬機器，請使用 VM 代理程式 (這會自動安裝適當的擴充功能)。 從 Azure 資源庫建立的虛擬機器上已經有 VM 代理程式。

### <a name="when-configuring-the-azure-backup-agent-i-am-prompted-to-enter-the-vault-credentials-do-vault-credentials-expire"></a>當設定 Azure 備份代理程式時，系統提示我要輸入保存庫認證。 保存庫認證是否過期？
是，保存庫認證將於 48 小時後過期。 若檔案已過期，請登入 Azure 入口網站，並從您的保存庫下載保存庫認證檔。

### <a name="what-types-of-drives-can-i-back-up-files-and-folders-from-br"></a>我可以從何種類型的磁碟機備份檔案和資料夾？ <br/>
您無法備份下列磁碟機/磁碟區：

* 卸除式媒體：所有備份項目來源必須報告為固定式。
* 唯讀磁碟區：必須為可寫入磁碟區，才能使磁碟區陰影複製服務 (VSS) 正常運作。
* 離線磁碟區：必須為線上磁碟區，才能使 VSS 正常運作。
* 網路共用：必須為本機磁碟區，才能使用線上備份來備份伺服器。
* 受 Bitlocker 保護的磁碟區：磁碟區必須先解除鎖定才能執行備份。
* 檔案系統識別碼：NTFS 是唯一支援的檔案系統。

### <a name="what-file-and-folder-types-can-i-back-up-from-my-serverbr"></a>我可以從伺服器備份何種類型的檔案和資料夾？<br/>
支援下列類型：

* 已加密
* 已壓縮
* 疏鬆
* 已壓縮 + 疏鬆
* 永久連結：不支援並略過
* 重新分析點：不支援並略過
* 已加密 + 疏鬆：不支援並略過
* 已壓縮資料流：不支援並略過
* 疏鬆資料流：不支援並略過

### <a name="can-i-install-the-azure-backup-agent-on-an-azure-vm-already-backed-by-the-azure-backup-service-using-the-vm-extension-br"></a>我可以在已由 Azure 備份服務所備份的 Azure VM 上使用 VM 擴充功能來安裝 Azure 備份代理程式嗎？ <br/>
當然。 Azure 備份使用 VM 擴充功能為 Azure VM 提供 VM 層級備份。 若要保護客體 Windows OS 上的檔案和資料夾﹐請在客體 Windows OS 上安裝 Azure 備份代理程式。

### <a name="can-i-install-the-azure-backup-agent-on-an-azure-vm-to-back-up-files-and-folders-present-on-temporary-storage-provided-by-the-azure-vm-br"></a>我可以在 Azure VM 上安裝 Azure 備份代理程式來備份 Azure VM 所提供的暫存儲存體中存在的檔案和資料夾嗎？ <br/>
是。 在客體 Windows OS 上安裝 Azure 備份代理程式，並將檔案和資料夾備份至暫存儲存體。 一旦抹除暫存儲存體資料，備份工作就會失敗。此外，如果暫存儲存體資料已遭刪除，您只能還原至非變動性儲存體。

### <a name="whats-the-minimum-size-requirement-for-the-cache-folder-br"></a>什麼是快取資料夾的最低大小需求？ <br/>
快取資料夾的大小可決定您正在備份的資料量。 相較於備份資料的大小總計，快取資料夾的磁碟區至少應該是 5-10% 的可用空間。 如果此磁碟區低於 5% 的可用空間，請增加磁碟區大小，或[將快取資料夾移至具有足夠可用空間的磁碟區](backup-azure-file-folder-backup-faq.md#backup)。

### <a name="how-do-i-register-my-server-to-another-datacenterbr"></a>我如何向其他資料中心註冊我的伺服器？<br/>
備份資料會傳送至保存庫的資料中心以進行註冊。 若要變更資料中心，最簡單的方式是將代理程式解除安裝並重新安裝，然後向所需資料中心的新保存庫進行註冊。

### <a name="does-the-azure-backup-agent-work-on-a-server-that-uses-windows-server-2012-deduplication-br"></a>Azure 備份代理程式是否在使用 Windows Server 2012 重複資料刪除的伺服器上運作？ <br/>
是。 當代理程式服務準備備份作業時，會將重複資料刪除的資料轉換成一般資料。 它接著會最佳化資料以備份，加密資料，然後將加密的資料傳送至線上備份服務。

## <a name="prerequisites-and-dependencies"></a>先決條件和相依性
### <a name="what-features-of-microsoft-azure-recovery-services-mars-agent-require-net-framework-452-and-higher"></a>Microsoft Azure 復原服務 (MARS) 代理程式的哪些功能需要 .NET Framework 4.5.2 和更新版本？
可讓個別檔案和資料夾從*復原資料*精靈還原的[立即還原](backup-azure-restore-windows-server.md#use-instant-restore-to-recover-data-to-the-same-machine)功能，需要 .NET Framework 4.5.2 或更高版本。

## <a name="backup"></a>Backup 
### <a name="how-do-i-change-the-cache-location-specified-for-the-azure-backup-agentbr"></a>如何變更為 Azure 備份代理程式指定的快取位置？<br/>
使用下列清單來變更快取位置。

1. 在提高權限的命令提示字元中執行下列命令以停止備份引擎：

    ```PS C:\> Net stop obengine``` 
  
2. 請勿移動檔案。 相反地，請將快取空間資料夾複製到具有足夠空間的其他磁碟機。 確認備份使用新的快取空間之後，就可以移除原始的快取空間。
3. 更新下列登錄項目，其具備新快取空間資料夾的路徑。<br/>

    | 登錄路徑 | 登錄金鑰 | 值 |
    | --- | --- | --- |
    | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config` |ScratchLocation |*「新的快取資料夾位置」* |
    | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider` |ScratchLocation |*「新的快取資料夾位置」* |

4. 在提高權限的命令提示字元中執行下列命令以重新啟動備份引擎：

    ```PS C:\> Net start obengine```

一旦在新的快取位置成功完成備份建立，您就可以移除原始的快取資料夾。


### <a name="where-can-i-put-the-cache-folder-for-the-azure-backup-agent-to-work-as-expectedbr"></a>我可以把快取資料夾放在何處，以讓 Azure 備份代理程式如預期般運作？<br/>
快取資料夾不建議使用下列位置︰

* 網路共用或卸除式媒體︰快取資料夾必須是在需要使用線上備份進行備份之伺服器的本機位置。 不支援網路位置或卸除式媒體，例如 USB 磁碟機
* 離線磁碟區︰快取資料夾必須在線上才能使用 Azure 備份代理程式進行預期的備份

### <a name="are-there-any-attributes-of-the-cache-folder-that-are-not-supportedbr"></a>快取資料夾是否有任何不受支援的屬性？<br/>
快取資料夾不支援下列屬性或其組合︰

* 已加密
* 已刪除重複資料
* 已壓縮
* 疏鬆
* 重新分析點

快取資料夾和中繼資料 VHD 都不具有 Azure 備份代理程式所需的屬性。

### <a name="is-there-a-way-to-adjust-the-amount-of-bandwidth-used-by-the-backup-servicebr"></a>是否有辦法調整備份服務所使用的頻寬量？<br/>
  是，使用備份代理程式中的 [變更屬性]  選項來調整頻寬。 您可以調整頻寬量以及使用該頻寬的時間。 如需逐步指示，請參閱**[啟用網路節流](backup-configure-vault.md#enable-network-throttling)**。

## <a name="manage-backups"></a>管理備份
### <a name="what-happens-if-i-rename-a-windows-server-that-is-backing-up-data-to-azurebr"></a>若重新命名正在將資料備份至 Azure 的 Windows 伺服器，則會發生什麼情況？<br/>
當您重新命名伺服器時，所有目前設定的備份都會停止。 向備份保存庫註冊伺服器的新名稱。 當您向保存庫註冊新名稱時，第一個備份作業是*完整*備份。 如果您需要復原已備份到採用舊伺服器名稱之保存庫的資料，請使用 [復原資料] 精靈中的 [[其他伺服器](backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine)] 選項。

### <a name="what-is-the-maximum-file-path-length-that-can-be-specified-in-backup-policy-using-azure-backup-agent-br"></a>使用 Azure 備份代理程式時，最多可在備份代理程式中指定多長的檔案路徑？ <br/>
Azure 備份代理程式依存於 NTFS。 [檔案路徑長度規格受限於 Windows API](https://msdn.microsoft.com/library/aa365247.aspx#fully_qualified_vs._relative_paths)。 如果您要保護之檔案的檔案路徑長度超過 Windows API 所允許的長度，請備份父資料夾或磁碟機。  

### <a name="what-characters-are-allowed-in-file-path-of-azure-backup-policy-using-azure-backup-agent-br"></a>使用 Azure 備份代理程式時，Azure 備份原則的檔案路徑中允許哪些字元？ <br>
 Azure 備份代理程式依存於 NTFS。 它可讓 [NTFS 支援的字元](https://msdn.microsoft.com/library/aa365247.aspx#naming_conventions) 成為檔案規格的一部分。 
 
### <a name="i-receive-the-warning-azure-backups-have-not-been-configured-for-this-server-even-though-i-configured-a-backup-policy-br"></a>我收到「Azure 備份尚未針對此伺服器設定」警告，即使我已設定備份原則 <br/>
當本機伺服器儲存的備份排程設定與備份保存庫儲存的設定不同時，會發生此警告。 當伺服器或設定已復原至已知的良好狀態時，備份排程可能會失去同步處理。 如果您收到此警告，請[重新設定備份原則](backup-azure-manage-windows-server.md)，然後 [立即執行備份] 以重新同步處理本機伺服器與 Azure。
