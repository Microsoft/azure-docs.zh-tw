---
title: 大量邀請 B2B 共同作業使用者的教學課程 - Azure Active Directory | Microsoft Docs
description: 在此教學課程中，您會了解如何使用 PowerShell 和 CSV 檔案，對外部 Azure AD B2B 共同作業使用者傳送大量邀請。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: tutorial
ms.date: 08/14/2018
ms.author: mimart
author: msmimart
ms.reviewer: mal
ms.openlocfilehash: c24a82d5765fef01eab9ae24f637c215c62e822d
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45986897"
---
# <a name="tutorial-bulk-invite-azure-ad-b2b-collaboration-users"></a>教學課程：大量邀請 Azure AD B2B 共同作業使用者

如果您使用 Azure Active Directory (Azure AD) B2B 共同作業來與外部合作夥伴合作，您便可以同時邀請多位來賓使用者存取您的組織資源。 在此教學課程中，您會了解如何使用 PowerShell 對外部使用者傳送大量邀請。 具體而言，您會執行下列操作：

> [!div class="checklist"]
> * 準備一個含有使用者資訊的逗點分隔值 (CSV) 檔案
> * 執行 PowerShell 指令碼來傳送邀請
> * 確認是否已將使用者新增至目錄

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 

## <a name="prerequisites"></a>先決條件

### <a name="install-the-latest-azureadpreview-module"></a>安裝最新版 AzureADPreview 模組
請務必安裝最新版的 Azure AD PowerShell for Graph 模組 (AzureADPreview)。 

首先，檢查您已安裝哪些模組。 請以已提高權限的使用者身分 (以系統管理員身分執行) 開啟 Windows PowerShell，然後執行下列命令：
 
````powershell  
Get-Module -ListAvailable AzureAD*
````

根據輸出，執行下列其中一個操作︰

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

### <a name="get-test-email-accounts"></a>取得測試電子郵件帳戶

您需要有兩個以上可接收邀請的測試電子郵件帳戶。 這些帳戶必須來自您的組織外部。 您可以使用任何類型的帳戶，包括社交帳戶，例如 gmail.com 或 outlook.com 地址。

## <a name="prepare-the-csv-file"></a>準備 CSV 檔案

在 Microsoft Excel 中，建立一個含有受邀者使用者名稱和電子郵件地址的 CSV 檔案。 請務必包含 **Name** 和 **InvitedUserEmailAddress** 欄位標題。 

例如，請以下列格式建立工作表：


![顯示等待使用者接受的 PowerShell 輸出](media/tutorial-bulk-invite/AddUsersExcel.png)

請將檔案儲存為 **C:\BulkInvite\Invitations.csv**。 

如果您沒有 Excel，則可以在任何文字編輯器 (例如「記事本」) 中建立 CSV 檔案。 請以逗點分隔每個值，並以換行分隔每個資料列。 

## <a name="sign-in-to-your-tenant"></a>登入您的租用戶

執行下列命令以連線至租用戶網域：

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```
例如： `Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`。

出現提示時，請輸入您的認證。

## <a name="send-bulk-invitations"></a>傳送大量邀請

若要傳送邀請，請執行下列 PowerShell 指令碼 (其中 **c:\bulkinvite\invitations.csv** 是 CSV 檔案的路徑)： 

```powershell
$invitations = import-csv c:\bulkinvite\invitations.csv
   
$messageInfo = New-Object Microsoft.Open.MSGraph.Model.InvitedUserMessageInfo
   
$messageInfo.customizedMessageBody = "Hello. You are invited to the Contoso organization."
   
foreach ($email in $invitations) 
   {New-AzureADMSInvitation `
      -InvitedUserEmailAddress $email.InvitedUserEmailAddress `
      -InvitedUserDisplayName $email.Name `
      -InviteRedirectUrl https://myapps.azure.com `
      -InvitedUserMessageInfo $messageInfo `
      -SendInvitationMessage $true
   }
```
此指令碼會對 Invitations.csv 檔案中的電子郵件地址傳送邀請。 針對每位使用者，您應該會看到與以下類似的輸出：

![顯示等待使用者接受的 PowerShell 輸出](media/tutorial-bulk-invite/B2BBulkImport.png)

## <a name="verify-users-exist-in-the-directory"></a>確認使用者存在於目錄中

若要確認是否已將受邀使用者新增至 Azure AD，請執行下列命令：
```powershell
 Get-AzureADUser -Filter "UserType eq 'Guest'"
```
您應該會看到系統以 *emailaddress*#EXT#@*domain*格式的使用者主體名稱 (UPN) 列出您所邀請的使用者。 例如 *lstokes_fabrikam.com#EXT#@contoso.onmicrosoft.com*，其中 contoso.onmicrosoft.com 是您傳送邀請時的來源組織。

## <a name="clean-up-resources"></a>清除資源

當您不再需要測試使用者帳戶時，可以刪除目錄中的測試使用者帳戶。 請執行下列命令來僅刪除使用者帳戶：

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```
例如：`Remove-AzureADUser -ObjectId "lstokes_fabrikam.com#EXT#@contoso.onmicrosoft.com"`


## <a name="next-steps"></a>後續步驟
在此教學課程中，您已對組織外部的來賓使用者傳送大量邀請。 接著，將了解邀請兌換程序如何運作。

> [!div class="nextstepaction"]
> [了解 Azure AD B2B 共同作業邀請兌換程序](redemption-experience.md)

