---
title: Azure 媒體服務和 Apple FairPlay 授權支援 | Microsoft Docs
description: 本主題說明 Apple FairPlay 授權需求和設定的概觀。
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
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: 19f382de3ffe11253005f5fa2874ee817abaeed3
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376749"
---
# <a name="apple-fairplay-license-requirements-and-configuration"></a>Apple FairPlay 授權需求和設定 

Azure 媒體服務可讓您使用 **Apple FairPlay** (AES-128 CBC) 來加密您的 HLS 內容。 媒體服務也提供傳遞 FairPlay 授權的服務。 當播放程式嘗試播放以 FairPlay 保護的內容時，會將要求傳送到授權傳遞服務來取得授權。 如果授權服務核准要求，就會發出授權以傳送給用戶端，並將它用來解密和播放所指定內容。

媒體服務也會提供您可以用來設定 FairPlay 授權的 API。 本主題討論 FairPlay 授權需求，並示範如何使用媒體服務 API 設定 **FairPlay** 授權。 

## <a name="requirements"></a>需求

當使用媒體服務搭配 **Apple FairPlay** 來加密您的 HLS 內容，以及使用媒體服務來傳遞 FairPlay 授權時，需要下列項目：

* 註冊 [Apple Development Program](https://developer.apple.com/)。
* Apple 要求內容擁有者必須取得 [部署套件](https://developer.apple.com/contact/fps/)。 說明您已使用媒體服務實作金鑰安全性模組 (KSM)，現在想要求最終的 FPS 套件。 最終 FPS 套件包含相關指示，用以產生憑證和取得應用程式密碼金鑰 (ASK)。 您可使用 ASK 來設定 FairPlay。
* 必須在媒體服務金鑰/授權傳遞端設定下列各項︰

    * **應用程式憑證 (AC)**︰這是包含私密金鑰的 .pfx 檔案。 您可建立這個檔案並以密碼加密。 .pfx 檔案應該是 Base64 格式。

        下列步驟說明如何產生 FairPlay 的 .pfx 憑證檔案：

        1. 從 https://slproweb.com/products/Win32OpenSSL.html 安裝 OpenSSL。

            移至 FairPlay 憑證和其他 Apple 提供檔案所在的資料夾。
        2. 從命令列執行下列命令。 這會將 .cer 檔案轉換成 .pem 檔案。

            "C:\OpenSSL-Win32\bin\openssl.exe" x509 -inform der -in FairPlay.cer -out FairPlay-out.pem
        3. 從命令列執行下列命令。 這會將 .pem 檔案轉換為包含私密金鑰的 .pfx 檔案。 OpenSSL 程式會接著要求 .Pfx 檔案的密碼。

            "C:\OpenSSL-Win32\bin\openssl.exe" pkcs12 -export -out FairPlay-out.pfx -inkey privatekey.pem -in FairPlay-out.pem -passin file:privatekey-pem-pass.txt
            
    * **應用程式憑證密碼** - 用來建立 .pfx 檔案的密碼。
    * **ASK**：當您使用 Apple 開發人員入口網站產生憑證時，會收到此金鑰。 每個開發小組都會收到一個唯一的 ASK。 儲存一份 ASK，並將它存放在安全的地方。 您必須使用媒體服務將 ASK 設定為 FairPlayAsk。
    
* FPS 用戶端必須設定下列各項︰

  * **應用程式憑證 (AC)**：這是 .cer/.der 檔案，其中包含作業系統用來加密某些承載的公開金鑰。 媒體服務必須了解它，因為播放程式需要它。 金鑰傳遞服務會使用對應的私密金鑰來解密。

* 若要播放 FairPlay 加密的資料流，請先取得真正的 ASK，然後產生真正的憑證。 該處理程序會建立所有 3 個部分︰

  * .der 檔案
  * .pfx 檔案
  * .pfx 的密碼

## <a name="fairplay-and-player-apps"></a>FairPlay 和播放應用程式

當您透過 **Apple FairPlay** 加密內容時，個別的視訊和音訊範例會使用 **AES-128 CBC** 模式加密。 **FairPlay 串流** (FPS) 已整合至裝置工作系統，在 iOS 和 Apple 電視上具有原生支援。 OS X 上的 Safari 使用加密媒體擴充功能 (EME) 介面支援來啟用 FPS。

Azure 媒體播放器也支援 FairPlay 播放。 如需詳細資訊，請參閱 [Azure 媒體播放器文件](https://amp.azure.net/libs/amp/latest/docs/index.html) \(英文\)。

您可以使用 iOS SDK 開發自己的播放應用程式。 為了能夠播放 FairPlay 內容，您必須實作授權交換通訊協定。 此通訊協定不是由 Apple 指定。 而是由每個應用程式如何傳送金鑰傳遞要求而決定。 媒體服務 FairPlay 金鑰傳遞服務會對即將到來的 SPC 視為如下列格式的 www-form-url 已編碼張貼訊息：

```
spc=<Base64 encoded SPC>
```

## <a name="fairplay-configuration-net-example"></a>FairPlay 設定 .NET 範例

您可以使用媒體服務 API 來設定 FairPlay 授權。 當播放程式嘗試播放以 FairPlay 保護的內容時，會將要求傳送到授權傳遞服務來取得授權。 如果授權服務核准要求，服務就會發出授權。 授權會傳送至用戶端，並用來解密及播放指定內容。

> [!NOTE]
> 您通常只想要設定一次 FairPlay 原則選項，因為您只會有一組憑證和 ASK。

下列範例使用[媒體服務 .NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.models?view=azure-dotnet) 設定授權。

```csharp
private static ContentKeyPolicyFairPlayConfiguration ConfigureFairPlayPolicyOptions()
{

    string askHex = "";
    string FairPlayPfxPassword = "";

    var appCert = new X509Certificate2("FairPlayPfxPath", FairPlayPfxPassword, X509KeyStorageFlags.Exportable);

    byte[] askBytes = Enumerable
        .Range(0, askHex.Length)
        .Where(x => x % 2 == 0)
        .Select(x => Convert.ToByte(askHex.Substring(x, 2), 16))
        .ToArray();

    ContentKeyPolicyFairPlayConfiguration fairPlayConfiguration =
    new ContentKeyPolicyFairPlayConfiguration
    {
        Ask = askBytes,
        FairPlayPfx =
                Convert.ToBase64String(appCert.Export(X509ContentType.Pfx, FairPlayPfxPassword)),
        FairPlayPfxPassword = FairPlayPfxPassword,
        RentalAndLeaseKeyType =
                ContentKeyPolicyFairPlayRentalAndLeaseKeyType
                .PersistentUnlimited,
        RentalDuration = 2249
    };

    return fairPlayConfiguration;
}
```

## <a name="next-steps"></a>後續步驟

查看如何[使用 DRM 保護](protect-with-drm.md)