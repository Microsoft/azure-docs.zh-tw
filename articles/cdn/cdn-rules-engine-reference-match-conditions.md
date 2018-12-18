---
title: Azure CDN 規則引擎比對條件 | Microsoft Docs
description: Azure 內容傳遞網路規則引擎比對條件的參考文件。
services: cdn
documentationcenter: ''
author: Lichard
manager: akucer
editor: ''
ms.assetid: 669ef140-a6dd-4b62-9b9d-3f375a14215e
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/21/2017
ms.author: rli
ms.openlocfilehash: f8dac5469e7160fae93e8251ab7f4195a383f8b4
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30173316"
---
# <a name="azure-cdn-rules-engine-match-conditions"></a>Azure CDN 規則引擎比對條件 
本文會針對 Azure 內容傳遞網路 (CDN) [規則引擎](cdn-rules-engine.md)列出可用比對條件的詳細描述。

規則的第二個部分是比對條件。 比對條件會識別特定類型的要求，系統將針對這類要求執行一組功能。

例如，您可以使用比對條件：
- 篩選特定位置的內容要求。
- 篩選從特定 IP 位址或國家/地區產生的要求。
- 依標頭資訊篩選要求。

## <a name="always-match-condition"></a>一律比對條件

「一律」比對條件可將一組預設功能套用至所有要求。

Name | 目的
-----|--------
[一律](#always) | 將一組預設功能套用至所有要求。

## <a name="device-match-condition"></a>裝置比對條件

「裝置」比對條件可識別從行動裝置根據其屬性所提出的要求。  

Name | 目的
-----|--------
[裝置](#device) | 識別從行動裝置根據其屬性所提出的要求。

## <a name="location-match-conditions"></a>位置比對條件

「位置」比對條件可根據要求者的位置識別要求。

Name | 目的
-----|--------
[AS 號碼](#as-number) | 識別源自特定網路的要求。
[國家/地區](#country) | 識別源自特定國家/地區的要求。

## <a name="origin-match-conditions"></a>原點比對條件

「原點」比對條件可識別指向內容傳遞網路儲存體或客戶原始伺服器的要求。

Name | 目的
-----|--------
[CDN 原點](#cdn-origin) | 識別儲存在內容傳遞網路儲存體中的內容要求。
[客戶原點](#customer-origin) | 識別對儲存於特定客戶原始伺服器上之內容的要求。

## <a name="request-match-conditions"></a>要求比對條件

「要求」比對條件可根據要求的屬性識別它們。

Name | 目的
-----|--------
[用戶端 IP 位址](#client-ip-address) | 識別源自特定 IP 位址的要求。
[Cookie 參數](#cookie-parameter) | 檢查與每個適用於指定值之要求相關聯的 Cookie。
[Cookie 參數 Regex](#cookie-parameter-regex) | 檢查與每個適用於規則運算式之要求相關聯的 Cookie。
[邊緣 Cname](#edge-cname) | 識別指向特定邊緣 CNAME 的要求。
[參考網域](#referring-domain) | 識別從指定主機名稱轉介的要求。
[要求標頭常值](#request-header-literal) | 識別包含設為指定值之指定標頭的要求。
[要求標頭 Regex](#request-header-regex) | 識別包含指定標頭的要求，該標頭已設定為符合特定規則運算式的值。
[要求標頭萬用字元](#request-header-wildcard) | 識別包含指定標頭的要求，該標頭已設定為符合特定模式的值。
[要求方法](#request-method) | 依其 HTTP 方法來識別要求。
[要求配置](#request-scheme) | 依其 HTTP 通訊協定來識別要求。

## <a name="url-match-conditions"></a>URL 比對條件

「URL」比對條件可根據要求的 URL 識別它們。

Name | 目的
-----|--------
[URL 路徑目錄](#url-path-directory) | 依其相對路徑來識別要求。
[URL 路徑副檔名](#url-path-extension) | 依其副檔名來識別要求。
[URL 路徑檔案名稱](#url-path-filename) | 依其檔案名稱來識別要求。
[URL 路徑常值](#url-path-literal) | 比較要求的相對路徑與指定的值。
[URL 路徑 Regex](#url-path-regex) | 比較要求的相對路徑與指定的規則運算式。
[URL 路徑萬用字元](#url-path-wildcard) | 比較要求的相對路徑與指定的模式。
[URL 查詢常值](#url-query-literal) | 比較要求的查詢字串與指定的值。
[URL 查詢參數](#url-query-parameter) | 識別包含指定查詢字串參數的要求，該參數已設定為符合特定模式的值。
[URL 查詢 Regex](#url-query-regex) | 識別包含指定查詢字串參數的要求，該參數已設定為符合特定規則運算式的值。
[URL 查詢萬用字元](#url-query-wildcard) | 根據要求的查詢字串來比較指定的值。


## <a name="reference-for-rules-engine-match-conditions"></a>規則引擎比對條件的參考

---
### <a name="always"></a>一律

「一律」比對條件可將一組預設功能套用至所有要求。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="as-number"></a>AS 號碼 
AS 號碼網路是由其自發系統編號 (ASN) 所定義。 

[符合]/[不符合] 選項決定符合「AS 號碼」比對條件所依據的條件：
- **符合**：用戶端網路的 ASN 必須符合其中一個指定的 ASN。 
- **不符合**：用戶端網路的 ASN 不得符合任何一個指定的 ASN。

重要資訊：
- 使用單一空格來分隔每個 ASN，藉以指定多個 ASN。 例如，64514 64515 會比對來自 64514 或 64515 的要求。
- 某些要求可能不會傳回有效的 ASN。 問號 (?) 會比對無法判斷有效 ASN 的要求。
- 指定所需網路的整個 ASN。 部分值無法進行比對。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="cdn-origin"></a>CDN 原點
當符合下列兩個條件時，即符合「CDN 原點」比對條件：
- 已要求 CDN 儲存體的內容。
- 要求 URI 使用此比對條件中所定義之內容存取點 (例如 /000001) 的類型：
  - CDN URL：要求 URI 必須包含選取的內容存取點。
  - 邊緣 CNAME URL：對應的邊緣 CNAME 設定必須指向選取的內容存取點。
  
重要資訊：
 - 內容存取點可識別應該提供所要求內容的服務。
 - 請勿使用 AND IF 陳述式來結合特定比對條件。 例如，結合「CDN 原點」比對條件與「客戶原點」比對條件會建立可能永遠無法比對的比對模式。 因此，不可透過 AND IF 陳述式結合兩個「CDN 原點」比對條件。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="client-ip-address"></a>用戶端 IP 位址
[符合]/[不符合] 選項決定符合「用戶端 IP 位址」比對條件所依據的條件：
- **符合**：用戶端的 IP 位址必須符合其中一個指定的 IP 位址。 
- **不符合**：用戶端的 IP 位址不得符合任何一個指定的 IP 位址。 

重要資訊：
- 使用 CIDR 標記法。
- 使用單一空格來分隔每個 IP 位址及/或 IP 位址區塊，藉以指定多個 IP 位址及/或 IP 位址區塊。 例如︰
  - **IPv4 範例**：1.2.3.4 10.20.30.40 會比對來自位址 1.2.3.4 或 10.20.30.40 的任何要求。
  - **IPv6 範例**：1:2:3:4:5:6:7:8 10:20:30:40:50:60:70:80 會比對來自位址 1:2:3:4:5:6:7:8 或 10:20:30:40:50:60:70:80 的任何要求。
- IP 位址區塊的語法是基底 IP 位址，後面接著正斜線和首碼大小。 例如︰
  - **IPv4 範例**：5.5.5.64/26 會比對來自 5.5.5.64 到 5.5.5.127 的任何要求。
  - **IPv6 範例**：1:2:3:/48 會比對來自位址 1:2:3:0:0:0:0:0 到 1:2:3:ffff:ffff:ffff:ffff:ffff 的任何要求。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="cookie-parameter"></a>Cookie 參數
[符合]/[不符合] 選項決定符合「Cookie 參數」比對條件所依據的條件。
- **符合**：要求必須包含指定的 Cookie，且其值至少符合此比對條件中所定義的其中一個值。
- **不符合**：要求必須符合下列任一準則：
  - 它不包含指定的 Cookie。
  - 它包含指定的 Cookie，但其值不符合此比對條件中所定義的任何值。
  
重要資訊：
- Cookie 名稱： 
  - 您在指定 Cookie 名稱時，並不支援包括星號 (*) 等萬用字元值，因此只有精確 Cookie 名稱相符項目才符合比較的資格。
  - 此比對條件的每個執行個體只能指定一個 Cookie 名稱。
  - Cookie 名稱比較不區分大小寫。
- Cookie 值： 
  - 使用單一空格來分隔每個 Cookie 值，藉以指定多個 Cookie 值。
  - Cookie 值可以使用[萬用字元值](cdn-rules-engine-reference.md#wildcard-values)。 
  - 如果未指定萬用字元值，則只有精確相符項目才符合此比對條件。 例如，指定 "Value" 會比對 "Value"，但不會比對 "Value1" 或 "Value2"。
  - 使用 [忽略大小寫] 選項來控制是否對要求的 Cookie 值執行區分大小寫比較。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="cookie-parameter-regex"></a>Cookie 參數 Regex
「Cookie 參數 Regex」比對條件會定義 Cookie 名稱和值。 您可以使用[規則運算式](cdn-rules-engine-reference.md#regular-expressions)來定義所需的 Cookie 值。 

[符合]/[不符合] 選項決定符合「Cookie 參數 RegEx」比對條件所依據的條件。
- **符合**：要求必須包含指定的 Cookie，且其值符合指定的規則運算式。
- **不符合**：要求必須符合下列任一準則：
  - 它不包含指定的 Cookie。
  - 它包含指定的 Cookie，但其值不符合指定的規則運算式。
  
重要資訊：
- Cookie 名稱： 
  - 您在指定 Cookie 名稱時，並不支援規則運算式和包括星號 (*) 等萬用字元值，因此只有精確 Cookie 名稱相符項目才符合比較的資格。
  - 此比對條件的每個執行個體只能指定一個 Cookie 名稱。
  - Cookie 名稱比較不區分大小寫。
- Cookie 值： 
  - Cookie 值可以使用規則運算式。
  - 使用 [忽略大小寫] 選項來控制是否對要求的 Cookie 值執行區分大小寫比較。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

--- 
### <a name="country"></a>國家 (地區)
您可以透過國碼 (地區碼) 來指定國家/地區。 

[符合]/[不符合] 選項決定符合「國家/地區」比對條件所依據的條件：
- **符合**：要求必須包含指定的國碼 (地區碼) 值。 
- **不符合**：要求不得包含指定的國碼 (地區碼) 值。

重要資訊：
- 使用單一空格來分隔每個國碼 (地區碼)，藉以指定多個國碼 (地區碼)。
- 當您指定國碼 (地區碼) 時，不支援萬用字元。
- 國碼 (地區碼) "EU" 和 "AP" 不包含這些區域中的所有 IP 位址。
- 某些要求可能不會傳回有效的國碼 (地區碼)。 問號 (?) 會比對無法判斷有效國碼 (地區碼) 的要求。
- 國碼 (地區碼) 會區分大小寫。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

#### <a name="implementing-country-filtering-by-using-the-rules-engine"></a>使用規則引擎來實作「國家/地區篩選條件」
此比對條件可讓您以要求的來源位置為基礎，執行許多自訂項目。 例如，您可以透過下列設定來複寫「國家/地區篩選條件」功能的行為：

- URL 路徑萬用字元比對：將 [URL 路徑萬用字元比對條件](#url-path-wildcard)設定為要保護的目錄。 
    將星號附加至相對路徑的結尾，以確保此規則會限制對其所有子系的存取。

- 國家/地區比對：將「國家/地區」比對條件設為一組所需的國家/地區。
   - 允許：將「國家/地區」比對條件設為 [不符合]，僅允許指定的國家/地區才能存取儲存在 URL 路徑萬用字元比對條件所定義之位置中的內容。
   - 封鎖：將「國家/地區」比對條件設為 [符合]，以封鎖指定的國家/地區存取儲存在 URL 路徑萬用字元比對條件所定義之位置中的內容。

- 拒絕存取 (403) 功能：啟用[拒絕存取 (403) 功能](cdn-rules-engine-reference-features.md#deny-access-403)以複寫「國家/地區篩選條件」功能的允許或封鎖部分。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="customer-origin"></a>客戶原點

重要資訊： 
- 不論內容的要求方式是透過 CDN URL 或指向所選客戶原點的邊緣 CNAME URL，都符合「客戶原點」比對條件。
- 您無法從 [客戶原點] 頁面刪除規則所參考的客戶原點設定。 嘗試刪除客戶原點設定之前，請確定下列設定未參考它：
  - 客戶原點比對條件
  - 邊緣 CNAME 設定
- 請勿使用 AND IF 陳述式來結合特定比對條件。 例如，結合「客戶原點」比對條件與「CDN 原點」比對條件會建立可能永遠無法比對的比對模式。 因此，不可透過 AND IF 陳述式結合兩個「客戶原點」比對條件。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="device"></a>裝置

「裝置」比對條件可識別從行動裝置根據其屬性所提出的要求。 行動裝置偵測可以透過 [WURFL](http://wurfl.sourceforge.net/) 達成。 

[符合]/[不符合] 選項決定符合「裝置」比對條件所依據的條件：
- **符合**：要求者的裝置必須符合指定的值。 
- **不符合**：要求者的裝置不得符合指定的值。

重要資訊：

- 使用 [忽略大小寫] 選項來指定所指定的值是否區分大小寫。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

#### <a name="string-type"></a>字串類型
WURFL 功能通常會接受數字、字母和符號的任何組合。 由於這項功能具有彈性的本質，您必須選擇如何解譯與此比對條件相關聯的值。 下表描述可用選項組：

類型     | 說明
---------|------------
常值  | 選取此選項可防止大部分字元使用其[常值](cdn-rules-engine-reference.md#literal-values)來採用特殊的意義。
萬用字元 | 選取此選項以使用所有 [萬用字元] [萬用字元值](cdn-rules-engine-reference.md#wildcard-values)。
Regex    | 選取此選項可使用[規則運算式](cdn-rules-engine-reference.md#regular-expressions)。 規則運算式適用於定義字元的模式。

#### <a name="wurfl-capabilities"></a>WURFL 功能
WURFL 功能是指說明行動裝置的類別。 選取的功能會決定要用來識別要求的行動裝置描述之類型。

下表列出規則引擎的 WURFL 功能及其變數。
<br>
> [!NOTE] 
> 下列變數在 [修改用戶端要求標頭] 和 [修改用戶端回應標頭] 功能中受到支援。

功能 | 變數 | 說明 | 範例值
-----------|----------|-------------|----------------
品牌名稱 | %{wurfl_cap_brand_name} | 字串，表示裝置的品牌名稱。 | Samsung
裝置作業系統 | %{wurfl_cap_device_os} | 字串，表示裝置上安裝的作業系統。 | iOS
裝置作業系統版本 | %{wurfl_cap_device_os_version} | 字串，表示裝置上安裝的作業系統版本號碼。 | 1.0.1
雙向 | %{wurfl_cap_dual_orientation} | 布林值，表示裝置是否支援雙向。 | true
HTML 慣用 DTD | %{wurfl_cap_html_preferred_dtd} | 字串，表示 HTML 內容的行動裝置慣用文件類型定義 (DTD)。 | None<br/>xhtml_basic<br/>html5
內嵌影像 | %{wurfl_cap_image_inlining} | 布林值，表示裝置是否支援 Base64 編碼映像。 | false
是 Android | %{wurfl_vcap_is_android} | 布林值，表示裝置是否使用 Android 作業系統。 | true
是 iOS | %{wurfl_vcap_is_ios} | 布林值，表示裝置是否使用 iOS。 | false
是智慧型電視 | %{wurfl_cap_is_smarttv} | 布林值，表示裝置是否為智慧型電視。 | false
是智慧型手機 | %{wurfl_vcap_is_smartphone} | 布林值，表示裝置是否為智慧型手機。 | true
是平板電腦 | %{wurfl_cap_is_tablet} | 布林值，表示裝置是否為平板電腦。 這項描述與作業系統無關。 | true
是無線裝置 | %{wurfl_cap_is_wireless_device} | 布林值，表示裝置是否視為無線裝置。 | true
行銷名稱 | %{wurfl_cap_marketing_name} | 字串，表示裝置的行銷名稱。 | BlackBerry 8100 Pearl
行動瀏覽器 | %{wurfl_cap_mobile_browser} | 字串，表示用來向裝置要求內容的瀏覽器。 | Chrome
行動瀏覽器版本 | %{wurfl_cap_mobile_browser_version} | 字串，表示用來向裝置要求內容的瀏覽器版本。 | 31
模型名稱 | %{wurfl_cap_model_name} | 字串，表示裝置的模型名稱。 | s3
漸進式下載 | %{wurfl_cap_progressive_download} | 布林值，指出裝置是否支援音訊和視訊仍在下載時播放。 | true
發行日期 | %{wurfl_cap_release_date} | 字串，表示裝置新增至 WURFL 資料庫的年和月。<br/><br/>格式： `yyyy_mm` | 2013_december
解析高度 | %{wurfl_cap_resolution_height} | 整數，表示裝置的高度，以像素為單位。 | 768
解析寬度 | %{wurfl_cap_resolution_width} | 整數，表示裝置的寬度，以像素為單位。 | 1024

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="edge-cname"></a>邊緣 Cname
重要資訊： 
- 可用的邊緣 CNAME 清單僅限於 [邊緣 CNAME] 頁面上已針對將要設定規則引擎之平台上設定的邊緣 CNAME。
- 嘗試刪除邊緣 CNAME 設定之前，請確定「邊緣 Cname」比對條件未參考它。 您無法從 [邊緣 CNAME] 頁面刪除規則中已定義的邊緣 CNAME 設定。 
- 請勿使用 AND IF 陳述式來結合特定比對條件。 例如，結合「邊緣 Cname」比對條件與「客戶原點」比對條件會建立可能永遠無法比對的比對模式。 因此，不可透過 AND IF 陳述式結合兩個「邊緣 Cname」比對條件。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="referring-domain"></a>轉介網域
與提出內容要求之查閱者建立關聯的主機名稱決定是否符合「參考網域」比對條件。 

[符合]/[不符合] 選項決定符合「參考網域」比對條件所依據的條件：
- **符合**：參考的主機名稱必須符合指定的值。 
- **不符合**：參考的主機名稱不得符合指定的值。

重要資訊：
- 使用單一空格來分隔每個主機名稱，藉以指定多個主機名稱。
- 此比對條件支援[萬用字元值](cdn-rules-engine-reference.md#wildcard-values)。
- 如果指定的值不包含星號，則必須完全符合查閱者的主機名稱。 例如，指定 "mydomain.com" 不會比對 "www.mydomain.com"。
- 使用 [忽略大小寫] 選項來控制是否要執行區分大小寫比較。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---  
### <a name="request-header-literal"></a>要求標頭常值
[符合]/[不符合] 選項決定符合「要求標頭常值」比對條件所依據的條件。
- **符合**：要求必須包含指定的標頭。 其值必須符合此比對條件中所定義的值。
- **不符合**：要求必須符合下列任一準則：
  - 它不包含指定的標頭。
  - 它包含指定的標頭，但其值不符合此比對條件中所定義的標頭。
  
重要資訊：
- 標頭名稱比較一律不區分大小寫。 使用 [忽略大小寫] 選項來控制標頭值比較是否區分大小寫。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---  
### <a name="request-header-regex"></a>要求標頭 Regex
[符合]/[不符合] 選項決定符合「要求標頭 RegEx」比對條件所依據的條件。
- **符合**：要求必須包含指定的標頭。 其值必須符合指定的[規則運算式](cdn-rules-engine-reference.md#regular-expressions)中所定義的模式。
- **不符合**：要求必須符合下列任一準則：
  - 它不包含指定的標頭。
  - 它包含指定的標頭，但其值不符合指定的規則運算式。

重要資訊：
- 標頭名稱： 
  - 標頭名稱比較不區分大小寫。
  - 將標頭名稱中的空格取代為 "%20"。 
- 標頭值： 
  - 標頭值可以使用規則運算式。
  - 使用 [忽略大小寫] 選項來控制標頭值比較是否區分大小寫。
  - 僅當標頭值精確符合至少其中一個指定的模式時，才符合比對條件。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時 

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="request-header-wildcard"></a>要求標頭萬用字元
[符合]/[不符合] 選項決定符合「要求標頭萬用字元」比對條件所依據的條件。
- **符合**：要求必須包含指定的標頭。 其值必須至少符合此比對條件中所定義的其中一個值。
- **不符合**：要求必須符合下列任一準則：
  - 它不包含指定的標頭。
  - 它包含指定的標頭，但其值不符合任何一個指定的值。
  
重要資訊：
- 標頭名稱： 
  - 標頭名稱比較不區分大小寫。
  - 標頭名稱中的空格應該以 "%20" 取代。 您也可以使用此值來指定標頭值中的空格。
- 標頭值： 
  - 標頭值可以使用[萬用字元值](cdn-rules-engine-reference.md#wildcard-values)。
  - 使用 [忽略大小寫] 選項來控制標頭值比較是否區分大小寫。
  - 當標頭值完精確合至少其中一個指定的模式時，才符合比對條件。
  - 使用單一空格來分隔每個值，藉以指定多個值。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="request-method"></a>要求方法
僅當透過所選要求方法要求資產時，才符合「要求方法」比對條件。 可用的要求方法包括：
- GET
- HEAD 
- POST 
- OPTIONS 
- PUT 
- 刪除 
- TRACE 
- CONNECT 

重要資訊：
- 根據預設，只有 GET 要求方法可以產生網路上的快取內容。 所有其他要求方法則是透過網路進行 Proxy 處理。
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="request-scheme"></a>要求配置
僅當透過所選通訊協定要求資產時，才符合「要求配置」比對條件。 可用的通訊協定為： 
- HTTP
- HTTPS

重要資訊：
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
  - 完成快取填滿
  - 預設的內部最大壽命
  - 強制執行內部最大壽命
  - 忽略原始的 No-Cache
  - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-path-directory"></a>URL 路徑目錄
依要求的相對路徑來加以識別，但不包括所要求資產的檔案名稱。

[符合]/[不符合] 選項決定符合「URL 路徑目錄」比對條件所依據的條件。
- **符合**：要求必須包含相對的 URL 路徑，但不包括符合指定 URL 模式的檔案名稱。
- **不符合**：要求必須包含相對的 URL 路徑，但不包括不符合指定 URL 模式的檔案名稱。

重要資訊：
- 使用 [相對於] 選項以指定 URL 比較要在內容存取點之前還是之後開始。 內容存取點是路徑的一部分，會出現在 Verizon CDN 主機名稱和所要求資產的相對路徑 (例如，/800001/CustomerOrigin) 之間。 它會依伺服器類型 (例如，CDN 或客戶原點) 和您的客戶帳戶編號來識別位置。

   下列的值可供 [相對於] 選項使用：
   - **根**：表示 URL 比較點會緊接在 CDN 主機名稱之後。 

     例如：http:\//wpc.0001.&lt;domain&gt;/**800001/myorigin/myfolder**/index.htm

   - **原點**：表示 URL 比較點會在內容存取點 (例如，/000001 或 /800001/myorigin) 之後。 相對於原點目錄，\*.azureedge.net CNAME 依預設會建立在 Verizon CDN 主機名稱上，因此 Azure CDN 使用者應該使用**原點**值。 

     例如：https:\//&lt;endpoint&gt;.azureedge.net/**myfolder**/index.htm 

     此 URL 會指向下列 Verizon CDN 主機名稱：http:\//wpc.0001.&lt;domain&gt;/800001/myorigin/**myfolder**/index.htm

- 在 URL 比較之前，邊緣 CNAME URL 會重寫為 CDN URL。

    例如，以下這兩個 URL 都指向相同的資產，因此具有相同的 URL 路徑。
    - CDN URL：http:\//wpc.0001.&lt;domain&gt;/800001/CustomerOrigin/path/asset.htm
    
    - 邊緣 CNAME URL：http:\//&lt;endpoint&gt;.azureedge.net/path/asset.htm
    
    其他資訊：
    - 自訂網域：https:\//my.domain.com/path/asset.htm
    
    - URL 路徑 (相對於根)：/800001/CustomerOrigin/path/
    
    - URL 路徑 (相對於原點)：/path/

- 用於 URL 比較的 URL 部分剛好結束於所要求資產的檔案名稱之前。 這種類型路徑中的最後一個字元是句尾斜線。
    
- 將 URL 路徑模式中的任何空格取代為 "%20"。
    
- 每個 URL 路徑模式可包含一個或多個星號 (*)，其中每個星號都符合一個或多個字元序列。
    
- 使用單一空格來分隔模式中的每個 URL 路徑，藉以指定模式中的多個 URL 路徑。

    例如：*/sales/ */marketing/

- URL 路徑規格可以使用[萬用字元值](cdn-rules-engine-reference.md#wildcard-values)。

- 使用 [忽略大小寫] 選項來控制是否要執行區分大小寫比較。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-path-extension"></a>URL 路徑的副檔名
依所要求資產的副檔名來識別要求。

[符合]/[不符合] 選項決定符合「URL 路徑副檔名」比對條件所依據的條件。
- **符合**：要求的 URL 必須包含精確符合指定之模式的副檔名。

   例如，如果您指定 "htm"，就會比對 "htm" 資產，但不會比對 "html" 資產。  

- **不符合**：要求的 URL 必須包含不符合指定之模式的副檔名。

重要資訊：
- 指定要在 [值] 方塊中比對的副檔名。 請勿包含前置句號；例如，使用 htm 而不是 .htm。

- 使用 [忽略大小寫] 選項來控制是否要執行區分大小寫比較。

- 使用單一空格來分隔每個副檔名，藉以指定多個副檔名。 

    例如：htm html

- 例如，指定 "htm" 會比對 "htm" 資產，但不會比對 "html" 資產。


#### <a name="sample-scenario"></a>範例案例

下列範例設定會假設當要求符合其中一個指定的副檔名時，就符合比對條件。

值規格：asp aspx php html

當此比對條件找到以下列副檔名結尾的 URL 時，就符合此比對條件：
- .asp
- .aspx
- .php
- .html

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-path-filename"></a>URL 路徑的檔案名稱
依所要求資產的檔案名稱來識別要求。 基於這個比對條件的目的，檔案名稱會包含所要求資產的名稱、句號和副檔名 (例如，index.html)。

[符合]/[不符合] 選項決定符合「URL 路徑檔名」比對條件所依據的條件。
- **符合**：要求必須在其 URL 路徑中包含符合指定之模式的檔案名稱。
- **不符合**：要求必須在其 URL 路徑中包含不符合指定之模式的檔案名稱。

重要資訊：
- 使用 [忽略大小寫] 選項來控制是否要執行區分大小寫比較。

- 若要指定多個副檔名，請使用單一空格來分隔每個副檔名。

    例如：index.htm index.html

- 將檔案名稱值中的空格取代為 "%20"。
    
- 檔案名稱值可以使用[萬用字元值](cdn-rules-engine-reference.md#wildcard-values)。 例如，每個檔案名稱模式可包含一個或多個星號 (*)，其中每個星號都符合一個或多個字元序列。
    
- 如果未指定萬用字元，則只有精確相符項目才符合此比對條件。

    例如，指定 "presentation.ppt" 會比對名為 "presentation.ppt" 的資產，但不會比對名為 "presentation.pptx" 的資產。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-path-literal"></a>URL 路徑常值
比較要求的 URL 路徑 (包括檔案名稱) 與指定的值。

[符合]/[不符合] 選項決定符合「URL 路徑常值」比對條件所依據的條件。
- **符合**：要求必須包含符合指定之模式的 URL 路徑。
- **不符合**：要求必須包含不符合指定之模式的 URL 路徑。

重要資訊：
- 使用 [相對於] 選項以指定 URL 比較點要在內容存取點之前還是之後開始。 

    下列的值可供 [相對於] 選項使用：
     - **根**：表示 URL 比較點會緊接在 CDN 主機名稱之後。

       例如：http:\//wpc.0001.&lt;domain&gt;/**800001/myorigin/myfolder/index.htm**

     - **原點**：表示 URL 比較點會在內容存取點 (例如，/000001 或 /800001/myorigin) 之後。 相對於原點目錄，\*.azureedge.net CNAME 依預設會建立在 Verizon CDN 主機名稱上，因此 Azure CDN 使用者應該使用**原點**值。 

       例如：https:\//&lt;endpoint&gt;.azureedge.net/**myfolder/index.htm**

     此 URL 會指向下列 Verizon CDN 主機名稱：http:\//wpc.0001.&lt;domain&gt;/800001/myorigin/**myfolder/index.htm**

- 在 URL 比較之前，邊緣 CNAME URL 會重寫為 CDN URL。

    例如，以下這兩個 URL 都指向相同的資產，因此具有相同的 URL 路徑：
    - CDN URL：http:\//wpc.0001.&lt;domain&gt;/800001/CustomerOrigin/path/asset.htm
    - 邊緣 CNAME URL：http:\//&lt;endpoint&gt;.azureedge.net/path/asset.htm
    
    其他資訊：
    
    - URL 路徑 (相對於根)：/800001/CustomerOrigin/path/asset.htm
   
    - URL 路徑 (相對於原點)：/path/asset.htm

- 會忽略 URL 中的查詢字串。
- 使用 [忽略大小寫] 選項來控制是否要執行區分大小寫比較。
- 系統會以此比對條件所指定的值，與用戶端所提出的精確要求之相對路徑進行比較。

- 若要將對特定目錄所做的所有要求進行比對，請使用 [URL 路徑目錄](#url-path-directory)或 [URL 路徑萬用字元](#url-path-wildcard)比對條件。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-path-regex"></a>URL 路徑 Regex
比較要求的 URL 路徑與指定的[規則運算式](cdn-rules-engine-reference.md#regular-expressions)。

[符合]/[不符合] 選項決定符合「URL 路徑 RegEx」比對條件所依據的條件。
- **符合**：要求必須包含符合指定之規則運算式的 URL 路徑。
- **不符合**：要求必須包含不符合指定之規則運算式的 URL 路徑。

重要資訊：
- 在 URL 比較之前，邊緣 CNAME URL 會重寫為 CDN URL。 
 
    例如，這兩個 URL 都指向相同的資產，因此具有相同的 URL 路徑。

     - CDN URL：http:\//wpc.0001.&lt;domain&gt;/800001/CustomerOrigin/path/asset.htm

     - 邊緣 CNAME URL：http:\//my.domain.com/path/asset.htm
    
    其他資訊：
    
     - URL 路徑：/800001/CustomerOrigin/path/asset.htm

- 會忽略 URL 中的查詢字串。
    
- 使用 [忽略大小寫] 選項來控制是否要執行區分大小寫比較。
    
- URL 路徑中的空格應該以 "%20" 取代。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-path-wildcard"></a>URL 路徑萬用字元
比較要求的相對 URL 路徑與指定的萬用字元模式。

[符合]/[不符合] 選項決定符合「URL 路徑萬用字元」比對條件所依據的條件。
- **符合**：要求必須包含符合指定之萬用字元模式的 URL 路徑。
- **不符合**：要求必須包含不符合指定之萬用字元模式的 URL 路徑。

重要資訊：
- [相對於] 選項：此選項決定 URL 比較點要在內容存取點之前還是之後開始。

   這個選項具有下列值：
     - **根**：表示 URL 比較點會緊接在 CDN 主機名稱之後。

       例如：http:\//wpc.0001.&lt;domain&gt;/**800001/myorigin/myfolder/index.htm**

     - **原點**：表示 URL 比較點會在內容存取點 (例如，/000001 或 /800001/myorigin) 之後。 相對於原點目錄，\*.azureedge.net CNAME 依預設會建立在 Verizon CDN 主機名稱上，因此 Azure CDN 使用者應該使用**原點**值。 

       例如：https:\//&lt;endpoint&gt;.azureedge.net/**myfolder/index.htm**

     此 URL 會指向下列 Verizon CDN 主機名稱：http:\//wpc.0001.&lt;domain&gt;/800001/myorigin/**myfolder/index.htm**

- 在 URL 比較之前，邊緣 CNAME URL 會重寫為 CDN URL。

    例如，以下這兩個 URL 都指向相同的資產，因此具有相同的 URL 路徑：
     - CDN URL：http://wpc.0001.&lt;domain&gt;/800001/CustomerOrigin/path/asset.htm
     - 邊緣 CNAME URL：http:\//&lt;endpoint&gt;.azureedge.net/path/asset.htm
    
    其他資訊：
    
     - URL 路徑 (相對於根)：/800001/CustomerOrigin/path/asset.htm
    
     - URL 路徑 (相對於原點)：/path/asset.htm
    
- 使用單一空格來分隔每個 URL 路徑，藉以指定多個 URL 路徑。

   例如：/marketing/asset.* /sales/\*.htm

- 會忽略 URL 中的查詢字串。
    
- 使用 [忽略大小寫] 選項來控制是否要執行區分大小寫比較。
    
- 將 URL 路徑中的空格取代為 "%20"。
    
- 針對 URL 路徑指定的值可以使用[萬用字元值](cdn-rules-engine-reference.md#wildcard-values)。 每個 URL 路徑模式可包含一個或多個星號 (*)，其中每個星號都可能符合一個或多個字元序列。

#### <a name="sample-scenarios"></a>範例案例

下表中的範例設定會假設當要求符合指定的 URL 模式時，就符合比對條件：

值                   | 相對於    | 結果 
------------------------|----------------|-------
\*/test.html \*/test.php  | 根或原點 | 要求此模式比對任何資料夾中名為 "test.html" 或 "test.php" 的資產。
/80ABCD/origin/text/*   | 根           | 要求的資產符合下列準則時，會比對此模式： <br />- 必須位於名為 "origin" 的客戶原點。 <br />- 相對路徑必須以名為 "text" 的資料夾為開頭。 也就是所要求的資產可以位於 "text" 資料夾中，或是位於該資料夾的其中一個遞迴子資料夾中。
/css/ /js/          | 根或原點 | 此模式會依所有包含 css 或 js 資料夾的 CDN 或邊緣 CNAME URL 進行比對。
*.jpg *.gif *.png       | 根或原點 | 此模式會依所有以 .jpg、.gif 或 .png 結尾的 CDN 或邊緣 CNAME URL 進行比對。 指定此模式的替代方式是使用 [URL 路徑副檔名比對條件](#url-path-extension)。
/images/\* /media/\*      | 來源         | 此模式會依相對路徑開頭為「映像」或「媒體」資料夾的 CDN 或邊緣 CNAME URL 進行比對。 <br />- CDN URL：http:\//wpc.0001.&lt;domain&gt;/800001/myorigin/images/sales/event1.png<br />- 範例邊緣 CNAME URL：http:\//cdn.mydomain.com/images/sales/event1.png

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-query-literal"></a>URL 查詢常值
比較要求的查詢字串與指定的值。

[符合]/[不符合] 選項決定符合「URL 查詢常值」比對條件所依據的條件。
- **符合**：要求必須包含符合指定之查詢字串的 URL 查詢字串。
- **不符合**：要求必須包含不符合指定之查詢字串的 URL 查詢字串。

重要資訊：

- 只有精確查詢字串相符項目符合這項比對條件。
    
- 使用 [忽略大小寫] 選項來控制查詢字串比較是否區分大小寫。
    
- 請勿在查詢字串值文字中包含前置問號 (?)。
    
- 某些字元需要 URL 編碼。 使用百分比符號以 URL 編碼下列字元：

   Character | URL 編碼
   ----------|---------
   空白字元     | %20
   &         | %25

- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
   - 完成快取填滿
   - 預設的內部最大壽命
   - 強制執行內部最大壽命
   - 忽略原始的 No-Cache
   - 內部最大過時

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-query-parameter"></a>URL 查詢參數
識別包含指定之查詢字串參數的要求。 這個參數會設定為符合指定之模式的值。 要求 URL 中的查詢字串參數 (例如，parameter=value) 會決定是否符合此條件。 此比對條件會依查詢字串參數的名稱來識別查詢字串參數，並接受一個或多個參數值的值。 

[符合]/[不符合] 選項決定符合「URL 查詢參數」比對條件所依據的條件。
- **符合**：要求必須包含指定的參數，且其值至少符合此比對條件中所定義的其中一個值。
- **不符合**：要求必須符合下列任一準則：
  - 它不包含指定的參數。
  - 它包含指定的參數，但其值不符合此比對條件中所定義的任何值。

此比對條件提供簡易的方式來指定參數名稱/值的組合。 如果您要比對查詢字串參數，為取得更大的彈性，請考量使用 [URL 查詢萬用字元](#url-query-wildcard)比對條件。

重要資訊：
- 此比對條件的每個執行個體只能指定一個 URL 查詢參數名稱。
    
- 指定參數名稱時，並不支援萬用字元值，因此只有精確參數名稱相符項目才符合比較的資格。
- 參數值可以包含[萬用字元值](cdn-rules-engine-reference.md#wildcard-values)。
   - 每個參數值模式可包含一個或多個星號 (*)，其中每個星號都可能符合一個或多個字元序列。
   - 某些字元需要 URL 編碼。 使用百分比符號以 URL 編碼下列字元：

       Character | URL 編碼
       ----------|---------
       空白字元     | %20
       &         | %25

- 使用單一空格來分隔每個查詢字串參數值，藉以指定多個查詢字串參數值。 當要求包含其中一個指定的名稱/值組合時，才符合此比對條件。

   - 範例 1：

     - 組態:

       ValueA ValueB

     - 此設定符合下列查詢字串參數：

       Parameter1=ValueA
    
       Parameter1=ValueB

   - 範例 2：

     - 組態: 

        Value%20A Value%20B

     - 此設定符合下列查詢字串參數：

       Parameter1=Value%20A

       Parameter1=Value%20B

- 僅當至少其中一個指定的查詢字串名稱/值組合有精確相符項目時，才符合此比對條件。

   如果您使用先前範例中的設定，就不會將名稱/值組合 "Parameter1=ValueAdd" 視為相符。 不過，如果您指定下列任何一個值，它就會比對該名稱/值組合：

   - ValueA ValueB ValueAdd
   - ValueA* ValueB

- 使用 [忽略大小寫] 選項來控制查詢字串比較是否區分大小寫。
    
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
   - 完成快取填滿
   - 預設的內部最大壽命
   - 強制執行內部最大壽命
   - 忽略原始的 No-Cache
   - 內部最大過時

#### <a name="sample-scenarios"></a>範例案例
下列範例會示範這個選項在特定情況中的運作方式：

Name  | 值 |  結果
------|-------|--------
User  | Joe   | 當所要求 URL 的查詢字串為 "?user=joe" 時，會比對此模式。
User  | *     | 當所要求 URL 的查詢字串包含 User 參數時，會比對此模式。
電子郵件 | Joe\* | 當所要求 URL 的查詢字串包含以 "Joe" 為開頭的 Email 參數時，會比對此模式。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-query-regex"></a>URL 查詢 Regex
識別包含指定之查詢字串參數的要求。 這個參數會設定為符合指定之[規則運算式](cdn-rules-engine-reference.md#regular-expressions)的值。

[符合]/[不符合] 選項決定符合「URL 查詢 RegEx」比對條件所依據的條件。
- **符合**：要求必須包含符合指定之規則運算式的 URL 查詢字串。
- **不符合**：要求必須包含不符合指定之規則運算式的 URL 查詢字串。

重要資訊：
- 只有精確符合指定的規則運算式才符合此比對條件。
    
- 使用 [忽略大小寫] 選項來控制查詢字串比較是否區分大小寫。
    
- 基於此選項的目的，查詢字串是從查詢字串的問號 (?) 分隔符號之後的第一個字元為開頭。
    
- 某些字元需要 URL 編碼。 使用百分比符號以 URL 編碼下列字元：

   Character | URL 編碼 | 值
   ----------|--------------|------
   空白字元     | %20          | \%20
   &         | %25          | \%25

   請注意必須逸出百分比符號。

- 要在規則運算式中包含一個反斜線的規則運算式雙逸出特殊字元 (例如，\^$.+)。

   例如︰

   值 | 解譯為 
   ------|---------------
   \\+    | +
   \\\\+   | \\+

- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
   - 完成快取填滿
   - 預設的內部最大壽命
   - 強制執行內部最大壽命
   - 忽略原始的 No-Cache
   - 內部最大過時


[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

---
### <a name="url-query-wildcard"></a>URL 查詢萬用字元
根據要求的查詢字串來比較指定的值。

[符合]/[不符合] 選項決定符合「URL 查詢萬用字元」比對條件所依據的條件。
- **符合**：要求必須包含符合指定之萬用字元值的 URL 查詢字串。
- **不符合**：要求必須包含不符合指定之萬用字元值的 URL 查詢字串。

重要資訊：
- 基於此選項的目的，查詢字串是從查詢字串的問號 (?) 分隔符號之後的第一個字元為開頭。
- 參數值可以包含[萬用字元值](cdn-rules-engine-reference.md#wildcard-values)：
   - 每個參數值模式可包含一個或多個星號 (*)，其中每個星號都可能符合一個或多個字元序列。
   - 某些字元需要 URL 編碼。 使用百分比符號以 URL 編碼下列字元：

     Character | URL 編碼
     ----------|---------
     空白字元     | %20
     &         | %25

- 使用單一空格來分隔每個值，藉以指定多個值。

   例如：Parameter1=ValueA ValueB Parameter1=ValueC&Parameter2=ValueD

- 只有精確符合至少其中一個指定的查詢字串模式才符合此比對條件。
    
- 使用 [忽略大小寫] 選項來控制查詢字串比較是否區分大小寫。
    
- 由於追蹤快取設定的行為，因而導致此比對條件與下列功能不相容：
   - 完成快取填滿
   - 預設的內部最大壽命
   - 強制執行內部最大壽命
   - 忽略原始的 No-Cache
   - 內部最大過時

#### <a name="sample-scenarios"></a>範例案例
下列範例會示範這個選項在特定情況中的運作方式：

 Name                 | 說明
 ---------------------|------------
user=joe              | 當所要求 URL 的查詢字串為 "?user=joe" 時，會比對此模式。
\*user=\* \*optout=\* | 當 CDN URL 查詢包含 user 或 optout 參數時，會比對此模式。

[回到頁首](#match-conditions-for-the-azure-cdn-rules-engine)

</br>

## <a name="next-steps"></a>後續步驟
* [Azure 內容傳遞網路概觀](cdn-overview.md)
* [規則引擎參考](cdn-rules-engine-reference.md)
* [規則引擎條件運算式](cdn-rules-engine-reference-conditional-expressions.md)
* [規則引擎功能](cdn-rules-engine-reference-features.md)
* [使用規則引擎覆寫預設的 HTTP 行為](cdn-rules-engine.md)

