---
title: 針對使用 Azure AD Connect 同步執行的密碼雜湊同步處理進行疑難排解 | Microsoft Docs
description: 本文提供有關如何針對密碼雜湊同步處理問題進行疑難排解的資訊。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/13/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 036933c6b6e86856871c5f59f08fea20a0343ad7
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310017"
---
# <a name="troubleshoot-password-hash-synchronization-with-azure-ad-connect-sync"></a>針對使用 Azure AD Connect 同步執行的密碼雜湊同步處理進行疑難排解
本主題提供如何針對密碼雜湊同步處理問題進行疑難排解的步驟。 如果密碼未如預期般同步，可能會影響一部分使用者或所有使用者。

對於 1.1.614.0 版或更新版本的 Azure Active directory (Azure AD) Connect 部署，請使用精靈中的疑難排解工作，針對密碼雜湊同步處理問題進行疑難排解：

* 如果是未同步任何密碼的問題，請參閱[未同步任何密碼：使用疑難排解工作進行疑難排解](#no-passwords-are-synchronized-troubleshoot-by-using-the-troubleshooting-task)一節。

* 如果是個別物件的問題，請參閱[一個物件未同步密碼：使用疑難排解工作進行疑難排解](#one-object-is-not-synchronizing-passwords-troubleshoot-by-using-the-troubleshooting-task)一節。

對於 1.1.524.0 版或更新版本的部署，有一個診斷 Cmdlet 可讓您針對密碼雜湊同步處理問題進行疑難排解：

* 如果是未同步任何密碼的問題，請參閱[未同步任何密碼：使用診斷 Cmdlet 進行疑難排解](#no-passwords-are-synchronized-troubleshoot-by-using-the-diagnostic-cmdlet)一節。

* 如果是個別物件的問題，請參閱[一個物件未同步密碼：使用診斷 Cmdlet 進行疑難排解](#one-object-is-not-synchronizing-passwords-troubleshoot-by-using-the-diagnostic-cmdlet)一節。

舊版的 Azure AD Connect 部署：

* 如果是未同步任何密碼的問題，請參閱[未同步任何密碼：手動疑難排解步驟](#no-passwords-are-synchronized-manual-troubleshooting-steps)一節。

* 如果是個別物件的問題，請參閱[一個物件未同步密碼：手動疑難排解步驟](#one-object-is-not-synchronizing-passwords-manual-troubleshooting-steps)一節。



## <a name="no-passwords-are-synchronized-troubleshoot-by-using-the-troubleshooting-task"></a>未同步任何密碼：使用疑難排解工作進行疑難排解
您可以使用疑難排解工作來查明未同步任何密碼的原因。

> [!NOTE]
> 疑難排解工作僅適用於 Azure AD Connect 1.1.614.0 版或更新版本。

### <a name="run-the-troubleshooting-task"></a>執行疑難排解工作
針對未同步任何密碼的問題進行疑難排解：

1. 在您的 Azure AD Connect 伺服器上，使用 [以系統管理員身分執行] 選項開啟新的 Windows PowerShell 工作階段。

2. 執行 `Set-ExecutionPolicy RemoteSigned` 或 `Set-ExecutionPolicy Unrestricted`。

3. 啟動 Azure AD Connect 精靈。

4. 瀏覽至 [其他工作] 頁面，選取 [疑難排解]，然後按 [下一步]。

5. 在 [疑難排解] 頁面上，按一下 [啟動]，以在 PowerShell 中啟動疑難排解功能表。

6. 在主功能表中，選取 [針對密碼雜湊同步處理進行疑難排解]。

7. 在子功能表中，選取 [密碼雜湊同步處理完全無法運作]。

### <a name="understand-the-results-of-the-troubleshooting-task"></a>了解疑難排解工作的結果
疑難排解工作會執行下列檢查：

* 確認已針對您的 Azure AD 租用戶啟用密碼雜湊同步處理功能。

* 確認 Azure AD Connect 伺服器不是處於預備模式。

* 針對每個現有的內部部署 Active Directory 連接器 (對應至現有的 Active Directory 樹系)：

   * 確認已啟用密碼雜湊同步處理功能。
   
   * 在「Windows 應用程式事件記錄檔」中搜尋密碼雜湊同步處理活動訊號事件。

   * 針對內部部署 Active Directory 連接器下的每個 Active Directory 網域：

      * 確認可從 Azure AD Connect 伺服器存取網域。

      * 確認內部部署 Active Directory 連接器所使用的 Active Directory Domain Services (AD DS) 帳戶具有密碼雜湊同步處理所需的正確使用者名稱、密碼及權限。

下圖說明在單一網域、內部部署 Active Directory 拓撲上，執行此 Cmdlet 的結果：

![密碼雜湊同步處理的診斷輸出](./media/tshoot-connect-password-hash-synchronization/phsglobalgeneral.png)

本節的其餘部分說明工作所傳回的特定結果和對應的問題。

#### <a name="password-hash-synchronization-feature-isnt-enabled"></a>密碼雜湊同步處理功能未啟用
如果您尚未使用 Azure AD Connect 精靈來啟用密碼雜湊同步處理，就會傳回下列錯誤：

![密碼雜湊同步處理未啟用](./media/tshoot-connect-password-hash-synchronization/phsglobaldisabled.png)

#### <a name="azure-ad-connect-server-is-in-staging-mode"></a>Azure AD Connect 伺服器處於預備模式
如果 Azure AD Connect 伺服器處於預備模式，就會暫時停用密碼雜湊同步處理，而會傳回下列錯誤：

![Azure AD Connect 伺服器處於預備模式](./media/tshoot-connect-password-hash-synchronization/phsglobalstaging.png)

#### <a name="no-password-hash-synchronization-heartbeat-events"></a>沒有密碼雜湊同步處理活動訊號事件
每個內部部署 Active Directory 連接器都有自己的密碼雜湊同步處理通道。 當密碼雜湊同步處理通道已建立，而且沒有任何密碼變更需要同步時，在「Windows 應用程式事件記錄檔」下每隔 30 分鐘會產生一次活動訊號事件 (EventId 654)。 對於每個內部部署 Active Directory 連接器，此 Cmdlet 會搜尋過去三小時內對應的活動訊號事件。 如果找不到活動訊號事件，則會傳回下列錯誤：

![沒有密碼雜湊同步處理活動訊號事件](./media/tshoot-connect-password-hash-synchronization/phsglobalnoheartbeat.png)

#### <a name="ad-ds-account-does-not-have-correct-permissions"></a>AD DS 帳戶沒有正確的權限
如果內部部署 Active Directory 連接器用來同步密碼雜湊的 AD DS 帳戶沒有適當的權限，則會傳回下列錯誤：

![不正確的認證](./media/tshoot-connect-password-hash-synchronization/phsglobalaccountincorrectpermission.png)

#### <a name="incorrect-ad-ds-account-username-or-password"></a>不正確的 AD DS 帳戶使用者名稱或密碼
如果內部部署 Active Directory 連接器用來同步密碼雜湊的 AD DS 帳戶有不正確的使用者名稱或密碼，則會傳回下列錯誤：

![不正確的認證](./media/tshoot-connect-password-hash-synchronization/phsglobalaccountincorrectcredential.png)



## <a name="one-object-is-not-synchronizing-passwords-troubleshoot-by-using-the-troubleshooting-task"></a>一個物件未同步密碼：使用疑難排解工作進行疑難排解

您可以使用疑難排解工作來判斷一個物件未同步密碼的原因。

> [!NOTE]
> 疑難排解工作僅適用於 Azure AD Connect 1.1.614.0 版或更新版本。

### <a name="run-the-diagnostics-cmdlet"></a>執行診斷 Cmdlet
若要針對特定使用者物件的問題進行疑難排解：

1. 在您的 Azure AD Connect 伺服器上，使用 [以系統管理員身分執行] 選項開啟新的 Windows PowerShell 工作階段。

2. 執行 `Set-ExecutionPolicy RemoteSigned` 或 `Set-ExecutionPolicy Unrestricted`。

3. 啟動 Azure AD Connect 精靈。

4. 瀏覽至 [其他工作] 頁面，選取 [疑難排解]，然後按 [下一步]。

5. 在 [疑難排解] 頁面上，按一下 [啟動]，以在 PowerShell 中啟動疑難排解功能表。

6. 在主功能表中，選取 [針對密碼雜湊同步處理進行疑難排解]。

7. 在子功能表中，選取 [未同步特定使用者帳戶的密碼]。

### <a name="understand-the-results-of-the-troubleshooting-task"></a>了解疑難排解工作的結果
疑難排解工作會執行下列檢查：

* 在 Active Directory 連接器空間、Metaverse 和 Azure AD 連接器空間中，檢查 Active Directory 物件的狀態。

* 確認有已啟用密碼雜湊同步處理的同步處理規則，且已套用至 Active Directory 物件。

* 試著擷取並顯示上次嘗試同步物件密碼的結果。

下圖說明針對單一物件的密碼雜湊同步處理進行疑難排解時，執行此 Cmdlet 的結果：

![密碼雜湊同步處理的診斷輸出 - 單一物件](./media/tshoot-connect-password-hash-synchronization/phssingleobjectgeneral.png)

本節的其餘部分說明 Cmdlet 所傳回的特定結果和對應的問題。

#### <a name="the-active-directory-object-isnt-exported-to-azure-ad"></a>Active Directory 物件未匯出至 Azure AD
因為 Azure AD 租用戶中沒有對應的物件，所以這個內部部署 Active Directory 帳戶的密碼雜湊同步處理失敗。 傳回下列錯誤：

![遺漏 Azure AD 物件](./media/tshoot-connect-password-hash-synchronization/phssingleobjectnotexported.png)

#### <a name="user-has-a-temporary-password"></a>使用者有暫時密碼
目前，Azure AD Connect 不支援與 Azure AD 同步暫時密碼。 如果對內部部署 Active Directory 使用者設定 [在下次登入時變更密碼] 選項，密碼就視為暫時。 傳回下列錯誤：

![未匯出暫時密碼](./media/tshoot-connect-password-hash-synchronization/phssingleobjecttemporarypassword.png)

#### <a name="results-of-last-attempt-to-synchronize-password-arent-available"></a>上次嘗試同步密碼沒有結果
根據預設，Azure AD Connect 會將密碼雜湊同步處理嘗試的結果儲存達七天。 如果選取的 Active Directory 物件沒有可用的結果，則會傳回下列警告：

![單一物件的診斷輸出 - 沒有密碼同步記錄](./media/tshoot-connect-password-hash-synchronization/phssingleobjectnohistory.png)



## <a name="no-passwords-are-synchronized-troubleshoot-by-using-the-diagnostic-cmdlet"></a>未同步任何密碼：使用診斷 Cmdlet 進行疑難排解
您可以使用 `Invoke-ADSyncDiagnostics` Cmdlet 來查明未同步任何密碼的原因。

> [!NOTE]
> `Invoke-ADSyncDiagnostics` Cmdlet 僅適用於 Azure AD Connect 1.1.524.0 版或更新版本。

### <a name="run-the-diagnostics-cmdlet"></a>執行診斷 Cmdlet
針對未同步任何密碼的問題進行疑難排解：

1. 在您的 Azure AD Connect 伺服器上，使用 [以系統管理員身分執行] 選項開啟新的 Windows PowerShell 工作階段。

2. 執行 `Set-ExecutionPolicy RemoteSigned` 或 `Set-ExecutionPolicy Unrestricted`。

3. 執行 `Import-Module ADSyncDiagnostics`。

4. 執行 `Invoke-ADSyncDiagnostics -PasswordSync`。



## <a name="one-object-is-not-synchronizing-passwords-troubleshoot-by-using-the-diagnostic-cmdlet"></a>一個物件未同步密碼：使用診斷 Cmdlet 進行疑難排解
您可以使用 `Invoke-ADSyncDiagnostics` Cmdlet 來判斷一個物件未同步密碼的原因。

> [!NOTE]
> `Invoke-ADSyncDiagnostics` Cmdlet 僅適用於 Azure AD Connect 1.1.524.0 版或更新版本。

### <a name="run-the-diagnostics-cmdlet"></a>執行診斷 Cmdlet
針對未同步任何使用者密碼的問題進行疑難排解：

1. 在您的 Azure AD Connect 伺服器上，使用 [以系統管理員身分執行] 選項開啟新的 Windows PowerShell 工作階段。

2. 執行 `Set-ExecutionPolicy RemoteSigned` 或 `Set-ExecutionPolicy Unrestricted`。

3. 執行 `Import-Module ADSyncDiagnostics`。

4. 執行下列 Cmdlet：
   ```
   Invoke-ADSyncDiagnostics -PasswordSync -ADConnectorName <Name-of-AD-Connector> -DistinguishedName <DistinguishedName-of-AD-object>
   ```
   例如︰
   ```
   Invoke-ADSyncDiagnostics -PasswordSync -ADConnectorName "contoso.com" -DistinguishedName "CN=TestUserCN=Users,DC=contoso,DC=com"
   ```



## <a name="no-passwords-are-synchronized-manual-troubleshooting-steps"></a>未同步任何密碼：手動疑難排解步驟
請依照下列步驟來判斷未同步任何密碼的原因：

1. Connect 伺服器是否是處於[預備模式](how-to-connect-sync-operations.md#staging-mode)？ 處於預備模式的伺服器不會同步處理任何密碼。

2. 執行[取得密碼同步設定的狀態](#get-the-status-of-password-sync-settings)一節中的指令碼。 它可讓您大致了解密碼同步設定作業。  

    ![來自密碼同步設定的 PowerShell 指令碼輸出](./media/tshoot-connect-password-hash-synchronization/psverifyconfig.png)  

3. 如果 Azure AD 中未啟用此功能，或未啟用同步通道狀態，請執行 Connect 安裝精靈。 選取 [自訂同步處理選項] 並取消選取密碼同步。這項變更會暫時停用此功能。 然後再次執行精靈並重新啟用密碼同步處理。再次執行指令碼，確認組態正確無誤。

4. 查看事件記錄中是否有錯誤。 尋找下列事件，這些事件會指出問題所在︰
    * 來源：「目錄同步作業」識別碼：0、611、652、655。如果您看到這些事件，即表示有連線問題。 事件記錄訊息包含發生問題的樹系資訊。 如需詳細資訊，請參閱[連線問題](#connectivity problem)。

5. 如果您沒有看到任何活動訊號，或任何其他操作都沒有作用，請執行[觸發所有密碼的完整同步](#trigger-a-full-sync-of-all-passwords)。 只執行一次指令碼。

6. 請參閱[針對一個未同步密碼的物件進行疑難排解](#one-object-is-not-synchronizing-passwords)一節。

### <a name="connectivity-problems"></a>連線問題

您是否能夠與 Azure AD 連線？

帳戶是否具備可讀取所有網域中密碼雜湊的必要權限？ 如果您使用 [快速設定] 來安裝 Connect，權限應該已經正確。 

如果您使用自訂安裝，請執行下列動作來手動設定權限：
    
1. 若要尋找 Active Directory 連接器所使用的帳戶，請啟動 **Synchronization Service Manager**。 
 
2. 移至 [連接器]，然後尋找您要進行疑難排解的內部部署 Active Directory 樹系。 
 
3. 選取 [連接器]，然後按一下 [屬性]。 
 
4. 前往 [連線至 Active Directory 樹系] 。  
    
    ![Active Directory 連接器所使用的帳戶](./media/tshoot-connect-password-hash-synchronization/connectoraccount.png)  
    記下使用者名稱和帳戶所在的網域。
    
5. 啟動 **Active Directory 使用者和電腦**，然後確認您稍早發現的帳戶，已在樹系中所有網域的根目錄上設定下列權限：
    * 複寫目錄變更
    * 複寫目錄變更 (全部)

6. Azure AD Connect 是否可以連線到網域控制站？ 如果 Connect 伺服器無法連線到所有網域控制站，請設定 [只使用慣用的網域控制站]。  
    
    ![Active Directory 連接器所使用的網域控制站](./media/tshoot-connect-password-hash-synchronization/preferreddc.png)  
    
7. 返回 **Synchronization Service Manager**，並**設定目錄磁碟分割**。 
 
8. 在 [選取目錄分割] 中選取您的網域，選取 [只使用慣用的網域控制站] 核取方塊，然後按一下 [設定]。 

9. 在清單中，輸入 Connect 應該用來進行密碼同步的網域控制站。此相同清單也用於匯入和匯出。 針對您的所有網域執行這些步驟。

10. 如果此指令碼顯示沒有活動訊號，請執行[觸發所有密碼的完整同步](#trigger-a-full-sync-of-all-passwords)中的指令碼。

## <a name="one-object-is-not-synchronizing-passwords-manual-troubleshooting-steps"></a>一個物件未同步密碼：手動疑難排解步驟
您可以藉由檢閱物件的狀態，輕鬆地針對密碼雜湊同步處理問題進行疑難排解。

1. 在 **Active Directory 使用者和電腦**中，搜尋使用者，然後確認已清除 [使用者必須在下次登入時變更密碼] 核取方塊。  

    ![Active Directory 生產力密碼](./media/tshoot-connect-password-hash-synchronization/adprodpassword.png)  

    如果選取此核取方塊，請要求使用者登入並變更密碼。 暫時密碼不會與 Azure AD 同步。

2. 如果 Active Directory 中的密碼看起來正確，請在同步處理引擎中追蹤使用者。 從內部部署 Active Directory 到 Azure AD 追蹤使用者，就能知道物件是否有描述性錯誤。

    a. 啟動 [Synchronization Service Manager](how-to-connect-sync-service-manager-ui.md)。

    b. 按一下 [連接器] 。

    c. 選取使用者所在的 **Active Directory 連接器**。

    d. 選取 [搜尋連接器空間] 。

    e. 在 [範圍] 方塊中，選取 [DN 或錨點]，然後輸入您要進行疑難排解之使用者的完整 DN。

    ![在連接器空間中依 DN 搜尋使用者](./media/tshoot-connect-password-hash-synchronization/searchcs.png)  

    f. 找出您要搜尋的使用者，然後按一下 [屬性] 來查看所有屬性。 如果該使用者不在搜尋結果中，請確認您的[篩選規則](how-to-connect-sync-configure-filtering.md)，且務必執行[套用並驗證變更](how-to-connect-sync-configure-filtering.md#apply-and-verify-changes)，如此 Connect 中才會顯示該使用者。

    g. 若要查看物件在過去一週的密碼同步詳細資料，請按一下 [記錄]。  

    ![物件記錄詳細資料](./media/tshoot-connect-password-hash-synchronization/csobjectlog.png)  

    如果物件記錄是空的，則表示 Azure AD Connect 無法從 Active Directory 讀取密碼雜湊。 請繼續進行[連線錯誤](#connectivity-errors)疑難排解。 如果您看到 [成功] 以外的任何其他值，請參考[密碼同步記錄](#password-sync-log)中的表格。

    h. 選取 [歷程] 索引標籤，並確認至少有一個同步規則的 [PasswordSync] 資料行是 **True**。 在預設組態中，同步規則的名稱是 **In from AD - User AccountEnabled**。  

    ![使用者的相關歷程資訊](./media/tshoot-connect-password-hash-synchronization/cspasswordsync.png)  

    i. 按一下 [Metaverse 物件屬性] 來顯示使用者屬性清單。  

    ![Metaverse 資訊](./media/tshoot-connect-password-hash-synchronization/mvpasswordsync.png)  

    確認沒有任何 **cloudFiltered** 屬性存在。 確定網域屬性 (domainFQDN 和 domainNetBios) 具有預期的值。

    j. 按一下 [連接器] 索引標籤。請確定您看到已連線至內部部署 Active Directory 和 Azure AD 的連接器。

    ![Metaverse 資訊](./media/tshoot-connect-password-hash-synchronization/mvconnectors.png)  

    k. 選取代表 Azure AD 的資料列，按一下 [屬性]，然後按一下 [歷程] 索引標籤。連接器空間物件應該有一個輸出規則的 [PasswordSync] 資料行設為 **True**。 在預設組態中，同步規則的名稱是 **Out to AAD - User Join**。  

    ![連接器空間物件屬性對話方塊](./media/tshoot-connect-password-hash-synchronization/cspasswordsync2.png)  

### <a name="password-sync-log"></a>密碼同步記錄
[狀態] 欄可以有下列值︰

| 狀態 | 說明 |
| --- | --- |
| 成功 |已成功同步處理密碼。 |
| FilteredByTarget |密碼會設為 [使用者必須在下次登入時變更密碼] 。 未同步處理密碼。 |
| NoTargetConnection |Metaverse 或 Azure AD 連接器空間中沒有任何物件。 |
| SourceConnectorNotPresent |在內部部署 Active Directory 連接器空間中找不到任何物件。 |
| TargetNotExportedToDirectory |尚未匯出 Azure AD 連接器空間中的物件。 |
| MigratedCheckDetailsForMoreInfo |記錄項目建立於組建 1.0.9125.0 之前，並且以其舊版的狀態顯示。 |
| Error |服務傳回未知的錯誤。 |
| 不明 |嘗試處理密碼雜湊的批次時發生錯誤。  |
| MissingAttribute |無法使用 Azure AD Domain Services 所需的特定屬性 (例如，Kerberos 雜湊)。 |
| RetryRequestedByTarget |先前無法使用 Azure AD Domain Services 所需的特定屬性 (例如，Kerberos 雜湊)。 會嘗試重新同步處理使用者的密碼雜湊。 |

## <a name="scripts-to-help-troubleshooting"></a>協助疑難排解的指令碼

### <a name="get-the-status-of-password-sync-settings"></a>取得密碼同步設定的狀態
```
Import-Module ADSync
$connectors = Get-ADSyncConnector
$aadConnectors = $connectors | Where-Object {$_.SubType -eq "Windows Azure Active Directory (Microsoft)"}
$adConnectors = $connectors | Where-Object {$_.ConnectorTypeName -eq "AD"}
if ($aadConnectors -ne $null -and $adConnectors -ne $null)
{
    if ($aadConnectors.Count -eq 1)
    {
        $features = Get-ADSyncAADCompanyFeature -ConnectorName $aadConnectors[0].Name
        Write-Host
        Write-Host "Password sync feature enabled in your Azure AD directory: "  $features.PasswordHashSync
        foreach ($adConnector in $adConnectors)
        {
            Write-Host
            Write-Host "Password sync channel status BEGIN ------------------------------------------------------- "
            Write-Host
            Get-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector.Name
            Write-Host
            $pingEvents =
                Get-EventLog -LogName "Application" -Source "Directory Synchronization" -InstanceId 654  -After (Get-Date).AddHours(-3) |
                    Where-Object { $_.Message.ToUpperInvariant().Contains($adConnector.Identifier.ToString("D").ToUpperInvariant()) } |
                    Sort-Object { $_.Time } -Descending
            if ($pingEvents -ne $null)
            {
                Write-Host "Latest heart beat event (within last 3 hours). Time " $pingEvents[0].TimeWritten
            }
            else
            {
                Write-Warning "No ping event found within last 3 hours."
            }
            Write-Host
            Write-Host "Password sync channel status END ------------------------------------------------------- "
            Write-Host
        }
    }
    else
    {
        Write-Warning "More than one Azure AD Connectors found. Please update the script to use the appropriate Connector."
    }
}
Write-Host
if ($aadConnectors -eq $null)
{
    Write-Warning "No Azure AD Connector was found."
}
if ($adConnectors -eq $null)
{
    Write-Warning "No AD DS Connector was found."
}
Write-Host
```

#### <a name="trigger-a-full-sync-of-all-passwords"></a>觸發所有密碼的完整同步處理
> [!NOTE]
> 只執行一次這個指令碼。 如果需要執行多次，則表示問題出在其他地方。 若要針對問題進行疑難排解，請連絡 Microsoft 支援服務。

您可以使用下列指令碼來觸發所有密碼的完整同步︰

```
$adConnector = "<CASE SENSITIVE AD CONNECTOR NAME>"
$aadConnector = "<CASE SENSITIVE AAD CONNECTOR NAME>"
Import-Module adsync
$c = Get-ADSyncConnector -Name $adConnector
$p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter "Microsoft.Synchronize.ForceFullPasswordSync", String, ConnectorGlobal, $null, $null, $null
$p.Value = 1
$c.GlobalParameters.Remove($p.Name)
$c.GlobalParameters.Add($p)
$c = Add-ADSyncConnector -Connector $c
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $aadConnector -Enable $false
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $aadConnector -Enable $true
```

## <a name="next-steps"></a>後續步驟
* [使用 Azure AD Connect 同步來實作密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md) \(英文\)
* [Azure AD Connect 同步：自訂同步處理選項](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)
