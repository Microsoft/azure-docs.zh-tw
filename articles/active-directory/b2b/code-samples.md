---
title: Azure Active Directory B2B 共同作業程式碼與 PowerShell 範例 | Microsoft Docs
description: Azure Active Directory B2B 共同作業的程式碼與 PowerShell 範例
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: sample
ms.date: 04/11/2017
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 9d2fa8ec96f629ff85d785897398bc9af52ab3bc
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49165754"
---
# <a name="azure-active-directory-b2b-collaboration-code-and-powershell-samples"></a>Azure Active Directory B2B 共同作業程式碼與 PowerShell 範例

## <a name="powershell-example"></a>PowerShell 範例
您可以從您儲存在 .CSV 檔案中的電子郵件地址大量邀請外部使用者。

1. 準備 .CSV 檔案 建立新的 CSV 檔案並將它命名為 invitations.csv。 在此範例中，檔案會儲存在 C:\data 中，並且包含下列資訊：
  
  名稱                  |  InvitedUserEmailAddress
  --------------------- | --------------------------
  Gmail B2B 受邀者     | b2binvitee@gmail.com
  Outlook B2B 受邀者   | b2binvitee@outlook.com


2. 取得最新的 Azure AD PowerShell。若要使用新的 Cmdlet，您必須安裝已更新的 Azure AD PowerShell 模組 (可從 [Powershell 模組的發行頁面](https://www.powershellgallery.com/packages/AzureADPreview)下載)

3. 登入您的租用戶

    ```powershell
    $cred = Get-Credential
    Connect-AzureAD -Credential $cred
    ```

4. 執行 PowerShell Cmdlet

  ```powershell
  $invitations = import-csv C:\data\invitations.csv
  $messageInfo = New-Object Microsoft.Open.MSGraph.Model.InvitedUserMessageInfo
  $messageInfo.customizedMessageBody = "Hey there! Check this out. I created an invitation through PowerShell"
  foreach ($email in $invitations) {New-AzureADMSInvitation -InvitedUserEmailAddress $email.InvitedUserEmailAddress -InvitedUserDisplayName $email.Name -InviteRedirectUrl https://wingtiptoysonline-dev-ed.my.salesforce.com -InvitedUserMessageInfo $messageInfo -SendInvitationMessage $true}
  ```

此 Cmdlet 會傳送邀請給 invitations.csv 中的電子郵件地址。 此 Cmdlet 的其他功能包括︰
- 自訂電子郵件訊息中的文字
- 包括受邀使用者的顯示名稱
- 傳送訊息給 CC 或完全隱藏電子郵件訊息

## <a name="code-sample"></a>程式碼範例
此處示範如何以僅限應用程式模式呼叫邀請 API，以取得您要邀請 B2B 使用者存取之資源的兌換 URL。 目標是傳送自訂邀請電子郵件。 電子郵件可使用 HTTP 用戶端來編寫，因此您可以自訂其外觀並透過圖形 API 傳送。

```csharp
namespace SampleInviteApp
{
    using System;
    using System.Linq;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    class Program
    {
        /// <summary>
        /// Microsoft graph resource.
        /// </summary>
        static readonly string GraphResource = "https://graph.microsoft.com";
 
        /// <summary>
        /// Microsoft graph invite endpoint.
        /// </summary>
        static readonly string InviteEndPoint = "https://graph.microsoft.com/v1.0/invitations";
 
        /// <summary>
        ///  Authentication endpoint to get token.
        /// </summary>
        static readonly string EstsLoginEndpoint = "https://login.microsoftonline.com";
 
        /// <summary>
        /// This is the tenantid of the tenant you want to invite users to.
        /// </summary>
        private static readonly string TenantID = "";
 
        /// <summary>
        /// This is the application id of the application that is registered in the above tenant.
        /// The required scopes are available in the below link.
        /// https://developer.microsoft.com/graph/docs/api-reference/v1.0/api/invitation_post
        /// </summary>
        private static readonly string TestAppClientId = "";
 
        /// <summary>
        /// Client secret of the application.
        /// </summary>
        private static readonly string TestAppClientSecret = @"";
 
        /// <summary>
        /// This is the email address of the user you want to invite.
        /// </summary>
        private static readonly string InvitedUserEmailAddress = @"";
 
        /// <summary>
        /// This is the display name of the user you want to invite.
        /// </summary>
        private static readonly string InvitedUserDisplayName = @"";
 
        /// <summary>
        /// Main method.
        /// </summary>
        /// <param name="args">Optional arguments</param>
        static void Main(string[] args)
        {
            Invitation invitation = CreateInvitation();
            SendInvitation(invitation);
        }
 
        /// <summary>
        /// Create the invitation object.
        /// </summary>
        /// <returns>Returns the invitation object.</returns>
        private static Invitation CreateInvitation()
        {
            // Set the invitation object.
            Invitation invitation = new Invitation();
            invitation.InvitedUserDisplayName = InvitedUserDisplayName;
            invitation.InvitedUserEmailAddress = InvitedUserEmailAddress;
            invitation.InviteRedirectUrl = "https://www.microsoft.com";
            invitation.SendInvitationMessage = true;
            return invitation;
        }
 
        /// <summary>
        /// Send the guest user invite request.
        /// </summary>
        /// <param name="invitation">Invitation object.</param>
        private static void SendInvitation(Invitation invitation)
        {
            string accessToken = GetAccessToken();
 
            HttpClient httpClient = GetHttpClient(accessToken);
 
            // Make the invite call. 
            HttpContent content = new StringContent(JsonConvert.SerializeObject(invitation));
            content.Headers.Add("ContentType", "application/json");
            var postResponse = httpClient.PostAsync(InviteEndPoint, content).Result;
            string serverResponse = postResponse.Content.ReadAsStringAsync().Result;
            Console.WriteLine(serverResponse);
        }
 
        /// <summary>
        /// Get the HTTP client.
        /// </summary>
        /// <param name="accessToken">Access token</param>
        /// <returns>Returns the Http Client.</returns>
        private static HttpClient GetHttpClient(string accessToken)
        {
            // setup http client.
            HttpClient httpClient = new HttpClient();
            httpClient.Timeout = TimeSpan.FromSeconds(300);
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
            httpClient.DefaultRequestHeaders.Add("client-request-id", Guid.NewGuid().ToString());
            Console.WriteLine(
                "CorrelationID for the request: {0}",
                httpClient.DefaultRequestHeaders.GetValues("client-request-id").Single());
            return httpClient;
        }
 
        /// <summary>
        /// Get the access token for our application to talk to microsoft graph.
        /// </summary>
        /// <returns>Returns the access token for our application to talk to microsoft graph.</returns>
        private static string GetAccessToken()
        {
            string accessToken = null;
 
            // Get the access token for our application to talk to microsoft graph.
            try
            {
                AuthenticationContext testAuthContext =
                    new AuthenticationContext(string.Format("{0}/{1}", EstsLoginEndpoint, TenantID));
                AuthenticationResult testAuthResult = testAuthContext.AcquireTokenAsync(
                    GraphResource,
                    new ClientCredential(TestAppClientId, TestAppClientSecret)).Result;
                accessToken = testAuthResult.AccessToken;
            }
            catch (AdalException ex)
            {
                Console.WriteLine("An exception was thrown while fetching the token: {0}.", ex);
                throw;
            }
 
            return accessToken;
        }
 
        /// <summary>
        /// Invitation class.
        /// </summary>
        public class Invitation
        {
            /// <summary>
            /// Gets or sets display name.
            /// </summary>
            public string InvitedUserDisplayName { get; set; }
 
            /// <summary>
            /// Gets or sets display name.
            /// </summary>
            public string InvitedUserEmailAddress { get; set; }
 
            /// <summary>
            /// Gets or sets a value indicating whether Invitation Manager should send the email to InvitedUser.
            /// </summary>
            public bool SendInvitationMessage { get; set; }
 
            /// <summary>
            /// Gets or sets invitation redirect URL
            /// </summary>
            public string InviteRedirectUrl { get; set; }
        }
    }
}
```


## <a name="next-steps"></a>後續步驟

- [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)

