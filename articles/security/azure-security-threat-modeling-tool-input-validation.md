---
title: 輸入驗證 - Microsoft 威脅模型化工具 - Azure | Microsoft Docs
description: 降低威脅模型化工具所暴露的威脅
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.openlocfilehash: f49692a97053482f6522fed7bec91db6cac4e15e
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306319"
---
# <a name="security-frame-input-validation--mitigations"></a>安全性架構︰輸入驗證 | 風險降低 
| 產品/服務 | 文章 |
| --------------- | ------- |
| **Web 應用程式** | <ul><li>[停用所有使用未受信任樣式表之轉換的 XSLT 指令碼](#disable-xslt)</li><li>[確定可能包含使用者可控制內容的每個頁面都選擇不要自動探查 MIME](#out-sniffing)</li><li>[強化或停用 XML 實體解析](#xml-resolution)</li><li>[利用 http.sys 的應用程式會執行 URL 標準化驗證](#app-verification)</li><li>[確定在接受使用者的檔案時已備妥適當的控制](#controls-users)</li><li>[確定 Web 應用程式有使用 type-safe 參數來存取資料](#typesafe)</li><li>[使用個別的模型繫結類別或繫結篩選清單來防止 MVC 大量指派弱點](#binding-mvc)</li><li>[先將未受信任的 Web 輸出編碼再進行轉譯](#rendering)</li><li>[對所有字串類型的模型屬性執行輸入驗證和篩選](#typemodel)</li><li>[應該對接受所有字元的表單欄位 (例如 RTF 編輯器) 套用清理](#richtext)</li><li>[請勿將沒有內建編碼的 DOM 元素指派給接收器](#inbuilt-encode)</li><li>[驗證應用程式內的所有重新導向皆已關閉或安全完成](#redirect-safe)</li><li>[對控制器方法所接受的所有字串類型參數實作輸入驗證](#string-method)</li><li>[為規則運算式處理設定逾時上限以防止因規則運算式不正確而導致 DoS](#dos-expression)</li><li>[避免在 Razor 檢視中使用 Html.Raw](#html-razor)</li></ul> | 
| **資料庫** | <ul><li>[請勿在預存程序中使用動態查詢](#stored-proc)</li></ul> |
| **Web API** | <ul><li>[確定 Web API 方法已完成模型驗證](#validation-api)</li><li>[對 Web API 方法所接受的所有字串類型參數實作輸入驗證](#string-api)</li><li>[確定 Web API 有使用 type-safe 參數來存取資料](#typesafe-api)</li></ul> | 
| **Azure Document DB** | <ul><li>[針對 Azure Cosmos DB 使用參數化 SQL 查詢](#sql-docdb)</li></ul> | 
| **WCF** | <ul><li>[透過結構描述繫結驗證 WCF 輸入](#schema-binding)</li><li>[透過參數偵測器驗證 WCF 輸入](#parameters)</li></ul> |

## <a id="disable-xslt"></a>停用所有使用未受信任樣式表之轉換的 XSLT 指令碼

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [XSLT 安全性](https://msdn.microsoft.com/library/ms763800(v=vs.85).aspx)、[XsltSettings.EnableScript 屬性](http://msdn.microsoft.com/library/system.xml.xsl.xsltsettings.enablescript.aspx) |
| **步驟** | XSLT 支援在使用 `<msxml:script>` 元素的樣式表內編寫指令碼。 這可讓自訂函式用於 XSLT 轉換。 指令碼會在執行轉換的程序內容下執行。 在未受信任的環境時必須停用 XSLT 指令碼，以防止執行未受信任的程式碼。 如果使用 .NET：預設會停用 XSLT 指令碼；不過，您必須確認它尚未透過 `XsltSettings.EnableScript` 屬性加以明確啟用。|

### <a name="example"></a>範例 

```csharp
XsltSettings settings = new XsltSettings();
settings.EnableScript = true; // WRONG: THIS SHOULD BE SET TO false
```

### <a name="example"></a>範例
如果使用 MSXML 6.0，預設會停用 XSLT 指令碼；不過，您必須確認它尚未透過 XML DOM 物件屬性 AllowXsltScript 加以明確啟用。 

```csharp
doc.setProperty("AllowXsltScript", true); // WRONG: THIS SHOULD BE SET TO false
```

### <a name="example"></a>範例
如果您使用 MSXML 5 或較低版本，預設會啟用 XSLT 指令碼，因此您必須明確地加以停用。 請將 XML DOM 物件屬性 AllowXsltScript 設為 false。 

```csharp
doc.setProperty("AllowXsltScript", false); // CORRECT. Setting to false disables XSLT scripting.
```

## <a id="out-sniffing"></a>確定可能包含使用者可控制內容的每個頁面都選擇不要自動探查 MIME

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [IE8 安全性單元五 - 完善的保護](http://blogs.msdn.com/ie/archive/2008/07/02/ie8-security-part-v-comprehensive-protection.aspx)  |
| **步驟** | <p>對於可能包含使用者可控制內容的每個頁面，您必須使用 HTTP 標頭 `X-Content-Type-Options:nosniff`。 若要符合此需求，您可以單獨針對可能包含使用者可控制內容的頁面逐頁設定此必要標頭，或者，您也可以針對應用程式中的所有頁面全域設定此標頭。</p><p>從 Web 伺服器傳遞過來之檔案的每個類型都有相關聯的 [MIME 類型](http://en.wikipedia.org/wiki/Mime_type) (也稱為「內容類型」)，以描述內容的本質 (也就是影像、文字、應用程式等)</p><p>X-Content-Type-Options 標頭是可讓開發人員指定不應對其內容探查 MIME 的 HTTP 標頭。 此標頭是設計用來降低 MIME 探查攻擊的風險。 Internet Explorer 8 (IE8) 已新增對此標頭的支援</p><p>只有 Internet Explorer 8 (IE8) 的使用者能夠受益於 X-Content-Type-Options。 舊版 Internet Explorer 目前不採用 X-Content-Type-Options 標頭</p><p>Internet Explorer 8 (和更新版本) 是主要瀏覽器中唯一實作了可選擇不要 MIME 探查功能的瀏覽器。 當其他主要瀏覽器 (Firefox、Safari、Chrome) 實作了類似功能時，我們會更新這項建議，使其也包含這些瀏覽器的語法</p>|

### <a name="example"></a>範例
若要為應用程式中的所有頁面全域啟用必要標頭，您可以執行下列其中一項︰ 

* 如果應用程式是由 Internet Information Services (IIS) 7 所裝載，請在 web.config 檔案中新增標頭 

```
<system.webServer> 
  <httpProtocol> 
    <customHeaders> 
      <add name=""X-Content-Type-Options"" value=""nosniff""/>
    </customHeaders>
  </httpProtocol>
</system.webServer> 
```

* 透過全域 Application\_BeginRequest 新增標頭 

``` 
void Application_BeginRequest(object sender, EventArgs e)
{
  this.Response.Headers[""X-Content-Type-Options""] = ""nosniff"";
} 
```

* 實作自訂 HTTP 模組 

``` 
public class XContentTypeOptionsModule : IHttpModule 
  {
    #region IHttpModule Members 
    public void Dispose() 
    { 

    } 
    public void Init(HttpApplication context)
    { 
      context.PreSendRequestHeaders += newEventHandler(context_PreSendRequestHeaders); 
    } 
    #endregion 
    void context_PreSendRequestHeaders(object sender, EventArgs e) 
      { 
        HttpApplication application = sender as HttpApplication; 
        if (application == null) 
          return; 
        if (application.Response.Headers[""X-Content-Type-Options ""] != null) 
          return; 
        application.Response.Headers.Add(""X-Content-Type-Options "", ""nosniff""); 
      } 
  } 

``` 

* 您可以透過將必要標頭新增至個別回應，來單獨針對特定頁面啟用此標頭︰ 

```
this.Response.Headers[""X-Content-Type-Options""] = ""nosniff""; 
``` 

## <a id="xml-resolution"></a>強化或停用 XML 實體解析

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [XML 實體擴充](http://capec.mitre.org/data/definitions/197.html)、[XML 阻斷服務攻擊與防禦](http://msdn.microsoft.com/magazine/ee335713.aspx)、[MSXML 安全性概觀](http://msdn.microsoft.com/library/ms754611(v=VS.85).aspx)、[保護 MSXML 程式碼的最佳作法](http://msdn.microsoft.com/library/ms759188(VS.85).aspx)、[NSXMLParserDelegate 通訊協定參考](http://developer.apple.com/library/ios/#documentation/cocoa/reference/NSXMLParserDelegate_Protocol/Reference/Reference.html)、[解析外部參考](https://msdn.microsoft.com/library/5fcwybb2.aspx) |
| **步驟**| <p>雖然未被廣泛使用，但有一項 XML 功能可讓 XML 剖析器使用在文件本身內或從外部來源所定義的值，來擴充巨集實體。 例如，文件可能會使用值 "Microsoft" 定義實體 "companyname"，以便在每次文件中出現文字 "&companyname;" 時，就自動以文字 Microsoft 加以取代。 或者，文件可能會定義參考外部 Web 服務的實體 "MSFTStock"，以擷取 Microsoft 股票的目前股價。</p><p>然後，每當文件中出現 "&MSFTStock;" 時，就自動以目前股價加以取代。 不過，這項功能可能會遭到濫用，來造成阻斷服務 (DoS) 狀況。 攻擊者可將多個實體進行巢狀處理，來製造會以指數方式擴張的 XML 炸彈 (XML bomb)，以耗用系統上所有可用的記憶體。 </p><p>或者，他也可以建立外部參考，以往回串流無限量的資料，或讓執行緒停止回應。 因此，所有小組必須完全停用其應用程式並未使用的內部及/或外部 XML 實體解析，或以手動方式限制可供應用程式取用來解析實體的記憶體和時間量 (如果這項功能有其絕對必要)。 如果應用程式不需要解析實體，則請停用。 </p>|

### <a name="example"></a>範例
針對 .NET Framework 程式碼，您可以使用下列方法︰

```csharp
XmlTextReader reader = new XmlTextReader(stream);
reader.ProhibitDtd = true;

XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = true;
XmlReader reader = XmlReader.Create(stream, settings);

// for .NET 4
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;
XmlReader reader = XmlReader.Create(stream, settings);
```
請注意，`ProhibitDtd` 在 `XmlReaderSettings` 中的預設值為 true，但在 `XmlTextReader` 中則為 false。 如果您使用 XmlReaderSettings，則不需要將 ProhibitDtd 明確設為 true，但為了安全起見，建議您這麼做。 也請注意，XmlDocument 類別預設會允許解析實體。 

### <a name="example"></a>範例
若要停用 XmlDocuments 的實體解析，請使用 Load 方法的 `XmlDocument.Load(XmlReader)` 多載，並在 XmlReader 引數中設定適當屬性以停用解析，如下列程式碼所示︰ 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = true;
XmlReader reader = XmlReader.Create(stream, settings);
XmlDocument doc = new XmlDocument();
doc.Load(reader);
```

### <a name="example"></a>範例
如果應用程式無法停用實體解析，請根據應用程式的需求將 XmlReaderSettings.MaxCharactersFromEntities 屬性設為合理的值。 這會限制潛在指數擴張 DoS 攻擊所會造成的影響。 下列程式碼提供此方法的範例︰ 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = false;
settings.MaxCharactersFromEntities = 1000;
XmlReader reader = XmlReader.Create(stream, settings);
```

### <a name="example"></a>範例
如果您需要解析內嵌實體，但不需要解析外部實體，請將 XmlReaderSettings.XmlResolver 屬性設為 null。 例如︰ 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = false;
settings.MaxCharactersFromEntities = 1000;
settings.XmlResolver = null;
XmlReader reader = XmlReader.Create(stream, settings);
```
請注意，在 MSXML6 中，ProhibitDTD 預設會設為 true (停用 DTD 處理)。 針對 Apple OSX/iOS 程式碼，您可以使用的 XML 剖析器有兩個︰NSXMLParser 和 libXML2。 

## <a id="app-verification"></a>利用 http.sys 的應用程式會執行 URL 標準化驗證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>任何使用 http.sys 的應用程式皆應遵循下列指導方針︰</p><ul><li>將 URL 長度限制在不超過 16,384 個字元 (ASCII 或 Unicode)。 這是以預設 Internet Information Services (IIS) 6 設定為基礎的 URL 長度絕對上限。 網站長度應盡可能不超過此上限</li><li>使用標準的 .NET Framework 檔案 I/O 類別 (例如 FileStream)，因為這些類別會利用 .NET FX 中的標準化規則</li><li>明確建置已知檔案名稱的允許清單</li><li>明確拒絕不會提供 UrlScan 拒絕的已知檔案類型︰exe、bat、cmd、com、htw、ida、idq、htr、idc、shtm[l]、stm、printer、ini、pol、dat 檔案</li><li>捕捉下列例外狀況：<ul><li>System.ArgumentException (針對裝置名稱)</li><li>System.NotSupportedException (針對資料串流)</li><li>System.IO.FileNotFoundException (針對無效的逸出檔名)</li><li>System.IO.DirectoryNotFoundException (針對無效的逸出目錄)</li></ul></li><li>「請勿」對外呼叫 Win32 檔案 I/O API。 針對無效的 URL，正常傳回 400 錯誤給使用者，並記錄實際錯誤。</li></ul>|

## <a id="controls-users"></a>確定在接受使用者的檔案時已備妥適當的控制

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [不受限制的檔案上傳](https://www.owasp.org/index.php/Unrestricted_File_Upload)、[檔案簽章資料表](http://www.garykessler.net/library/file_sigs.html) |
| **步驟** | <p>對應用程式來說，上傳的檔案就代表著重大風險。</p><p>許多攻擊的第一步就是將一些程式碼放到所要攻擊的系統。 然後，攻擊行動只需要找到方法來執行程式碼即可。 使用檔案上傳可幫助攻擊者完成第一步。 檔案上傳不受限制的後果各不相同，可能是系統遭到完整接管、檔案系統或資料庫超載、將攻擊行動轉送到後端系統，或者就只是損毀這麼簡單。</p><p>一切取決於應用程式會怎麼處理上傳的檔案，尤其是會儲存在哪裡。 伺服器端缺少驗證上傳檔案的動作。 請針對檔案上傳功能實作下列安全性控制︰</p><ul><li>副檔名檢查 (只應接受一組有效的允許檔案類型)</li><li>檔案大小上限</li><li>檔案不應上傳至 Web 根目錄；上傳位置應該是非系統磁碟機上的目錄</li><li>應遵循命名慣例，讓上傳的檔案以隨機方式命名，以避免覆寫檔案</li><li>應先對檔案進行防毒掃描再將其寫入磁碟</li><li>確保會驗證檔案名稱和其他任何中繼資料 (例如，檔案路徑) 中是否有惡意字元</li><li>應檢查檔案格式簽章，以防止使用者上傳偽裝的檔案 (例如，將副檔名改為 txt 以上傳 exe 檔案)</li></ul>| 

### <a name="example"></a>範例
如需上述最後一點關於檔案格式簽章驗證的資訊，請參閱下面的類別以了解詳情︰ 

```csharp
        private static Dictionary<string, List<byte[]>> fileSignature = new Dictionary<string, List<byte[]>>
                    {
                    { ".DOC", new List<byte[]> { new byte[] { 0xD0, 0xCF, 0x11, 0xE0, 0xA1, 0xB1, 0x1A, 0xE1 } } },
                    { ".DOCX", new List<byte[]> { new byte[] { 0x50, 0x4B, 0x03, 0x04 } } },
                    { ".PDF", new List<byte[]> { new byte[] { 0x25, 0x50, 0x44, 0x46 } } },
                    { ".ZIP", new List<byte[]> 
                                            {
                                              new byte[] { 0x50, 0x4B, 0x03, 0x04 },
                                              new byte[] { 0x50, 0x4B, 0x4C, 0x49, 0x54, 0x55 },
                                              new byte[] { 0x50, 0x4B, 0x53, 0x70, 0x58 },
                                              new byte[] { 0x50, 0x4B, 0x05, 0x06 },
                                              new byte[] { 0x50, 0x4B, 0x07, 0x08 },
                                              new byte[] { 0x57, 0x69, 0x6E, 0x5A, 0x69, 0x70 }
                                                }
                                            },
                    { ".PNG", new List<byte[]> { new byte[] { 0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A } } },
                    { ".JPG", new List<byte[]>
                                    {
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE1 },
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE8 }
                                    }
                                    },
                    { ".JPEG", new List<byte[]>
                                        { 
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 }
                                        }
                                        },
                    { ".XLS", new List<byte[]>
                                            {
                                              new byte[] { 0xD0, 0xCF, 0x11, 0xE0, 0xA1, 0xB1, 0x1A, 0xE1 },
                                              new byte[] { 0x09, 0x08, 0x10, 0x00, 0x00, 0x06, 0x05, 0x00 },
                                              new byte[] { 0xFD, 0xFF, 0xFF, 0xFF }
                                            }
                                            },
                    { ".XLSX", new List<byte[]> { new byte[] { 0x50, 0x4B, 0x03, 0x04 } } },
                    { ".GIF", new List<byte[]> { new byte[] { 0x47, 0x49, 0x46, 0x38 } } }
                };

        public static bool IsValidFileExtension(string fileName, byte[] fileData, byte[] allowedChars)
        {
            if (string.IsNullOrEmpty(fileName) || fileData == null || fileData.Length == 0)
            {
                return false;
            }

            bool flag = false;
            string ext = Path.GetExtension(fileName);
            if (string.IsNullOrEmpty(ext))
            {
                return false;
            }

            ext = ext.ToUpperInvariant();

            if (ext.Equals(".TXT") || ext.Equals(".CSV") || ext.Equals(".PRN"))
            {
                foreach (byte b in fileData)
                {
                    if (b > 0x7F)
                    {
                        if (allowedChars != null)
                        {
                            if (!allowedChars.Contains(b))
                            {
                                return false;
                            }
                        }
                        else
                        {
                            return false;
                        }
                    }
                }

                return true;
            }

            if (!fileSignature.ContainsKey(ext))
            {
                return true;
            }

            List<byte[]> sig = fileSignature[ext];
            foreach (byte[] b in sig)
            {
                var curFileSig = new byte[b.Length];
                Array.Copy(fileData, curFileSig, b.Length);
                if (curFileSig.SequenceEqual(b))
                {
                    flag = true;
                    break;
                }
            }

            return flag;
        }
```

## <a id="typesafe"></a>確定 Web 應用程式有使用 type-safe 參數來存取資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>如果您使用參數集合，SQL 會將輸入視為常值而非可執行程式碼。 參數集合可用來對輸入資料強制執行類型和長度條件約束。 超出範圍的值會觸發例外狀況。 如果未使用 type-safe SQL 參數，攻擊者或許就能執行內嵌於未經篩選之輸入中的插入式攻擊。</p><p>請在建構 SQL 查詢時使用 type-safe 參數，以避免未經篩選的輸入中可能發生的 SQL 插入式攻擊。 type-safe 參數可與預存程序和動態 SQL 陳述式搭配使用。 資料庫會將參數視為常值而非可執行程式碼。 參數的類型和長度也會受到檢查。</p>|

### <a name="example"></a>範例 
下列程式碼示範如何在呼叫預存程序時，搭配使用 type-safe 參數與 SqlParameterCollection。 

```csharp
using System.Data;
using System.Data.SqlClient;

using (SqlConnection connection = new SqlConnection(connectionString))
{ 
DataSet userDataset = new DataSet(); 
SqlDataAdapter myCommand = new SqlDataAdapter("LoginStoredProcedure", connection); 
myCommand.SelectCommand.CommandType = CommandType.StoredProcedure; 
myCommand.SelectCommand.Parameters.Add("@au_id", SqlDbType.VarChar, 11); 
myCommand.SelectCommand.Parameters["@au_id"].Value = SSN.Text; 
myCommand.Fill(userDataset);
}  
```
在上述程式碼範例中，輸入值不能超過 11 個字元。 如果資料不符合參數所定義的類型或長度，SqlParameter 類別就會擲回例外狀況。 

## <a id="binding-mvc"></a>使用個別的模型繫結類別或繫結篩選清單來防止 MVC 大量指派弱點

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | [中繼資料屬性](http://msdn.microsoft.com/library/system.componentmodel.dataannotations.metadatatypeattribute)、[公用金鑰安全性弱點和風險降低](https://github.com/blog/1068-public-key-security-vulnerability-and-mitigation)、[在 ASP.NET MVC 中大量指派的完整指南](http://odetocode.com/Blogs/scott/archive/2012/03/11/complete-guide-to-mass-assignment-in-asp-net-mvc.aspx)、[使用 MVC 開始使用 EF](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/implementing-basic-crud-functionality-with-the-entity-framework-in-asp-net-mvc-application#overpost) |
| **步驟** | <ul><li>**我應該在何時尋找 over-posting 弱點？-** over-posting 弱點可能發生在您從使用者輸入繫結模型類別的任何位置。 MVC 等架構可在自訂 .NET 類別中代表使用者資料，包括簡單的 CLR 物件 (POCO)。 MVC 會自動在這些模型類別中填入要求中的資料，以方便代表要處理的使用者輸入。 當這些類別包括不應該由使用者設定的屬性時，應用程式很容易遭受 over-posting 攻擊，因而允許使用者控制應用程式永遠不會需要的資料。 和 MVC 模型繫結一樣，Entity Framework 等物件/關聯式對應程式之類的資料庫存取技術，通常也支援使用 POCO 物件來代表資料庫資料。 這些資料模型類別同樣能代表要處理的資料庫資料，就和 MVC 在處理使用者輸入時一樣。 因為 MVC 和資料庫都支援類似模型，例如 POCO 物件，想要將相同的類別重複用於這兩種用途似乎是輕鬆無比的事。 這種作法無法保持關注點分離，而這一塊區域通常也會讓模型繫結能夠接觸到非預期屬性，而實現 over-posting 攻擊。</li><li>**為何不該使用未經篩選的資料庫模型類別來做為 MVC 動作的參數？-** 因為 MVC 模型繫結會在該類別繫結任何項目。 即使資料未出現在檢視中，惡意使用者也可以傳送內含此資料的 HTTP 要求，MVC 會很樂意繫結它，因為您的動作說明資料庫類別是它應該接受做為使用者輸入的資料外觀。</li><li>**為何我該關心用於模型繫結的外觀？-** 搭配過度廣泛的模型使用 ASP.NET MVC 模型繫結，會讓應用程式暴露在 over-posting 攻擊範圍內。 over-posting 可讓攻擊者將應用程式資料的使用範圍變更為開發人員預定用途之外，例如覆寫某商品的價格或帳戶的安全性權限。 應用程式應使用動作專屬的繫結模型 (或特別允許的屬性篩選清單) 來提供明確合約，規定要透過模型繫結允許哪些不受信任的輸入。</li><li>**要擁有個別的繫結模型是否只要複製程式碼即可？-** 否，此作業和關注點分離有關。 若您在動作方法中重複使用資料庫模型，就表示該類別中的屬性 (或子屬性) 可供使用者在 HTTP 要求中設定。 如果您不想讓 MVC 這麼做，就需要使用篩選清單或個別的類別外觀來告訴 MVC 哪些資料可以來自使用者輸入。</li><li>**如果我有專門用於使用者輸入的繫結模型，我是否必須複製我所有的資料註解屬性？-** 不一定。 您可以在資料庫模型類別上使用 MetadataTypeAttribute 來連結至模型繫結類別上的中繼資料。 但請注意，MetadataTypeAttribute 所參考的類型必須是參考類型的子集 (可以有較少屬性，但不得超過)。</li><li>**在使用者輸入模型和資料庫模型之間來回移動資料很麻煩。是否可以使用反映功能直接複製所有屬性？-** 是。 繫結模型中唯一會出現的屬性，是您已確定可安全做為使用者輸入的屬性。 沒有任何安全性方面的理由可阻止您使用反映功能來複製這兩個模型之間共同存在的所有屬性。</li><li>**若是使用 [Bind(Exclude ="â€¦")]？我能否使用此方法而不使用個別的繫結模型？-** 不建議使用這個方法。 使用 [Bind(Exclude ="â€¦")] 表示任何新的屬性都會預設為可繫結。 當您新增屬性時，有一個必須記得才能讓一切保持安全的額外步驟，而非讓設計預設就保持安全。 依靠開發人員在每次新增屬性時檢查這份清單是件危險的事。</li><li>**[Bind(Include ="â€¦")] 對於 [編輯] 作業是否有用？-**  否。 [Bind(Include ="â€¦")] 僅適用於 INSERT 形式的作業 (例如新增資料)。 若為更新樣式的作業 (修改現有資料)，請使用另一種方法，例如擁有個別的繫結模型，或將允許屬性的明確清單傳遞給 UpdateModel 或 TryUpdateModel。 在 [編輯] 作業上新增 [Bind(Include ="â€¦")] 屬性表示 MVC 將會建立物件執行個體，並只設定所列出的屬性，將所有其他的屬性保留為預設值。 當資料存留下來時，便會完全取代現有實體，將任何遭忽略屬性的值重設為預設值。 例如，如果在 [編輯] 作業的 [Bind(Include ="â€¦")] 屬性中省略了 IsAdmin，則透過此動作編輯名稱的任何使用者都會重設為 IsAdmin = false (已編輯的任何使用者都會失去管理員狀態)。 如果您想要防止更新某些屬性，請使用上述其他方法的其中一個。 請注意，某些版本的 MVC 工具會產生控制器類別，且 [編輯] 動作上會包含 [Bind(Include ="â€¦")]，代表從該清單中移除屬性會避免過度貼文 (over-posting) 攻擊。 但如上所述，該方法不會如預期運作，反而會將所忽略屬性中的任何資料重設為預設值。</li><li>**對於 [建立] 作業，使用 [Bind(Include ="â€¦")] 而不是個別的繫結模型時，是否有任何需要注意的事項？-** 是。 首先，這個方法不適用於編輯案例，需要維護兩種不同方法來降低所有 over-posting 弱點。 第二，使用個別的繫結模型必定需要分別處理使用者輸入的圖形與用於持續性的圖形，但使用 [Bind(Include ="â€¦")] 則不必如此。 第三，請注意，[Bind(Include ="â€¦")] 只能處理最上層屬性；您無法僅允許屬性中的部分子屬性 (例如 "Details.Name")。 最後，也許是最重要的一點，使用 [Bind(Include ="â€¦")] 會新增額外的步驟，只要將類別用於模型繫結，就必須記得進行此步驟。 若您將新的動作方法直接繫結至資料類別，且忘記包含 [Bind(Include ="â€¦")] 屬性，就可能容易遭受過度貼文的攻擊，因此 [Bind(Include ="â€¦")] 方法依預設是較不安全的。 如果您使用 [Bind(Include ="â€¦")]，每次您的資料類別顯示為動作方法參數時，請務必一律記得加以指定。</li><li>**對於 [建立] 作業，將 [Bind(Include ="â€¦")] 屬性放在模型類別本身上會如何？這種方法不能讓我不再需要記得在每個動作方法放置此屬性嗎？-** 這種方法在某些情況下有用。 在模型類型本身上使用 [Bind(Include ="â€¦")] (而非在動作參數上使用這個類別)，並不需要在每個動作方法上記得包含 [Bind(Include ="â€¦")] 屬性。 直接在類別上使用此屬性實際上會為此類別建立不同的介面區域，以供繫結模型。 不過，此方法只允許在每個模型類別上使用一個模型繫結外觀。 如果某個動作方法需要允許某欄位進行模型繫結 (例如，會更新使用者角色的僅限系統管理員動作)，而其他動作需要防止此欄位進行模型繫結，這個方法就無法運作。 每一個類別只能有一個模型繫結圖形；如果不同的動作需要不同的模型繫結圖形，則必須在動作方法上使用個別的模型繫結類別或是個別的 [Bind(Include ="â€¦")] 屬性，來代表這些個別的圖形。</li><li>**何謂繫結模型？它們是和檢視模型相同的概念嗎？-** 這兩者是有相關性的概念。 繫結模型一詞是指動作的參數清單中所使用的模型類別 (從 MVC 模型繫結傳遞至動作方法的外觀)。 檢視模型一詞則是指從動作方法傳遞至檢視的模型類別。 要將資料從動作方法傳遞至檢視時，通常會使用檢視專屬模型這個方法。 此外觀通常也適用於模型繫結，而且檢視模型一詞可用來指稱在兩個位置所使用的相同模型。 準確地說，此程序專門討論繫結模型，將重點放在傳遞給動作的外觀，而這對於大量指派來說很重要。</li></ul>| 

## <a id="rendering"></a>先將未受信任的 Web 輸出編碼再進行轉譯

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、Web Form、MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | [如何在 ASP.NET 中防止跨網站指令碼](http://msdn.microsoft.com/library/ms998274.aspx)、[跨網站指令碼](http://cwe.mitre.org/data/definitions/79.html)、[XSS (跨網站指令碼) 防護功能提要](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet) |
| **步驟** | 跨網站指令碼 (通常縮寫為 XSS) 是針對線上服務或從 Web 取用輸入之任何應用程式/元件的攻擊媒介。 XSS 弱點可能會讓攻擊者透過易受攻擊的 Web 應用程式，對其他使用者的電腦執行指令碼。 惡意指令碼可用來竊取 Cookie 或是透過 JavaScript 竄改受害者的電腦。 若要預防 XSS，您可以驗證使用者輸入、確保其格式正確無誤並先加以編碼，再轉譯於網頁中。 使用 Web Protection Library 即可進行輸入驗證和輸出編碼。 若為受控碼 (C\#、VB.net 等)，請根據為使用者輸入建立資訊清單的內容，使用一個或多個來自 Web Protection (Anti-XSS) Library 的適當編碼方法︰| 

### <a name="example"></a>範例

```csharp
* Encoder.HtmlEncode 
* Encoder.HtmlAttributeEncode 
* Encoder.JavaScriptEncode 
* Encoder.UrlEncode
* Encoder.VisualBasicScriptEncode 
* Encoder.XmlEncode 
* Encoder.XmlAttributeEncode 
* Encoder.CssEncode 
* Encoder.LdapEncode 
```

## <a id="typemodel"></a>對所有字串類型的模型屬性執行輸入驗證和篩選

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | [新增驗證](http://www.asp.net/mvc/overview/getting-started/introduction/adding-validation)、[驗證 MVC 應用程式中的模型資料](http://msdn.microsoft.com/library/dd410404(v=vs.90).aspx)、[ASP.NET MVC 應用程式的指導原則](http://msdn.microsoft.com/magazine/dd942822.aspx) |
| **步驟** | <p>所有輸入參數都必須先驗證再用於應用程式，以確保應用程式不受惡意使用者輸入所危害。 請採取白名單驗證策略，在伺服器端上使用規則運算式驗證來驗證輸入值。 傳遞至方法的未清理使用者輸入/參數會造成程式碼插入弱點。</p><p>對於 Web 應用程式，進入點還可能包括表單欄位、QueryStrings、Cookie、HTTP 標頭和 Web 服務參數。</p><p>在繫結模型時，必須執行下列輸入驗證檢查︰</p><ul><li>模型屬性應該使用 RegularExpression 註解來加以註解，以便接受允許的字元和最大允許長度</li><li>控制器方法應該執行 ModelState 有效性</li></ul>|

## <a id="richtext"></a>應該對接受所有字元的表單欄位 (例如 RTF 編輯器) 套用清理

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [將不安全的輸入編碼](https://msdn.microsoft.com/library/ff647397.aspx#paght000003_step3)， [HTML 清理程式](https://github.com/mganss/HtmlSanitizer) |
| **步驟** | <p>找出您想要使用的所有靜態標記標籤。 常見的作法是將格式限制為安全的 HTML 元素，例如 `<b>`(粗體) 和 `<i>` (斜體)。</p><p>在寫入資料前，先對它進行 HTML 編碼。 此動作可讓惡意指令碼變得安全，因為這會讓它以文字形式進行處理，而不是以可執行程式碼的形式。</p><ol><li>將 ValidateRequest ="false" 屬性新增至 \@ Page 指示詞，以停用 ASP.NET 要求驗證</li><li>使用 HtmlEncode 方法將字串輸入編碼</li><li>使用 StringBuilder 並呼叫其取代方法，選擇性移除您想要允許之 HTML 元素上的編碼</li></ol><p>參考中的頁面會藉由設定 `ValidateRequest="false"` 來停用 ASP.NET 要求驗證。 它會對輸入進行 HTML 編碼，並選擇性地允許 `<b>` 和 `<i>`。或者，也可以使用 .NET 程式庫來清理 HTML。</p><p>HtmlSanitizer 是用來從可能會導致 XSS 攻擊的建構中清理 HTML 片段和文件的 .NET 程式庫。 它會使用 AngleSharp 來剖析、操作及轉譯 HTML 與 CSS。 HtmlSanitizer 可使用 NuGet 套件的形式來安裝，並可視需要透過相關 HTML 或 CSS 清理方法，在伺服器端上傳遞使用者輸入。 請注意，只有在迫不得已時才可考慮使用清理做為安全性控制方法。</p><p>輸入驗證和輸出編碼會是較佳的安全性控制方法。</p> |

## <a id="inbuilt-encode"></a>請勿將沒有內建編碼的 DOM 元素指派給接收器

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 許多 javascript 函式預設不會進行編碼。 透過這類函式將不受信任的輸入指派至 DOM 元素時，可能會導致執行跨網站指令碼 (XSS)。| 

### <a name="example"></a>範例
以下是不安全的範例︰ 

```
document.getElementByID("div1").innerHtml = value;
$("#userName").html(res.Name);
return $('<div/>').html(value)
$('body').append(resHTML);   
```
請勿使用 `innerHtml`；而應使用 `innerText`。 同樣地，不要使用 `$("#elm").html()`，而應使用 `$("#elm").text()` 

## <a id="redirect-safe"></a>驗證應用程式內的所有重新導向皆已關閉或安全完成

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [OAuth 2.0 授權架構 - 開啟重新導向程式](http://tools.ietf.org/html/rfc6749#section-10.15) |
| **步驟** | <p>要求重新導向至使用者所提供位置的應用程式設計，必須將可能的重新導向目標限制在預先定義的「安全」網站或網域清單。 應用程式中的所有重新導向都必須是封閉/安全的。</p><p>作法：</p><ul><li>找出所有重新導向</li><li>為每個重新導向實作適當的風險降低措施。 適當的風險降低措施包括重新導向允許清單或使用者確認。 如果具有開啟重新導向弱點的網站或服務使用 Facebook/OAuth/OpenID 身分識別提供者，攻擊者便可竊取使用者的登入權杖，然後假扮該使用者。 這是使用 OAuth 時的固有風險，在 RFC 6749「OAuth 2.0 授權架構」的 10.15 節「開啟重新導向」有其記載。同樣地，魚叉式網路釣魚攻擊也會使用開啟重新導向來入侵取得使用者的認證</li></ul>|

## <a id="string-method"></a>對控制器方法所接受的所有字串類型參數實作輸入驗證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | [驗證 MVC 應用程式中的模型資料](http://msdn.microsoft.com/library/dd410404(v=vs.90).aspx)、[ASP.NET MVC 應用程式的指導原則](http://msdn.microsoft.com/magazine/dd942822.aspx) |
| **步驟** | 對於只接受基本資料類型而不接受模型來做為引數的方法，應該使用規則運算式進行輸入驗證。 在此，Regex.IsMatch 應搭配使用有效的 regex 模式。 如果輸入不符合指定的規則運算式，就不該再繼續進行控制，而且應該顯示關於驗證失敗的適當警告。| 

## <a id="dos-expression"></a>為規則運算式處理設定逾時上限以防止因規則運算式不正確而導致 DoS

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、Web Form、MVC5、MVC6  |
| **屬性**              | N/A  |
| **參考**              | [DefaultRegexMatchTimeout 屬性](https://msdn.microsoft.com/library/system.web.configuration.httpruntimesection.defaultregexmatchtimeout.aspx) |
| **步驟** | 若要確保以錯誤格式建立的規則運算式不會遭受阻斷服務攻擊，而造成大量回溯，請設定全域的預設逾時。 如果處理時間超過所定義的上限，便會擲回逾時例外狀況。 若未做任何設定，則逾時時間是無限。| 

### <a name="example"></a>範例
例如，下列組態會在處理時間超過 5 秒時擲回 RegexMatchTimeoutException︰ 

```csharp
<httpRuntime targetFramework="4.5" defaultRegexMatchTimeout="00:00:05" />
```

## <a id="html-razor"></a>避免在 Razor 檢視中使用 Html.Raw

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| 步驟 | ASP.Net 網頁 (Razor) 會執行自動 HTML 編碼。 內嵌程式碼區塊 (@ blocks) 所列印的所有字串會自動進行 HTML 編碼。 不過，在叫用 `HtmlHelper.Raw` 方法時，它會傳回非 HTML 編碼的標記。 如果使用 `Html.Raw()` 協助程式方法，它會略過 Razor 所提供的自動編碼保護。|

### <a name="example"></a>範例
以下是不安全的範例︰ 

```csharp
<div class="form-group">
            @Html.Raw(Model.AccountConfirmText)
        </div>
        <div class="form-group">
            @Html.Raw(Model.PaymentConfirmText)
        </div>
</div>
```
除非您需要顯示標記，否則請勿使用 `Html.Raw()`。 這個方法不會隱含執行輸出編碼。 請使用其他 ASP.NET 協助程式，例如 `@Html.DisplayFor()` 

## <a id="stored-proc"></a>請勿在預存程序中使用動態查詢

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>SQL 插入式攻擊利用輸入驗證弱點在資料庫中執行任意命令。 當您的應用程式使用輸入來建構動態 SQL 陳述式以存取資料庫時，便可能發生此弱點。 如果您的程式碼使用預存程序來傳遞包含原始使用者輸入的字串，也可能發生此弱點。 攻擊者使用 SQL 插入式攻擊，即可在資料庫中執行任意命令。 所有 SQL 陳述式 (包括預存程序的 SQL 陳述式) 都必須予以參數化。 參數化的 SQL 陳述式會接受對 SQL 具有特殊意義的字元 (例如單引號) 而不會有任何問題，因為它們屬於強型別。 |

### <a name="example"></a>範例
以下是不安全的動態預存程序範例︰ 

```csharp
CREATE PROCEDURE [dbo].[uspGetProductsByCriteria]
(
  @productName nvarchar(200) = NULL,
  @startPrice float = NULL,
  @endPrice float = NULL
)
AS
 BEGIN
  DECLARE @sql nvarchar(max)
  SELECT @sql = ' SELECT ProductID, ProductName, Description, UnitPrice, ImagePath' +
       ' FROM dbo.Products WHERE 1 = 1 '
       PRINT @sql
  IF @productName IS NOT NULL
     SELECT @sql = @sql + ' AND ProductName LIKE ''%' + @productName + '%'''
  IF @startPrice IS NOT NULL
     SELECT @sql = @sql + ' AND UnitPrice > ''' + CONVERT(VARCHAR(10),@startPrice) + ''''
  IF @endPrice IS NOT NULL
     SELECT @sql = @sql + ' AND UnitPrice < ''' + CONVERT(VARCHAR(10),@endPrice) + ''''

  PRINT @sql
  EXEC(@sql)
 END
```

### <a name="example"></a>範例
以下是以安全方式實作的相同預存程序︰ 
```csharp
CREATE PROCEDURE [dbo].[uspGetProductsByCriteriaSecure]
(
             @productName nvarchar(200) = NULL,
             @startPrice float = NULL,
             @endPrice float = NULL
)
AS
       BEGIN
             SELECT ProductID, ProductName, Description, UnitPrice, ImagePath
             FROM dbo.Products where
             (@productName IS NULL or ProductName like '%'+ @productName +'%')
             AND
             (@startPrice IS NULL or UnitPrice > @startPrice)
             AND
             (@endPrice IS NULL or UnitPrice < @endPrice)         
       END
```

## <a id="validation-api"></a>確定 Web API 方法已完成模型驗證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | [ASP.NET Web API 中的模型驗證](http://www.asp.net/web-api/overview/formats-and-model-binding/model-validation-in-aspnet-web-api) |
| **步驟** | 當用戶端傳送資料到 Web API 時，必須先驗證資料再進行任何處理。 對於可接受模型做為輸入的 ASP.NET Web API，請在模型上使用資料註解，以對模型屬性設定驗證規則。|

### <a name="example"></a>範例
下列程式碼示範相同案例︰ 

```csharp
using System.ComponentModel.DataAnnotations;

namespace MyApi.Models
{
    public class Product
    {
        public int Id { get; set; }
        [Required]
        [RegularExpression(@"^[a-zA-Z0-9]*$", ErrorMessage="Only alphanumeric characters are allowed.")]
        public string Name { get; set; }
        public decimal Price { get; set; }
        [Range(0, 999)]
        public double Weight { get; set; }
    }
}
```

### <a name="example"></a>範例
在 API 控制器的動作方法中，模型的有效性必須明確檢查，如下所示： 

```csharp
namespace MyApi.Controllers
{
    public class ProductsController : ApiController
    {
        public HttpResponseMessage Post(Product product)
        {
            if (ModelState.IsValid)
            {
                // Do something with the product (not shown).

                return new HttpResponseMessage(HttpStatusCode.OK);
            }
            else
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
            }
        }
    }
}
```

## <a id="string-api"></a>對 Web API 方法所接受的所有字串類型參數實作輸入驗證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、MVC 5、MVC 6 |
| **屬性**              | N/A  |
| **參考**              | [驗證 MVC 應用程式中的模型資料](http://msdn.microsoft.com/library/dd410404(v=vs.90).aspx)、[ASP.NET MVC 應用程式的指導原則](http://msdn.microsoft.com/magazine/dd942822.aspx) |
| **步驟** | 對於只接受基本資料類型而不接受模型來做為引數的方法，應該使用規則運算式進行輸入驗證。 在此，Regex.IsMatch 應搭配使用有效的 regex 模式。 如果輸入不符合指定的規則運算式，就不該再繼續進行控制，而且應該顯示關於驗證失敗的適當警告。|

## <a id="typesafe-api"></a>確定 Web API 有使用 type-safe 參數來存取資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>如果您使用參數集合，SQL 會將輸入視為常值而非可執行程式碼。 參數集合可用來對輸入資料強制執行類型和長度條件約束。 超出範圍的值會觸發例外狀況。 如果未使用 type-safe SQL 參數，攻擊者或許就能執行內嵌於未經篩選之輸入中的插入式攻擊。</p><p>請在建構 SQL 查詢時使用 type-safe 參數，以避免未經篩選的輸入中可能發生的 SQL 插入式攻擊。 type-safe 參數可與預存程序和動態 SQL 陳述式搭配使用。 資料庫會將參數視為常值而非可執行程式碼。 參數的類型和長度也會受到檢查。</p>|

### <a name="example"></a>範例
下列程式碼示範如何在呼叫預存程序時，搭配使用 type-safe 參數與 SqlParameterCollection。 

```csharp
using System.Data;
using System.Data.SqlClient;

using (SqlConnection connection = new SqlConnection(connectionString))
{ 
DataSet userDataset = new DataSet(); 
SqlDataAdapter myCommand = new SqlDataAdapter("LoginStoredProcedure", connection); 
myCommand.SelectCommand.CommandType = CommandType.StoredProcedure; 
myCommand.SelectCommand.Parameters.Add("@au_id", SqlDbType.VarChar, 11); 
myCommand.SelectCommand.Parameters["@au_id"].Value = SSN.Text; 
myCommand.Fill(userDataset);
}  
```
在上述程式碼範例中，輸入值不能超過 11 個字元。 如果資料不符合參數所定義的類型或長度，SqlParameter 類別就會擲回例外狀況。 

## <a id="sql-docdb"></a>針對 Cosmos DB 使用參數化 SQL 查詢

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure Document DB | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [在 Azure Cosmos DB 中宣佈 SQL 參數化](https://azure.microsoft.com/blog/announcing-sql-parameterization-in-documentdb/) |
| **步驟** | 雖然 Azure Cosmos DB 只支援唯讀查詢，但如果查詢是藉由串連使用者輸入來建構，仍可發動 SQL 插入式攻擊。 使用者只要藉由編寫惡意 SQL 查詢，即可在相同集合內存取他們不該能夠存取的資料。 如果查詢是根據使用者輸入所建構而成，請使用參數化的 SQL 查詢。 |

## <a id="schema-binding"></a>透過結構描述繫結驗證 WCF 輸入

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、NET Framework 3 |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff647820.aspx) |
| **步驟** | <p>缺少驗證機制會導致不同類型的插入式攻擊。</p><p>訊息驗證代表 WCF 應用程式保護措施中的一條防禦線。 利用這個方法，您可以使用結構描述來驗證訊息，以防止惡意用戶端攻擊 WCF 服務作業。 驗證用戶端所收到的所有訊息可防止用戶端遭到惡意服務攻擊。 訊息驗證可讓您在作業取用訊息合約或資料合約時驗證訊息，使用參數驗證則無法這麼做。 訊息驗證可讓您在結構描述內建立驗證邏輯，進而提供更大的彈性並縮減開發時間。 結構描述可以重複使用到組織中的不同應用程式，而為如何代表資料建立標準。 此外，在它們取用涉及代表商務邏輯之合約的更複雜資料類型時，訊息驗證可讓您保護作業。</p><p>若要執行訊息驗證，您可以先建置結構描述來代表服務的作業和這些作業所取用的資料類型。 然後，您可以建立 .NET 類別來實作自訂用戶端訊息偵測器和自訂發送器訊息偵測器，以驗證服務所傳送/接收的訊息。 接下來，您可以實作自訂端點行為，在用戶端和服務上啟用訊息驗證。 最後，您可以對類別實作自訂組態元素，以便您可以在服務或用戶端的組態檔中，公開擴充的自訂端點行為</p>|

## <a id="parameters"></a>透過參數偵測器驗證 WCF 輸入

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、NET Framework 3 |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff647875.aspx) |
| **步驟** | <p>輸入和資料驗證代表 WCF 應用程式保護措施中的一條重要防禦線。 您應該驗證 WCF 服務作業中公開的所有參數，以防止服務遭受惡意用戶端攻擊。 相反地，您還應該驗證用戶端所收到的所有傳回值，以防止用戶端遭受惡意服務攻擊</p><p>WCF 會提供不同的擴充點，讓您藉由建立自訂擴充功能來自訂 WCF 執行階段行為。 訊息偵測器和參數偵測器這兩個擴充性機制可用來加強對用戶端和服務之間所傳遞之資料的控制力。 您應該使用參數偵測器來驗證輸入，而只將訊息偵測器用於當您需要檢查流入及流出服務的整個訊息時。</p><p>若要執行輸入驗證，您需要建置 .NET 類別，並實作自訂參數偵測器，以對服務中的作業參數進行驗證。 接著，您需要實作自訂端點行為，在用戶端和服務上啟用驗證。 最後，您可以對類別實作自訂組態元素，以便您可以在服務或用戶端的組態檔中，公開擴充的自訂端點行為</p>|
