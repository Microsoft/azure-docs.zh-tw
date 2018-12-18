---
title: 如何使用 Twilio for Voice and SMS (Java) | Microsoft Docs
description: 了解如何在 Azure 上使用 Twilio API 服務撥打電話及傳送簡訊。 程式碼範例以 Java 撰寫。
services: ''
documentationcenter: java
author: devinrader
manager: twilio
editor: mollybos
ms.assetid: f3508965-5527-4255-9d51-5d5f926f4d43
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 11/25/2014
ms.author: microsofthelp@twilio.com
ms.openlocfilehash: 5a1b2ffa160a31b639605242b651dc8d14e7a01b
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23040023"
---
# <a name="how-to-use-twilio-for-voice-and-sms-capabilities-in-java"></a>如何在 Java 中透過 Twilio 使用語音和簡訊功能
本指南示範如何在 Azure 上透過 Twilio API 服務執行常見的程式設計工作。 涵蓋的案例包括打電話和傳送簡訊 (SMS)。 如需有關如何在應用程式中使用 Twilio 語音和 SMS 的詳細資訊，請參閱 [後續步驟](#NextSteps) 一節。

## <a id="WhatIs"></a>什麼是 Twilio？
Twilio 是一種電話語音 Web 服務 API，能夠讓您使用現有的 Web 語言和技術建立語音和 SMS 應用程式。 Twilio 算是協力廠商服務 (並非 Azure 功能，也並非 Microsoft 產品)。

**Twilio 語音** 可讓應用程式撥打和接聽電話。 **Twilio SMS** 可以讓您的應用程式撰寫和接收 SMS 訊息。 **Twilio Client** 可以讓您的應用程式在現有網際網路連線 (包括行動連線) 中啟用語音通訊。

## <a id="Pricing"></a>Twilio 定價和特別優惠
[Twilio 定價][twilio_pricing] (英文) 提供 Twilio 的定價資訊。 Azure 客戶可獲得[特殊優惠][special_offer]：免費 1000 則文字簡訊或接聽 1000 分鐘電話。 若要註冊獲得這項優惠或取得詳細資訊，請造訪 [http://ahoy.twilio.com/azure][special_offer] (英文)。

## <a id="Concepts"></a>概念
Twilio API 是一套為應用程式提供語音和簡訊功能的 RESTful API。 用戶端程式庫有多種語言版本，相關清單請參閱 [Twilio API 程式庫][twilio_libraries]。

Twilio API 的兩大重點是 Twilio 動詞和 Twilio 標記語言 (TwiML)。

### <a id="Verbs"></a>Twilio 動詞
API 採用 Twilio 動詞。例如，**&lt;Say&gt;** 動詞指示 Twilio 在通話中用語音傳遞訊息。

以下是 Twilio 動詞清單。

* **&lt;撥號&gt;**：使撥號者接通另一支電話。
* **&lt;收集&gt;**：收集電話按鍵上輸入的號碼。
* **&lt;掛斷&gt;**：結束通話。
* **&lt;播放&gt;**：播放音訊檔案。
* **&lt;佇列&gt;**︰新增至呼叫端佇列。
* **&lt;暫停&gt;**：靜候一段指定的秒數。
* **&lt;記錄&gt;**：錄製來電者的語音並傳回含有錄音的檔案 URL。
* **&lt;重新導向&gt;**：將通話或簡訊的控制權移轉至不同 URL 的 TwiML。
* **&lt;拒絕&gt;**：拒絕 Twilio 號碼的來電而不計費。
* **&lt;說出&gt;**：將來電的文字轉換成語音。
* **&lt;Sms&gt;**：傳送簡訊。

### <a id="TwiML"></a>TwiML
TwiML 是以 Twilio 動詞為基礎的一組 XML 指令，可指示 Twilio 如何處理來電或簡訊。

例如，下列 TwiML 會將文字 **Hello World!** 轉換 成語音。

```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <Response>
       <Say>Hello World!</Say>
    </Response>
```

當應用程式呼叫 Twilio API 時，其中一個 API 參數是傳回 TwiML 回應的 URL。 在開發用途上，您可以使用 Twilio 提供的 URL 來提供應用程式所使用的 TwiML 回應。 您也可以裝載您自己的 URL 來產生 TwiML 回應，另一種選擇是使用 **TwiMLResponse** 物件。

如需 Twilio 動詞、屬性和 TwiML 的詳細資訊，請參閱 [TwiML][twiml]。 如需 Twilio API 的詳細資訊，請參閱 [Twilio API][twilio_api]。

## <a id="CreateAccount"></a>建立 Twilio 帳戶
準備取得 Twilio 帳戶時，請至[試用 Twilio][try_twilio] 註冊。 您可以先使用免費帳戶，稍後再升級帳戶。

註冊 Twilio 帳戶時，您會收到帳戶識別碼和驗證權杖。 兩者皆為呼叫 Twilio API 所需。 為了防止未經授權存取您的帳戶，您妥善保管驗證權杖。 在 [Twilio 主控台][twilio_console]的 **ACCOUNT SID** 和 **AUTH TOKEN** 欄位中，分別可檢視您的帳戶識別碼和驗證權杖。

## <a id="create_app"></a>建立 Java 應用程式
1. 取得 Twilio JAR 並將它加到您的 Java 組建路徑和 WAR 部署組件。 在 [https://github.com/twilio/twilio-java][twilio_java] 上，您可以下載 GitHub 來源及建立自己的 JAR，或下載預先建置的 JAR (可能有相依性)。
2. 確定 JDK 的 **cacerts** 金鑰存放區包含 MD5 指紋為 67:CB:9D:C0:13:24:8A:82:9B:B2:17:1E:D1:1B:EC:D4 (序號為 35:DE:F4:CF 且 SHA1 指紋為 D2:32:09:AD:23:D3:14:23:21:74:E4:0D:7F:9D:62:13:97:86:63:3A) 的 Equifax 安全憑證授權單位憑證。 這是 [https://api.twilio.com][twilio_api_service] 服務的憑證授權單位 (CA) 憑證，會在您使用 Twilio API 時受到呼叫。 如需關於確定 JDK 的 **cacerts** 金鑰存放區包含正確 CA 憑證的詳細資訊，請參閱[新增憑證至 Java CA 憑證存放區][add_ca_cert]。

[如何從 Azure 中的 Java 應用程式使用 Twilio 撥打電話][howto_phonecall_java]中提供使用 Java 版 Twilio 用戶端程式庫的詳細指示。

## <a id="configure_app"></a>設定應用程式以使用 Twilio 程式庫
在您的程式碼中，您可以針對於要在應用程式中使用的 Twilio 封裝或類別，在其原始程式檔的頂端加入 **import** 陳述式。

對於 Java 原始程式檔：

```java
    import com.twilio.*;
    import com.twilio.rest.api.*;
    import com.twilio.type.*;
    import com.twilio.twiml.*;
```

對於 Java 伺服器頁面 (JSP) 原始程式檔：

```java
    import="com.twilio.*"
    import="com.twilio.rest.api.*"
    import="com.twilio.type.*"
    import="com.twilio.twiml.*"
 ```
 
端視要使用的 Twilio 封裝或類別而定，您的 **import** 陳述式可能不同。

## <a id="howto_make_call"></a>作法：撥出電話
以下顯示如何使用 **Call** 類別撥出電話。 此程式碼也使用 Twilio 提供的網站來傳回 Twilio 標記語言 (TwiML) 回應。 請將 **from** 和 **to** 電話號碼換成您的值，在執行程式碼之前，請記得先驗證 Twilio 帳戶的 **from** 電話號碼。

```java
    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    String accountSID = "your_twilio_account_SID";
    String authToken = "your_twilio_authentication_token";

    // Initialize the Twilio client.
    Twilio.init(accountSID, authToken);

    // Use the Twilio-provided site for the TwiML response.
    URI uri = new URI("http://twimlets.com/message" +
            "?Message%5B0%5D=Hello%20World%21");

    // Declare To and From numbers
    PhoneNumber to = new PhoneNumber("NNNNNNNNNN");
    PhoneNumber from = new PhoneNumber("NNNNNNNNNN");

    // Create a Call creator passing From, To and URL values
    // then make the call by executing the create() method
    Call.creator(to, from, uri).create();
```

如需 **Call.creator** 方法中傳遞之參數的詳細資訊，請參閱 [http://www.twilio.com/docs/api/rest/making-calls][twilio_rest_making_calls] (英文)。

如前所述，此程式碼使用 Twilio 提供的網站來傳回 TwiML 回應。 您可以改用自己的網站提供 TwiML 回應；如需詳細資訊，請參閱 [如何在 Azure 上的 Java 應用程式中提供 TwiML 回應](#howto_provide_twiml_responses)。

## <a id="howto_send_sms"></a>作法：傳送簡訊
以下顯示如何使用 **Message** 類別傳送 SMS 訊息。 **from** 號碼 **4155992671** 是 Twilio 提供來傳送 SMS 訊息的試用帳戶。 執行程式碼之前，必須驗證您 Twilio 帳戶的 **to** 號碼。

```java
    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    String accountSID = "your_twilio_account_SID";
    String authToken = "your_twilio_authentication_token";

    // Initialize the Twilio client.
    Twilio.init(accountSID, authToken);

    // Declare To and From numbers and the Body of the SMS message
    PhoneNumber to = new PhoneNumber("+14159352345"); // Replace with a valid phone number for your account.
    PhoneNumber from = new PhoneNumber("+14158141829"); // Replace with a valid phone number for your account.
    String body = "Where's Wallace?";

    // Create a Message creator passing From, To and Body values
    // then send the SMS message by calling the create() method
    Message sms = Message.creator(to, from, body).create();
```

如需 **Message.creator** 方法中傳遞之參數的詳細資訊，請參閱 [http://www.twilio.com/docs/api/rest/sending-sms][twilio_rest_sending_sms] (英文)。

## <a id="howto_provide_twiml_responses"></a>作法：從您自己的網站提供 TwiML 回應
當您的應用程式呼叫 Twilio API 時 (例如透過 **CallCreator.create** 方法)，Twilio 將傳送您的要求到應該傳送 TwiML 回應的 URL。 前述範例使用 Twilio 提供的 URL [http://twimlets.com/message][twimlet_message_url]。 (雖然 TwiML 是針對供 Web 服務使用而設計，但您可以在瀏覽器中檢視 TwiML。 例如，按一下 [http://twimlets.com/message][twimlet_message_url] 可查看空白 **&lt;Response&gt;** 元素，又例如，按一下 [http://twimlets.com/message?Message%5B0%5D=Hello%20World%21][twimlet_message_url_hello_world] 可查看包含 **&lt;Say&gt;** 元素的 **&lt;Response&gt;** 元素。)

除了依賴 Twilio 提供的 URL，您也可以建立自己的 URL 網站來傳回 HTTP 回應。 您可以使用任何語言建立會傳回 HTTP 回應的網站；本主題假設您將該 URL 裝載在 JSP 頁面中。

下列 JSP 頁面將引起 TwiML 說出 **Hello World** 作為回應 (在通話中)。

```xml
    <%@ page contentType="text/xml" %>
    <Response>
        <Say>Hello World!</Say>
    </Response>
```

下列 JSP 頁面將引起 TwiML 有以下回應：說出一些文字、停頓數次，然後說出 Twilio API 版本和 Azure 角色名稱資訊。

```xml
    <%@ page contentType="text/xml" %>
    <Response>
        <Say>Hello from Azure!</Say>
        <Pause></Pause>
        <Say>The Twilio API version is <%= request.getParameter("ApiVersion") %>.</Say>
        <Say>The Azure role name is <%= System.getenv("RoleName") %>.</Say>
        <Pause></Pause>
        <Say>Good bye.</Say>
    </Response>
```

**ApiVersion** 參數會出現在 Twilio 語音要求 (而非 SMS 要求) 中。 若要查看 Twilio 語音和 SMS 要求的可用要求參數，請分別參閱 <https://www.twilio.com/docs/api/twiml/twilio_request> (英文) 及 <https://www.twilio.com/docs/api/twiml/sms/twilio_request> (英文)。 **RoleName** 環境參數會隨附在 Azure 部署中。 (如果您要新增自訂環境參數，以便可以從 **System.getenv** 選擇這些參數，請參閱[其他角色組態設定的環境變數][misc_role_config_settings]一節。)

設定 JSP 頁面來提供 TwiML 回應之後，請使用 JSP 頁面的 URL 作為傳遞到 **Call.creator** 方法的 URL。 例如，如果將名稱為 MyTwiML 的 Web 應用程式部署到 Azure 託管服務，而且 JSP 頁面的名稱是 mytwiml.jsp，則可以將 URL 傳遞到 **Call.creator**，如下所示：

```java
    // Declare To and From numbers and the URL of your JSP page
    PhoneNumber to = new PhoneNumber("NNNNNNNNNN");
    PhoneNumber from = new PhoneNumber("NNNNNNNNNN");
    URI uri = new URI("http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.jsp");

    // Create a Call creator passing From, To and URL values
    // then make the call by executing the create() method
    Call.creator(to, from, uri).create();
```

另一個選項是透過 **com.twilio.twiml** 封裝中的 **VoiceResponse** 類別進行 TwiML 回應。

如需關於透過 Java 在 Azure 中使用 Twilio 的詳細資訊，請參閱[如何從 Azure 中的 Java 應用程式使用 Twilio 撥打電話][howto_phonecall_java]。

## <a id="AdditionalServices"></a>如何：使用其他 Twilio 服務
除了此處所示的範例以外，Twilio 還提供網頁式 API，方便您從 Azure 應用程式中充份利用其他 Twilio 功能。 如需完整詳細資料，請參閱 [Twilio API 文件][twilio_api_documentation]。

## <a id="NextSteps"></a>後續步驟
了解基本的 Twilio 服務之後，請參考下列連結以取得更多資訊：

* [Twilio 安全性方針][twilio_security_guidelines]
* [Twilio 作法與範例程式碼][twilio_howtos]
* [Twilio 快速入門教學課程][twilio_quickstarts]
* [GitHub 上的 Twilio][twilio_on_github]
* [洽詢 Twilio 支援][twilio_support]

[twilio_java]: https://github.com/twilio/twilio-java
[twilio_api_service]: https://api.twilio.com
[add_ca_cert]: java-add-certificate-ca-store.md
[howto_phonecall_java]: partner-twilio-java-phone-call-example.md
[misc_role_config_settings]: http://msdn.microsoft.com/library/windowsazure/hh690945.aspx
[twimlet_message_url]: http://twimlets.com/message
[twimlet_message_url_hello_world]: http://twimlets.com/message?Message%5B0%5D=Hello%20World%21
[twilio_rest_making_calls]: http://www.twilio.com/docs/api/rest/making-calls
[twilio_rest_sending_sms]: http://www.twilio.com/docs/api/rest/sending-sms
[twilio_pricing]: http://www.twilio.com/pricing
[special_offer]: http://ahoy.twilio.com/azure
[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api]: http://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_console]:  https://www.twilio.com/console
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified#
[twilio_api_documentation]: http://www.twilio.com/docs
[twilio_security_guidelines]: http://www.twilio.com/docs/security
[twilio_howtos]: http://www.twilio.com/docs/howto
[twilio_on_github]: https://github.com/twilio
[twilio_support]: http://www.twilio.com/help/contact
[twilio_quickstarts]: http://www.twilio.com/docs/quickstart
