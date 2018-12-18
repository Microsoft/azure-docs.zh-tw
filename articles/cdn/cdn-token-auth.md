---
title: 使用權杖驗證保護 Azure CDN 資產 | Microsoft Docs
description: 了解如何使用權杖驗證來保護對 Azure CDN 資產的存取。
services: cdn
documentationcenter: .net
author: zhangmanling
manager: zhangmanling
editor: ''
ms.assetid: 837018e3-03e6-4f9c-a23e-4b63d5707a64
ms.service: cdn
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 11/17/2017
ms.author: mezha
ms.openlocfilehash: aaec713a7680aeda8317f5af41b9b99bcbdca4b7
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30905141"
---
# <a name="securing-azure-cdn-assets-with-token-authentication"></a>使用權杖驗證保護 Azure CDN 資產

[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="overview"></a>概觀

權杖驗證是一種機制，可讓您防止「Azure 內容傳遞網路」(CDN) 將資產提供給未經授權的用戶端。 執行權杖驗證通常是為了防止內容「熱連結」(hotlinking)，在熱連結中，不同的網站 (例如留言板) 會在未經許可的情況下使用您的資產。 熱連結會影響您的內容傳遞成本。 藉由在 CDN 上啟用權杖驗證，就會由 CDN 邊緣伺服器先驗證要求之後，再由 CDN 傳遞內容。 

## <a name="how-it-works"></a>運作方式

權杖驗證會藉由提出要求需包含權杖值 (其中包含關於要求者的編碼資訊) 的要求，來確認要求是由受信任的網站所產生。 只有在編碼資訊符合需求時，才會將內容提供給要求者，否則會拒絕要求。 您可以使用下列一或多個參數來設定需求：

- Country (國家/地區)︰允許或拒絕來自以其[國碼/地區碼](https://msdn.microsoft.com/library/mt761717.aspx)指定之國家/地區的要求。
- URL：只允許與指定資產或路徑相符的要求。
- Host (主機)︰允許或拒絕在要求標頭中使用指定主機的要求。
- Referrer (訪客來源)︰允許或拒絕來自指定訪客來源的要求。
- IP address (IP 位址)︰只允許來自特定 IP 位址或 IP 子網路的要求。
- Protocol (通訊協定)︰根據用來要求內容的通訊協定來允許或拒絕要求。
- Expiration time (到期時間)︰指派日期和時段，以確保連結只能在有限的時段內維持有效。

如需詳細資訊，請參閱[設定權杖驗證](#setting-up-token-authentication)中每個參數的詳細設定範例。

>[!IMPORTANT] 
> 如果已針對此帳戶上的任何路徑啟用權杖授權，則標準快取模式就會是唯一可用於查詢字串快取的模式。 如需詳細資訊，請參閱[使用查詢字串控制 Azure CDN 快取行為](cdn-query-string-premium.md)。

## <a name="reference-architecture"></a>參考架構

以下工作流程圖說明 CDN 如何使用權帳驗證來與您的 Web 應用程式搭配運作。

![CDN 權杖驗證工作流程](./media/cdn-token-auth/cdn-token-auth-workflow2.png)

## <a name="token-validation-logic-on-cdn-endpoint"></a>CDN 端點上的權杖驗證邏輯
    
以下流程圖說明當 CDN 端點上已設定權杖驗證時，Azure CDN 如何驗證用戶端要求。

![CDN 權杖驗證邏輯](./media/cdn-token-auth/cdn-token-auth-validation-logic.png)

## <a name="setting-up-token-authentication"></a>設定權杖驗證

1. 從 [Azure 入口網站](https://portal.azure.com)，瀏覽至您的 CDN 設定檔，然後選取 [管理] 來啟動補充入口網站。

    ![CDN 設定檔 [管理] 按鈕](./media/cdn-token-auth/cdn-manage-btn.png)

2. 將滑鼠移至 [HTTP 大型] 上，然後選取飛出視窗上的 [權杖驗證]。 接著，您便可以依下列方式設定加密金鑰和加密參數：

    1. 建立一或多個加密金鑰。 加密金鑰會區分大小寫，可包含英數字元的任何組合。 不允許任何其他類型的字元，包括空格。 長度上限是 250 個字元。 為了確保加密金鑰為隨機的，建議使用 [OpenSSL 工具](https://www.openssl.org/)建立金鑰。 

       OpenSSL 工具具有以下語法：

       ```rand -hex <key length>```

       例如︰

       ```OpenSSL> rand -hex 32``` 

       若要避免產生停機時間，請建立主要和備份金鑰。 當主要金鑰正在更新時，可用備份金鑰存取您的內容，讓存取不致於中斷。
    
    2. 在 [Primary Key] \(主要金鑰\) 方塊中輸入唯一加密金鑰，並視需要在 [Backup Key] \(備份金鑰\) 方塊中輸入備份金鑰。

    3. 從每個金鑰的 [最低加密版本] 清單中選取其最低加密版本，然後選取 [更新]：
       - **V2**：指出金鑰可用來產生 2.0 和 3.0 版權杖。 只有當您會將舊的 2.0 版加密金鑰轉換成 3.0 版金鑰，才使用此選項。
       - **V3**：(建議選項) 指出金鑰僅可用來產生 3.0 版權杖。

    ![CDN 權杖驗證安裝識別碼](./media/cdn-token-auth/cdn-token-auth-setupkey.png)
    
    4. 使用加密工具設定加密參數，並產生權杖。 使用加密工具時，您可以根據到期時間、國家/地區、訪客來源、通訊協定及用戶端 IP (任意組合) 來允許或拒絕要求。 雖然組成權杖的參數沒有任何數目和組合上的限制，但是權杖的總長度限制為 512 個字元。 

       ![CDN 加密工具](./media/cdn-token-auth/cdn-token-auth-encrypttool.png)

       請在 [加密工具] 區段中，為下列一或多個加密參數輸入值： 

       > [!div class="mx-tdCol2BreakAll"] 
       > <table>
       > <tr>
       >   <th>參數名稱</th> 
       >   <th>說明</th>
       > </tr>
       > <tr>
       >    <td><b>ec_expire</b></td>
       >    <td>會為權杖指派一個到期時間，在此時間之後，權杖就會過期。 在到期時間之後提交的要求都會遭到拒絕。 這個參數使用 Unix 時間戳記，是以自標準 Unix Epoch `1/1/1970 00:00:00 GMT` 算起的秒數為依據。 (您可以使用線上工具在標準時間與 Unix 時間之間轉換)。> 
       >    例如，如果您想要讓權杖在 `12/31/2016 12:00:00 GMT` 到期，請輸入 Unix 時間戳記值 `1483185600`。 
       > </tr>
       > <tr>
       >    <td><b>ec_url_allow</b></td> 
       >    <td>可讓您為特定資產或路徑量身打造權杖。 若要求的 URL 是以特定的相對路徑開頭，它會限制對該要求的存取。 URL 會區分大小寫。 請利用以逗號分隔每個路徑的方式來輸入多個路徑；請勿加上空格。 您可以視需求而定，設定不同的值來提供不同層級的存取。> 
       >    例如，就 URL `http://www.mydomain.com/pictures/city/strasbourg.png` 而言，會針對下列輸入值允許下列要求： 
       >    <ul>
       >       <li>輸入值 `/`：允許所有要求。</li>
       >       <li>輸入值 `/pictures`：允許下列要求： <ul>
       >          <li>`http://www.mydomain.com/pictures.png`</li>
       >          <li>`http://www.mydomain.com/pictures/city/strasbourg.png`</li>
       >          <li>`http://www.mydomain.com/picturesnew/city/strasbourgh.png`</li>
       >       </ul></li>
       >       <li>輸入值 `/pictures/`：只允許包含 `/pictures/` 路徑的要求。 例如： `http://www.mydomain.com/pictures/city/strasbourg.png`。</li>
       >       <li>輸入值 `/pictures/city/strasbourg.png`：只允許此特定路徑和資產的要求。</li>
       >    </ul>
       > </tr>
       > <tr>
       >    <td><b>ec_country_allow</b></td> 
       >    <td>只允許來自一或多個指定國家/地區的要求。 來自其他所有國家/地區的要求將會遭到拒絕。 對每個國家/地區使用兩個字母 [ISO 3166 國家/地區碼](https://msdn.microsoft.com/library/mt761717.aspx)，並將每個國家/地區以逗號分隔；請勿加上空格。 例如，如果您只想允許來自美國和法國的存取，請輸入 `US,FR`。</td>
       > </tr>
       > <tr>
       >    <td><b>ec_country_deny</b></td> 
       >    <td>拒絕來自一或多個指定國家/地區的要求。 來自其他所有國家/地區的要求將會獲得允許。 實作與 <b>ec_country_allow</b> 參數相同。 如果 <b>ec_country_allow</b> 和 <b>ec_country_deny</b> 參數中同時出現國家/地區碼，<b>ec_country_allow</b> 參數具有高優先順序。</td>
       > </tr>
       > <tr>
       >    <td><b>ec_ref_allow</b></td>
       >    <td>只允許來自指定訪客來源的要求。 訪客來源會識別連結到所要求之資源的網頁 URL。 請勿在參數值中包含通訊協定。>    
       >    以下是允許的輸入類型：
       >    <ul>
       >       <li>主機名稱或主機名稱和路徑。</li>
       >       <li>多個訪客來源。 若要新增多個訪客來源，請以逗號分隔每個訪客來源；請勿加上空格。 如果您指定訪客來源值，但因瀏覽器設定而導致不會在要求中傳送訪客來源資訊，系統預設將會拒絕要求。</li> 
       >       <li>具有遺漏或空白訪客來源資訊的要求。 根據預設，<b>ec_ref_allow</b> 參數會封鎖這些類型的要求。 若要允許這些要求，請輸入文字 "missing"，或輸入空白值 (使用尾端逗號)。</li> 
       >       <li>子網域。 若要允許子網域，請輸入星號 (\*)。 例如，若要允許 `contoso.com` 的所有子網域，請輸入 `*.contoso.com`。</li>
       >    </ul>     
       >    例如，若要允許下列存取要求：來自 `www.contoso.com` 的要求、來自 `contoso2.com` 底下所有子網域的要求，以及訪客來源為空白或遺漏的要求，請輸入 `www.contoso.com,*.contoso.com,missing`。</td>
       > </tr>
       > <tr> 
       >    <td><b>ec_ref_deny</b></td>
       >    <td>拒絕來自指定訪客來源的要求。 實作與 <b>ec_ref_allow</b> 參數相同。 如果 <b>ec_ref_allow</b> 和 <b>ec_ref_deny</b> 參數中同時出現訪客來源，<b>ec_ref_allow</b> 參數具有高優先順序。</td>
       > </tr>
       > <tr> 
       >    <td><b>ec_proto_allow</b></td> 
       >    <td>只允許來自指定通訊協定的要求。 有效值為 `http`、`https` 或 `http,https`。</td>
       > </tr>
       > <tr>
       >    <td><b>ec_proto_deny</b></td>
       >    <td>拒絕來自指定通訊協定的要求。 實作與 <b>ec_proto_allow</b> 參數相同。 如果 <b>ec_proto_allow</b> 和 <b>ec_proto_deny</b> 參數中同時出現通訊協定，<b>ec_proto_allow</b> 參數具有高優先順序。</td>
       > </tr>
       > <tr>
       >    <td><b>ec_clientip</b></td>
       >    <td>僅限指定要求者的 IP 位址才能存取。 支援 IPV4 和 IPV6。 您可以指定單一要求 IP 位址或與指定子網路關聯的 IP 子位址。 例如，`11.22.33.0/22` 可允許來自 IP 位址 11.22.32.1 到 11.22.35.254 的要求。</td>
       > </tr>
       > </table>

    5. 輸入完加密參數值之後，請從 [要加密的金鑰] 清單中選取要加密的金鑰 (如果您已建立主要金鑰和備份金鑰)。
    
    6. 從 [加密版本] 清單中選取加密版本：版本 2 請選 [V2]，版本 3 請選 [V3] \(建議選項)。 

    7. 選取 [加密] 來產生權杖。

    權杖產生之後，會顯示在 [產生的權杖] 方塊中。 若要使用此權杖，將它以查詢字串的形式，附加在 URL 路徑中檔案的結尾。 例如： `http://www.domain.com/content.mov?a4fbc3710fd3449a7c99986b`。
        
    8. 視需要使用解密工具來測試您的權杖，使您能夠檢視權杖的參數。 將權杖值貼到 [Token to Decrypt] \(要解密的權杖\) 方塊中。 從 [要解密的金鑰] 清單中選取要使用的加密金鑰，然後選取 [解密]。

    權杖被解密之後，其參數會顯示在 [原始參數] 方塊中。

    9. 視需要自訂要求遭拒時傳回的回應碼類型。 選取 [已啟用]，然後從 [回應碼] 清單中選取回應碼。 [標頭名稱] 會自動設為 [位置]。 選取 [儲存] 來實作新的回應碼。 針對特定回應碼，您還必須在 [標頭值] 方塊中輸入您錯誤頁面的 URL。 預設選取的回應碼是 **403** (禁止)。 

3. 在 [HTTP 大型] 底下，選取 [規則引擎]。 您需使用此規則引擎來定義路徑，以套用功能、啟用權杖驗證功能，以及啟用其他與權杖驗證相關的功能。 如需詳細資訊，請參閱[規則引擎參考](cdn-rules-engine-reference.md)。

    1. 選取現有規則或建立新規則，以定義您想要套用權杖驗證的資產或路徑。 
    2. 若要在規則啟用權杖驗證，請從 **功能** 清單中選取 **[Token Auth](cdn-rules-engine-reference-features.md#token-auth)** 權杖驗證，然後選取 **已啟用**。 如果您要更新規則，請選取 [更新]，或如果您要建立規則，請選取 [新增]。
        
    ![CDN 規則引擎權杖驗證啟用範例](./media/cdn-token-auth/cdn-rules-engine-enable2.png)

4. 在規則引擎中，您還可以啟用其他權杖驗證相關功能。 若要啟用下列任何功能，請從 [功能] 清單中選取該功能，然後選取 [已啟用]。
    
    - **[Token Auth Denial Code](cdn-rules-engine-reference-features.md#token-auth-denial-code)** \(權杖驗證拒絕碼\)：決定當要求遭拒時傳回給使用者的回應類型。 這裡設定的規則會覆寫權杖型驗證頁面之 [Custom Denial Handling] \(自訂拒絕處理\) 區段中設定的回應碼。

    - **[Token Auth Ignore URL Case](cdn-rules-engine-reference-features.md#token-auth-ignore-url-case)** \(權杖驗證忽略 URL 大小寫\)︰決定用來驗證權杖的 URL 是否區分大小寫。

    - **[Token Auth Parameter](cdn-rules-engine-reference-features.md#token-auth-parameter)** \(權杖驗證參數\)︰會將出現在所要求 URL 中的權杖驗證查詢字串參數重新命名。 
        
    ![CDN 規則引擎權杖驗證設定範例](./media/cdn-token-auth/cdn-rules-engine2.png)

5. 您可以存取 [GitHub](https://github.com/VerizonDigital/ectoken).中的原始程式碼來自訂權杖。
可用的語言包括：
    
   - C
   - C#
   - PHP
   - Perl
   - Java
   - Python 

## <a name="azure-cdn-features-and-provider-pricing"></a>Azure CDN 功能和提供者定價

如需功能的相關資訊，請參閱 [Azure CDN 產品功能](cdn-features.md)。 如需價格的詳細資訊，請參閱[內容傳遞網路的價格](https://azure.microsoft.com/pricing/details/cdn/)。
