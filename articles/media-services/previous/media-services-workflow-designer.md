---
title: 使用工作流程設計工具建立進階編碼工作流程 | Microsoft Docs
description: 深入了解如何使用工作流程設計工具建立進階編碼工作流程。
services: media-services
documentationcenter: ''
author: anilmur
manager: cfowler
editor: ''
ms.assetid: 004815f2-0761-4706-87a1-675ba36e0322
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2017
ms.author: juliako;johndeu;anilmur
ms.openlocfilehash: d75a3b58934b3da05a15700ecaf82226b549fffa
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785757"
---
# <a name="create-advanced-encoding-workflows-with-workflow-designer"></a>使用工作流程設計工具建立進階編碼工作流程
## <a name="overview"></a>概觀
**工作流程設計工具**是 Windows 傳統型工具，用來設計和建立自訂工作流程，以便用於**媒體編碼器高階工作流程**進行編碼。
藉由使用工作流程設計工具的強大功能，您可以設計和建立複雜的工作流程，在 **Media Encoder Premium** 中執行。  

根據輸入來源檔案的屬性，工作流程可以包含客戶決策邏輯和分支。 您可以建立工作流程，具有可覆寫屬性與動態值，讓最複雜的編碼工作易於在雲端中重複及自訂。

您可以建立的範例工作流程包括：

* 決策型工作流程，會檢查要解析的來源內容，並且只會編碼想要的輸出追蹤。  藉由消除倍增的來源內容不經意地產生的浪費的追蹤，而有所助益。
* 多個輸入檔案可以用來支援標題、重疊和連結在一起的內容。 

這項工具也可用來修改任何 [已發佈工作流程](media-services-workflow-designer.md#existing_workflows)。 

> [!NOTE]
> 若想取得工作流程設計工具的複本，請連絡 mepd@microsoft.com。
> 
> 

建立工作流程檔案後，此檔案可上傳做為資產，然後用來編碼媒體檔案。 如需如何使用 **.NET** 以 **Media Encoder Premium Workflow** 進行編碼的相關資訊，請參閱[使用 Media Encoder Premium Workflow 進行進階編碼](media-services-encode-with-premium-workflow.md)。

## <a id="existing_workflows"></a>修改現有的工作流程
可以使用設計工具來修改預設的 [已發佈工作流程](media-services-workflow-designer.md#existing_workflows) 。 您可在 [這裡](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows)取得預設的工作流程檔案。 資料夾也包含這些檔案的說明。

下列影片示範如何使用設計工具。

### <a name="day-1--getting-started"></a>第 1 天 – 開始使用
第 1 天影片涵蓋：

* 設計工具概觀
* 基本工作流程 – "Hello World"
* 建立多個搭配 Azure 媒體服務串流的輸出 MP4 檔案

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-1/player]
> 
> 

### <a name="day-2"></a>第 2 天
第 2 天影片涵蓋：

* 不同來源檔案的案例 – 處理音訊
* 使用進階邏輯的工作流程
* 圖形階段

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-2/player]
> 
> 

### <a name="day-3"></a>第 3 天
第 3 天影片涵蓋：

* 工作流程/藍圖中的指令碼處理
* 目前編碼器的限制
* 問答集

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Premium-Encoder-Workflow-Designer-Training-Videos-Day-3/player]
> 
> 

## <a name="next-step"></a>後續步驟
檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

如果您需要支援或有任何關於在工作流程設計工具中建立自訂工作流程的問題，請傳送電子郵件至 mepd@microsoft.com。

## <a name="see-also"></a>另請參閱
[Azure Premium 編碼器工作流程設計工具訓練影片](http://johndeutscher.com/2015/07/06/azure-premium-encoder-workflow-designer-training-videos/)

