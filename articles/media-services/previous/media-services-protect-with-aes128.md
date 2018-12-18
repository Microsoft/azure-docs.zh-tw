---
title: 使用 AES-128 動態加密和金鑰傳遞服務 | Microsoft Docs
description: 使用 Microsoft Azure 媒體服務來傳遞您使用 AES 128 位元加密金鑰加密的內容。 媒體服務也提供加密金鑰傳遞服務，將加密金鑰傳遞至授權的使用者。 本主題展示如何利用 AES-128 動態加密，以及使用金鑰傳遞服務。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 4d2c10af-9ee0-408f-899b-33fa4c1d89b9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/18/2018
ms.author: juliako
ms.openlocfilehash: 9f3fe36eab7dc7fd1921c4e225b5a173fe2e9243
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364868"
---
# <a name="use-aes-128-dynamic-encryption-and-the-key-delivery-service"></a>使用 AES-128 動態加密和金鑰傳遞服務
> [!div class="op_single_selector"]
> * [.NET](media-services-protect-with-aes128.md)
> * [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
> * [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
>  

> [!NOTE]
> 若要取得最新的 Java SDK 版本並開始使用 Java 進行開發，請參閱[開始使用適用於 Azure 媒體服務的 Java 用戶端 SDK](https://docs.microsoft.com/azure/media-services/media-services-java-how-to-use)。 <br/>
> 若要下載適用於媒體服務的最新 PHP SDK，請在 [Packagist 存放庫](https://packagist.org/packages/microsoft/windowsazure#v0.5.7)中尋找 0.5.7 版的 Microsoft/WindowsAzure 套件。  

## <a name="overview"></a>概觀
> [!NOTE]
> 如需如何使用進階加密標準 (AES) 傳遞到 macOS 上的 Safari 來加密內容的相關資訊，請參閱[此部落格文章](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/)。
> 如需如何使用 AES 加密保護媒體內容的概觀，請參閱[此視訊](https://channel9.msdn.com/Shows/Azure-Friday/Azure-Media-Services-Protecting-your-Media-Content-with-AES-Encryption)。
> 
> 

 您可以利用 128 位元加密金鑰，使用媒體服務提供 HTTP Live Streaming (HLS) 和透過 AES 加密的 Smooth Streaming。 媒體服務也提供加密金鑰傳遞服務，將加密金鑰傳遞至授權的使用者。 如果您需要媒體服務來加密資產，就需要建立加密金鑰與資產的關聯，同時設定金鑰的授權原則。 播放器要求串流時，媒體服務便會透過 AES 加密，使用指定的金鑰動態加密您的內容。 為了將串流解密，播放程式將向金鑰傳遞服務要求金鑰。 為了決定使用者是否有權取得金鑰，服務會評估為金鑰指定的授權原則。

媒體服務支援多種方式來驗證提出金鑰要求的使用者。 內容金鑰授權原則可能有一個或多個授權限制，可能是 Open 或權杖限制。 權杖限制原則必須伴隨 Security Token Service (STS) 所發出的權杖。 媒體服務支援使用[簡單 Web 權杖](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_2) (SWT) 和 [JSON Web 權杖](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_3) (JWT) 格式的權杖。 如需詳細資訊，請參閱[設定內容金鑰的授權原則](media-services-protect-with-aes128.md#configure_key_auth_policy)。

若要利用動態加密，您需有一個資源，其中包含一組多位元速率 MP4 檔案或多位元速率 Smooth Streaming 來源檔案。 您也需要設定資產的傳遞原則 (本文稍後會加以描述)。 然後，根據串流 URL 中指定的格式，隨選資料流處理伺服器確保會以您選取的通訊協定傳遞串流。 如此一來，您只須以單一儲存格式儲存及支付檔案。 媒體服務會根據用戶單的要求建置及傳遞適當的回應。

本文有助於開發人員開發提供受保護媒體的應用程式。 本文示範如何利用授權原則設定金鑰傳遞服務，只讓已獲授權的用戶端能夠收到加密金鑰。 它也會展示如何使用動態加密。


## <a name="aes-128-dynamic-encryption-and-key-delivery-service-workflow"></a>使用 AES-128 動態加密和金鑰傳遞服務工作流程

當您利用 AES 並使用媒體服務金鑰傳遞服務，以及使用動態加密來加密資產時，請執行下列一般步驟：

1. [建立資產並將檔案上傳到資產](media-services-protect-with-aes128.md#create_asset)。

2. [將包含檔案的資產編碼為自適性位元速率 MP4 集](media-services-protect-with-aes128.md#encode_asset)。

3. [建立內容金鑰，並將它與編碼的資產產生關聯](media-services-protect-with-aes128.md#create_contentkey)。 在媒體服務中，內容金鑰包含資產的加密金鑰。

4. [設定內容金鑰的授權原則](media-services-protect-with-aes128.md#configure_key_auth_policy)。 您必須設定內容金鑰授權原則。 用戶端必須先符合原則，系統才會將內容金鑰傳遞給用戶端。

5. [設定資產的傳遞原則](media-services-protect-with-aes128.md#configure_asset_delivery_policy)。 傳遞原則設定包含金鑰取得 URL 和初始化向量 (IV)。 (AES-128 需要相同的 IV 來加密和解密。)組態還包括傳遞通訊協定 (例如，MPEG-DASH、HLS、Smooth Streaming 或全部)、動態加密的類型 (例如，信封或無動態加密)。

    您可以將不同的原則套用至相同資產上的每一個通訊協定。 例如，您可以將 PlayReady 加密套用到 Smooth/DASH，以及將 AES 信封加密套用到 HLS。 傳遞原則中未定義的任何通訊協定都會封鎖串流。 (範例之一是如果您新增僅指定 HLS 作為通訊協定的單一原則)。例外情形是當您完全未定義資產傳遞原則時。 接著，允許所有通訊協定，不受阻礙。

6. [建立隨選定位器](media-services-protect-with-aes128.md#create_locator)以取得串流 URL。

本文也會示範[用戶端應用程式如何向金鑰傳遞服務要求金鑰](media-services-protect-with-aes128.md#client_request)。

您可以在本文結尾處找到完整的 [.NET 範例](media-services-protect-with-aes128.md#example)。

下圖示範上述的工作流程。 這裡的權杖用於驗證。

![利用 AES 128 保護](./media/media-services-content-protection-overview/media-services-content-protection-with-aes.png)

本文的其餘部分會提供說明、程式碼範例，以及示範如何達成上述工作之主題的連結。

## <a name="current-limitations"></a>目前的限制
如果您新增或更新資產的傳遞原則，就必須刪除現有的定位器，並建立新的定位器。

## <a id="create_asset"></a>建立資產並將檔案上傳到資產
如需管理、編碼及串流處理您的視訊，您必須先將內容上傳到媒體服務。 在上傳後，您的內容就會安全地儲存在雲端，以進一步進行處理和串流處理。 

如需詳細資訊，請參閱[將檔案上傳到媒體服務帳戶](media-services-dotnet-upload-files.md)。

## <a id="encode_asset"></a>將包含檔案的資產編碼為自適性位元速率 MP4 集
使用動態加密時，您要建立一個資產，其中包含一組多位元速率 MP4 檔案，或多位元速率 Smooth Streaming 來源檔案。 然後，隨選串流處理伺服器會以資訊清單或片段要求中的指定格式作為基礎，確保您以自己選取的通訊協定接收串流。 然後，您只需要以單一儲存格式就能儲存及支付檔案。 媒體服務會根據用戶單的要求建置及傳遞適當的回應。 如需詳細資訊，請參閱[動態封裝概觀](media-services-dynamic-packaging-overview.md)。

>[!NOTE]
>建立媒體服務帳戶時，預設串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您需要串流內容的串流端點就必須處於 [執行中] 狀態。 
>
>此外，為了使用動態封裝和動態加密，您的資產必須包含一組調適性位元速率 MP4，或調適性位元速率 Smooth Streaming 檔案。

如需編碼方式的指示，請參閱[使用媒體編碼器標準將資產編碼](media-services-dotnet-encode-with-media-encoder-standard.md)。

## <a id="create_contentkey"></a>建立內容金鑰並將它與編碼的資產產生關聯
在媒體服務中，內容金鑰包含您要加密資產時使用的金鑰。

如需詳細資訊，請參閱[建立內容金鑰](media-services-dotnet-create-contentkey.md)。

## <a id="configure_key_auth_policy"></a>設定內容金鑰的授權原則
媒體服務支援多種方式來驗證提出金鑰要求的使用者。 您必須設定內容金鑰授權原則。 用戶端 (播放器) 必須先符合原則，系統才會將金鑰傳遞給用戶端。 內容金鑰授權原則可能會有一個或多個授權限制：Open、權杖限制或 IP 限制。

如需詳細資訊，請參閱[設定內容金鑰授權原則](media-services-dotnet-configure-content-key-auth-policy.md)。

## <a id="configure_asset_delivery_policy"></a>設定資產傳遞原則
設定資產的傳遞原則。 資產傳遞原則組態包括：

* 金鑰取得 URL。 
* 用於信封加密的初始化向量 (IV)。 AES-128 需要相同的 IV 來加密和解密。 
* 資產傳遞通訊協定 (例如，MPEG DASH、HLS、Smooth Streaming 或全部)。
* 動態加密的類型 (例如，AES 信封) 或沒有動態加密。 

如需詳細資訊，請參閱[設定資產傳遞原則](media-services-dotnet-configure-asset-delivery-policy.md)。

## <a id="create_locator"></a>建立隨選串流定位器以取得串流 URL
您必須為使用者提供 Smooth Streaming、DASH 或 HLS 的串流 URL。

> [!NOTE]
> 如果您新增或更新資產的傳遞原則，就必須刪除現有的定位器，並建立新的定位器。
> 
> 

如需有關如何發佈資產，並建置串流 URL 的指示，請參閱 [建置串流 URL](media-services-deliver-streaming-content.md)。

## <a name="get-a-test-token"></a>取得測試權杖
根據用於金鑰授權原則的權杖限制取得測試權杖。

```csharp
    // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
    // back into a TokenRestrictionTemplate class instance.
    TokenRestrictionTemplate tokenTemplate = 
        TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

    // Generate a test token based on the data in the given TokenRestrictionTemplate.
    //The GenerateTestToken method returns the token without the word "Bearer" in front
    //so you have to add it in front of the token string. 
    string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate);
    Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
```

您可以使用 [Azure 媒體服務播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)來測試您的串流。

## <a id="client_request"></a>您的用戶端如何從金鑰傳遞服務要求金鑰？
在上一個步驟中，您可以建構指向資訊清單檔案的 URL。 您的用戶端必須從串流資訊清單檔案擷取所需的資訊，才能向金鑰傳遞服務提出要求。

### <a name="manifest-files"></a>資訊清單檔案
用戶端必須從資訊清單檔案擷取 URL (其中也包含內容金鑰識別碼 [kid]) 值。 用戶端接著會嘗試從金鑰傳遞服務取得加密金鑰。 用戶端也必須擷取 IV 值，並使用它來進行串流解密。 下列程式碼片段說明 Smooth Streaming 資訊清單的 <Protection> 元素：

```xml
    <Protection>
      <ProtectionHeader SystemID="B47B251A-2409-4B42-958E-08DBAE7B4EE9">
        <ContentProtection xmlns:sea="urn:mpeg:dash:schema:sea:2012" schemeIdUri="urn:mpeg:dash:sea:2012">
          <sea:SegmentEncryption schemeIdUri="urn:mpeg:dash:sea:aes128-cbc:2013"/>
          <sea:KeySystem keySystemUri="urn:mpeg:dash:sea:keysys:http:2013"/>
          <sea:CryptoPeriod IV="0xD7D7D7D7D7D7D7D7D7D7D7D7D7D7D7D7" 
                            keyUriTemplate="https://wamsbayclus001kd-hs.cloudapp.net/HlsHandler.ashx?
                                            kid=da3813af-55e6-48e7-aa9f-a4d6031f7b4d"/>
        </ContentProtection>
      </ProtectionHeader>
    </Protection>
```

在 HLS 的案例中，根資訊清單會分成區段檔案。 

例如，根資訊清單是： http://test001.origin.mediaservices.windows.net/8bfe7d6f-34e3-4d1a-b289-3e48a8762490/BigBuckBunny.ism/manifest(format=m3u8-aapl)。 它包含區段檔案名稱的清單。

    . . . 
    #EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=630133,RESOLUTION=424x240,CODECS="avc1.4d4015,mp4a.40.2",AUDIO="audio"
    QualityLevels(514369)/Manifest(video,format=m3u8-aapl)
    #EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=965441,RESOLUTION=636x356,CODECS="avc1.4d401e,mp4a.40.2",AUDIO="audio"
    QualityLevels(842459)/Manifest(video,format=m3u8-aapl)
    …

如果您在文字編輯器中開啟其中一個區段檔案 (例如， http://test001.origin.mediaservices.windows.net/8bfe7d6f-34e3-4d1a-b289-3e48a8762490/BigBuckBunny.ism/QualityLevels(514369)/Manifest(video，格式 = m3u8 aapl)，它包含 #EXT-X-KEY，表示檔案已加密。

    #EXTM3U
    #EXT-X-VERSION:4
    #EXT-X-ALLOW-CACHE:NO
    #EXT-X-MEDIA-SEQUENCE:0
    #EXT-X-TARGETDURATION:9
    #EXT-X-KEY:METHOD=AES-128,
    URI="https://wamsbayclus001kd-hs.cloudapp.net/HlsHandler.ashx?
         kid=da3813af-55e6-48e7-aa9f-a4d6031f7b4d",
            IV=0XD7D7D7D7D7D7D7D7D7D7D7D7D7D7D7D7
    #EXT-X-PROGRAM-DATE-TIME:1970-01-01T00:00:00.000+00:00
    #EXTINF:8.425708,no-desc
    Fragments(video=0,format=m3u8-aapl)
    #EXT-X-ENDLIST

>[!NOTE] 
>如果您計劃在 Safari 中播放 AES 加密的 HLS，請參閱[這篇部落格](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/)。

### <a name="request-the-key-from-the-key-delivery-service"></a>從金鑰傳遞服務要求金鑰

下列程式碼展示如何使用金鑰傳遞 URI (擷取自資訊清單) 和權杖，將要求傳送至媒體服務金鑰傳遞服務。 (本文未說明如何從 STS 取得 SWT。)

```csharp
    private byte[] GetDeliveryKey(Uri keyDeliveryUri, string token)
    {
        HttpWebRequest request = (HttpWebRequest)WebRequest.Create(keyDeliveryUri);

        request.Method = "POST";
        request.ContentType = "text/xml";
        if (!string.IsNullOrEmpty(token))
        {
            request.Headers[AuthorizationHeader] = token;
        }
        request.ContentLength = 0;

        var response = request.GetResponse();

        var stream = response.GetResponseStream();
        if (stream == null)
        {
            throw new NullReferenceException("Response stream is null");
        }

        var buffer = new byte[256];
        var length = 0;
        while (stream.CanRead && length <= buffer.Length)
        {
            var nexByte = stream.ReadByte();
            if (nexByte == -1)
            {
                break;
            }
            buffer[length] = (byte)nexByte;
            length++;
        }
        response.Close();

        // AES keys must be exactly 16 bytes (128 bits).
        var key = new byte[length];
        Array.Copy(buffer, key, length);
        return key;
    }
```

## <a name="protect-your-content-with-aes-128-by-using-net"></a>使用 .NET 透過 AES-128 保護內容

### <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

1. 設定您的開發環境，並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。

2. 將下列元素新增至 app.config 檔案中定義的 appSettings：

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

### <a id="example"></a>範例

以本章節中所顯示的程式碼覆寫 Program.cs 檔案中的程式碼。
 
>[!NOTE]
>對於不同的媒體服務原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的天數/存取權限，請使用相同的原則識別碼。 例如，為預定要長時間維持就地 (非上傳原則) 的定位器原則。 如需詳細資訊，請參閱[使用媒體服務 .NET SDK 管理資產和相關的實體](media-services-dotnet-manage-entities.md#limit-access-policies)中的「限制存取原則」一節。

請務必更新變數，以指向您的輸入檔案所在的資料夾。

[!code-csharp[Main](../../../samples-mediaservices-encryptionaes/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs)]

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
