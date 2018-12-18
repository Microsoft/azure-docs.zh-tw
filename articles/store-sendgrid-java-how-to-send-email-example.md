---
title: store-sendgrid-java-how-to-send-email-example
description: 如何在 Azure 部署中使用 SendGrid 透過 Java 傳送電子郵件
services: ''
documentationcenter: java
author: thinkingserious
manager: sendgrid
editor: mollybos
ms.assetid: af096a73-6985-4350-92e4-ce1722c8d72f
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 10/30/2014
ms.author: vibhork;dominic.may@sendgrid.com;elmer.thomas@sendgrid.com
ms.openlocfilehash: 7db7e00ec3f93695cc6bc3369fe1b00582740d0e
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45578067"
---
# <a name="how-to-send-email-using-sendgrid-from-java-in-an-azure-deployment"></a>如何在 Azure 部署中使用 SendGrid 透過 Java 傳送電子郵件
下列範例將說明如何從 Azure 代管的網頁上使用 SendGrid 傳送電子郵件。 產生的應用程式會提示使用者提供電子郵件值，如下列螢幕擷取畫面所示。

![Email form][emailform]

產生的電子郵件將會類似下列螢幕擷取畫面。

![Email message][emailsent]

您必須執行下列動作才能使用本主題中的程式碼：

1. 例如，從 <http://www.oracle.com/technetwork/java/javamail/index.html> 取得 Javax.mail JAR。
2. 將 JAR 新增至您的 Java 建置路徑。
3. 如果您使用 Eclipse 建立此 Java 應用程式，您可以使用 Eclipse 的部署組件功能在應用程式部署檔案 (WAR) 中加入 SendGrid 程式庫。 如果您並非使用 Eclipse 建立此 Java 應用程式，請確定這些程式庫與您的 Java 應用程式包含在相同的 Azure 角色內，且已新增至應用程式的類別路徑。

您還必須擁有自己的 SendGrid 使用者名稱和密碼，才能傳送電子郵件。 若要開始使用 SendGrid，請參閱 [如何使用 SendGrid 透過 Java 傳送電子郵件](store-sendgrid-java-how-to-send-email.md)。

此外，強烈建議您熟悉 [在 Eclipse 建立 Azure 的 Hello World 應用程式](http://msdn.microsoft.com/library/windowsazure/hh690944)上的資訊，或熟悉其他在 Azure 中代管 Java 應用程式的技術 (如果您未使用 Eclipse)。

## <a name="create-a-web-form-for-sending-email"></a>建立用以傳送電子郵件的 Web 表單
下列程式碼將說明如何建立 Web 表單，以擷取傳送電子郵件所需的使用者資料。 根據本文的用途，我們將此 JSP 檔案命名為 **emailform.jsp**。

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
        pageEncoding="ISO-8859-1" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Email form</title>
    </head>
    <body>
     <p>Fill in all fields and click <b>Send this email</b>.</p>
     <br/>
      <form action="sendemail.jsp" method="post">
       <table>
         <tr>
           <td>To:</td>
           <td><input type="text" size=50 name="emailTo">
           </td>
         </tr>
         <tr>
           <td>From:</td>
           <td><input type="text" size=50 name="emailFrom">
           </td>
         </tr>
         <tr>
           <td>Subject:</td>
           <td><input type="text" size=100 name="emailSubject" value="My email subject">
           </td>
         </tr>
         <tr>
           <td>Text:</td>
           <td><input type="text" size=400 name="emailText" value="Hello,<p>This is my message.</p>Thank you." />
           </td>
         </tr>
         <tr>
           <td>SendGrid user name:</td>
           <td><input type="text" name="sendGridUser">
           </td>
         </tr>
         <tr>
           <td>SendGrid password:</td>
           <td><input type="password" name="sendGridPassword">
           </td>
         </tr>
         <tr>
           <td colspan=2><input type="submit" value="Send this email">
           </td>
         </tr>
       </table>
     </form>
     <br/>
    </body>
    </html>

## <a name="create-the-code-to-send-the-email"></a>建立用以傳送電子郵件的程式碼
下列程式碼會在您完成 emailform.jsp 中的表單時受到呼叫，可用來建立電子郵件訊息並加以傳送。 根據本文的用途，我們將此 JSP 檔案命名為 **sendemail.jsp**。

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
        pageEncoding="ISO-8859-1" import="javax.activation.*, javax.mail.*, javax.mail.internet.*, java.util.Date, java.util.Properties" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Email processing happens here</title>
    </head>
    <body>
        <b>This is my send mail page.</b><p/>
     <%

     final String sendGridUser = request.getParameter("sendGridUser");
     final String sendGridPassword = request.getParameter("sendGridPassword");

     class SMTPAuthenticator extends Authenticator
     {
       public PasswordAuthentication getPasswordAuthentication()
       {
            String username = sendGridUser;
            String password = sendGridPassword;

            return new PasswordAuthentication(username, password);   
       }
     }
     try
     {

         // The SendGrid SMTP server.
         String SMTP_HOST_NAME = "smtp.sendgrid.net";

         Properties properties;

         properties = new Properties();

         // Specify SMTP values.
         properties.put("mail.transport.protocol", "smtp");
         properties.put("mail.smtp.host", SMTP_HOST_NAME);
         properties.put("mail.smtp.port", 587);
         properties.put("mail.smtp.auth", "true");

         // Display the email fields entered by the user. 
         out.println("Value entered for email Subject: " + request.getParameter("emailSubject") + "<br/>");        
         out.println("Value entered for email      To: " + request.getParameter("emailTo") + "<br/>");
         out.println("Value entered for email    From: " + request.getParameter("emailFrom") + "<br/>");
         out.println("Value entered for email    Text: " + "<br/>" + request.getParameter("emailText") + "<br/>");

         // Create the authenticator object.
         Authenticator authenticator = new SMTPAuthenticator();

         // Create the mail session object.
         Session mailSession;
         mailSession = Session.getDefaultInstance(properties, authenticator);

         // Display debug information to stdout, useful when using the
         // compute emulator during development.
         mailSession.setDebug(true);

         // Create the message and message part objects.
         MimeMessage message;
         Multipart multipart;
         MimeBodyPart messagePart; 

         message = new MimeMessage(mailSession);

         multipart = new MimeMultipart("alternative");
         messagePart = new MimeBodyPart();
         messagePart.setContent(request.getParameter("emailText"), "text/html");
         multipart.addBodyPart(messagePart);            

         // Specify the email To, From, Subject and Content. 
         message.setFrom(new InternetAddress(request.getParameter("emailFrom")));
         message.addRecipient(Message.RecipientType.TO, new InternetAddress(request.getParameter("emailTo")));
         message.setSubject(request.getParameter("emailSubject")); 
         message.setContent(multipart);

         // Uncomment the following if you want to add a footer.
         // message.addHeader("X-SMTPAPI", "{\"filters\": {\"footer\": {\"settings\": {\"enable\":1,\"text/html\": \"<html>This is my <b>email footer</b>.</html>\"}}}}");

         // Uncomment the following if you want to enable click tracking.
         // message.addHeader("X-SMTPAPI", "{\"filters\": {\"clicktrack\": {\"settings\": {\"enable\":1}}}}");

         Transport transport;
         transport = mailSession.getTransport();
         // Connect the transport object.
         transport.connect();
         // Send the message.
         transport.sendMessage(message,  message.getRecipients(Message.RecipientType.TO));
         // Close the connection.
         transport.close();

        out.println("<p>Email processing completed.</p>");

     }
     catch (Exception e)
     {
         out.println("<p>Exception encountered: " + 
                            e.getMessage()     +
                            "</p>");   
     }
    %>

    </body>
    </html>

除了傳送電子郵件以外，emailform.jsp 也會將結果提供給使用者，如下列螢幕擷取畫面所示：

![Send mail result][emailresult]

## <a name="next-steps"></a>後續步驟
將您的應用程式部署至計算模擬器，並在瀏覽器中執行 emailform.jsp，輸入表單中的值，按一下 [Send this email] ，然後在 sendemail.jsp 中檢視結果。

此程式碼可說明如何在 Azure 上的 Java 中使用 SendGrid。 在部署至生產環境中的 Azure 之前，您可以新增更多錯誤處理或其他功能。 例如︰ 

* 您可以使用 Azure 儲存體 Blob 或 SQL Database 來儲存電子郵件地址和電子郵件訊息，而不使用 Web 表單。 如需在 Java 中使用 Azure 儲存體 Blob 的相關資訊，請參閱 [如何從 Java 使用 Blob 儲存體服務](https://azure.microsoft.com/develop/java/how-to-guides/blob-storage/)。 如需在 Java 中使用 SQL Database 的相關資訊，請參閱 [在 Java 中使用 SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-connect-query-java)。
* 您可以使用 `RoleEnvironment.getConfigurationSettings` ，從部署的組態設定中擷取 SendGrid 使用者名稱和密碼，而不使用 Web 表單擷取這些值。 如需 `RoleEnvironment` 類別的相關資訊，請參閱[在 JSP 中使用 Azure Service Runtime Library](http://msdn.microsoft.com/library/windowsazure/hh690948)，以及 <http://dl.windowsazure.com/javadoc> 上的 Azure Service Runtime 套件文件。
* 如需在 Java 中使用 SendGrid 的詳細資訊，請參閱 [如何使用 SendGrid 透過 Java 傳送電子郵件](store-sendgrid-java-how-to-send-email.md)。

[emailform]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaEmailform.jpg
[emailsent]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaEmailSent.jpg
[emailresult]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaResult.jpg
