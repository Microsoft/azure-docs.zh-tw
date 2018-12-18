---
title: 將備份保存庫升級為 Azure 備份的復原服務保存庫
description: 將備份保存庫升級為復原服務保存庫，以取得新功能，例如資源管理員 VM 的備份、增強的安全性、Windows 伺服器的 VMware VM 備份和系統狀態備份
services: backup
author: trinadhk
manager: vijayts
ms.service: backup
ms.topic: conceptual
ms.date: 02/10/2017
ms.author: trinadhk
ms.openlocfilehash: 358c7b0e61d1c85e236007c2495941a31502431a
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34606854"
---
# <a name="backup-vault-upgraded-to-recovery-services-vault"></a>將備份保存庫升級為復原服務保存庫
本文提供復原服務保存庫提供服務的概觀、將現有備份保存庫升級為復原服務保存庫的常見問題集，與升級後步驟。 復原服務保存庫是您存放備份資料之備份保存庫的 Azure Resource Manager 對等項目。 資料通常是資料的副本，或是虛擬機器 (VM)、工作負載、伺服器或工作站的設定資訊，無論是內部部署或是在 Azure 中。

## <a name="what-is-a-recovery-services-vault"></a>什麼是復原服務保存庫？
復原服務保存庫是 Azure 中的線上儲存實體，用來保存備份副本、復原點及備份原則等資料。 您可以使用復原服務保存庫來保存各種 Azure 服務 (例如 IaaS VM (Linux 或 Windows) 和 Azure SQL Database) 的備份資料。 復原服務保存庫支援 System Center DPM、Windows Server、Azure 備份伺服器等等。 復原服務保存庫可輕鬆地組織您的備份資料，同時可減輕管理負擔。

## <a name="comparing-recovery-services-vaults-and-backup-vaults"></a>比較復原服務保存庫與備份保存庫
復原服務保存庫是以 Azure 的 Azure Resource Manager 模型作為基礎，而備份保存庫則是以 Azure Service Manager 模型為作為基礎。 當您將備份保存庫升級至復原服務保存庫時，備份資料在升級程序期間及升級程序之後都會維持不變。 復原服務保存庫可提供備份保存庫無法使用的功能，例如︰

- **可協助保護備份資料安全的增強功能**︰透過復原服務保存庫，Azure 備份能提供安全性功能來保護雲端備份。 這些安全性功能可確保您能保護您的備份，並安全地從雲端備份將資料復原，即使生產和備份伺服器遭到入侵也一樣。 [深入了解](backup-azure-security-feature.md)

- **將您的混合式 IT 環境集中監視**︰透過復原服務保存庫，您不只可以監視 [Azure IaaS VM](backup-azure-manage-vms.md)，還可以從中央入口網站監視[內部部署資產](backup-azure-manage-windows-server.md#manage-backup-items)。 [深入了解](http://azure.microsoft.com/blog/alerting-and-monitoring-for-azure-backup)

- **角色型存取控制 (RBAC)**：RBAC 提供 Azure 中的更細緻存取權管理。 [Azure 提供各種內建角色](../role-based-access-control/built-in-roles.md)，且 Azure Backup 有三個[內建的角色可用來管理復原點](backup-rbac-rs-vault.md)。 復原服務保存庫與 RBAC 相容，且會對一組定義之使用者角色的備份和還原存取權限加以限制。 [深入了解](backup-rbac-rs-vault.md)

- **保護 Azure 虛擬機器的所有設定**︰復原服務保存庫會保護以 Resource Manager 為基礎的 VM，包括進階磁碟、受控磁碟及加密的 VM。 將備份保存庫升級至復原服務保存庫，讓您有機會可將以 Service Manager 為基礎的 VM 升級至以 Resource Manager 為基礎的 VM。 在升級保存庫時，您可以保留以 Service Manager 為基礎的 VM 復原點，並設定已升級 (已啟用 Resource Manager) 的 VM 保護。 [深入了解](http://azure.microsoft.com/blog/azure-backup-recovery-services-vault-ga)

- **IaaS VM 的立即還原**︰您可以使用復原服務保存庫，從 IaaS VM 還原檔案和資料夾，而非還原整個 VM，這樣可加速還原時間。 IaaS VM 的立即還原適用於 Windows 和 Linux VM。 [深入了解](http://azure.microsoft.com/blog/instant-file-recovery-from-azure-linux-vm-backup-using-azure-backup-preview)

> [!NOTE]
> 如果您向備份保存庫註冊的項目具有 2.0.9083.0 之前的 MARS 代理程式，[下載最新的 MARS 代理程式]( http://download.microsoft.com/download/F/4/B/F4B06356-150F-4DB0-8AD8-95B4DB4BBF7C/MARSAgentInstaller.exe)版本，以獲得復原服務保存庫所有功能的優點。 
> 

## <a name="managing-your-recovery-services-vaults"></a>管理復原服務保存庫
下列畫面所顯示的新復原服務保存庫，是在 Azure 入口網站中從備份保存庫進行升級的。 升級過的保存庫會顯示在名為 “Default-RecoveryServices-ResourceGroup-geo” 的預設資源群組中。 範例：如果您的備份保存庫位於美國西部，它會在名為 Default-RecoveryServices-ResourceGroup-westus 的預設 RG 中。
> [!NOTE]
> 針對 CPS 標準客戶，在保存庫升級之後，資源群組不會變更，維持與升級之前相同。

第一個畫面所顯示的保存庫儀表板，會顯示保存庫的金鑰實體。
![從備份保存庫升級的復原服務保存庫範例](./media/backup-azure-upgrade-backup-to-recovery-services/upgraded-rs-vault-in-dashboard.png)

第二個畫面所顯示的可用說明連結，會協助您開始使用復原服務保存庫。

![[快速入門] 刀鋒視窗中的說明連結](./media/backup-azure-upgrade-backup-to-recovery-services/quick-start-w-help-links.png)

## <a name="post-upgrade-steps"></a>升級後步驟
復原服務保存庫支援在備份原則中指定時區資訊。 在保存庫順利升級後，請從 [保存庫設定] 功能表移至 [備份] 原則，並更新保存庫中所設定之各個原則的時區資訊。 此畫面會顯示您在建立原則時使用每個當地時區所指定的備份排程時間。 

## <a name="enhanced-security"></a>強化的安全性
當備份保存庫升級為復原服務保存庫時，系統會自動開啟該保存庫的安全性設定。 當安全性設定開啟時，某些作業 (例如刪除備份或變更複雜密碼) 會需要 [Azure Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md) PIN 碼。 如需強化的安全性的詳細資訊，請參閱[用來保護混合式備份的安全性功能](backup-azure-security-feature.md)一文。 開啟強化的安全性時，在您從保存庫中刪除復原點資訊後，資料最多保留 14 天的時間。 客戶需支付此安全性資料的儲存體費用。 安全性資料保留原則適用於 Azure 備份代理程式、Azure 備份伺服器和 System Center Data Protection Manager 所備份的復原點。 

## <a name="gather-data-on-your-vault"></a>在保存庫上收集資料
在升級為復原服務保存庫後，請為 Azure 備份設定報告 (適用於 IaaS VM 和 Microsoft Azure 復原服務代理程式)，並使用 Power BI 來存取報告。 如需有關如何收集資料的其他資訊，請參閱[設定 Azure 備份報告](backup-azure-configure-reports.md)一文。

## <a name="frequently-asked-questions"></a>常見問題集

**升級計畫是否會影響我進行中的備份？**</br>
編號 在升級期間和升級之後，進行中的備份持續不受干擾。

**對於我現有的工具來說，此升級有何意義？**</br>
您必須將現有自動化功能或工具更新為 Resource Manager 部署模型，以確保它在移轉之後仍可繼續運作。 如需了解 [Resource Manager 部署模型](backup-client-automation.md)，請參閱 PowerShell Cmdlet 參考。

**升級之後可以復原嗎？**</br>
編號 將資源成功升級之後，即不支援復原。

**在升級後可以檢視我的傳統保存庫嗎？**</br>
編號 您在升級後無法檢視或管理您的傳統保存庫。 您只能將新的 Azure 入口網站用於保存庫的所有管理動作。

**為什麼我看不到升級的保存庫受到 MARS 代理程式保護的伺服器？**</br>
您必須安裝最新的 MARS 代理程式，才能看到在您的保存庫中受到 MARS 代理程式保護的所有伺服器。 您可以從[這裡]( http://download.microsoft.com/download/F/4/B/F4B06356-150F-4DB0-8AD8-95B4DB4BBF7C/MARSAgentInstaller.exe)下載最新版本的代理程式。

**我在升級之後看不到受到 MARS 代理程式保護之伺服器的備份原則**</br>
保存庫的備份原則可能過期，因此無法同步處理至升級的保存庫。 請更新原則，以確保您能夠繼續在升級的保存庫中看到原則。
若要更新原則，請移至 MARS 代理程式並更新設定的備份原則。

**為什麼我無法在升級之後更新我的備份原則？**</br>
當您位於舊的備份代理程式，並且選取比允許的最小值還小的最小保留期間時，就會發生這種情況。 當備份保存庫升級為復原服務保存庫時，系統會自動開啟該保存庫的安全性設定。 為了確保一律有可用的有效復原點數目，有一些最小保留期間需要維持，作為安全性功能。 如需詳細資訊，請參閱[這裡](backup-azure-security-feature.md)。
此外，您必須將 Azure 備份代理程式更新為最新版本，以獲得 Azure 備份最新功能的優點。

**我已更新我的代理程式，但是在升級過後一天，仍然看不到任何物件進行同步處理**</br>
請檢查您是否向多個保存庫註冊同一部電腦。 請確定您查看的保存庫與向 MARS 代理程式註冊的保存庫相同。 若要找出向您的 MARS 代理程式註冊的保存庫，請開啟 Windows 登錄並且檢查 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config 底下 ServiceResourceName 機碼的值。向該 MARS 代理程式註冊的保存庫會在這裡顯示。 如果在您的系統中看不到 ServiceResourceName 機碼，請將 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config 底下 ResourceId 和 MachineId 機碼的值傳送給我們，我們會協助您解決這個問題。

**為什麼我在升級之後看不到資源的作業資訊？**</br>
當您將備份保存庫升級至復原服務保存庫時，會取得監視備份 (MARS 代理程式和 IaaS) 的新功能。 監視資訊最多需花費 12 小時才能與服務進行同步處理。

**如何回報問題？**</br>
如果保存庫升級過程中發生任何失敗，請記下錯誤所列出的 OperationId。 Microsoft 支援服務會積極解決問題。 您可以尋求協助，或將包含訂用帳戶識別碼、保存庫名稱和 OperationId 的電子郵件傳送至 rsvaultupgrade@service.microsoft.com。 我們會試著盡快解決問題。 除非 Microsoft 明確指示，否則請勿重試作業。

## <a name="next-steps"></a>後續步驟
使用下列文章來：</br>
[備份 IaaS VM](backup-azure-arm-vms-prepare.md)</br>
[備份 Azure 備份伺服器](backup-azure-microsoft-azure-backup.md)</br>
[備份 Windows Server](backup-configure-vault.md)
