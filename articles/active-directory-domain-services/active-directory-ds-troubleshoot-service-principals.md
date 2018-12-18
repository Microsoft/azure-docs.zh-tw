---
title: Azure Active Directory Domain Services：針對服務主體設定進行疑難排解 | Microsoft Docs
description: 針對 Azure AD Domain Services 的服務主體設定進行疑難排解
services: active-directory-ds
documentationcenter: ''
author: eringreenlee
manager: ''
editor: ''
ms.assetid: f168870c-b43a-4dd6-a13f-5cfadc5edf2c
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: ergreenl
ms.openlocfilehash: 5bc1212cc6e894cd82a60abb42f92893c0bb2d43
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39579539"
---
# <a name="troubleshoot-invalid-service-principal-configuration-for-your-managed-domain"></a>針對受控網域的無效服務主體組態進行疑難排解

本文將協助您針對導致下列警示訊息的服務主體相關設定錯誤進行疑難排解：

## <a name="alert-aadds102-service-principal-not-found"></a>警示 AADDS102：找不到服務主體

**警示訊息：***Azure AD 目錄中已刪除要讓 Azure AD Domain Services 正常運作所需的服務主體。此設定會影響 Microsoft 監視、管理、修補及同步處理受控網域的能力。*

[服務主體](../active-directory/develop/app-objects-and-service-principals.md)是 Microsoft 用來管理、更新和維護受控網域的應用程式。 如果將它們刪除，就會使 Microsoft 無法為網域提供服務。


## <a name="check-for-missing-service-principals"></a>檢查是否有遺失的服務主體
請使用下列步驟來判斷必須重新建立的服務主體：

1. 在 Azure 入口網站中，瀏覽至[企業應用程式 - 所有應用程式](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps)頁面。
2. 在 [顯示] 下拉式清單中，選取 [所有應用程式]，然後按一下 [套用]。
3. 使用下表，將應用程式識別碼貼到搜尋方塊然後按 Enter，以搜尋各個應用程式識別碼。 如果搜尋結果是空的，您就必須遵循「解決方案」資料行中的步驟來重新建立服務主體。

| 應用程式識別碼 | 解決方案 |
| :--- | :--- | :--- |
| 2565bd9d-da50-47d4-8b85-4c97f669dc36 | [使用 PowerShell 重新建立遺失的服務主體](#recreate-a-missing-service-principal-with-powershell) |
| 443155a6-77f3-45e3-882b-22b3a8d431fb | [重新註冊 Microsoft.AAD 命名空間](#re-register-to-the-microsoft-aad-namespace-using-the-azure-portal) |
| abba844e-bc0e-44b0-947a-dc74e5d09022  | [重新註冊 Microsoft.AAD 命名空間](#re-register-to-the-microsoft-aad-namespace-using-the-azure-portal) |
| d87dcbc6-a371-462e-88e3-28ad15ec4e64 | [會自動更正的服務主體](#service-principals-that-self-correct) |

## <a name="recreate-a-missing-service-principal-with-powershell"></a>使用 PowerShell 重新建立遺失的服務主體
如果 Azure AD 目錄中遺漏識別碼為 ```2565bd9d-da50-47d4-8b85-4c97f669dc36``` 的服務主體，請執行下列步驟。

**解決方案：** 您需要 Azure AD PowerShell 才能完成這些步驟。 如需安裝 Azure AD PowerShell 的相關資訊，請參閱[這篇文章](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?view=azureadps-2.0.)。

若要解決此問題，請在 PowerShell 視窗中輸入下列命令：
1. 安裝 Azure AD PowerShell 模組，並加以匯入。

    ```powershell
    Install-Module AzureAD
    Import-Module AzureAD
    ```

2. 執行下列 PowerShell 命令，檢查目錄中是否遺失 Azure AD Domain Services 所需的服務主體：

    ```powershell
    Get-AzureAdServicePrincipal -filter "AppId eq '2565bd9d-da50-47d4-8b85-4c97f669dc36'"
    ```

3. 輸入下列 PowerShell 命令，以建立服務主體：

    ```powershell
    New-AzureAdServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
    ```

4. 在遺失的服務主體建立後，請等待兩個小時再檢查受控網域的健康情況。


## <a name="re-register-to-the-microsoft-aad-namespace-using-the-azure-portal"></a>使用 Azure 入口網站重新註冊 Microsoft AAD 命名空間
如果 Azure AD 目錄中遺漏識別碼為 ```443155a6-77f3-45e3-882b-22b3a8d431fb``` 或 ```abba844e-bc0e-44b0-947a-dc74e5d09022``` 的服務主體，請執行下列步驟。

**解決方案：** 使用下列步驟，在目錄上還原 Domain Services：

1. 在 Azure 入口網站中，瀏覽至[訂用帳戶](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)頁面。
2. 從與受控網域相關聯的資料表中選擇訂用帳戶
3. 使用左側導覽，選擇 [資源提供者]
4. 在資料表中搜尋 "Microsoft.AAD"，然後按一下 [重新註冊]
5. 若要確認警示狀況已解除，請在兩小時後檢查受控網域的健康情況頁面。


## <a name="service-principals-that-self-correct"></a>會自動更正的服務主體
如果 Azure AD 目錄中遺漏識別碼為 ```d87dcbc6-a371-462e-88e3-28ad15ec4e64``` 的服務主體，請執行下列步驟。

**解決方案：** Azure AD Domain Services 可在此服務主體遺失時、設定不正確或遭到刪除時偵測到。 服務會自動重新建立此服務主體。 不過，您將必須刪除應用程式，以及使用已刪除之應用程式的物件，因為當憑證變換時，新的服務主體將無法再修改應用程式和物件。 這將在您的在網域上導致新的錯誤。 請依照 [AADDS105 的區段](#alert-aadds105-password-synchronization-application-is-out-of-date)中說明的步驟來避免此問題。 接著，請在兩個小時之後檢查受控網域的健康情況，以確認新的服務主體已重新建立。


## <a name="alert-aadds105-password-synchronization-application-is-out-of-date"></a>警示 AADDS105：密碼同步處理應用程式已過期

**警示訊息：** 應用程式識別碼為 “d87dcbc6-a371-462e-88e3-28ad15ec4e64” 的服務主體已刪除，然後重新建立。 重新建立會在為您受控網域提供服務所需的 Azure AD Domain Services 資源上留下不一致的權限。 受控網域上的密碼同步處理可能會受到影響。


**解決方案：** 您需要 Azure AD PowerShell 才能完成這些步驟。 如需安裝 Azure AD PowerShell 的相關資訊，請參閱[這篇文章](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?view=azureadps-2.0.)。

若要解決此問題，請在 PowerShell 視窗中輸入下列命令：
1. 安裝 Azure AD PowerShell 模組，並加以匯入。

    ```powershell
    Install-Module AzureAD
    Import-Module AzureAD
    ```
2. 使用下列 PowerShell 命令刪除舊的應用程式和物件

    ```powershell
    $app = Get-AzureADApplication -Filter "IdentifierUris eq 'https://sync.aaddc.activedirectory.windowsazure.com'"
    Remove-AzureADApplication -ObjectId $app.ObjectId
    $spObject = Get-AzureADServicePrincipal -Filter "DisplayName eq 'Azure AD Domain Services Sync'"
    Remove-AzureADServicePrincipal -ObjectId $app.ObjectId
    ```
3. 兩者皆刪除之後，系統將會修復本身，然後重新建立密碼同步處理所需的應用程式。 若要確認警示已修復，請在兩小時後檢查網域的健康情況。


## <a name="contact-us"></a>與我們連絡
請連絡 Azure Active Directory Domain Services 產品小組， [分享意見或尋求支援](active-directory-ds-contact-us.md)。
