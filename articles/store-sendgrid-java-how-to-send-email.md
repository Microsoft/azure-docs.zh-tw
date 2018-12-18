---
title: 如何使用 SendGrid 電子郵件服務 (Java) | Microsoft Docs
description: 了解如何在 Azure 使用 SendGrid 電子郵件服務傳送電子郵件。 程式碼範例以 Java 撰寫。
services: ''
documentationcenter: java
author: thinkingserious
manager: sendgrid
editor: mollybos
ms.assetid: cc75c43e-ede9-492b-98c2-9147fcb92c21
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 10/30/2014
ms.author: elmer.thomas@sendgrid.com; erika.berkland@sendgrid.com; vibhork
ms.openlocfilehash: cadd771a516270faf23f18a36f54fa9a1541e6d5
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38698665"
---
# <a name="how-to-send-email-using-sendgrid-from-java"></a>如何使用 SendGrid 透過 Java 傳送電子郵件
本指南示範如何在 Azure 上透過 SendGrid 電子郵件服務執行常見程式設計工作。 相關範例是以 Java 撰寫的。 涵蓋的案例包括**建構電子郵件**、**傳送電子郵件**、**新增附件**、**使用篩選器**及**更新屬性**。 如需有關 SendGrid 及傳送電子郵件的詳細資訊，請參閱[後續步驟](#next-steps)一節。

## <a name="what-is-the-sendgrid-email-service"></a>什麼是 SendGrid 電子郵件服務？
SendGrid 是 [雲端架構電子郵件服務]，能提供可靠的 [交易式電子郵件傳遞]、擴充性和即時分析，以及有彈性的 API 來輕鬆進行自訂整合。 常見的 SendGrid 使用案例包括：

* 自動傳送回條給客戶
* 管理通訊群組清單，以便將每月電子傳單和特別供應項目傳送給客戶
* 收集封鎖的電子郵件、客戶的回應情形等項目的即時度量
* 產生報表，協助找出趨勢
* 轉寄客戶查詢
* 透過電子郵件從您的應用程式傳送通知

如需詳細資訊，請參閱 <http://sendgrid.com>。

## <a name="create-a-sendgrid-account"></a>建立 SendGrid 帳戶
[!INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="how-to-use-the-javaxmail-libraries"></a>如何：使用 javax.mail 程式庫
取得 javax.mail 程式庫，例如從 <http://www.oracle.com/technetwork/java/javamail> 取得並將其匯入您的程式碼。 使用 javax.mail 程式庫來傳送採用 SMTP 之電子郵件的高層級程序就是執行下列動作：

1. 指定 SMTP 值 (包括 SMTP 伺服器)，對 SendGrid 而言是 smtp.sendgrid.net。

```
        import java.util.Properties;
        import javax.activation.*;
        import javax.mail.*;
        import javax.mail.internet.*;

        public class MyEmailer {
           private static final String SMTP_HOST_NAME = "smtp.sendgrid.net";
           private static final String SMTP_AUTH_USER = "your_sendgrid_username";
           private static final String SMTP_AUTH_PWD = "your_sendgrid_password";

           public static void main(String[] args) throws Exception{
               new MyEmailer().SendMail();
           }

           public void SendMail() throws Exception
           {
              Properties properties = new Properties();
                 properties.put("mail.transport.protocol", "smtp");
                 properties.put("mail.smtp.host", SMTP_HOST_NAME);
                 properties.put("mail.smtp.port", 587);
                 properties.put("mail.smtp.auth", "true");
                 // …
```

1. 擴充 *javax.mail.Authenticator* 類別，以及在 *getPasswordAuthentication* 方法的實作中，傳回您的 SendGrid 使用者名稱和密碼。  

       private class SMTPAuthenticator extends javax.mail.Authenticator {
       public PasswordAuthentication getPasswordAuthentication() {
          String username = SMTP_AUTH_USER;
          String password = SMTP_AUTH_PWD;
          return new PasswordAuthentication(username, password);
       }
2. 透過 *javax.mail.Session* 物件建立經過驗證的電子郵件工作階段。  

       Authenticator auth = new SMTPAuthenticator();
       Session mailSession = Session.getDefaultInstance(properties, auth);
3. 建立郵件並指派 [收件者]、[寄件者]、[主旨] 和內容值。 這顯示在[如何：建立電子郵件](#how-to-create-an-email)一節中。
4. 透過 *javax.mail.Transport* 物件傳送郵件。 這顯示在 [如何：傳送電子郵件][#how-to-send-an-email] 一節中。

## <a name="how-to-create-an-email"></a>如何：建立電子郵件
下列程式碼顯示如何指定電子郵件的值。

    MimeMessage message = new MimeMessage(mailSession);
    Multipart multipart = new MimeMultipart("alternative");
    BodyPart part1 = new MimeBodyPart();
    part1.setText("Hello, Your Contoso order has shipped. Thank you, John");
    BodyPart part2 = new MimeBodyPart();
    part2.setContent(
        "<p>Hello,</p>
        <p>Your Contoso order has <b>shipped</b>.</p>
        <p>Thank you,<br>John</br></p>",
        "text/html");
    multipart.addBodyPart(part1);
    multipart.addBodyPart(part2);
    message.setFrom(new InternetAddress("john@contoso.com"));
    message.addRecipient(Message.RecipientType.TO,
       new InternetAddress("someone@example.com"));
    message.setSubject("Your recent order");
    message.setContent(multipart);

## <a name="how-to-send-an-email"></a>如何：傳送電子郵件
下列程式碼顯示如何傳送電子郵件。

    Transport transport = mailSession.getTransport();
    // Connect the transport object.
    transport.connect();
    // Send the message.
    transport.sendMessage(message, message.getAllRecipients());
    // Close the connection.
    transport.close();

## <a name="how-to-add-an-attachment"></a>如何：新增附件
下列程式碼顯示如何新增附件。

    // Local file name and path.
    String attachmentName = "myfile.zip";
    String attachmentPath = "c:\\myfiles\\";
    MimeBodyPart attachmentPart = new MimeBodyPart();
    // Specify the local file to attach.
    DataSource source = new FileDataSource(attachmentPath + attachmentName);
    attachmentPart.setDataHandler(new DataHandler(source));
    // This example uses the local file name as the attachment name.
    // They could be different if you prefer.
    attachmentPart.setFileName(attachmentName);
    multipart.addBodyPart(attachmentPart);

## <a name="how-to-use-filters-to-enable-footers-tracking-and-analytics"></a>如何：使用篩選器來啟用頁尾、追蹤和分析
SendGrid 運用「篩選器」提供其他電子郵件功能。 這些設定可新增到電子郵件以啟用特定功能，例如啟用點擊追蹤、Google 分析、訂閱追蹤等。 如需完整的篩選器清單，請參閱[篩選器設定][Filter Settings]。

* 下列程式碼顯示如何插入頁尾篩選器，以使 HTML 文字出現在傳送之電子郵件的底部。

      message.addHeader("X-SMTPAPI",
          "{\"filters\":
          {\"footer\":
          {\"settings\":
          {\"enable\":1,\"text/html\":
          \"<html><b>Thank you</b> for your business.</html>\"}}}}");
* 另一個篩選器範例就是點擊追蹤。 假設您的電子郵件文字包含超連結 (如下所示)，而您想要追蹤點擊率：

      messagePart.setContent(
          "Hello,
          <p>This is the body of the message. Visit
          <a href='http://www.contoso.com'>http://www.contoso.com</a>.</p>
          Thank you.",
          "text/html");
* 若要啟用點擊追蹤，請使用下列程式碼：

      message.addHeader("X-SMTPAPI",
          "{\"filters\":
          {\"clicktrack\":
          {\"settings\":
          {\"enable\":1}}}}");

## <a name="how-to-update-email-properties"></a>如何：更新電子郵件屬性
某些電子郵件屬性可使用 **set Property** 進行覆寫，或使用 **add Property** 進行附加。

例如，若要指定 **ReplyTo** 地址，請使用下列程式碼：

    InternetAddress addresses[] =
        { new InternetAddress("john@contoso.com"),
          new InternetAddress("wendy@contoso.com") };

    message.setReplyTo(addresses);

若要增加 [副本] 收件者，請使用下列程式碼：

    message.addRecipient(Message.RecipientType.CC, new
    InternetAddress("john@contoso.com"));

## <a name="how-to-use-additional-sendgrid-services"></a>如何：使用其他 SendGrid 服務
SendGrid 提供的網頁式 API 可供從 Azure 應用程式運用其他 SendGrid 功能。 如需完整詳細資料，請參閱 [SendGrid API 文件][SendGrid API documentation]。

## <a name="next-steps"></a>後續步驟
了解 SendGrid 電子郵件服務的基本概念後，請參考下列連結以取得更多資訊。

* 示範在 Azure 部署中使用 SendGrid 的範例：[如何在 Azure 部署中使用 SendGrid 透過 Java 傳送電子郵件](store-sendgrid-java-how-to-send-email-example.md)
* SendGrid Java SDK: <https://sendgrid.com/docs/Code_Examples/java.html>
* SendGrid API 文件：<https://sendgrid.com/docs/API_Reference/index.html>
* Azure 客戶的 SendGrid 特別供應項目：<https://sendgrid.com/windowsazure.html>

[http://sendgrid.com]: https://sendgrid.com
[http://sendgrid.com/pricing.html]: http://sendgrid.com/pricing.html
[http://www.sendgrid.com/azure.html]: https://www.sendgrid.com/windowsazure.html
[http://sendgrid.com/features]: https://sendgrid.com/features
[http://www.oracle.com/technetwork/java/javamail]: http://www.oracle.com/technetwork/java/javamail/index.html
[Filter Settings]: https://sendgrid.com/docs/API_Reference/Web_API/filter_settings.html
[SendGrid API documentation]: https://sendgrid.com/docs/API_Reference/index.html
[http://sendgrid.com/azure.html]: https://sendgrid.com/windowsazure.html
[雲端架構電子郵件服務]: https://sendgrid.com/email-solutions
[交易式電子郵件傳遞]: https://sendgrid.com/transactional-email
