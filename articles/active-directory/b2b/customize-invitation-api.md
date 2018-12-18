---
title: Azure Active Directory B2B 共同作業 API 和自訂 | Microsoft Docs
description: Azure Active Directory B2B 共同作業讓企業合作夥伴選擇性地存取您的公司應用程式，以支援公司間的關係
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: reference
ms.date: 04/11/2017
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 20c824da82d6e3e66bfa2d7447c8a9573cbdce69
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45985808"
---
# <a name="azure-active-directory-b2b-collaboration-api-and-customization"></a>Azure Active Directory B2B 共同作業 API 和自訂

有許多客戶告訴我們他們想要以最適合其組織的方式自訂邀請程序。 使用我們的 API，您可以進行自訂。 [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation)

## <a name="capabilities-of-the-invitation-api"></a>邀請 API 的功能
API 提供下列功能：

1. 使用*任何*電子郵件地址來邀請外部使用者。

    ```
    "invitedUserDisplayName": "Sam"
    "invitedUserEmailAddress": "gsamoogle@gmail.com"
    ```

2. 自訂使用者接受其邀請之後將會看到的內容。

    ```
    "inviteRedirectUrl": "https://myapps.microsoft.com/"
    ```

3. 選擇透過我們的服務 傳送標準邀請郵件

    ```
    "sendInvitationMessage": true
    ```

  使用您可以自訂的訊息傳送給收件者

    ```
    "customizedMessageBody": "Hello Sam, let's collaborate!"
    ```

4. 此外，您也可以選擇在邀請共同作業者時將相關人員新增到副本收件者。

5. 或者，透過選擇不以 Azure AD 傳送通知，以完全自訂您的邀請與上線工作流程。

    ```
    "sendInvitationMessage": false
    ```

  在此案例中，您會從 API 取得兌換 URL，您可以將它內嵌在電子郵件範本、IM 或其他發佈方法。

6. 最後，若您是系統管理員，您可以選擇邀請使用者做為成員。

    ```
    "invitedUserType": "Member"
    ```


## <a name="authorization-model"></a>授權模型
API 可以在下列授權模型中執行：

### <a name="app--user-mode"></a>應用程式 + 使用者模式
在此模式中，使用 API 的任何使用者必須有建立 B2B 邀請的權限。

### <a name="app-only-mode"></a>僅應用程式模式
在僅應用程式模式中，應用程式需要 User.Invite.All 範圍，邀請才能成功。

如需詳細資訊，請參閱： https://graph.microsoft.io/docs/authorization/permission_scopes


## <a name="powershell"></a>PowerShell
您現在可以輕鬆地使用 PowerShell 來新增並邀請外部使用者加入組織。 使用 Cmdlet 建立邀請：

```
New-AzureADMSInvitation
```

您可以使用下列選項：

* -InvitedUserDisplayName
* -InvitedUserEmailAddress
* -SendInvitationMessage
* -InvitedUserMessageInfo

您也可以在 [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation) 中查看邀請 API 參考

## <a name="next-steps"></a>後續步驟

- [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
- [B2B 共同作業邀請電子郵件的元素](invitation-email-elements.md)
- [B2B 共同作業邀請兌換](redemption-experience.md)
- [在沒有邀請的情況下新增 B2B 共同作業使用者](add-user-without-invite.md)

