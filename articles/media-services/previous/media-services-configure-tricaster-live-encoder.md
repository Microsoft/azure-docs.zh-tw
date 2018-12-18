---
title: 設定 NewTek TriCaster 編碼器來傳送單一位元速率的即時串流 | Microsoft Docs
description: 本主題示範如何設定 Tricaster 即時編碼器，藉此將單一位元速率的串流傳送到已啟用即時編碼的 AMS 通道。
services: media-services
documentationcenter: ''
author: cenkdin
manager: cfowler
editor: ''
ms.assetid: 8973181a-3059-471a-a6bb-ccda7d3ff297
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 01/05/2017
ms.author: juliako;cenkd;anilmur
ms.openlocfilehash: 8084f32ac8cc2184d93796468ad66fb73398e876
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783777"
---
# <a name="use-the-newtek-tricaster-encoder-to-send-a-single-bitrate-live-stream"></a>使用 NewTek TriCaster 編碼器來傳送單一位元速率的即時串流
> [!div class="op_single_selector"]
> * [Tricaster](media-services-configure-tricaster-live-encoder.md)
> * [Elemental Live](media-services-configure-elemental-live-encoder.md)
> * [Wirecast](media-services-configure-wirecast-live-encoder.md)
> * [FMLE](media-services-configure-fmle-live-encoder.md)
>
>

本文示範如何設定 [NewTek TriCaster](http://newtek.com/products/tricaster-40.html) 即時編碼器，藉此將單一位元速率的即時串流傳送到已啟用即時編碼的 AMS 通道。 如需詳細資訊，請參閱 [使用啟用的通道以 Azure 媒體服務執行即時編碼](media-services-manage-live-encoder-enabled-channels.md)。

本教學課程示範如何使用 Azure 媒體服務總管 (AMSE) 工具管理 Azure 媒體服務 (AMS)。 此工具只會在 Windows 電腦上執行。 如果您是用 Mac 或 Linux，請使用 Azure 入口網站建立[通道](media-services-portal-creating-live-encoder-enabled-channel.md#create-a-channel)和[程式](media-services-portal-creating-live-encoder-enabled-channel.md)。

> [!NOTE]
> 使用 Tricaster 將分佈摘要傳送到針對即時編碼啟用的 AMS 通道時，如果您使用 Tricaster 的某些功能 (例如，在摘要之間快速切割或切換靜態圖像)，您的即時事件可能會出現視訊/音訊干擾。 AMS 小組正著手於修正這些問題，在解決之前，不建議使用這些功能。
>
>

## <a name="prerequisites"></a>先決條件
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

    ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster1.png)

2. 指定通道名稱，描述欄位為選填。 在 [頻道設定] 下方，針對 [即時編碼] 選項選取 [標準]，並將 [輸入通訊協定] 設定為 [RTMP]。 您可以將所有其他設定保留現狀。

    請確認已選取 [ **立即啟動新頻道** ]。

3. 按一下 [ **建立頻道**]。

   ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster2.png)

> [!NOTE]
> 通道約需 20 分鐘的時間即可啟動。
>
>

當頻道啟動時，您可以[設定編碼器](media-services-configure-tricaster-live-encoder.md#configure_tricaster_rtmp)。

> [!IMPORTANT]
> 只要通道進入就緒狀態，就會開始計費。 如需詳細資訊，請參閱 [通道的狀態](media-services-manage-live-encoder-enabled-channels.md#states)。
>
>

## <a id=configure_tricaster_rtmp></a>設定 NewTek TriCaster 編碼器
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
1. 依據使用的視訊輸入來源建立新的 **NewTek TriCaster** 專案。
2. 進入該專案後，請尋找 [資料流]  按鈕，然後按一下該按鈕旁邊的齒輪圖示，以存取串流組態功能表。

    ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster3.png)
3. 開啟功能表後，按一下 [連接] 標題下的 [新增]  。 當系統提示您選取連接類型時，請選取 [Adobe Flash] 。

    ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster4.png)
4. 按一下 [確定]。
5. 現在按一下 [串流設定檔] 下方的下拉式清單箭頭，並瀏覽至 [瀏覽]，即可匯入 FMLE 設定檔。

    ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster5.png)
6. 瀏覽至儲存已設定之 FMLE 設定檔的位置。
7. 選取它，然後按下 [確定] 。

    上傳設定檔後，請繼續進行下一個步驟。
8. 取得通道的輸入 URL，才能將它指派給 Tricaster 的 **RTMP 端點**。

    瀏覽回到 AMSE 工具，並檢查通道的完成狀態。 狀態從 [啟動中] 變更為 [執行中] 後，您便可取得輸入 URL。

    頻道執行時，以滑鼠右鍵按一下頻道名稱，向下瀏覽讓滑鼠游標停留在 [複製輸入 URL 到剪貼簿]，然後選取 [主要輸入 URL]。  

    ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster6.png)
9. 在 Tricaster 專案中，於 [Flash Server] 下的 [位置] 欄位貼上這項資訊。 此外也請在 [資料流 ID]  欄位中指派串流名稱。

    如果串流資訊已新增至 FMLE 設定檔，您也可以按一下 [匯入設定]、瀏覽到已儲存的 FMLE 設定檔，並按一下 [確定]，來匯入至此區段。 相關的 [Flash Server] 欄位應該填入 FMLE 的資訊。

    ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster7.png)
10. 完成後，按一下畫面底部的 [確定]  。 輸入至 Tricaster 的視訊和音訊已就緒時，請按一下 [資料流]  按鈕，開始串流至 AMS。

     ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster11.png)

> [!IMPORTANT]
> 在您按一下 [資料流] 之前，**必須**確保通道已就緒。
> 此外，請務必不要讓通道在沒有輸入比重摘要的情況下，處於就緒狀態超過 15 分鐘。
>
>

## <a name="test-playback"></a>測試播放
瀏覽至 AMSE 工具，然後以滑鼠右鍵按一下要測試的通道。 在功能表中，將滑鼠游標停留在 [播放預覽]，並選取 [使用 Azure 媒體播放器]。  

    ![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster8.png)

如果播放器中出現串流，則編碼器已妥善設定為連接到 AMS。

如果收到錯誤，則必須重設通道，且編碼器設定需要調整。 如需指引，請參閱[疑難排解](media-services-troubleshooting-live-streaming.md)一文。  

## <a name="create-a-program"></a>建立程式
1. 一旦確認通道播放沒問題後，請建立程式。 在 AMSE 工具的 [Live] 索引標籤下，於程式區域內按一下滑鼠右鍵，並選取 [建立新的程式]。  

    ![Tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster9.png)
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

## <a name="next-step"></a>後續步驟
檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
