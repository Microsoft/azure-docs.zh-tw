---
title: 使用 AMS REST API 以儲存體加密來加密您的內容
description: 了解如何使用 AMS REST API 以儲存體加密來加密您的內容。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: a0a79f3d-76a1-4994-9202-59b91a2230e0
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/10/2017
ms.author: juliako
ms.openlocfilehash: 2fd4c91a8151067c0e9cc9000c158e48cb2cd8a5
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785997"
---
# <a name="encrypting-your-content-with-storage-encryption"></a>以儲存體加密來加密您的內容

高度建議使用 AES-256 位元加密對您的內容進行本機加密，然後將其上傳到將靜止加密儲存的 Azure 儲存體。

本文概述 AMS 儲存體加密，並說明如何上傳儲存體加密內容︰

* 建立內容金鑰。
* 建立資產。 建立資產時，請將 AssetCreationOption 設為 StorageEncryption。
  
     加密的資產會與內容金鑰相關聯。
* 將內容金鑰連結到資產。  
* 在 AssetFile 實體上設定加密相關的參數。

## <a name="considerations"></a>考量 

如果您想要傳遞儲存體加密資產，就必須設定資產的傳遞原則。 資產可以串流處理之前，串流伺服器會移除儲存體加密，並使用指定的傳遞原則來串流您的內容。 如需詳細資訊，請參閱 [設定資產傳遞原則](media-services-rest-configure-asset-delivery-policy.md)。

在媒體服務中存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。 如需詳細資訊，請參閱 [媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。 

## <a name="connect-to-media-services"></a>連線到媒體服務

如需連線至 AMS API 的詳細資訊，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 

## <a name="storage-encryption-overview"></a>儲存體加密概觀
AMS 儲存體加密會將 **AES-CTR** 模式加密套用至整個檔案。  AES CTR 模式是可加密任意長度資料且不需填補的區塊編碼器。 其作業方式是使用 AES 演算法加密計數器區塊，並且對要加密或解密資料的 AES 輸出進行 XOR 處理。  所使用計數器區塊的建構方式是將 InitializationVector 的值複製到計數器值的位元組 0 到 7，而且計數器值的位元組 8 到 15 設定為零。 在 16 位元組的計數器區塊中，位元組 8 到 15 (即最低位元組) 是用作簡單 64 位元不帶正負號的整數，而且每個後續處理的資料區塊都會加一，並會保持網路位元組順序。 如果此整數達到最大值 (0xFFFFFFFFFFFFFFFF)，則增加它時會將區塊計數器重設為零 (位元組 8 到 15)，而不影響計數器的其他 64 位元 (即位元組 0 到 7)。   為了維護 AES CTR 模式加密的安全性，對每個檔案而言，每個內容金鑰的指定金鑰識別碼應該為唯一，而且檔案的長度應該小於 2^64 個區塊。  此唯一值是為了確保計數器值絕不會與指定的索引鍵重複使用。 如需 CTR 模式的詳細資訊，請參閱 [此 Wiki 頁面](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CTR) (Wiki 文章使用 "Nonce" 這個字，而非 "InitializationVector")。

使用 **儲存體加密** ，使用 AES-256 位元加密對您的純文字內容進行本機加密，然後將其上傳到已靜止加密儲存的 Azure 儲存體。 使用儲存體加密保護的資產會在編碼之前自動解除加密並放在加密的檔案系統中，並且選擇性地在上傳為新的輸出資產之前重新加密。 儲存體加密的主要使用案例是，讓您可以使用強式加密來保護磁碟中靜止的高品質輸入媒體檔。

若要傳遞儲存體加密資產，您必須設定資產的傳遞原則，讓媒體服務知道您的內容傳遞方式。 串流處理資產之前，串流伺服器會移除儲存體加密，並使用指定的傳遞原則來串流處理您的內容 (例如，AES、一般加密或不加密)。

## <a name="create-contentkeys-used-for-encryption"></a>建立要用於加密的 ContentKey
加密的資產必須與儲存體加密金鑰相關聯。 先建立要用於加密的內容金鑰，再建立資產檔案。 本節說明如何建立內容金鑰。

以下是產生要與要加密資產相關聯之內容金鑰的一般步驟。 

1. 對於儲存體加密，請隨機產生 32 個位元組的 AES 金鑰。 
   
    32 個位元組的 AES 金鑰是您資產的內容金鑰，這表示與該資產相關聯的所有檔案都必須在解密期間使用相同的內容金鑰。 
2. 呼叫 [GetProtectionKeyId](https://docs.microsoft.com/rest/api/media/operations/rest-api-functions#getprotectionkeyid) 和 [GetProtectionKey](https://msdn.microsoft.com/library/azure/jj683097.aspx#getprotectionkey) 方法，以取得用來將內容金鑰加密時必須使用的正確 X.509 憑證。
3. 使用 X.509 憑證的公開金鑰將您的內容金鑰加密。 
   
   媒體服務 .NET SDK 會使用 RSA 和 OAEP 來執行加密作業。  您可以在 [EncryptSymmetricKeyData 函式](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs)中查看 .NET 範例。
4. 建立使用金鑰識別碼和內容金鑰計算的總和檢查碼值。 下列 .NET 範例會使用金鑰識別碼和明文內容金鑰的 GUID 部分計算總和檢查碼。

    ```csharp
            public static string CalculateChecksum(byte[] contentKey, Guid keyId)
            {
                const int ChecksumLength = 8;
                const int KeyIdLength = 16;

                byte[] encryptedKeyId = null;

                // Checksum is computed by AES-ECB encrypting the KID
                // with the content key.
                using (AesCryptoServiceProvider rijndael = new AesCryptoServiceProvider())
                {
                    rijndael.Mode = CipherMode.ECB;
                    rijndael.Key = contentKey;
                    rijndael.Padding = PaddingMode.None;

                    ICryptoTransform encryptor = rijndael.CreateEncryptor();
                    encryptedKeyId = new byte[KeyIdLength];
                    encryptor.TransformBlock(keyId.ToByteArray(), 0, KeyIdLength, encryptedKeyId, 0);
                }

                byte[] retVal = new byte[ChecksumLength];
                Array.Copy(encryptedKeyId, retVal, ChecksumLength);

                return Convert.ToBase64String(retVal);
            }
    ```

5. 用您在先前步驟中收到的 **EncryptedContentKey** (轉換為 base64 編碼的字串)、**ProtectionKeyId**、**ProtectionKeyType**、**ContentKeyType** 和 **Checksum** 值建立內容金鑰。

    對於儲存體加密，要求本文中應該包含下列屬性。

    要求本文屬性    | 說明
    ---|---
    id | 使用下列格式產生 ContentKey 識別碼："nb:kid:UUID:<NEW GUID>"。
    ContentKeyType | 內容金鑰類型是可定義索引鍵的整數。 若為儲存體加密格式，此值為 1。
    EncryptedContentKey | 我們會建立新的內容金鑰值，其為 256 位元 (32 位元組) 的值。 此金鑰是藉由針對 GetProtectionKeyId 與 GetProtectionKey 方法執行 HTTP GET 要求，使用我們從 Microsoft Azure 媒體服務擷取的儲存體加密 X.509 憑證來加密的。 範例請參閱下列 .NET 程式碼︰[這裡](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs)定義的 **EncryptSymmetricKeyData**方法。
    ProtectionKeyId | 這是適用於儲存體加密 X.509 憑證的保護金鑰識別碼，可用來加密我們的內容金鑰。
    ProtectionKeyType | 這是適用於保護金鑰的加密類型，可用來將內容金鑰加密。 針對本文範例，此值為 StorageEncryption(1)。
    Checksum |MD5 會針對內容金鑰計算出總和檢查碼。 它是使用內容金鑰來將內容識別碼加密計算而得的。 範例程式碼示範如何計算總和檢查碼。


### <a name="retrieve-the-protectionkeyid"></a>擷取 ProtectionKeyId
下列範例示範如何為加密內容金鑰時您必須使用的憑證擷取 ProtectionKeyId (憑證指紋)。 執行此步驟，以確定您的電腦上已經有適當的憑證。

要求：

    GET https://media.windows.net/api/GetProtectionKeyId?contentKeyType=0 HTTP/1.1
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    User-Agent: Microsoft ADO.NET Data Services
    Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423034908&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=7eSLe1GHnxgilr3F2FPCGxdL2%2bwy%2f39XhMPGY9IizfU%3d
    x-ms-version: 2.17
    Host: media.windows.net

回應：

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Length: 139
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Server: Microsoft-IIS/8.5
    request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
    x-ms-request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Wed, 04 Feb 2015 02:42:52 GMT

    {"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String","value":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C"}

### <a name="retrieve-the-protectionkey-for-the-protectionkeyid"></a>擷取 ProtectionKeyId 的 ProtectionKey
下列範例顯示如何使用您在上一個步驟中收到的 ProtectionKeyId 擷取 X.509 憑證。

要求：

    GET https://media.windows.net/api/GetProtectionKey?ProtectionKeyId='7D9BB04D9D0A4A24800CADBFEF232689E048F69C' HTTP/1.1
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    User-Agent: Microsoft ADO.NET Data Services
    Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-e769-2233-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423141026&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=lDBz5YXKiWe5L7eXOHsLHc9kKEUcUiFJvrNFFSksgkM%3d
    x-ms-version: 2.17
    x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
    Host: media.windows.net

回應：

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Length: 1227
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Server: Microsoft-IIS/8.5
    x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
    request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
    x-ms-request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Thu, 05 Feb 2015 07:52:30 GMT

    {"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String",
    "value":"MIIDSTCCAjGgAwIBAgIQqf92wku/HLJGCbMAU8GEnDANBgkqhkiG9w0BAQQFADAuMSwwKgYDVQQDEyN3YW1zYmx1cmVnMDAxZW5jcnlwdGFsbHNlY3JldHMtY2VydDAeFw0xMjA1MjkwNzAwMDBaFw0zMjA1MjkwNzAwMDBaMC4xLDAqBgNVBAMTI3dhbXNibHVyZWcwMDFlbmNyeXB0YWxsc2VjcmV0cy1jZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzR0SEbXefvUjb9wCUfkEiKtGQ5Gc328qFPrhMjSo+YHe0AVviZ9YaxPPb0m1AaaRV4dqWpST2+JtDhLOmGpWmmA60tbATJDdmRzKi2eYAyhhE76MgJgL3myCQLP42jDusWXWSMabui3/tMDQs+zfi1sJ4Ch/lm5EvksYsu6o8sCv29VRwxfDLJPBy2NlbV4GbWz5Qxp2tAmHoROnfaRhwp6WIbquk69tEtu2U50CpPN2goLAqx2PpXAqA+prxCZYGTHqfmFJEKtZHhizVBTFPGS3ncfnQC9QIEwFbPw6E5PO5yNaB68radWsp5uvDg33G1i8IT39GstMW6zaaG7cNQIDAQABo2MwYTBfBgNVHQEEWDBWgBCOGT2hPhsvQioZimw8M+jOoTAwLjEsMCoGA1UEAxMjd2Ftc2JsdXJlZzAwMWVuY3J5cHRhbGxzZWNyZXRzLWNlcnSCEKn/dsJLvxyyRgmzAFPBhJwwDQYJKoZIhvcNAQEEBQADggEBABcrQPma2ekNS3Wc5wGXL/aHyQaQRwFGymnUJ+VR8jVUZaC/U/f6lR98eTlwycjVwRL7D15BfClGEHw66QdHejaViJCjbEIJJ3p2c9fzBKhjLhzB3VVNiLIaH6RSI1bMPd2eddSCqhDIn3VBN605GcYXMzhYp+YA6g9+YMNeS1b+LxX3fqixMQIxSHOLFZ1G/H2xfNawv0VikH3djNui3EKT1w/8aRkUv/AAV0b3rYkP/jA1I0CPn0XFk7STYoiJ3gJoKq9EMXhit+Iwfz0sMkfhWG12/XO+TAWqsK1ZxEjuC9OzrY7pFnNxs4Mu4S8iinehduSpY+9mDd3dHynNwT4="}

### <a name="create-the-content-key"></a>建立內容金鑰
在擷取 X.509 憑證並使用其公開金鑰將內容金鑰加密之後，建立 **ContentKey** 實體並據以設定其屬性值。

建立內容金鑰時必須設定的其中一個值是類型。 使用儲存體加密時，此值應設定為 '1'。 

下列範例示範如何建立 **ContentKey** 且針對儲存體加密設定 **ContentKeyType** ("1")，**ProtectionKeyType** 則設定為 "0"，表示保護金鑰識別碼是 X.509 憑證指紋。  

要求

    POST https://media.windows.net/api/ContentKeys HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    User-Agent: Microsoft ADO.NET Data Services
    Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423034908&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=7eSLe1GHnxgilr3F2FPCGxdL2%2bwy%2f39XhMPGY9IizfU%3d
    x-ms-version: 2.17
    Host: media.windows.net
    {
    "Name":"ContentKey",
    "ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C", 
    "ContentKeyType":"1", 
    "ProtectionKeyType":"0",
    "EncryptedContentKey":"your encrypted content key",
    "Checksum":"calculated checksum"
    }

回應：

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 777
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://media.windows.net/api/ContentKeys('nb%3Akid%3AUUID%3A9c8ea9c6-52bd-4232-8a43-8e43d8564a99')
    Server: Microsoft-IIS/8.5
    request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
    x-ms-request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Wed, 04 Feb 2015 02:37:46 GMT

    {"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeys/@Element",
    "Id":"nb:kid:UUID:9c8ea9c6-52bd-4232-8a43-8e43d8564a99","Created":"2015-02-04T02:37:46.9684379Z",
    "LastModified":"2015-02-04T02:37:46.9684379Z",
    "ContentKeyType":1,
    "EncryptedContentKey":"your encrypted content key",
    "Name":"ContentKey",
    "ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C",
    "ProtectionKeyType":0,
    "Checksum":"calculated checksum"}

## <a name="create-an-asset"></a>建立資產
下列範例示範如何建立資產。

**HTTP 要求**

    POST https://media.windows.net/api/Assets HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
    x-ms-version: 2.17
    Host: media.windows.net

    {"Name":"BigBuckBunny" "Options":1}

**HTTP 回應**

如果成功，則會傳回下列回應：

    HTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 452
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
    Server: Microsoft-IIS/8.5
    x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
    request-id: e98be122-ae09-473a-8072-0ccd234a0657
    x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Sun, 18 Jan 2015 22:06:40 GMT
    {  
       "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
       "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
       "State":0,
       "Created":"2015-01-18T22:06:40.6010903Z",
       "LastModified":"2015-01-18T22:06:40.6010903Z",
       "AlternateId":null,
       "Name":"BigBuckBunny.mp4",
       "Options":1,
       "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
       "StorageAccountName":"storagetestaccount001"
    }

## <a name="associate-the-contentkey-with-an-asset"></a>將 ContentKey 與資產產生關聯
建立 ContentKey 之後, ，使用 $links 作業將其與資產產生關聯，如下列範例所示：

要求：

    POST https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Afbd7ce05-1087-401b-aaae-29f16383c801')/$links/ContentKeys HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Content-Type: application/json
    Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423141026&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=lDBz5YXKiWe5L7eXOHsLHc9kKEUcUiFJvrNFFSksgkM%3d
    x-ms-version: 2.17
    Host: media.windows.net

    {"uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeys('nb%3Akid%3AUUID%3A01e6ea36-2285-4562-91f1-82c45736047c')"}

回應：

    HTTP/1.1 204 No Content 

## <a name="create-an-assetfile"></a>建立 AssetFile
[AssetFile](https://docs.microsoft.com/rest/api/media/operations/assetfile) 實體代表儲存在 blob 容器中的視訊或音訊檔案。 資產檔案一律會與資產相關聯，而資產可包含一或多個資產檔案。 如果資產檔案物件並未與 blob 容器中的數位檔案相關聯，媒體服務編碼器工作將會失敗。

**AssetFile** 執行個體和實際媒體檔是兩個不同的物件。 AssetFile 執行個體包含媒體檔案的相關中繼資料，而媒體檔案包含實際的媒體內容。

當您將數位媒體檔案上傳至 Blob 容器之後，您將使用 **MERGE** HTTP 要求，以媒體檔案的相關資訊來更新 AssetFile (未顯示在本文章中)。 

**HTTP 要求**

    POST https://media.windows.net/api/Files HTTP/1.1
    Content-Type: application/json
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-4ca2-2233-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
    x-ms-version: 2.17
    Host: media.windows.net
    Content-Length: 164

    {  
       "IsEncrypted":"true",
       "EncryptionScheme" : "StorageEncryption", 
       "EncryptionVersion" : "1.0",       
       "EncryptionKeyId" : "nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510",
       "InitializationVector" : "397304628502661816</d:InitializationVector",
       "Options":0,
       "IsPrimary":"false",
       "MimeType":"video/mp4",
       "Name":"BigBuckBunny.mp4",
       "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
    }

**HTTP 回應**

    HTTP/1.1 201 Created
    Cache-Control: no-cache
    Content-Length: 535
    Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
    Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
    Server: Microsoft-IIS/8.5
    request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
    x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
    X-Content-Type-Options: nosniff
    DataServiceVersion: 3.0;
    X-Powered-By: ASP.NET
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Date: Mon, 19 Jan 2015 00:34:07 GMT

    {  
       "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Files/@Element",
       "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
       "Name":"BigBuckBunny.mp4",
       "ContentFileSize":"0",
       "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
       "EncryptionVersion": "1.0",
       "EncryptionScheme": "StorageEncryption",
       "IsEncrypted":true,
       "EncryptionKeyId":"nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510",
       "InitializationVector":"397304628502661816</d:InitializationVector",
       "IsPrimary":false,
       "LastModified":"2015-01-19T00:34:08.1934137Z",
       "Created":"2015-01-19T00:34:08.1934137Z",
       "MimeType":"video/mp4",
       "ContentChecksum":null
    }
