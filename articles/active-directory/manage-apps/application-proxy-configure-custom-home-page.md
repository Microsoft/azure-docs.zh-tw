---
title: 使用 Azure AD 應用程式 Proxy 為發佈的應用程式設定自訂首頁 | Microsoft Docs
description: 涵蓋 Azure AD 應用程式 Proxy 連接器的基本概念
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/08/2017
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: 10bcb3c0c4842202f95cdc1fff30d12b7a8fbbc2
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39366018"
---
# <a name="set-a-custom-home-page-for-published-apps-by-using-azure-ad-application-proxy"></a>使用 Azure AD 應用程式 Proxy 為發佈的應用程式設定自訂首頁

本文討論如何設定應用程式來將使用者導向自訂首頁。 當您使用應用程式 Proxy 發佈應用程式時，您會設定內部 URL，但有時這不是您的使用者應該先看到的頁面。 設定自訂首頁，讓使用者在存取應用程式時能前往正確的頁面。 不論使用者從 Azure Active Directory 存取面板或 Office 365 應用程式啟動器存取應用程式，都能看到您設定的自訂首頁。

當使用者啟動應用程式時，預設會將他們導向已發佈應用程式的根網域 URL。 登陸頁面通常設定為首頁 URL。 如果您想要讓應用程式使用者登陸應用程式內的特定頁面，請使用 Azure AD PowerShell 模組定義自訂首頁 URL。 

公司為什麼要設定自訂首頁，以下即是一例：
- 在您的公司網路內部，使用者會前往 *https://ExpenseApp/login/login.aspx*，以登入並存取您的應用程式。
- 由於您有應用程式 Proxy 需要在資料夾結構最上層存取的其他資產 (例如影像)，因此會以 *https://ExpenseApp* 作為內部 URL 來發佈應用程式。
- 預設外部 URL 是 *https://ExpenseApp-contoso.msappproxy.net*，其不會將使用者帶往登入頁面。  
- 設定 *https://ExpenseApp-contoso.msappproxy.net/login/login.aspx* 作為首頁 URL。 

>[!NOTE]
>當您將已發佈應用程式的存取權提供給使用者時，應用程式會顯示在 [Azure AD 存取面板](../user-help/active-directory-saas-access-panel-introduction.md)和 [Office 365 應用程式啟動器](https://blogs.office.com/2016/09/27/introducing-the-new-office-365-app-launcher)。

## <a name="before-you-start"></a>開始之前

設定首頁 URL 之前，請記住下列需求︰

* 確保您指定的路徑是根網域 URL 的子網域路徑。

  舉例來說，如果根網域 URL 為 https://apps.contoso.com/app1/，則您所設定的首頁 URL 開頭必須是 https://apps.contoso.com/app1/。

* 若變更已發佈的應用程式，則變更可能會重設首頁 URL 的值。 當您未來更新應用程式時，您應該重新檢查並視需要更新首頁 URL。

## <a name="change-the-home-page-in-the-azure-portal"></a>在 Azure 入口網站中變更首頁

1. 以系統管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至 [Azure Active Directory] > [應用程式註冊]，然後從清單中選擇您的應用程式。 
3. 從設定中選取 [屬性]。
4. 使用新路徑更新 [首頁 URL] 欄位。 

   ![提供新的首頁 URL](./media/application-proxy-configure-custom-home-page/homepage.png)

5. 選取 [儲存]。

## <a name="change-the-home-page-with-powershell"></a>使用 PowerShell 變更首頁

### <a name="install-the-azure-ad-powershell-module"></a>安裝 Azure AD PowerShell 模組

使用 PowerShell 定義自訂首頁 URL 之前，請先安裝 Azure AD PowerShell 模組。 您可以從 [PowerShell 資源庫](https://www.powershellgallery.com/packages/AzureAD/2.0.0.131)下載此套件，其使用圖形 API 端點。 

若要安裝套件，請遵循下列步驟：

1. 開啟標準 PowerShell 視窗，然後執行下列命令：

    ```
     Install-Module -Name AzureAD
    ```
    若您以非系統管理員身分執行此命令，請使用 `-scope currentuser` 選項。
2. 在安裝期間，選取 [Y] 以從 Nuget.org 安裝兩個套件。兩個套件都是必要套件。 

### <a name="find-the-objectid-of-the-app"></a>尋找應用程式的 ObjectID

取得應用程式中的 ObjectID，然後依其首頁搜尋應用程式。

1. 在相同的 PowerShell 視窗中，匯入 Azure AD 模組。

    ```
    Import-Module AzureAD
    ```

2. 以租用戶系統管理員身分登入 Azure AD 模組。

    ```
    Connect-AzureAD
    ```
3. 根據應用程式的首頁 URL 尋找它。 您可以在入口網站中找到 URL，請前往 [Azure Active Directory] > [企業應用程式] > [所有應用程式]。 這個範例會使用 sharepoint-iddemo。

    ```
    Get-AzureADApplication | where { $_.Homepage -like “sharepoint-iddemo” } | fl DisplayName, Homepage, ObjectID
    ```
4. 您應會取得如下所示的結果。 將 ObjectID GUID 複製以在下一節中使用。

    ```
    DisplayName : SharePoint
    Homepage    : https://sharepoint-iddemo.msappproxy.net/
    ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
    ```

### <a name="update-the-home-page-url"></a>更新首頁 URL

建立首頁 URL，並使用該值更新您的應用程式。 繼續使用相同的 PowerShell 視窗執行下列命令。 或者，如果您使用新的 PowerShell 視窗，請再次使用 `Connect-AzureAD` 登入 Azure AD 模組。 

1. 確認您有正確的應用程式，並將 *8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4* 取代為您在前一節複製的 ObjectID。

    ```
    Get-AzureADApplication -ObjectId 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4.
    ```

 您現在已確認應用程式，可以開始依照下列指示更新首頁。

2. 建立空白應用程式物件以存放您要進行的變更。 這個變數會包含您要更新的值。 此步驟不會建立任何項目。

    ```
    $appnew = New-Object “Microsoft.Open.AzureAD.Model.Application”
    ```

3. 將首頁 URL 設定為您想要的值。 此值必須是已發佈應用程式的子網域路徑。 例如，如果您將首頁 URL 從 *https://sharepoint-iddemo.msappproxy.net/* 變更為 *https://sharepoint-iddemo.msappproxy.net/hybrid/*，則應用程式使用者會直接前往自訂首頁。

    ```
    $homepage = “https://sharepoint-iddemo.msappproxy.net/hybrid/”
    ```
4. 使用您在「步驟 1：尋找應用程式的 ObjectID」中複製的 GUID (ObjectID)，進行更新。

    ```
    Set-AzureADApplication -ObjectId 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4 -Homepage $homepage
    ```
5. 若要確認變更成功，請重新啟動應用程式。

    ```
    Get-AzureADApplication -ObjectId 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
    ```

>[!NOTE]
>您對應用程式所做的任何變更都可能會重設首頁 URL。 如果您的首頁 URL 重設，請重複本節中的步驟重新設定。

## <a name="next-steps"></a>後續步驟

- [使用 Azure AD 應用程式 Proxy 啟用 SharePoint 的遠端存取](application-proxy-integrate-with-sharepoint-server.md)
- [在 Azure 入口網站中啟用應用程式 Proxy](application-proxy-enable.md)
