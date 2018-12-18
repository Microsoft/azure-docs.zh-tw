---
title: 使用 Azure 入口網站設定內容保護原則 | Microsoft Docs
description: 本文章示範如何使用 Azure 入口網站設定內容保護原則。 文章也會說明如何啟用資產的動態加密。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 270b3272-7411-40a9-ad42-5acdbba31154
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/25/2017
ms.author: juliako
ms.openlocfilehash: c46faf2298ebaac4f40fb1d18cbfca83076e0d4f
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39423531"
---
# <a name="configure-content-protection-policies-by-using-the-azure-portal"></a>使用 Azure 入口網站設定內容保護原則
 您可以透過 Azure 媒體服務來保護媒體從離開電腦到進行儲存、處理和傳遞時的安全。 您可以利用媒體服務傳遞，使用進階加密標準 (AES) (使用 128 位元加密金鑰) 所動態加密的內容。 您也可以將它與一般加密 (CENC) 搭配使用，方法是使用 PlayReady 和/或 Widevine 數位版權管理 (DRM) 和 Apple FairPlay。 

媒體服務提供一種服務，將 DRM 授權和 AES 清除金鑰傳遞給授權用戶端。 您可以使用 Azure 入口網站，針對所有加密類型建立一個金鑰/授權的授權原則。

本文示範如何使用入口網站設定內容保護原則。 文章也會說明如何將動態加密套用到資產。

## <a name="start-to-configure-content-protection"></a>開始設定內容保護
若要使用媒體服務帳戶，並使用入口網站來設定全域內容保護，請採取下列步驟：

1. 在[入口網站](https://portal.azure.com/)中，選取您的媒體服務帳戶。

1. 選取 [設定] > [內容保護]。

    ![內容保護](./media/media-services-portal-content-protection/media-services-content-protection001.png)

## <a name="keylicense-authorization-policy"></a>金鑰/授權的授權原則
媒體服務支援多種方式來驗證提出金鑰或授權要求的使用者。 您必須設定內容金鑰授權原則。 接著，您的用戶端必須先符合原則，系統才會將金鑰/授權傳遞給它。 內容金鑰授權原則可能有一個或多個授權限制，可能是 Open 或權杖限制。

您可以使用入口網站，針對所有加密類型建立一個金鑰/授權的授權原則。

### <a name="open-authorization"></a>Open 授權
Open 限制表示系統會將金鑰傳遞給提出金鑰要求的任何人。 這項限制可用於測試用途。 

### <a name="token-authorization"></a>權杖授權
權杖限制原則必須伴隨 Security Token Service (STS) 所發出的權杖。 媒體服務支援使用簡單 Web 權杖 (SWT) 和 JSON Web 權杖 (JWT) 格式的權杖。 媒體服務不提供 STS。 您可以建立自訂 STS，或使用 Azure 存取控制服務來發行權杖。 STS 必須設定為建立使用指定的索引鍵和問題宣告您在權杖限制組態中指定簽署的權杖。 如果權杖有效，且權杖中的宣告符合針對金鑰 (或授權) 所設定的宣告，媒體服務金鑰傳遞服務會將所要求的金鑰 (或授權) 傳回給用戶端。

設定權杖限制的原則時，您必須指定主要驗證金鑰、簽發者和對象參數。 主要驗證金鑰包含簽署權杖用的金鑰。 簽發者為發行權杖的安全性權杖服務。 對象 (有時稱為範圍) 描述權杖或權杖獲授權存取之資源的用途。 媒體服務金鑰傳遞服務會驗證權杖中的這些值符合在範本中的值。

![金鑰/授權的授權原則](./media/media-services-portal-content-protection/media-services-content-protection002.png)

## <a name="playready-license-template"></a>PlayReady 授權範本
PlayReady 授權範本會設定已在您的 PlayReady 授權上啟用的功能。 如需 PlayReady 授權範本的詳細資訊，請參閱[媒體服務 PlayReady 授權範本概觀](media-services-playready-license-template-overview.md)。

### <a name="nonpersistent"></a>非持續性
如果您將授權設定為非持續性，則僅當播放器使用授權時，才會保留在記憶體中。  

![非持續性內容保護](./media/media-services-portal-content-protection/media-services-content-protection003.png)

### <a name="persistent"></a>持續性
如果您將授權設定為持續性，它會儲存在用戶端上的永續性儲存體中。

![持續性內容保護](./media/media-services-portal-content-protection/media-services-content-protection004.png)

## <a name="widevine-license-template"></a>Widevine 授權範本
Widevine 授權範本會設定已在您 Widevine 授權上啟用的功能。

### <a name="basic"></a>基本
當您選取 [基本] 時，會使用所有預設值來建立範本。

### <a name="advanced"></a>進階
如需 Widevine 權限範本的詳細資訊，請參閱 [Widevine 授權範本概觀](media-services-widevine-license-template-overview.md)。

![進階內容保護](./media/media-services-portal-content-protection/media-services-content-protection005.png)

## <a name="fairplay-configuration"></a>FairPlay 組態
若要啟用 FairPlay 加密，請選取 **FairPlay 設定**。 然後選取**應用程式憑證**，並輸入**應用程式秘密金鑰**。 如需 FairPlay 設定和需求的詳細資訊，請參閱[使用 Apple FairPlay 或 Microsoft PlayReady 保護 HLS 內容](media-services-protect-hls-with-FairPlay.md)。

![FairPlay 組態](./media/media-services-portal-content-protection/media-services-content-protection006.png)

## <a name="apply-dynamic-encryption-to-your-asset"></a>將動態加密套用到您的資產
若要利用動態封裝功能，請將您的來源檔案編碼成一組調適性位元速率 MP4 檔案。

### <a name="select-an-asset-that-you-want-to-encrypt"></a>選取您要加密的資產
若要查看您所有的資產，請選取 [設定] > [資產]。

![資產選項](./media/media-services-portal-content-protection/media-services-content-protection007.png)

### <a name="encrypt-with-aes-or-drm"></a>使用 AES 或 DRM 加密
當您針對資產選取 [加密] 時，會看到兩個選擇：**AES** 或 **DRM**。 

#### <a name="aes"></a>AES
AES 清除金鑰加密會在所有串流通訊協定上啟用︰Smooth Streaming、HLS 和 MPEG DASH。

![加密設定](./media/media-services-portal-content-protection/media-services-content-protection008.png)

#### <a name="drm"></a>DRM
1. 選取 **DRM** 後，您會看到不同的內容保護原則 (必須由此點設定) 和一組串流通訊協定：

    a. **PlayReady 和 Widevine 與 MPEG-DASH** 會使用 PlayReady 與 Widevine DRM 將您的 MPEG DASH 串流動態加密。

    b. **PlayReady 和 Widevine 與 MPEG-DASH + FairPlay 與 HLS** 會使用 PlayReady 與 Widevine DRM 將您的 MPEG DASH 串流動態加密。 此選項也會使用 FairPlay 將您的 HLS 串流加密。

    c. **PlayReady 僅與 Smooth Streaming、HLS 和 MPEG-DASH** 會使用 PlayReady DRM 將 Smooth Streaming、HLS、MPEG-DASH 串流動態加密。

    d. **Widevine 僅與 MPEG-DASH** 會使用 Widevine DRM 將您的 MPEG-DASH 動態加密。
    
    e. **FairPlay 僅與 HLS** 會使用 FairPlay 將您的 HLS 串流動態加密。

1. 若要啟用 FairPlay 加密，請在 [內容保護全域設定] 刀鋒視窗中選取 [FairPlay 設定]。 然後選取 [應用程式憑證]，並輸入**應用程式秘密金鑰**。

    ![加密類型](./media/media-services-portal-content-protection/media-services-content-protection009.png)

1. 您進行加密選取後，請選取 [套用]。

>[!NOTE] 
>如果您計劃在 Safari 中播放 AES 加密的 HLS，請參閱 [Safari 中加密的 HLS](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/)。

## <a name="next-steps"></a>後續步驟
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

