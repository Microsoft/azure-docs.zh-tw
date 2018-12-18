---
title: 範本
description: 本主題說明 Azure 通知中樞的範本。
services: notification-hubs
documentationcenter: .net
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: a41897bb-5b4b-48b2-bfd5-2e3c65edc37e
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 3e587bdf0efc7c5b416183640abb19286a5cff31
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33776646"
---
# <a name="templates"></a>範本
## <a name="overview"></a>概觀
範本可讓用戶端應用程式指定它要接收的確切通知格式。 使用範本時，app 可以獲得數個不同的好處，包括下列各項：

* 跨平台的後端
* 個人化的通知
* 用戶端版本獨立性
* 容易當地語系化

本節提供兩個深入的範例，說明如何使用範本來傳送以您在所有平台的所有裝置為目標的跨平台通知，以及如何針對每部裝置將廣播通知個人化。

## <a name="using-templates-cross-platform"></a>跨平台使用範本
傳送推播通知的標準方式是，針對要傳送的每個通知，傳送一個特定的承載給平台通知服務 (WNS、APNS)。 例如，若要傳送警示給 APNS，則承載會是形式如下的 JSON 物件：

    {"aps": {"alert" : "Hello!" }}

若要在「Windows 市集」應用程式上傳送類似的快顯通知訊息，則 XML 承載如下：

    <toast>
      <visual>
        <binding template=\"ToastText01\">
          <text id=\"1\">Hello!</text>
        </binding>
      </visual>
    </toast>

您可以為 MPNS (Windows Phone) 和 GCM (Android) 平台建立類似的承載。

這項要求會強制 app 後端為每個平台產生不同的承載，而有效地讓後端負責 app 展示層的一部分。 一些考量包括當地語系化和圖形配置 (尤其是針對包含各種類型之磚通知的「Windows 市集」應用程式)。

「通知中樞」範本功能可讓用戶端 app 建立特殊的註冊 (稱為範本註冊)，其中除了包含一組標記之外，還包含一個範本。 「通知中樞」範本功能可讓用戶端 app 將裝置與範本建立關聯，不論您使用的是「安裝」(慣用) 還是「註冊」。 在前述的承載範例中，唯一的平台獨立資訊是實際的警示訊息 (Hello!)。 範本是「通知中樞」的一組指示，有關如何針對該特定用戶端 app 的註冊，設定平台獨立訊息的格式。 在上面範例中，平台獨立訊息是單一屬性： **message = Hello!**。

下圖說明整個程序：

![](./media/notification-hubs-templates/notification-hubs-hello.png)

iOS 用戶端 app 註冊的範本如下：

    {"aps": {"alert": "$(message)"}}

對應的「Windows 市集」用戶端 app 範本是：

    <toast>
        <visual>
            <binding template=\"ToastText01\">
                <text id=\"1\">$(message)</text>
            </binding>
        </visual>
    </toast>

請注意，實際的訊息會替代 $(message) 運算式。 這個運算式會指示「通知中樞」在每次傳送訊息給這個特定的註冊時，都建立依循它的訊息並將通用值切換進來。

如果您使用的是「安裝」模型，則安裝 “templates” 機碼會保有多個範本的 JSON。 如果您使用的是「註冊」模型，則用戶端應用程式可以建立多個註冊以使用多個範本；例如，一個範本用於警示訊息，一個範本用於磚更新。 用戶端應用程式也可以混合使用原生註冊 (無範本的註冊) 與範本註冊。

「通知中樞」會針對每個範本傳送一個通知，而不會考慮它們是否屬於相同的用戶端 app。 這種行為可用來將平台獨立通知轉譯成更多的通知。 例如，對「通知中樞」而言，相同的平台獨立訊息可以順暢地在快顯通知警示與磚更新中轉譯，而不需要後端知道它。 某些平台 (例如 iOS) 可能會將在短時間內傳送給相同裝置的多個通知摺疊起來。

## <a name="using-templates-for-personalization"></a>使用範本來進行個人化
使用範本的另一個好處是能夠使用「通知中樞」依每一註冊執行通知個人化。 例如，假設有一個天氣 app，此 app 會顯示含有特定位置天氣狀況的磚。 使用者可以在攝氏或華氏溫度及單日或五日預測之間做選擇。 使用範本時，每個用戶端 app 安裝項可以註冊所需的格式 (1 日攝氏、1 日華氏、 5 日攝氏、 5 日華氏)，然後讓後端傳送一個含有填寫這些範本 (例如，使用攝氏和華氏溫度的 5 日預報) 所需之一切資訊的單一訊息。

使用攝氏溫度的 1 日預報範本如下：

    <tile>
      <visual>
        <binding template="TileWideSmallImageAndText04">
          <image id="1" src="$(day1_image)" alt="alt text"/>
          <text id="1">Seattle, WA</text>
          <text id="2">$(day1_tempC)</text>
        </binding>  
      </visual>
    </tile>

傳送給「通知中樞」的訊息會包含下列所有屬性：

<table border="1">

<tr><td>day1_image</td><td>day2_image</td><td>day3_image</td><td>day4_image</td><td>day5_image</td></tr>

<tr><td>day1_tempC</td><td>day2_tempC</td><td>day3_tempC</td><td>day4_tempC</td><td>day5_tempC</td></tr>

<tr><td>day1_tempF</td><td>day2_tempF</td><td>day3_tempF</td><td>day4_tempF</td><td>day5_tempF</td></tr>
</table><br/>

藉由使用此模式，後端只需傳送單一訊息，而不需儲存 app 使用者的特定個人化選項。 下圖說明這個案例：

![](./media/notification-hubs-templates/notification-hubs-registration-specific.png)

## <a name="how-to-register-templates"></a>如何註冊範本
若要使用「安裝」模型 (慣用) 或「註冊」模型以範本進行註冊，請參閱 [註冊管理](notification-hubs-push-notification-registration-management.md)。

## <a name="template-expression-language"></a>範本運算式語言
範本僅限於 XML 或 JSON 文件格式。 此外，您只能將運算式放在特定的位置；例如，如果是 XML，只能放在節點屬性或值中，如果是 JSON，則只能放在字串屬性值中。

下表顯示範本中允許使用的語言：

| 運算是 | 說明 |
| --- | --- |
| $(prop) |具有指定名稱之事件屬性的參考。 屬性名稱不區分大小寫。 如果屬性不存在，這個運算式就會解析成屬性的文字值或空字串。 |
| $(prop, n) |同上，但文字會明確裁剪成 n 字元，例如 $(title, 20) 會將 title 屬性內容裁剪成 20 個字元。 |
| .(prop, n) |同上，但文字會在裁剪之後，後面加上三個點。 裁剪的字串與字尾的總大小不會超過 n 個字元。 .(title, 20) 搭配 “This is the title line” 輸入屬性會產生 **This is the title...** |
| %(prop) |與 $(name) 類似，但輸出是以 URI 編碼。 |
| #(prop) |用於 JSON 範本 (例如，用於 iOS 與 Android 範本)。<br><br>除了是用於 JSON 範本 (例如，Apple 範本) 之外，此函式的運作方式與先前指定的 $(prop) 完全相同。 在此案例中，如果此函式不是包含在 “{‘,’}” 中 (例如 ‘myJsonProperty’ : ‘#(name)’)，並且其評估結果為 Javascript 格式的數字 (例如 regexp: (0&#124;(&#91;1-9&#93;&#91;0-9&#93;*))(\.&#91;0-9&#93;+)?((e&#124;E)(+&#124;-)?&#91;0-9&#93;+)?)，則輸出 JSON 會是數字。<br><br>例如，‘badge: ‘#(name)’ 會變成 ‘badge’ : 40 (而不是 ‘40‘)。 |
| ‘text’ 或 “text” |常值。 常值包含以單引號或雙引號括住的任意文字。 |
| expr1 + expr2 |將兩個運算式聯結成單一字串的串連運算子 |

運算式可以是前述任一形式。

使用串連時，整個運算式必須包含在 {} 中。 例如 {$(prop) + ‘ - ’ + $(prop2)}。 |

例如，以下的範本不是有效的 XML 範本：

    <tile>
      <visual>
        <binding $(property)>
          <text id="1">Seattle, WA</text>
        </binding>  
      </visual>
    </tile>


如前文所述，使用串連時，運算式必須包含在大括號中。 例如︰

    <tile>
      <visual>
        <binding template="ToastText01">
          <text id="1">{'Hi, ' + $(name)}</text>
        </binding>  
      </visual>
    </tile>

