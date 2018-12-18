---
title: 使用 Azure Active Directory B2C 設定註冊，並以 Amazon 帳戶登入 | Microsoft Docs
description: 使用 Azure Active Directory B2C，讓具有 Amazon 帳戶的客戶得以註冊和登入您的應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 5bc1e0faf01125f498eb339bba841b96f6de9d82
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47181799"
---
# <a name="set-up-sign-up-and-sign-in-with-an-amazon-account-using-azure-active-directory-b2c"></a>使用 Azure Active Directory B2C 設定註冊，並以 Amazon 帳戶登入

## <a name="create-an-amazon-application"></a>建立 Amazon 應用程式

若要在 Azure Active Directory (Azure AD) B2C 中使用 Amazon 帳戶做為識別提供者，您需要在代表該帳戶的租用戶中建立應用程式。 如果您還沒有 Amazon 帳戶，可以至 [http://www.amazon.com/](http://www.amazon.com/) 取得。

1. 以您的 Amazon 帳戶認證登入 [Amazon Developer Center (Amazon 開發人員中心)](https://login.amazon.com/)。
2. 若您尚未執行此動作，請按一下 [註冊] 、遵循開發人員註冊步驟，並接受原則。
3. 選取 [註冊新應用程式]。
4. 請輸入 [名稱]、[描述]，以及 [隱私權注意事項 URL]，然後按一下 [儲存]。 隱私權注意事項是您要提供給使用者之隱私權資訊的管理頁面。
5. 在 [Web 設定] 區段中，複製 [用戶端識別碼] 的值。 選取 [顯示密碼] 以取得用戶端密碼，然後複製該密碼。 您必須使用這兩個值，將 Amazon 帳戶設為租用戶中的識別提供者。 **用戶端密碼** 是重要的安全性認證。
6. 在 [Web 設定] 區段中，選取 [編輯]，然後在 [允許的 JavaScript 原始來源] 中，輸入 `https://your-tenant-name.b2clogin.com`，並在 [允許的傳回 URL] 中，輸入 `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`。 以您的租用戶名稱取代 `your-tenant-name`。 即使租用戶在 Azure AD B2C 中是使用大寫字母來定義的，您還是需要在輸入租用戶名稱時，全部使用小寫字母。
7. 按一下 [檔案] 。

## <a name="configure-an-amazon-account-as-an-identity-provider"></a>將 Amazon 帳戶設為識別提供者

1. 以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下頂端功能表中的**目錄和訂用帳戶篩選**，然後選擇包含您租用戶的目錄，以確定您使用的是包含 Azure AD B2C 租用戶的目錄。
3. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。
4. 選取 [識別提供者]，然後選取 [新增]。
5. 輸入 [名稱]。 例如，輸入 *Amazon*。
6. 依序選取 [識別提供者類型]、[Amazon]，然後按一下 [確定]。
7. 選取 [設定此識別提供者]，並輸入 [用戶端識別碼] (您先前記錄的**用戶端識別碼**) ，然後輸入 [用戶端密碼] (您記錄之先前建立的 Amazon 應用程式的**用戶端密碼**)。
8. 依序按一下 [確定] 和 [建立]，以儲存您的 Amazon 設定。

