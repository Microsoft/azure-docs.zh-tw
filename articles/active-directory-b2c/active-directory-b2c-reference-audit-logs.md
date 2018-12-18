---
title: Azure Active Directory B2C 中的稽核記錄範例和定義 | Microsoft Docs
description: 存取 Azure AD B2C 稽核記錄的指南和範例。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.date: 08/04/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 67e57faf37697697bee74597a40db39149699fe5
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49320232"
---
# <a name="accessing-azure-ad-b2c-audit-logs"></a>存取 Azure AD B2C 稽核記錄

Azure Active Directory B2C (Azure AD B2C) 會發出稽核記錄，其中包含有關 B2C 資源、發行的權杖和系統管理員存取的活動資訊。 本文概述透過稽核記錄可取得的資訊，以及說明如何從您的 Azure AD B2C 租用戶存取此資料。

> [!IMPORTANT]
> 稽核記錄只會保留七天。 如果您需要更長的保留期，請使用以下所示的其中一個方法來規劃下載與儲存您的記錄。 

## <a name="overview-of-activities-available-in-the-b2c-category-of-audit-logs"></a>稽核記錄 B2C 類別中可取得活動的概觀
稽核記錄中的 **B2C** 類別包含以下類型的活動：
|活動類型 |說明  |
|---------|---------|
|Authorization |有關授權使用者存取 B2C 資源的活動 (例如，系統管理員存取 B2C 原則清單)         |
|目錄 |當系統管理員使用 Azure 入口網站登入時，所擷取目錄屬性的相關活動 |
|Application | B2C 應用程式上的 CRUD 作業 |
|Key |B2C 金鑰容器所儲存金鑰上的 CRUD 作業 |
|資源 |B2C 資源 (例如，原則和身分識別提供者) 上的 CRUD 作業
|驗證 |使用者認證和權杖發行的驗證|

> [!NOTE]
> 如需使用者物件 CRUD 活動，請參閱**核心目錄**類別。

## <a name="example-activity"></a>範例活動
以下範例顯示當使用者以外部識別提供者登入時所擷取的資料：![稽核記錄 - 範例](./media/active-directory-b2c-reference-audit-logs/audit-logs-example.png)

## <a name="accessing-audit-logs-through-the-azure-portal"></a>透過 Azure 入口網站來存取稽核記錄
1. 移至 [Azure 入口網站](https://portal.azure.com)。 確定您位於 B2C 目錄中。
2. 在左邊 [我的最愛] 列中，按一下 [Azure Active Directory] 
    
    ![稽核記錄 - AAD 按鈕](./media/active-directory-b2c-reference-audit-logs/audit-logs-portal-aad.png)

1. 在 [活動] 下，按一下 [稽核記錄]

    ![稽核記錄 - 記錄區段](./media/active-directory-b2c-reference-audit-logs/audit-logs-portal-section.png)

2. 在 [類別] 投寄箱中，選取 [B2C]
3. 按一下 [套用]

    ![稽核記錄 - 類別](./media/active-directory-b2c-reference-audit-logs/audit-logs-portal-category.png)

您會看到過去七天所記錄活動的清單。 
- 使用 [活動資源類型] 下拉式清單，根據上述活動類型來進行篩選
- 使用 [日期範圍] 下拉式清單，以篩選所要顯示活動的日期範圍
- 如果您按一下清單中的特定列，右邊的關聯式方塊會顯示與該活動相關的額外屬性
- 按一下 [下載]，將活動下載為 csv 檔案

## <a name="accessing-audit-logs-through-the-azure-ad-reporting-api"></a>透過 Azure AD 報告 API 來存取稽核記錄
稽核記錄的發行管線與 Azure Active Directory 的其他活動相同，因此可以透過 [Azure Active Directory 報告 API](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-api-audit-reference) \(機器翻譯\) 來存取。 

### <a name="prerequisites"></a>必要條件
若要向 Azure AD 報告 API 驗證，需要先註冊應用程式。 請務必遵循[存取 Azure AD 報告 API 的必要條件](https://azure.microsoft.com/documentation/articles/active-directory-reporting-api-getting-started/) \(機器翻譯\) 中的步驟。

### <a name="accesing-the-api"></a>存取 API
若要透過 API 下載 Azure AD B2C 稽核記錄，建議將記錄篩選至 **B2C** 類別。 若要依類別篩選，請如下所示，在呼叫 Azure AD 報告 API 端點時使用查詢字串參數：

`https://graph.windows.net/your-b2c-tentant.onmicrosoft.com/activities/audit?api-version=beta&$filter=category eq 'B2C'`

### <a name="powershell-script"></a>PowerShell 指令碼
以下範例指令碼可以使用 PowerShell 來查詢 Azure AD 報告 API 的，並將結果儲存成 JSON 檔案：

```powershell
# This script will require registration of a Web Application in Azure Active Directory (see https://azure.microsoft.com/documentation/articles/active-directory-reporting-api-getting-started/)

# Constants
$ClientID       = "your-client-application-id-here"       # Insert your application's Client ID, a Globally Unique ID (registered by Global Admin)
$ClientSecret   = "your-client-application-secret-here"   # Insert your application's Client Key/Secret string
$loginURL       = "https://login.microsoftonline.com"     
$tenantdomain   = "your-b2c-tenant.onmicrosoft.com"       # AAD B2C Tenant; for example, contoso.onmicrosoft.com
$resource       = "https://graph.windows.net"             # Azure AD Graph API resource URI
$7daysago       = "{0:s}" -f (get-date).AddDays(-7) + "Z" # Use 'AddMinutes(-5)' to decrement minutes, for example
Write-Output "Searching for events starting $7daysago"

# Create HTTP header, get an OAuth2 access token based on client id, secret and tenant domain
$body       = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
$oauth      = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=1.0 -Body $body

# Parse audit report items, save output to file(s): auditX.json, where X = 0 thru n for number of nextLink pages
if ($oauth.access_token -ne $null) {   
    $i=0
    $headerParams = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}
    $url = 'https://graph.windows.net/' + $tenantdomain + '/activities/audit?api-version=beta&$filter=category eq ''B2C''and activityDate gt ' + $7daysago 

    # loop through each query page (1 through n)
    Do{
        # display each event on the console window
        Write-Output "Fetching data using Uri: $url"
        $myReport = (Invoke-WebRequest -UseBasicParsing -Headers $headerParams -Uri $url)
        foreach ($event in ($myReport.Content | ConvertFrom-Json).value) {
            Write-Output ($event | ConvertTo-Json)
        }

        # save the query page to an output file
        Write-Output "Save the output to a file audit$i.json"
        $myReport.Content | Out-File -FilePath audit$i.json -Force
        $url = ($myReport.Content | ConvertFrom-Json).'@odata.nextLink'
        $i = $i+1
    } while($url -ne $null)
} else {
    Write-Host "ERROR: No Access Token"
}
```

