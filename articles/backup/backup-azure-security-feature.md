---
title: 利用安全性功能協助保護使用 Azure 備份的混合式備份
description: 了解如何使用 Azure 備份的安全性功能，讓備份更安全
services: backup
author: trinadhk
manager: vijayts
ms.service: backup
ms.topic: conceptual
ms.date: 06/08/2017
ms.author: trinadhk
ms.openlocfilehash: 32eba23202eca1c71c4f3b01dcb364281cb2eb60
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "40246429"
---
# <a name="security-features-to-help-protect-hybrid-backups-that-use-azure-backup"></a>利用安全性功能協助保護使用 Azure 備份的混合式備份
現在越來越重視安全性問題，例如惡意程式碼、勒索軟體和入侵。 這些安全性問題在成本和資料方面付出的代價很高。 為了防範這類攻擊，Azure 備份現在提供安全性功能來協助保護混合式備份。 本文說明如何使用 Azure 復原服務代理程式和 Azure 備份伺服器以啟用及使用這些功能。 這些功能包括：

- **預護**。 每當執行重要作業 (如變更複雜密碼) 時，就會多一道驗證。 這項驗證用來確保只有具備有效 Azure 認證的使用者，才能執行這類作業。
- **警示**。 每當執行重要作業 (如刪除備份資料) 時，就會傳送電子郵件通知給訂用帳戶管理員。 這封電子郵件可確保使用者迅速獲知這類動作。
- **復原**。 已刪除的備份資料會額外保留 14 天 (從刪除日期算起)。 這可確保指定時間週期內的資料復原能力，所以即使發生攻擊，也不會遺失任何資料。 而且會維護更多的復原點，以防範資料損毀。

> [!NOTE]
> 如果您使用基礎結構即服務 (IaaS) VM 備份，則不應該啟用安全性功能。 這些功能尚無法用於 IaaS VM 備份，所以啟用它們不會有任何影響。 只有當您使用下列項目，才應該啟用安全性功能︰ <br/>
>  * **Azure 備份代理程式**。 至少為代理程式 2.0.9052 版。 啟用這些功能之後，您應該升級為此版本的代理程式，以執行重要作業。 <br/>
>  * **Azure 備份伺服器**。 至少為 Azure 備份代理程式 2.0.9052 版與 Azure 備份伺服器 Update 1。 <br/>
>  * **System Center Data Protection Manager**。 至少為 Azure 備份代理程式 2.0.9052 版與 Data Protection Manager 2012 R2 UR12 或 Data Protection Manager 2016 UR2。 <br/> 


> [!NOTE]
> 這些功能僅適用於復原服務保存庫。 所有新建立的復原服務保存庫預設都已經啟用這些功能。 對於現有的復原服務保存庫，使用者可使用下節中提及的步驟來啟用這些功能。 這些功能啟用之後，它們會套用至所有向保存庫註冊的復原服務代理程式電腦、Azure 備份伺服器執行個體和 Data Protection Manager 伺服器。 啟用此設定是一次性動作，啟用之後便無法停用這些功能。
>

## <a name="enable-security-features"></a>啟用安全性功能
如果您在建立復原服務保存庫，您可以使用所有安全性功能。 如果您在使用現有的保存庫，請依照下列這些步驟啟用安全性功能︰

1. 使用您的 Azure 認證登入 Azure 入口網站。
2. 選取 [瀏覽]，然後輸入 [復原服務]。

    ![Azure 入口網站瀏覽選項的螢幕擷取畫面](./media/backup-azure-security-feature/browse-to-rs-vaults.png) <br/>

    復原服務保存庫清單隨即出現。 從此清單中選取保存庫。 選取的保存庫儀表板隨即開啟。
3. 從保存庫下顯示的項目清單中，按一下 [設定] 之下的 [屬性]。

    ![復原服務保存庫選項的螢幕擷取畫面](./media/backup-azure-security-feature/vault-list-properties.png)
4. 按一下 [安全性設定] 之下的 [更新]。

    ![復原服務保存庫屬性的螢幕擷取畫面](./media/backup-azure-security-feature/security-settings-update.png)

    更新連結會開啟 [安全性設定] 刀鋒視窗，其中提供功能的摘要並讓您啟用它們。
5. 從 [是否已設定 Azure Multi-Factor Authentication?] 下拉式清單中，選取一個值以確認您是否已啟用 [Azure Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md)。 如果已啟用，當您登入 Azure 入口網站時，系統會要求您從另一個裝置 (例如行動電話) 進行驗證。

   當您在備份中執行重要作業時，您必須輸入安全 PIN 碼 (可在 Azure 入口網站上取得)。 啟用 Multi-Factor Authentication 可多一道安全性。 只有具備有效 Azure 認證且從第二個裝置驗證的授權使用者，才能存取 Azure 入口網站。
6. 若要儲存安全性設定，請選取 [啟用]，然後按一下 [儲存]。 只有當您在上一個步驟從 [是否已設定 Azure Multi-Factor Authentication?] 清單中選取一個值之後，才能選取 [啟用]。

    ![安全性設定的螢幕擷取畫面](./media/backup-azure-security-feature/enable-security-settings-dpm-update.png)

## <a name="recover-deleted-backup-data"></a>復原已刪除的備份資料
如果已執行**停止備份並刪除備份資料**作業，備份會保留已刪除的備份資料額外 14 天，不會立即將它刪除。 若要在 14 天的期間內還原此資料，請執行下列步驟，根據您使用的工具而定：

**Azure 復原服務代理程式**使用者：

1. 如果原本進行備份的電腦仍可使用，請使用 Azure 復原服務中的[將資料復原到相同電腦](backup-azure-restore-windows-server.md#use-instant-restore-to-recover-data-to-the-same-machine)，以便從所有舊的復原點復原資料。
2. 如果此電腦無法使用，請使用[復原到其他電腦](backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine)，以使用另一部 Azure 復原服務電腦來取得此資料。

若為 **Azure 備份伺服器**使用者：

1. 如果原本進行備份的伺服器仍可使用，請重新保護已刪除的資料來源，並使用**復原資料**功能來從舊的復原點復原資料。
2. 如果此伺服器無法使用，請使用[從其他 Azure 備份伺服器復原資料](backup-azure-alternate-dpm-server.md)，以使用另一個 Azure 備份伺服器執行個體來取得此資料。

**Data Protection Manager** 使用者：

1. 如果原本進行備份的伺服器仍可使用，請重新保護已刪除的資料來源，並使用**復原資料**功能來從舊的復原點復原資料。
2. 如果此伺服器無法使用，請使用[新增外部 DPM](backup-azure-alternate-dpm-server.md)，以使用另一部 Data Protection Manager 伺服器來取得此資料。

## <a name="prevent-attacks"></a>防止攻擊
已新增檢查來確保只有有效使用者才能執行各種作業。 其中包括新增額外一道驗證，並維護復原用途所需的最小保留範圍。

### <a name="authentication-to-perform-critical-operations"></a>用來執行重要作業的驗證
針對重要作業新增額外一層驗證時，系統會提示您在執行**停止保護並刪除資料**和**變更複雜密碼**作業時輸入安全性 PIN 碼。

> [!NOTE]

> 目前並未針對 DPM 和 MABS 提供安全性 PIN 碼的支援以**停止保護並刪除資料**。 

若要收到這個 PIN：

1. 登入 Azure 入口網站。
2. 瀏覽至 [復原服務保存庫] > [設定] > [屬性]。
3. 在 [安全性 PIN 碼] 底下，按一下 [產生]。 這會開啟刀鋒視窗，其中包含要在 Azure 復原服務代理程式使用者介面中輸入的 PIN。
    此 PIN 碼的有效時間只有五分鐘，而且會在該期間後自動產生。

### <a name="maintain-a-minimum-retention-range"></a>維護最小的保留範圍
為了確保永遠都有有效的復原點可用，已新增下列檢查︰

- 若為每日保留，最少應該保留**七**天。
- 若為每週保留，最少應該保留**四**週。
- 若為每月保留，最少應該保留**三**個月。
- 若為每年保留，最少應該保留**一**年。

## <a name="notifications-for-critical-operations"></a>重要作業的通知
執行重要作業時，訂用帳戶管理員通常會收到一封含有作業相關詳細資訊的電子郵件通知。 您可以使用 Azure 入口網站，設定這些通知的其他電子郵件收件者。

本文所提的安全性功能可提供對付目標攻擊的防禦機制。 更重要的是，如果發生攻擊，這些功能可讓您復原資料。

## <a name="troubleshooting-errors"></a>錯誤疑難排解
| 作業 | 錯誤詳細資料 | 解決方案 |
| --- | --- | --- |
| 原則變更 |無法修改備份原則。 錯誤：由於發生內部服務錯誤 [0x29834]，導致目前的作業失敗。 請稍後再重試操作。 如果問題持續發生， 請連絡 Microsoft 支援服務。 |**原因：**<br/>當安全性設定已啟用，而您嘗試將保留範圍縮減至低於上面指定的最小值，且您使用不受支援的版本 (本文的第一個附註會指出支援的版本) 時，就會出現此錯誤。 <br/>**建議的動作：**<br/> 在此情況下，您應該將保留期限設定為高於指定的最小保留期限 (若是每日則 7 天、若是每週則 4 週、若是每月則 3 個月，若是每年則 1 年)，以繼續進行與原則有關的更新。 (選擇性) 較好的方法是更新備份代理程式、Azure 備份伺服器和/或 DPM UR，以利用所有安全性更新。 |
| 變更複雜密碼 |輸入的安全性 PIN 碼不正確。 (識別碼：100130) 請提供正確的安全性 PIN 碼以完成此作業。 |**原因：**<br/> 當您在執行重要作業 (例如變更複雜密碼) 時輸入無效或已到期的安全性 PIN 碼時，就會出現此錯誤。 <br/>**建議的動作：**<br/> 若要完成作業，您必須輸入有效的安全性 PIN 碼。 若要取得 PIN 碼，請登入 Azure 入口網站，並瀏覽至 [復原服務保存庫] > [設定] > [屬性] > [產生安全性 PIN 碼]。 請使用這個 PIN 碼來變更複雜密碼。 |
| 變更複雜密碼 |作業失敗。 識別碼：120002 |**原因：**<br/>當安全性設定已啟用，而您嘗試變更複雜密碼，且您使用不受支援的版本時，就會出現此錯誤 (本文的第一個附註會指出有效的版本)。<br/>**建議的動作：**<br/> 若要變更複雜密碼，您必須先將備份代理程式至少更新為最低版本 2.0.9052、將 Azure 備份伺服器至少更新為更新 1，和/或將 DPM 至少更新為 DPM 2012 R2 UR12 或 DPM 2016 UR2 (下面的下載連結)，然後輸入有效的安全性 PIN 碼。 若要取得 PIN 碼，請登入 Azure 入口網站，並瀏覽至 [復原服務保存庫] > [設定] > [屬性] > [產生安全性 PIN 碼]。 請使用這個 PIN 碼來變更複雜密碼。 |

## <a name="next-steps"></a>後續步驟
* [開始使用 Azure 復原服務保存庫](backup-azure-vms-first-look-arm.md)以啟用這些功能。
* [下載最新的 Azure 復原服務代理程式](http://aka.ms/azurebackup_agent)，以協助保護 Windows 電腦及防護備份資料以免遭受攻擊。
* [下載最新的 Azure 備份伺服器](https://aka.ms/latest_azurebackupserver)，以協助保護工作負載及防護備份資料以免遭受攻擊。
* [下載適用於 System Center 2012 R2 Data Protection Manager 的 UR12](https://support.microsoft.com/help/3209592/update-rollup-12-for-system-center-2012-r2-data-protection-manager) 或[下載適用於 System Center 2016 Data Protection Manager 的 UR2](https://support.microsoft.com/help/3209593/update-rollup-2-for-system-center-2016-data-protection-manager)，以協助保護工作負載及防護備份資料以免遭受攻擊。
