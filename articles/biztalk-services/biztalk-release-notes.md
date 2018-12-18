---
title: Azure BizTalk 服務的版本資訊 | Microsoft Docs
description: 列出 Azure BizTalk 服務的已知問題
services: biztalk-services
documentationcenter: ''
author: msftman
manager: erikre
editor: ''
ms.assetid: f4906fdc-4cd9-4a57-a007-a88c2e51a18f
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2016
ms.author: deonhe
ms.openlocfilehash: db79ee85c19e5f3cdced9b894aca48de4d7d1c42
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576465"
---
# <a name="release-notes-for-azure-biztalk-services"></a>Azure BizTalk 服務的版本資訊

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

Microsoft Azure BizTalk 服務的版本資訊包含此版本已知的問題。

## <a name="whats-new-in-the-november-update-of-biztalk-services"></a>BizTalk 服務 11 月份更新的新增功能
* BizTalk 服務入口網站中可以啟用靜態加密。 請參閱 [在 BizTalk 服務入口網站中啟用靜態加密](https://msdn.microsoft.com/library/azure/dn874052.aspx)。

## <a name="update-history"></a>更新歷程記錄
### <a name="october-update"></a>10 月更新
* 支援組織帳戶：  
  * **情節**：您使用 Microsoft 帳戶註冊了 BizTalk 服務部署 (例如 user@live.com)。 在此情節中，只有 Microsoft 帳戶使用者可以使用 BizTalk 服務入口網站來管理 BizTalk 服務。 無法使用組織帳戶。  
  * **情節**：您使用 Azure Active Directory 中的組織帳戶註冊了 BizTalk 服務部署 (例如 user@fabrikam.com 或 user@contoso.com)。 在此情節中，只有相同組織內的 Azure Active Directory 使用者可以使用 BizTalk 服務入口網站來管理 BizTalk 服務。 無法使用 Microsoft 帳戶。  
* 當您建立 BizTalk 服務時，您會自動在 BizTalk 服務入口網站中註冊。
  * **案例**：您第一次登入 Azure，建立 BizTalk 服務，然後選取 [管理]。 BizTalk 服務入口網站開啟時，BizTalk 服務自會動註冊，並且可用於您的部署。  
    請參閱 [在 BizTalk 服務入口網站註冊和更新 BizTalk 服務部署](https://msdn.microsoft.com/library/azure/hh689837.aspx)。  

### <a name="august-14-update"></a>8 月 14 日更新
* 合約與橋接器分離 – 現在，交易夥伴合約和橋接器在 BizTalk 服務入口網站中會分離。 您現在分開建立合約和橋接器，而在執行階段，橋接器會根據 EDI 訊息中的值解析為合約。 請參閱[在 Azure BizTalk 服務中建立合約](https://msdn.microsoft.com/library/azure/hh689908.aspx)、[使用 BizTalk 服務入口網站建立 EDI 橋接器](https://msdn.microsoft.com/library/azure/dn793986.aspx)、[使用 BizTalk 服務入口網站建立 AS2 橋接器](https://msdn.microsoft.com/library/azure/dn793993.aspx)和[橋接器在執行階段如何解析合約？](https://msdn.microsoft.com/library/azure/dn794001.aspx)  
* 用於建立合約範本的選項已終止。  
* 對於傳送端合約，您現在可以為每個結構描述指定不同的分隔符號集。 此設定是在傳送端合約的通訊協定設定下指定。 如需詳細資訊，請參閱[在 Azure BizTalk 服務中建立 X12 合約](https://msdn.microsoft.com/library/azure/hh689847.aspx)和[在 Azure BizTalk 服務中建立 EDIFACT合約](https://msdn.microsoft.com/library/azure/dn606267.aspx)。 兩個新的實體也為了相同目的而新增至 TPM OM API。 請參閱 [X12DelimiterOverrides](https://msdn.microsoft.com/library/azure/dn798749.aspx) 和 [EDIFACTDelimiterOverride](https://msdn.microsoft.com/library/azure/dn798748.aspx)。  
* 現在支援標準 XSD 建構，包括衍生類型。 請參閱[在您的對應中使用標準 XSD 建構](https://msdn.microsoft.com/library/azure/dn793987.aspx)和[在對應案例和範例中使用衍生類型](https://msdn.microsoft.com/library/azure/dn793997.aspx)。  
* AS2 支援用於訊息簽署的新 MIC 演算法，以及新的加密演算法。 請參閱 [在 Azure BizTalk 服務中建立 AS2合約](https://msdn.microsoft.com/library/azure/hh689890.aspx)。  
  ## <a name="know-issues"></a>已知問題

### <a name="connectivity-issues-after-biztalk-services-portal-update"></a>BizTalk 服務入口網站更新之後的連線能力問題
  當升級 BizTalk 服務來導入服務的變更時，如果 BizTalk 服務入口網站中已開啟，BizTalk 服務入口網站的連線能力可能會發生問題。  
  解決方法是重新啟動瀏覽器、刪除瀏覽器快取，或在私用模式下啟動入口網站。  

### <a name="visual-studio-ide-cannot-locate-the-artifact-if-you-click-an-error-or-warning-in-a-biztalk-services-project"></a>如果您在 BizTalk 服務專案中按一下錯誤或警告，Visual Studio IDE 找不到構件
安裝 Visual Studio 2012 Update 3 RC 1 來修正問題。  

### <a name="custom-binding-project-reference"></a>自訂繫結專案參考
假設 Visual Studio 方案中的 BizTalk 服務專案有下列情形：  

* 在相同的 Visual Studio 方案中，有一個 BizTalk 服務專案和一個自訂繫結專案。 BizTalk 服務專案參考此自訂繫結專案檔。
* BizTalk 服務專案參考自訂繫結/行為 DLL。

您在 Visual Studio 中成功「建置」方案。 然後，您「重建」或「清除」方案。 之後，當您再次重建或清除時，就發生下列錯誤：  
  無法將檔案 <Path to DLL> 複製到 “bin\Debug\FileName.dll”。 由於已有另一個處理序正在使用該檔案，所以無法存取該檔案。  

#### <a name="workaround"></a>因應措施
* 如果已安裝 [Visual Studio 2012 Update 3](https://docs.microsoft.com/visualstudio/releasenotes/vs2012-update3-vs) ，您有下列兩個選項：
  
  * 重新啟動 Visual Studio，或
  * 重新啟動方案。 然後，在方案上只執行「建置」。  
* 如果未安裝 [Visual Studio 2012 Update 3](https://docs.microsoft.com/visualstudio/releasenotes/vs2012-update3-vs) ，請開啟 [工作管理員]，按一下 [處理程序] 索引標籤，按一下 MSBuild.exe 處理程序，然後按一下 [結束處理程序] 按鈕。  

### <a name="routing-to-basichttprelay-endpoints-is-not-supported-from-bridges-and-biztalk-services-portal-if-non-printable-characters-are-promoted-as-http-headers"></a>如果不可列印字元升級為 HTTP 標頭，則不支援從橋接器和 BizTalk 服務入口網站路由傳送至 BasicHttpRelay 端點
如果您在訊息的升級屬性中使用不可列印字元，這些訊息無法路由傳送至使用 BasicHttpRelay 繫結的轉送目的地。 此外，可供追蹤的升級屬性對 blob 而言是 URL 編碼，對目的地而言是未編碼。  

### <a name="mdn-is-sent-asynchronously-even-if-the-send-asynchronous-mdn-option-is-unchecked"></a>即使未核取 [傳送非同步 MDN] 選項，也會以非同步方式傳送 MDN
考慮此情況 - 如果您選取 [傳送非同步 MDN] 核取方塊、指定非同步 MDN 要傳送到的 URL，然後取消核取 [傳送非同步 MDN] 核取方塊，則即使未選取傳送非同步 MDN 的選項，MDN 仍會傳送至指定的 URL。  
解決方法是在取消核取 [傳送非同步 MDN]  核取方塊之前，必須清除指定的 URL，然後部署 AS2 合約。  

### <a name="whitespace-characters-beyond-a-valid-interchange-cause-an-empty-message-to-be-sent-to-the-suspend-endpoint"></a>超出有效交換的空白字元會導致空的訊息傳送至暫停端點
如果有超出 IEA 區段的空格，解譯器會將此情形視為目前交換結束，並查看下一組空白字元當做下一個訊息。 因為這不是有效的交換，您可能會發現一個成功的訊息傳送至路由目的地，還有一個空的訊息傳送至暫停端點。  

### <a name="tracking-in-biztalk-services-portal"></a>在 BizTalk 服務入口網站中追蹤
擷取的追蹤事件以 EDI 訊息處理和任何相互關聯為範圍。 如果訊息在通訊協定階段之外失敗，追蹤會顯示為成功。 在此情況下，請參閱 [追蹤] 的 [詳細資料] 欄下的 [記錄] 區段，以取得錯誤詳細資料。
X12 接收和傳送設定 ([在 Azure BizTalk 服務中建立 X12 合約](https://msdn.microsoft.com/library/azure/hh689847.aspx)) 提供通訊協定階段的相關資訊。  

### <a name="update-agreement"></a>更新合約
BizTalk 服務入口網站可讓您在設定合約時修改身分識別的辨識符號。 這會導致屬性不一致。 比方說，合約使用 ZZ:1234567，而辨識符號為 ZZ:7654321。 在 BizTalk 服務入口網站設定檔設定中，您將 ZZ:1234567 變更為 01:ChangedValue。 您開啟合約，顯示的是 01:ChangedValue 而非 ZZ:1234567。
若要修改身分識別的辨識符號，請刪除合約，更新夥伴設定檔中的 [身分識別]  ，然後重新建立合約。  

> AZURE.WARNING 此行為影響 X12 和 AS2。  
> 
> 

### <a name="as2-attachments"></a>AS2 附件
傳送和接收時不支援 AS2 訊息的附件。 具體來說，將會自動忽略附件，並將訊息內文當成一般 AS2 訊息來處理。  

### <a name="resources-remembering-path"></a>資源：記住路徑
新增 **資源**時，對話視窗可能不記得先前用來新增資源的路徑。 若要記住先前用過的路徑，請嘗試在 Internet Explorer 中將 BizTalk 服務入口網站新增至 [信任的網站]  。  

### <a name="if-you-rename-the-entity-name-of-a-bridge-and-close-the-project-without-saving-changes-opening-the-entity-again-results-in-an-error"></a>如果您重新命名橋接器的實體名稱，然後關閉專案而不儲存變更，則重新開啟實體會導致錯誤
依序假設有下列情況：  

* 將橋接器 (例如 XML 單向橋接器) 新增至 BizTalk 服務專案  
* 指定 [實體名稱] 屬性的值來重新命名橋接器。 這會以您指定的名稱來重新命名相關聯的 .bridgeconfig 檔案。  
* 關閉 .bcs 檔案 (藉由關閉 Visual Studio 中的索引標籤)，而不儲存變更。  
* 從 [方案總管] 再次開啟 .bcs 檔案。  
  您會注意到相關聯的 .bridgeconfig 檔案具有您指定的新名稱，而在設計介面上的實體名稱還是舊名稱。 如果您嘗試按兩下橋接器元件來開啟橋接器設定，您會收到下列錯誤：  
  `‘<old name>’ Entity’s associated file ‘<old name>.bridgeconfig’ does not exist` 若要避免發生這種情況，請確定您會在將 BizTalk 服務專案中的實體重新命名之後儲存變更。  
  
### <a name="biztalk-service-project-builds-successfully-even-if-an-artifact-has-been-excluded-from-a-visual-studio-project"></a>即使已經從 Visual Studio 專案中排除構件，BizTalk 服務專案仍會成功建置
假設您將構件 (例如，XSD 檔案) 新增至 BizTalk 服務專案，在橋接器組態中加入此構件 (例如，將它指定為要求訊息類型)，然後從 Visual Studio 專案中排除它。 在這種情況下，只要已刪除的構件仍留在它加入 Visual Studio 專案時的相同磁碟位置中，則建置專案不會發生任何錯誤。
  
### <a name="the-biztalk-service-project-does-not-check-for-schema-availability-while-configuring-the-bridges"></a>設定橋接器時，BizTalk 服務專案不會檢查結構描述是否可用
在 BizTalk 服務專案中，如果加入至專案的結構描述又匯入另一個結構描述，BizTalk 服務專案不會檢查匯入的結構描述是否已加入至專案。 如果您嘗試建置此專案，則不會發生任何建置錯誤。
  
### <a name="the-response-message-for-a-xml-request-reply-bridge-is-always-of-charset-utf-8"></a>XML 要求-回覆橋接器的回應訊息一律為 UTF-8 字元集
此版本中，來自 XML 要求-回覆橋接器的回應訊息的字元集永遠設定為 UTF-8。
  
### <a name="user-defined-datatypes"></a>使用者定義的資料類型
BizTalk Adapter Service 功能內的 BizTalk Adapter Pack 配接器可以在配接器作業中利用使用者定義的資料類型。
當利用使用者定義的資料類型時，請將檔案 (.dll) 複製到 drive:\Program Files\Microsoft BizTalk Adapter Service\BAServiceRuntime\bin\，或裝載 BizTalk Adapter Service 服務的伺服器上的全域組件快取 (GAC)。 否則，用戶端可能會發生下列錯誤：  
```
<s:Fault xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
<faultcode>s:Client</faultcode>
<faultstring xml:lang="en-US">The UDT with FullName "File, FileUDT, Version=Value, Culture=Value, PublicKeyToken=Value" could not be loaded. Try placing the assembly containing the UDT definition in the Global Assembly Cache.</faultstring>
<detail>
  <AFConnectRuntimeFault xmlns="http://Microsoft.ApplicationServer.Integration.AFConnect/2011" xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
    <ExceptionCode>ERROR_IN_SENDING_MESSAGE</ExceptionCode>
  </AFConnectRuntimeFault>
</detail>
</s:Fault>
```  
  
> [!IMPORTANT]
> 建議使用 GACUtil.exe 將檔案安裝至全域組件快取。 GACUtil.exe 說明如何使用這項工具和 Visual Studio 命令列選項。  
> 
> 

### <a name="restarting-the-biztalk-adapter-service-web-site"></a>重新啟動 BizTalk Adapter Service 網站
安裝 **BizTalk Adapter Service 執行階段*** 會在 IIS 中建立 **BizTalk Adapter Service** 網站，其中包含 **BAService** 應用程式。 **BAService** 應用程式在內部使用轉送繫結，將內部部署服務端點的範圍延伸至雲端。 對於內部部署裝載的服務，只有當內部部署服務啟動時，對應的轉送端點才會註冊在服務匯流排上。  

如果您停止並啟動應用程式，自動啟動應用程式的設定不會有作用。 所以當 **BAService** 停止時，您一律必須改為重新啟動 **BizTalk Adapter Service** 網站。 請勿啟動或停止 **BAService** 應用程式。

### <a name="special-characters-should-not-be-used-for-address-and-entity-names-of-lob-components"></a>LOB 元件的位址和實體名稱中不應該使用特殊字元
您不應該在 LOB 元件的位址和實體名稱中使用特殊字元。 否則，部署 BizTalk 服務專案時會發生錯誤。 如果使用 '%' 之類的字元，BizTalk Adapter Service 網站可能會進入停止狀態，您必須手動啟動它。

### <a name="test-map-with-get-context-property"></a>使用取得內容屬性來測試對應
如果轉換包含**取得內容屬性**對應作業，**測試對應**將會失敗。 暫時的解決方法是以含有虛擬資料的字串串連對應作業，取代 **取得內容屬性** 對應作業。 這將填入目標結構描述，還可讓您測試其他轉換功能。

### <a name="test-map-property-does-not-display"></a>測試對應屬性未顯示
**測試對應** 屬性未顯示在 Visual Studio 中。 如果 [屬性] 視窗和 [方案總管] 視窗未同時停駐，就可能發生這種情形。 若要解決此問題，請停駐 [屬性] 和 [方案總管] 視窗。  

### <a name="datetime-reformat-drop-down-is-grayed-out"></a>日期時間重新格式化下拉式清單變成灰色
當日期時間重新格式化對應作業加入至設計介面並設定時，[格式] 下拉式清單可能會變成灰色。如果電腦顯示器設定為 [中 – 125%] 或 [大 – 150%]，就可能發生這種情形。 若要解決此問題，請使用下列步驟將顯示器設定為 [小 – 100% (預設)]  ：  

1. 開啟 [控制台]，然後按一下 [外觀及個人化]。
2. 按一下 [顯示] 。
3. 按一下 [小 – 100% (預設)]，然後按一下 [套用]。

[格式]  下拉式清單現在應該如預期般運作。

### <a name="duplicate-agreements-in-the-biztalk-services-portal"></a>BizTalk 服務入口網站中的合約重複
請考慮下列狀況：

1. 使用交易夥伴管理 OM API 建立合約。
2. 在 BizTalk 服務入口網站中使用兩個不同的索引標籤開啟合約。
3. 從這兩個索引標籤同時部署合約。
4. 結果，兩個合約都會部署，導致 BizTalk 服務入口網站中出現重複的項目

**因應措施**。 在 BizTalk 服務入口網站中開啟其中一個重複的合約，並解除部署。  

### <a name="bridges-do-not-use-updated-certificate-even-after-a-certificate-has-been-updated-in-the-artifact-store"></a>即使已在構件存放區更新憑證，橋接器也不會使用更新的憑證
請考慮下列情況：  

**案例 1：使用指紋型憑證來保護從橋接器到服務端點的訊息傳輸**  
假設您在 BizTalk 服務專案中使用指紋型憑證。 您在 BizTalk 服務入口網站中以相同名稱但不同指紋更新憑證，但未相應地更新 BizTalk 服務專案。 在這種情況下，橋接器可能會繼續處理訊息，因為較舊的憑證資料可能仍在通道快取中。 隨後，訊息處理就失敗。  

**因應措施**：在 BizTalk 服務專案中更新憑證，並重新部署專案。  

**案例 2：使用名稱型行為來識別憑證，以保護從橋接器到服務端點的訊息傳輸**

假設您在 BizTalk 服務專案中使用名稱型行為來識別憑證。 您在 BizTalk 服務入口網站中更新憑證，但未相應地更新 BizTalk 服務專案。 在這種情況下，橋接器可能會繼續處理訊息，因為較舊的憑證資料可能仍在通道快取中。 隨後，訊息處理就失敗。  

**因應措施**：在 BizTalk 服務專案中更新憑證，並重新部署專案。  

### <a name="bridges-continue-to-process-messages-even-when-the-sql-database-is-offline"></a>即使 SQL 資料庫離線，橋接器還是繼續處理訊息
即使 Microsoft Azure SQL Database (其中儲存執行資訊，例如已部署的構件和管線) 離線，BizTalk 服務橋接器還是繼續處理訊息一段時間。 這是因為 BizTalk 服務使用快取的構件和橋接器組態。
若不想讓橋接器在 SQL Database 離線時處理任何訊息，您可以使用 BizTalk 服務 PowerShell Cmdlet 來停止或暫停 BizTalk 服務。 關於用於管理作業的 Windows PowerShell Cmdlet，請參閱 [Azure BizTalk 服務管理範例](http://go.microsoft.com/fwlink/p/?LinkID=329019) 。  

### <a name="reading-the-xml-message-within-a-bridges-custom-code-component-includes-an-extra-bom-character"></a>在橋接器的自訂程式碼元件內讀取 XML 訊息會包含額外的 BOM 字元
假設您想要在橋接器的自訂程式碼內讀取 XML 訊息。 如果您使用 .NET API System.Text.Encoding.UTF8.GetString(bytes)，輸出中的訊息開頭會包含額外的 BOM 字元。 因此，如果您不希望輸出包含額外的 BOM 字元，您必須使用 ```System.IO.StreamReader().ReadToEnd()```。

### <a name="sending-messages-to-a-bridge-using-wcf-does-not-scale"></a>使用 WCF 將訊息傳送至橋接器不會調整
使用 WCF 傳送至橋接器的訊息不會調整。 如果您想要可調整的用戶端，則應該改為使用 HttpWebRequest。

### <a name="upgrade-token-provider-error-after-upgrading-from-biztalk-services-preview-to-general-availability-ga"></a>升級：從 BizTalk 服務預覽版升級至公開上市 (GA) 後發生權杖提供者錯誤
與作用中批次之間有 EDI 或 AS2 合約。 當 BizTalk 服務從預覽版升級至 GA 時，可能會發生下列情況：

* 錯誤：權杖提供者無法提供安全性權杖。 權杖提供者傳回訊息：無法解析遠端名稱。
* 批次工作已取消。

**因應措施**：當 BizTalk 服務更新至公開上市 (GA) 之後，重新部署合約。  

### <a name="upgrade-toolbox-shows-the-old-bridge-icons-after-upgrading-the-biztalk-services-sdk"></a>升級：升級 BizTalk 服務 SDK 之後，工具箱顯示舊的橋接器圖示
升級舊版的 BizTalk 服務 SDK (以舊的圖示代表橋接器) 之後，工具箱會繼續顯示橋接器的舊圖示。 不過，如果您將橋接器加入至 BizTalk 服務專案設計工具介面，介面會顯示新的圖示。  

**因應措施**。 您可以刪除 <system drive>:\Users\<user>\AppData\Local\Microsoft\VisualStudio\11.0 下的 .tbd 檔案來解決這個問題。  

### <a name="upgrade-biztalk-portal-update-from-preview-to-ga-might-show-an-error-indicating-that-the-edi-capability-is-not-available"></a>升級：BizTalk 入口網站從預覽版更新至 GA 時可能會顯示錯誤，指出 EDI 功能無法使用
如果您在 BizTalk 服務從預覽版升級至 GA 時登入 BizTalk 服務入口網站，您在入口網站上可能會看到下列錯誤：  

這個版本的 Microsoft Azure BizTalk 服務中無法使用此功能。 若要使用這些功能，請切換到適當的版本。  

**解決方式**：登出入口網站，關閉再開啟瀏覽器，然後登入入口網站。  

### <a name="upgrade-new-tracking-data-does-not-show-up-after-biztalk-services-is-upgraded-to-ga"></a>升級：BizTalk 服務升級至 GA 之後，新的追蹤資料未出現
假設您在 BizTalk 服務預覽版訂用帳戶上已部署 XML 橋接器。 您將訊息傳送至橋接器，對應的追蹤資料出現在 BizTalk 服務入口網站。 現在，如果 BizTalk 服務入口網站和 BizTalk 服務執行階段位元升級至 GA，而且您將訊息傳送至先前部署的相同橋接器端點，則升級之後所傳送的訊息不會顯示追蹤資料。  

### <a name="pipelines-versus-bridges"></a>管線與橋接器
在這整份文件，「管線」和「橋接器」這兩個詞彙交替使用。 基本上，兩者都代表同一件事，也就是部署在 BizTalk 服務上的訊息處理單元。  

### <a name="concepts"></a>概念
[BizTalk 服務](https://msdn.microsoft.com/library/azure/hh689864.aspx)   

