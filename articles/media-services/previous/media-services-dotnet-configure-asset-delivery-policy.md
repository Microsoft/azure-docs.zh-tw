---
title: 使用 .NET SDK 設定資產傳遞原則 | Microsoft Docs
description: 本主題說明如何使用 Azure 媒體服務 .NET SDK 設定不同的資產傳遞原則。
services: media-services
documentationcenter: ''
author: Mingfeiy
manager: cfowler
editor: ''
ms.assetid: 3ec46f58-6cbb-4d49-bac6-1fd01a5a456b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/05/2018
ms.author: juliako
ms.openlocfilehash: aee2477e0633974cba42ab26e102323cb9606810
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33784547"
---
# <a name="configure-asset-delivery-policies-with-net-sdk"></a>使用 .NET SDK 設定資產傳遞原則
[!INCLUDE [media-services-selector-asset-delivery-policy](../../../includes/media-services-selector-asset-delivery-policy.md)]

## <a name="overview"></a>概觀
如果您打算傳遞加密的資產，媒體服務內容傳遞工作流程的其中一個步驟，是設定資產的傳遞原則。 資產傳遞原則會告訴媒體服務您想要如何傳遞資產：您的資產應該動態封裝成哪個串流通訊協定 (如 MPEG DASH、HLS、Smooth Streaming 或所有)，您是否想要動態加密您的資產及其方式 (信封或一般加密)。

本文討論建立和設定資產傳遞原則的原因與方法。

>[!NOTE]
>建立 AMS 帳戶時，會將**預設**串流端點新增至處於 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。 
>
>此外，為了能夠使用動態封裝和動態加密功能，您的資產必須包含一組調適性位元速率 MP4 或調適性位元速率 Smooth Streaming 檔案。

您可以將不同的原則套用至相同的資產。 例如，您可以將 PlayReady 加密套用到 Smooth Streaming，將 AES 信封加密套用到 MPEG DASH 和 HLS。 傳遞原則中未定義的任何通訊協定 (例如，您加入單一原則，它只有指定 HLS 做為通訊協定) 將會遭到封鎖無法串流。 例外情形是當您完全未定義資產傳遞原則時。 那麼，將允許所有通訊協定，不受阻礙。

如果您想要傳遞儲存體加密資產，就必須設定資產的傳遞原則。 資產可以串流處理之前，串流伺服器會移除儲存體加密，並使用指定的傳遞原則來串流您的內容。 例如，若要傳遞使用進階加密標準 (AES) 信封加密金鑰加密的資產，請將原則類型設定為 **DynamicEnvelopeEncryption**。 如果您要移除儲存體加密，並且不受阻礙地串流資產，請將原則類型設定為 **NoDynamicEncryption**。 下列範例示範如何設定這些原則類型。

視您如何設定資產傳遞原則而定，您可以動態封裝、加密，以及串流下列串流通訊協定：Smooth Streaming、HLS 和 MPEG DASH。

下列清單顯示您用來串流 Smooth、HLS 和 DASH 的格式。

Smooth Streaming：

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest

HLS：

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=m3u8-aapl)

MPEG DASH

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=mpd-time-csf)

## <a name="considerations"></a>考量
* 刪除 AssetDeliveryPolicy 之前，您應該刪除所有與資產相關聯的串流定位器。 如有需要，您稍後可以使用新的 AssetDeliveryPolicy 來建立新的串流定位器。
* 未設定資產傳遞原則時，將無法於儲存空間已加密的資產建立串流定位器。  如果資產的儲存空間未加密，系統會讓您建立定位器，並直接串流資產而不使用資產傳遞原則。
* 您可以有多個資產傳遞原則與一個資產關聯，但只能指定一種方法處理特定的 AssetDeliveryProtocol。  這表示如果您嘗試連結二個指定 AssetDeliveryProtocol.SmoothStreaming 通訊協定的傳遞原則時將會導致錯誤，因為當用戶端發出 Smooth Streaming 要求時，系統會不知道要套用哪一個原則。
* 如果您有包含現有串流定位器的資產，則您無法將新原則連接到該資產 (您可以解除現有原則與資產的連結，或更新與資產關聯的傳遞原則)。  您必須先移除串流定位器，調整原則，然後重新建立串流定位器。  重新建立串流定位器時，您可以使用同一個 locatorId，但必須確定不會對用戶端造成問題，因為原始或下游 CDN 可能會快取內容。

## <a name="clear-asset-delivery-policy"></a>清除資產傳遞原則

下列 **ConfigureClearAssetDeliveryPolicy** 方法指定不套用動態加密，以及使用下列任何通訊協定來傳遞資料流：MPEG DASH、HLS 和 Smooth Streaming 通訊協定。 您可能想要將此原則套用到您的儲存體加密資產。

如需建立 AssetDeliveryPolicy 時可以指定之值的相關資訊，請參閱[定義 AssetDeliveryPolicy 時使用的類型](#types) 一節。

```csharp
    static public void ConfigureClearAssetDeliveryPolicy(IAsset asset)
    {
        IAssetDeliveryPolicy policy =
        _context.AssetDeliveryPolicies.Create("Clear Policy",
        AssetDeliveryPolicyType.NoDynamicEncryption,
        AssetDeliveryProtocol.HLS | AssetDeliveryProtocol.SmoothStreaming | AssetDeliveryProtocol.Dash, null);
        
        asset.DeliveryPolicies.Add(policy);
    }
```
## <a name="dynamiccommonencryption-asset-delivery-policy"></a>DynamicCommonEncryption 資產傳遞原則

下列 **CreateAssetDeliveryPolicy** 方法會建立 **AssetDeliveryPolicy**，而後者設定成將動態一般加密 (**DynamicCommonEncryption**) 套用至 Smooth Streaming 通訊協定 (將會封鎖其他通訊協定進行串流處理)。 此方法會採用兩個參數：**Asset** (您要套用傳遞原則的資產) 和 **IContentKey** (**CommonEncryption** 類型的內容金鑰，如需詳細資訊，請參閱：[建立內容金鑰](media-services-dotnet-create-contentkey.md#common_contentkey))。

如需建立 AssetDeliveryPolicy 時可以指定之值的相關資訊，請參閱[定義 AssetDeliveryPolicy 時使用的類型](#types) 一節。

```csharp
    static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
    {
        Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);
        
        Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
                new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
            {
                {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()},
            };
    
            var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                    "AssetDeliveryPolicy",
                AssetDeliveryPolicyType.DynamicCommonEncryption,
                AssetDeliveryProtocol.SmoothStreaming,
                assetDeliveryPolicyConfiguration);
    
            // Add AssetDelivery Policy to the asset
            asset.DeliveryPolicies.Add(assetDeliveryPolicy);
    
            Console.WriteLine();
            Console.WriteLine("Adding Asset Delivery Policy: " +
                assetDeliveryPolicy.AssetDeliveryPolicyType);
     }
```

Azure 媒體服務也可讓您加入 Widevine 加密。 下列範例會示範 PlayReady 和 Widevine 如何新增到資產傳遞原則。

```csharp
    static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
    {
        // Get the PlayReady license service URL.
        Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);


        // GetKeyDeliveryUrl for Widevine attaches the KID to the URL.
        // For example: https://amsaccount1.keydelivery.mediaservices.windows.net/Widevine/?KID=268a6dcb-18c8-4648-8c95-f46429e4927c.  
        // The WidevineBaseLicenseAcquisitionUrl (used below) also tells Dynamaic Encryption 
        // to append /? KID =< keyId > to the end of the url when creating the manifest.
        // As a result Widevine license acquisition URL will have KID appended twice, 
        // so we need to remove the KID that in the URL when we call GetKeyDeliveryUrl.

        Uri widevineUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.Widevine);
        UriBuilder uriBuilder = new UriBuilder(widevineUrl);
        uriBuilder.Query = String.Empty;
        widevineUrl = uriBuilder.Uri;

        Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
        {
            {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()},
            {AssetDeliveryPolicyConfigurationKey.WidevineLicenseAcquisitionUrl, widevineUrl.ToString()}

        };

        var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryption,
            AssetDeliveryProtocol.Dash,
            assetDeliveryPolicyConfiguration);


        // Add AssetDelivery Policy to the asset
        asset.DeliveryPolicies.Add(assetDeliveryPolicy);

    }
```
> [!NOTE]
> 以 Widevine 加密時，就只能使用 DASH 來傳遞。 請務必在資產傳遞通訊協定中指定 DASH。
> 
> 

## <a name="dynamicenvelopeencryption-asset-delivery-policy"></a>DynamicEnvelopeEncryption 資產傳遞原則
下列 **CreateAssetDeliveryPolicy** 方法會建立 **AssetDeliveryPolicy**，而後者設定成將動態信封加密 (**DynamicEnvelopeEncryption**) 套用至 Smooth Streaming、HLS 及 DASH 通訊協定 (若您決定不指定某些通訊協定，系統將會封鎖它們進行串流處理)。 此方法會採用兩個參數：**Asset** (您要套用傳遞原則的資產) 和 **IContentKey** (**EnvelopeEncryption** 類型的內容金鑰，如需詳細資訊，請參閱：[建立內容金鑰](media-services-dotnet-create-contentkey.md#envelope_contentkey))。

如需建立 AssetDeliveryPolicy 時可以指定之值的相關資訊，請參閱[定義 AssetDeliveryPolicy 時使用的類型](#types) 一節。   

```csharp
    private static void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
    {

        //  Get the Key Delivery Base Url by removing the Query parameter.  The Dynamic Encryption service will
        //  automatically add the correct key identifier to the url when it generates the Envelope encrypted content
        //  manifest.  Omitting the IV will also cause the Dynamice Encryption service to generate a deterministic
        //  IV for the content automatically.  By using the EnvelopeBaseKeyAcquisitionUrl and omitting the IV, this
        //  allows the AssetDelivery policy to be reused by more than one asset.
        //
        Uri keyAcquisitionUri = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.BaselineHttp);
        UriBuilder uriBuilder = new UriBuilder(keyAcquisitionUri);
        uriBuilder.Query = String.Empty;
        keyAcquisitionUri = uriBuilder.Uri;

        // The following policy configuration specifies: 
        //   key url that will have KID=<Guid> appended to the envelope and
        //   the Initialization Vector (IV) to use for the envelope encryption.
        Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string> 
        {
            {AssetDeliveryPolicyConfigurationKey.EnvelopeBaseKeyAcquisitionUrl, keyAcquisitionUri.ToString()},
        };

        IAssetDeliveryPolicy assetDeliveryPolicy =
            _context.AssetDeliveryPolicies.Create(
                        "AssetDeliveryPolicy",
                        AssetDeliveryPolicyType.DynamicEnvelopeEncryption,
                        AssetDeliveryProtocol.SmoothStreaming | AssetDeliveryProtocol.HLS | AssetDeliveryProtocol.Dash,
                        assetDeliveryPolicyConfiguration);

        // Add AssetDelivery Policy to the asset
        asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        Console.WriteLine();
        Console.WriteLine("Adding Asset Delivery Policy: " + assetDeliveryPolicy.AssetDeliveryPolicyType);
    }
```

## <a id="types"></a>定義 AssetDeliveryPolicy 時使用的類型

### <a id="AssetDeliveryProtocol"></a>AssetDeliveryProtocol

下列列舉描述您可以針對資產傳遞通訊協定設定的值。

```csharp
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
```
### <a id="AssetDeliveryPolicyType"></a>AssetDeliveryPolicyType

下列列舉描述您可以針對資產傳遞原則類型設定的值。  
```csharp
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
```
### <a id="ContentKeyDeliveryType"></a>ContentKeyDeliveryType

下列列舉描述您可用來設定將內容金鑰傳遞給用戶端之方法的值。
  ```csharp  
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
```
### <a id="AssetDeliveryPolicyConfigurationKey"></a>AssetDeliveryPolicyConfigurationKey

下列列舉描述您可以設定的值，以便設定用來取得資產傳遞原則特定設定的金鑰。
```csharp
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
```
## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

