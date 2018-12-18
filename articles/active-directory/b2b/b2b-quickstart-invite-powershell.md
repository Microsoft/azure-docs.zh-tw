---
title: 使用 PowerShell 為 Azure Active Directory B2B 共同作業新增來賓使用者的快速入門 | Microsoft Docs
description: 在此快速入門中，您將了解如何使用 PowerShell 向外部的 Azure AD B2B 共同作業使用者傳送邀請。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: mimart
author: msmimart
ms.reviewer: mal
ms.openlocfilehash: f0dc63d84ec7583e721b116b450c890d46524622
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45986557"
---
# <a name="quickstart-add-a-guest-user-with-powershell"></a>快速入門：使用 PowerShell 新增來賓使用者

使用 Azure Active Directory B2B 共同作業，您就可以透過多種方式邀請外部夥伴加入您的應用程式與服務。 在先前的快速入門中，您已了解如何直接在 Azure Active Directory 系統管理入口網站中新增來賓使用者。 您也可以使用 PowerShell 一次新增一個來賓使用者，或是大量新增來賓使用者。 在此快速入門中，您將使用 New-AzureADMSInvitation 命令，將一個來賓使用者新增到您的 Azure 租用戶。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 

## <a name="prerequisites"></a>必要條件

### <a name="install-the-latest-azureadpreview-module"></a>安裝最新的 AzureADPreview 模組
請務必安裝最新版的 Azure AD PowerShell for Graph 模組 (AzureADPreview)。 

首先，檢查您已安裝哪些模組。 請以已提高權限的使用者身分 (以系統管理員身分執行) 開啟 Windows PowerShell，然後執行下列命令：
 
````powershell  
Get-Module -ListAvailable AzureAD*
````

如果顯示 AzureADPreview 模組，但沒有訊息指出有更高版本，則表示您已就緒。 否則，根據輸出，執行下列其中一個動作︰

- 如果未傳回任何結果，請執行下列命令來安裝 AzureADPreview 模組：
  
   ````powershell  
   Install-Module AzureADPreview
   ````
- 如果結果中僅顯示 AzureAD 模組，請執行下列命令來安裝 AzureADPreview 模組： 

   ````powershell 
   Uninstall-Module AzureAD 
   Install-Module AzureADPreview 
   ````
- 如果結果中僅顯示 AzureADPreview 模組，但您收到指出有更新版本的訊息，請執行下列命令來更新該模組： 

   ````powershell 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
  ````

您可能會收到提示，指出您正從未受信任的存放庫安裝模組。 如果您先前未將 PSGallery 存放庫設定為受信任的存放庫，就會發生此情況。 請按 **Y** 以安裝模組。

### <a name="get-a-test-email-account"></a>取得測試電子郵件帳戶

您需要可接收邀請的測試電子郵件帳戶。 這些帳戶必須來自您的組織外部。 您可以使用任何類型的帳戶，包括社交帳戶，例如 gmail.com 或 outlook.com 地址。

## <a name="sign-in-to-your-tenant"></a>登入您的租用戶

執行下列命令以連線到租用戶網域：

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```
例如：`Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`。

出現提示時，請輸入您的認證。

## <a name="send-an-invitation"></a>傳送邀請

1. 若要向測試電子郵件帳戶傳送邀請，請執行下列 PowerShell 命令 (將 **"Sanda"** 與 **sanda@fabrikam.com** 取代為您的測試電子郵件帳戶名稱與電子郵件地址)： 

   ```powershell
   New-AzureADMSInvitation -InvitedUserDisplayName "Sanda" -InvitedUserEmailAddress sanda@fabrikam.com -InviteRedirectURL https://myapps.azure.com -SendInvitationMessage $true
   ```
2. 此命令會向指定的電子郵件地址傳送邀請。 請檢查輸出，它看起來應類似下列範例：

   ![顯示等待使用者接受的 PowerShell 輸出](media/quickstart-invite-powershell/powershell-azureadmsinvitation-result.png)

## <a name="verify-the-user-exists-in-the-directory"></a>確認使用者存在於目錄中

1. 若要確認是否已將受邀使用者新增到 Azure AD，請執行下列命令：
 
   ```powershell
   Get-AzureADUser -Filter "UserType eq 'Guest'"
   ```
3. 請檢查輸出，以確認系統以 *emailaddress*#EXT#@*domain* 格式的使用者主體名稱 (UPN) 列出您所邀請的使用者。 例如 *sanda_fabrikam.com#EXT#@contoso.onmicrosoft.com*，其中 contoso.onmicrosoft.com 是您傳送邀請時的來源組織。

   ![PowerShell 輸出顯示新增的來賓使用者](media/quickstart-invite-powershell/powershell-guest-user-added.png)

## <a name="clean-up-resources"></a>清除資源

當您不再需要測試使用者帳戶時，可以刪除目錄中的測試使用者帳戶。 請執行下列命令來刪除使用者帳戶：

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```
例如：`Remove-AzureADUser -ObjectId "sanda_fabrikam.com#EXT#@contoso.onmicrosoft.com"`


## <a name="next-steps"></a>後續步驟
在此快速入門中，您會使用 PowerShell 邀請單一來賓使用者並將他新增到您的目錄中。 接下來，了解如何使用 PowerShell 大量邀請來賓使用者。

> [!div class="nextstepaction"]
> [教學課程：大量邀請 Azure AD B2B 共同作業使用者](tutorial-bulk-invite.md)
