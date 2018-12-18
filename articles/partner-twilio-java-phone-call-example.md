---
title: 如何從 Twilio 撥打電話 (Java) | Microsoft Docs
description: 了解如何在 Azure 上的 Java 應用程式中使用 Twilio 從網頁撥打電話。
services: ''
documentationcenter: java
author: devinrader
manager: twilio
editor: mollybos
ms.assetid: 0381789e-e775-41a0-a784-294275192b1d
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 11/25/2014
ms.author: microsofthelp@twilio.com
ms.openlocfilehash: 9ef754e9952bcbd47d668331e906b19ad582b90c
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818593"
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-java-application-on-azure"></a>如何在 Azure 上的 Java 應用程式中使用 Twilio 撥打電話
下列範例將說明如何從 Azure 代管的網頁上使用 Twilio 撥打電話。 產生的應用程式會提示使用者提供電話值，如下列螢幕擷取畫面所示。

![Azure Call Form Using Twilio and Java][twilio_java]

您必須執行下列動作才能使用本主題中的程式碼：

1. 取得 Twilio 帳戶和驗證權杖。 若要開始使用 Twilio，請在 [http://www.twilio.com/pricing][twilio_pricing] 上評估價格。 您可以在 [https://www.twilio.com/try-twilio][try_twilio] 註冊。 如需 Twilio 所提供的 API 相關資訊，請參閱 [http://www.twilio.com/api][twilio_api]。
2. 取得 Twilio JAR。 在 [https://github.com/twilio/twilio-java][twilio_java_github] 上，您可以下載 GitHub 來源及建立自己的 JAR，或下載預先建置的 JAR (可能有相依性)。
   本主題中的程式碼是以預先建置的 TwilioJava-3.3.8-with-dependencies JAR 撰寫的。
3. 將 JAR 新增至您的 Java 建置路徑。
4. 如果您使用 Eclipse 建立此 Java 應用程式，請使用 Eclipse 的部署組件功能在應用程式部署檔案 (WAR) 中加入 Twilio JAR。 如果您並非使用 Eclipse 建立此 Java 應用程式，請確定 Twilio JAR 與您的 Java 應用程式包含在相同的 Azure 角色內，且已新增至應用程式的類別路徑。
5. 確定您的 cacerts 金鑰存放區包含 Equifax Secure Certificate Authority 憑證，且具有 MD5 指模 67:CB:9D:C0:13:24:8A:82:9B:B2:17:1E:D1:1B:EC:D4 (序號為 35:DE:F4:CF，SHA1 指模為 D2:32:09:AD:23:D3:14:23:21:74:E4:0D:7F:9D:62:13:97:86:63:3A)。 這是 [https://api.twilio.com][twilio_api_service] 服務的憑證授權單位 (CA) 憑證，會在您使用 Twilio API 時受到呼叫。 如需將此 CA 憑證新增至 JDK cacert 存放區的相關資訊，請參閱[新增憑證至 Java CA 憑證存放區][add_ca_cert]。

此外，強烈建議您熟悉[使用 Eclipse 的 Azure 工具組建立 Hello World 應用程式][azure_java_eclipse_hello_world]上的資訊，或熟悉其他在 Azure 中代管 Java 應用程式的技術 (如果您未使用 Eclipse)。

## <a name="create-a-web-form-for-making-a-call"></a>建立用以撥打電話的 Web 表單
下列程式碼將說明如何建立 Web 表單，以擷取撥打電話所需的使用者資料。 在此範例中，我們新建立了名為 **TwilioCloud** 的動態 Web 專案，並將 **callform.jsp** 新增為 JSP 檔案。

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
        pageEncoding="ISO-8859-1" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Automated call form</title>
    </head>
    <body>
     <p>Fill in all fields and click <b>Make this call</b>.</p>
     <br/>
      <form action="makecall.jsp" method="post">
       <table>
         <tr>
           <td>To:</td>
           <td><input type="text" size=50 name="callTo" value="" />
           </td>
         </tr>
         <tr>
           <td>From:</td>
           <td><input type="text" size=50 name="callFrom" value="" />
           </td>
         </tr>
         <tr>
           <td>Call message:</td>
           <td><input type="text" size=400 name="callText" value="Hello. This is the call text. Good bye." />
           </td>
         </tr>
         <tr>
           <td colspan=2><input type="submit" value="Make this call" />
           </td>
         </tr>
       </table>
     </form>
     <br/>
    </body>
    </html>

## <a name="create-the-code-to-make-the-call"></a>建立用以撥打電話的程式碼
下列程式碼會在使用者完成 callform.jsp 所顯示的表單時受到呼叫，可用來建立通話訊息及產生通話。 在此範例中，我們將名為 **makecall.jsp** 的 JSP 檔案新增至 **TwilioCloud** 專案。 (在下方的程式碼中，請使用您的 Twilio 帳戶和驗證權杖，而不要使用指派給 **accountSID** 和 **authToken** 的預留位置值。)

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    import="java.util.*"
    import="com.twilio.*"
    import="com.twilio.sdk.*"
    import="com.twilio.sdk.resource.factory.*"
    import="com.twilio.sdk.resource.instance.*"
    pageEncoding="ISO-8859-1" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Call processing happens here</title>
    </head>
    <body>
        <b>This is my make call page.</b><p/>
     <%
    try 
    {
         // Use your account SID and authentication token instead
         // of the placeholders shown here.
         String accountSID = "your_twilio_account";
         String authToken = "your_twilio_authentication_token";

         // Instantiate an instance of the Twilio client.     
         TwilioRestClient client;
         client = new TwilioRestClient(accountSID, authToken);

         // Retrieve the account, used later to retrieve the CallFactory.
         Account account = client.getAccount();

         // Display the client endpoint. 
         out.println("<p>Using Twilio endpoint " + client.getEndpoint() + ".</p>");

         // Display the API version.
         String APIVERSION = TwilioRestClient.DEFAULT_VERSION;
         out.println("<p>Twilio client API version is " + APIVERSION + ".</p>");

         // Retrieve the values entered by the user.
         String callTo = request.getParameter("callTo");  
         // The Outgoing Caller ID, used for the From parameter,
         // must have previously been verified with Twilio.
         String callFrom = request.getParameter("callFrom");
         String userText = request.getParameter("callText");

         // Replace spaces in the user's text with '%20', 
         // to make the text suitable for a URL.
         userText = userText.replace(" ", "%20");

         // Create a URL using the Twilio message and the user-entered text.
         String Url="http://twimlets.com/message";
         Url = Url + "?Message%5B0%5D=" + userText;

         // Display the message URL.
         out.println("<p>");
         out.println("The URL is " + Url);
         out.println("</p>");

         // Place the call From, To and URL values into a hash map. 
         HashMap<String, String> params = new HashMap<String, String>();
         params.put("From", callFrom);
         params.put("To", callTo);
         params.put("Url", Url);

         CallFactory callFactory = account.getCallFactory();
         Call call = callFactory.create(params);
         out.println("<p>Call status: " + call.getStatus()  + "</p>"); 
    } 
    catch (TwilioRestException e) 
    {
        out.println("<p>TwilioRestException encountered: " + e.getMessage() + "</p>");
        out.println("<p>StackTrace: " + e.getStackTrace().toString() + "</p>");
    }
    catch (Exception e) 
    {
        out.println("<p>Exception encountered: " + e.getMessage() + "");
        out.println("<p>StackTrace: " + e.getStackTrace().toString() + "</p>");
    }
    %>
    </body>
    </html>

除了撥打電話以外，makecall.jsp 也會顯示 Twilio 端點、API 版本和通話狀態。 下列螢幕擷取畫面顯示其範例：

![Azure Call Response Using Twilio and Java][twilio_java_response]

## <a name="run-the-application"></a>執行應用程式
以下是執行應用程式的高階步驟；如需這些步驟的詳細資訊，請參閱[使用 Eclipse 的 Azure 工具組建立 Hello World 應用程式][azure_java_eclipse_hello_world]。

1. 將您的 TwilioCloud WAR 匯出至 Azure **approot** 資料夾。 
2. 修改 **startup.cmd** ，以將 TwilioCloud WAR 解壓縮。
3. 針對計算模擬器編譯您的應用程式。
4. 在計算模擬器中開始進行部署。
5. 開啟瀏覽器並執行 **http://localhost:8080/TwilioCloud/callform.jsp**。
6. 輸入表單中的值，按一下 [Make this call] ，然後在 makecall.jsp 中檢視結果。

當您做好部署至 Azure 的準備後，您可以針對雲端環境重新編譯部署、部署至 Azure，以及在瀏覽器中執行 http://*your_hosted_name*.cloudapp.net/TwilioCloud/callform.jsp (請將 *your_hosted_name* 取代為您的值)。

## <a name="next-steps"></a>後續步驟
此程式可說明在 Azure 上的 Java 中使用 Twilio 的基本功能。 在部署至生產環境中的 Azure 之前，您可以新增更多錯誤處理或其他功能。 例如︰

* 除了使用 Web 表單以外，您也可以使用 Azure 儲存體 Blob 或 SQL Database 來儲存電話號碼和通話文字。 如需在 Java 中使用 Azure 儲存體 Blob 的相關資訊，請參閱[如何使用 Java 的 Blob 儲存體服務][howto_blob_storage_java]。 
* 您可以使用 **RoleEnvironment.getConfigurationSettings** ，從部署的組態設定中擷取 Twilio 帳戶 ID 和驗證權杖，而不要在 makecall.jsp 中進行值的硬式編碼。 如需 **RoleEnvironment** 類別的相關資訊，請參閱[在 JSP 中使用 Azure Service Runtime Library][azure_runtime_jsp]，以及 [http://dl.windowsazure.com/javadoc][azure_javadoc] 上的 Azure Service Runtime 套件文件。
* makecall.jsp 程式碼會將 Twilio 提供的 URL [http://twimlets.com/message][twimlet_message_url] 指派給 **URL** 變數。 此 URL 會提供 Twilio 標記語言 (TwiML) 回應，告知 Twilio 應如何執行通話。 例如，傳回的 TwiML 可能會包含 **&lt;Say&gt;** 動詞，而產生要傳達給受話方的文字。 除了使用 Twilio 提供的 URL 以外，您也可以建置自己的服務來回應 Twilio 的要求；如需詳細資訊，請參閱[如何在 Java 中透過 Twilio 使用語音和簡訊功能][howto_twilio_voice_sms_java]。 如需 TwiML 的詳細資訊，請參閱 [http://www.twilio.com/docs/api/twiml][twiml]，如需 **&lt;Say&gt;** 和其他 Twilio 動詞的詳細資訊，則請參閱 [http://www.twilio.com/docs/api/twiml/say][twilio_say]。
* 閱讀 Twilio 安全性方針，網址為 [https://www.twilio.com/docs/security][twilio_docs_security]。

如需 Twilio 的其他資訊，請參閱 [https://www.twilio.com/docs][twilio_docs]。

## <a name="see-also"></a>另請參閱
* [如何在 Java 中透過 Twilio 使用語音和簡訊功能][howto_twilio_voice_sms_java]
* [新增憑證至 Java CA 憑證存放區][add_ca_cert]

[twilio_pricing]: http://www.twilio.com/pricing
[try_twilio]: http://www.twilio.com/try-twilio
[twilio_api]: http://www.twilio.com/api
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
[twilio_java_github]: http://github.com/twilio/twilio-java
[twimlet_message_url]: http://twimlets.com/message
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api_service]: http://api.twilio.com
[add_ca_cert]: java-add-certificate-ca-store.md
[azure_java_eclipse_hello_world]: https://docs.microsoft.com/java/azure/eclipse/azure-toolkit-for-eclipse-create-hello-world-web-app 
[howto_twilio_voice_sms_java]: partner-twilio-java-how-to-use-voice-sms.md
[howto_blob_storage_java]: http://www.windowsazure.com/develop/java/how-to-guides/blob-storage/
[howto_sql_azure_java]: http://msdn.microsoft.com/library/windowsazure/hh749029.aspx
[azure_runtime_jsp]: http://msdn.microsoft.com/library/windowsazure/hh690948.aspx
[azure_javadoc]: http://dl.windowsazure.com/javadoc
[twilio_docs_security]: http://www.twilio.com/docs/security
[twilio_docs]: http://www.twilio.com/docs
[twilio_say]: http://www.twilio.com/docs/api/twiml/say
[twilio_java]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaCallForm.jpg
[twilio_java_response]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaMakeCall.jpg
