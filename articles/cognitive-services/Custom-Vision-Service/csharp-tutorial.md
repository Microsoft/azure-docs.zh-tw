---
title: 教學課程：使用 C# 建置自訂視覺服務的 Windows 應用程式
titlesuffix: Azure Cognitive Services
description: 建立專案、新增標籤、上傳影像、為您的專案定型，以及使用預設端點進行預測。
services: cognitive-services
author: anrothMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: tutorial
ms.date: 05/03/2018
ms.author: anroth
ms.openlocfilehash: 9e5ed71d4620f7ffeac8acb15f90d67964a86870
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46366636"
---
# <a name="tutorial-use-the-custom-vision-service-from-a-c-application"></a>教學課程：從 C# 應用程式使用自訂視覺服務

了解如何從 C# 應用程式使用自訂視覺服務。 建立它之後，您就可以新增標記、上傳影像、為專案定型、取得專案的預設預測端點 URL，並使用端點以程式設計方式測試影像。 使用自訂視覺服務 API，利用此開放原始碼範例作為範本，來建置您自己適用於 Windows 的應用程式。

## <a name="prerequisites"></a>必要條件

* 適用於 Windows 的任何 Visual Studio 2017 版本。

## <a name="get-the-custom-vision-sdk-and-samples"></a>取得自訂視覺 SDK 與範例
若要建置此範例，您需要自訂視覺 SDK NuGet 套件：

* [Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/)
* [Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/)

您可以將影像與 [C# 範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/CustomVision)一起下載。

## <a name="get-the-training-and-prediction-keys"></a>取得定型與預測金鑰

若要取得此範例中所使用的金鑰，請瀏覽[自訂視覺網頁](https://customvision.ai) \(英文\)，然後選取右上方的__齒輪圖示__。 在 [帳戶] 區段中，複製 [定型金鑰] 和 [預測金鑰] 欄位的值。

![金鑰 UI 的影像](./media/csharp-tutorial/training-prediction-keys.png)

## <a name="understand-the-code"></a>了解程式碼

在 Visual Studio 中，開啟位於 SDK 專案 `Samples/CustomVision.Sample/` 目錄的專案。

此應用程式會使用您稍早擷取的定型金鑰，來建立名為 __My New Project__ 的新專案。 接著，它會上傳影像以便為分類器定型並進行測試。 分類器會識別樹是 __Hemlock__ 還是 __Japanese Cherry__。

下列程式碼片段會實作此範例的主要功能：

* __建立新的自訂視覺服務專案__：

    ```csharp
     // Create a new project
    Console.WriteLine("Creating new project:");
    var project = trainingApi.CreateProject("My New Project");
    ```

* __在專案中建立標記__：

    ```csharp
    // Make two tags in the new project
    var hemlockTag = trainingApi.CreateTag(project.Id, "Hemlock");
    var japaneseCherryTag = trainingApi.CreateTag(project.Id, "Japanese Cherry");
    ```

* __上傳和標記影像__：

    ```csharp
    // Add some images to the tags
    Console.WriteLine("\tUploading images");
    LoadImagesFromDisk();

    // Images can be uploaded one at a time
    foreach (var image in hemlockImages)
    {
        using (var stream = new MemoryStream(File.ReadAllBytes(image)))
        {
            trainingApi.CreateImagesFromData(project.Id, stream, new List<string>() { hemlockTag.Id.ToString() });
        }
    }

    // Or uploaded in a single batch 
    var imageFiles = japaneseCherryImages.Select(img => new ImageFileCreateEntry(Path.GetFileName(img), File.ReadAllBytes(img))).ToList();
    trainingApi.CreateImagesFromFiles(project.Id, new ImageFileCreateBatch(imageFiles, new List<Guid>() { japaneseCherryTag.Id }));
    ```

* __為分類器定型__：

    ```csharp
    // Now there are images with tags start training the project
    Console.WriteLine("\tTraining");
    var iteration = trainingApi.TrainProject(project.Id);

    // The returned iteration will be in progress, and can be queried periodically to see when it has completed
    while (iteration.Status == "Completed")
    {
        Thread.Sleep(1000);

        // Re-query the iteration to get it's updated status
        iteration = trainingApi.GetIteration(project.Id, iteration.Id);
    }
    ```

* __針對預測端點設定預設反覆項目__：

    ```csharp
    // The iteration is now trained. Make it the default project endpoint
    iteration.IsDefault = true;
    trainingApi.UpdateIteration(project.Id, iteration.Id, iteration);
    Console.WriteLine("Done!\n");
    ```

* __建立預測端點__：
 
    ```csharp
    // Create a prediction endpoint, passing in obtained prediction key
    PredictionEndpoint endpoint = new PredictionEndpoint() { ApiKey = predictionKey };
    ```
 
* __將影像傳送到預測端點__：

    ```csharp
    // Make a prediction against the new project
    Console.WriteLine("Making a prediction:");
    var result = endpoint.PredictImage(project.Id, testImage);

    // Loop over each prediction and write out the results
    foreach (var c in result.Predictions)
    {
        Console.WriteLine($"\t{c.TagName}: {c.Probability:P1}");
    }
    ```

## <a name="run-the-application"></a>執行應用程式

1. 進行下列變更，將定型金鑰和預測金鑰新增到應用程式：

    * 將您的__定型金鑰__新增到下一行：

        ```csharp
        string trainingKey = "<your key here>";
        ```

    * 將您的__預測金鑰__新增到下一行：

        ```csharp
        string predictionKey = "<your key here>";
        ```

2. 執行應用程式。 當應用程式執行時，會將下列輸出寫入主控台：

    ```
    Creating new project:
            Uploading images
            Training
    Done!

    Making a prediction:
            Hemlock: 95.0%
            Japanese Cherry: 0.0%
    ```

3. 按任意鍵以結束應用程式。
