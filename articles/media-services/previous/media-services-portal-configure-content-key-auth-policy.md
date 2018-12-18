---
title: 使用 Azure 入口網站設定內容金鑰授權原則 | Microsoft Docs
description: 了解如何設定內容金鑰的授權原則。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: ee82a3fa-c34b-48f2-a108-8ba321f1691e
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/09/2018
ms.author: juliako
ms.openlocfilehash: db0117b0b4ddee002fc69d71e78eca2b9008e4f6
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33941023"
---
# <a name="configure-a-content-key-authorization-policy"></a>設定內容金鑰授權原則
[!INCLUDE [media-services-selector-content-key-auth-policy](../../../includes/media-services-selector-content-key-auth-policy.md)]

## <a name="overview"></a>概觀
 您可以使用 Azure 媒體服務來傳遞受到進階加密標準 (AES) 保護的 MPEG DASH、Smooth Streaming 和 HTTP Live Streaming (HLS) 串流，方法是使用 128 位元加密金鑰或 [PlayReady 數位版權管理 (DRM)](https://www.microsoft.com/playready/overview/)。 使用媒體服務，您還可以傳遞使用 Widevine DRM 加密的 DASH 串流。 PlayReady 和 Widevine 是依照一般加密 (ISO/IEC 23001-7 CENC) 規格加密。

媒體服務也提供金鑰/授權傳遞服務，用戶端可以從該處取得 AES 金鑰或 PlayReady/Widevine 授權，以便播放加密的內容。

本文示範如何使用 Azure 入口網站設定內容金鑰授權原則。 金鑰稍後可以用來動態加密您的內容。 您目前可以加密 HLS、MPEG-DASH 和 Smooth Streaming 格式。 您無法加密漸進式下載。

當播放器要求設定為動態加密的串流時，媒體服務會使用設定的金鑰，以 AES 或 DRM 加密將您的內容動態加密。 為了將串流解密，播放程式將向金鑰傳遞服務要求金鑰。 為了決定使用者是否有權取得金鑰，服務會評估為金鑰指定的授權原則。

如果您預計有多個內容金鑰，或需要指定金鑰/授權傳遞服務 URL，而非媒體服務金鑰傳遞服務，請使用媒體服務 .NET SDK 或 REST API。 如需詳細資訊，請參閱

* [使用媒體服務 .NET SDK 設定內容金鑰授權原則](media-services-dotnet-configure-content-key-auth-policy.md)
* [使用媒體服務 REST API 設定內容金鑰授權原則](media-services-rest-configure-content-key-auth-policy.md)

### <a name="some-considerations-apply"></a>適用一些考量事項
* 建立媒體服務帳戶時，預設串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流處理您的內容，並利用動態封裝和動態加密功能，您的串流端點必須處於 [執行中] 狀態。 
* 您的資產必須包含一組調適性位元速率 MP4 或調適性位元速率 Smooth Streaming 檔案。 如需詳細資訊，請參閱 [為資產編碼](media-services-encode-asset.md)。
* 金鑰傳遞服務會快取 ContentKeyAuthorizationPolicy 和其相關物件 (原則選項和限制) 15 分鐘。 您可以建立 ContentKeyAuthorizationPolicy 並指定要使用權杖限制、對其進行測試，然後將原則更新為開放的限制。 此程序需要大約 15 分鐘，然後原則才能切換為開放版本。
* 媒體服務串流端點會在預檢回應中，將 CORS Access-Control-Allow-Origin 標頭的值設定為萬用字元 "\*"。 此值適用於大多數的播放程式，包括 Azure 媒體播放器、Roku 和 JWPlayer 等其他播放程式。 不過，有些利用 dash.js 的播放器則不適用，因為，如果將認證模式設定為 "include"，其 dash.js 中的 XMLHttpRequest 就不允許萬用字元 "\*" 作為 Access-Control-Allow-Origin 的值。 對於 dash.js 中此一限制的因應措施是，如果您從單一網域裝載用戶端，媒體服務就能在預檢回應標頭中指定該網域。 如需協助，請透過 Azure 入口網站開啟支援票證。

## <a name="configure-the-key-authorization-policy"></a>設定金鑰授權原則
若要設定金鑰授權原則，請選取 [ **內容保護** ] 頁面。

媒體服務支援多種方式來驗證提出金鑰要求的使用者。 內容金鑰授權原則可以有 open、token 或 IP 授權限制。 (可以使用 REST 或 .NET SDK 來設定 IP。)

### <a name="open-restriction"></a>Open 限制
Open 限制表示系統會傳送金鑰給提出金鑰要求的任何人。 這項限制可用於測試用途。

![OpenPolicy][open_policy]

### <a name="token-restriction"></a>權杖限制
若要選擇權杖限制原則，請選取 [權杖] 按鈕。

權杖限制原則必須伴隨 Security Token Service (STS) 所發出的權杖。 媒體服務支援使用簡單 Web 權杖 ([SWT](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_2)) 和 JSON Web 權杖 (JWT) 格式的權杖。 如需詳細資訊，請參閱 [JWT 驗證](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)。

媒體服務不提供 STS。 您可以建立自訂的 STS 來核發權杖。 STS 必須設定為建立使用指定的索引鍵和問題宣告您在權杖限制組態中指定簽署的權杖。 如果權杖有效，且權杖中的宣告符合為內容金鑰設定的宣告，媒體服務金鑰傳遞服務會將加密金鑰傳回給用戶端。

設定權杖限制的原則時，您必須指定主要驗證金鑰、簽發者和對象參數。 主要驗證金鑰包含簽署權杖用的金鑰。 簽發者為發行權杖的 STS。 對象 (有時稱為範圍) 描述權杖或權杖獲授權存取之資源的用途。 媒體服務金鑰傳遞服務會驗證權杖中的這些值符合在範本中的值。

### <a name="playready"></a>PlayReady
當您使用 PlayReady 保護內容時，必須在驗證原則中指定的其中一件事，是定義 PlayReady 授權範本的 XML 字串。 依預設，會設定下列原則：

    <PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1">
          <LicenseTemplates>
            <PlayReadyLicenseTemplate><AllowTestDevices>true</AllowTestDevices>
              <ContentKey i:type="ContentEncryptionKeyFromHeader" />
              <LicenseType>Nonpersistent</LicenseType>
              <PlayRight>
                <AllowPassingVideoContentToUnknownOutput>Allowed</AllowPassingVideoContentToUnknownOutput>
              </PlayRight>
            </PlayReadyLicenseTemplate>
          </LicenseTemplates>
        </PlayReadyLicenseResponseTemplate>

您可以選取 [匯入原則 xml] 按鈕，並提供與[媒體服務 PlayReady 授權範本概觀](media-services-playready-license-template-overview.md)中定義之 XML 結構描述符合的不同 XML。

## <a name="next-steps"></a>後續步驟
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

[open_policy]: ./media/media-services-portal-configure-content-key-auth-policy/media-services-protect-content-with-open-restriction.png
[token_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-token-restriction.png

