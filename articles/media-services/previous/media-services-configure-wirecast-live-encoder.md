---
title: 設定 Telestream Wirecast 編碼器來傳送單一位元速率的即時串流 | Microsoft Docs
description: '本主題示範如何設定 Wirecast 即時編碼器，藉此將單一位元速率的即時串流傳送到已啟用即時編碼的 AMS 通道。 '
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 0d2f1e81-51a6-4ca9-894a-6dfa51ce4c70
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 01/05/2017
ms.author: juliako;cenkdin;anilmur
ms.openlocfilehash: 6cc4c0b01511309766e48c3d671ee897e5d6f326
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669108"
---
# <a name="use-the-wirecast-encoder-to-send-a-single-bitrate-live-stream"></a>使用 Wirecast 編碼器來傳送單一位元速率的即時串流
> [!div class="op_single_selector"]
> * [Wirecast](media-services-configure-wirecast-live-encoder.md)
> * [Tricaster](media-services-configure-tricaster-live-encoder.md)
> * [FMLE](media-services-configure-fmle-live-encoder.md)
>
>

本文示範如何設定 [Telestream Wirecast](http://www.telestream.net/wirecast/overview.htm) 即時編碼器，藉此將單一位元速率的即時串流傳送到已啟用即時編碼的 AMS 通道。  如需詳細資訊，請參閱 [使用啟用的通道以 Azure 媒體服務執行即時編碼](media-services-manage-live-encoder-enabled-channels.md)。

本教學課程示範如何使用 Azure 媒體服務總管 (AMSE) 工具管理 Azure 媒體服務 (AMS)。 此工具只會在 Windows 電腦上執行。 如果您是用 Mac 或 Linux，請使用 Azure 入口網站建立[通道](media-services-portal-creating-live-encoder-enabled-channel.md#create-a-channel)和[程式](media-services-portal-creating-live-encoder-enabled-channel.md)。

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

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast1.png)

2. 指定通道名稱，描述欄位為選填。 在 [頻道設定] 下方，針對 [即時編碼] 選項選取 [標準]，並將 [輸入通訊協定] 設定為 [RTMP]。 您可以將所有其他設定保留現狀。

    請確認已選取 [ **立即啟動新頻道** ]。

3. 按一下 [ **建立頻道**]。

   ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast2.png)

> [!NOTE]
> 通道約需 20 分鐘的時間即可啟動。
>
>

當頻道啟動時，您可以[設定編碼器](media-services-configure-wirecast-live-encoder.md#configure_wirecast_rtmp)。

> [!IMPORTANT]
> 只要通道進入就緒狀態，就會開始計費。 如需詳細資訊，請參閱 [通道的狀態](media-services-manage-live-encoder-enabled-channels.md#states)。
>
>

## <a name="a-idconfigurewirecastrtmp-aconfigure-the-telestream-wirecast-encoder"></a><a id="configure_wirecast_rtmp" /a>設定 Telestream Wirecast 編碼器
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
1. 在使用的機器上開啟 Telestream Wirecast 應用程式，並設定 RTMP 串流。
2. 瀏覽至 [輸出] 索引標籤，並選取 [輸出設定...] 以設定輸出。

    請確定 [輸出目的地] 已設為 [RTMP Server]。
3. 按一下 [確定]。
4. 在 [設定] 頁面上，將 [目的地] 欄位設為 [Azure 媒體服務]。

    編碼設定檔已預先選取為 [Azure H.264 720p 16:9 (1280x720)] 。 若要自訂這些設定，請選取下拉式清單右邊的齒輪圖示，然後選擇 [新增預設]。

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast3.png)
5. 設定編碼器預設。

    為預設命名，並檢查下列建議的設定：

    **影片**

   * 編碼器：MainConcept H.264
   * 每秒畫面格數：30
   * 平均位元速率：5000 kbit/秒 (可視網路限制加以調整)
   * 設定檔：主要
   * 畫面間隔：60 個畫面

    **音訊**

   * 目標位元速率：192 kbit/秒
   * 取樣速率：44.100 kHz

     ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast4.png)
6. 按下 [儲存] 。

    [編碼] 欄位現在會有可供選取的新建立設定檔。

    請確定已選取新的設定檔。
7. 取得通道的輸入 URL，才能將它指派給 Wirecast 的 **RTMP 端點**。

    瀏覽回到 AMSE 工具，並檢查通道的完成狀態。 狀態從 [啟動中] 變更為 [執行中] 後，您便可取得輸入 URL。

    頻道執行時，以滑鼠右鍵按一下頻道名稱，向下瀏覽讓滑鼠游標停留在 [複製輸入 URL 到剪貼簿]，然後選取 [主要輸入 URL]。  

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast6.png)
8. 在 Wirecast [輸出設定] 視窗中，於輸出區段的 [位址] 欄位中貼上這項資訊，並指派串流名稱。

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast5.png)

1. 選取 [確定] 。
2. 在 **Wirecast** 主畫面上，確認視訊和音訊的輸入來源已就緒，然後按下左上角的 [資料流]。

   ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast7.png)

> [!IMPORTANT]
> 在您按一下 [資料流] 之前，**必須**確保通道已就緒。
> 此外，請務必不要讓通道在沒有輸入比重摘要的情況下，處於就緒狀態超過 15 分鐘。
>
>

## <a name="test-playback"></a>測試播放

瀏覽至 AMSE 工具，然後以滑鼠右鍵按一下要測試的通道。 在功能表中，將滑鼠游標停留在 [播放預覽]，並選取 [使用 Azure 媒體播放器]。  

    ![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast8.png)

如果播放器中出現串流，則編碼器已妥善設定為連接到 AMS。

如果收到錯誤，則必須重設通道，且編碼器設定需要調整。 如需指引，請參閱[疑難排解](media-services-troubleshooting-live-streaming.md)一文。  

## <a name="create-a-program"></a>建立程式
1. 一旦確認通道播放沒問題後，請建立程式。 在 AMSE 工具的 [Live] 索引標籤下，於程式區域內按一下滑鼠右鍵，並選取 [建立新的程式]。  

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast9.png)
2. 為程式命名，並視需要調整 [封存時間長度] \(預設為 4 小時)。 您也可以指定儲存體位置，或保留為預設值。  
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
