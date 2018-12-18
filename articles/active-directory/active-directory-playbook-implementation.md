---
title: Azure Active Directory PoC 腳本實作 | Microsoft Docs
description: 探索並快速實作身分識別和存取管理案例
services: active-directory
keywords: azure active directory, 腳本, 概念證明, PoC
documentationcenter: ''
author: dstefanMSFT
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/12/2017
ms.author: dstefan
ms.openlocfilehash: 2f01bdee5ffcc9e2f48278c7209da40ef64cc030
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34068849"
---
# <a name="azure-active-directory-proof-of-concept-playbook-implementation"></a>Azure Active Directory 概念證明腳本：實作

## <a name="foundation---syncing-ad-to-azure-ad"></a>基礎 - 同步處理 AD 到 Azure AD 

混合式身分識別是大多數已擁有內部部署目錄之企業客戶的基礎建置。 在這裡，我們的目標是要花最少的時間，來展示實際的身分識別與存取案例的價值。 

| 案例 | 建置組塊| 
| --- | --- |  
| [將您的內部部署身分識別擴充至雲端](#extending-your-on-premises-identity-to-the-cloud) | [目錄同步作業 - 密碼雜湊同步處理](active-directory-playbook-building-blocks.md#directory-synchronization---password-hash-sync-phs---new-installation) <br/>**注意**︰如果您已經有 DirSync/ADSync 或舊版的 Azure AD Connect，則這是選擇性步驟。 本指南中的某些案例可能需要較新版本的 Azure AD Connect。  <br/>[商標](active-directory-playbook-building-blocks.md#branding) | 
| [使用群組指派 Azure AD 授權](#assigning-azure-ad-licenses-using-groups) | [群組型授權](active-directory-playbook-building-blocks.md#group-based-licensing) |


### <a name="extending-your-on-premises-identity-to-the-cloud"></a>將您的內部部署身分識別擴充至雲端 

1. Bob 是任職於 Contoso 的 Active Directory 系統管理員。 公司要求他為一組使用者啟用身分識別即服務。 在執行過 Azure AD Connect 精靈之後，雲端中已有目標使用者的身分識別。 
2. Bob 要求 Susie (其中一位目標使用者) 存取 Azure Active Directory 存取面板，並確認她可以進行驗證。 Susie 看到了加上商標的登入頁面，以及一個可供啟用未來之應用程式存取權的空白存取面板。

### <a name="assigning-azure-ad-licenses-using-groups"></a>使用群組指派 Azure AD 授權 

1. Bob 是 Azure AD 全域管理員，他想要在一開始導入 Azure AD 時順便為一組特定使用者配置 Azure AD 授權。
2. Bob 建立了試驗使用者群組。 
3. Bob 將授權指派給該群組
4. Susie (其中一位資訊工作者) 因為職責的關係而加入到安全性群組
5. 經過一段時間，Susie 取得了 Azure AD Premium 授權的存取權。 這可在之後實現更多個 POC 案例。

## <a name="theme---lots-of-apps-one-identity"></a>佈景主題 - 許多應用程式、一個身分識別

| 案例 | 建置組塊| 
| --- | --- |  
| [整合 SaaS 應用程式 - 同盟 SSO](#integrate-saas-applications---federated-sso) | [SaaS 同盟 SSO 組態](active-directory-playbook-building-blocks.md#saas-federated-sso-configuration) <br/>[群組 - 委派的擁有權](active-directory-playbook-building-blocks.md#groups---delegated-ownership) |
| [整合 SaaS 應用程式 - 密碼 SSO](#integrate-saas-applications---password-sso) | [SaaS 密碼 SSO 組態](active-directory-playbook-building-blocks.md#saas-password-sso-configuration) |
| [SSO 和身分識別生命週期事件](#sso-and-identity-lifecycle-events) | [SaaS 和身分識別生命週期](active-directory-playbook-building-blocks.md#saas-and-identity-lifecycle) |
| [安全的共用帳戶存取](#secure-access-to-shared-accounts) | [SaaS 共用的帳戶設定](active-directory-playbook-building-blocks.md#saas-shared-accounts-configuration) |
| [安全的內部部署應用程式遠端存取](#secure-remote-access-to-on-premises-applications) | [應用程式 Proxy 組態](active-directory-playbook-building-blocks.md#app-proxy-configuration) |
| [同步處理 LDAP 身分識別至 Azure AD](#synchronize-ldap-identities-to-azure-ad) |  [一般 LDAP 連接器組態](active-directory-playbook-building-blocks.md#generic-ldap-connector-configuration) |

### <a name="integrate-saas-applications---federated-sso"></a>整合 SaaS 應用程式 - 同盟 SSO 

1. Bob 是 Azure AD 全域管理員，行銷部門請他幫他們啟用 ServiceNow 執行個體的存取權。 Bob 在 Azure AD 的文件中找到逐步解說教學課程並照著做，然後將指派應用程式使用者的權限委派給行銷小組主管 Kevin。 
2. Kevin 以 ServiceNow 權利擁有者的身分登入，並將 Susie 指派給應用程式。 Kevin 同時注意到 ServiceNow 已自動建立 Susie 的設定檔
3. Susie 是行銷部門的資訊工作者。 她登入 Azure AD，並在 myapps 入口網站中找到她已獲得指派的所有 SaaS 應用程式。 她可以從該處順暢地存取 ServiceNow。
4. 行銷部門想要稽核誰存取過 ServiceNow。 Bob 下載活動報告，並透過電子郵件向 Kevin 分享該報告。  

### <a name="sso-and-identity-lifecycle-events"></a>SSO 和身分識別生命週期事件

1. Susie 向公司請假，根據公司的政策，公司將她的內部部署 AD 帳戶暫時停用。 Susie 現在無法登入 Azure AD，因此無法存取 ServiceNow。 
2. Susie 從行銷部門平調到銷售部門。 Kevin 把她在 ServiceNow 的存取權移除。 Susie 登入 Azure AD myapps 後，隨即發現她無法再看到 [ServiceNow] 圖格。 10 分鐘過後，Kevin 確認 Susie 的帳戶早已從 ServiceNow 管理主控台停用。

### <a name="integrate-saas-applications---password-sso"></a>整合 SaaS 應用程式 - 密碼 SSO

1. Bob 設定 Atlassian HipChat 的存取權。 HipChat 已整合了密碼 SSO，並對 Susie 授與存取權
2. Susie 登入 myapps 入口網站，看到有連結可供下載 Azure AD IE 瀏覽器擴充功能，於是她隨即下載
3. 在按一下後，系統提示她輸入 HipChat 使用者名稱和密碼認證。 這是一次性作業，完成之後她就能存取 HipChat
4. 幾天之後，Susie 再次開啟 myapps 入口網站並按一下 HipChat。 這一次，她在存取時沒有遇到任何問題
5. HipChat 應用程式的擁有者 Kevin 想要稽核誰存取過應用程式。 Bob 下載稽核報告，並透過電子郵件向 Kevin 分享該報告。 

### <a name="secure-access-to-shared-accounts"></a>安全的共用帳戶存取 

1. Bob 負責保護銷售團隊成員所共用的 Twitter 名稱。 他將 Twitter 新增為 SSO 應用程式，並將它指派給銷售團隊的安全性群組。 他拿到共用帳戶的認證，並在系統中提供此認證。 
2. 由於很多人都知道這個共用的 Twitter 認證，此認證已不再值得信任。 Bob 於是啟用自動變換 Twitter 密碼的功能。
3. 身為銷售團隊一員的 Susie 登入 myapps 入口網站，然後看到有連結可供下載 Azure AD IE 瀏覽器擴充功能。 她因此安裝了該擴充功能。
4. 在按一下後，她直接存取了 Twitter。 她不知道密碼。
5. Arnold 也是銷售團隊的一員。 他在步驟 3-4 遇到了和 Susie 一樣的體驗
6. 銷售部門想要稽核誰存取過 Twitter。 Bob 下載活動報告，並透過電子郵件向 Kevin 分享該報告。 

### <a name="secure-remote-access-to-on-premises-applications"></a>安全的內部部署應用程式遠端存取

1. Bob 是 Azure AD 全域管理員，許多人向他提出要求，希望讓遠端工作的員工可以存取數個實用的內部部署資源，例如費用報銷應用程式。 他照著[應用程式 Proxy 文件](manage-apps/application-proxy-enable.md)的指示，安裝了連接器並將費用報銷發佈為應用程式 Proxy 的應用程式。 
2. Bob 向 Susie 分享了費用報銷應用程式的外部 URL，因為 Susie 也是其中一位需要遠端存取的員工。 她存取該連結，在對 AAD 進行驗證之後，她終於能夠存取費用報銷應用程式，可在遠端工作時繼續保持生產力。 
3. 接下來，Bob 使用相同的程序繼續發佈其他的內部部署應用程式，並視需要對使用者提供存取權。 他為比較敏感的已發佈應用程式新增了條件式存取和 Multi-Factor Authentication，以提升其安全性。

### <a name="synchronize-ldap-identities-to-azure-ad"></a>同步處理 LDAP 身分識別至 Azure AD

1. Bob 所任職公司的身分識別基礎結構很複雜。 大部分使用者都是在 Windows Server Active Directory Domain Services (ADDS) 內進行維護。 其中有些人則由 Active Directory 輕量型目錄服務 (ADLDS) 內的 HR 系統進行管理。
2. Bob 負責讓所有使用者 (也包括不在 ADDS 內的使用者) 能夠存取 SaaS 應用程式。
3. Bob 設定了一般 LDAP 連接器以在 Azure AD Connect 中從 ADLDS 提取資料。
4. Bob 建立了同步處理規則，以便將 LDAP 使用者填入 Metaverse 和 Azure AD
5. 身為 LDAP 使用者的 Susie 使用已同步處理的身分識別來存取其 SaaS 應用程式



> [!IMPORTANT] 
> 這是一種需要對 FIM/MIM 有某種程度熟悉的進階組態。 如果在生產環境中使用，建議您瀏覽[頂級支援](https://support.microsoft.com/premier)以查看此組態的相關問題。



## <a name="theme---increase-your-security"></a>佈景主題 - 增加您的安全性 

| 案例 | 建置組塊| 
| --- | --- |  
| [安全的系統管理員帳戶存取權](#secure-administrator-account-access) | [使用電話進行 Azure MFA](active-directory-playbook-building-blocks.md#azure-multi-factor-authentication-with-phone-calls) |
| [安全的應用程式存取](#secure-access-to-applications) | [SaaS 應用程式的條件式存取](active-directory-playbook-building-blocks.md#mfa-conditional-access-for-saas-applications) |
| [啟用即時系統管理](#enable-just-in-time-jit-administration) | [Privileged Identity Management](active-directory-playbook-building-blocks.md#privileged-identity-management-pim) |
| [根據風險保護身分識別](#protect-identities-based-on-risk) | [探索風險事件](active-directory-playbook-building-blocks.md#discovering-risk-events) <br/>[部署登入風險原則](active-directory-playbook-building-blocks.md#deploying-sign-in-risk-policies) |
| [使用憑證式驗證無需密碼進行驗證](#authenticate-without-passwords-using-certificate-based-authentication) | [設定憑證式驗證](active-directory-playbook-building-blocks.md#configuring-certificate-based-authentication)

### <a name="secure-administrator-account-access"></a>安全的系統管理員帳戶存取權

1. Bob 是 Azure AD 全域管理員。 他已將 Stuart 識別為服務的共同管理員。 
2. Bob 將 Stuart 的帳戶設定為永遠需要 MFA 以改善安全性狀態
3. Stuart 登入 Azure 入口網站，並注意到他必須註冊自己的電話號碼才能繼續登入
4. Stuart 後續的登入現在都會以 Multi-Factor Authentication 進行保護，現在他接到一通電話要驗證其身分識別。

### <a name="secure-access-to-applications"></a>安全的應用程式存取

1. Kevin 是 ServiceNow 的業務擁有者。 該公司現在希望這些使用者在從公司網路外部進行存取時，使用 MFA 來登入。
2. Bob 是我們的 Azure AD 全域管理員，他在 ServiceNow 應用程式中新增了條件式存取原則，來為外部存取啟用 MFA
3. Susie 是我們的資訊工作者，她登入 myapps 入口網站，然後按一下 [ServiceNow] 圖格。 她現在正在接受 MFA 的查問。

### <a name="enable-just-in-time-jit-administration"></a>啟用即時 (JIT) 系統管理

1. Bob 和 Stuart 是 Azure AD 全域管理員。 他們想要在為管理角色啟用 JIT 存取權的同時，留下特殊權限角色的使用記錄。
2. Bob 在 Azure AD 租用戶中啟用 PIM，並成為安全性系統管理員。 他將自己和 Stuart 的全域管理員角色成員資格從永久變成合格。
3. Bob 和 Stuart 現在需要先透過 Azure 入口網站啟用其角色，才能變更 Azure AD 組態。 

### <a name="protect-identities-based-on-risk"></a>根據風險保護身分識別 

1. Susie 是一位資訊工作者，她嘗試從 Tor 瀏覽器登入。 
2. Bob 查看了 Azure AD Identity Protection 儀表板，並從匿名 IP 位址看見 Susie 的登入。 安全性小組想要使用 MFA 查問進行這類存取的使用者
3. Bob 讓 Azure AD Identity Protection 原則針對中度以上風險的事件查問 MFA
4. 時間流逝，Susie 再次從 Tor 瀏覽器登入。 這次，她將會看到 MFA 查問

### <a name="authenticate-without-passwords-using-certificate-based-authentication"></a>使用憑證式驗證無需密碼進行驗證

1. Bob 是金融機構的全域管理員，該機構禁止使用密碼作為其應用程式的驗證因素。
2. Bob 在 ADFS 與 Azure AD 上啟用並強制執行憑證驗證
3. Susie 在存取應用程式時會收到系統提示，要求她使用憑證進行驗證

## <a name="theme---scale-with-self-service"></a>佈景主題 - 使用自助式縮放比例

| 案例 | 建置組塊| 
| --- | --- |  
| [自助式密碼重設](#self-service-password-reset) | [自助式密碼重設](active-directory-playbook-building-blocks.md#self-service-password-reset) |
| [自助式存取應用程式](#self-service-access-to-applications) | [自助式存取應用程式](active-directory-playbook-building-blocks.md#self-service-access-to-application-management) |

### <a name="self-service-password-reset"></a>自助式密碼重設 

1. Bob 是 Azure AD 全域管理員，並已為一小群使用者啟用自助式密碼管理，Susie 也包括在內。 
2. Susie 登入 myapps 入口網站後看到訊息，內容是要求她註冊安全性資訊以供日後的密碼重設事件使用。
3. 時間快轉幾天，Susie 忘記其密碼，因此她透過 Azure AD 入口網站重設密碼

### <a name="self-service-access-to-applications"></a>自助式存取應用程式 

1. Kevin 是 ServiceNow 的業務擁有者。 他想讓使用者在需要時再「註冊」ServiceNow，而不是一次就將所有使用者新增到 ServiceNow
2. Bob 是我們的 Azure AD 全域管理員，他修改了 ServiceNow 應用程式以啟用自助式要求
3. Susie 是我們的資訊工作者，她登入 myapps 入口網站並按一下 [新增更多應用程式] 按鈕，隨後就看到 ServiceNow 已成為其中一個建議應用程式。 接下來，她瀏覽回 myapps 入口網站，並看到 ServiceNow 應用程式。

[!INCLUDE [active-directory-playbook-toc](../../includes/active-directory-playbook-steps.md)]