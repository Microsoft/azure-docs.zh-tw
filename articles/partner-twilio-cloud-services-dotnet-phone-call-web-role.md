---
title: 如何從 Twilio 撥打電話 (.NET) | Microsoft Docs
description: 了解如何在 Azure 上使用 Twilio API 服務撥打電話及傳送簡訊。 程式碼範例以 .NET 撰寫。
services: ''
documentationcenter: .net
author: devinrader
manager: timlt
editor: ''
ms.assetid: 789185ad-69dc-4e9e-a936-42e0a25315c8
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 05/04/2016
ms.author: microsofthelp@twilio.com
ms.openlocfilehash: cd9792881182fbe90d9c210130ae8a34b12da363
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/08/2017
ms.locfileid: "26365999"
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-web-role-on-azure"></a>如何在 Azure 上的 Web 角色中使用 Twilio 撥打電話
本指南將說明如何從 Azure 代管的網頁上使用 Twilio 撥打電話。 產生的應用程式會提示使用者利用指定的號碼和訊息撥打電話，如下列螢幕擷取畫面所示。

![Azure call form using Twilio and ASP.NET][twilio_dotnet_basic_form]

## <a name="twilio-prereqs"></a>必要條件
您必須執行下列動作才能使用本主題中的程式碼：

1. 從 [Twilio 主控台][twilio_console]取得 Twilio 帳戶和驗證權杖。 若要開始使用 Twilio，請在 [https://www.twilio.com/try-twilio][try_twilio] 上註冊。 您可以在 [http://www.twilio.com/pricing][twilio_pricing] 上評估價格。 如需 Twilio 所提供之 API 的相關資訊，請參閱 [http://www.twilio.com/voice/api][twilio_api]。
2. 將「Twilio .NET 程式庫」新增至您的 Web 角色。 請參閱本主題稍後的「將 Twilio 程式庫新增至 Web 角色專案」。

您應知悉如何[在 Azure 上建立基本 Web 角色][azure_webroles_get_started]。

## <a name="howtocreateform"></a>作法：建立用以撥打電話的 Web 表單
<a id="use_nuget"></a>將 Twilio 程式庫新增至 Web 角色專案：

1. 在 Visual Studio 中開啟方案。
2. 以滑鼠右鍵按一下 [參考] 。
3. 按一下 [管理 NuGet 封裝] 。
4. 按一下 [線上] 。
5. 在搜尋線上方塊中，輸入 twilio。
6. 在 Twilio 套件上按一下 [安裝]  。

下列程式碼將說明如何建立 Web 表單，以擷取撥打電話所需的使用者資料。 在此範例中，會建立名為 **TwilioCloud** 的 ASP.NET Web 角色。

```aspx
<%@ Page Title="Home Page" Language="C#" MasterPageFile="~/Site.master"
    AutoEventWireup="true" CodeBehind="Default.aspx.cs"
    Inherits="WebRole1._Default" %>

<asp:Content ID="HeaderContent" runat="server" ContentPlaceHolderID="HeadContent">
</asp:Content>
<asp:Content ID="BodyContent" runat="server" ContentPlaceHolderID="MainContent">
    <div>
        <asp:BulletedList ID="varDisplay" runat="server" BulletStyle="NotSet">
        </asp:BulletedList>
    </div>
    <div>
        <p>Fill in all fields and click <b>Make this call</b>.</p>
        <div>
            To:<br /><asp:TextBox ID="toNumber" runat="server" /><br /><br />
            Message:<br /><asp:TextBox ID="message" runat="server" /><br /><br />
            <asp:Button ID="callpage" runat="server" Text="Make this call"
                onclick="callpage_Click" />
        </div>
    </div>
</asp:Content>
```

## <a id="howtocreatecode"></a>作法：建立用以撥打電話的程式碼
下列程式碼會在使用者完成表單時受到呼叫，可用來建立通話訊息及產生通話。 在此範例中，程式碼會在表單按鈕的 onclick 事件處理常式中執行。 (在下方的程式碼中，請使用您的 Twilio 帳戶和驗證權杖，而不要使用指派給 `accountSID` 和 `authToken` 的預留位置值。)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using Twilio;
using Twilio.Http;
using Twilio.Types;
using Twilio.Rest.Api.V2010;

namespace WebRole1
{
    public partial class _Default : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {

        }

        protected void callpage_Click(object sender, EventArgs e)
        {
            // Call porcessing happens here.

            // Use your account SID and authentication token instead of
            // the placeholders shown here.
            var accountSID = "ACNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN";
            var authToken =  "NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN";

            // Instantiate an instance of the Twilio client.
            TwilioClient.Init(accountSID, authToken);

            // Retrieve the account, used later to retrieve the
            var account = AccountResource.Fetch(accountSID);

            this.varDisplay.Items.Clear();

            if (this.toNumber.Text == "" || this.message.Text == "")
            {
                this.varDisplay.Items.Add(
                        "You must enter a phone number and a message.");
            }
            else
            {
                // Retrieve the values entered by the user.
                var to = PhoneNumber(this.toNumber.Text);
                var from = new PhoneNumber("+14155992671");
                var myMessage = this.message.Text;

                // Create a URL using the Twilio message and the user-entered
                // text. You must replace spaces in the user's text with '%20'
                // to make the text suitable for a URL.
                var url = $"http://twimlets.com/message?Message%5B0%5D={myMessage.Replace(" ", "%20")}";
                var twimlUri = new Uri(url);

                // Display the endpoint, API version, and the URL for the message.
                this.varDisplay.Items.Add($"Using Twilio endpoint {
                }");
                this.varDisplay.Items.Add($"Twilioclient API Version is {apiVersion}");
                this.varDisplay.Items.Add($"The URL is {url}");

                // Place the call.
                var call = CallResource.create(to, from, url: twimlUri);
                this.varDisplay.Items.Add("Call status: " + call.Status);
            }
        }
    }
}
```

通話建立後，會顯示 Twilio 端點、API 版本和通話狀態。 下列螢幕擷取畫面顯示執行範例的輸出。

![Azure call response using Twilio and ASP.NET][twilio_dotnet_basic_form_output]

如需 TwiML 的相關資訊，請參閱 [http://www.twilio.com/docs/api/twiml][twiml]。 如需 &lt;Say&gt; 和其他 Twilio 動詞的詳細資訊，請參閱 [http://www.twilio.com/docs/api/twiml/say][twilio_say]。

## <a id="nextsteps"></a>後續步驟
此程式可說明在 Azure 上的 ASP.NET Web 角色中使用 Twilio 的基本功能。 在部署至生產環境中的 Azure 之前，您可以新增更多錯誤處理或其他功能。 例如︰

* 除了使用 Web 表單以外，您也可以使用 Azure Blob 儲存體或 Azure SQL Database 執行個體來儲存電話號碼和通話文字。 如需在 Azure 中使用 Blob 的相關資訊，請參閱[如何在 .NET 中使用 Azure Blob 儲存體服務][howto_blob_storage_dotnet]。 如需使用 SQL Database 的相關資訊，請參閱[如何在 .NET 應用程式中使用 Azure SQL Database][howto_sql_azure_dotnet]。
* 您可以使用 `RoleEnvironment.getConfigurationSettings`，從部署的組態設定中擷取 Twilio 帳戶 ID 和驗證權杖，而不要在表單中進行值的硬式編碼。 如需 `RoleEnvironment` 類別的相關資訊，請參閱 [Microsoft.WindowsAzure.ServiceRuntime 命名空間][azure_runtime_ref_dotnet]。
* 閱讀 [https://www.twilio.com/docs/security][twilio_docs_security] 上的 Twilio 安全性指引。
* 在 [https://www.twilio.com/docs][twilio_docs] 上深入了解 Twilio。

## <a name="seealso"></a>另請參閱
* [如何透過 Twilio 來使用 Azure 的語音和簡訊功能](twilio-dotnet-how-to-use-for-voice-sms.md)

[twilio_console]: https://www.twilio.com/console
[twilio_pricing]: http://www.twilio.com/pricing
[try_twilio]: http://www.twilio.com/try-twilio
[twilio_api]: http://www.twilio.com/voice/api
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified

[twilio_dotnet_basic_form]: ./media/partner-twilio-cloud-services-dotnet-phone-call-web-role/WA_twilio_dotnet_basic_form.png
[twilio_dotnet_basic_form_output]: ./media/partner-twilio-cloud-services-dotnet-phone-call-web-role/WA_twilio_dotnet_basic_form_output.png

[twiml]: http://www.twilio.com/docs/api/twiml



[howto_twilio_voice_sms_dotnet]: /develop/net/how-to-guides/twilio/

[howto_blob_storage_dotnet]: https://www.windowsazure.com/develop/net/how-to-guides/blob-storage/

[howto_sql_azure_dotnet]: https://www.windowsazure.com/develop/net/how-to-guides/sql-database/


[twilio_docs_security]: http://www.twilio.com/docs/security
[twilio_docs]: http://www.twilio.com/docs
[twilio_say]: http://www.twilio.com/docs/api/twiml/say


[azure_runtime_ref_dotnet]: http://msdn.microsoft.com/library/windowsazure/microsoft.windowsazure.serviceruntime.aspx
[azure_webroles_get_started]: https://docs.microsoft.com/azure/cloud-services/cloud-services-dotnet-get-started
