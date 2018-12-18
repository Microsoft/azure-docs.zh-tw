---
title: Azure AD Connect：從舊版升級 | Microsoft Docs
description: 說明將 Azure Active Directory Connect 升級至最新版本的不同方法，包括就地升級和變換移轉。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 31f084d8-2b89-478c-9079-76cf92e6618f
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Identity
ms.date: 07/18/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 17333f6a5e78fb6da607e93cf696bfc766daf3ae
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46309937"
---
# <a name="azure-ad-connect-upgrade-from-a-previous-version-to-the-latest"></a>Azure AD Connect：從舊版升級到最新版本
本主題說明各種可用於將 Azure Active Directory (Azure AD) Connect 安裝升級到最新版本的方法。 我們建議您讓自己的 Azure AD Connect 保持在最新版本。 當您進行大幅組態變更時，也會使用[變換移轉](#swing-migration)一節中的步驟。

如果您想要從 DirSync 升級，請改為參閱[從 Azure AD 同步作業工具 (DirSync) 升級](how-to-dirsync-upgrade-get-started.md)。

您可以使用幾種不同的策略來升級 Azure AD Connect。

| 方法 | 說明 |
| --- | --- |
| [自動升級](how-to-connect-install-automatic-upgrade.md) |對於使用快速安裝的客戶，這是最簡單的方法。 |
| [就地升級](#in-place-upgrade) |如果您只有一台伺服器，您可以在該伺服器上就地升級安裝。 |
| [變換移轉](#swing-migration) |如果您有兩台伺服器，可以將其中一台升級成新的版本或組態，然後在您準備好時變更作用中的伺服器。 |

如需權限資訊，請參閱[升級所需的權限](reference-connect-accounts-permissions.md#upgrade)。

> [!NOTE]
> 讓新的 Azure AD Connect 伺服器開始將變更同步處理至 Azure AD 之後，就不可再復原為使用 DirSync 或 Azure AD 同步。不支援從 Azure AD Connect 降級至舊版用戶端，包括 DirSync 和 Azure AD 同步，因為可能會導致 Azure AD 中發生遺失資料等問題。

## <a name="in-place-upgrade"></a>就地升級
就地升級適用於從 Azure AD Sync 或 Azure AD Connect 移轉， 但不適用於從 DirSync 移轉，也不適用於任何利用 Forefront Identity Manager (FIM) + Azure AD 連接器的解決方案。

您只有一台伺服器，且擁有的物件少於 100000 個時，這個方法是最適合您。 如果現成的同步處理規則有任何變更，升級後會發生完整匯入和完整同步處理。 此方法可確保將新的組態套用到系統中所有現有的物件。 這項執行可能需要幾個小時的時間，視同步化引擎作業範圍內的物件數目而定。 標準差異同步處理排程器 (預設每隔 30 分鐘同步處理一次) 會暫停，但密碼同步處理會繼續進行。 我們建議您在週末時進行就地升級。 如果新的 Azure AD Connect 版本對現成的組態沒有任何變更，則會改為啟動一般的差異匯入/同步處理。  
![就地升級](./media/how-to-upgrade-previous-version/inplaceupgrade.png)

如果您已對現成的同步處理規則進行變更，則系統會在升級時會將這些規則設定回預設組態。 如要確保您的組態在系統升級之後會保留下來，請確保依照[變更預設組態的最佳做法](how-to-connect-sync-best-practices-changing-default-configuration.md)所述來變更組態。

在就地升級期間，可能會傳入變更而必須在升級完成之後執行特定的同步處理活動 (包括「完整匯入」步驟和「完整同步處理」步驟)。 若要延遲這類活動，請參閱[如何延遲升級之後的完整同步處理](#how-to-defer-full-synchronization-after-upgrade)一節。

如果您使用 Azure AD Connect 搭配非標準連接器 (例如「一般 LDAP 連接器」和「一般 SQL 連接器」)，就必須在進行就地升級之後，在 [Synchronization Service Manager](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-service-manager-ui-connectors) 中重新整理對應的連接器設定。 如需有關如何重新整理連接器設定的詳細資料，請參閱[連接器版本發行歷程記錄 - 疑難排解](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-connector-version-history#troubleshooting)文章小節。 如果您未重新整理設定，該連接器的匯入和匯出執行步驟將無法正確運作。 您將會在應用程式事件記錄檔中收到下列訊息：*"Assembly version in AAD Connector configuration ("X.X.XXX.X") is earlier than the actual version ("X.X.XXX.X") of "C:\Program Files\Microsoft Azure AD Sync\Extensions\Microsoft.IAM.Connector.GenericLdap.dll"* (AAD 連接器設定中的組件版本 ("X.X.XXX.X") 比 "C:\Program Files\Microsoft Azure AD Sync\Extensions\Microsoft.IAM.Connector.GenericLdap.dll" 的實際版本舊)。

## <a name="swing-migration"></a>變換移轉
如果您的伺服器部署很複雜，或是您的物件很多，在使用中的系統上進行就地升級可能並不實際。 就某些客戶而言，此程序可能需花好幾天的時間，而在這段期間並不會處理任何差異變更。 當您打算對您的組態進行大幅變更，而且想要在這些變更推送至雲端前試用看看，也可以使用這個方法。

這類案例的建議方法是改用變換移轉。 您需要 (至少) 兩台伺服器，一台是作用中伺服器，而另一台是預備伺服器。 作用中伺服器 (在下圖中以藍色實線顯示) 會負責處理作用中的負載， 而預備伺服器 (在下圖中以紫色虛線顯示) 會為了新的版本或組態預作準備。 準備就緒時，這台伺服器的狀態會變成作用中。 先前的作用中伺服器現已安裝舊的版本或組態，而您會將它變成預備伺服器，然後進行升級。

在兩部伺服器可以使用不同的版本。 例如，打算解除任務的作用中伺服器可以使用 Azure AD 同步，而新的預備伺服器可以使用 Azure AD Connect。 如果您使用變換移轉來開發新的組態，最好兩部伺服器上有相同的版本。  
![預備伺服器](./media/how-to-upgrade-previous-version/stagingserver1.png)

> [!NOTE]
> 有些客戶在此情況下，會使用三或四台伺服器。 在預備伺服器升級後，您沒有備份伺服器可進行[災害復原](how-to-connect-sync-operations.md#disaster-recovery)。 如果您有三或四台伺服器，您可以準備一組有新版本的主要/待命伺服器，以確保永遠都有預備伺服器可立即接管工作。

下列步驟也適用於從 Azure AD Sync 升級的案例，或是利用 FIM + Azure AD 連接器的解決方案。 這些步驟並不適用於 DirSync，但[升級 Azure Active Directory 同步作業 (DirSync)](how-to-dirsync-upgrade-get-started.md) 中可找到含 DirSync 適用步驟的相同變換移轉方法 (也稱為平行部署)。

### <a name="use-a-swing-migration-to-upgrade"></a>使用變換移轉進行升級
1. 如果兩部伺服器都使用 Azure AD Connect，而您打算只進行一項組態變更，請確定作用中伺服器和預備伺服器均使用相同的版本。 這樣一來，您稍後更容易比較出差異。 如果您從 Azure AD 同步進行升級，這些伺服器會有不同的版本。 如果您從舊版 Azure AD Connect 進行升級，最好從使用相同版本的兩部伺服器著手，但並非必要。
2. 如果您已建立自訂組態，但預備伺服器並沒有此組態，請遵循[將自訂組態從作用中伺服器移到預備伺服器](#move-a-custom-configuration-from-the-active-server-to-the-staging-server)之下的步驟進行。
3. 如果您要從舊版的 Azure AD Connect 升級，請將預備伺服器升級到最新版本。 如果您要從 Azure AD 同步進行移動，請在預備伺服器上安裝 Azure AD Connect。
4. 讓同步處理引擎在預備伺服器上執行完整匯入及完整同步處理的作業。
5. 請使用[驗證伺服器的組態](how-to-connect-sync-operations.md#verify-the-configuration-of-a-server)中「驗證」之下的步驟，以確認新的組態並未造成任何非預期的變更。 如果發現預期以外的變更，請遵循相關步驟，加以修正、執行匯入及同步處理作業，然後驗證資料，直到資料看起來沒問題為止。
6. 將預備伺服器切換成作用中伺服器。 這就是[驗證伺服器組態](how-to-connect-sync-operations.md#verify-the-configuration-of-a-server)中的最後一個步驟「切換作用中伺服器」。
7. 如果您要升級 Azure AD Connect，請將目前處於預備模式的伺服器升級到最新版本。 依照與先前相同的步驟，來為資料及組態升級。 如果您是從 Azure AD Sync 進行升級，現在可以關閉舊伺服器並解除任務。

### <a name="move-a-custom-configuration-from-the-active-server-to-the-staging-server"></a>將自訂組態從作用中伺服器移到預備伺服器
如果您已變更作用中伺服器的組態，就必須確保相同的變更會套用到預備伺服器。 若要協助進行這項操作，您可以使用 [Azure AD Connect 組態文件產生器](https://github.com/Microsoft/AADConnectConfigDocumenter)。

您可以移動使用 PowerShell 建立的自訂同步處理規則。 您必須用相同的方式將其他變更套用在兩個系統上，而無法移轉變更。 [組態文件產生器](https://github.com/Microsoft/AADConnectConfigDocumenter)可協助您比較兩個系統，以確保它們完全相同。 此工具也可協助自動執行本節中的步驟。

您需要用相同的方式在兩部伺服器上設定下列事項︰

* 對相同樹系的連線
* 任何網域及 OU 篩選
* 相同的選用功能，例如密碼同步處理及密碼回寫功能

**移動自訂同步處理規則**  
若要移動自訂同步處理規則，請執行下列作業：

1. 開啟作用中伺服器上的 [同步處理規則編輯器]  。
2. 選取自訂規則。 按一下 [匯出]。 此時會出現一個 [記事本] 視窗。 將暫存檔案儲存成副檔名為 PS1 的檔案。 這會讓該檔案變成 PowerShell 指令碼。 將該 PS1 檔案複製到預備伺服器上。  
   ![同步處理規則匯出](./media/how-to-upgrade-previous-version/exportrule.png)
3. 預備伺服器的連接器 GUID 會跟作用中伺服器的不一樣，必須加以變更。 若要取得 GUID，請啟動 [同步處理規則編輯器]、選取某個代表同一個已連線系統的現成規則，然後按一下 [匯出]。 請用預備伺服器的 GUID 取代 PS1 檔中的 GUID。
4. 在 PowerShell 命令提示字元中執行 PS1 檔， 以便在預備伺服器上建立自訂同步處理規則。
5. 針對所有自訂規則重複此步驟。

## <a name="how-to-defer-full-synchronization-after-upgrade"></a>如何延遲升級之後的完整同步處理
在就地升級期間，可能會傳入變更而必須執行特定的同步處理活動 (包括「完整匯入」步驟和「完整同步處理」步驟)。 例如，連接器結構描述變更時，需要在受影響的連接器上執行**完整匯入**步驟，而全新的同步化規則變更則需要執行**完整同步處理**步驟。 在升級期間，Azure AD Connect 會決定需要何種同步處理活動，並將其記錄為「覆寫」。 在下列同步處理週期內，同步處理排程器會挑出這些覆寫並執行。 覆寫成功執行之後就會移除。

在某些情況下，您可能不想在升級之後立即進行這些覆寫。 例如，您有很多個同步處理的物件，而且想要讓這些同步處理步驟在下班之後執行。 移除這些覆寫：

1. 在升級期間，**清除** [在設定完成時開始同步處理程序] 選項。 這會停用同步處理排程器，並且避免在移除覆寫之前自動開始同步處理週期。

   ![DisableFullSyncAfterUpgrade](./media/how-to-upgrade-previous-version/disablefullsync01.png)

2. 升級完成之後，執行下列 Cmdlet 來找出已新增哪些覆寫：`Get-ADSyncSchedulerConnectorOverride | fl`

   >[!NOTE]
   > 覆寫是連接器專屬。 在下列範例中，「完整匯入」步驟和「完整同步處理」步驟已新增至內部部署 AD 連接器與 Azure AD 連接器。

   ![DisableFullSyncAfterUpgrade](./media/how-to-upgrade-previous-version/disablefullsync02.png)

3. 記下現有已新增的覆寫。
   
4. 若要在任意連接器上同時移除完整匯入和完整同步處理的覆寫，請執行下列 Cmdlet：`Set-ADSyncSchedulerConnectorOverride -ConnectorIdentifier <Guid-of-ConnectorIdentifier> -FullImportRequired $false -FullSyncRequired $false`

   若要移除所有連接器上的覆寫，請執行下列 PowerShell 指令碼：

   ```
   foreach ($connectorOverride in Get-ADSyncSchedulerConnectorOverride)
   {
       Set-ADSyncSchedulerConnectorOverride -ConnectorIdentifier $connectorOverride.ConnectorIdentifier.Guid -FullSyncRequired $false -FullImportRequired $false
   }
   ```

5. 若要繼續排程器，請執行下列 Cmdlet：`Set-ADSyncScheduler -SyncCycleEnabled $true`

   >[!IMPORTANT]
   > 請記得儘早執行必要的同步處理步驟。 您可以使用 Synchronization Service Manager 來手動執行這些步驟，或使用 Set-ADSyncSchedulerConnectorOverride Cmdlet 將覆寫加回。

若要在任意連接器上同時新增完整匯入和完整同步處理的覆寫，請執行下列 Cmdlet：`Set-ADSyncSchedulerConnectorOverride -ConnectorIdentifier <Guid> -FullImportRequired $true -FullSyncRequired $true`

## <a name="troubleshooting"></a>疑難排解
下一節包含遇到 Azure AD Connect 升級問題時，您可以使用的疑難排解和資訊。

### <a name="azure-active-directory-connector-missing-error-during-azure-ad-connect-upgrade"></a>Azure AD Connect 升級期間發生 Azure Active Directory 連接器遺漏錯誤

當您從舊版升級 Azure AD Connect 時，您可能會在升級開始時遇到下列錯誤 

![Error](./media/how-to-upgrade-previous-version/error1.png)

因為識別碼為 b891884f-051e-4a83-95af-2544101c9083 的 Azure Active Directory 連接器不存在於目前的 Azure AD Connect 設定中，所以發生此錯誤。 若要確認情況就是這樣，請開啟 PowerShell 視窗，並執行 Cmdlet `Get-ADSyncConnector -Identifier b891884f-051e-4a83-95af-2544101c9083`

```
PS C:\> Get-ADSyncConnector -Identifier b891884f-051e-4a83-95af-2544101c9083
Get-ADSyncConnector : Operation failed because the specified MA could not be found.
At line:1 char:1
+ Get-ADSyncConnector -Identifier b891884f-051e-4a83-95af-2544101c9083
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ReadError: (Microsoft.Ident...ConnectorCmdlet:GetADSyncConnectorCmdlet) [Get-ADSyncConne
   ctor], ConnectorNotFoundException
    + FullyQualifiedErrorId : Operation failed because the specified MA could not be found.,Microsoft.IdentityManageme
   nt.PowerShell.Cmdlet.GetADSyncConnectorCmdlet

```

此 PowerShell Cmdlet 會回報**找不到指定 MA** 錯誤。

之所以發生這種情況，原因是目前的 Azure AD Connect 設定不支援用於升級。 

如果您想要安裝較新版的 Azure AD Connect：關閉 [Azure AD Connect 精靈]、將現有的 Azure AD Connect 解除安裝，並使用較新的 Azure AD Connect 來執行全新安裝。



## <a name="next-steps"></a>後續步驟
深入了解[將內部部署身分識別與 Azure Active Directory 整合](whatis-hybrid-identity.md)。
