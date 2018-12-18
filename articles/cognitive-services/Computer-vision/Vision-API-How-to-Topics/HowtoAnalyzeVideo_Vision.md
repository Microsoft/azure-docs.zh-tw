---
title: 範例：使用電腦視覺 API 進行即時影片分析
titlesuffix: Azure Cognitive Services
description: 了解如何使用電腦視覺 API，針對從即時影片資料流中擷取的畫面，執行近乎即時的分析。
services: cognitive-services
author: KellyDF
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: sample
ms.date: 01/20/2017
ms.author: kefre
ms.openlocfilehash: 058f2ad58665a88d2d3cf3ce20b43ac0fad30000
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45983190"
---
# <a name="how-to-analyze-videos-in-real-time"></a>如何即時分析影片
本指南將示範如何在從即時視訊資料流擷取的畫面上，執行近乎即時的分析。 這類系統中的基本元件如下：

- 從影片來源取得畫面
- 選取要分析的畫面
- 將這些畫面提交給 API
- 取用 API 呼叫所傳回的每一筆分析結果

這些範例是用 C# 撰寫的，而且程式碼可於下列 GitHub 位置取得：[https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) \(英文\)。

## <a name="the-approach"></a>方法
在視訊資料流上執行近乎即時分析時，有多種方式可以解決所遇到的問題。 我們會先舉出其中的三種方法，並從複雜度較低的方法開始。

### <a name="a-simple-approach"></a>簡易方法
針對近乎即時的分析系統，最簡單的設計即是無限迴圈，在其中的每一個反覆項目中，我們會抓取一個畫面、進行分析，然後取用結果：
```CSharp
while (true)
{
    Frame f = GrabFrame();
    if (ShouldAnalyze(f))
    {
        AnalysisResult r = await Analyze(f);
        ConsumeResult(r);
    }
}
```
如果我們的分析包含輕量型用戶端演算法，這個方法就很合適。 不過，當我們的分析是在雲端進行時，所涉及的延遲便意謂著 API 呼叫可能需要數秒鐘的時間。在這段期間內，我們不會擷取影像，而執行緒基本上也不執行任何動作。 因此，我們的畫面播放速率上限將會受限於 API 呼叫的延遲。

### <a name="parallelizing-api-calls"></a>平行處理 API 呼叫
雖然簡易的單一執行緒迴圈很適合用於輕量型的用戶端演算法，但在面對雲端 API 呼叫所涉及的延遲時，則無法很好地因應。 若要解決這個問題，就是讓長時間執行的 API 呼叫與畫面抓取平行執行。 在 C# 中，我們可以使用以工作為基礎的平行處理機制來達到這個目的，例如：
```CSharp
while (true)
{
    Frame f = GrabFrame();
    if (ShouldAnalyze(f))
    {
        var t = Task.Run(async () => 
        {
            AnalysisResult r = await Analyze(f);
            ConsumeResult(r);
        }
    }
}
```
此方法會讓每個分析在個別的工作中進行，而該工作可在我們繼續抓取新畫面時於背景執行。 它可避免在等候 API 呼叫返回時封鎖主執行緒。不過，我們已喪失簡易版本所提供的部分保證，也就是多個 API 呼叫可能會以平行方式進行，而所傳回的結果可能順序不正確。 此方法也可能造成多個執行緒同時進入 ConsumeResult() 函式。如果函式不是安全執行緒，此情況就會相當危險。 最後，這個簡易程式碼不會追蹤所建立的工作，因此例外狀況會以無訊息方式消失。 因此，我們要新增的最後一個要素是「取用者」執行緒。此執行緒會追蹤分析工作、引發例外狀況、終止長時間執行的工作，並確保依正確順序一次取用一個結果。

### <a name="a-producer-consumer-design"></a>「生產者-取用者」設計
在我們的最後一個「產生者-取用者」系統中，會有一個看起來類似於前述無限迴圈的產生者執行緒。 不過，產生者不會在一有分析結果可用時便立即取用這些結果，而是只會將工作置於佇列中來進行追蹤。
```CSharp
// Queue that will contain the API call tasks. 
var taskQueue = new BlockingCollection<Task<ResultWrapper>>();
     
// Producer thread. 
while (true)
{
    // Grab a frame. 
    Frame f = GrabFrame();
 
    // Decide whether to analyze the frame. 
    if (ShouldAnalyze(f))
    {
        // Start a task that will run in parallel with this thread. 
        var analysisTask = Task.Run(async () => 
        {
            // Put the frame, and the result/exception into a wrapper object.
            var output = new ResultWrapper(f);
            try
            {
                output.Analysis = await Analyze(f);
            }
            catch (Exception e)
            {
                output.Exception = e;
            }
            return output;
        }
        
        // Push the task onto the queue. 
        taskQueue.Add(analysisTask);
    }
}
```
我們還有一個取用者執行緒，會從佇列中取出工作、等候這些工作完成，然後顯示結果或引發被擲回的例外狀況。 我們可以使用佇列，以便能確保依正確順序一次取用一個結果，而無須限制系統的最大畫面播放速率。
```CSharp
// Consumer thread. 
while (true)
{
    // Get the oldest task. 
    Task<ResultWrapper> analysisTask = taskQueue.Take();
 
    // Await until the task is completed. 
    var output = await analysisTask;
     
    // Consume the exception or result. 
    if (output.Exception != null)
    {
        throw output.Exception;
    }
    else
    {
        ConsumeResult(output.Analysis);
    }
}
```

## <a name="implementing-the-solution"></a>實作解決方案
### <a name="getting-started"></a>開始使用
為了讓您的應用程式能夠儘快啟動並正常運作，我們已實作上述系統，目的在讓它既有足夠的彈性來實作許多案例，又能操作簡便。 若要存取其程式碼，請前往 [https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis) \(英文\)。

此程式庫包含 FrameGrabber 類別，它會實作上述的「產生者-取用者」系統，以處理來自網路攝影機的影片畫面。 使用者可以指定 API 呼叫的確切形式，而該類別則會使用事件，在取得新畫面或有新分析結果可供使用時，通知呼叫程式碼。

為了說明一些可能性，以下列舉兩個使用此程式庫的範例應用程式。 第一個是一個簡易主控台應用程式，以下是重新產生的簡化版。 它會從預設的網路攝影機抓取畫面，然後提交給「臉部 API」來進行臉部偵測。
```CSharp
using System;
using VideoFrameAnalyzer;
using Microsoft.ProjectOxford.Face;
using Microsoft.ProjectOxford.Face.Contract;
     
namespace VideoFrameConsoleApplication
{
    class Program
    {
        static void Main(string[] args)
        {
            // Create grabber, with analysis type Face[]. 
            FrameGrabber<Face[]> grabber = new FrameGrabber<Face[]>();
            
            // Create Face API Client. Insert your Face API key here.
            FaceServiceClient faceClient = new FaceServiceClient("<subscription key>");

            // Set up our Face API call.
            grabber.AnalysisFunction = async frame => return await faceClient.DetectAsync(frame.Image.ToMemoryStream(".jpg"));

            // Set up a listener for when we receive a new result from an API call. 
            grabber.NewResultAvailable += (s, e) =>
            {
                if (e.Analysis != null)
                    Console.WriteLine("New result received for frame acquired at {0}. {1} faces detected", e.Frame.Metadata.Timestamp, e.Analysis.Length);
            };
            
            // Tell grabber to call the Face API every 3 seconds.
            grabber.TriggerAnalysisOnInterval(TimeSpan.FromMilliseconds(3000));

            // Start running.
            grabber.StartProcessingCameraAsync().Wait();

            // Wait for keypress to stop
            Console.WriteLine("Press any key to stop...");
            Console.ReadKey();
            
            // Stop, blocking until done.
            grabber.StopProcessingAsync().Wait();
        }
    }
}
```
第二個範例應用程式比較有趣，它可讓您選擇要在影片畫面上呼叫的 API。 應用程式的左側會顯示即時影片的預覽，右側則會顯示疊加在對應畫面上的 API 最新結果。

在大多數模式中，左側即時影片與右側視覺化分析之間會有明顯的延遲。 此延遲是進行 API 呼叫所需的時間。 此延遲現象有個例外，就是 "EmotionsWithClientFaceDetect" 模式。此模式會先使用 OpenCV 在用戶端電腦本機上執行臉部偵測，然後才將任何影像提交給「認知服務」。 藉由這個做法，我們便可立即以視覺方式呈現所偵測到的臉部，然後再於稍後 API 呼叫返回時更新表情。 這示範了一個「混合式」方法的可能性，其中可以在用戶端上執行一些簡單的處理，然後再視需要使用「認知服務 API」，以更進階的分析加強這個處理。

![HowToAnalyzeVideo](../../Video/Images/FramebyFrame.jpg)

### <a name="integrating-into-your-codebase"></a>整合至程式碼基底
若要開始使用此範例，請按照以下步驟執行：

1. 從[訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)取得「視覺 API」的 API 金鑰。 若要進行影片畫面分析，適用的 API 包括：
    - [電腦視覺 API](https://docs.microsoft.com/azure/cognitive-services/computer-vision/home)
    - [表情 API](https://docs.microsoft.com/azure/cognitive-services/emotion/home) \(英文\)
    - [臉部 API](https://docs.microsoft.com/azure/cognitive-services/face/overview)
2. 複製 [Cognitive-Samples-VideoFrameAnalysis](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) \(英文\) GitHub 存放庫

3. 在 Visual Studio 2015 中開啟範例，然後建置並執行範例應用程式：
    - 就 BasicConsoleSample 而言，「臉部 API」金鑰會直接以硬式編碼編寫在 [BasicConsoleSample/Program.cs](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/blob/master/Windows/BasicConsoleSample/Program.cs) \(英文\) 中。
    - 就 LiveCameraSample 而言，應該在應用程式的 [設定] 窗格中輸入金鑰。 這些金鑰會以使用者資料的形式跨工作階段保存。
        

當您做好整合準備時，**只需從自己的專案參考 VideoFrameAnalyzer 程式庫**即可。 



## <a name="developer-code-of-conduct"></a>開發人員行為準則
和所有「認知服務」一樣，使用我們的 API 和範例進行開發的開發人員必須遵守 [Microsoft 認知服務的開發人員管理辦法](https://azure.microsoft.com/support/legal/developer-code-of-conduct/) \(英文\)。 


VideoFrameAnalyzer 的影像、語音、影片或文字理解功能會使用「Azure 認知服務」。 Microsoft 會收到您透過此應用程式上傳的影像、聲音、影片和其他資料，而且可能使用它們來改善服務。 當您的應用程式將使用者的資料傳送給「Azure 認知服務」時，請務必協助保護這些使用者。 


## <a name="summary"></a>總結
在本指南中，您已了解如何使用「臉部」、「電腦視覺」和「表情」API，在即時視訊資料流上執行近乎即時的分析，以及如何使用我們的範例程式碼來開始設計程式。 您可以使用 [Microsoft 認知服務註冊頁面](https://azure.microsoft.com/try/cognitive-services/)的免費 API 金鑰來開始建置應用程式。 

歡迎您在 [GitHub 存放庫](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) \(英文\) 中提供意見反應和建議。若要提供更多廣泛的 API 意見反應，請到我們的 [UserVoice 網站](https://cognitive.uservoice.com/) \(英文\)。

