---
title: 使用 AES Azure 媒體服務動態加密 | Microsoft Docs
description: 使用 Microsoft Azure 媒體服務來傳遞您使用 AES 128 位元加密金鑰加密的內容。 媒體服務也提供加密金鑰傳遞服務，將加密金鑰傳遞至授權的使用者。 本主題展示如何利用 AES-128 動態加密，以及使用金鑰傳遞服務。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: 8d071f64df0097b4029884a9efa84c6f2708fd44
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376596"
---
# <a name="use-aes-128-dynamic-encryption-and-the-key-delivery-service"></a>使用 AES-128 動態加密和金鑰傳遞服務

您可以利用 128 位元加密金鑰，使用媒體服務提供 HTTP 即時串流 (HLS)、MPEG-DASH，和透過 AES 加密的 Smooth Streaming。 媒體服務也提供加密金鑰傳遞服務，將加密金鑰傳遞至授權的使用者。 如果您需要媒體服務來加密資產，就需要建立加密金鑰與 StreamingLocator 的關聯，同時設定內容金鑰原則。 播放器要求串流時，媒體服務便會透過 AES 加密，使用指定的金鑰動態加密您的內容。 為了將串流解密，播放程式將向金鑰傳遞服務要求金鑰。 為了判斷使用者是否有權取得金鑰，服務會評估您為金鑰指定的內容金鑰原則。

本文是根據 [EncryptWithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES) 範例。 此範例示範如何建立編碼轉換，該轉換會使用內建預設的自適性位元速率編碼，並且直接從 [HTTP 來源 URL](job-input-from-http-how-to.md) 內嵌檔案。 然後使用 AES (ClearKey) 加密來發行輸出資產。 範例中的輸出是 Azure 媒體播放器 URL，包括播放內容所需的 DASH 資訊清單和 AES 權杖。 範例會將 JWT 權杖的到期時間設定為 1 小時。 您可以開啟瀏覽器並貼上產生的 URL，以啟動 Azure 媒體播放器示範頁面，其中已為您填入 URL 和權杖，格式如下：```https://ampdemo.azureedge.net/?url= {dash Manifest URL} &aes=true&aestoken=Bearer%3D{ JWT Token here}```。

> [!NOTE]
> 您可以使用多種加密類型 (AES-128、PlayReady、Widevine、FairPlay) 來加密每項資產。 請參閱[串流通訊協定和加密類型](content-protection-overview.md#streaming-protocols-and-encryption-types)，以查看哪些組合可行。

## <a name="prerequisites"></a>必要條件

需要有下列項目，才能完成教學課程。

* 請檢閱[內容保護概觀](content-protection-overview.md)文章。
* 安裝 Visual Studio Code 或 Visual Studio
* 建立新的 Azure 媒體服務帳戶，如[此快速入門](create-account-cli-quickstart.md)所述。
* 藉由遵循[存取 API](access-api-cli-how-to.md) 以取得使用媒體服務 API 所需的認證

## <a name="download-code"></a>下載程式碼

使用以下命令將包含完整 .NET 範例 (在本文中討論) 的 GitHub 存放庫複製到您的電腦：

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
「使用 AES-128 加密」範例位於 [EncryptWithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES) 資料夾中。

> [!NOTE]
> 此範例會在您每次執行應用程式時建立唯一的資源。 一般而言，您會重複使用現有的資源，例如轉換和原則 (如果現有資源具有必要的組態)。 

## <a name="start-using-media-services-apis-with-net-sdk"></a>開始搭配使用媒體服務 API 與 .NET SDK

若要開始搭配使用媒體服務 API 與 .NET，您需要建立 **AzureMediaServicesClient** 物件。 若要建立物件，您需要提供必要的認證，讓用戶端使用 Azure AD 連線至 Azure。 在您於本文一開始複製的程式碼中，**GetCredentialsAsync** 函式會根據本機設定檔中提供的認證建立 ServiceClientCredentials 物件。 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateMediaServicesClient)]

## <a name="create-an-output-asset"></a>建立輸出資產  

輸出[資產](https://docs.microsoft.com/rest/api/media/assets)會儲存您的編碼作業結果。  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateOutputAsset)]
 
## <a name="get-or-create-an-encoding-transform"></a>取得或建立編碼轉換

建立新的[轉換](https://docs.microsoft.com/rest/api/media/transforms)執行個體時，您需要指定想要其產生的輸出是什麼。 必要的參數是 **TransformOutput** 物件，如下列程式碼所示。 每個 **TransformOutput** 都會包含 **Preset (預設)**。 **Preset** 會描述影片和/或音訊處理作業的逐步指示，以產生所需的 **TransformOutput**。 本文中所述的範例會使用稱為 **AdaptiveStreaming** 的內建 Preset。 Preset 會根據輸入解析度和位元速率，將輸入影片編碼為自動產生的位元速率階梯 (位元速率-解析度配對)，並產生 H.264 影片與 AAC 音訊標準 (對應到每個 位元速率-解析度配對) 的 ISO MP4 檔案。 

在建立新的[轉換](https://docs.microsoft.com/rest/api/media/transforms)之前，您應該先使用 **Get** 方法檢查是否已有轉換存在，如後續程式碼所示。  在媒體服務 v3 中，如果實體不存在，對實體執行的 **Get** 方法會傳回 **null** (檢查名稱時不區分大小寫)。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#EnsureTransformExists)]

## <a name="submit-job"></a>提交作業

如同前面所述，[轉換](https://docs.microsoft.com/rest/api/media/transforms)物件是配方，而[作業](https://docs.microsoft.com/rest/api/media/jobs)則是實際要求媒體服務，將**轉換**套用至指定的輸入影片或音訊內容。 **作業**會指定輸入影片的位置、輸出的位置等資訊。

在本教學課程中，我們會根據直接從 [HTTP 來源 URL](job-input-from-http-how-to.md) 內嵌的檔案，建立作業的輸入。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>請等待作業完成

此作業需要一些時間來完成，而您可以選擇在完成時收到通知。 下列程式碼範例說明如何輪詢服務以取得[作業](https://docs.microsoft.com/rest/api/media/jobs)狀態。 對生產應用程式而言，輪詢不是建議的最佳做法，因為可能會發生延遲。 如果過度使用帳戶，輪詢可能會進行節流處理。 開發人員應改為使用事件方格。 請參閱[將事件路由至自訂 Web 端點](job-state-events-cli-how-to.md)。

**作業**通常會經歷下列狀態：**已排程**、**已排入佇列**、**正在處理**、**已完成** (最後一個狀態)。 如果作業發生錯誤，您會收到**錯誤**狀態。 如果正在取消作業，您會收到**正在取消**的狀態，以及完成時的**已取消**狀態。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#WaitForJobToFinish)]

## <a name="create-a-contentkeypolicy"></a>建立 ContentKeyPolicy

內容金鑰可提供資產的安全存取。 您必須建立 **ContentKeyPolicy**，其中會設定內容金鑰傳遞給終端用戶端的方式。 內容金鑰會與 **StreamingLocator** 相關聯。 媒體服務也提供加密金鑰傳遞服務，將加密金鑰傳遞至授權的使用者。 

播放器要求串流時，媒體服務會使用指定的金鑰動態加密您的內容 (在此案例中是使用 AES 加密)。為了將串流解密，播放程式將向金鑰傳遞服務要求金鑰。 為了判斷使用者是否有權取得金鑰，服務會評估您為金鑰指定的內容金鑰原則。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="create-a-streaminglocator"></a>建立 StreamingLocator

編碼完成且設定好內容金鑰原則後，下一個步驟是要讓用戶端可播放輸出資產中的視訊。 您可透過兩個步驟即可完成： 

1. 建立 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators)
2. 建置用戶端可使用的串流 URL。 

建立 **StreamingLocator** 的程序稱為發佈。 根據預設，**StreamingLocator** 會在進行 API 呼叫後立即生效，而且會持續運作到遭到刪除為止 (除非您有設定選擇性的開始和結束時間)。 

建立 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 時，您必須指定需要的 **StreamingPolicyName**。 在本教學課程中，我們會使用其中一個 PredefinedStreamingPolicies，這會告訴 Azure 媒體服務如何發行內容以進行串流。 在此範例中，會套用 AES Envelope 加密 (也稱為 ClearKey 加密，因為金鑰傳遞至播放用戶端是透過 HTTPS 而不是 DRM 授權)。

> [!IMPORTANT]
> 使用自訂的 [StreamingPolicy](https://docs.microsoft.com/rest/api/media/streamingpolicies) 時，您應該為媒體服務帳戶設計一組受限的這類原則，並且在需要相同的加密選項和通訊協定時，對 StreamingLocators 重新使用這些原則。 媒體服務帳戶有 StreamingPolicy 項目的數量配額。 不建議您對每個 StreamingLocator 建立新的 StreamingPolicy。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateStreamingLocator)]

## <a name="get-a-test-token"></a>取得測試權杖
        
在本教學課程中，我們會指定內容金鑰原則具有權杖限制。 權杖限制原則必須伴隨 Security Token Service (STS) 所發出的權杖。 媒體服務支援 [JSON Web 權杖](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_3) (JWT) 格式的權杖，這就是我們在範例中設定的項目。

ContentKeyIdentifierClaim 用於 ContentKeyPolicy，表示向「金鑰傳遞」服務提出的權杖必須具有 ContentKey 的識別碼。 在此範例中，我們在建立 StreamingLocator 時不指定內容金鑰，系統會為我們隨機建立一個金鑰。 為了產生測試權杖，我們必須取得 ContentKeyId，以將其放入 ContentKeyIdentifierClaim 宣告中。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetToken)]

## <a name="build-a-dash-streaming-url"></a>建置 DASH 串流 URL

建立了 [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) 之後，現在您就可以取得串流 URL。 若要建置 URL，您需要串連 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/streamingendpoints) 主機名稱和 **StreamingLocator** 路徑。 此範例會使用預設的 **StreamingEndpoint**。 初次建立媒體服務帳戶時，此預設的 **StreamingEndpoint** 會處於停止狀態，因此您需要呼叫 **Start**。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetMPEGStreamingUrl)]

## <a name="clean-up-resources-in-your-media-services-account"></a>清除媒體服務帳戶中的資源

一般而言，您應清除所有項目，只保留想要重複使用的物件 (您通常會重複使用轉換並且保存 StreamingLocators 等)。 如果您想要在實驗之後有乾淨的帳戶，您應刪除不打算重複使用的資源。  例如，下列程式碼會刪除作業。

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CleanUp)]

## <a name="next-steps"></a>後續步驟

查看如何[使用 DRM 保護](protect-with-drm.md)
