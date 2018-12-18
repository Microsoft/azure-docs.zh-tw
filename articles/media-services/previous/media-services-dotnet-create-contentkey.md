---
title: 使用 .NET 建立 ContentKeys
description: 了解如何建立提供資產安全存取的內容金鑰。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 225b05e5-7d30-409c-b5b7-3ef0634310c7
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/20/2017
ms.author: juliako
ms.openlocfilehash: 53df4c4cef19f6eef99aa15bb265317aa0cd1d58
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33782737"
---
# <a name="create-contentkeys-with-net"></a>使用 .NET 建立 ContentKeys
> [!div class="op_single_selector"]
> * [REST](media-services-rest-create-contentkey.md)
> * [.NET](media-services-dotnet-create-contentkey.md)
> 
> 

媒體服務可讓您建立資產及傳遞已加密的資產。 **ContentKey** 提供**資產**的安全存取。 

當您建立新的資產時 (例如，[將檔案上傳](media-services-dotnet-upload-files.md)之前)，您可以指定下列加密選項：**StorageEncrypted**、**CommonEncryptionProtected** 或 **EnvelopeEncryptionProtected**。 

當您將資產傳遞至您的用戶端時，您可以使用下列兩個加密的其中一個[設定動態加密的資產](media-services-dotnet-configure-asset-delivery-policy.md)：**DynamicEnvelopeEncryption** 或 **DynamicCommonEncryption**。

加密的資產必須與 **ContentKey**相關聯。 本文說明如何建立內容金鑰。

> [!NOTE]
> 使用 Media Services .NET SDK 建立新的 **StorageEncrypted** 資產時，會自動建立 **ContentKey**，並將其與資產連結。
> 
> 

## <a name="contentkeytype"></a>ContentKeyType
您在建立內容金鑰時必須設定的其中一個值就是內容金鑰類型。 選擇下列值之一。 

```csharp
    public enum ContentKeyType
    {
        /// <summary>
        /// Specifies a content key for common encryption.
        /// </summary>
        /// <remarks>This is the default value.</remarks>
        CommonEncryption = 0,

        /// <summary>
        /// Specifies a content key for storage encryption.
        /// </summary>
        StorageEncryption = 1,

        /// <summary>
        /// Specifies a content key for configuration encryption.
        /// </summary>
        ConfigurationEncryption = 2,

        /// <summary>
        /// Specifies a content key for Envelope encryption.  Only used internally.
        /// </summary>
        EnvelopeEncryption = 4
    }
```

## <a id="envelope_contentkey"></a>建立信封類型 ContentKey
下列程式碼片段會建立信封加密類型的內容金鑰。 然後建立金鑰與所指定資產的關聯。

```csharp
    static public IContentKey CreateEnvelopeTypeContentKey(IAsset asset)
    {
        // Create envelope encryption content key
        Guid keyId = Guid.NewGuid();
        byte[] contentKey = GetRandomBuffer(16);

        IContentKey key = _context.ContentKeys.Create(
                                keyId,
                                contentKey,
                                "ContentKey",
                                ContentKeyType.EnvelopeEncryption);

        asset.ContentKeys.Add(key);

        return key;
    }

    static private byte[] GetRandomBuffer(int size)
    {
        byte[] randomBytes = new byte[size];
        using (RNGCryptoServiceProvider rng = new RNGCryptoServiceProvider())
        {
            rng.GetBytes(randomBytes);
        }

        return randomBytes;
    }

call

    IContentKey key = CreateEnvelopeTypeContentKey(encryptedsset);
```


## <a id="common_contentkey"></a>建立一般類型 ContentKey
下列程式碼片段會建立一般加密類型的內容金鑰。 然後建立金鑰與所指定資產的關聯。

```csharp
    static public IContentKey CreateCommonTypeContentKey(IAsset asset)
    {
        // Create common encryption content key
        Guid keyId = Guid.NewGuid();
        byte[] contentKey = GetRandomBuffer(16);

        IContentKey key = _context.ContentKeys.Create(
                                keyId,
                                contentKey,
                                "ContentKey",
                                ContentKeyType.CommonEncryption);

        // Associate the key with the asset.
        asset.ContentKeys.Add(key);

        return key;
    }

    static private byte[] GetRandomBuffer(int length)
    {
        var returnValue = new byte[length];

        using (var rng =
            new System.Security.Cryptography.RNGCryptoServiceProvider())
        {
            rng.GetBytes(returnValue);
        }

        return returnValue;
    }
call

    IContentKey key = CreateCommonTypeContentKey(encryptedsset); 
```

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

