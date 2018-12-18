---
title: 使用 Azure 媒體服務保護您的內容 | Microsoft Docs
description: 此文章簡介如何利用 Media Services 保護內容。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 81bc00e1-dcda-4d69-b9ab-8768b793422b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/01/2018
ms.author: juliako
ms.openlocfilehash: 63cf7633a2280682b3a3da7e8939e71e83ee8f3b
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37342105"
---
# <a name="content-protection-overview"></a>內容保護概觀
 您可以使用 Azure 媒體服務來保護媒體從離開電腦到進行儲存、處理和傳遞時的安全。 使用媒體服務，您就能傳遞利用進階加密標準 (AES-128) 或下列三個主要數位版權管理 (DRM) 系統中任一個所動態加密的即時與隨選內容：Microsoft PlayReady、Google Widevine 和 Apple FairPlay。 媒體服務也提供服務，可傳遞 AES 金鑰和 DRM (PlayReady、Widevine 和 FairPlay) 授權給授權用戶端。 

下圖說明媒體服務內容保護工作流程： 

![利用 PlayReady 保護](./media/media-services-content-protection-overview/media-services-content-protection-with-multi-drm.png)

本文說明可了解如何使用媒體服務進行內容保護的相關概念與術語。 本文也會提供討論如何保護內容的文章連結。 

## <a name="dynamic-encryption"></a>動態加密
 您可以使用媒體服務，來傳遞藉由使用 PlayReady、Widevine 或 FairPlay 並透過 AES 未加密金鑰或 DRM 加密所動態加密的內容。 您目前可以加密 HTTP 即時串流 (HLS)、MPEG DASH 和 Smooth Streaming 格式。 不支援在漸進式下載上加密。 每一種加密方法都支援下列串流處理通訊協定：

- AES：MPEG DASH、 Smooth Streaming 和 HLS
- PlayReady：MPEG DASH、 Smooth Streaming 和 HLS
- Widevine：MPEG DASH
- FairPlay：HLS

若要加密資產，您需要建立加密內容金鑰與您資產的關聯，同時也要設定金鑰的授權原則。 內容金鑰可由您指定或由媒體服務自動產生。

您也需要設定資產的傳遞原則。 如果您想要串流處理儲存體加密的資產，請務必透過設定資產傳遞原則來指定資產傳遞方式。

播放程式要求串流時，媒體服務便會使用 AES 清除金鑰或 DRM 加密，使用指定的金鑰動態加密您的內容。 為了將串流解密，播放程式會向媒體服務金鑰傳遞服務要求金鑰。 為了決定使用者是否有權取得金鑰，服務會評估為金鑰指定的授權原則。

## <a name="aes-128-clear-key-vs-drm"></a>AES-128 未加密金鑰與DRM
客戶通常不知道他們應該使用 AES 加密，還是 DRM 系統。 這兩個系統的主要差異是，使用 AES 加密時，系統會以未加密格式 (「純文字」) 將內容金鑰傳輸至用戶端。 因此，就能在用戶端的網路追蹤中，以純文字形式檢視用來加密內容的金鑰。 AES-128 未加密金鑰加密適用於檢視者為受信任合作對象的使用案例 (例如，將在公司內部散發的公司影片加密，以供員工檢視)。

相較於 AES-128 未加密金鑰加密，PlayReady、Widevine 及 FairPlay 全都會提供較高層級的加密。 內容金鑰會以加密格式傳輸。 此外，解密會在安全環境中，惡意使用者更難攻擊的作業系統層級上進行處理。 針對檢視者可能不是受信任的合作對象，而且您需要最高層級安全性的使用案例，建議使用 DRM。

## <a name="storage-encryption"></a>儲存體加密
您可以藉由儲存體加密，使用 AES 256 位元加密對您的明文內容進行本機加密。 然後，您可以將它上傳到 Azure 儲存體，以便以靜止狀態加密儲存。 以儲存體加密保護的資產會自動解除加密並在編碼前放置在加密的檔案系統中。 在以新輸出資產的形式上傳回去之前，系統會選擇性地重新加密這些資產。 儲存體加密的主要使用案例是，讓您可以使用強式加密來保護磁碟中靜止的高品質輸入媒體檔。

若要傳遞儲存體加密資產，您必須設定資產的傳遞原則，讓媒體服務知道您的內容傳遞方式。 串流處理資產之前，串流伺服器會使用指定的傳遞原則 (例如，AES、一般加密或不加密) 來解密並串流處理您的內容。

## <a name="types-of-encryption"></a>加密的類型
Playready 和 Widevine 會利用一般加密 (AES CTR 模式)。 FairPlay 會利用 AES CBC 模式加密。 AES-128 未加密金鑰加密會利用信封加密。

## <a name="licenses-and-keys-delivery-service"></a>授權和金鑰傳遞服務
媒體服務提供一種金鑰傳遞服務，可將 DRM (PlayReady、Widevine 與 FairPlay) 授權和 AES 金鑰傳遞給授權用戶端。 若要設定您授權和金鑰的授權和驗證原則，才能使用 [Azure 入口網站](media-services-portal-protect-content.md)、REST API 或 Media Services SDK for .NET。

## <a name="control-content-access"></a>控制內容存取
您可以設定內容金鑰授權原則來控制可存取內容的人員。 內容金鑰授權原則支援 open 或權杖限制。

### <a name="open-authorization"></a>Open 授權
利用 open 授權原則，可將內容金鑰傳送給任何用戶端 (無限制)。

### <a name="token-authorization"></a>權杖授權
使用權杖限制的授權原則時，系統只會將內容金鑰傳送給可在金鑰/授權要求中提供有效 JSON Web 權杖 (JWT) 或簡單 Web 權杖 (SWT) 的用戶端。 這個權杖必須由 Security Token Service (STS) 來核發。 您可以使用 Azure Active Directory 作為 STS，或部署自訂 STS。 STS 必須設定為建立使用指定的索引鍵和問題宣告您在權杖限制組態中指定簽署的權杖。 如果權杖有效，且權杖中的宣告符合針對金鑰/授權所設定的宣告，則媒體服務金鑰傳遞服務會將所要求的金鑰/授權傳回給用戶端。

設定權杖限制的原則時，您必須指定主要驗證金鑰、簽發者和對象參數。 主要驗證金鑰包含簽署權杖用的金鑰。 簽發者為發行權杖的安全性權杖服務。 對象 (有時稱為「範圍」) 描述權杖或權杖獲授權存取之資源的用途。 媒體服務金鑰傳遞服務會驗證權杖中的這些值符合在範本中的值。

## <a name="streaming-urls"></a>串流 URL
如果使用一個以上 DRM 來加密您的資產，請使用串流 URL 中的加密標籤：(format='m3u8-aapl', encryption='xxx')。

您必須考量下列事項：

* 無法指定一個以上的加密類型。
* 如果只有一個加密套用到資產，則無須在 URL 中指定加密類型。
* 加密類型不區分大小寫。
* 可以指定下列加密類型︰
  * **cenc**︰適用於 PlayReady 或 Widevine (一般加密)
  * **cbcs-aapl**：適用於 FairPlay (AES CBC 加密)
  * **cbc**：適用於 AES 信封加密

## <a name="next-steps"></a>後續步驟
下列文章描述可協助您開始使用內容保護的後續步驟：

* [使用儲存體加密保護](media-services-rest-storage-encryption.md)
* [使用 AES 加密保護](media-services-protect-with-aes128.md)
* [使用 PlayReady 和/或 Widevine 保護](media-services-protect-with-playready-widevine.md)
* [使用 FairPlay 保護](media-services-protect-hls-with-FairPlay.md)

## <a name="related-links"></a>相關連結

* [JWT 權杖驗證](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)
* [整合 Azure 媒體服務 OWIN MVC 型應用程式與 Azure Active Directory 並根據 JWT 宣告限制內容金鑰傳遞](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)

[content-protection]: ./media/media-services-content-protection-overview/media-services-content-protection.png
