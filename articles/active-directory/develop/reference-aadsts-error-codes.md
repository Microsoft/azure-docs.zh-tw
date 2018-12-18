---
title: Azure Active Directory 驗證與授權錯誤碼 | Microsoft Docs
description: 了解從 Azure AD 安全性權杖服務 (STS) 所傳回的 AADSTS 錯誤碼。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: hirsin, justhu
ms.custom: aaddev
ms.openlocfilehash: 0643ae39d24f8ef0c1c1c18dcf73cf840c95e277
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956318"
---
# <a name="authentication-and-authorization-error-codes"></a>驗證與授權錯誤碼

正在尋找有關從 Azure AD 安全性權杖服務 (STS) 所傳回 AADSTS 錯誤碼的資訊嗎？ 請閱讀此文件，以尋找 AADSTS 錯誤描述、修正程式，以及一些建議的因應措施。

> [!NOTE]
> 此資訊為初步資訊，隨時可能變更。 有任何疑問或找不到您要尋找的內容嗎？ 建立 GitHub 問題，或參閱[適用於開發人員的支援和協助選項](active-directory-develop-help-support.md)，了解您可以取得協助和支援的其他方法。

## <a name="aadsts-error-codes"></a>AADSTS 錯誤碼

| Error | 說明 |
|---|---|
| AADSTS16000 | SelectUserAccount - 這是 Azure AD 所擲回的插斷，會導致 UI 允許使用者從多個有效的 SSO 工作階段進行選取。 此錯誤很常見，如果指定 `prompt=none`，則可能會傳回給應用程式。 |
| AADSTS16001 | UserAccountSelectionInvalid - 如果使用者按一下工作階段選取邏輯已遭拒絕的圖格，則會看到此錯誤。 若已觸發，此錯誤可讓使用者藉由從已更新的圖格/工作階段清單進行挑選，或選擇另一個帳戶來復原。 此錯誤可能因為程式碼缺失或競爭條件而發生。 |
| AADSTS16002 | AppSessionSelectionInvalid - 不符合應用程式指定的 SID 需求。  |
| AADSTS16003 | SsoUserAccountNotFoundInResourceTenant |
| AADSTS17003 | CredentialKeyProvisioningFailed |
| AADSTS20001 | WsFedSignInResponseError - 同盟識別提供者有問題。 請連絡 IDP 以解決此問題。 |
| AADSTS20012 | WsFedMessageInvalid - 同盟識別提供者有問題。 請連絡 IDP 以解決此問題。 |
| AADSTS20033 | FedMetadataInvalidTenantName - 同盟識別提供者有問題。 請連絡 IDP 以解決此問題。 |
| AADSTS40008 | OAuth2IdPUnretryableServerError - 同盟識別提供者有問題。 請連絡 IDP 以解決此問題。 |
| AADSTS40009 | OAuth2IdPRefreshTokenRedemptionUserError - 同盟識別提供者有問題。 請連絡 IDP 以解決此問題。 |
| AADSTS40010 | OAuth2IdPRetryableServerError - 同盟識別提供者有問題。 請連絡 IDP 以解決此問題。 |
| AADSTS40015 | OAuth2IdPAuthCodeRedemptionUserError - 同盟識別提供者有問題。 請連絡 IDP 以解決此問題。 |
| AADSTS50000 | TokenIssuanceError - 登入服務有問題。 請[開啟支援票證](../fundamentals/active-directory-troubleshooting-support-howto.md)以解決此問題。 |
| AADSTS50001 | InvalidResource - 資源已停用或不存在。 請檢查您應用程式的程式碼，確保您已經為您嘗試存取的資源指定確切的資源 URL。  |
| AADSTS50002 | NotAllowedTenant - 登入失敗，因為租用戶上的 Proxy 存取受限。 如果該租用戶原則是您自有的，您可以變更受限租用戶的設定以修正此問題。 |
| AADSTS50003 | MissingSigningKey - 登入失敗，因為遺漏簽署金鑰或憑證。 這可能是因為應用程式中未設定任何簽署金鑰。 請參閱下列文件所述的解決方式：[https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#certificate-or-key-not-configured](https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#certificate-or-key-not-configured)。 如果還是有問題，請連絡應用程式擁有者或應用程式管理員。 |
| AADSTS50005 | DevicePolicyError - 使用者嘗試透過條件式存取原則從目前不受支援的平台登入裝置。 |
| AADSTS50006 | InvalidSignature - 簽章驗證失敗，因為簽章無效。 |
| AADSTS50007 | PartnerEncryptionCertificateMissing - 找不到此應用程式的夥伴加密憑證。 請向 Microsoft [開啟支援票證](../fundamentals/active-directory-troubleshooting-support-howto.md)以修正此問題。 |
| AADSTS50008 | InvalidSamlToken - 權杖中的 SAML 判斷提示遺漏或設定不正確。 請連絡同盟提供者。 |
| AADSTS50010 | AudienceUriValidationFailed - 應用程式的對象 URI 驗證失敗，因為未設定權杖對象。 |
| AADSTS50011 | InvalidReplyTo - 回覆地址遺漏、設定不正確或不符合針對應用程式所設定的回覆地址。 請嘗試使用下列文件所列出的解決方式：[https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#the-reply-address-does-not-match-the-reply-addresses-configured-for-the-application](https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#the-reply-address-does-not-match-the-reply-addresses-configured-for-the-application)。 如果還是有問題，請連絡應用程式擁有者或應用程式管理員。 |
| AADSTS50012 | AuthenticationFailed - 驗證因為下列其中一個原因而失敗：<ul><li>簽署憑證的主旨名稱未獲授權</li><li>已授權的主體名稱找不到相符的受信任授權原則</li><li>憑證鏈結無效</li><li>簽署憑證無效</li><li>未在租用戶上設定原則</li><li>簽署憑證的指紋未獲授權</li><li>用戶端判斷提示包含無效的簽章</li></ul> |
| AADSTS50013 | InvalidAssertion - 判斷提示因為各種原因而無效：權杖簽發者不符合其有效時間範圍內的 API 版本；過期；格式不正確；判斷提示中的重新整理權杖不是主要的重新整理權杖。 |
| AADSTS50014 | GuestUserInPendingState |
| AADSTS50015 | ViralUserLegalAgeConsentRequiredState |
| AADSTS50017 | CertificateValidationFailed - 憑證驗證失敗，原因如下：<ul><li>在受信任的憑證清單中找不到發行憑證</li><li>找不到預期的 CrlSegment</li><li>在受信任的憑證清單中找不到發行憑證</li><li>在設定差異 CRL 發佈點時未指定對應的 CRL 發佈點</li><li>由於逾時問題而無法擷取有效的 CRL 區段</li><li>無法下載 CRL</li></ul>請連絡租用戶管理員。 |
| AADSTS50020 | UserUnauthorized - 使用者未獲授權呼叫此端點。 |
| AADSTS50027 | InvalidJwtToken - JWT 權杖無效，原因如下：<ul><li>未包含 nonce 宣告、子宣告</li><li>主旨識別碼不相符</li><li>idToken 宣告中的宣告重複</li><li>未預期的簽發者</li><li>未預期的對象</li><li>不在其有效時間範圍內 </li><li>權杖格式不正確</li><li>來自簽發者的外部 ID 權杖未能通過簽章驗證。</li></ul> |
| AADSTS50029 | URI 無效 - 網域名稱包含無效字元。 請連絡租用戶管理員。 |
| AADSTS50032 | WeakRsaKey - 表示錯誤的使用者嘗試使用弱式 RSA 金鑰。 |
| AADSTS50033 | RetryableError - 表示與資料庫作業無關的暫時性錯誤。 |
| AADSTS50034 | UserAccountNotFound - 若要登入此應用程式，必須將帳戶新增至目錄。 |
| AADSTS50042 | UnableToGeneratePairwiseIdentifierWithMissingSalt - 主體中遺漏欲產生成對識別碼所需的 salt。 請連絡租用戶管理員。 |
| AADSTS50043 | UnableToGeneratePairwiseIdentifierWithMultipleSalts |
| AADSTS50048 | SubjectMismatchesIssuer - 主體與用戶端判斷提示中的簽發者宣告不符。 請連絡租用戶管理員。 |
| AADSTS50049 | NoSuchInstanceForDiscovery - 未知或無效的執行個體。 |
| AADSTS50050 | MalformedDiscoveryRequest - 要求格式不正確。 |
| AADSTS50053 | IdsLocked - 帳戶遭到鎖定，因為使用者嘗試使用不正確的使用者識別碼或密碼登入太多次。 |
| AADSTS50055 | InvalidPasswordExpiredPassword - 密碼已過期。 |
| AADSTS50056 | 密碼無效或為 Null - 存放區中沒有此使用者的密碼。 |
| AADSTS50057 | UserDisabled - 使用者帳戶已停用。 系統管理員已停用該帳戶。 |
| AADSTS50058 | UserInformationNotProvided - 這表示使用者並未登入。 這是很常見的錯誤，當使用者未經驗證且尚未登入時就會發生。</br>如果使用者先前剛登入的 SSO 內容中促成此錯誤，這表示 SSO 工作階段找不到或無效。</br>如果已指定 prompt=none，此錯誤可能會傳回給應用程式。 |
| AADSTS50059 | MissingTenantRealmAndNoUserInformationProvided - 在要求中找不到租用戶識別資訊，或任何提供的認證均未隱含租用戶識別資訊。 使用者可以連絡租用戶管理員，協助解決問題。 |
| AADSTS50061 | SignoutInvalidRequest - 登出要求無效。 |
| AADSTS50064 | CredentialAuthenticationError |
| AADSTS50068 | SignoutInitiatorNotParticipant |
| AADSTS50070 | SignoutUnknownSessionIdentifier |
| AADSTS50071 | SignoutMessageExpired |
| AADSTS50072 | UserStrongAuthEnrollmentRequiredInterrupt - 使用者需要註冊第二個因素驗證 (互動式)。 |
| AADSTS50074 | UserStrongAuthClientAuthNRequiredInterrupt - 需要強大的驗證，而且使用者並未通過 MFA 挑戰。 |
| AADSTS50076 | UserStrongAuthClientAuthNRequired - 使用者必須使用多重要素驗證來存取此資源。 使用資源的新授權要求進行重試。 |
| AADSTS50079 | UserStrongAuthEnrollmentRequired - 由於系統管理員所做的組態變更，或因為使用者已移至新位置，所以使用者需要使用多重要素驗證。 |
| AADSTS50085 | 重新整理權杖需要社交 IDP 登入。 讓使用者嘗試以使用者名稱/密碼重新登入 |
| AADSTS50086 | SasNonRetryableError |
| AADSTS50087 | SasRetryableError |
| AADSTS50089 | 流程權杖過期 - 驗證失敗。 讓使用者嘗試以使用者名稱/密碼重新登入 |
| AADSTS50097 | DeviceAuthenticationRequired - 需要裝置驗證。 |
| AADSTS50099 | PKeyAuthInvalidJwtUnauthorized - JWT 簽章無效。 |
| AADSTS50105 | EntitlementGrantsNotFound - 未將登入的使用者指派給所登入應用程式的角色。 請將使用者指派給應用程式。 取得詳細資訊：[https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#user-not-assigned-a-role](https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#user-not-assigned-a-role)。 |
| AADSTS50107 | InvalidRealmUri - 所要求的同盟領域物件不存在。 請連絡租用戶管理員。 |
| AADSTS50120 | ThresholdJwtInvalidJwtFormat - JWT 標頭有問題。 請連絡租用戶管理員。 |
| AADSTS50124 | ClaimsTransformationInvalidInputParameter - 宣告轉換所包含的輸入參數無效。 請連絡租用戶管理員以更新原則。 |
| AADSTS50125 | PasswordResetRegistrationRequiredInterrupt - 因為密碼重設或密碼註冊項目，登入作業已中斷。 |
| AADSTS50126 | InvalidUserNameOrPassword - 驗證認證時發生錯誤，因為使用者名稱或密碼無效。 |
| AADSTS50127 | BrokerAppNotInstalled - 使用者必須安裝訊息代理程式應用程式來存取此內容。 |
| AADSTS50128 | 網域名稱無效 - 在要求中找不到租用戶識別資訊，或任何提供的認證均未隱含租用戶識別資訊|
| AADSTS50129 | DeviceIsNotWorkplaceJoined - 註冊裝置需加入工作場所網路。 |
| AADSTS50131 | ConditionalAccessFailed - 表示各種條件式存取錯誤，例如：不正確的 Windows 裝置狀態，要求因為可疑的活動、存取原則或安全性原則決策而遭到封鎖。 |
| AADSTS50132 | SsoArtifactInvalidOrExpired - 工作階段因為密碼到期或近期密碼變更而無效。 |
| AADSTS50133 | SsoArtifactRevoked - 工作階段因為密碼到期或近期密碼變更而無效。 |
| AADSTS50134 | DeviceFlowAuthorizeWrongDatacenter |
| AADSTS50135 | PasswordChangeCompromisedPassword - 帳戶有風險，因此必須變更密碼。 |
| AADSTS50136 | RedirectMsaSessionToApp - 偵測到單一 MSA 工作階段。 |
| AADSTS50139 | SessionMissingMsaOAuth2RefreshToken |
| AADSTS50140 | KmsiInterrupt - 由於在使用者登入時出現「讓我保持登入」插斷，所以發生此錯誤。 請使用相互關聯識別碼、要求識別碼和錯誤碼[開啟支援票證](../fundamentals/active-directory-troubleshooting-support-howto.md)，以取得更多詳細資料。 |
| AADSTS50143 | 工作階段不符 - 工作階段無效，原因是由於資源不同而導致使用者租用戶與網域提示不符。 請使用相互關聯識別碼、要求識別碼和錯誤碼[開啟支援票證](../fundamentals/active-directory-troubleshooting-support-howto.md)，以取得更多詳細資料。 |
| AADSTS50144 | InvalidPasswordExpiredOnPremPassword - 使用者的 Active Directory 密碼已到期。 為使用者產生新密碼，或讓使用者使用自助式重設工具來重設其密碼。 |
| AADSTS50146 | MissingCustomSigningKey - 在設定此應用程式時，必須使用應用程式專屬的簽署金鑰。 在設定應用程式時未使用這樣的金鑰，或該金鑰已過期或尚未生效。 |
| AADSTS50147 | MissingCodeChallenge |
| AADSTS50155 | DeviceAuthenticationFailed - 此使用者的裝置驗證失敗。 |
| AADSTS50158 | ExternalSecurityChallenge - 不符合外部安全性挑戰。 |
| AADSTS50161 | InvalidExternalSecurityChallengeConfiguration - 外部提供者所傳送的宣告不足，或是遺漏向外部提供者所要求的宣告。 |
| AADSTS50166 | ExternalClaimsProviderThrottled - 無法將要求傳送給宣告提供者。 |
| AADSTS50168 | ChromeBrowserSsoInterruptRequired - 用戶端能夠透過 Windows 10 帳戶擴充功能取得 SSO 權杖，但是在要求中找不到權杖，或提供的權杖已過期。 |
| AADSTS50169 | InvalidRequestBadRealm - 此領域不是當前服務命名空間所設定的領域。 |
| AADSTS50170 | MissingExternalClaimsProviderMapping |
| AADSTS50177 | ExternalChallengeNotSupportedForPassthroughUsers - 未針對傳遞使用者支援外部挑戰。 |
| AADSTS50178 | SessionControlNotSupportedForPassthroughUsers - 未針對傳遞使用者支援工作階段控制。 |
| AADSTS50180 | WindowsIntegratedAuthMissing - 需要整合式 Windows 驗證。 為租用戶啟用無縫 SSO。 |
| AADSTS50187 | DeviceInformationNotProvided |
| AADSTS51000 | RequiredFeatureNotEnabled |
| AADSTS51001 | DomainHintMustbePresent - 網域提示必須與內部部署安全性識別碼或 內部部署 UPN 一起顯示。 |
| AADSTS51004 | UserAccountNotInDirectory - 目錄中不存在使用者帳戶。 |
| AADSTS51005 | TemporaryRedirect - 相當於 HTTP 狀態 307，這表示所要求的資訊位於位置標頭中指定的 URI。 當您收到此狀態時，請遵循與回應相關聯的位置標頭。 當原始要求方法為 POST 時，重新導向的要求也會使用 POST 方法。 |
| AADSTS51006 | ForceReauthDueToInsufficientAuth - 需要整合式 Windows 驗證。 使用遺漏整合式 Windows 驗證宣告的工作階段權杖來登入的使用者。 請要求使用者再次登入。 |
| AADSTS52004 | DelegationDoesNotExistForLinkedIn - 使用者尚未同意存取 LinkedIn 資源。 |
| AADSTS53000 | DeviceNotCompliant - 條件式存取原則需要相容的裝置，但裝置不相容。 使用者必須向 Intune 等獲得核准的 MDM 提供者註冊其裝置。 |
| AADSTS53001 | DeviceNotDomainJoined - 條件式存取原則需要已加入網域的裝置，但該裝置尚未加入網域。 請使用者使用已加入網域的裝置。 |
| AADSTS53002 | ApplicationUsedIsNotAnApprovedApp - 所使用的應用程式不是已核准進行條件式存取的應用程式。 使用者所使用的應用程式，必須是已核准使用的應用程式清單其中一個，才能取得存取權。 |
| AADSTS53003 | BlockedByConditionalAccess - 存取遭到條件式存取原則封鎖。 存取原則不允許發行權杖。 |
| AADSTS53004 | ProofUpBlockedDueToRisk - 使用者必須先完成多重要素驗證註冊程序，才能存取此內容。 使用者應該註冊多重要素驗證。 |
| AADSTS54000 | MinorUserBlockedLegalAgeGroupRule |
| AADSTS65001 | DelegationDoesNotExist - 使用者或系統管理員尚未同意使用識別碼為 X 的應用程式。針對此使用者和資源傳送互動式授權要求。 |
| AADSTS65004 | UserDeclinedConsent - 使用者不同意存取應用程式。 讓使用者重試登入，並同意應用程式|
| AADSTS65005 | MisconfiguredApplication - 應用程式所需資源存取清單不包含資源可探索的應用程式，或用戶端應用程式已要求存取未在其所需資源存取清單中指定的資源，或 Graph 服務傳回不正確的要求或找不到資源。 如果應用程式支援 SAML，則可能是您在設定應用程式時所使用的識別碼 (實體) 不正確。 請嘗試使用下列連結中針對 SAML 所列出的解決方案：[https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#no-resource-in-requiredresourceaccess-list](https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery?/?WT.mc_id=DMC_AAD_Manage_Apps_Troubleshooting_Nav#no-resource-in-requiredresourceaccess-list) |
| AADSTS67003 | ActorNotValidServiceIdentity |
| AADSTS70000 | InvalidGrant - 驗證失敗。 重新整理權杖無效。 錯誤原因如下：<ul><li>權杖繫結標頭是空的</li><li>權杖繫結雜湊不相符</li></ul> |
| AADSTS70001 | UnauthorizedClient - 應用程式已停用。 |
| AADSTS70002 | InvalidClient - 驗證認證時發生錯誤。 指定的 client_secret 與此用戶端預期的值不符。 請修正 client_secret，然後再試一次。 如需詳細資訊，請參閱[使用授權碼來要求存取權杖](v1-protocols-oauth-code.md#use-the-authorization-code-to-request-an-access-token)。 |
| AADSTS70003 | UnsupportedGrantType - 應用程式傳回的授與類型不受支援。 |
| AADSTS70004 | InvalidRedirectUri - 應用程式傳回的重新導向 URI 無效。 用戶端所指定的重新導向位址，與已設定的位址或 OIDC 核准清單上的位址全都不符。 |
| AADSTS70005 | UnsupportedResponseType - 應用程式傳回的回應類型不受支援，原因如下：<ul><li>未對應用程式啟用 'token' 回應類型</li><li>'id_token' 回應類型需要 'OpenID' 範圍 - 在編碼的 wctx 中包含不受支援的 OAuth 參數值</li></ul> |
| AADSTS70007 | UnsupportedResponseMode - 應用程式在要求權杖時傳回的 `response_mode` 值不受支援。  |
| AADSTS70008 | ExpiredOrRevokedGrant - 重新整理權杖因為非使用狀態而過期。 此權杖是在 XXX 發出，而且有一段時間為非使用中。 |
| AADSTS70011 | InvalidScope - 應用程式所要求的範圍無效。 |
| AADSTS70012 | MsaServerError - 在驗證 MSA (取用者) 使用者時，發生伺服器錯誤。 請再試一次。 如果持續失敗，請[開啟支援票證](../fundamentals/active-directory-troubleshooting-support-howto.md) |
| AADSTS70016 | AuthorizationPending - OAuth 2.0 裝置流程錯誤。 授權已擱置。 裝置會重試輪詢要求。 |
| AADSTS70018 | BadVerificationCode - 驗證碼無效，因為使用者針對裝置碼流程所輸入的使用者代碼錯誤。 授權未獲核准。 |
| AADSTS70019 | CodeExpired - 驗證碼過期。 請讓使用者重試登入。 |
| AADSTS75001 | BindingSerializationError - SAML 訊息繫結期間發生錯誤。 |
| AADSTS75003 | UnsupportedBindingError - 應用程式傳回了與未支援繫結相關的錯誤 (無法透過 HTTP POST 以外的繫結傳送 SAML 通訊協定回應)。 |
| AADSTS75005 | Saml2MessageInvalid - Azure AD 不支援應用程式針對 SSO 傳送的 SAML 要求。 |
| AADSTS75008 | RequestDeniedError - 應用程式所提出的要求遭拒，因為 SAML 要求有未預期的目的地。 |
| AADSTS75011 | NoMatchedAuthnContextInOutputClaims - 使用者用來向服務進行驗證的驗證方法，與所要求的驗證方法不符。 |
| AADSTS75016 | Saml2AuthenticationRequestInvalidNameIDPolicy - SAML2 驗證要求所具有的 NameIdPolicy 無效。 |
| AADSTS80001 | OnPremiseStoreIsNotAvailable - 驗證代理程式無法連線至 Active Directory。 確定代理程式伺服器和必須驗證其密碼的使用者都是相同 AD 樹系的成員，而且都能連線到 Active Directory。 |
| AADSTS80002 | OnPremisePasswordValidatorRequestTimedout - 密碼驗證要求逾時。確定 Active Directory 可用，並且會回應來自代理程式的要求。 |
| AADSTS80005 | OnPremisePasswordValidatorUnpredictableWebException - 處理驗證代理程式所傳來的回應時，發生未知錯誤。 重試要求。 如果持續失敗，請[開啟支援票證](../fundamentals/active-directory-troubleshooting-support-howto.md)，以取得有關錯誤的詳細資料。 |
| AADSTS80007 | OnPremisePasswordValidatorErrorOccurredOnPrem - 驗證代理程式無法驗證使用者的密碼。 請檢查代理程式記錄檔以了解詳細資訊，並確認 Active Directory 如預期般運作。 |
| AADSTS80010 | OnPremisePasswordValidationEncryptionException - 驗證代理程式無法將密碼解密。 |
| AADSTS80012 | OnPremisePasswordValidationAccountLogonInvalidHours - 使用者在允許的時間外 (此值是在 AD 中指定的) 嘗試登入。 |
| AADSTS80013 | OnPremisePasswordValidationTimeSkew - 無法完成驗證嘗試，因為執行驗證代理程式的機器和 AD 之間有時間差異。 請修正時間同步問題。 |
| AADSTS81004 | DesktopSsoIdentityInTicketIsNotAuthenticated - Kerberos 驗證嘗試失敗。 |
| AADSTS81005 | DesktopSsoAuthenticationPackageNotSupported - 不支援驗證套件。 |
| AADSTS81006 | DesktopSsoNoAuthorizationHeader - 找不到授權標頭。 |
| AADSTS81007 | DesktopSsoTenantIsNotOptIn - 租用戶未啟用無縫 SSO。 |
| AADSTS81009 | DesktopSsoAuthorizationHeaderValueWithBadFormat - 無法驗證使用者的 Kerberos 票證。 |
| AADSTS81010 | DesktopSsoAuthTokenInvalid - 無縫式 SSO 失敗，因為使用者的 Kerberos 票證已過期或無效。 |
| AADSTS81011 | DesktopSsoLookupUserBySidFailed - 找不到以使用者的 Kerberos 票證中資訊為基礎的使用者物件。 |
| AADSTS81012 | DesktopSsoMismatchBetweenTokenUpnAndChosenUpn - 嘗試登入 Azure AD 的使用者與登入裝置的使用者不同。 |
| AADSTS90002 | InvalidTenantName - 資料存放區中找不到租用戶名稱。 檢查以確定您有正確的租用戶識別碼。 |
| AADSTS90004 | InvalidRequestFormat |
| AADSTS90005 | InvalidRequestWithMultipleRequirements |
| AADSTS90006 | ExternalServerRetryableError |
| AADSTS90007 | InvalidSessionId |
| AADSTS90008 | TokenForItselfRequiresGraphPermission |
| AADSTS90009 | TokenForItselfMissingIdenticalAppIdentifier |
| AADSTS90010 | NotSupported |
| AADSTS90012 | RequestTimeout |
| AADSTS90013 | InvalidUserInput |
| AADSTS90014 | MissingRequiredField - 當認證中未出現預期的欄位時，此錯誤碼可能會出現於各種情況。 |
| AADSTS90015 | QueryStringTooLong |
| AADSTS90016 | MissingRequiredClaim |
| AADSTS90019 | MissingTenantRealm - Azure AD 無法判斷要求中的租用戶識別碼。 |
| AADSTS90022 | AuthenticatedInvalidPrincipalNameFormat - 主體名稱格式無效，或不符合預期的 `name[/host][@realm]` 格式。 主體名稱為必要項目，主機和領域是選擇性項目，而且可能設定為 null。 |
| AADSTS90023 | InvalidRequest |
| AADSTS90024 | RequestBudgetExceededError |
| AADSTS90033 | MsodsServiceUnavailable |
| AADSTS90036 | MsodsServiceUnretryableFailure |
| AADSTS90038 | NationalCloudTenantRedirection - 指定的租用戶 'Y' 屬於國家雲端 'X'。 目前的雲端執行個體 'Z' 不會與 X 同盟。傳回了雲端重新導向錯誤。 |
| AADSTS90043 | NationalCloudAuthCodeRedirection |
| AADSTS90051 | InvalidNationalCloudId |
| AADSTS90055 | TenantThrottlingError - 有太多傳入要求。 此例外狀況因為封鎖的租用戶而擲回。 |
| AADSTS90056 | BadResourceRequest - 若要兌換存取權杖的程式碼，應用程式應將 POST 要求傳送至 `/token` 端點。 此外，在此之前，您應該提供授權碼並以 POST 要求將它傳送到 `/token` 端點。 如需 OAuth 2.0 授權碼流程的概觀，請參閱這篇文章：[https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-oauth-code](https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-oauth-code)。 您必須先將使用者導向 `/authorize` 端點，這會傳回 authorization_code。 將要求張貼至 `/token` 端點，使用者就會取得存取權杖。 登入 Azure 入口網站，然後檢查 [應用程式註冊] > [端點]，確認已正確設定兩個端點。 |
| AADSTS90072 | PassThroughUserMfaError |
| AADSTS90081 | OrgIdWsFederationMessageInvalid |
| AADSTS90082 | OrgIdWsFederationNotSupported |
| AADSTS90084 | OrgIdWsFederationGuestNotAllowed |
| AADSTS90085 | OrgIdWsFederationSltRedemptionFailed |
| AADSTS90086 | OrgIdWsTrustDaTokenExpired |
| AADSTS90087 | OrgIdWsFederationMessageCreationFromUriFailed |
| AADSTS90090 | GraphRetryableError |
| AADSTS90091 | GraphServiceUnreachable |
| AADSTS90092 | GraphNonRetryableError |
| AADSTS90093 | GraphUserUnauthorized - 傳回的圖表包含要求的禁止錯誤碼。 |
| AADSTS90094 | AdminConsentRequired |
| AADSTS90100 | InvalidRequestParameter |
| AADSTS90101 | InvalidEmailAddress |
| AADSTS90102 | InvalidUriParameter |
| AADSTS90107 | InvalidXml |
| AADSTS90114 | InvalidExpiryDate |
| AADSTS90117 | InvalidRequestInput |
| AADSTS90119 | InvalidUserCode |
| AADSTS90120 | InvalidDeviceFlowRequest |
| AADSTS90121 | InvalidEmptyRequest |
| AADSTS90123 | IdentityProviderAccessDenied |
| AADSTS90124 | V1ResourceV2GlobalEndpointNotSupported |
| AADSTS90125 | DebugModeEnrollTenantNotFound |
| AADSTS90126 | DebugModeEnrollTenantNotInferred |
| AADSTS90130 | NonConvergedAppV2GlobalEndpointNotSupported |
| AADSTS120000 | PasswordChangeIncorrectCurrentPassword |
| AADSTS120002 | PasswordChangeInvalidNewPasswordWeak |
| AADSTS120003 | PasswordChangeInvalidNewPasswordContainsMemberName |
| AADSTS120004 | PasswordChangeOnPremComplexity |
| AADSTS120005 | PasswordChangeOnPremSuccessCloudFail |
| AADSTS120008 | PasswordChangeAsyncJobStateTerminated |
| AADSTS120011 | PasswordChangeAsyncUpnInferenceFailed |
| AADSTS120012 | PasswordChangeNeedsToHappenOnPrem |
| AADSTS120013 | PasswordChangeOnPremisesConnectivityFailure |
| AADSTS120014 | PasswordChangeOnPremUserAccountLockedOutOrDisabled |
| AADSTS120015 | PasswordChangeADAdminActionRequired |
| AADSTS120016 | PasswordChangeUserNotFoundBySspr |
| AADSTS120018 | PasswordChangePasswordDoesnotComplyFuzzyPolicy |
| AADSTS120020 | PasswordChangeFailure |
| AADSTS120021 | PartnerServiceSsprInternalServiceError |
| AADSTS130004 | NgcKeyNotFound |
| AADSTS130005 | NgcInvalidSignature |
| AADSTS130006 | NgcTransportKeyNotFound |
| AADSTS130007 | NgcDeviceIsDisabled |
| AADSTS130008 | NgcDeviceIsNotFound |
| AADSTS135010 | KeyNotFound |
| AADSTS140000 | InvalidRequestNonce |
| AADSTS140001 | InvalidSessionKey |
| AADSTS165000 | InvalidRequestCanary |
| AADSTS165004 | InvalidApiCanary |
| AADSTS165900 | InvalidApiRequest |
| AADSTS220450 | UnsupportedAndroidWebViewVersion |
| AADSTS220501 | InvalidCrlDownload |
| AADSTS221000 | DeviceOnlyTokensNotSupportedByResource - 此資源並未設定為接受僅限裝置的權杖。 |
| AADSTS240001 | BulkAADJTokenUnauthorized |
| AADSTS240002 | RequiredClaimIsMissing |
| AADSTS700020 | InteractionRequired |
| AADSTS700022 | InvalidMultipleResroucesScope |
| AADSTS700023 | InvalidResourcelessScope |
| AADSTS900000 | WebWatsonEnvironmentError |
| AADSTS1000000 | UserNotBoundError |
| AADSTS1000002 | BindCompleteInterruptError |

## <a name="next-steps"></a>後續步驟

* 有任何疑問或找不到您要尋找的內容嗎？ 建立 GitHub 問題，或參閱[適用於開發人員的支援和協助選項](active-directory-develop-help-support.md)，了解您可以取得協助和支援的其他方法。