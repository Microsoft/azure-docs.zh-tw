---
title: 將備份保存庫升級至復原服務保存庫
description: 將 Azure 備份保存庫升級至復原服務保存庫的指示和支援資訊。
services: backup
author: markgalioto
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 1/4/2018
ms.author: sogup
ms.openlocfilehash: 551bc2aa4ff80feb3f28b5698e25dfd1b03dc870
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34607332"
---
# <a name="upgrade-a-backup-vault-to-a-recovery-services-vault"></a>將備份保存庫升級至復原服務保存庫

本文說明如何將備份保存庫升級至復原服務保存庫。 升級程序不會影響任何執行中的備份作業，且不會遺失備份資料。 將備份保存庫升級至復原服務保存庫的主要理由：
- 備份保存庫的所有功能都會保留在復原服務保存庫中。
- 復原服務保存庫較備份保存庫具有更多功能，包括更完善的安全性、整合式監控、更快速還原和項目等級還原。
- 從經過改良和簡化的入口網站管理備份項目。
- 新功能只適用於復原服務保存庫。

## <a name="impact-to-operations-during-upgrade"></a>升級期間對作業的影響

將備份保存庫升級至復原服務保存庫時，您的資料平面作業不會受到影響。 所有備份作業都會照常繼續進行，且任何使用中的還原作業都會繼續而不受中斷。 升級期間，管理作業會發生短暫的停機時間，且您無法保護新的項目或建立臨機操作備份作業。 升級期間不會執行 IaaS VM 的還原作業。 保存庫升級需要一小時才能完成。 在完成後，復原服務保存庫會取代備份保存庫。

## <a name="changes-to-your-automation-and-tool-after-upgrading"></a>升級之後對自動化及工具的變更

當您在準備保存庫升級的基礎結構時，必須更新您現有的自動化或工具，確保它在升級之後會繼續運作。
如需了解 [Resource Manager 部署模型](backup-client-automation.md)，請參閱 PowerShell Cmdlet 參考。


## <a name="before-you-upgrade"></a>您在升級之前

您在將備份保存庫升級至復原服務保存庫之前，請檢查下列問題。

- **代理程式最低版本**︰若要升級您的保存庫，請確定 Microsoft Azure 復原服務 (MARS) 代理程式至少是 2.0.9083.0 版。 如果 MARS 代理程式低於 2.0.9083.0 版，則在開始升級程序前，請先更新代理程式。
- **以執行個體為基礎的計費模型**︰復原服務保存庫僅支援以執行個體為基礎的計費模型。 如果您的備份保存庫是使用舊版以儲存體為基礎的計費模型，請在升級期間轉換計費模型。
- **沒有持續進行的備份設定作業**︰在升級期間，會限制存取管理平面。 完成所有的管理平面動作，然後再開始升級。

## <a name="using-powershell-scripts-to-upgrade-your-vaults"></a>使用 PowerShell 指令碼來升級您的保存庫

您可以使用 PowerShell 指令碼，將備份保存庫升級至復原服務保存庫。 檢查您擁有觸發保存庫升級所需的 PowerShell 元件。 備份保存庫的 PowerShell 指令碼不適用於復原服務保存庫。 準備好您的環境來升級保存庫︰

1. 安裝或將 [Windows Management Framework (WMF) 升級至第 5 版](https://www.microsoft.com/download/details.aspx?id=50395)或以上版本。
2. [安裝 Azure PowerShell MSI](https://github.com/Azure/azure-powershell/releases/download/v3.8.0-April2017/azure-powershell.3.8.0.msi)。
3. 下載 [PowerShell 指令碼](https://aka.ms/vaultupgradescript2)來升級您的保存庫。

### <a name="run-the-powershell-script"></a>執行 PowerShell 指令碼

使用下列指令碼來升級您的保存庫。 下列範例指令碼具有參數的說明。

RecoveryServicesVaultUpgrade-1.0.2.ps1 **-SubscriptionID** `<subscriptionID>` **-VaultName** `<vaultname>` **-Location** `<location>` **-ResourceType** `BackupVault` **-TargetResourceGroupName** `<rgname>`

**SubscriptionID** - 正在進行升級的保存庫之訂用帳戶識別碼編號。<br/>
**VaultName** - 正在進行升級之備份保存庫的名稱。<br/>
**Location** - 正在進行升級之保存庫的位置。<br/>
**ResourceType** - 使用 BackupVault。<br/>
**TargetResourceGroupName** - 由於您要將保存庫升級至以 Resource Manager 為基礎的部署，因此請指定「資源群組」。 您可以使用現有的資源群組，或提供新的名稱來建立一個新的群組。 如果您將資源群組的名稱拼錯，可以建立新的資源群組。 若要深入了解資源群組，請閱讀此[資源群組概觀](../azure-resource-manager/resource-group-overview.md#resource-groups)。

>[!NOTE]
> 資源群組名稱具有條件約束。 請務必遵循指導方針；若無法這樣做，可能導致保存庫升級失敗。
>
>執行指令碼期間，**Azure 美國政府**客戶需要將環境設為「AzureUSGovernment」。
>執行指令碼期間，**Azure 中國**客戶需要將環境設為「AzureChinaCloud」。

下列程式碼片段是您 PowerShell 命令的範例︰

```
RecoveryServicesVaultUpgrade.ps1 -SubscriptionID 53a3c692-5283-4f0a-baf6-49412f5ebefe -VaultName "TestVault" -Location "Australia East" -ResourceType BackupVault -TargetResourceGroupName "ContosoRG"
```

您也可以執行不含任何參數的指令碼，系統會要求您提供所有必要參數的輸入。

PowerShell 指令碼會提示您輸入認證。 輸入您的認證兩次︰一次為 Service Manager 帳戶，第二次為 Resource Manager 帳戶。

### <a name="pre-requisites-checking"></a>必要條件檢查
一旦您輸入 Azure 認證後，Azure 就會檢查您的環境是否符合下列必要條件︰

- **代理程式最低版本** - 將備份保存庫升級至復原服務保存庫需要至少 2.0.9083.0 版的 MARS 代理程式。 如果您的項目所登錄之備份保存庫是包含 2.0.9083.0 版以前的代理程式，必要條件檢查就會失敗。 如果必要條件檢查失敗，請更新代理程式，然後嘗試再次升級保存庫。 您可以從 [http://download.microsoft.com/download/F/4/B/F4B06356-150F-4DB0-8AD8-95B4DB4BBF7C/MARSAgentInstaller.exe](http://download.microsoft.com/download/F/4/B/F4B06356-150F-4DB0-8AD8-95B4DB4BBF7C/MARSAgentInstaller.exe) 下載最新版本的代理程式。
- **持續的設定作業**︰如果有人要設定的是已設定為要升級之備份保存庫的作業，或是要將項目進行登錄，必要條件檢查就會失敗。 完成設定或完成登錄項目，然後再開始進行保存庫升級程序。
- **以儲存體為基礎的計費模型**︰復原服務保存庫支援以執行個體為基礎的計費模型。 如果您執行的保存庫升級是在使用以儲存體為基礎之計費模型的備份保存庫上，系統會提示您升級計費模型以及保存庫。 否則，您可以先更新計費模型，然後再執行保存庫升級。
- 識別復原服務保存庫的資源群組。 若要充分利用 Resource Manager 部署功能，您必須將復原服務保存庫放入資源群組。 如果您不知道要使用哪一個資源群組，請提供名稱，升級程序就會為您建立資源群組。 升級程序也會將保存庫與新的資源群組建立關聯。

升級程序完成檢查必要條件後，程序會提示您開始進行保存庫升級。 在確認之後，升級程序通常需要大約 15 至 20 分鐘的時間才能完成，視您保存庫的大小而定。 如果您的是大型保存庫，升級可能需要長達 90 分鐘。

## <a name="managing-your-recovery-services-vaults"></a>管理復原服務保存庫

下列畫面所顯示的新復原服務保存庫，是在 Azure 入口網站中從備份保存庫進行升級的。 第一個畫面所顯示的保存庫儀表板，會顯示保存庫的金鑰實體。

![從備份保存庫升級的復原服務保存庫範例](./media/backup-azure-upgrade-backup-to-recovery-services/upgraded-rs-vault-in-dashboard.png)

第二個畫面所顯示的可用說明連結，會協助您開始使用復原服務保存庫。

![[快速入門] 刀鋒視窗中的說明連結](./media/backup-azure-upgrade-backup-to-recovery-services/quick-start-w-help-links.png)

## <a name="post-upgrade-steps"></a>升級後步驟
復原服務保存庫支援在備份原則中指定時區資訊。 在保存庫順利升級後，請從 [保存庫設定] 功能表移至 [備份] 原則，並更新保存庫中所設定之各個原則的時區資訊。 此畫面會顯示您在建立原則時使用每個當地時區所指定的備份排程時間。 

## <a name="enhanced-security"></a>強化的安全性

當備份保存庫升級為復原服務保存庫時，系統會自動開啟該保存庫的安全性設定。 當安全性設定開啟時，某些作業 (例如刪除備份或變更複雜密碼) 會需要 [Azure Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md) PIN 碼。 如需強化的安全性的詳細資訊，請參閱[用來保護混合式備份的安全性功能](backup-azure-security-feature.md)一文。 

開啟強化的安全性時，在您從保存庫中刪除復原點資訊後，資料最多保留 14 天的時間。 客戶需支付此安全性資料的儲存體費用。 安全性資料保留原則適用於 Azure 備份代理程式、Azure 備份伺服器和 System Center Data Protection Manager 所備份的復原點。 

## <a name="gather-data-on-your-vault"></a>在保存庫上收集資料

在升級為復原服務保存庫後，請為 Azure 備份設定報告 (適用於 IaaS VM 和 Microsoft Azure 復原服務 (MARS))，並使用 Power BI 來存取報告。 如需有關如何收集資料的其他資訊，請參閱[設定 Azure 備份報告](backup-azure-configure-reports.md)一文。

## <a name="frequently-asked-questions"></a>常見問題集

**升級計畫是否會影響我進行中的備份？**</br>
編號 在升級期間和升級之後，進行中的備份持續不受干擾。

**如果我不打算立即升級，那麼我的保存庫會怎麼樣？**</br>
所有新功能只適用於復原服務保存庫，因此您最好將保存庫升級。 2017 年 9 月 1 日起，Microsoft 會開始將備份保存庫自動升級為復原服務保存庫。 2017 年 11 月 30 日之後，您就無法再使用 PowerShell 建立備份保存庫。 您的保存庫會其間的任何時間自動升級。 Microsoft 建議您盡快升級您的保存庫。

**對於我現有的工具來說，此升級有何意義？**</br>
將您的工具升級至 Resource Manager 部署模型。 復原服務保存庫是打造來用於 Resource Manager 部署模型的。 因此，請務必要規劃 Resource Manager 部署模型，並考量保存庫的相異之處。 

**升級期間是否會頻繁停機？**</br>
依您所升級的資源數目而定。 對於較小型的部署 (數十個受保護的執行個體)，整個升級應該花費不到 20 分鐘的時間。 對於較大型的部署，應該花費最多一小時的時間。

**升級之後可以復原嗎？**</br>
編號 將資源成功升級之後，即不支援復原。

**我是否可以驗證訂用帳戶或資源，以查看是否能夠將它們升級？**</br>
是。 升級的第一個步驟是驗證這些資源是否能夠升級。 如果必要條件驗證失敗，您將會收到無法完成升級的所有原因相關訊息。

**可以升級以 CSP 為基礎的備份保存庫嗎？**</br>
編號 您目前無法升級以 CSP 為基礎的備份保存庫。 我們會在後續版本中新增以 CSP 為基礎之備份保存庫的升級支援。

**在升級後可以檢視我的傳統保存庫嗎？**</br>
編號 您在升級後無法檢視或管理您的傳統保存庫。 您只能將新的 Azure 入口網站用於保存庫的所有管理動作。

**我升級失敗，但要求進行更新之代理程式所在的電腦已不存在。這種情況下我該怎麼做？**</br>
如果您需要使用存放區作為這台電腦長期保存的備份，就無法升級保存庫。 在日後的版本中，我們會新增這類保存庫的升級支援。
如果您不需要再儲存此電腦的備份，請從保存庫取消登錄此電腦，然後再次嘗試升級。

**為什麼我在升級之後看不到資源的作業資訊？**</br>
當您將備份保存庫升級至復原服務保存庫時，會取得監視備份 (MARS 代理程式和 IaaS) 的新功能。 監視資訊最多需花費 12 小時才能與服務進行同步處理。

**如何回報問題？**</br>
如果保存庫升級過程中發生任何失敗，請記下錯誤所列出的 OperationId。 Microsoft 支援服務會積極解決問題。 您可以尋求協助，或將包含訂用帳戶識別碼、保存庫名稱和 OperationId 的電子郵件傳送至 rsvaultupgrade@service.microsoft.com。 我們會試著盡快解決問題。 除非 Microsoft 明確指示，否則請勿重試作業。


## <a name="next-steps"></a>後續步驟
使用下列文章︰</br>
[備份 IaaS VM](backup-azure-arm-vms-prepare.md)</br>
[備份 Azure 備份伺服器](backup-azure-microsoft-azure-backup.md)</br>
[備份 Windows Server](backup-configure-vault.md).
