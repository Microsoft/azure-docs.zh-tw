---
title: 設定 FMLE 編碼器來傳送單一位元速率的即時串流 | Microsoft Docs
description: 本主題示範如何設定 Flash Media Live Encoder (FMLE) 編碼器，藉此將單一位元速率的即時串流傳送到已啟用即時編碼的 AMS 通道。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 3113f333-517a-47a1-a1b3-57e200c6b2a2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 01/05/2017
ms.author: juliako;cenkdin;anilmur
ms.openlocfilehash: 1a7cbd19b89663ab874fc5a7a86587e292b86f81
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665880"
---
# <a name="use-the-fmle-encoder-to-send-a-single-bitrate-live-stream"></a>使用 FMLE 編碼器來傳送單一位元速率的即時串流
> [!div class="op_single_selector"]
> * [FMLE](media-services-configure-fmle-live-encoder.md)
> * [Tricaster](media-services-configure-tricaster-live-encoder.md)
> * [Wirecast](media-services-configure-wirecast-live-encoder.md)
>
>

本文示範如何設定 [Flash Media Live Encoder](http://www.adobe.com/products/flash-media-encoder.html) (FMLE) 編碼器，藉此將單一位元速率的即時串流傳送到已啟用即時編碼的 AMS 通道。 如需詳細資訊，請參閱 [使用啟用的通道以 Azure 媒體服務執行即時編碼](media-services-manage-live-encoder-enabled-channels.md)。

本教學課程示範如何使用 Azure 媒體服務總管 (AMSE) 工具管理 Azure 媒體服務 (AMS)。 此工具只會在 Windows 電腦上執行。 如果您是用 Mac 或 Linux，請使用 Azure 入口網站建立[通道](media-services-portal-creating-live-encoder-enabled-channel.md#create-a-channel)和[程式](media-services-portal-creating-live-encoder-enabled-channel.md)。

本教學課程說明如何使用 AAC。 不過，依預設 FMLE 不支援 AAC。 您必須購買 AAC 編碼的外掛程式，例如從 MainConcept 購買 [AAC 外掛程式](http://www.mainconcept.com/products/plug-ins/plug-ins-for-adobe/aac-encoder-fmle.html)

## <a name="prerequisites"></a>必要條件
* [建立 Azure 媒體服務帳戶](media-services-portal-create-account.md)
* 確定有執行中的「串流端點」。 如需詳細資訊，請參閱 [在媒體服務帳戶中管理串流端點](media-services-portal-manage-streaming-endpoints.md)
* 安裝最新版的 [AMSE](https://github.com/Azure/Azure-Media-Services-Explorer) 工具。
* 啟動工具並連接到您的 AMS 帳戶。

## <a name="tips"></a>祕訣
* 請盡可能使用實體的有線網際網路連線。
* 判斷頻寬需求的一項法則是將串流位元速率加倍。 雖然這不是強制性需求，卻有助於減輕網路阻塞的影響。
* 使用軟體型編碼器時，請關閉任何不必要的程式。

## <a name="create-a-channel"></a>建立通道
1. 在 AMSE 工具中，瀏覽至 [Live] 索引標籤，然後在通道區域內按一下滑鼠右鍵。 從功能表選取 [建立通道...]  。

    ![FMLE](./media/media-services-fmle-live-encoder/media-services-fmle1.png)

2. 指定通道名稱，描述欄位為選填。 在 [頻道設定] 下方，針對 [即時編碼] 選項選取 [標準]，並將 [輸入通訊協定] 設定為 [RTMP]。 您可以將所有其他設定保留現狀。

    請確認已選取 [ **立即啟動新頻道** ]。

3. 按一下 [ **建立頻道**]。

   ![FMLE](./media/media-services-fmle-live-encoder/media-services-fmle2.png)

> [!NOTE]
> 通道約需 20 分鐘的時間即可啟動。
>
>

當頻道啟動時，您可以[設定編碼器](media-services-configure-fmle-live-encoder.md#configure_fmle_rtmp)。

> [!IMPORTANT]
> 請注意，只要通道進入就緒狀態，就會開始計費。 如需詳細資訊，請參閱 [頻道的狀態](media-services-manage-live-encoder-enabled-channels.md#states)。
>
>

## <a id=configure_fmle_rtmp></a>設定 FMLE 編碼器
在本教學課程中，我們會使用下列輸出設定。 本章節的其餘部分將詳細說明組態步驟。

**視訊**：

* 轉碼器：H.264
* 設定檔：高 (層級 4.0)
* 位元速率：5000 kbps
* 主要畫面格：2 秒 (60 秒)
* 畫面播放速率：30

**音訊**：

* 轉碼器：AAC (LC)
* 位元速率：192 kbps
* 取樣速率：44.1 kHz

### <a name="configuration-steps"></a>組態步驟
1. 瀏覽至所使用之機器上 Flash Media Live Encoder (FMLE) 的介面。

    介面是各種設定的主頁面。 請記下以下的建議設定，以開始透過 FMLE 使用串流。

   * 格式：H.264 畫面播放速率：30.00
   * 輸入大小：1280 x 720
   * 位元速率：5000 Kbps (可視網路限制加以調整)  

     ![FMLE](./media/media-services-fmle-live-encoder/media-services-fmle3.png)

     當使用交錯式來源時，請勾選 [非交錯] 選項
2. 選取 [格式] 旁邊的扳手圖示，這些額外的設定應該是：

   * 設定檔：主要
   * 層級：4.0
   * 主要畫面格頻率：2 秒

     ![FMLE](./media/media-services-fmle-live-encoder/media-services-fmle4.png)
3. 設定下列重要的音訊設定：

   * 格式：AAC
   * 取樣速率：44100 Hz
   * 位元速率：192 kbps

     ![FMLE](./media/media-services-fmle-live-encoder/media-services-fmle5.png)
4. 取得頻道的輸入 URL，以將其指派給 FMLE 的 **RTMP 端點**。

    瀏覽回到 AMSE 工具，並檢查通道的完成狀態。 狀態從 [啟動中] 變更為 [執行中] 後，您便可取得輸入 URL。

    頻道執行時，以滑鼠右鍵按一下頻道名稱，向下瀏覽讓滑鼠游標停留在 [複製輸入 URL 到剪貼簿]，然後選取 [主要輸入 URL]。  

    ![FMLE](./media/media-services-fmle-live-encoder/media-services-fmle6.png)
5. 在輸出區段裡的 [ **FMS URL** ] 欄位中貼上這項資訊，並指派串流名稱。

    ![FMLE](./media/media-services-fmle-live-encoder/media-services-fmle7.png)

    如需額外的備援，請用次要輸入 URL 重複這些步驟。
6. 選取 [ **連接**]。

> [!IMPORTANT]
> 在您按一下 [連接] 之前，**必須**先確保頻道已就緒。
> 此外，請務必不要讓通道在沒有輸入比重摘要的情況下，處於就緒狀態超過 15 分鐘。
>
>

## <a name="test-playback"></a>測試播放

瀏覽至 AMSE 工具，然後以滑鼠右鍵按一下要測試的通道。 在功能表中，將滑鼠游標停留在 [播放預覽]，並選取 [使用 Azure 媒體播放器]。  

    ![fmle](./media/media-services-fmle-live-encoder/media-services-fmle8.png)

如果播放器中出現串流，則編碼器已妥善設定為連接到 AMS。

如果收到錯誤，則必須重設通道，且編碼器設定需要調整。 如需指引，請參閱[疑難排解](media-services-troubleshooting-live-streaming.md)一文。  

## <a name="create-a-program"></a>建立程式
1. 一旦確認通道播放沒問題後，請建立程式。 在 AMSE 工具的 [Live] 索引標籤下，於程式區域內按一下滑鼠右鍵，並選取 [建立新的程式]。  

    ![FMLE](./media/media-services-fmle-live-encoder/media-services-fmle9.png)
2. 為程式命名，並視需要調整 **封存時間長度** (預設為 4 小時)。 您也可以指定儲存體位置，或保留為預設值。  
3. 勾選 [現在啟動程式]  方塊。
4. 按一下 [建立程式] 。  

    >[!NOTE]
    >建立程式時所使用的時間會比建立通道時少。
        
5. 一旦程式開始執行，請在程式上按一下滑鼠右鍵，並瀏覽至 [播放程式]，然後選取 [使用 Azure 媒體播放器] 確認播放。  
6. 一經確認後，再次於該程式上按一下滑鼠右鍵，並選取 [複製輸出 URL 到剪貼簿] \(或從 [程式資訊和設定] 功能表選項擷取這項資訊)。

串流現在已經可以內嵌於播放程式中，或散發給某個對象，以供即時檢視。  

## <a name="troubleshooting"></a>疑難排解
如需指引，請參閱[疑難排解](media-services-troubleshooting-live-streaming.md)一文。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
