---
title: 使用 Azure 媒體服務 v3 對自訂轉換進行編碼 | Microsoft Docs
description: 本主題說明如何使用 Azure 媒體服務 v3 對自訂轉換進行編碼。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: c3318020b54beb121ac48d993a8ce794ac8e377f
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376630"
---
# <a name="how-to-encode-with-a-custom-transform"></a>如何對自訂轉換進行編碼

使用 Azure 媒體服務來編碼時，您可以使用依據業界最佳做法所建議的其中一個內建預設來快速開始 (如[串流檔案](stream-files-tutorial-with-api.md)教學課程的示範)，也可以選擇建置以特定案例或裝置需求為標的的自訂預設。 

> [!Note]
> 在 Azure 媒體服務 v3 中，所有的編碼位元速率單位都是「位元/秒」。 這不同於 REST v2 媒體編碼器標準預設。 例如，v2 中的位元速率會指定為 128，但在 v3 中，則會是 128000。

## <a name="download-the-sample"></a>下載範例

使用以下命令將包含完整 .NET Core 範例的 GitHub 存放庫複製到您的機器：  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials.git
 ```
 
自訂預設的範例位於 [EncodeCustomTransform](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/EncodeCustomTransform/) 資料夾。

## <a name="create-a-transform-with-a-custom-preset"></a>使用自訂預設建立轉換 

建立新的[轉換](https://docs.microsoft.com/rest/api/media/transforms)時，您需要指定想要其產生的輸出是什麼。 必要的參數是 **TransformOutput** 物件，如下列程式碼所示。 每個 **TransformOutput** 都會包含 **Preset (預設)**。 **Preset** 會描述影片和/或音訊處理作業的逐步指示，以產生所需的 **TransformOutput**。 下列 **TransformOutput** 會建立自訂轉碼器和圖層輸出設定。

建立[轉換](https://docs.microsoft.com/rest/api/media/transforms)時，您應該先使用 **Get** 方法檢查是否已有轉換存在，如下列程式碼所示。  在媒體服務 v3 中，如果實體不存在，對實體執行的 **Get** 方法會傳回 **null** (檢查名稱時不區分大小寫)。

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/EncodeCustomTransform/MediaV3ConsoleApp/Program.cs#EnsureTransformExists)]

## <a name="next-steps"></a>後續步驟

[串流檔案](stream-files-tutorial-with-api.md) 
