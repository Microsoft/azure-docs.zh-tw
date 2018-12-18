---
title: 使用 Azure Active Directory B2C 設定註冊，並以微博帳戶登入 | Microsoft Docs
description: 使用 Azure Active Directory B2C，讓具有微博帳戶的客戶得以註冊和登入您的應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 4600ec9d76f682f0dedf8c802d23b32fd6126481
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47181493"
---
# <a name="set-up-sign-up-and-sign-in-with-a-weibo-account-using-azure-active-directory-b2c"></a>使用 Azure Active Directory B2C 設定註冊，並以微博帳戶登入

> [!NOTE]
> 此功能處於預覽狀態。
> 

## <a name="create-a-weibo-application"></a>建立 Weibo 應用程式

若要在 Azure Active Directory (Azure AD) B2C 中使用微博帳戶做為識別提供者，您需要在代表該帳戶的租用戶中建立應用程式。 如果您還沒有微博帳戶，可以在 [http://weibo.com/signup/signup.php?lang=en-us](http://weibo.com/signup/signup.php?lang=en-us) 中取得。

1. 使用您的微博帳戶認證登入[微博開發人員入口網站](http://open.weibo.com/)。
2. 登入之後，在右上角選取您的顯示名稱。
3. 在下拉式清單中，選取 [编辑开发者信息] \(編輯開發人員資訊)。
4. 輸入必要資訊，並選取 [提交]。
5. 完成電子郵件驗證程序。
6. 移至[識別驗證頁面](http://open.weibo.com/developers/identity/edit)。
7. 輸入必要資訊，並選取 [提交]。

### <a name="register-a-weibo-application"></a>註冊 Weibo 應用程式

1. 移至[新的 Weibo 應用程式註冊頁面](http://open.weibo.com/apps/new)。
2. 輸入必要的應用程式資訊。
3. 選取 [创建]\(建立\)。
4. 複製**應用程式金鑰**和**應用程式密碼**的值。 您需要這兩個值，才能將識別提供者新增至您的租用戶。
5. 上傳所需的相片，並輸入所需的資訊。
6. 選取 [保存以上信息]\(儲存\)。
7. 選取 [高级信息]\(進階資訊\)。
8. 選取 OAuth2.0 [授权设置] (\重新導向 URL\) 旁的 [编辑] \(編輯\)。
9. 針對 OAuth2.0 [授权设置] \(重新導向 URL) 輸入 `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`。 例如，如果您的租用戶名稱是 contoso，請將 URL 設定為 `https://contoso.b2clogin.com/contoso.onmicrosoft.com/oauth2/authresp`。
10. 選取 [提交]。  

## <a name="configure-a-weibo-account-as-an-identity-provider"></a>將微博帳戶設為識別提供者

1. 以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下頂端功能表中的**目錄和訂用帳戶篩選**，然後選擇包含您租用戶的目錄，以確定您使用的是包含 Azure AD B2C 租用戶的目錄。
3. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。
4. 選取 [識別提供者]，然後選取 [新增]。
5. 提供 [名稱]。 例如，輸入*微博*。
6. 依序選取 [識別提供者類型]、[微博 (預覽)]，然後按一下 [確定]。
7. 選取 [設定此識別提供者]，並輸入 [用戶端識別碼] (您先前記錄的應用程式金鑰)，然後輸入 [用戶端密碼] (您先前建立的微博應用程式的應用程式密碼)。
8. 依序按一下 [確定] 和 [建立]，儲存您的 Weibo 設定。
