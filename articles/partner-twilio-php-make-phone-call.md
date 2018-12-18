---
title: 如何從 Twilio 撥打電話 (PHP) | Microsoft Docs
description: 了解如何在 Azure 上使用 Twilio API 服務撥打電話及傳送簡訊。 範例適用於 PHP 應用程式。
documentationcenter: php
services: ''
author: devinrader
manager: twilio
editor: mollybos
ms.assetid: 44e35adc-be06-4700-beee-8c9e2c20c540
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 11/25/2014
ms.author: microsofthelp@twilio.com
ms.openlocfilehash: 9866a196b3be10548d7a431430e570b41c190fc0
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23039263"
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-php-application-on-azure"></a>如何在 Azure 上的 PHP 應用程式中使用 Twilio 撥打電話
下列範例將說明如何從 Azure 代管的 PHP 網頁上使用 Twilio 撥打電話。 產生的應用程式會提示使用者提供電話值，如下列螢幕擷取畫面所示。

![Azure Call Form Using Twilio and PHP][twilio_php]

您必須執行下列動作才能使用本主題中的程式碼：

1. 從您的 [Twilio 主控台][twilio_console]取得 Twilio 帳戶和驗證權杖。 若要開始使用 Twilio，請在 [http://www.twilio.com/pricing][twilio_pricing] 上評估價格。 您可以在 [https://www.twilio.com/try-twilio][try_twilio] 上註冊試用帳戶。
2. 取得 [適用於 PHP 的 Twilio 程式庫](https://github.com/twilio/twilio-php) ，或以 PEAR 封裝的形式進行安裝。 如需詳細資訊，請參閱 [讀我檔案](https://github.com/twilio/twilio-php/blob/master/README.md)。
3. 安裝 Azure SDK for PHP。 
<!-- For an overview of the SDK and instructions on installing it, see [Set up the Azure SDK for PHP](app-service-web/web-sites-php-mysql-deploy-use-git.md) -->

## <a name="create-a-web-form-for-making-a-call"></a>建立用以撥打電話的 Web 表單
下列 HTML 程式碼將說明如何建置網頁 (**callform.html**)，以擷取撥打電話所需的使用者資料：

```html
<!DOCTYPE html>
<html>
<head>
  <title>Automated call form</title>
</head>
<body>
  <h1>Automated Call Form</h1>
  <p>Fill in all fields and click <b>Make this call</b>.</p>
  <form action="makecall.php" method="post">
    <table>
      <tr>
        <td>To:</td>
        <td><input name="callTo" size="50" type="text" value=""></td>
      </tr>
      <tr>
        <td>From:</td>
        <td><input name="callFrom" size="50" type="text" value=""></td>
      </tr>
      <tr>
        <td>Call message:</td>
        <td><input name="callText" size="100" type="text" value="Hello. This is the call text. Good bye."></td>
      </tr>
      <tr>
        <td colspan="2"><input type="submit" value="Make this call"></td>
      </tr>
    </table>
  </form><br>
</body>
</html>
```

## <a name="create-the-code-to-make-the-call"></a>建立用以撥打電話的程式碼
下列程式碼將說明如何建置會在使用者提交 **callform.html** 所顯示的表單時受到呼叫的 **makecall.php**。 下方顯示的程式碼會建立通話訊息及產生通話。 同時，請務必使用來自 [Twilio 主控台][twilio_console]的 Twilio 帳戶和驗證權杖，而不是下方程式碼中指派給 **$sid** 和 **$token** 的預留位置值。

```html
<html>
<head><title>Making call...</title></head>
<body>
<p>Your call is being made.</p>

<?php
require_once 'path/to/vendor/autoload.php';

$sid   = "your_account_sid";
$token = "your_authentication_token";

$from_number = $_POST['callFrom']; // Calls must be made from a registered Twilio number.
$to_number   = $_POST['callTo'];
$message     = $_POST['callText'];

$client = new Twilio\Rest\Client($sid, $token);

$call = $client->calls->create(
            $to_number,
            $from_number,
            array('url' => http://twimlets.com/message?Message=' . urlencode($message))
        );

echo "Call status: " . $call->status . "<br />";
echo "URI resource: " . $call->uri . "<br />";
?>
</body>
</html>
```

除了撥打電話以外，**makecall.php** 也會顯示某些通話中繼資料，如以下影像中所示。 如需通話中繼資料的詳細資訊，請參閱 [https://www.twilio.com/docs/api/rest/call#instance-properties][twilio_call_properties]。

![Azure Call Response Using Twilio and PHP][twilio_php_response]

## <a name="run-the-application"></a>執行應用程式
下一個步驟是[將應用程式部署至 Azure Web Apps 與 Git](app-service/app-service-web-get-started-php.md) (其中的資訊並非全部與之相關)。 

## <a name="next-steps"></a>後續步驟
此程式可說明在 Azure 上的 PHP 中使用 Twilio 的基本功能。 在部署至生產環境中的 Azure 之前，您可以新增更多錯誤處理或其他功能。 例如︰

* 除了使用 Web 表單以外，您也可以使用 Azure 儲存體 Blob 或 SQL Database 來儲存電話號碼和通話文字。 如需在 PHP 中使用 Azure 儲存體 Blob 的相關資訊，請參閱[搭配使用 Azure 儲存體與 PHP 應用程式][howto_blob_storage_php]。 如需在 PHP 中使用 SQL Database 的相關資訊，請參閱[搭配使用 SQL Database 與 PHP 應用程式][howto_sql_azure_php]。
* **makecall.php** 程式碼會使用 Twilio 提供的 URL ([http://twimlets.com/message][twimlet_message_url]) 來提供 Twilio 標記語言 (TwiML) 回應，以告知 Twilio 應如何執行通話。 例如，傳回的 TwiML 可能會包含 `<Say>` 動詞，而產生要傳達給受話方的文字。 除了使用 Twilio 提供的 URL 以外，您也可以建置自己的服務來回應 Twilio 的要求；如需詳細資訊，請參閱[如何在 PHP 中透過 Twilio 使用語音和簡訊功能][howto_twilio_voice_sms_php]。 如需 TwiML 的詳細資訊，請參閱 [http://www.twilio.com/docs/api/twiml][twiml]；如需 `<Say>` 和其他 Twilio 動詞的詳細資訊，請參閱 [http://www.twilio.com/docs/api/twiml/say][twilio_say]。
* 閱讀 [https://www.twilio.com/docs/security][twilio_docs_security] 上的 Twilio 安全性指引。

如需 Twilio 的其他資訊，請參閱 [https://www.twilio.com/docs][twilio_docs]。

## <a name="see-also"></a>另請參閱
* [如何在 PHP 中透過 Twilio 使用語音和簡訊功能](partner-twilio-php-how-to-use-voice-sms.md)

[twilio_console]: https://www.twilio.com/console
[twilio_pricing]: http://www.twilio.com/pricing
[try_twilio]: http://www.twilio.com/try-twilio
[twilio_api]: http://www.twilio.com/docs/api
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified
[twimlet_message_url]: http://twimlets.com/message
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api_service]: http://api.twilio.com
[build_php_azure_app]: http://azurephp.interoperabilitybridges.com/articles/build-and-deploy-a-windows-azure-php-application
[howto_twilio_voice_sms_php]: partner-twilio-php-how-to-use-voice-sms.md
[howto_blob_storage_php]: http://azure.microsoft.com/documentation/articles/storage-php-how-to-use-blobs/
[howto_sql_azure_php]: http://azure.microsoft.com/documentation/articles/sql-database-php-how-to-use/
[twilio_call_properties]: https://www.twilio.com/docs/api/rest/call#instance-properties
[twilio_docs_security]: http://www.twilio.com/docs/security
[twilio_docs]: http://www.twilio.com/docs
[twilio_say]: http://www.twilio.com/docs/api/twiml/say
[ssl_validation]: http://readthedocs.org/docs/twilio-php/en/latest/usage/rest.html
[twilio_php]: ./media/partner-twilio-php-make-phone-call/WA_TwilioPHPCallForm.jpg
[twilio_php_response]: ./media/partner-twilio-php-make-phone-call/WA_TwilioPHPMakeCall.jpg
[twilio_php_github]: https://github.com/twilio/twilio-php
