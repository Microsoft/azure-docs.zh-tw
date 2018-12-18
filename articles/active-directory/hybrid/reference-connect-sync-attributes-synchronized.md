---
title: Azure AD Connect 所同步處理的屬性 | Microsoft Docs
description: 列出要同步處理至 Azure Active Directory 的屬性。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: c2bb36e0-5205-454c-b9b6-f4990bcedf51
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/05/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 9ea901d51403096166b367a1586cb26d2d914e2d
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306527"
---
# <a name="azure-ad-connect-sync-attributes-synchronized-to-azure-active-directory"></a>Azure AD Connect 同步處理：將屬性同步處理至 Azure Active Directory
本主題列出 Azure AD Connect 同步處理所同步處理的屬性。  
屬性會依相關的 Azure AD App 來分組。

## <a name="attributes-to-synchronize"></a>要同步處理的屬性
常見的問題是同步處理的最少屬性清單為何 。 建議的預設方法是保留預設屬性，這樣可以在雲端建構完整的 GAL (全域通訊清單)，並取得 Office 365 工作負載的所有功能。 在某些情況下，貴組織會有某些不想同步處理至雲端的屬性，因為這些屬性包含機密或 PII (個人識別資訊) 資料，如以下範例所示：  
![不正確的屬性](./media/reference-connect-sync-attributes-synchronized/badextensionattribute.png)

在此情況下，請從本主題中的屬性清單著手，然後識別出包含機密或 PII 資料而不能同步處理的屬性。 接著，在安裝期間使用 [Azure AD 應用程式和屬性篩選](how-to-connect-install-custom.md#azure-ad-app-and-attribute-filtering)將這些屬性取消選取。

> [!WARNING]
> 將屬性取消選取時，您應該小心，只將絕對不可能進行同步處理的屬性取消選取。 取消選取其他屬性可能對功能造成負面的影響。
>
>

## <a name="office-365-proplus"></a>Office 365 ProPlus
| 屬性名稱 | 使用者 | 註解 |
| --- |:---:| --- |
| accountEnabled |X |定義是否啟用帳戶。 |
| cn |X | |
| displayName |X | |
| objectSID |X |機械屬性。 AD 使用者識別碼，可用來維持 Azure AD 和 AD 之間的同步處理。 |
| pwdLastSet |X |機械屬性。 用來得知何時要讓已經發行的權杖失效。 供密碼雜湊同步處理、傳遞驗證和同盟使用。 |
| sourceAnchor |X |機械屬性。 不可變的識別碼，可維持 ADDS 與 Azure AD 之間的關聯性。 |
| usageLocation |X |機械屬性。 使用者的國家/地區。 用於授權指派。 |
| userPrincipalName |X |UPN 是使用者的登入識別碼。 最常與 [mail] 值相同。 |

## <a name="exchange-online"></a>Exchange Online
| 屬性名稱 | 使用者 | 連絡人 | 群組 | 註解 |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |定義是否啟用帳戶。 |
| assistant |X |X | | |
| altRecipient |X | | |需要 Azure AD Connect 1.1.552.0 組建版本或更新版本。 |
| authOrig |X |X |X | |
| c |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| company |X |X | | |
| countryCode |X |X | | |
| department |X |X | | |
| 說明 |X |X |X | |
| displayName |X |X |X | |
| dLMemRejectPerms |X |X |X | |
| dLMemSubmitPerms |X |X |X | |
| extensionAttribute1 |X |X |X | |
| extensionAttribute10 |X |X |X | |
| extensionAttribute11 |X |X |X | |
| extensionAttribute12 |X |X |X | |
| extensionAttribute13 |X |X |X | |
| extensionAttribute14 |X |X |X | |
| extensionAttribute15 |X |X |X | |
| extensionAttribute2 |X |X |X | |
| extensionAttribute3 |X |X |X | |
| extensionAttribute4 |X |X |X | |
| extensionAttribute5 |X |X |X | |
| extensionAttribute6 |X |X |X | |
| extensionAttribute7 |X |X |X | |
| extensionAttribute8 |X |X |X | |
| extensionAttribute9 |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| homePhone |X |X | | |
| info |X |X |X |群組目前未使用這個屬性。 |
| Initials |X |X | | |
| l |X |X | | |
| legacyExchangeDN |X |X |X | |
| mailNickname |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| member | | |X | |
| mobile |X |X | | |
| msDS-HABSeniorityIndex |X |X |X | |
| msDS-PhoneticDisplayName |X |X |X | |
| msExchArchiveGUID |X | | | |
| msExchArchiveName |X | | | |
| msExchAssistantName |X |X | | |
| msExchAuditAdmin |X | | | |
| msExchAuditDelegate |X | | | |
| msExchAuditDelegateAdmin |X | | | |
| msExchAuditOwner |X | | | |
| msExchBlockedSendersHash |X |X | | |
| msExchBypassAudit |X | | | |
| msExchBypassModerationLink | | |X |可用於 Azure AD Connect 1.1.524.0 版中 |
| msExchCoManagedByLink | | |X | |
| msExchDelegateListLink |X | | | |
| msExchELCExpirySuspensionEnd |X | | | |
| msExchELCExpirySuspensionStart |X | | | |
| msExchELCMailboxFlags |X | | | |
| msExchEnableModeration |X | |X | |
| msExchExtensionCustomAttribute1 |X |X |X |Exchange Online 目前未使用這個屬性。 |
| msExchExtensionCustomAttribute2 |X |X |X |Exchange Online 目前未使用這個屬性。 |
| msExchExtensionCustomAttribute3 |X |X |X |Exchange Online 目前未使用這個屬性。 |
| msExchExtensionCustomAttribute4 |X |X |X |Exchange Online 目前未使用這個屬性。 |
| msExchExtensionCustomAttribute5 |X |X |X |Exchange Online 目前未使用這個屬性。 |
| msExchHideFromAddressLists |X |X |X | |
| msExchImmutableID |X | | | |
| msExchLitigationHoldDate |X |X |X | |
| msExchLitigationHoldOwner |X |X |X | |
| msExchMailboxAuditEnable |X | | | |
| msExchMailboxAuditLogAgeLimit |X | | | |
| msExchMailboxGuid |X | | | |
| msExchModeratedByLink |X |X |X | |
| msExchModerationFlags |X |X |X | |
| msExchRecipientDisplayType |X |X |X | |
| msExchRecipientTypeDetails |X |X |X | |
| msExchRemoteRecipientType |X | | | |
| msExchRequireAuthToSendTo |X |X |X | |
| msExchResourceCapacity |X | | | |
| msExchResourceDisplay |X | | | |
| msExchResourceMetaData |X | | | |
| msExchResourceSearchProperties |X | | | |
| msExchRetentionComment |X |X |X | |
| msExchRetentionURL |X |X |X | |
| msExchSafeRecipientsHash |X |X | | |
| msExchSafeSendersHash |X |X | | |
| msExchSenderHintTranslations |X |X |X | |
| msExchTeamMailboxExpiration |X | | | |
| msExchTeamMailboxOwners |X | | | |
| msExchTeamMailboxSharePointUrl |X | | | |
| msExchUserHoldPolicies |X | | | |
| msOrg-IsOrganizational | | |X | |
| objectSID |X | |X |機械屬性。 AD 使用者識別碼，可用來維持 Azure AD 和 AD 之間的同步處理。 |
| oOFReplyToOriginator | | |X | |
| otherFacsimileTelephone |X |X | | |
| otherHomePhone |X |X | | |
| otherTelephone |X |X | | |
| pager |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| proxyAddresses |X |X |X | |
| publicDelegates |X |X |X | |
| pwdLastSet |X | | |機械屬性。 用來得知何時要讓已經發行的權杖失效。 供密碼同步處理和同盟使用。 |
| reportToOriginator | | |X | |
| reportToOwner | | |X | |
| securityEnabled | | |X |衍生自 groupType |
| sn |X |X | | |
| sourceAnchor |X |X |X |機械屬性。 不可變的識別碼，可維持 ADDS 與 Azure AD 之間的關聯性。 |
| st |X |X | | |
| streetAddress |X |X | | |
| targetAddress |X |X | | |
| telephoneAssistant |X |X | | |
| telephoneNumber |X |X | | |
| thumbnailphoto |X |X | | |
| title |X |X | | |
| unauthOrig |X |X |X | |
| usageLocation |X | | |機械屬性。 使用者的國家/地區。 用於授權指派。 |
| userCertificate |X |X | | |
| userPrincipalName |X | | |UPN 是使用者的登入識別碼。 最常與 [mail] 值相同。 |
| userSMIMECertificates |X |X | | |
| wWWHomePage |X |X | | |

## <a name="sharepoint-online"></a>SharePoint Online
| 屬性名稱 | 使用者 | 連絡人 | 群組 | 註解 |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |定義是否啟用帳戶。 |
| authOrig |X |X |X | |
| c |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| company |X |X | | |
| countryCode |X |X | | |
| department |X |X | | |
| 說明 |X |X |X | |
| displayName |X |X |X | |
| dLMemRejectPerms |X |X |X | |
| dLMemSubmitPerms |X |X |X | |
| extensionAttribute1 |X |X |X | |
| extensionAttribute10 |X |X |X | |
| extensionAttribute11 |X |X |X | |
| extensionAttribute12 |X |X |X | |
| extensionAttribute13 |X |X |X | |
| extensionAttribute14 |X |X |X | |
| extensionAttribute15 |X |X |X | |
| extensionAttribute2 |X |X |X | |
| extensionAttribute3 |X |X |X | |
| extensionAttribute4 |X |X |X | |
| extensionAttribute5 |X |X |X | |
| extensionAttribute6 |X |X |X | |
| extensionAttribute7 |X |X |X | |
| extensionAttribute8 |X |X |X | |
| extensionAttribute9 |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| hideDLMembership | | |X | |
| homePhone |X |X | | |
| info |X |X |X | |
| Initials |X |X | | |
| ipPhone |X |X | | |
| l |X |X | | |
| mail |X |X |X | |
| mailNickname |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| member | | |X | |
| middleName |X |X | | |
| mobile |X |X | | |
| msExchTeamMailboxExpiration |X | | | |
| msExchTeamMailboxOwners |X | | | |
| msExchTeamMailboxSharePointLinkedBy |X | | | |
| msExchTeamMailboxSharePointUrl |X | | | |
| objectSID |X | |X |機械屬性。 AD 使用者識別碼，可用來維持 Azure AD 和 AD 之間的同步處理。 |
| oOFReplyToOriginator | | |X | |
| otherFacsimileTelephone |X |X | | |
| otherHomePhone |X |X | | |
| otherIpPhone |X |X | | |
| otherMobile |X |X | | |
| otherPager |X |X | | |
| otherTelephone |X |X | | |
| pager |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| postOfficeBox |X |X | |SharePoint Online 目前未使用這個屬性。 |
| preferredLanguage |X | | | |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |機械屬性。 用來得知何時要讓已經發行的權杖失效。 供密碼雜湊同步處理、傳遞驗證和同盟使用。 |
| reportToOriginator | | |X | |
| reportToOwner | | |X | |
| securityEnabled | | |X |衍生自 groupType |
| sn |X |X | | |
| sourceAnchor |X |X |X |機械屬性。 不可變的識別碼，可維持 ADDS 與 Azure AD 之間的關聯性。 |
| st |X |X | | |
| streetAddress |X |X | | |
| targetAddress |X |X | | |
| telephoneAssistant |X |X | | |
| telephoneNumber |X |X | | |
| thumbnailphoto |X |X | | |
| title |X |X | | |
| unauthOrig |X |X |X | |
| url |X |X | | |
| usageLocation |X | | |機械屬性。 使用者的國家/地區。 用於授權指派。 |
| userPrincipalName |X | | |UPN 是使用者的登入識別碼。 最常與 [mail] 值相同。 |
| wWWHomePage |X |X | | |

## <a name="lync-online-subsequently-known-as-skype-for-business"></a>Lync Online (即後來的商務用 Skype)
| 屬性名稱 | 使用者 | 連絡人 | 群組 | 註解 |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |定義是否啟用帳戶。 |
| c |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| company |X |X | | |
| department |X |X | | |
| 說明 |X |X |X | |
| displayName |X |X |X | |
| facsimiletelephonenumber |X |X |X | |
| givenName |X |X | | |
| homePhone |X |X | | |
| ipPhone |X |X | | |
| l |X |X | | |
| mail |X |X |X | |
| mailNickname |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| member | | |X | |
| mobile |X |X | | |
| msExchHideFromAddressLists |X |X |X | |
| msRTCSIP-ApplicationOptions |X | | | |
| msRTCSIP-DeploymentLocator |X |X | | |
| msRTCSIP-Line |X |X | | |
| msRTCSIP-OptionFlags |X |X | | |
| msRTCSIP-OwnerUrn |X | | | |
| msRTCSIP-PrimaryUserAddress |X |X | | |
| msRTCSIP-UserEnabled |X |X | | |
| objectSID |X | |X |機械屬性。 AD 使用者識別碼，可用來維持 Azure AD 和 AD 之間的同步處理。 |
| otherTelephone |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| preferredLanguage |X | | | |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |機械屬性。 用來得知何時要讓已經發行的權杖失效。 供密碼雜湊同步處理、傳遞驗證和同盟使用。 |
| securityEnabled | | |X |衍生自 groupType |
| sn |X |X | | |
| sourceAnchor |X |X |X |機械屬性。 不可變的識別碼，可維持 ADDS 與 Azure AD 之間的關聯性。 |
| st |X |X | | |
| streetAddress |X |X | | |
| telephoneNumber |X |X | | |
| thumbnailphoto |X |X | | |
| title |X |X | | |
| usageLocation |X | | |機械屬性。 使用者的國家/地區。 用於授權指派。 |
| userPrincipalName |X | | |UPN 是使用者的登入識別碼。 最常與 [mail] 值相同。 |
| wWWHomePage |X |X | | |

## <a name="azure-rms"></a>Azure RMS
| 屬性名稱 | 使用者 | 連絡人 | 群組 | 註解 |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |定義是否啟用帳戶。 |
| cn |X | |X |一般名稱或別名。 最常見的前置詞是 [mail] 值。 |
| displayName |X |X |X |字串，表示通常會顯示為易記名稱 (名字姓氏) 的名稱。 |
| mail |X |X |X |電子郵件地址。 |
| member | | |X | |
| objectSID |X | |X |機械屬性。 AD 使用者識別碼，可用來維持 Azure AD 和 AD 之間的同步處理。 |
| proxyAddresses |X |X |X |機械屬性。 由 Azure AD 所使用。 包含使用者的所有次要電子郵件地址。 |
| pwdLastSet |X | | |機械屬性。 用來得知何時要讓已經發行的權杖失效。 |
| securityEnabled | | |X |衍生自 groupType。 |
| sourceAnchor |X |X |X |機械屬性。 不可變的識別碼，可維持 ADDS 與 Azure AD 之間的關聯性。 |
| usageLocation |X | | |機械屬性。 使用者的國家/地區。 用於授權指派。 |
| userPrincipalName |X | | |這個 UPN 是使用者的登入識別碼。 最常與 [mail] 值相同。 |

## <a name="intune"></a>Intune
| 屬性名稱 | 使用者 | 連絡人 | 群組 | 註解 |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |定義是否啟用帳戶。 |
| c |X |X | | |
| cn |X | |X | |
| 說明 |X |X |X | |
| displayName |X |X |X | |
| mail |X |X |X | |
| mailNickname |X |X |X | |
| member | | |X | |
| objectSID |X | |X |機械屬性。 AD 使用者識別碼，可用來維持 Azure AD 和 AD 之間的同步處理。 |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |機械屬性。 用來得知何時要讓已經發行的權杖失效。 供密碼雜湊同步處理、傳遞驗證和同盟使用。 |
| securityEnabled | | |X |衍生自 groupType |
| sourceAnchor |X |X |X |機械屬性。 不可變的識別碼，可維持 ADDS 與 Azure AD 之間的關聯性。 |
| usageLocation |X | | |機械屬性。 使用者的國家/地區。 用於授權指派。 |
| userPrincipalName |X | | |UPN 是使用者的登入識別碼。 最常與 [mail] 值相同。 |

## <a name="dynamics-crm"></a>Dynamics CRM
| 屬性名稱 | 使用者 | 連絡人 | 群組 | 註解 |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |定義是否啟用帳戶。 |
| c |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| company |X |X | | |
| countryCode |X |X | | |
| 說明 |X |X |X | |
| displayName |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| l |X |X | | |
| managedBy | | |X | |
| manager |X |X | | |
| member | | |X | |
| mobile |X |X | | |
| objectSID |X | |X |機械屬性。 AD 使用者識別碼，可用來維持 Azure AD 和 AD 之間的同步處理。 |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| preferredLanguage |X | | | |
| pwdLastSet |X | | |機械屬性。 用來得知何時要讓已經發行的權杖失效。 供密碼雜湊同步處理、傳遞驗證和同盟使用。 |
| securityEnabled | | |X |衍生自 groupType |
| sn |X |X | | |
| sourceAnchor |X |X |X |機械屬性。 不可變的識別碼，可維持 ADDS 與 Azure AD 之間的關聯性。 |
| st |X |X | | |
| streetAddress |X |X | | |
| telephoneNumber |X |X | | |
| title |X |X | | |
| usageLocation |X | | |機械屬性。 使用者的國家/地區。 用於授權指派。 |
| userPrincipalName |X | | |UPN 是使用者的登入識別碼。 最常與 [mail] 值相同。 |

## <a name="3rd-party-applications"></a>協力廠商應用程式
此群組是一組屬性，用來作為一般工作負載或應用程式所需的最基本屬性。 它可以用於另一節中未列出的工作負載或用於非 Microsoft 應用程式。 它會明確用於下列︰

* Yammer (使用的只有 User)
* [SharePoint 等資源所提供的混合式企業對企業 (B2B) 跨組織共同作業案例](http://go.microsoft.com/fwlink/?LinkId=747036)

此群組是一組屬性，是未使用 Azure AD 目錄來支援 Office 365、Dynamics 或 Intune 時所能使用的屬性。 它包含一小組的核心屬性。

| 屬性名稱 | 使用者 | 連絡人 | 群組 | 註解 |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |定義是否啟用帳戶。 |
| cn |X | |X | |
| displayName |X |X |X | |
| givenName |X |X | | |
| mail |X | |X | |
| managedBy | | |X | |
| mailNickname |X |X |X | |
| member | | |X | |
| objectSID |X | | |機械屬性。 AD 使用者識別碼，可用來維持 Azure AD 和 AD 之間的同步處理。 |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |機械屬性。 用來得知何時要讓已經發行的權杖失效。 供密碼雜湊同步處理、傳遞驗證和同盟使用。 |
| sn |X |X | | |
| sourceAnchor |X |X |X |機械屬性。 不可變的識別碼，可維持 ADDS 與 Azure AD 之間的關聯性。 |
| usageLocation |X | | |機械屬性。 使用者的國家/地區。 用於授權指派。 |
| userPrincipalName |X | | |UPN 是使用者的登入識別碼。 最常與 [mail] 值相同。 |

## <a name="windows-10"></a>Windows 10
已加入網域的 Windows 10 電腦 (裝置) 會將某些屬性同步處理至 Azure AD。 如需這些案例的詳細資訊，請參閱 [將已加入網域的裝置連接到 Azure AD 以體驗 Windows 10](../active-directory-azureadjoin-devices-group-policy.md)。 這些屬性一律會進行同步處理，而且 Windows 10 不會顯示為您可以取消選取的應用程式。 Windows 10 已加入網域的電腦是透過填入屬性 userCertificate 來識別。

| 屬性名稱 | 裝置 | 註解 |
| --- |:---:| --- |
| accountEnabled |X | |
| deviceTrustType |X |已加入網域的電腦的硬式編碼值。 |
| displayName |X | |
| ms-DS-CreatorSID |X |也稱為 registeredOwnerReference。 |
| objectGUID |X |也稱為 deviceID。 |
| objectSID |X |也稱為 onPremisesSecurityIdentifier。 |
| operatingSystem |X |也稱為 deviceOSType。 |
| operatingSystemVersion |X |也稱為 deviceOSVersion。 |
| userCertificate |X | |

下列「使用者」  屬性是您已選取的其他應用程式以外的屬性。  

| 屬性名稱 | 使用者 | 註解 |
| --- |:---:| --- |
| domainFQDN |X |也稱為 dnsDomainName。 例如 contoso.com。 |
| domainNetBios |X |也稱為 netBiosName。 例如 CONTOSO。 |
| msDS-KeyCredentialLink |X |一旦使用者在 Windows Hello 企業版中註冊後。 | 

## <a name="exchange-hybrid-writeback"></a>Exchange 混合回寫
當您選擇啟用「Exchange 混合」 時，系統會將這些屬性從 Azure AD 寫回到內部部署 Active Directory。 根據您的 Exchange 版本有可能會同步處理較少的屬性。

| 屬性名稱 (Connect UI) |屬性名稱 (內部部署 AD) | 使用者 | 連絡人 | 群組 | 註解 |
| --- |:---:|:---:|:---:| --- |---|
| msDS-ExternalDirectoryObjectID| ms-DS-External-Directory-Object-Id |X | | |衍生自 Azure AD 中的 cloudAnchor。 這個屬性是 Exchange 2016 和 Windows Server 2016 AD 中的新屬性。 |
| msExchArchiveStatus| ms-Exch-ArchiveStatus |X | | |線上封存：可讓客戶封存郵件。 |
| msExchBlockedSendersHash| ms-Exch-BlockedSendersHash |X | | |篩選：從用戶端回寫內部部署篩選及線上安全和已封鎖的寄件者資料。 |
| msExchSafeRecipientsHash| ms-Exch-SafeRecipientsHash  |X | | |篩選：從用戶端回寫內部部署篩選及線上安全和已封鎖的寄件者資料。 |
| msExchSafeSendersHash| ms-Exch-SafeSendersHash  |X | | |篩選：從用戶端回寫內部部署篩選及線上安全和已封鎖的寄件者資料。 |
| msExchUCVoiceMailSettings| ms-Exch-UCVoiceMailSettings |X | | |啟用整合通訊 (UM) - 線上語音信箱：供 Microsoft Lync Server 整合用來向 Lync Server 內部部署表示使用者在線上服務中有語音信箱。 |
| msExchUserHoldPolicies| ms-Exc-hUserHoldPolicies |X | | |訴訟資料暫留：啟用雲端服務來識別哪些使用者正處於訴訟資料暫留狀態。 |
| proxyAddresses| proxyAddresses |X |X |X |只會插入 Exchange Online 的 x500 位址。 |
| publicDelegates| ms-Exch-Public-Delegates  |X | | |將 Exchange Online 信箱的 SendOnBehalfTo 權限授與使用內部部署 Exchange 信箱的使用者。 需要 Azure AD Connect 1.1.552.0 組建版本或更新版本。 |

## <a name="exchange-mail-public-folder"></a>Exchange 郵件公用資料夾
當您選擇啟用 [Exchange 郵件公用資料夾] 時，系統會將內部部署 Active Directory 的這些屬性同步到 Azure AD。

| 屬性名稱 | 公用資料夾 | 註解 |
| --- | :---:| --- |
| displayName | X |  |
| mail | X |  |
| msExchRecipientTypeDetails | X |  |
| objectGUID | X |  |
| proxyAddresses | X |  |
| targetAddress | X |  |

## <a name="device-writeback"></a>裝置回寫
裝置物件是在 Active Directory 中建立。 這些物件可以是加入 Azure AD 的裝置，或加入網域的 Windows 10 電腦。

| 屬性名稱 | 裝置 | 註解 |
| --- |:---:| --- |
| altSecurityIdentities |X | |
| displayName |X | |
| dn |X | |
| msDS-CloudAnchor |X | |
| msDS-DeviceID |X | |
| msDS-DeviceObjectVersion |X | |
| msDS-DeviceOSType |X | |
| msDS-DeviceOSVersion |X | |
| msDS-DevicePhysicalIDs |X | |
| msDS-KeyCredentialLink |X |只在 Windows Server 2016 AD 結構描述才有 |
| msDS-IsCompliant |X | |
| msDS-IsEnabled |X | |
| msDS-IsManaged |X | |
| msDS-RegisteredOwner |X | |

## <a name="notes"></a>注意
* 使用「替代識別碼」時，內部部署屬性 userPrincipalName 會與 Azure AD 屬性 onPremisesUserPrincipalName 進行同步處理。 「替代識別碼」屬性 (例如 mail) 會與 Azure AD 屬性 userPrincipalName 進行同步處理。
* 在上面的清單中，物件類型 **User** 也適用於物件類型 **iNetOrgPerson**。

## <a name="next-steps"></a>後續步驟
深入了解 [Azure AD Connect 同步](how-to-connect-sync-whatis.md) 組態。

深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
