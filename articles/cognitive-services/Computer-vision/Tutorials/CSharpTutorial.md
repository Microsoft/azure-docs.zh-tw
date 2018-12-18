---
title: 電腦視覺 API C# 教學課程 | Microsoft Docs
description: 探索使用「Microsoft 認知服務」中「電腦視覺 API」的基本 Windows 應用程式。 在影像中執行 OCR、建立縮圖，以及處理視覺特徵。
services: cognitive-services
author: KellyDF
manager: corncar
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: article
ms.date: 05/22/2017
ms.author: kefre
ms.openlocfilehash: f2aeb1734f8858ed8b80e5cdf48ef7e558703919
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368850"
---
# <a name="computer-vision-api-c35-tutorial"></a>電腦視覺 API C&#35; 教學課程

探索使用「電腦視覺 API」在影像中執行光學字元辨識 (OCR)、建立智慧型裁剪縮圖以及偵測、分類、標記和描述視覺特徵 (包括臉部) 的 Windows 應用程式。 以下範例可讓您提交影像 URL 或儲存在本機的檔案。 您可以將這個開放原始碼範例當作一個範本，來利用「視覺 API」和 WPF (Windows Presentation Foundation，.NET Framework 的組件) 建置自己的 Windows 應用程式。

### <a name="prerequisites"></a>先決條件

#### <a name="platform-requirements"></a>平台需求

下列範例已利用 [Visual Studio 2015，Community Edition](https://www.visualstudio.com/downloads/) 針對.NET Framework 進行開發。

#### <a name="subscribe-to-computer-vision-api-and-get-a-subscription-key"></a>訂閱電腦視覺 API 並取得訂用帳戶金鑰 

建立範例之前，您必須先訂閱「電腦視覺 API」，這是「Microsoft 認知服務」(先前稱為 Project Oxford) 的組件。 如需訂用帳戶和金鑰管理詳細資料，請參閱[訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)。 本教學課程可以使用主要金鑰，也可以使用次要金鑰。 

> [!NOTE]
> 本教學課程依設計會使用 **westcentralus** 區域中的訂用帳戶金鑰。 「電腦視覺」免費試用版中產生的訂用帳戶金鑰會使用 **westcentralus** 區域，因此這些金鑰可以正確運作。 如果您透過 [https://azure.microsoft.com/](https://azure.microsoft.com/) 使用 Azure 帳戶來產生訂用帳戶金鑰，則必須指定 **westcentralus** 區域。 在 **westcentralus** 區域外產生的金鑰將無法運作。

#### <a name="get-the-client-library-and-example"></a>取得用戶端程式庫和範例

您可以透過 [SDK](https://www.github.com/microsoft/cognitive-vision-windows)將「電腦視覺 API」用戶端程式庫和範例應用程式複製到電腦。 請勿將其下載為 ZIP 檔案。

### <a name="Step1">步驟 1：安裝範例</a>

在您的「GitHub 桌面」中，開啟 [Sample-WPF\VisionAPI-WPF-Samples.sln]。

### <a name="Step2">步驟 2：建置範例</a>

* 按 Ctrl+Shift+B，或按一下功能區功能表上的 [建置]，然後選取 [建置方案]。

### <a name="Step3">步驟 3： 執行範例</a>

1. 建置完成之後，按 **F5** 或按一下功能區功表上的 [啟動] 來執行範例。
2. 找出文字編輯方塊顯示「請在這裡貼上您的訂用帳戶金鑰來開始」的「電腦視覺 API」使用者介面視窗。
您可以按一下 [儲存金鑰] 按鈕來選擇將訂用帳戶金鑰保存到 PC 或膝上型電腦。 當您想要從系統中刪除訂用帳戶金鑰時，按一下 [刪除金鑰]，即可從 PC 或膝上型電腦中移除金鑰。

    ![視覺訂用帳戶金鑰](../Images/Vision_UI_Subscription.PNG)

3. 在 [選取案例] 底下，按一下以使用六個案例其中之一，然後依照畫面上的指示進行操作。 Microsoft 會收到您上傳的影像，而且可能使用它們來改善「電腦視覺 API」及相關服務。 藉由提交影像，即表示您確認已遵守我們的[開發人員行為準則](https://azure.microsoft.com/support/legal/developer-code-of-conduct/)。

    ![分析影像介面](../Images/Analyze_Image_Example.PNG)

4. 有一些要與此範例應用程式搭配使用的範例影像。 您可以在「臉部 API」Windows Github 存放庫上的 [Data 資料夾](https://github.com/Microsoft/Cognitive-Face-Windows/tree/master/Data)中找到這些影像。 請注意，這些影像是在 [LICENSE-IMAGE](https://github.com/Microsoft/Cognitive-Face-Windows/blob/master/LICENSE-IMAGE.md)合約下授權使用的。

### <a name="Review">檢閱和學習</a>

既然您已有一個能執行的應用程式，現在讓我們檢閱此範例應用程式如何與「認知服務」技術整合。 這可讓您利用「Microsoft 電腦視覺 API」，更輕鬆地繼續建置此應用程式或開發您自己的應用程式。

此範例應用程式會利用「電腦視覺 API 用戶端程式庫」，這是一個適用於「Microsoft 電腦視覺 API」的精簡版 C# 用戶端包裝函式。 當您按照上述步驟建置範例應用程式時，是從 NuGet 套件取得「用戶端程式庫」。 您可以在 **Vision**、**Windows**、**Client Library** 底下標題為 **Client Library** 的資料夾中，檢閱「用戶端程式庫」原始程式碼，這是上述＜先決條件＞中所提及下載檔案存放庫的一部分。

您也可以在 [方案總管] 中了解如何使用「用戶端程式庫」程式碼：在 [VisionAPI-WPF_Samples] 底下，展開 [AnalyzePage.xaml] 以找出 [AnalyzePage.xaml.cs]，這可用來將影像提交給影像分析端點。 按兩下 .xaml.cs 檔案，以在 Visual Studio 的新視窗中開啟這些檔案。

在檢閱「視覺用戶端程式庫」在我們範例應用程式中的使用方式同時，讓我們看看來自 **AnalyzePage.xaml.cs** 的兩個程式碼片段。 此檔案包含指出 “KEY SAMPLE CODE STARTS HERE” 和 “KEY SAMPLE CODE ENDS HERE” 的程式碼註解，可幫助您找出以下重新產生的程式碼片段。

分析端點可以接受影像 URL 或二進位影像資料 (以八位元資料流的形式) 作為輸入。 首先，您需找到 using 指示詞，如此才能使用「視覺用戶端程式庫」。

```
                // ----------------------------------------------------------------------
                // KEY SAMPLE CODE STARTS HERE
                // Use the following namespace for VisionServiceClient 
                // ---------------------------------------------------------------------- 
                using Microsoft.ProjectOxford.Vision; 
                using Microsoft.ProjectOxford.Vision.Contract; 
                // ----------------------------------------------------------------------
                // KEY SAMPLE CODE ENDS HERE 
                // ----------------------------------------------------------------------

```
**UploadAndAnalyzeImage(…)** 此程式碼片段示範如何使用「用戶端程式庫」，將訂用帳戶金鑰和儲存在本機的影像提交給「電腦視覺 API」服務的分析端點。

```
    private async Task<AnalysisResult> UploadAndAnalyzeImage(string imageFilePath)
    {
        // -----------------------------------------------------------------------
        // KEY SAMPLE CODE STARTS HERE
        // -----------------------------------------------------------------------  
        //
        // Create Project Oxford Computer Vision API Service client
        //
        VisionServiceClient VisionServiceClient = new VisionServiceClient(SubscriptionKey);
        Log("VisionServiceClient is created");
    
        using (Stream imageFileStream = File.OpenRead(imageFilePath))
        {
            //
            // Analyze the image for all visual features
            //
            Log("Calling VisionServiceClient.AnalyzeImageAsync()...");
         VisualFeature[] visualFeatures = new VisualFeature[] { VisualFeature.Adult, VisualFeature.Categories, VisualFeature.Color, VisualFeature.Description, VisualFeature.Faces, VisualFeature.ImageType, VisualFeature.Tags };
            AnalysisResult analysisResult = await VisionServiceClient.AnalyzeImageAsync(imageFileStream, visualFeatures);
            return analysisResult;
        }
    
        // -----------------------------------------------------------------------
        // KEY SAMPLE CODE ENDS HERE
        // -----------------------------------------------------------------------
        }
```
**AnalyzeUrl(…)** 此程式碼片段示範如何使用「用戶端程式庫」，將訂用帳戶金鑰和相片 URL 提交給「電腦視覺 API」服務的分析端點。

```
    private async Task<AnalysisResult> AnalyzeUrl(string imageUrl)
    {
        // -----------------------------------------------------------------------
        // KEY SAMPLE CODE STARTS HERE
        // -----------------------------------------------------------------------
    
        //
        // Create Project Oxford Computer Vision API Service client
        //
     VisionServiceClient VisionServiceClient = new VisionServiceClient(SubscriptionKey);
        Log("VisionServiceClient is created");
    
        //
        // Analyze the url for all visual features
        //
        Log("Calling VisionServiceClient.AnalyzeImageAsync()...");
        VisualFeature[] visualFeatures = new VisualFeature[] { VisualFeature.Adult, VisualFeature.Categories, VisualFeature.Color, VisualFeature.Description, VisualFeature.Faces, VisualFeature.ImageType, VisualFeature.Tags };
        AnalysisResult analysisResult = await VisionServiceClient.AnalyzeImageAsync(imageUrl, visualFeatures);
     return analysisResult;
    }
        // -----------------------------------------------------------------------
        // KEY SAMPLE CODE ENDS HERE
        // -----------------------------------------------------------------------
```
**其他網頁和端點** 透過查看範例中的其他頁面，即可了解如何與「電腦視覺 API」服務所公開的其他端點互動；例如 OCR 端點便顯示在 OCRPage.xaml.cs 所包含的程式碼中 

### <a name="Related">相關主題</a>
 * [開始使用臉部 API](../../Face/Tutorials/FaceAPIinCSharpTutorial.md)
 
 


