---
title: Azure Active Directory Connect：針對無縫單一登入進行疑難排解 | Microsoft Docs
description: 本主題說明如何針對 Azure Active Directory 無縫單一登入進行疑難排解
services: active-directory
author: billmath
ms.reviewer: swkrish
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.topic: article
ms.date: 09/24/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: a020f0f22f16d8aaa959c41a912ca5839be05312
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47055895"
---
# <a name="troubleshoot-azure-active-directory-seamless-single-sign-on"></a>針對 Azure Active Directory 無縫單一登入進行疑難排解

這篇文章可協助您尋找有關 Azure Active Directory (Azure AD) 無縫單一登入 (無縫 SSO) 常見問題的疑難排解資訊。

## <a name="known-issues"></a>已知問題

- 在某些情況下，啟用無縫 SSO 最久可能需要 30 分鐘。
- 如果您在租用戶上將「無縫 SSO」停用再重新啟用，使用者將必須等到他們的已快取 Kerberos 票證到期 (有效期通常為 10 小時) 之後，才能使用單一登入體驗。
- 不支援 Edge 瀏覽器。
- 如果無縫 SSO 成功，使用者就沒有機會選取 [讓我保持登入]。 由於這個行為而使得 [SharePoint 和 OneDrive 對應案例](https://support.microsoft.com/help/2616712/how-to-configure-and-to-troubleshoot-mapped-network-drives-that-connec) \(機器翻譯\) 無法運作。
- 使用非互動式流程，來支援 Office 365 Win32 用戶端 (Outlook、Word、Excel 和其他產品) 16.0.8730.xxxx 版和更新版本。 不支援其他版本；在那些版本中，使用者將輸入其使用者名稱 (但不輸入密碼) 來登入。 針對 OneDrive，您必須啟用 [OneDrive 無訊息設定功能](https://techcommunity.microsoft.com/t5/Microsoft-OneDrive-Blog/Previews-for-Silent-Sync-Account-Configuration-and-Bandwidth/ba-p/120894) \(英文\) 以獲得無訊息登入體驗。
- 在 Firefox 的私用瀏覽模式中，無法使用無縫 SSO。
- Internet Explorer 在開啟「增強保護」模式時也無法使用無縫 SSO。
- iOS 和 Android 上的行動瀏覽器無法使用無縫 SSO。
- 如果使用者是 Active Directory 中過多群組的成員之一，該使用者的 Kerberos 票證可能會太大而無法處理，並且會導致無縫式 SSO 失敗。 Azure AD HTTPS 要求可以有大小上限為 50 KB 的標頭；Kerberos 票證必須小於該限制，才能容納其他 Azure AD 成品 (通常是 2 - 5 KB)，例如 Cookie。 建議您降低使用者的群組成員資格，然後再試一次。
- 如果您正在同步處理 30 個或更多的 Active Directory 樹系，便無法透過 Azure AD Connect 啟用無縫 SSO。 您可以在租用戶上[手動啟用](#manual-reset-of-the-feature)此功能，以解決這個問題。
- 將 Azure AD 服務 URL (https://autologon.microsoftazuread-sso.com)) 新增至「信任的網站區域」而非「近端內部網路區域」，會「阻止使用者登入」。
- 無縫 SSO 會使用適用於 Kerberos 的 **RC4_HMAC_MD5** 加密類型。 在 Active Directory 設定中停用 **RC4_HMAC_MD5** 加密類型會中斷無縫 SSO。 在您的「群組原則管理編輯器」工具中，請確定在 [電腦設定] -> [Windows 設定] -> [安全性設定] -> [本機原則] -> [安全性選項] -> [網路安全性: 設定 Kerberos 允許的加密類型] 底下，[RC4_HMAC_MD5] 的原則值為 [已啟用]。 此外，無縫 SSO 無法使用其他加密類型，因此請確定它們**已停用**。

## <a name="check-status-of-feature"></a>檢查功能的狀態

確認您租用戶端的無縫 SSO 功能仍為 [已啟用]。 您可以前往 [Azure Active Directory 管理中心](https://aad.portal.azure.com/)的 [Azure AD Connect] 窗格來檢查狀態。

![Azure Active Directory 管理中心：Azure AD Connect 窗格](./media/tshoot-connect-sso/sso10.png)

一路按一下來查看已針對「無縫 SSO」啟用的所有 AD 樹系。

![Azure Active Directory 管理中心：無縫 SSO 窗格](./media/tshoot-connect-sso/sso13.png)

## <a name="sign-in-failure-reasons-in-the-azure-active-directory-admin-center-needs-a-premium-license"></a>Azure Active Directory 管理中心上的登入失敗原因 (需要 Premium 授權)

如果您的租用戶已有與其相關聯的 Azure AD Premium 授權，您也可以查看 [Azure Active Directory 管理中心](https://aad.portal.azure.com/)上的[登入活動報告](../reports-monitoring/concept-sign-ins.md)。

![Azure Active Directory 管理中心：登入報告](./media/tshoot-connect-sso/sso9.png)

瀏覽至 [Azure Active Directory 管理中心](https://aad.portal.azure.com/)中的 [Azure Active Directory] > [登入]，然後選取特定使用者的登入活動。 尋找 [登入錯誤碼] 欄位。 使用下表，將該欄位的值對應至失敗原因和解決方式：

|登入錯誤碼|登入失敗原因|解決方案
| --- | --- | ---
| 81001 | 使用者的 Kerberos 票證太大。 | 降低使用者的群組成員資格，並再試一次。
| 81002 | 無法驗證使用者的 Kerberos 票證。 | 請參閱[為檢查清單疑難排解](#troubleshooting-checklist)。
| 81003 | 無法驗證使用者的 Kerberos 票證。 | 請參閱[為檢查清單疑難排解](#troubleshooting-checklist)。
| 81004 | Kerberos 驗證嘗試失敗。 | 請參閱[為檢查清單疑難排解](#troubleshooting-checklist)。
| 81008 | 無法驗證使用者的 Kerberos 票證。 | 請參閱[為檢查清單疑難排解](#troubleshooting-checklist)。
| 81009 | 無法驗證使用者的 Kerberos 票證。 | 請參閱[為檢查清單疑難排解](#troubleshooting-checklist)。
| 81010 | 無縫式 SSO 失敗，因為使用者的 Kerberos 票證已過期或無效。 | 使用者需要從您公司網路內部已加入網域的裝置登入。
| 81011 | 無法根據使用者 Kerberos 票證中的資訊找到使用者物件。 | 使用 Azure AD Connect 將使用者資訊同步至 Azure AD。
| 81012 | 嘗試登入 Azure AD 的使用者與登入裝置的使用者不同。 | 使用者需要從不同的裝置登入。
| 81013 | 無法根據使用者 Kerberos 票證中的資訊找到使用者物件。 |使用 Azure AD Connect 將使用者資訊同步至 Azure AD。 

## <a name="troubleshooting-checklist"></a>疑難排解檢查清單

使用下列檢查清單，為無縫 SSO 問題疑難排解︰

- 請務必在 Azure AD Connect 上已啟用無縫 SSO 功能。 如果您無法啟用此功能 (例如，因為連接埠已封鎖)，請確定您已完成所有[必要條件](how-to-connect-sso-quick-start.md#step-1-check-the-prerequisites)。
- 如果您已在租用戶上同時啟用 [Azure AD 聯結](../active-directory-azureadjoin-overview.md)和「無縫 SSO」，請確定問題不是出在「Azure AD 聯結」上。 如果裝置既已向 Azure AD 註冊也加入網域，則來自「Azure AD 聯結」的 SSO 優先順序會高於「無縫 SSO」。 使用來自「Azure AD 聯結」的 SSO 時，使用者會看到指出「已連線到 Windows」的登入圖格。
- 確定 Azure AD URL (https://autologon.microsoftazuread-sso.com)) 是使用者內部網路區域設定的一部分。
- 確定公司裝置已加入 Active Directory 網域。 裝置「不」需要[加入 Azure AD](../active-directory-azureadjoin-overview.md)，無縫 SSO 就可運作。
- 確定使用者已透過 Active Directory 網域帳戶登入裝置。
- 確定使用者帳戶是來自已設定無縫 SSO 的 Active Directory 樹系。
- 確定裝置已連線到公司網路。
- 確定裝置的時間已經與 Active Directory 和網域控制站的時間同步，且彼此的時間差不到五分鐘。
- 請確認在您想要啟用無縫 SSO 的每個 AD 樹系中，`AZUREADSSOACCT` 電腦帳戶存在且已啟用。 如果電腦帳戶已遭刪除或遺失，您可以使用 [PowerShell Cmdlet](#manual-reset-of-the-feature) 來重新建立它們。
- 從命令提示字元使用 `klist` 命令，列出裝置上現有的 Kerberos 票證。 確認是否有核發給 `AZUREADSSOACCT` 電腦帳戶的票證。 使用者的 Kerberos 票證有效期通常為 10 個小時。 您的 Active Directory 可能有不同的設定。
- 如果您在租用戶上將「無縫 SSO」停用再重新啟用，使用者將必須等到他們的已快取 Kerberos 票證到期之後，才能使用單一登入體驗。
- 使用 `klist purge` 命令從裝置中清除現有的 Kerberos 票證，然後再試一次。
- 若要判斷是否有 JavaScript 相關問題，請檢閱瀏覽器的主控台記錄 (在 [開發人員工具] 底下)。
- 檢閱[網域控制站記錄](#domain-controller-logs)。

### <a name="domain-controller-logs"></a>網域控制站記錄

如果網域控制站已啟用成功稽核，則每次使用者透過無縫 SSO 登入時，系統都會在事件記錄中記錄一個安全性項目。 您可以使用下列查詢來尋找這些安全性事件。 (請查看與電腦帳戶 **AzureADSSOAcc$** 有關連的事件 **4769**。)

```
    <QueryList>
      <Query Id="0" Path="Security">
    <Select Path="Security">*[EventData[Data[@Name='ServiceName'] and (Data='AZUREADSSOACC$')]]</Select>
      </Query>
    </QueryList>
```

## <a name="manual-reset-of-the-feature"></a>手動重設該功能

如果疑難排解無法解決問題，您可以在租用戶上手動重設此功能。 請在執行 Azure AD Connect 的內部部署伺服器上依照下列步驟操作。

### <a name="step-1-import-the-seamless-sso-powershell-module"></a>步驟 1：匯入無縫 SSO PowerShell 模組

1. 首先，下載並安裝 [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/overview)。
2. 瀏覽至 `%programfiles%\Microsoft Azure Active Directory Connect` 資料夾。
3. 使用此命令匯入無縫 SSO PowerShell 模組：`Import-Module .\AzureADSSO.psd1`。

### <a name="step-2-get-the-list-of-active-directory-forests-on-which-seamless-sso-has-been-enabled"></a>步驟 2：取得已啟用無縫 SSO 的 Active Directory 樹系清單

1. 以系統管理員身分執行 PowerShell。 在 PowerShell 中，呼叫 `New-AzureADSSOAuthenticationContext`。 出現提示時，輸入您租用戶的全域管理員認證。
2. 呼叫 `Get-AzureADSSOStatus`。 此命令會提供已啟用這項功能的 Active Directory 樹系清單 (查看 [網域] 清單)。

### <a name="step-3-disable-seamless-sso-for-each-active-directory-forest-where-youve-set-up-the-feature"></a>步驟 3：已設定此功能的每個 Active Directory 樹系，均停用無縫 SSO

1. 呼叫 `$creds = Get-Credential`。 出現提示時，輸入預定 Active Directory 樹系的網域管理員認證。

    >[!NOTE]
    >我們會使用網域系統管理員的使用者名稱 (以使用者主體名稱 (UPN) (johndoe@contoso.com) 格式或網域限定的 SAM 帳戶名稱 (contoso\johndoe 或 contoso.com\johndoe) 格式提供)，來尋找預定的 AD 樹系。 如果您使用網域限定的 SAM 帳戶名稱，我們就會利用使用者名稱的網域部分，[使用 DNS 來尋找網域系統管理員的網域控制站](https://social.technet.microsoft.com/wiki/contents/articles/24457.how-domain-controllers-are-located-in-windows.aspx) \(英文\)。 如果您改用 UPN，我們就會先[將其轉譯成網域限定的 SAM 帳戶名稱](https://docs.microsoft.com/windows/desktop/api/ntdsapi/nf-ntdsapi-dscracknamesa) \(英文\)，然後再尋找適當的網域控制站。

2. 呼叫 `Disable-AzureADSSOForest -OnPremCredentials $creds`。 此命令會從此特定 Active Directory 樹系的內部部署網域控制站中移除 `AZUREADSSOACCT` 電腦帳戶。
3. 您已設定此功能的每個 Active Directory 樹系，均重複上述步驟。

### <a name="step-4-enable-seamless-sso-for-each-active-directory-forest"></a>步驟 4：為每個 Active Directory 樹系啟用無縫 SSO

1. 呼叫 `Enable-AzureADSSOForest`。 出現提示時，輸入預定 Active Directory 樹系的網域管理員認證。

   >[!NOTE]
   >我們會使用網域系統管理員的使用者名稱 (以使用者主體名稱 (UPN) (johndoe@contoso.com) 格式或網域限定的 SAM 帳戶名稱 (contoso\johndoe 或 contoso.com\johndoe) 格式提供)，來尋找預定的 AD 樹系。 如果您使用網域限定的 SAM 帳戶名稱，我們就會利用使用者名稱的網域部分，[使用 DNS 來尋找網域系統管理員的網域控制站](https://social.technet.microsoft.com/wiki/contents/articles/24457.how-domain-controllers-are-located-in-windows.aspx) \(英文\)。 如果您改用 UPN，我們就會先[將其轉譯成網域限定的 SAM 帳戶名稱](https://docs.microsoft.com/windows/desktop/api/ntdsapi/nf-ntdsapi-dscracknamesa) \(英文\)，然後再尋找適當的網域控制站。

2. 已設定此功能的每個 Active Directory 樹系，均重複上述步驟。

### <a name="step-5-enable-the-feature-on-your-tenant"></a>步驟 5。 啟用租用戶上的功能

若要在租用戶開啟此功能，請呼叫 `Enable-AzureADSSO -Enable $true`。
