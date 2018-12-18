---
title: 針對 Azure MFA NPS 擴充功能的錯誤碼進行移難排解 | Microsoft Docs
description: 協助您解決 Azure Multi-Factor Authentication NPS 擴充功能的問題
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 4097fab5610bf4bee6c14c65d3b45e0de818a0cc
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39160904"
---
# <a name="resolve-error-messages-from-the-nps-extension-for-azure-multi-factor-authentication"></a>解決 Azure Multi-Factor Authentication NPS 擴充功能的錯誤訊息

如果您使用 Azure Multi-Factor Authentication NPS 擴充功能時發生錯誤，請使用本文以快速取得解決方案。 

## <a name="troubleshooting-steps-for-common-errors"></a>為常見錯誤而設的疑難排解步驟

| 錯誤碼 | 疑難排解步驟 |
| ---------- | --------------------- |
| **CONTACT_SUPPORT** | 請[連絡支援人員](#contact-microsoft-support)，並提供步驟清單以利收集記錄。 請儘可能多提供錯誤發生前的相關資訊，包括租用戶識別碼和使用者主體名稱 (UPN) 等。 |
| **CLIENT_CERT_INSTALL_ERROR** | 可能是用戶端憑證的安裝方式或與您租用戶建立關聯的方式出了問題。 請遵循[針對 Azure MFA NPS 擴充功能進行移難排解](howto-mfa-nps-extension.md#troubleshooting)中的指示來調查用戶端憑證問題。 |
| **ESTS_TOKEN_ERROR** | 請遵循[針對 Azure MFA NPS 擴充功能進行移難排解](howto-mfa-nps-extension.md#troubleshooting)中的指示來調查用戶端憑證和 ADAL 權杖問題。 |
| **HTTPS_COMMUNICATION_ERROR** | NPS 伺服器無法接收來自 Azure MFA 的的回應。 請確定您的防火牆已開放讓 https://adnotifications.windowsazure.com 可雙向傳輸流量 |
| **HTTP_CONNECT_ERROR** | 在執行 NPS 延伸模組的伺服器上，確認您可以連線到 https://adnotifications.windowsazure.com 和 https://login.microsoftonline.com/。 如果無法載入這些站台，請針對該伺服器上的連線進行移難排解。 |
| **REGISTRY_CONFIG_ERROR** | 應用程式登錄中遺失機碼，可能是因為 [PowerShell 指令碼](howto-mfa-nps-extension.md#install-the-nps-extension)並未在安裝後執行。 錯誤訊息應包含遺失的機碼。 請確定您在 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa 下有該機碼。 |
| **REQUEST_FORMAT_ERROR** <br> RADIUS 要求中遺失必要的 RADIUS 使用者名稱\識別碼。請確定 NPS 會收到 RADIUS 要求 | 此錯誤通常反映的是安裝問題。 NPS 擴充功能必須安裝在可接收 RADIUS 要求的 NPS 伺服器。 NPS 伺服器若是作為 RDG 和 RRAS 等服務的相依項目安裝，則不會收到 RADIUS 要求。 NPS 擴充功能若透過這類安裝方式進行安裝就會無法運作，並會因為無法讀取來自驗證要求的詳細資料而發生錯誤。 |
| **REQUEST_MISSING_CODE** | 請確定 NPS 和 NAS 伺服器之間的密碼加密協定支援您使用的次要驗證方法。 **PAP** 支援雲端中 Azure MFA 的所有驗證方法：通話、單向簡訊、行動裝置應用程式通知，和行動裝置應用程式驗證碼。 **CHAPV2** 和 **EAP** 支援通話和行動裝置應用程式通知。 |
| **USERNAME_CANONICALIZATION_ERROR** | 請確定使用者已存在於內部部署的 Active Directory 執行個體中，而且 NPS 服務有權存取目錄。 如果您使用跨樹系信任，請[連絡支援人員](#contact-microsoft-support)以取得進一步協助。 |


   

### <a name="alternate-login-id-errors"></a>替代登入識別碼錯誤

| 錯誤碼 | 錯誤訊息 | 疑難排解步驟 |
| ---------- | ------------- | --------------------- |
| **ALTERNATE_LOGIN_ID_ERROR** | 錯誤：userObjectSid 查閱失敗 | 請確認使用者存在於內部部署 Active Directory 執行個體中。 如果您使用跨樹系信任，請[連絡支援人員](#contact-microsoft-support)以取得進一步協助。 |
| **ALTERNATE_LOGIN_ID_ERROR** | 錯誤：替代 LoginId 查閱失敗 | 請確認 LDAP_ALTERNATE_LOGINID_ATTRIBUTE 設定為[有效的 Active Directory 屬性](https://msdn.microsoft.com/library/ms675090(v=vs.85).aspx)。 <br><br> 如果 LDAP_FORCE_GLOBAL_CATALOG 設定為 True，或 LDAP_LOOKUP_FORESTS 設定具有非空白值，請確認您已設定通用類別目錄，且 AlternateLoginId 屬性已新增到其中。 <br><br> 如果 LDAP_LOOKUP_FORESTS 設定具有非空白值，請確認值是否正確。 如果有多個樹系名稱，名稱必須以分號分隔，而非空格。 <br><br> 如果上述步驟未能解決問題，請[連絡支援人員](#contact-microsoft-support)以取得更多協助。 |
| **ALTERNATE_LOGIN_ID_ERROR** | 錯誤：替代 LoginId 值是空的 | 請確認已為使用者設定 AlternateLoginId 屬性。 |


## <a name="errors-your-users-may-encounter"></a>您的使用者可能會遇到下列錯誤

| 錯誤碼 | 錯誤訊息 | 疑難排解步驟 |
| ---------- | ------------- | --------------------- |
| **AccessDenied** | 呼叫者租用戶並沒有執行使用者驗證的存取權限 | 請檢查租用戶網域和使用者主要名稱 (UPN) 的網域是否相同。 例如，請確定 user@contoso.com 正在嘗試驗證的是 Contoso 租用戶。 UPN 代表 Azure 中有效的租用戶使用者。 |
| **AuthenticationMethodNotConfigured** | 使用者未設定指定的驗證方法 | 請讓使用者根據[管理您的雙步驟驗證設定](../user-help/multi-factor-authentication-end-user-manage-settings.md)中的指示，來新增或驗證他們的驗證方法。 |
| **AuthenticationMethodNotSupported** | 不支援指定的驗證方法。 | 請收集所有包含此錯誤的記錄，並[連絡支援人員](#contact-microsoft-support)。 當您連絡支援人員時，請提供使用者名稱和觸發錯誤的次要驗證方法。 |
| **BecAccessDenied** | MSODS Bec 呼叫傳回拒絕存取，可能是因為使用者名稱未在租用戶中定義 | 使用者存在於 Active Directory 內部部署中，但未透過 AD Connect 同步至 Azure AD。 或者，租用戶遺失使用者。 請將使用者新增至 Azure AD，並讓使用者根據[管理您的雙步驟驗證設定](../user-help/multi-factor-authentication-end-user-manage-settings.md)中的指示，來新增他們的驗證方法。 |
| **InvalidFormat** 或 **StrongAuthenticationServiceInvalidParameter** | 無法辨識電話號碼格式 | 請使用者修正他們的驗證電話號碼。 |
| **InvalidSession** | 指定的工作階段無效或已過期 | 工作階段費時三分鐘以上才完成。 請確定使用者在驗證要求啟動後的三分鐘內輸入驗證碼或回應應用程式通知。 如果這麼做沒有修正問題，請檢查用戶端、NAS 伺服器、NPS 伺服器與 Azure MFA 端點之間沒有網路延遲的情形。  |
| **NoDefaultAuthenticationMethodIsConfigured** | 使用者未設定預設的驗證方法 | 請讓使用者根據[管理您的雙步驟驗證設定](../user-help/multi-factor-authentication-end-user-manage-settings.md)中的指示，來新增或驗證他們的驗證方法。 請確定使用者已選擇預設驗證方法，且已為他們的帳戶設定該方法。 |
| **OathCodePinIncorrect** | 輸入的代碼和 Pin 碼錯誤。 | 此錯誤不應出現在 NPS 擴充功能中。 如果您的使用者遇到此錯誤，請[連絡支援人員](#contact-microsoft-support)以取得疑難排解說明。 |
| **ProofDataNotFound** | 指定的驗證方法未設定證明資料。 | 請讓使用者根據[管理您的雙步驟驗證設定](../user-help/multi-factor-authentication-end-user-manage-settings.md)中的指示，嘗試不同驗證方法或新增驗證法。 如果您確認使用者已正確設定他們的驗證方法，但使用者仍持續看見此錯誤，請[連絡支援人員](#contact-microsoft-support)。 |
| **SMSAuthFailedWrongCodePinEntered** | 輸入的代碼和 Pin 碼錯誤。 (OneWaySMS) | 此錯誤不應出現在 NPS 擴充功能中。 如果您的使用者遇到此錯誤，請[連絡支援人員](#contact-microsoft-support)以取得疑難排解說明。 |
| **TenantIsBlocked** | 租用戶已遭封鎖 | 請從 Azure 入口網站中的 Azure AD 屬性頁面中取得 Directory 識別碼並[連絡支援人員](#contact-microsoft-support)。 |
| **UserNotFound** | 找不到指定的使用者 | 此租用戶在 Azure AD 中已不是顯示為作用中的租用戶。 請檢查您的訂用帳戶是否在作用中，以及您是否有必要的第一方應用程式。 也請確定憑證主體中的租用戶正確，且憑證仍然有效，並已註冊於服務主體下。 |

## <a name="messages-your-users-may-encounter-that-arent-errors"></a>使用者收到的訊息可能不是錯誤訊息

有時候，由於您使用者的驗證要求失敗，他們可能會收到來自 Multi-Factor Authentication 的訊息。 這不是產品組態中發生錯誤，而是說明為何驗證要求遭拒的內部警告。

| 錯誤碼 | 錯誤訊息 | 建議的步驟 | 
| ---------- | ------------- | ----------------- |
| **OathCodeIncorrect** | 輸入的代碼錯誤\OATH 代碼錯誤 | 這不是錯誤，是使用者輸入的代碼有誤。 | 使用者輸入了錯誤的代碼。 請要求新代碼並讓他們再試一次，或請他們重新登入。 | 
| **SMSAuthFailedMaxAllowedCodeRetryReached** | 已達重試代碼次數的上限 | 使用者的驗證嘗試失敗太多次。 視您的設定而定，使用者現在可能需要透過系統管理員來解除封鎖。  |
| **SMSAuthFailedWrongCodeEntered** | 輸入的代碼錯誤/文字訊息 OTP 不正確 | 使用者輸入了錯誤的代碼。 請要求新代碼並讓他們再試一次，或請他們重新登入。 |

## <a name="errors-that-require-support"></a>需要支援的錯誤

如果您遇到以下其中一種錯誤，我們建議您[連絡支援人員](#contact-microsoft-support)以取得診斷協助。 這些錯誤並沒有標準的解決步驟。 當您連絡支援人員時，請務必多提供導致此錯誤的步驟資訊，以及您的租用戶資訊。

| 錯誤碼 | 錯誤訊息 |
| ---------- | ------------- |
| **InvalidParameter** | 要求不能是 Null |
| **InvalidParameter** | 對 ReplicationScope:{0} 而言，ObjectId 不能是 Null 或空白 |
| **InvalidParameter** | 公司名稱長度 \{0}\ 超過允許的長度上限 {1} |
| **InvalidParameter** | UserPrincipalName 不能是 Null 或空白 |
| **InvalidParameter** | 提供的 TenantId 格式不正確 |
| **InvalidParameter** | SessionId 不能是 Null 或空白 |
| **InvalidParameter** | 無法解析任何來自要求或 Msods 的 ProofData。 ProofData 不明 |
| **InternalError** |  |
| **OathCodePinIncorrect** |  |
| **VersionNotSupported** |  |
| **MFAPinNotSetup** |  |

## <a name="next-steps"></a>後續步驟

### <a name="troubleshoot-user-accounts"></a>針對使用者帳戶進行移難排解

如果您的使用者[進行雙步驟驗證時發生問題](../user-help/multi-factor-authentication-end-user-troubleshoot.md)，請協助助他們自行診斷問題。 

### <a name="contact-microsoft-support"></a>連絡 Microsoft 支援

如需其他協助，請透過 [Azure Multi-Factor Authentication Server 支援](https://support.microsoft.com/oas/default.aspx?prid=14947)連絡支援專業人員。 連絡我們時，請盡量包含有關問題的最多資訊，這樣會十分有幫助。 您可以提供的資訊包含您看到錯誤的頁面、特定錯誤碼、特定工作階段 ID、使用者 (看到錯誤者) 識別碼，以及偵錯記錄。

若要收集偵錯記錄以支援診斷，請在 NPS 伺服器上使用下列步驟︰

1. 開啟登錄編輯程式，並瀏覽至 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa，將 **VERBOSE_LOG** 設定為 **TRUE**
2. 開啟系統管理員命令提示字元並執行下列命令︰

   ```
   Mkdir c:\NPS
   Cd NPS
   netsh trace start Scenario=NetConnection capture=yes tracefile=c:\NPS\nettrace.etl
   logman create trace "NPSExtension" -ow -o c:\NPS\NPSExtension.etl -p {7237ED00-E119-430B-AB0F-C63360C8EE81} 0xffffffffffffffff 0xff -nb 16 16 -bs 1024 -mode Circular -f bincirc -max 4096 -ets
   logman update trace "NPSExtension" -p {EC2E6D3A-C958-4C76-8EA4-0262520886FF} 0xffffffffffffffff 0xff -ets
   ```

3. 重現問題

4. 使用下列命令停止追蹤︰

   ```
   logman stop "NPSExtension" -ets
   netsh trace stop
   wevtutil epl AuthNOptCh C:\NPS\%computername%_AuthNOptCh.evtx
   wevtutil epl AuthZOptCh C:\NPS\%computername%_AuthZOptCh.evtx
   wevtutil epl AuthZAdminCh C:\NPS\%computername%_AuthZAdminCh.evtx
   Start .
   ```

5. 開啟登錄編輯程式，並瀏覽至 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa，將 **VERBOSE_LOG** 設定為 **FALSE**
6. 壓縮 C:\NPS 資料夾的內容，並將壓縮的檔案附加到支援案例。


