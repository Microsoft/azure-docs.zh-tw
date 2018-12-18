---
title: 如何使用 Twilio for Voice and SMS (Ruby) | Microsoft Docs
description: 了解如何在 Azure 上使用 Twilio API 服務撥打電話及傳送簡訊。 程式碼範例以 Ruby 撰寫。
services: ''
documentationcenter: ruby
author: devinrader
manager: twilio
editor: ''
ms.assetid: 60e512f6-fa47-47c0-aedc-f19bb72a1158
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: ruby
ms.topic: article
ms.date: 11/25/2014
ms.author: MicrosoftHelp@twilio.com
ms.openlocfilehash: 41b5383dd319f2cb6fad4316e963f86dd7a4bc61
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39036603"
---
# <a name="how-to-use-twilio-for-voice-and-sms-capabilities-in-ruby"></a>如何在 Ruby 中透過 Twilio 使用語音和簡訊功能
本指南示範如何在 Azure 上透過 Twilio API 服務執行常見的程式設計工作。 涵蓋的案例包括打電話和傳送簡訊 (SMS)。 如需有關如何在應用程式中使用 Twilio 語音和 SMS 的詳細資訊，請參閱 [後續步驟](#NextSteps) 一節。

## <a id="WhatIs"></a>什麼是 Twilio？
Twilio 是一種電話語音 Web 服務 API，能夠讓您使用現有的 Web 語言和技術建立語音和 SMS 應用程式。 Twilio 算是協力廠商服務 (並非 Azure 功能，也並非 Microsoft 產品)。

**Twilio 語音** 可讓應用程式撥打和接聽電話。 **Twilio SMS** 可以讓您的應用程式撰寫和接收 SMS 訊息。 **Twilio Client** 可以讓您的應用程式在現有網際網路連線 (包括行動連線) 中啟用語音通訊。

## 
  <a id="Pricing">
  </a>Twilio 定價和特別供應項目
[Twilio 定價][twilio_pricing] (英文) 提供 Twilio 的定價資訊。 Azure 客戶可獲得[特殊供應項目][special_offer]：免費 1000 則文字簡訊或接聽 1000 分鐘電話。 若要註冊以獲得此供應項目或取得詳細資訊，請造訪 [http://ahoy.twilio.com/azure][special_offer]。  

## <a id="Concepts"></a>概念
Twilio API 是一套為應用程式提供語音和簡訊功能的 RESTful API。 用戶端程式庫有多種語言版本，相關清單請參閱 [Twilio API 程式庫][twilio_libraries]。

### <a id="TwiML"></a>TwiML
TwiML 是一組以 XML 為基礎的指令，可指示 Twilio 如何處理來電或簡訊。

例如，下列 TwiML 會將 **Hello World** 文字轉換成語音。

    <?xml version="1.0" encoding="UTF-8" ?>
    <Response>
       <Say>Hello World</Say>
    </Response>

所有 TwiML 文件皆會以 `<Response>` 作為其根元素。 您可以由此處使用 Twilio 動詞定義應用程式的行為。

### <a id="Verbs"></a>TwiML 動詞
Twilio 動詞是指示 Twilio 應執行哪些 **動作**的 XML 標籤。 例如，**&lt;Say&gt;** 動詞會指示 Twilio 在通話中用語音傳遞訊息。 

以下是 Twilio 動詞清單。

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

如需 Twilio 動詞、屬性和 TwiML 的詳細資訊，請參閱 [TwiML][twiml]。 如需 Twilio API 的詳細資訊，請參閱 [Twilio API][twilio_api]。

## <a id="CreateAccount"></a>建立 Twilio 帳戶
準備取得 Twilio 帳戶時，請至[試用 Twilio][try_twilio] 註冊。 您可以先使用免費帳戶，稍後再升級帳戶。

註冊 Twilio 帳戶時，您會獲得可供應用程式使用的免費電話號碼。 您也會獲得帳戶 SID 和驗證權杖。 兩者皆為呼叫 Twilio API 所需。 為了防止未經授權存取您的帳戶，您妥善保管驗證權杖。 在 [Twilio 帳戶頁面][twilio_account] 的 **ACCOUNT SID** 和 **AUTH TOKEN** 欄位中，分別可檢視您的帳戶 SID 和驗證權杖。

### <a id="VerifyPhoneNumbers"></a>驗證電話號碼
除了 Twilio 提供給您的號碼以外，您也可以驗證您在應用程式中控管使用性的號碼 (也就是您的行動電話或家用電話號碼)。 

如需有關如何驗證電話號碼的詳細資訊，請參閱[管理電話號碼][verify_phone]。

## <a id="create_app"></a>建立 Ruby 應用程式
使用 Twilio 服務且執行於 Azure 的 Ruby 應用程式，與其他使用 Twilio 服務的 Ruby 應用程式並無不同。 雖然 Twilio 服務是以 REST 為基礎，並且可透過數種方式從 Ruby 撥打，但本文的重點是要說明如何搭配使用 Twilio 服務與[適用於 Ruby 的 Twilio 協助程式程式庫][twilio_ruby]。

首先，請[設定新的 Azure Linux VM][azure_vm_setup]，以作為新的 Ruby Web 應用程式主機。 請忽略建立 Rails 應用程式的相關步驟，直接設定 VM。 請確實建立具有外部連接埠 80 和內部連接埠 5000 的端點。

我們在下列範例中將使用 [Sinatra][sinatra]，這對 Ruby 而言是非常簡單的 Web 架構。 但適用於 Ruby 的 Twilio 協助程式程式庫是可以與任何其他 Web 架構搭配運作的，包括 Rails 上的 Ruby。

在您新的 VM 中加入 SSH，並為新的應用程式建立目錄。 請在該目錄中建立名為 Gemfile 的檔案，並將下列程式碼複製到檔案中：

    source 'https://rubygems.org'
    gem 'sinatra'
    gem 'thin'

在命令列上執行 `bundle install`。 這會安裝前述的相依性。 接著，建立名為 `web.rb`的檔案。 Web 應用程式的程式碼會存留於該處。 請將下列程式碼貼到檔案中：

    require 'sinatra'

    get '/' do
        "Hello Monkey!"
    end

此時，您應可執行 `ruby web.rb -p 5000`命令。 這會在連接埠 5000 上啟動小型 Web 伺服器。 您應可在瀏覽器中造訪您為 Azure VM 設定的 URL，而瀏覽至此應用程式。 只要您可在瀏覽器中存取您的 Web 應用程式，您即可開始建置 Twilio 應用程式。

## <a id="configure_app"></a>設定應用程式以使用 Twilio
您可以更新 `Gemfile` 以加入下一行程式碼，將 Web 應用程式設為使用 Twilio 程式庫：

    gem 'twilio-ruby'

在命令列上執行 `bundle install`。 接著，開啟 `web.rb` ，並將此行加入至頂端：

    require 'twilio-ruby'

至此一切皆已就緒，您可以在 Web 應用程式中使用適用於 Ruby 的 Twilio 協助程式程式庫。

## <a id="howto_make_call"></a>作法：撥出電話
下列程式碼將說明如何向外撥打電話。 主要的概念包括使用適用於 Ruby 的 Twilio 協助程式程式庫，來撥打 REST API 電話以及轉譯 TwiML。 請將 **From** 和 **To** 電話號碼換成您的值，在執行程式碼之前，請記得先驗證 Twilio 帳戶的 **From** 電話號碼。

將此函數新增至 `web.md`：

    # Set your account ID and authentication token.
    sid = "your_twilio_account_sid";
    token = "your_twilio_authentication_token";

    # The number of the phone initiating the call.
    # This should either be a Twilio number or a number that you've verified
    from = "NNNNNNNNNNN";

    # The number of the phone receiving call.
    to = "NNNNNNNNNNN";

    # Use the Twilio-provided site for the TwiML response.
    url = "http://yourdomain.cloudapp.net/voice_url";

    get '/make_call' do
      # Create the call client.
      client = Twilio::REST::Client.new(sid, token);

      # Make the call
      client.account.calls.create(to: to, from: from, url: url)
    end

    post '/voice_url' do
      "<Response>
         <Say>Hello Monkey!</Say>
       </Response>"
    end

您在瀏覽器中開啟 `http://yourdomain.cloudapp.net/make_call` 時，將觸發對 Twilio API 的呼叫來撥打電話。 `client.account.calls.create` 中的前兩個參數很容易理解：來電 (`from`) 號碼和去電 (`to`) 號碼。 

第三個參數 (`url`) 是 Twilio 要求取得相關指示以得知在電話接通時應執行何種動作的 URL。 在此案例中，我們設定的 URL (`http://yourdomain.cloudapp.net`) 會傳回簡易的 TwiML 文件，並使用 `<Say>` 動詞來執行文字轉換成語音的動作，對接聽電話的人說出 "Hello Monkey"。

## <a id="howto_recieve_sms"></a>如何：接收 SMS 簡訊
在前述範例中，我們撥打了 **外撥** 電話。 現在，我們要使用 Twilio 在註冊期間提供給我們的電話號碼來處理 **傳入的** 簡訊。

首先，請登入您的 [Twilio 儀表板][twilio_account]。 在頂端的導覽區中按一下「號碼」，然後按一下 Twilio 提供給您的號碼。 您會看見兩個可以設定的 URL。 語音要求 URL 和簡訊要求 URL。 這是在撥打電話或傳送簡訊至您的號碼時，Twilio 所將呼叫的 URL。 這些 URL 也稱為 "Web hook"。

我們想要處理傳入的 SMS 簡訊，因此，要將 URL 更新為 `http://yourdomain.cloudapp.net/sms_url`。 接著，按一下頁面底部的 [儲存變更]。 現在，要在 `web.rb` 中將應用程式程式化，以進行相關處理：

    post '/sms_url' do
      "<Response>
         <Message>Hey, thanks for the ping! Twilio and Azure rock!</Message>
       </Response>"
    end

進行變更後，請確實重新啟動 Web 應用程式。 現在，請拿起電話，傳送簡訊至您的 Twilio 號碼。 您應會立即收到簡訊回應，顯示 "Hey, thanks for the ping! Twilio and Azure rock!"。

## <a id="additional_services"></a>如何：使用其他 Twilio 服務
除了此處所示的範例以外，Twilio 還提供網頁式 API，方便您從 Azure 應用程式中充份利用其他 Twilio 功能。 如需完整詳細資料，請參閱 [Twilio API 文件][twilio_api_documentation]。

### <a id="NextSteps"></a>後續步驟
了解基本的 Twilio 服務之後，請參考下列連結以取得更多資訊：

* [Twilio 安全性方針][twilio_security_guidelines]
* [Twilio 作法與範例程式碼][twilio_howtos]
* [Twilio 快速入門教學課程][twilio_quickstarts] 
* [GitHub 上的 Twilio][twilio_on_github]
* [洽詢 Twilio 支援][twilio_support]

[twilio_ruby]: https://www.twilio.com/docs/ruby/install





[twilio_pricing]: http://www.twilio.com/pricing
[special_offer]: http://ahoy.twilio.com/azure
[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api]: http://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]:  https://www.twilio.com/user/account
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
[twilio_api_documentation]: http://www.twilio.com/api
[twilio_security_guidelines]: http://www.twilio.com/docs/security
[twilio_howtos]: http://www.twilio.com/docs/howto
[twilio_on_github]: https://github.com/twilio
[twilio_support]: http://www.twilio.com/help/contact
[twilio_quickstarts]: http://www.twilio.com/docs/quickstart
[sinatra]: http://www.sinatrarb.com/
[azure_vm_setup]: https://docs.microsoft.com/azure/virtual-machines/linux/classic/ruby-rails-web-app
