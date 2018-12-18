---
title: 在 Azure Active Directory B2C 中設定資源擁有者密碼認證流程 | Microsoft Docs
description: 了解如何在 Azure AD B2C 中設定資源擁有者密碼認證流程。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/07/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 519f13d668f09fb2d83e8f64767e195e2544fc1e
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43343281"
---
# <a name="configure-the-resource-owner-password-credentials-flow-in-azure-ad-b2c"></a>在 Azure AD B2C 中設定資源擁有者密碼認證流程

資源擁有者密碼認證 (ROPC) 流程是一項 OAuth 標準驗證流程，在此流程中，應用程式 (也稱為信賴憑證者) 會以有效認證 (例如使用者識別碼和密碼) 交換識別碼權杖、存取權杖和重新整理權杖。 

> [!NOTE]
> 這項功能處於預覽狀態。

Azure Active Directory (Azure AD) B2C 支援下列選項：

- **原生用戶端**：程式碼在使用者端的裝置上執行時，使用者會在驗證期間與之互動。 裝置可以是在原生作業系統 (例如 Android) 執行的行動應用程式，也可以是在瀏覽器 (例如 JavaScript) 執行的行動應用程式。
- **公用用戶端流程**：只會以 API 呼叫傳送應用程式所收集的使用者認證。 應用程式的認證不會傳送。
- **新增宣告**：可以變更識別碼權杖內容以新增宣告。 

不支援下列流程：

- **伺服器對伺服器**：身分識別保護系統必須要有從呼叫端 (原生用戶端) 收集到的可靠 IP 位址，以用於互動的過程中。 伺服器端的 API 呼叫只會使用伺服器的 IP 位址。 如果超出失敗驗證的動態閾值，身分識別保護系統可能會將重複的 IP 位址識別為攻擊者。
- **機密用戶端流程**：會驗證應用程式用戶端識別碼，但不會驗證應用程式密碼。

##  <a name="create-a-resource-owner-policy"></a>建立資源擁有者原則

1. 以 Azure AD B2C 租用戶的全域管理員身分登入 Azure 入口網站。
2. 若要切換為您的 Azure AD B2C 租用戶，請選取入口網站右上角的 B2C 租用戶。
3. 在 [原則] 下選取 [資源擁有者原則]。
4. 提供原則的名稱 (例如 *ROPC_Auth*)，然後選取 [應用程式宣告]。
5. 選取您的應用程式所需的應用程式宣告，例如 [顯示名稱]、[電子郵件地址] 和 [身分識別提供者]。
6. 選取 [確定]，然後選取 [建立]。

   然後，您會看到類似於下列範例的端點：

   `https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=B2C_1_ROPC_Auth`


## <a name="register-an-application"></a>註冊應用程式

1. 在 B2C 設定中選取 [應用程式]，然後選取 [新增]。
2. 輸入應用程式的名稱，例如 ROPC_Auth_app。
3. 針對 [Web 應用程式/Web API] 選取 [否]，然後針對 [原生用戶端] 選取 [是]。
4. 保留所有其他值，然後選取 [建立]。
5. 選取新的應用程式，並記下應用程式識別碼供稍後使用。

## <a name="test-the-policy"></a>測試原則

使用您最慣用的 API 開發應用程式產生 API 呼叫，並檢視回應以對您的原則偵錯。 使用下表中的資訊作為 POST 要求的本文，以建構與此類似的呼叫：
- 將 \<yourtenant.onmicrosoft.com> 取代為您的 B2C 租用戶名稱。
- 將 \<B2C_1A_ROPC_Auth> 取代為資源擁有者密碼認證原則的完整名稱。
- 將 \<bef2222d56-552f-4a5b-b90a-1988a7d634c3> 取代為您註冊中的應用程式識別碼。

`https://yourtenant.b2clogin.com/<yourtenant.onmicrosoft.com>/oauth2/v2.0/token?p=B2C_1_ROPC_Auth`

| Key | 值 |
| --- | ----- |
| username | leadiocl@outlook.com |
| password | Passxword1 |
| grant_type | password |
| scope | openid \<bef2222d56-552f-4a5b-b90a-1988a7d634c3> offline_access |
| client_id | \<bef2222d56-552f-4a5b-b90a-1988a7d634c3> |
| response_type | token id_token |

*Client_id* 是您先前記下的應用程式識別碼值。 *Offline_access* 是您要接收重新整理權杖時可選用的項目。 您使用的使用者名稱與密碼，必須是您 Azure AD B2C 租用戶中現有使用者的認證。

實際的 POST 要求如下所示：

```
POST /yourtenant.onmicrosoft.com/oauth2/v2.0/token?B2C_1_ROPC_Auth HTTP/1.1
Host: yourtenant.b2clogin.com
Content-Type: application/x-www-form-urlencoded

username=leadiocl%40trashmail.ws&password=Passxword1&grant_type=password&scope=openid+bef22d56-552f-4a5b-b90a-1988a7d634ce+offline_access&client_id=bef22d56-552f-4a5b-b90a-1988a7d634ce&response_type=token+id_token
```


離線存取的成功回應會類似於下列範例：

```
{ 
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ik9YQjNhdTNScWhUQWN6R0RWZDM5djNpTmlyTWhqN2wxMjIySnh6TmgwRlkifQ.eyJpc3MiOiJodHRwczovL3RlLmNwaW0ud2luZG93cy5uZXQvZjA2YzJmZTgtNzA5Zi00MDMwLTg1ZGMtMzhhNGJmZDllODJkL3YyLjAvIiwiZXhwIjoxNTEzMTMwMDc4LCJuYmYiOjE1MTMxMjY0NzgsImF1ZCI6ImJlZjIyZDU2LTU1MmYtNGE1Yi1iOTBhLTE5ODhhN2Q2MzRjZSIsIm9pZCI6IjNjM2I5NjljLThjNDktNGUxMS1hNGVmLWZkYjJmMzkyZjA0OSIsInN1YiI6Ik5vdCBzdXBwb3J0ZWQgY3VycmVudGx5LiBVc2Ugb2lkIGNsYWltLiIsImF6cCI6ImJlZjIyZDU2LTU1MmYtNGE1Yi1iOTBhLTE5ODhhN2Q2MzRjZSIsInZlciI6IjEuMCIsImlhdCI6MTUxMzEyNjQ3OH0.MSEThYZxCS4SevBw3-3ecnVLUkucFkehH-gH-P7SFcJ-MhsBeQEpMF1Rzu_R9kUqV3qEWKAPYCNdZ3_P4Dd3a63iG6m9TnO1Vt5SKTETuhVx3Xl5LYeA1i3Slt9Y7LIicn59hGKRZ8ddrQzkqj69j723ooy01amrXvF6zNOudh0acseszt7fbzzofyagKPerxaeTH0NgyOinLwXu0eNj_6RtF9gBfgwVidRy9OzXUJnqm1GdrS61XUqiIUtv4H04jYxDem7ek6E4jsH809uSXT0iD5_4C5bDHrpO1N6pXSasmVR9GM1XgfXA_IRLFU4Nd26CzGl1NjbhLnvli2qY4A", 
    "token_type": "Bearer", 
    "expires_in": "3600", 
    "refresh_token": "eyJraWQiOiJacW9pQlp2TW5pYVc2MUY0TnlfR3REVk1EVFBLbUJLb0FUcWQ1ZWFja1hBIiwidmVyIjoiMS4wIiwiemlwIjoiRGVmbGF0ZSIsInNlciI6IjEuMCJ9.aJ_2UW14dh4saWTQ0jLJ7ByQs5JzIeW_AU9Q_RVFgrrnYiPhikEc68ilvWWo8B20KTRB_s7oy_Eoh5LACsqU6Oz0Mjnh0-DxgrMblUOTAQ9dbfAT5WoLZiCBJIz4YT5OUA_RAGjhBUkqGwdWEumDExQnXIjRSeaUBmWCQHPPguV1_5wSj8aW2zIzYIMbofvpjwIATlbIZwJ7ufnLypRuq_MDbZhJkegDw10KI4MHJlJ40Ip8mCOe0XeJIDpfefiJ6WQpUq4zl06NO7j8kvDoVq9WALJIao7LYk_x9UIT-3d0W0eDBHGSRcNgtMYpymaN9ltx6djcEesXNn4CFnWG3g.y6KKeA9EcsW9zW-g.TrTSgn4WBt18gezegxihBla9SLSTC3YfDROQsL9K4yX4400FKlTlf-2l9CnpGTEdWXVi7sIMHCl8S4oUiXd-rvY2mn_NfDrbbVJfgKp1j7Nnq9FFyeJEFcP_FtUXgsNTG9iwfzWox04B1d845qNRWiS9N8BhAAAIdz5N0ChHuOxsVOC0Y_Ly3DNe-JQyXcq964M6-jp3cgi4UqMxT837L6pLY5Ih_iPsSfyHzstsFeqyUIktnzt1MpTlyW-_GDyFK1S-SyV8PPQ7phgFouw2jho1iboHX70RlDGYyVmP1CfQzKE_zWxj3rgaCZvYMWN8fUenoiatzhvWkUM7dhqKGjofPeL8rOMkhl6afLLjObzhUg3PZFcMR6guLjQdEwQFufWxGjfpvaHycZSKeWu6-7dF8Hy_nyMLLdBpUkdrXPob_5gRiaH72KvncSIFvJLqhY3NgXO05Fy87PORjggXwYkhWh4FgQZBIYD6h0CSk2nfFjR9uD9EKiBBWSBZj814S_Jdw6HESFtn91thpvU3hi3qNOi1m41gg1vt5Kh35A5AyDY1J7a9i_lN4B7e_pknXlVX6Z-Z2BYZvwAU7KLKsy5a99p9FX0lg6QweDzhukXrB4wgfKvVRTo.mjk92wMk-zUSrzuuuXPVeg", 
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ik9YQjNhdTNScWhUQWN6R0RWZDM5djNpTmlyTWhqN2wxMjIySnh6TmgwRlkifQ.eyJleHAiOjE1MTMxMzAwNzgsIm5iZiI6MTUxMzEyNjQ3OCwidmVyIjoiMS4wIiwiaXNzIjoiaHR0cHM6Ly90ZS5jcGltLndpbmRvd3MubmV0L2YwNmMyZmU4LTcwOWYtNDAzMC04NWRjLTM4YTRiZmQ5ZTgyZC92Mi4wLyIsInN1YiI6Ik5vdCBzdXBwb3J0ZWQgY3VycmVudGx5LiBVc2Ugb2lkIGNsYWltLiIsImF1ZCI6ImJlZjIyZDU2LTU1MmYtNGE1Yi1iOTBhLTE5ODhhN2Q2MzRjZSIsImFjciI6ImIyY18xYV9yZXNvdXJjZW93bmVydjIiLCJpYXQiOjE1MTMxMjY0NzgsImF1dGhfdGltZSI6MTUxMzEyNjQ3OCwib2lkIjoiM2MzYjk2OWMtOGM0OS00ZTExLWE0ZWYtZmRiMmYzOTJmMDQ5IiwiYXRfaGFzaCI6Ikd6QUNCTVJtcklwYm9OdkFtNHhMWEEifQ.iAJg13cgySsdH3cmoEPGZB_g-4O8KWvGr6W5VzRXtkrlLoKB1pl4hL6f_0xOrxnQwj2sUgW-wjsCVzMc_dkHSwd9QFZ4EYJEJbi1LMGk2lW-PgjsbwHPDU1mz-SR1PeqqJgvOqrzXo0YHXr-e07M4v4Tko-i_OYcrdJzj4Bkv7ZZilsSj62lNig4HkxTIWi5Ec2gD79bPKzgCtIww1KRnwmrlnCOrMFYNj-0T3lTDcXAQog63MOacf7OuRVUC5k_IdseigeMSscrYrNrH28s3r0JoqDhNUTewuw1jx0X6gdqQWZKOLJ7OF_EJMP-BkRTixBGK5eW2YeUUEVQxsFlUg" 
} 
```

## <a name="redeem-a-refresh-token"></a>兌換重新整理權杖

使用下表中的資訊作為要求的本文，以建構與此處所示範例類似的 POST 呼叫：

`https://yourtenant.b2clogin.com/<yourtenant.onmicrosoft.com>/oauth2/v2.0/token?p=B2C_1_ROPC_Auth`

| Key | 值 |
| --- | ----- |
| grant_type | refresh_token |
| response_type | id_token |
| client_id | \<bef2222d56-552f-4a5b-b90a-1988a7d634c3> |
| resource | \<bef2222d56-552f-4a5b-b90a-1988a7d634c3> |
| refresh_token | eyJraWQiOiJacW9pQlp2TW5pYVc2MUY0TnlfR3... |

*Client_id* 和 *resource* 是您先前記下的應用程式識別碼值。 *Refresh_token* 是您在先前所述的驗證呼叫中接收到的權杖。

成功的回應看起來如以下範例：

```
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrNHh5b2pORnVtMWtsMll0djhkbE5QNC1jNTdkTzZRR1RWQndhTmsifQ.eyJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vNTE2ZmMwNjUtZmYzNi00YjkzLWFhNWEtZDZlZWRhN2NiYWM4L3YyLjAvIiwiZXhwIjoxNTMzNjc2NTkwLCJuYmYiOjE1MzM2NzI5OTAsImF1ZCI6IjljNTA2MThjLWY5NTEtNDlhNS1iZmU1LWQ3ODA4NTEyMWMzYSIsImlkcCI6IkxvY2FsQWNjb3VudCIsInN1YiI6ImJmZDgwODBjLTBjNDAtNDNjYS05ZTI3LTUyZTAyNzIyNWYyMSIsIm5hbWUiOiJEYXZpZE11IiwiZW1haWxzIjpbImRhdmlkd20xMDMwQGhvdG1haWwuY29tIl0sInRmcCI6IkIyQ18xX1JPUENfQXV0aCIsImF6cCI6IjljNTA2MThjLWY5NTEtNDlhNS1iZmU1LWQ3ODA4NTEyMWMzYSIsInZlciI6IjEuMCIsImlhdCI6MTUzMzY3Mjk5MH0.RULWeBR8--s5cCGG6XOi8m-AGyCaASx9W5B3tNUQjbVkHnGdo2_OUrnVoOZ1PTcrc1b0PQM2kVWi7NpYn57ifnqL_feTJPDbj9FJ8BmyxULdoECWxSM6KHsOPWZOIg5y1lNwN_IQ2HNF6UaDyYf1ZIM-jHr-uSfUnQXyWRnGDwNKX7TQbFmFk4oFMbPxTE7ioWAmxSnroiiB4__P9D0rUM1vf_qfzemf2ErIWSF9rGtCNBG-BvJlr3ZMCxIhRiIWNM2bVY0i3Nprzj0V8_FM6q8U19bvg9yDEzUcbe_1PMqzP3IrXW9N1XvQHupsOj8Keb7SmpgY1GG091X6wBCypw",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrNHh5b2pORnVtMWtsMll0djhkbE5QNC1jNTdkTzZRR1RWQndhTmsifQ.eyJleHAiOjE1MzM2NzY1OTAsIm5iZiI6MTUzMzY3Mjk5MCwidmVyIjoiMS4wIiwiaXNzIjoiaHR0cHM6Ly9sb2dpbi5taWNyb3NvZnRvbmxpbmUuY29tLzUxNmZjMDY1LWZmMzYtNGI5My1hYTVhLWQ2ZWVkYTdjYmFjOC92Mi4wLyIsInN1YiI6ImJmZDgwODBjLTBjNDAtNDNjYS05ZTI3LTUyZTAyNzIyNWYyMSIsImF1ZCI6IjljNTA2MThjLWY5NTEtNDlhNS1iZmU1LWQ3ODA4NTEyMWMzYSIsImlhdCI6MTUzMzY3Mjk5MCwiYXV0aF90aW1lIjoxNTMzNjcyOTkwLCJpZHAiOiJMb2NhbEFjY291bnQiLCJuYW1lIjoiRGF2aWRNdSIsImVtYWlscyI6WyJkYXZpZHdtMTAzMEBob3RtYWlsLmNvbSJdLCJ0ZnAiOiJCMkNfMV9ST1BDX0F1dGgiLCJhdF9oYXNoIjoiYW5hZ3QtX1NveUtBQV9UNFBLaHN4dyJ9.bPzpUFh94XFHXC_yR6qH_Unf6_hN-9-BjDXOzrdb1AuoU6-owQ3fWDxNBUbYEPALid3sgm4qhJ6BROFKryD8aWfrNyaErnYZwZ6rliHk4foa3JsbDgM3yNGPL0hzOFpC4Y9QhUjNgQOxvnQLtqbHVNonSvBc7VVPAjBDza44GowmvLORfJ1qkTjdrFM75HlLVeQch8cUNf-Ova77JdG5WHgYgqRhAq1OhV68YgEpQkARyz77zbAz9zZEHZZlgsli8UV6C-CPcmoHbwS-85mLzF9nLxhzjgIXJwckB6I7lvTpfuRtaqZIb3pMYeHZJaxaNLDvq9Qe4N-danXABg1B2w",
    "token_type": "Bearer",
    "not_before": 1533672990,
    "expires_in": 3600,
    "expires_on": 1533676590,
    "resource": "bef2222d56-552f-4a5b-b90a-1988a7d634c3",
    "id_token_expires_in": 3600,
    "profile_info": "eyJ2ZXIiOiIxLjAiLCJ0aWQiOiI1MTZmYzA2NS1mZjM2LTRiOTMtYWE1YS1kNmVlZGE3Y2JhYzgiLCJzdWIiOm51bGwsIm5hbWUiOiJEYXZpZE11IiwicHJlZmVycmVkX3VzZXJuYW1lIjpudWxsLCJpZHAiOiJMb2NhbEFjY291bnQifQ",
    "refresh_token": "eyJraWQiOiJjcGltY29yZV8wOTI1MjAxNSIsInZlciI6IjEuMCIsInppcCI6IkRlZmxhdGUiLCJzZXIiOiIxLjAifQ..8oC4Q6aKdr35yMWm.p43lns-cfWNFbtmrhvtssQXCItb3E9aSLafZJ6nKnnpXGQ-ZapOOyH7hPK7AN_RT7NMsQwNdy0Fyv_hOMrFbMPZNvHSa91RsQIvBZ73-CVy0HNF0grSezjCATg4NVHfricuQVegEmZKFOoNP6TaMC2kIlEi3rhrrO8VE3ZFQ3Jjo6j91BJaE9ybb02HWOoKqlzHiazwQyUHujw_R0TyXaQCI_gtLARr5QUXm7hlAfHhxR9uewQKlRbeuMH8nCMLSMASCJyzfeSJTjXmA0F0VrXozrqzOJdyy0EETPR7oA48MJ9l6C2sy2ZELkqpOM3xhbhV-Re7nM09b8DeWuCw7VNTcQc9DKnIHDR-H5U2Tc-lMJQadgUNZv7KGSRGTyprWb7wF7FEPnRNID5PCDV_N_yoQpI7VvJO_NotXEgHFo7OHs5Gsgwpl5mrDtymYzIMM7onTflOlu46em_qltji7xcWNOuHq4AeOlcY9ZythZgJH7livljReTwyX8QuUwpomXVEUGDc5pAnvgSozxnUbM7AlwfUeJZRT45P7L7683RSqChdNxiQk0sXUECqxnFxMAz4VUzld2yFe-pzvxFF4_feQjBEmSCAvekpvJUrEticEs4QzByV5UZ2ZCKccijFTg4doACiCo_z13JTm47mxm-5jUhXOQqiL69oxztk.KqI-z2LlC77lvwqmeFtdGQ",
    "refresh_token_expires_in": 1209600
}
```

## <a name="implement-with-your-preferred-native-sdk-or-use-app-auth"></a>使用您慣用的原生 SDK 或 App-Auth 進行實作

Azure AD B2C 實作符合公用用戶端資源擁有者密碼認證的 OAuth 2.0 標準，且應與大部分的用戶端 SDK 相容。 我們已在生產環境中使用 AppAuth for iOS 和 AppAuth for Android 多方面測試此流程。 如需最新資訊，請參閱[實作現代化最佳做法的 OAuth 2.0 和 OpenID Connect 所適用的原生應用程式 SDK](https://appauth.io/)。

從 GitHub 下載已設定用於 Azure AD B2C 的實用範例 ([Android 版](https://aka.ms/aadb2cappauthropc)和 [iOS 版](https://aka.ms/aadb2ciosappauthropc))。




