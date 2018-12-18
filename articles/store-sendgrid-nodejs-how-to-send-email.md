---
title: 如何使用 SendGrid 電子郵件服務 (Node.js) | Microsoft Docs
description: 了解如何在 Azure 使用 SendGrid 電子郵件服務傳送電子郵件。 程式碼範例以 Node.js API 撰寫。
services: ''
documentationcenter: nodejs
author: erikre
manager: wpickett
editor: ''
ms.assetid: cac444b4-26b0-45ea-9c3d-eca28d57dacb
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 01/05/2016
ms.author: erikre
ms.openlocfilehash: 327cea3a24cc47a9cc463b37cc2346ebc475ef7f
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38701850"
---
# <a name="how-to-send-email-using-sendgrid-from-nodejs"></a>如何使用 SendGrid 透過 Node.js 傳送電子郵件
本指南示範如何在 Azure 上透過 SendGrid 電子郵件服務執行常見程式設計工作。 這些範例使用 Node.js API 撰寫。 涵蓋的案例包括**建構電子郵件**、**傳送電子郵件**、**新增附件**、**使用篩選器**及**更新屬性**。 如需有關 SendGrid 及傳送電子郵件的詳細資訊，請參閱 [後續步驟](#next-steps) 一節。

## <a name="what-is-the-sendgrid-email-service"></a>什麼是 SendGrid 電子郵件服務？
SendGrid 是 [雲端架構電子郵件服務]，能提供可靠的 [交易式電子郵件傳遞]、擴充性和即時分析，以及有彈性的 API 來輕鬆進行自訂整合。 常見的 SendGrid 使用案例包括：

* 自動傳送回條給客戶
* 管理通訊群組清單，以便將每月電子傳單和特別供應項目傳送給客戶
* 收集封鎖的電子郵件、客戶的回應情形等項目的即時度量
* 產生報表，協助找出趨勢
* 轉寄客戶查詢
* 透過電子郵件從您的應用程式傳送通知

如需詳細資訊，請參閱 [https://sendgrid.com](https://sendgrid.com)。

## <a name="create-a-sendgrid-account"></a>建立 SendGrid 帳戶
[!INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="reference-the-sendgrid-nodejs-module"></a>參考 SendGrid Node.js 模組
您可以使用下列命令，透過節點封裝管理員 (npm) 安裝適用於 Node.js 的 SendGrid 模組：

    npm install sendgrid

安裝之後，您可以使用下列程式碼要求您應用程式中的模組：

    var sendgrid = require('sendgrid')(sendgrid_username, sendgrid_password);

SendGrid 模組會匯出 **SendGrid** 和 **Email** 函數。
**SendGrid** 負責透過 Web API 傳送電子郵件，而 **Email** 則負責封裝電子郵件訊息。

## <a name="how-to-create-an-email"></a>如何：建立電子郵件
使用 SendGrid 模組建立電子郵件訊息涉及先使用 Email 函數建立電子郵件訊息，再使用 SendGrid 函數傳送該電子郵件訊息。 以下是使用 Email 函數建立新訊息的範例：

    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });

您也可以設定 html 屬性，為支援 HTML 訊息的用戶端指定一個 HTML 訊息。 例如︰

    html: This is a sample <b>HTML<b> email message.

同時設定 text 和 html 屬性可以為無法支援 HTML 訊息的用戶端提供正常的文字內容遞補。

如需有關 Email 函數所支援之所有屬性的詳細資訊，請參閱 [sendgrid-nodejs][sendgrid-nodejs]。

## <a name="how-to-send-an-email"></a>如何：傳送電子郵件
使用 Email 函數建立電子郵件訊息之後，您可以使用 SendGrid 所提供的 Web API 進行傳送。 

### <a name="web-api"></a>Web API
    sendgrid.send(email, function(err, json){
        if(err) { return console.error(err); }
        console.log(json);
    });

> [!NOTE]
> 上述範例示範的是傳入一個電子郵件物件和回呼函數，您也可以直接指定電子郵件屬性來直接叫用 send 函數。 例如︰  
> 
> `````
> sendgrid.send({
> to: 'john@contoso.com',
> from: 'anna@contoso.com',
> subject: 'test mail',
> text: 'This is a sample email message.'
> });
> `````
> 
> 

## <a name="how-to-add-an-attachment"></a>如何：新增附件
您可以透過在 **files** 屬性中指定檔案名稱和路徑，將附件新增至訊息中。 下列範例示範如何傳送附件：

    sendgrid.send({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.',
        files: [
            {
                filename:     '',           // required only if file.content is used.
                contentType:  '',           // optional
                cid:          '',           // optional, used to specify cid for inline content
                path:         '',           //
                url:          '',           // == One of these three options is required
                content:      ('' | Buffer) //
            }
        ],
    });

> [!NOTE]
> 使用 **files** 屬性時，必須要能夠透過 [fs.readFile](http://nodejs.org/docs/v0.6.7/api/fs.html#fs.readFile) 存取檔案。 如果您想要附加的檔案裝載於 Azure 儲存體中 (例如 Blob 容器中)，您就必須先將該檔案複製到本機儲存體或 Azure 磁碟機，才能使用 **files** 屬性以附件形式傳送它。
> 
> 

## <a name="how-to-use-filters-to-enable-footers-and-tracking"></a>如何：使用篩選器來啟用頁尾和追蹤
SendGrid 提供了運用篩選器的其他電子郵件功能。 這些設定可新增到電子郵件以啟用特定功能，例如啟用點擊追蹤、Google 分析、訂閱追蹤等。 如需完整的篩選器清單，請參閱[篩選器設定][Filter Settings]。

您可以使用 **filters** 屬性在訊息套用篩選器。
每個篩選器都是由包含篩選器特定設定的雜湊來指定。
下列範例示範頁尾和點選追蹤篩選器：

### <a name="footer"></a>頁尾
    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });

    email.setFilters({
        'footer': {
            'settings': {
                'enable': 1,
                'text/plain': 'This is a text footer.'
            }
        }
    });

    sendgrid.send(email);

### <a name="click-tracking"></a>點選追蹤
    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });

    email.setFilters({
        'clicktrack': {
            'settings': {
                'enable': 1
            }
        }
    });

    sendgrid.send(email);

## <a name="how-to-update-email-properties"></a>如何：更新電子郵件屬性
某些電子郵件屬性可使用 **set*Property*** 進行覆寫，或使用 **add*Property*** 進行附加。 例如，您可以使用下列方式新增其他收件者：

    email.addTo('jeff@contoso.com');

或使用下列方式設定篩選器：

    email.addFilter('footer', 'enable', 1);
    email.addFilter('footer', 'text/html', '<strong>boo</strong>');

如需詳細資訊，請參閱 [sendgrid-nodejs][sendgrid-nodejs]。

## <a name="how-to-use-additional-sendgrid-services"></a>如何：使用其他 SendGrid 服務
SendGrid 提供的網頁式 API 可供從 Azure 應用程式運用其他 SendGrid 功能。 如需完整詳細資料，請參閱 [SendGrid API 文件][SendGrid API documentation]。

## <a name="next-steps"></a>後續步驟
了解 SendGrid 電子郵件服務的基本概念後，請參考下列連結以取得更多資訊。

* SendGrid Node.js 模組存放機制：[sendgrid-nodejs][sendgrid-nodejs]
* SendGrid API 文件：<https://sendgrid.com/docs>
* Azure 客戶的 SendGrid 特別供應項目：[http://sendgrid.com/azure.html](https://sendgrid.com/windowsazure.html)

[special offer]: https://sendgrid.com/windowsazure.html
[sendgrid-nodejs]: https://github.com/sendgrid/sendgrid-nodejs
[Filter Settings]: https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html
[SendGrid API documentation]: https://sendgrid.com/docs
[雲端架構電子郵件服務]: https://sendgrid.com/email-solutions
[交易式電子郵件傳遞]: https://sendgrid.com/transactional-email
