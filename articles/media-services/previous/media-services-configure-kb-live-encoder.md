---
title: 設定 Haivision KB 編碼器來將單一位元速率即時串流傳送到 Azure | Microsoft Docs
description: 本主題示範如何設定 Haivision KB 即時編碼器，以將單一位元速率串流傳送到已啟用即時編碼的 AMS 通道。
services: media-services
documentationcenter: ''
author: dbgeorge
manager: vsood
editor: ''
ms.assetid: 0d2f1e81-51a6-4ca9-894a-6dfa51ce4c70
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 02/02/2018
ms.author: juliako;dbgeorge
ms.openlocfilehash: a36e12080cbbcb1a98bf786a6634959362cb52a7
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666444"
---
# <a name="use-the-haivision-kb-live-encoder-to-send-a-single-bitrate-live-stream"></a>使用 Haivision KB 即時編碼器來傳送單一位元速率即時串流
> [!div class="op_single_selector"]
> * [FMLE](media-services-configure-fmle-live-encoder.md)
> * [Haivision](media-services-configure-kb-live-encoder.md)
> * [Tricaster](media-services-configure-tricaster-live-encoder.md)
> * [Wirecast](media-services-configure-wirecast-live-encoder.md)

此主題說明如何設定 [Havision KB 即時編碼器](https://www.haivision.com/products/kb-series/)來傳送單一位元速率串流到已啟用即時編碼的 AMS 通道。 如需詳細資訊，請參閱 [使用啟用的通道以 Azure 媒體服務執行即時編碼](media-services-manage-live-encoder-enabled-channels.md)。

本教學課程示範如何使用 Azure 媒體服務總管 (AMSE) 工具管理 Azure 媒體服務 (AMS)。 此工具只會在 Windows 電腦上執行。 如果您是用 Mac 或 Linux，請使用 Azure 入口網站建立[通道](media-services-portal-creating-live-encoder-enabled-channel.md#create-a-channel)和[程式](media-services-portal-creating-live-encoder-enabled-channel.md)。

## <a name="prerequisites"></a>必要條件
*   存取執行 SW v5.01 或更新版本的 Haivision KB 編碼器。
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
[Haivision](./media/media-services-configure-kb-live-encoder/channel.png)
2. 指定通道名稱，描述欄位為選填。 在 [頻道設定] 下方，針對 [即時編碼] 選項選取 [標準]，並將 [輸入通訊協定] 設定為 [RTMP]。 您可以將所有其他設定保留現狀。 請確認已選取 [ **立即啟動新頻道** ]。
3. 按一下 [ **建立頻道**]。
[Haivision](./media/media-services-configure-kb-live-encoder/livechannel.png)

> [!NOTE]
> 通道約需 20 分鐘的時間即可啟動。

## <a name="configure-the-haivision-kb-encoder"></a>設定 Haivision KB 編碼器
在本教學課程中，我們會使用下列輸出設定。 本章節的其餘部分將詳細說明組態步驟。

視訊：
-   轉碼器：H.264
-   設定檔：高 (層級 4.0)
-   位元速率：5000 kbps
-   主要畫面格：2 秒 (60 秒)
-   畫面播放速率：30

音訊：
-   轉碼器：AAC (LC)
-   位元速率：192 kbps
-   取樣速率：44.1 kHz

## <a name="configuration-steps"></a>組態步驟
1.  登入 Haivision KB 使用者介面。
2.  按一下通道控制中心中的**功能表按鈕**，然後選取 [Add Channel] \(新增通道\)  
    ![螢幕擷取畫面 2017-08-14 at 9.15.09 AM.png](./media/media-services-configure-kb-live-encoder/step2.png)
3.  在 [名稱] 欄位中輸入**通道名稱**，然後按一下 [下一步]。  
    ![螢幕擷取畫面 2017-08-14 at 9.19.07 AM.png](./media/media-services-configure-kb-live-encoder/step3.png)
4.  從 [輸入來源] 下拉式清單選取 [Channel Input Source] \(通道輸入來源\)。
    ![螢幕擷取畫面 2017-08-14 at 9.20.44 AM.png](./media/media-services-configure-kb-live-encoder/step4.png)
5.  從 [Encoder Template] \(編碼器範本\) 下拉式清單選擇 [H264-720-AAC-192]，然後按一下下一步。
    ![螢幕擷取畫面 2017-08-14 at 9.23.15 AM.png](./media/media-services-configure-kb-live-encoder/step5.png)
6.  從 [Select New Output] \(選取新輸出\) 下拉式清單選擇 [RTMP]，然後按一下 [下一步]。  
    ![螢幕擷取畫面 2017-08-14 at 9.27.51 AM.png](./media/media-services-configure-kb-live-encoder/step6.png)
7.  從 [Channel Output] \(通道輸出\) 視窗，填入 Azure 串流資料。 從 [伺服器] 區域中的初始通道設定貼上 **RTMP** 連結。 在 [Output Name] \(輸出名稱\) 區域中，輸入通道名稱。 在 [Stream Name Template] \(串流名稱範本\) 區域中，使用範本 RTMPStreamName_%video_bitrate% 來為串流命名。
    ![螢幕擷取畫面 2017-08-14 at 9.33.17 AM.png](./media/media-services-configure-kb-live-encoder/step7.png)
8.  按一下 [下一步]，然後按一下 [完成]。
9.  按一下 [Play Button] \(播放按鈕\) 以啟動編碼器通道。  
    ![Haivision KB.png](./media/media-services-configure-kb-live-encoder/step9.png)

## <a name="test-playback"></a>測試播放
瀏覽至 AMSE 工具，然後以滑鼠右鍵按一下要測試的通道。 在功能表中，將滑鼠游標停留在 [播放預覽] \(Playback the Preview\) 上方，並選取 [使用 Azure 媒體播放器] \(with Azure Media Player\)。

如果播放器中出現串流，則編碼器已妥善設定為連接到 AMS。

如果收到錯誤，則必須重設通道，且編碼器設定需要調整。 如需 指導方針，請參閱疑難排解一文。

## <a name="create-a-program"></a>建立程式
1.  一旦確認通道播放沒問題後，請建立程式。 在 AMSE 工具的 [Live] \(即時\) 索引標籤下，於程式區域內按一下滑鼠右鍵，並選取 [Create New Program] \(建立新程式\)。
[Haivision](./media/media-services-configure-kb-live-encoder/program.png)
1.  為程式命名，並視需要調整 [Archive Window Length] \(封存時間長度\) (預設為 4 小時)。 您也可以指定儲存體位置，或保留為預設值。
2.  選取 [Start the Program now] \(現在啟動程式\) 方塊。
3.  按一下 [Create Program] \(建立程式\)。
4.  一旦程式開始執行，請在程式上按一下滑鼠右鍵，並瀏覽至 [Play back the program(s)] \(播放程式\)，然後選取 [with Azure Media Player] \(使用 Azure 媒體播放器\) 確認播放。
5.  一經確認後，再次於該程式上按一下滑鼠右鍵，並選取 [Copy the Output URL to Clipboard] \(複製輸出 URL 到剪貼簿\) (或從程式資訊和設定功能表選項擷取此資訊)。

串流現在已經可以內嵌於播放程式中，或散發給某個對象，以供即時檢視。

> [!NOTE]
> 建立程式時所使用的時間會比建立通道時少。