---
title: 使用 Azure Active Directory B2C 設定註冊，並以 QQ 帳戶登入 | Microsoft Docs
description: 使用 Azure Active Directory B2C，讓具有 QQ 帳戶的客戶得以註冊和登入您的應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 23e44763b773835583d255104771d2329851aac8
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47181409"
---
# <a name="set-up-sign-up-and-sign-in-with-a-qq-account-using-azure-active-directory-b2c"></a>使用 Azure Active Directory B2C 設定註冊，並以 QQ 帳戶登入

> [!NOTE]
> 此功能處於預覽狀態。
> 

## <a name="create-a-qq-application"></a>建立 QQ 應用程式

若要在 Azure Active Directory (Azure AD) B2C 中使用 QQ 帳戶做為識別提供者，您需要在代表該帳戶的租用戶中建立應用程式。 如果您還沒有 QQ 帳戶，可以至 [https://ssl.zc.qq.com/en/index.html?type=1&ptlang=1033](https://ssl.zc.qq.com/en/index.html?type=1&ptlang=1033) 取得。

### <a name="register-for-the-qq-developer-program"></a>註冊 QQ 開發人員計劃

1. 使用您的 QQ 帳戶認證登入 [QQ 開發人員入口網站](http://open.qq.com)。
2. 登入之後，請移至 [http://open.qq.com/reg](http://open.qq.com/reg)，將您自己註冊為開發人員。
3. 選取 [个人]\(個人開發人員\)。
4. 輸入必要資訊，並選取 [下一步]\(下一步\)。
5. 完成電子郵件驗證程序。 在註冊為開發人員之後，您必須等待數天以進行核准。 

### <a name="register-a-qq-application"></a>註冊 QQ 應用程式

1. 移至 [https://connect.qq.com/index.html](https://connect.qq.com/index.html)。
2. 選取 [应用管理]\(應用程式管理\)。
5. 選取 [创建应用]\(建立應用程式\)，然後輸入所需的資訊。
7. 在 [授权回调域]\(回呼 URL\) 中，輸入 `https://your-tenant-name.b2clogin.com/your-tenant-name}.onmicrosoft.com/oauth2/authresp`。 例如，如果您的 `tenant_name` 是 contoso，請將 URL 設為 `https://contoso.b2clogin.com/contoso.onmicrosoft.com/oauth2/authresp`。
8. 選取 [创建应用]\(建立應用程式\)。
9. 在 [確認] 頁面上，選取 [应用管理]\(應用程式管理\) 以返回應用程式管理頁面。
10. 在您建立的應用程式旁邊，選取 [查看]\(檢視\)。
11. 選取 [修改]\(編輯\)。
12. 複製**應用程式識別碼**和**應用程式金鑰**。 您需要這兩個值，才能將識別提供者新增至您的租用戶。

## <a name="configure-qq-as-an-identity-provider"></a>將 QQ 設為識別提供者

1. 以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下頂端功能表中的**目錄和訂用帳戶篩選**，然後選擇包含您租用戶的目錄，以確定您使用的是包含 Azure AD B2C 租用戶的目錄。
3. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。
4. 選取 [識別提供者]，然後選取 [新增]。
5. 提供 [名稱]。 例如，輸入 *QQ*。
6. 依序選取 [識別提供者類型]、[QQ (預覽)]，然後按一下 [確定]。
7. 選取 [設定此識別提供者]，並輸入 [用戶端識別碼] (您先前記錄的用戶端識別碼)，然後輸入 [用戶端密碼] (您先前建立的 QQ 應用程式的應用程式金鑰)。
8. 依序按一下 [確定] 和 [建立]，儲存您的 QQ 設定。

