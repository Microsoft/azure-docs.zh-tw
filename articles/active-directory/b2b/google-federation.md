---
title: 將 Google 新增為 Azure Active Directory B2B 的識別提供者 | Microsoft Docs
description: 與 Google 同盟以便讓來賓使用者使用自己的 Gmail 帳戶登入 Azure AD 應用程式
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 08/20/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: mal
ms.openlocfilehash: 396fb947a95ebc634ab0dea24d20f35126bc006e
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389440"
---
# <a name="add-google-as-an-identity-provider-for-b2b-guest-users"></a>將 Google 新增為 B2B 來賓使用者的識別提供者

藉由設定 Google 同盟，您可以讓受邀的使用者使用自己的 Google 帳戶登入共用應用程式和資源，而不需要建立 Microsoft 帳戶 (MSA) 或 Azure AD 帳戶。  
> [!NOTE]
> Google 來賓使用者必須使用包含租用戶內容的連結 (例如 `https://myapps.microsoft.com/?tenantid=<tenant id>`) 來登入。 應用程式和資源的直接連結只要包含租用戶內容，也可有同樣作用。 來賓使用者目前無法使用沒有租用戶內容的端點來登入。 例如，使用 `https://myapps.microsoft.com`、`https://portal.azure.com` 或小組通用端點將會導致錯誤。
 
## <a name="what-is-the-experience-for-the-google-user"></a>Google 使用者有何體驗？
當您傳送邀請給 Google Gmail 使用者時，來賓使用者應該會使用包含租用戶內容的連結來存取共用應用程式或資源。 其體驗會因為是否已登入 Google 而有所不同：
  - 如果來賓使用者未登入 Google，系統會提示他們登入 Google。
  - 如果來賓使用者已登入 Google，系統會提示他們選擇所要使用的帳戶。 他們必須選擇您用來邀請他們的帳戶。

來賓使用者若看到「標頭太長」錯誤，則可嘗試清除其 Cookie，也可開啟私密或匿名的視窗，然後再試著登入一次。

![以 Google 登入](media/google-federation/google-sign-in.png)

## <a name="step-1-configure-a-google-developer-project"></a>步驟 1︰設定 Google 開發人員專案
首先，在 Google 開發人員主控台中，建立新專案來取得用戶端識別碼和用戶端密碼，以供稍後新增至 Azure AD。 
1. 移至 Google API (網址為： https://console.developers.google.com)，並使用 Google 帳戶來登入。 建議您使用共用小組的 Google 帳戶。
2. 建立新專案：在儀表板上，選取 [建立專案]，然後選取 [建立]。 在 [新增專案] 頁面上，輸入 [專案名稱]，然後選取 [建立]。
   
   ![新增 Google 專案](media/google-federation/google-new-project.png)

3. 確實地在 [專案] 功能表中選取新專案。 然後，開啟左上方的功能表，然後選取 [API 和服務] > [認證]。

   ![Google API 認證](media/google-federation/google-api.png)
 
4. 選擇 [Oauth 同意畫面] 索引標籤，然後輸入 [向使用者顯示的產品名稱]。 (其他設定保留不變)。選取 [ **儲存**]。

   ![Google OAuth 同意畫面](media/google-federation/google-oauth-consent-screen.png)

5. 選擇 [認證] 索引標籤。在 [建立認證] 功能表中，選擇 [OAuth 用戶端識別碼]。

   ![Google API 認證](media/google-federation/google-api-credentials.png)

6. 在 [應用程式類型] 底下，選擇 [Web 應用程式]，然後在 [授權的重新導向 URI] 底下，輸入下列 URI：
   - `https://login.microsoftonline.com` 
   - `https://login.microsoftonline.com/te/<directory id>/oauth2/authresp` <br>(其中，`<directory id>` 是目錄識別碼)
   
    > [!NOTE]
    > 若要尋找目錄識別碼，請移至 https://portal.azure.com，然後在 [Azure Active Directory] 底下，選擇 [屬性] 並複製 [目錄識別碼]。

   ![建立 OAuth 用戶端識別碼](media/google-federation/google-create-oauth-client-id.png)

7. 選取 [建立] 。 複製用戶端識別碼和用戶端密碼，以便用來在 Azure AD 入口網站中新增識別提供者。

   ![OAuth 用戶端識別碼和用戶端密碼](media/google-federation/google-auth-client-id-secret.png)

## <a name="step-2-configure-google-federation-in-azure-ad"></a>步驟 2：在 Azure AD 中設定 Google 同盟 
現在，您會設定 Google 用戶端識別碼和用戶端密碼，可行方法是在 Azure AD 入口網站中加以輸入，也可以使用 PowerShell 來輸入。 請務必藉由使用 Gmail 地址邀請您自己，並嘗試使用受邀的 Google 帳戶兌換邀請，來測試 Google 同盟設定。 

#### <a name="to-configure-google-federation-in-the-azure-ad-portal"></a>在 Azure AD 入口網站中設定 Google 同盟 
1. 移至 [Azure 入口網站](https://portal.azure.com)。 在左窗格中，選取 [Azure Active Directory]。 
2. 選取 [組織關係]。
3. 選取 [識別提供者]，然後按一下 [Google] 按鈕。
4. 輸入名稱。 然後輸入您稍早取得的用戶端識別碼和用戶端密碼。 選取 [ **儲存**]。 

   ![新增 Google 識別提供者](media/google-federation/google-identity-provider.png)

#### <a name="to-configure-google-federation-by-using-powershell"></a>使用 PowerShell 設定 Google 同盟
1. 安裝最新版的 Azure AD PowerShell for Graph 模組 ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview))。
2. 執行下列命令：`Connect-AzureAD`。
3. 在出現登入提示時，使用受控的全域系統管理員帳戶來登入。  
4. 執行以下命令： 
   
   `New-AzureADMSIdentityProvider -Type Google -Name Google -ClientId [Client ID] -ClientSecret [Client secret]`
 
   > [!NOTE]
   > 請使用您在＜步驟 1︰設定 Google 開發人員專案＞所建立的應用程式中，所取得的用戶端識別碼和用戶端密碼。 如需詳細資訊，請參閱 [New-AzureADMSIdentityProvider](https://docs.microsoft.com/powershell/module/azuread/new-azureadmsidentityprovider?view=azureadps-2.0-preview) 一文。 
 
## <a name="how-do-i-remove-google-federation"></a>如何移除 Google 同盟？
您可以刪除 Google 同盟設定。 如果您這麼做，已兌換邀請的 Google 來賓使用者將無法登入，但您可以先從目錄刪除這些使用者，再重新邀請他們，藉此為他們提供資源存取權。 
 
### <a name="to-delete-google-federation-in-the-azure-ad-portal"></a>在 Azure AD 入口網站中刪除 Google 同盟： 
1. 移至 [Azure 入口網站](https://portal.azure.com)。 在左窗格中，選取 [Azure Active Directory]。 
2. 選取 [組織關係]。
3. 選取 [識別提供者]，然後按一下 [Google] 按鈕。
4. 選取 [Google]，然後選取 [刪除]。 
   
   ![已刪除社交識別提供者](media/google-federation/google-social-identity-providers.png)

1. 選取 [是] 以確認要刪除。 

### <a name="to-delete-google-federation-by-using-powershell"></a>使用 PowerShell 刪除 Google 同盟： 
1. 安裝最新版的 Azure AD PowerShell for Graph 模組 ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview))。
2. 執行 `Connect-AzureAD`。  
4. 在登入提示中，使用受控的全域系統管理員帳戶來登入。  
5. 輸入下列命令：

    `Remove-AzureADMSIdentityProvider -Id Google-OAUTH`

   > [!NOTE]
   > 如需詳細資訊，請參閱 [Remove-AzureADMSIdentityProvider](https://docs.microsoft.com/powershell/module/azuread/Remove-AzureADMSIdentityProvider?view=azureadps-2.0-preview)。 