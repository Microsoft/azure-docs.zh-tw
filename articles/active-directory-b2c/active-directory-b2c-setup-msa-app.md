---
title: 使用 Azure Active Directory B2C 設定註冊，並以 Microsoft 帳戶登入 | Microsoft Docs
description: 使用 Azure Active Directory B2C，讓具有 Microsoft 帳戶的客戶得以註冊和登入您的應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: e9c42429a8feb60ade27afe998cb9a267f1603dd
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47180898"
---
# <a name="set-up-sign-up-and-sign-in-with-a-microsoft-account-using-azure-active-directory-b2c"></a>使用 Azure Active Directory B2C 設定註冊，並以 Microsoft 帳戶登入

## <a name="create-a-microsoft-account-application"></a>建立 Microsoft 帳戶應用程式

若要在 Azure Active Directory (Azure AD) B2C 中使用 Microsoft 帳戶做為識別提供者，您需要在代表該帳戶的租用戶中建立應用程式。 如果您還沒有 Microsoft 帳戶，可以至 [https://www.live.com/](https://www.live.com/) 取得。

1. 請使用您的 Microsoft 帳戶認證登入 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)。
2. 在右上角選取 [新增應用程式]。
3. 輸入應用程式的**名稱**。 例如 *MSAapp1*。
4. 選取 [產生新密碼]，並確定您會複製該密碼，以便在設定識別提供者時使用。 此外，還要複製**應用程式識別碼**。 
5. 選取 [新增平台]，然後選擇 [Web]。
4. 在 [重新導向 URL] 中，輸入 `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`。 以您的租用戶名稱取代 `your-tenant-name`。
5. 選取 [ **儲存**]。

## <a name="configure-a-microsoft-account-as-an-identity-provider"></a>將 Microsoft 帳戶設為識別提供者

1. 以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下頂端功能表中的**目錄和訂用帳戶篩選**，然後選擇包含您租用戶的目錄，以確定您使用的是包含 Azure AD B2C 租用戶的目錄。
3. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。
4. 選取 [識別提供者]，然後選取 [新增]。
5. 提供 [名稱]。 例如，輸入 *MSA*。
6. 選取 [識別提供者類型]、選取 [Microsoft 帳戶]，然後按一下 [確定]。
7. 選取 [設定此識別提供者]，並輸入 [用戶端識別碼] (您先前記錄下來的應用程式識別碼)，然後輸入 [用戶端密碼] (您先前建立的 Microsoft 帳戶應用程式的密碼)。
8. 依序按一下 [確定] 與 [建立]，以儲存您的 Microsoft 帳戶設定。

