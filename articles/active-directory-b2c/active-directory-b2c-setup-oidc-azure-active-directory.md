---
title: 在 Azure Active Directory B2C 中使用內建原則新增 Azure AD 提供者 | Microsoft Docs
description: 了解如何新增 Open ID Connect 識別提供者 (Azure AD)。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/27/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: e09ad89f3225af9de40781fafc022c8326f80619
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338633"
---
# <a name="azure-active-directory-b2c-sign-in-using-azure-ad-accounts-through-a-built-in-policy"></a>Azure Active Directory B2C：透過內建原則使用 Azure AD 帳戶登入

>[!NOTE]
> 這項功能處於公開預覽狀態。 請勿在生產環境中使用這項功能。

本文示範為特定 Azure Active Directory (Azure AD) 組織內建原則的使用者啟用登入。

## <a name="create-an-azure-ad-app"></a>建立 Azure AD 應用程式

若要讓特定 Azure AD 組織的使用者登入，您需要在組織 Azure AD 租用戶內註冊應用程式。

>[!NOTE]
> 我們在下列指示中使用 "contoso.com" 作為組織的 Azure AD 租用戶，以及使用 "fabrikamb2c.onmicrosoft.com" 作為 Azure AD B2C 租用戶。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 在頂端列上，選取您的帳戶。 從 [目錄] 清單中，選擇您要註冊應用程式的組織 Azure AD 租用戶 (contoso.com)。
1. 選取左窗格中的 [所有服務]，然後搜尋「應用程式註冊」。
1. 選取 [新增應用程式註冊]。
1. 輸入應用程式的名稱 (例如，`Azure AD B2C App`)。
1. 選取 [Web 應用程式/API] 作為應用程式類型。
1. 針對 [登入 URL]，輸入下列 URL，其中 `yourtenant` 由 Azure AD B2C 租用戶的名稱 (`fabrikamb2c`) 取代：

    >[!NOTE]
    >在**登入 URL** 中，「yourtenant」的值必須全部小寫。

    ```Console
    https://yourtenant.b2clogin.com/te/yourtenant.onmicrosoft.com/oauth2/authresp
    ```

1. 儲存應用程式識別碼，在下一節中會將它當作用戶端識別碼使用。
1. 在 [設定] 刀鋒視窗下，選取 [金鑰]。
1. 在 [密碼] 區段下，輸入 [金鑰描述]，並將 [持續時間] 設定成 [永不過期]。 
1. 按一下 [儲存]，然後記下產生的金鑰 [值]，在下一節中會將它當作用戶端密碼使用。

## <a name="configure-azure-ad-as-an-identity-provider-in-your-tenant"></a>將 Azure AD 設定成您租用戶中的識別提供者

1. 在頂端列上，選取您的帳戶。 從 [目錄] 清單中，選擇 Azure AD B2C 租用戶 (fabrikamb2c.onmicrosoft.com)。
1. 在 Azure 入口網站中，[瀏覽至 [Azure AD B2C] 設定功能表](active-directory-b2c-app-registration.md#navigate-to-b2c-settings)。
1. 在 [Azure AD B2C] 設定 功能表中，按一下 [識別提供者]。
1. 按一下刀鋒視窗頂端的 [新增]  。
1. 針對身分識別提供者組態，提供容易辨識的 **名稱** 。 例如，輸入 "Contoso Azure AD"。
1. 按一下 [識別提供者類型]，選取 [Open ID Connect]，然後按一下 [確定]。
1. 按一下 [設定此識別提供者]。
1. 針對 [中繼資料 URL]，輸入以下 URL，並使用您的 Azure AD 租用戶名稱 (如 `contoso.com`) 取代 `yourtenant`：

    ```Console
    https://login.microsoftonline.com/yourtenant/.well-known/openid-configuration
    ```
1. 針對 [用戶端識別碼] 和 [用戶端密碼]，輸入上一節中的應用程式識別碼和金鑰。
1. 保留 [範圍] 的預設值，且應設定成 `openid`。
1. 保留 [回應類型] 的預設值，且應設定成 `code`。
1. 保留 [回應模式] 的預設值，且應設定成 `form_post`。
1. 您也可以選擇輸入 [網域] 的值 (如 `ContosoAD`)。 這是在要求中使用 *domain_hint* 參考此識別提供者時，要使用的值。 
1. 按一下 [確定]。
1. 在 [對應此識別提供者的宣告] 上按一下。
1. 針對 [使用者識別碼]，輸入 `oid`。
1. 針對 [顯示名稱]，輸入 `name`。
1. 針對 [名字]，輸入 `given_name`。
1. 針對 [姓氏]，輸入 `family_name`。
1. 針對 [電子郵件]，輸入 `unique_name`
1. 按一下 [確定]，然後按一下 [建立] 以儲存您的設定。

## <a name="next-steps"></a>後續步驟

將新建立的 Azure AD 身分識別提供者新增到內建原則，並提供意見反應到 [AADB2CPreview@microsoft.com](mailto:AADB2CPreview@microsoft.com)。
