---
title: Azure Active Directory Domain Services：受控網域中的同步處理 | Microsoft Docs
description: 了解 Azure Active Directory Domain Services 受控網域中的同步處理
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 57cbf436-fc1d-4bab-b991-7d25b6e987ef
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/30/2018
ms.author: maheshu
ms.openlocfilehash: 87ee4f2cbdd09a9284668ea298c0b3c3b1439c15
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503010"
---
# <a name="synchronization-in-an-azure-ad-domain-services-managed-domain"></a>Azure AD Domain Services 受控網域中的同步處理
下圖說明同步處理在 Azure AD Domain Services 受控網域中的運作方式。

![Azure AD Domain Services 中的同步處理](./media/active-directory-domain-services-design-guide/sync-topology.png)

## <a name="synchronization-from-your-on-premises-directory-to-your-azure-ad-tenant"></a>從內部部署目錄同步到 Azure AD 租用戶
使用 Azure AD Connect 同步可將使用者帳戶、群組成員資格及認證雜湊同步到 Azure AD 租用戶。 同步的對象包括使用者帳戶的屬性，例如 UPN 和內部部署 SID (安全性識別碼)。 如果您使用 Azure AD Domain Services，則 NTLM 和 Kerberos 驗證所需的認證雜湊也會同步到 Azure AD 租用戶。

如果您設定回寫，則在 Azure AD 目錄中發生的變更將會往回同步到您的內部部署 Active Directory。 例如，如果您使用 Azure AD 自助式密碼管理來變更密碼，系統就會在內部部署 AD 網域中更新變更過的密碼。

> [!NOTE]
> 請一律使用最新版的 Azure AD Connect 版本，以確保您已具備所有已知問題的修正。
>
>

## <a name="synchronization-from-your-azure-ad-tenant-to-your-managed-domain"></a>從 Azure AD 租用戶同步到受控網域
使用者帳戶、群組成員資格及認證雜湊會從 Azure AD 租用戶同步到 Azure AD Domain Services 受控網域。 此同步處理程序是自動執行的。 您不需要設定、監視或管理此同步處理程序。 根據您 Azure AD 目錄中的物件數目而定，首次同步處理可能需要花費數小時到數天的時間。 首次同步處理完成之後，在受控網域中更新 Azure AD 中所做的變更大約需要 20-30 分鐘。 此同步處理間隔適用於密碼變更或在 Azure AD 中所做的屬性變更。

此同步處理程序在本質上也是單向的。 除了您建立的所有自訂 OU 之外，您的受控網域大部分都是唯獨的。 因此，您無法對受控網域內的使用者屬性、使用者密碼或群組成員資格進行變更。 所以，並不會有將變更從受控網域反向同步到 Azure AD 租用戶的情況。

## <a name="synchronization-from-a-multi-forest-on-premises-environment"></a>從多樹系內部部署環境同步處理
許多組織有由多個帳戶樹系組成、相當複雜的內部部署身分識別基礎結構。 Azure AD Connect 支援將使用者、群組及認證雜湊從多樹系環境同步到 Azure AD 租用戶。

相對地，Azure AD 租用戶是一個簡單許多且單層式的命名空間。 若要讓使用者可靠地存取受 Azure AD 保護的應用程式，請解決不同樹系中各個使用者帳戶之間的 UPN 衝突。 Azure AD Domain Services 受控網域與 Azure AD 租用戶非常相似。 您會在受控網域中看到單層式 OU 結構。 儘管是從不同的內部部署網域或樹系進行同步處理，所有使用者帳戶和群組還是都會儲存於「AADDC 使用者」容器內。 您可能已在內部部署環境設定一個階層式 OU 結構。 您的受控網域仍然具有簡單的單層式 OU 結構。

## <a name="exclusions---what-isnt-synchronized-to-your-managed-domain"></a>排除項目 - 不會同步到受控網域的項目
下列物件或屬性不會同步到您的 Azure AD 租用戶或受控網域：

* **排除的屬性**：您可以使用 Azure AD Connect 來選擇將特定屬性排除，而不要將它們從內部部署網域同步到 Azure AD 租用戶。 這些排除的屬性將無法在您的受控網域中提供使用。
* **群組原則：** 在您內部部署網域中設定的「群組原則」不會同步到您的受控網域。
* **Sysvol 共用：** 同樣地，您內部部署網域上的 Sysvol 共用內容也不會同步到您的受控網域。
* **電腦物件：** 已加入您內部部署網域之電腦的電腦物件不會同步到您的受控網域。 這些電腦與您的受控網域之間並沒有信任關係，而只屬於您的內部部署網域。 在您的受控網域中，只有針對您已明確加入受控網域之網域的電腦，您才能找到其電腦物件。
* **使用者和群組的 SidHistory 屬性：** 來自您內部部署網域的主要使用者和主要群組 SID 會同步到您的受控網域。 不過，使用者和群組的現有 SidHistory 屬性不會從您的內部部署網域同步到您的受控網域。
* **組織單位 (OU) 結構：** 在您內部部署網域中定義的「組織單位」不會同步到您的受控網域。 您的受控網域中有兩個內建的 OU。 根據預設，您的受控網域會有單層式 OU 結構。 不過，您可以選擇[在您的受控網域中建立自訂 OU](active-directory-ds-admin-guide-create-ou.md)。

## <a name="how-specific-attributes-are-synchronized-to-your-managed-domain"></a>如何將特定屬性同步到受控網域
下表列出一些常見的屬性並說明如何將它們同步到受控網域。

| 受控網域中的屬性 | 來源 | 注意 |
|:--- |:--- |:--- |
| UPN |Azure AD 租用戶中的使用者 UPN 屬性 |來自您 Azure AD 租用戶的 UPN 屬性會依原狀同步到您的受控網域。 因此，登入您受控網域的最可靠方法是使用您的 UPN。 |
| SAMAccountName |Azure AD 租用戶中或自動產生的使用者 mailNickname 屬性 |SAMAccountName 屬性是來自您 Azure AD 租用戶中的 mailNickname 屬性。 如果有多個使用者帳戶具有相同的 mailNickname 屬性，系統就會自動產生 SAMAccountName。 如果使用者的 mailNickname 或 UPN 前置詞長度超過 20 個字元，系統就會自動產生 SAMAccountName 來滿足 SAMAccountName 屬性上的 20 個字元限制。 |
| 密碼 |來自 Azure AD 租用戶的使用者密碼 |系統會從您的 Azure AD 租用戶同步 NTLM 或 Kerberos 驗證所需的認證雜湊 (也稱為補充認證)。 如果您的 Azure AD 租用戶是已同步的租用戶，這些認證就會來自您的內部部署網域。 |
| 主要使用者/群組 SID |自動產生 |使用者/群組帳戶的主要 SID 會在您的受控網域中自動產生。 此屬性不會與您內部部署 AD 網域中的主要使用者/群組 SID 相符。 這種不相符的情況是因為受控網域的 SID 命名空間與您的內部部署網域不同。 |
| 使用者和群組的 SID 歷程記錄 |內部部署主要使用者和群組 SID |您的受控網域中使用者和群組的 SidHistory 屬性會設定為與內部部署網域中對應的主要使用者或群組 SID 相符。 此功能可協助以更簡單的方式將內部部署應用程式原封不動地轉移到受控網域，因為您不需要重新設定資源的 ACL。 |

> [!NOTE]
> **使用 UPN 格式來登入受控網域：** 系統可能會針對受控網域中的某些使用者帳戶自動產生 SAMAccountName 屬性。 如果有多個使用者具有相同的 mailNickname 屬性，或使用者的 UPN 前置詞太長，可能就會自動為這些使用者產生 SAMAccountName。 因此，SAMAccountName 格式 (例如 'CONTOSO100\joeuser') 並不一定是登入網域的可靠方法。 使用者的自動產生 SAMAccountName 可能會與其 UPN 前置詞不同。 請使用 UPN 格式 (例如 'joeuser@contoso100.com') 可靠地登入受控網域。
>
>

### <a name="attribute-mapping-for-user-accounts"></a>使用者帳戶的屬性對應
下表說明如何將您 Azure AD 租用戶中使用者物件的特定屬性同步到受控網域中對應的屬性。

| Azure AD 租用戶中的使用者屬性 | 受控網域中的使用者屬性 |
|:--- |:--- |
| accountEnabled |userAccountControl (設定或清除 ACCOUNT_DISABLED 位元) |
| city |l |
| country |co |
| department |department |
| displayName |displayName |
| facsimileTelephoneNumber |facsimileTelephoneNumber |
| givenName |givenName |
| jobTitle |title |
| mail |mail |
| mailNickname |msDS-AzureADMailNickname |
| mailNickname |SAMAccountName (有時可能會自動產生) |
| mobile |mobile |
| objectid |msDS-AzureADObjectId |
| onPremiseSecurityIdentifier |sidHistory |
| passwordPolicies |userAccountControl (設定或清除 DONT_EXPIRE_PASSWORD 位元) |
| physicalDeliveryOfficeName |physicalDeliveryOfficeName |
| postalCode |postalCode |
| preferredLanguage |preferredLanguage |
| state |st |
| streetAddress |streetAddress |
| surname |sn |
| telephoneNumber |telephoneNumber |
| userPrincipalName |userPrincipalName |

### <a name="attribute-mapping-for-groups"></a>群組的屬性對應
下表說明如何將您 Azure AD 租用戶中群組物件的特定屬性同步到受控網域中對應的屬性。

| Azure AD 租用戶中的群組屬性 | 受控網域中的群組屬性 |
|:--- |:--- |
| displayName |displayName |
| displayName |SAMAccountName (有時可能會自動產生) |
| mail |mail |
| mailNickname |msDS-AzureADMailNickname |
| objectid |msDS-AzureADObjectId |
| onPremiseSecurityIdentifier |sidHistory |
| securityEnabled |groupType |

## <a name="password-hash-synchronization-and-security-considerations"></a>密碼雜湊同步處理和安全性考量
當您啟用 Azure AD Domain Services 時，Azure AD 目錄就會以 NTLM 和 Kerberos 相容的格式產生並儲存密碼雜湊。 

針對現有的雲端使用者帳戶，由於 Azure AD 絕對不會儲存他們的純文字密碼，所以無法自動產生這些雜湊。 因此，Microsoft 要求[雲端使用者重設/變更其密碼](active-directory-ds-getting-started-password-sync.md)，以產生其密碼雜湊並儲存於 Azure AD 中。 針對在啟用 Azure AD Domain Services 之後於 Azure AD 中建立的任何雲端使用者帳戶，以 NTLM 和 Kerberos 相容的格式產生並儲存密碼雜湊。 

針對使用 Azure AD Connect 同步從內部部署 AD 同步處理的使用者帳戶，您需要[設定 Azure AD Connect 同步來以 NTLM 和 Kerberos 相容的格式同步處理密碼雜湊](active-directory-ds-getting-started-password-sync-synced-tenant.md)。

NTLM 和 Kerberos 相容的密碼雜湊一律會以加密方式儲存於 Azure AD 中。 這些雜湊均會加密，因此，只有 Azure AD Domain Services 具有解密金鑰的存取權。 Azure AD 中沒有任何其他服務或元件具有解密金鑰的存取權。 加密金鑰對於每個 Azure AD 租用戶都是唯一的。 Azure AD Domain Services 會將密碼雜湊同步處理到適用於您受控網域的網域控制站。 這些密碼雜湊會儲存在這些網域控制站上並受到保護，類似於將密碼儲存在 Windows Server AD 網域控制站上並受到保護。 這些受控網域控制站的磁碟均會進行待用加密。

## <a name="objects-that-are-not-synchronized-to-your-azure-ad-tenant-from-your-managed-domain"></a>不會從受控網域同步到 Azure AD 租用戶的物件
如本文前面某節所述，並不會有從受控網域往回同步到 Azure AD 租用戶的情況。 您可以選擇在您的受控網域中[建立自訂組織單位 (OU)](active-directory-ds-admin-guide-create-ou.md)。 此外，您還可以在這些自訂 OU 內建立其他 OU、使用者、群組或服務帳戶。 在自訂 OU 內建立的所有物件都不會往回同步到您的 Azure AD 租用戶。 這些物件僅供在您的受控網域內使用。 因此，不論是使用 Azure AD PowerShell Cmdlet、Azure AD Graph API，還是使用 Azure AD 管理 UI，都看不到這些物件。

## <a name="related-content"></a>相關內容
* [功能 - Azure AD 網域服務](active-directory-ds-features.md)
* [部署案例 - Azure AD 網域服務](active-directory-ds-scenarios.md)
* [Azure AD Domain Services 的網路考量](active-directory-ds-networking.md)
* [開始使用 Azure AD Domain Services](active-directory-ds-getting-started.md)
