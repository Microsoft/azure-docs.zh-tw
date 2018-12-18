---
title: 為 Azure Active Directory 中的 B2B 共同作業設定 SaaS 應用程式 | Microsoft Docs
description: Azure Active Directory B2B 共同作業的程式碼與 PowerShell 範例
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 05/23/2017
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 78f34183b30f8ea9fe44ddaf2b57b237cbb09749
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45985757"
---
# <a name="configure-saas-apps-for-b2b-collaboration"></a>為 B2B 共同作業設定 SaaS 應用程式

Azure Active Directory (Azure AD) B2B 共同作業可搭配與 Azure AD 整合的大部分應用程式使用。 在本節中，我們將逐步說明用於設定一些常見 SaaS 應用程式以搭配 Azure AD B2B 使用的指示。

在您查看應用程式特定指示之前，以下是一些主要規則：

* 大部分的應用程式都需要以手動方式進行使用者設定。 也就是說，必須在應用程式中手動建立使用者。

* 若為支援自動設定的應用程式 (例如 Dropbox)，則會從應用程式建立個別的邀請。 使用者務必接受每個邀請。

* 在使用者屬性中，若要減少與來賓使用者中受損使用者設定檔磁碟 (UPD) 相關的任何問題，一律將 [使用者識別碼] 設定為 [user.mail]。


## <a name="dropbox-business"></a>Dropbox Business

若要讓使用者使用其組織帳戶登入，您必須手動設定 Dropbox Business，才能使用 Azure AD 做為安全性聲明標記語言 (SAML) 識別提供者。 如果 Dropbox Business 已這樣設定，則無法顯示提示或允許使用者使用 Azure AD 進行登入。

1. 若要將 Dropbox Business 應用程式新增至 Azure AD，請選取左的窗格中的 [企業應用程式]，然後按一下 [新增]。

  ![企業應用程式頁面上的 [新增] 按鈕](media/configure-saas-apps/add-dropbox.png)

2. 在 [加入應用程式] 視窗的搜尋方塊中輸入 **dropbox**，然後選取結果清單中的 [商務用 Dropbox]。

  ![在 [加入應用程式] 頁面上搜尋 "dropbox"](media/configure-saas-apps/add-app-dialog.png)

3. 在 [單一登入] 頁面上，選取左窗格中的 [單一登入]，然後在 [使用者識別碼] 方塊中輸入 **user.mail**。 (其預設為 UPN。)

  ![為應用程式設定單一登入](media/configure-saas-apps/configure-app-sso.png)

4. 若要下載憑證以用於 Dropbox 設定，請選取 [設定 DropBox]，然後選取清單中的 [SAML 單一登入服務 URL]。

  ![下載可用於 Dropbox 設定的憑證](media/configure-saas-apps/download-certificate.png)

5. 從 [單一登入] 頁面，使用登入 URL 登入 Dropbox。

  ![Dropbox 登入頁面](media/configure-saas-apps/sign-in-to-dropbox.png)

6. 在功能表上，選取 [系統管理員主控台]。

  ![Dropbox 功能表上的 [系統管理員主控台] 連結](media/configure-saas-apps/dropbox-menu.png)

7. 在 [驗證] 對話方塊中，選取 [更多]上傳憑證，然後在 [登入 URL] 方塊中，輸入 SAML 單一登入 URL。

  ![摺疊的 [驗證] 對話方塊中的 [更多] 連結](media/configure-saas-apps/dropbox-auth-01.png)

  ![展開的 [驗證] 對話方塊中的 [登入 URL]](media/configure-saas-apps/paste-single-sign-on-URL.png)

8. 若要在 Azure 入口網站中設定自動使用者設定，請選取左窗格中的 [佈建]、選取 [佈建模式] 方塊中的 [自動]，然後選取 [授權]。

  ![在 Azure 入口網站中設定自動使用者佈建](media/configure-saas-apps/set-up-automatic-provisioning.png)

在 Dropbox 應用程式中設定來賓或成員使用者之後，他們就會收到來自 Dropbox 的個別邀請。 若要使用 Dropbox 單一登入，受邀者必須藉由按一下邀請中的連結來接受邀請。

## <a name="box"></a>Box
使用以 SAML 通訊協定為基礎的同盟，即可讓使用者使用其 Azure AD 帳戶來驗證 Box 來賓使用者。 在此程序中，您會將中繼資料上傳至 Box.com。

1. 從企業應用程式新增 Box 應用程式。

2. 請依下列順序設定單一登入：

  ![設定 Box 單一登入](media/configure-saas-apps/configure-box-sso.png)

 a. 在 [登入 URL] 方塊中，確定已在 Azure 入口網站中針對 Box 適當設定登入 URL。 此 URL 是 Box.com 租用戶的 URL。 它應該遵循命名慣例 *https://.box.com*。  
 [識別碼] 不適用於此應用程式，但它仍會顯示為必要欄位。

 b. 在 [使用者識別碼] 方塊中，輸入 **user.mail** (適用於來賓帳戶的 SSO)。

 c. 在 [SAML 簽署憑證] 之下，按一下 [建立新憑證]。

 d. 若要開始將 Box.com 租用戶設定成使用 Azure AD 做為識別提供者，請下載中繼資料檔案，然後將它儲存到本機磁碟機。

 e. 將設定單一登入的中繼資料檔案轉寄給 Box 支援小組。

3. 若要進行 Azure AD 自動使用者設定，請在左窗格中選取 [佈建]，然後選取 [授權]。

  ![授權 Azure AD 以連線到 Box](media/configure-saas-apps/auth-azure-ad-to-connect-to-box.png)

如同 Dropbox 受邀者，Box 受邀者也必須從 Box 應用程式兌換其邀請。

## <a name="next-steps"></a>後續步驟

請參閱下列有關 Azure AD B2B 共同作業的文章：

- [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
- [動態群組與 B2B 共同作業](use-dynamic-groups.md)
- [B2B 共同作業使用者宣告對應](claims-mapping.md)
- [Office 365 外部共用](o365-external-user.md)

