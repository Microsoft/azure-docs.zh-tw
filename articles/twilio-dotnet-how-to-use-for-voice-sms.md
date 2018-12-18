---
title: 如何使用 Twilio 來進行語音交談和傳送簡訊 (.NET) | Microsoft Docs
description: 了解如何在 Azure 上使用 Twilio API 服務撥打電話及傳送簡訊。 程式碼範例以 .NET 撰寫。
services: ''
documentationcenter: .net
author: devinrader
manager: twilio
editor: ''
ms.assetid: 74d4f3c9-f1cb-4968-b744-36b32cd0e834
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 04/24/2015
ms.author: MicrosoftHelp@twilio.com
ms.openlocfilehash: 1442e3af26ae87e645cf207228ed1197b2afdd4d
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23111786"
---
# <a name="how-to-use-twilio-for-voice-and-sms-capabilities-from-azure"></a>如何透過 Twilio 來使用 Azure 的語音和簡訊功能
本指南示範如何在 Azure 上透過 Twilio API 服務執行常見的程式設計工作。 涵蓋的案例包括打電話和傳送簡訊 (SMS)。 如需有關如何在應用程式中使用 Twilio 語音和 SMS 的詳細資訊，請參閱[後續步驟](#NextSteps)一節。

## <a id="WhatIs"></a>什麼是 Twilio？
Twilio 正在形塑商業環境的未來，可讓開發人員將語音、VoIP 和訊息傳送內嵌到應用程式中。 它們將雲端、全球化環境中所需的整個基礎結構虛擬化，透過 Twilio 通訊 API 平台來揭露基礎結構。 輕鬆就可建立和擴充應用程式。 享受隨收隨付定價的彈性和雲端可靠性的好處。

**Twilio 語音** 可讓應用程式撥打和接聽電話。 **Twilio 簡訊**可讓應用程式收發簡訊。 **Twilio 用戶端**可讓您從任何電話、平板電腦或瀏覽器撥打 VoIP 電話，且支援 WebRTC。

## <a id="Pricing"></a>Twilio 定價和特別優惠
升級 Twilio 帳戶的 Azure 客戶，可[特別獲贈](http://www.twilio.com/azure)價值 $10 的 Twilio 點數。 此 Twilio 點數可用來折抵任何 Twilio 使用量 ($10 點數相當於最多傳送 1,000 則簡訊，或最多接收 1000 分鐘的撥入語音，視電話號碼所在地點或通話目的地而定)。 請至 [ahoy.twilio.com/azure](http://ahoy.twilio.com/azure) 兌換 Twilio 點數來開始使用。

Twilio 是隨收隨付的服務。 不需要設定費，隨時都可結清帳戶。 如需詳細資訊，請參閱 [Twilio 價格](http://www.twilio.com/voice/pricing)。

## <a id="Concepts"></a>概念
Twilio API 是一套為應用程式提供語音和簡訊功能的 RESTful API。 用戶端程式庫有多種語言版本，相關清單請參閱 [Twilio API 程式庫][twilio_libraries]。

Twilio API 的兩大重點是 Twilio 動詞和 Twilio 標記語言 (TwiML)。

### <a id="Verbs"></a>Twilio 動詞
API 採用 Twilio 動詞。例如，**&lt;Say&gt;** 動詞指示 Twilio 在通話中用語音傳遞訊息。

以下是 Twilio 動詞清單。  如需了解其他動詞和功能，請參閱 [Twilio 標記語言文件](http://www.twilio.com/docs/api/twiml)。

* **&lt;撥號&gt;**：使撥號者接通另一支電話。
* **&lt;收集&gt;**：收集電話按鍵上輸入的號碼。
* **&lt;掛斷&gt;**：結束通話。
* **&lt;播放&gt;**：播放音訊檔案。
* **&lt;暫停&gt;**：靜候一段指定的秒數。
* **&lt;記錄&gt;**：錄製來電者的語音並傳回含有錄音的檔案 URL。
* **&lt;重新導向&gt;**：將通話或簡訊的控制權移轉至不同 URL 的 TwiML。
* **&lt;拒絕&gt;**：拒絕 Twilio 號碼的來電而不計費
* **&lt;說出&gt;**：將來電的文字轉換成語音。
* **&lt;Sms&gt;**：傳送簡訊。

### <a id="TwiML"></a>TwiML
TwiML 是以 Twilio 動詞為基礎的一組 XML 指令，可指示 Twilio 如何處理來電或簡訊。

例如，下列 TwiML 會將 **Hello World** 文字轉換成語音。

    <?xml version="1.0" encoding="UTF-8" ?>
    <Response>
      <Say>Hello World</Say>
    </Response>

當應用程式呼叫 Twilio API 時，其中一個 API 參數是傳回 TwiML 回應的 URL。 在開發用途上，您可以使用 Twilio 提供的 URL 來提供應用程式所使用的 TwiML 回應。 您也可以裝載您自己的 URL 來產生 TwiML 回應，另一種選擇是使用 **TwiMLResponse** 物件。

如需 Twilio 動詞、屬性和 TwiML 的詳細資訊，請參閱 [TwiML][twiml]。 如需 Twilio API 的詳細資訊，請參閱 [Twilio API][twilio_api]。

## <a id="CreateAccount"></a>建立 Twilio 帳戶
準備取得 Twilio 帳戶時，請至[試用 Twilio][try_twilio] 註冊。 您可以先使用免費帳戶，稍後再升級帳戶。

註冊 Twilio 帳戶時，您會收到帳戶識別碼和驗證權杖。 兩者皆為呼叫 Twilio API 所需。 為了防止未經授權存取您的帳戶，您妥善保管驗證權杖。 在 [Twilio 帳戶頁面][twilio_account] 的 **ACCOUNT SID** 和 **AUTH TOKEN** 欄位中，分別可檢視您的帳戶識別碼和驗證權杖。

## <a id="create_app"></a>建立 Azure 應用程式
裝載已啟用 Twilio 功能之應用程式的 Azure 應用程式，與其他任何 Azure 應用程式並無不同。 您可以新增 Twilio .NET 程式庫並設定角色使用 Twilio .NET 程式庫。
如需有關建立初始 Azure 專案的詳細資訊，請參閱[使用 Visual Studio 建立 Azure 專案][vs_project]。

## <a id="configure_app"></a>設定應用程式使用 Twilio 程式庫
Twilio 提供一套 .NET 協助程式庫，內已封裝 Twilio 的各種組件，讓您簡單又輕鬆地與 Twilio REST API 和 Twilio 用戶端互動，以產生 TwiML 回應。

Twilio 為 .NET 開發人員提供五套程式庫：
程式庫|說明
---|---
Twilio.API|核心 Twilio 程式庫將 Twilio REST API 包裝在一個好用的 .NET 程式庫中。 此程式庫適用於 .NET、Silverlight 和 Windows Phone 7。
Twilio.TwiML|輕鬆利用 .NET 來產生 TwiML 標記。
Twilio.MVC|對於使用 ASP.NET MVC 的開發人員，此程式庫包含 TwilioController、TwiML ActionResult 和要求驗證屬性。
Twilio.WebMatrix|對於使用 Microsoft 免費 WebMatrix 開發工具的開發人員，此程式庫包含各種 Twilio 動作的 Razor 語法協助程式。
Twilio.Client.Capability|包含適用於 Twilio 用戶端 JavaScript SDK 的功能權杖產生器。

請注意，所有程式庫都需要有 .NET 3.5、Silverlight 4 或 Windows Phone 7 或更新版本。

本指南提供的範例使用 Twilio.API 程式庫。

程式庫可以[使用 NuGet 套件管理員延伸模組安裝](http://www.twilio.com/docs/csharp/install) (適用於 Visual Studio 2010 到 2015)。  原始程式碼可裝載於 [GitHub][twilio_github_repo]，它包含一個 Wiki，提供完整的程式庫使用說明文件。

依預設，Microsoft Visual Studio 2010 會安裝 NuGet 1.2 版。 安裝 Twilio 程式庫需要有 NuGet 1.6 版或更新版本。 如需有關安裝或更新 NuGet 的詳細資訊，請參閱 [http://nuget.org/][nuget]。

> [!NOTE]
> 若要安裝最新版的 NuGet，您必須先使用 Visual Studio 擴充功能管理員來解除安裝已載入的版本。 若要這麼做，您必須以系統管理員身分執行 Visual Studio。 否則，[解除安裝] 按鈕會停用。
>
>

### <a id="use_nuget"></a>將 Twilio 程式庫新增至 Visual Studio 專案：
1. 在 Visual Studio 中開啟方案。
2. 以滑鼠右鍵按一下 [參考]。
3. 按一下 [管理 NuGet 套件...]
4. 按一下 [線上]。
5. 在搜尋線上方塊中，輸入 twilio。
6. 在 Twilio 套件上按一下 [安裝]。

## <a id="howto_make_call"></a>作法：撥出電話
以下顯示如何使用 **CallResource** 類別撥出電話。 此程式碼也使用 Twilio 提供的網站來傳回 Twilio 標記語言 (TwiML) 回應。 請將 **to** 和 **from** 電話號碼換成您的值，在執行程式碼之前，請記得先驗證 Twilio 帳戶的 **from** 電話號碼。

    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    const string accountSID = "your_twilio_account";
    const string authToken = "your_twilio_authentication_token";

    // Initialize the TwilioClient.
    TwilioClient.Init(accountSID, authToken);

    // Use the Twilio-provided site for the TwiML response.
    var url = "http://twimlets.com/message";
    url = $"{url}?Message%5B0%5D=Hello%20World";

    // Set the call From, To, and URL values to use for the call.
    // This sample uses the sandbox number provided by
    // Twilio to make the call.
    var call = CallResource.Create(
        to: new PhoneNumber("+NNNNNNNNNN"),
        from: new PhoneNumber("NNNNNNNNNN"),
        url: new Uri(url));
        }

如需 **CallResource.Create** 方法中傳遞之參數的詳細資訊，請參閱 [http://www.twilio.com/docs/api/rest/making-calls][twilio_rest_making_calls]。

如前所述，此程式碼使用 Twilio 提供的網站來傳回 TwiML 回應。 您可以改用您自己的網站來提供 TwiML 回應。 如需詳細資訊，請參閱[如何：從您自己的網站提供 TwiML 回應](#howto_provide_twiml_responses)。

## <a id="howto_send_sms"></a>作法：傳送簡訊
下列螢幕擷取畫面顯示如何使用 **MessageResource** 類別來傳送簡訊。 **from** 號碼由 Twilio 提供給試用帳戶來傳送簡訊。 執行程式碼之前，必須驗證 Twilio 帳戶的 **to** 號碼。

    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    const string accountSID = "your_twilio_account";
    const string authToken = "your_twilio_authentication_token";

    // Initialize the TwilioClient.
    TwilioClient.Init(accountSID, authToken);

    try
    {
        // Send an SMS message.
        var message = MessageResource.Create(
            to: new PhoneNumber("+12069419717"),
            from: new PhoneNumber("+14155992671"),
            body: "This is my SMS message.");
    }
    catch (TwilioException ex)
    {
        // An exception occurred making the REST call
        Console.WriteLine(ex.Message);
    }

## <a id="howto_provide_twiml_responses"></a>作法：從您自己的網站提供 TwiML 回應
當您的應用程式呼叫 Twilio API 時 (例如透過 **CallResource.Create** 方法)，Twilio 會將您的要求傳送到應該傳送 TwiML 回應的 URL。 [作法：撥出電話](#howto_make_call)中的範例使用 Twilio 提供的 URL [http://twimlets.com/message][twimlet_message_url] 傳回回應。

> [!NOTE]
> 雖然 TwiML 是專供 Web 服務使用，但您也可以在瀏覽器中檢視 TwiML。 例如，按一下 [http://twimlets.com/message][twimlet_message_url] 可查看空白 &lt;Response&gt; 元素，又例如，按一下 [http://twimlets.com/message?Message%5B0%5D=Hello%20World](http://twimlets.com/message?Message%5B0%5D=Hello%20World) 可查看包含 &lt;Say&gt; 元素的 &lt;Response&gt; 元素。
>
>

除了依賴 Twilio 提供的 URL，您也可以建立自己的 URL 網站來傳回 HTTP 回應。 您可以使用任何可傳回 HTTP 回應的語言來建立網站。 本主題假設您從 ASP.NET 通用處理常式來裝載 URL。

下列 ASP.NET 處理常式建構一個 TwiML 回應，在來電中顯示 **Hello World** 。

    using System.Text;
    using System.Web;

    namespace WebRole1
    {
        /// <summary>
        /// Summary description for Handler1
        /// </summary>
        public class Handler1 : IHttpHandler
        {
            public void ProcessRequest(HttpContext context)
            {
                const string twiMLResponse =
                    "<Response><Say>Hello World.</Say></Response>";
                
                context.Response.Clear();
                context.Response.ContentType = "text/xml";
                context.Response.ContentEncoding = Encoding.UTF8;
                context.Response.Write(twiMLResponse);
                context.Response.End();
            }

            public bool IsReusable
            {
                get
                {
                    return false;
                }
            }
        }
    }
    
從以上範例中可知，TwiML 回應只是一份 XML 文件。 Twilio.TwiML 程式庫包含可為您產生 TwiML 的類別。 下列範例會產生同上的回應，但是使用 **VoiceResponse** 類別。

    using System.Web;
    using Twilio.TwiML;

    namespace WebRole1
    {
        /// <summary>
        /// Summary description for Handler1
        /// </summary>
        public class Handler1 : IHttpHandler
        {

            public void ProcessRequest(HttpContext context)
            {
                var twiml = new VoiceResponse();
                twiml.Say("Hello World.");

                context.Response.Clear();
                context.Response.ContentType = "text/xml";
                context.Response.Write(twiml.ToString());
                context.Response.End();
            }

            public bool IsReusable
            {
                get
                {
                    return false;
                }
            }
        }
    }

如需 TwiML 的詳細資訊，請參閱 [https://www.twilio.com/docs/api/twiml](https://www.twilio.com/docs/api/twiml)。

設定好如何提供 TwiML 回應之後，就可以將 URL 傳入 **CallResource.Create** 方法。 例如，如果您將名為 MyTwiML 的 Web 應用程式部署至 Azure 雲端服務，且您的 ASP.NET 處理常式名稱為 mytwiml.ashx，則可以如下列程式碼範例所示，將 URL 傳遞至 **CallResource.Create**：

    // This sample uses the sandbox number provided by Twilio to make the call.
    // Place the call.
    var call = CallResource.Create(
        to: new PhoneNumber("+NNNNNNNNNN"),
        from: new PhoneNumber("NNNNNNNNNN"),
        url: new Uri("http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.ashx"));
        }

如需有關在 Azure with ASP.NET 上使用 Twilio 的詳細資訊，請參閱[如何在 Azure 的 Web 角色中使用 Twilio 來撥打電話][howto_phonecall_dotnet]。

[!INCLUDE [twilio-additional-services-and-next-steps](../includes/twilio-additional-services-and-next-steps.md)]

[howto_phonecall_dotnet]: partner-twilio-cloud-services-dotnet-phone-call-web-role.md

[twimlet_message_url]: http://twimlets.com/message

[twilio_rest_making_calls]: http://www.twilio.com/docs/api/rest/making-calls

[vs_project]:http://msdn.microsoft.com/library/windowsazure/ee405487.aspx
[nuget]:http://nuget.org/
[twilio_github_repo]:https://github.com/twilio/twilio-csharp

[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api]: http://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]:  https://www.twilio.com/console
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified
