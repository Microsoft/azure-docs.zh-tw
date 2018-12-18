---
title: 呼叫包含驗證的 Azure 儲存體服務 REST API 作業 | Microsoft Docs
description: 呼叫包含驗證的 Azure 儲存體服務 REST API 作業
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 05/22/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 78e2620ba6e5e29a1f1ac9719b709d5a2f468122
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39529037"
---
# <a name="using-the-azure-storage-rest-api"></a>使用 Azure 儲存體 REST API

本文說明如何使用 Blob 儲存體服務 REST API 以及如何驗證對服務的呼叫。 它是從對 REST 一無所知且不知道如何進行 REST 呼叫之開發人員的觀點來撰寫。 我們會查看 REST 呼叫的參考文件，了解如何將它轉譯為實際的 REST 呼叫 – 哪些欄位放到哪裡？ 了解如何設定 REST 呼叫之後，您可以運用此知識來使用任何其他儲存體服務 REST API。

## <a name="prerequisites"></a>必要條件 

應用程式會列出儲存體帳戶的 blob 儲存體中的容器。 若要試用本文中的程式碼，您需要下列項目︰ 

* 使用下列工作負載安裝 [Visual Studio 2017](https://www.visualstudio.com/visual-studio-homepage-vs.aspx)：
    - Azure 開發

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

* 一般用途的儲存體帳戶。 如果您還沒有儲存體帳戶，請參閱[建立儲存體帳戶](storage-quickstart-create-account.md)。

* 本文中的範例示範如何列出儲存體帳戶中的容器。 若要查看輸出，請在開始前，先將一些容器新增至儲存體帳戶中的 blob 儲存體。

## <a name="download-the-sample-application"></a>下載範例應用程式

範例應用程式是以 C# 撰寫的主控台應用程式。

使用 [git](https://git-scm.com/) 將應用程式的複本下載至您的開發環境。 

```bash
git clone https://github.com/Azure-Samples/storage-dotnet-rest-api-with-auth.git
```

此命令會將存放庫複製到本機的 git 資料夾。 若要開啟 Visual Studio 解決方案，請找到並開啟 storage-dotnet-rest-api-with-auth 資料夾，然後按兩下 StorageRestApiAuth.sln。 

## <a name="what-is-rest"></a>什麼是 REST？

REST 表示*具像狀態傳輸*。 如需特定定義，請查看[維基百科](http://en.wikipedia.org/wiki/Representational_state_transfer)。

基本上，REST 是您可以在呼叫 API 或讓 API 可供呼叫時使用的架構。 它與任一端的情況，以及在傳送或接收 REST 呼叫時使用哪些其他軟體無關。 您可以撰寫在 Mac、Windows、Linux、Android 手機或平板電腦、iPhone、iPod 或網站上執行的應用程式，並針對上述這些平台使用相同的 REST API。 呼叫 REST API 時，可以傳入及/或傳出資料。 REST API 不在意它是從哪個平台呼叫 – 重要的是在要求中傳遞的資訊以及在回應中提供的資料。

了解如何使用 REST 是很有用的技能。 Azure 產品小組經常發行新功能。 很多次，新功能可透過 REST 介面存取，但尚未透過**所有**的儲存體用戶端程式庫或 UI (例如 Azure 入口網站) 呈現。 如果您一直想使用最新且最佳的產品，則必須學習 REST。 此外，如果您想要撰寫自己的程式庫來與 Azure 儲存體互動，或想要利用不具 SDK 或儲存體用戶端程式庫的程式設計語言存取 Azure 儲存體，您可以使用 REST API。

## <a name="about-the-sample-application"></a>關於範例應用程式

範例應用程式會列出儲存體帳戶中的容器。 了解 REST API 文件中的資訊如何與實際的程式碼相互關聯後，就比較容易找出其他 REST 呼叫。 

如果您查看 [Blob 服務 REST API](/rest/api/storageservices/Blob-Service-REST-API)，您會看到可以在 blob 儲存體上執行的所有作業。 儲存體用戶端程式庫以 REST API 為主的包裝函式，可讓您輕鬆地存取儲存體，而不需直接使用 REST API。 但如上所述，有時候您想要使用 REST API，而不是儲存體用戶端程式庫。

## <a name="rest-api-reference-list-containers-api"></a>REST API 參考：列出容器 API

讓我們看看 REST API 參考中的 [ListContainers](/rest/api/storageservices/List-Containers2) 作業頁面，以便在附有程式碼的下一節中瞭解要求和回應中的某些欄位來自何處。

**要求方法**：GET。 此動詞命令是您指定作為要求物件屬性的 HTTP 方法。 視您呼叫的 API 而定，此動詞命令的其他值包括 HEAD、PUT 和 DELETE。

**要求 URI**：https://myaccount.blob.core.windows.net/?comp=list 這是從 blob 儲存體帳戶端點 `http://myaccount.blob.core.windows.net` 和資源字串 `/?comp=list` 建立而來。

[URI 參數](/rest/api/storageservices/List-Containers2#uri-parameters)：呼叫 ListContainers 時，您有其他查詢參數可以使用。 其中有幾個參數是呼叫的 timeout (以秒為單位) 以及用於篩選的 prefix。

另一個有用參數是 maxresults:，而如果可用的容器超出這個值，回應主體會包含 NextMarker 元素，以指出要在下一個要求中傳回的下一個容器。 若要使用此功能，您可在提出下一個要求時，於 URI 中提供 NextMarker 值作為 marker 參數。 使用此功能時，類似於逐頁瀏覽結果。 

若要使用其他參數，請將它們附加至資源字串，其值如下列範例所示：

```
/?comp=list&timeout=60&maxresults=100
```

[要求標頭](/rest/api/storageservices/List-Containers2#request-headers)**：** 此區段列出必要及選用的要求標頭。 三個必要標頭：Authorization 標頭、x-ms-date (包含要求的 UTC 時間) 及 x-ms-version (指定要使用的 REST API 版本)。 選擇性地在標頭中包含 *x-ms-client-request-id* – 您可以將此欄位的值設定為任意值；若已啟用記錄功能，則會寫入至儲存體分析記錄中。

[要求主體](/rest/api/storageservices/List-Containers2#request-body)**：** ListContainers 沒有要求主體。 要求主體使用於上傳 blob 時的所有 PUT 作業，也會使用於 SetContainerAccessPolicy，其可讓您以要套用之預存存取原則的 XML 清單傳送。 [使用共用存取簽章 (SAS)](storage-dotnet-shared-access-signature-part-1.md) 一文會討論預存存取原則。

[回應狀態碼](/rest/api/storageservices/List-Containers2#status-code)**：** 告知您需要知道的任何狀態碼。 在此範例中，HTTP 狀態碼 200 表示「正常」。 如需完整的 HTTP 狀態碼清單，請參閱[狀態碼定義](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。 若要查看儲存體 REST API 特有的錯誤碼，請參閱[常見的 REST API 錯誤碼](/rest/api/storageservices/common-rest-api-error-codes)

[回應標頭](/rest/api/storageservices/List-Containers2#response-headers)**：** 包括「內容類型」、x-ms-request-id (您傳入的要求識別碼 (適用的話))、x-ms-version (表示使用的 Blob 服務版本) 和「日期」 (UTC，告知提出要求的時間)。

[回應主體](/rest/api/storageservices/List-Containers2#response-body)：這個欄位是一種 XML 結構，可提供所要求的資料。 在此範例中，回應是容器及其屬性的清單。

## <a name="creating-the-rest-request"></a>建立 REST 要求

開始前的注意事項 – 基於安全性考量，在生產環境中執行時，一律使用 HTTPS，而不是 HTTP。 基於此練習的目的，您應該使用 HTTP，以便您檢視要求和回應資料。 若要檢視實際 REST 呼叫中的要求和回應資訊，您可以下載 [Fiddler](http://www.telerik.com/fiddler) 或類似的應用程式。 在 Visual Studio 解決方案中，儲存體帳戶名稱和金鑰都已硬式編碼在類別中，而 ListContainersAsyncREST 方法會將儲存體帳戶名稱和儲存體帳戶金鑰傳遞給用來建立 REST 要求之各種元件的方法。 在真實世界的應用程式中，儲存體帳戶名稱和金鑰會位於組態檔、環境變數，或從 Azure Key Vault 擷取而來。

在我們的範例專案中，用於建立授權標頭的程式碼位於個別的類別中，其概念是您無法取得整個類別，將它新增至自己的解決方案並依「現況」使用它。 授權標頭程式碼適用於對 Azure 儲存體的大部分 REST API 呼叫。

若要建立要求 (也就是 HttpRequestMessage 物件)，請移至 Program.cs 中的 ListContainersAsyncREST。 建立要求的步驟如下： 

* 建立要用於呼叫服務的 URI。 
* 建立 HttpRequestMessage 物件並設定承載。 ListContainersAsyncREST 的承載為 null，因為我們並未傳入任何項目。
* 新增 x-ms-date 和 x-ms-version 的要求標頭。
* 取得授權標頭並加以新增。

您需要的一些基本資訊： 

*  對 ListContainers 而言，**方法**是 `GET`。 此值會在具現化要求時設定。 
*  **資源**是 URI 的查詢部份，其可指出正在呼叫哪一個 API，所以此值為 `/?comp=list`。 如前文所述，資源位於參考文件頁面上，可顯示 [ListContainers API](/rest/api/storageservices/List-Containers2) 的相關資訊。
*  建立該儲存體帳戶的 Blob 服務端點並串連資源，即可建構 URI。 **要求 URI** 的值最終是 `http://contosorest.blob.core.windows.net/?comp=list`。
*  對 ListContainers 而言，**requestBody** 為 null，而且沒有額外的**標頭**。

不同的 API 可能有其他要傳入的參數，例如 ifMatch。 舉例來說，在呼叫 PutBlob 時，您可能會使用 ifMatch。 在此情況下，您將 ifMatch 設定為 eTag，而它只會在您提供的 eTag 符合 blob 上目前的 eTag 時更新 blob。 如果有其他人在擷取 eTag 後更新 blob，將不會覆寫其變更。 

首先，設定 `uri` 和 `payload`。 

```csharp
// Construct the URI. This will look like this:
//   https://myaccount.blob.core.windows.net/resource
String uri = string.Format("http://{0}.blob.core.windows.net?comp=list", storageAccountName);

// Set this to whatever payload you desire. Ours is null because 
//   we're not passing anything in.
Byte[] requestPayload = null;
```

接著，將要求具現化，將方法設定為 `GET` 並提供 URI。

```csharp 
//Instantiate the request message with a null payload.
using (var httpRequestMessage = new HttpRequestMessage(HttpMethod.Get, uri)
{ Content = (requestPayload == null) ? null : new ByteArrayContent(requestPayload) })
{
```

新增 x-ms-date 和 x-ms-version 的要求標頭。 程式碼中的這個位置也就是您新增呼叫所需之任何其他要求標頭的位置。 在此範例中，沒有任何額外的標頭。 SetContainerACL 是傳入額外標頭的 API 範例之一。 對於 Blob 儲存體，它會新增名為 "x-ms-blob-public-access" 的標頭以及存取層級的值。

```csharp
    // Add the request headers for x-ms-date and x-ms-version.
    DateTime now = DateTime.UtcNow;
    httpRequestMessage.Headers.Add("x-ms-date", now.ToString("R", CultureInfo.InvariantCulture));
    httpRequestMessage.Headers.Add("x-ms-version", "2017-07-29");
    // If you need any additional headers, add them here before creating
    //   the authorization header. 
```

呼叫可建立授權標頭的方法，並將它新增至要求標頭。 您會理解如何建立本文後面的授權標頭。 此方法名稱是 GetAuthorizationHeader，您可以在此程式碼片段中看到該方法：

```csharp
    // Get the authorization header and add it.
    httpRequestMessage.Headers.Authorization = AzureStorageAuthenticationHelper.GetAuthorizationHeader(
        storageAccountName, storageAccountKey, now, httpRequestMessage);
```

此時，`httpRequestMessage` 包含帶有授權標頭的 REST 要求。 

## <a name="call-the-rest-api-with-the-request"></a>使用要求呼叫 REST API

您現在有此要求，即可呼叫 SendAsync 以傳送 REST 要求。 SendAsync 可呼叫 API 並重新取得回應。 檢查回應 StatusCode (200 表示「正常」)，然後剖析回應。 在此情況下，您會取得容器的 XML 清單。 讓我們查看用於呼叫 GetRESTRequest 方法以建立要求的程式碼，執行此要求，然後檢查容器清單的回應。

```csharp 
    // Send the request.
    using (HttpResponseMessage httpResponseMessage = 
      await new HttpClient().SendAsync(httpRequestMessage, cancellationToken))
    {
        // If successful (status code = 200), 
        //   parse the XML response for the container names.
        if (httpResponseMessage.StatusCode == HttpStatusCode.OK)
        {
            String xmlString = await httpResponseMessage.Content.ReadAsStringAsync();
            XElement x = XElement.Parse(xmlString);
            foreach (XElement container in x.Element("Containers").Elements("Container"))
            {
                Console.WriteLine("Container name = {0}", container.Element("Name").Value);
            }
        }
    }
}
```

如果您在呼叫 SendAsync 時執行網路 sniffer (例如 [Fiddler](https://www.telerik.com/fiddler))，您可以看到要求和回應資訊。 讓我看看。 儲存體帳戶的名稱為 contosorest。

**要求：**

```
GET /?comp=list HTTP/1.1
```

**要求標頭：**

```
x-ms-date: Thu, 16 Nov 2017 23:34:04 GMT
x-ms-version: 2014-02-14
Authorization: SharedKey contosorest:1dVlYJWWJAOSHTCPGiwdX1rOS8B4fenYP/VrU0LfzQk=
Host: contosorest.blob.core.windows.net
Connection: Keep-Alive
```

**執行後傳回的狀態碼和回應標頭：**

```
HTTP/1.1 200 OK
Content-Type: application/xml
Server: Windows-Azure-Blob/1.0 Microsoft-HTTPAPI/2.0
x-ms-request-id: 3e889876-001e-0039-6a3a-5f4396000000
x-ms-version: 2017-07-29
Date: Fri, 17 Nov 2017 00:23:42 GMT
Content-Length: 1511
```

**回應主體 (XML)：** 對於 ListContainers，這會顯示容器和其屬性的清單。

```xml  
<?xml version="1.0" encoding="utf-8"?>
<EnumerationResults 
  ServiceEndpoint="http://contosorest.blob.core.windows.net/">
  <Containers>
    <Container>
      <Name>container-1</Name>
      <Properties>
        <Last-Modified>Thu, 16 Mar 2017 22:39:48 GMT</Last-Modified>
        <Etag>"0x8D46CBD5A7C301D"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
      </Properties>
    </Container>
    <Container>
      <Name>container-2</Name>
      <Properties>
        <Last-Modified>Thu, 16 Mar 2017 22:40:50 GMT</Last-Modified>
        <Etag>"0x8D46CBD7F49E9BD"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
      </Properties>
    </Container>
    <Container>
      <Name>container-3</Name>
      <Properties>
        <Last-Modified>Thu, 16 Mar 2017 22:41:10 GMT</Last-Modified>
        <Etag>"0x8D46CBD8B243D68"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
      </Properties>
    </Container>
    <Container>
      <Name>container-4</Name>
      <Properties>
        <Last-Modified>Thu, 16 Mar 2017 22:41:25 GMT</Last-Modified>
        <Etag>"0x8D46CBD93FED46F"</Etag>
        <LeaseStatus>unlocked</LeaseStatus>
        <LeaseState>available</LeaseState>
        </Properties>
      </Container>
      <Container>
        <Name>container-5</Name>
        <Properties>
          <Last-Modified>Thu, 16 Mar 2017 22:41:39 GMT</Last-Modified>
          <Etag>"0x8D46CBD9C762815"</Etag>
          <LeaseStatus>unlocked</LeaseStatus>
          <LeaseState>available</LeaseState>
        </Properties>
      </Container>
  </Containers>
  <NextMarker />
</EnumerationResults>
```

既然您了解如何建立要求、呼叫服務，以及剖析結果，讓我們看看如何建立授權標頭。 建立該標頭很複雜，但好消息是一旦讓程式碼運作，它就適用於所有的儲存體服務 REST API。

## <a name="creating-the-authorization-header"></a>建立授權標頭

> [!TIP]
> Azure 儲存體現在支援 Blob 和佇列服務的 Azure Active Directory (Azure AD) 整合 (預覽)。 Azure AD 可提供更簡單的 Azure 儲存體要求授權體驗。 如需使用 Azure AD 來授權 REST 作業的詳細資訊，請參閱[使用 Azure Active Directory 進行驗證 (預覽)](https://docs.microsoft.com/rest/api/storageservices/authenticate-with-azure-active-directory)。 如需 Azure AD 與 Azure 儲存體整合的概觀，請參閱[使用 Azure Active Directory 來驗證 Azure 儲存體的存取權 (預覽)](storage-auth-aad.md)。

有一篇在概念上說明 (沒有程式碼) 如何執行 [Azure 儲存體服務驗證](/rest/api/storageservices/Authorization-for-the-Azure-Storage-Services)的文章。
讓我們萃取該文章的精華，並顯示程式碼。

首先，使用共用金鑰驗證。 授權標頭格式如下所示：

```  
Authorization="SharedKey <storage account name>:<signature>"  
```

簽章欄位是雜湊式訊息驗證碼 (HMAC)，該驗證碼是從要求建構而來並使用 SHA256 演算法進行計算，然後使用 Base64 編碼方式進行編碼。 懂了嗎？ (撐著點，您甚至還沒聽過「正式」這個字。)

此程式碼片段會顯示共用金鑰簽章字串的格式：

```csharp  
StringToSign = VERB + "\n" +  
               Content-Encoding + "\n" +  
               Content-Language + "\n" +  
               Content-Length + "\n" +  
               Content-MD5 + "\n" +  
               Content-Type + "\n" +  
               Date + "\n" +  
               If-Modified-Since + "\n" +  
               If-Match + "\n" +  
               If-None-Match + "\n" +  
               If-Unmodified-Since + "\n" +  
               Range + "\n" +  
               CanonicalizedHeaders +  
               CanonicalizedResource;  
```

大部分的上述欄位都很少使用。 對於 Blob 儲存體，您可以指定 VERB、md5、內容長度、正式標頭和正式資源。 您可以將其他欄位空白 (但放入 `\n`，讓它知道是空白的)。

什麼是 CanonicalizedHeaders 和 CanonicalizedResource？ 好問題。 事實上，正式 (canonicalized) 是什麼意思？ Microsoft Word 甚至不會將它視為一個字。 以下是[維基百科對於正式化 (canonicalization) 的說明](http://en.wikipedia.org/wiki/Canonicalization)：*在資訊科學中，正式化 (有時稱為標準化或正規化) 是一種程序，用於將具有一種以上可能表示法的資料轉換為「標準」、「正規」或正式形式。* 一般來說，這表示取得項目 (例如正式標頭中的標頭) 清單，並將它們標準化成為所需的格式。 基本上，Microsoft 已決定格式，而您必須符合它。

讓我們從這兩個正式欄位著手，因為需有它們才能建立授權標頭。

**正式標頭：**

若要建立此值，請擷取以 "x-ms-" 開頭的標頭並加以排序，然後將它們格式化成 `[key:value\n]` 執行個體的字串 (串連成一個字串)。 此範例中，正式標頭如下所示： 

```
x-ms-date:Fri, 17 Nov 2017 00:44:48 GMT\nx-ms-version:2017-07-29\n
```

以下是用來建立該輸出的程式碼：

```csharp 
private static string GetCanonicalizedHeaders(HttpRequestMessage httpRequestMessage)
{
    var headers = from kvp in httpRequestMessage.Headers
                    where kvp.Key.StartsWith("x-ms-", StringComparison.OrdinalIgnoreCase)
                    orderby kvp.Key
                    select new { Key = kvp.Key.ToLowerInvariant(), kvp.Value };

    StringBuilder sb = new StringBuilder();

    // Create the string in the right format; this is what makes the headers "canonicalized" --
    //   it means put in a standard format. http://en.wikipedia.org/wiki/Canonicalization
    foreach (var kvp in headers)
    {
        StringBuilder headerBuilder = new StringBuilder(kvp.Key);
        char separator = ':';

        // Get the value for each header, strip out \r\n if found, then append it with the key.
        foreach (string headerValues in kvp.Value)
        {
            string trimmedValue = headerValues.TrimStart().Replace("\r\n", String.Empty);
            headerBuilder.Append(separator).Append(trimmedValue);

            // Set this to a comma; this will only be used 
            //   if there are multiple values for one of the headers.
            separator = ',';
        }
        sb.Append(headerBuilder.ToString()).Append("\n");
    }
    return sb.ToString();
}      
```

**正式資源**

這部分的簽章字串代表作為要求目標的儲存體帳戶。 請記住，要求 URI 是 `<http://contosorest.blob.core.windows.net/?comp=list>`，具有實際帳戶名稱 (在此情況下是 `contosorest`)。 在此範例中，傳回的內容如下：

```
/contosorest/\ncomp:list
```

如果您有查詢參數，這些參數也包含在內。 以下的程式碼也會處理其他查詢參數，以及具有多個值的查詢參數。 請記住，您將此程式碼建置成適用於所有的 REST API，所以即使 ListContainers 方法不需要所有的可能值，您都想全都包含。

```csharp 
private static string GetCanonicalizedResource(Uri address, string storageAccountName)
{
    // The absolute path will be "/" because for we're getting a list of containers.
    StringBuilder sb = new StringBuilder("/").Append(storageAccountName).Append(address.AbsolutePath);

    // Address.Query is the resource, such as "?comp=list".
    // This ends up with a NameValueCollection with 1 entry having key=comp, value=list.
    // It will have more entries if you have more query parameters.
    NameValueCollection values = HttpUtility.ParseQueryString(address.Query);

    foreach (var item in values.AllKeys.OrderBy(k => k))
    {
        sb.Append('\n').Append(item).Append(':').Append(values[item]);
    }

    return sb.ToString();
}
```

現在已設定正式字串，讓我們看看如何建立授權標頭本身。 首先以本文先前顯示的 StringToSign 格式，建立訊息簽章的字串。 這個概念比較容易說明在程式碼中使用註解，所以下面是傳回授權標頭的最後一個方法：

```csharp
internal static AuthenticationHeaderValue GetAuthorizationHeader(
    string storageAccountName, string storageAccountKey, DateTime now,
    HttpRequestMessage httpRequestMessage, string ifMatch = "", string md5 = "")
{
    // This is the raw representation of the message signature.
    HttpMethod method = httpRequestMessage.Method;
    String MessageSignature = String.Format("{0}\n\n\n{1}\n{5}\n\n\n\n{2}\n\n\n\n{3}{4}",
                method.ToString(),
                (method == HttpMethod.Get || method == HttpMethod.Head) ? String.Empty
                  : httpRequestMessage.Content.Headers.ContentLength.ToString(),
                ifMatch,
                GetCanonicalizedHeaders(httpRequestMessage),
                GetCanonicalizedResource(httpRequestMessage.RequestUri, storageAccountName),
                md5);

    // Now turn it into a byte array.
    byte[] SignatureBytes = Encoding.UTF8.GetBytes(MessageSignature);

    // Create the HMACSHA256 version of the storage key.
    HMACSHA256 SHA256 = new HMACSHA256(Convert.FromBase64String(storageAccountKey));

    // Compute the hash of the SignatureBytes and convert it to a base64 string.
    string signature = Convert.ToBase64String(SHA256.ComputeHash(SignatureBytes));

    // This is the actual header that will be added to the list of request headers.
    AuthenticationHeaderValue authHV = new AuthenticationHeaderValue("SharedKey",
        storageAccountName + ":" + Convert.ToBase64String(SHA256.ComputeHash(SignatureBytes)));
    return authHV;
}
```

當您執行此程式碼時，所產生的 MessageSignature 如下所示：

```
GET\n\n\n\n\n\n\n\n\n\n\n\nx-ms-date:Fri, 17 Nov 2017 01:07:37 GMT\nx-ms-version:2017-07-29\n/contosorest/\ncomp:list
```

以下是最終的 AuthorizationHeader 值：

```
SharedKey contosorest:Ms5sfwkA8nqTRw7Uury4MPHqM6Rj2nfgbYNvUKOa67w=
```

AuthorizationHeader 是在張貼回應之前，放在要求標頭中的最後一個標頭。

其涵蓋您需要知道的一切，以及可組成類別的程式碼，該類別可用來呼叫儲存體服務 REST API。

## <a name="how-about-another-example"></a>另一個範例如何？ 

讓我們看看如何變更程式碼以對容器 container-1 呼叫 ListBlobs。 這與用於列出容器的程式碼幾乎相同，唯一的差異在於 URI 及剖析回應的方式。 

如果您查看 [ListBlobs](/rest/api/storageservices/List-Blobs) 的參考文件，您會發現此方法為 GET 且 RequestURI 為：

```
https://myaccount.blob.core.windows.net/container-1?restype=container&comp=list
```

在 ListContainersAsyncREST 中，將設定 URI 的程式碼變更為 ListBlobs 的 API。 容器的名稱是 **container-1**。

```csharp
String uri = 
    string.Format("http://{0}.blob.core.windows.net/container-1?restype=container&comp=list",
      storageAccountName);

```

然後在您處理回應的地方，變更程式碼以尋找 blob，而不是容器。

```csharp
foreach (XElement container in x.Element("Blobs").Elements("Blob"))
{
    Console.WriteLine("Blob name = {0}", container.Element("Name").Value);
}
```

當您執行此範例時，您會取得如下所示的結果：

**正式標頭：**

```
x-ms-date:Fri, 17 Nov 2017 05:16:48 GMT\nx-ms-version:2017-07-29\n
```

**正式資源：**

```
/contosorest/container-1\ncomp:list\nrestype:container
```

**MessageSignature：**

```
GET\n\n\n\n\n\n\n\n\n\n\n\nx-ms-date:Fri, 17 Nov 2017 05:16:48 GMT
  \nx-ms-version:2017-07-29\n/contosorest/container-1\ncomp:list\nrestype:container
```

**AuthorizationHeader：**

```
SharedKey contosorest:uzvWZN1WUIv2LYC6e3En10/7EIQJ5X9KtFQqrZkxi6s=
```

下列值來自 [Fiddler](http://www.telerik.com/fiddler)：

**要求：**

```
GET http://contosorest.blob.core.windows.net/container-1?restype=container&comp=list HTTP/1.1
```

**要求標頭：**

```
x-ms-date: Fri, 17 Nov 2017 05:16:48 GMT
x-ms-version: 2017-07-29
Authorization: SharedKey contosorest:uzvWZN1WUIv2LYC6e3En10/7EIQJ5X9KtFQqrZkxi6s=
Host: contosorest.blob.core.windows.net
Connection: Keep-Alive
```

**執行後傳回的狀態碼和回應標頭：**

```
HTTP/1.1 200 OK
Content-Type: application/xml
Server: Windows-Azure-Blob/1.0 Microsoft-HTTPAPI/2.0
x-ms-request-id: 7e9316da-001e-0037-4063-5faf9d000000
x-ms-version: 2017-07-29
Date: Fri, 17 Nov 2017 05:20:21 GMT
Content-Length: 1135
```

**回應主體 (XML)：** 此 XML 回應會顯示 blob 和其屬性的清單。 

```xml
<?xml version="1.0" encoding="utf-8"?>
<EnumerationResults 
    ServiceEndpoint="http://contosorest.blob.core.windows.net/" ContainerName="container-1">
    <Blobs>
        <Blob>
            <Name>DogInCatTree.png</Name>
            <Properties><Last-Modified>Fri, 17 Nov 2017 01:41:14 GMT</Last-Modified>
            <Etag>0x8D52D5C4A4C96B0</Etag>
            <Content-Length>419416</Content-Length>
            <Content-Type>image/png</Content-Type>
            <Content-Encoding />
            <Content-Language />
            <Content-MD5 />
            <Cache-Control />
            <Content-Disposition />
            <BlobType>BlockBlob</BlobType>
            <LeaseStatus>unlocked</LeaseStatus>
            <LeaseState>available</LeaseState>
            <ServerEncrypted>true</ServerEncrypted>
            </Properties>
        </Blob>
        <Blob>
            <Name>GuyEyeingOreos.png</Name>
            <Properties>
                <Last-Modified>Fri, 17 Nov 2017 01:41:14 GMT</Last-Modified>
                <Etag>0x8D52D5C4A25A6F6</Etag>
                <Content-Length>167464</Content-Length>
                <Content-Type>image/png</Content-Type>
                <Content-Encoding />
                <Content-Language />
                <Content-MD5 />
                <Cache-Control />
                <Content-Disposition />
                <BlobType>BlockBlob</BlobType>
                <LeaseStatus>unlocked</LeaseStatus>
                <LeaseState>available</LeaseState>
                <ServerEncrypted>true</ServerEncrypted>
            </Properties>
            </Blob>
        </Blobs>
    <NextMarker />
</EnumerationResults>
```

## <a name="summary"></a>總結

在本文中，您已了解如何對 blob 儲存體 REST API 提出要求，以擷取容器清單或容器中的 blob 清單。 您也了解如何建立 REST API 呼叫的授權簽章、如何在 REST 要求中使用它，以及如何檢查回應。

## <a name="next-steps"></a>後續步驟

* [Blob 服務 REST API](/rest/api/storageservices/blob-service-rest-api)
* [檔案服務 REST API](/rest/api/storageservices/file-service-rest-api)
* [佇列服務 REST API](/rest/api/storageservices/queue-service-rest-api)