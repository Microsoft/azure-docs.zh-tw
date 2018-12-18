---
title: 使用 Azure 媒體分析修訂臉部逐步解說 | Microsoft Docs
description: 本主題逐步說明如何使用 Azure 媒體服務總管 (AMSE) 和 Azure 媒體修訂器視覺化檢視 (開放原始碼工具) 來執行完整的修訂工作流程。
services: media-services
documentationcenter: ''
author: Lichard
manager: cfowler
editor: ''
ms.assetid: d6fa21b8-d80a-41b7-80c1-ff1761bc68f2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 09/03/2017
ms.author: rli; juliako;
ms.openlocfilehash: d7a6abaeff0e939ed9e299e16e2fe59b65127aba
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785957"
---
# <a name="redact-faces-with-azure-media-analytics-walkthrough"></a>使用 Azure 媒體分析修訂臉部逐步解說

## <a name="overview"></a>概觀

**Azure 媒體修訂器** 是 [Azure 媒體分析](media-services-analytics-overview.md) 媒體處理器 (MP)，可在雲端提供可調整的臉部修訂。 臉部修訂可讓您修改視訊，以模糊所選人物的臉部。 在公共安全和新聞媒體案例中，您可能會想要使用臉部修訂服務。 若要手動修訂包含多個臉部的幾分鐘影片，可能要花上數小時的時間，若使用此服務，則只需要幾個簡單的步驟就能完成臉部修訂程序。 如需詳細資訊，請參閱[此](https://azure.microsoft.com/blog/azure-media-redactor/)部落格。

如需 **Azure 媒體修訂器**的詳細資訊，請參閱[臉部修訂概觀](media-services-face-redaction.md)主題。

本主題逐步說明如何使用 Azure 媒體服務總管 (AMSE) 和 Azure 媒體修訂器視覺化檢視 (開放原始碼工具) 來執行完整的修訂工作流程。

如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/redaction-preview-available-globally) 。

## <a name="azure-media-services-explorer-workflow"></a>Azure 媒體服務總管工作流程

開始使用修訂器最簡單的方法，是使用 github 上的開放原始碼 AMSE 工具。 如果您不需要註解 json 或臉部 jpg 影像的存取權，您可以透過 [Combined] \(合併) 模式執行簡單的工作流程。

### <a name="download-and-setup"></a>下載及安裝

1. 從[這裡](https://github.com/Azure/Azure-Media-Services-Explorer)下載 AMSE 工具。
1. 使用您的服務金鑰登入您的媒體服務帳戶。

    若要取得帳戶名稱和金鑰資訊，請移至 [Azure 入口網站](https://portal.azure.com/)，然後選取 AMS 帳戶。 選取 [Settings] \(設定) > [Keys] \(金鑰)。 [管理金鑰] 視窗會顯示帳戶名稱以及主要和次要金鑰。 複製帳戶名稱和主要金鑰的值。

![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough001.png)

### <a name="first-pass--analyze-mode"></a>第一階段 – 分析模式

1. 透過 [Asset] \(資產) –> [Upload] \(上傳) 或以拖放方式上傳您的媒體檔案。 
1. 按一下滑鼠右鍵，使用 [Media Analytics] \(媒體分析) –> [Azure Media Redactor] \(Azure 媒體修訂器) –> [Analyze mode] \(分析模式) 處理您的媒體檔案。 


![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough002.png)

![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough003.png)

輸出會包含具有臉部位置資料的註解 json 檔案，以及每個所偵測臉部的 jpg。 

![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough004.png)

### <a name="second-pass--redact-mode"></a>第二階段 – 修訂模式

1. 將原始的視訊資產上傳至第一階段中的輸出，並設為主要資產。 

    ![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough005.png)

2. (選擇性) 上傳包含您想修訂之識別碼清單 (以新行分隔) 的 'Dance_idlist.txt' 檔案。 

    ![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough006.png)

3. (選擇性) 編輯 annotations.json 檔案，例如增加周框方塊界限。 
4. 以滑鼠右鍵按一下第一階段中的輸出資產，選取修訂器，並以 [Redact] \(修訂) 模式執行。 

    ![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough007.png)

5. 下載或分享最終修訂的輸出資產。 

    ![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough008.png)

## <a name="azure-media-redactor-visualizer-open-source-tool"></a>Azure 媒體修訂器視覺化檢視開放原始碼工具

開放原始碼[視覺化檢視工具](https://github.com/Microsoft/azure-media-redactor-visualizer)旨在協助剛開始註解格式操作的開發人員剖析和使用輸出。

複製儲存機制後，為了執行專案，您必須從[官方網站](https://ffmpeg.org/download.html)下載 FFMPEG。

如果您是嘗試剖析 JSON 註解資料的開發人員，請在 Models.MetaData 內尋找範例程式碼的範例。

### <a name="set-up-the-tool"></a>設定工具

1.  下載和建置整個方案。 

    ![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough009.png)

2.  從[這裡](https://ffmpeg.org/download.html)下載 FFMPEG。 此專案原先是使用含靜態連結的版本 be1d324 (2016-10-04) 所開發。 
3.  將 ffmpeg.exe 和 ffprobe.exe 複製到和 AzureMediaRedactor.exe 相同的輸出資料夾。 

    ![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough010.png)

4. 執行 AzureMediaRedactor.exe。 

### <a name="use-the-tool"></a>使用工具

1. 在您的 Azure 媒體服務帳戶中使用 [Analyze] \(分析) 模式的 [Redactor MP] \(修訂器 MP) 處理您的視訊。 
2. 下載原始視訊檔和「修訂 - 分析」工作的輸出。 
3. 執行視覺化檢視應用程式，然後選擇上述檔案。 

    ![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough011.png)

4. 預覽檔案。 透過右側資訊看板選取您想要模糊化的臉部。 
    
    ![臉部修訂](./media/media-services-redactor-walkthrough/media-services-redactor-walkthrough012.png)

5.  下方的文字欄位會更新為臉部識別碼。 建立名為 "idlist.txt" 的檔案，其中包含這些識別碼清單 (以新行分隔)。 

    >[!NOTE]
    > idlist.txt 應該以 ANSI 格式儲存。 您可以使用 [記事本] 來儲存為 ANSI 格式。
    
6.  將這個檔案上傳到步驟 1 中的輸出資產。 另將原始的視訊上傳到這個資產，並設為主要資產。 
7.  在這個資產上使用「修訂」模式執行「修訂」工作，以取得最終修訂的視訊。 

## <a name="next-steps"></a>後續步驟 

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>相關連結
[Azure 媒體服務分析概觀](media-services-analytics-overview.md)

[Azure 媒體分析示範](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)

[宣布推出適用於 Azure 媒體分析的臉部修訂功能](https://azure.microsoft.com/blog/azure-media-redactor/)
