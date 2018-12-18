---
title: 使用 Azure 媒體服務保護您的內容 | Microsoft Docs
description: 此文章簡介如何利用 Media Services 保護內容。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/30/2018
ms.author: juliako
ms.openlocfilehash: 2a3e94d37a926bd36b780b45eb3d6cb29fb73597
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521749"
---
# <a name="content-protection-overview"></a>內容保護概觀

您可以使用 Azure 媒體服務來保護媒體從離開電腦到進行儲存、處理和傳遞時的安全。 使用媒體服務，您就能傳遞利用進階加密標準 (AES-128) 或下列三個主要數位版權管理 (DRM) 系統中任一個所動態加密的即時與隨選內容：Microsoft PlayReady、Google Widevine 和 Apple FairPlay。 媒體服務也提供服務，可傳遞 AES 金鑰和 DRM (PlayReady、Widevine 和 FairPlay) 授權給授權用戶端。 

下圖說明媒體服務內容保護工作流程： 

![保護內容](./media/content-protection/content-protection.png)

&#42; *動態加密支援 AES-128 「清除金鑰」、CBCS 和 CENC。如需詳細資訊，請參閱[這裡](#streaming-protocols-and-encryption-types)的支援矩陣。*

本文說明可了解如何使用媒體服務進行內容保護的相關概念與術語。 本文還有[常見問題集](#faq)小節並提供可說明如何保護內容的文章連結。 

## <a name="main-components-of-the-content-protection-system"></a>內容保護系統的主要元件

若要成功完成「內容保護」系統/應用程式的設計，您需要徹底了解投入量範圍。 以下清單提供您必須實作的三部分概觀。 

1. Azure 媒體服務程式碼
  
  * PlayReady、Widevine 和/或 FairPlay 授權範本。 範本可讓您設定每個已使用 DRM 的使用權限
  * 授權傳遞授權可根據 JWT 中的宣告指定授權檢查的邏輯
  * 內容金鑰、串流通訊協定與套用的對應 DRM，定義 DRM 加密

  > [!NOTE]
  > 您可以使用多種加密類型 (AES-128、PlayReady、Widevine、FairPlay) 來加密每個資產。 請參閱[串流通訊協定和加密類型](#streaming-protocols-and-encryption-types)，以查看哪些組合可行。
  
  下列文章說明使用 AES 和 (或) DRM 加密內容的步驟： 
  
  * [使用 AES 加密保護](protect-with-aes128.md)
  * [使用 DRM 保護](protect-with-drm.md)

2. 使用 AES 或 DRM 用戶端的播放器。 以播放器 SDK (原生或以瀏覽器為基礎) 為基礎的影片播放器應用程式必須符合下列需求：
  * 播放器 SDK 支援所需的 DRM 用戶端
  * 播放器 SDK 支援所需的串流通訊協定：Smooth、DASH 和/或 HLS
  * 播放器 SDK 必須能夠處理在授權取得要求中傳遞 JWT 權杖
  
    您可以使用 [Azure 媒體播放器 API](http://amp.azure.net/libs/amp/latest/docs/) 來建立播放器。 使用 [Azure 媒體播放器的 ProtectionInfo API](http://amp.azure.net/libs/amp/latest/docs/) 來指定要在不同的 DRM 平台上使用哪個 DRM 技術。

    若要測試 AES 或 CENC (Widevine 和/或 PlayReady) 加密的內容，您可以使用 [Azure 媒體播放器](https://ampdemo.azureedge.net/azuremediaplayer.html) \(英文\)。 務必按一下 [進階選項] 並檢查您的加密選項。

    如果您想要測試 FairPlay 加密內容，請使用[此測試播放程式](http://aka.ms/amtest)。 該播放程式支援 Widevine、PlayReady 及 FairPlay DRM，以及 AES-128 清除金鑰加密。 您必須選擇正確的瀏覽器以測試不同的 DRM：Chrome/Opera/Firefox 適用於 Widevine、MS Edge/IE11 適用於 PlayReady、macOS 上的 Safari 適用於 FairPlay。

3. 安全性權杖服務 (STS) 會發出 JSON Web 權杖 (JWT) 作為後端資源存取的存取權杖。 您可以使用 AMS 授權傳遞服務作為後端資源。 STS 必須定義以下項目：

  * 簽發者和對象 (或範圍)
  * 宣告取決於內容保護中的商務需求
  * 簽章驗證的對稱或非對稱驗證
  * 金鑰變換支援 (如有必要)

    您可以使用[此 STS 工具](https://openidconnectweb.azurewebsites.net/DRMTool/Jwt)測試 STS，此工具可支援全部 3 種類型的驗證金鑰：對稱、非對稱，或具有金鑰變換的 AAD。 

> [!NOTE]
> 強烈建議專注並完整測試每個部分 (如上所述) 再移到下一個部分。 若要測試您的「內容保護」系統，請使用上述清單中指定的工具。  

## <a name="streaming-protocols-and-encryption-types"></a>串流通訊協定和加密類型

您可以使用媒體服務，來傳遞藉由使用 PlayReady、Widevine 或 FairPlay 並透過 AES 未加密金鑰或 DRM 加密所動態加密的內容。 您目前可以加密 HTTP 即時串流 (HLS)、MPEG DASH 和 Smooth Streaming 格式。 每個通訊協定支援下列加密方法：

|通訊協定|容器格式|加密配置|
|---|---|---|---|
|MPEG-DASH|全部|AES|
||CSF(fmp4) |CENC (Widevine + PlayReady) |
||CMAF(fmp4)|CENC (Widevine + PlayReady)|
|HLS|全部|AES|
||MPG2-TS |CBCS (Fairplay) |
||MPG2-TS |CENC (PlayReady) |
||CMAF(fmp4) |CENC (PlayReady) |
|Smooth Streaming|fMP4|AES|
||fMP4 | CENC (PlayReady) |

## <a name="dynamic-encryption"></a>動態加密

在媒體服務 v3 中，內容金鑰會與 StreamingLocator 相關聯 (請參閱[本例](protect-with-aes128.md))。 如果使用媒體服務金鑰傳遞服務，您應自動產生內容金鑰。 如果您使用自己的金鑰傳遞服務，或需要處理高可用性案例，其中需要在兩個資料中心具有相同的內容金鑰，則您應自行產生內容金鑰。

播放程式要求串流時，媒體服務便會使用 AES 清除金鑰或 DRM 加密，使用指定的金鑰動態加密您的內容。 為了將串流解密，播放程式會向媒體服務金鑰傳遞服務或您指定的金鑰傳遞服務要求金鑰。 為了決定使用者是否有權取得金鑰，服務會評估您為金鑰指定的內容金鑰原則。

## <a name="aes-128-clear-key-vs-drm"></a>AES-128 未加密金鑰與DRM

客戶通常不知道他們應該使用 AES 加密，還是 DRM 系統。 這兩個系統的主要差異是，使用 AES 加密時，系統會以未加密格式 (「純文字」) 將內容金鑰傳輸至用戶端。 因此，就能在用戶端的網路追蹤中，以純文字形式檢視用來加密內容的金鑰。 AES-128 未加密金鑰加密適用於檢視者為受信任合作對象的使用案例 (例如，將在公司內部散發的公司影片加密，以供員工檢視)。

相較於 AES-128 未加密金鑰加密，PlayReady、Widevine 及 FairPlay 全都會提供較高層級的加密。 內容金鑰會以加密格式傳輸。 此外，解密會在安全環境中，惡意使用者更難攻擊的作業系統層級上進行處理。 針對檢視者可能不是受信任的合作對象，而且您需要最高層級安全性的使用案例，建議使用 DRM。

## <a name="storage-side-encryption"></a>儲存端加密

若要保護待用資產，資產應該透過儲存端加密來進行加密。 下表顯示儲存端加密在媒體服務 v3 中的運作方式：

|加密選項|說明|媒體服務 v3|
|---|---|---|---|
|媒體服務的儲存體加密| AES-256 加密，由媒體服務管理金鑰|不受支援<sup>(1)</sup>|
|[待用資料的儲存體服務加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)|Azure 儲存體提供的伺服器端加密，由 Azure 或客戶管理金鑰|支援|
|[儲存體用戶端加密](https://docs.microsoft.com/azure/storage/common/storage-client-side-encryption)|Azure 儲存體提供的用戶端加密，由客戶管理 Key Vault 中的金鑰|不支援|

<sup>1</sup> 在媒體服務 v3 中，如果您的資產是以媒體服務 v2 建立，則儲存體加密 (AES-256 加密) 只對回溯相容性有所支援。 這表示 v3 可用於現有的儲存體加密資產，但不允許建立新的。

## <a name="licenses-and-keys-delivery-service"></a>授權和金鑰傳遞服務

媒體服務提供一種金鑰傳遞服務，可將 DRM (PlayReady、Widevine 與 FairPlay) 授權和 AES 金鑰傳遞給授權用戶端。 您可以使用 REST API 或媒體服務用戶端程式庫，來設定授權和金鑰的授權和驗證原則。

## <a name="control-content-access"></a>控制內容存取

您可以設定內容金鑰原則來控制可存取內容的人員。 媒體服務支援多種方式來驗證提出金鑰要求的使用者。 您必須設定內容金鑰原則。 用戶端 (播放器) 必須先符合原則，系統才會將金鑰傳遞給用戶端。 內容金鑰原則可具有**開啟**或**權杖**限制。 

使用權杖限制的內容金鑰原則時，系統只會將內容金鑰傳送給可在金鑰/授權要求中提供有效 JSON Web 權杖 (JWT) 或簡單 Web 權杖 (SWT) 的用戶端。 這個權杖必須由 Security Token Service (STS) 來核發。 您可以使用 Azure Active Directory 作為 STS，或部署自訂 STS。 STS 必須設定為建立使用指定的索引鍵和問題宣告您在權杖限制組態中指定簽署的權杖。 如果權杖有效，且權杖中的宣告符合針對金鑰/授權所設定的宣告，則媒體服務金鑰傳遞服務會將所要求的金鑰/授權傳回給用戶端。

設定權杖限制的原則時，您必須指定主要驗證金鑰、簽發者和對象參數。 主要驗證金鑰包含簽署權杖用的金鑰。 簽發者為發行權杖的安全性權杖服務。 對象 (有時稱為「範圍」) 描述權杖或權杖獲授權存取之資源的用途。 媒體服務金鑰傳遞服務會驗證權杖中的這些值符合在範本中的值。

## <a name="a-idfaqfrequently-asked-questions"></a><a id="faq"/>常見問題集

### <a name="question"></a>問題

如何使用 Azure 媒體服務 (AMS) v3 以及使用 AMS 授權/金鑰傳遞服務，實作多重 DRM (PlayReady、Widevine 和 FairPlay) 系統？

### <a name="answer"></a>Answer

如需對於端對端案例，請參閱[下列程式碼範例](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs)。 

下列範例示範如何：

1. 建立並設定 ContentKeyPolicies。

  此範例包含可設定 [PlayReady](playready-license-template-overview.md)、[Widevine](widevine-license-template-overview.md) 和 [FairPlay](fairplay-license-overview.md) 授權的函式。

    ```
    ContentKeyPolicyPlayReadyConfiguration playReadyConfig = ConfigurePlayReadyLicenseTemplate();
    ContentKeyPolicyWidevineConfiguration widevineConfig = ConfigureWidevineLicenseTempate();
    ContentKeyPolicyFairPlayConfiguration fairPlayConfig = ConfigureFairPlayPolicyOptions();
    ```

2. 建立已設定為要串流處理已加密資產的 StreamingLocator。 

  在此範例中，我們會將 **StreamingPolicyName** 設定為 **PredefinedStreamingPolicy.SecureStreaming**，它可支援信封和 cenc 加密，並可在 StreamingLocator 上設定兩個內容金鑰。 

  如果您也想要使用 FairPlay 進行加密，請將 **StreamingPolicyName** 設定為 **PredefinedStreamingPolicy.SecureStreamingWithFairPlay**。

3. 建立測試權杖。

  **GetTokenAsync** 方法顯示如何建立測試權杖。
  
4. 建置串流 URL。

  **GetDASHStreamingUrlAsync** 方法說明如何建置串流 URL。 在此案例中，URL 會串流處理 **DASH** 內容。

### <a name="question"></a>問題

如何以及在何處取得 JWT 權杖，再用來要求授權或金鑰？

### <a name="answer"></a>Answer

1. 用於生產時，您需要有 Security Token Service (STS) (Web 服務)，才能根據 HTTPS 要求發出 JWT 權杖。 用於測試時，您可以使用 [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs) 中定義的 **GetTokenAsync** 方法所示的程式碼。
2. 播放器必須在使用者經過驗證之後，向 STS 提出此類權杖的要求，並將它指派為權杖的值。 您可以使用 [Azure 媒體播放器 API](https://amp.azure.net/libs/amp/latest/docs/)。

* 如需執行 STS (使用對稱和非對稱金鑰) 的範例，請參閱 [http://aka.ms/jwt](http://aka.ms/jwt)。 
* 如需以使用這類 JWT 權杖的 Azure 媒體播放器為基礎的播放器範例，請參閱 [http://aka.ms/amtest](http://aka.ms/amtest) (展開 "player_settings" 連結以查看權杖輸入)。

### <a name="question"></a>問題

如何授權要求以使用 AES 加密來串流處理影片？

### <a name="answer"></a>Answer

正確的方法是利用 STS (Secure Token Service)：

在 STS 中，根據使用者設定檔，新增不同的宣告 (例如「進階使用者」、「基本使用者」、「免費試用使用者」)。 在 JWT 中使用不同的宣告，使用者可以看見不同的內容。 當然，對於不同的內容/資產，ContentKeyPolicyRestriction 會有對應的 RequiredClaims。

使用 Azure 媒體服務 API 來設定授權/金鑰傳遞並將您的資產加密 (請參考[這個範例](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES/Program.cs)。

## <a name="next-steps"></a>後續步驟

請查看下列文章：

  * [使用 AES 加密保護](protect-with-aes128.md)
  * [使用 DRM 保護](protect-with-drm.md)

在[設計具有存取控制的多重 drm 保護系統](design-multi-drm-system-with-access-control.md)中可以找到其他資訊


