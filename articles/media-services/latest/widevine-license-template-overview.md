---
title: Azure 媒體服務與 Widevine 授權範本的概觀 | Microsoft Docs
description: 本主題提供了用來設定 Widevine 授權之 Widevine 授權範本的概觀。
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: bd09205d4bdc60891044728cf0dcafa092ae9846
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49377438"
---
# <a name="widevine-license-template-overview"></a>Widevine 授權範本概觀 

Azure 媒體服務可讓您使用 **Google Widevine** 來加密您的內容。 媒體服務也提供傳遞 Widevine 授權的服務。 您可以使用 Azure 媒體服務 API 來設定 Widevine 授權。 當播放程式嘗試播放以 Widevine 保護的內容時，會將要求傳送到授權傳遞服務來取得授權。 如果授權服務核准要求，服務就會發出授權。 授權會傳送至用戶端，並用來解密及播放指定內容。

Widevine 授權要求會格式化為 JSON 訊息。  

>[!NOTE]
> 您可以建立不包含值的空白訊息，也就是 "{}"。 授權範本會以預設值建立。 大部分案例的預設工作。 以 Microsoft 為基礎的授權傳遞案例應一律使用預設值。 若需要設定 "provider" 與 "content_id" 值，提供者必須符合 Widevine 認證。

    {  
       "payload":"<license challenge>",
       "content_id": "<content id>"
       "provider": "<provider>"
       "allowed_track_types":"<types>",
       "content_key_specs":[  
          {  
             "track_type":"<track type 1>"
          },
          {  
             "track_type":"<track type 2>"
          },
          …
       ],
       "policy_overrides":{  
          "can_play":<can play>,
          "can persist":<can persist>,
          "can_renew":<can renew>,
          "rental_duration_seconds":<rental duration>,
          "playback_duration_seconds":<playback duration>,
          "license_duration_seconds":<license duration>,
          "renewal_recovery_duration_seconds":<renewal recovery duration>,
          "renewal_server_url":"<renewal server url>",
          "renewal_delay_seconds":<renewal delay>,
          "renewal_retry_interval_seconds":<renewal retry interval>,
          "renew_with_usage”:<renew with usage>
       }
    }

## <a name="json-message"></a>JSON 訊息

| 名稱 | 值 | 說明 |
| --- | --- | --- |
| payload |Base64 編碼字串 |用戶端傳送的授權要求。 |
| content_id |Base64 編碼字串 |用來針對每個 content_key_specs.track_type 衍生金鑰識別碼與內容金鑰的識別碼。 |
| provider |字串 |用來查閱內容金鑰和原則。 若將 Microsoft 金鑰傳遞用於 Widevine 授權傳遞，系統會忽略此參數。 |
| policy_name |字串 |先前已登錄原則的名稱。 選用。 |
| allowed_track_types |列舉 |SD_ONLY 或 SD_HD。 控制在授權中包含的內容金鑰。 |
| content_key_specs |JSON 結構的陣列，請參閱＜內容金鑰規格＞一節。  |更細部控制要傳回的內容金鑰。 如需詳細資訊，請參閱＜內容金鑰規格＞一節。 只可以指定 allowed_track_types 和 content_key_specs 值的其中一個。 |
| use_policy_overrides_exclusively |布林值，true 或 false |使用 policy_overrides 所指定的原則屬性，並略過先前儲存的所有原則。 |
| policy_overrides |JSON 結構，請參閱＜原則覆寫＞一節。 |此授權的原則設定。  如果此資產具有預先定義的原則，就會使用這些指定的值。 |
| session_init |JSON 結構，請參閱＜工作階段初始化＞一節。 |選擇性資料會傳遞至授權。 |
| parse_only |布林值，true 或 false |剖析授權要求，但不會發出任何授權。 不過，授權要求中的值會在回應中傳回。 |

## <a name="content-key-specs"></a>內容金鑰規格
如果有預先存在的原則，則不需要在內容金鑰規格中指定任何值。與此內容相關聯的預先存在原則會用來決定輸出保護，例如高頻寬數位內容保護 (HDCP) 和複製衍生管理系統 (CGMS)。 如果預先存在的原則未向 Widevine 授權伺服器登錄，內容提供者可以在授權要求中插入值。   

必須對所有追蹤指定每個 content_key_specs 值，不論 use_policy_overrides_exclusively 選項為何。 

| 名稱 | 值 | 說明 |
| --- | --- | --- |
| content_key_specs track_type |字串 |追蹤類型名稱。 如果授權要求中指定 content_key_specs，則請務必明確指定所有追蹤類型。 未這樣做會導致無法播放過去 10 秒。 |
| content_key_specs  <br/> security_level |uint32 |定義用戶端對於播放的穩健性需求。 <br/> 以軟體為基礎白箱加密是必要的。 <br/> 軟體加密和模糊化的解碼器是必要的。 <br/> 金鑰資料和加密作業必須在支援硬體的受信任執行環境中執行。 <br/> 內容的加密和解密必須在支援硬體的受信任執行環境中執行。  <br/> 加密、解密和媒體 (壓縮和未壓縮) 的所有處理必須在支援硬體的受信任執行環境中處理。 |
| content_key_specs <br/> required_output_protection.hdc |字串，以下項目的其中一個：HDCP_NONE、HDCP_V1、HDCP_V2 |指出是否需要 HDCP。 |
| content_key_specs <br/>索引鍵 |Base64 <br/>編碼的字串 |要用於此追蹤的內容金鑰。如果指定，則需要 track_type 或 key_id。 內容提供者可使用此選項插入此追蹤的內容金鑰，而不是讓 Widevine 授權伺服器產生或查閱金鑰。 |
| content_key_specs.key_id |Base64 編碼的二進位字串，16 位元組 |金鑰的唯一識別碼。 |

## <a name="policy-overrides"></a>原則覆寫
| 名稱 | 值 | 說明 |
| --- | --- | --- |
| policy_overrides&#46;can_play |布林值，true 或 false |表示允許播放內容。 預設值為 false。 |
| policy_overrides&#46;can_persist |布林值，true 或 false |表示可以將授權保存到非揮發性儲存體，供離線使用。 預設值為 false。 |
| policy_overrides&#46;can_renew |布林值，true 或 false |表示允許更新此授權。 如果為 true，則在授權期間可以透過活動訊號延長。 預設值為 false。 |
| policy_overrides&#46;license_duration_seconds |int64 |指出此特定授權的期間。 值為 0 表示期間沒有限制。 預設值為 0。 |
| policy_overrides&#46;rental_duration_seconds |int64 |指出允許播放的期間。 值為 0 表示期間沒有限制。 預設值為 0。 |
| policy_overrides&#46;playback_duration_seconds |int64 |在授權期間內開始播放後的檢視時段。 值為 0 表示期間沒有限制。 預設值為 0。 |
| policy_overrides&#46;renewal_server_url |字串 |此授權的所有活動訊號 (更新) 要求會導向到指定 URL。 只有在 can_renew 為 true 時才會使用這個欄位。 |
| policy_overrides&#46;renewal_delay_seconds |int64 |license_start_time 之後經過幾秒才會第一次嘗試更新。 只有在 can_renew 為 true 時才會使用這個欄位。 預設值為 0。 |
| policy_overrides&#46;renewal_retry_interval_seconds |int64 |指定若發生失敗，後續授權更新要求之間的延遲秒數。 只有在 can_renew 為 true 時才會使用這個欄位。 |
| policy_overrides&#46;renewal_recovery_duration_seconds |int64 |嘗試進行更新時可繼續播放的時段，不過因為授權伺服器發生後端問題而未成功。 值為 0 表示期間沒有限制。 只有在 can_renew 為 true 時才會使用這個欄位。 |
| policy_overrides&#46;renew_with_usage |布林值，true 或 false |表示開始使用時會傳送授權以進行更新。 只有在 can_renew 為 true 時才會使用這個欄位。 |

## <a name="session-initialization"></a>工作階段初始化
| 名稱 | 值 | 說明 |
| --- | --- | --- |
| provider_session_token |Base64 編碼字串 |此工作階段權杖會傳回到授權，並且會在後續的更新作業中存在。 工作階段權杖不會在工作階段之外保存。 |
| provider_client_token |Base64 編碼字串 |要在授權回應中傳回的用戶端權杖。 如果授權要求包含用戶端權杖，則會忽略此值。 用戶端權杖會在授權工作階段之外保存。 |
| override_provider_client_token |布林值，true 或 false |如果為 false 並且授權要求包含用戶端權杖，請使用來自要求的權杖，即使此結構中已指定用戶端權杖亦然。 如果為 true，則一律使用這個結構中指定的權杖。 |

## <a name="configure-your-widevine-license-with-net"></a>使用 .NET 設定您的 Widevine 授權 

媒體服務提供可讓您設定 Widevine 授權的類別。 若要建構授權，請將 JSON 傳遞至 [WidevineTemplate](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.contentkeypolicywidevineconfiguration.widevinetemplate?view=azure-dotnet#Microsoft_Azure_Management_Media_Models_ContentKeyPolicyWidevineConfiguration_WidevineTemplate)。

若要設定範本，您可以：

### <a name="directly-construct-a-json-string"></a>直接建構 JSON 字串

這個方法可能容易發生錯誤。 建議使用其他方法，如[定義所需的類別並序列化為 JSON](#classes) 中所述。

    ```csharp
    ContentKeyPolicyWidevineConfiguration objContentKeyPolicyWidevineConfiguration = new ContentKeyPolicyWidevineConfiguration
    {
        WidevineTemplate = @"{""allowed_track_types"":""SD_HD"",""content_key_specs"":[{""track_type"":""SD"",""security_level"":1,""required_output_protection"":{""hdcp"":""HDCP_V2""}}],""policy_overrides"":{""can_play"":true,""can_persist"":true,""can_renew"":false}}"
    };
    ```

### <a id="classes"></a> 定義所需的類別並序列化為 JSON

#### <a name="define-classes"></a>定義類別

下列範例示範對應至 Widevine JSON 結構描述的類別定義範例。 您可以將類別具現化，然後再將它們序列化為 JSON 字串。  

    ```csharp
    public class PolicyOverrides
    {
        public bool CanPlay { get; set; }
        public bool CanPersist { get; set; }
        public bool CanRenew { get; set; }
        public int RentalDurationSeconds { get; set; }    //Indicates the time window while playback is permitted. A value of 0 indicates that there is no limit to the duration. Default is 0.
        public int PlaybackDurationSeconds { get; set; }  //The viewing window of time after playback starts within the license duration. A value of 0 indicates that there is no limit to the duration. Default is 0.
        public int LicenseDurationSeconds { get; set; }   //Indicates the time window for this specific license. A value of 0 indicates that there is no limit to the duration. Default is 0.
    }

    public class ContentKeySpec
    {
        public string TrackType { get; set; }
        public int SecurityLevel { get; set; }
        public OutputProtection RequiredOutputProtection { get; set; }
    }

    public class OutputProtection
    {
        public string HDCP { get; set; }
    }

    public class WidevineTemplate
    {
        public string AllowedTrackTypes { get; set; }
        public ContentKeySpec[] ContentKeySpecs { get; set; }
        public PolicyOverrides PolicyOverrides { get; set; }
    }
    ```

#### <a name="configure-the-license"></a>設定授權

使用上一節中所定義的類別，建立用來設定 [WidevineTemplate](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models.contentkeypolicywidevineconfiguration.widevinetemplate?view=azure-dotnet#Microsoft_Azure_Management_Media_Models_ContentKeyPolicyWidevineConfiguration_WidevineTemplate) 的 JSON：

```csharp
private static ContentKeyPolicyWidevineConfiguration ConfigureWidevineLicenseTempate()
{
    WidevineTemplate template = new WidevineTemplate()
    {
        AllowedTrackTypes = "SD_HD",
        ContentKeySpecs = new ContentKeySpec[]
        {
            new ContentKeySpec()
            {
                TrackType = "SD",
                SecurityLevel = 1,
                RequiredOutputProtection = new OutputProtection()
                {
                    HDCP = "HDCP_V2"
                }
            }
        },
        PolicyOverrides = new PolicyOverrides()
        {
            CanPlay = true,
            CanPersist = true,
            CanRenew = false,
            RentalDurationSeconds = 2592000,
            PlaybackDurationSeconds = 10800,
            LicenseDurationSeconds = 604800,
        }
    };

    ContentKeyPolicyWidevineConfiguration objContentKeyPolicyWidevineConfiguration = new ContentKeyPolicyWidevineConfiguration
    {
        WidevineTemplate = Newtonsoft.Json.JsonConvert.SerializeObject(template)
    };
    return objContentKeyPolicyWidevineConfiguration;
}
```

## <a name="next-steps"></a>後續步驟

查看如何[使用 DRM 保護](protect-with-drm.md)