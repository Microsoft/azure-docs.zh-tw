---
title: IoT 裝置佈建服務中的安全性端點 | Microsoft Docs
description: 概念 - 如何控制後端應用程式之 IoT 裝置佈建服務的存取權。 包含安全性權杖的相關資訊。
author: wesmc7777
manager: timlt
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 09/28/2017
ms.author: wesmc
ms.openlocfilehash: e476ca498e4dc1b36d18927beddc812d6d803120
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818505"
---
# <a name="control-access-to-azure-iot-hub-device-provisioning-service"></a>控制 Azure IoT 中樞裝置佈建服務的存取權

本文說明用來保護 IoT 裝置佈建服務的選項。 佈建服務使用權限，授與每個端點的存取權。 權限可根據功能限制服務執行個體的存取權。

本文章說明：

* 您可以授與後端應用程式，以存取您的佈建服務的不同權限。
* 它用來確認權限的驗證程序和權杖。

### <a name="when-to-use"></a>使用時機

您必須具有適當權限才能存取任何佈建服務端點。 例如，後端應用程式必須包含權杖，其中包含安全性認證，以及傳送到服務的每個訊息。

## <a name="access-control-and-permissions"></a>存取控制及權限

您可以透過下列方式授與[權限](#device-provisioning-service-permissions)：

* **共用存取授權原則**。 共用存取原則可以授與上面所列[權限](#device-provisioning-service-permissions)的任意組合。 您可以在 [Azure 入口網站][lnk-management-portal]中定義原則，或使用 [裝置佈建服務 REST API][lnk-resource-provider-apis] 以程式設計方式定義原則。 新建立的佈建服務有下列預設原則︰

* **provisioningserviceowner**︰具備所有權限的原則。

> [!NOTE]
> 如需詳細資訊，請參閱[權限](#device-provisioning-service-permissions)。

## <a name="authentication"></a>驗證

Azure IoT 中樞裝置佈建服務可根據共用存取原則驗證權杖，以授與端點的存取權。 安全性認證 (例如對稱金鑰) 決不會在網路上傳送。

> [!NOTE]
> 如同 [Azure Resource Manager][lnk-azure-resource-manager] 中的所有提供者一樣，裝置佈建服務資源提供者也是透過您的 Azure 訂用帳戶而受保護。

如需如何建構和使用安全性權杖的詳細資訊，請參閱下一節。

HTTP 是唯一支援的通訊協定，是以在 **Authorization** 要求標頭中包含有效權杖的方式實作驗證。

#### <a name="example"></a>範例
```csharp
SharedAccessSignature sr = 
   mydps.azure-devices-provisioning.net&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501&skn=provisioningserviceowner`\
```

> [!NOTE]
> [Azure IoT 裝置佈建服務 SDK][lnk-sdks] 會在連接至服務時自動產生權杖。

## <a name="security-tokens"></a>安全性權杖

裝置佈建服務使用安全性權杖來驗證服務，以避免透過線路傳送金鑰。 此外，安全性權杖有時效性和範圍的限制。 [Azure IoT 裝置佈建服務 SDK][lnk-sdks] 能在不需要任何特殊組態的情況下，自動產生權杖。 在某些案例中，您必須直接產生及使用安全性權杖。 這類案例包括直接使用 HTTP 介面。

### <a name="security-token-structure"></a>安全性權杖結構

透過安全性權杖，您可以將 IoT 裝置佈建服務內特定功能的限時存取權限授與服務。 若要取得連接到佈建服務的授權，服務必須傳送以共用的存取或對稱金鑰簽章的安全性權杖。

經過共用存取金鑰簽署的權限，能授與所有與共用存取原則權限相關聯之功能的存取權限。 

安全性權杖具有下列格式：

`SharedAccessSignature sig={signature}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

以下是預期的值：

| 值 | 說明 |
| --- | --- |
| {signature} |HMAC-SHA256 簽章字串，格式為： `{URL-encoded-resourceURI} + "\n" + expiry`。 **重要事項**：金鑰是從 base64 解碼而來，並且會做為用來執行 HMAC-SHA256 計算的金鑰。|
| {expiry} |從新紀元時間 (Epoch) 1970 年 1 月 1日 00:00:00 UTC 時間至今秒數的 UTF8 字串。 |
| {URL-encoded-resourceURI} | 小寫資源 URI 的小寫 URL 編碼。 可使用此權杖存取之端點的 URI 前置詞 (依區段)，開頭為 IoT 裝置佈建服務的主機名稱 (無通訊協定)。 例如： `mydps.azure-devices-provisioning.net`。 |
| {policyName} |此權杖所參考的共用存取原則名稱。 |

**前置詞的注意事項**︰URI 前置詞是依區段 (而不是依字元) 計算。 例如，`/a/b` 是 `/a/b/c` 的前置詞，而不是 `/a/bc` 的前置詞。

下列 Node.js 程式碼片段顯示稱為 **generateSasToken** 的函式，它會從輸入 `resourceUri, signingKey, policyName, expiresInMins` 計算權杖。 下一節將詳細說明如何初始化不同權杖使用案例的不同輸入。

```nodejs
var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
    resourceUri = encodeURIComponent(resourceUri);

    // Set expiration in seconds
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    expires = Math.ceil(expires);
    var toSign = resourceUri + '\n' + expires;

    // Use crypto
    var hmac = crypto.createHmac('sha256', new Buffer(signingKey, 'base64'));
    hmac.update(toSign);
    var base64UriEncoded = encodeURIComponent(hmac.digest('base64'));

    // Construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
    + base64UriEncoded + "&se=" + expires + "&skn="+ policyName;
    return token;
};
```

做為比較，用來產生安全性權杖的對等 Python 程式碼是︰

```python
from base64 import b64encode, b64decode
from hashlib import sha256
from time import time
from urllib import quote_plus, urlencode
from hmac import HMAC

def generate_sas_token(uri, key, policy_name, expiry=3600):
    ttl = time() + expiry
    sign_key = "%s\n%d" % ((quote_plus(uri)), int(ttl))
    print sign_key
    signature = b64encode(HMAC(b64decode(key), sign_key, sha256).digest())

    rawtoken = {
        'sr' :  uri,
        'sig': signature,
        'se' : str(int(ttl)),
        'skn' : policy_name
    }

    return 'SharedAccessSignature ' + urlencode(rawtoken)
```

> [!NOTE]
> 由於權杖的時效性會在 IoT 裝置佈建服務機器上驗證，因此產生權杖之機器上的時鐘飄移必須降低最低。

### <a name="use-security-tokens-from-service-components"></a>使用來自服務元件的安全性權杖

服務元件只能使用授與適當權限的共用存取原則來產生安全性權杖，如上所述。

以下是在端點上公開的服務功能︰

| 端點 | 功能 |
| --- | --- |
| `{your-service}.azure-devices-provisioning.net/enrollments` |使用裝置佈建服務提供裝置註冊作業。 |
| `{your-service}.azure-devices-provisioning.net/enrollmentGroups` |提供用於管理裝置註冊群組的作業。 |
| `{your-service}.azure-devices-provisioning.net/registrations/{id}` |提供用於擷取及管理裝置註冊狀態的作業。 |


舉例來說，使用名為 **enrollmentread** 的預先建立共用存取原則產生的服務，將會使用下列參數來建立權杖︰

* 資源 URI： `{mydps}.azure-devices-provisioning.net`、
* 簽署金鑰︰ `enrollmentread` 原則的其中一個金鑰、
* 原則名稱： `enrollmentread`、
* 任何到期 time.backn

![在入口網站中建立裝置佈建服務執行個體的共用存取原則][img-add-shared-access-policy]

```nodejs
var endpoint ="mydps.azure-devices-provisioning.net";
var policyName = 'enrollmentread'; 
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

結果 (授與所有註冊記錄的讀取權限) 會是︰

`SharedAccessSignature sr=mydps.azure-devices-provisioning.net&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=enrollmentread`

## <a name="reference-topics"></a>參考主題：

下列參考主題會提供您有關控制 IoT 裝置佈建服務存取權的詳細資訊。

### <a name="device-provisioning-service-permissions"></a>裝置佈建服務權限

下表列出可用來控制您的 IoT 裝置佈建服務存取權的權限。

| 權限 | 注意 |
| --- | --- |
| **ServiceConfig** |授與變更服務設定的存取權。 <br/>後端雲端服務會使用此權限。 |
| **EnrollmentRead** |授與對裝置註冊和註冊群組進行讀取的權限。 <br/>後端雲端服務會使用此權限。 |
| **EnrollmentWrite** |授與對裝置註冊和註冊群組進行寫入的權限。 <br/>後端雲端服務會使用此權限。 |
| **RegistrationStatusRead** |授與對裝置註冊狀態進行讀取的權限。 <br/>後端雲端服務會使用此權限。 |
| **RegistrationStatusWrite**  |授與對裝置註冊狀態進行刪除的權限。 <br/>後端雲端服務會使用此權限。 |

<!-- links and images -->

[img-add-shared-access-policy]: ./media/how-to-control-access/how-to-add-shared-access-policy.PNG
[lnk-sdks]: ../iot-hub/iot-hub-devguide-sdks.md
[lnk-management-portal]: https://portal.azure.com
[lnk-azure-resource-manager]: ../azure-resource-manager/resource-group-overview.md
[lnk-resource-provider-apis]: https://docs.microsoft.com/rest/api/iot-dps/
