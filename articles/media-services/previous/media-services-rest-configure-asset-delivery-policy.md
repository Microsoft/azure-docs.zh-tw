---
title: 使用媒體服務 REST API 設定資產傳遞原則 | Microsoft Docs
description: 本主題說明如何使用媒體服務 REST API 設定不同的資產傳遞原則。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 5cb9d32a-e68b-4585-aa82-58dded0691d0
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/07/2017
ms.author: juliako
ms.openlocfilehash: d6f18363cceaf279d92ada77f52d39b7f1d12f65
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785967"
---
# <a name="configuring-asset-delivery-policies"></a>設定資產傳遞原則
[!INCLUDE [media-services-selector-asset-delivery-policy](../../../includes/media-services-selector-asset-delivery-policy.md)]

如果您打算傳遞動態加密的資產，媒體服務內容傳遞工作流程的其中一個步驟，是設定資產的傳遞原則。 資產傳遞原則會告訴媒體服務您想要如何傳遞資產：您的資產應該動態封裝成哪個串流通訊協定 (如 MPEG DASH、HLS、Smooth Streaming 或所有)，您是否想要動態加密您的資產及其方式 (信封或一般加密)。

本主題討論建立和設定資產傳遞原則的原因與方法。

>[!NOTE]
>建立 AMS 帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。 
>
>此外，為了能夠使用動態封裝和動態加密功能，您的資產必須包含一組調適性位元速率 MP4 或調適性位元速率 Smooth Streaming 檔案。

您可以將不同的原則套用至相同的資產。 例如，您可以將 PlayReady 加密套用到 Smooth Streaming，將 AES 信封加密套用到 MPEG DASH 和 HLS。 傳遞原則中未定義的任何通訊協定 (例如，您加入單一原則，它只有指定 HLS 做為通訊協定) 將會遭到封鎖無法串流。 這個狀況的例外情形是您完全沒有定義資產傳遞原則之時。 那麼，將允許所有通訊協定，不受阻礙。

如果您想要傳遞儲存體加密資產，就必須設定資產的傳遞原則。 資產可以串流處理之前，串流伺服器會移除儲存體加密，並使用指定的傳遞原則來串流您的內容。 例如，若要傳遞使用進階加密標準 (AES) 信封加密金鑰加密的資產，請將原則類型設定為 **DynamicEnvelopeEncryption**。 如果您要移除儲存體加密，並且不受阻礙地串流資產，請將原則類型設定為 **NoDynamicEncryption**。 下列範例示範如何設定這些原則類型。

視您如何設定資產傳遞原則而定，您可以動態封裝、動態加密，以及串流下列串流通訊協定：Smooth Streaming、HLS、MPEG DASH 資料流。

下列清單顯示您用來串流 Smooth、HLS、DASH 的格式。

Smooth Streaming：

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest

HLS：

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=m3u8-aapl)

MPEG DASH

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=mpd-time-csf)


如需有關如何發佈資產，並建置串流 URL 的指示，請參閱 [建置串流 URL](media-services-deliver-streaming-content.md)。

## <a name="considerations"></a>考量
* 當資產的 OnDemand (串流) 定位器仍存在時，您無法刪除與該資產關聯的 AssetDeliveryPolicy。 建議刪除原則之前，先將該原則從資產移除。
* 未設定資產傳遞原則時，將無法於儲存空間已加密的資產建立串流定位器。  如果資產的儲存空間未加密，系統會讓您建立定位器，並直接串流資產而不使用資產傳遞原則。
* 您可以有多個資產傳遞原則與一個資產關聯，但只能指定一種方法處理特定的 AssetDeliveryProtocol。  這表示如果您嘗試連結二個指定 AssetDeliveryProtocol.SmoothStreaming 通訊協定的傳遞原則時將會導致錯誤，因為當用戶端發出 Smooth Streaming 要求時，系統會不知道要套用哪一個原則。
* 如果您有包含現有串流定位器的資產，則您無法將新原則連接到該資產，請解除現有原則與資產的連結，或更新與資產關聯的傳遞原則。  您必須先移除串流定位器，調整原則，然後重新建立串流定位器。  重新建立串流定位器時，您可以使用同一個 locatorId，但必須確定不會對用戶端造成問題，因為原始或下游 CDN 可能會快取內容。

>[!NOTE]

>在媒體服務中存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。 如需詳細資訊，請參閱 [媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。

## <a name="connect-to-media-services"></a>連線到媒體服務

如需連線至 AMS API 的詳細資訊，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 

## <a name="clear-asset-delivery-policy"></a>清除資產傳遞原則
### <a id="create_asset_delivery_policy"></a>建立資產傳遞原則
下列 HTTP 要求會建立資產傳遞原則，指定不要套用動態加密，並以下列任何通訊協定傳遞資料流：MPEG DASH、HLS 和 Smooth Streaming 通訊協定。 

如需建立 AssetDeliveryPolicy 時可以指定之值的相關資訊，請參閱[定義 AssetDeliveryPolicy 時使用的類型](#types) 一節。   

要求：

    POST https://media.windows.net/api/AssetDeliveryPolicies HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-version: 2.17
    x-ms-client-request-id: 4651882c-d7ad-4d5e-86ab-f07f47dcb41e
    Host: media.windows.net

    {"Name":"Clear Policy",
    "AssetDeliveryProtocol":7,
    "AssetDeliveryPolicyType":2,
    "AssetDeliveryConfiguration":null}

回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 363
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://media.windows.net/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3A92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd')
    Server: Microsoft-IIS/8.5
    x-ms-client-request-id: 4651882c-d7ad-4d5e-86ab-f07f47dcb41e
    request-id: 6aedbf93-4bc2-4586-8845-fd45590136af
    x-ms-request-id: 6aedbf93-4bc2-4586-8845-fd45590136af
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Sun, 08 Feb 2015 06:21:27 GMT

    {"odata.metadata":"https://media.windows.net/api/$metadata#AssetDeliveryPolicies/@Element",
    "Id":"nb:adpid:UUID:92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd",
    "Name":"Clear Policy",
    "AssetDeliveryProtocol":7,
    "AssetDeliveryPolicyType":2,
    "AssetDeliveryConfiguration":null,
    "Created":"2015-02-08T06:21:27.6908329Z",
    "LastModified":"2015-02-08T06:21:27.6908329Z"}

### <a id="link_asset_with_asset_delivery_policy"></a>連結資產與資產傳遞原則
下列 HTTP 要求會將指定的資產連結至資產傳遞原則。

要求：

    POST https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A86933344-9539-4d0c-be7d-f842458693e0')/$links/DeliveryPolicies HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Content-Type: application/json
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-version: 2.17
    x-ms-client-request-id: 56d2763f-6e72-419d-ba3c-685f6db97e81
    Host: media.windows.net

    {"uri":"https://media.windows.net/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3A92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd')"}

回應：

    HTTP/1.1 204 No Content


## <a name="dynamicenvelopeencryption-asset-delivery-policy"></a>DynamicEnvelopeEncryption 資產傳遞原則
### <a name="create-content-key-of-the-envelopeencryption-type-and-link-it-to-the-asset"></a>建立 EnvelopeEncryption 類型的內容金鑰，並將它連結到資產
當指定 DynamicEnvelopeEncryption 傳遞原則時，您必須將資產連結到 EnvelopeEncryption 類型的內容金鑰。 如需詳細資訊，請參閱 [建立內容金鑰](media-services-rest-create-contentkey.md)。

### <a id="get_delivery_url"></a>取得傳遞 URL
針對前一個步驟中建立的內容金鑰的指定傳遞方法，取得傳遞 URL。 用戶端會使用傳回的 URL 要求 AES 金鑰或 PlayReady 授權，以便播放受保護的內容。

指定要在 HTTP 要求主體中取得的 URL 類型。 如果您要使用 PlayReady 保護您的內容，請要求媒體服務 PlayReady 授權取得 URL，並針對 keyDeliveryType 使用 1：{"keyDeliveryType":1}。 如果您要使用信封加密保護您的內容，請針對 keyDeliveryType 指定 2，來要求金鑰取得 URL：{"keyDeliveryType":2}。

要求：

    POST https://media.windows.net/api/ContentKeys('nb:kid:UUID:dc88f996-2859-4cf7-a279-c52a9d6b2f04')/GetKeyDeliveryUrl HTTP/1.1
    Content-Type: application/json
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-version: 2.17
    x-ms-client-request-id: 569d4b7c-a446-4edc-b77c-9fb686083dd8
    Host: media.windows.net
    Content-Length: 21

    {"keyDeliveryType":2}

回應：

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Length: 198
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Server: Microsoft-IIS/8.5
    x-ms-client-request-id: 569d4b7c-a446-4edc-b77c-9fb686083dd8
    request-id: d26f65d2-fe65-4136-8fcf-31545be68377
    x-ms-request-id: d26f65d2-fe65-4136-8fcf-31545be68377
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Sun, 08 Feb 2015 21:42:30 GMT

    {"odata.metadata":"media.windows.net/api/$metadata#Edm.String","value":"https://amsaccount1.keydelivery.mediaservices.windows.net/?KID=dc88f996-2859-4cf7-a279-c52a9d6b2f04"}


### <a name="create-asset-delivery-policy"></a>建立資產傳遞原則
下列 HTTP 要求會建立 **AssetDeliveryPolicy**，它會設定為將動態信封加密 (**DynamicEnvelopeEncryption**) 套用到 **HLS** 通訊協定 (在此範例中，其他通訊協定將會被封鎖，無法串流)。 

如需建立 AssetDeliveryPolicy 時可以指定之值的相關資訊，請參閱[定義 AssetDeliveryPolicy 時使用的類型](#types) 一節。   

要求：

    POST https://media.windows.net/api/AssetDeliveryPolicies HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    User-Agent: Microsoft ADO.NET Data Services
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-version: 2.17
    x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
    Host: media.windows.net

    {"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":4,"AssetDeliveryPolicyType":3,"AssetDeliveryConfiguration":"[{\"Key\":2,\"Value\":\"https:\\/\\/amsaccount1.keydelivery.mediaservices.windows.net\\/\"}]"}


回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 460
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: media.windows.net/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3Aec9b994e-672c-4a5b-8490-a464eeb7964b')
    Server: Microsoft-IIS/8.5
    x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
    request-id: c2a1ac0e-9644-474f-b38f-b9541c3a7c5f
    x-ms-request-id: c2a1ac0e-9644-474f-b38f-b9541c3a7c5f
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Mon, 09 Feb 2015 05:24:38 GMT

    {"odata.metadata":"media.windows.net/api/$metadata#AssetDeliveryPolicies/@Element","Id":"nb:adpid:UUID:ec9b994e-672c-4a5b-8490-a464eeb7964b","Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":4,"AssetDeliveryPolicyType":3,"AssetDeliveryConfiguration":"[{\"Key\":2,\"Value\":\"https:\\/\\/amsaccount1.keydelivery.mediaservices.windows.net\\/\"}]","Created":"2015-02-09T05:24:38.9167436Z","LastModified":"2015-02-09T05:24:38.9167436Z"}


### <a name="link-asset-with-asset-delivery-policy"></a>連結資產與資產傳遞原則
請參閱 [連結資產與資產傳遞原則](#link_asset_with_asset_delivery_policy)

## <a name="dynamiccommonencryption-asset-delivery-policy"></a>DynamicCommonEncryption 資產傳遞原則
### <a name="create-content-key-of-the-commonencryption-type-and-link-it-to-the-asset"></a>建立 CommonEncryption 類型的內容金鑰，並將它連結到資產
當指定 DynamicCommonEncryption 傳遞原則時，您必須將資產連結到 CommonEncryption 類型的內容金鑰。 如需詳細資訊，請參閱 [建立內容金鑰](media-services-rest-create-contentkey.md)。

### <a name="get-delivery-url"></a>取得傳遞 URL
針對前一個步驟中建立的內容金鑰的 PlayReady 傳遞方法，取得傳遞 URL。 用戶端會使用傳回的 URL 要求 PlayReady 授權，以便播放受保護的內容。 如需詳細資訊，請參閱 [取得傳遞 URL](#get_delivery_url)。

### <a name="create-asset-delivery-policy"></a>建立資產傳遞原則
下列 HTTP 要求會建立 **AssetDeliveryPolicy**，它會設定為將動態一般加密 (**DynamicCommonEncryption**) 套用到 **Smooth Streaming** 通訊協定 (在此範例中，其他通訊協定將會被封鎖，無法串流)。 

如需建立 AssetDeliveryPolicy 時可以指定之值的相關資訊，請參閱[定義 AssetDeliveryPolicy 時使用的類型](#types) 一節。   

要求：

    POST https://media.windows.net/api/AssetDeliveryPolicies HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    User-Agent: Microsoft ADO.NET Data Services
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-version: 2.17
    x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
    Host: media.windows.net

    {"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":1,"AssetDeliveryPolicyType":4,"AssetDeliveryConfiguration":"[{\"Key\":2,\"Value\":\"https:\\/\\/amsaccount1.keydelivery.mediaservices.windows.net\/PlayReady\/"}]"}


如果您想要使用 Widevine DRM 保護內容，請將 AssetDeliveryConfiguration 值更新為使用 WidevineLicenseAcquisitionUrl (其值為 7) 和指定授權傳遞服務的 URL。 您可以使用下列 AMS 合作夥伴來助您傳遞 Widevine 授權：[Axinom](http://www.axinom.com/press/ibc-axinom-drm-6/)、[EZDRM](http://ezdrm.com/)、[castLabs](http://castlabs.com/company/partners/azure/)。

例如︰ 

    {"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":2,"AssetDeliveryPolicyType":4,"AssetDeliveryConfiguration":"[{\"Key\":7,\"Value\":\"https:\\/\\/example.net\/WidevineLicenseAcquisition\/"}]"}

> [!NOTE]
> 以 Widevine 加密時，就只能使用 DASH 來傳遞。 請務必在資產傳遞通訊協定中指定 DASH (2)。
> 
> 

### <a name="link-asset-with-asset-delivery-policy"></a>連結資產與資產傳遞原則
請參閱 [連結資產與資產傳遞原則](#link_asset_with_asset_delivery_policy)

## <a id="types"></a>定義 AssetDeliveryPolicy 時使用的類型

### <a name="assetdeliveryprotocol"></a>AssetDeliveryProtocol

下列列舉描述您可以針對資產傳遞通訊協定設定的值。

    [Flags]
    public enum AssetDeliveryProtocol
    {
        /// <summary>
        /// No protocols.
        /// </summary>
        None = 0x0,

        /// <summary>
        /// Smooth streaming protocol.
        /// </summary>
        SmoothStreaming = 0x1,

        /// <summary>
        /// MPEG Dynamic Adaptive Streaming over HTTP (DASH)
        /// </summary>
        Dash = 0x2,

        /// <summary>
        /// Apple HTTP Live Streaming protocol.
        /// </summary>
        HLS = 0x4,

        ProgressiveDownload = 0x10, 
 
        /// <summary>
        /// Include all protocols.
        /// </summary>
        All = 0xFFFF
    }

### <a name="assetdeliverypolicytype"></a>AssetDeliveryPolicyType

下列列舉描述您可以針對資產傳遞原則類型設定的值。  

    public enum AssetDeliveryPolicyType
    {
        /// <summary>
        /// Delivery Policy Type not set.  An invalid value.
        /// </summary>
        None,

        /// <summary>
        /// The Asset should not be delivered via this AssetDeliveryProtocol. 
        /// </summary>
        Blocked, 

        /// <summary>
        /// Do not apply dynamic encryption to the asset.
        /// </summary>
        /// 
        NoDynamicEncryption,  

        /// <summary>
        /// Apply Dynamic Envelope encryption.
        /// </summary>
        DynamicEnvelopeEncryption,

        /// <summary>
        /// Apply Dynamic Common encryption.
        /// </summary>
        DynamicCommonEncryption
        }

### <a name="contentkeydeliverytype"></a>ContentKeyDeliveryType

下列列舉描述您可用來設定將內容金鑰傳遞給用戶端之方法的值。
    
    public enum ContentKeyDeliveryType
    {
        /// <summary>
        /// None.
        ///
        </summary>
        None = 0,

        /// <summary>
        /// Use PlayReady License acquistion protocol
        ///
        </summary>
        PlayReadyLicense = 1,

        /// <summary>
        /// Use MPEG Baseline HTTP key protocol.
        ///
        </summary>
        BaselineHttp = 2,

        /// <summary>
        /// Use Widevine License acquistion protocol
        ///
        </summary>
        Widevine = 3

    }


### <a name="assetdeliverypolicyconfigurationkey"></a>AssetDeliveryPolicyConfigurationKey

下列列舉描述您可以設定的值，以便設定用來取得資產傳遞原則特定設定的金鑰。

    public enum AssetDeliveryPolicyConfigurationKey
    {
        /// <summary>
        /// No policies.
        /// </summary>
        None,

        /// <summary>
        /// Exact Envelope key URL.
        /// </summary>
        EnvelopeKeyAcquisitionUrl,

        /// <summary>
        /// Base key url that will have KID=<Guid> appended for Envelope.
        /// </summary>
        EnvelopeBaseKeyAcquisitionUrl,

        /// <summary>
        /// The initialization vector to use for envelope encryption in Base64 format.
        /// </summary>
        EnvelopeEncryptionIVAsBase64,

        /// <summary>
        /// The PlayReady License Acquisition Url to use for common encryption.
        /// </summary>
        PlayReadyLicenseAcquisitionUrl,

        /// <summary>
        /// The PlayReady Custom Attributes to add to the PlayReady Content Header
        /// </summary>
        PlayReadyCustomAttributes,

        /// <summary>
        /// The initialization vector to use for envelope encryption.
        /// </summary>
        EnvelopeEncryptionIV,

        /// <summary>
        /// Widevine DRM acquisition url
        /// </summary>
        WidevineLicenseAcquisitionUrl
    }

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

