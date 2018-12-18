---
title: 搭配使用 DRM 動態加密授權傳遞服務與 Azure 媒體服務 | Microsoft Docs
description: 您可以使用 Azure 媒體服務，傳遞以 Microsoft PlayReady、Google Widevine 或 Apple FairPlay 授權加密的串流。
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: 2a8a00ab034016e7121e4601b3ff5a16d8c721ac
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49395067"
---
# <a name="use-drm-dynamic-encryption-and-license-delivery-service"></a>使用 DRM 動態加密與授權傳遞服務

您可以使用 Azure 媒體服務來傳遞受到 [PlayReady 數位版權管理 (DRM)](https://www.microsoft.com/playready/overview/) 保護的 MPEG-DASH、Smooth Streaming 和 HTTP 即時串流 (HLS) 串流。 您也可以使用媒體服務來傳遞含有 **Google Widevine** DRM 授權的加密 DASH 串流。 PlayReady 和 Widevine 是依照一般加密 (ISO/IEC 23001-7 CENC) 規格加密。 媒體服務也可讓您使用 **Apple FairPlay** (AES-128 CBC) 來加密您的 HLS 內容。 

此外，媒體服務也提供傳遞 PlayReady、Widevine 和 FairPlay DRM 授權的服務。 當使用者要求受 DRM 保護的內容時，播放器應用程式會向媒體服務授權服務要求授權。 如果播放器應用程式已獲得授權，媒體服務授權服務就會發出授權給播放器。 授權包含解密金鑰，可被用戶端播放器用來解密和串流處理內容。

本文以[使用 DRM 進行加密](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM)範例為基礎。 此範例的作用之一是示範如何：

* 建立編碼轉換，該轉換會使用內建預設的自適性位元速率編碼，並且直接從 [HTTP 來源 URL](job-input-from-http-how-to.md) 內嵌檔案。
* 設定用來驗證權杖的簽署金鑰。
* 設定要使用指定的組態傳遞金鑰所須符合的內容金鑰原則需求 (限制)。 

    * 組態 
    
        此範例會設定 [PlayReady](playready-license-template-overview.md) 和 [Widevine](widevine-license-template-overview.md) 授權，使其可由媒體服務授權傳遞服務來傳遞。 雖然此範例應用程式不會設定 [FairPlay](fairplay-license-overview.md) 授權，但還是包含可用來設定 FairPlay 的方法。 如果您要的話，您可以新增 FairPlay 組態作為另一個選項。

    * 限制

        應用程式會設定原則的 JWT 權杖類型限制。

* 針對指定的資產並使用指定的串流原則名稱建立 StreamingLocator。 在此案例中，會使用預先定義的原則。 它會在 StreamingLocator 上設定兩個內容金鑰：AES-128 (信封) 和 CENC (PlayReady 與 Widevine)。  
    
    在 StreamingLocator 建立後，輸出資產即會發佈並可供用戶端用於播放。

    > [!NOTE]
    > 請確定您想要串流處理的來源 StreamingEndpoint 處於 [執行中] 狀態。

* 建立 Azure 媒體播放器的 URL，其中包含播放 PlayReady 加密內容所需的 DASH 資訊清單和 PlayReady 權杖。 範例會將權杖的到期時間設定為 1 小時。 

    您可以開啟瀏覽器並貼上產生的 URL，以啟動 Azure 媒體播放器示範頁面，其中已為您填入 URL 和權杖。  

    ![使用 DRM 保護](./media/protect-with-drm/playready_encrypted_url.png)

> [!NOTE]
> 您可以使用多種加密類型 (AES-128、PlayReady、Widevine、FairPlay) 來加密每項資產。 請參閱[串流通訊協定和加密類型](content-protection-overview.md#streaming-protocols-and-encryption-types)，以查看哪些組合可行。

本文中說明的範例會產生下列結果：

![使用 DRM 保護](./media/protect-with-drm/ams_player.png)

## <a name="prerequisites"></a>必要條件

需要有下列項目，才能完成教學課程。

* 請檢閱[內容保護概觀](content-protection-overview.md)文章。
* 檢閱[設計具有存取控制的多重 DRM 內容保護系統](design-multi-drm-system-with-access-control.md)
* 安裝 Visual Studio Code 或 Visual Studio
* 建立新的 Azure 媒體服務帳戶，如[此快速入門](create-account-cli-quickstart.md)所述。
* 藉由遵循[存取 API](access-api-cli-how-to.md) 以取得使用媒體服務 API 所需的認證
* 在應用程式組態檔 (appsettings.json) 中設定適當的值。

## <a name="download-code"></a>下載程式碼

使用以下命令將包含完整 .NET 範例 (在本文中討論) 的 GitHub 存放庫複製到您的電腦：

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
「使用 DRM 進行加密」範例位於 [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM) 資料夾中。

> [!NOTE]
> 此範例會在您每次執行應用程式時建立唯一的資源。 一般而言，您會重複使用現有的資源，例如轉換和原則 (如果現有資源具有必要的組態)。 

## <a name="start-using-media-services-apis-with-net-sdk"></a>開始搭配使用媒體服務 API 與 .NET SDK

若要開始搭配使用媒體服務 API 與 .NET，您需要建立 **AzureMediaServicesClient** 物件。 若要建立物件，您需要提供必要的認證，讓用戶端使用 Azure AD 連線至 Azure。 在您於本文一開始複製的程式碼中，**GetCredentialsAsync** 函式會根據本機設定檔中提供的認證建立 ServiceClientCredentials 物件。 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateMediaServicesClient)]

## <a name="create-an-output-asset"></a>建立輸出資產  

輸出[資產](https://docs.microsoft.com/rest/api/media/assets)會儲存您的編碼作業結果。  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateOutputAsset)]
 
## <a name="get-or-create-an-encoding-transform"></a>取得或建立編碼轉換

建立新的[轉換](https://docs.microsoft.com/rest/api/media/transforms)執行個體時，您需要指定想要其產生的輸出是什麼。 必要的參數是 **TransformOutput** 物件，如下列程式碼所示。 每個 **TransformOutput** 都會包含 **Preset (預設)**。 **Preset** 會描述影片和/或音訊處理作業的逐步指示，以產生所需的 **TransformOutput**。 本文中所述的範例會使用稱為 **AdaptiveStreaming** 的內建 Preset。 Preset 會根據輸入解析度和位元速率，將輸入影片編碼為自動產生的位元速率階梯 (位元速率-解析度配對)，並產生 H.264 影片與 AAC 音訊標準 (對應到每個 位元速率-解析度配對) 的 ISO MP4 檔案。 

在建立新的[轉換](https://docs.microsoft.com/rest/api/media/transforms)之前，您應該先使用 **Get** 方法檢查是否已有轉換存在，如後續程式碼所示。  在媒體服務 v3 中，如果實體不存在，對實體執行的 **Get** 方法會傳回 **null** (檢查名稱時不區分大小寫)。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#EnsureTransformExists)]

## <a name="submit-job"></a>提交作業

如同前面所述，[轉換](https://docs.microsoft.com/rest/api/media/transforms)物件是配方，而[作業](https://docs.microsoft.com/rest/api/media/jobs)則是實際要求媒體服務，將**轉換**套用至指定的輸入影片或音訊內容。 **作業**會指定輸入影片的位置、輸出的位置等資訊。

在本教學課程中，我們會根據直接從 [HTTP 來源 URL](job-input-from-http-how-to.md) 內嵌的檔案，建立作業的輸入。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>請等待作業完成

此作業需要一些時間來完成，而您可以選擇在完成時收到通知。 下列程式碼範例說明如何輪詢服務以取得[作業](https://docs.microsoft.com/rest/api/media/jobs)狀態。 對生產應用程式而言，輪詢不是建議的最佳做法，因為可能會發生延遲。 如果過度使用帳戶，輪詢可能會進行節流處理。 開發人員應改為使用事件方格。 請參閱[將事件路由至自訂 Web 端點](job-state-events-cli-how-to.md)。

**作業**通常會經歷下列狀態：**已排程**、**已排入佇列**、**正在處理**、**已完成** (最後一個狀態)。 如果作業發生錯誤，您會收到**錯誤**狀態。 如果正在取消作業，您會收到**正在取消**的狀態，以及完成時的**已取消**狀態。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#WaitForJobToFinish)]

## <a name="create-a-contentkeypolicy"></a>建立 ContentKeyPolicy

內容金鑰可提供資產的安全存取。 您必須建立內容金鑰原則，該原則會設定內容金鑰傳送給終端用戶端的方式。 內容金鑰會與 StreamingLocator 相關聯。 媒體服務也提供加密金鑰傳遞服務，可將加密金鑰和授權傳遞給已授權的使用者。 

您必須設定要使用指定的組態傳遞金鑰所須符合的內容金鑰原則需求 (限制)。 在此範例中，我們會設定下列組態和需求：

* 組態 

    設定 [PlayReady](playready-license-template-overview.md) 和 [Widevine](widevine-license-template-overview.md) 授權，使其可由媒體服務授權傳遞服務來傳遞。 雖然此範例應用程式不會設定 [FairPlay](fairplay-license-overview.md) 授權，但還是包含可用來設定 FairPlay 的方法。 您可以新增 FairPlay 組態作為另一個選項。

* 限制

    應用程式會設定原則的 JWT 權杖類型限制。

播放程式要求串流時，媒體服務便會使用指定的金鑰動態加密您的內容。 為了將串流解密，播放程式將向金鑰傳遞服務要求金鑰。 為了判斷使用者是否有權取得金鑰，服務會評估您為金鑰指定的內容金鑰原則。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="create-a-streaminglocator"></a>建立 StreamingLocator

編碼完成且設定好內容金鑰原則後，下一個步驟是要讓用戶端可播放輸出資產中的視訊。 您可透過兩個步驟即可完成： 

1. 建立 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators)
2. 建置用戶端可使用的串流 URL。 

建立 **StreamingLocator** 的程序稱為發佈。 根據預設，**StreamingLocator** 會在進行 API 呼叫後立即生效，而且會持續運作到遭到刪除為止 (除非您有設定選擇性的開始和結束時間)。 

建立 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 時，您必須指定需要的 **StreamingPolicyName**。 在本教學課程中，我們會使用其中一個預先定義的 StreamingPolicies，這會告訴 Azure 媒體服務如何發行內容以進行串流。 在此範例中，我們將 StreamingLocator.StreamingPolicyName 設定為 "Predefined_MultiDrmCencStreaming" 原則。 此原則會指出您要在定位器上產生並設定兩個內容金鑰 (信封和 CENC)。 因此，會套用信封、PlayReady 及 Widevine 加密 (會根據設定的 DRM 授權將金鑰傳遞至播放用戶端)。 如果您也想要使用 CBCS (FairPlay) 為串流加密，請使用 "Predefined_MultiDrmStreaming"。 

> [!IMPORTANT]
> 使用自訂的 [StreamingPolicy](https://docs.microsoft.com/rest/api/media/streamingpolicies) 時，您應該為媒體服務帳戶設計一組受限的這類原則，並且在需要相同的加密選項和通訊協定時，對 StreamingLocators 重新使用這些原則。 媒體服務帳戶有 StreamingPolicy 項目的數量配額。 不建議您對每個 StreamingLocator 建立新的 StreamingPolicy。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateStreamingLocator)]

## <a name="get-a-test-token"></a>取得測試權杖
        
在本教學課程中，我們會指定內容金鑰原則具有權杖限制。 權杖限制原則必須伴隨 Security Token Service (STS) 所發出的權杖。 媒體服務支援 [JSON Web 權杖](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_3) (JWT) 格式的權杖，這就是我們在範例中設定的項目。

ContentKeyIdentifierClaim 用於 ContentKeyPolicy，表示向金鑰傳遞服務提出的權杖必須具有 ContentKey 的識別碼。 在此範例中，我們在建立 StreamingLocator 時不指定內容金鑰，系統會為我們隨機建立一個金鑰。 為了產生測試權杖，我們必須取得 ContentKeyId，以將其放入 ContentKeyIdentifierClaim 宣告中。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetToken)]

## <a name="build-a-dash-streaming-url"></a>建置 DASH 串流 URL

建立了 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 之後，現在您就可以取得串流 URL。 若要建置 URL，您需要串連 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/streamingendpoints) 主機名稱和 **StreamingLocator** 路徑。 此範例會使用預設的 **StreamingEndpoint**。 初次建立媒體服務帳戶時，此預設的 **StreamingEndpoint** 會處於停止狀態，因此您需要呼叫 **Start**。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetMPEGStreamingUrl)]

## <a name="clean-up-resources-in-your-media-services-account"></a>清除媒體服務帳戶中的資源

一般而言，您應清除所有項目，只保留想要重複使用的物件 (您通常會重複使用轉換並且保存 StreamingLocators 等)。 如果您想要在實驗之後有乾淨的帳戶，您應刪除不打算重複使用的資源。  例如，下列程式碼會刪除作業。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CleanUp)]

## <a name="next-steps"></a>後續步驟

了解如何[使用 AES-128 保護](protect-with-aes128.md)
