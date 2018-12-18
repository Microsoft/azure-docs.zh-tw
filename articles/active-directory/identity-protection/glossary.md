---
title: Azure Active Directory Identity Protection 詞彙 | Microsoft Docs
description: Azure Active Directory Identity Protection 詞彙
services: active-directory
keywords: azure active directory identity protection, cloud app discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則, 詞彙
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: 833119a5-33d6-4482-adda-fa35218c72c3
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2018
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: 3ea57ac39d48f51c7bbee1d149597c459fc5d547
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005508"
---
# <a name="azure-active-directory-identity-protection-glossary"></a>Azure Active Directory Identity Protection 詞彙
### <a name="at-risk-user"></a>有風險 (使用者)
具有一或多個作用中風險事件的使用者。 

### <a name="atypical-sign-in-location"></a>非典型登入位置
從特定使用者、類似使用者或租用戶不常用的地理位置登入。

### <a name="azure-ad-identity-protection"></a>Azure AD Identity Protection
Azure Active Directory 的安全性模組，可供整合檢視會影響組織身分識別的風險事件和潛在弱點。

### <a name="conditional-access"></a>條件式存取
用來保護資源存取的原則。 條件式存取規則會儲存在 Azure Active Directory 中，並在授與資源的存取權之前由 Azure AD 評估。  範例規則包括根據使用者位置、裝置健康狀態或使用者驗證方法來限制存取。

### <a name="credentials"></a>認證
包含識別碼以及用來取得存取本機和網路資源之識別證明的資訊。 認證範例包括使用者名稱和密碼、智慧卡和憑證。

### <a name="event"></a>Event
Azure Active Directory 中的活動記錄。

### <a name="false-positive-risk-event"></a>誤判 (風險事件)
由 Identity Protection 使用者手動設定的風險事件狀態，表示此風險事件已經過調查而且被誤標為風險事件。

### <a name="identity"></a>身分識別
必須透過以密碼或憑證等準則為基礎的驗證作業來確認的個人或實體。

### <a name="identity-risk-event"></a>身分識別風險事件
由 Identity Protection 標示為異常的 AAD 事件，有可能表示身分識別已被入侵。

### <a name="ignored-risk-event"></a>已忽略 (風險事件)
由 Identity Protection 使用者手動設定的風險事件狀態，表示此風險事件已關閉，而不需採取補救動作。

### <a name="impossible-travel-from-atypical-locations"></a>不可能來自非典型位置的移動
偵測到同一位使用者有兩次登入時所觸發的風險事件，其中至少一次登入是來自非典型登入位置，而且登入之間的時間比在兩地之間實際移動所需的最短時間還要短。  

### <a name="investigation"></a>調查
檢閱風險事件相關活動、記錄檔和其他相關資訊的程序，以決定是否需要採取補救或緩和步驟，了解身分識別是否及如何遭到入侵，以及了解遭到入侵的身分識別如何被利用。

### <a name="leaked-credentials"></a>認證外洩
當我們的研究人員發現目前的使用者認證 (使用者名稱和密碼) 被公開張貼在黑暗網路 (Dark Web) 中時所觸發的風險事件。

### <a name="mitigation"></a>緩和
此動作可限制或消除攻擊者利用遭到入侵的身分識別或裝置的能力，但不需將身分識別或裝置還原至安全的狀態。 緩和並未解決先前與身分識別或裝置相關聯的風險事件。

### <a name="multi-factor-authentication"></a>Multi-Factor Authentication
此種驗證方法需要兩個或更多驗證方法，其中可能包含使用者擁有的事物 (例如憑證)、使用者知道的事物 (例如使用者名稱、密碼或通關密語)、實體屬性 (例如指紋) 以及個人屬性 (例如個人簽章)。

### <a name="offline-detection"></a>離線偵測
針對已經發生的事件，偵測異常狀況及評估事件的風險 (例如事後的登入嘗試)。

### <a name="policy-condition"></a>原則條件
安全性原則的一部分，定義原則中包含或排除的實體 (群組、使用者、應用程式、裝置平台、裝置狀態、IP 範圍、用戶端類型)。

### <a name="policy-rule"></a>原則規則
安全性原則的一部分，說明會觸發此原則的情況，以及原則觸發時所採取的動作。

### <a name="prevention"></a>預防
預防藉由不當使用疑似或已知遭到入侵的身分識別或裝置來傷害組織的動作。 預防動作並不會保護裝置或身分識別的安全，且不會解決先前的風險事件。

### <a name="privileged-user"></a>特殊權限 (使用者)
在發生風險事件時，擁有永久或暫時系統管理權限，可存取 Azure Active Directory 中一或多項資源的使用者，例如全域管理員、計費管理員、服務管理員、使用者管理員和密碼管理員。 

### <a name="real-time"></a>即時
請參閱即時偵測。

### <a name="real-time-detection"></a>即時偵測
在允許事件繼續進行之前，偵測異常狀況及評估事件的風險 (例如登入嘗試)。

### <a name="remediated-risk-event"></a>已補救 (風險事件)
Identity Protection 自動設定的風險事件狀態，表示已使用此風險事件類型的標準補救動作來補救此風險事件。 例如，當使用者密碼重設時，會自動補救許多指出先前密碼已遭入侵的風險事件。

### <a name="remediation"></a>補救
用來保護先前疑似或已知遭到入侵的身分識別或裝置的動作。 補救動作可讓身分識別或裝置還原到安全的狀態，以及解決先前與身分識別或裝置相關聯的風險事件。

### <a name="resolved-risk-event"></a>已解決 (風險事件)
由 Identity Protection 使用者手動設定的風險事件狀態，表示使用者在 Identity Protection 外部採取適當的補救動作，而且應該將風險事件視為已關閉。

### <a name="risk-event-status"></a>風險事件狀態
風險事件的屬性，指出事件是否為作用中，若已關閉，則會指出其關閉原因。

### <a name="risk-event-type"></a>風險事件類型
風險事件的類別，指出導致事件被視為有風險的異常類型。

### <a name="risk-level-risk-event"></a>風險層級 (風險事件)
指出風險事件的嚴重性 (高、中或低)，可協助 Identity Protection 使用者排定為了降低組織風險而採取之行動的優先順序。 

### <a name="risk-level-sign-in"></a>風險層級 (登入)
指出特定登入 (其他人正嘗試使用使用者的身分識別) 的可能性 (高、中或低)。

### <a name="risk-level-user-compromise"></a>風險層級 (使用者入侵)
指出身分識別遭到入侵的可能性 (高、中或低)。

### <a name="risk-level-vulnerability"></a>風險層級 (弱點)
指出弱點的嚴重性 (高、中或低)，可協助 Identity Protection 使用者排定為了降低組織風險而採取之行動的優先順序。

### <a name="secure-identity"></a>安全 (身分識別)
採取補救動作 (例如變更密碼或機器重新安裝映像)，將可能遭到入侵的身分識別還原至未遭入侵的狀態。

### <a name="security-policy"></a>安全性原則
原則規則和條件的集合。 原則可以套用至各種實體，例如使用者、群組、應用程式、裝置、裝置平台、裝置狀態、IP 範圍和 Auth2.0 用戶端類型。 啟用原則後，就會在納入原則的實體被核發資源的權杖時進行評估。

### <a name="sign-in-v"></a>登入 (動詞)
在 Azure Active Directory 中驗證身分識別。

### <a name="sign-in-n"></a>登入 (名詞)
在 Azure Active Directory 中驗證身分識別的程序或動作，以及擷取這項作業的事件。

### <a name="sign-in-from-anonymous-ip-address"></a>從匿名 IP 位址登入
從被視為匿名 Proxy IP 位址的 IP 位址成功登入之後所觸發的風險事件。

### <a name="sign-in-from-infected-device"></a>從受感染的裝置登入
當登入源自已知由一或多個遭入侵裝置 (這類裝置會主動嘗試與 Bot 伺服器通訊) 使用的 IP 位址時所觸發的風險事件。

### <a name="sign-in-from-ip-address-with-suspicious-activity"></a>從具有可疑活動的 IP 位址登入
從短期內透過多個使用者帳戶多次嘗試登入失敗的 IP 位址成功登入之後所觸發的風險事件。

### <a name="sign-in-from-unfamiliar-location"></a>從不熟悉的位置登入
當使用者從新的位置 (IP、經緯度和 ASN) 成功登入時所觸發的風險事件。

### <a name="sign-in-risk"></a>登入風險
請參閱風險層級 (登入)

### <a name="sign-in-risk-policy"></a>登入風險原則
一個條件式存取原則，可評估特定登入的風險，並根據預先定義的條件和規則來套用緩和動作。

### <a name="user-compromise-risk"></a>使用者入侵風險
請參閱風險層級 (使用者入侵)

### <a name="user-risk"></a>使用者風險
請參閱風險層級 (使用者入侵)。

### <a name="user-risk-policy"></a>使用者風險原則
一個條件式存取原則，可根據預先定義的條件和規則來考量及套用緩和動作。

### <a name="users-flagged-for-risk"></a>標示有風險的使用者
具有作用中或已補救風險事件的使用者

### <a name="vulnerability"></a>弱點
Azure Active Directory 中的組態或狀況，此組態或狀況會使目錄容易受到入侵或威脅的影響。

## <a name="see-also"></a>另請參閱
* [Azure Active Directory Identity Protection](../active-directory-identityprotection.md)

