---
title: 停用 Azure AD SSPR 和 MFA 的聚合式註冊 (公開預覽)
description: 停用 Azure AD Multi-Factor Authenticaiton 和自助密碼重設註冊 (公開預覽)
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 08/02/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry, michmcla
ms.openlocfilehash: 3ce08f67f001a7c43602627b9eeda3ad60f867c1
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39623216"
---
# <a name="disable-azure-ad-converged-registration-public-preview"></a>停用 Azure AD 聚合式註冊 (公開預覽)

當使用者在新的聚合式體驗中註冊其電話號碼和/或行動裝置應用程式時，我們的服務會為該使用者的那些方法戳記一組旗標 (StrongAuthenticationMethods)。 這項功能允許使用者在需要 MFA 時，使用這些方法執行 Azure Multi-Factor Authentication (MFA)。

使用者透過新體驗註冊的方法具有 StrongAuthenticationMethods 屬性集。 公開預覽推出後，也會發生這種行為。 如果系統管理員啟用預覽、使用者透過新體驗註冊，然後系統管理員停用預覽，用戶可能會在不知不覺中註冊MFA。

如果已完成聚合式註冊的使用者瀏覽至目前的 SSPR 註冊頁面，則在 [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup)，系統將提示他們在存取該頁面之前執行 MFA。 從技術觀點來看，此步驟是預期的行為，但對於先前僅註冊 SSPR 的使用者，此步驟會是新的行為。 雖然這個額外步驟會藉由提供一層額外的安全性改善使用者的安全性狀態，但系統管理員可能會想要復原其使用者，使它們不再能夠執行 MFA。  

## <a name="how-to-roll-back-users"></a>如何復原使用者

如果您作為系統管理員想要重設使用者的 MFA 設定，我們已建立 PowerShell 指令碼，該指令碼將清除使用者的行動裝置應用程式和/或電話號碼的 StrongAuthenticationMethods 屬性。 為您的使用者執行這個指令碼表示他們必須視需要重新註冊 MFA。 我們建議在復原所有受影響的使用者之前，使用一或兩個使用者測試復原。

下列步驟可協助您復原使用者群組：

### <a name="pre-requisites"></a>先決條件

1. 您必須安裝適當的 Azure AD PowerShell 模組。 在 PowerShell 視窗中，執行下列命令來安裝模組：

   ```powershell
   Install-Module -Name MSOnline
   Import-Module MSOnline
   ```

1. 將受影響的使用者物件識別碼/識別碼清單，以每行一個識別碼的文字檔儲存到您的電腦。 記下檔案的所在位置。
1. 將下列指令碼儲存至您的電腦，並記下指令碼的位置：

```powershell
<# 
//********************************************************
//*                                                      *
//*   Copyright (C) Microsoft. All rights reserved.      *
//*                                                      *
//********************************************************
#>

param($path)

# Define Remediation Fn
function RemediateUser {

    param  
    (
        $ObjectId
    )

    $user = Get-MsolUser -ObjectId $ObjectId

    Write-Host "Checking if user is eligible for rollback: UPN: "  $user.UserPrincipalName  " ObjectId: "  $user.ObjectId -ForegroundColor Yellow

    $hasMfaRelyingParty = $false
    foreach($p in $user.StrongAuthenticationRequirements)
    {
        if ($p.RelyingParty -eq "*")
        {
            $hasMfaRelyingParty = $true
            Write-Host "User was enabled for per-user MFA." -ForegroundColor Yellow
        }
    }

    if ($user.StrongAuthenticationMethods.Count -gt 0 -and -not $hasMfaRelyingParty)
    {
        Write-Host $user.UserPrincipalName " is eligible for rollback" -ForegroundColor Yellow
        Write-Host "Rolling back user ..." -ForegroundColor Yellow
        Reset-MsolStrongAuthenticationMethodByUpn -UserPrincipalName $user.UserPrincipalName
        Write-Host "Successfully rolled back user " $user.UserPrincipalName -ForegroundColor Green
    }
    else
    {
        Write-Host $user.UserPrincipalName " is not eligible for rollback. No action required."
    }

    Write-Host ""
    Start-Sleep -Milliseconds 750
}

# Connect
Import-Module MSOnline
Connect-MsolService

foreach($line in Get-Content $path)
{
    RemediateUser -ObjectId $line
}
```

### <a name="rollback"></a>復原

在 PowerShell 視窗中，在更新反白顯示的位置之後執行下列命令。 出現提示時，輸入全域管理員認證。 該指令碼將輸出每個使用者更新作業的結果。

`<script location> -path <user file location>`

## <a name="disable-preview-experience"></a>停用預覽體驗

若要停用使用者的預覽體驗，請完成下列步驟：

1. 以全域系統管理員或使用者系統管理員身分登入 Azure 入口網站。
2. 瀏覽至 [Azure Active Directory]、[使用者設定]、[管理存取面板預覽功能的設定]。
3. 在 [使用者可以使用預覽功能註冊與管理安全性資訊] 底下，將選取器設定為 [無]，然後按一下 [儲存]。

將不再提示使用者使用預覽體驗進行註冊。

## <a name="next-steps"></a>後續步驟

[深入了解自助密碼重設和 Azure Multi-Factor Authentication 的聚合式註冊的公開預覽](concept-registration-mfa-sspr-converged.md)