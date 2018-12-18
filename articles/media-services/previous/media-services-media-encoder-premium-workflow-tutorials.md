---
title: 進階媒體編碼器 Premium 工作流程教學課程
description: 本文件包含的逐步解說可示範如何使用媒體編碼器 Premium 工作流程執行進階的工作，以及如何使用工作流程設計工具建立複雜的工作流程。
services: media-services
documentationcenter: ''
author: xstof
manager: cfowler
editor: ''
ms.assetid: 1ba52865-b4a8-4ca0-ac96-920d55b9d15b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2017
ms.author: christoc;xpouyat;juliako
ms.openlocfilehash: 3cba7a6a8bf6fbe199664b738b68ceca3a7746f8
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783947"
---
# <a name="advanced-media-encoder-premium-workflow-tutorials"></a>進階媒體編碼器 Premium 工作流程教學課程
## <a name="overview"></a>概觀
本文件包含的逐步解說可示範如何使用**工作流程設計工具**自訂工作流程。 您可以在[這裡](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows/PremiumEncoderWorkflowSamples)尋找實際的工作流程檔案。  

## <a name="toc"></a>目錄
本文涵蓋下列主題：

* [將 MXF 編碼為單一位元速率 MP4](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4)
  * [開始新的工作流程](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_start_new)
  * [使用媒體檔案輸入](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_file_input)
  * [檢查媒體串流](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_streams)
  * [為產生的 MP4 檔案加入視訊編碼器](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_file_generation)
  * [對音訊串流編碼](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_audio)
  * [將音訊和視訊串流多工處理為 MP4 容器](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_audio_and_fideo)
  * [寫入 MP4 檔案](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_writing_mp4)
  * [從輸出檔案建立媒體服務資產](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_asset_from_output)
  * [在本機測試完成的工作流程](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_test)
* [將 MXF 編碼為多位元速率 MP4 - 動態封裝已啟用](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging)
  * [加入一或多個其他的 MP4 輸出](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging_more_outputs)
  * [設定檔案輸出名稱](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging_conf_output_names)
  * [加入個別的曲目](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging_audio_tracks)
  * [新增 "ISM" SMIL 檔案](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging_ism_file)
* [將 MXF 編碼為多位元速率 MP4 - 增強的藍圖](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to__multibitrate_MP4)
  * [要增強的工作流程概觀](#workflow-overview-to-enhance)
  * [檔案命名慣例](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to__multibitrate_MP4_file_naming)
  * [發佈元件屬性至工作流程根目錄](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to__multibitrate_MP4_publishing)
  * [讓產生的輸出檔案名稱依賴發佈的屬性值](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to__multibitrate_MP4_output_files)
* [加入縮圖至多位元速率 MP4 輸出](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4)
  * [要加入縮圖的目標工作流程概觀](#workflow-overview-to-add-thumbnails-to)
  * [加入 JPG 編碼](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4__with_jpg)
  * [處理色彩空間轉換](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4_color_space)
  * [寫入縮圖](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4_writing_thumbnails)
  * [偵測工作流程中的錯誤](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4_errors)
  * [工作流程完成](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4_finish)
* [多位元速率 MP4 輸出以時間為基礎的修剪](media-services-media-encoder-premium-workflow-tutorials.md#time_based_trim)
  * [要開始加入修剪的目標工作流程概觀](media-services-media-encoder-premium-workflow-tutorials.md#time_based_trim_start)
  * [使用串流修剪器](media-services-media-encoder-premium-workflow-tutorials.md#time_based_trim_use_stream_trimmer)
  * [工作流程完成](media-services-media-encoder-premium-workflow-tutorials.md#time_based_trim_finish)
* [推出指令碼元件](media-services-media-encoder-premium-workflow-tutorials.md#scripting)
  * [工作流程內的指令碼：Hello World](media-services-media-encoder-premium-workflow-tutorials.md#scripting_hello_world)
* [多位元速率 MP4 輸出以畫面格為基礎的修剪](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim)
  * [要開始加入修剪的目標藍圖概觀](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim_start)
  * [使用剪輯清單 XML](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim_clip_list)
  * [透過指令碼元件修改剪輯清單](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim_modify_clip_list)
  * [加入 ClippingEnabled 便利屬性](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim_clippingenabled_prop)

## <a id="MXF_to_MP4"></a>將 MXF 編碼為單一位元速率 MP4
本節示範如何使用來自 .MXF 輸入檔案 AAC-HE 編碼的音訊來建立單一位元速率 .MP4 檔案。

### <a id="MXF_to_MP4_start_new"></a>開始新的工作流程
開啟「工作流程設計工具」，然後選取 [檔案] > [新增工作區] > [轉碼藍圖]

新的工作流程會顯示三個元素︰

* 主要來源檔案
* 剪輯清單 XML
* 輸出檔案/資產  

![新編碼工作流程](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-transcode-blueprint.png)

*新編碼工作流程*

### <a id="MXF_to_MP4_with_file_input"></a>使用媒體檔案輸入
為了接受輸入媒體檔案，您會從新增媒體檔案輸入元件開始。 若要將元件加入至工作流程，請在 [儲存機制] 搜尋方塊中尋找它，然後將所需的項目拖曳至設計工具窗格。 請對「媒體檔案輸入」重複此動作，並將 [主要來源檔案] 元件從 [媒體檔案輸入] 連線至 [檔案名稱] 輸入接點。

![連接的媒體檔案輸入](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-file-input.png)

*連接的媒體檔案輸入*

一開始，識別在設計自訂工作流程時要使用的適當範例檔案。 若要這樣做，請按一下設計工具窗格背景，並在右手邊屬性窗格中尋找 [主要來源檔案] 屬性。 按一下資料夾圖示，然後選取所需的檔案來測試工作流程。 媒體檔案輸入元件會檢查檔案，並填入其輸出接點，以反映它檢查的範例檔案詳細資料。

![填入媒體檔案輸入](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-populated-media-file-input.png)

*填入媒體檔案輸入*

現在已填入輸入，下一個步驟是設定輸出編碼設定。 類似於設定主要來源檔案的方式，現在設定輸出資料夾變數的屬性，就在其下方。

![設定輸入和輸出屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-configured-io-properties.png)

*設定輸入和輸出屬性*

### <a id="MXF_to_MP4_streams"></a>檢查媒體串流
通常您想要知道經過工作流程之後串流的外觀。 若要在工作流程中的任何一點檢查串流，只要按一下任何元件上的輸出或輸入接點。 在此情況下，請嘗試從 [媒體檔案輸入] 按一下 [未壓縮的影片] 輸出接點。 對話方塊隨即開啟，讓您檢查輸出影片。

![檢查未壓縮的視訊輸出接點](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-inspecting-uncompressed-video-output.png)

*檢查未壓縮的視訊輸出接點*

在此案例中，它會顯示影片包含要對一段接近 2 分鐘的影片，以 4:2:2 取樣每秒 24 個畫面格處理 1920x1080 輸入。

### <a id="MXF_to_MP4_file_generation"></a>為產生的 MP4 檔案加入視訊編碼器
現在，[未壓縮的視訊] 和多個 [未壓縮的音訊] 輸出接點可供用於媒體檔案輸入。 為了對輸入影片編碼，必須將編碼元件新增至工作流程 - 在此情況下用於產生 .MP4 檔案。

若要將視訊串流編碼成 H.264，請將 AVC 視訊編碼器元件加入至設計工具介面。 此元件會將未壓縮的視訊串流做為輸入，並在其輸出接點上提供 AVC 壓縮視訊串流。

![未連接的 AVC 編碼器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-unconnected-avc-encoder.png)

*未連接的 AVC 編碼器*

其屬性會決定編碼確切發生的方式。 讓我們看看一些更重要的設定：

* 輸出寬度和輸出高度：決定編碼影片的解析度。 在此情況下，640x360 是正確的設定。
* 畫面格速率：設定為通過時，它只會採用來源畫面格速率，不過可加以覆寫。 這類的畫面格速率轉換並非動作補償。
* 設定檔和層級：決定 AVC 設定檔和層級。 若要便利地取得不同層級和設定檔的詳細資訊，請按一下 [AVC 視訊編碼器] 元件的問號圖示，說明頁面便會顯示有關每個層級的詳細資料。 在此範例中，使用層級 3.2 的主要設定檔 (預設值)。
* 速率控制模式和位元速率 (kbps)：在此案例中，選擇使用 1200 kbps 常數位元速率 (CBR) 輸出
* 影片格式：提供關於會寫入至 H.264 串流的 VUI (影片可用性資訊) 的資訊 (解碼器可能會用來加強顯示，但對正確解碼並非必要的輔助資訊)：
* NTSC (一般用於美國或日本，使用 30 fps)
* PAL (一般用於歐洲地區，使用 25 fps)
* GOP 大小模式：針對我們的目的，設定固定的 GOP 大小，主要間隔為 2 秒，並且關閉 GOP。 2 秒的設定可確保與 Azure 媒體服務所提供之動態封裝的相容性。

若要摘要 AVC 編碼器，請將 [未壓縮的影片] 輸出接點從 [媒體檔案輸入] 元件連線到 [AVC 編碼器] 的 [未壓縮的影片] 輸入接點。

![連接的 AVC 編碼器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connected-avc-encoder.png)

*連接的 AVC 主要編碼器*

### <a id="MXF_to_MP4_audio"></a>對音訊串流編碼
此時，仍需壓縮原始未壓縮的音訊串流。 如需壓縮音訊串流，請將 AAC 編碼器 (Dolby) 元件新增至工作流程。

![未連接的 AVC 編碼器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-unconnected-aac-encoder.png)

*未連接的 AAC 編碼器*

現在有不相容性：AAC 編碼器只有單一未壓縮音訊輸入接點，而媒體檔案輸入可能會有兩個不同的未壓縮音訊串流可用：一個用於左聲道，一個用於右聲道。 (如果您正在處理環繞音效，則是 6 聲道。)因此，不可能直接將音訊從 [媒體檔案輸入] 來源連接至 AAC 音訊編碼器。 AAC 元件預期稱為「交錯」的音訊串流：具有左右聲道並彼此交錯的單一串流。 一旦我們從來源媒體檔案知道哪一個音訊曲目在來源中的哪個位置後，可以使用正確指派的左右喇叭位置來產生這類的交錯音訊串流。

首先，使用者需要從需要的來源音訊聲道產生交錯的串流。 音訊串流交錯器元件會為我們處理。 將它加入至工作流程，並從 [媒體檔案輸入] 將音訊輸出連接到它。

![連接音訊串流交錯器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connected-audio-stream-interleaver.png)

*連接音訊串流交錯器*

既然我們已經有交錯的音訊串流，我們仍未指派左或右喇叭的位置。 為了指定位置，我們可以利用「喇叭位置指定器」。

![加入喇叭位置指定器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-adding-speaker-position-assigner.png)

*加入喇叭位置指定器*

設定 [喇叭位置指定器] 以搭配使用透過 [自訂] 編碼器預設過濾器和稱為 "2.0 (L,R)" 之聲道預設的立體聲輸入串流。 (這會將左喇叭位置指派為聲道 1，右喇叭位置指定為聲道 2。)

將 [喇叭位置指定器] 的輸出連接到 [AAC 編碼器] 的輸入。 然後，告訴 AAC 編碼器使用 "2.0 (L,R)" 聲道預設，讓它知道要將立體聲音訊處理為輸入。

### <a id="MXF_to_MP4_audio_and_fideo"></a>將音訊和視訊串流多工處理為 MP4 容器
假設我們有 AVC 編碼的視訊串流和 AAC 編碼的音訊串流，我們可以將兩者擷取為 .MP4 容器。 將不同的串流混合為單一串流的程序稱為「多工處理」(multiplexing，或 "muxing")。 在此情況下，我們要將音訊及影片串流交錯到單一一致的 .MP4 套件。 為 .MP4 容器協調此動作的元件稱為 ISO MPEG-4 多工器。 將其中一個加入至設計工具介面，並將 AVC 視訊編碼器和 AAC 編碼器連接到其輸入。

![連接的 MPEG4 多工器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connected-mpeg4-multiplexer.png)

*連接的 MPEG4 多工器*

### <a id="MXF_to_MP4_writing_mp4"></a>寫入 MP4 檔案
寫入輸出檔時，會使用「檔案輸出」元件。 我們可以將它連接到 ISO MPEG-4 多工器的輸出，讓其輸出寫入至磁碟。 若要這樣做，請將容器 (MPEG-4) 輸出接點連接到 [檔案輸出] 的 [寫入] 輸入接點。

![連接的檔案輸出](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connected-file-output.png)

*連接的檔案輸出*

所使用的檔案名稱取決於檔案屬性。 雖然可以將該屬性硬式編碼為特定值，使用者最可能需要改為透過運算式來設定它。

若要讓工作流程透過運算式自動判斷輸出 [檔案名稱] 屬性，請按一下 [檔案名稱] 旁邊的按鈕 (資料夾圖示旁)。 從下拉式功能表選取 [運算式]。 這會顯示運算式編輯器。 先清除編輯器的內容。

![空白的運算式編輯器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-empty-expression-editor.png)

*空白的運算式編輯器*

運算式編輯器可讓您以一或多個變數的混合輸入任何常值。 以貨幣符號開頭的變數。 當您按下 $ 鍵時，編輯器會顯示下拉式清單方塊，其中含有可用變數的選擇。 在此案例中，我們將使用輸出目錄變數與基礎輸入檔案名稱變數的組合：

    ${ROOT_outputWriteDirectory}\\${ROOT_sourceFileBaseName}.MP4

![填入運算式編輯器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-expression-editor.png)

*填入運算式編輯器*

> [!NOTE]
> 若要在 Azure 中查看編碼作業的輸出檔，您必須在運算式編輯器中提供值。
>
>

當您按 [確定] 確認運算式時，[屬性] 視窗將會預覽在此時間點的檔案屬性所解析的值。

![檔案運算式解析輸出目錄](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-file-expression-resolves-output-dir.png)

*檔案運算式解析輸出目錄*

### <a id="MXF_to_MP4_asset_from_output"></a>從輸出檔案建立媒體服務資產
雖然我們已編寫 MP4 輸出檔，但仍需要指出此檔案屬於媒體服務會因為執行此工作流程而產生的輸出資產。 在此端，會使用工作流程畫布上的 [輸出檔案/資產] 節點。 所有連入此節點中的檔案就會成為所產生之 Azure 媒體服務資產的一部分。

將 [檔案輸出] 元件連接到 [輸出檔案/資產] 元件以完成工作流程。

![工作流程完成](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-finished-workflow.png)

*工作流程完成*

### <a id="MXF_to_MP4_test"></a>在本機測試完成的工作流程
若要在本機測試工作流程，請按頂端工具列中的 [播放] 按鈕。 當工作流程完成執行時，請檢查設定的輸出資料夾中產生的輸出。 您會看到從 MXF 輸入來源檔案進行編碼完成的 MP4 輸出檔。

## <a id="MXF_to_MP4_with_dyn_packaging"></a>將 MXF 編碼為 MP4 - 多位元速率動態封裝已啟用
本逐步解說會使用單一 .MXF 輸入檔案中 AAC 編碼的音訊來建立一組多位元速率 MP4 檔案。

想要將多位元速率資產輸出用於結合 Azure 媒體服務提供的動態封裝功能時，將需要對每個不同的位元速率與解析度產生多個結合 GOP 的 MP4 檔案。 若要這樣做， [將 MXF 編碼為單一位元速率 MP4](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4) 逐步解說提供不錯的起點。

![開始工作流程](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-starting-workflow.png)

*開始工作流程*

### <a id="MXF_to_MP4_with_dyn_packaging_more_outputs"></a>加入一或多個其他的 MP4 輸出
我們產生的 Azure 媒體服務資產中的每個 MP4 檔案，都會支援不同的位元速率與解析度。 讓我們加入一或多個 MP4 輸出檔案至工作流程。

若要確定我們使用相同的設定來建立影片編碼器，最方便的方式是複製現有的 AVC 影片編碼器，並設定其他解析度及位元速率的組合 (讓我們新增 960x540，每秒 25 個畫面格，2.5 Mbps 的組合)。 若要複製現有的編碼器，請在設計工具介面複製貼上。

將 [媒體檔案輸入] 的 [未壓縮的視訊] 輸出接點連接到我們的新 AVC 元件。

![連接第二個 AVC 編碼器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-second-avc-encoder-connected.png)

*連接第二個 AVC 編碼器*

現在對我們的新 AVC 編碼器採用設定，以 2.5 Mbps 輸出 960x540。 (對此使用其屬性 [輸出寬度]、[輸出高度] 和 [位元速率 (kbps)]。)

假設我們想要使用產生的資產搭配 Azure 媒體服務的動態封裝，串流處理端點必須能夠從這些 MP4 檔案產生 HLS/分散的 MP4/DASH 片段，這些片段會以要在不同的位元速率之間切換來獲得單一順暢的連續視訊和音訊體驗的用戶端的方式彼此完全對應。 為了達到這個目的，我們需要確保兩個 MP4 檔案的 AVC 編碼器與 GOP (「一組圖片」) 大小的屬性設定為 2 秒，您可以透過以下方式完成：

* 將 GOP 大小模式設定為固定 GOP 大小，以及
* 主要畫面格間隔設為兩秒。
* 同時將 [GOP IDR 控制] 設為 [關閉 GOP] 以確保所有 GOP 獨立而沒有相依性

為了讓您更輕鬆了解此工作流程，請將第一個 AVC 編碼器重新命名為「AVC 影片編碼器 640x360 1200 kbps」，將第二個 AVC 編碼器重新命名為「AVC 影片編碼器 960x540 2500 kbps」。

現在加入第二個「ISO MPEG-4 多工器」和第二個 [檔案輸出]。 將多工器連接至新的 AVC 編碼器，並確定其輸出導向到 [檔案輸出]。 然後將 AAC 音訊編碼器輸出連接至新的多工器輸入。 接著就可以將 [檔案輸出] 連接至 [輸出檔案/資產] 節點，以將它加入將建立的 [媒體服務資產]。

![連接第二個 Muxer 和檔案輸出](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-second-muxer-file-output-connected.png)

*連接第二個 Muxer 和檔案輸出*

為了與 Azure 媒體服務動態封裝的相容性，請將多工器的 [區塊模式] 設定為 GOP 計數或持續時間，並將每個區塊的 GOP 設為 1。 (這應該是預設值。)

![Muxer 區塊模式](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-muxer-chunk-modes.png)

*Muxer 區塊模式*

注意：您可以對要加入至資產輸出的任何其他位元速率和解析度組合重複此程序。

### <a id="MXF_to_MP4_with_dyn_packaging_conf_output_names"></a>設定檔案輸出名稱
我們已將一個以上的單一檔案加入至輸出資產。 這使得您需要確定每個輸出檔案的檔案名稱會彼此不同，並甚至可能套用檔案命名慣例，使得您能夠從檔案名稱清楚知道要處理的是什麼。

檔案輸出命名可以透過設計工具中的運算式來控制。 開啟其中一個 [檔案輸出] 元件的屬性窗格，然後開啟 [檔案屬性] 的運算式編輯器。 我們的第一個輸出檔是透過下列運算式設定 (請參閱從 [MXF 到單一位元速率 MP4 輸出](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4)的教學課程)：

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}.MP4

這表示我們的檔案名稱由兩個變數決定：要寫入的輸出目錄和來源檔案基礎名稱。 前者會在工作流程根目錄上公開為屬性，後者則是由連入的檔案決定。 輸出目錄是您用於本機測試的目錄；當 Azure 媒體服務中以雲端為基礎的媒體處理器執行工作流程時，這個屬性會由工作流程引擎覆寫。
若要提供這兩個輸出檔案一致的輸出命名，請將第一個檔案命名運算式變更為：

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_640x360_1.MP4

並將第二個變更為：

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_960x540_2.MP4

執行中繼的測試回合，以確定正確產生這兩個 MP4 輸出檔案。

### <a id="MXF_to_MP4_with_dyn_packaging_audio_tracks"></a>加入個別的曲目
如我們稍後所見，在產生要隨著我們的 MP4 輸出檔案傳送之 .ism 檔案時，我們也將需要僅限音訊的 MP4 檔案作為彈性資料流的曲目。 若要建立此檔案，請將額外的 Muxer 加入至工作流程 (ISO-MPEG-4 多工器)，並 AAC 編碼器的輸出接點與其曲目 1 輸入接點連接。

![加入音訊 Muxer](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-audio-muxer-added.png)

*加入音訊 Muxer*

建立第三個 [檔案輸出] 元件，以從 Muxer 輸出輸出串流，並將檔案命名運算式設定為：

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_128kbps_audio.MP4

![音訊 Muxer 建立輸出檔案](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-audio-muxer-creating-file-output.png)

*音訊 Muxer 建立輸出檔案*

### <a id="MXF_to_MP4_with_dyn_packaging_ism_file"></a>加入 .ISM SMIL 檔案
為了讓動態封裝能在我們媒體服務資產中結合這兩個 MP4 檔案 (僅限音訊的 MP4) 運作，我們也需要資訊清單檔案 (也稱為 "SMIL" 檔案：同步多媒體整合語言)。 這個檔案可向 Azure 媒體服務指出哪些 MP4 檔案可供動態封裝，以及要考量進行音訊串流的檔案。 具有單一的音訊串流之一組 MP4 的一般資訊清單檔看起來像這樣：

```xml
    <?xml version="1.0" encoding="utf-8" standalone="yes"?>
    <smil xmlns="http://www.w3.org/2001/SMIL20/Language">
      <head>
        <meta name="formats" content="mp4" />
      </head>
      <body>
        <switch>
          <video src="H264_1900kbps_AAC_und_ch2_96kbps.mp4" />
          <video src="H264_1300kbps_AAC_und_ch2_96kbps.mp4" />
          <video src="H264_900kbps_AAC_und_ch2_96kbps.mp4" />
          <audio src="AAC_ch2_96kbps.mp4" title="AAC_und_ch2_96kbps" />
        </switch>
      </body>
    </smil>
```

.ism 檔案中包含 switch 陳述式、每個個別 MP4 視訊檔案的參考，此外還有只包含音訊的 MP4 的一個 (或多個) 音訊檔案的參考。

為我們的一組 MP4 產生資訊清單檔案，可透過名為「AMS 資訊清單寫入器」的元件完成。 若要使用它，請將它拖曳到介面上並將 [寫入完成] 輸出接點從三個 [檔案輸出] 元件連接到 [AMS 資訊清單寫入器] 輸入。 然後務必將 [AMS 資訊清單寫入器] 的輸出連接到 [輸出檔案/資產]。

如同對我們的其他檔案輸出元件，使用運算式來設定 .ism 檔案輸出名稱：

    ${ROOT_outputWriteDirectory}\\${ROOT_sourceFileBaseName}_manifest.ism

我們完成的工作流程看起來如下：

![MXF 到多位元速率 MP4 的工作流程完成](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-finished-mxf-to-multibitrate-mp4-workflow.png)

*MXF 到多位元速率 MP4 的工作流程完成*

## <a id="MXF_to__multibitrate_MP4"></a>將 MXF 編碼為多位元速率 MP4 - 增強的藍圖
在[前一個工作流程逐步解說](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging)中，我們已了解可以如何將單一 MXF 輸入資產轉換成輸出資產，其具有多位元速率 MP4 檔案、僅限音訊的 MP4 檔案和用於與 Azure 媒體服務動態封裝結合使用的資訊清單檔。

此逐步解說將示範如何加強一些層面，並使它更便利。

### <a id="MXF_to_multibitrate_MP4_overview"></a>要增強的工作流程概觀
![要增強的多位元速率 MP4 工作流程](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-multibitrate-mp4-workflow-to-enhance.png)

*要增強的多位元速率 MP4 工作流程*

### <a id="MXF_to__multibitrate_MP4_file_naming"></a>檔案命名慣例
在先前的工作流程中，我們已指定簡單的運算式作為產生輸出檔案名稱的基礎。 不過，我們有一些重複項目：所有的個別輸出檔案元件都指定了這類運算式。

例如，我們的第一個視訊檔案的檔案輸出元件是使用此運算式設定：

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_640x360_1.MP4

等候第二個視訊輸出時，我們有如下的運算式：

    ${ROOT_outputWriteDirectory}\\${ROOT_sourceFileBaseName}_960x540_2.MP4

如果我們可以移除某些重複項目，並改為使項目更易於設定，是不是比較清楚、較不容易出錯且更方便？ 幸好我們可以：設計工具的運算式功能結合能夠在我們的工作流程根目錄上建立自訂屬性，提供我們多一層的便利性。

假設我們將從個別 MP4 檔案的位元速率推動檔案名稱的組態。 我們將力求在一個集中位置 (在我們圖形的根目錄) 設定的這些位元速率，將從該位置存取它們，以設定及推動檔案名稱產生。 為了這樣做，我們會從將來自兩個 AVC 編碼器的位元速率屬性發佈到我們的工作流程根目錄開始，使其成為可從根目錄及從 AVC 編碼器存取。 (即使顯示在兩個不同的位置，只有單一的基礎值。)

### <a id="MXF_to__multibitrate_MP4_publishing"></a>發佈元件屬性至工作流程根目錄
開啟第一個 AVC 編碼器，移至 [位元速率 (kbps)] 屬性，並從下拉式清單中選擇 [發佈]。

![發佈位元速率屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publishing-bitrate-property.png)

*發佈位元速率屬性*

設定 [發佈] 對話方塊，以發佈至我們的工作流程圖形的根目錄，使用發佈的名稱 "video1bitrate" 以及可讀取的顯示名稱「視訊 1 位元速率」。 設定名為「串流處理位元速率」的自訂群組名稱，並按 [發佈]。

![發佈位元速率屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publishing-dialog-for-bitrate-property.png)

*位元速率屬性的發佈對話方塊*

對第二個 AVC 編碼器的位元速率屬性重複相同的動作，並在相同的自訂群組「串流處理位元速率」中將它命名為 "video2bitrate"，以及顯示名稱「視訊 2 位元速率」。

如果我們現在檢查工作流程根目錄屬性，就會看到我們的自訂群組顯示這兩個發佈的屬性。 兩個都會反映其各自的 AVC 編碼器位元速率的值。

![工作流程根目錄上發佈的位元速率屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-published-bitrate-props-on-workflow-root.png)

每當我們想要從程式碼或透過運算式存取這些屬性時，我們可以這麼做：

* 從根目錄下之元件中的內嵌程式碼：node.getPropertyAsString('../video1bitrate',null)
* 在運算式內：${ROOT_video1bitrate}

讓我們透過也在其上發佈我們的曲目位元速率來完成「串流處理位元速率」群組。 在 AAC 編碼器的屬性內，搜尋「位元速率」設定，並從它旁邊的下拉式清單中選取 [發佈]。 發佈到在我們的自訂群組「串流處理位元速率」內具有名稱 "audio1bitrate" 和顯示名稱「音訊 1 位元速率」的圖形根目錄中。

![音訊位元速率的發佈對話方塊](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publishing-dialog-for-audio-bitrate.png)

*音訊位元速率的發佈對話方塊*

![在根目錄產生視訊和音訊屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-resulting-video-and-audio-props-on-root.png)

*在根目錄產生視訊和音訊屬性*

對這三個值的任何變更也會重新設定並變更所連結 (和發佈來源位置) 之個別元件的值。

### <a id="MXF_to__multibitrate_MP4_output_files"></a>讓產生的輸出檔案名稱依賴發佈的屬性值
不要對我們產生的檔案名稱進行硬式編碼，我們現在可以在每個「檔案輸出」元件上變更檔案名稱，以仰賴我們在圖形根目錄上發佈的運算式屬性。 從我們的第一個檔案輸出開始，尋找檔案屬性，然後編輯運算式，如下：

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_${ROOT_video1bitrate}kbps.MP4

可以透過在運算式視窗中時按鍵盤上的貨幣符號來存取及輸入此運算式中的不同參數。 其中一個可用的參數是我們稍早發佈的 video1bitrate 屬性。

![存取運算式內的參數](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-accessing-parameters-within-an-expression.png)

*存取運算式內的參數*

對第二個視訊的檔案輸出執行相同的動作：

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_${ROOT_video2bitrate}kbps.MP4

以及對僅音訊檔案輸出：

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_${ROOT_audio1bitrate}bps_audio.MP4

如果我們現在變更任何視訊或音訊檔案的位元速率，將重新設定個別的編碼器，而將會對所有項目自動使用以位元速率為基礎的檔案名稱慣例。

## <a id="thumbnails_to__multibitrate_MP4"></a>加入縮圖至多位元速率 MP4 輸出
從 [透過 MXF 輸入產生多位元速率 MP4 輸出](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging)的工作流程開始，我們現在將尋求將縮圖加入到輸出。

### <a id="thumbnails_to__multibitrate_MP4_overview"></a>要加入縮圖的目標工作流程概觀
![要從中開始的多位元速率 MP4 工作流程](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-multibitrate-mp4-workflow-to-start-from.png)

*要從中開始的多位元速率 MP4 工作流程*

### <a id="thumbnails_to__multibitrate_MP4__with_jpg"></a>加入 JPG 編碼
我們的縮圖產生核心會是可以輸出 JPG 檔案的 JPG 編碼器元件。

![JPG 編碼器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-jpg-encoder.png)

*JPG 編碼器*

不過我們無法直接將未壓縮的視訊串流從媒體檔案輸入連接到 JPG 編碼器。 相反地，它預期會收到個別的畫面格。 我們可以透過「視訊畫面格閘道」元件執行此動作。

![將畫面格閘道連接到 JPG 編碼器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connect-frame-gate-to-jpg-encoder.png)

*將畫面格閘道連接到 JPG 編碼器*

畫面格閘道會每隔許多秒或畫面格執行一次，可讓視訊畫面格傳遞。 它發生的間隔和時間位移可在屬性中設定。

![視訊畫面格閘道屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-video-frame-gate-properties.png)

*視訊畫面格閘道屬性*

讓我們透過將模式設為時間 (秒) 及間隔設為 60，每隔一分鐘建立縮圖。

### <a id="thumbnails_to__multibitrate_MP4_color_space"></a>處理色彩空間轉換
雖然邏輯上可看到畫面格閘道和媒體檔案輸入的未壓縮的視訊接點現在已可連接，如果我們想執行此動作，則會收到警告。

![輸入色彩空間錯誤](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-input-color-space-error.png)

*輸入色彩空間錯誤*

這是因為在我們原始的原始未壓縮之影片串流 (來自我們的 MXF) 中，色彩資訊的表示方式與 JPG 編碼器所預期的不同。 更具體來說，預期會流入稱為 "RGB" 或「灰階」的「色彩空間」。 這表示，視訊框架閘道的輸入影片串流必須先套用有關其色彩空間的轉換。

拖曳到工作流程上的 [色彩空間轉換器 - Intel]，並將它連接到我們的畫面格閘道。

![連接色彩空間轉換器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connect-color-space-convertor.png)

*連接色彩空間轉換器*

在屬性視窗中，從 [預設] 清單選擇 BGR 24 項目。

### <a id="thumbnails_to__multibitrate_MP4_writing_thumbnails"></a>寫入縮圖
不同於我們的 MP4 影片，JPG 編碼器元件會輸出多個檔案。 為了解決這個問題，可以使用「場景搜尋 JPG 檔案寫入器」元件：它會採用傳入的 JPG 縮圖並加以寫出，每個檔案名稱結尾加上不同的數字。 (數字通常指出縮圖取自串流中的秒數/單位數。)

![推出場景搜尋 JPG 檔案寫入器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-scene-search-jpg-file-writer.png)

*推出場景搜尋 JPG 檔案寫入器*

使用以下運算式設定輸出資料夾路徑屬性：${ROOT_outputWriteDirectory}

和使用下列設定檔案名稱前置詞屬性：

    ${ROOT_sourceFileBaseName}_thumb_

前置詞會決定縮圖檔案命名的方式。 它們的前端會加上數字，指出串流中縮圖的位置。

![場景搜尋 JPG 檔案寫入器屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-scene-search-jpg-file-writer-properties.png)

*場景搜尋 JPG 檔案寫入器屬性*

將 [場景搜尋 JPG 檔案寫入器] 連接至 [輸出檔案/資產] 節點。

### <a id="thumbnails_to__multibitrate_MP4_errors"></a>偵測工作流程中的錯誤
將色彩空間轉換器的輸入連接到原始未壓縮的視訊輸出。 現在，對工作流程執行本機測試回合。 工作流程很可能會突然停止執行，並在發生錯誤之元件上以紅色外框指出：

![色彩空間轉換器錯誤](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-color-space-converter-error.png)

*色彩空間轉換器錯誤*

按一下色彩空間轉換器元件右上角的小型紅色 "E" 圖示，以查看編碼嘗試失敗的原因。

![色彩空間轉換器錯誤對話方塊](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-color-space-converter-error-dialog.png)

*色彩空間轉換器錯誤對話方塊*

結果，如您所見，對於我們要求 YUV 到 RGB 的轉換，色彩空間轉換器的傳入色彩空間標準必須為 rec601。 顯然我們的串流並未指出它是 rec601。 (Rec 601 是以數位視訊格式編碼交錯式類比視訊訊號的標準。 它會指定涵蓋 720 亮度取樣和每一行 360 色度取樣的作用中區域。 色彩編碼系統稱為 YCbCr 4:2:2。)

為了修正此問題，我們會在串流的中繼資料上指出我們要處理 rec601 內容。 若要這樣做，我們將使用「視訊資料類型更新器」元件，我們會將它放在原始來源和色彩空間轉換元件之間。 此資料類型的更新器可讓您手動更新特定視訊資料類型屬性。 將它設定，以指出 "Rec 601" 的色彩空間標準。 在尚未定義色彩空間的情況下，這會導致影片資料類型更新器以 "Rec 601" 色彩空間標記串流。 (它不會覆寫任何現有的中繼資料，除非已勾選 [覆寫] 核取方塊。)

![更新資料類型更新程式上的色彩空間標準](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-update-color-space-standard-on-data-type.png)

*更新資料類型更新程式上的色彩空間標準*

### <a id="thumbnails_to__multibitrate_MP4_finish"></a>工作流程完成
現在，我們完成了工作流程，接著執行另一個測試回合來查看它傳遞。

![具有縮圖的多個 mp4 輸出完成的工作流程](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-finished-workflow-for-multi-mp4-thumbnails.png)

*具有縮圖的多個 mp4 輸出完成的工作流程*

## <a id="time_based_trim"></a>多位元速率 MP4 輸出以時間為基礎的修剪
從 [透過 MXF 輸入產生多位元速率 MP4 輸出](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging)的工作流程開始，我們現在將尋求以時間戳記為基礎修剪來源視訊。

### <a id="time_based_trim_start"></a>要開始加入修剪的目標工作流程概觀
![要加入修剪的目標開始工作流程](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-starting-workflow-to-add-trimming.png)

*要加入修剪的目標開始工作流程*

### <a id="time_based_trim_use_stream_trimmer"></a>使用串流修剪器
串流修剪器元件可讓您根據計時資訊 (秒、分等等)，修剪輸入串流的開頭和結尾。修剪器不支援以畫面格為基礎的修剪。

![串流修剪器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-stream-trimmer.png)

*串流修剪器*

不要直接將 AVC 編碼器和喇叭位置指定器連結至媒體檔案輸入，我們會將它們放在串流修剪器之間。 (一個用於視訊訊號，一個用於交錯的音訊訊號。)

![在內部放入串流修剪器](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-put-stream-trimmer-in-between.png)

*在內部放入串流修剪器*

讓我們設定修剪器，使得我們只會處理 15 秒的視訊和音訊及 60 秒視訊。

移至影片串流修剪器的屬性，並設定開始時間 (15 秒) 和結束時間 (60 秒) 屬性。 若要確定我們的音訊和影片修剪器一律會同時設定為相同的開始與結束值，我們會將它們發佈至工作流程根目錄。

![串流修剪器的發佈開始時間屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publish-start-time-from-stream-trimmer.png)

*串流修剪器的發佈開始時間屬性*

![發佈屬性對話方塊的開始時間](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publish-dialog-for-start-time.png)

*發佈屬性對話方塊的開始時間*

![發佈屬性對話方塊的結束時間](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publish-dialog-for-end-time.png)

*發佈屬性對話方塊的結束時間*

如果我們現在檢查我們的工作流程根目錄，這兩個屬性將會整齊地顯示，且可從該處設定。

![在根目錄可取得發佈的屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-published-properties-available-on-root.png)

*在根目錄可取得發佈的屬性*

現在從音訊修剪器開啟修剪屬性，並使用參考工作流程根目錄上所發佈屬性的運算式來設定開始和結束時間。

針對音訊修剪開始時間：

    ${ROOT_TrimmingStartTime}

和其結束時間：

    ${ROOT_TrimmingEndTime}

### <a id="time_based_trim_finish"></a>工作流程完成
![工作流程完成](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-finished-workflow-time-base-trimming.png)

*工作流程完成*

## <a id="scripting"></a>推出指令碼元件
指令碼元件可以在我們的工作流程執行階段期間執行任意指令碼。 有四個可以執行的不同指令碼，每個都具有特定特性，以及在工作流程生命週期中的位置：

* **commandScript**
* **realizeScript**
* **processInputScript**
* **lifeCycleScript**

指令碼元件的文件會更詳細說明上述各項。 在 [下一節](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim)， **realizeScript** 指令碼元件用來在工作流程啟動時快速建構剪輯清單 XML。 在元件安裝期間會呼叫此指令碼，這種情況在其生命週期中只會發生一次。

### <a id="scripting_hello_world"></a>工作流程內的指令碼：Hello World
將指令碼元件拖曳至設計工具介面上，並重新命名 (例如，"SetClipListXML")。

![加入指令碼元件](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-add-scripted-comp.png)

*加入指令碼元件*

檢查指令碼元件的屬性時，會顯示四種不同的指令碼類型，而每個可設定到不同的指令碼。

![指令碼元件屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-scripted-comp-properties.png)

*指令碼元件屬性*

清除 processInputScript，並為 realizeScript 開啟編輯器。 現在我們已設定好並準備要開始指令碼。

指令碼是以 Groovy 編寫，它是 Java 平台適用的動態編譯指令碼語言，其維持與 Java 的相容性。 事實上，大部分的 Java 程式碼是有效的 Groovy 程式碼。

讓我們在 realizeScript 的內容中編寫一個簡單的 Hello World Groovy 指令碼。 在編輯器中輸入下列命令：

    node.log("hello world");

現在執行本機測試回合。 在這項執行之後，(透過 [指令碼元件] 上的 [系統] 索引標籤) 檢查 [記錄] 屬性。

![Hello World 記錄輸出](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-log-output.png)

*Hello World 記錄輸出*

我們呼叫記錄方法所在的節點物件，是指我們目前的「節點」或是我們正在編寫指令碼的元件。 每個元件因此具備可透過系統索引標籤輸出記錄資料的能力。在此情況下，我們會輸出字串常值 "Hello World"。 在此處需要了解的是這可以證明是非常重要的偵錯工具，讓您深入了解指令碼實際上做些什麼。

從我們的指令碼環境內，我們也可以存取其他元件的屬性。 試試看：

```java
    //inspect current node:
    def nodepath = node.getNodePath();
    node.log("this node path: " + nodepath);

    //walking up to other nodes:
    def parentnode = node.getParentNode();
    def parentnodepath = parentnode.getNodePath();
    node.log("parent node path: " + parentnodepath);

    //read properties from a node:
    def sourceFileExt = parentnode.getPropertyAsString( "sourceFileExtension", null );
    def sourceFileName = parentnode.getPropertyAsString("sourceFileBaseName", null);
    node.log("source file name with extension " + sourceFileExt + " is: " + sourceFileName);
```

我們的記錄視窗顯示如下：

![用於存取節點路徑的記錄輸出](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-log-output2.png)

*用於存取節點路徑的記錄輸出*

## <a id="frame_based_trim"></a>多位元速率 MP4 輸出以畫面格為基礎的修剪
從 [透過 MXF 輸入產生多位元速率 MP4 輸出](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging)的工作流程開始，我們現在將尋求以畫面格計數為基礎修剪來源視訊。

### <a id="frame_based_trim_start"></a>要開始加入修剪的目標藍圖概觀
![要開始加入修剪的目標工作流程](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-workflow-start-adding-trimming-to.png)

*要開始加入修剪的目標工作流程*

### <a id="frame_based_trim_clip_list"></a>使用剪輯清單 XML
在所有先前的工作流程教學課程中，我們使用「媒體檔案輸入」元件做為我們的視訊輸入來源。 不過，在此特定案例中，我們將改為使用剪輯清單來源元件。 這應該不是最好的工作方式；只在有實際原因這麼做時，才使用剪輯清單來源 (如同在下列情況下，我們會使用剪輯清單修剪功能)。

若要從我們的「媒體檔案輸入」切換到「剪輯清單來源」，請將 [剪輯清單來源] 元件拖曳至設計介面，並將 [剪輯清單 XML] 接點連接至工作流程設計工具的 [剪輯清單 XML] 節點。 這會根據我們的輸入影片，以輸出接點填入剪輯清單來源。 現在，從剪輯清單來源將「未壓縮的影片」和「未壓縮的音訊」接點連線至相應的「AVC 編碼器」和「音訊串流交錯器」。 現在移除媒體檔案輸入。

![以剪輯清單來源取代媒體檔案輸入](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-replaced-media-file-with-clip-source.png)

*以剪輯清單來源取代媒體檔案輸入*

剪輯清單來源元件會將它視為輸入「剪輯清單 XML」。 選取要在本機測試的來源檔案，會為您自動填入此剪輯清單 XML。

![自動填入剪輯清單 XML 屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-auto-populated-clip-list-xml-property.png)

*自動填入剪輯清單 XML 屬性*

更仔細一點觀察 XML，這是其外觀：

![編輯剪輯清單對話方塊](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-edit-clip-list-dialog.png)

*編輯剪輯清單對話方塊*

不過這不會反映剪輯清單 XML 的功能。 我們擁有的其中一個選項是在視訊和音訊來源下加入「修剪」元素，像這樣：

![加入修剪元素至剪輯清單](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-adding-trim-element-to-clip-list.png)

*加入修剪元素至剪輯清單*

如果您修改類似以上的剪輯清單 XML，並執行本機測試回合，您會看到視訊已正確在視訊中修剪為 10 到 20 秒。

不過，相對於當您執行本機執行時發生的情況，在 Azure 媒體服務中執行的工作流程中，這個相同的剪輯清單 XML 將不會有相同的效果。 Azure Premium 編碼器啟動時，每次都會根據提供給編碼作業的輸入檔產生剪輯清單 XML。 這表示，我們在 XML 上執行的任何變更不幸地會被覆寫。

若要克服剪輯清單 XML 在編碼作業開始時遭到抹除，我們可以在工作流程開始之後將它快速重新產生。 透過稱為「指令碼元件」的項目即可採取這種自訂動作。 如需詳細資訊，請參閱 [推出指令碼元件](media-services-media-encoder-premium-workflow-tutorials.md#scripting)。

將指令碼元件拖曳至設計工具介面上，並重新命名為 "SetClipListXML"。

![加入指令碼元件](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-add-scripted-comp.png)

*加入指令碼元件*

檢查指令碼元件的屬性時，會顯示四種不同的指令碼類型，而每個可設定到不同的指令碼。

![指令碼元件屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-scripted-comp-properties.png)

*指令碼元件屬性*

### <a id="frame_based_trim_modify_clip_list"></a>透過指令碼元件修改剪輯清單
在我們可以將工作流程啟動時產生的剪輯清單 XML 重新寫入之前，必須存取剪輯清單 XML 屬性和內容。 我們可以像這樣執行：

```java
    // get cliplist xml:
    def clipListXML = node.getProperty("../clipListXml");
    node.log("clip list xml coming in: " + clipListXML);
```

![記錄傳入剪輯清單](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-incoming-clip-list-logged.png)

*記錄傳入剪輯清單*

首先，我們必須決定我們需要修剪影片的哪一個點到哪一個點。 為了讓它方便工作流程較不具技術性的使用者，請將兩個屬性發佈至圖形的根目錄。 若要這樣做，請以滑鼠右鍵按一下設計工具介面並選取 [新增屬性]：

* 第一個屬性："ClippingTimeStart"，類型："TIMECODE"
* 第二個屬性："ClippingTimeEnd"，類型："TIMECODE"

![加入屬性對話方塊的剪輯開始時間](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-clip-start-time.png)

*加入屬性對話方塊的剪輯開始時間*

![工作流程根目錄上發佈的剪輯時間屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-clip-time-props.png)

*工作流程根目錄上發佈的剪輯時間屬性*

將這兩個屬性設定為適當的值：

![設定剪輯開始和結束屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-configure-clip-start-end-prop.png)

*設定剪輯開始和結束屬性*

現在，從我們的指令碼內，我們就可以存取這兩個屬性，像這樣：

```java
    // get start and end of clipping:
    def clipstart = node.getProperty("../ClippingTimeStart").toString();
    def clipend = node.getProperty("../ClippingTimeEnd").toString();

    node.log("clipping start: " + clipstart);
    node.log("clipping end: " + clipend);
```

![顯示剪輯的開始與結束的記錄視窗](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-show-start-end-clip.png)

*顯示剪輯的開始與結束的記錄視窗*

讓我們使用簡單的規則運算式，將時間碼字串剖析為更方便使用的格式：

```java
    //parse the start timing:
    def startregresult = (~/(\d\d:\d\d:\d\d:\d\d)\/(\d\d)/).matcher(clipstart);
    startregresult.matches();
    def starttimecode = startregresult.group(1);
    node.log("timecode start is: " + starttimecode);
    def startframerate = startregresult.group(2);
    node.log("framerate start is: " + startframerate);

    //parse the end timing:
    def endregresult = (~/(\d\d:\d\d:\d\d:\d\d)\/(\d\d)/).matcher(clipend);
    endregresult.matches();
    def endtimecode = endregresult.group(1);
    node.log("timecode end is: " + endtimecode);
    def endframerate = endregresult.group(2);
    node.log("framerate end is: " + endframerate);
```

![具有剖析的時間碼輸出的記錄檔視窗](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-output-parsed-timecode.png)

*具有剖析的時間碼輸出的記錄檔視窗*

在手邊備有這項資訊，我們現在可以修改剪輯清單 XML 以反映影片所需的畫面格精確剪輯的開始和結束時間。

![要加入修剪元素的指令碼](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-add-trim-elements.png)

*要加入修剪元素的指令碼*

這是透過一般的字串處理作業完成。 產生的經修改剪輯清單 XML 會透過 "setProperty" 方法寫回工作流程根目錄上的 clipListXML 屬性。 在另一個測試回合之後記錄視窗會向我們顯示下列：

![記錄產生的剪輯清單](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-log-result-clip-list.png)

*記錄產生的剪輯清單*

執行測試回合以查看視訊和音訊串流剪輯的情況。 不過，由於您將對修剪點的使用不同值進行多個測試回合，您會發現，這些將不會被納入考量！ 這是因為設計工具不同於 Azure 執行階段，不會在每次執行時覆寫剪輯清單 XML。 這就表示，只有在您第一次設定輸入和輸出點時，會導致 XML 轉換，所有其他時候，我們的成立條件子句 (if(clipListXML.indexOf("<trim>") == -1)) 會在一個元素已存在時，避免工作流程新增另一個修剪元素。

為了讓工作流程方便在本機測試，我們最好加入一些管理程式碼，其會檢查是否已經存在修剪元素。 如果是的話，我們可以在繼續之前，將 XML 修改為新的值來將它移除。 不要使用純文字字串操作，透過實際的 XML 物件模型剖析執行此動作可能更安全。

在我們可以加入這類程式碼之前，我們還需要先在指令碼的開頭加入我們一些匯入陳述式：

```java
    import javax.xml.parsers.*;
    import org.xml.sax.*;
    import org.w3c.dom.*;
    import javax.xml.*;
    import javax.xml.xpath.*;
    import javax.xml.transform.*;
    import javax.xml.transform.stream.*;
    import javax.xml.transform.dom.*;
```

在此之後，我們可以加入必要的清除程式碼：

```java
    //for local testing: delete any pre-existing trim elements from the clip list xml by parsing the xml into a DOM:
    DocumentBuilderFactory factory=DocumentBuilderFactory.newInstance();
    DocumentBuilder builder=factory.newDocumentBuilder();
    InputSource is=new InputSource(new StringReader(clipListXML));
    Document dom=builder.parse(is);

    //find the trim element inside videoSource and audioSource and remove it if it exists already:
    XPath xpath = XPathFactory.newInstance().newXPath();
    String findAllTrimElements = "//trim";
    NodeList trimelems = xpath.evaluate(findAllTrimElements,dom,XPathConstants.NODESET);

    //copy trim nodes into a "to-be-deleted" collection
    Set<Element> elementsToDelete = new HashSet<Element>();
    for (int i = 0; i < trimelems.getLength(); i++) {
        Element e = (Element)trimelems.item(i);
        elementsToDelete.add(e);
    }

    node.log("about to delete any existing trim nodes");
     //delete the trim nodes:
    elementsToDelete.each{
        e -> e.getParentNode().removeChild(e);
    };
    node.log("deleted any existing trim nodes");

    //serialize the modified clip list xml dom into a string:
    def transformer = TransformerFactory.newInstance().newTransformer();
    StreamResult result = new StreamResult(new StringWriter());
    DOMSource source = new DOMSource(dom);
    transformer.transform(source, result);
    clipListXML = result.getWriter().toString();
```

這個程式碼會放在我們加入修剪元素至剪輯清單 XML 的點的正上方。

此時，只要我們需要，可以在讓變更隨著時間套用時執行及修改工作流程。    

### <a id="frame_based_trim_clippingenabled_prop"></a>加入 ClippingEnabled 便利屬性
因為您可能不要一律進行修剪，讓我們透過加入方便的布林值旗標 (可指出是否要啟用修剪/剪輯) 來完成工作流程。

如同以往，發佈新的屬性到我們稱為 "ClippingEnabled" (類型 "BOOLEAN") 的工作流程根目錄。

![發佈啟用剪輯屬性](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-enable-clip.png)

*發佈啟用剪輯屬性*

使用以下簡單的成立條件子句，我們可以檢查是否需要修剪，並決定是否因此需要修改剪輯清單。

```java
    //check if clipping is required:
    def clippingrequired = node.getProperty("../ClippingEnabled");
    node.log("clipping required: " + clippingrequired.toString());
    if(clippingrequired == null || clippingrequired == false)
    {
        node.setProperty("../clipListXml",clipListXML);
        node.log("no clipping required");
        return;
    }
```

### <a id="code"></a>完整程式碼

```java
    import javax.xml.parsers.*;
    import org.xml.sax.*;
    import org.w3c.dom.*;
    import javax.xml.*;
    import javax.xml.xpath.*;
    import javax.xml.transform.*;
    import javax.xml.transform.stream.*;
    import javax.xml.transform.dom.*;

    // get cliplist xml:
    def clipListXML = node.getProperty("../clipListXml");
    node.log("clip list xml coming in: \n" + clipListXML);
    // get start and end of clipping:
    def clipstart = node.getProperty("../ClippingTimeStart").toString();
    def clipend = node.getProperty("../ClippingTimeEnd").toString();

    //parse the start timing:
    def startregresult = (~/(\d\d:\d\d:\d\d:\d\d)\/(\d\d)/).matcher(clipstart);
    startregresult.matches();
    def starttimecode = startregresult.group(1);
    node.log("timecode start is: " + starttimecode);
    def startframerate = startregresult.group(2);
    node.log("framerate start is: " + startframerate);

    //parse the end timing:
    def endregresult = (~/(\d\d:\d\d:\d\d:\d\d)\/(\d\d)/).matcher(clipend);
    endregresult.matches();
    def endtimecode = endregresult.group(1);
    node.log("timecode end is: " + endtimecode);
    def endframerate = endregresult.group(2);

    node.log("framerate end is: " + endframerate);

    //for local testing: delete any pre-existing trim elements
    //from the clip list xml by parsing the xml into a DOM:

    DocumentBuilderFactory factory=DocumentBuilderFactory.newInstance();
    DocumentBuilder builder=factory.newDocumentBuilder();
    InputSource is=new InputSource(new StringReader(clipListXML));
    Document dom=builder.parse(is);

    //find the trim element inside videoSource and audioSource and remove it if it exists already:
    XPath xpath = XPathFactory.newInstance().newXPath();
    String findAllTrimElements = "//trim";
    NodeList trimelems = xpath.evaluate(findAllTrimElements, dom, XPathConstants.NODESET);

    //copy trim nodes into a "to-be-deleted" collection
    Set<Element> elementsToDelete = new HashSet<Element>();
    for (int i = 0; i < trimelems.getLength(); i++) {
        Element e = (Element)trimelems.item(i);
        elementsToDelete.add(e);
    }

    node.log("about to delete any existing trim nodes");
    //delete the trim nodes:
    elementsToDelete.each{ e ->
        e.getParentNode().removeChild(e);
    };
    node.log("deleted any existing trim nodes");

    //serialize the modified clip list xml dom into a string:
    def transformer = TransformerFactory.newInstance().newTransformer();
    StreamResult result = new StreamResult(new StringWriter());
    DOMSource source = new DOMSource(dom);
    transformer.transform(source, result);
    clipListXML = result.getWriter().toString();

    //check if clipping is required:
    def clippingrequired = node.getProperty("../ClippingEnabled");
    node.log("clipping required: " + clippingrequired.toString());
    if(clippingrequired == null || clippingrequired == false)
    {
        node.setProperty("../clipListXml",clipListXML);
        node.log("no clipping required");
        return;
    }

    //add trim elements to cliplist xml
    if ( clipListXML.indexOf("<trim>") == -1 )
    {
        //trim video
        clipListXML = clipListXML.replace("<videoSource>","<videoSource>\n <trim>\n <inPoint fps=\""+
            startframerate +"\">" + starttimecode +
            "</inPoint>\n" + "<outPoint fps=\"" + endframerate +"\"> " + endtimecode +
            " </outPoint>\n </trim> \n");
        //trim audio
        clipListXML = clipListXML.replace("<audioSource>","<audioSource>\n <trim>\n <inPoint fps=\""+
            startframerate +"\">" + starttimecode +
            "</inPoint>\n" + "<outPoint fps=\""+ endframerate +"\">" +
            endtimecode + "</outPoint>\n </trim>\n");
        node.log( "clip list going out: \n" +clipListXML );
        node.setProperty("../clipListXml",clipListXML);
    }
```

## <a name="also-see"></a>另請參閱
[介紹 Azure 媒體服務中的 Premium 編碼](http://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services)

[如何使用 Azure 媒體服務中的 Premium 編碼](http://azure.microsoft.com/blog/2015/03/06/how-to-use-premium-encoding-in-azure-media-services)

[透過 Azure 媒體服務編碼的隨選內容](media-services-encode-asset.md#media-encoder-premium-workflow)

[Media Encoder Premium Workflow 格式和轉碼器](media-services-premium-workflow-encoder-formats.md)

[範例工作流程檔案](http://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows)

[Azure 媒體服務總管工具](http://aka.ms/amse)

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
