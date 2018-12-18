---
title: 了解 Azure IoT 中樞安全性 | Microsoft Docs
description: 開發人員指南 - 如何控制裝置應用程式和後端應用程式存取 IoT 中樞。 包含安全性權杖和 X.509 憑證支援的相關資訊。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: dobett
ms.openlocfilehash: 4e9a5808a718909b21698b551f516a238e3934b0
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45605771"
---
# <a name="control-access-to-iot-hub"></a>控制 IoT 中樞的存取權

本文章說明用來保護 Azure IoT 中樞的選項。 IoT 中樞使用「權限」，授與每個 IoT 中樞端點的存取權。 權限可根據功能限制 IoT 中樞的存取權。

本文將介紹：

* 您可以授與裝置或後端應用程式，以存取您的 IoT 中樞的不同權限。
* 它用來確認權限的驗證程序和權杖。
* 如何設定認證範圍，以限制存取特定資源。
* IoT 中樞支援 X.509 憑證。
* 使用現有的裝置身分識別登錄或驗證結構描述的自訂裝置驗證機制。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

您必須具有適當權限才能存取任何 IoT 中樞端點。 例如，裝置必須包含權杖，其中包含安全性認證，以及傳送到 IoT 中樞的每個訊息。

## <a name="access-control-and-permissions"></a>存取控制及權限

您可以透過下列方式授與[權限](#iot-hub-permissions)：

* **IoT 中樞層級的共用存取原則**。 共用存取原則可以授與上面所列[權限](#iot-hub-permissions)的任意組合。 您可以在 [Azure 入口網站][lnk-management-portal]中定義原則，使用 [IoT 中樞資源 REST API][lnk-resource-provider-apis] 以程式設計方式定義原則，或使用 [az iot hub policy](https://docs.microsoft.com/cli/azure/iot/hub/policy?view=azure-cli-latest) CLI 定義原則。 新建立的 IoT 中樞有下列預設原則︰
  
  | 共用的存取原則 | 權限 |
  | -------------------- | ----------- |
  | iothubowner | 所有權限 |
  | service | **ServiceConnect** 權限 |
  | 裝置 | **DeviceConnect** 權限 |
  | registryRead | **RegistryRead** 權限 |
  | registryReadWrite | **RegistryRead** 和 **RegistryWrite** 權限 |

* **各裝置的安全性認證**。 每個 IoT 中樞都包含[身分識別登錄][lnk-identity-registry]。 對於此身分識別登錄中的每個裝置，您可以設定安全性認證，以對應的裝置端點為範圍來授與 **DeviceConnect** 權限。

例如，在典型的 IoT 解決方案中︰

* 裝置管理元件使用 registryReadWrite  原則。
* 事件處理器元件使用 service  原則。
* 執行階段裝置商務邏輯元件使用 service 原則。
* 個別裝置會使用 IoT 中樞身分識別登錄內儲存的認證進行連接。

> [!NOTE]
> 如需詳細資訊，請參閱[權限](#iot-hub-permissions)。

## <a name="authentication"></a>驗證

Azure IoT 中樞可根據共用存取原則和身分識別登錄安全性認證驗證權杖，以授與端點的存取權。

安全性認證 (例如對稱金鑰) 決不會在網路上傳送。

> [!NOTE]
> 如同 [Azure Resource Manager][lnk-azure-resource-manager] 中的所有提供者一樣，Azure IoT 中樞資源提供者也是透過您的 Azure 訂用帳戶而受保護。

如需如何建構和使用安全性權杖的詳細資訊，請參閱 [IoT 中樞安全性權杖][lnk-sas-tokens]。

### <a name="protocol-specifics"></a>通訊協定詳細規格

每個支援的通訊協定 (例如 MQTT、AMQP 及 HTTPS) 會以不同的方式傳輸權杖。

使用 MQTT 時，CONNECT 封包會有作為 ClientId 的 deviceId、Username 欄位中會有 `{iothubhostname}/{deviceId}`，而 Password 欄位中則會有 SAS 權杖。 `{iothubhostname}` 應該是 IoT 中樞的完整 CName (例如 contoso.azure-devices.net)。

使用 [AMQP][lnk-amqp] 時，IoT 中樞支援 [SASL PLAIN][lnk-sasl-plain] 和 [AMQP 宣告式安全性][lnk-cbs]。

如果使用以 AMQP 宣告為基礎的安全性，標準會指定如何傳輸這些權杖。

在 SASL PLAIN 中， **username** 可以是：

* `{policyName}@sas.root.{iothubName}`，如果使用 IoT 中樞層級權杖。
* `{deviceId}@sas.{iothubname}`，如果使用裝置範圍權杖。

在這兩種情況下，密碼欄位都會包含 [IoT 中樞安全性權杖][lnk-sas-tokens]中所述的權杖。

HTTPS 實作驗證的方式是在 **Authorization** 要求標頭中包含有效的權杖。

#### <a name="example"></a>範例

使用者名稱 (DeviceId 區分大小寫)︰ `iothubname.azure-devices.net/DeviceId`

密碼 (您可以使用 [Device Explorer][lnk-device-explorer] 工具或 CLI 擴充功能命令 [az iot hub generate-sas-token](https://docs.microsoft.com/cli/azure/ext/azure-cli-iot-ext/iot/hub?view=azure-cli-latest#ext-azure-cli-iot-ext-az-iot-hub-generate-sas-token)，或是[適用於 Visual Studio Code 的 Azure IoT Toolkit 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)來產生 SAS 權杖)：

`SharedAccessSignature sr=iothubname.azure-devices.net%2fdevices%2fDeviceId&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501`

> [!NOTE]
> [Azure IoT SDK][lnk-sdks] 會在連接至服務時自動產生權杖。 在某些情況下，Azure IoT SDK 不支援所有的通訊協定或所有驗證方法。

### <a name="special-considerations-for-sasl-plain"></a>SASL PLAIN 的特殊考量

搭配 AMQP 使用 SASL PLAIN 時，連接至 IoT 中樞的用戶端可為每個 TCP 連線使用單一權杖。 當權杖過期時，TCP 連線會中斷服務連線，並觸發重新連線。 此行為雖不會對後端應用程式造成問題，但是對裝置應用程式不利，原因如下︰

* 閘道器通常會代表許多裝置連線。 使用 SASL PLAIN 時，它們必須針對連接至 IoT 中樞的每個裝置不同的建立 TCP 連線。 這個案例會大幅提高電力與網路資源的耗用量，並增加每個裝置連線的延遲。
* 在每個權杖到期後，增加使用要重新連接的資源通常會對資源受限的裝置有不良影響。

## <a name="scope-iot-hub-level-credentials"></a>設定 IoT 中樞層級認證的範圍

使用受限制的資源 URI 建立權杖，可以設定 IoT 中樞層級安全性原則的範圍。 例如，要從裝置傳送「裝置到雲端」訊息的端點是 **/devices/{deviceId}/messages/events**。 您也可以使用 IoT 中樞層級的共用存取原則搭配 **DeviceConnect** 權限，簽署 resourceURI 為 **/devices/{deviceId}** 的權杖。 這個做法會建立僅可用來代表裝置 **deviceId** 傳送訊息的權杖。

這個機制類似於[事件中樞發行者原則][lnk-event-hubs-publisher-policy]，讓您可實作自訂驗證方法。

## <a name="security-tokens"></a>安全性權杖

IoT 中樞使用安全性權杖來驗證裝置和服務，以避免透過線路傳送金鑰。 此外，安全性權杖有時效性和範圍的限制。 [Azure IoT SDK][lnk-sdks] 能在不需要任何特殊組態的情況下自動產生權杖。 在某些案例中，您必須直接產生及使用安全性權杖。 這類案例包括：

* MQTT、AMQP 或 HTTPS 介面的直接使用。
* 權杖服務模式的實作，如[自訂裝置驗證][lnk-custom-auth]中所述。

IoT 中樞也可允許裝置使用 [X.509 憑證][lnk-x509]向 IoT 中樞進行驗證。

### <a name="security-token-structure"></a>安全性權杖結構

透過安全性權杖，您可以將 IoT 中樞內特定功能的限時存取權限授與裝置和服務。 若要取得連接到 IoT 中樞的授權，裝置和服務必須傳送以共用的存取或對稱金鑰簽章的安全性權杖。 這些金鑰會與裝置身分識別一同儲存在身分識別登錄中。

經過共用存取金鑰簽署的權限，能授與所有與共用存取原則權限相關聯之功能的存取權限。 以裝置身分識別對稱金鑰簽署的權杖只會授與相關裝置身分識別的 **DeviceConnect** 權限。

安全性權杖具有下列格式：

`SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

以下是預期的值：

| 值 | 說明 |
| --- | --- |
| {signature} |HMAC-SHA256 簽章字串，格式為： `{URL-encoded-resourceURI} + "\n" + expiry`。 **重要事項**：金鑰是從 base64 解碼而來，並且會做為用來執行 HMAC-SHA256 計算的金鑰。 |
| {resourceURI} |可使用此權杖存取之端點的 URI 前置詞 (依區段)，開頭為 IoT 中樞的主機名稱 (無通訊協定)。 例如， `myHub.azure-devices.net/devices/device1` |
| {expiry} |從新紀元時間 (Epoch) 1970 年 1 月 1日 00:00:00 UTC 時間至今秒數的 UTF8 字串。 |
| {URL-encoded-resourceURI} |小寫資源 URI 的小寫 URL 編碼 |
| {policyName} |此權杖所參考的共用存取原則名稱。 在權杖參考裝置登錄認證的情況下不存在。 |

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

    // Construct autorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
    + base64UriEncoded + "&se=" + expires;
    if (policyName) token += "&skn="+policyName;
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
        'se' : str(int(ttl))
    }

    if policy_name is not None:
        rawtoken['skn'] = policy_name

    return 'SharedAccessSignature ' + urlencode(rawtoken)
```

C# 中用來產生安全性權杖的功能是：

```csharp
using System;
using System.Globalization;
using System.Net;
using System.Net.Http;
using System.Security.Cryptography;
using System.Text;

public static string generateSasToken(string resourceUri, string key, string policyName, int expiryInSeconds = 3600)
{
    TimeSpan fromEpochStart = DateTime.UtcNow - new DateTime(1970, 1, 1);
    string expiry = Convert.ToString((int)fromEpochStart.TotalSeconds + expiryInSeconds);

    string stringToSign = WebUtility.UrlEncode(resourceUri) + "\n" + expiry;

    HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String(key));
    string signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));

    string token = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}", WebUtility.UrlEncode(resourceUri), WebUtility.UrlEncode(signature), expiry);

    if (!String.IsNullOrEmpty(policyName))
    {
        token += "&skn=" + policyName;
    }

    return token;
}

```


> [!NOTE]
> 由於權杖的時效性會在 IoT 中樞機器上驗證，因此產生權杖之機器上的時鐘飄移必須降低最低。

### <a name="use-sas-tokens-in-a-device-app"></a>在裝置應用程式中使用 SAS 權杖

利用安全性權杖取得「IoT 中樞」之 **DeviceConnect** 權限的方法有兩種：使用[身分識別登錄的對稱裝置金鑰](#use-a-symmetric-key-in-the-identity-registry)，或使用[共用存取金鑰](#use-a-shared-access-policy)。

請記住，所有可從裝置存取的功能，在設計上會於前置詞為 `/devices/{deviceId}`的端點公開。

> [!IMPORTANT]
> IoT 中樞驗證特定裝置的唯一方法是使用裝置身分識別對稱金鑰。 在使用共用存取原則來存取裝置功能的案例中，方案必須將發出安全性權杖之元件視為信任的子元件。

面向裝置的端點是 (不論通訊協定為何)︰

| 端點 | 功能 |
| --- | --- |
| `{iot hub host name}/devices/{deviceId}/messages/events` |傳送裝置到雲端的訊息。 |
| `{iot hub host name}/devices/{deviceId}/messages/devicebound` |接收雲端到裝置的訊息。 |

### <a name="use-a-symmetric-key-in-the-identity-registry"></a>使用身分識別登錄中的對稱金鑰

使用裝置身分識別對稱金鑰來產生權杖時，會省略權杖的 policyName (`skn`) 元素。

例如，為存取所有裝置功能而建立的權杖應具有下列參數︰

* 資源 URI： `{IoT hub name}.azure-devices.net/devices/{device id}`、
* 簽署金鑰︰ `{device id}` 身分識別的任何對稱金鑰、
* 無原則名稱、
* 任何到期時間。

使用前述 Node.js 函式的範例為︰

```nodejs
var endpoint ="myhub.azure-devices.net/devices/device1";
var deviceKey ="...";

var token = generateSasToken(endpoint, deviceKey, null, 60);
```

結果 (將所有功能的存取權限授與 device1) 為︰

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697`

> [!NOTE]
> 您可以使用 [Device Explorer][lnk-device-explorer] 工具或 CLI 擴充功能命令 [az iot hub generate-sas-token](https://docs.microsoft.com/cli/azure/ext/azure-cli-iot-ext/iot/hub?view=azure-cli-latest#ext-azure-cli-iot-ext-az-iot-hub-generate-sas-token)，或是[適用於 Visual Studio Code 的 Azure IoT Toolkit 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)來產生 SAS 權杖。

### <a name="use-a-shared-access-policy"></a>使用共用存取原則

當您從共用的存取原則建立權杖時，請將 `skn` 欄位設為原則的名稱。 此原則必須授與 **DeviceConnect** 權限。

使用共用存取原則來存取裝置功能的兩個主要案例包括︰

* [雲端通訊協定閘道][lnk-endpoints]、
* 用來實作自訂驗證配置的[權杖服務][lnk-custom-auth]。

由於共用存取原則可能可以授與以任何裝置身分連線的存取權限，因此在建立安全性權杖時，請務必使用正確的資源 URI。 此設定對權杖服務來說特別重要，因為它們必須使用資源 URI 來設定權杖的範圍，以便納入特定裝置。 這點與通訊協定閘道的關聯性比較薄弱，因為它們已經在為所有裝置調節流量。

舉例來說，權杖服務如果使用名為 **device** 的預先建立共用存取原則，將會使用下列參數來建立權杖︰

* 資源 URI： `{IoT hub name}.azure-devices.net/devices/{device id}`、
* 簽署金鑰︰ `device` 原則的其中一個金鑰、
* 原則名稱： `device`、
* 任何到期時間。

使用前述 Node.js 函式的範例為︰

```nodejs
var endpoint ="myhub.azure-devices.net/devices/device1";
var policyName = 'device';
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

結果 (將所有功能的存取權限授與 device1) 為︰

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697&skn=device`

通訊協定閘道可以針對所有裝置都使用相同權杖，只要將資源 URI 設定為 `myhub.azure-devices.net/devices`即可。

### <a name="use-security-tokens-from-service-components"></a>使用來自服務元件的安全性權杖

服務元件只能使用授與適當權限的共用存取原則來產生安全性權杖，如上所述。

以下是在端點上公開的服務功能︰

| 端點 | 功能 |
| --- | --- |
| `{iot hub host name}/devices` |建立、更新、擷取及刪除裝置身分識別。 |
| `{iot hub host name}/messages/events` |接收裝置到雲端的訊息。 |
| `{iot hub host name}/servicebound/feedback` |接收雲端到裝置之訊息的意見反應。 |
| `{iot hub host name}/devicebound` |傳送雲端到裝置的訊息。 |

舉例來說，服務如果使用名為 **registryRead** 的預先建立共用存取原則，將會使用下列參數來建立權杖︰

* 資源 URI： `{IoT hub name}.azure-devices.net/devices`、
* 簽署金鑰︰ `registryRead` 原則的其中一個金鑰、
* 原則名稱： `registryRead`、
* 任何到期時間。

```nodejs
var endpoint ="myhub.azure-devices.net/devices";
var policyName = 'registryRead';
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

結果 (授與所有裝置身分識別的讀取權限) 會是︰

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=registryRead`

## <a name="supported-x509-certificates"></a>支援的 X.509 憑證

您可以使用任何 X.509 憑證來利用 IoT 中樞驗證裝置，只要將憑證指紋或憑證授權單位 (CA) 上傳至 Azure IoT 中樞即可。 使用憑證指紋來進行驗證時，只會驗證提供的憑證指紋與設定的憑證指紋是否相符。 使用憑證授權單位來進行驗證時，則會驗證憑證鏈結。 

支援的憑證包含：

* **現有的 X.509 憑證**。 裝置可能已經有與其關聯的 X.509 憑證。 裝置可以使用此憑證向「IoT 中樞」進行驗證。 適用於指紋或 CA 驗證。 
* **CA 簽署的 X.509 憑證**。 若要識別裝置並向 IoT 中樞驗證它，您可以使用「憑證授權單位」(CA) 所產生和簽署的 X.509 憑證。 適用於指紋或 CA 驗證。
* **自我產生及自我簽署的 X-509 憑證**。 裝置製造商或公司內部的部署人員可以產生這些憑證，並將對應的私密金鑰 (和憑證) 存放在裝置上。 您可以使用 [OpenSSL][lnk-openssl] 和 [Windows SelfSignedCertificate][lnk-selfsigned] 公用程式之類的工具來達到此目的。 只適用於指紋驗證。 

裝置可以使用 X.509 憑證或安全性權杖來進行驗證，但不可同時使用兩者。

如需使用憑證授權單位進行驗證的詳細資訊，請參閱[使用 X.509 CA 憑證進行裝置驗證](iot-hub-x509ca-overview.md)。

### <a name="register-an-x509-certificate-for-a-device"></a>註冊裝置的 X.509 憑證

[適用於 C# 的 Azure IoT 服務 SDK][lnk-service-sdk] (版本 1.0.8+) 支援註冊使用 X.509 憑證來進行驗證的裝置。 其他 API (例如匯入/匯出裝置) 也支援 X.509 憑證。

您也可以使用 CLI 擴充功能命令 [az iot hub device-identity](https://docs.microsoft.com/cli/azure/ext/azure-cli-iot-ext/iot/hub/device-identity?view=azure-cli-latest) 來設定裝置的 X.509 憑證。

### <a name="c-support"></a>C\# 支援

**RegistryManager** 類別提供一個程式設計方式來註冊裝置。 特別是，**AddDeviceAsync** 和 **UpdateDeviceAsync** 方法可讓您在「IoT 中樞」身分識別登錄中註冊和更新裝置。 這兩種方法都會採用 **Device** 執行個體做為輸入。 **Device** 類別包含 **Authentication** 屬性，這可讓您指定主要和次要 X.509 憑證指紋。 憑證指紋代表 X.509 憑證的 SHA-256 雜湊 (使用二進位 DER 編碼來儲存)。 您可以選擇指定主要指紋或次要指紋，或是同時指定兩者。 支援主要和次要指紋是為了處理憑證變換情況。

以下是使用 X.509 憑證指紋來註冊裝置的範例 C\# 程式碼片段︰

```csharp
var device = new Device(deviceId)
{
  Authentication = new AuthenticationMechanism()
  {
    X509Thumbprint = new X509Thumbprint()
    {
      PrimaryThumbprint = "B4172AB44C28F3B9E117648C6F7294978A00CDCBA34A46A1B8588B3F7D82C4F1"
    }
  }
};
RegistryManager registryManager = RegistryManager.CreateFromConnectionString(deviceGatewayConnectionString);
await registryManager.AddDeviceAsync(device);
```

### <a name="use-an-x509-certificate-during-run-time-operations"></a>在執行階段作業期間使用 X.509 憑證

[適用於 .NET 的 Azure IoT 裝置 SDK][lnk-client-sdk] (版本 1.0.11+) 支援使用 X.509 憑證。

### <a name="c-support"></a>C\# 支援

**DeviceAuthenticationWithX509Certificate** 類別支援使用 X.509 憑證來建立 **DeviceClient** 執行個體。 X.509 憑證必須為 PFX (也稱為 PKCS #12) 格式，其中包含私密金鑰。

以下是範例程式碼片段：

```csharp
var authMethod = new DeviceAuthenticationWithX509Certificate("<device id>", x509Certificate);

var deviceClient = DeviceClient.Create("<IotHub DNS HostName>", authMethod);
```

## <a name="custom-device-and-module-authentication"></a>自訂裝置與模組驗證

您可以使用 IoT 中樞[身分識別登錄][lnk-identity-registry]，利用[權杖][lnk-sas-tokens]來設定依裝置/模組的安全性認證和存取控制。 如果 IoT 解決方案已經有自訂身分識別登錄及/或驗證配置，請考慮建立「權杖服務」，將這個基礎結構與 IoT 中樞整合。 如此一來，您可以在解決方案中使用其他 IoT 功能。

權杖服務是自訂雲端服務。 建立具備 **DeviceConnect** 或 **ModuleConnect** 權限的 IoT 中樞「共用存取原則」，以建立「裝置範圍」或「模組範圍」權杖。 這些權杖可讓裝置與模組連線到 IoT 中樞。

![權杖服務模式的步驟][img-tokenservice]

以下是權杖服務模式的主要步驟：

1. 為您的 IoT 中樞建立具備 **DeviceConnect** 或 **ModuleConnect** 權限的 IoT 中樞共用存取原則。 您可以在 [Azure 入口網站][lnk-management-portal]中或以程式設計方式建立此原則。 權杖服務會使用此原則簽署它所建立的權杖。
1. 當裝置/模組需要存取 IoT 中樞時，它會向您的權杖服務要求已簽署的權杖。 裝置可以使用您的自訂身分識別登錄/驗證配置來進行驗證，以判斷權杖服務用來建立權杖的裝置/模組身分識別。
1. 權杖服務會傳回權杖。 權杖藉由使用 `/devices/{deviceId}` 或 `/devices/{deviceId}/module/{moduleId}` 作為 `resourceURI` 來建立，以 `deviceId` 作為要進行驗證的裝置，或以 `moduleId` 作為要進行驗證的模組。 權杖服務會使用共用存取原則來建構權杖。
1. 裝置/模組直接透過 IoT 中樞使用權杖。

> [!NOTE]
> 您可以使用 .NET 類別 [SharedAccessSignatureBuilder][lnk-dotnet-sas] 或 Java 類別 [IotHubServiceSasToken][lnk-java-sas] 在權杖服務中建立權杖。

權杖服務可以視需要設定權杖到期日。 權杖到期時，IoT 中樞會切斷裝置/模組連線。 然後，裝置/模組必須向權杖服務要求新權杖。 使用過短的到期時間會增加裝置/模組與權杖服務上的負載。

為了讓裝置/模組連線至中樞，即使其使用權杖而不是金鑰來連線，您仍必須將它新增至 IoT 中樞身分識別登錄。 因此，您可以利用在[身分識別登錄][lnk-identity-registry]中啟用或停用裝置/模組身分識別，繼續使用依裝置/依模組存取控制。 此方法可減輕使用較長到期時間權杖的風險。

### <a name="comparison-with-a-custom-gateway"></a>和自訂閘道器的比較

權杖服務模式為使用 IoT 中樞實作自訂身分識別登錄/驗證配置的建議方式。 建議此模式是因為 IoT 中樞會繼續處理大部分的解決方案流量。 不過，如果自訂驗證配置與通訊協定密不可分，您可能需要「自訂閘道」來處理所有流量。 使用[傳輸層安全性 (TLS) 和預先共用金鑰 (PSK)][lnk-tls-psk] 是這類案例的範例之一。 如需詳細資訊，請參閱[通訊協定閘道][lnk-protocols]一文。

## <a name="reference-topics"></a>參考主題：

下列參考主題會提供您關於控制 IoT 中樞存取權的詳細資訊。

## <a name="iot-hub-permissions"></a>IoT 中樞權限

下表列出可用來控制您的 IoT 中樞存取權的權限。

| 權限 | 注意 |
| --- | --- |
| **RegistryRead** |為身分識別登錄授與讀取權限。 如需詳細資訊，請參閱[身分識別登錄][lnk-identity-registry]。 <br/>後端雲端服務會使用此權限。 |
| **RegistryReadWrite** |為身分識別登錄授與讀取和寫入權限。 如需詳細資訊，請參閱[身分識別登錄][lnk-identity-registry]。 <br/>後端雲端服務會使用此權限。 |
| **ServiceConnect** |授與雲端服務面向通訊和監視端點的存取權。 <br/>授與權限以接收裝置到雲端的訊息、傳送雲端到裝置的訊息，以及擷取對應的傳遞通知。 <br/>授與權限以擷取檔案上傳的傳遞認可。 <br/>授與權限以存取對應項，以便更新標籤及所需屬性、擷取報告的屬性，以及執行查詢。 <br/>後端雲端服務會使用此權限。 |
| **DeviceConnect** |授與裝置面向端點的存取權。 <br/>授與權限以傳送裝置到雲端的訊息和接收雲端到裝置的訊息。 <br/>授與權限以從裝置執行檔案上傳。 <br/>授與權限以接收裝置對應項所需的屬性通知，並更新裝置對應項報告的屬性。 <br/>授與權限以執行檔案上傳。 <br/>裝置會使用此權限。 |

## <a name="additional-reference-material"></a>其他參考資料

IoT 中樞開發人員指南中的其他參考主題包括︰

* [IoT 中樞端點][lnk-endpoints]說明每個 IoT 中樞公開給執行階段和管理作業的各種端點。
* [節流和配額][lnk-quotas]描述適用於 IoT 中樞服務的配額和節流行為。
* [Azure IoT 裝置和服務 SDK][lnk-sdks] 列出各種語言 SDK，可供您在開發與「IoT 中樞」互動的裝置和服務應用程式時使用。
* [IoT 中樞查詢語言][lnk-query]描述可用來從 IoT 中樞擷取有關裝置對應項和作業之資訊的查詢語言。
* [IoT 中樞 MQTT 支援][lnk-devguide-mqtt]針對 MQTT 通訊協定提供 IoT 中樞支援的詳細資訊。

## <a name="next-steps"></a>後續步驟

現在您已了解如何控制存取 IoT 中樞，接下來您可能對下列 IoT 中樞開發人員指南主題感興趣︰

* [使用裝置對應項同步處理狀態和組態][lnk-devguide-device-twins]
* [在裝置上叫用直接方法][lnk-devguide-directmethods]
* [排程多個裝置上的作業][lnk-devguide-jobs]

如果您想要嘗試本文章所述的概念，請參閱下列 IoT 中樞教學課程：

* [開始使用 Azure IoT 中樞][lnk-getstarted-tutorial]
* [如何使用 IoT 中樞傳送雲端到裝置訊息][lnk-c2d-tutorial]
* [如何處理 IoT 中樞裝置到雲端訊息][lnk-d2c-tutorial]

<!-- links and images -->

[img-tokenservice]: ./media/iot-hub-devguide-security/tokenservice.png
[lnk-endpoints]: iot-hub-devguide-endpoints.md
[lnk-quotas]: iot-hub-devguide-quotas-throttling.md
[lnk-sdks]: iot-hub-devguide-sdks.md
[lnk-query]: iot-hub-devguide-query-language.md
[lnk-devguide-mqtt]: iot-hub-mqtt-support.md
[lnk-openssl]: https://www.openssl.org/
[lnk-selfsigned]: https://docs.microsoft.com/powershell/module/pkiclient/new-selfsignedcertificate

[lnk-resource-provider-apis]: https://docs.microsoft.com/rest/api/iothub/iothubresource
[lnk-sas-tokens]: iot-hub-devguide-security.md#security-tokens
[lnk-amqp]: https://www.amqp.org/
[lnk-azure-resource-manager]: ../azure-resource-manager/resource-group-overview.md
[lnk-cbs]: https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc
[lnk-event-hubs-publisher-policy]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab
[lnk-management-portal]: https://portal.azure.com
[lnk-sasl-plain]: http://tools.ietf.org/html/rfc4616
[lnk-identity-registry]: iot-hub-devguide-identity-registry.md
[lnk-dotnet-sas]: https://msdn.microsoft.com/library/microsoft.azure.devices.common.security.sharedaccesssignaturebuilder.aspx
[lnk-java-sas]: https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.service.auth._iot_hub_service_sas_token
[lnk-tls-psk]: https://tools.ietf.org/html/rfc4279
[lnk-protocols]: iot-hub-protocol-gateway.md
[lnk-custom-auth]: iot-hub-devguide-security.md#custom-device-and-module-authentication
[lnk-x509]: iot-hub-devguide-security.md#supported-x509-certificates
[lnk-devguide-device-twins]: iot-hub-devguide-device-twins.md
[lnk-devguide-directmethods]: iot-hub-devguide-direct-methods.md
[lnk-devguide-jobs]: iot-hub-devguide-jobs.md
[lnk-service-sdk]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/service
[lnk-client-sdk]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/device
[lnk-device-explorer]: https://github.com/Azure/azure-iot-sdk-csharp/blob/master/tools/DeviceExplorer
[lnk-getstarted-tutorial]: quickstart-send-telemetry-node.md
[lnk-c2d-tutorial]: iot-hub-csharp-csharp-c2d.md
[lnk-d2c-tutorial]: tutorial-routing.md
