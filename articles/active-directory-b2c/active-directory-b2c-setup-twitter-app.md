---
title: 使用 Azure Active Directory B2C 設定註冊，並以 Twitter 帳戶登入 | Microsoft Docs
description: 使用 Azure Active Directory B2C，讓具有 Twitter 帳戶的客戶得以註冊和登入您的應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/19/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 819e6364aebab11097260c54575ab65914293ce7
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47180524"
---
# <a name="set-up-sign-up-and-sign-in-with-a-twitter-account-using-azure-active-directory-b2c"></a>使用 Azure Active Directory B2C 設定註冊，並以 Twitter 帳戶登入

## <a name="create-an-application"></a>建立應用程式

若要使用 Twitter 作為 Azure AD B2C 中的識別提供者，您需要建立 Twitter 應用程式。 如果您還沒有 Twitter 帳戶，可以在 [https://twitter.com/signup](https://twitter.com/signup) 中取得。

1. 使用您的 Twitter 帳戶認證登入 [Twitter 開發人員](https://developer.twitter.com/en/apps) \(英文\) 網站。
2. 選取 [Create an app] \(建立應用程式\)。
3. 輸入**應用程式名稱**和**應用程式說明**。
4. 在 [Website URL] \(網站 URL\) 中，輸入 `https://your-tenant.b2clogin.com`。 以您的租用戶名稱取代 `your-tenant`。 例如： https://contosob2c.b2clogin.com。
5. 在 [回呼 URL] 中輸入 `https://your-tenant.b2clogin.com/your-tenant.onmicrosoft.com/your-policy-Id/oauth1/authresp`。 將 `your-tenant` 取代為您的租用戶名稱，並將 `your-policy-Id` 取代為您的原則識別碼。 例如： `b2c_1A_signup_signin_twitter`。 即使租用戶在 Azure AD B2C 中是使用大寫字母來定義的，您還是需要在輸入租用戶名稱時，全部使用小寫字母。
6. 在頁面底部，閱讀並接受條款，然後選取 [Create] \(建立\)。
7. 在 [App details] \(應用程式詳細資料\) 頁面上，選取 [Edit] \(編輯\) > [Edit details] \(編輯詳細資料\)、核取 [Enable Sign in with Twitter] \(使用 Twitter 啟用登入\) 的方塊，然後選取 [Save] \(儲存\)。
8. 選取 [Keys and tokens] \(金鑰和權杖\)，並記錄**取用者 API 金鑰**和**取用者 API 祕密金鑰**的值，以供稍後使用。

## <a name="configure-twitter-as-an-identity-provider-in-your-tenant"></a>在租用戶中將 Twitter 設為識別提供者

1. 以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下頂端功能表中的**目錄和訂用帳戶篩選**，然後選擇包含您租用戶的目錄，以確定您使用的是包含 Azure AD B2C 租用戶的目錄。
3. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。
4. 選取 [識別提供者]，然後選取 [新增]。
5. 提供 [名稱]。 例如，輸入 *Twitter*。
6. 依序選取 [識別提供者類型]、[Twitter]，然後按一下 [確定]。
7. 選取 [設定此識別提供者]，然後針對**用戶端識別碼**輸入 API 金鑰，並針對**用戶端密碼**輸入 API 祕密金鑰。
8. 依序按一下 [確定] 和 [建立]，儲存您的 Twitter 設定。