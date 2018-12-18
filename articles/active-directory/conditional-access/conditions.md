---
title: 什麼是 Azure Active Directory 條件式存取中的條件？ | Microsoft Docs
description: 了解如何在 Azure Active Directory 條件式存取中使用條件來觸發原則。
services: active-directory
keywords: 應用程式的條件式存取, Azure AD 條件式存取, 安全存取公司資源, 條件式存取原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 8c1d978f-e80b-420e-853a-8bbddc4bcdad
ms.service: active-directory
ms.component: conditional-access
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/13/2018
ms.author: markvi
ms.reviewer: calebb
ms.openlocfilehash: 9feb6ef5b708813c2f73a70a930cabfd69dff114
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141099"
---
# <a name="what-are-conditions-in-azure-active-directory-conditional-access"></a>什麼是 Azure Active Directory 條件式存取中的條件？ 

您可以使用 [Azure Active Directory (Azure AD) 條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)，控制授權使用者將如何存取您的雲端應用程式。 在條件式存取原則中，您會定義對觸發原則原因 ("When this happens") 做出的回應 ("Then do this")。 

![原因和回應](./media/conditions/10.png)


在條件式存取內容中，我們將**發生此情況時**稱為**條件**。 **則執行此動作**稱為**存取控制**。 條件與存取控制的組合即代表條件式存取原則。

![條件式存取原則](./media/conditions/61.png)


您未在條件式存取原則中設定的條件將不會套用。 某些條件會[強制](best-practices.md)將條件式存取原則套用至環境。 

本文將簡要說明條件及如何在條件式存取原則中使用條件。 

## <a name="users-and-groups"></a>使用者和群組

使用者和群組條件是條件式存取原則中的必要條件。 在原則中，您可以選取 [所有使用者] 或選取特定的使用者或群組。

![使用者和群組](./media/conditions/111.png)

當您選取 [所有使用者] 時，您的原則會套用至目錄內的所有使用者，包括來賓使用者。

當您選取 [使用者和群組] 時，您可以設定下列選項：

* [所有來賓使用者] 會以 B2B 來賓使用者作為原則的目標。 此條件會比對出 **userType** 屬性設為 [來賓] 的任何使用者帳戶。 如果在 Azure AD 的邀請流程中建立帳戶時需立即套用原則，便可使用此設定。

* [目錄角色] 會根據使用者的角色指派來設定原則的目標。 此條件支援目錄角色，例如 [全域管理員] 或 [密碼管理員]。

* [使用者和群組] 會以幾組特定的使用者作為目標。 例如，在選取 HR 應用程式作為雲端應用程式時，您可以選取包含 HR 部門所有成員的群組。 群組可以是 Azure AD 中任何類型的群組，包括動態或已指派的安全性與通訊群組。

您也可以將特定使用者或群組自原則中排除。 其中一個常見的使用案例，是原則強制執行多重要素驗證 (MFA) 時的服務帳戶。 

對於部署新原則來說，以特定的幾組使用者為目標會相當有用。 在新原則中，您應該僅以一組初始使用者為目標來驗證原則行為。 



## <a name="cloud-apps"></a>雲端應用程式 

雲端應用程式是網站或服務。 受 Azure 應用程式 Proxy 保護的網站，也是雲端應用程式。 如需所支援雲端應用程式的詳細描述，請參閱[雲端應用程式指派](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference#cloud-apps-assignments)。 

**雲端應用程式**條件是條件式存取原則中的必要條件。 在原則中，您可以選取 [所有雲端應用程式] 或選取特定的應用程式。

![包含雲端應用程式](./media/conditions/03.png)

選取：

- [所有雲端應用程式]，以針對要套用至整個組織的原則設定基準。 在偵測到任何雲端應用程式有登入風險時，請為需要執行多重要素驗證的原則使用此選項。 套用至**所有雲端應用程式**的原則，也會套用至所有網站和服務的存取。 此設定並非僅限於 [選取應用程式] 清單上顯示的雲端應用程式。 

- 個別雲端應用程式，以依據原則將特定服務當作目標。 例如，您可以要求使用者必須具備[符合規範的裝置](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-mam#app-based-or-compliant-device-policy-for-exchange-online-and-sharepoint-online)，才能存取 SharePoint Online。 當使用者存取 SharePoint 內容時，此原則也會套用至其他服務。 Microsoft Teams 即為一例。 

您可以將特定應用程式從原則中排除。 不過，這些應用程式仍須遵循它們存取的服務所套用的原則。 



## <a name="sign-in-risk"></a>登入風險

登入風險是可能性 (高、中或低) 的指標，代表執行登入嘗試的使用者不是使用者帳戶合法擁有者的可能性。 Azure AD 會計算使用者登入期間的登入風險層級。 您可以使用計算出的登入風險層級作為條件式存取原則中的條件。

![登入風險層級](./media/conditions/22.png)

若要使用此條件，您必須[啟用 Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection-enable)。
 
此條件的常見使用案例，是採取下列保護措施的原則： 

- 封鎖具有高度登入風險的使用者。 此保護措施可防止可能的非合法使用者存取您的雲端應用程式。 
- 針對具有中度登入風險的使用者，要求執行多重要素驗證。 藉由強制執行多重要素驗證，您可以更加確信執行登入的使用者是帳戶的合法擁有者。

如需詳細資訊，請參閱[有風險的登入](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-security-risky-sign-ins)。  

## <a name="device-platforms"></a>裝置平台

裝置平台是以您裝置上執行的作業系統為特徵。 Azure AD 會使用裝置 (例如使用者代理程式) 所提供的資訊來識別平台。 這項資訊未經驗證。 我們建議，所有平台均應套用原則。 原則應封鎖存取、要求須符合 Microsoft Intune 原則，或要求裝置必須已加入網域。 依預設會將原則套用至所有裝置平台。 


![設定裝置平台](./media/conditions/24.png)

如需支援的裝置平台清單，請參閱[裝置平台條件](technical-reference.md#device-platform-condition)。


此條件的其中一個常見使用案例，就是限制只有[受控的裝置](require-managed-devices.md)才能存取您雲端應用程式的原則。 如需更多案例 (包括裝置平台條件)，請參閱 [Azure Active Directory 應用程式型條件式存取](app-based-conditional-access.md)。



## <a name="device-state"></a>裝置狀態

裝置狀態條件會將已加入混合式 Azure AD 的裝置和標示為符合規範的裝置，從條件式存取原則中排除。 當原則只應套用至未受控裝置，以提供多一層工作階段安全性時，此條件將有其效用。 例如裝置未受控時，可以只強制執行 Microsoft Cloud App Security 工作階段控制。 


![設定裝置狀態](./media/conditions/112.png)

如果您想要封鎖未受控裝置的存取，請實作[裝置型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access#app-based-or-compliant-device-policy-for-exchange-online-and-sharepoint-online)。


## <a name="locations"></a>位置

使用位置，可讓您根據嘗試連線的位置定義相關條件。 

![設定位置](./media/conditions/25.png)

此條件的常見使用案例，是採取下列保護措施的原則：

- 針對在公司網路以外存取服務的使用者，要求執行多重要素驗證。  

- 針對從特定國家或地區存取服務的使用者，封鎖存取權。 

如需詳細資訊，請參閱[什麼是 Azure Active Directory 條件式存取中的位置條件？](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-locations)。


## <a name="client-apps"></a>用戶端應用程式

使用用戶端應用程式條件，可讓您將原則套用至不同類型的應用程式。 範例包括網站、服務、行動應用程式和桌面應用程式。 



應用程式會進行如下的分類：

- 網站或服務 - 如果應用程式針對機密用戶端使用 Web SSO 通訊協定、SAML、WS-Fed 或 OpenID Connect。

- 行動應用程式或桌面應用程式 - 如果應用程式針對原生用戶端使用行動應用程式 OpenID Connect。

如需可在條件式存取原則中使用的用戶端應用程式清單，請參閱「Azure Active Directory 條件式存取的技術參考」中的[用戶端應用程式條件](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference#client-apps-condition)。

此條件的常見使用案例，是採取下列保護措施的原則： 

- 針對將大量資料下載到裝置的行動應用程式或桌面應用程式，要求使用[符合規範的裝置](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access#app-based-or-compliant-device-policy-for-exchange-online-and-sharepoint-online)。 在此同時，仍允許從任何裝置以瀏覽器進行存取。

- 封鎖來自任何 Web 應用程式的存取，但允許來自行動應用程式和桌面應用程式的存取。

您可以將此條件套用至 Web SSO 和新式驗證通訊協定。 您也可以將其套用至使用 Microsoft Exchange ActiveSync 的郵件應用程式。 範例包括大部分智慧型手機上的原生郵件應用程式。 

只有在 Microsoft Office 365 Exchange Online 是您唯一選取的雲端應用程式時，才可以選取用戶端應用程式條件。

![雲端應用程式](./media/conditions/32.png)

只有在原則中並未設定其他條件時，才支援選取 **Exchange ActiveSync** 作為用戶端應用程式條件。 不過，您可以將此條件的範圍縮小到僅套用至支援的平台。

 
![僅將原則套用至支援的平台](./media/conditions/33.png)

只將此條件套用到支援的平台，相當於套用到[裝置平台條件](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-mam#app-based-or-compliant-device-policy-for-exchange-online-and-sharepoint-online)中的所有裝置平台。

![設定裝置平台](./media/conditions/34.png)


 如需詳細資訊，請參閱這些文章：

- [設定 SharePoint Online 和 Exchange Online，以便採用 Azure Active Directory 條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-no-modern-authentication)。
 
- [Azure Active Directory 應用程式型條件式存取](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access)。 


### <a name="legacy-authentication"></a>舊版驗證  

條件式存取現在適用於不支援新式驗證的舊版 Microsoft Office 用戶端。 它也適用於使用 POP、IMAP、SMTP 等郵件通訊協定的用戶端。 使用傳統驗證，可讓您設定**封鎖其他用戶端的存取**之類的原則。


![設定用戶端應用程式](./media/conditions/160.png)  


#### <a name="known-issues"></a>已知問題

- 為**其他用戶端**設定原則會讓整個組織封鎖特定用戶端，例如 SPConnect。 之所以執行此封鎖，是因為舊版用戶端以非預期的方式進行驗證。 主要的 Office 應用程式 (例如，較舊的 Office 用戶端) 不會有這個問題。 

- 原則最慢可能需要 24 小時才會生效。 


#### <a name="frequently-asked-questions"></a>常見問題集

**問：** 此驗證是否會封鎖 Microsoft Exchange Web 服務？

這取決於 Exchange Web 服務所使用的驗證通訊協定。 如果 Exchange Web 服務應用程式使用新式驗證，則會在**行動應用程式和桌面用戶端**用戶端應用程式的涵蓋範圍內。 基本驗證則會在**其他用戶端**用戶端應用程式的涵蓋範圍內。


**問：** 我可以對**其他用戶端**使用哪些控制項？

您可以對**其他用戶端**設定任何控制項。 不過，所有案例的使用者體驗都會是封鎖存取。 **其他用戶端**不支援 MFA、符合規範的裝置、網域加入等控制項。 
 
**問：** 我可以對**其他用戶端**使用哪些條件？

您可以對**其他用戶端**設定任何條件。

**問：** Exchange ActiveSync 是否支援所有條件和控制項？

否。 下列清單摘錄了 Exchange ActiveSync 支援： 

- Exchange ActiveSync 僅支援以使用者和群組為目標。 它不支援來賓或角色。 如果使用了來賓或角色條件，則會封鎖所有使用者。 Exchange ActiveSync 會封鎖所有使用者，因為它無法判斷原則是否應套用至該使用者。

- Exchange ActiveSync 僅適用於作為雲端應用程式的 Microsoft Exchange Online。 

- Exchange ActiveSync 不支援用戶端應用程式本身以外的任何條件。 

- Exchange ActiveSync 可以設定任何控制項。 裝置合規性以外的所有控制項都會導致封鎖。

**問：** 原則之後是否會預設為適用於所有用戶端應用程式？

否。 預設原則行為不會有任何變更。 原則預設會繼續適用於瀏覽器和行動應用程式和桌面用戶端。



## <a name="next-steps"></a>後續步驟

- 若想了解如何設定條件式存取原則，請參閱[快速入門：利用 Azure Active Directory 條件式存取來要求特定應用程式需使用 MFA](app-based-mfa.md)。

- 若要為您的環境設定條件式存取原則，請參閱 [Azure Active Directory 中條件式存取的最佳做法](best-practices.md)。 

