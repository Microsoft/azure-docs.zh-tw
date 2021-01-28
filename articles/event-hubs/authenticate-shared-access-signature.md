---
title: 使用共用存取簽章來驗證對 Azure 事件中樞的存取
description: 本文說明如何使用共用存取簽章來驗證事件中樞資源的存取權。
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: ff141fc1bb681e2356a4471dfdc808d622fd76b5
ms.sourcegitcommit: 04297f0706b200af15d6d97bc6fc47788785950f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98986488"
---
# <a name="authenticate-access-to-event-hubs-resources-using-shared-access-signatures-sas"></a>使用 (SAS) 的共用存取簽章來驗證事件中樞資源的存取權
共用存取簽章 (SAS) 可讓您更精確地控制您授與具有共用存取簽章之用戶端的存取類型。 以下是您可以在 SAS 中設定的一些控制項： 

- SAS 有效的間隔，包括開始時間和到期時間。
- SAS 所授與的權限。 例如，「事件中樞」命名空間的 SAS 可能會授與「接聽」許可權，但不授與「傳送」許可權。
- 只有出示有效認證的用戶端才能將資料傳送到事件中樞。
- 用戶端無法模擬另一個用戶端。
- 可封鎖惡意用戶端，讓它無法將資料傳送到事件中樞。

本文涵蓋使用 SAS 驗證事件中樞資源的存取權。 若要瞭解如何使用 SAS 來 **授權** 存取事件中樞資源，請參閱 [這篇文章](authorize-access-shared-access-signature.md)。 

> [!NOTE]
> Microsoft 建議您盡可能使用 Azure AD 認證作為安全性最佳做法，而不是使用共用存取簽章，這可能更容易遭到入侵。 雖然您可以繼續使用共用存取簽章 (SAS) 將更細緻的存取權授與事件中樞資源，Azure AD 提供類似的功能，而不需要管理 SAS 權杖或擔心撤銷遭盜用的 SAS。
> 
> 如需 Azure 事件中樞中的 Azure AD 整合的詳細資訊，請參閱 [使用 Azure AD 授權事件中樞的存取權](authorize-access-azure-active-directory.md)。 


## <a name="configuring-for-sas-authentication"></a>設定 SAS 驗證
您可以在事件中樞命名空間上設定 EventHubs 共用存取授權規則，或在事件中樞) 中設定實體 (事件中樞實例或 Kafka 主題。 目前不支援在取用者群組上設定共用存取授權規則，但您可以使用在命名空間或實體上設定的規則來保護取用者群組的存取。 

下圖顯示如何將授權規則套用至範例實體。 

![設定授權規則](./media/authenticate-shared-access-signature/configure-sas-authorization-rule.png)

在此範例中， (ExampleNamespace) 的事件中樞命名空間範例有兩個實體： eh1 和 topic1。 授權規則是在實體層級以及命名空間層級定義。  

ManageRuleNS、sendRuleNS 和 listenRuleNS 授權規則都適用于事件中樞實例 eh1 和主題 t1。 ListenRule-eh 和 sendRule-eh 授權規則只適用于事件中樞實例 eh1，而 sendRuleT 授權規則只適用于主題 topic1。 

使用 sendRuleNS 授權規則時，用戶端應用程式可以同時傳送至 eh1 和 topic1。 使用 sendRuleT 授權規則時，它只會強制執行 topic1 的細微存取，因此使用此規則來存取的用戶端應用程式現在無法傳送至 eh1，而只會傳送至 topic1。

## <a name="generate-a-shared-access-signature-token"></a>產生共用存取簽章權杖 
任何有權存取授權規則名稱及其簽署金鑰之一的用戶端，都可以產生 SAS 權杖。 權杖是以下列格式編寫字串而產生的：

- `se`  –權杖到期即時。 在權杖到期時，從00:00:00 年1月1970日 epoch UTC 起的整數反映秒 (UNIX epoch) 
- `skn` –授權規則的名稱，也就是 SAS 金鑰名稱。
- `sr` –所存取資源的 URI。
- `sig` 信號.

簽章字串是根據上一節中所述的資源 URI (範圍所計算的 256 SHA-1 雜湊，) 和權杖到期即時的字串表示（以 CRLF 分隔）。

雜湊計算看起來類似下列虛擬程式碼，會傳回 256 位元/32 位元組的雜湊值。 

```
SHA-256('https://<yournamespace>.servicebus.windows.net/'+'\n'+ 1438205742)
```

權杖會包含非雜湊值，因此接收者可以用相同的參數重新計算雜湊，驗證簽發者擁有有效的簽署金鑰。

資源 URI 是宣告其存取權之服務匯流排資源的完整 URI。 例如，HTTP:// <namespace> . servicebus.windows.net/， <entityPath> 也 `sb://<namespace>.servicebus.windows.net/<entityPath>;` 就是 `http://contoso.servicebus.windows.net/eventhubs/eh1` 。

URI 必須是百分比編碼。

用於簽署的共用存取授權規則必須設定於此 URI 或其中一個階層式上層所指定的實體。 例如，先前範例中的 `http://contoso.servicebus.windows.net/eventhubs/eh1` 或 `http://contoso.servicebus.windows.net`。

SAS 權杖對於以 <resourceURI> 簽章字串中使用的所有資源都有效。

> [!NOTE]
> 您可以使用共用存取原則來產生事件中樞的存取權杖。 如需詳細資訊，請參閱 [共用存取授權原則](authorize-access-shared-access-signature.md#shared-access-authorization-policies)。

### <a name="generating-a-signaturetoken-from-a-policy"></a>從原則) 產生簽章 (權杖 
下一節說明如何使用共用存取簽章原則產生 SAS 權杖。

#### <a name="nodejs"></a>NodeJS

```javascript
function createSharedAccessToken(uri, saName, saKey) { 
    if (!uri || !saName || !saKey) { 
            throw "Missing required parameter"; 
        } 
    var encoded = encodeURIComponent(uri); 
    var now = new Date(); 
    var week = 60*60*24*7;
    var ttl = Math.round(now.getTime() / 1000) + week;
    var signature = encoded + '\n' + ttl; 
    var signatureUTF8 = utf8.encode(signature); 
    var hash = crypto.createHmac('sha256', saKey).update(signatureUTF8).digest('base64'); 
    return 'SharedAccessSignature sr=' + encoded + '&sig=' +  
        encodeURIComponent(hash) + '&se=' + ttl + '&skn=' + saName; 
```

#### <a name="java"></a>Java

```java
private static String GetSASToken(String resourceUri, String keyName, String key)
  {
      long epoch = System.currentTimeMillis()/1000L;
      int week = 60*60*24*7;
      String expiry = Long.toString(epoch + week);

      String sasToken = null;
      try {
          String stringToSign = URLEncoder.encode(resourceUri, "UTF-8") + "\n" + expiry;
          String signature = getHMAC256(key, stringToSign);
          sasToken = "SharedAccessSignature sr=" + URLEncoder.encode(resourceUri, "UTF-8") +"&sig=" +
                  URLEncoder.encode(signature, "UTF-8") + "&se=" + expiry + "&skn=" + keyName;
      } catch (UnsupportedEncodingException e) {

          e.printStackTrace();
      }

      return sasToken;
  }


public static String getHMAC256(String key, String input) {
    Mac sha256_HMAC = null;
    String hash = null;
    try {
        sha256_HMAC = Mac.getInstance("HmacSHA256");
        SecretKeySpec secret_key = new SecretKeySpec(key.getBytes(), "HmacSHA256");
        sha256_HMAC.init(secret_key);
        Encoder encoder = Base64.getEncoder();

        hash = new String(encoder.encode(sha256_HMAC.doFinal(input.getBytes("UTF-8"))));

    } catch (InvalidKeyException e) {
        e.printStackTrace();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
   } catch (IllegalStateException e) {
        e.printStackTrace();
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    }

    return hash;
}
```
#### <a name="php"></a>PHP

```php
function generateSasToken($uri, $sasKeyName, $sasKeyValue) 
{ 
    $targetUri = strtolower(rawurlencode(strtolower($uri))); 
    $expires = time();  
    $expiresInMins = 60; 
    $week = 60*60*24*7;
    $expires = $expires + $week; 
    $toSign = $targetUri . "\n" . $expires; 
    $signature = rawurlencode(base64_encode(hash_hmac('sha256',             
     $toSign, $sasKeyValue, TRUE))); 
    
    $token = "SharedAccessSignature sr=" . $targetUri . "&sig=" . $signature . "&se=" . $expires .      "&skn=" . $sasKeyName; 
    return $token; 
}
```

#### <a name="c"></a>C#

```csharp
private static string createToken(string resourceUri, string keyName, string key)
{
    TimeSpan sinceEpoch = DateTime.UtcNow - new DateTime(1970, 1, 1);
    var week = 60 * 60 * 24 * 7;
    var expiry = Convert.ToString((int)sinceEpoch.TotalSeconds + week);
    string stringToSign = HttpUtility.UrlEncode(resourceUri) + "\n" + expiry;
    HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key));
    var signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));
    var sasToken = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}&skn={3}", HttpUtility.UrlEncode(resourceUri), HttpUtility.UrlEncode(signature), expiry, keyName);
    return sasToken;
}
```

## <a name="authenticating-event-hubs-publishers-with-sas"></a>使用 SAS 驗證事件中樞發行者 
事件發行者會定義事件中樞的虛擬端點。 發行者只能用來將訊息傳送至事件中樞，而不會接收訊息。

一般而言，事件中樞會針對每個用戶端採用一個發行者。 系統會將傳送到事件中樞之任何發行者的所有訊息，排入該事件中樞內的佇列中。 發行者可啟用更細緻的存取控制。

系統會為每個「事件中樞」用戶端指派一個唯一權杖，該權杖會上傳到用戶端。 系統會產生權杖，讓每個唯一的權杖將存取權授與不同的唯一發行者。 持有權杖的用戶端只能傳送到一個發行者，而沒有其他發行者。 如果有多個用戶端共用相同的權杖，則每個用戶端都會共用發行者。

所有權杖都是使用 SAS 金鑰來指派。 一般而言，所有權杖都會經過同一個金鑰簽署。 用戶端不會察覺到可防止用戶端製造權杖的金鑰。 用戶端會在相同的權杖到期之前運作。

例如，若要定義範圍僅限於傳送/發佈至事件中樞的授權規則，您必須定義傳送授權規則。 這可以在命名空間層級進行，或為特定實體提供更細微的範圍 (事件中樞實例或主題) 。 具有這類細微存取範圍的用戶端或應用程式稱為「事件中樞發行者」。 若要這樣做，請依照下列步驟執行：

1. 在您想要發佈的實體上建立 SAS 金鑰，以指派其上的 **傳送** 範圍。 如需詳細資訊，請參閱 [共用存取授權原則](authorize-access-shared-access-signature.md#shared-access-authorization-policies)。
2. 使用步驟1中所產生的金鑰，針對特定發行者產生具有到期時間的 SAS 權杖。

    ```csharp
    var sasToken = SharedAccessSignatureTokenProvider.GetPublisherSharedAccessSignature(
                new Uri("Service-Bus-URI"),
                "eventub-name",
                "publisher-name",
                "sas-key-name",
                "sas-key",
                TimeSpan.FromMinutes(30));
    ```
3. 將權杖提供給發行者用戶端，該用戶端只能傳送至實體和權杖授與存取權的發行者。

    一旦權杖到期，用戶端就會失去其存取權，以傳送/發行至實體。 


> [!NOTE]
> 雖然不建議這麼做，但您可以為裝置提供可授與事件中樞或命名空間存取權的權杖。 任何持有此權杖的裝置都可以將訊息直接傳送到該事件中樞。 此外，無法封鎖裝置以傳送至該事件中樞。
> 
> 建議您一律提供特定且細微的範圍。

> [!IMPORTANT]
> 建立權杖之後，系統就會為每個用戶端佈建其自己的唯一權杖。
>
> 當用戶端將資料傳送至事件中樞時，它會以權杖標記其要求。 為了防止攻擊者竊聽及竊取權杖，用戶端與事件中樞之間的通訊必須透過已加密的通道進行。
> 
> 如果權杖遭攻擊者竊取，攻擊者便可以模擬權杖遭竊的用戶端。 封鎖發行者時，會轉譯該用戶端，直到收到使用不同發行者的新權杖為止。


## <a name="authenticating-event-hubs-consumers-with-sas"></a>使用 SAS 驗證事件中樞取用者 
若要驗證從事件中樞產生者所產生之資料取用的後端應用程式，事件中樞權杖驗證要求其用戶端必須擁有指派給其事件中樞命名空間或事件中樞實例或主題的 **管理** 許可權或 **接聽** 許可權。 使用取用者群組從事件中樞取用資料。 雖然 SAS 原則提供您細微的範圍，但此範圍只會在實體層級定義，而不是在取用者層級定義。 這表示在命名空間層級或事件中樞實例或主題層級所定義的許可權，將會套用至該實體的取用者群組。

## <a name="next-steps"></a>後續步驟
查看下列文章：

- [使用 SAS 進行授權](authenticate-shared-access-signature.md)
- [使用 Azure 角色型存取控制 (Azure RBAC) 進行授權 ](authenticate-shared-access-signature.md)
- [深入瞭解事件中樞](event-hubs-about.md)

請參閱下列相關文章：

- [使用 Azure Active Directory 驗證從應用程式 Azure 事件中樞的要求](authenticate-application.md)
- [使用 Azure Active Directory 來驗證受控識別，以存取事件中樞資源](authenticate-managed-identity.md)
- [使用 Azure Active Directory 授權存取事件中樞資源](authorize-access-azure-active-directory.md)
- [使用共用存取簽章授權存取事件中樞資源](authorize-access-shared-access-signature.md)
