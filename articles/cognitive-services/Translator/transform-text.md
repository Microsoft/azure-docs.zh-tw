---
title: 使用 Microsoft Translator Text API 來轉換文字 | Microsoft Docs
description: 使用 Microsoft Translator Text API 來轉換文字。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.component: translator-text
ms.topic: article
ms.date: 12/14/2017
ms.author: v-jansko
ms.openlocfilehash: 5aa88f471a33a668b8385e46c2105a3139c142cb
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368642"
---
# <a name="how-to-use-the-transformtext-method"></a>如何使用 TransformText 方法

> [!NOTE]
> 此方法已被取代。 Translator Text API V3.0 中不提供此功能。

TransformText 方法是一個社交媒體的文字正規化函式，會傳回輸入內容的正規化形式。 針對機器翻譯或其他預期要有整齊輸入文字 (通常在社交媒體或使用者產生的內容中不會有) 的應用程式，此方法可用來作為前處理步驟。 此函式目前只對英文輸入有用。 

此方法是一種 RESTful 服務，透過 HTTP 來使用 GET。 它支援簡單的 XML 和 JSON 序列化。

## <a name="parameters"></a>參數

| 參數 | 說明 |
|:---|:---|
| Authorization 標頭 | **必要** 用來識別應用程式的 HTTP 標頭。 使用索引鍵：“Authorization” 和值："Bearer" + " " + 存取權杖。 如需詳細資料，請參閱這裡。|
| 語言 | **必要** 一個代表語言代碼的字串。 此參數僅支援英文，以 **en** 作為語言名稱。|
| category | **選擇性** 一個包含翻譯之分類或領域的字串。 此參數僅支援預設選項 **general**。|
| sentence | **必要** 您想要校正的句子。 |

## <a name="return-value"></a>傳回值

傳回值會提供轉換後的句子。

> [!div class="tabbedCodeSnippets"]
```json
GetTranslationsResponse Microsoft.Translator.GetTranslations(appId, text, from, to, maxTranslations, options); TransformTextResponse
{
int ec;            // A positive number representing an error condition
string em;         // A descriptive error message 
string sentence;   // transformed text
}
```

## <a name="example"></a>範例 

```csharp
using System;
using System.Text;
using System.Net;
using System.IO;
//Requires System.Runtime.Serialization assembly to be referenced
using System.Runtime.Serialization.Json;
using System.Runtime.Serialization;
//Requires System.Web assembly to be referenced
using System.Web;
using System.Media;
using System.Threading;
namespace MicrosoftTranslatorSdk.HttpSamples
{
    class Program
    {
        static void Main(string[] args)
        {
            AdmAccessToken admToken;
            string headerValue;
            //Get Client Id and Client Secret from https://datamarket.azure.com/developer/applications/
            //Refer obtaining AccessToken (http://msdn.microsoft.com/library/hh454950.aspx) 
            AdmAuthentication admAuth = new AdmAuthentication("clientID", "client secret");
            
            try
            {
                admToken = admAuth.GetAccessToken();
                // Create a header with the access_token property of the returned token
                headerValue = "Bearer " + admToken.access_token;
                TransformTextMethod(headerValue);
                Console.WriteLine("Press any key to continue...");
                Console.ReadKey(true);
            }
            catch (WebException e)
            {
                ProcessWebException(e);
                Console.WriteLine("Press any key to continue...");
                Console.ReadKey(true);
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
                Console.WriteLine("Press any key to continue...");
                Console.ReadKey(true);
            }
        }
        private static void TransformTextMethod(string authToken)
        {
            Console.WriteLine("Enter Text to transform: (e.g Dis is 2 strange i juss wanna go home sooooooon)");
            string textToTransform = Console.ReadLine();
            string language = "en";
            string domain = "general";
            //Keep appId parameter blank as we are sending access token in authorization header.
            string url = string.Format("http://api.microsofttranslator.com/V3/json/TransformText?sentence={0}&category={1}&language={2}",textToTransform,domain,language); ;
            TransformTextResponse transformTextResponse= new TransformTextResponse();
            HttpWebRequest httpWebRequest = (HttpWebRequest)WebRequest.Create(url);
            httpWebRequest.Headers.Add("Authorization", authToken);
            using (WebResponse response = httpWebRequest.GetResponse())
            {
                using( StreamReader sr = new StreamReader(response.GetResponseStream()))
                {
                    using (MemoryStream ms = new MemoryStream(Encoding.UTF8.GetBytes(sr.ReadToEnd())))
                    {
                        DataContractJsonSerializer serializer = new DataContractJsonSerializer(typeof(TransformTextResponse));
                        //Get deserialized TransformTextResponse object from JSON stream
                        transformTextResponse = (TransformTextResponse)serializer.ReadObject(ms);
                        Console.WriteLine("Transformed sentence: {0}", transformTextResponse.sentence);
                    }
                }
            }
        }
        private static void ProcessWebException(WebException e)
        {
            Console.WriteLine("{0}", e.ToString());
            // Obtain detailed error information
            string strResponse = string.Empty;
            using (HttpWebResponse response = (HttpWebResponse)e.Response)
            {
                using (Stream responseStream = response.GetResponseStream())
                {
                    using (StreamReader sr = new StreamReader(responseStream, System.Text.Encoding.ASCII))
                    {
                        strResponse = sr.ReadToEnd();
                    }
                }
            }
            Console.WriteLine("Http status code={0}, error message={1}", e.Status, strResponse);
        }
    }
    [DataContract]
    public class TransformTextResponse
    {
        [DataMember]
        public int ec { get; set; }
        [DataMember]
        public string em { get; set; }
        [DataMember]
        public string sentence { get; set; }
    }
    [DataContract]
    public class AdmAccessToken
    {
        [DataMember]
        public string access_token { get; set; }
        [DataMember]
        public string token_type { get; set; }
        [DataMember]
        public string expires_in { get; set; }
        [DataMember]
        public string scope { get; set; }
    }
    public class AdmAuthentication
    {
        public static readonly string DatamarketAccessUri = "https://datamarket.accesscontrol.windows.net/v2/OAuth2-13";
        private string clientId;
        private string clientSecret;
        private string request;
        private AdmAccessToken token;
        private Timer accessTokenRenewer;
        //Access token expires every 10 minutes. Renew it every 9 minutes only.
        private const int RefreshTokenDuration = 9;
        public AdmAuthentication(string clientId, string clientSecret)
        {
            this.clientId = clientId;
            this.clientSecret = clientSecret;
            //If clientid or client secret has special characters, encode before sending request
            this.request = string.Format("grant_type=client_credentials&client_id={0}&client_secret={1}&scope=http://api.microsofttranslator.com", HttpUtility.UrlEncode(clientId), HttpUtility.UrlEncode(clientSecret));
            this.token = HttpPost(DatamarketAccessUri, this.request);
            //renew the token every specfied minutes
            accessTokenRenewer = new Timer(new TimerCallback(OnTokenExpiredCallback), this, TimeSpan.FromMinutes(RefreshTokenDuration), TimeSpan.FromMilliseconds(-1));
        }
        public AdmAccessToken GetAccessToken()
        {
            return this.token;
        }
        private void RenewAccessToken()
        {
            AdmAccessToken newAccessToken = HttpPost(DatamarketAccessUri, this.request);
            this.token = newAccessToken;
            Console.WriteLine(string.Format("Renewed token for user: {0} is: {1}", this.clientId, this.token.access_token));
        }
        private void OnTokenExpiredCallback(object stateInfo)
        {
            try
            {
                RenewAccessToken();
            }
            catch (Exception ex)
            {
                Console.WriteLine(string.Format("Failed renewing access token. Details: {0}", ex.Message));
            }
            finally
            {
                try
                {
                    accessTokenRenewer.Change(TimeSpan.FromMinutes(RefreshTokenDuration), TimeSpan.FromMilliseconds(-1));
                }
                catch (Exception ex)
                {
                    Console.WriteLine(string.Format("Failed to reschedule the timer to renew access token. Details: {0}", ex.Message));
                }
            }
        }
        private AdmAccessToken HttpPost(string DatamarketAccessUri, string requestDetails)
        {
            //Prepare OAuth request 
            WebRequest webRequest = WebRequest.Create(DatamarketAccessUri);
            webRequest.ContentType = "application/x-www-form-urlencoded";
            webRequest.Method = "POST";
            byte[] bytes = Encoding.ASCII.GetBytes(requestDetails);
            webRequest.ContentLength = bytes.Length;
            using (Stream outputStream = webRequest.GetRequestStream())
            {
                outputStream.Write(bytes, 0, bytes.Length);
            }
            using (WebResponse webResponse = webRequest.GetResponse())
            {
                DataContractJsonSerializer serializer = new DataContractJsonSerializer(typeof(AdmAccessToken));
                //Get deserialized object from JSON stream
                AdmAccessToken token = (AdmAccessToken)serializer.ReadObject(webResponse.GetResponseStream());
                return token;
            }
        }
    }
} 

```
