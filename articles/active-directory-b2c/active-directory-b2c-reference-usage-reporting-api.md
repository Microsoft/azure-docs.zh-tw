---
title: Azure Active Directory B2C 中的使用方式報告 API 範例和定義 | Microsoft Docs
description: 取得關於 Azure AD B2C 租用戶使用者、驗證及多重要素驗證報告的相關指南和範例。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.date: 08/04/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 544b0618f9135b684846c42bb7edeb37cf599883
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37445529"
---
# <a name="accessing-usage-reports-in-azure-ad-b2c-via-the-reporting-api"></a>透過報告 API 存取 Azure AD B2C 中的使用報告

Azure Active Directory B2C (Azure AD B2C) 會根據使用者登入和 Azure Multi-Factor Authentication 提供驗證。 驗證可跨識別提供者提供給應用程式系列的使用者。 當您知道已在租用戶中註冊的使用者數目、使用者用來註冊的提供者，以及不同類型的驗證次數時，就能以如下方式回答問題：
* 過去 10 天內各類型識別提供者 (例如，Microsoft 或 LinkedIn 帳戶) 註冊的使用者數目？
* 上個月使用 Multi-Factor Authentication 成功完成多少次驗證？
* 這個月已完成幾次登入式驗證？ 依每一天計算？ 依每一應用程式計算？
* 如何評估我的 Azure AD B2C 租用戶活動的每月預期成本？

本文章著重於與計費活動有關的報告，而計費活動是以使用者數目、可計費的登入式驗證及多重要素驗證為根據。


## <a name="prerequisites"></a>先決條件
開始之前，您必須先完成[存取 Azure AD 報告 API 的必要條件](https://azure.microsoft.com/documentation/articles/active-directory-reporting-api-getting-started/)中的步驟。 建立應用程式、取得密碼，並授與對您的 Azure AD B2C 租用戶報告的存取權限。 這裡也提供了 Bash 指令碼和 Python 指令碼的範例。 

## <a name="powershell-script"></a>PowerShell 指令碼
此指令碼示範如何使用 `TimeStamp` 參數和 `ApplicationId` 篩選器來建立四份使用報告。

```powershell
# This script will require the Web Application and permissions setup in Azure Active Directory

# Constants
$ClientID      = "your-client-application-id-here"  
$ClientSecret  = "your-client-application-secret-here"
$loginURL      = "https://login.microsoftonline.com"
$tenantdomain  = "your-b2c-tenant-domain.onmicrosoft.com"  
# Get an Oauth 2 access token based on client id, secret and tenant domain
$body          = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
$oauth         = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=1.0 -Body $body
if ($oauth.access_token -ne $null) {
    $headerParams  = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}

    Write-host Data from the tenantUserCount report
    Write-host ====================================================
     # Returns a JSON document for the report
    $myReport = (Invoke-WebRequest -Headers $headerParams -Uri "https://graph.windows.net/$tenantdomain/reports/tenantUserCount?api-version=beta")
    Write-host $myReport.Content

    Write-host Data from the tenantUserCount report with datetime filter
    Write-host ====================================================
    $myReport = (Invoke-WebRequest -Headers $headerParams -Uri "https://graph.windows.net/$tenantdomain/reports/tenantUserCount?%24filter=TimeStamp+gt+2016-10-15&api-version=beta")
    Write-host $myReport.Content

    Write-host Data from the b2cAuthenticationCountSummary report
    Write-host ====================================================
    $myReport = (Invoke-WebRequest -Headers $headerParams -Uri "https://graph.windows.net/$tenantdomain/reports/b2cAuthenticationCountSummary?api-version=beta")
    Write-host $myReport.Content

    Write-host Data from the b2cAuthenticationCount report with datetime filter
    Write-host ====================================================
    $myReport = (Invoke-WebRequest -Headers $headerParams -Uri "https://graph.windows.net/$tenantdomain/reports/b2cAuthenticationCount?%24filter=TimeStamp+gt+2016-09-20+and+TimeStamp+lt+2016-10-03&api-version=beta")
    Write-host $myReport.Content

    Write-host Data from the b2cAuthenticationCount report with ApplicationId filter
    Write-host ====================================================
    # Returns a JSON document for the " " report
        $myReport = (Invoke-WebRequest -Headers $headerParams -Uri "https://graph.windows.net/$tenantdomain/reports/b2cAuthenticationCount?%24filter=ApplicationId+eq+ada78934-a6da-4e69-b816-10de0d79db1d&api-version=beta")
    Write-host $myReport.Content

    Write-host Data from the b2cMfaRequestCountSummary
    Write-host ====================================================
    $myReport = (Invoke-WebRequest -Headers $headerParams -Uri "https://graph.windows.net/$tenantdomain/reports/b2cMfaRequestCountSummary?api-version=beta")
    Write-host $myReport.Content

    Write-host Data from the b2cMfaRequestCount report with datetime filter
    Write-host ====================================================
    $myReport = (Invoke-WebRequest -Headers $headerParams -Uri "https://graph.windows.net/$tenantdomain/reports/b2cMfaRequestCount?%24filter=TimeStamp+gt+2016-09-10+and+TimeStamp+lt+2016-10-04&api-version=beta")
    Write-host $myReport.Content

    Write-host Data from the b2cMfaRequestCount report with ApplicationId filter
    Write-host ====================================================
    $myReport = (Invoke-WebRequest -Headers $headerParams -Uri "https://graph.windows.net/$tenantdomain/reports/b2cMfaRequestCountSummary?%24filter=ApplicationId+eq+ada78934-a6da-4e69-b816-10de0d79db1d&api-version=beta")
     Write-host $myReport.Content

} else {
    Write-Host "ERROR: No Access Token"
    }
```


## <a name="usage-report-definitions"></a>使用報告定義
* **tenantUserCount**：過去 30 天，租用戶中每一天各類型識別提供者的使用者數目。 (`TimeStamp` 篩選器可選擇性提供從指定日期至目前日期的使用者計數)。 此報告提供：
  * **TotalUserCount**：所有使用者物件的數目。
  * **OtherUserCount**：Azure Active Directory 使用者 (而非 Azure AD B2C 使用者) 的數目。
  * **LocalUserCount**：Azure AD B2C 使用者帳戶的數目 (這些帳戶是以 Azure AD B2C 租用戶本機的認證所建立)。

* **AlternateIdUserCount**：已向外部識別提供者 (例如 Facebook、Microsoft 帳戶或另一個 Azure Active Directory 租用戶，亦稱為 `OrgId`) 註冊的 Azure AD B2C 使用者數目。

* **b2cAuthenticationCountSummary**：過去 30 天，每天及各類型驗證流程的每日可計費驗證次數摘要。

* **b2cAuthenticationCount**：某段時間週期內的驗證次數。 預設值為過去 30 天。  (選擇性：開頭和結尾的 `TimeStamp` 參數會定義特定的期間)。輸出包括 `StartTimeStamp` (此租用戶的最早活動日期) 和 `EndTimeStamp` (最新的更新)。

* **b2cMfaRequestCountSummary**：依日期及類型 (SMS 或語音) 區分的每日多重要素驗證次數摘要。


## <a name="limitations"></a>限制
使用者計數的資料每隔 24 到 48 小時會重新整理。 驗證則是每天更新數次。 使用 `ApplicationId` 篩選器時，空白報告回應是因為發生下列其中一種情況：
  * 應用程式識別碼不存在於租用戶中。 請確定它是正確的。
  * 應用程式識別碼存在，但報告期間內找不到任何資料。 請檢閱您的日期/時間參數。


## <a name="next-steps"></a>後續步驟
### <a name="monthly-bill-estimates-for-azure-ad"></a>評估 Azure AD 的每月帳單
搭配[最新 Azure AD B2C 價格](https://azure.microsoft.com/pricing/details/active-directory-b2c/)，您可以評估每日、每週、每月的 Azure 耗用量。  當您規劃的租用戶行為變更可能會影響整體成本時，評估特別實用。 您可以在[連結的 Azure 訂用帳戶](active-directory-b2c-how-to-enable-billing.md)中檢閱實際成本。

### <a name="options-for-other-output-formats"></a>其他輸出格式的選項
下列程式碼示範將輸出傳送至 JSON、名稱值清單及 XML 的範例：
```powershell
# to output to JSON use following line in the PowerShell sample
$myReport.Content | Out-File -FilePath name-your-file.json -Force

# to output the content to a name value list
($myReport.Content | ConvertFrom-Json).value | Out-File -FilePath name-your-file.txt -Force

# to output the content in XML use the following line
(($myReport.Content | ConvertFrom-Json).value | ConvertTo-Xml).InnerXml | Out-File -FilePath name-your-file.xml -Force
```
