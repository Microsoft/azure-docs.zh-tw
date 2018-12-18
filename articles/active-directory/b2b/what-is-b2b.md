---
title: 什麼是 Azure Active Directory B2B 共同作業？ | Microsoft Docs
description: Azure Active Directory B2B 共同作業支援來賓使用者存取權，以便您可以安全地共用資源，並與外部夥伴共同作業。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: overview
ms.date: 09/14/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: mal
ms.openlocfilehash: 609f48d0c2ba41d658850ade367d9b225941b105
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45982603"
---
# <a name="what-is-guest-user-access-in-azure-active-directory-b2b"></a>什麼是 Azure Active Directory B2B 中的來賓使用者存取權？

Azure Active Directory (Azure AD) 企業對企業 (B2B) 共同作業，可讓您與來賓使用者和來自其他組織的外部夥伴安全地共用公司的應用程式與服務，同時持續控制貴公司的資料。 即使沒有 Azure AD 或 IT 部門，也可以安全地與外部合作夥伴 (無論大型或小型) 合作。 透過簡單的邀請和兌換程序，夥伴可以使用自己的認證來存取您的公司資源。 開發人員可以使用 Azure AD 企業對企業 API 來自訂邀請程序，或撰寫自助式註冊入口網站等應用程式。

觀看影片以了解如何透過邀請來賓使用者使用自己的身分識別登入您的公司應用程式與服務，安全地與他們共同作業。

下列影片提供實用的概觀。

>[!VIDEO https://www.youtube.com/embed/AhwrweCBdsc]

## <a name="collaborate-with-any-partner-using-their-identities"></a>使用他們的身分識別與任何夥伴共同作業
使用 Azure AD B2B，夥伴可使用自己的身分識別管理解決方案，因此您的組織不會產生外部系統管理額外負荷。 
- 夥伴使用他們自己的身分識別與認證；Azure AD 不是必需的。 
- 您不需要管理外部帳戶或密碼。 
- 您不需要同步處理帳戶或管理帳戶的生命週期。  

![新增成員](media/what-is-b2b/add-member.png)

## <a name="invite-guest-users-with-a-simple-invitation-and-redemption-process"></a>使用簡單的邀請與兌換程序邀請來賓使用者
來賓使用者使用自己的公司、學校或社交身分識別登入您的應用程式與服務。 如果來賓使用者沒有 Microsoft 帳戶或 Azure AD 帳戶，系統就會在他們兌換其邀請時為他們建立一個帳戶。 
- 使用來賓使用者選擇的電子郵件身分識別邀請他們。
- 傳送應用程式的直接連結，或向來賓使用者本身的存取面板傳送邀請。 
- 來賓使用者只需幾個簡單的兌換步驟即可登入。

![存取面板](media/what-is-b2b/consentscreen.png)

## <a name="use-policies-to-securely-share-your-apps-and-services"></a>使用原則來安全地共用您的應用程式與服務
您可以使用授權原則保護您的公司內容。 可以強制執行條件式存取原則，例如多重要素驗證：
- 在租用戶層級。
- 在應用程式層級。
- 針對特定來賓使用者來保護公司應用程式與資料。

![新增來賓使用者](media/what-is-b2b/tutorial-mfa-policy-2.png)


## <a name="easily-add-guest-users-in-the-azure-ad-portal"></a>輕鬆地在 Azure AD 入口網站中新增來賓使用者

系統管理員可以在 Azure 入口網站中輕鬆地將來賓使用者新增到您的組織。
- 在 Azure AD 中建立新的來賓使用者，類似於加入新使用者的方式。
- 來賓使用者會立即收到可自訂的邀請，讓他們登入存取面板。
- 目錄中的來賓使用者可以指派至應用程式或群組。  

![新增來賓使用者](media/what-is-b2b/adding-b2b-users-admin.png)

## <a name="let-application-and-group-owners-manage-their-own-guest-users"></a>讓應用程式與群組擁有者管理自己的來賓使用者

您可以將來賓使用者管理委派給應用程式擁有者，讓他們可以將來賓使用者直接新增到他們想要共用的任何應用程式，無論它是否為 Microsoft 應用程式。 
 - 系統管理員可設定自助式應用程式與群組管理。
 - 非系統管理員必須使用其[存取面板](https://myapps.microsoft.com)將來賓使用者新增到應用程式或群組。

![新增來賓使用者](media/what-is-b2b/access-panel-manage-app.png)

## <a name="use-apis-and-sample-code-to-easily-build-applications-to-onboard"></a>使用 API 和範例程式碼來輕鬆建置要上架的應用程式

讓您的外部合作夥伴以針對您組織需求量身打造的方式上線使用
- 使用 [B2B 共同作業邀請 API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation) 來自訂上架體驗，包括建立自助式註冊入口網站。 
- 使用我們在 [Github](https://github.com/Azure/active-directory-dotnet-graphapi-b2bportal-web) 提供自助式入口網站的範例程式碼。

![註冊入口網站](media/what-is-b2b/sign-up-portal.png)

## <a name="next-steps"></a>後續步驟

- [Azure AD B2B 共同作業授權指導方針](licensing-guidance.md)
- [在入口網站中新增 B2B 共同作業來賓使用者](add-users-administrator.md)
- [了解邀請兌換程序](redemption-experience.md)
- 一如往常，如有任何意見反應、討論及建議，請透過我們的 [Microsoft 技術社群 ](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-B2B/bd-p/AzureAD_B2b) (英文) 與產品小組聯繫。