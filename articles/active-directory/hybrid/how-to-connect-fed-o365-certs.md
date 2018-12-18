---
title: Office 365 和 Azure AD 使用者的憑證更新 | Microsoft Docs
description: 本文說明 Office 365 使用者如何解決收到電子郵件通知續約憑證的問題。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: curtand
ms.assetid: 543b7dc1-ccc9-407f-85a1-a9944c0ba1be
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/20/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 0205552c073bf03c40da4d108256fa09e71fbbe3
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46309887"
---
# <a name="renew-federation-certificates-for-office-365-and-azure-active-directory"></a>更新 Office 365 和 Azure Active Directory 的同盟憑證
## <a name="overview"></a>概觀
為了讓 Azure Active Directory (Azure AD) 與 Active Directory Federation Services (AD FS) 之間能夠成功地同盟，AD FS 用來向 Azure AD 簽署安全性權杖的憑證應該符合 Azure AD 中所設定的憑證。 任何不相符都可能導致信任受損。 Azure AD 會確保這項資訊在您部署 AD FS 和 Web 應用程式 Proxy (適用於外部網路存取) 時保持同步。

本文提供您其他資訊，以便在下列情況時管理權杖簽署憑證，並讓憑證與 Azure AD 保持同步︰

* 您不會部署 Web 應用程式 Proxy，因此無法在外部網路取得同盟中繼資料。
* 您不會對權杖簽署憑證使用預設的 AD FS 設定。
* 您要使用協力廠商的識別提供者。

## <a name="default-configuration-of-ad-fs-for-token-signing-certificates"></a>權杖簽署憑證的預設 AD FS 設定
權杖簽署和權杖解密憑證通常是自我簽署的憑證，有效期為一年。 根據預設，AD FS 包含名為 **AutoCertificateRollover**的自動更新程序。 如果您使用 AD FS 2.0 或更新版本，Office 365 和 Azure AD 在您的憑證到期之前會自動進行更新。

### <a name="renewal-notification-from-the-office-365-portal-or-an-email"></a>從 Office 365 入口網站或電子郵件更新通知
> [!NOTE]
> 如果您收到電子郵件或入口網站通知，要求您更新 Office 憑證，請參閱 [管理權杖簽署憑證的變更](#managecerts) ，檢查您是否需要採取任何動作。 Microsoft 已知可能會有在不需要採取任何動作的情況下仍送出憑證更新通知的問題。
>
>

Azure AD 會嘗試監視同盟中繼資料，並依照此中繼資料的指示更新權杖簽署憑證。 在權杖簽署憑證到期前 30 天，Azure AD 會藉由輪詢同盟中繼資料，檢查是否已有新的憑證可供使用。

* 如果它能成功輪詢同盟中繼資料並擷取新的憑證，使用者就不會收到電子郵件通知或 Office 365 入口網站發出的警告。
* 如果它無法擷取新的權杖簽署憑證，不論原因是無法取得同盟中繼資料，還是自動憑證變換並未啟用，Azure AD 都會發出電子郵件通知和 O365 入口網站中的警告。

![Office 365 入口網站通知](./media/how-to-connect-fed-o365-certs/notification.png)

> [!IMPORTANT]
> 如果您使用 AD FS，為確保商務持續性，請確認您的伺服器具備下列更新，以免發生已知問題驗證失敗。 這可減少在此更新和未來更新期間的已知 AD FS Proxy 伺服器問題︰
>
> Server 2012 R2 - [Windows Server 2014 年 5 月彙總套件](http://support.microsoft.com/kb/2955164)
>
> Server 2008 R2 和 2012 - [在 Windows Server 2012 或 Windows 2008 R2 SP1 中透過 Proxy 驗證失敗](http://support.microsoft.com/kb/3094446)
>
>

## 檢查是否需要更新憑證 <a name="managecerts"></a>
### <a name="step-1-check-the-autocertificaterollover-state"></a>步驟 1︰檢查 AutoCertificateRollover 狀態
在 AD FS 伺服器上開啟 Powershell。 檢查 AutoCertificateRollover 值是否已設定為 True。

    Get-Adfsproperties

![AutoCertificateRollover](./media/how-to-connect-fed-o365-certs/autocertrollover.png)

>[!NOTE] 
>如果您使用 AD FS 2.0，請先執行 Add-Pssnapin Microsoft.Adfs.Powershell。

### <a name="step-2-confirm-that-ad-fs-and-azure-ad-are-in-sync"></a>步驟 2︰確認 AD FS 和 Azure AD 已同步
在 AD FS 伺服器上開啟 MSOnline PowerShell 提示字元，並連線到 Azure AD。

> [!NOTE]
> MSOL-Cmdlet 屬於 MSOnline PowerShell 模組。
> 您可以直接從 PowerShell 資源庫下載 MSOnline PowerShell 模組。
> 
>

    Install-Module MSOnline

使用 MSOnline PowerShell 模組連線至 Azure AD。

    Import-Module MSOnline
    Connect-MsolService

檢查 AD FS 和 Azure AD 信任屬性中針對指定網域所設定的憑證。

    Get-MsolFederationProperty -DomainName <domain.name> | FL Source, TokenSigningCertificate

![Get-MsolFederationProperty](./media/how-to-connect-fed-o365-certs/certsync.png)

如果這兩個輸出中的指紋相符，您的憑證便已與 Azure AD 同步。

### <a name="step-3-check-if-your-certificate-is-about-to-expire"></a>步驟 3︰檢查憑證是否即將到期
在 Get-MsolFederationProperty 或 Get-AdfsCertificate 的輸出中，檢查「不晚於」之下的日期。 如果日期相隔不到 30 天，您應採取動作。

| AutoCertificateRollover | 憑證與 Azure AD 同步 | 可公開取得同盟中繼資料 | 有效期 | 動作 |
|:---:|:---:|:---:|:---:|:---:|
| 是 |yes |是 |- |不需採取動作。 請參閱 [自動更新權杖簽署憑證](#autorenew)。 |
| 是 |否 |- |小於 15 天 |立即更新。 請參閱 [手動更新權杖簽署憑證](#manualrenew)。 |
| 否 |- |- |少於 30 天 |立即更新。 請參閱 [手動更新權杖簽署憑證](#manualrenew)。 |

\[-]  無關緊要

## 自動更新權杖簽署憑證 (建議選項) <a name="autorenew"></a>
如果下列兩種情況成立，您不需要執行任何手動步驟︰

* 您已部署能夠從外部網路存取同盟中繼資料的 Web 應用程式 Proxy。
* 您目前使用 AD FS 預設設定 (已啟用 AutoCertificateRollove)。

檢查下列各項，確認可以自動更新憑證。

**1.AD FS 屬性 AutoCertificateRollover 必須設定為 True。** 這表示 AD FS 會在舊憑證到期之前，自動產生新的權杖簽署和權杖解密憑證。

**2.可公開取得 AD FS 同盟中繼資料。** 從公用網際網路 (離開公司網路) 的電腦瀏覽到下列 URL 檢查同盟中繼資料是否可公開存取：

https://(your_FS_name)/federationmetadata/2007-06/federationmetadata.xml

將其中的 `(your_FS_name) `取代為您的組織使用的同盟服務主機名稱，例如 fs.contoso.com。  如果您能夠成功確認上述兩個設定，您就不必執行任何動作。  

範例： https://fs.contoso.com/federationmetadata/2007-06/federationmetadata.xml
## 手動更新權杖簽署憑證 <a name="manualrenew"></a>
您可以選擇手動更新權杖簽署憑證。 例如，下列案例可能比較適合進行手動更新︰

* 權杖簽署憑證不是自我簽署憑證。 最常見的原因是您的組織管理從組織的憑證授權單位註冊的 AD FS 憑證。
* 網路安全性不允許公開取得同盟中繼資料。

在這些案例中，每當您更新權杖簽署憑證時，您還必須使用 PowerShell 命令 Update-MsolFederatedDomain 更新 Office 365 網域。

### <a name="step-1-ensure-that-ad-fs-has-new-token-signing-certificates"></a>步驟 1︰確認 AD FS 具有新的權杖簽署憑證
**非預設設定**

如果您處於非預設的 AD FS 設定 (也就是 **AutoCertificateRollover** 設定為 **False**)，則您想必也是使用自訂憑證 (非自我簽署)。 如需如何更新 AD FS 權杖簽署憑證的詳細資訊，請參閱 [給未使用 AD FS 自我簽署憑證之客戶的指導方針](https://msdn.microsoft.com/library/azure/JJ933264.aspx#BKMK_NotADFSCert)。

**無法公開取得同盟中繼資料**

另一方面，如果 **AutoCertificateRollover** 設定為 **True**，但無法公開取得同盟中繼資料，請先確定 AD FS 已產生新的權杖簽署憑證。 遵循下列步驟，確認您有新的權杖簽署憑證：

1. 確認您已登入主要 AD FS 伺服器。
2. 開啟 PowerShell 命令視窗並執行下列命令，檢查 AD FS 中目前的簽署憑證：

    PS C:\>Get-ADFSCertificate -CertificateType token-signing

   > [!NOTE]
   > 如果您使用 AD FS 2.0，則必須先執行 Add-Pssnapin Microsoft.Adfs.Powershell。
   >
   >
3. 查看命令輸出中所列的任何憑證。 如果 AD FS 已產生新的憑證，您應該會在輸出中看到兩個憑證：一個 **IsPrimary** 值是 **True**，而 **NotAfter** 日期是 5 天內，另一個 **IsPrimary** 是 **False**，而 **NotAfter** 大約在未來一年。
4. 如果您只看到一個憑證，而 **NotAfter** 日期為 5 天內，您必須執行產生新的憑證。
5. 若要產生新憑證，請在 PowerShell 命令提示字元中執行下列命令： `PS C:\>Update-ADFSCertificate –CertificateType token-signing`。
6. 再次執行下列命令驗證更新：PS C:\>Get-ADFSCertificate -CertificateType token-signing

現在應該會列出兩個憑證，一個的 **NotAfter** 日期大約在未來一年，且 **IsPrimary** 值是 **False**。

### <a name="step-2-update-the-new-token-signing-certificates-for-the-office-365-trust"></a>步驟 2︰更新 Office 365 信任的新權杖簽署憑證
使用要用於信任的新權杖簽署憑證更新 Office 365，如下所示。

1. 開啟適用於 Windows PowerShell 的 Microsoft Azure Active Directory 模組。
2. 執行 $cred=Get-Credential。 當此 Cmdlet 提示您輸入認證時，請輸入您的雲端服務系統管理員帳戶認證。
3. 執行 Connect-MsolService –Credential $cred。此 Cmdlet 可讓您連線到雲端服務。 在您執行由工具安裝的任何其他 Cmdlet 之前，必須先建立讓您連線到雲端服務的環境。
4. 如果您不是在 AD FS 主要同盟伺服器的電腦上執行這些命令，請執行 Set-MSOLAdfscontext -Computer &lt;AD FS primary server&gt;，其中 &lt;AD FS primary server&gt; 是主要 AD FS 伺服器的內部 FQDN 名稱。 此 Cmdlet 會建立讓您連線到 AD FS 的環境。
5. 執行 Update-MSOLFederatedDomain –DomainName &lt;domain&gt;。 此 Cmdlet 會將 AD FS 的設定更新成雲端服務，並設定兩者之間的信任關係。

> [!NOTE]
> 如果您需要支援多個頂層網域，例如 contoso.com 和 fabrikam.com，則您使用任何 Cmdlet 時必須搭配使用 **SupportMultipleDomain** 參數。 如需詳細資訊，請參閱 [支援多個頂層網域](how-to-connect-install-multiple-domains.md)。
>


## 使用 Azure AD Connect 修復 Azure AD 信任 <a name="connectrenew"></a>
如果您已使用 Azure AD Connect 設定 AD FS 伺服器陣列和 Azure AD 信任，則可以使用 Azure AD Connect 來偵測是否需要對權杖簽署憑證採取任何動作。 如果您需要更新憑證，可以使用 Azure AD Connect 這樣做。

如需詳細資訊，請參閱 [修復信任](how-to-connect-fed-management.md)。

## <a name="ad-fs-and-azure-ad-certificate-update-steps"></a>AD FS 與 Azure AD 憑證更新步驟
權杖簽署憑證是標準 X509 憑證，用來安全地簽署同盟伺服器簽發的所有權杖。 權杖解密憑證是標準 X509 憑證，用來解密任何傳入權杖。 

依預設，會將 AD FS 設定為在初始設定時和當憑證接近其到期日時自動產生權杖簽署和權杖解密憑證。

Azure AD 會嘗試在目前憑證到期前 30 天內，從同盟服務中繼資料中取得新的憑證。 如果當時沒有新的憑證可用，則 Azure AD 將繼續定期每天監視中繼資料。 中繼資料一有新的憑證可用，系統就會使用新的憑證資訊更新網域的同盟設定。 您可以使用 `Get-MsolDomainFederationSettings`，驗證您是否在 NextSigningCertificate / SigningCertificate 中看到新的憑證。

如需 AD FS 中權杖簽署憑證的詳細資訊，請參閱[取得和設定 AD FS 的權杖簽署和權杖解密憑證](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configure-ts-td-certs-ad-fs)
